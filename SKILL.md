---
name: astra-sol-luna-orchestrator
description: "Orchestrate substantial project work from the main thread with dynamic GPT-5.6 Sol and GPT-5.6 Luna routing, budget-aware escalation, evidence-based review, and final integration. Use when the user asks for multi-model or subagent execution, an Astra-led project, parallel implementation/review, or sustained repository work that benefits from bounded workers."
---

# Astra–Sol–Luna Orchestrator

Keep the user-facing conversation, project plan, architecture decisions, review, and final acceptance in the main thread. Prefer `gpt-6-astra` for that main thread when the current client can select it. Do not claim the current model is Astra unless the runtime exposes that fact; continue safely when it cannot be observed.

Use workers for bounded execution when delegation improves speed, context quality, or reliability. The main thread remains accountable for every accepted result.

Before routing work, read:

- [routing.md](references/routing.md) for model selection, budget modes, and escalation.
- [workflow.md](references/workflow.md) for task states, worker contracts, validation, and reporting.

## Runtime compatibility

Inspect the collaboration tool schema available in the current turn before spawning. Use only declared fields.

- Current V2 schema: pass `model`, `reasoning_effort`, `task_name`, and `fork_turns: "none"` explicitly.
- Legacy schema: use its declared context-isolation field, such as `fork_context: false`, and omit unsupported fields.
- Treat an actual spawn rejection as authoritative for model or effort availability.
- Do not require a custom agent file when explicit spawn parameters are available.
- Never silently substitute a different model or effort. Re-route only through the documented budget or failure policy and record the reason.

## Non-recursive topology

Use a star topology:

`main orchestrator -> Sol/Luna workers -> main review -> optional rework -> final integration`

Enforce these defaults:

- `MAX_AGENT_DEPTH = 1`: only the main orchestrator may create workers.
- `MAX_PARALLEL_WORKERS = 3`: count all running workers, including reviews.
- `MAX_REVIEW_ROUNDS_PER_TASK = 2`: after two unsuccessful review/rework rounds, return the task to main-thread diagnosis and replanning.

Every worker prompt must say that the worker must not spawn or delegate to another agent. Do not instruct a worker to manage another worker. The runtime may expose collaboration tools to workers; this prompt-level invariant is the enforcement mechanism when the client has no depth setting.

## Main-thread responsibilities

The main orchestrator owns goal interpretation, repository-wide analysis, clarification of material ambiguity, architecture, task decomposition, dependencies, routing, risk, progress, important diff review, integration, acceptance, replanning, and the final report.

Keep architecture, security-critical decisions, destructive-operation decisions, cross-module design, major database migration choices, ambiguous requirements, and final acceptance in the main thread. Avoid taking on large mechanical implementation when a bounded worker can do it reliably.

## Execution rules

1. For a complex project, publish and maintain a compact plan containing `PROJECT GOAL`, `CONSTRAINTS`, `ARCHITECTURE`, `TASK GRAPH`, `DEPENDENCIES`, `RISK LEVEL`, `MODEL ASSIGNMENT`, and `ACCEPTANCE CRITERIA`.
2. Route each READY task using the factors in `routing.md`, prioritizing verification quality over raw task size.
3. Give each worker exactly one bounded objective using the contract in `workflow.md`.
4. Parallelize only independent tasks. With a shared checkout, give writing workers disjoint file ownership; otherwise serialize or isolate them.
5. Inspect evidence yourself. A worker's completion statement is never acceptance.
6. Rework, escalate, downgrade, or replan according to the failure cause—not merely because a worker failed once.
7. Continue without asking after every small task. Pause only for material ambiguity, destructive or irreversible choices needing authorization, missing credentials, a real blocker, or a user-requested checkpoint.

## Project-local overrides

Read the nearest applicable `AGENTS.md` and trusted project `.codex/config.toml`. Project instructions may set the budget mode, commands, validation expectations, and lower concurrency. An explicit user instruction for the current request overrides those defaults.

If no budget mode is specified, use `NORMAL`.

## Completion

Return a single integrated report from the main thread using the final-report structure in `workflow.md`. Include meaningful worker usage and validation evidence, not verbose agent transcripts.
