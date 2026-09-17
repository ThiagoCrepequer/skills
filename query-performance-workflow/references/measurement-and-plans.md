# Measurement and execution plans

## Identify what was measured

Keep query fingerprints associated with the report date and source. A rank is not a stable identifier. Distinguish Query Insights CPU/load from `pg_stat_statements` execution time, calls, and database-wide totals. Statement statistics may span resets, normalized parameter values, and several callers.

Capture server version, database role/environment, schema/index definitions (including predicates and column order), constraints and nullability, table cardinalities, statistics dates, and relevant planner settings. Do not infer bloat from dead-tuple estimates alone or run maintenance merely because statistics look old.

For application latency, distinguish database execution, planning, network, ORM hydration, DTO mapping, additional queries, and serialization. Database time does not measure the entire request.

## Safe executable diagnostic packets

Give the user a short ordered packet, not scattered fragments:

1. A fresh-session read-only transaction with scoped statement/lock/idle timeouts appropriate to the workload. For PostgreSQL a reasonable starting diagnostic budget is 30 seconds, 2 seconds, and 60 seconds respectively; adapt explicitly, not automatically after timeout.
2. Complete `PREPARE` statements with correct parameter types, or complete SQL with correctly typed literals.
3. `EXPLAIN (VERBOSE, SETTINGS)` first when the workload is unknown or risky.
4. If acceptable, `EXPLAIN (ANALYZE, BUFFERS, VERBOSE, SETTINGS)` on each variant, repeated with identical parameters.
5. Equivalence checks in one short consistent snapshot where feasible.
6. `ROLLBACK`, then `DEALLOCATE` named prepared statements when applicable.

Before delivering runnable SQL, resolve all arguments: tenant, authorized locations, dates, enum values, search patterns, active/removed flags, offset, page size. Never ship `...`, `TENANT_REAL`, or fake UUIDs as ready-to-run commands. If inputs are missing, supply a scoped discovery query or ask for the missing authorization context. Database membership does not prove that a user is authorized for every unit.

Explain session boundaries: prepared statements require the same connection; after an error the transaction needs rollback; a timeout is a stop signal. `SET LOCAL` ends with the transaction. Committing a genuinely read-only diagnostic transaction does not persist these local settings or mutate ordinary table data, so there is no data change to undo.

`EXPLAIN ANALYZE` executes the statement. A `SELECT` can call side-effecting functions or external services; read-only transactions are not a universal sandbox. Inspect functions/procedures before executing them. Do not assume rollback undoes external effects or sequence advancement. Keep long snapshots short to avoid operational impact. Production DDL and maintenance need separate authorization and an operational plan.

## Representative scenarios

Use real but privately handled bind values. Broad operator-name search, selective name/identifier search, zero results, deep pages, and different authorization sets stress different branches. Compare active and inactive sets when relevant. Include patients without units and forbidden-only units; an OR may short-circuit and leave `NOT EXISTS` unexecuted.

Large eligible counts are candidate stress cases, not proof of worst latency. Start with safe estimated plans. Do not launch expensive exhaustive discovery across production merely to find an absolute worst case.

## Interpret work, not node labels

- Planner cost is not milliseconds. Estimates and actuals answer different questions.
- Node times are inclusive and generally averaged per loop; avoid summing parent/child time or buffers. Use loops to locate repeated correlated work and use the top-level execution time for elapsed comparison.
- Compare rows entering/leaving filters, estimate errors, loops, buffers, reads, I/O timing, heap fetches, sort spills, parallel workers, and limit early-exit behavior.
- `never executed` means no runtime evidence for that branch. Zero returned rows may still require a full scan.
- Sequential scans can be optimal for small tables or large matching fractions. Bitmap OR/AND can be effective. An index-only scan may still fetch heap pages for visibility checks.
- Estimates off by an order of magnitude warrant investigation; they do not by themselves authorize `ANALYZE` or prove it will fix the query.

## Fair comparison

Use the same data snapshot for equivalence where practical. Repeat each variant at least twice and alternate order for meaningful comparisons. Separate first-read and warmed measurements; record workload/cache differences. Do not flush production caches to manufacture a cold test.

Report sample count and ranges; do not infer percentiles from two samples. A 490 ms run containing 428 ms of reads cannot be compared fairly with an earlier all-hit 49 ms run as evidence of a rewrite regression. Also compare buffer work and identical branches.

`force_custom_plan` can isolate parameter-sensitive behavior, but it is not proof of normal prepared-statement performance. Check the application's preparation behavior and automatic/custom/generic plans when parameter skew matters. Partial-index applicability can differ for generic parameterized predicates. Local PostgreSQL features must not be assumed available in an older production major version.

Large result printing and EXPLAIN instrumentation add overhead. Use aggregate equivalence output to avoid exposing data. `TIMING OFF` can reduce node instrumentation overhead when row counts/buffers matter more than per-node timing; label that measurement separately.
