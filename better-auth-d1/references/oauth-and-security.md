# OAuth e segurança da integração

## CSRF, state e PKCE são garantias relacionadas, não sinônimos

A RFC 9700 admite PKCE como proteção CSRF quando o suporte do servidor de
autorização está assegurado. Vincule challenge/verifier à transação e ao browser,
use S256 e mantenha a integridade do contexto/destino. A defesa contra downgrade
exige rejeitar verifier para code emitido sem challenge. Uma fixture que faz
isso não prova que o provider real o faz.

A RFC 6749 §4.1.2 proíbe reutilização de authorization code pelo cliente; o
servidor deve negar repetição e deveria revogar tokens emitidos quando possível.
Uma única sessão final pode esconder duas token requests e risco de revogação.
Conte submissões do mesmo code, além de contar sessões.

Exclusão atômica de state é um mecanismo possível, não o único permitido pela
RFC. Mesmo adotada, não prova que novo state não reenviará um code anterior.
Preserve os checks nativos de state/cookie/prazo; não use PKCE para dispensá-los
por suposição nem para dispensar CSRF dos comandos próprios da aplicação.

## Call graph qualificado em 1.7.7

No caminho **database state storage** exercitado, o parser OAuth usa
`findVerificationValue` e depois `deleteVerificationByIdentifier`. Duas leituras
concorrentes podem preceder a exclusão e submeter o mesmo code ao token endpoint.
Isso foi reproduzido com Drizzle/D1 e com o adapter D1 nativo.

O adapter já tem `consumeOne`, e `internalAdapter.consumeVerificationValue`
pode usá-lo; o parser exercitado não chama essa operação. Ter a primitiva,
mudar banco/adapter ou adicionar `getAndDelete` a secondary storage não prova
que o fluxo OAuth ganhou essa garantia. Revalide a fonte da release/config
instalada, inclusive alternativas de state em cookie e caminhos adicionados
por plugins. Não generalize o achado para toda release ou todo endpoint.

Ao testar expiração, identifique a fonte de prazo realmente lida pelo callback.
Na release qualificada, o parser verifica `expiresAt` no payload serializado
de state. Alterar apenas a coluna `verification.expiresAt` não simula esse
prazo vencido. Alinhe payload, coluna e unidades conforme o cenário; teste o
resultado HTTP e a ausência de token request, não só o valor no banco.

Uma composição via API pública `init(context)` decorou o provider GitHub nativo
em `validateAuthorizationCode`, após os checks nativos e antes da token request.
A reserva D1 permitiu uma submissão em concorrência entre instâncias auth e
impediu replay com novo state. Foi uma política adicional da aplicação, não
um fix oficial nem ativação de uma implementação OAuth pronta da biblioteca.

## Reserva durável de code, quando adotada

Qualifique primeiro a necessidade, a extensão pública da release e todos os
caminhos de troca. Se uma release oficial já satisfaz o contrato, não adicione
outro mecanismo apenas por ter existido um defeito anterior.

A composição comprovada seguiu estas invariantes:

- Hash de tupla inequívoca com propósito, authorization server/issuer, client ID
  e code. Não particionar por state, verifier, redirect ou instância: esses
  valores não autorizam repetir o mesmo code. Não armazenar a credencial bruta.
- Chave única em D1 e `INSERT ... ON CONFLICT DO NOTHING RETURNING` antes de
  egress, sem SELECT seguido de INSERT e sem mutex local/KV como exclusão mútua.
- Somente retorno positivo validado permite chamar o provider original com
  PKCE, client credentials e redirect intactos. Conflito, falha ou resposta
  ambígua/malformada nega a troca.
- A reserva não é removida após token recusado, timeout, crash, resposta perdida
  ou falha de Account/Session. Recuperação usa novo OAuth/code. Não há transação
  D1+provider nem promessa de exactly-once delivery.
- Plugin/issuer/client estáticos precisam de escopo inequívoco. Na prova, ausência,
  configuração dinâmica e providers com mesmo identificador foram negados.
  Isso qualifica aquele escopo; não significa que providers dinâmicos são
  universalmente inválidos. Eles exigem resolver/validar o escopo com segurança.
- Preserve a ordem de plugins; uma substituição posterior do provider ou endpoint
  alternativo de troca pode contornar a reserva. Reteste a composição final.

A reserva antecede a confirmação externa de PKCE. Quem já conhece o code da
vítima e possui outro state/cookie válido pode reservá-lo com verifier errado,
forçando novo OAuth da vítima sem ganhar sua identidade. Isso é um custo de
disponibilidade a considerar. Proteja codes de logs/telemetria e mantenha
rate-limit. Não tente resolver esse custo permitindo retry do mesmo code com
verifier diferente.

### Retenção, limpeza e restore

A retenção deve exceder a validade **assegurada** do code no issuer/configuração
qualificados, com margem e relógio consistente. Não derive o prazo apenas da
recomendação da RFC ou de outro provider. O exemplo provado usou 24 h, relógio
SQLite UTC e lotes indexados de 100; esses números não são requisitos universais.

Limpeza e reserva no mesmo `D1.batch()` têm rollback conjunto se o INSERT falha.
Nunca apagar reserva viva nem renovar duplicata como lease. Planeje manutenção
quando não houver novos logins, capacidade/cadência e alertas sem payload.
Limpeza oportunista não promete remoção física durante ociosidade. A operação
deve ser limitada por lote e retomável, sem loop ilimitado no callback.

Após retenção, valores antigos podem ser submetidos ao servidor e negados por
expirados. Isso não é deduplicação eterna nem autorização para reutilizar uma
credencial válida. Sem prazo externo qualificado, não declare a limpeza segura.

Restaurar um banco que perdeu reservas vivas exige uma política de retomada:
bloquear callbacks/invalidar fluxos pendentes durante a validade externa ou
outro procedimento comprovado que preserve a proteção. Teste manutenção e
falhas antes de anunciar uma garantia operacional.

## Sessão, superfície pública e dados

Inspecione respostas **reais** da release: DTO nativo de sessão/listagem e APIs
de account/token podem conter segredos que não pertencem ao contrato público
da aplicação. Se o desenho preserva sessão HttpOnly, não devolva `session.token`
em JSON acessível por JavaScript. Access/refresh tokens, secrets e verifier
ficam no servidor; não entram em HTML hidratado, redirects ou telemetry.

Escolha métodos/paths/providers necessários. Não exponha todo endpoint interno
por disponibilidade; se montar o handler completo, qualifique cada capacidade
publicamente acessível, sua autorização e seus outputs. Traduza DTOs por campos
explícitos, sem regex ou monkey patch do payload nativo. Preserve **todos** os
`Set-Cookie` ao adaptar resposta e renovação.

Cookies de sessão no browser devem corresponder ao modelo: HttpOnly/Secure,
path e escopo mínimo, host-only quando compartilhar domínios não é necessário,
SameSite compatível com o callback. Teste expiração, renovação, logout/revogação,
vários dispositivos e concorrência. Cache/cookie cache precisa de política
explícita para revogação; login local com cache desabilitado não qualifica cache.

Use criptografia nativa de OAuth tokens quando disponível/necessária
(`account.encryptOAuthTokens` na release qualificada). Prove ciphertext no D1 e
uso/decriptação pelo caminho servidor; isso não torna todos os cookies ou campos
criptografados. Rotação de secrets, refresh e recuperação exigem provas próprias.
Não crie criptografia caseira para substituir uma capacidade suportada.

## Checks que dependem da aplicação

Estes itens são alvos de revisão/teste, não capacidades todas provadas no ensaio:

- Base URL, origens confiáveis e redirect URIs exatos por ambiente; não confiar
  em Host/forwarded headers arbitrários, wildcards de preview ou destino externo.
- CSRF em comandos autenticados por cookie, com origem/token/protocolo adequado.
  CORS e SameSite não substituem a proteção. Callback OAuth é uma exceção de
  protocolo específica; não estenda bypass de CSRF a toda a API.
- GET/HEAD de aplicação não fazem mutações sensíveis. Testar métodos/content type,
  URLs ambíguas, paths alternativos e erros sem exposição de pessoas/segredos.
- Rate-limit e counters compartilhados: concorrência entre instâncias, origem da
  chave/IP atrás de proxies e falha do storage. Rate-limit desligado em fixture
  não é configuração de produção.
- Quando houver linking explícito, reauth, password/reset, magic link, OTP, MFA,
  passkeys, API keys ou SSO, qualificar as rotas e credenciais desses recursos.
  Não habilitá-los para preencher uma checklist nem herdar a prova OAuth.
- Autenticação não é autorização de recursos. Em aplicações com tenant/papéis,
  testar escopo de sessão/comando, troca de identidade, revogação e autoridade
  na mutação. Não criar permissões a partir de email/provider por inferência.
- Logs, traces e error reporting devem usar códigos operacionais, sem exceção
  arbitrária, params de callback, cookies, Authorization ou respostas do provider.
  Auditoria é observabilidade controlada; banco criptografado não protege logs.

As fontes normativas e da release estão em [sources.md](sources.md).
