---
title: "Engineering Reliable Coding Agents"
date: 2026-05-18T10:00:00+08:00
lastmod: 2026-09-09T00:00:00+08:00
draft: false
categories: ["AI Agents"]
description: "A harness-first approach to specifications, persistent state, verification, and safe autonomy for coding agents."
aliases:
  - /posts/harness-engineering/harness-engineering-architecture/
---

# Engineering Reliable Coding Agents

A coding agent can make a locally correct change while the project around it becomes less coherent.

Consider a three-session authentication refactor. The first session chooses RS256 and implements token validation. The second session cannot recover that decision, infers HS256 instead, and introduces a conflicting path. The third session resolves the conflict but also rewrites an unrelated password module. Each session can plausibly report success. Taken together, the repository has drifted.

This is the reliability gap in agentic software engineering: model capability is only one variable. The surrounding environment determines whether useful reasoning survives contact with a real codebase, multiple sessions, incomplete specifications, and destructive tools.

OpenAI uses the term **harness engineering** for the work of designing that environment. In its account of building an internal product with Codex, three engineers guided roughly 1,500 pull requests over five months while agents wrote the repository's code, tests, documentation, and infrastructure. The interesting result is not the volume of generated code. It is the change in the engineers' job: they spent more time specifying intent, shaping the repository, and building feedback loops that made agent work verifiable.

This essay develops a practical model of that harness. Its central claim is simple:

> Reliable agent behavior is a property of the complete engineering system, not of the model in isolation.

## The capability–reliability gap

When a coding agent fails, teams often blame reasoning quality. In practice, many failures begin elsewhere.

### Underspecified work

"Implement authentication" leaves a large solution space open. It does not say which protocol to use, where state belongs, which modules are protected, how errors should be represented, or what evidence counts as completion. The model fills those gaps with plausible defaults. A plausible choice is not necessarily the intended choice.

The remedy is not an enormous prompt. It is an executable specification that makes boundaries and acceptance criteria explicit.

### Lost state between sessions

Real changes frequently outlive one context window or one agent session. Architectural decisions, deliberate compromises, unfinished edges, and rejected alternatives disappear unless they are written into the repository.

A larger context window delays this problem; it does not remove it. Session history is not a durable project record.

### Premature completion

An agent tends to evaluate its work through the same assumptions it used to implement it. If those assumptions are incomplete, self-review can reproduce the original blind spot. A successful happy path then becomes a declaration that the feature is complete.

Completion needs an external anchor: tests, type checks, static analysis, browser assertions, or a review performed against explicit criteria.

These failure modes point to three different missing mechanisms:

| Failure | Missing mechanism | Typical symptom |
| --- | --- | --- |
| Underspecification | Scope and acceptance criteria | A reasonable but unwanted implementation |
| Cross-session state loss | Durable project state | Contradictory decisions across sessions |
| Premature completion | Independent verification | A passing demo with broken edge cases |

## A harness is a closed-loop control system

Without a harness, the workflow is open-loop:

```text
instruction -> generation -> human inspection
```

The human must notice every deviation and supply every correction. This works for short, supervised tasks, but it does not scale to long-running or parallel work.

A closed-loop workflow feeds observed results back into the next action:

```text
specify -> execute -> verify -> record state -> continue or correct
```

The harness supplies five cooperating subsystems:

1. **Instructions** define stable operating rules and safety boundaries.
2. **State** preserves decisions and progress between sessions.
3. **Scope** constrains which parts of the repository may change.
4. **Verification** turns completion into an observable condition.
5. **Lifecycle** makes initialization, execution, and handoff repeatable.

The harness does not prescribe every reasoning step. It constrains actions and makes their consequences visible. This distinction matters. Trying to enumerate every possible thought produces brittle prompts; defining allowed effects, required evidence, and recovery paths produces a system that can tolerate different reasoning paths.

## Treat the repository as the system of record

Anything required by a later session should live in a durable, reviewable location. For a small project, a minimal structure might be:

```text
.
├── AGENTS.md
├── docs/
│   ├── architecture.md
│   └── progress.md
├── feature-list.json
├── scripts/
│   └── bootstrap.sh
└── tests/
    └── e2e/
```

The filenames are not important. The division of responsibility is.

- `AGENTS.md` contains stable rules that apply every time an agent enters the repository.
- `architecture.md` records decisions whose lifetime is longer than the current task.
- `progress.md` describes dynamic state: what is done, what is blocked, and what should happen next.
- `feature-list.json` expresses machine-checkable acceptance criteria and dependencies.
- `bootstrap.sh` makes the environment reproducible.
- `tests/e2e/` verifies behavior at a boundary users care about.

Code is part of this record. An agent should inspect the actual implementation before inferring what the system ought to contain. Documentation that disagrees with code is not context; it is a defect the harness must expose.

### State should preserve decisions, not narrate activity

A useful progress file is not a chronological transcript. It is a compressed handoff to the next worker:

```markdown
## Current state
- Complete: login endpoint and token verification
- In progress: password reset flow
- Blocked: email integration requires a provider decision

## Decisions
- RS256: downstream services verify tokens with a public key
- Access token lifetime: 15 minutes
- Refresh token lifetime: 7 days

## Next actions
1. Implement the email adapter behind the existing interface
2. Run the password-reset end-to-end suite
3. Update the acceptance record with observed results
```

The key field is often **why**. Recording only what changed forces later sessions to reverse-engineer the decision and makes accidental reversal likely.

## Separate instructions by rate of change

A single giant instruction file mixes permanent safety rules with temporary task context. As it grows, conflicts become harder to resolve and important constraints become harder to retrieve.

A more robust design separates instructions by how quickly they change:

```text
stable       AGENTS.md              safety, tools, completion rules
slow         architecture.md        project decisions and prohibited patterns
fast         progress.md            current scope, blockers, next actions
fast         feature-list.json      acceptance criteria and task status
```

The stable layer should answer four questions with very little ambiguity:

1. Which tools may the agent use?
2. Which paths or systems are protected?
3. When must the agent stop and ask a human?
4. What evidence is required before reporting completion?

More instructions are not automatically better. The goal is a small set of high-signal constraints that are consistently activated, with detail delegated to the layer that owns it.

## Make completion executable

Human task trackers tolerate statements such as "support user registration." An agent needs criteria it can test:

```json
{
  "id": "auth-001",
  "name": "User registration",
  "status": "in_progress",
  "acceptance_criteria": [
    "POST /api/auth/register returns 201 with a user_id",
    "a duplicate email returns 409",
    "a password shorter than eight characters returns 400",
    "the registration end-to-end suite passes"
  ]
}
```

Each criterion describes an observable outcome. The implementation remains open-ended, but the definition of done does not.

This changes the role of end-to-end tests. They are still regression protection, but for an agent they also provide a live control signal:

```text
implement -> run targeted test -> inspect failure -> correct -> rerun
```

Tests must be runnable without hidden human setup. If they depend on a developer remembering to start a service, configure an undocumented secret, or prepare local data, the feedback loop is incomplete.

Dependencies should also be explicit. An agent that autonomously chooses task order can begin an interface before its underlying data contract is stable. A gate can encode that `auth-foundation` must pass before profile and OAuth work begins. This is useful for humans too: it turns an implicit dependency graph into shared project state.

## Design a deliberate session lifecycle

Initialization should be a phase, not a suggestion. Before editing, an agent should be able to establish:

- the repository rules;
- the current task and its boundaries;
- the state of the working tree;
- the available build and test commands;
- a passing baseline or a documented pre-existing failure.

The point is not ceremony. It is to prevent the agent from interpreting an unknown repository state as a blank slate.

The session should end with an equally explicit handoff. A new engineer or agent ought to continue without asking the previous session what happened. That requires synchronized state, runnable tests, named blockers, and no unexplained intermediate changes.

Two forms of scope failure deserve different controls:

- **Overreach** occurs when an agent improves unrelated code while completing the assigned task. Protected paths, explicit file scope, and change review constrain it.
- **Under-finishing** occurs when an implementation omits errors, edge cases, documentation, or cleanup that belong to the task. Acceptance criteria and tests constrain it.

One policy cannot solve both problems. Narrow permissions reduce overreach but cannot prove completeness; more tests increase coverage but cannot stop an unrelated refactor.

## Observability and independent evaluation

An agentic workflow needs enough telemetry to reconstruct what was attempted, which evidence was available, what changed, and why the system accepted the result. This does not require storing private chain-of-thought. Tool calls, diffs, test results, task transitions, and concise decision records are usually the more useful signals.

For higher-risk work, separate implementation from evaluation. An independent reviewer—human, deterministic tool, or a second agent—checks the result against the acceptance criteria rather than inheriting the implementer's narrative. The evaluator should return structured evidence:

```json
{
  "feature_id": "auth-001",
  "verdict": "partial",
  "passing": [
    "successful registration returns 201",
    "duplicate email returns 409"
  ],
  "failing": [
    "short password returns 500 instead of 400",
    "three end-to-end tests fail"
  ]
}
```

Role separation is not free. It adds latency and compute cost, so the degree of independence should follow the risk. A documentation typo may need only a diff. An authentication or migration change deserves stronger gates.

## Autonomy requires containment

As agents gain access to shells, networks, browsers, and external services, reliability becomes inseparable from security. Repeated permission prompts are not a complete control: they slow work and can train users to approve without reading.

A safer model combines a narrow action boundary with escalation at trust crossings. Filesystem and network sandboxing reduce the blast radius of a mistaken or manipulated action. Destructive operations, external communication, credential access, and changes outside the workspace remain explicit decision points.

Anthropic reports that sandboxing Claude Code reduced permission prompts in its internal use while enforcing filesystem and network boundaries. The broader lesson is independent of one product: safe autonomy comes from making the routine path both constrained and low-friction.

## A practical minimum harness

Teams do not need a large orchestration platform to begin. A useful first version can be small:

1. Write one short repository guide containing tool, safety, and completion rules.
2. Define acceptance criteria before implementation begins.
3. Provide one reproducible environment bootstrap command.
4. Make targeted tests executable by the agent.
5. Record decisions and blockers in a compact handoff file.
6. Review the final diff against the original scope.
7. Add stronger isolation and independent evaluation where consequences justify them.

The system should evolve from observed failure. If agents repeatedly forget a command, improve discovery. If they edit unrelated files, tighten scope. If they pass unit tests but break the user journey, add a boundary-level check. A harness is maintained engineering infrastructure, not a prompt written once.

## Conclusion

Model improvements will continue to expand what coding agents can do. They will not eliminate the need for clear intent, durable state, verification, containment, and recovery. Faster processors did not make operating-system isolation obsolete; more capable agents do not make engineering controls obsolete.

The most productive question is therefore not only, "Which model is best?" It is also:

> What environment lets this model do useful work repeatedly, while making failure visible and recoverable?

That environment is the harness. Building it is becoming part of software engineering itself.

## References

- OpenAI, [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/), February 11, 2026.
- Anthropic, [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices), April 18, 2025.
- Anthropic, [Beyond permission prompts: making Claude Code more secure and autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing), October 20, 2025.
