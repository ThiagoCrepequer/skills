---
name: test-quality
description: Teach, design, write, strengthen, or review trustworthy, high-ROI tests in any language or framework. Use when defining what a good test is, writing unit/component/integration/contract tests, eliminating test theatre and AI test-slop, choosing test seams and layers, designing discriminating oracles and competing fixtures, auditing false-positive or brittle suites, or interpreting coverage and mutation scores. Do not use merely to run an unchanged test suite.
license: MIT
---

# Test Quality

Produce executable evidence that promised behavior works and that plausible defects would be detected. Optimize for **confidence per maintenance cost**, not test count, assertion count, snapshots, or raw line coverage.

Production code is an asset; every test is a liability with ongoing maintenance cost. A weak test carries **double negative ROI**: it costs time to write and maintain while creating false confidence that unverified paths work.

## Define a good test

A good test is a **controlled causal experiment** over a public contract. It establishes a meaningful precondition (including competing data), performs one coherent stimulus at a public seam, and verifies externally observable consequences—both required outcomes and prohibited side effects—with an independently justified oracle.

Every worthwhile test must simultaneously satisfy two properties:

- **Bug-sensitive (Protection against regressions):** a realistic defect in the protected rule, boundary, query, or state transition makes the test fail.
- **Refactor-tolerant (Resistance to refactoring):** an internal rewrite (extracting helpers, changing data structures, splitting components, rewriting queries) that preserves the observable contract keeps the test passing.

Quality never comes from a testing library, mock framework, assertion count, coverage percentage, or mutation tool. Those tools support the causal story; they do not create it.

Before authoring or auditing substantive tests, read [references/good-test-principles.md](references/good-test-principles.md) and [references/anti-slop-catalog.md](references/anti-slop-catalog.md).

## Pass the three-question ROI gate before writing any test

Before writing or keeping a test, answer three blocking questions:

1. **Counterfactual sensitivity:** *Which plausible defect in production code makes this test fail?*
   If the only defect it catches is a missing export, a renamed symbol, or a type mismatch already caught by the compiler/typechecker, **do not write the test**.
2. **Refactor tolerance:** *If the internal implementation is rewritten while preserving the public contract, does this test stay green?*
   If it breaks because it asserts private methods, internal hook/helper call order, or incidental markup/SQL formatting, move the test to the public seam.
3. **Layer fidelity:** *Does the real subsystem capable of causing the defect participate in the test, or was it replaced by a mock/fake shortcut?*
   If a test claims to prove filtering, sorting, aggregation, serialization, authorization, or UI state transitions while a mock or hand-rolled fake bypasses or re-implements that exact logic, the test is tautological theatre.

## Discover the real contract

Inspect the requirement, production diff, domain glossary/ADRs, callers, neighboring tests, and project conventions before editing. Do not infer the contract solely from the implementation under test—otherwise bugs in the implementation become codified as expected behavior. Do not add test libraries, plugins, thresholds, or CI configuration unless authorized.

For each material promise identify:

- **observer:** domain caller, API consumer, end user, assistive technology, persistence boundary, security boundary, or external protocol;
- **activating precondition and competing state:** what must exist to trigger the rule, and what competing valid records/states must exist so filtering, scoping, or precedence is actually tested;
- **stimulus:** the single command, query, request, event, or user workflow;
- **observable outcome:** returned value, persisted state transition, rendered UI/accessibility state, or outbound payload;
- **prohibited effects:** untouched competing records, unchanged fields on partial updates, absent duplicate requests/calls, rolled-back state on failure, and cross-tenant/cross-user isolation;
- **equivalence partitions and exact boundaries:** below, at, and above transition thresholds;
- **lowest faithful layer:** the cheapest execution boundary where the real failure mode can occur.

Use a compact behavior matrix when multiple conditions, roles, or lifecycle states interact. Do not generate generic null/empty/loading quotas unless the unit under test owns that contract.

## Select the faithful layer and seam

Test at **seams**—stable public boundaries where behavior is observable without reaching inside internals. Push complex branching, calculation, state derivation, and layout/scheduling math into a **pure functional core** so most edge cases can be verified with fast, zero-mock unit tests, while integration/component tests verify the **imperative shell** (wiring, persistence, network, auth, UI).

Choose the lowest layer that includes the risky subsystem:

- **pure domain rule, state derivation, calculator, parser, or validator:** direct unit test with plain inputs and exact literal outputs;
- **service or module orchestrating domain rules and persistence:** service/module seam against a real test database or a semantically faithful fake;
- **SQL predicates, joins, sorting, aggregations, constraints, transactions, or row-level security:** real database engine (or embedded equivalent), never an AST-sniffing mock or a fake that re-implements aggregations in memory;
- **HTTP routing, serialization, validation, auth/RBAC middleware, or error mapping:** full request-to-response pipeline through the application router;
- **interactive UI, form validation, accessibility semantics, and network-driven state:** rendered component/screen through user-facing queries and network boundary interception;
- **browser layout, real focus navigation, storage/cookie policies, or multi-page journeys:** real browser harness when simulated DOM cannot represent the risk.

Read [references/seams-layers-and-doubles.md](references/seams-layers-and-doubles.md) when choosing layers, designing seams, or placing test doubles.

## Build a discriminating oracle and adversarial fixtures

Name tests after the **domain promise, activating condition, and observable outcome**, not the method name or internal choreography. Keep `Given-When-Then` or `Arrange-Act-Assert` clear through structure.

Assert every semantically relevant dimension of the outcome:

- **exact domain values** rather than only `not null`, `defined`, `status 200`, `length > 0`, or `toHaveBeenCalled()`;
- **competing data exclusion:** prove that non-matching rows, other tenants' records, or unauthorized items present in the fixture are excluded;
- **state integrity on partial updates and rejections:** prove that updating field A preserves field B, that cancelling/deleting releases capacity or updates lifecycle state, and that rejected operations (`400`/`403`/`404`/domain errors) leave persisted state and outbound channels untouched;
- **cardinality and ordering:** prove absence of duplicates and exact ordering when promised (using insertion order different from expected sort order);
- **independent expected values:** derive expected outputs from hand-calculated examples, domain specifications, or invariants—never by copying or calling the production formula.

Read [references/oracles-fixtures-and-boundaries.md](references/oracles-fixtures-and-boundaries.md) for oracle design, adversarial fixtures, boundary analysis, and negative assertions.

## Eliminate test theatre and AI test-slop

Reject and remove tests that inflate suite counts without catching defects:

- **Inline self-testing theatre:** defining mock arrays/objects inside a test file and asserting `.filter()` / `.reduce()` / transformations written inline in the test body without invoking production code.
- **Compiler/reflection redundancy:** asserting `expect(Component).toBeDefined()`, `expect(typeof fn).toBe("function")`, or checking that static schema/object properties exist when static typechecking already guarantees them.
- **Tautological oracles:** computing `expected` in the test using the same expression or helper as production code.
- **Tautological fakes and over-mocking:** mocking the unit under test, configuring a mock to return the exact value immediately asserted, matching query parameters against *any* column in a fake DB, or re-implementing SQL metrics/sorting in test-helper code instead of executing the production query.
- **Unasserted or choreography-only spies:** creating spies that are never asserted, or asserting internal call choreography while ignoring the externally visible result.
- **Metadata-only smoke placeholders:** testing only static page titles or route metadata constants while leaving forms, loaders, permissions, and state transitions untested.

See [references/anti-slop-catalog.md](references/anti-slop-catalog.md) for concrete before-and-after replacements.

## Enforce isolation, determinism, and async correctness

Every test must own its mutable state and pass alone, with neighbors, repeatedly, and in any execution order:

- **Fresh state per test:** create fresh database/fake state, caches, stores, routers, and mutable fixtures per test. Builders may share immutable defaults but must return fresh instances.
- **Controlled time and randomness:** inject or freeze clocks, explicit timezones, and deterministic seeds. Never assert against wall-clock `now()` with loose tolerances.
- **Deterministic async & races:** await every stimulus and terminal state. Use explicit deferred promises, latches, or barriers to test race conditions, stale-response suppression, and timeouts. Never use arbitrary `sleep`/`setTimeout` delays to wait for async work, and never put side effects inside retrying assertion callbacks.
- **Clean restoration:** restore mocks, spies, environment variables, timers, network handlers, and storage after each test.

Read [references/isolation-determinism-and-async.md](references/isolation-determinism-and-async.md) whenever time, concurrency, transactions, caches, or async workflows are involved.

## Verify proportionally

Gather cheap, high-signal evidence first:

1. Run the target test and verify it actually executes the intended assertions (and fails when the protected production logic is broken).
2. Run the nearest affected module/package suite.
3. Inspect changed-code branch coverage and missed decision paths when coverage tooling exists.
4. Run targeted mutation testing (e.g., PIT, Stryker, mutmut) when configured or requested, or perform explicit counterfactual review on every changed branch.
5. Run higher-fidelity integration/browser suites or repeated/shuffled runs when wiring, SQL, or concurrency risks apply.

Never report `No tests found`, an interrupted run, or stale terminal output as passing. Separate pre-existing baseline failures from regressions caused by the current change.

## Review and report

Use [references/review-rubric-and-metrics.md](references/review-rubric-and-metrics.md) before finishing or when auditing an existing test suite. Block approval on any false-positive oracle, inline test theatre, mocked-away risk subsystem, missing competing fixture, leaked mutable state, arbitrary sleep, unawaited async work, or unexplained mutant survivor in critical logic.

Report concisely:

- contracts, invariants, boundaries, and prohibited outcomes proved;
- chosen test layers/seams and why they are faithful to the risk;
- exact commands run, test counts, and results;
- coverage or mutation findings when applicable;
- what was not run and what residual risks remain unproved.
