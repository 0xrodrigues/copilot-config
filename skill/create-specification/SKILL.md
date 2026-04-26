---
name: create-specification
description: 'Gera o TASK.md e as specs necessárias para implementação assistida por IA a partir de um ticket do Jira (via MCP) ou de uma descrição inline. Busca automaticamente páginas do Confluence linkadas no ticket. Compatível com os tipos feature, bug e refactor.'
---

# Criar Especificações

Seu objetivo é gerar o `TASK.md` e todas as specs necessárias para que a IA implemente o trabalho com controle de contexto, rastreabilidade e sem retrabalho.

O `TASK.md` é o artefato central. Ele deve ser legível por qualquer pessoa do time — dev, QA, PO — sem exigir conhecimento técnico de implementação. É o histórico do que foi feito, o contexto para sessões futuras da IA, e a fonte para atualizações no Jira e documentações no Confluence.

Esta skill opera em dois modos:
- **Modo Jira**: recebe um link ou ID de ticket, busca dados do Jira e páginas do Confluence linkadas via MCP.
- **Modo Express**: coleta informações mínimas diretamente na conversa.

---

## Contexto do Projeto

Antes de qualquer outra ação, leia dois arquivos na raiz do projeto:

**1. `docs/AGENTS.md`** — se existir, é o índice de todos os padrões do time. Não leia todos os arquivos que ele referencia — use-o como mapa. Navegue até os documentos específicos em `docs/patterns/` apenas quando precisar de informação concreta sobre aquele padrão.

**2. `TASK.md` na raiz** — se existir, leia para entender o contexto de trabalho em andamento antes de criar novos arquivos.

Se nenhum dos arquivos existir, prossiga normalmente.

---

## Etapa 1 — Identificação do Modo

Analise o argumento recebido:

- **URL ou ID do Jira** (ex.: `https://empresa.atlassian.net/browse/PROJ-123` ou `PROJ-123`) → **Modo Jira**
- **Vazio ou descrição inline** → **Modo Express**

---

### Modo Jira

**Passo 1 — Buscar dados do ticket:**

Use o MCP do Jira para buscar:
- Summary (título)
- Descrição completa
- Tipo (Story, Task, Bug, Sub-task)
- Critérios de Aceitação
- Componentes / Labels
- Links (PRs, páginas do Confluence, tickets relacionados)

**Passo 2 — Buscar páginas do Confluence linkadas:**

Identifique todos os links do Confluence nos campos do ticket. Para cada link encontrado, use o MCP do Confluence para buscar o conteúdo completo da página. Essas páginas geralmente contêm o refinamento técnico, regras de negócio e contexto que não estão no ticket.

Se não houver links do Confluence, prossiga apenas com os dados do Jira.

**Passo 3 — Mapeamento de tipo:**

| Tipo no Jira | Tipo de spec |
|---|---|
| Story, Task, Sub-task | `feature` |
| Bug | `bug` |
| Refactor (label ou convenção) | `refactor` |

Se o tipo não for mapeável, pergunte antes de continuar.

**Passo 4 — Avaliação de completude:**

Com todos os dados coletados (Jira + Confluence), avalie em três dimensões:

**1. Clareza do problema**
Você consegue descrever com precisão o que deve ser construído ou corrigido? A descrição está clara ou está vaga, contraditória ou incompleta?

**2. Critérios de aceitação**
Existem critérios explícitos ou deriváveis da descrição? São testáveis no formato Dado/Quando/Então?

**3. Regras de negócio e restrições**
Há regras de negócio, restrições técnicas ou dependências que impactam a implementação e estão documentadas?

**Se qualquer dimensão estiver insuficiente**, não prossiga. Agrupe todas as dúvidas em um único bloco:

> "O ticket [ID] tem lacunas que podem gerar retrabalho. Preciso entender:
> - [pergunta 1]
> - [pergunta 2]"

Só avance quando tiver clareza suficiente.

---

### Modo Express

**Rodada 1:**
- Qual o tipo de trabalho? (`feature`, `bug` ou `refactor`)
- Descreva brevemente o que precisa ser feito.

**Rodada 2** (após receber as respostas):

*Feature:*
- Quais são os requisitos funcionais principais?
- Existem regras de negócio explícitas?
- Existem restrições técnicas?
- Como validar que está correto? (critérios de aceitação no formato Dado/Quando/Então)

*Bug:*
- Qual o comportamento atual e o esperado?
- Qual componente ou camada está envolvido?
- O que a correção não pode quebrar?
- Como provar que o bug foi corrigido?

*Refactor:*
- O que está problemático e qual o estado alvo?
- Quais interfaces ou contratos não podem mudar?
- Como provar que o comportamento externo não mudou?

---

## Etapa 2 — Análise, Tamanho e Planejamento

### Classificação de tamanho

| Tamanho | Critério |
|---------|----------|
| **Small** | 1 responsabilidade, sem integração nova, sem impacto em contrato público |
| **Medium** | 2–3 responsabilidades, ou integração existente, ou impacto em produção |
| **Large** | 4+ responsabilidades, ou nova integração, ou mudança de contrato público |

### Seções condicionais nas specs

Verifique se o trabalho se enquadra em algum dos casos abaixo e ative as seções correspondentes nas specs:

| Condição | Seção adicional obrigatória na spec |
|----------|-------------------------------------|
| Impacta endpoint público ou contrato Kafka | Rollback Plan |
| Novo consumer ou producer Kafka | Propagação MDC / headers |
| Envolve dados sensíveis (PII, financeiro) | Mascaramento e segurança |
| Mudança em query crítica ou schema | Impacto em dados existentes |

### Padrões obrigatórios em toda spec de feature ou bug

Independente do contexto, toda spec de `feature` ou `bug` deve incluir na seção **Padrões Aplicáveis**:

- `docs/patterns/logging.md` — pontos obrigatórios de log por camada
- `docs/patterns/conventions/language.md` — idioma de logs e mensagens

E deve incluir a seção **Pontos de Log** declarando explicitamente, por camada coberta pela spec, quais eventos serão logados, em qual nível e com quais campos. O agente não deve decidir isso em runtime.

### Ordem de specs

Ao dividir responsabilidades, use a sequência natural do stack do time como referência — consulte `docs/patterns/stack.md` e `docs/patterns/package-structure.md` para entender as camadas e a ordem correta. Specs sem dependências externas vêm primeiro. A numeração reflete a ordem obrigatória de implementação.

### Apresentação do plano

Apresente ao usuário:

1. Tipo de trabalho identificado
2. Tamanho classificado (Small / Medium / Large) e justificativa
3. Caminho base: `/specs/{tipo}/{nome}/`
4. Lista de specs na ordem de implementação:
   - `spec-01-{titulo}.md` — [escopo de uma linha]
   - `spec-02-{titulo}.md` — [depende de spec-01; escopo de uma linha]
5. Seções condicionais ativadas e motivo
6. **Se Modo Jira:** sub-tarefas que serão criadas no ticket `[ID]`

Verifique se `/specs/{tipo}/{nome}/` já existe. Se sim, pergunte se deseja sobrescrever, usar nome diferente ou cancelar. Aguarde antes de continuar.

**Aguarde confirmação do usuário antes de prosseguir para a Etapa 3.**

---

## Etapa 3 — Geração dos Arquivos

Após confirmação, siga esta ordem:

**Se Modo Jira:** crie as sub-tarefas no ticket pai antes de criar qualquer arquivo:
- Título: `[spec-NN] Título da especificação`
- Tipo: Sub-task
- Status: To Do
- Descrição: escopo de uma linha da spec

Guarde os IDs retornados — eles serão usados no `TASK.md`.

Crie os arquivos na seguinte estrutura:

```
/specs/
  features/{feature-name}/
    TASK.md
    spec-01-{titulo}.md
    spec-02-{titulo}.md
  bugs/{bug-name}/
    TASK.md
    spec-01-{titulo}.md
  refactor/{refactor-name}/
    TASK.md
    spec-01-{titulo}.md
```

Após criar todos os arquivos, informe ao usuário: lista de caminhos gerados, IDs das sub-tarefas criadas no Jira (se Modo Jira) e campos com gap pendente marcados com `⚠️ requer definição`.

---

## TASK.md — Documento Central

O `TASK.md` é o primeiro arquivo que qualquer agente ou desenvolvedor deve ler. É legível por qualquer pessoa do time. Não contém código nem detalhes de implementação — esses ficam nas specs. É o registro funcional completo do trabalho: o que foi pedido, o que foi feito, as decisões tomadas.

Durante a implementação, **apenas o `TASK.md` é atualizado** — nenhuma chamada ao Jira ou Confluence é feita. Ao final, uma skill dedicada lê este arquivo e executa todas as atualizações externas usando os links aqui registrados.

```md
---
type: [feature | bug | refactor]
size: [small | medium | large]
name: [nome do trabalho em kebab-case]
jira_ticket: [ID do ticket pai — omitir se Modo Express]
jira_ticket_url: [URL completa do ticket pai — omitir se Modo Express]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
status: [planned | in-progress | completed]
---

# [Nome do Trabalho]

[Resumo de duas a quatro frases em linguagem clara: o que este trabalho entrega ou corrige, qual problema resolve e qual o valor para o negócio ou sistema.]

## Contexto

[Contexto necessário para entender o trabalho: de onde veio o requisito, qual o cenário atual, por que isso precisa ser feito. Sem jargões técnicos desnecessários.]

## Escopo

**O que será feito:**
- [item 1]
- [item 2]

**O que não será feito:**
- [item — e por quê, se relevante]

## Requisitos Funcionais

- **RF-001**: [o sistema deve...]
- **RF-002**: [o sistema deve...]

## Regras de Negócio

- **RN-001**: [regra de negócio explícita]
- **RN-002**: [regra de negócio explícita]

> Se não houver regras de negócio identificadas, remova esta seção.

## Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: Dado [contexto], Quando [ação], Então [resultado esperado]

## Especificações Técnicas

> Esta tabela controla o progresso da implementação. Atualize **imediatamente** ao iniciar e ao concluir cada spec. Nenhuma chamada externa é feita durante a implementação — o Jira será atualizado ao final pela skill de update.

| # | Arquivo | Escopo | Status | Jira ID | Jira URL |
|---|---------|--------|--------|---------|----------|
| 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending | [ID sub-tarefa — omitir colunas Jira se Modo Express] | [URL completa] |
| 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending | [ID sub-tarefa] | [URL completa] |

## Protocolo de Execução

> **Leia antes de escrever qualquer código.**

**Ciclo por spec:**

1. Leia a spec completamente.
2. Atualize o status da spec neste arquivo para `in-progress` e atualize `last_updated`.
3. Implemente o que a spec descreve — nada além do que a spec descreve.
4. Valide todos os Critérios de Aceitação da spec.
5. Escreva o resumo na seção **Resumos de Implementação** deste arquivo, em linguagem clara, sem código.
6. Atualize o status da spec para `done` e atualize `last_updated`.
7. Somente então avance para a próxima spec.

> Nenhuma chamada ao Jira ou Confluence é feita durante a implementação. O `TASK.md` é a única fonte de verdade durante o desenvolvimento. As atualizações externas acontecem ao final, via skill dedicada.

**Regra de decisão em runtime:**

Se durante a implementação surgir uma situação não coberta pela spec — ambiguidade, decisão técnica não documentada, comportamento inesperado de dependência — **PARE imediatamente**. Não tome a decisão sozinho. Registre o que encontrou em **Decisões e Desvios** e aguarde instrução explícita antes de continuar.

Situações que exigem parada obrigatória:
- A spec descreve um comportamento que conflita com o código existente
- A spec não cobre um caso de erro identificado durante a implementação
- Implementar a spec exige alterar algo fora do escopo declarado
- Há dúvida sobre qual padrão em `docs/patterns/` se aplica

**Conclusão da implementação:**

Quando todas as specs estiverem `done`:
1. Atualize o status deste arquivo para `completed` e preencha **Resultado Final**.
2. Informe ao usuário que a implementação foi concluída e que o `TASK.md` está pronto para ser processado pela skill de update do Jira e Confluence.

## Resumos de Implementação

> Preenchido incrementalmente. Escreva o resumo de cada spec **imediatamente** após concluí-la, antes de avançar para a próxima. Em linguagem clara — sem código, sem jargões desnecessários.

### spec-01-{titulo}.md

`status: pending`

<!-- Ao concluir: descreva o que foi implementado, o comportamento entregue e qualquer decisão relevante tomada. -->

### spec-02-{titulo}.md

`status: pending`

<!-- Ao concluir: descreva o que foi implementado, o comportamento entregue e qualquer decisão relevante tomada. -->

## Decisões e Desvios

> Registre aqui toda decisão tomada em runtime que não estava prevista na spec original, com data e justificativa. Se a decisão ainda não foi tomada, registre como pendente e aguarde instrução.

- **[YYYY-MM-DD]**: [decisão ou desvio] — [motivo]

## Resultado Final

> Preenchido após todas as specs estarem `done`. Fonte para a skill de update — usada para atualizar o Jira e criar documentação no Confluence.

**O que foi entregue**: [síntese funcional do que foi construído ou corrigido]
**Fora do escopo**: [o que ficou de fora e por quê]
**Débitos técnicos**: [limitações conhecidas, melhorias postergadas, riscos identificados]
```

---

## Templates de Especificação

As specs contêm os detalhes técnicos de implementação. São lidas pelo agente que implementa, não pelo time como um todo. Ao preencher seções de padrões e estratégia de testes, navegue pelos arquivos em `docs/patterns/` via `docs/AGENTS.md` — não suponha os padrões do time.

### Template: Feature

> Para tamanho Small, inclua apenas as seções 1 a 5 e 7. Seções condicionais se aplicam normalmente. Ao incluir uma seção condicional, remova o marcador `[CONDICIONAL]` do título.

~~~md
---
title: [Título Conciso]
type: feature
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
jira_subtask: [ID da sub-tarefa — omitir se Modo Express]
---

> Em caso de dúvida sobre padrões, convenções ou decisões arquiteturais durante a implementação, consulte `docs/AGENTS.md`.

# Introdução

[O que esta spec entrega e qual responsabilidade ela cobre.]

## 1. Propósito & Escopo

**Em escopo**: [o que esta spec cobre]
**Fora de escopo**: [o que esta spec não cobre — seja explícito]

## 2. Requisitos & Restrições

- **REQ-001**: [requisito funcional]
- **CON-001**: [restrição técnica ou de negócio]
- **GUD-001**: [diretriz recomendada — não obrigatória]

## 3. Contratos de Dados & Interfaces

[Interfaces, endpoints REST, eventos Kafka, schemas de tabelas envolvidos. Consulte `docs/patterns/layers/controller.md` para os padrões de contrato do time.]

## 4. Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: Dado [contexto], Quando [ação], Então [resultado esperado]

## 5. Estratégia de Testes

[O que testar, em qual nível (unitário/integração) e cenários críticos. Consulte `docs/patterns/testing.md` para os frameworks e padrões de teste do time.]

## 6. Pontos de Log

> Seção obrigatória em toda spec de feature ou bug. Declare explicitamente os pontos de log por camada coberta por esta spec. Consulte `docs/patterns/logging.md` para níveis e campos obrigatórios.

| Camada | Evento | Nível | Campos obrigatórios |
|--------|--------|-------|---------------------|
| Controller | [ex.: requisição recebida] | INFO | [ex.: nuCustomer, operationType] |
| Service | [ex.: desvio de negócio identificado] | WARN | [ex.: motivo, nuCustomer] |
| Repository | [ex.: consulta executada] | INFO | [ex.: parâmetros da query] |

## 7. Padrões Aplicáveis

[Liste os arquivos de `docs/patterns/` que se aplicam diretamente a esta spec, pelo caminho relativo. Não resuma o conteúdo — apenas referencie para que o agente saiba quais abrir durante a implementação.]

Obrigatórios em toda spec de feature ou bug:
- `docs/patterns/logging.md`
- `docs/patterns/conventions/language.md`

## 7. Casos Extremos & Exemplos

```plaintext
// Caso principal:
//   <exemplo de entrada e saída esperada>
//
// Caso de erro:
//   <entrada inválida e comportamento esperado>
```

## [CONDICIONAL] 8. Propagação MDC / Headers Kafka

> Incluir apenas se esta spec envolve novo consumer ou producer Kafka.

[Descreva os requisitos de propagação de contexto. Consulte `docs/patterns/logging.md` para os interceptors e campos MDC obrigatórios.]

## [CONDICIONAL] 9. Dados Sensíveis & Mascaramento

> Incluir apenas se esta spec manipula PII ou dados financeiros.

- Campos sensíveis identificados: [lista]
- O que não deve aparecer em logs: [lista explícita]

[Consulte `docs/patterns/logging.md` para a estratégia de mascaramento configurada no time.]

## [CONDICIONAL] 10. Rollback Plan

> Incluir apenas se esta spec altera endpoint público, contrato Kafka ou schema de tabela.

- **Trigger**: [quando acionar rollback]
- **Passos**: [sequência de ações para reverter]
- **Impacto do rollback**: [o que é afetado ao reverter]

## 11. Especificações Relacionadas

- [spec-NN-{titulo}.md](./spec-NN-{titulo}.md) — [descrição do relacionamento]
~~~

---

### Template: Bug

~~~md
---
title: [Título Descrevendo o Bug]
type: bug
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
jira_subtask: [ID da sub-tarefa — omitir se Modo Express]
severity: [critical | high | medium | low]
---

> Em caso de dúvida sobre padrões, convenções ou decisões arquiteturais durante a implementação, consulte `docs/AGENTS.md`.

# Introdução

[O que está acontecendo de errado e qual o impacto observado.]

## 1. Propósito & Escopo

**Em escopo**: [o que esta spec cobre para corrigir o bug]
**Fora de escopo**: [o que não será alterado nesta correção]

## 2. Causa Raiz

- **Componente afetado**: [módulo, serviço ou camada]
- **Causa identificada**: [descrição técnica da causa]
- **Hipótese alternativa**: [se houver]

## 3. Comportamento Atual vs. Esperado

| Aspecto | Atual | Esperado |
|---------|-------|----------|
| [ex.: resposta da API] | [o que acontece] | [o que deveria acontecer] |

## 4. Passos de Reprodução

**Pré-condições**: [estado inicial necessário]

1. [Passo 1]
2. [Passo 2]
3. [Resultado observado]

**Ambiente**: [produção / staging / local]

## 5. Requisitos da Correção

- **REQ-001**: A correção deve [comportamento esperado]
- **CON-001**: A correção não deve alterar [contrato / interface existente]

## 6. Critérios de Aceitação

- **AC-001**: Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
- **AC-002**: Os cenários existentes de [funcionalidade relacionada] continuam funcionando

## 7. Estratégia de Testes

[Como garantir que o bug não volta. Consulte `docs/patterns/testing.md` para frameworks e padrões de teste do time.]

## 8. Pontos de Log

> Seção obrigatória. Declare os pontos de log impactados ou adicionados por esta correção. Consulte `docs/patterns/logging.md`.

| Camada | Evento | Nível | Campos obrigatórios |
|--------|--------|-------|---------------------|
| [camada] | [evento] | [nível] | [campos] |

## 9. Padrões Aplicáveis

[Liste os arquivos de `docs/patterns/` relevantes para esta correção, pelo caminho relativo.]

Obrigatórios em toda spec de feature ou bug:
- `docs/patterns/logging.md`
- `docs/patterns/conventions/language.md`

## 9. Casos Extremos & Dados de Teste

```plaintext
// Entrada que causa o bug:
//   <exemplo>
//
// Entrada que deve funcionar corretamente após a correção:
//   <exemplo>
```

## [CONDICIONAL] 10. Rollback Plan

> Incluir apenas se a correção altera endpoint público, contrato Kafka ou schema de tabela.

- **Trigger**: [quando acionar rollback]
- **Passos**: [sequência de ações para reverter]

## 11. Especificações Relacionadas

- [spec-NN-{titulo}.md](./spec-NN-{titulo}.md) — [descrição]
~~~

---

### Template: Refactor

~~~md
---
title: [Título Descrevendo o Refactor]
type: refactor
size: [small | medium | large]
implementation_order: [número sequencial]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
jira_subtask: [ID da sub-tarefa — omitir se Modo Express]
---

> Em caso de dúvida sobre padrões, convenções ou decisões arquiteturais durante a implementação, consulte `docs/AGENTS.md`.

# Introdução

[O que será refatorado e qual o ganho esperado.]

## 1. Propósito & Escopo

**Em escopo**: [o que será refatorado]
**Fora de escopo**: [o que não será alterado — seja explícito]

## 2. Motivação Técnica

- **Problema atual**: [ex.: violação de padrão em `docs/patterns/`, acoplamento, duplicação]
- **Impacto do problema**: [ex.: dificuldade de testar, risco de bugs]

## 3. Estado Atual vs. Estado Alvo

| Aspecto | Atual | Alvo |
|---------|-------|------|
| [ex.: estrutura de pacotes] | [como está] | [como ficará] |

## 4. Requisitos & Restrições

- **REQ-001**: O comportamento externo observável deve permanecer idêntico
- **CON-001**: A interface pública de [componente] não deve ser alterada

## 5. Interfaces & Contratos Preservados

[Contratos, APIs e interfaces que não podem mudar.]

## 6. Estratégia de Refactor

1. [Primeiro passo]
2. [Segundo passo]
3. [Terceiro passo]

**Riscos identificados**: [pontos de acoplamento, dados em produção afetados]

## 7. Padrões Aplicáveis

[Liste os arquivos de `docs/patterns/` que motivam ou guiam este refactor, pelo caminho relativo.]

## 8. Estratégia de Testes

[Como garantir que nenhum comportamento externo foi alterado. Consulte `docs/patterns/testing.md` para frameworks e padrões de teste do time.]

## 9. Critérios de Aceitação

- **AC-001**: Todos os testes existentes passam sem modificação de comportamento
- **AC-002**: [critério técnico de melhoria]
- **AC-003**: A interface pública de [componente] permanece compatível com todos os consumidores

## 10. Exemplos

```plaintext
// Antes:
//   <trecho representativo do estado atual>
//
// Depois:
//   <trecho representativo do estado alvo>
```

## 11. Especificações Relacionadas

- [spec-NN-{titulo}.md](./spec-NN-{titulo}.md) — [descrição]
~~~

---

## Boas Práticas

- Preencha `date_created` e `last_updated` com a data atual no formato `YYYY-MM-DD`.
- O `TASK.md` usa linguagem clara e funcional — sem código, sem jargões desnecessários. As specs contêm os detalhes técnicos.
- Cada spec cobre uma única responsabilidade. Se o escopo exigir mais de três grupos de requisitos não relacionados, divida em specs separadas.
- Cada spec deve ser autossuficiente — o agente não deve precisar ler outras specs para entender o que implementar.
- Nunca gere specs com base em informações insuficientes. Agrupe todas as dúvidas em um único bloco e aguarde resposta.
- Campos marcados com `⚠️ requer definição` nunca devem ser preenchidos com suposições — sinalize e aguarde.
- Seções condicionais só aparecem na spec se a condição foi ativada na Etapa 2. Ao incluir uma seção condicional, remova o marcador `[CONDICIONAL]` do título.
- Para tamanho Small, inclua apenas as seções 1 a 5 e 7 do template de Feature.
- Consulte sempre `docs/AGENTS.md` como ponto de entrada — navegue até os arquivos específicos de `docs/patterns/` conforme necessário. Não suponha os padrões do time.