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

Task size alone does not choose Sol.

## Reasoning effort

- Luna default: `low` for searches, test runs, summaries, and highly mechanical work; use `medium` when a clear implementation still needs local judgment. Increase further only when the runtime supports it and an observed need justifies the cost.
- Sol default: `medium`. Use `high` only for genuinely difficult reasoning, security-sensitive logic, subtle concurrency, or a demonstrated medium-effort reasoning failure.
- Explicitly pass the selected model and effort on every spawn. Give task names a stable suffix such as `_luna_low`, `_luna_medium`, `_sol_medium`, or `_sol_high` that matches the actual arguments.

## Budget modes

### NORMAL

- Main/Astra: planning, architecture, important reviews, integration, acceptance.
- Sol: primary substantial implementation and complex debugging.
- Luna: exploration, testing, mechanical tasks, and low-risk implementation.

### CONSERVATIVE

- Main/Astra: initial plan, architecture, major checkpoints, final review.
- Sol: difficult implementation, high-risk code, and work Luna cannot complete reliably.
- Luna: all other tasks that have clear boundaries and adequate validation.

### LOW_BUDGET

- Main/Astra: initial decomposition, critical architecture decisions, important checkpoints, final acceptance.
- Sol: critical implementation, difficult bugs, security-sensitive code, and escalation after a genuine Luna capability failure.
- Luna: every task that can be verified reliably.

An explicit `BUDGET_MODE=NORMAL|CONSERVATIVE|LOW_BUDGET` in the user's request overrides the project default for that request. Never downgrade critical architecture to Luna solely to save quota.

## Quota pressure

When the user reports tight quota or a runtime signal indicates it, re-route before stopping:

- Sol pressure: move routine work and medium work with strong validation to Luna; retain hard work on Sol and pair critical work with focused main review.
- Astra pressure: reduce checkpoint frequency and reserve the main thread for architecture, critical decisions, and final review.
- If the runtime merely does not expose the main model identity, continue with the main thread owning architecture. If the runtime explicitly rejects Astra and the user's constraint requires Astra itself for a major architecture decision, explain that blocker rather than assigning the decision to Luna.

Do not repeatedly probe account limits. Use an available read-only usage tool only when the request or observed failure makes quota material.

## Failure classification and escalation

Classify a failed attempt before changing models:

1. **Specification, plan, architecture, or requirement failure:** return to the main thread for clarification or replanning.
2. **Environment, permission, credential, dependency, model availability, quota, or flaky-tool failure:** re-route when policy permits; otherwise fix or report the external cause. More reasoning is not the remedy.
3. **Verification failure with a clear localized fix:** rework on the same worker class, within the review-round limit.
4. **Reasoning-capability failure:** Luna -> Sol Medium; Sol Medium -> Sol High.

After two unsuccessful review/rework rounds for one task, stop recursive retries and replan in the main thread.

## Dynamic downgrade

If investigation shows that a Sol-assigned task or its remaining siblings are mechanical, rule-bound, reversible, and strongly testable, route the later work to Luna. Record the change in the task graph. The goal is the lowest-cost model that can finish reliably.
