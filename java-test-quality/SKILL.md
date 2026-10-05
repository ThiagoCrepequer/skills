---
name: java-test-quality
description: Teach, design, write, strengthen, or review trustworthy tests for Java and Quarkus code. Use for regression tests, weak or false-positive tests, JUnit strategy, Quarkus component/HTTP/persistence/security tests, boundary analysis, state isolation, concurrency, JaCoCo coverage quality, PIT mutation testing, or test-quality audits. Do not use merely to run an unchanged suite.
license: MIT
---

# Java and Quarkus Test Quality

Produce executable evidence that promised behavior works and that plausible defects would be detected. Optimize for confidence per maintenance cost, not test count or line coverage.

## Define a good test

A good test is an executable example of a contract. It establishes a meaningful precondition, performs one coherent stimulus, and checks externally observable consequences with an independently justified oracle.

It must be both:

- **bug-sensitive:** a realistic defect in the protected rule makes it fail;
- **refactor-tolerant:** an internal rewrite that preserves the contract keeps it passing.

Quality does not come from JUnit, Mockito, `@QuarkusTest`, assertion count, coverage, or mutation score. Those tools support the causal story; they do not create it.

Before authoring or reviewing substantive tests, read [references/good-test-principles.md](references/good-test-principles.md).

## Discover the real contract

Inspect the requirement, production diff, callers, neighboring tests, build configuration, and project conventions before editing. Do not infer behavior solely from the method under test. Do not add libraries, plugins, thresholds, or CI configuration unless the user authorized tooling changes.

For each material promise identify:

- observer: domain caller, API consumer, database, security boundary, or external system;
- precondition and stimulus;
- success result, rejection semantics, state transition, and prohibited effects;
- equivalence partitions and exact transition boundaries;
- plausible defects the test must discriminate;
- lowest test layer faithful to the risk.

Use a compact behavior matrix when several rules or paths interact. It is a reasoning aid, not mandatory ceremony. Do not generate generic null/corrupt cases unless the contract accepts, rejects, or is realistically exposed to them.

## Select the faithful layer

Choose the lowest layer that can observe the real failure, then add a higher layer only when framework wiring is itself a risk:

- pure rule/value object/state transition: plain JUnit;
- CDI selection, interceptor, config injection, or small bean graph: `@QuarkusComponentTest` when available;
- SQL, Panache/JPA mapping, constraints, transactions, locking, or database-specific types: real supported database through the existing Dev Services/Testcontainers harness;
- validation, JSON, HTTP status/headers, security, filters, exception mapping, or transaction boundary: `@QuarkusTest` with the project's HTTP client;
- packaged/native/runtime behavior: existing integration-test layer only when that deployment boundary matters;
- outbound HTTP/messaging: real adapter plus a controllable remote stub or broker boundary.

Read [references/quarkus-test-layers.md](references/quarkus-test-layers.md) whenever framework, database, HTTP, security, or integration behavior is involved.

## Build a discriminating oracle

Name tests after the rule and outcome, not the production method. Keep Given-When-Then or Arrange-Act-Assert visible through structure; comments are optional.

Assert every semantically relevant facet of the coherent outcome:

- exact value or meaningful fields rather than only non-null, presence, size, or status;
- collection contents, multiplicity, filtering, and ordering only when promised;
- domain exception type and stable semantics, plus unchanged state and absent side effects;
- persisted values, constraints, version/state transitions, and transaction result when persistence is the behavior;
- outbound payload, destination/key, cardinality, and forbidden duplicate calls when the interaction is the contract;
- authorization and tenant/owner isolation with competing otherwise-valid data.

One test may contain several related assertions when they describe one atomic business outcome. Avoid tests whose expected value calls or reproduces the production algorithm. Read [references/assertions-fixtures-doubles.md](references/assertions-fixtures-doubles.md) for oracle, fixture, builder, mock, fake, and interaction guidance.

## Make plausible defects observable

For every changed decision, consider counterfactuals such as inverted/broadened comparisons, missing filters, wrong tenant, null/default returns, duplicates, swallowed exceptions, partial writes, wrong error mapping, retry off-by-one, or lost terminal events. The test must fail for relevant counterfactuals even when no mutation tool is installed.

For a regression, establish red/green evidence when practical: the new test fails in the known-bad state and passes with the fix. Do not leave temporary source mutations in the worktree.

When PIT already exists or mutation configuration is requested, use it to sample discrimination strength on changed business code. Triage survivors individually; do not chase 100% blindly. Read [references/mutation-and-metrics.md](references/mutation-and-metrics.md).

When configuring or upgrading tools, consult [references/sources.md](references/sources.md) and the project's pinned versions rather than guessing current options.

## Enforce isolation and determinism

Each test must create or explicitly receive all mutable state it relies on. Builders may share immutable defaults but must return fresh aggregates. Restore mocks, static state, system properties, locale, security identity, clocks, executors, caches, database rows, and external resources at the correct scope.

Tests must pass alone, with neighbors, repeatedly, and under supported order/parallel settings. Never repair leakage with test ordering. Replace sleeps with latches, barriers, controllable futures, virtual/fixed clocks, or bounded observation of a real completion condition. Preserve the seed for generated data.

Read [references/advanced-risks.md](references/advanced-risks.md) for time, transactions, concurrency, idempotency, retries, streams, and tenant/security scenarios.

## Verify proportionally

Use cheap evidence first:

1. Run the changed test and confirm the intended assertion path executes.
2. Run the nearest affected package/module suite.
3. Inspect changed-code branches and missed complexity when coverage is configured.
4. Run targeted PIT when configured or requested; inspect survivors, no-coverage, timeout, and errors.
5. Exercise repeated/adversarial schedules or integration/package layers when the risk warrants them.

Never report `No tests found`, an interrupted run, or a stale earlier run as passing. Separate unrelated baseline failures. Do not claim a full suite, native build, PostgreSQL behavior, or mutation gate unless it actually completed at that scope.

## Review and report

Use [references/review-rubric.md](references/review-rubric.md) before delivery or when auditing existing tests. Block acceptance on a false-positive oracle, wrong layer, missing material boundary, state leakage, arbitrary sleep, swallowed/unawaited async work, or an unexplained non-equivalent survivor in changed critical logic.

Report:

- contracts, boundaries, and prohibited outcomes proved;
- chosen test layers and why they are faithful;
- exact commands, counts, and results;
- coverage/mutation scope and categories when run;
- accepted survivor/exclusion with concrete justification;
- what was not run and what the tests do not prove.
