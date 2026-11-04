---
name: solution-quality
description: Guide implementation, architecture, bug fixes, and refactoring toward evidence-backed, testable-by-construction code that addresses the underlying problem, preserves contracts, and keeps complexity proportional to the task. Use when defining what high-quality code is, choosing design patterns vs. avoiding anti-patterns, separating pure domain logic from I/O, designing deep module interfaces, investigating defects, or assessing workarounds. Does not own release, Jira, or PR workflows.
license: MIT
---

# Solution Quality

Choose the simplest design that satisfies the actual requirement, addresses the demonstrated cause, makes invalid states difficult or impossible to represent, and is testable by construction with evidence proportional to its risk. Optimize for **leverage, locality, correctness, and maintainability** under the task's constraints—not the smallest diff, an arbitrary line-count rule, or an imagined universal framework.

## Apply proportionately

Read applicable repository instructions and reuse evidence already established in the task. Respect the user's scope and authorization. This skill complements an active workflow (and pairs directly with `test-quality` and `tdd`); it does not restart its steps, duplicate its gates, or grant permission to publish or operate production systems.

- For a clear, localized change, inspect the relevant contract, implement it cleanly at the owning seam, and run focused checks. No formal plan, alternative count, research quota, or separate report is required.
- Investigate more deeply when behavior is unclear, evidence conflicts, or the change affects shared contracts, concurrency, security, data integrity, multi-step transactions, or operational limits. File count alone does not determine risk.
- Read only the matching cases in [references/decision-examples.md](references/decision-examples.md) when deciding whether a retry, fallback, cache, adapter, refactor, transaction boundary, or mitigation is justified.
- Read [references/code-design-and-patterns.md](references/code-design-and-patterns.md) when structuring modules, untangling mixed business/I/O logic, choosing design patterns by problem pressure, or auditing implementation anti-patterns.

## Define high-quality code (the five structural properties)

Quality is not adherence to a rigid catalog of GoF templates; it is the presence of five structural properties that keep software adaptable and testable across any stack:

1. **Deep modules over shallow wrappers (High Leverage, High Locality):**
   A good module hides complex mechanics and enforces invariants (such as tenant isolation, safe pagination, or transaction scoping) behind a small, intuitive interface. Callers gain **leverage** (much capability per unit of interface learned) and maintainers gain **locality** (invariants live in one place, not copy-pasted across $N$ callers). Apply the **deletion test**: if removing an abstraction makes the system simpler, it was a shallow pass-through; if removing it forces $N$ callers to repeat complex guards, it is earning its keep.
2. **Functional Core, Imperative Shell (Testable by Construction):**
   Separate *deciding what should happen* from *executing I/O and side effects*. Keep complex branching, state-machine precedence, recurrence/schedule math, layout geometry, and diffing inside pure, deterministic functions over plain data structures (**Functional Core**). Keep HTTP handlers, database queries, and UI components as thin orchestrators that load state, invoke the pure core, and persist or render results (**Imperative Shell**). When a behavior requires ten mocks to test, treat that friction as a design smell: extract the decision core.
3. **Parse, don't validate (Make Illegal States Unrepresentable):**
   Validate and normalize raw inputs once at the system boundary (seam), transforming untrusted primitives, strings, or loose maps into canonical, strongly typed domain structures. Do not pass raw composite strings (`"${id}_${timestamp}"`), unparsed JSON text, or ambiguous optional flags deep into services where every caller must defensively split strings or `try { JSON.parse }`. Model independent facts independently (e.g., separate lifecycle timestamps rather than a single overwritten status string).
4. **Atomicity and single source of truth:**
   Multi-step mutations that belong to one business operation must share a single transactional boundary (`Unit of Work`). Never leave half-updated state when step 2 of 3 fails, and never maintain parallel columns or client/server caches that silently drift without a defined reconciliation rule.
5. **Truthful failure over compensating cascades:**
   Enforce preconditions at their owning boundary and fail explicitly with structured domain errors (`code`, `status`, typed exception) when required invariants or dependencies fail. Never swallow errors to return fabricated domain data (e.g., hardcoded fake users/items when an API call fails), and never branch on human-readable error message substrings (`err.message.includes(...)`) to drive control flow.

## Establish the problem and its owner

Identify the observable outcome, the affected inputs and users, and the contracts that must remain true. For a feature, establish its acceptance behavior and integration seams. For a defect, distinguish the symptom, the suspected mechanism, and the evidence connecting them.

Trace the relevant path from input through the responsible module to the failure. A stack trace identifies where an error surfaced; it does not by itself identify where it originated. Fix a bad producer at its boundary while retaining necessary validation at trust boundaries. Do not broaden the investigation to unrelated parts of the system.

Reproduce the defect when practical. Otherwise, use targeted inspection, controlled experiments, or existing telemetry to test a specific hypothesis. Label uncertainty; do not invent a root cause. When an attempted fix fails, revisit the hypothesis before adding another compensating patch.

## Resolve uncertainty with useful resources

Start with the owning code, callers, tests, configuration, installed versions, and project decisions (`GLOSSARY.md`, ADRs). Consult relevant available skills and documentation when they can change the implementation or verification. For version-sensitive APIs, unfamiliar behavior, or external defects, verify against official documentation, source, release notes, or a reproducible upstream issue for the actual version.

Form a concrete question before a lookup and stop researching once the material uncertainty is resolved. Do not browse, load every skill, introduce tools, or install dependencies simply to demonstrate diligence. Keep confidential data out of external searches.

## Choose patterns by problem pressure, never by speculation

Compare alternatives only when a consequential tradeoff exists. Prefer supported capabilities and established project seams when they fit. Apply patterns only when a concrete structural pressure demands them (see [references/code-design-and-patterns.md](references/code-design-and-patterns.md)):

- **Pure Policy / Calculator Function:** when branching, math, or state derivation grows complex inside an I/O handler or UI component.
- **Port & Adapter (Seam):** when a boundary must genuinely vary (e.g., production external service vs. deterministic test fake). **One adapter means a hypothetical seam; two adapters mean a real one.** Never create single-implementation interfaces just for ceremony.
- **Scoped Context / Boundary Wrapper:** when an invariant (auth, tenant scope, tracing, transaction) must apply compulsorily without threading context parameters (`db`, `orgId`, `userId`) through every function signature (*Context Plumbing*).
- **State / Query Factory:** when 2+ workflows share the exact same state machine or invariant (e.g., server-side pagination, tri-state sorting, filter reset), whereas superficially similar code with different business policies should stay separate.
- **Batch Loader / Projection (`WHERE id IN (...)` / `JOIN`):** when enriches or link checks would otherwise issue $N+1$ queries inside loops.

## Never deform production code to appease weak test doubles

High-quality implementation and high-quality testing (`test-quality`) must never undermine each other:

- **No test-only backdoors in production:** Never add `if (!db) return fakeSuccess` or silent no-ops in production services merely so under-configured unit tests can pass without providing required dependencies.
- **No in-memory full-table scans to work around mock limitations:** Never replace SQL `WHERE`, `JOIN`, `GROUP BY`, or `ORDER BY` operations with unbounded `findMany()` table dumps and JavaScript `.filter()` / `.slice()` loops just because a hand-rolled test mock cannot parse SQL expressions. Upgrade the test harness (or test against a real embedded/containerized database) and keep production queries bounded and index-friendly.

## Distinguish durable solutions from temporary mitigation

A compatibility boundary or resilience mechanism can be a permanent design choice. A temporary workaround needs an evidenced constraint that prevents the direct fix, an isolated implementation at the boundary, meaningful validation, and a concrete removal condition. Record that rationale near the code or in the existing task/PR.

If restoring service urgently is within scope, a bounded mitigation may precede diagnosis; state what it restores, what remains unresolved, and how to verify and reverse it. Do not claim that mitigation proves the cause is fixed.

## Verify and finish

Choose checks that could expose the original failure or invalidate the solution (and apply `test-quality` standards when writing or updating tests):

- Exercise the real mechanism when it matters: database constraints and transactions for persistence invariants, generated SQL and batch counts for query behavior, and controlled response order for asynchronous UI.
- Verify both the positive contract (expected return/state) and negative invariants (sibling fields preserved on partial `PATCH` updates, no cross-tenant leakage, no partial writes on failure).
- Inspect the final diff for hidden behavior changes, redundant compensations, `N+1` query loops, and unnecessary complexity.

Report the result, the decisive rationale, verification actually performed, and material remaining limits in the user's language.
