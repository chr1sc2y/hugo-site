---
title: "Claude Code Architecture: Four Core Designs of Modern AI Agents"
date: 2026-04-01T10:00:00+08:00
draft: false
categories: ["AI Agent"]
description: "A translated technical note on Claude Code Architecture: Four Core Designs of Modern AI Agents, preserving the examples and context of the original article."
---
# Claude Code Architecture: Four Core Designs of Modern AI Agents

> Originally published in Chinese on 2026-04-01; this English edition preserves the original scope and technical context.

Claude Code, Anthropic's AI programming agent released in February 2025 and internally known as **Tengu**, evolved rapidly from its first v0.2 beta release to v2.1.88. Over that period it introduced background agents, an automatic permission mode, MCP server integration, and multi-agent collaboration through Agent Teams. Unlike LangChain and Google ADK, which present themselves as frameworks, Claude Code is an end-user product. It does not primarily offer an SDK for building other agents; it acts as a programming partner in the terminal. That positioning creates a different set of architectural constraints: framework-level extensibility and composability matter less, while security, context management, streaming interaction, and error recovery must meet production standards.

On March 29, 2026, the complete TypeScript source code of Claude Code was accidentally leaked due to a configuration error during an npm release. Security researcher Chaofan Shou discovered that Anthropic, while updating the version to v2.1.88, left behind the `.map` file (Source Map) in the build artifacts. This file pointed to an uncompressed source code archive stored in Cloudflare R2. Multiple GitHub mirror repositories quickly backed up the code. Although Anthropic quickly updated and removed the old version, the source code was no longer recoverable.

The leaked code repository contains over 2,000 files and over 500,000 lines of TypeScript code, encompassing the complete agent loop, implementations of over 40 tools, permission pipelines, context compression strategies, and 44 internal feature flags behind unpublished features. This accidental leak provides a rare opportunity to examine the internal design of industrial-grade AI agents at the source code level. Anthropic's core design philosophy in Claude Code is "Less scaffolding, more model," meaning that the system complexity is transferred from orchestration layers to the model itself. This means there are no DAGs, classifiers, or RAG systems; the core of the architecture is a simple inference-execution loop.

## 1 The New Work Paradigm of AI Agents and Programmers

### 1.1 From Code Completion to Autonomous Execution
2023 Before, the impact of AI on the workflow of programmers mainly resided at the "completion" level. GitHub Copilot generates code snippets at the cursor position, and developers review each line to decide whether to accept them. This interactive mode is fundamentally human-driven: developers are responsible for breaking down tasks, determining implementation paths, and writing code skeletons, with AI providing acceleration on local details.

Since 2024, programming AI tools are undergoing a paradigm shift from Copilot to Agent. Products like Cursor, Windsurf, and Claude Code no longer wait for developers to indicate each line. Instead, they accept a high-level goal ("refactor the database access layer of this module") and autonomously search the codebase, read files, write code, run tests, and fix errors until the task is complete or a human-in-the-loop is needed. The core of this transformation lies in the Agent's ability to perform an autonomous reasoning-action loop, breaking down a vague goal into specific step sequences and dynamically adjusting the plan based on feedback during execution. In a five-month internal experiment, OpenAI generated approximately 100,000 lines of code using Codex Agent with just three engineers, shifting the engineers' role from "writing code" to "designing the environment, describing intentions, and building feedback loops." This transformation is redefining the content of software engineering jobs.

### 1.2 The Four Core Dimensions of Programming Agents

Understanding the architecture design of a programming Agent can be approached through four dimensions. These four dimensions are not unique to Claude Code but are the consensus framework in the current AI Agent domain, with different Agent products differing in their design choices on each dimension, which constitute the core differences between them.

The first dimension is **reasoning and planning**. How does the Agent convert the user's high-level goal into executable step sequences? What is the core reasoning loop of the Agent? How does it adjust the plan when encountering unexpected situations during execution? Claude Code, Cursor, and OpenAI Codex all use some form of ReAct loop as the core of reasoning, but there are significant differences in terms of complexity and model trust.

The second dimension is **memory**. How does the Agent manage its limited context window? When the conversation history and tool results keep accumulating, approaching the token limit, how does the system decide what to retain, discard, or compress? Claude Code employs a five-layer progressive compression pipeline, while Cursor relies on codebase indexing and embedding retrieval. These strategies reflect different answers to the question of "how an Agent should understand the codebase."

The third dimension is **tool usage**. How does the Agent interact with the external environment? How is the tool system defined, how are multiple tools executed in parallel, and how are errors handled during execution?
The fourth dimension is **perception and safety**. How does an agent assess the risk of its actions? How does an agent balance autonomy and safety when it can execute arbitrary Bash commands on a user's machine?

The overall architecture of Claude Code consists of several key components including the user interface, natural language processing engine, and backend database.

Before diving into a layer-by-layer analysis of Claude Code's structure, let's have an overall macroscopic understanding of its hierarchical structure:
```
┌─────────────────────────────────────────────────────────────────┐
│                       User Interface Layer                                  │
│       REPL (Ink/React) │ SDK / Headless Mode │ Remote Mode (WebSocket)   │
├─────────────────────────────────────────────────────────────────┤
│ Query Engine │ Session State │ Message Lifecycle │ SDK Gateway │
├─────────────────────────────────────────────────────────────────┤
│                       Agent Loop                                  │
│ Prepare Context → Call Model → Execute Tool → Recursion           │
├──────────────┬──────────────┬───────────────────────────────────┤
│   Tool System    │   Authority Pipeline    │          Context Management                 │
│  Interface Contract     │  Seven Step Judgement     │  Automatic Compression │ Micro Compression                  │
│  Dynamic Orchestration     │  Pluggable Functions   │  Historical Truncation │ Context Folding              │
│  Stream Executor   │  Hook System   │  Budget for Tool Results                       │
│              │              │  Session Memory                           │
├──────────────┴──────────────┴───────────────────────────────────┤
│                       API Communication Layer                                  │
│          Stream Transmission │ Retry/ Degradation │ Model Routing                         │
├─────────────────────────────────────────────────────────────────┤
│                       Infrastructure Layer                                  │
│      Global State │ Analytics │ MCP │ OAuth │ Configuration Management                  │
└─────────────────────────────────────────────────────────────────┘
```
## 2 Reasoning and Planning

### 2.1 ReAct Model and Design Choice of Single Loop

Modern AI Agent's inference core largely follows the **ReAct** (Reasoning + Acting) paradigm: in each interaction round, the model first reasons (generates thought process and action plan), then executes the action (invokes a tool), and finally reasons again based on the action's outcome. This "think-do-observe" cycle repeats until the model judges the task complete. The official technical documentation of OpenAI Codex calls this cycle an "agent loop," with its core structure matching Claude Code: receive input → generate response or tool call → execute tool → append result to prompt → loop until a final response is generated.

When examining the Runnable pipeline of LangChain, the event-driven Agent tree of Google ADK, or the multi-role dialogue system of AutoGen, we find that these frameworks introduce additional orchestration mechanisms atop the ReAct loop. Claude Code makes an entirely different choice: all orchestration logic is concentrated in a single loop of a single module, rejecting the introduction of DAGs, state machines, or graph execution engines. This is not a decision about coding style but about the **allocation of complexity budget**: explicit structuring of orchestration logic is replaced with implicit trust in the model's inference capabilities. System correctness is more dependent on the consistency of the model's behavior rather than the structural guarantees of the code.

The core inference loop of Claude Code implements the full semantic of the ReAct framework: each iteration first goes through five layers of contextual compression (which will be expanded in Chapter 3), then initiates a streaming model invocation. If a tool invocation request is included in the model response, the tool is invoked and its result is appended to the context, starting the next iteration. When the model no longer requests tool invocation, the loop terminates. The entire process is implemented asynchronously as a generator, allowing each step in the loop to push events upstream (stream text, tool progress, compression notifications) to the upstream. This design aligns with Google ADK's design for the Agent base class, which returns an asynchronous event stream without introducing additional event buses or publish-subscribe mechanisms.
### 2.2 A Single Inference Round Lifecycle

A single inference round begins with **context preparation**. The system first extracts the compressed boundary from the message list, then sequentially goes through the tool result budget clipping, historical truncation, micro compression, context folding, and automatic compression (details of each layer's mechanism are presented in Chapter 3). The prepared message list then undergoes **system prompt assembly**: the system prompt is split into **static segments** (introduction, system rules, task description, operation guide, tool description, tone requirement) and **dynamic segments** (memory, MCP instructions, draft board), separated by an explicit buffer boundary. Static segments can be cached and reused between API calls, while dynamic segments need to be rebuilt each time. This prompt cache boundary design directly impacts API costs: when a static segment is hit in the cache, subsequent calls only need to pay for the token of the dynamic segment.

After context preparation, the system initiates a streaming model call through the API communication layer. This involves a common economic trade-off in AI Agent systems: **thinking mode selection**. Claude Code defaults to an adaptive thinking mode, delegating the decision on inference depth entirely to the model, meaning that the same user query can consume drastically different token budgets at different times, making API costs unpredictable. For Anthropic itself, this unpredictability can be absorbed through server-side capacity planning; however, for enterprises using API keys to access Claude Code, the adaptive mode means they cannot set a reliable cost ceiling for a single session. The system retains the fixed budget mode and disables thinking mode as escape valves, but the default choice of adaptive mode indicates that Anthropic judges the value of user retention from improved quality over cost predictability for enterprise sales.

### 2.3 Context Preparation Details

Context preparation involves several steps:

1. **Message List Extraction**: The system extracts the compressed boundary from the message list.
2. **Tool Result Budget Clipping**: The tool results are clipped based on the budget.
3. **Historical Truncation**: The historical data is truncated to prevent the model from relying on outdated information.
4. **Micro Compression**: The messages are compressed to reduce the size of the context.
5. **Context Folding**: The context is folded to reduce the number of tokens.
6. **Automatic Compression**: The context is further compressed to ensure it fits within the budget.

### 2.4 System Prompt Assembly

The system prompt is assembled into two segments:

1. **Static Segments**: These include the introduction, system rules, task description, operation guide, tool description, and tone requirement.
2. **Dynamic Segments**: These include memory, MCP instructions, and the draft board.

The static segments can be cached and reused between API calls, while the dynamic segments need to be rebuilt each time.
Model responses are returned incrementally by tokens, with the system concurrently performing two tasks during the streaming reception process: pushing text incrementally to the upstream to achieve word-by-word display effects, and identifying and collecting tool invocation requests within the response. If the streaming tool executor is enabled, tool execution can even begin before the model is generating subsequent outputs (Chapter 4 will delve into this in detail). Once the model response is complete and includes tool invocation requests, the results of the execution of all tools are packaged according to the Anthropic message protocol and appended to the message list. The system then builds a new loop state and begins the next iteration.

### 2.3 State Management and Testability

State management across agent-loop iterations is especially easy to lose control of. Claude Code packages ten fields—including the message list, context-compaction tracking, output-token recovery counts, and interrupt-hook state—into an explicit state type. Each iteration destructures that object at the beginning and assigns a complete replacement at the end to commit its changes. This pattern uses the TypeScript compiler as a guardrail: each of the loop's seven early-restart points must construct a complete new state object, and omitting a field causes a compile error. One field records the reason for the previous transition, such as normal recursion, an interrupt hook blocking execution, or continuation after exhausting the token budget. Automated tests use it to assert that recovery paths were triggered correctly.
### 2.4 Engineering Resilience: Model Degradation, Streamed Backoff, and Retry

Real-world API calls are far more complex than textbook descriptions. Claude Code implements a multi-layered backoff mechanism to ensure resilience within the inference loop.

The first layer is **model degradation**. When the retry layer determines that the current model is unavailable and triggers degradation, the loop discards all intermediate results of the current iteration, switches to a backup model, and starts the iteration anew.

The second layer is **streamed backoff**. When an unrecoverable error occurs during the stream transmission process, the partially received model response is marked as invalid, the executing tool is canceled, and the loop backs off to the non-streamed mode for retry.

At the API communication layer, the retry logic adopts differentiated strategies for different types of errors. The retry strategy for the 529 error (service overload) exemplifies a classic design principle in distributed systems: **load-sensitive retry levels**. During capacity scaling, the retry requests themselves become a load component. The source code comments estimate an amplification factor of 3-10 times. This aligns with the observed data of **retry storms** (retry storms) in large-scale microservice systems. Claude Code's solution is to differentiate the request sources into foreground (directly perceptible by users) and background (such as summary generation, title suggestion, and classification). It only bears the load cost for foreground requests. The "fast fail" strategy for background tasks means that users may notice degraded secondary functionalities like delayed summaries or non-updated titles, but the reliability of core interactions is ensured. This intentional **graceful degradation** decision, and the prioritized queues in network congestion control, are akin to the load shedding practices in Site Reliability Engineering (SRE).

Additionally, the system implements a **streamed watchdog**. If no new stream events are received within the configured timeout window, the request is actively canceled and retried to prevent hung connections from permanently blocking the Agent loop.

OpenAI also mentions similar engineering challenges in the Codex technical documentation: the quadratic growth of prompts and cache misses leading to performance issues. Both systems face similar underlying challenges, but Claude Code demonstrates a more systematic approach to handling foreground and background requests.
When the complexity of a programming task exceeds the processing capability of a single inference loop, Claude Code achieves multi-level planning through the Sub-Agent mechanism. The main Agent can generate sub-Agents to handle independent sub-tasks, with each sub-Agent having its own 200K context window and returning only the summary results to the parent Agent:
```
Main Agent (200K Context Window)
    │
   ├── Task: "Search all files that use the deprecated API"
    │       └── Sub-Agent (independent with a context window of 200K)
    │               ├── Grep → Read → Read → ...
│               └── Return summary result → Main Agent
    │
   ├── Task: "Generate migration plans for each file"
    │       └── Sub-Agent (independent with 200K context window)
    │               └── ...
    │
── Comprehensive results, perform final modifications.
```
An important architectural constraint is a **recursion depth limit of 1**, where no sub-Agent can generate its own sub-Agent. This seemingly simple constraint addresses three levels of issues: First, **cost control**, exponential growth in recursive Agent recursion costs is compressed to linear growth with the depth limit; Second, **debug traceability**, any tool call can only trace back two levels (main Agent → sub-Agent → tool), making the monitoring system unnecessary for handling any depth of call stacks; Third, **predictable isolation of context**, each sub-Agent has its own 200K context window but only returns a summary result to the parent Agent, establishing a clear information bottleneck.

In the task planning layer, Claude Code also provides an explicit task list tool as a planning method. Its state management isolates sessions or Agent instances. When all tasks are marked as completed, the list is automatically cleared. This "explicit task list" planning method is more reliable than implicitly relying on the model's contextual memory, but it also depends on the model's willingness to use the planning tool.

The source code also includes modules related to the coordinator and the swarm. The feature flag list leaked contains a switch for Coordinator Mode. This is a phased evolution path from a single agent to sub-agents with a depth of 1, and then to a multi-agent coordination mode. This contrasts with Google ADK's "framework-first" approach, which offers sequential, parallel, and cyclic orchestration modes from the start.

## Three Memory and Context Management

### 3.1 Source of Context Window Pressure

AI Agent's memory issues stand out particularly in programming scenarios. A typical refactoring task might involve reading dozens of source files (each with hundreds of lines), executing multiple searches (each returning dozens of matches), and running test commands (outputting possibly thousands of lines). These tool results can quickly fill a 200K context window. The more challenging part is that the agent cannot simply discard early tool results, as subsequent inference may depend on content read from or search results obtained earlier.
### 3.2 Five-Layer Compress Pipeline

Claude Code employs a pipeline that compresses the context at the beginning of each inference iteration. Each layer employs different strategies and costs to reduce the context:
```
┌──────────────────────────────────────────────┐
│              Original Message List              │
│    (Contains All Historical Conversations and Tool Results)                  │
└──────────────────┬───────────────────────────┘
                   ▼
┌──────────────────────────────────────────────┐
│    First Layer: Budget Culling of Tool Results      │
│    Super-large tool outputs → Persist to disk, replaced with previews      │
└──────────────────┬───────────────────────────┘
                   ▼
┌──────────────────────────────────────────────┐
│    Second Layer: Truncating Historical Boundaries      │
│    Messages before the compression boundary are removed│
└──────────────────┬───────────────────────────┘
                   ▼
┌──────────────────────────────────────────────┐
│    Third Layer: Micro Compression (Cache Editing)        │
│    The API layer's cache editing mechanism deletes old tool results      │
│    Local messages are not modified; it only takes effect at the API request layer      │
└──────────────────┬───────────────────────────┘
                   ▼
┌──────────────────────────────────────────────┐
│    Fourth Layer: Context Folding                             │
│    Read Projection merges historical messages in a summarized manner            │
│    Original messages are not modified                        │
└──────────────────┬───────────────────────────┘
                   ▼
┌──────────────────────────────────────────────┐
│    Fifth Layer: Automatic Compression              │
│    Call the model to generate dialogue summary, and replace all historical       │
│    Threshold = Context Window - Output Reserved - Buffer         │
└──────────────────┬───────────────────────────┘
                   ▼
python
sent_into_model_api = "Sent Into Model API"

```
### 3.3 Micro Compression and Circuit Breaker

The third layer, **micro compression**, is a mechanism often overlooked but designed with precision. It is not an LLM summarizer but a precise pruning system based on API caching edits. Micro compression maintains a whitelist of compressible tools (file read, command execution, search, file matching, web access, file editing, file write), and decides which old tool outputs to delete based on token estimates. The API layer's caching mechanism then makes the deletion effective. The key is that **the local message array is not modified**, and the deletion only takes effect at the API request layer. This means that if the API cache is hot, micro compression can reduce context without disrupting the cache; if the cache is cold, the system switches to a strategy based on time for trimming. This is a trade-off of adding client complexity to reduce API costs.

The fourth layer, **context folding**, is also a non-destructive operation. The folded view is a **read-time projection**, with summary messages stored in independent folding storage rather than the main message array. Folding runs before automatic compression, and if folding is sufficient to bring the context below the threshold, automatic compression will not trigger, thus preserving more original details. When encountering a prompt overflow error, the system uses an overflow recovery mechanism to empty the stored folding to free space.

Only when all four layers are insufficient, the fifth layer, **automatic compression**, is triggered. It makes a single model call (closing the thinking mode to save tokens) to generate a summary of the entire conversation, and inserts the summary as a new compression boundary into the message list.

### 3.4 Circuit Breaker and Session Memory

The trigger threshold for automatic compression is defined as: the size of the context window minus the maximum output token count of the model, minus 13000 tokens of buffer. Once triggered, the system invokes a structured compression prompt (containing analysis and summary parts) to generate a conversation summary. If the summary request itself encounters a prompt overflow error, the system will truncate the head of the API round groups and retry.

The compression operation itself consumes model calls, and during high load, API calls may be unavailable. For this, the source code implements a circuit breaker mode: stopping retries after 3 consecutive failures. The source code comments reference a specific data point: on March 10, 2026, data showed that 50 or more consecutive compression failures occurred 1279 times (up to 3272 times) per day, resulting in approximately 250,000 API calls wasted daily globally.

### 3.5 Session Memory and Project-Level Memory

For session-level memory, the system maintains a separate memory for session-specific data. This ensures that session-specific context and state are preserved across API calls. For project-level memory, the system maintains a global memory that persists across sessions, allowing shared state and context to be maintained across different sessions.
Pipeline compression addresses the issue of "context not fitting." Claude Code's memory system also needs to handle another problem: degradation of compressed information over time. To address this, Claude Code implements two complementary long-term memory schemes.

First, it is **Session Memory**, which periodically extracts key information from the conversation history without blocking the main flow of conversation using a background forked agent. This information is then written into a session memory file in Markdown format. The triggering mechanism is based on thresholds: the session is initialized for the first time after a certain length of conversation, and then it is updated incrementally after a certain number of tool calls.

Second one is `MEMORY.md` for **project-level memory**, which is loaded in the system prompt's memory segment. `MEMORY.md` has a clear size limit: no more than 200 lines or 25000 bytes. Anything beyond this will be truncated. The memory content is organized into index files and topic files, and it is converted into a part of the system prompt upon loading.

Both of these solutions avoid vector databases and embedding retrieval, opting for the most straightforward file I/O approach. Markdown files are human-readable, making it possible to directly open and check them when issues arise, avoiding implicit failure modes such as index synchronization or embedding drift. However, the limitations of this approach are clear: when session history spans multiple distinct topics, linear Markdown notes struggle to support precise semantic retrieval. In contrast, LangGraph's Checkpointer achieves precise state tracking by saving full snapshot states at every super-step, albeit at the cost of higher storage overhead and more complex infrastructure dependencies. Both approaches reflect different trade-offs around "How should an Agent remember its past."

### 3.5 Task Budget: Cross-boundary Budget Coordination

Context management also overlooks one subtle dimension: **task budget**. Claude Code needs to track the compressed remaining budget allocated to APIs. There is a subtle client-server coordination issue here: before compression, the server can see the full conversation history and compute the budget usage on its own; but after compression, the server can only see a summary, underestimating the budget used. Thus, the client needs to maintain a counter of the remaining budget and inform the server of the current remaining value in each API request. This cross-compression boundary state coordination is a universal design challenge in any agent system that supports contextual compression.

## 4 TOOL USAGE

The more capable the tool's execution, the higher the requirements for its design. This chapter approaches design philosophy, gradually delving into interface design, execution orchestration, and error handling.

### 4.1 Primitive Combinations vs Specialized Tools
Claude Code makes a clear choice in tool design: providing a small set of generic primitives rather than a large set of specialized tools, allowing the model to decide how to combine primitives to complete a task. The core tool set is quite restrained: BashTool, FileReadTool, FileWriteTool, FileEditTool, GlobTool, GrepTool, AgentTool, TodoWriteTool, WebSearchTool, and WebFetchTool, approximately 15 in total. Advanced tools such as REPLTool, CronTool, and MonitorTool are hidden behind feature flags or internal user type checks, removed in the external build process through dead code elimination.

BashTool combined with GrepTool and FileEditTool, theoretically, can accomplish nearly all programming tasks. This contrasts with the approach of providing a large set of specialized tools in the LangChain ecosystem. The "primitive composition" strategy places higher demands on the model's inferential capabilities. If the model is not strong enough, a small set of primitives can limit the performance of the Agent. This is a bet on the continuous improvement of model capabilities.
To balance the large number of available tools and the limited prompt space, Claude Code introduced a **lazy loading** mechanism. Tools can be marked as lazy-loaded, and API calls only send the schema for tools that the model "discovered". Other tools are indicated by a lazy flag, informing the API to request the full definitions only when needed. The tool search implements retrieval based on keyword scores, supporting multi-selection and inclusion/exclusion query syntax. As a result, even with 40+ registered tools and all MCP tools registered, the number of schema sent per API call remains controllable.

### 4.2 Design Principles for Tool Interfaces

Each tool needs to implement a set of methods defined by a unified interface contract. The core design idea of this interface is that the behavior attributes of the tool are determined by the input, rather than the tool type. Concurrency safety, read-only, and whether it is reversible are methods that take parameters, not static attributes. This means that the same `BashTool` is concurrent-safe when executing `ls`, but not when executing `rm -rf`. The behavior of interrupt attributes defines how the tool should be canceled or blocked when the user sends new messages during the execution of the tool: file editing should be canceled, but a running compilation command should not be interrupted. Each tool also declares a limit on the size of the results; any results exceeding this limit are persisted to disk and replaced with previews (this is directly related to the tool result budget clipping in Chapter 3).

### 4.3 BashTool: The Core and Most Dangerous Tool

In all tools, the implementation of BashTool is the most complex due to its simultaneous possession of the most powerful execution capabilities and the highest security risks. Its permission checks are delegated to a dedicated command security analysis module, which introduces a tree-sitter AST parser to perform syntactical security checks on shell commands at the level of syntax trees. The commands are then evaluated by breaking them down into subcommands, with a limit of 50 subcommands set to prevent overly complex commands from bypassing parsing or triggering a Denial of Service (DoS). Commands exceeding this limit are marked as needing user confirmation by default.

BashTool's execution path adapts based on context: in sub Agents, directory changes are prohibited; when sed edit mode is detected, it shorts to a dedicated path for better diff display; in sandbox mode, it executes commands using a dedicated sandbox runtime in a restricted environment.

### 4.4 Parallel and Serial Dynamic Scheduling

When a model requests multiple tool invocations in a single response, the orchestration layer needs to decide which can run in parallel and which must be executed sequentially:
```
Tool Invocation Request Sequence: [Grep, Read, Read, Bash, Read, Read]
                       │
                       ▼
                  ┌──────────┐
| Partition Algorithm |
                  └────┬─────┘
                       ▼
Batch 1 (Concurrent)       Batch 2 (Exclusive)      Batch 3 (Concurrent)
    ┌─────────────┐   ┌──────────┐    ┌─────────────┐
    │ Grep ║ Read │   │   Bash   │    │ Read ║ Read │
│ ║ Read      │   │ (Exclusive)    │    │             │
    └─────────────┘   └──────────┘    └─────────────┘
     Parallel Execution                          Serial Execution                          Parallel Execution

      (Limit: 10)                          (Limit: 10)
```
Algorithm for partitioning iterates over all tool invocation requests, grouping consecutive concurrent-safe tools into the same batch. When concurrent safety determination itself throws an exception (such as Bash tool fails to parse a shell command), the system defaults to treating that tool as non-concurrent-safe. This "fallback to the conservative path" strategy runs throughout the codebase.

Except for batch execution paths, the streaming tool executor implements more aggressive optimizations: execution of received tool calls begins before the model's streaming response is complete. Each model-generated tool call request is added to the queue, and execution begins immediately if the concurrency condition is met. Each tool goes through four states: queued, executing, completed, and delivered.

In the flow tool executor, one noteworthy design is the **sibling abort**: only errors from Bash tools trigger a sibling abort. Often, there are implicit dependency chains between Bash commands (`mkdir` failing means subsequent `cp` commands lose their meaning). In contrast, commands like Read or WebFetch are typically independent; failure of one should not affect others. This differentiation in handling the semantic differences between different tools reflects the considerations in controlling error propagation in production-grade agent systems.

## Five Perception and Security

The more capable the execution of the tool, the more critical its behavior constraints become. Claude Code implements an agent's perception of risk through a multi-layered security system covering the entire depth from global policies to individual command parsing.

### 5.1 Seven-Step Permission Pipeline

Claude Code's core for permission determination consists of a seven-step pipeline:
```
Tool Invocation Request
    │
    ▼
┌───────────────────────────────────────┐
│ Step 1: Global Deny Rule                    │  ← Hard Deny Rule, cannot be overridden.
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
Step Two: Global Confirmation Rules                    ← Requires User Confirmation
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
Third Step: Tool Self-Permission Check                 # Each tool checks its own permissions based on the input.
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
**Step Four: Tool Implementation Layer Refusal**
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
Step Five: Interact with User Requirements Check                    │  ← Need User Interaction
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
Step Six: Content-Based Confirmation Rules
└───────────────────┬───────────────────┘
                    ▼
┌───────────────────────────────────────┐
Step 7: Safety guardrails ← Protect `.git/`, `.claude/`, and other sensitive paths
│  (Protected Directories and Configurations)                 │     shell configuration, etc.
└───────────────────┬───────────────────┘
                    ▼
Authority determines (allow / deny / inquire)
```
Permission determination first checks global prohibition and confirmation rules, and then delegates the decision-making authority to the permission check method of the tool itself. Taking Bash as an example, its permission check is delegated to a dedicated command security analysis module, which parses the command using tree-sitter to perform AST-level analysis and assess the security of each sub-command. The permission logic is not concentrated in a single giant function but is distributed between global rules and the implementations of each tool. The benefit of this layered approach is that each tool can customize its security policies based on its semantics. The global rules handle cross-cutting concerns such as protected directories.

### 5.2 Permissive Function Interface for Pluggable Permissions

The key abstraction in a permissions system is to encapsulate permission checks as a **pluggable asynchronous function interface**: Given a tool, input, and context, it returns one of "allow", "deny", or "ask" decisions. This function has different implementations in various runtime environments: In REPL mode, confirmation requests are pushed into the UI queue to await user interaction. In SDK mode, it uses a structured IO channel or automatically denies. In remote mode, it connects via a WebSocket bridge to the remote client. The tool execution pipeline is completely agnostic to the specific presentation of permissions, all handled by a unified function signature.

In the implementation of interactive permission confirmation, there is a subtle detail: the system asynchronously runs a background security classifier while the user is interacting. They engage in a "race". To prevent accidental key presses (such as pressing Enter before the classifier returns a result) from canceling the classifier's check, the system sets a "first interaction grace period" of 200 milliseconds. This approach of balancing the immediacy of user experience with the completeness of security checks is a noteworthy pattern in security system design.

### 5.3 Hook System and Monotonicity of deny

Claude Code allows users to register hooks to customize permission policies before and after tool calls. A key security invariant exists in the interaction between hooks and the permission pipeline: **a hook's "allow" decision cannot override a "deny" rule**. Even if a user's pre-hook returns "allow", the global deny rule will still take effect. "Deny" is **monotonic** in this system; any denial at any stage is final and cannot be reversed. From a security engineering perspective, this constraint is correct: in a deep defense system, inner layers should not have the capability to weaken outer protections. However, this also means that the hook system's flexibility is limited; users cannot use hooks to "unlock" operations prohibited by global rules.

### 5.4 Gradient Spectrum and Sandbox of Permission Mode

Claude Code offers five modes of permission, from most conservative to most radical: default (prompt confirmation on first use of a new tool), plan (read-only analysis, disallowing write operations), acceptedEdits (automatically accept file edits, Bash still requires confirmation), auto (automatically approve, run background security classifier), bypassPermissions (skip prompts, except for protected directories).

### 6.1 Key Discoveries in Four Dimensions

In the realms of deduction and planning, Claude Code shifts the complexity budget from orchestration to model inference, encapsulating all orchestration logic within the simplest of loop structures. The feasibility of this design hinges on establishing robust mechanisms for error recovery, state management, and cost control around the loop. The introduction of adaptive thinking budgets achieves a dynamic equilibrium between inference quality and API costs. The caching boundary design for prompts reduces the overhead of repeated transmissions across multiple dialogues.

In the memory dimension, the five-tier progressive compression pipeline is one of the most refined engineering designs in this source code. Microcompression reduces context and uses read-time projection for context folding via API caching instead of LLM summaries. These "non-destructive" compression strategies are more precise and easier to debug than full-summary approaches. However, a memory retrieval scheme that does not use embeddings suffers from inherent limitations in semantic precision.

In the dimension of tool usage, a flexible yet space-efficient tool system is constructed using a few common primitives and a lazy loading mechanism. The streaming executor's streaming startup and sibling cancellation mechanisms demonstrate a fine balance between tool execution efficiency and error control.

In the domains of Perception and Security, the seven-step permission pipeline, monotonic denial constraints, AST-level command parsing, and sandbox isolation form a comprehensive layered defense. The background classifier in auto mode achieves a balanced trade-off between autonomy and security, but the "execute first, audit later" mode still has room for improvement in mitigating risks associated with irreversible operations.

### 6.2 Code Quality
From a code quality perspective, the defensive engineering patterns in Claude Code go beyond the usual "good code" to the level of **implementing organizational strategies through type systems**. The metadata for privacy compliance uses a type system-level enforcement mechanism: a "bottom type" (bottom type in TypeScript) is defined, making it impossible for any string containing user code or file paths to compile unless a explicit type assertion is made in the code. This is essentially a written declaration of "I have confirmed that this data does not contain sensitive information," forming traceable compliance records during code reviews. This moves privacy compliance from runtime checks to type checks at the development stage, a lightweight form of **taint analysis**.
The information density of the comments is another highlight. The warning comment at the beginning of the global state module ("do not add more state here") conveys architectural intent rather than code semantics. The comment referencing production data from March 10, 2026, to justify the circuit breaker thresholds is an audit record of engineering decisions. These comments share the characteristic of answering "why" rather than "what." Defensive programming permeates all layers, and testability is ensured through explicit separation of production dependencies via dependency injection.

### 6.3 AI Programming Agent Evolution Directions

Claude Code's architecture choices suggest several evolving directions in the field of AI programming Agents.

The philosophy of "Less scaffolding, more model" is based on the premise that model inferential capabilities will continue to improve. If this premise holds true, today's design, which is "overly trusting of models" (with minimal primitives, no explicit task planners, no RAG), may prove to be the right direction in the future. Conversely, if the improvement in model capabilities hits a bottleneck, more scaffolding and more structured orchestration may become necessary again. The Codex team at OpenAI has found in practice that agents remain "vulnerable" beyond the training data distribution and require human supervision for production use. This suggests that the current "trust the model" strategy may have its boundaries.

Expanding the context window changes the economics of the compression strategy. As the window expands from 200K to even 1M, some layers in the five-layer compression pipeline may become unnecessary, but the importance of Session Memory, a long-term memory scheme, increases. Longer sessions mean more information that needs to be maintained across sessions.

Multi-Agent Collaboration is still in its early stages of exploration. Claude Code begins its journey at a depth of 1 from sub-Agent, indicating that the team is evolving towards more complex multi-Agent models with the existence of coordinator and swarms modules in the source code. Solutions for coordinating permissions, sharing context, and allocating task budgets among multiple Agents are not yet mature on the engineering front.

### 6.4 Hidden Eggs in the Source Code

In 500,000 lines of code, there are not all serious engineering logic. The leaked source code also reveals several amusing details that reflect Anthropic team's product thinking and engineering culture from a side view.
**BUDDY Cyber Pet System**

Claude Code incorporates a complete virtual pet system (hidden behind the BUDDY feature flag), featuring 18 species (ducks, dragons, capybaras, ghosts, salamanders, etc.), each with a gacha mechanism divided into five rarity tiers (common 60%, legendary 1%). Pets come with accessories like hats and eyes, and possess five RPG-style attributes: Debugging Power (DEBUGGING), Patience (PATIENCE), Chaos (CHAOS), Wisdom (WISDOM), and Snarkiness (SNARK). Each pet is generated deterministically based on the user ID, utilizing a minimal pseudorandom number generator called "Mulberry32." The source code even includes a comment explaining the hexadecimal encoding of species names: to avoid triggering a string scanner at build time, one species name coincidentally matches an internal model code at Anthropic.

The system was originally slated to be released as an Easter egg from April 1st to 7th, 2026. The release strategy notes state: "Using local time rather than UTC ensures a 24-hour rolling window across time zones, maintaining sustained discussion on Twitter, while avoiding the peak of instantaneous generation at UTC midnight."

**autoDream: Nightly Memory Consolidation**

The KAIROS feature flag houses an "Always-On Claude" feature, which includes an autoDream background memory consolidation system. The prompt for this system begins with: "You are performing a dream — a reflective pass over your memory files." The system employs a PID-based file lock to prevent multiple instances of Claude Code from simultaneously "dreaming." The consolidation process is divided into four stages: Orientation (Orient), Collection (Gather), Consolidation (Consolidate), and Pruning (Trim). This design conceptually parallels the human process of consolidating sleep-related memories.

**Archaeology of Model Migration**

The migration directory in the source code contains a complete chain of model renaming migrations: Sonnet 1M → Sonnet 4.5 → Sonnet 4.6 → Legacy Opus → Opus 1M. Each time Anthropic renames a model, the Claude Code client must execute a migration to update user-saved preferences. These migration files serve as a geological record of Anthropic's model naming changes, embodying an implicit maintenance cost associated with rapid model iterations.
For engineers interested in the design of AI Agent systems, Claude Code's source code offers a unique reference: not how an Agent should be designed according to the wishes of the framework creator, but how an actual product-level Agent is built after being validated by millions of users. The gap between these two, may be the most noteworthy part of the current AI Agent engineering process.
