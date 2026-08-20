# Java Test Quality

**[Instalar com skills.sh](https://skills.sh/thiagocrepequer/java-test-quality/java-test-quality)**

```bash
npx skills add ThiagoCrepequer/java-test-quality
```

Skill para agentes de código projetarem, escreverem e revisarem testes confiáveis em Java e Quarkus.

## O que ela faz

Orienta o agente a transformar regras de negócio e contratos técnicos em evidências executáveis. A skill cobre JUnit, componentes CDI, HTTP, persistência real, segurança, concorrência, isolamento de estado, JaCoCo e PIT Mutation Testing.

Ela ajuda a evitar testes que passam sem provar o comportamento, como asserções apenas de presença, mocks que removem o risco testado, fixtures sem dados concorrentes e métricas de cobertura usadas como sinônimo de qualidade.

## Filosofia

Um bom teste deve ser simultaneamente:

- **sensível a bugs:** uma regressão plausível quebra o teste;
- **tolerante a refatorações:** mudanças internas que preservam o contrato mantêm o teste passando;
- **fiel ao risco:** banco, HTTP, CDI ou segurança participam quando fazem parte do comportamento;
- **isolado e determinístico:** nenhum cenário depende de ordem, estado compartilhado, sleeps ou relógio real;
- **honesto:** relata exatamente o que foi validado e o que permanece fora do escopo.

Mutation score e cobertura são sinais de diagnóstico. A confiança vem de um contrato claro, fixtures discriminantes e um oráculo independente capaz de rejeitar resultados incorretos.

## Conteúdo

O entrypoint está em [`java-test-quality/SKILL.md`](java-test-quality/SKILL.md). As referências aprofundam princípios de bons testes, camadas do Quarkus, asserções, fixtures, test doubles, riscos concorrentes e análise de mutações.

## Uso

```text
$java-test-quality Escreva testes de regressão para este serviço Quarkus e prove o isolamento entre tenants.
```

Licenciado sob [MIT](LICENSE).
