# Anti-slop catalog: recognizing and eliminating test theatre

Test theatre ("test-slop") consists of tests that pass in CI, inflate test counts and coverage metrics, and consume maintenance effort while having zero or near-zero ability to detect production defects.

During authoring or code review, treat all seven patterns below as **blocking defects**.

---

## 1. Inline self-testing theatre (zero production code exercised)

**The smell:** The test file defines a local mock array or object, runs inline transformations (`.filter()`, `.reduce()`, `.map()`, regexes, or conditionals) written directly inside the test body, and asserts the result without calling any production function or rendering any component.

**Bad (Tests the language runtime, not the system):**

```ts
const mockClients = [
  { id: "1", name: "Ana", isActive: true },
  { id: "2", name: "Bruno", isActive: false },
];

it("filters active clients accurately", () => {
  const active = mockClients.filter((c) => c.isActive);
  expect(active).toHaveLength(1);
  expect(active[0].name).toBe("Ana");
});
```

*Why it fails the ROI gate:* You could delete the entire production codebase and this test would still pass.

**Good (Exercises the real production seam with competing data):**

```ts
it("returns only active clients for the current tenant when filtering by status", async () => {
  await seedClient({ tenantId: "org_a", name: "Ana", isActive: true });
  await seedClient({ tenantId: "org_a", name: "Bruno", isActive: false });
  await seedClient({ tenantId: "org_b", name: "Carla", isActive: true });

  const page = await clientService.list({ isActive: true, page: 1, pageSize: 10 });

  expect(page.items.map((c) => c.name)).toEqual(["Ana"]);
});
```

---

## 2. Compiler and reflection redundancy (`toBeDefined` / `typeof` / schema mirrors)

**The smell:** Tests that import a class, function, component, or static ORM schema object and only assert that the symbol exists (`expect(Component).toBeDefined()`, `expect(typeof fn).toBe("function")`, `expect(table.columns.name).toBeDefined()`).

**Bad (Duplicates the typechecker):**

```ts
it("exports ConfirmDeleteDialog component", () => {
  expect(ConfirmDeleteDialog).toBeDefined();
  expect(typeof ConfirmDeleteDialog).toBe("function");
});

it("defines client table columns and organization reference", () => {
  const cols = getTableColumns(clientTable);
  expect(cols.id).toBeDefined();
  expect(cols.organizationId).toBeDefined();
  expect(organizationTable).toBeDefined();
});
```

*Why it fails the ROI gate:* Static typechecking (`tsc`, `javac`, `cargo check`, `mypy`) already verifies that exported symbols and static properties exist. Notice also how `expect(organizationTable).toBeDefined()` claims to test a foreign-key relationship without inspecting any constraint metadata or database behavior.

**Good (Tests behavioral contracts or real database constraints):**

- For UI components: mount the dialog, trigger the destructive button, and verify the callback fires and loading state disables duplicate clicks.
- For database schemas: either test real constraint/migration behavior against a database engine (e.g., inserting a duplicate or null required value fails, or RLS policies block cross-tenant reads) or omit static column-name reflection tests entirely.

---

## 3. Tautological oracles (mirroring production formulas)

**The smell:** The test calculates its `expected` value by copying the production formula, calling the same helper, or repeating the same branching logic.

**Bad (Passes by construction even if the rule is wrong):**

```ts
it("calculates partner commission and salon quota", () => {
  const gross = 200_00;
  const rate = 0.4;
  const expected = Math.round(gross * rate); // Mirrors production formula
  expect(calculateCommission(gross, rate)).toBe(expected);
});
```

**Good (Uses independent, hand-worked domain examples and boundary cases):**

```ts
it("splits a R$ 200,00 service into R$ 80,00 partner commission and R$ 120,00 salon quota at 40%", () => {
  expect(calculateSplit(200_00, 0.40)).toEqual({
    partnerCommissionCents: 80_00,
    salonQuotaCents: 120_00,
  });
});

it("conserves every cent on odd rounding boundaries (R$ 100,00 at 33.33%)", () => {
  const split = calculateSplit(100_00, 0.3333);
  expect(split.partnerCommissionCents + split.salonQuotaCents).toBe(100_00);
});
```

---

## 4. Tautological fakes and leaky mock engines

**The smell:** A hand-rolled fake database or mock adapter takes shortcuts that bypass the production code's actual predicates, sorting, or aggregations:

- **Any-column parameter matching:** extracting bind parameters from a query AST and checking `Object.values(row).some(val => val === param)`—which passes even if the production code compared the wrong column (`WHERE parent_id = $1` instead of `WHERE id = $1`).
- **Hardcoded metric re-implementation in the fake:** when `service.getMetrics()` builds SQL `COUNT` / `AVG` expressions, the fake DB ignores the SQL AST and computes the metrics using JavaScript `.filter()` written inside the test utility. The test then tests the test utility, not the service's SQL!
- **No-op `ORDER BY` in the fake:** the fake chains `.orderBy()` as an identity function `() => rows` without sorting, while fixtures are inserted in pre-sorted order.

**Good:**

- For SQL queries, aggregations, sorting, and RLS: run against a real embedded or containerized database engine (e.g., PGlite, SQLite, Testcontainers) or inspect the exact bound SQL/parameters when testing a query builder unit.
- Always seed records in **non-sorted order** when testing sorting, and include **competing rows** with overlapping values in other columns so column-targeted predicates are proved.

---

## 5. Mocking the subject or asserting what the mock was told to return

**The smell:** A test stubs a dependency to return a prefabricated object and then asserts that the caller returned that same object unchanged, or mocks the very hook/client/mapper whose behavior is claimed by the test.

**Bad:**

```ts
it("creates a user with normalized email", async () => {
  mockRepo.save.mockResolvedValue({ id: "1", email: "ana@example.com" });
  const result = await service.register({ email: "  ANA@EXAMPLE.COM " });
  expect(result.email).toBe("ana@example.com");
});
```

*Why it fails the ROI gate:* Even if `service.register` never normalized the email, `mockRepo.save` returns `"ana@example.com"` unconditionally, so the assertion always passes.

**Good:**

Either use an in-memory fake repository that stores and returns whatever was actually passed to it, or assert the exact normalized input received at the boundary.

---

## 6. Presence-only oracles and unasserted spies

**The smell:**

- Checking only `expect(result).toBeDefined()`, `expect(res.status).toBe(200)`, or `expect(list).not.toBeEmpty()` when specific records, field values, or exclusions are the contract.
- Creating a spy (`const mockSend = vi.fn()`) and passing it into the function under test, but forgetting to assert `expect(mockSend).toHaveBeenCalledWith(...)`.
- Asserting only `expect(mockRepo.save).toHaveBeenCalled()` without checking the arguments or the resulting state.

**Good:**

Always assert the **semantic content** of the output and, when an outbound interaction is the contract, assert the **exact payload, recipient/destination, and call count (`toHaveBeenCalledTimes(1)`)**.

---

## 7. Metadata-only route/screen smoke placeholders

**The smell:** Creating `feature.test.tsx` files for pages or routes that only invoke a static metadata helper (`expect(meta()[0].title).toContain("Clients")`) or check query-key constants (`expect(keys.all).toEqual(["clients"])`) while leaving the screen's interactive workflows, form validations, and error states completely untested.

**Good:**

Delete redundant constant-mirror tests or consolidate them into a single contract test, and dedicate screen test files to user-observable workflows (rendering data, filtering, submitting valid/invalid forms, handling server errors, and enforcing permissions).
