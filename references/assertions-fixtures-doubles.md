# Assertions, fixtures, and test doubles

## Assertions that discriminate

Prefer assertions that describe the consumer-visible contract:

- scalar: exact value, tolerance only when domain-approved;
- money: `BigDecimal` value plus scale/rounding when contractual;
- optional/nullable: exact absent/present semantics and contained value;
- object: relevant fields or recursive comparison with explicitly ignored volatile fields;
- collection: exact content and multiplicity; order only when promised;
- exception: specific domain type and stable fields/message fragment;
- state transition: before/after state, version, audit, and terminal status;
- side effect: full relevant payload, destination/key, count, and forbidden calls.

`assertNotNull`, `isPresent`, `isNotEmpty`, `isTrue`, HTTP status, or mock call count can support an oracle but rarely constitute one alone.

Do not assert unstable timestamps, generated IDs, log text, SQL formatting, or internal call order unless they are the contract. Over-specification makes tests refactor-sensitive without improving defect detection.

## Expected values

Derive expectations independently:

- hand-calculate a small representative example;
- use a domain table/specification;
- compare against a trusted independent implementation only when genuinely independent;
- assert invariants for broad generated input;
- use explicit golden data only when reviewed and stable.

Never call the method under test, reuse its private helper, or duplicate its conditional structure to calculate expected output.

## Fixtures and builders

Good fixtures expose relevant differences and hide irrelevant noise:

- name values by business role (`OTHER_TENANT`, `INACTIVE_PROTOCOL`);
- default only irrelevant valid fields;
- make builders return fresh objects and collections;
- avoid giant object mothers whose implicit defaults activate unrelated behavior;
- keep required relationships visible near the scenario;
- create competing negative records for filters and authorization.

Avoid shared mutable entity instances, incrementing global counters, uncontrolled current time, environment-dependent locale/zone, and reliance on database insertion order.

## Parameterized tests

Use parameterization when many examples express the same rule and failure remains readable. Give cases semantic names or include expected values. Do not compress different business rules or complex setups into an unreadable table.

## Property-based testing

Use the project's existing property-based library when invariants span a large input space: round-trip, monotonicity, idempotency, conservation, normalization, or parser robustness. Constrain generators to the real domain, retain seeds/shrunk counterexamples, and complement properties with named boundary examples.

Do not add a property library without authorization. Generated coverage does not replace a trustworthy oracle.

## Mocks and strictness

Mock only controlled boundaries. Use strict stubbing where supported so unused stubs reveal confused setup. Reset/restore static mocks and spies reliably. Prefer a small fake when stateful behavior is easier to observe than a web of stubs.

Warning signs:

- the mock returns the exact object later asserted unchanged;
- every internal collaborator is mocked;
- the test verifies calls but not returned/persisted outcome;
- stubbing reproduces the production branch logic;
- a repository or mapper is mocked in a test claiming query/mapping correctness;
- `verifyNoMoreInteractions` blocks harmless refactors without protecting a promise.

## Exceptions and negative behavior

Use the narrowest stable exception contract. After asserting rejection, also prove relevant negative state: no persistence, no outbound call, unchanged version/balance, or retained retryability. Never catch an exception in the test merely to make it pass.

## Snapshots and golden files

Snapshots can support large stable protocols or generated documents, but require focused semantic assertions for critical rules. Keep them small, deterministic, reviewable, and free of volatile data. Updating a snapshot is a contract change and must be reviewed as such.
