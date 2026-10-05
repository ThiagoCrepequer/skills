# OAuth and integration security

## CSRF, state, and PKCE provide distinct related guarantees

RFC 9700 permits PKCE as CSRF protection when authorization-server support is
assured. Bind the challenge/verifier to the transaction and browser, use S256,
and preserve context/destination integrity. Downgrade protection requires rejecting
a verifier for a code issued without a challenge. A fixture doing this does not
prove that the real provider does it.

RFC 6749 §4.1.2 prohibits the client from reusing an authorization code; the
server must deny reuse and should revoke issued tokens when possible.
A single final session can hide two token requests and a revocation risk.
Count submissions of the same code as well as sessions.

Atomic state deletion is one possible mechanism, not the only one allowed by
the RFC. Even when adopted, it does not prove that new state cannot resubmit an
earlier code. Preserve native state/cookie/expiry checks; do not assume PKCE lets
you dispense with them or with CSRF protection for the application's own commands.

## Qualified call graph in 1.7.7

In the exercised **database state storage** path, the OAuth parser uses
`findVerificationValue` and then `deleteVerificationByIdentifier`. Two concurrent
reads can precede deletion and submit the same code to the token endpoint.
This was reproduced with Drizzle/D1 and with the native D1 adapter.

The adapter already has `consumeOne`, and `internalAdapter.consumeVerificationValue`
can use it; the exercised parser does not call that operation. Having the primitive,
changing the database/adapter, or adding `getAndDelete` to secondary storage does
not prove the OAuth flow acquired that guarantee. Revalidate the source of the
installed release/configuration, including cookie-state alternatives and paths
added by plugins. Do not generalize the finding to every release or endpoint.

When testing expiration, identify the expiry source actually read by the callback.
In the qualified release, the parser checks `expiresAt` in the serialized state
payload. Changing only the `verification.expiresAt` column does not simulate that
expiry. Align the payload, column, and units to the scenario; test the HTTP outcome
and absence of a token request, not just the database value.

A composition using the public `init(context)` API decorated the native GitHub
provider's `validateAuthorizationCode`, after native checks and before the token
request. The D1 reservation allowed one submission under concurrency between auth
instances and prevented replay with new state. This was an additional application
policy, not an official fix or activation of a ready-made library OAuth implementation.

## Durable code reservation, when adopted

First qualify the need, the release's public extension point, and every exchange
path. If an official release already satisfies the contract, do not add another
mechanism merely because an earlier defect existed.

The proven composition followed these invariants:

- Hash an unambiguous tuple containing the purpose, authorization server/issuer,
  client ID, and code. Do not partition by state, verifier, redirect, or instance:
  those values do not authorize reuse of the same code. Do not store the raw credential.
- Use a unique D1 key and `INSERT ... ON CONFLICT DO NOTHING RETURNING` before
  egress, without SELECT followed by INSERT or a local mutex/KV for mutual exclusion.
- Only a validated positive result allows calling the original provider with
  PKCE, client credentials, and redirect intact. Conflict, failure, or an
  ambiguous/malformed response denies the exchange.
- Do not remove the reservation after token rejection, timeout, crash, lost
  response, or Account/Session failure. Recovery uses a new OAuth flow/code.
  There is no D1+provider transaction or promise of exactly-once delivery.
- Static plugin/issuer/client configurations need unambiguous scope. In the
  proof, missing or dynamic configuration and providers sharing an identifier
  were denied. This qualifies that scope; it does not mean dynamic providers
  are universally invalid. They require secure scope resolution/validation.
- Preserve plugin order; a later provider replacement or alternative exchange
  endpoint can bypass the reservation. Retest the final composition.

The reservation precedes external PKCE confirmation. Someone who already knows
the victim's code and has another valid state/cookie can reserve it with an
incorrect verifier, forcing a new OAuth flow for the victim without obtaining
their identity. Consider this availability cost. Protect codes from logs/telemetry
and maintain rate limiting. Do not address this cost by permitting retries of
the same code with a different verifier.

### Retention, cleanup, and restore

Retention must exceed the code's **assured** validity in the qualified issuer/configuration,
with a margin and a consistent clock. Do not derive the duration solely from the
RFC recommendation or another provider. The proven example used 24 h, the SQLite
UTC clock, and indexed batches of 100; these numbers are not universal requirements.

Cleanup and reservation in the same `D1.batch()` roll back together if the INSERT
fails. Never delete a live reservation or renew a duplicate as a lease. Plan
maintenance when no new logins occur, capacity/cadence, and payload-free alerts.
Opportunistic cleanup does not promise physical removal during idle periods.
The operation must be bounded by batch and resumable, without an unbounded callback loop.

After retention, old values can be submitted to the server and denied as expired.
This is not permanent deduplication or authorization to reuse a valid credential.
Without qualified external expiry, do not declare cleanup safe.

Restoring a database that lost live reservations requires a recovery policy:
block callbacks/invalidate pending flows during the external validity period,
or use another proven procedure preserving the protection. Test maintenance and
failures before announcing an operational guarantee.

## Sessions, public surfaces, and data

Inspect the release's **actual** responses: native session/listing DTOs and
account/token APIs can contain secrets outside the application's public contract.
If the design preserves an HttpOnly session, do not return `session.token` in
JavaScript-accessible JSON. Access/refresh tokens, secrets, and the verifier stay
on the server; they do not enter hydrated HTML, redirects, or telemetry.

Choose necessary methods/paths/providers. Do not expose every internal endpoint
merely because it is available; if mounting the full handler, qualify each publicly
accessible capability, its authorization, and its outputs. Translate DTOs with
explicit fields, without regex or monkey-patching the native payload. Preserve
**all** `Set-Cookie` headers when adapting responses and renewal.

Browser session cookies must match the model: HttpOnly/Secure, minimal path and
scope, host-only when domain sharing is unnecessary, and SameSite compatible
with the callback. Test expiration, renewal, logout/revocation, multiple devices,
and concurrency. Cache/cookie cache needs an explicit revocation policy; local
login with caching disabled does not qualify caching.

Use native OAuth-token encryption when available/necessary
(`account.encryptOAuthTokens` in the qualified release). Prove ciphertext in D1
and use/decryption through the server path; this does not encrypt all cookies
or fields. Secret rotation, refresh, and recovery need their own evidence.
Do not create homegrown encryption to replace a supported capability.

## Application-dependent checks

These items are review/test targets, not capabilities all proven in the experiment:

- Exact base URL, trusted origins, and redirect URIs per environment; do not trust
  arbitrary Host/forwarded headers, preview wildcards, or external destinations.
- CSRF protection for cookie-authenticated commands, using an appropriate
  origin/token/protocol. CORS and SameSite do not replace this protection. The
  OAuth callback is a specific protocol exception; do not extend its CSRF bypass
  to the entire API.
- Application GET/HEAD requests do not perform sensitive mutations. Test methods/content
  types, ambiguous URLs, alternative paths, and errors without exposing people/secrets.
- Shared rate limits and counters: concurrency between instances, key/IP origin
  behind proxies, and storage failure. A fixture with rate limiting disabled
  is not production configuration.
- When explicit linking, reauth, password/reset, magic link, OTP, MFA, passkeys,
  API keys, or SSO exist, qualify their routes and credentials. Do not enable
  them to fill a checklist or inherit the OAuth proof.
- Authentication is not resource authorization. In tenant/role applications,
  test session/command scope, identity switching, revocation, and authority at
  mutation time. Do not infer permissions from email/provider.
- Logs, traces, and error reporting must use operational codes, without arbitrary
  exceptions, callback parameters, cookies, Authorization, or provider responses.
  Auditing is controlled observability; an encrypted database does not protect logs.

Normative and release sources are listed in [sources.md](sources.md).
