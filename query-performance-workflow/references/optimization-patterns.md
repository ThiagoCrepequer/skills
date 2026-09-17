# Optimization patterns and decision criteria

## Typed predicates and existing indexes

First check whether the predicate matches the leading columns, data types, expressions, and partial predicate of an existing index. Adding the correct owner tenant to an event lookup can eliminate a table scan using an existing `(tenant_id, event_id, created_at)` index. Do not substitute the caller's tenant if ownership can differ; verify the domain flow.

For a UUID column, prefer converting the trusted typed parameter rather than every indexed value. PostgreSQL examples of the mechanism are `location_id = ANY (CAST(:locations AS uuid[]))` for a compatible array bind, or `location_id = ANY (CAST(string_to_array(:locations, ',') AS uuid[]))` for an existing comma-separated string contract. These are ORM-style illustrative fragments, not directly executable PostgreSQL packets.

Preserve empty/null parameter behavior and validate malformed input at its existing boundary. UUID comparison accepts forms that text comparison may reject; establish whether canonical UUID serialization is guaranteed. Preserve the no-location `NOT EXISTS` branch, tenant filters, and deduplication. Avoid broad changes to authorization while optimizing its predicate.

Hibernate SQL-fragment filters may auto-inject aliases into type tokens such as `uuid`. Inspect actual generated SQL. If needed, use supported explicit alias placeholders with automatic alias deduction disabled, and verify alias targets against the actual entity/table mapping and Hibernate version. Do not guess entity names.

## Replace repeated correlated work with set-based work

Look for per-row subplans scanning the same eligible set. Pre-aggregate at the correct business grain, then join results without multiplying rows. Preserve empty-set SUM/null behavior, duplicate semantics, numeric precision, denominator handling, and date boundaries. An index alone does not remove thousands of repeated subplans.

For “latest row per key,” window ranking or a supported distinct-per-group strategy can replace a correlated top-one subquery. Apply tenant/provider/removal eligibility before ranking when the original winner was selected from that eligible set. SQL equality excludes null grouping keys in some correlated forms; window partitions include them unless explicitly filtered.

Keep ordering/null precedence identical. PostgreSQL `DESC` defaults to nulls first. Ties without a stable tiebreaker make exact winner identity under-specified; do not claim byte-for-byte equivalence or silently add a new business rule. Diagnose ties and agree on deterministic behavior separately if needed.

Billing aggregation requires explicit grains: one row per group/period may require deduplication before totals. Half-open date intervals can improve access but are equivalent only if they preserve the original calendar/time-zone and inclusive/exclusive contract. Shared/batched computation must not multiply totals through many-to-many joins.

## Expose selective root-table predicates

Conditions on an optional joined row can prevent useful filtering at the root table. A recurrence query may benefit from an additional root-FK predicate separating recurrent events from date-overlapping non-recurrent events.

Retain original joined recurrence eligibility while proving the prefilter is redundant for all valid rows. Check foreign keys, dangling references, soft-deleted or ORM-filtered joined rows, and null dates. `joined.id IS NULL` is not universally equivalent to `root.foreign_key IS NULL`.

Remove an optional professionals join only when it has no required filter, projection, existence, or permission role. To-one mandatory joins may enforce existence and should not be removed casually. Keep DISTINCT until uniqueness is separately proven. An OR-to-UNION rewrite adds duplicate and branch-maintenance concerns; choose it only when measured gains justify them across representative tenants.

## Formula and loading costs

Entity formulas execute when the entity is loaded, even if the caller does not use the value. Move expensive metrics to explicit projections/read queries for actual consumers; batch by the bounded set of IDs where appropriate. Sorting or filtering by a metric must happen over the entire eligible set before pagination, not by sorting one already-selected page.

For EAGER-to-LAZY changes, inventory mapper, serializer, export, job, and detached consumers. Resolve each use case with DTO projection or intentional fetch plan and map within its legitimate transaction/session boundary. Preserve full detail DTOs even when list summaries are lightweight. Do not fix lazy exceptions with global eager loading, open-session-in-view, or blanket initialization.

Batch fetching is for measured repeated association loads within an applicable persistence context. It does not collapse repeated independent search/count requests. Measure per-request query counts and callers first; caching, request coalescing, and endpoint batching are separate designs with freshness, permission, and authorization implications.

## Search and indexes

Do not silently change wildcard, accent/case, empty-string, identifier-normalization, or removed-location behavior. Trigram indexes may help substring searches, but very short patterns, multi-column ORs, and normalization functions need actual evidence. A STABLE `unaccent` function cannot simply be mislabeled IMMUTABLE to enable an expression index. Normalized stored data has consistency and migration costs.

An index proposal must identify the access path it improves, its predicate/order, existing overlap, expected selectivity, and write/storage cost. For reverse association lookup, check whether the existing composite key starts with the opposite column. Do not rely on newer-version skip-scan behavior in older production servers.

Preserve null ordering when considering top-one index ordering. A small in-memory sort of thirteen rows is often not worth a new index. Index-only behavior depends on visibility, not just coverage. Concurrent index creation has transaction and failure-recovery constraints; plan verification of validity and rollback separately from application changes.
