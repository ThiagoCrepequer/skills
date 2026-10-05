# What a good React test is

## A controlled user-centered experiment

A test has four essential parts:

1. **Precondition:** the exact rendered, data, auth, routing, or cache state that matters.
2. **Workflow:** the user interaction or public API stimulus under examination.
3. **Observation:** visible/accessibility output, request, navigation, storage, or other public consequence.
4. **Oracle:** an independently justified decision about correctness.

Given-When-Then and Arrange-Act-Assert help readers see the story, but headings alone do not create quality. Setup without user/business meaning is noise. A workflow unrelated to the test name obscures cause. An oracle unable to distinguish plausible wrong UI behavior creates false confidence.

## The two-sided standard

A trustworthy frontend test is:

- sensitive to behavior regressions;
- tolerant of behavior-preserving refactors;
- faithful to browser/network/framework risk;
- isolated and deterministic;
- diagnostic when it fails;
- proportionate to the importance of the flow.

A test coupled to component internals breaks on harmless rewrites. A test that only checks a mocked callback can pass while the user flow is broken. A component test that mocks its own hook proves the mock composition, not the real component behavior.

## Start from the user/public promise

Prefer:

```text
Given a valid editable form
When the user submits a name containing surrounding whitespace
Then one request contains the normalized name and confirmation appears only after success
```

Avoid `shouldCallHandleSubmit`, `renders component`, or `setsLoadingTrue`. These describe implementation, not value.

Before writing, answer:

1. Who observes the behavior?
2. Which initial state activates it?
3. What can the user see, do, or receive?
4. What forbidden result must remain absent?
5. Which plausible regression should the scenario catch?
6. Does JSDOM represent the relevant behavior, or is a browser required?
7. Why is the expected result correct independently of component code?

## Strong versus weak proof

Weak:

```tsx
fireEvent.click(screen.getByTestId('save'))
expect(saveMock).toHaveBeenCalled()
```

This can pass with an inaccessible control, wrong payload, duplicate submission, premature success, or broken error state.

Stronger conceptual shape, adapted to the repository's network harness:

```tsx
const user = userEvent.setup()
renderScreen()

await user.type(screen.getByRole('textbox', { name: /name/i }), '  Ana  ')
await user.click(screen.getByRole('button', { name: /save/i }))

expect(capturedRequest.body).toEqual({ name: 'Ana' })
expect(capturedRequest.count).toBe(1)
expect(await screen.findByRole('status')).toHaveTextContent('Saved')
```

The stronger test crosses the UI and network boundary and observes the user result. A separate failure scenario should prove a visible error and absence of success. If duplicate prevention is promised, exercise rapid clicks and assert one request.

## Counterfactual review without Stryker

Ask whether the test fails if:

- a visibility/validation condition is inverted;
- a required request field is omitted or unnormalized;
- loading never ends;
- success appears on error;
- the submit handler runs twice;
- a disabled/hidden action still sends a request;
- empty and error states are swapped;
- the wrong route/query parameter is used;
- the response for request A overwrites newer request B;
- cache/store state leaks between users or tests.

If a relevant regression survives, strengthen the state, layer, interaction, or oracle. Mutation tooling automates only some source transformations.

## Meaningful fixtures

Data must make the distinction observable:

- filter tests need visible matching and otherwise-valid non-matching items;
- permission tests need a forbidden action and proof no request occurred;
- ordering tests need input order different from promised display order;
- empty tests must return an actual successful empty response, not an unresolved request;
- error tests need a controlled failure and absence of contradictory success;
- race tests need independently controlled deferred responses;
- cache isolation tests need otherwise-similar users/tenants with different data.

Use named values and small builders. Create fresh mutable data. Avoid giant fixtures, module-level mutable objects, uncontrolled current time, and random values that hide expected outcomes.

## One coherent reason to fail

One assertion per test is not a quality rule. A successful submit can coherently assert normalized request, single cardinality, loading termination, confirmation, and navigation. These describe one user outcome.

Split when preconditions, workflow, business explanation, or expected outcome differ. Avoid giant journeys testing unrelated features.

## Test doubles and boundaries

- A **stub** supplies a controlled answer.
- A **fake** implements a simplified stateful boundary.
- A **spy/mock** records an interaction.
- A **dummy** fills an unused dependency.

Prefer network interception for component/server interaction. Do not mock the hook, query library, fetch/Axios client, schema, or child component whose behavior is part of the claim. Mock true external boundaries and browser APIs that the current environment cannot provide, then add a browser test if the browser behavior itself matters.

Direct callback assertions are valid when the callback is the component's public contract. For product screens, visible behavior and actual request/navigation state are usually stronger.

## Queries are part of the contract

Prefer queries users and assistive technologies can rely on:

1. role with accessible name;
2. label text for form controls;
3. visible text or display value;
4. test ID only when no semantic/user-facing selector exists.

A failed semantic query may reveal inaccessible markup. Do not replace it reflexively with a test ID.

## Failure quality

A failure should identify the broken user promise, activating state, expected observation, and actual observation. Prefer semantic matchers and focused DOM/request diffs. Avoid broad snapshots, CSS selectors, giant provider fixtures, helpers hiding the action/assertion, and regex so broad it matches contradictory content.

## Metrics support; they do not define quality

Coverage can reveal code never executed. Mutation can reveal execution without discrimination. Flake rate can reveal unreliable evidence. Runtime can reveal unusable feedback.

A high mutation score can come from implementation-coupled assertions while a network or browser contract remains broken. A valuable component/browser test may not be selected by a narrow source mutation run. Interpret metrics through the public promise.

## Definition of done

A test is ready when its author can explain:

1. the user/public promise protected;
2. why the initial state activates it;
3. why the expected result is independently correct;
4. at least one plausible regression it catches;
5. why the environment/layer is faithful;
6. how async work and mutable state are isolated;
7. what the test does not prove.
