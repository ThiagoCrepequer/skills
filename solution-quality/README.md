# Solution Quality

[Install from the skills repository](https://github.com/ThiagoCrepequer/skills)

```bash
npx skills add ThiagoCrepequer/skills --skill solution-quality
```

A language- and framework-agnostic agent skill for writing high-leverage, testable-by-construction software, choosing design patterns by problem pressure, and solving root causes with proportionate evidence.

## What it does

It answers **"What is high-quality code?"**, **"How do I write code that is naturally testable?"**, and **"How do I choose the right design pattern without over-engineering or falling into common anti-patterns?"**

It complements [`test-quality`](../test-quality/SKILL.md): while `test-quality` ensures tests are bug-sensitive and refactor-tolerant, `solution-quality` ensures the production architecture has clean seams, deep modules, and pure decision cores that make trustworthy testing effortless.

## Core principles

1. **Deep Modules over Shallow Wrappers:** Hide complex mechanics and compulsory invariants (tenant isolation, pagination, auth, transactions) behind small, safe interfaces that eliminate context plumbing and make misuse impossible by construction.
2. **Functional Core, Imperative Shell:** Separate pure business decisions, calculations, and state derivations from I/O, persistence, and UI orchestration.
3. **Parse, Don't Validate (Make Illegal States Unrepresentable):** Transform untrusted inputs into canonical types at the boundary instead of scattering string splitting, `JSON.parse` fallbacks, and defensive checks across callers.
4. **Atomicity & Truthful Failure:** Execute multi-step mutations inside a single transaction boundary and fail explicitly with structured errors—never fabricate fake success data or deform production queries to appease weak test mocks.
5. **Patterns by Problem Pressure:** Apply design patterns only when a concrete structural pain demands them, preserving versatility for each scenario.

## Contents

- [`SKILL.md`](SKILL.md): Core implementation protocol and structural quality rules.
- [`references/code-design-and-patterns.md`](references/code-design-and-patterns.md): Practical guide to deep modules, Functional Core / Imperative Shell, pressure-driven design patterns, and high-cost implementation anti-patterns.
- [`references/decision-examples.md`](references/decision-examples.md): Concrete engineering scenarios contrasting tempting shortcuts with legitimate solutions, required evidence, and boundaries.

## Usage

```text
$solution-quality Refactor this module to separate domain decisions from persistence, eliminate N+1 queries, and preserve transaction integrity.
```

Licensed under [MIT](LICENSE).
