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
Concise summary; files changed; commands and results; evidence; risks; unresolved issues.
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
What the main/Astra role, Sol, and Luna actually did.

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
