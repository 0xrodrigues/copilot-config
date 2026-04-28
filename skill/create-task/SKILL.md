---
name: create-task
description: Gera apenas o TASK.md com a visão funcional completa de uma demanda (feature, bug ou refactor) a partir de um ticket Jira ou descrição inline. Não gera especificações técnicas. Primeiro passo do fluxo: create-task → research-task → create-specs. Usar quando o usuário menciona "criar task", "capturar demanda", "registrar ticket" ou "TASK.md".
---

# Criar TASK.md

Gera o documento `TASK.md` com a visão funcional completa da demanda — o que será feito, por quê e como validar. É o primeiro passo do fluxo de três etapas:

1. **`create-task`** ← você está aqui — captura a demanda e gera o `TASK.md`
2. **`research-task`** — explora o projeto e gera o `CONTEXT.md`
3. **`create-specs`** — lê `TASK.md` + `CONTEXT.md` e gera as especificações técnicas

## Contexto do Projeto

Antes de qualquer ação, leia dois arquivos na raiz do projeto:

1. `docs/AGENTS.md` — se existir, use como mapa dos padrões do time. Não leia todos os arquivos referenciados — navegue apenas quando precisar de informação concreta.
2. `TASK.md` na raiz — se existir, leia para entender o contexto de trabalho em andamento.

Se nenhum existir, prossiga normalmente.

---

## Etapa 1 — Identificação do Modo

Analise o argumento recebido:

- **URL ou ID do Jira** (ex.: `https://empresa.atlassian.net/browse/PROJ-123` ou `PROJ-123`) → **Modo Jira**
- **Vazio ou descrição inline** → **Modo Express**

---

### Modo Jira

Busque via MCP do Jira: summary, descrição completa, tipo, critérios de aceitação, componentes/labels e links.

Identifique todos os links do Confluence no ticket e busque o conteúdo de cada página via MCP do Confluence. Essas páginas geralmente contêm regras de negócio e refinamento técnico ausentes no ticket.

**Mapeamento de tipo:**

| Tipo no Jira | Tipo de task |
|---|---|
| Story, Task, Sub-task | `feature` |
| Bug | `bug` |
| Refactor (label ou convenção) | `refactor` |

Se o tipo não for mapeável, pergunte antes de continuar.

**Avalie completude em três dimensões:**

1. **Clareza do problema** — é possível descrever com precisão o que deve ser construído ou corrigido?
2. **Critérios de aceitação** — existem critérios explícitos ou deriváveis, testáveis no formato Dado/Quando/Então?
3. **Regras de negócio e restrições** — há regras, restrições técnicas ou dependências documentadas que impactam a implementação?

Se qualquer dimensão estiver insuficiente, agrupe todas as dúvidas em um único bloco e aguarde resposta antes de continuar.

---

### Modo Express

**Rodada 1:**
- Qual o tipo de trabalho? (`feature`, `bug` ou `refactor`)
- Descreva brevemente o que precisa ser feito.

**Rodada 2** (após receber as respostas):

*Feature:* requisitos funcionais principais, regras de negócio explícitas, restrições técnicas, critérios de aceitação no formato Dado/Quando/Então.

*Bug:* comportamento atual e esperado, componente ou camada envolvida, o que a correção não pode quebrar, como provar que o bug foi corrigido.

*Refactor:* o que está problemático e qual o estado alvo, interfaces ou contratos que não podem mudar, como provar que o comportamento externo não mudou.

---

## Etapa 2 — Classificação e Apresentação

**Classificação de tamanho:**

| Tamanho | Critério |
|---------|----------|
| **Small** | 1 responsabilidade, sem integração nova, sem impacto em contrato público |
| **Medium** | 2–3 responsabilidades, ou integração existente, ou impacto em produção |
| **Large** | 4+ responsabilidades, ou nova integração, ou mudança de contrato público |

Apresente ao usuário:

1. Tipo identificado e tamanho (com justificativa em uma frase)
2. Caminho do arquivo a ser criado: `/specs/{tipo}/{nome}/TASK.md`
3. Campos com informação insuficiente marcados com `⚠️ requer definição`

Verifique se `/specs/{tipo}/{nome}/` já existe. Se sim, pergunte se deseja sobrescrever, usar nome diferente ou cancelar.

**Aguarde confirmação antes de gerar o arquivo.**

---

## Etapa 3 — Geração do TASK.md

Crie o arquivo em `/specs/{tipo}/{nome}/TASK.md` usando o template abaixo:

```md
---
type: [feature | bug | refactor]
size: [small | medium | large]
name: [nome do trabalho em kebab-case]
jira_ticket: [ID do ticket — omitir se Modo Express]
jira_ticket_url: [URL completa do ticket — omitir se Modo Express]
date_created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
status: planned
---

# [Nome do Trabalho]

[Resumo de 2 a 4 frases em linguagem clara: o que este trabalho entrega ou corrige, qual problema resolve e qual o valor para o negócio ou sistema.]

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

> Remova esta seção se não houver regras de negócio identificadas.

- **RN-001**: [regra de negócio explícita]

## Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: Dado [contexto], Quando [ação], Então [resultado esperado]
```

Após criar o arquivo, informe o caminho gerado e os campos marcados com `⚠️ requer definição`.

**Próximo passo:** rode a skill `research-task` para explorar o projeto, mapear arquivos, classes e impactos, e gerar o `CONTEXT.md`. Após isso, rode `create-specs` para gerar as especificações técnicas.

---

## Boas Práticas

- Preencha `date_created` e `last_updated` com a data atual no formato `YYYY-MM-DD`.
- O `TASK.md` usa linguagem clara e funcional — sem código, sem jargões desnecessários.
- Campos marcados com `⚠️ requer definição` nunca devem ser preenchidos com suposições.
- Nunca gere o arquivo com informações insuficientes. Agrupe todas as dúvidas em um único bloco e aguarde resposta.
