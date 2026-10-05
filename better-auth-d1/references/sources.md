# Fontes primárias e atualização

Conferidas em **2026-10-05**. As observações de implementação estão qualificadas
para Better Auth/core/Drizzle adapter **1.7.7** e Drizzle ORM **0.45.3**. Reabra as
fontes atuais e confira a release instalada quando usar a skill; uma versão
mais recente pode corrigir o fluxo ou mudar schema, APIs e requisitos.

## Adapter, schema e runtime

- [Better Auth: Drizzle adapter](https://better-auth.com/docs/adapters/drizzle):
  pacote, schema/mapeamentos, relações e joins.
- [Better Auth: outros bancos](https://better-auth.com/docs/adapters/other-relational-databases)
  e [kysely-d1](https://github.com/aidenwallis/kysely-d1): dialect comunitário;
  [anúncio 1.5, D1 nativo](https://better-auth.com/blog/1-5#cloudflare-d1-support)
  distingue o binding auto-detectado. Suporte batch não prova a transação de
  cada operação de negócio.
- [Better Auth: database](https://better-auth.com/docs/concepts/database):
  schema, migrations, hooks e secondary storage.
- [D1.batch](https://developers.cloudflare.com/d1/worker-api/d1-database/#batch):
  execução transacional e rollback por falha de statement.
- [Drizzle D1](https://orm.drizzle.team/docs/sqlite/connect-cloudflare-d1),
  [driver 0.45.3](https://github.com/drizzle-team/drizzle-orm/blob/0.45.3/drizzle-orm/src/d1/session.ts)
  e [SQLite date/time](https://www.sqlite.org/lang_datefunc.html):
  driver, implementação de transaction/batch e relógio UTC.
- [Vitest Workers](https://developers.cloudflare.com/workers/testing/vitest-integration/):
  runtime de testes e integração de bindings.
- [SCIM](https://better-auth.com/docs/plugins/scim): conferir incompatibilidade
  D1 e requisitos de transação do plugin antes de incluí-lo.

## Fonte da release qualificada

- [State 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/state.ts):
  caminho database find/delete e validação de cookie/prazo.
- [Callback 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/api/routes/callback.ts):
  ordem de validação e troca do code.
- [OAuth linking 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/oauth2/link-account.ts):
  User/Account, email recebido/local e recuperação sob a política escolhida.
- [Drizzle adapter 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/drizzle-adapter/src/drizzle-adapter.ts),
  [dialect Kysely 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/kysely-adapter/src/dialect.ts)
  e [contrato consumeOne](https://better-auth.com/docs/guides/create-a-db-adapter#consumeone-method):
  primitiva atômica versus call site efetivamente usado.
- [Plugins](https://better-auth.com/docs/concepts/plugins) e
  [genericOAuth 1.7.7](https://github.com/better-auth/better-auth/blob/v1.7.7/packages/better-auth/src/plugins/generic-oauth/index.ts):
  extensão pública de contexto/providers por init; verificar tipos e ordem.

## Identidade, sessão e protocolo

- [Users/accounts](https://better-auth.com/docs/concepts/users-accounts#account-linking):
  linking padrão, desativação e riscos de trusted providers.
- [Options](https://better-auth.com/docs/reference/options) e
  [Security](https://better-auth.com/docs/reference/security): configuração
  efetiva de tokens/cookies/origins e proteções da biblioteca. Não inferir que
  cobrem automaticamente comandos próprios ou todos os defaults do projeto.
- [RFC 9700 §2.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1),
  [§4.7.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.7.1) e
  [§4.8.2](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.8.2):
  premissas PKCE/CSRF, integridade de estado e downgrade.
- [RFC 6749 §4.1.2](https://www.rfc-editor.org/rfc/rfc6749.html#section-4.1.2):
  uso único e expiração de authorization code.
- [RFC 7636, apêndice B](https://www.rfc-editor.org/rfc/rfc7636.html#appendix-B):
  vetor independente para challenge S256.
- [GitHub OAuth web flow](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps)
  e [GitHub App user token](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app):
  S256/verifier correspondentes. A página OAuth App informa dez minutos para
  code; a página GitHub App consultada não repete o prazo nem explicita toda
  a regra de downgrade. Não extrapolar para GHES/outros providers ou declarar
  esses comportamentos remotamente provados.

A ausência de detalhe na documentação não prova ausência de proteção. Registre
incerteza e obtenha evidência específica; não atribua uma vulnerabilidade ao
provider a partir de um fake deliberadamente permissivo.
