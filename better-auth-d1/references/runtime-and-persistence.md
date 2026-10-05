# Runtime, persistence, and recovery

## Reference evidence

Verified on **2026-10-05**, with Better Auth/core/Drizzle adapter **1.7.7**,
Drizzle ORM **0.45.3**, and local Workerd and D1. Do not generalize to another
release, plugin, provider, schema, or storage configuration.

| Exercised configuration | Local observation |
| --- | --- |
| Drizzle SQLite/D1, `transaction: false` | Native login works; User and Account are separate INSERTs. |
| Account INSERT fails after User | User persists; Account/Session are not completed. |
| Implicit linking disabled | A new OAuth flow with the same email finds the partial User and can end in `account_not_linked`. |
| Native linking, received and local email verified, `trustedProviders: []` | A new OAuth flow with a new code recovers Account and authenticates while preserving the original User. |
| Received or local email unverified | Implicit linking is denied in this configuration. |
| Email changes before Account creation | Another User can be created; the earlier one remains incomplete. |
| Native D1 adapter from the same release | Retains the relevant partial-signup and OAuth-parser limitations. |
| Direct `consumeOne`/verification consumption | Atomic consumption produces one winner; this does not mean the OAuth parser uses it. |

Linking-based recovery does not roll back writes or make User/Account atomic.
Disabling linking can be a correct identity choice; in that case, the recovery
strategy must satisfy the project's contract through another supported path.
Do not silently enable linking to bypass the error.

## Minimum inventory

Confirm **resolved** versions, not just peer ranges:

- Better Auth, core, adapters, Drizzle ORM/Kit, and enabled plugins;
- the actual package/import in use and any duplicate core/adapter packages;
- Worker bindings, compatibility date/flags, and test-runner configuration;
- generated schema, table/field names and mappings, relationships, and migrations;
- verification/state storage, session, cookie cache, and secondary storage;
- origin/base URL per environment, client/provider, and registered callbacks.

Use project commands; do not assume examples with `@latest` identify a
reproducible composition. Generate the schema with a compatible CLI, review its SQL,
and apply it only in the authorized environment. Do not run migrations per login request.

## Transactions and batches

In the Drizzle D1 0.45.3 driver, `transaction()` emits BEGIN/COMMIT/ROLLBACK;
the method's presence and type do not demonstrate that the binding supports
interactive transactions. The qualified Drizzle adapter retained `transaction: false`.

`D1.batch()` executes statements in a transaction; a statement failure aborts
the sequence. This does not automatically group two sequential operations made
by Better Auth. Trace the callback and observe the actual writes before promising
atomicity. The library's announcement of D1/batch support does not prove batching
in every signup or plugin flow.

Also distinguish zero rows from an error: a guard affecting zero rows can be
followed by another INSERT in the same batch. Dependent writes must carry the
authorization/state condition or use a supported mechanism that prevents the effect.
Test the absence of rows/effects when the guard fails.

Native D1 was added to Better Auth before the qualified release. The
[documentation for other databases](https://better-auth.com/docs/adapters/other-relational-databases)
still lists `kysely-d1` as a community dialect; this does not invalidate the
native path documented in the version 1.5 announcement. Qualify the installed path.
Switching Drizzle to Kysely/D1 does not fix a find/delete sequence in the OAuth core.

## Schema and constraints

Compare the ORM schema with the physical D1 table: columns, timestamp types/units,
nullability, defaults, relationships, indexes, and FKs. In-memory SQLite in Node
does not replace this evidence. Verify the adapter's translation of booleans/dates.

External identity must use the stable subject in the correct issuer namespace,
not email or username. When the contract requires a single link per identity,
prove the corresponding physical uniqueness, including multiple issuer/client
namespaces when applicable. Do not impose a simple key without qualifying the
provider/plugin semantics. Concurrent duplicates need a safe outcome and observable
recovery; a constraint throwing an error does not prove a completed user experience.

Core D1/Drizzle support does not imply support for every plugin. Verify each
plugin's transaction, schema, and runtime requirements; the consulted SCIM
documentation reports D1 incompatibility because it requires interactive transactions.
Do not enable SAML/SCIM, organization, or extra providers merely because they are available.

## Linking policy and signup effects

In the qualified configuration, native linking required a verified received email
and a verified local User. `trustedProviders` can waive received-email verification;
`requireLocalEmailVerified` and other flags can change the protection. Record the
effective value, not just the intent. Different providers need their own guarantees
and reauthentication when the product requires it.

Verified email means current control of a transferable/reusable address.
Discuss reuse, compromise, and managed-provider risks before approving implicit
association. Do not treat the link as historical proof that different subjects
have always belonged to the same person.

Native login with unverified email can be allowed while unverified linking is
denied. These are separate policies. Operations requiring current address ownership
cannot rely solely on an old `emailVerified` value.

Avoid granting resources, permissions, or external effects solely from
`user.create`: the callback can fail before Account/Session. When such effects
exist, choose a completed-login/linking boundary and prove idempotency.
Define the policy for incomplete User records, reconciliation, and cleanup without
deleting a valid identity or transferring permissions because emails happen to match.

## Security and performance need their own evidence

Measure query/round-trip counts and relevant plans/indexes in the actual composition.
Joins help only when the adapter and correct relationships support them; a documented
performance claim is not an application benchmark. Lower latency does not authorize
unbounded authorization caching or stale revocation reads. Qualify consistency when
using D1 Sessions/read replication.

Do not add KV, another database, a replica, or a cache layer to try to solve
something whose call graph remains incorrect. Persistence changes must preserve
the identity, recovery, concurrency, and migration evidence.
