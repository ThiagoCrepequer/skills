# Runtime, persistência e recuperação

## Evidência de referência

Verificada em **2026-10-05**, com Better Auth/core/Drizzle adapter **1.7.7**,
Drizzle ORM **0.45.3**, Workerd e D1 locais. Não generalizar para outra release,
plugin, provider, schema ou configuração de storage.

| Configuração exercitada | Observação local |
| --- | --- |
| Drizzle SQLite/D1, `transaction: false` | Login nativo funciona; User e Account são INSERTs separados. |
| Falha no INSERT Account após User | User persiste; Account/Session não são concluídas. |
| Linking implícito desabilitado | Novo OAuth com mesmo email encontra User parcial e pode terminar `account_not_linked`. |
| Linking nativo, email recebido e local verificados, `trustedProviders: []` | Novo OAuth com code novo recupera Account e autentica mantendo o User original. |
| Email recebido ou local não verificado | O vínculo implícito é negado nessa configuração. |
| Email muda antes de criar Account | Pode ser criado outro User; o anterior continua incompleto. |
| Adapter D1 nativo da mesma release | Conserva as limitações relevantes de cadastro parcial e parser OAuth. |
| `consumeOne`/consumo de verification diretamente | Consumo atômico dá um vencedor; isso não significa que o parser OAuth o usa. |

A recuperação por linking não faz rollback nem torna User/Account atômicos.
Desabilitar linking pode ser uma escolha correta de identidade; nesse caso a
estratégia de recuperação precisa satisfazer o contrato do projeto por outro
caminho suportado. Não habilite linking silenciosamente para contornar o erro.

## Inventário mínimo

Confirme versões **resolvidas**, e não apenas ranges de peers:

- Better Auth, core, adapters, Drizzle ORM/Kit e plugins habilitados;
- pacote/import realmente usado e eventuais duplicatas de core/adapter;
- Worker bindings, compatibility date/flags e configuração do runner de testes;
- schema gerado, nomes/mapeamentos de tabelas/campos, relações e migrations;
- armazenamento de verification/state, sessão, cookie cache e secondary storage;
- origem/base URL por ambiente, client/provider e callbacks registrados.

Use os comandos do projeto; não assuma que exemplos com `@latest` identificam
uma composição reproduzível. Gere schema com CLI compatível, revise seu SQL e
aplique somente no ambiente autorizado. Não rode migrations por request de login.

## Transação não é batch

No driver Drizzle D1 0.45.3, `transaction()` emite BEGIN/COMMIT/ROLLBACK; a presença
do método e seu tipo não demonstram que o binding suporta transação interativa.
O adapter Drizzle qualificado manteve `transaction: false`.

`D1.batch()` executa statements em uma transação; falha de statement aborta a
sequência. Isso não agrupa automaticamente duas operações sequenciais feitas
por Better Auth. Trace o callback e observe as escritas reais antes de prometer
atomicidade. O anúncio de suporte D1/batch da biblioteca não prova batch em
cada fluxo de signup ou plugin.

Também não confunda zero rows com erro: uma guarda que afeta zero rows pode
ser seguida por outro INSERT no mesmo batch. Escritas dependentes devem carregar
a condição de autorização/estado ou um mecanismo suportado que impeça o efeito.
Teste a ausência de linhas/efeitos quando a guarda falha.

O D1 nativo foi adicionado ao Better Auth antes da release qualificada. A
[documentação de outros bancos](https://better-auth.com/docs/adapters/other-relational-databases)
ainda lista `kysely-d1` como dialect comunitário; isso não invalida o caminho
nativo documentado no anúncio da versão 1.5. Qualifique o caminho instalado.
Trocar Drizzle por Kysely/D1 não corrige uma sequência find/delete no core OAuth.

## Schema e constraints

Compare o schema ORM com a tabela física D1: colunas, tipos/unidades de timestamps,
nullability, defaults, relações, índices e FKs. Um SQLite Node em memória não
substitui essa prova. Verifique a tradução de booleans/datas pelo adapter.

A identidade externa deve usar o subject estável no namespace correto do issuer,
não email ou username. Quando o contrato exige um único vínculo por identidade,
prove a unicidade física correspondente, incluindo namespaces de múltiplos
issuers/clientes se aplicável. Não imponha uma chave simples sem qualificar a
semântica do provider/plugin. Duplicata concorrente precisa ter resultado seguro
e recuperação observável; uma constraint lançando erro não prova UX concluída.

D1/Drizzle suportados no core não significam suporte para todo plugin. Verifique
requisitos de transações, schema e runtime de cada plugin; a documentação SCIM
consultada informa incompatibilidade com D1 por exigir transações interativas.
Não habilite SAML/SCIM, organization ou providers extras por disponibilidade.

## Política de linking e efeitos de cadastro

Na configuração qualificada, linking nativo exigiu email recebido verificado e
User local verificado. `trustedProviders` pode dispensar a verificação recebida;
`requireLocalEmailVerified` e outros flags podem alterar a proteção. Registre o
valor efetivo, não apenas a intenção. Providers diferentes precisam de garantias
próprias e reautenticação quando o produto a exigir.

Email verificado é controle atual de um endereço transferível/reutilizável.
Debata risco de reaproveitamento, comprometimento e providers administrados antes
de aprovar associação implícita. Não trate o vínculo como prova histórica de que
subjects diferentes sempre pertencem à mesma pessoa.

Login nativo com email não verificado pode ser permitido mesmo quando linking
não verificado é negado. São políticas distintas. Operações que exigem ownership
atual do endereço não podem usar somente um `emailVerified` antigo.

Evite conceder recursos, permissões ou efeitos externos apenas a partir de
`user.create`: o callback pode falhar antes de Account/Session. Quando tais efeitos
existirem, escolha uma fronteira de login/vínculo concluído e prove idempotência.
Defina a política de User incompleto, reconciliação e limpeza sem apagar uma
identidade válida ou transferir permissões por coincidência de email.

## Segurança e performance têm provas próprias

Meça número de queries/round trips e planos/índices relevantes na composição
real. Joins só ajudam quando adapter e relações corretas os suportam; uma
alegação de ganho na documentação não é benchmark da aplicação. A redução de
latência não autoriza cache de autorização sem prazo ou leitura obsoleta de
revogação. Qualifique consistência se usar D1 Sessions/read replication.

Não adicione KV, outro banco, replica ou uma camada de cache por tentativa de
resolver algo cujo call graph continua incorreto. Mudanças de persistência devem
preservar as provas de identidade, recuperação, concorrência e migração.
