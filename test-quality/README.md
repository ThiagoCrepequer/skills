# Test Quality

[Install from the skills repository](https://github.com/ThiagoCrepequer/skills)

```bash
npx skills add ThiagoCrepequer/skills --skill test-quality
```

A language- and framework-agnostic agent skill for designing, writing, strengthening, and auditing high-ROI software tests.

## What it does

It teaches coding agents how to answer **"What is a good test?"** and **"How do I write a test worth its maintenance cost?"** across any stack (backend, frontend, domain libraries, APIs, databases, and distributed systems).

It enforces first-principles test engineering:

- **Controlled causal experiments** at stable public seams rather than implementation-coupled unit tests.
- **Counterfactual sensitivity** (would a realistic production bug force this test to fail?) paired with **refactor tolerance** (does an internal rewrite keep it green?).
- **Adversarial fixtures with competing data** so filtering, sorting, tenant isolation, and state transitions are genuinely proved.
- **Positive and negative state oracles** verifying both the promised result and the absence of prohibited side effects.
- **Anti-slop enforcement** that detects and eliminates inline self-testing theatre, compiler-redundant reflection checks, tautological oracles, and fake databases/mocks that bypass the risky subsystem.

## Philosophy

Every test is a liability with ongoing maintenance cost. A test is only worth keeping when it maximizes confidence per maintenance cost:

- **Bug-sensitive:** a plausible defect in a rule, boundary, query, or lifecycle transition makes it fail.
- **Refactor-tolerant:** internal restructuring that preserves observable behavior keeps it passing.
- **Faithful to the risk:** the subsystem capable of causing the defect (SQL engine, serializer, auth guard, state machine, or DOM) participates rather than being mocked away.
- **Isolated and deterministic:** no scenario depends on execution order, shared mutable state, arbitrary sleeps, or wall-clock time.
- **Honest:** coverage and mutation scores are treated as diagnostic signals, never substitutes for a discriminating oracle.

## Contents

The entrypoint is [`SKILL.md`](SKILL.md). Supporting references are loaded on demand:

- [`references/good-test-principles.md`](references/good-test-principles.md): causal structure, the four pillars of test value, counterfactual review, and test economics.
- [`references/anti-slop-catalog.md`](references/anti-slop-catalog.md): concrete taxonomy of low-value and false-positive tests with before/after replacements.
- [`references/seams-layers-and-doubles.md`](references/seams-layers-and-doubles.md): Functional Core / Imperative Shell, seam selection, layer fidelity, and test double placement.
- [`references/oracles-fixtures-and-boundaries.md`](references/oracles-fixtures-and-boundaries.md): discriminating positive/negative oracles, competing fixtures, equivalence partitions, and boundary analysis.
- [`references/isolation-determinism-and-async.md`](references/isolation-determinism-and-async.md): state isolation, deterministic clocks/seeds, race control, and async completion.
- [`references/review-rubric-and-metrics.md`](references/review-rubric-and-metrics.md): `strong` / `adequate` / `blocking` audit rubric and honest interpretation of coverage and mutation metrics.

## Usage

```text
$test-quality Audit the test suite for false positives, test theatre, and missing boundary coverage, then replace weak tests with contract-driven tests.
```

Licensed under [MIT](LICENSE).
