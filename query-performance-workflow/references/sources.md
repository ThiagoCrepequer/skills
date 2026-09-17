# Sources and version discipline

Consulted September 2026. PostgreSQL 15 links match the historical production environment; Hibernate 7.1 links explain mechanisms, not the version to assume for a new project. Resolve the actual dependency/server versions before applying features or configuration.

## Official database documentation

- [PostgreSQL: Using EXPLAIN](https://www.postgresql.org/docs/15/using-explain.html) — estimated versus actual work, loops, buffers, plan nodes.
- [EXPLAIN command](https://www.postgresql.org/docs/15/sql-explain.html) — ANALYZE execution, instrumentation options, settings and formats.
- [PREPARE](https://www.postgresql.org/docs/15/sql-prepare.html) — parameter types and custom/generic plan behavior.
- [Index-only scans](https://www.postgresql.org/docs/15/indexes-index-only-scans.html) — visibility-map and heap-access constraints.
- [Partial indexes](https://www.postgresql.org/docs/15/indexes-partial.html) — predicate implication and parameterized-query limitations.
- [CREATE INDEX](https://www.postgresql.org/docs/15/sql-createindex.html) — concurrent-build restrictions, operational cost and failure handling.
- [Combining queries](https://www.postgresql.org/docs/15/queries-union.html) — UNION/INTERSECT/EXCEPT and duplicate semantics.
- [SET](https://www.postgresql.org/docs/15/sql-set.html) — SET LOCAL and transaction boundaries.
- [pg_trgm](https://www.postgresql.org/docs/15/pgtrgm.html) — substring-index mechanisms and selectivity limitations.

## ORM and application testing

- [Hibernate query language](https://docs.hibernate.org/orm/7.1/querylanguage/html_single/) — supported query constructs, associations, foreign-key expressions, aggregation/window capabilities.
- [Hibernate introduction](https://docs.hibernate.org/orm/7.1/introduction/html_single/) — fetching, persistence contexts, projections and loading tradeoffs.
- [Hibernate Filter annotation](https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/Filter.html) — alias deduction and explicit alias control.
- [Quarkus testing guide](https://quarkus.io/guides/getting-started-testing) — real application tests, transactions and test harness boundaries.

## Methodological inputs

The creation process consulted available `skill-creator`, `postgres-pro`, `sql-optimization-patterns`, `java-test-quality`, and `quarkus-patterns` skills, alongside application investigation records. Their useful ideas were synthesized into this self-contained workflow rather than copied as required dependencies. Generic heuristics were narrowed where measured evidence contradicted them—for example, sequential scans are not categorically bad and batch fetching does not solve every high-call-count query.

Case-study timings and decisions come from the anonymized investigation records described in [case studies](case-studies.md), not from these public documentation pages. They are illustrative historical evidence, not vendor performance claims.
