# React test quality review rubric

Rate each changed promise `strong`, `adequate`, or `blocking`, and cite code/evidence.

## User/public contract and causal story

Strong: name and setup explain the observer, initial state, coherent workflow, and public outcome without reading component internals.

Ask: What can the user see/do? Why does this state activate the behavior? Is setup noise hiding the cause?

Blocking: implementation-only name, render smoke test without outcome, or stale expectation preserving removed behavior.

## Oracle discrimination

Strong: semantic UI/request/navigation/storage assertions cover success and prohibited outcomes with independently derived expectations.

Ask: Would wrong payload, duplicate request, premature success, wrong route, stale overwrite, or permission leak fail?

Blocking: presence/callback count as sole oracle, mocked return asserted unchanged, swallowed async error, or broad snapshot.

## States, boundaries, and competing data

Strong: applicable initial/loading/success/empty/error/permission transitions and true boundaries are represented; fixtures make filtering/scoping/races observable.

Blocking: happy path only for changed decision logic or generic state quotas unrelated to owned behavior.

## Faithful layer and doubles

Strong: real component/hook/client/browser subsystem capable of causing the defect participates; doubles sit at external boundaries.

Ask: Was the hook, client, provider, child, router, or browser behavior being claimed replaced by a mock?

Blocking: mocked hook claimed as component integration, mocked navigation claimed as routing proof, or JSDOM claimed as layout/browser proof.

## Async and isolation

Strong: awaited workflows, observable completion, fresh store/cache/router/handlers/user, reliable restoration, and controlled race order.

Blocking: arbitrary sleep, side effect inside `waitFor`, unawaited work, silenced `act`/promise error, leaked handlers/timers/storage/cache, or order dependence.

## Mutation and metrics

Strong: relevant mutants are killed/triaged or counterfactual review shows discrimination; metric scope is honest.

Blocking: relevant survivor unexplained, no-coverage/error counted as killed, or line coverage used as proof of user behavior.

## Diagnostics and maintenance

Strong: one coherent user outcome, semantic queries/diffs, concise providers/fixtures, and tolerance for internal refactors.

Blocking: giant ambiguous journey, CSS/private-state assertions, giant snapshots, factory ignoring overrides, or production branching duplicated in tests.

## Delivery

Do not approve with any blocking finding. An adequate rating requires stated residual risk. Report exact commands, scope, results, baseline failures, and what was not proved.
