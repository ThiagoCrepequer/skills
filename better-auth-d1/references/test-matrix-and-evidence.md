# Matriz de testes e evidência

## Monte uma prova fiel à integração

Execute Better Auth, adapter escolhido, binding D1, schema e cookies/redirects
reais em Workerd pela integração Vitest oficial Cloudflare ou composição
suportada equivalente. Use as versões/configs do projeto. Não substitua o
binding por um SQLite Node e declare suporte Workers.

Controle o HTTP externo do provider e negue egress inesperado. Use apenas
identidades/credenciais sintéticas. A fixture deve validar issuer/client/redirect,
PKCE S256, formato/verifier, uso único e prazo de code quando necessários ao
oráculo; não precisa replicar o core auth. Separe um provider adversarial que
omite uma defesa para expor a premissa externa.

Hash/challenge deve ter um oráculo independente: confronte implementações
independentes ou o vetor da RFC 7636, não apenas duas funções copiadas. Não
fabrique sucesso porque o fake sempre aceita token requests.

## Cenários mínimos da responsabilidade implementada

Selecione os fluxos efetivamente habilitados e documente exclusões por escopo.
Os cenários abaixo foram exercitados localmente na composição de referência,
exceto os marcados como integração/operação futura.

| Fronteira | Estímulo | Oráculo que muda a decisão |
| --- | --- | --- |
| Schema | ORM versus tabela física; FK inválida | Mapeamento correto e constraint real negando escrita inválida. |
| Signup | OAuth nativo com perfil/email privado verificado | User/Account/Session vinculados, cookie esperado, token OAuth cifrado. |
| Escritas | Abort no INSERT User, Account e Session, separadamente | Estado persistido após cada interrupção e ausência de sucesso prematuro. |
| Recuperação | Novo OAuth/code depois da falha Account | Login conclui; User original preservado quando esse é o contrato. |
| Email | Recebido/local verificados e cada verificação ausente | Linking permitido ou negado conforme política, sem escrita/transferência indevida. |
| Signup não verificado | Provider retorna email não verificado | Não confundir signup permitido com ownership para linking/operação sensível. |
| Identidade | Subject vinculado muda email para o de outro User | Vínculo original preservado; identidade não transferida por email. |
| Cadastro parcial | Email muda após falha Account | User incompleto é observado e sua autoridade/limpeza não são presumidas. |
| Corrida de identidade | Fluxos concorrentes do mesmo subject | Constraints/resultado convergem; novo fluxo recupera falha concorrente. |
| State/browser | State alterado, cookie ausente/estranho e prazo vencido | Nega antes da token request, sem sessão/efeito novo. |
| Replay sequencial | Cookie original + state/code consumidos | Nenhuma nova sessão nem segunda submissão naquele fluxo. |
| Replay concorrente | Mesmo state/cookie/code em instâncias auth distintas | Uma token request por code quando esse é o gate; observar também User/Account/Session. |
| Novo state | Code já reservado/consumido com novo state válido | Reserva nega antes de egress; nova autorização/code continua funcionando. |
| PKCE | Dois fluxos nativos | Challenges distintos S256 e verifier correspondente no request real. |
| Injeção | Code de outro fluxo com state/cookie válidos da vítima | Mismatch PKCE negado; ausência de identidade/sessão adversária. |
| Downgrade | Code emitido sem challenge, com verifier no callback | Fake conforme nega; fake adversarial demonstra a premissa, não vulnerabilidade real. |
| DTO/paths | Sessão, listagem, token/account e paths não aprovados | Campos permitidos; ausência de segredos em JSON/HTML/redirect/logs. |

Não conclua que o provider real revoga tokens, rejeita downgrade ou aplica
expiração porque o fake o faz. Uma única sessão não satisfaz sozinha o gate de
não reutilização do cliente.

## Se adotar reserva de code

Teste a reserva pela rota/provider real, além de sua primitiva de storage:

- Instâncias auth independentes compartilhando D1, duplicata sequencial e code
  reapresentado com novo state. Uma instância só pode esconder mutex local.
- Falha D1 antes da reserva, resposta do binding inválida/ambígua e concorrência:
  nada chega ao token endpoint sem confirmação de vitória.
- Reserva confirmada e resposta do token endpoint perdida depois de aceite:
  repetição não reenvia; novo OAuth/code recupera. A fixture deve consumir o
  code antes de lançar a falha de rede.
- Falha de Account/Session após envio: marcador persiste e login novo recupera.
- Escopos distintos de issuer/client e configurações ambíguas de provider/plugin.
- Expiração pelo relógio definido, duplicata sem renovar prazo, limpeza limitada
  e manutenção retomada após ociosidade; nenhum marcador vivo é removido.
- Falha de INSERT faz rollback da limpeza se ambos estão no mesmo batch.
- Após retenção, code externo expirado é negado; prove também **code ainda não
  consumido**, para não confundir expiração com rejeição por uso anterior/PKCE.
- Sem credencial/PII no marcador, DTO ou observabilidade.

Use fault injection no D1 real, por exemplo trigger temporária com RAISE(ABORT),
para interromper uma escrita exata; remova a trigger entre cenários. Isso não é
um mecanismo de recuperação de produção. Falhas de formato da resposta do
binding são doubles explícitos no limite D1, não uma implementação fake do ORM.

## Qualidade do gate

Para correção, prove red/green pelo comportamento, não por erro de import/setup.
Não apague a reprodução anterior nem inverta seu significado. Uma baseline sem
plugin pode esperar duas requests para documentar o defeito; o teste da composição
aceita deve exigir uma. Identifique configurações no nome/arrange dos testes.

Asserts de viabilidade são comuns e entram nos comandos normais de teste/CI
quando a composição é aprovada. Diagnóstico verde isolado não é aceite.
Não use `it.fails`, skip, rebaixamento de threshold ou exclusão da implementação
para fabricar prontidão. Preserve a cobertura contratada pelo projeto; percentual
alto e contagem de testes não substituem os cenários críticos.

Aguarde todas as operações, restaure mocks/clock/global fetch e resete D1/cookies
entre casos. Use controle de falhas/respostas e condições observáveis em vez de
sleeps. Ao modificar um fake, confirme que a alteração fortalece o limite externo
e não o transforma em espelho do algoritmo do plugin.

## O que exige validação adicional

| Evidência | O que comprova | O que não comprova |
| --- | --- | --- |
| Types/build/dry-run | Compatibilidade estática e composição empacotável | Login, atomicidade ou comportamento do provider. |
| Workerd/D1 local | SQL/adapters/callbacks e corridas exercitados | Coordenação regional, crash físico, restore remoto ou entrega de scheduler. |
| Browser com backend local | Cookies, redirects, renovação e fluxo sob browser real | Configuração OAuth/PKCE do provider remoto. |
| Smoke remoto autorizado | Amostra da configuração real, scopes/PKCE/prazo/cookies | Todas as corridas/falhas ou prova universal de disponibilidade. |

Qualifique antes de promover, conforme o fluxo: PKCE mismatch/downgrade e prazo
no provider real; callback/origins registrados; renewal/logout/cache e CSRF;
rate-limit distribuído; refresh concorrente/rotação de secrets; migração/restore;
retomada e manutenção. São gates a definir/executar, não resultados herdados
da prova local. Não crie recursos ou execute ações remotas sem autorização.

## Registro compacto de evidência

Para cada garantia registre:

- contrato e política de identidade; versões e configuração efetiva;
- caminho no código da release e fonte primária com data;
- cenário e estado inicial, estímulo, falha injetada e oráculo independente;
- primeira falha esperada, correção pública suportada, comandos e resultado;
- fronteira exercitada e condições ainda não provadas;
- decisão: aprovado nesse escopo, bloqueado ou investigação pendente.

Não marque uma autenticação inteira como segura a partir de um caso ou métrica.
