# Test matrix and evidence

## Build evidence faithful to the integration

Run real Better Auth, the selected adapter, D1 binding, schema, and cookies/redirects
in Workerd through the official Cloudflare Vitest integration or an equivalent
supported composition. Use the project's versions/configuration. Do not replace
the binding with Node SQLite and claim Workers support.

Control the provider's external HTTP and deny unexpected egress. Use only
synthetic identities/credentials. The fixture must validate issuer/client/redirect,
PKCE S256, format/verifier, single use, and code expiry when needed by the oracle;
it need not replicate the auth core. Use a separate adversarial provider that
omits a defense to expose the external assumption.

Hash/challenge needs an independent oracle: compare independent implementations
or use the RFC 7636 vector, not just two copied functions. Do not manufacture
success by having the fake always accept token requests.

## Minimum scenarios for the implemented responsibility

Select actually enabled flows and document exclusions by scope.
The scenarios below were exercised locally in the reference composition,
except those marked as future integration/operational work.

| Boundary | Stimulus | Decision-changing oracle |
| --- | --- | --- |
| Schema | ORM versus physical table; invalid FK | Correct mapping and a real constraint denying an invalid write. |
| Signup | Native OAuth with a verified private profile/email | Linked User/Account/Session, expected cookie, encrypted OAuth token. |
| Writes | Abort User, Account, and Session INSERTs separately | Persisted state after each interruption and no premature success. |
| Recovery | New OAuth flow/code after Account failure | Login completes; original User preserved when that is the contract. |
| Email | Received/local email verified and each verification absent | Linking allowed or denied by policy, without unauthorized writes/transfers. |
| Unverified signup | Provider returns unverified email | Allowed signup is not mistaken for ownership for linking/sensitive operations. |
| Identity | Linked subject changes email to another User's address | Original link preserved; identity not transferred by email. |
| Partial signup | Email changes after Account failure | Incomplete User observed; its authority/cleanup are not assumed. |
| Identity race | Concurrent flows for the same subject | Constraints/outcomes converge; a new flow recovers from a concurrent failure. |
| State/browser | Altered state, missing/unrecognized cookie, and expired state | Denied before the token request, without a new session/effect. |
| Sequential replay | Original cookie + consumed state/code | No new session or second submission in that flow. |
| Concurrent replay | Same state/cookie/code in distinct auth instances | One token request per code when required by the gate; also observe User/Account/Session. |
| New state | Already reserved/consumed code with new valid state | Reservation denies before egress; a new authorization/code still works. |
| PKCE | Two native flows | Distinct S256 challenges and matching verifier in the actual request. |
| Injection | Another flow's code with the victim's valid state/cookie | PKCE mismatch denied; no attacker identity/session. |
| Downgrade | Code issued without a challenge, with a verifier in the callback | Conforming fake denies; adversarial fake demonstrates the assumption, not a real vulnerability. |
| DTO/paths | Session, listing, token/account, and unapproved paths | Allowed fields; no secrets in JSON/HTML/redirects/logs. |

Do not conclude that the real provider revokes tokens, rejects downgrade, or
enforces expiration because the fake does. A single session does not, by itself,
satisfy the client's code-non-reuse gate.

## When adopting code reservation

Test the reservation through the actual route/provider as well as its storage primitive:

- Independent auth instances sharing D1, sequential duplicates, and a code
  resubmitted with new state. A single instance can hide a local mutex.
- D1 failure before reservation, an invalid/ambiguous binding response, and concurrency:
  nothing reaches the token endpoint without confirmed reservation success.
- Confirmed reservation and a token-endpoint response lost after acceptance:
  a retry does not resubmit; a new OAuth flow/code recovers. The fixture must
  consume the code before throwing the network failure.
- Account/Session failure after submission: the marker persists and a new login recovers.
- Distinct issuer/client scopes and ambiguous provider/plugin configurations.
- Expiration using the defined clock, duplicates without renewed expiry, bounded
  cleanup, and maintenance resumed after idle periods; no live marker is removed.
- INSERT failure rolls back cleanup when both are in the same batch.
- After retention, an expired external code is denied; also prove this with a
  **code not yet consumed**, to distinguish expiration from prior-use/PKCE rejection.
- No credential/PII in the marker, DTO, or observability.

Use fault injection in real D1, for example a temporary trigger with RAISE(ABORT),
to interrupt an exact write; remove the trigger between scenarios. This is not a
production recovery mechanism. Binding-response format failures are explicit
doubles at the D1 boundary, not a fake ORM implementation.

## Gate quality

For a fix, prove red/green through behavior, not an import/setup error.
Do not delete the earlier reproduction or invert its meaning. A baseline without
the plugin can expect two requests to document the defect; the accepted composition's
test must require one. Identify configurations in test names/arrangement.

Viability assertions are ordinary assertions and join normal test/CI commands
once the composition is approved. An isolated passing diagnosis is not acceptance.
Do not use `it.fails`, skips, lowered thresholds, or implementation exclusions to
manufacture readiness. Preserve the project's required coverage; a high percentage
or test count does not replace critical scenarios.

Await every operation, restore mocks/clock/global fetch, and reset D1/cookies
between cases. Use controlled failures/responses and observable conditions instead
of sleeps. When changing a fake, confirm that the change strengthens the external
boundary rather than making it mirror the plugin's algorithm.

## Evidence requiring additional validation

| Evidence | What it proves | What it does not prove |
| --- | --- | --- |
| Types/build/dry-run | Static compatibility and a packageable composition | Login, atomicity, or provider behavior. |
| Local Workerd/D1 | Exercised SQL/adapters/callbacks and races | Regional coordination, physical crashes, remote restore, or scheduler delivery. |
| Browser with local backend | Cookies, redirects, renewal, and flow in a real browser | The remote provider's OAuth/PKCE configuration. |
| Authorized remote smoke test | Sample of actual configuration, scopes/PKCE/expiry/cookies | Every race/failure or a universal availability guarantee. |

Qualify before promotion, as relevant to the flow: PKCE mismatch/downgrade and
expiry at the real provider; registered callbacks/origins; renewal/logout/cache
and CSRF; distributed rate limiting; concurrent refresh/secret rotation;
migration/restore; recovery and maintenance. These gates need definition/execution;
they are not results inherited from local evidence. Do not create resources or
perform remote actions without authorization.

## Compact evidence record

For each guarantee, record:

- contract and identity policy; versions and effective configuration;
- release code path and dated primary source;
- scenario and initial state, stimulus, injected failure, and independent oracle;
- first expected failure, supported public fix, commands, and result;
- exercised boundary and conditions still unproven;
- decision: approved within this scope, blocked, or investigation pending.

Do not declare all authentication secure from one case or metric.
