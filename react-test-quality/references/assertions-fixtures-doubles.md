# React assertions, fixtures, and test doubles

## Semantic assertions

Prefer observations meaningful to users or public consumers:

- element is visible, enabled/disabled, checked, selected, expanded, focused;
- accessible name, description, role, error association, or status/live announcement;
- exact visible value/text where the wording is contractual;
- request method, URL, normalized body, relevant headers, and cardinality;
- navigation destination and parameters;
- storage/cache effect scoped to the correct user/tenant/key;
- absence of contradictory UI or forbidden request/effect.

Presence alone is insufficient when visibility, state, value, or interaction matters. `toBeInTheDocument` can support an oracle but rarely proves a workflow by itself.

Use `getBy*` for expected immediate presence, `findBy*` for eventual presence, and `queryBy*` for absence. Avoid broad text regexes that also match an error or hidden duplicate.

## Negative assertions

Absence should prove a promised prohibition, not merely test implementation timing. Examples:

- success is absent after server failure;
- forbidden action is absent/disabled and no request occurred;
- stale data does not replace newer results;
- duplicate clicks do not produce duplicate requests;
- validation error prevents navigation/storage mutation.

Pair negative UI assertions with request/effect assertions when the forbidden effect matters.

## Fixtures and render helpers

Good helpers create the smallest production-representative environment and return useful public controls such as `user`, router history, or request capture. They must create fresh store/query client/router/data on every call.

Avoid:

- one global store or query client;
- module-level mutable fixtures;
- giant render helpers with invisible default permissions/routes/data;
- factories that ignore overrides;
- default handlers that accidentally satisfy every scenario;
- setup that preconfigures the exact state the action is meant to produce.

Make business distinctions visible: authorized versus unauthorized user, matching versus competing item, first versus stale response, empty success versus error.

## Mocks and spies

Restore mocks, spies, globals, environment, and timers after each test. Prefer network interception over mocking fetch/Axios/query hooks. Mock a child only when its implementation is deliberately out of scope and preserve the public interaction contract; excessive child mocks can turn a screen test into a wiring test.

Warning signs:

- mocked hook returns exactly what the test later asserts;
- component under test or essential child is replaced;
- every provider is mocked;
- spy call is asserted without user-visible/request outcome;
- `vi.mock`/`jest.mock` module state leaks across cases;
- `mockImplementation` recreates production branching.

## Snapshots

Snapshots are secondary evidence for small stable structures, serialized protocols, or carefully reviewed accessibility trees. They must not be the sole oracle for business behavior. Avoid huge DOM snapshots, volatile IDs/timestamps, and blind update workflows.

## Tables and parameterization

Use `it.each`/`test.each` when cases express the same rule and each failure is labeled. Do not compress different UI states/workflows into a table that hides setup and intent.

## Expected values

Derive expectations from the requirement, hand-worked example, API contract fixture, or invariant. Do not import production constants/functions merely to calculate the expected output when those definitions are what the test should protect. Shared protocol constants are appropriate only when the shared contract itself is authoritative.
