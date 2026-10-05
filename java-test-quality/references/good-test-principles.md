# What a good Java test is

## A controlled causal experiment

A test has four essential parts:

1. **Precondition:** the exact domain state that activates a promise.
2. **Stimulus:** the command, query, event, or request under examination.
3. **Observation:** results visible at the chosen boundary.
4. **Oracle:** an independently justified decision about correctness.

Given-When-Then and Arrange-Act-Assert help readers see this structure, but headings do not create quality. Setup without business meaning is noise. An action unrelated to the test name obscures causality. An oracle unable to distinguish plausible wrong implementations creates false confidence.

## The two-sided standard

A trustworthy test is:

- sensitive to broken behavior;
- tolerant of behavior-preserving refactors;
- faithful to the layer that can create the defect;
- isolated and deterministic;
- diagnostic when it fails;
- proportionate to the risk.

A test that fails on every refactor is brittle. A test that survives a wrong result is permissive. A test that mocks away the risky subsystem proves a different system. A test that sometimes passes is unreliable evidence.

## Start from the promise

Prefer:

```text
Given an active tenant with an associated protocol and a tenant-level plan-only protocol
When available protocols are requested for the patient
Then both valid protocols are returned exactly once and no protocol from another tenant appears
```

Avoid names such as `testFindAll` or `shouldCallRepository`. They describe code shape, not responsibility.

Before writing, answer:

1. Who observes the promise?
2. Which state activates it?
3. What outcome and prohibited outcome matter?
4. Which plausible defect should this example catch?
5. Which real subsystem must participate?
6. Why is the expected value correct without consulting the implementation?

## Strong versus weak oracles

Weak:

```java
assertNotNull(result);
assertFalse(result.isEmpty());
verify(repository).findActive();
```

This can pass with wrong records, duplicates, wrong tenant, or incorrect order.

Stronger:

```java
assertThat(result)
    .extracting(Protocol::id, Protocol::name)
    .containsExactlyInAnyOrder(
        tuple(PLAN_ONLY_ID, "Plan only"),
        tuple(ASSOCIATED_ID, "Associated")
    );
assertThat(result).extracting(Protocol::id).doesNotHaveDuplicates();
```

Assert only contract-relevant fields. Exact whole-object comparison is appropriate when the whole value is the contract; otherwise it can couple the test to incidental fields.

Expected values should come from a hand-worked example, domain rule, trusted independent source, or invariant. Never calculate expected output by calling the same production path or copying its branching algorithm into a helper.

## Counterfactual review without PIT

Ask whether the test fails if:

- `>` becomes `>=`;
- `AND` becomes `OR`;
- a required predicate is removed;
- a different tenant/owner is used;
- an empty collection becomes `null`;
- one row becomes zero or two rows;
- an exception is caught and ignored;
- a write occurs before validation or occurs twice;
- retry count changes by one;
- a terminal event is dropped.

If a relevant wrong implementation survives, change the fixture, layer, or oracle. Mutation testing automates some of this reasoning; it does not replace it.

## Meaningful fixtures

Data must make the protected distinction observable:

- filter tests need matching and otherwise-valid non-matching rows;
- tenant tests need a competing row from another tenant;
- ordering tests need insertion order different from promised order;
- uniqueness tests need overlapping sources capable of producing duplicates;
- transition tests need prior state and invalid competing transitions;
- boundary tests need values below, at, and above the actual threshold.

Use named constants and builders that expose domain roles. Keep irrelevant fields neutral. Builders must create fresh mutable objects. Random UUIDs can isolate records, but should not hide expected outcomes; generated tests must preserve reproducible seeds.

## One coherent reason to fail

One assertion per test is not a quality rule. A transfer can coherently assert returned status, both balances, a ledger entry, and absence of duplicate notification. These are facets of one atomic promise.

Split scenarios when they have different preconditions, actions, business explanations, or expected failures. Keep related assertions together when splitting would obscure the outcome.

## Test doubles

- A **stub** supplies controlled answers.
- A **fake** offers a simplified stateful boundary.
- A **spy/mock** records or verifies an interaction.
- A **dummy** fills an unused dependency.

Place doubles beyond the behavior under test. A domain service may stub an external payment provider. An adapter test should exercise the real adapter against a mock server. A repository query must use the real supported database. Do not mock the mapper, serializer, validator, or policy whose behavior the test claims to prove.

Interaction verification is valid when the interaction is an observable responsibility. Assert payload, destination/key, cardinality, and absence of forbidden calls. Avoid verifying internal call choreography.

## Boundary example

Rule: totals strictly above 100 receive a 10% discount.

```java
@ParameterizedTest(name = "total {0} produces payable amount {1}")
@CsvSource({
    "99.99, 99.99",
    "100.00, 100.00",
    "101.00, 90.90"
})
void discount_applies_only_above_the_threshold(
        String total,
        String expectedPayable) {
    var result = service.calculate(new BigDecimal(total));

    assertThat(result.payable())
        .isEqualByComparingTo(new BigDecimal(expectedPayable));
}
```

These examples distinguish `>` from `>=` and detect removal of the discount. If rounding is part of the contract, add independently calculated rounding examples.

## Failure quality

A failure should reveal the broken promise, activating scenario, expected result, and observed result. Prefer semantic assertion diffs and concise fixtures. Avoid giant multi-behavior tests, broad snapshots, unlabeled loops, helpers that hide the action/assertion, and exception assertions broader than the contract.

## Metrics support; they do not define quality

Coverage can reveal code or decisions never executed. Mutation can reveal execution without discrimination. Flake rate can reveal unreliable evidence. Runtime can reveal an unusable feedback loop.

A high score can still be produced by assertions on incidental implementation details and miss an HTTP, database, or security contract. A valuable integration test may not be selected by a narrow mutation run. Always interpret numbers through the protected promise and layer.

## Definition of done

A test is ready when its author can explain:

1. the promise protected;
2. why the fixture activates it;
3. why the expected result is independently correct;
4. at least one plausible defect it catches;
5. why the layer is faithful;
6. how state, time, and resources are isolated;
7. what the test does not prove.
