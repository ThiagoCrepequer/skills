# Async behavior and state isolation

## Await the real completion condition

Await `userEvent` calls. Use `findBy*` for eventual appearance and `waitForElementToBeRemoved` for disappearance. Use `waitFor` only around assertions that should eventually become true; never issue clicks, renders, requests, or other side effects inside its retried callback.

Do not finish a test while promises, subscriptions, callbacks, or assertions remain pending. Treat `act` warnings and unhandled rejections as defects in the test or product, not console noise to suppress.

## Timers

Use real timers unless the behavior is explicitly time-based. For debounce, throttle, timeout, retry, or polling, use the runner's fake timers and integrate them with `userEvent`'s timer advancement when supported. Restore real timers reliably.

Never use arbitrary `setTimeout`/sleep delays to wait for UI. Advance to the meaningful boundary and assert immediately before/at/after when the contract depends on time.

## Deferred responses and races

Control response order explicitly:

1. start request A and hold its response;
2. trigger request B and resolve it;
3. assert B's result;
4. resolve A;
5. assert A cannot overwrite newer state if that is the promise.

Use deferred promises or per-request network handlers. A random delay does not prove the disputed ordering.

Cover cancellation/unmount, duplicate submission, optimistic update rollback, and retry only when owned by the changed behavior.

## Fresh state

Create per test:

- store and reducers/middleware state;
- query/cache client with retries disabled or explicitly controlled as the harness requires;
- router/history and initial route;
- network handlers/request capture;
- authenticated session/tenant;
- mutable data and `userEvent` instance.

Reset handlers after each test and fail on unhandled requests when supported. Clear storage, cookies, observers, globals, environment stubs, and document-level listeners. Auto DOM cleanup does not reset non-DOM state.

## Cache and tenant/user isolation

Use different otherwise-valid users/tenants and distinct responses. Assert query keys/cache entries cannot leak results. A single-user fixture cannot prove scoping.

## Router and storage

Use a real in-memory/test router where possible and assert public location/parameters. Do not mock navigation functions when the routing result is the behavior. For local/session storage, start clean and assert exact key/value/removal semantics; use a browser layer when cross-context behavior matters.

## Streaming and subscriptions

Test protocol-level behavior, not only final hook state: headers/framing/parser input, named events, malformed data, terminal marker, EOF, error, cancellation/unsubscribe, and no updates after unmount when relevant. JSDOM may not represent real streaming/browser cancellation; use the existing browser or lower-level protocol layer where necessary.

## Browser isolation

Browser tests should use a fresh browser context/page state per test and uniquely provision server records. Do not rely on cleanup alone for cookies, visited state, or storage. Avoid serial/order dependence unless the user flow itself is explicitly sequential and isolated within one test.
