---
title: "Google ADK Architecture: Building AI Agent Systems with Software Engineering Principles"
date: 2026-02-27T23:01:12+08:00
draft: false
categories: ["AI Agent"]
description: "A translated technical note on Google ADK Architecture: Building AI Agent Systems with Software Engineering Principles, preserving the examples and context of the original article."
---
# Google ADK Architecture: Building AI Agent Systems with Software Engineering Principles

> Originally published in Chinese on 2026-02-27; this English edition preserves the original scope and technical context.

ADK (Agent Development Kit) is an AI agent development framework that Google open-sourced in 2025. It has more than 18,000 GitHub stars and implementations for Python, TypeScript, Go, and Java. Its design goal is explicit: bring agent development back into the discipline of software engineering instead of leaving it at the prompt-engineering stage. That requires clear abstraction layers, composable modules, deterministic execution flows, and production-grade state management.

Current frameworks in the AI Agent domain exhibit a bifurcated trend: one end is LangChain, a highly flexible but loosely coupled framework, and the other end is the managed services from various cloud vendors. ADK aims to find a balance between these two extremes—maintaining the flexibility of code-first while providing sufficient structured constraints to support complex multi-Agent systems. Whether this positioning holds up requires a thorough dissection at the architectural level.

Before delving into the details, let's first understand the overall hierarchical structure of ADK from a macro perspective:
```
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                          │
│              adk web / adk run / adk api_server                 │
├─────────────────────────────────────────────────────────────────┤
│                       Runner (Event Loop)                       │
│         yield/pause/resume cycle ── Event processing            │
├───────────────┬───────────────┬─────────────────────────────────┤
│  Agent Layer  │   Tool Layer  │         Flow Layer              │
│  BaseAgent    │   BaseTool    │     BaseLlmFlow                 │
│  ├ LlmAgent   │  ├ FunctionTool│     ├ SingleFlow               │
│  ├ Sequential │  ├ AgentTool  │     └ AutoFlow                  │
│  ├ Parallel   │  ├ MCPTool    │       (LLM request/response     │
│  ├ Loop       │  └ ...        │        + tool execution loop)   │
│  └ Custom     │               │                                 │
├───────────────┴───────────────┴─────────────────────────────────┤
│                      Model Abstraction                          │
│         BaseLlm ── LLMRegistry ── Gemini / OpenAI / ...        │
├─────────────────────────────────────────────────────────────────┤
│                      Services Layer                             │
│    SessionService  │  ArtifactService  │  MemoryService         │
│    (state + history)  (binary blobs)     (cross-session search) │
├─────────────────────────────────────────────────────────────────┤
│                      Infrastructure                             │
│        InMemory / Database / GCS / Vertex AI Agent Engine       │
└─────────────────────────────────────────────────────────────────┘
```
## One-Agent Architecture: From Single Abstraction to Three-Level Categorization

### 1.1 BaseAgent as a Unified Foundation

Bottom-up, the Infrastructure layer provides pluggable storage backends; the Services layer manages the persistence of state, artifacts, and memory; the Model layer abstracts the differences between various LLM providers; the Agent/Tool/Flow layers constitute the core execution logic; the Runner layer drives the entire event loop; the Application layer then provides interfaces for developers and end-users.

## 1.1 BaseAgent as a Unified Foundation

The ADK Agent framework is built on top of the `BaseAgent` base class, from which all types of agents inherit. `BaseAgent` itself inherits from `pydantic BaseModel`, meaning that each agent instance is essentially a data model that can be serialized and validated—rather than traditional objects driven by behavior. This is a noteworthy design choice: the introduction of `pydantic` ensures that agent configurations are inherently type-checked, default values are filled in, and JSON serialization is possible, laying the groundwork for declarative configuration (Agent Config).

`BaseAgent` defines several key interface contracts. `run_async` is the unified entry method for the Agent. It is marked with the `@final` decorator to prevent overriding, allowing subclasses to inject their execution logic by implementing `_run_async_impl`. The use of this template method pattern ensures that the framework can fix the handling of callbacks and tracing within `run_async`, leaving the points of change for core business logic to subclasses. A clear structure of this implementation can be seen from the source code of `run_async`.
```python
@final
async def run_async(self, parent_context: InvocationContext) -> AsyncGenerator[Event, None]:
    with tracer.start_as_current_span(f'invoke_agent {self.name}') as span:
        ctx = self._create_invocation_context(parent_context)
        if event := await self._handle_before_agent_callback(ctx):
            yield event
            if ctx.end_invocation:
                return
        async with Aclosing(self._run_async_impl(ctx)) as agen:
            async for event in agen:
                yield event
        if event := await self._handle_after_agent_callback(ctx):
            yield event
```
The return type of `run_async` is `AsyncGenerator[Event, None]`. This is no coincidence—async generators form the foundation of the entire runtime event loop, which will be discussed in more detail later.

Between agents, the relationship is established through the `sub_agents` field, forming an agent tree. During the `model_post_init` phase, the framework automatically sets the `parent_agent` reference for each child agent and enforces the singleton parent rule: an agent instance can only be added as a child of one parent agent. To reuse the same logic, a new instance must be created using the `clone` method. This constraint avoids the management difficulties that arise when the agent tree degenerates into a DAG or more complex topologies—managing context and lifecycle in a multi-agent system becomes extremely complicated when an agent is simultaneously involved in two execution paths.

### 1.2 Three Types of Agent Classification Logic

On `BaseAgent`, ADK categorizes Agents into three types: LLM Agents, Workflow Agents, and Custom Agents. This categorization is essentially a division based on the determinism of the execution logic of the Agents.
```
                         BaseAgent
                        (pydantic BaseModel)
                             │
              ┌──────────────┼──────────────────┐
              ▼              ▼                   ▼
          LlmAgent      Workflow Agents      Custom Agent
       (non-deterministic)  (deterministic)   (user-defined)
              │              │
              │    ┌─────────┼──────────┐
              │    ▼         ▼          ▼
              │  Sequential  Parallel   Loop
              │  Agent       Agent      Agent
              │
         ┌────┴────┐
         ▼         ▼
    BaseLlmFlow   Tools
    (LLM calls)   (FunctionTool, AgentTool, ...)
```
LLM Agent (henceforth referred to as `LlmAgent`, with an alias `Agent`) is an agent whose behavior is non-deterministic. Given the same input, an LLM may produce different responses, choose different tools, or delegate control to different sub-Agents. The configuration options of `LlmAgent` are extensive, including model selection (`model`), system instructions (`instruction`), tool lists (`tools`), and output keys (`output_key`). Almost all parameters related to interactions with an LLM are defined on this class. Notably, the inheritance mechanism of the `model` field: if an `LlmAgent` does not explicitly specify a model, it searches up the Agent tree for ancestor nodes' model configurations. If no model is specified throughout the tree, it falls back to the class-level default model. This inheritance strategy reduces redundancy in model configurations across multi-Agent systems.

Workflow Agents are deterministic orchestrators, comprising `SequentialAgent`, `ParallelAgent`, and `LoopAgent`. They do not invoke LLMs but are responsible for scheduling the execution of sub-Agents according to predefined patterns (sequential, parallel, loop). For instance, the `_run_async_impl` of `SequentialAgent` is straightforward: it sequentially calls the `run_async` method of each sub-Agent in the `sub_agents` list, yielding the events one by one. Sub-Agents communicate through a shared session state – the previous Agent writes results into a state's key, and the next Agent reads from the template variables of the instruction.

The utility of this three-tier classification lies in finding the appropriate boundary between "allowing LLM to make autonomous decisions" and "hard-coding a workflow with code." Workflow Agents provide a middle ground: the macro structure of the workflow is deterministic, but each step can be driven by LLMs. For example, a content review pipeline can be concatenated with `SequentialAgent` for tasks such as "extract key information" → "risk assessment" → "generate report," ensuring the overall workflow does not deviate due to LLM hallucinations, while each step can fully leverage LLM capabilities.
```python
extract = LlmAgent(name="Extract", model="gemini-2.5-flash",
instruction="Extract key entities and events from the text", output_key="entities")
assess  = LlmAgent(name="Assess", model="gemini-2.5-flash",
instruction="Based on {entities} evaluate the risk level", output_key="risk")
report  = LlmAgent(name="Report", model="gemini-2.5-flash",
Generate a review report based on the risk level {risk} as per Marcus.

pipeline = SequentialAgent(name="ContentReview", sub_agents=[extract, assess, report])
```
### 1.3 Two Paradigms of Multi-Agent Collaboration

In ADK, there are two fundamentally different paradigms of collaboration among multi-Agent: LLM-Driven Dynamic Delegation (Agent Transfer) and Workflow Agent-Driven Static Orchestration.
```
Dynamic Delegation (Agent Transfer)              Static Orchestration (Workflow Agent)
┌──────────────────────┐              ┌──────────────────────┐
│   Coordinator (LLM)  │              │   SequentialAgent    │
│                      │              │                      │
│  "I need weather     │              │   step1 ──► step2    │
│   info, let me       │              │              │       │
│   transfer to..."    │              │              ▼       │
│         │            │              │            step3     │
│    transfer_to_agent │              │  (deterministic      │
│         │            │              │   execution order)   │
│         ▼            │              │                      │
│   WeatherAgent (LLM) │              └──────────────────────┘
│   "The temp is..."   │
│         │            │              ┌──────────────────────┐
│      escalate        │              │   ParallelAgent      │
│         │            │              │                      │
│         ▼            │              │  ┌─────┐  ┌─────┐   │
│   Coordinator (LLM)  │              │  │ A   │  │ B   │   │
│   continues...       │              │  └─────┘  └─────┘   │
└──────────────────────┘              │  (concurrent exec)   │
                                      └──────────────────────┘
```
In the dynamic delegation mode, a list of sub-Agents of an `LlmAgent` is automatically converted into `AgentTool` descriptions and injected into the LLM prompt of the parent Agent. When the LLM determines that a particular task should be handled by one of the sub-Agents, it issues a `transfer_to_agent` function call, which the framework captures and transfers control to the target Agent. After the transfer of control, the context of the original Agent is temporarily stored, and the target Agent begins executing within its own context until it explicitly escalates control back. All events generated during this period are recorded in the event history of the same session, ensuring the integrity of the conversation history.

In the static orchestration mode, the implementation of `ParallelAgent` stands out. For each sub-Agent, it creates a separate branch (by modifying the `InvocationContext.branch` field). Although multiple sub-Agents share the same session state, the isolation of branches ensures that the event history is logically separated. However, since the state is shared, concurrent branches that write to the same key can lead to race conditions—developers are explicitly warned to use different state keys for parallel branches in the documentation.

These two paradigms can be nested and combined. A typical pattern involves using a `SequentialAgent` to define a pipeline, where one step of the pipeline is an `LlmAgent`, which in turn has multiple sub-Agents that can be dynamically delegated. This combination allows the system to have a deterministic backbone with flexible periphery.

## 2 Runtime Engine: Event Loop and State Management

### 2.1 Coroutine-based Event Loop

The core runtime of ADK is an event loop, operating in a highly similar manner to Python's generator coroutines. The `Runner`, as the orchestrator, and the `Agent` (along with its associated tools and callbacks) serve as the execution logic, collaborating through the `Event` object.
```
User Query
    │
    ▼
┌─────────┐     yield Event      ┌──────────────────┐
│  Runner  │◄────────────────────│  Agent Logic      │
│          │                     │  (AsyncGenerator) │
│ 1.receive│                     │                   │
│ 2.commit │─── resume ─────────►│ yield event_1     │
│   state  │                     │ # ── pause ──     │
│ 3.forward│◄────────────────────│                   │
│   to UI  │                     │ # ── resumed ──   │
│ 4.resume │─── resume ─────────►│ yield event_2     │
│          │                     │ # ── pause ──     │
│          │◄────────────────────│                   │
│ ...      │                     │ ...               │
│          │                     │ (generator done)  │
└─────────┘                      └──────────────────┘
```
Executing logic constructs an `Event` every time it needs to report results externally, invoke a tool, or submit state changes. It passes the `Event` back to the `Runner` via `yield`. Upon receiving the `Event`, the `Runner` persists the side effects (e.g., `state_delta`, `artifact_delta`) within the session through `SessionService`. It then forwards the event to its upstream (typically UI or API layer) and finally notifies the execution logic to continue. When resuming execution, the logic can be assured that the state changes yielded previously have been reliably submitted.

This `yield/pause/resume` loop and the working principle of Python asynchronous generators are the same — the `_run_async_impl` method of the Agent is an `AsyncGenerator`, where each `yield` acts like a coroutine's suspension point, and the `Runner`'s `async for` consumption of events acts like the driving side of a coroutine. If one has understood the implementation of generators in CPython (where the `YIELD_VALUE` instruction retrieves the top-of-stack data, modifies the frame pointer, and uses `goto` to exit the loop), one can understand the origins of this ADK design concept — merely swapping the role of the stack frame with `InvocationContext`, and replacing bytecode execution with the business logic of the Agent.

Compared to callback chains or message queues, AsyncGenerator offers two clear advantages as the execution model: firstly, the execution logic can be written in a seemingly synchronous linear flow, without needing to split it into multiple callback functions or state machine states; secondly, `yield` provides a natural synchronous point, making the sequential semantic of state submission clear and predictable.

### 2.2 Event: Runtime Information Carrier

`Event` is a core data structure in the ADK runtime. Each `Event` consists of fields such as `invocation_id` (identifying the user request invocation), `author` (the name of the Agent that produced the event), `branch` (branch identifier for execution), `content` (carrier of the event's content), and `actions` (declarations of side effects carried by the event).

The `content` field adheres to the `types.Content` structure of the Gemini API, which can encompass various types of `Part`, including text, function calls, and function responses. This means an `Event` can express semantics such as "Agent said a sentence", "Agent requested the invocation of a tool", and "Tool returned a result".
The `actions` field (of type `EventActions`) is crucial for understanding the ADK state management model. It includes fields such as `state_delta` (a dictionary describing the incremental changes to the session state), `artifact_delta` (records of binary artifact changes), `transfer_to_agent` (the target Agent for transferring control), and `escalate` (escalating control to a higher level). State changes are not made directly to the session object but are declared in the `actions` of the `Event`, unified by the `Runner` during the event processing stage. This "declarative side effects" design pattern is common in event sourcing architectures, making each state change traceable and replayable. A typical state change workflow looks like this:
```python
# Agent's Internal Execution Logic
ctx.session.state['status'] = 'processing'
event = Event(
    author=self.name,
    actions=EventActions(state_delta={'status': 'processing'}),
   content=types.Content(parts=[types.Part(text="start processing")])
)
yield event
# --- Pause, Runner submits `state_delta` to `SessionService` ---
# --- Resume execution, `state['status']` is now reliably persisted ---
current = ctx.session.state['status']  # 'processing'
```
Here is an interesting trade-off: ADK allows for what we call "dirty reads" (dirty read). Within a single invocation, a callback modifies the state, and subsequent tools can immediately read this modification even if the associated Event has not yet been formally submitted by the `Runner`. This improves coordination efficiency between different components within the same execution step, but it introduces a consistency risk: if the invocation terminates abnormally before submission, the modified state read during this invocation will be lost. The documentation cautions developers against relying on dirty reads for critical state transitions, which is a pragmatic but somewhat compromising approach.

### 2.3 A Three-Level Context Model of Session, State, and Memory
```
┌───────────────────────────────────────────────────────────────┐
│                     Memory (MemoryService)                     │
│          Cross-session Long-term Semantic Memory, Supports Search and Retrieval          │
│          scope: Across All Sessions                                      │
├───────────────────────────────────────────────────────────────┤
│                     Session (SessionService)                   │
│    ┌─────────────────────┐  ┌──────────────────────────────┐  │
│    │    Event History     │  │         State (dict)         │  │
│    │  [event_0, event_1,  │  │  "app:config"  → global     │  │
│    │   event_2, ...]      │  │  "user:pref"   → per-user   │  │
│    │  (ordered log of     │  │  "cart_items"   → session    │  │
│    │   all interactions)  │  │  "temp:scratch" → invocation │  │
│    └─────────────────────┘  └──────────────────────────────┘  │
│          scope: Single conversation                           │
├───────────────────────────────────────────────────────────────┤
│                     Invocation                                 │
│          Process for a single user request                           │
│          scope: Single request (temp: State of prefix is valid only within this scope)   │
└───────────────────────────────────────────────────────────────┘
```
Session represents a complete conversation thread. It contains the event history (a sequential list of all Events), the current state data (state), and session-level metadata. Each time a user initiates a request, the `Runner` loads the corresponding Session from the `SessionService`, adds the user input as the first Event to the history, and then starts the execution of the Agent. The lifecycle of the Session is managed by the `SessionService`, and ADK provides an `InMemorySessionService` (for development testing) and a persistent implementation based on database or cloud services.

State is the key-value data stored in a Session. In addition to the regular state, ADK introduces state keys prefixed with `app:` and `user:` to represent application-level state and user-level state, respectively, and prefixed with `temp:` for temporary state (valid only for the current invocation). This prefix convention is a lightweight namespace mechanism to avoid introducing a more complex multi-layer state store design.

Memory is long-term memory across sessions, managed by the `MemoryService`. It supports ingesting the content of historical Sessions into the memory store and retrieving them via semantic search. The responsibilities of Memory and Sessions are distinct: Sessions focus on "what was said in the current conversation," while Memory focuses on "what is worth remembering from past conversations."

These three tiers cover the typical data-lifetime requirements in an agent system: invocation-level (`temp state`), session-level (`regular state + event history`), and cross-session-level (`memory + app/user state`). Developers must decide where information belongs based on their needs; the framework does not impose stronger constraints or offer much guidance, which may be an area for future improvement.

## 3 Tool System: From Function to Ecosystem

### 3.1 Tool Abstract Layer
The ADK tool system is built around the `BaseTool` base class. The most common tool type is `FunctionTool`, which wraps a regular Python function to make it callable as a tool by the Agent. The framework generates the schema description for the tool based on the function signature's type annotations and passes it to the LLM for function calls. This means that developers need to write a function with type annotations and docstrings to make the Agent callable — the development experience is very low.

On `FunctionTool`, ADK also provides `AgentTool` (a tool that wraps one Agent into another), `LongRunningFunctionTool` (a tool that supports asynchronous long-running tasks). The design of `AgentTool` is particularly noteworthy: when the `sub_agents` of an `LlmAgent` are handled by the framework, each sub-Agent is implicitly wrapped by `AgentTool`, making the LLM of the parent Agent able to trigger execution of the sub-Agent through a function call. This "Agent as Tool" unified abstraction eliminates the semantic gap between inter-Agent calls and Agent tool calls.

The execution flow of the tool is embedded with a rich set of callback hooks: `before_tool_callback` triggers before the tool execution, allowing for interception or modification of the tool's input parameters; `after_tool_callback` triggers after the tool returns, allowing for modification or replacement of the tool's output. These hooks enable clean injection of cross-cutting logic such as logging, permission checks, and input/output validation without modifying the tool's implementation.

### 3.2 Tool Confirmation Mechanism and Security Considerations

ADK introduces a tool confirmation mechanism to implement Human-in-the-Loop control. When a tool is marked as needing confirmation, the Agent does not immediately invoke the tool upon receiving a request; instead, it first sends a confirmation request to the user for approval before proceeding. This mechanism is critical in scenarios involving external side effects (sending emails, modifying databases, executing transactions, etc.).

From a security perspective, ADK faces deeper challenges. Within the Agent system, the output of LLM is directly used to drive tool invocation and control flow transfer between Agents. This means that prompt injection attacks can directly affect the execution path of the system. ADK sets up a dedicated security chapter in its documentation to discuss these issues. However, objectively, the current protective measures are still mainly at the level of best practices and architectural recommendations, lacking a fully automated security defense.

## 4 LLM Interaction Layer: Design of Flow
### 4.1 BaseLlmFlow and Request/Response Pipeline

`LlmAgent` does not directly interact with the LLM API but rather manages the entire process through the `BaseLlmFlow` intermediary layer. The responsibilities of the Flow include converting the session's event history into a list of messages that the LLM can consume (content assembly), constructing the LLM request (including system instructions and tool declarations), invoking the LLM API, handling the LLM response (text output or function call), and executing tools when needed and providing the results back to the LLM.

The core loop of this Flow can be summarized as follows:
```
                    ┌──────────────────┐
                    │  Content Assembly │
                    │  (history → msgs) │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │   Call LLM API    │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
               ┌────│ Response Type?   │────┐
               │    └──────────────────┘    │
          text │                            │ function_call
               ▼                            ▼
    ┌──────────────────┐         ┌──────────────────┐
    │ yield text Event  │         │  Execute Tool     │
    │ (final response)  │         │  yield FC Event   │
    └──────────────────┘         └────────┬─────────┘
                                          ▼
                                 ┌──────────────────┐
                                 │ yield FR Event    │
                                 │ (function result) │
                                 └────────┬─────────┘
                                          │
                                          └──── loop back ────►
```
Every LLM response and tool invocation result is packaged as an Event and yielded, ensuring that the `Runner` can track and persist the intermediate states of each step.

The Flow layer also handles streaming responsibilities. When using the streaming LLM API, the LLM's responses are progressively returned in units of tokens. The Flow packages these partial responses as `partial=True` events, which the `Runner` forwards directly to the UI for streaming display. However, no state commits are executed on these partial events—only the complete response triggers the persistence of the state. This distinction ensures the atomicity of the state is not compromised by streaming.

### 4.2 Model Abstraction and Multi-Model Support

ADK uses the `BaseLlm` abstract class and `LLMRegistry` to achieve model interchangeability. Although the framework is deeply optimized for Google's Gemini series models (using the `google-genai` SDK), developers can integrate any LLM by implementing the `BaseLlm` interface. `LLMRegistry` is a global registry that maps model name strings (such as `"gemini-2.5-flash"` or `"openai/gpt-4o"`) to the corresponding `BaseLlm` implementation class. The `model` field of `LlmAgent` accepts either a string or a `BaseLlm` instance, with the latter being resolved at runtime by the Registry.

This Registry Pattern allows model switching to occur without modifying the Agent code – merely registering different model implementations during initialization. However, this also means that developers must themselves ensure compatibility for differences in capabilities (such as support for function calling, context window size, multimodal capabilities, etc.). The framework does not perform capability checks at compile time or initialization.

## Five Deployment and Observability

ADK supports multi-layered run modes from local development to production deployment. `adk web` starts a development server with a Web UI, allowing direct interaction and debugging with the Agent in a browser; `adk run` provides command-line interaction mode; `adk api_server` exposes RESTful APIs for external system integration. In production environments, the Agent can be containerized and deployed to Cloud Run, or managed and scaled using Vertex AI Agent Engine.
## Observability

ADK provides distributed tracing capabilities through the integration of OpenTelemetry. A tracing span is created at the entry point of `BaseAgent.run_async`, ensuring that each call of an Agent is represented as a complete chain in the tracing system. This is crucial for debugging execution paths and performance bottlenecks in multi-Agent systems.

## Evaluation

ADK also offers a valuable infrastructure called evaluation. Using the `adk eval` command, developers can assess the performance of Agents against predefined test sets, including the quality of final responses and the correctness of step-by-step execution paths. This enables the establishment of a continuous integration and regression testing process for Agent development, similar to traditional software.

## Design Trade-offs and Areas for Improvement

Several design decisions in ADK warrant further discussion.

**Firstly, the choice of pydantic as the base for Agents.** pydantic offers excellent configuration validation and serialization capabilities but introduces significant runtime overhead. In high-throughput Agent services, the pydantic validation at the instance creation and event creation stages could become a performance bottleneck. This needs to be benchmarked under actual workload conditions.

**Secondly, the concurrency safety model for session state.** In the `ParallelAgent` scenario, multiple sub-Agents share the same state dictionary. The framework leaves the responsibility of avoiding race conditions entirely to the developer (via different keys). This approach works well in small-scale systems but becomes problematic in complex multi-Agent systems where managing key namespaces manually can lead to errors. One possible improvement direction is to introduce domain-level state isolation for Agents or provide a conflict-resolution mechanism similar to CRDTs.

**Thirdly, the control flow semantics of Agent Transfer.** Currently, Agent Transfer is triggered by function calls from the LLM, meaning the occurrence of Transfer depends on the LLM's judgment rather than the developer's explicit code. In scenarios requiring precise control, this non-determinism may not be ideal. ADK provides a deterministic alternative through Workflow Agents but the choice and combination of these two approaches require some learning cost for new users of the framework.
Finally, ADK, as a relatively young framework (open-sourced in 2025), is still rapidly expanding its ecosystem. The integrated list currently covers the Google Cloud suite, multiple mainstream MCP tools, and observability solutions. However, there is still room for interoperability across frameworks (such as interoperability with LangChain and CrewAI Agents). The introduction of the A2A (Agent-to-Agent) protocol is a step in this direction, but it still has a long way to become a standard in the industry.

Overall, the architecture design of ADK reflects Google's engineering experience in large-scale distributed systems—clear abstraction layers, event-based decoupling design, declarative side-effect management, and pluggable service backends. It is not a one-size-fits-all framework but a product that finds its place between "structure" and "flexibility" from an engineering perspective. For teams building production-grade Agent systems, ADK provides a worthy starting point.
