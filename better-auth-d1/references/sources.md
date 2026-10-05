# Primary sources and updates

Checked on **2026-10-05**. Implementation observations are qualified for
Better Auth/core/Drizzle adapter **1.7.7** and Drizzle ORM **0.45.3**. Reopen current
sources and check the installed release when using this skill; a newer version
can fix the flow or change the schema, APIs, and requirements.

## Adapter, schema, and runtime

- [Better Auth: Drizzle adapter](https://better-auth.com/docs/adapters/drizzle):
  package, schema/mappings, relationships, and joins.
- [Better Auth: other databases](https://better-auth.com/docs/adapters/other-relational-databases)
  and [kysely-d1](https://github.com/aidenwallis/kysely-d1): community dialect;
  the [1.5 announcement, native D1](https://better-auth.com/blog/1-5#cloudflare-d1-support)
  distinguishes the auto-detected binding. Batch support does not prove the
  transaction boundary of every business operation.
- [Better Auth: database](https://better-auth.com/docs/concepts/database):
  schema, migrations, hooks, and secondary storage.
- [D1.batch](https://developers.cloudflare.com/d1/worker-api/d1-database/#batch):
  transactional execution and rollback on statement failure.
- [Drizzle D1](https://orm.drizzle.team/docs/sqlite/connect-cloudflare-d1),
  [driver 0.45.3](https://github.com/drizzle-team/drizzle-orm/blob/0.45.3/drizzle-orm/src/d1/session.ts),
  and [SQLite date/time](https://www.sqlite.org/lang_datefunc.html):
  driver, transaction/batch implementation, and UTC clock.
- [Vitest Workers](https://developers.cloudflare.com/workers/testing/vitest-integration/):
  test runtime and binding integration.
- [SCIM](https://better-auth.com/docs/plugins/scim): check D1 incompatibility
  and the plugin's transaction requirements before including it.

## Qualified release source

- [State 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/state.ts):
  database find/delete path and cookie/expiry validation.
- [Callback 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/api/routes/callback.ts):
  validation order and code exchange.
- [OAuth linking 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/oauth2/link-account.ts):
  User/Account, received/local email, and recovery under the chosen policy.
- [Drizzle adapter 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/drizzle-adapter/src/drizzle-adapter.ts),
  [Kysely dialect 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/kysely-adapter/src/dialect.ts),
  and [consumeOne contract](https://better-auth.com/docs/guides/create-a-db-adapter#consumeone-method):
  atomic primitive versus the call site actually used.
- [Plugins](https://better-auth.com/docs/concepts/plugins) and
  [genericOAuth 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/plugins/generic-oauth/index.ts):
  public context/provider extension through init; verify types and order.

## Identity, session, and protocol

- [Users/accounts](https://better-auth.com/docs/concepts/users-accounts#account-linking):
  default linking, disabling it, and trusted-provider risks.
- [Options](https://better-auth.com/docs/reference/options) and
  [Security](https://better-auth.com/docs/reference/security): effective
  token/cookie/origin configuration and library protections. Do not infer that
  they automatically cover application commands or all project defaults.
- [RFC 9700 §2.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1),
  [§4.7.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.7.1), and
  [§4.8.2](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.8.2):
  PKCE/CSRF assumptions, state integrity, and downgrade.
- [RFC 6749 §4.1.2](https://www.rfc-editor.org/rfc/rfc6749.html#section-4.1.2):
  single-use authorization codes and expiration.
- [RFC 7636, Appendix B](https://www.rfc-editor.org/rfc/rfc7636.html#appendix-B):
  independent S256 challenge vector.
- [GitHub OAuth web flow](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps)
  and [GitHub App user token](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app):
  matching S256/verifier. The OAuth App page states ten minutes for the code;
  the consulted GitHub App page does not repeat the duration or fully specify
  the downgrade rule. Do not extrapolate to GHES/other providers or claim these
  behaviors have been proven remotely.

Missing detail in documentation does not prove missing protection. Record
uncertainty and obtain specific evidence; do not attribute a vulnerability to
the provider based on a deliberately permissive fake.
