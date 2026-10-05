# Choosing Java and Quarkus test layers

## Plain JUnit

Use for pure domain rules, value objects, parsers, formatters, policies, and deterministic state transitions. Instantiate the subject directly. Do not boot Quarkus only to test pure logic.

Good proof: exact output/invariant and meaningful boundaries. Not proved: CDI resolution, interceptors, configuration, serialization, persistence, security, or HTTP mapping.

## Quarkus component tests

Use `@QuarkusComponentTest` when the behavior depends on CDI injection, qualifiers, interceptors, configuration, or a small bean graph but does not need the whole application. Keep unsatisfied dependencies explicit and controlled. Do not let auto-created mocks conceal a dependency that should be real for the claimed behavior.

Good proof: component behavior with relevant CDI mechanics. Not proved: full HTTP pipeline, complete application discovery, real database, or packaged runtime.

## Repository and database tests

Use the same database engine as production when dialect, SQL, JSON, arrays, collation, constraints, sequences, locking, timestamps, or query plans can affect correctness. Reuse the project's Quarkus Dev Services/Testcontainers setup. H2 is not proof of PostgreSQL semantics.

Persist competing rows that expose missing predicates. Assert exact IDs/fields, multiplicity, pagination/order contract, and persisted state. For constraints, assert the database rejection and transaction aftermath.

Shared containers share infrastructure, not application state. Clean or namespace rows explicitly. A test transaction rolls back only work participating in that transaction; an HTTP request may use a different transaction.

## Quarkus HTTP/application tests

Use `@QuarkusTest` when behavior includes:

- Jakarta Validation;
- JSON field names, omission/null semantics, enum handling, or serialization;
- media type and headers;
- status and exception mapping;
- filters/interceptors;
- authentication, roles, permissions, tenant context;
- transaction boundary or application wiring.

Assert status, stable response/error body, relevant headers, state, and side effects. A 2xx alone is weak. A resource test that mocks the service proves only the resource boundary; add a deeper layer if business logic is part of the claim.

## Security and tenancy

For a secured behavior, include applicable scenarios:

- unauthenticated;
- authenticated without required role/permission;
- correctly authorized;
- correct role but wrong tenant/owner/location scope;
- confidential fields absent from serialized output.

Do not disable authorization in a test that claims to prove it. Test identities must express relevant roles, permissions, and attributes. Include an otherwise-valid competing entity outside the allowed scope.

## Outbound adapters

Exercise the real adapter against WireMock or the project's equivalent. Verify request path, method, headers, serialization, response parsing, timeout, retry policy, and error mapping. Mock the remote server, not the adapter.

For messaging, exercise serialization/key/headers and consumer acknowledgement/redelivery semantics at the existing broker/test boundary when those are contractual.

## Integration/package/native tests

Use the packaged or native layer only for risks that differ from JVM/application tests: reflection/serialization registration, native resources, filesystem/container packaging, startup configuration, or runtime protocol behavior. Do not duplicate the entire JVM suite without a risk-based reason.

## Cross-layer contracts

DTO/API changes need at least one real serialization or HTTP test. Database-backed endpoint changes often need both a focused repository test and one HTTP contract test. A unit test cannot prove a JSON contract; an HTTP test with all business services mocked cannot prove the business rule.

State validation honestly: a focused layer is evidence for that layer, not proof of the whole system.
