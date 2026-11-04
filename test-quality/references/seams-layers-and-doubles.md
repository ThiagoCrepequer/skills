# Seams, test layers, and test doubles

## Functional Core, Imperative Shell

The highest-leverage architectural pattern for testability is separating **pure domain decisions (Functional Core)** from **I/O, framework wiring, and side effects (Imperative Shell)**:

- **Functional Core:** calculations, state-machine transitions, status precedence rules, recurrence expansion math, calendar overlap/geometry layout, permission evaluation, and payload validation/normalization.
  - Takes plain data structures in and returns plain data structures (or decisions/commands) out.
  - Tested via fast, deterministic, zero-mock unit tests covering exhaustive equivalence partitions and boundary conditions in milliseconds.
- **Imperative Shell:** HTTP controllers, database queries/transactions, auth middlewares, message brokers, and UI components.
  - Thin orchestrators that fetch state, call the Functional Core, and apply side effects.
  - Tested via fewer, high-fidelity integration or component tests with real or boundary-faithful infrastructure.

When a test is painful to set up because it requires 15 mocks to reach an `if` statement, treat that pain as a design signal: extract the decision logic into a pure function and test it directly.

## What is a seam?

A **seam** is a stable public boundary where you can stimulate behavior and observe outcomes without coupling to internal implementation details:

- Public function or module export (`deriveAppointmentStatus(timestamps)`, `computeTimeGridDayLayout(events)`)
- Domain service interface with explicit context boundary (`runWithTenant({ db, orgId }, () => service.create(...))`)
- HTTP application boundary (`app.request("/api/resource", { method: "POST", body })`)
- Rendered UI component boundary queried via accessible roles and labels (`getByRole("button", { name: /save/i })`)

Never test private methods, internal component state hooks, unexported helpers, or database tables bypassed around the service contract when the public seam can observe the behavior.

## Choosing the lowest faithful layer

Always choose the **lowest test layer that includes the subsystem capable of creating the defect**:

| Risk under test | Faithful layer | Unfaithful layer (avoid) |
| :--- | :--- | :--- |
| Pure calculation, state precedence, schedule/layout math, schema validation | Plain unit test on pure function/schema | Full browser or HTTP test for every math edge case |
| SQL joins, `WHERE` predicates, `ORDER BY`, `GROUP BY` metrics, DB constraints, RLS policies | Real database engine (embedded/in-memory DB or container) | Mocked ORM chains (`mockSelect().mockWhere()`) or fake DB that re-implements metrics in JS |
| HTTP status codes, auth/tenant middleware, payload validation, error serialization | In-process HTTP request through router (`app.request` / `MockMvc` / `TestClient`) | Calling controller method directly bypassing middleware and validation |
| Outbound HTTP client / webhook / email adapter | Real adapter + network-level interception or local stub server | Mocking the adapter method itself |
| Interactive form validation, loading/skeleton persistence, optimistic UI, accessible controls | Component test with real providers + network interception + semantic DOM queries | Checking `typeof Component === "function"` or mocking child form/hooks |
| CSS layout geometry, real browser focus/navigation, service workers, cross-tab storage | Real headless browser (Playwright / browser mode) | JSDOM / happy-dom (which do not compute real layout) |

Add a higher-layer test only when wiring between layers is itself a distinct risk.

## Test doubles: types and placement

Use precise vocabulary for test doubles:

- **Dummy:** inert value passed only to satisfy a signature; never used.
- **Stub:** provides canned responses to queries from the system under test.
- **Fake:** working, lightweight implementation of a boundary with real stateful semantics (e.g., an in-memory repository or embedded database that genuinely stores, filters, and isolates records).
- **Spy / Mock:** records outbound interactions when the side effect itself (e.g., sending an email, publishing an event, charging a card) is the external contract.

### Rules for test doubles

1. **Mock only at external system boundaries** you do not own or cannot run deterministically in-process (third-party payment gateways, email/SMS providers, external APIs, system clock, randomness).
2. **Never mock your own domain collaborators, mappers, validators, or query hooks** inside a test claiming to verify their integrated behavior.
3. **Prefer stateful Fakes over webs of Stubs:** A well-tested Fake (or embedded DB) verifies state transitions naturally (`create` followed by `get` returns the created item; `delete` followed by `get` returns `null`) without brittle call-sequence assertions.
4. **Audit every Fake for fidelity:** A Fake must enforce the same invariants (column matching, uniqueness, tenant scoping, sorting) that the real boundary enforces. Never let a Fake hardcode the expected answer for a specific test.
5. **Avoid `verifyNoMoreInteractions` on internal collaborators:** Assert absence of forbidden calls only at external side-effect boundaries (e.g., proving no payment charge or email was sent when validation fails).
