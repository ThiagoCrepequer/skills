# React Test Quality

[Install from the skills repository](https://github.com/ThiagoCrepequer/skills)

```bash
npx skills add ThiagoCrepequer/skills --skill react-test-quality
```

An agent skill for designing, writing, strengthening, and reviewing trustworthy tests in React and JavaScript/TypeScript projects.

## What it does

It teaches coding agents to turn user flows and public contracts into executable evidence. It covers Testing Library, Vitest/Jest, realistic interactions, accessibility, network behavior, forms, stores, caches, routing, asynchronous races, browser tests, and StrykerJS.

It rejects tests that pass without proving the experience: presence-only assertions, mocked-away hooks or clients, broad snapshots, shared state, arbitrary waits, and success states that do not validate the request and response.

## Philosophy

A good test should be:

- **Bug-sensitive:** a plausible regression makes it fail.
- **Refactor-tolerant:** components, hooks, and state can change internally without breaking the contract.
- **User-centered:** it interacts and observes through the public, accessible surface.
- **Isolated and deterministic:** every test receives fresh stores, caches, handlers, timers, and data.
- **Honest:** it distinguishes component, network, and browser evidence and states what remains unproved.

Mutation score and coverage are diagnostic signals. Confidence comes from meaningful states, faithful interactions, and an independent oracle capable of detecting incorrect outcomes, forbidden effects, and race conditions.

## Contents

The entrypoint is [`SKILL.md`](SKILL.md). Its references cover good-test principles, frontend test layers, semantic assertions, fixtures, test doubles, asynchronous isolation, and mutation analysis.

## Usage

```text
$react-test-quality Test this form and prove its payload, visible outcome, and duplicate-submission prevention.
```

Licensed under [MIT](LICENSE).
