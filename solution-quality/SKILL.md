---
name: solution-quality
description: Guide implementation, bug fixes, and refactoring toward evidence-backed solutions that address the underlying problem, preserve relevant contracts, and keep complexity proportional to the task. Use when choosing or implementing a technical approach, investigating a defect, or assessing a proposed workaround. Does not own release, Jira, or PR workflows.
---

# Solution Quality

Choose the simplest solution that satisfies the actual requirement, addresses the demonstrated cause, and has evidence proportional to its risk. Optimize for correctness and maintainability under the task's constraints, not the smallest diff or an imagined perfect architecture.

## Apply proportionately

Read applicable repository instructions and reuse evidence already established in the task. Respect the user's scope and authorization. This skill complements an active workflow; it does not restart its steps, duplicate its gates, or grant permission to publish or operate production systems.

- For a clear, localized change, inspect the relevant contract, implement it, and run focused checks. No formal plan, alternative count, research quota, or separate report is required.
- Investigate more deeply when behavior is unclear, evidence conflicts, or the change affects shared contracts, concurrency, security, data integrity, or operational limits. File count alone does not determine risk.
- Read only the matching cases in [decision examples](references/decision-examples.md) when deciding whether a retry, fallback, cache, adapter, refactor, or mitigation is justified. These are conditional examples, not mandatory patterns.

## Establish the problem and its owner

Identify the observable outcome, the affected inputs and users, and the contracts that must remain true. For a feature, establish its acceptance behavior and integration points. For a defect, distinguish the symptom, the suspected mechanism, and the evidence connecting them.

Trace the relevant path from input through the responsible component to the failure. A stack trace identifies where an error surfaced; it does not by itself identify where it originated. Fix a bad producer while retaining necessary validation at trust boundaries. Do not broaden the investigation to unrelated parts of the system.

Reproduce the defect when practical. Otherwise, use targeted inspection, controlled experiments, or existing telemetry to test a specific hypothesis. Label uncertainty; do not invent a root cause. When an attempted fix fails, revisit the hypothesis before adding another compensating patch. Keep unrelated findings separate.

## Resolve uncertainty with useful resources

Start with the owning code, callers, tests, configuration, installed versions, and project decisions. Consult relevant available skills and documentation when they can change the implementation or verification. For version-sensitive APIs, unfamiliar behavior, or external defects, verify against official documentation, source, release notes, or a reproducible upstream issue for the actual version. Use web search to locate that evidence; a search snippet is not sufficient support.

Form a concrete question before a lookup and stop researching once the material uncertainty is resolved. Do not browse, load every skill, introduce tools, or install dependencies simply to demonstrate diligence. If a resource is unavailable, use adequate alternatives and state remaining uncertainty. Missing optional guidance is not a blocker; preserve mandatory dependencies of the workflow already in use. Keep confidential data out of external searches.

## Choose and implement the appropriate solution

Compare alternatives only when a consequential tradeoff exists. Prefer supported capabilities and established project seams when they fit. A larger cohesive fix can be simpler than several local exceptions; a small direct implementation can be better than a general framework.

Evaluate the relevant effects: contract correctness, compatibility, ownership, operational cost, testability, and future maintenance. Do not sacrifice a required invariant for a smaller diff or a faster benchmark. Do not preserve a known defective pattern merely for consistency. Explain meaningful departures from conventions.

Keep rules at their responsible boundary and make invalid states and failures explicit. Avoid speculative abstractions, duplicate sources of truth, unrelated refactors, and configuration added only for hypothetical needs. Refactor when the scoped change demonstrably needs it; include affected callers and tests.

Treat retries, sleeps, caches, defaults, type assertions, suppressions, and adapters as prompts to examine intent, not automatic violations. Reject uses that hide a broken contract or bypass a required check. Do not swallow unexpected failures, fabricate successful results, weaken authorization, or change test expectations solely to accept a regression. If the intended contract legitimately changes, update its tests with evidence of that requirement.

## Distinguish durable solutions from temporary mitigation

A compatibility boundary or resilience mechanism can be a permanent design choice. A temporary workaround needs an evidenced constraint that prevents the direct fix, an isolated implementation, meaningful validation, and a concrete removal condition. Record that rationale near the code or in the existing task/PR; do not require a new tracking system.

Check supported configuration or an upgrade before maintaining a workaround, including compatibility and migration costs. Do not assume the latest release is safe or necessary. If restoring service urgently is within scope, a bounded mitigation may precede diagnosis; state what it restores, what remains unresolved, and how to verify and reverse it. Do not claim that mitigation proves the cause is fixed.

Ask only when a missing product decision, material scope expansion, or additional permission is necessary. Present the evidence and recommendation, and continue independent authorized work. Routine engineering choices need no new approval.

## Verify and finish

Choose checks that could expose the original failure or invalidate the solution. For a reproducible bug, prefer a regression that fails before the fix and passes afterward for the expected reason. Reuse sufficient existing coverage. Do not impose new tests on trivial changes merely to satisfy a ritual.

Exercise the real mechanism when it matters: database constraints for concurrent persistence, controlled response order for asynchronous UI, generated SQL for ORM behavior, and representative measurements for performance. Mocks, compilation, or a passing happy path alone do not establish those properties. Run required repository checks and focused adjacent regressions; broaden validation when new risks or failures justify it.

Inspect the final diff for hidden behavior changes, redundant compensations, and unnecessary complexity. Finish when the scoped outcome and relevant invariants have sufficient evidence and required checks are satisfied. Do not keep searching for marginal improvements. When evidence or access is insufficient, identify the precise limitation and safe next step instead of claiming success or blocking unrelated work.

Report the result, the decisive rationale, verification actually performed, and material remaining limits in the user's language. Keep the explanation proportional; reuse the active workflow's report instead of producing a second checklist.
