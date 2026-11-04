# Discriminating oracles, adversarial fixtures, and boundaries

## Building a discriminating oracle

An oracle is the mechanism by which a test decides whether the observed outcome is right or wrong. A weak oracle accepts broken implementations; an over-specified oracle breaks on harmless refactors.

### Strong vs. weak oracles across domains

| Contract type | Weak oracle (permissive) | Strong oracle (discriminating) |
| :--- | :--- | :--- |
| **Query / Filter** | `expect(items).not.toBeEmpty()` | Exact matching IDs/fields returned **and** competing non-matching/other-tenant items excluded |
| **Sorting** | Asserting items inserted in already-sorted order | Seeding items out of order (e.g., `[B, C, A]`) and asserting `[A, B, C]` for `ASC` and `[C, B, A]` for `DESC` |
| **Creation / Mutation** | `expect(res.status).toBe(201)` | Status + persisted record has expected fields + spoofed/protected fields (e.g., `organizationId`, `id`, `isOwner`) were stripped or overridden |
| **Partial Update (`PATCH`)** | Checking only the updated field (`expect(updated.isActive).toBe(false)`) | Checking both the updated field **and** that untouched fields (`email`, `role`, `workingHours`, `tags`) were preserved |
| **Lifecycle / Cancellation** | Checking `status === "cancelled"` | Checking status + downstream invariant (e.g., the cancelled time slot is now bookable by another client without conflict) |
| **Rejection (`400`/`403`/`404`)** | Checking only `expect(fn).toThrow()` or `status === 404` | Checking specific error code/type **plus** negative state oracle (target record in DB was not modified/deleted; no outbound event/email was emitted) |
| **UI State / Loading** | `expect(container).toBeDefined()` | Skeleton hides stale row text while keeping toolbar/headers interactive; error banner appears and success toast does not appear on `500` |

### Independent expected values

Always derive expected values from an independent source of truth:

- **Hand-worked literal examples:** calculate the exact expected numbers, dates, or strings by hand from the domain rules (`expect(total).toBe(150)`).
- **Domain invariants & round-trips:** for wide input spaces, assert algebraic laws—e.g., `calculateCountFromEndDate(calculateEndDateFromCount(start, count)) === count`, or `sum(splits) === grossTotal`.
- **Golden specifications:** explicit domain decision tables reviewed by humans.

Never compute expected values by calling the function under test or duplicating its conditional/loop logic in the test.

## Adversarial and competing fixtures

A fixture is only meaningful if a broken implementation would return the wrong fixture data. Design every fixture with **competing records**:

1. **Filter & search tests:** Seed at least one record that matches the filter, one record that fails predicate A, and one record that fails predicate B.
2. **Multi-tenant & authorization tests:** Always seed Tenant Alpha and Tenant Beta with realistic records. Prove that Tenant Beta listing records sees only Beta records, that `GET / PATCH / DELETE` on Alpha's ID returns `404`/`403` and leaves Alpha's state intact, and that passing Alpha's ID as a foreign key (e.g., `roleId` or `tagId`) in Beta's payload is rejected.
3. **Uniqueness & deduplication tests:** Seed overlapping sources that would produce duplicates if `DISTINCT` or set deduplication were omitted.
4. **Scoped catalog tests:** When entities share a table discriminated by `type` (e.g., `tag` with `type: "client" | "member"`), seed the same name under both types and prove they do not collide and cannot be cross-assigned.
5. **Clean builder defaults:** Use small factory functions (`makeMember({ id: "m1", isOwner: true })`) that provide valid defaults for irrelevant fields while forcing the test author to pass the fields that drive the scenario at the call site. Builders must always return fresh object instances.

## Equivalence partitioning and boundary analysis

Do not write ten tests with random values from the middle of the same valid partition (`age = 25`, `age = 30`, `age = 35`). Instead:

1. Identify the **equivalence partitions** (e.g., before shift, during morning shift, inside lunch break, during afternoon shift, after shift).
2. Test **exact boundary transitions** (` threshold - 1`, `threshold`, `threshold + 1`):
   - Time intervals: adjacent appointments (`10:00–11:00` and `11:00–12:00` must *not* conflict) vs. 1-minute overlap (`10:00–11:00` and `10:59–12:00` *must* conflict) vs. buffer time overlap.
   - Pagination: empty dataset (`0` items -> `totalPages: 0` or `1`), exact page boundary (`10` items with `pageSize: 10` -> `1` page), and `11` items (`2` pages, second page has `1` item).
   - Recurrence & dates: month/year rollover (Dec 31 -> Jan 1), leap years, and timezone offsets (local midnight vs. UTC ISO 8601).

## Parameterized and property-based tests

- **Parameterized tests (`test.each` / `@ParameterizedTest`):** Use when multiple inputs exercise the same rule or decision table (e.g., precedence hierarchy of status timestamps or validation boundary cases). Include the scenario reason and expected outcome in each row label so failures are immediately diagnostic. Do not cram unrelated workflows with different setup requirements into a single table.
- **Property-based tests:** Use when invariants must hold across a vast input space (serialization round-trips, parser safety, mathematical conservation, layout non-overlap). Always log or fix the random seed so failures are 100% reproducible, and pair properties with explicit named edge-case examples.
