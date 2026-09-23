# Astra–Sol–Luna Orchestrator

A reusable Codex Skill for main-thread orchestration with dynamic routing between GPT-6 Astra, GPT-5.6 Sol, and GPT-5.6 Luna workers.

## Highlights

- Keeps architecture, important decisions, review, and final acceptance in the main thread.
- Routes work by reasoning difficulty, risk, reversibility, and verification quality rather than code volume alone.
- Runs under any capable main-thread model; an Astra main thread is preferred when selected, but is not required.
- Caps automatically selected Astra and Sol reasoning at `medium`; Luna may use `high` only for difficult, bounded, strongly verifiable work.
- Supports `NORMAL`, `CONSERVATIVE`, and `LOW_BUDGET` modes.
- Escalates only after classifying the failure, and returns expensive-model failures to main-thread replanning instead of automatically raising effort above `medium`.
- Limits orchestration to a depth-1 star topology, three concurrent workers, and two review rounds per task.
- Requires explicit worker contracts and evidence-based acceptance.
- Reports requested model and reasoning effort for every worker attempt, distinguishing rejected attempts from executed work, with result and validation evidence.

## Install

Place this repository at:

```text
${CODEX_HOME:-$HOME/.codex}/skills/astra-sol-luna-orchestrator
```

Start a task with:

```text
$astra-sol-luna-orchestrator Complete this project: <goal>
```

Select a budget mode when needed:

```text
$astra-sol-luna-orchestrator BUDGET_MODE=LOW_BUDGET Complete this project: <goal>
```

## Model and usage behavior

You do not need to select GPT-6 Astra to activate this Skill. If the main thread uses GPT-5.6 Sol and the client supports explicit per-worker model and reasoning-effort selection, it may call a bounded Astra worker at `medium` or lower while Sol remains responsible for orchestration and the final answer.

Reasoning limits are applied only to individual worker dispatches. The Skill never edits user or project configuration to pin the main-thread model or reasoning effort, so users remain free to choose any available reasoning level when starting a conversation.

Each subagent performs its own model and tool calls, so multi-agent runs consume more tokens than comparable single-agent runs. The Skill therefore delegates only bounded work whose speed, context isolation, or independent verification benefit justifies the additional usage.

## Upstream and redistribution note

This Skill was inspired by and developed from the workflow in [irons163/three-tier-agent-orchestrator](https://github.com/irons163/three-tier-agent-orchestrator). The upstream repository did not contain a license file when this package was prepared. Keep this repository private unless and until redistribution terms are confirmed with the upstream author.

See [NOTICE.md](NOTICE.md) for provenance and redistribution guidance. No open-source license is asserted by this repository.
