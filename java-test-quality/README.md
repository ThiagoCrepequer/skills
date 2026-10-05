# Java Test Quality

[Install from the skills repository](https://github.com/ThiagoCrepequer/skills)

```bash
npx skills add ThiagoCrepequer/skills --skill java-test-quality
```

An agent skill for designing, writing, strengthening, and reviewing trustworthy tests in Java and Quarkus projects.

## What it does

It teaches coding agents to turn business rules and technical contracts into executable evidence. It covers JUnit, CDI components, HTTP contracts, real persistence, security, concurrency, state isolation, JaCoCo, and PIT mutation testing.

It rejects tests that pass without proving behavior: presence-only assertions, mocks that remove the risk under test, fixtures without competing data, arbitrary sleeps, leaked state, and coverage treated as a synonym for quality.

## Philosophy

A good test should be:

- **Bug-sensitive:** a plausible regression makes it fail.
- **Refactor-tolerant:** internal changes that preserve the contract keep it passing.
- **Faithful to the risk:** database, HTTP, CDI, or security participates when it owns the behavior.
- **Isolated and deterministic:** no scenario depends on order, shared state, sleeps, or wall-clock time.
- **Honest:** its validation scope and remaining uncertainty are reported precisely.

Mutation score and coverage are diagnostic signals. Confidence comes from a clear contract, discriminating fixtures, and an independent oracle capable of rejecting incorrect outcomes.

## Contents

The entrypoint is [`SKILL.md`](SKILL.md). Its references cover good-test principles, Quarkus test layers, assertions, fixtures, test doubles, concurrency risks, and mutation analysis.

## Usage

```text
$java-test-quality Write regression tests for this Quarkus service and prove tenant isolation.
```

Licensed under [MIT](LICENSE).
