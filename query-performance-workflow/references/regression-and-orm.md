# Behavior, persistence, and generated SQL

## Build tests that could reject the wrong rewrite

Inventory existing coverage first. Reuse tests only if they exercise the relevant behavior and would fail for the plausible regression. Add focused fixtures before changing the implementation; use a real compatible database and real ORM mappings for SQL/filters. External systems may remain mocked.

Choose applicable rows from this matrix rather than mechanically generating every combination:

| Risk | Competing fixture / assertion |
| --- | --- |
| Tenant isolation | Equivalent-looking rows in another tenant must remain absent. |
| Unit authorization | Allowed-only, forbidden-only, allowed plus forbidden, several allowed, no unit; assert IDs and no duplicates. |
| Empty permissions | Verify the actual existing contract, including no-unit visibility; do not invent a policy. |
| Soft removal / active | Matching removed/inactive alternatives must not leak where excluded. |
| Latest per key | Older eligible, newer ineligible, null key/date, tied date; assert documented winner rules. |
| Recurrence | No recurrence, bounded/unbounded rule, excluded rule, event dates outside range with valid recurrence. |
| Dates | Exactly at both boundaries, just outside, repository time zone and timestamp precision. |
| Aggregation | Duplicate join rows, no history, zero denominator, negative delta, decimal rounding, multiple periods. |
| Pagination | More than one page; no overlaps/omissions under stable ordering; metric order before pagination. |
| API / lazy loading | Detail fields and summary fields, export/job consumers, mapping and serialization after service return. |

Expected values should be independent of the rewritten algorithm. Do not make the oracle execute the same new predicate or only assert a count. Use explicit expected IDs, values, ordering, and forbidden rows. Do not assert millisecond timing in ordinary regression tests.

Flush and clear persistence state where needed to avoid first-level-cache false positives. Ensure fixture setup transactions and tenant/security contexts match test-harness semantics. An HTTP request may not share the test method's transaction. A rollback annotation alone is not proof of test-data cleanup across HTTP boundaries.

## Equivalence before speed claims

Compare both directions, not just counts. For result sets with possible duplicates, use `EXCEPT ALL` in both directions; ordinary `EXCEPT` ignores multiplicity. Compare the full observable projection or separately validate dependent fields, not just IDs if values changed.

Perform SQL comparisons in the same short repeatable-read snapshot when feasible, returning only counts of mismatches. Example report fields: baseline rows, candidate rows, baseline-only rows, candidate-only rows. Row counts alone are insufficient.

Set equivalence does not validate ordering or pagination. Check ordered pages separately. Existing ties can make page membership nondeterministic; investigate rather than adding an unapproved tiebreaker. Floating-point/decimal tolerances must come from the business contract, not hide discrepancies.

Large baseline queries may time out. Do not repeatedly run them or increase the timeout automatically. Use estimated plans, bounded representative fixtures, or smaller valid scenarios and clearly retain the limitation: the full baseline was not measured. Never compare semantically different tenant/date scopes as if equivalent.

## Lazy-loading regressions

Exercise the actual service/HTTP boundary instead of keeping a test transaction open across every read. Mapping in a test-only open session can conceal production `LazyInitializationException`.

Assert fields likely to disappear in a lightweight projection: email, active status, roles, authorship, or other established detail fields—not merely ID/name. Keep summary and detail contracts separate. Test list/count/detail/create/update/export paths actually affected by the association change.

Bounded query-count checks can protect a measured N+1 fix, but capture statements for the request under test, excluding fixture setup. A mapping annotation assertion does not prove the response is complete or the number of queries acceptable.

## Verify the ORM, not a reconstruction

Capture executed statements through a supported SQL inspector/logger or test instrumentation, scoped to the relevant request. Protect bind values. Save the SQL shape, bind types, parameter mapping, ORM/database versions, and relevant filter activation.

Check these common differences:

- Required owner-tenant and permission predicates are present.
- Casts target parameters, not indexed columns, and SQL aliases are valid.
- A recurrence prefilter references the root foreign-key column when intended.
- Removed optional joins stay absent only in applicable filter scenarios.
- Formula expressions or eager joins really disappear from unrelated loads.
- Limit/offset, sorting, null precedence, and count/list filters remain consistent.

Generated aliases and formatting are not behavior. Prefer structural assertions and successful real execution over brittle exact-text snapshots. Then run EXPLAIN on the captured statement with production-equivalent typed values; label any hand reconstruction explicitly.

## Acceptance gate

An optimization is ready for the requested handoff when the relevant behavioral tests pass, SQL capture confirms the intended mechanism, representative plans support the improvement, and remaining limitations are explicit. Production plans alone do not prove API compatibility; local tests alone do not prove production speed. If a gate cannot be completed, report partial validation rather than claiming full safety.
