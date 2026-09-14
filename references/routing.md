# Dynamic routing

## Routing order

For each task, evaluate:

- reasoning difficulty
- implementation difficulty
- ambiguity
- blast radius
- security risk
- reversibility
- verification quality
- task size
- dependency complexity

Verification quality is the strongest cost-control signal. A large mechanical change with complete automated validation can go to Luna. A small authentication or authorization change belongs with Sol and requires focused main-thread review.

Keep a task in the main thread when it primarily decides architecture, resolves ambiguous requirements, authorizes destructive action, spans multiple modules, defines a major migration, or makes the final acceptance decision.

## Worker selection

Choose `gpt-5.6-luna` when the task is clear, bounded, reversible, and cheaply verifiable. Typical work includes repository exploration, file location, repetitive edits, mechanical refactors, formatting, lint fixes, documentation, simple tests, test execution, log collection, dependency inventory, low-risk configuration, and large rule-driven changes with strong automated checks.

Choose `gpt-5.6-sol` when the task requires substantial reasoning or implementation: complex refactors or debugging, algorithms, core backend/frontend logic, database or concurrency work, performance, security-sensitive implementation, difficult test failures, or deep code review.

Choose a bounded `gpt-6-astra` worker only when the main thread is not already Astra, the spawn API supports explicit model and effort selection, and an architecture consultation, high-risk review, or difficult synthesis materially benefits from Astra. The worker advises the main thread; it does not become the orchestrator or own final acceptance.

Task size alone does not choose Sol.

## Cost-aware reasoning policy

For automatic routing, classify `gpt-6-astra` and `gpt-5.6-sol` as expensive models and `gpt-5.6-luna` as the cost-efficient model.

- Astra default: `medium` for bounded architecture consultation or high-risk review; use `low` for a simpler focused check. Automatic ceiling: `medium`.
- Sol default: `medium`; use `low` when the task is straightforward. Automatic ceiling: `medium`.
- Luna default: `low` for searches, test runs, summaries, and highly mechanical work; use `medium` when a clear implementation still needs local judgment. `high` is allowed only for difficult work that remains tightly bounded and strongly verifiable. Automatic ceiling: `high`.
- Never select `high`, `xhigh`, `max`, or `ultra` for Astra or Sol merely because a task is difficult or a prior attempt failed. A higher-than-`medium` expensive-model effort requires the user to explicitly name both the exact model and the exact higher effort level for the current task.
- Never select `xhigh`, `max`, or `ultra` automatically for Luna.
- Prefer explicitly passing the selected model and effort on every spawn. Use guaranteed inherited values only under the legacy compatibility exception in `SKILL.md`. Give task names a stable suffix such as `_luna_low`, `_luna_medium`, `_luna_high`, `_sol_medium`, or `_astra_medium` that matches the actual configuration.

The Skill cannot lower or replace the model or effort of a main-thread turn that is already running. The UI, CLI, or applicable `config.toml` selects that value before Skill instructions execute. When the active main-thread setting is observable and violates the default ceiling without an explicit user override, do not compound the cost with high-effort expensive workers; report the mismatch and recommend changing the session or configuration for the next turn.

## Main-thread model and cross-model dispatch

The Skill does not require an Astra main thread. A Sol main thread may orchestrate the workflow and, when the active spawn schema accepts an explicit `model` and `reasoning_effort`, call an Astra worker for one bounded consultation or review. The Sol main thread still owns planning, integration, acceptance, and the user-facing report. Record the Astra dispatch and its reasoning effort exactly like any other worker attempt.

Do not spawn Astra merely to simulate an Astra-led main thread. Use it only when its bounded contribution is worth the additional usage and can be reviewed by the actual main thread.

## Budget modes

### NORMAL

- Main thread: planning, architecture, important reviews, integration, acceptance.
- Sol: primary substantial implementation and complex debugging.
- Luna: exploration, testing, mechanical tasks, and low-risk implementation.

### CONSERVATIVE

- Main thread: initial plan, architecture, major checkpoints, final review.
- Sol: difficult implementation, high-risk code, and work Luna cannot complete reliably.
- Luna: all other tasks that have clear boundaries and adequate validation.

### LOW_BUDGET

- Main thread: initial decomposition, critical architecture decisions, important checkpoints, final acceptance.
- Sol: critical implementation, difficult bugs, security-sensitive code, and escalation after a genuine Luna capability failure.
- Luna: every task that can be verified reliably.

An explicit `BUDGET_MODE=NORMAL|CONSERVATIVE|LOW_BUDGET` in the user's request overrides the project default for that request. Never downgrade critical architecture to Luna solely to save quota.

## Quota pressure

When the user reports tight quota or a runtime signal indicates it, re-route before stopping:

- Sol pressure: move routine work and medium work with strong validation to Luna; retain hard work on Sol and pair critical work with focused main review.
- Astra-worker pressure: avoid Astra workers except for a bounded critical consultation; keep architecture, critical decisions, and final review in the actual main thread.
- If the runtime merely does not expose the main model identity, continue with the main thread owning architecture. If the runtime explicitly rejects Astra and the user's constraint requires Astra itself for a major architecture decision, explain that blocker rather than assigning the decision to Luna.

Do not repeatedly probe account limits. Use an available read-only usage tool only when the request or observed failure makes quota material.

## Failure classification and escalation

Classify a failed attempt before changing models:

1. **Specification, plan, architecture, or requirement failure:** return to the main thread for clarification or replanning.
2. **Environment, permission, credential, dependency, model availability, quota, or flaky-tool failure:** re-route when policy permits; otherwise fix or report the external cause. More reasoning is not the remedy.
3. **Verification failure with a clear localized fix:** rework on the same worker class, within the review-round limit.
4. **Reasoning-capability failure:** Luna Low/Medium -> Luna High when the task remains bounded and strongly verifiable, or -> Sol Medium when broader reasoning is needed. Sol Medium or Astra Medium -> main-thread diagnosis and replanning. Exceed an expensive model's `medium` ceiling only after an explicit user request naming the exact model and exact higher effort level.

After two unsuccessful review/rework rounds for one task, stop recursive retries and replan in the main thread.

## Dynamic downgrade

If investigation shows that a Sol-assigned task or its remaining siblings are mechanical, rule-bound, reversible, and strongly testable, route the later work to Luna. Record the change in the task graph. The goal is the lowest-cost model that can finish reliably.
