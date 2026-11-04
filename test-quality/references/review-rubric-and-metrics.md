# Test quality review rubric and metrics guide

Use this rubric when auditing an existing test suite or reviewing new/modified tests before delivery. Rate each changed or audited contract `strong`, `adequate`, or `blocking`, and cite concrete file and line evidence.

---

## 1. Contract and causal story

- **Strong:** Test name and setup clearly communicate the observer, activating precondition, single coherent stimulus at a public seam, and expected domain/user outcome without reading production internals.
- **Ask:** What real capability does this prove? Why does this fixture activate the rule? Is setup noise obscuring causality?
- **Blocking:**
  - Inline self-testing theatre (testing `.filter()` / `.reduce()` written inside the test file without invoking production code).
  - Compiler/reflection redundancy (`expect(typeof X).toBe("function")`, static ORM column-existence mirrors).
  - Method-call or choreography-only test names (`shouldCallRepository`, `renders component`) without behavioral verification.

## 2. Oracle discrimination (positive and negative)

- **Strong:** Independently derived expectations verify all relevant facets of the outcome—exact values, persisted state transitions, preserved fields on partial updates, outbound payloads/cardinality, and absence of prohibited side effects on failure or rejection.
- **Ask:** Would a wrong record, wrong tenant, omitted field, wiped sibling column on `PATCH`, duplicate side effect, or premature success fail this test?
- **Blocking:**
  - Presence/status/call-count as sole oracle (`toBeDefined()`, `status === 200`, `not.toBeEmpty()`).
  - Tautological oracle (recomputing expected output using the production formula).
  - Unasserted spy (`const spy = vi.fn()` passed in, never asserted).
  - Rejected mutation (`400`/`403`/`404`) tested without verifying that underlying state remained untouched.
  - Broad, unreviewed snapshots used as the sole behavioral check.

## 3. Boundaries and competing data

- **Strong:** Equivalence partitions and exact boundary transitions (`below`, `at`, `above`) are represented; fixtures include competing valid records (other tenant, inactive item, non-matching filter, out-of-order insertion) so filtering, scoping, and sorting are genuinely tested.
- **Ask:** If a `WHERE` clause, tenant check, or `ORDER BY` were removed or inverted (`AND` -> `OR`), would competing fixture data expose the bug?
- **Blocking:**
  - Happy-path-only coverage for changed branching/decision logic.
  - Filter, search, or multi-tenant test seeded only with matching/single-tenant rows.
  - Sorting test seeded with rows already in the expected sort order.

## 4. Faithful layer and test doubles

- **Strong:** The real subsystem capable of causing the defect (pure calculator, SQL engine, HTTP middleware pipeline, state store, or accessible DOM) participates; doubles sit strictly at external boundaries.
- **Ask:** Was the database query, validator, mapper, hook, router, or auth guard being claimed replaced by a mock or a shortcut fake?
- **Blocking:**
  - Mocking the unit under test or asserting the value a stub was configured to return.
  - Tautological fake DB that matches bind parameters against *any* column (`Object.values(row).some(...)`), ignores `ORDER BY`, or re-implements SQL aggregations (`COUNT`/`AVG`/`SUM`) in JavaScript inside the test helper.
  - Claiming authorization/tenant isolation while security middleware is bypassed.

## 5. Isolation, determinism, and async discipline

- **Strong:** Fresh state per test, injected/frozen clock and timezone, deterministic seeds, awaited stimuli, and controlled deferred promises for async races. Passes alone, repeatedly, and in any order.
- **Blocking:**
  - Shared mutable module-level fixtures or leaked cache/DB/mock state across tests.
  - Arbitrary `sleep` / `setTimeout` delays used to wait for async completion.
  - Side effects placed inside retrying `waitFor` callbacks.
  - Unawaited promises or silenced framework/console warnings.

## 6. Mutation testing and coverage honesty

- **Strong:** Relevant mutants in changed business logic are killed (via mutation tooling such as PIT/Stryker or explicit counterfactual review); coverage and mutation scope are reported accurately.
- **Blocking:**
  - Relevant non-equivalent mutant survivor in critical logic left unexplained.
  - Treating raw line coverage as proof of correctness while assertions are weak or tautological.
  - Gaming denominators by mutating/covering boilerplate while ignoring core decision branches.

## 7. Diagnostics and maintenance cost

- **Strong:** One coherent reason to fail per test, semantic failure diffs, concise builders with domain-meaningful overrides, and high tolerance for internal refactoring.
- **Blocking:**
  - Giant multi-journey tests where a failure could mean ten different broken rules.
  - Brittle assertions on private state, CSS utility classes, or internal helper call order.
  - Factories that silently ignore caller overrides.

---

## Interpreting metrics honestly

Metrics are **diagnostic instruments, never the goal**:

- **Line coverage** only reveals code that was never executed. A line executed by a test with no meaningful assertion has 100% line coverage and 0% bug sensitivity.
- **Branch coverage** reveals decision outcomes (`true`/`false`) never taken, but still cannot tell whether the oracle checked the result.
- **Mutation score / test strength** measures whether tests notice small syntactic changes (`>` to `>=`, `+` to `-`, removed call). Even a high mutation score can be gamed by implementation-coupled mocks if the integration seam is broken.

### Survivor triage workflow

When a counterfactual or mutant survives in changed logic, classify and resolve it:

1. **Not executed:** add a test at the lowest faithful layer, or remove dead production code.
2. **Weak oracle:** strengthen the assertion to check the exact value, state transition, or side effect.
3. **Missing boundary or competing row:** add the boundary input (`threshold` vs `threshold + 1`) or competing fixture row.
4. **Wrong layer (mocked away):** replace the mock/shortcut fake with the real subsystem or faithful fake.
5. **True semantic equivalence:** document why the mutation cannot change observable behavior; never blindly chase 100%.

---

## Delivery gate

Do not approve a test suite change when any **blocking** finding remains. When delivering or auditing tests, always state:

1. The public contracts, boundaries, and negative invariants proved.
2. Any test-slop removed or replaced and why.
3. Exact commands executed, test counts, and results.
4. What was not executed and what residual risk remains.
