# Anonymized investigation cases

These cases summarize an application/ORM/PostgreSQL investigation from August–September 2026. Ranks belong to different reports and are not stable identifiers. Measurements are historical scenario observations, not benchmarks to reproduce blindly. Production parameters and identifying data are intentionally omitted. “Implemented” does not imply independent deployment verification or a full-suite run.

## 1. Billing snapshots — repeated aggregation and period identity

The original August groups 1/5 triggered aggregate-query and index work. The implementation introduced dedicated snapshot querying and period handling. Review found a period-deduplication issue requiring a separate correction and regression coverage. The user later reported deployment.

Lesson: an apparently harmless aggregation refactor can alter monetary totals when group/period identity or join multiplicity changes. Verify the business grain, periods, precision, eligibility, and duplicate handling. Do not present unrecorded production speedups as measured results.

## 2. Form loading — EAGER to LAZY is an API-contract change risk

Groups 2/3 were associated with broad form/answer/user loading. The work mapped consumers, introduced deliberate loading/projection paths, and migrated relationships to LAZY with mapping and HTTP contract tests.

A review correction restored full user details on detail/create responses while preserving lightweight list summaries. Assertions limited to names would miss missing email, active status, or roles. Mapping needed to occur while the legitimate session was available.

Lesson: avoiding a lazy exception is not enough; response completeness matters. No numeric end-to-end gain is established here.

## 3. Progress ranking — repeated correlated aggregates

Groups 6/9 repeated period/total calculations per patient. A structural local baseline took about 3.10 seconds for 9,495 patients; one period subplan ran once per patient and generated over 617,000 buffer hits. A later implementation split data retrieval into base and aggregate projections with persistence regressions and calculation tests.

Lesson: aggregate at the correct grain instead of repeatedly scanning history. Separate repository improvements from recommendations not yet proven implemented; do not infer that every suggested index or database-side pagination change shipped. The local baseline does not establish a production before/after ratio.

## 4. Patient formulas — avoid computing unused metrics

August groups 7/8/10/11/15/16/19/21 implicated three entity formulas. The implementation removed implicit formula loading and introduced explicit metric projections and metric-aware sorting, with regression tests.

Lesson: list consumers not needing metrics should not pay for them. Consumers needing them must retain values and ordering. Sorting a page after metric calculation is not equivalent to sorting the eligible population before pagination.

## 5. Latest event status — use the existing tenant-leading index

Production PostgreSQL 15 had an index on tenant, event, and descending creation time. Filtering by event alone caused a parallel scan of roughly 4.6 million history rows. Adding the correct owner tenant used the existing index.

Observed scenarios: one-history event approximately 306.8 ms → 0.080 ms; thirteen-history event approximately 315.9 ms → 0.102 ms. The latter still sorted thirteen rows, which was not the bottleneck. An earlier absent-event benchmark also showed improvement but was insufficient alone.

Lesson: validate existing and absent rows and correct ownership; do not add an index merely to remove a tiny sort. Local behavior on a newer PostgreSQL major version was not proof for production 15.

## 6. Distinct diagnoses — latest eligible row per CID

A correlated latest-row query timed out at 60 seconds in a representative production scenario. A window-ranking candidate processed 2,592 eligible rows into 46 winners in approximately 13.5 ms. The original estimated plan exposed repeated subqueries. Subsequent application work and SQL validation retained the selected implementation.

Lesson: filter eligible provider tenants/removal before selecting winners; exclude null CID when preserving the original equality semantics. Respect descending-date null precedence and under-specified ties. The timeout is a lower bound, not a measured 60-second execution time; the candidate result is not by itself proof of final ORM latency.

## 7. Indicator IDs — unnecessary join and root recurrence predicate

Baseline queries joined professionals even without a professional filter. Candidates removed that unused join and made recurrence/non-recurrence eligibility visible on the root event FK while retaining original joined recurrence conditions. Existing indexes became useful through bitmap access paths.

An OR/UNION-family alternative did not consistently win across tenants. The selected, less complicated root-predicate approach received persistence tests and Hibernate SQL capture. Equivalence reports showed 10,964 and 9,053 matching IDs, with zero differences in both directions in their respective scenarios. Final supplied plans showed roughly 158–189 ms in one scenario and 72–83 ms in another; historical baselines were roughly 292–360 ms and 288–304 ms. These were not a controlled single-session latency distribution.

Lesson: choose across representative scenarios, not the best isolated run. Preserve permissions, mandatory patient existence, recurrence rules, date boundaries, and professional-filter behavior. No new index is required merely because an existing index becomes usable.

## 8. Patient search/count — cast the parameter, not UUID columns

September ranks 7/8 were different from August formula groups. The authorization filter converted every location UUID to text. The candidate compared UUID to a UUID array, leaving the no-unit `NOT EXISTS` branch unchanged.

Two scenarios produced identical sets: 4,297/4,297 and 2,748/2,748, with zero differences both ways. Initial count samples improved approximately 68–79 ms → 48–49 ms and 22–23 ms → 13–14 ms. In the narrower case, access changed from a table scan to indexed lookup. Later generated-SQL measurements had warm counts around 49 ms and 19 ms, while first-read runs reached 490 ms and 132 ms, largely explained by reported I/O waits of 428 ms and 104 ms.

Implementation required actual Hibernate filter validation, including alias handling for the UUID type. Real-persistence regressions covered allowed/forbidden/no-unit/multiple-unit/tenant/pagination behavior. In the supplied production plans the no-unit branch was never executed; production timing did not cover it.

Lesson: reduced conversion CPU and index eligibility can matter even for fast queries. Do not promise identical speedups across cache states, nor claim an index-only scan avoids heap fetches. Do not alter authorization semantics while changing types.

## 9. Frequent queries, batch loading, and deferred investigations

- A concern about hundreds of thousands of calls prompted consideration of batch loading. Aggregate frequency alone did not establish N+1; the user deferred this path and chose the narrow typed-predicate change. Measure per-request call chains before proposing batch or caching.
- Holiday-rule optimization was explicitly deferred because another task owned it. Do not reintroduce it opportunistically.
- Reverse patient/location lookup, attempt/date queries, and large recurrence-exception batches were investigated as possible targets. Local index/scan observations were proposals, not completed production optimizations. A sequential scan returning a substantial fraction of a small table can be reasonable.
- Other high-frequency status/relationship queries and a costly dashboard routine remained separate follow-up candidates. Inspect routine side effects before treating a SELECT wrapper as safe to execute with ANALYZE.

## 10. Diagnostic workflow failures that changed the process

- Literal placeholders produced zero-row timings and an invalid timestamp. Ready-to-run packets must contain real typed parameters; discovery of units is not authorization to use all units.
- A syntax error near `offset` reinforced the need for complete statements and unambiguous parameter names rather than disconnected fragments.
- An accidental COMMIT of a read-only diagnostic transaction did not require data rollback; scoped settings end with the transaction. This does not generalize to arbitrary SELECT functions or write transactions.
- A costly baseline timeout was treated as evidence and a stopping boundary, not a reason to raise production timeout blindly.

The recurring decision pattern was: isolate one cause, characterize behavior, test a reversible SQL candidate, verify the generated ORM SQL, compare representative plans, then stop or implement within scope.
