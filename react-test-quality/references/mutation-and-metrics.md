# Mutation testing and quality metrics for React/JavaScript

## Interpret metrics correctly

Line coverage asks whether code executed. Branch coverage asks whether decision outcomes executed. Mutation testing asks whether tests distinguish selected source changes from the original.

Track when available:

- changed-code line and branch coverage;
- survived, killed, no-coverage, timeout, and error mutants;
- mutation score under StrykerJS's configured denominator;
- runtime and flake/retry count separately;
- component/browser contract coverage qualitatively, because source coverage does not express it fully.

Metrics do not prove user behavior, accessibility, browser fidelity, or absence of unmodeled regressions. Retries must not turn flakes green.

## StrykerJS workflow

Inspect `package.json`, runner, configuration, and pinned versions. Use the runner matching the repository, commonly Jest or Vitest. Narrow early analysis to changed production logic and exclude tests, declarations, generated files, stories, fixtures, and configuration only for concrete reasons.

Confirm relevant tests are selected. Related-test optimization can miss integration tests that do not import mutated modules directly. Browser-mode support differs by runner/version; consult current official documentation.

Do not add Stryker, change mutators/exclusions/thresholds, or add CI without authorization.

## Survivor triage

Classify every relevant survivor:

1. **No execution:** add the faithful component/network/browser layer or acknowledge scope.
2. **Weak oracle:** assert visible state, request, navigation, storage, or prohibited effect.
3. **Missing boundary/state:** add the discriminating example.
4. **Wrong layer or mocked-away behavior:** restore the real component/hook/client boundary.
5. **Equivalent/unobservable:** document concrete equivalence; suppress narrowly only when reviewed.
6. **Generated/defensive/JSX noise outside contract:** precise exclusion, never denominator gaming.
7. **Timeout/error:** fix determinism/configuration; never count as killed.

Do not assert internal class names/call order merely to kill a mutant. That increases mutation score while reducing refactor tolerance.

## Thresholds

Prefer measured ratchets over universal targets. Changed critical behavior should not regress; an unexplained relevant survivor can block even when the aggregate score passes.

Use `thresholds.break` only after baseline measurement, survivor/exclusion review, runtime data, and team agreement. A global score can hide a weak critical form or authorization flow behind well-tested utilities.

## Without Stryker

List plausible regressions and map each to a state/action/assertion that catches it. Establish red/green regression evidence when practical. Report that mutation analysis was not run.

## Coverage

Coverage guides investigation; it is not behavioral proof. Inspect changed branches. A rendered line may be covered while assertions ignore its output. JSX and transpilation can make line counts noisy; prioritize promised state transitions and public observations.
