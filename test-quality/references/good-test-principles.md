# What a good test is

## The economics of testing

Production code delivers capability; tests are an insurance policy with an ongoing maintenance tax. Every test added to a repository has a continuous cost (execution time, cognitive load, maintenance during refactors).

A test is only a net asset when its **risk reduction exceeds its maintenance cost**. A bad test carries **double negative ROI**:

1. **False positives (brittle tests):** fail when internal structure changes while user/caller behavior remains intact, training engineers to distrust or blindly update tests.
2. **False negatives (permissive / theatre tests):** stay green while real defects ship, creating the dangerous illusion of safety.

## A controlled causal experiment

Regardless of language or framework, every trustworthy test has four essential parts:

1. **Precondition:** the exact domain, persistence, auth, or UI state that activates a specific promise—including competing data that must remain unaffected.
2. **Stimulus:** one coherent command, query, request, event, or user workflow executed through a public boundary (seam).
3. **Observation:** consequences visible at the public boundary (return value, state transition, rendered output, outbound protocol message, and absence of prohibited side effects).
4. **Oracle:** an independently justified decision about why the observed outcome is correct.

`Given-When-Then` and `Arrange-Act-Assert` make this causal chain visible, but comments alone do not create quality. Setup without business meaning is noise. An action disconnected from the test name obscures causality. An oracle unable to reject plausible wrong implementations creates false confidence.

## The four pillars of test value

Evaluating any test comes down to four properties:

1. **Protection against regressions (bug sensitivity):** probability that a real defect in business logic, boundaries, or integration wiring turns the test red.
2. **Resistance to refactoring (refactor tolerance):** probability that an internal code restructuring preserving observable behavior keeps the test green.
3. **Fast feedback:** speed and determinism of execution.
4. **Maintainability & diagnostics:** how clearly a failure identifies *which* promise broke and *why*.

You maximize **bug sensitivity** and **refactor tolerance** simultaneously by coupling tests to **observable contracts at public seams**, never to internal implementation details.

## Start from the promise, not the method

Prefer names and structures that state the protected invariant:

```text
Given an active tenant with a scheduled appointment from 10:00 to 11:00
When that appointment is cancelled
Then the 10:00–11:00 slot becomes available for a new booking without conflict
```

Avoid names such as `testUpdateAppointment`, `shouldCallRepository`, or `renders component`. They describe code shape or choreography, not a behavioral guarantee.

Before writing a test, answer seven questions:

1. **Observer:** Who relies on this promise (caller, user, API client, security boundary)?
2. **Precondition:** Which state activates it, and what competing state could confuse a broken implementation?
3. **Positive & negative outcome:** What must happen, and what forbidden side effect must *not* happen?
4. **Counterfactual:** Which realistic bug in production code would make this test fail?
5. **Subsystem participation:** Which real subsystem (calculator, SQL engine, HTTP pipeline, DOM) must participate so the bug can actually surface?
6. **Independent oracle:** Why is the expected value right without consulting the production implementation?
7. **Residual risk:** What does this test intentionally *not* prove?

## Counterfactual review (mental mutation testing)

Even when no mutation-testing tool is installed, review every test against plausible counterfactual defects in the production code. Ask whether the test fails if:

- `>` becomes `>=` or `<`;
- `AND` becomes `OR`, or a `WHERE` / `.filter()` predicate is deleted;
- tenant, owner, or user scoping is omitted;
- a partial update overwrites unrelated existing fields with `null` or defaults;
- a rejected request (`400`/`403`/`404`) still mutates state before throwing;
- an early return or swallowed exception skips a required side effect;
- a duplicate submission or retry executes an effect twice;
- an older async response overwrites a newer one;
- insertion order happens to match expected sort order while `ORDER BY` is missing.

If a plausible wrong implementation survives while the test stays green, strengthen the fixture, move to a more faithful layer, or sharpen the oracle.

## One coherent reason to fail

"One assertion per test" is a myth; **one coherent behavioral outcome per test** is the rule.

A single test for a cross-tenant `PATCH` attempt should assert *both* that the response is `404 Not Found` *and* that the target tenant's record in storage remains unchanged. A single test for a form submission can assert the normalized request payload, single request cardinality, and visible confirmation state. Those assertions are facets of one atomic promise.

Split tests when preconditions, stimuli, or business rules differ. Keep related assertions together when they prove the positive and negative sides of the same action.
