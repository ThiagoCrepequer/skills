# Code design, patterns by problem pressure, and anti-patterns

High-quality code is not achieved by memorizing a fixed catalog of design patterns or forcing every feature into the same template. It is achieved by minimizing cognitive load, isolating complexity behind deep interfaces, separating pure decisions from side effects, and choosing patterns strictly in response to real structural pressure.

---

## 1. Deep modules vs. shallow abstractions

A module (a function, class, hook, service, or package) has an **interface** (everything a caller must know to use it safely) and an **implementation** (the mechanics hidden inside).

- **Deep module (desirable):** Small, simple interface + rich internal implementation that absorbs complexity and enforces invariants automatically.
  - *Example:* A tenant-scoped database boundary (`const tenant = useTenantDb()`) whose `findFirst`, `insert`, `update`, `delete`, and `paginate` methods automatically prepend `organization_id` filters, override spoofed `organizationId` fields on insert, strip `id` and `organizationId` on update, and wrap writes in a transaction with row-level security config. Callers cannot forget tenant isolation even if they try.
- **Shallow module (avoid):** An interface nearly as complex as its implementation—pass-through wrappers, single-implementation interfaces, or helpers that require callers to pass five context parameters (`db`, `orgId`, `userId`, `logger`, `tx`) and remember temporal call ordering (`init()` before `execute()`).

**The Deletion Test:** Imagine deleting the module and inlining its body into its callers. If the overall system becomes simpler, the module was shallow indirection. If deleting it forces $N$ callers to duplicate complex guards, parsing, or state management, the module is deep and earning its keep.

---

## 2. Functional Core, Imperative Shell (Testability by Construction)

Code becomes hard to test and brittle to change when **deciding what to do** is interleaved with **performing I/O**:

- **Functional Core:** Pure functions operating on plain data structures with zero I/O, zero database/network calls, and zero framework globals.
  - Status derivation from independent timestamps (`deriveAppointmentStatus(timestamps)`).
  - Schedule/recurrence expansion, split truncation (`UNTIL=`), and conflict interval math.
  - Visual layout and geometry calculations (column overlap clustering, break-time coordinates).
  - Permission resolution and set diffing (`toInsert`, `toDelete` link calculations).
- **Imperative Shell:** Thin orchestrators (HTTP route handlers, persistence services, UI components) that:
  1. Read external state (DB rows, HTTP input, DOM event).
  2. Call the **Functional Core** to compute the decision, state transition, or mutation plan.
  3. Apply the side effects atomically (e.g., inside one database transaction).

**Design diagnostic:** Whenever testing a business rule seems to require mocking five repositories or mounting a 900-line UI component, do not write five mocks—extract the pure decision into the Functional Core and test it directly with plain data.

---

## 3. Parse, don't validate & making illegal states unrepresentable

Do not validate untrusted data at the edge only to pass raw strings, untyped maps, or ambiguous flags into core logic:

- **Parse at the seam:** Transform raw inputs once into canonical domain types (e.g., normalizing legacy permission aliases and wildcards into a deduplicated `PermissionKey[]` at the boundary, or using distinct input schemas for creation vs. partial `PATCH` updates so default values never overwrite omitted fields).
- **Do not encode multiple concepts into one string:** Avoid composite string IDs such as `"${masterId}_${isoDate}"` that force every downstream service and UI route to run `lastIndexOf("_")` and regex checks to guess whether an ID is a persisted record or a virtual occurrence. Represent distinct domain concepts with explicit fields or discriminated unions.
- **Model independent facts independently:** Instead of a single mutable `status` column that destroys history when updated (e.g., overwriting "confirmed" when the client arrives), record independent domain facts (`confirmedAt`, `arrivedAt`, `startedAt`, `finishedAt`, `cancelledAt`) and derive the visual status purely.
- **Use structured error codes, never message substrings:** Drive error recovery and UI branching from stable machine-readable codes (`error.code === "conflict"`, HTTP `409`), never `error.message.toLowerCase().includes("conflito")`.

---

## 4. Choosing design patterns by problem pressure (not dogma)

Never introduce a pattern "just in case." Apply a pattern only when the codebase exhibits its specific structural pain:

| Structural pain / symptom | Pattern to apply | When NOT to apply it |
| :--- | :--- | :--- |
| Complex branching, math, or string manipulation buried inside a 150-line DB service or UI component | **Pure Policy / Functional Core Extraction** | Trivial 2-line assignments with no branching |
| Need to swap an external boundary (payment, email, clock, DB) between production and deterministic tests | **Port & Adapter (Seam)** | Only 1 implementation exists and no test/runtime variation is needed |
| Every function signature threads the same ambient parameters (`db`, `orgId`, `traceId`) down 4 layers (*Context Plumbing*) | **Scoped Context (`AsyncLocalStorage` / Request Scope)** | Explicit data parameters that genuinely vary per call |
| 2+ screens or services implement the exact same state machine (pagination, tri-state sort cycling, debounce, page-1 reset) | **State / Query Factory (`useDataTableState`, `tenant.paginate`)** | Two places have 3 similar lines of formatting that follow *different* business policies |
| Multi-step writes (`update` old series + `delete` exceptions + `insert` new series + sync tags) risk partial corruption on failure | **Unit of Work / Explicit Transaction Scope (`tenant.transaction`)** | Single atomic statement or read-only query |
| Fetching related data inside `.map()` / `for` loops causes $N+1$ database or network round-trips | **Batch Loader / SQL `IN (...)` or `JOIN` Projection** | Single-record lookup where batching adds complexity without reducing round-trips |
| Remote operation may time out after succeeding on the provider | **Idempotency Key + Status Reconciliation** | Local in-memory or naturally idempotent read operation |

---

## 5. High-cost implementation anti-patterns to eliminate

During implementation or refactoring, actively avoid and remove these six anti-patterns:

### A. Compensating fallback cascades (fabricating success)
Catching an API or database failure and returning fictional domain entities (e.g., hardcoded sample clients/doctors when a fetch fails, or fake fallback IDs like `"default_noshow"`) so the UI appears to work. This corrupts user trust and forces backend services to add compensating string-matching hacks when those fake IDs are submitted back.
- *Fix:* Confine fallbacks strictly to optional presentation defaults; surface real failures honestly through structured error states.

### B. Deforming production code for weak test doubles
Adding `if (!db) return fakeSuccess` branches in production services, or replacing SQL `WHERE` / `JOIN` / `ORDER BY` queries with unbounded `findMany()` full-table loads and JavaScript `for` loops just because a hand-rolled test mock cannot evaluate SQL predicates.
- *Fix:* Keep production queries pushed down to the database engine, and test them against a faithful embedded/test database.

### C. Unbounded queries and $N+1$ loops
Loading every row of a tenant table into memory to filter by date range or join relations in JavaScript, or calling `await tenant.findFirst(...)` inside `rows.map(...)`.
- *Fix:* Push date/status predicates and joins into SQL, and batch relation lookups with `inArray(table.foreignId, rowIds)`.

### D. Partial updates that wipe sibling fields
Reusing a creation schema with `.default(...)` values for `PATCH` requests (causing omitted fields to be overwritten with defaults), or failing to preserve existing fields (`calendarColor`, `workingHours`, `userId`) when toggling a single flag like `isActive`.
- *Fix:* Define dedicated partial update schemas where omitted keys remain `undefined`, and verify sibling field preservation in tests.

### E. Non-transactional multi-step writes
Executing sequential `insert`, `update`, and `delete` statements across related tables (or running tag/link synchronization after the transaction block closes) so a mid-flight error leaves orphaned or inconsistent state.
- *Fix:* Wrap the entire logical mutation inside a single transaction boundary.

### F. Shotgun surgery (duplicated invariants)
Copy-pasting security or normalization helpers (`escapeSqlLike`, role permission parsing, tag name normalization, default seed catalogs) across multiple modules where a bug fix in one copy leaves the others vulnerable.
- *Fix:* Consolidate shared invariants into a single authoritative module at the appropriate package layer.
