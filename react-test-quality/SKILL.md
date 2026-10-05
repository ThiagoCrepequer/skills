---
name: react-test-quality
description: Teach, design, write, strengthen, or review trustworthy tests for React and JavaScript/TypeScript frontends. Use for regression tests, weak or false-positive tests, Testing Library strategy, user interactions, API/network behavior, async state, stores/caches/routers, browser tests, accessibility-facing behavior, Vitest/Jest coverage quality, StrykerJS mutation testing, flakiness, or test-quality audits. Do not use merely to run an unchanged suite.
license: MIT
---

# React and JavaScript Test Quality

Produce executable evidence that users and consumers receive the promised behavior and that plausible regressions would be detected. Optimize for confidence per maintenance cost, not test count, snapshots, or line coverage.

## Define a good frontend test

A good test is an executable example of a user-facing or public contract. It establishes meaningful state, performs one coherent user workflow or API stimulus, and checks observable consequences with an independently justified oracle.

It must be both:

- **bug-sensitive:** a realistic defect in the protected behavior makes it fail;
- **refactor-tolerant:** component extraction, hook rewrites, state-library changes, or other internal refactors preserving the contract keep it passing.

Quality does not come from Testing Library, Vitest/Jest, assertion count, snapshots, coverage, or mutation score. Tools support the causal story; they do not create it.

Before substantive authoring or review, read [references/good-test-principles.md](references/good-test-principles.md).

## Discover the real contract

Inspect the requirement, production diff, rendered flow, API contract, callers, neighboring tests, test setup, and project conventions before editing. Do not infer the promise solely from component internals. Do not add libraries, global configuration, thresholds, or CI without authorization.

For each material behavior identify:

- observer: user, assistive-technology user, API consumer, router, storage/cache consumer, or external analytics/system;
- initial visible/data state and coherent workflow;
- success, empty, validation, permission, and error outcomes that are actually contractual;
- prohibited results such as premature success, duplicate request, stale overwrite, forbidden action, or data leakage;
- boundaries and race orderings that can change the result;
- lowest test layer faithful to the risk.

Use a compact behavior matrix when several states interact. It is a reasoning aid, not a quota. Do not generate every generic loading/error/null case unless the component owns that behavior.

## Select the faithful layer

Choose the lowest layer that observes the real risk:

- pure transformation/schema/policy: plain unit test;
- component rendering, accessible interaction, form behavior, and network-driven UI: React Testing Library with the project's normal providers and network interception;
- reusable hook/library public contract: direct hook test only when the hook itself is the public unit; otherwise exercise it through a representative component;
- request construction/parsing: real client/adapter with network interception rather than mocking the adapter itself;
- routing, browser APIs, focus across pages, downloads, storage isolation, real layout, service workers, streaming, or critical journeys: existing browser/component-browser/E2E layer.

Read [references/react-test-layers.md](references/react-test-layers.md) whenever choosing between unit, component, network, and browser evidence.

## Interact and assert through the user surface

Use the repository's Testing Library conventions. Prefer `screen` and queries resembling how users find elements: role plus accessible name, label, visible text/display value, and test ID only as a justified last resort. Create `userEvent.setup()` inside the test and await interactions.

Assert the complete relevant outcome:

- visible text/state, accessibility semantics, focus, enabled/disabled/checked/selected state;
- exact transformed request payload, URL/method/relevant headers, and request count when submission is part of the contract;
- loading termination, success only after success, error visibility, and absence of contradictory UI;
- navigation target and parameters when promised;
- optimistic state and rollback;
- absence of forbidden request/callback/storage mutation;
- stale-response, duplicate-click, cancellation, and ordering behavior when applicable.

One test may contain several related assertions when they describe one coherent user outcome. Do not assert internal state, private hook calls, CSS classes, or child-component choreography when a public observation exists.

Read [references/assertions-fixtures-doubles.md](references/assertions-fixtures-doubles.md) for semantic queries, requests, fixtures, providers, mocks, snapshots, and negative assertions.

## Make plausible regressions observable

For each changed decision consider counterfactuals: condition inverted, validation boundary moved, request field omitted, wrong normalized value, success shown before response, double submit, denied action still sent, empty/error conflated, older response overwrites newer state, cleanup loses a terminal update, or tenant/user cache key omitted. The test must fail for relevant counterfactuals even without mutation tooling.

For a regression, establish red/green evidence when practical: the new test fails in the known-bad state and passes with the fix. Do not leave temporary source mutations.

When StrykerJS already exists or mutation configuration is requested, target changed logic and triage survivors. Do not chase 100% blindly or mutate JSX noise merely to raise a denominator. Read [references/mutation-and-metrics.md](references/mutation-and-metrics.md).

When configuring or upgrading tools, consult [references/sources.md](references/sources.md) and the project's pinned versions rather than guessing current options.

## Enforce async correctness and fresh state

Await every relevant interaction and eventual state. Use `findBy*` for appearance, `queryBy*` for absence, and bounded `waitFor` only around an assertion expected to become true. Never place side effects inside a retrying `waitFor` callback. Do not silence `act` warnings or unhandled promise rejections.

Create a fresh store, query/cache client, router, session, mutable fixture, and `userEvent` instance per test unless immutability is proven. Reset network handlers and restore mocks, globals, environment, observers, storage, and timers. Replace sleeps with observable completion, deferred responses, or correctly integrated fake timers.

Read [references/async-state-isolation.md](references/async-state-isolation.md) for timers, requests, races, caches, router/storage, streaming, and browser isolation.

## Verify proportionally

Use cheap evidence first:

1. Run the changed test and confirm the intended assertions execute.
2. Run the nearest affected suite/module.
3. Inspect changed-code branches when coverage is configured.
4. Run targeted StrykerJS when configured/requested; inspect survived, no-coverage, timeout, and error mutants.
5. Run browser/component-browser evidence when JSDOM cannot represent the risk.
6. Repeat or vary order when state leakage/races are plausible.

Never report `No test files found`, an interrupted run, or stale earlier output as passing. Separate unrelated baseline failures. Do not claim a full suite, real browser, live backend, visual layout, or mutation gate unless it actually completed at that scope.

## Review and report

Use [references/review-rubric.md](references/review-rubric.md) before delivery or when auditing existing tests. Block acceptance on a false-positive oracle, mocked-away behavior, missing material state, leaked global/cache/network state, unawaited work, arbitrary sleeps, or an unexplained relevant survivor.

Report:

- user/public contracts, transitions, and prohibited outcomes proved;
- chosen layers and why they are faithful;
- exact commands, test counts, and results;
- coverage/mutation scope and categories when run;
- accepted survivor/exclusion with concrete justification;
- what was not run and what the tests do not prove.
