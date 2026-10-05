# Java test quality review rubric

Rate each changed promise `strong`, `adequate`, or `blocking`, and cite code/evidence.

## Contract and causal story

Strong: the name and setup explain a domain promise, activating state, coherent action, and observable outcome without reading production internals.

Ask: Who relies on this? Why does this fixture activate the rule? Is setup noise hiding the cause?

Blocking: method-call name only, no meaningful assertion, or a test preserved after the business contract changed.

## Oracle discrimination

Strong: independently derived expectations cover all relevant outcome dimensions and prohibited effects.

Ask: Would wrong record, tenant, cardinality, state, payload, exception, or duplicate side effect fail?

Blocking: non-null/status/presence/call-count as sole oracle, tautological expected value, swallowed exception, or opaque snapshot.

## Boundaries and competing data

Strong: equivalence partitions and below/at/above transitions are represented; fixtures contain otherwise-valid competing rows.

Ask: Can omission of each important predicate or transition be observed?

Blocking: happy path only for changed decision logic or arbitrary invalid cases replacing real domain boundaries.

## Faithful layer and doubles

Strong: the real subsystem capable of causing the defect participates; doubles sit beyond the behavior under test.

Ask: Has a mock replaced database, mapper, validator, security, serialization, or adapter behavior being claimed?

Blocking: mocked persistence claimed as SQL proof, disabled security claimed as authorization proof, or internal choreography verification instead of outcome.

## Isolation and determinism

Strong: fresh state, explicit clock/zone/seed, bounded synchronization, reliable cleanup, and order/parallel independence.

Blocking: shared mutable fixture, leaked static/mock/cache/database state, arbitrary sleep, early test completion, order dependency, or retry masking flakiness.

## Mutation and metrics

Strong: relevant mutants are killed/triaged or explicit counterfactuals show discrimination; metric scope is reported honestly.

Blocking: relevant non-equivalent survivor accepted without explanation, no-coverage/error counted as killed, or coverage used as proof of behavior.

## Diagnostics and maintenance

Strong: one coherent reason to fail, semantic diffs, concise fixtures, and tolerance for behavior-preserving refactors.

Blocking: giant ambiguous test, duplicated production algorithm, copy-paste builders with inconsistent defaults, or assertions on irrelevant implementation details.

## Delivery

Do not approve with any blocking finding. An adequate rating requires stated residual risk. Report exact commands, scope, results, baseline failures, and what was not proved.
