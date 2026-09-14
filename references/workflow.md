# Project workflow

## Project plan

For substantial work, keep this concise control block in the main thread:

```text
PROJECT GOAL
CONSTRAINTS
ARCHITECTURE
TASK GRAPH
DEPENDENCIES
RISK LEVEL
MODEL ASSIGNMENT
ACCEPTANCE CRITERIA
```

Use task states `TODO`, `READY`, `RUNNING`, `REVIEW`, `BLOCKED`, `FAILED`, and `DONE`. A task becomes `DONE` only after main-thread evidence review.

## Dispatch ledger

The main orchestrator must keep a compact in-thread ledger for every worker execution request, including new spawns and follow-ups sent to an existing worker. Record one row per attempt, not merely one row per logical task:

| Field | Meaning |
| --- | --- |
| Task ID | Stable logical task id, such as `T3` |
| Attempt | Monotonically increasing attempt number for that task |
| Objective | Short, user-comprehensible objective |
| Task name | Exact spawned task name |
| Model argument | Requested `model`; for executed work, the value accepted by the spawn or follow-up request or a contractually guaranteed inherited value; for a rejected spawn, the attempted value labeled as not used |
| Effort argument | Requested `reasoning_effort`; for executed work, the value accepted by the spawn or follow-up request or a contractually guaranteed inherited value; for a rejected spawn, the attempted value labeled as not used |
| Dispatch outcome | `SPAWN_PENDING`, `LAUNCHED`, `SPAWN_FAILED`, `FOLLOWUP_SENT`, or `FOLLOWUP_FAILED` |
| Attempt state | `RUNNING`, `RETURNED`, `ACCEPTED`, `REJECTED`, `FAILED`, or `INTERRUPTED` |
| Review round | `0`, `1`, or `2` |
| Routing reason | Initial assignment, same-class rework, escalation, downgrade, or failure classification |
| Validation | Concise evidence and the main-thread acceptance decision |

Create the row as `SPAWN_PENDING` before spawning. Change it to `LAUNCHED` and `RUNNING` only after the API accepts the explicit model and effort. A rejected spawn becomes `SPAWN_FAILED`; describe its model and effort as attempted, not used. If the runtime returns resolved configuration, compare it with the request and reject a mismatch as an invalid attempt. If the active schema cannot set both fields explicitly, do not delegate when exact attribution is required.

Every retry, rework, escalation, downgrade, or follow-up gets a new attempt row linked by Task ID. Set `FOLLOWUP_SENT` only after the API accepts the follow-up. When a follow-up supplies explicit model or effort overrides, record those accepted arguments. When it omits either field, reuse the worker's accepted configuration only if the active API contract guarantees inheritance. If inheritance is not guaranteed, do not send that follow-up under exact-attribution reporting; spawn a new worker with explicit configuration instead. Apply the same resolved-configuration mismatch check when follow-up results expose resolved values. Logical task state moves to `RUNNING` after successful launch or follow-up, `REVIEW` after return, and `DONE` only after main-thread acceptance.

Prefer explicit model and effort on every dispatch. When the active schema cannot set both fields, delegate only if the API contract guarantees the inherited values, those values are observable, and they comply with `routing.md`; otherwise keep the task in the main thread. Accepted explicit spawn or follow-up arguments, plus qualifying inherited fields, are authoritative for executed work. Do not infer them from a task-name suffix or worker self-identification. Keep main-thread work separate; if its model cannot be observed, label it `main thread (model unverified)` rather than claiming Astra. Do not persist the ledger to repository files unless requested. Do not record secrets, full prompts, transcripts, agent/thread identifiers, absolute paths, hidden reasoning, raw chain of thought, or noisy logs.

## Worker contract

Every worker prompt must contain these headings and concrete content:

```text
TASK
One bounded task and a unique task id.

CONTEXT
Only the facts and inputs needed for this task. Do not rely on inherited chat history.

OBJECTIVE
One observable outcome.

SCOPE
Allowed operations and responsibility boundaries.

FILES / MODULES
Exact paths or explicitly read-only discovery scope.

DO NOT TOUCH
Adjacent files, interfaces, behavior, credentials, and unrelated user changes to preserve.
Do not spawn, delegate to, or coordinate any other agent.

ACCEPTANCE CRITERIA
Checks that must be true before reporting completion.

VALIDATION METHOD
Exact tests, lint, typecheck, build, runtime check, benchmark, screenshot, log, or file inspection.

EXPECTED OUTPUT
Concise summary; files changed; commands and results; evidence; risks; unresolved issues. Do not claim a model identity or reasoning effort; the orchestrator records those from spawn arguments.
```

Use `fork_turns: "none"` when the active spawn schema supports it. Put every necessary fact in `CONTEXT`. Use task names containing only lowercase letters, digits, and underscores, with a model/effort suffix matching the actual spawn.

## Main-thread review

Move a returned task to `REVIEW`, then examine evidence appropriate to the risk:

- actual `git diff` or file contents for material changes
- targeted tests and failure output
- lint, typecheck, and build results
- runtime behavior or generated artifacts
- benchmarks for performance claims
- screenshots for visual claims
- logs for operational claims

Protect unrelated and pre-existing user changes. Confirm scope and acceptance criteria before marking `DONE`.

If evidence fails, identify whether the cause is implementation, reasoning, plan, environment, or requirements. Then perform a bounded same-class rework, escalate, or return to main-thread replanning as described in `routing.md`.

## Final report

Only the main orchestrator reports to the user. Use these sections when applicable:

```text
## Result
Whether the project is complete.

## Implemented
What changed.

## Architecture decisions
Important choices and tradeoffs.

## Agent usage
Start with a brief account of main-thread planning, review, and integration. Then include every dispatch-ledger attempt using this table:

| Task ID | Attempt | Objective | Model | Reasoning effort | Dispatch | Result | Routing/retry reason | Validation and acceptance |
| --- | ---: | --- | --- | --- | --- | --- | --- | --- |

Include failed spawns, rejected, superseded, retried, escalated, downgraded, interrupted, and successful attempts. Use accepted dispatch arguments or contractually guaranteed inherited fields for executed work; label rejected spawn configurations as attempted, not used. Do not collapse multiple attempts into one row.

## Validation
Tests, build, lint, typecheck, runtime checks, and their results.

## Remaining issues
Known gaps or "None".

## Important files changed
The small set of useful file links.

## Recommended next step
Only when a useful next action remains.
```

Do not dump raw agent transcripts or present worker self-reports as validation.
