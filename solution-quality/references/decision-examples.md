# Decision examples

These are constructed engineering scenarios, not reports of production fixes. Their contracts are assumptions stated for comparison. Verify the actual project's contract and versions before applying a pattern. Read the matching case, not the entire catalog by default. Each case contrasts a tempting shortcut with a justified solution and evidence that could disprove it.

## Contents

- [Retry after an uncertain remote outcome](#retry-after-an-uncertain-remote-outcome)
- [Fallback without inventing success](#fallback-without-inventing-success)
- [Cache with a defined freshness contract](#cache-with-a-defined-freshness-contract)
- [Timeout versus genuinely long work](#timeout-versus-genuinely-long-work)
- [Duplicate writes under concurrency](#duplicate-writes-under-concurrency)
- [Out-of-order UI responses](#out-of-order-ui-responses)
- [Nulls and types at a boundary](#nulls-and-types-at-a-boundary)
- [Compatibility adapter for an external defect](#compatibility-adapter-for-an-external-defect)
- [Query optimization without semantic loss](#query-optimization-without-semantic-loss)
- [Abstraction with a demonstrated responsibility](#abstraction-with-a-demonstrated-responsibility)
- [Unexpected failures and misleading green tests](#unexpected-failures-and-misleading-green-tests)
- [Urgent mitigation with an explicit limit](#urgent-mitigation-with-an-explicit-limit)

## Retry after an uncertain remote outcome

**Context.** A remote payment creation request times out. The caller cannot tell whether the provider accepted it. Duplicate charges violate the contract.

**Tempting shortcut.** Retry every exception with a new request identifier, then report failure when attempts expire. This can create duplicate operations and falsely declare a completed payment unsuccessful.

**Legitimate solution.** Inspect the provider's actual idempotency and reconciliation contract. Persist the logical operation's identifier and reuse it across safe retries with the same payload. Classify retryable failures, limit attempts and total elapsed time, and avoid independent retries at every layer. Treat an unresolved outcome as pending/unknown and reconcile it through the supported status or event mechanism. If the provider offers no safe deduplication or reconciliation, do not blindly replay the write.

**Evidence.** Simulate acceptance followed by a lost response, a process restart, duplicate delivery, and retry exhaustion. Verify one effect, stable operation identity, and truthful unresolved status. A deterministic validation error should not enter the retry loop.

**Boundary.** Backoff helps transient failures; it cannot repair an invalid payload. Provider retention windows and replay semantics matter. See [AWS on retries and side effects](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) and [Stripe's provider-specific idempotency contract](https://docs.stripe.com/api/idempotent_requests); do not assume every provider behaves like Stripe.

## Fallback without inventing success

**Context.** Product requirements allow an optional recommendation panel to disappear when its service is unavailable. Order submission must still report its real outcome.

**Tempting shortcut.** Use a shared catch-all handler that converts all service errors into empty data or a successful response.

**Legitimate solution.** Confine degradation to the optional panel: show the agreed unavailable state, keep failure evidence in appropriate telemetry, and leave the rest of the page usable. Keep an empty successful result distinguishable from failed loading where that distinction affects the user. For order submission, expose failure or uncertain completion and a safe recovery path.

**Evidence.** Exercise empty results, service failure, recovery, and successful recommendations. Separately fail order submission and verify it never displays a false confirmation. Verify diagnostics remain available without logging sensitive payloads.

**Boundary.** A fallback is valid because the product permits that degraded behavior. Never infer permission to substitute estimated prices, clinical values, or authorization decisions merely because a default keeps the screen working. Ask about the unresolved behavior only when existing requirements do not answer it.

## Cache with a defined freshness contract

**Context.** A read-heavy reference catalog changes infrequently. The product explicitly permits up to one minute of staleness. Measurement shows repeated equivalent reads are a material cost.

**Tempting shortcut.** Cache every response by URL to conceal redundant requests. Ignore tenants, permissions, mutation behavior, and cache growth.

**Legitimate solution.** First remove accidental request duplication if that is the demonstrated cause. If legitimate repeated reads remain costly, use the existing cache mechanism with bounded storage, a freshness policy, and keys including the inputs that actually determine the result. Keep authorization enforcement effective on cache hits. Invalidate or version entries on writes when required by the freshness contract; do not introduce extra infrastructure when a simpler fix meets the objective.

**Evidence.** Compare hit and miss behavior, expiry, updates, tenant separation, and permission changes. Measure representative hit rates and resource use. Verify that cache failure has an explicit behavior rather than returning another tenant's or indefinitely stale data.

**Boundary.** The one-minute allowance belongs to this catalog. It does not establish an acceptable delay for access revocation or inventory reservation. If the uncached path already meets the objective, extra cache complexity may be unjustified.

## Timeout versus genuinely long work

**Context.** An export exceeds its request deadline. It might be performing redundant queries, waiting on a dependency, or doing legitimately large work.

**Tempting shortcut.** Increase a global timeout until one run succeeds, or immediately add a queue without measuring where time is spent.

**Legitimate solution.** Attribute the elapsed time first. Fix redundant work when it explains the failure. A bounded timeout increase can be correct for a measured, supported operation when caller/proxy deadlines and resource limits permit it. When legitimate work exceeds the synchronous contract, consider the existing job mechanism with durable status, authorized result retrieval, bounded resource use, and defined retry/cancellation behavior.

**Evidence.** Run typical and large supported inputs; measure duration and resource pressure. Verify timeout/failure visibility. For a job design, exercise worker interruption, duplicate delivery, result ownership, and the promised cancellation semantics.

**Boundary.** Neither a queue nor a larger timeout is inherently superior. A queue introduces lifecycle and recovery obligations; a larger timeout retains resources longer. Choose based on the required workload and operational constraints, not the desire to silence a timeout.

## Duplicate writes under concurrency

**Context.** Two workers can create the same booking for a resource and a fixed time slot. The established rule allows only one booking for that tenant/resource/slot.

**Tempting shortcut.** Disable a UI button, check whether the booking exists before inserting, add a sleep, or put an in-memory lock in one application process.

**Legitimate solution.** Enforce the exact invariant at the shared persistence boundary, for example with a composite uniqueness constraint when fixed-slot equality truly represents the rule. Handle its specific conflict with the agreed business response. UI prevention can improve usability but cannot protect all writers. Plan existing-duplicate cleanup and migration behavior rather than deleting data silently.

**Evidence.** Use independent database connections and coordinate competing attempts at the vulnerable boundary. Verify one accepted booking, the intended conflict for the other, and continued validity of bookings in different tenants or slots. Include nullability and active/cancelled rules if applicable.

**Boundary.** Arbitrary overlapping time intervals require a different invariant; a uniqueness constraint on start time is insufficient. A database constraint also does not deduplicate external side effects. See [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) and verify the target database's actual semantics.

## Out-of-order UI responses

**Context.** A search for “ca” starts before “cat” but finishes later. The UI must show results and errors only for the current query.

**Tempting shortcut.** Debounce input or add a delay and assume responses will arrive in order. Suppress an effect dependency warning to reduce requests.

**Legitimate solution.** Use the application's existing query library or request lifecycle mechanism to associate results with the current query and prevent obsolete requests from committing state. For a small manual effect, cleanup or request identity can discard stale completions. Cancel obsolete work where supported, while preserving the state-ownership guarantee even when cancellation is late or unavailable. Debouncing may still be useful to reduce request volume.

**Evidence.** Resolve controlled promises in reverse order. Verify that stale success and stale failure cannot replace the current result, error, or loading state. Exercise unmount and query changes without timing-dependent sleeps.

**Boundary.** Cancellation saves work; debounce reduces volume; guarding state updates addresses ordering. Use the mechanisms actually needed instead of introducing a new data framework for one small effect. See [React's explanation of effect cleanup and response races](https://react.dev/reference/react/useEffect#fetching-data-with-effects).

## Nulls and types at a boundary

**Context.** An external response may omit a required invoice amount, while a customer's display name is legitimately optional.

**Tempting shortcut.** Cast the response to the desired type, assert non-null, or apply `amount ?? 0` so rendering and compilation succeed.

**Legitimate solution.** Validate untrusted input at its boundary and represent the resulting valid domain state explicitly. Reject or surface an incomplete invoice when its amount is required. For the optional display name, an agreed label such as “Unnamed customer” is a legitimate presentation default. Reuse the project's validation approach; avoid rebuilding the entire type system.

**Evidence.** Exercise missing, malformed, and valid amounts, including a real zero if permitted. Verify missing data never becomes a fabricated monetary value. Verify that the optional name displays its intended fallback without changing stored identity.

**Boundary.** A type assertion can be justified when a runtime invariant is already established and the type system cannot express it locally. Keep the assertion narrow and explain the evidence. Assertions and defaults are not inherently defective; using them as substitutes for a required invariant is.

## Compatibility adapter for an external defect

**Context.** A supported SDK version incorrectly decodes a documented empty optional field. A minimal reproduction and upstream issue establish the defect. The corrected version also requires a migration outside the current task's scope.

**Tempting shortcut.** Monkey-patch the SDK globally, catch every decoding exception, or copy its implementation into multiple callers.

**Legitimate solution.** Check whether supported configuration or a compatible upgrade solves the issue first. If neither fits the authorized constraints, isolate a narrow adapter at the integration boundary, handling only the documented representation. Preserve other errors. Record the affected version, evidence, and why the upgrade is deferred. Define removal as completing the compatible upgrade and verifying the same contract without the adapter.

**Evidence.** Exercise the exact defective response through the real decoding path and confirm that unrelated malformed fields still fail. Verify normal values and callers. Link the actual upstream evidence in the implementation; this constructed example is not evidence of a real SDK defect.

**Boundary.** A permanent adapter between two genuinely different domain models need not have an artificial expiry date. A temporary patch for a known vendor bug should have a removal condition. Never normalize malformed security or financial data merely to keep processing.

## Query optimization without semantic loss

**Context.** A paginated list makes repeated related-entity queries. It also enforces tenant access, sorting, and a total count.

**Tempting shortcut.** Cache the endpoint, load the entire dataset into memory, or add an unrestricted join and deduplicate results after pagination.

**Legitimate solution.** Capture the actual generated SQL and attribute the repeated work. Choose a supported projection, batch load, or equivalent query strategy that addresses the measured cause while preserving filtering, cardinality, ordering, pagination, and transaction boundaries. A join is appropriate only when its row multiplication and pagination semantics are accounted for.

**Evidence.** Compare ordinary, empty, and high-cardinality cases, including multiple children per parent, sort ties, and unauthorized records. Verify the generated SQL, result/count equivalence, and work performed under representative parameters. A faster handwritten query alone does not establish application improvement.

**Boundary.** Do not require native SQL, an index, or a new cache by default. Use `query-performance-workflow` when available for deeper measurement guidance. Its absence does not make a straightforward, adequately verified fix impossible.

## Abstraction with a demonstrated responsibility

**Context.** Three import paths implement the same established identifier normalization rule and have begun producing inconsistent results. Elsewhere, two short formatters look similar but follow different business policies.

**Tempting shortcut.** Add another special case to each importer, or create a configurable universal transformation framework for every formatter.

**Legitimate solution.** Consolidate the shared normalization rule behind a small, explicit interface and update its real callers. Keep unrelated formatting policies separate even if a few lines remain duplicated. If the current change only affects one small formatter, a direct edit may be sufficient.

**Evidence.** Test the normalization contract and each affected caller's integration. Include conflicting or invalid inputs that previously diverged. Confirm that unrelated formatting semantics are preserved.

**Boundary.** The reason to extract is shared responsibility and coordinated change, not reaching an arbitrary duplication count. Conversely, minimizing changed lines does not justify several divergent copies of one invariant. Do not invent plugins, strategy registries, or configuration for consumers that do not exist.

## Unexpected failures and misleading green tests

**Context.** A report throws when required persisted data is missing. An existing test expects the report's real content, and the product has not approved partial output.

**Tempting shortcut.** Catch the exception and return an empty report; change the test to expect empty output; or mock away the data-loading path that fails.

**Legitimate solution.** Identify why required data is absent: invalid creation, incomplete migration, incorrect query, or a mistaken assumption about the contract. Fix the demonstrated cause within scope and retain a truthful error when required input remains invalid. If partial reports are a genuine new requirement, clarify their semantics and test that explicit contract instead of silently accepting data loss.

**Evidence.** Confirm valid inputs produce the expected report and invalid inputs produce the intended failure. Exercise the real loading boundary if the defect is in persistence. Ensure the regression fails for the original defect rather than for broken test setup.

**Boundary.** Tests can be wrong or outdated. Change them when requirements or demonstrated invariants justify it, not solely because the implementation fails them. Likewise, a narrowly justified static-analysis suppression is different from disabling a check across the project to hide an unexplained failure.

## Urgent mitigation with an explicit limit

**Context.** A newly released optional report feature exhausts workers. The user has authorized service restoration and the established operational controls permit disabling that feature.

**Tempting shortcut.** Keep adding workers or disable broad validation until alarms disappear, then call the defect fixed.

**Legitimate solution.** Use the scoped feature control or an appropriate rollback to reduce immediate impact while preserving core service behavior. Capture diagnostic evidence where practical without delaying necessary restoration. Verify recovery, disclose the unavailable feature, and continue diagnosis within the authorized task. Record what permits re-enabling the feature and how to reverse the mitigation.

**Evidence.** Observe worker/resource recovery and core request success, confirm the feature state and its user-facing behavior, and preserve the remaining uncertainty. Later, validate the permanent fix under the triggering workload before claiming resolution.

**Boundary.** Restoring service does not prove a root cause. Rollback requires compatible data and deployment state; a feature flag may leave queued work running. Check those conditions. This example does not grant production permissions or require complete diagnosis before an authorized emergency mitigation.
