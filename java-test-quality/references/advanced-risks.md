# Advanced Java and Quarkus test risks

## Time

Inject or control `Clock`; use explicit `ZoneId`. Cover inclusivity/exclusivity, date rollover, DST, expiration at the exact instant, and precision/truncation only when domain-relevant. Do not make assertions against wall-clock `now()` with tolerance as a substitute for control.

## Transactions

Test commit and rollback at the boundary where the transaction actually runs. Verify no partial state after exceptions. For nested/new transactions, HTTP requests, async observers, or outbox publication, do not assume the test method's rollback contains all work.

## Concurrency

Make the disputed schedule deterministic:

1. block operation A at a known point with a latch/barrier/controllable future;
2. start operation B or close/cancel;
3. release A deliberately;
4. assert final state, version/cardinality, terminal delivery, and absence of loss/duplication;
5. repeat the focused schedule when scheduler variation remains meaningful.

Never coordinate with `Thread.sleep`. A long sleep makes timing likely, not causal.

## Idempotency and retries

Assert first execution, exact replay result, no duplicate side effects, different-payload conflict, lease/timeout behavior when applicable, and retry boundaries. Distinguish attempts from successful effects. A mock invocation count alone may miss duplicate persistence or emission.

## Async/reactive code and streams

Await terminal completion and assert all signals relevant to the contract: items, order if promised, error/cancellation, backpressure behavior, and terminal marker. Do not let a test finish before callbacks/assertions run. Use latches or reactive test subscribers rather than sleeps.

For SSE or streaming protocols, test content type, framing, event/data semantics, malformed input, required terminal markers, EOF, cancellation, and no dropped accepted terminal event when those behaviors are in scope.

## External failures

Cover representative transport timeout, connection failure, non-success response, malformed response, retryable/non-retryable mapping, and circuit/fallback behavior only when configured by the contract. Assert no forbidden duplicate effect and correct local state.

## Caches and singletons

Use unique keys and explicit invalidation. Restore global caches/singletons/static state. Prove tenant/user scoping with competing keys and ensure tests pass independently of order.

## Database-specific behavior

When testing PostgreSQL-specific behavior, use PostgreSQL. Include null semantics, case/whitespace, JSON/array operators, timezone, uniqueness, foreign keys, optimistic/pessimistic locking, and pagination/order only as relevant. Correct local query results do not prove production performance; planner evidence is a separate concern.

## Security

Prove authentication and authorization independently. Include cross-tenant/cross-owner attempts even with an otherwise authorized role. Assert response redaction and absence of persistence/outbound effects on denial.
