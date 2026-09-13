# Astra–Sol–Luna Orchestrator

A reusable Codex Skill for main-thread orchestration with dynamic routing between GPT-5.6 Sol and GPT-5.6 Luna workers.

## Highlights

- Keeps architecture, important decisions, review, and final acceptance in the main thread.
- Routes work by reasoning difficulty, risk, reversibility, and verification quality rather than code volume alone.
- Supports `NORMAL`, `CONSERVATIVE`, and `LOW_BUDGET` modes.
- Escalates Luna to Sol, or Sol Medium to Sol High, only after classifying the failure.
- Limits orchestration to a depth-1 star topology, three concurrent workers, and two review rounds per task.
- Requires explicit worker contracts and evidence-based acceptance.
- Reports every worker attempt with the exact dispatched model, reasoning effort, result, and validation evidence.

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

## Upstream and redistribution note

This Skill was inspired by and developed from the workflow in [irons163/three-tier-agent-orchestrator](https://github.com/irons163/three-tier-agent-orchestrator). The upstream repository did not contain a license file when this package was prepared. Keep this repository private unless and until redistribution terms are confirmed with the upstream author.

See [NOTICE.md](NOTICE.md) for provenance and redistribution guidance. No open-source license is asserted by this repository.
