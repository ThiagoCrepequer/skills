# State isolation, determinism, and asynchronous correctness

A flaky test—one that passes or fails without any code change—destroys trust in the entire suite. Never mask flakiness with retries, test ordering, or longer sleeps.

## State isolation

Every test must be a self-contained world that passes when run alone, in a full suite, repeatedly, or in shuffled/parallel order:

1. **No shared mutable fixtures:** Never mutate module-level objects or arrays across tests. Factory functions must return fresh copies (`structuredClone` or fresh object literals) on every call.
2. **Fresh infrastructure per test:** Instantiate a fresh in-memory database state, cache/query client, state store, router, and user session per test.
3. **Strict cleanup:** Restore all mocks, spies, environment variables, global overrides, timers, and DOM nodes in `afterEach`.
4. **Scoped context propagation:** When production code relies on scoped context (`AsyncLocalStorage`, request-scoped DI, transaction-local session variables like PostgreSQL `set_config(..., true)`), test both:
   - execution inside the valid context;
   - fast failure when invoked outside the context;
   - non-leakage between concurrent or sequential contexts.

## Time, timezones, and randomness

Never let wall-clock time, host timezone, or unseeded randomness determine whether a test passes:

1. **Injected or frozen clocks:** Pass an explicit `now` parameter, a `Clock` interface (e.g., `steppingClock()`), or deterministic fake timers (`vi.useFakeTimers()` / `vi.setSystemTime(new Date("2026-11-04T12:00:00.000Z"))`).
2. **Never assert `now()` with loose tolerances:** Instead of `expect(Date.now() - record.createdAt).toBeLessThan(1000)`, freeze or inject the clock and assert the exact timestamp.
3. **Timezone discipline:** Test timestamps across local-to-UTC boundaries explicitly. Ensure date construction rules (e.g., local calendar date vs. UTC ISO string) are tested with timestamps near midnight (`23:00` local vs. `02:00` next day UTC) where `-03:00` or `+02:00` offsets expose off-by-one-day bugs.
4. **Deterministic IDs and seeds:** When testing sorting, pagination, or diffing, use deterministic IDs or seeded generators rather than asserting against unpredictable UUID ordering.

## Asynchronous correctness and race conditions

Asynchronous bugs (unhandled rejections, stale responses overwriting newer state, double submissions, premature completion, timeouts) require deterministic control over event ordering:

1. **Await every stimulus and terminal state:** Never fire an async action without awaiting its observable completion.
2. **Ban arbitrary sleeps (`setTimeout` / `Thread.sleep`):** Waiting `100ms` is either too slow (wasting CI time) or too short under CPU load (causing flakes). Instead:
   - await a deterministic promise, latch, barrier, or event;
   - use fake timers (`advanceTimersByTime`) when testing debounce, throttle, polling, or timeout durations;
   - use bounded semantic polling (`findBy*`, `waitFor`, `Awaitility`) strictly around an observable assertion—never place side effects or clicks inside a retrying `waitFor` callback.
3. **Controlled deferred promises for races and loading states:**
   To test in-flight loading state, duplicate-click prevention, or out-of-order network responses (Request A starts before Request B, but Request B resolves first and Request A resolves later):
   - Create controllable deferred promises (`Promise.withResolvers()` or a `createDeferred()` helper) at the boundary.
   - Assert the intermediate loading/disabled UI while the promise is pending.
   - Resolve responses in the exact adversarial order (`resolveB()`, then `resolveA()`) and assert that stale Request A does not overwrite newer Request B's state.
4. **Zero unhandled warnings:** Treat unhandled promise rejections, unawaited state updates after unmount, and framework `act(...)` warnings as test defects to be fixed at the root cause, never silenced with console suppression.
