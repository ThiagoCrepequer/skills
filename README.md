# Java Test Quality

`java-test-quality` is a Codex skill for designing, writing, strengthening, and reviewing trustworthy tests in Java and Quarkus projects.

Its purpose is not to maximize test count or line coverage. It helps a coding agent produce executable evidence that a business or technical contract works and that realistic defects would be detected.

## Why this skill exists

A test can execute every line and still prove almost nothing. Common false positives include:

- asserting only that a result is non-null or non-empty;
- verifying an internal mock call without checking the outcome;
- mocking the repository, mapper, validator, or adapter whose behavior is supposedly under test;
- using fixtures that contain no competing data, so missing filters remain invisible;
- relying on test order, shared state, wall-clock time, or arbitrary sleeps;
- treating a coverage percentage or mutation score as proof of correctness.

This skill teaches the agent to recognize and avoid those patterns.

## Core philosophy

A good test is an executable example of a contract. It establishes a meaningful precondition, performs one coherent stimulus, and checks observable consequences with an independently justified oracle.

The skill applies two complementary standards:

- **Bug-sensitive:** a realistic defect in the protected behavior makes the test fail.
- **Refactor-tolerant:** an internal rewrite that preserves the contract keeps the test passing.

Mutation testing supports this philosophy by sampling whether tests distinguish changed logic. It is a quality signal, not the definition of quality and not a reason to pursue 100% blindly.

## What it covers

- Behavior-oriented test design and Given-When-Then reasoning.
- Strong assertions and independently derived expected values.
- Equivalence partitions, exact boundaries, transitions, and prohibited outcomes.
- Plain JUnit tests for domain rules and deterministic logic.
- Quarkus component tests for CDI, interceptors, and configuration behavior.
- `@QuarkusTest` for HTTP, validation, serialization, security, and application wiring.
- PostgreSQL/Testcontainers or Dev Services for real persistence semantics.
- Authorization, tenant, owner, and location isolation.
- Transactions, retries, idempotency, time, streams, and concurrency.
- Test fixtures, builders, stubs, fakes, mocks, and spies.
- JaCoCo coverage interpretation and PIT mutation-survivor triage.
- Detection of flaky tests, state leakage, weak oracles, and mocked-away behavior.
- Honest reporting of validation scope and residual risk.

## How the skill works

The entrypoint is [`SKILL.md`](SKILL.md). It gives the agent the essential workflow:

1. Discover the real contract and its observer.
2. Identify plausible defects and meaningful boundaries.
3. Choose the lowest test layer faithful to the risk.
4. Build fixtures that make wrong behavior observable.
5. Write a discriminating oracle.
6. Isolate state, time, and resources.
7. Verify proportionally and report exactly what was proved.

Detailed guidance is loaded from `references/` only when relevant:

- `good-test-principles.md`: what makes a test trustworthy.
- `quarkus-test-layers.md`: choosing unit, component, database, HTTP, security, or runtime tests.
- `assertions-fixtures-doubles.md`: strong oracles, test data, builders, and doubles.
- `advanced-risks.md`: time, transactions, concurrency, idempotency, streams, and security.
- `mutation-and-metrics.md`: JaCoCo and PIT without metric gaming.
- `review-rubric.md`: blocking criteria for test-quality reviews.
- `sources.md`: primary documentation for tool configuration.

## When to use it

Invoke the skill when asking Codex to:

- write tests for a feature or bug fix;
- review whether existing tests can produce false positives;
- strengthen a weak regression test;
- choose between unit, component, persistence, HTTP, or integration tests;
- improve test isolation or diagnose flakiness;
- interpret coverage or mutation results;
- audit the quality of a Java or Quarkus test suite.

It is intentionally not used merely to run an unchanged test suite.

## Installation

Install from GitHub with the Codex skill installer:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo ThiagoCrepequer/java-test-quality \
  --path . \
  --ref main \
  --name java-test-quality
```

The skill is installed at:

```text
$CODEX_HOME/skills/java-test-quality
```

When `CODEX_HOME` is unset, Codex uses `~/.codex`, so the default path is `~/.codex/skills/java-test-quality`.

It becomes available to Codex on the next turn or session after installation.

## Invocation

Explicit invocation:

```text
$java-test-quality Write regression tests for this Quarkus service and prove the authorization boundary.
```

Other examples:

```text
$java-test-quality Review these JUnit tests for false positives and state leakage.

$java-test-quality Design the correct test layers for this repository query and REST endpoint.

$java-test-quality Triage these surviving PIT mutants and strengthen only the relevant tests.
```

The skill also supports automatic selection when a request clearly concerns Java or Quarkus test quality.

## Expected agent output

When work is complete, the agent should report:

- contracts, boundaries, and prohibited outcomes proved;
- why each test layer is faithful to the risk;
- exact commands, test counts, and results;
- mutation and coverage scope when those tools were run;
- justified survivors or exclusions;
- baseline failures, validations not run, and what the tests do not prove.

The result should be a test suite that fails for meaningful regressions, remains stable through behavior-preserving refactors, and communicates its confidence boundary honestly.
