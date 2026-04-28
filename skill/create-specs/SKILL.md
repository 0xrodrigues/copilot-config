---
name: create-specs
description: Lê o TASK.md e o CONTEXT.md de uma demanda e gera as especificações técnicas precisas para implementação. Usar após research-task (que produz o CONTEXT.md), ou quando o usuário menciona "gerar specs", "criar especificações", "especificações técnicas" ou "spec-NN".
---

# Criar Especificações Técnicas

Lê o `TASK.md` e o `CONTEXT.md` da demanda e produz as especificações técnicas — os documentos que guiam a implementação com precisão, rastreabilidade e sem retrabalho.

**Pré-requisito:** o `CONTEXT.md` deve ter sido gerado pela skill `research-task`. Sem ele, as specs serão imprecisas.

---

## Etapa 1 — Localizar os Documentos

Se o usuário informou caminhos, use-os diretamente.

Caso contrário, procure por `TASK.md` e `CONTEXT.md` no mesmo diretório. Caminhos comuns:
- `/specs/{tipo}/{nome}/TASK.md`
- `/specs/{tipo}/{nome}/CONTEXT.md`

Se `CONTEXT.md` não existir, **não prossiga**. Oriente o usuário a rodar `research-task` primeiro.

Leia ambos os arquivos completamente antes de continuar.

---

## Etapa 2 — Análise e Planejamento

Com `TASK.md` e `CONTEXT.md` em mãos, planeje as especificações.

### Divisão de specs

- Cada spec cobre uma única responsabilidade — uma camada, um módulo ou um contrato
- Specs sem dependências externas vêm primeiro
- Use a ordem natural do stack: consulte `docs/patterns/stack.md` e `docs/patterns/package-structure.md` quando disponíveis; caso contrário, use a ordem inferida do `CONTEXT.md`
- Para tamanho `small`: uma única spec com as seções 1–5 e 7 do template de feature

### Seções condicionais — ative quando a condição for verdadeira

| Condição identificada no CONTEXT.md | Seção adicional obrigatória |
|-------------------------------------|----------------------------|
| Endpoint público ou contrato Kafka será alterado | Rollback Plan |
| Novo consumer ou producer Kafka | Propagação MDC / headers |
| Manipula PII ou dados financeiros | Mascaramento e segurança |
| Mudança em query crítica ou schema de banco | Impacto em dados existentes |

### Apresente ao usuário

1. Lista de specs na ordem de implementação:
   - `spec-01-{titulo}.md` — [escopo de uma linha]
   - `spec-02-{titulo}.md` — [depende de spec-01; escopo de uma linha]
2. Seções condicionais ativadas e motivo
3. Campos sem informação suficiente: `⚠️ requer definição`

Verifique se os arquivos de spec já existem no diretório. Se sim, pergunte se deseja sobrescrever ou cancelar.

**Aguarde confirmação antes de gerar os arquivos.**

---

## Etapa 3 — Geração das Especificações

Após confirmação, crie cada spec em `/specs/{tipo}/{nome}/spec-NN-{titulo}.md` usando o template correspondente ao tipo da tarefa.

Preencha cada spec com informações concretas do `CONTEXT.md` — classes, métodos, contratos, padrões e riscos identificados. Não suponha o que pode ser verificado.

---

### Template: Feature

```md
---
title: [Título Conciso]
type: feature
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
---

> Em caso de dúvida sobre padrões, convenções ou decisões arquiteturais, consulte `docs/AGENTS.md`.

# Introdução

[O que esta spec entrega e qual responsabilidade ela cobre.]

## 1. Propósito & Escopo

**Em escopo**: [o que esta spec cobre]
**Fora de escopo**: [o que esta spec não cobre — seja explícito]

## 2. Requisitos & Restrições

- **REQ-001**: [requisito funcional]
- **CON-001**: [restrição técnica ou de negócio]
- **GUD-001**: [diretriz recomendada — não obrigatória]

## 3. Classes e Métodos

> Derivado do CONTEXT.md — use nomes reais do código.

**Criar:**
- `[Classe].[método](params): ReturnType` — [responsabilidade]

**Modificar:**
- `[Classe].[método]` — [comportamento atual] → [o que deve mudar]

## 4. Contratos de Dados & Interfaces

[Endpoints REST, eventos Kafka, schemas de tabelas envolvidos. Inclua assinaturas concretas.]

## 5. Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: Dado [contexto], Quando [ação], Então [resultado esperado]

## 6. Estratégia de Testes

[O que testar, em qual nível (unitário/integração) e cenários críticos. Consulte `docs/patterns/testing.md`.]

## 7. Pontos de Log

> Obrigatório em toda spec de feature. Consulte `docs/patterns/logging.md`.

| Camada | Evento | Nível | Campos obrigatórios |
|--------|--------|-------|---------------------|
| [camada] | [evento] | [nível] | [campos] |

## 8. Padrões Aplicáveis

[Arquivos de `docs/patterns/` que se aplicam diretamente a esta spec, pelo caminho relativo.]

Obrigatórios:
- `docs/patterns/logging.md`
- `docs/patterns/conventions/language.md`

## 9. Casos Extremos & Exemplos

\`\`\`plaintext
// Caso principal:
//   <exemplo de entrada e saída esperada>
//
// Caso de erro:
//   <entrada inválida e comportamento esperado>
\`\`\`

## [CONDICIONAL] Propagação MDC / Headers Kafka

> Incluir apenas se esta spec envolve novo consumer ou producer Kafka.

[Requisitos de propagação de contexto. Consulte `docs/patterns/logging.md`.]

## [CONDICIONAL] Dados Sensíveis & Mascaramento

> Incluir apenas se esta spec manipula PII ou dados financeiros.

- Campos sensíveis identificados: [lista]
- O que não deve aparecer em logs: [lista explícita]

## [CONDICIONAL] Rollback Plan

> Incluir apenas se esta spec altera endpoint público, contrato Kafka ou schema de tabela.

- **Trigger**: [quando acionar rollback]
- **Passos**: [sequência de ações para reverter]
- **Impacto do rollback**: [o que é afetado ao reverter]

## Especificações Relacionadas

- [spec-NN-{titulo}.md](./spec-NN-{titulo}.md) — [descrição do relacionamento]
```

---

### Template: Bug

```md
---
title: [Título Descrevendo o Bug]
type: bug
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
severity: [critical | high | medium | low]
---

> Em caso de dúvida sobre padrões, consulte `docs/AGENTS.md`.

# Introdução

[O que está acontecendo de errado e qual o impacto observado.]

## 1. Propósito & Escopo

**Em escopo**: [o que esta spec cobre para corrigir o bug]
**Fora de escopo**: [o que não será alterado]

## 2. Causa Raiz

- **Componente afetado**: [módulo, classe ou método — use nomes reais do CONTEXT.md]
- **Causa identificada**: [descrição técnica da causa]

## 3. Classes e Métodos

**Modificar:**
- `[Classe].[método]` — [o que está errado e o que deve mudar]

## 4. Comportamento Atual vs. Esperado

| Aspecto | Atual | Esperado |
|---------|-------|----------|
| [ex.: resposta da API] | [o que acontece] | [o que deveria acontecer] |

## 5. Passos de Reprodução

**Pré-condições**: [estado inicial necessário]

1. [Passo 1]
2. [Resultado observado]

## 6. Requisitos da Correção

- **REQ-001**: A correção deve [comportamento esperado]
- **CON-001**: A correção não deve alterar [contrato / interface existente]

## 7. Critérios de Aceitação

- **AC-001**: Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
- **AC-002**: Os cenários existentes de [funcionalidade relacionada] continuam funcionando

## 8. Estratégia de Testes

[Como garantir que o bug não volta. Consulte `docs/patterns/testing.md`.]

## 9. Pontos de Log

> Obrigatório. Consulte `docs/patterns/logging.md`.

| Camada | Evento | Nível | Campos obrigatórios |
|--------|--------|-------|---------------------|
| [camada] | [evento] | [nível] | [campos] |

## 10. Padrões Aplicáveis

Obrigatórios:
- `docs/patterns/logging.md`
- `docs/patterns/conventions/language.md`

## [CONDICIONAL] Rollback Plan

> Incluir apenas se a correção altera endpoint público, contrato Kafka ou schema.

- **Trigger**: [quando acionar rollback]
- **Passos**: [sequência de ações para reverter]
```

---

### Template: Refactor

```md
---
title: [Título Descrevendo o Refactor]
type: refactor
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
---

> Em caso de dúvida sobre padrões, consulte `docs/AGENTS.md`.

# Introdução

[O que será refatorado e qual o ganho esperado.]

## 1. Propósito & Escopo

**Em escopo**: [o que será refatorado]
**Fora de escopo**: [o que não será alterado — seja explícito]

## 2. Motivação Técnica

- **Problema atual**: [violação de padrão, acoplamento, duplicação — com referência ao arquivo/classe real]
- **Impacto do problema**: [dificuldade de testar, risco de bugs]

## 3. Classes e Métodos

**Modificar:**
- `[Classe].[método]` — [estado atual] → [estado alvo]

**Mover / Renomear:**
- `[Classe]` em `[arquivo atual]` → `[novo nome / local]`

## 4. Estado Atual vs. Estado Alvo

| Aspecto | Atual | Alvo |
|---------|-------|------|
| [ex.: estrutura de pacotes] | [como está] | [como ficará] |

## 5. Interfaces & Contratos Preservados

[Contratos, APIs e interfaces que não podem mudar. Use nomes reais do CONTEXT.md.]

## 6. Estratégia de Refactor

1. [Primeiro passo]
2. [Segundo passo]

**Riscos identificados**: [do CONTEXT.md — acoplamentos, dados em produção afetados]

## 7. Estratégia de Testes

[Como garantir que nenhum comportamento externo foi alterado. Consulte `docs/patterns/testing.md`.]

## 8. Critérios de Aceitação

- **AC-001**: Todos os testes existentes passam sem modificação de comportamento
- **AC-002**: [critério técnico de melhoria]
- **AC-003**: A interface pública de [componente] permanece compatível

## 9. Padrões Aplicáveis

[Arquivos de `docs/patterns/` que motivam ou guiam este refactor.]
```

---

## Etapa 4 — Atualizar o TASK.md

Após criar todas as specs, adicione a seção abaixo ao `TASK.md` existente e atualize o campo `last_updated`:

```md
## Especificações Técnicas

> Atualize o status à medida que implementar cada spec.

| # | Arquivo | Escopo | Status |
|---|---------|--------|--------|
| 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending |
| 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending |
```

Informe ao usuário: lista de todos os caminhos criados, campos marcados com `⚠️ requer definição` e próximos passos para iniciar a implementação.

---

## Boas Práticas

- Use nomes reais de classes, métodos e arquivos do `CONTEXT.md` — nunca genéricos.
- Cada spec deve ser autossuficiente. O agente que implementa não precisa ler outras specs.
- Campos incertos nunca devem ser preenchidos com suposições: use `⚠️ requer definição`.
- Seções condicionais só aparecem se a condição foi ativada na Etapa 2.
- Para tamanho `small`, inclua apenas as seções 1–6 e 8 do template de feature.
