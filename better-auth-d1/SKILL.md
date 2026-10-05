---
name: better-auth-d1
description: Implementar, investigar ou revisar autenticação com Better Auth, Cloudflare D1 e Drizzle ORM, usando provas reais de persistência, recuperação OAuth, concorrência e segurança de sessão. Use também para comparar o adapter D1 nativo ou qualificar upgrades dessa combinação; não para migração genérica de banco ou auditoria sem relação com autenticação.
license: MIT
---

# Better Auth no D1 com Drizzle

Produza uma composição suportada e uma prova reproduzível das suas garantias.
Compatibilidade de tipos, um login bem-sucedido ou uma sessão final não provam
atomicidade, recuperação de cadastro ou ausência de repetição de code.

## Parta da configuração real

Leia as instruções do projeto e determine o pedido: investigação, implementação,
revisão ou upgrade. Identifique versões resolvidas no lockfile, adapter/imports,
config auth, plugins, schema físico, migrations, bindings, compatibility date,
cookies e superfícies HTTP. Consulte documentação oficial e fonte da release
instalada; registre data, divergências e premissas do provider.

Use [runtime e persistência](references/runtime-and-persistence.md) para
qualificar D1/Drizzle, schema, cadastro parcial e linking. A evidência datada ali
é um ponto de partida, não recomendação de fixar uma versão antiga.

## Separe as garantias

1. **Persistência:** observe a ordem e a fronteira real das escritas. Não equipare
   `db.transaction()` a `D1.batch()` nem habilite transação interativa sem suporte.
2. **Identidade:** escolha conscientemente a política de linking. Email verificado
   prova controle atual do endereço, não identidade histórica ou grant de acesso.
3. **OAuth:** verifique vínculo com browser/transação, integridade de state, PKCE
   e uso único de code separadamente. Uma primitiva atômica no adapter só protege
   o fluxo que realmente a chama.
4. **Sessão/HTTP:** preserve as proteções nativas e defina explicitamente endpoints,
   cookies, DTOs, CSRF de comandos e logging conforme a aplicação.

Leia [OAuth e segurança](references/oauth-and-security.md) para essas fronteiras.
Uma reserva durável por code é uma composição possível quando necessária e
suportada pela release; não instale um plugin preventivamente nem copie o core.

## Prove antes de aceitar

Use Better Auth, adapter e D1 reais no runtime Workers. Controle HTTP do provider
externo e injete falhas de persistência explicitamente; não simule o adapter ou
seu algoritmo para declarar a integração aprovada. Faça um teste focado falhar
pela causa investigada antes de corrigir comportamento.

Selecione os cenários pertinentes em
[matriz de testes e evidência](references/test-matrix-and-evidence.md): falhas em
User/Account/Session, novo OAuth, linking positivo/negativo, callbacks concorrentes,
replay com novo state, PKCE/downgrade, cookies, exposição de tokens e limites da
reserva, quando adotada. Conte token requests por code, além de observar sessão
e linhas persistidas. Reexecute a composição final e preserve gates existentes.

Distinga teste local, tipo/build/dry-run e integração remota. Duas instâncias auth
no mesmo isolate demonstram concorrência nessa composição; não provam failover
regional. Uma fixture que rejeita downgrade não prova enforcement no provider
real. Não declare um crash físico a partir de uma exceção simulada.

## Corrija a causa sem mudar a política silenciosamente

Prefira configuração/API pública suportada ou release oficial qualificada.
Não altere linking, verificação de email, origem/redirect ou storage apenas para
obter testes verdes. Não substitua schema, adapter ou `transaction: true` por
suposição. Não reabra uma credencial após resultado externo incerto.

Se uma garantia essencial continua falhando, registre a causa e a alternativa
suportada; não marque a integração como pronta. Um baseline diagnóstico pode
reproduzir o defeito, mas os asserts de aceite devem continuar normais e integrar
os testes/CI após a composição ser aprovada. Não usar `it.fails`, skips ou
exclusões de cobertura para liberar o comportamento desejado.

## Entregue evidência útil

Registre configuração/versões, contrato, cenário, oráculo, primeira falha,
comando e resultado, limites e próximo gate. Relacione as fontes primárias de
[referência](references/sources.md) à afirmação que sustentam. Atualize os docs
operacionais quando a solução adicionar persistência, retenção ou manutenção.

Siga as autorizações do pedido: investigar ou adicionar código não autoriza
criar OAuth Apps, secrets, banco remoto, migration, deploy ou enviar mensagens.
Use credenciais/identidades sintéticas em fixtures e nunca publique dados reais.
