---
name: create-specification
description: 'Gera especificações prontas para IA a partir de um ticket do Jira (via MCP), de um Design Document ou de uma descrição inline (modo express). Aceita link/ID do Jira, caminho de arquivo ou descrição direta na conversa. Compatível com os tipos feature, bug e refactor.'
---

# Criar Especificações

Seu objetivo é gerar todas as especificações necessárias para que a IA possa implementar o trabalho com máximo controle de contexto e rastreabilidade.

Esta skill opera em três modos:

- **Modo Jira**: recebe um link ou ID de ticket do Jira, busca os dados via MCP e gera as specs a partir deles.
- **Modo arquivo**: recebe o caminho de um Design Document existente e gera as specs a partir dele.
- **Modo express**: quando nenhuma das entradas acima é fornecida, coleta as informações mínimas diretamente na conversa.

---

## Contexto do Projeto

Antes de qualquer outra ação, tente ler o arquivo `ENGINEERING_STANDARDS.md` na raiz do projeto.

- Se existir, extraia a stack tecnológica (linguagem, frameworks, banco de dados, mensageria) e os padrões arquiteturais relevantes.
- Use essas informações para preencher as seções de estratégia de testes e dependências nas specs geradas.
- Se não existir, prossiga normalmente — deixe as seções dependentes de stack com `[definir conforme stack do projeto]`.

---

## Etapa 1 — Identificação do Modo

Analise o argumento recebido e o contexto da conversa:

- Se contiver uma **URL do Jira** (ex.: `https://empresa.atlassian.net/browse/PROJ-123`) ou um **ID de ticket** no formato `PROJ-123`, ative o **Modo Jira**.
- Se contiver um **caminho de arquivo** válido (ex.: `docs/design-docs/features/minha-feature.md`), ative o **Modo Arquivo**.
- Se estiver vazio, em branco, ou o usuário informar que não tem ticket nem arquivo (ex.: "não tenho", "express", "-"), ative o **Modo Express**.

### Modo Jira

Extraia o ID do ticket do argumento recebido — o argumento pode vir em qualquer um destes formatos:

- **URL completa:** `https://empresa.atlassian.net/browse/PROJ-123` → extrair o ID `PROJ-123`
- **ID direto:** `PROJ-123` → usar diretamente
- **Linguagem natural com link:** `"crie specs para essa tarefa https://..."` → extrair o ID da URL

Use o MCP do Confluence/Jira para buscar os dados do ticket:

- **Summary** (título da história ou tarefa)
- **Descrição** (contexto, motivação, detalhes do refinamento)
- **Tipo** (Story, Task, Bug, Sub-task)
- **Critérios de Aceitação** (campo dedicado ou itens listados na descrição)
- **Componentes / Labels** (se disponíveis)
- **Links** (PRs, páginas do Confluence, dependências)

**Mapeamento de tipo para spec:**

| Tipo no Jira | Tipo de spec |
|---|---|
| Story, Task, Sub-task | `feature` |
| Bug | `bug` |
| Refactor (label ou convenção) | `refactor` |

Se o tipo não for mapeável claramente, pergunte ao usuário antes de continuar.

**Entendimento do Problema**: antes de prosseguir para a Etapa 2, avalie se você tem informações suficientes para gerar especificações sem ambiguidade. O objetivo é evitar retrabalho — specs geradas com base em informações incompletas ou mal escritas resultam em implementações erradas.

Faça essa avaliação em três dimensões:

**1. Clareza do problema**
- Você consegue descrever com precisão o que deve ser construído ou corrigido?
- A descrição do ticket é suficientemente clara, ou está vaga, contraditória ou incompleta?

**2. Critérios de aceitação**
- Existem critérios de aceitação explícitos ou é possível derivá-los claramente da descrição?
- Os critérios são testáveis (formato Dado/Quando/Então ou equivalente)?

**3. Restrições e contexto técnico**
- Há informações sobre integrações, contratos de dados ou restrições que impactam a implementação?
- Existem dependências com outros sistemas ou tickets que precisam ser consideradas?

**Se qualquer dimensão estiver insuficiente**, não prossiga para a Etapa 2. Em vez disso, faça as perguntas necessárias ao usuário em um único bloco — agrupe todas as dúvidas, não faça uma pergunta por vez. Seja direto sobre o que está faltando:

> "O ticket [ID] tem algumas lacunas que podem gerar retrabalho nas specs. Preciso entender:
> - [pergunta sobre clareza do problema]
> - [pergunta sobre critério de aceitação ausente]
> - [pergunta sobre restrição técnica]"

Só avance para a Etapa 2 quando tiver clareza suficiente para gerar especificações sem lacunas críticas.

### Modo Arquivo

Leia o arquivo informado em `${input:DesignDocPath}` na íntegra.

O Design Document é composto por duas partes:
- **Parte 1 — Narrativa**: contexto de produto, problema, usuários, escopo e critérios de sucesso de negócio
- **Parte 2 — Appendix Técnico**: endpoints, schema, regras de negócio, restrições técnicas e critérios de aceitação

Use **ambas as partes** para montar as specs. A Parte 1 fornece contexto e motivação; a Parte 2 fornece os contratos e requisitos testáveis.

**Verificação de Gaps**: antes de prosseguir, verifique se a Parte 2 contém entradas marcadas com `⚠️ requer definição`.

- Se houver gaps em campos críticos (critérios de aceitação, regras de negócio ou contratos de dados), informe o usuário, liste os gaps e pergunte se deseja preencher agora ou gerar as specs com as lacunas sinalizadas.
- Se os gaps forem em seções opcionais (referências, questões em aberto), prossiga e sinalize nas specs geradas.

### Modo Express

Se nenhum caminho foi fornecido, inicie a coleta inline. Faça em **duas rodadas**: **Rodada 1** — pergunte tipo e descrição. **Rodada 2** — após receber as respostas, faça as questões específicas do tipo identificado.

**Rodada 1 — Identificação:**
- Qual o tipo de trabalho? (`feature`, `bug` ou `refactor`)
- Descreva brevemente o que precisa ser feito.

**Rodada 2 — Questões específicas** (faça apenas após receber as respostas da Rodada 1):

*Para Feature:*
- Quais são os requisitos funcionais principais?
- Existe alguma restrição técnica ou regra de negócio explícita?
- Como validar que está correto? (critérios de aceitação no formato Dado/Quando/Então)

*Para Bug:*
- Qual o comportamento atual e o esperado?
- Qual componente ou camada está envolvido?
- O que a correção não pode quebrar?
- Como provar que o bug foi corrigido?

*Para Refactor:*
- O que está problemático e qual o estado alvo?
- Quais interfaces ou contratos não podem mudar?
- Como provar que o comportamento externo não mudou?

Após coletar as respostas, prossiga para o Planejamento (Etapa 2) usando as informações coletadas no lugar do Design Document.

---

## Etapa 2 — Análise e Planejamento

Com as informações disponíveis (do Jira, do arquivo ou da coleta inline), identifique:

- O **tipo de trabalho**: `feature`, `bug` ou `refactor`
- O **nome do trabalho**: slug em kebab-case para o nome da pasta (ex.: `customer-eligibility`, `fix-payment-timeout`, `extract-auth-service`)
- As **responsabilidades distintas** que precisam de specs separadas

> Uma spec por responsabilidade. Se duas responsabilidades compartilham o mesmo contrato de dados e critérios de aceitação, podem ser agrupadas. Se não, devem ser separadas.

Verifique se a pasta `/specs/{tipo}/{nome}/` já existe:

- Se **não existir**, prossiga normalmente.
- Se **já existir**, informe o usuário e pergunte se deseja sobrescrever o conteúdo existente, usar um nome diferente ou cancelar. Aguarde a decisão antes de continuar.

Antes de apresentar o plano, ordene as specs pela sequência em que devem ser implementadas:

- Specs sem dependências externas devem vir primeiro.
- Specs que dependem de contratos ou componentes gerados por outra spec devem vir depois.
- A numeração (`spec-01`, `spec-02`...) é a ordem de implementação — não é arbitrária.

Apresente ao usuário o plano com:

1. O tipo de trabalho identificado
2. O caminho base que será criado: `/specs/{tipo}/{nome}/`
3. A lista de specs planejadas **na ordem de implementação**:
   - `spec-01-{titulo}.md` — [descrição de uma linha do escopo]
   - `spec-02-{titulo}.md` — [depende de spec-01; descrição de uma linha do escopo]
4. A estrutura do `IMPLEMENTATION.md` que será gerado
5. **Se Modo Jira:** informe que as seguintes sub-tarefas serão criadas no ticket `[ID]`:
   - `[título da spec-01]`
   - `[título da spec-02]`

Aguarde a confirmação do usuário antes de prosseguir para a Etapa 3.

---

## Etapa 3 — Geração dos Arquivos

Após a confirmação, siga esta ordem:

**Se Modo Jira:** antes de criar qualquer arquivo, use o MCP para criar as sub-tarefas no ticket pai — uma por spec planejada:

- **Título:** o mesmo título da spec (ex.: `[spec-01] Título da especificação`)
- **Tipo:** Sub-task
- **Status inicial:** To Do
- **Descrição:** o escopo de uma linha da spec conforme definido no plano

Guarde os IDs retornados (ex.: `PROJ-124`, `PROJ-125`) — eles serão usados ao escrever o `IMPLEMENTATION.md`.

Em seguida, crie todos os arquivos na seguinte estrutura:

```
/specs/
  features/{feature-name}/
    IMPLEMENTATION.md
    spec-01-{titulo}.md
    spec-02-{titulo}.md
  bugs/{bug-name}/
    IMPLEMENTATION.md
    spec-01-{titulo}.md
  refactor/{refactor-name}/
    IMPLEMENTATION.md
    spec-01-{titulo}.md
```

### Convenção de nomenclatura

- **Tipo `feature`**: `/specs/features/{feature-name}/`
- **Tipo `bug`**: `/specs/bugs/{bug-name}/`
- **Tipo `refactor`**: `/specs/refactor/{refactor-name}/`
- **Nome da pasta**: kebab-case, descritivo, sem versão
- **Specs**: `spec-NN-{titulo}.md` onde `NN` é sequencial com zero à esquerda (01, 02, 03...)

Após criar todos os arquivos, informe ao usuário: a lista de caminhos gerados, os IDs das sub-tarefas criadas no Jira (se Modo Jira) e qualquer campo que ficou com gap pendente (marcado com `⚠️ requer definição`).

---

## IMPLEMENTATION.md — Índice e Registro de Implementação

```md
---
type: [feature | bug | refactor]
name: [nome do trabalho]
jira_ticket: [ID do ticket pai — ex.: PROJ-123 — omitir se não vier do Jira]
design_doc: [caminho do Design Document de origem — omitir se modo express ou Jira]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
status: [planned | in-progress | completed]
---

# [Nome do Trabalho]

[Resumo de uma a três frases do que este trabalho entrega ou corrige.]

## Especificações

<!-- Se jira_ticket estiver preenchido, use a tabela com coluna Jira: -->
| # | Arquivo | Escopo | Status | Jira |
|---|---------|--------|--------|------|
| 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending | [ID da sub-tarefa] |
| 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending | [ID da sub-tarefa] |

<!-- Se jira_ticket NÃO estiver preenchido, omita a coluna Jira: -->
<!-- | # | Arquivo | Escopo | Status | -->
<!-- |---|---------|--------|--------| -->
<!-- | 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending | -->
<!-- | 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending | -->

> **Protocolo de atualização:** atualize este arquivo **imediatamente** após cada ação — não agrupe atualizações no final.
> - Ao **iniciar** uma spec: altere seu status para `in-progress`, o status do `IMPLEMENTATION.md` para `in-progress` e atualize `last_updated`. Se houver `jira_ticket`, atualize a sub-tarefa correspondente para **In Progress** via MCP.
> - Ao **concluir** uma spec: altere seu status para `done`, escreva o resumo na seção **Resumos de Implementação** e atualize `last_updated`. Se houver `jira_ticket`, atualize a sub-tarefa para **Done** e adicione um comentário com o resumo da implementação via MCP. Só então avance para a próxima.
> - Não inicie a spec seguinte sem que o resumo da atual esteja registrado.

## Sequência de Implementação

> Esta ordem é **obrigatória**. Não inicie uma spec sem que a anterior esteja concluída.

1. `spec-01-{titulo}.md` — [motivo]
2. `spec-02-{titulo}.md` — [depende de spec-01]

## Resumos de Implementação

> Preenchido incrementalmente — escreva o resumo de cada spec **imediatamente** após concluí-la.
> O objetivo é permitir entender o que foi construído sem precisar ler as specs. Se precisar de mais detalhes, consulte a spec correspondente.

### spec-01-{titulo}.md

`status: pending`

<!-- Ao concluir: descreva os componentes criados ou alterados, o comportamento entregue e decisões técnicas relevantes. -->

### spec-02-{titulo}.md

`status: pending`

<!-- Ao concluir: descreva os componentes criados ou alterados, o comportamento entregue e decisões técnicas relevantes. -->

## Decisões e Desvios

[Registre aqui decisões tomadas em runtime que divergiram do plano original, com justificativa.]

- **[data]**: [decisão ou desvio] — [motivo]

## Resultado Final

> Preenchido após todas as specs estarem com status `done`.

**O que foi entregue**: [síntese do conjunto — não repita os resumos individuais, foque no valor entregue como um todo]
**Fora do escopo**: [o que ficou de fora e por quê]
**Débitos técnicos**: [limitações conhecidas, melhorias postergadas ou riscos identificados]

## Protocolo de Execução

> **Regras obrigatórias — leia antes de escrever qualquer código.**
>
> - Implemente **uma spec por vez**, na ordem numerada. Nunca inicie a spec `N+1` sem que a spec `N` esteja com status `done`.
> - Se durante a implementação surgir uma decisão que diverge do planejado, registre **imediatamente** em **Decisões e Desvios** antes de continuar.
>
> **Ciclo por spec:**
> 1. Leia a spec completamente.
> 2. Atualize o status para `in-progress` e `last_updated` neste arquivo.
> 3. Se `jira_ticket` estiver preenchido: atualize a sub-tarefa correspondente para **In Progress** via MCP.
> 4. Implemente o que a spec descreve.
> 5. Valide todos os Critérios de Aceitação da spec.
> 6. Escreva o resumo em **Resumos de Implementação**: componentes criados/alterados, comportamento entregue, decisões técnicas relevantes.
> 7. Atualize o status para `done` e `last_updated`.
> 8. Se `jira_ticket` estiver preenchido: atualize a sub-tarefa para **Done** e adicione um comentário via MCP com o mesmo conteúdo do resumo escrito no passo 6.
> 9. Somente então avance para a próxima spec.
>
> **Conclusão:** quando todas as specs estiverem `done`, atualize o status deste arquivo para `completed` e preencha **Resultado Final**.
```

---

## Templates de Especificação por Tipo de Trabalho

### Template: Feature

~~~md
---
title: [Título Conciso Descrevendo o Foco da Especificação]
type: feature
implementation_order: [número sequencial — ex.: 1]
version: [ex.: 1.0]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
owner: [Equipe/Indivíduo responsável]
tags: [lista de tags relevantes]
---

# Introdução

[Breve introdução à especificação e ao objetivo que ela pretende atingir.]

## 1. Propósito & Escopo

[O que esta spec cobre e o que está fora do escopo. Público-alvo e premissas assumidas.]

## 2. Definições

[Siglas, abreviações e termos específicos do domínio usados nesta spec.]

## 3. Requisitos, Restrições & Diretrizes

- **REQ-001**: Requisito funcional
- **SEC-001**: Requisito de segurança
- **CON-001**: Restrição técnica ou de negócio
- **GUD-001**: Diretriz recomendada

## 4. Interfaces & Contratos de Dados

[Interfaces, APIs, contratos de dados e pontos de integração.]

## 5. Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: O sistema deve [comportamento] quando [condição]

## 6. Estratégia de Automação de Testes

- **Níveis de Teste**: Unitário, Integração, Ponta a Ponta
- **Frameworks**: [conforme stack do projeto]
- **Gestão de Dados de Teste**: [criação e limpeza]
- **Integração CI/CD**: [pipeline do projeto]
- **Requisitos de Cobertura**: [limites mínimos]
- **Testes de Performance**: [se aplicável]

## 7. Justificativa & Contexto

[Raciocínio por trás dos requisitos e decisões. Referencie o Design Document quando relevante.]

## 8. Dependências & Integrações Externas

*Omita as categorias que não se aplicam ao escopo desta spec.*

### Sistemas Externos
- **EXT-001**: [Nome do sistema] - [Propósito e tipo de integração]

### Serviços de Terceiros
- **SVC-001**: [Nome do serviço] - [Capacidades e SLA requeridos]

### Dependências de Infraestrutura
- **INF-001**: [Componente] - [Requisitos e restrições]

### Dependências de Dados
- **DAT-001**: [Fonte de dados] - [Formato, frequência e acesso]

### Dependências de Plataforma
- **PLT-001**: [Plataforma/runtime] - [Restrições de versão e justificativa]

### Conformidade
- **COM-001**: [Requisito regulatório] - [Impacto na implementação]

## 9. Exemplos & Casos Extremos

```plaintext
// Caso principal:
//   <exemplo>
//
// Caso extremo ou de erro:
//   <comportamento esperado>
```

## 10. Critérios de Validação

[O que deve ser satisfeito para que esta spec seja considerada implementada corretamente.]

## 11. Especificações Relacionadas / Leitura Adicional

- [Spec Relacionada](./spec-NN-{titulo}.md) — [descrição do relacionamento]
- [Documentação Externa](https://exemplo.com/docs) — [o que pode ser encontrado]
~~~

---

### Template: Bug

~~~md
---
title: [Título Conciso Descrevendo o Bug]
type: bug
implementation_order: [número sequencial — ex.: 1]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
owner: [Equipe/Indivíduo responsável]
severity: [critical | high | medium | low]
tags: [lista de tags relevantes]
---

# Introdução

[Descrição objetiva do bug: o que está acontecendo de errado e qual o impacto.]

## 1. Propósito & Escopo

[O que esta spec cobre para corrigir o bug e o que está fora do escopo desta correção.]

## 2. Definições

[Termos específicos do domínio usados nesta spec.]

## 3. Causa Raiz

- **Componente afetado**: [nome do módulo, serviço ou camada]
- **Causa identificada**: [descrição técnica]
- **Hipótese alternativa**: [se houver]

## 4. Comportamento Atual vs. Esperado

| Aspecto | Comportamento Atual | Comportamento Esperado |
|---------|--------------------|-----------------------|
| [ex.: resposta da API] | [o que acontece] | [o que deveria acontecer] |

## 5. Passos de Reprodução

**Pré-condições**: [estado inicial necessário]

1. [Passo 1]
2. [Passo 2]
3. [Resultado observado]

**Ambiente**: [produção / staging / local]

## 6. Requisitos da Correção

- **REQ-001**: A correção deve [comportamento esperado]
- **CON-001**: A correção não deve alterar [contrato / interface existente]

## 7. Interfaces & Contratos de Dados Afetados

[Contratos de dados, APIs ou interfaces impactados pela correção.]

## 8. Critérios de Aceitação

- **AC-001**: Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
- **AC-002**: Os cenários existentes de [funcionalidade relacionada] continuam funcionando

## 9. Estratégia de Testes

- **Teste de regressão**: [como garantir que o bug não volta]
- **Frameworks**: [conforme stack do projeto]
- **Cenários a cobrir**: [lista de cenários críticos]

## 10. Dependências & Contexto Técnico

- **DEP-001**: [Dependência ou componente relacionado] - [como impacta a correção]

## 11. Exemplos & Dados de Teste

```plaintext
// Entrada que causa o bug:
//   <exemplo>
//
// Entrada que deve funcionar corretamente após a correção:
//   <exemplo>
```

## 12. Especificações Relacionadas / Leitura Adicional

- [Spec Relacionada](./spec-NN-{titulo}.md) — [descrição]
- [Documentação Externa](https://exemplo.com/docs) — [contexto relevante]
~~~

---

### Template: Refactor

~~~md
---
title: [Título Conciso Descrevendo o Refactor]
type: refactor
implementation_order: [número sequencial — ex.: 1]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
owner: [Equipe/Indivíduo responsável]
tags: [lista de tags relevantes]
---

# Introdução

[Descrição objetiva do que será refatorado e qual o ganho esperado.]

## 1. Propósito & Escopo

**Em escopo**: [lista do que será refatorado]
**Fora de escopo**: [lista do que não será alterado]

## 2. Definições

[Termos específicos usados nesta spec.]

## 3. Motivação Técnica

- **Problema atual**: [ex.: acoplamento, duplicação, complexidade ciclomática]
- **Impacto do problema**: [ex.: dificuldade de testar, risco de bugs]
- **Métricas atuais** *(se disponíveis)*: [cobertura, tempo de build, dependências]

## 4. Estado Atual vs. Estado Alvo

| Aspecto | Estado Atual | Estado Alvo |
|---------|-------------|-------------|
| [ex.: estrutura de módulos] | [como está] | [como ficará] |
| [ex.: cobertura de testes] | [% atual] | [% alvo] |

## 5. Requisitos & Restrições

- **REQ-001**: O comportamento externo observável deve permanecer idêntico
- **CON-001**: A interface pública de [componente] não deve ser alterada

## 6. Interfaces & Contratos Preservados

[Contratos, APIs e interfaces que devem ser preservados sem alteração.]

## 7. Estratégia de Refactor

1. [Primeiro passo]
2. [Segundo passo]
3. [Terceiro passo]

**Riscos identificados**: [pontos de acoplamento, dados em produção afetados]

## 8. Estratégia de Testes

- **Abordagem**: [ex.: garantir cobertura antes de refatorar]
- **Frameworks**: [conforme stack do projeto]
- **Testes de regressão**: [o que deve ser validado ao final]
- **Métricas de melhoria**: [como medir que o objetivo foi atingido]

## 9. Critérios de Aceitação

- **AC-001**: Todos os testes existentes passam sem modificação de comportamento
- **AC-002**: [Critério técnico de melhoria]
- **AC-003**: A interface pública de [componente] permanece compatível com consumidores existentes

## 10. Dependências & Componentes Afetados

- **DEP-001**: [Componente afetado] - [natureza do impacto e ação necessária]

## 11. Exemplos

```plaintext
// Antes do refactor:
//   <trecho representativo do estado atual>
//
// Depois do refactor:
//   <trecho representativo do estado alvo>
```

## 12. Especificações Relacionadas / Leitura Adicional

- [Spec Relacionada](./spec-NN-{titulo}.md) — [descrição]
- [Documentação Externa](https://exemplo.com/docs) — [contexto relevante]
~~~

---

## Boas Práticas Gerais

- Preencha `date_created` e `last_updated` com a data atual no formato `YYYY-MM-DD`.
- Use linguagem precisa, explícita e sem ambiguidades.
- Distinga claramente entre requisitos, restrições e recomendações.
- Defina todas as siglas e termos específicos do domínio.
- Garanta que cada spec seja autossuficiente — não exija que o leitor consulte outras specs para entender o escopo.
- Cada spec deve cobrir uma única responsabilidade. Se o escopo exigir mais de três grupos de requisitos não relacionados, divida em specs separadas.
- Ao referenciar specs internas, use sempre o caminho relativo (`./spec-NN-{titulo}.md`).
- Se o Design Document de origem contiver `⚠️ requer definição`, sinalize a lacuna na spec — nunca preencha com suposições.
- No modo express, se as respostas coletadas forem insuficientes para gerar critérios de aceitação testáveis, pergunte antes de gerar — specs sem AC são inutilizáveis para implementação.
- No modo Jira, nunca gere specs com base em tickets vagos ou mal escritos. Agrupe todas as dúvidas em um único bloco de perguntas e aguarde as respostas antes de continuar — o custo de perguntar é menor que o custo de retrabalho.

---

## Nota para Implementação

O `IMPLEMENTATION.md` gerado contém a seção **Protocolo de Execução** com todas as regras de sequenciamento e o ciclo obrigatório por spec. Qualquer agente ou desenvolvedor que iniciar a implementação deve ler essa seção antes de escrever código.
