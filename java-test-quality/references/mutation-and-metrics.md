# Mutation testing and quality metrics for Java

## Interpret metrics correctly

Line coverage asks whether instructions executed. Branch coverage asks whether decision outcomes executed. Missed complexity points to untested decision structure. Mutation testing asks whether tests distinguish selected changed bytecode from the original.

Track when available:

- changed-code line and branch coverage;
- missed branches/complexity in changed business logic;
- killed, survived, no-coverage, timeout, and error mutants;
- mutation score under PIT's denominator;
- test strength: killed divided by killed plus survived when reported;
- runtime and flake/retry count separately.

Metrics cannot prove the contract, layer fidelity, or absence of unmodeled defects. Never use retries to turn flakes green.

## PIT workflow

Inspect the Maven/Gradle build and existing PIT/JUnit integration before choosing versions or commands. A common Maven invocation is:

```text
./mvnw test-compile org.pitest:pitest-maven:mutationCoverage
```

Start with changed business packages/classes and relevant tests. Confirm PIT actually selects the expected tests; integration/Quarkus tests may not participate automatically in every setup. Use dry-run/history/incremental facilities only when supported by the pinned project version.

Do not insert `LATEST` into a build. Do not add PIT, alter mutators, exclusions, thresholds, or CI without authorization.

## Survivor triage

Classify every relevant survivor:

1. **No execution:** add the faithful test layer or acknowledge missing scope.
2. **Weak oracle:** assert exact output/state/effect.
3. **Missing boundary/truth-table case:** add the discriminating example.
4. **Wrong layer:** replace mocked proof with database/HTTP/component evidence.
5. **Equivalent/unobservable:** document concrete semantic equivalence; suppress narrowly only when stable and reviewed.
6. **Generated/defensive/logging code outside contract:** precise exclusion, never broad denominator gaming.
7. **Timeout/error:** fix determinism/configuration; never count as killed.

Do not rewrite production code solely to make PIT happy unless the rewrite genuinely improves design while preserving behavior.

## Thresholds

Prefer a measured ratchet over aspirational universal numbers. Changed critical code should not lower the accepted baseline; an unexplained non-equivalent survivor in changed behavior blocks acceptance even if the aggregate score passes.

Select `mutationThreshold`, `testStrengthThreshold`, and precision only after baseline measurement, equivalent/exclusion review, runtime data, and team agreement. A global score can hide weak critical code behind strong trivial code.

## Without PIT

Perform explicit counterfactual review: list plausible wrong implementations and show which input/assertion catches each. Red/green regression evidence is especially valuable. Report that mutation analysis was not run; do not manufacture a mutation claim.

## Coverage

Coverage is a map for investigation, not an acceptance proof. Inspect branches and missed complexity in changed decisions. A line can be covered while all assertions ignore its result. Exception behavior may require explicit tests even when branch counters do not model it as a branch.
