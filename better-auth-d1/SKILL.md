---
name: better-auth-d1
description: Implement, investigate, or review authentication with Better Auth, Cloudflare D1, and Drizzle ORM using real evidence of persistence, OAuth recovery, concurrency, and session security. Also use to compare the native D1 adapter or qualify upgrades of this combination; not for generic database migration or audits unrelated to authentication.
license: MIT
---

# Better Auth on D1 with Drizzle

Deliver a supported composition and reproducible evidence of its guarantees.
Type compatibility, a successful login, or a final session does not prove
atomicity, signup recovery, or the absence of authorization-code reuse.

## Start from the actual configuration

Read project instructions and determine the request: investigation, implementation,
review, or upgrade. Identify resolved lockfile versions, adapter/imports,
auth configuration, plugins, physical schema, migrations, bindings, compatibility date,
cookies, and HTTP surfaces. Consult official documentation and the installed
release's source; record the date, discrepancies, and provider assumptions.

Use [runtime and persistence](references/runtime-and-persistence.md) to
qualify D1/Drizzle, schema, partial signup, and linking. Its dated evidence
is a starting point, not a recommendation to pin an old version.

## Separate the guarantees

1. **Persistence:** observe the actual write order and boundary. Do not equate
   `db.transaction()` with `D1.batch()` or enable unsupported interactive transactions.
2. **Identity:** deliberately choose the linking policy. Verified email
   proves current control of the address, not historical identity or an access grant.
3. **OAuth:** verify browser/transaction binding, state integrity, PKCE,
   and single-use codes separately. An atomic adapter primitive only protects
   the flow that actually calls it.
4. **Session/HTTP:** preserve native protections and explicitly define endpoints,
   cookies, DTOs, command CSRF protection, and logging for the application.

Read [OAuth and security](references/oauth-and-security.md) for these boundaries.
A durable per-code reservation is one possible composition when necessary and
supported by the release; do not install a plugin preemptively or copy the core.

## Prove before accepting

Use real Better Auth, the adapter, and D1 in the Workers runtime. Control the
external provider's HTTP and explicitly inject persistence failures; do not fake
the adapter or its algorithm to declare the integration approved. Make a focused
test fail for the investigated cause before correcting behavior.

Select relevant scenarios from the
[test matrix and evidence](references/test-matrix-and-evidence.md): failures in
User/Account/Session, a new OAuth flow, positive/negative linking, concurrent callbacks,
replay with new state, PKCE/downgrade, cookies, token exposure, and reservation
boundaries when adopted. Count token requests per code alongside observing sessions
and persisted rows. Rerun the final composition and preserve existing gates.

Distinguish local tests, type/build/dry-run checks, and remote integration. Two auth
instances in the same isolate demonstrate concurrency in that composition; they do not
prove regional failover. A fixture that rejects downgrade does not prove enforcement
by the real provider. Do not claim a physical crash from a simulated exception.

## Fix the cause without silently changing policy

Prefer supported public configuration/APIs or a qualified official release.
Do not change linking, email verification, origin/redirect, or storage merely
to make tests pass. Do not replace the schema, adapter, or `transaction: true`
based on assumptions. Do not reopen a credential after an uncertain external outcome.

If an essential guarantee still fails, record the cause and supported alternative;
do not mark the integration ready. A diagnostic baseline can reproduce the defect,
but acceptance assertions must remain ordinary assertions and join the tests/CI
once the composition is approved. Do not use `it.fails`, skips, or coverage
exclusions to approve the desired behavior.

## Deliver useful evidence

Record configuration/versions, contract, scenario, oracle, first failure,
command and result, limitations, and the next gate. Associate the primary
[reference sources](references/sources.md) with the claims they support. Update
operational docs when the solution adds persistence, retention, or maintenance.

Follow the request's authorization: investigating or adding code does not authorize
creating OAuth Apps, secrets, a remote database, migrations, deployment, or sending messages.
Use synthetic credentials/identities in fixtures and never publish real data.
