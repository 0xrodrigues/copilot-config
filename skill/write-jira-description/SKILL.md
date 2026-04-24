---
name: write-jira-description
description: 'Cria ou reformata a descrição de um ticket do Jira a partir de uma conversa com o usuário ou de uma descrição existente e vaga. Gera conteúdo IA-friendly e legível por humanos, e escreve diretamente no ticket via MCP. Compatível com os tipos feature, bug e refactor.'
---

# Escrever Descrição do Jira

Seu objetivo é gerar uma descrição estruturada, clara e IA-friendly para um ticket do Jira, e publicá-la diretamente no ticket via MCP.

A descrição gerada deve ser:
- **Legível por humanos**: qualquer pessoa do time — dev, QA, PO — entende o que precisa ser feito
- **IA-friendly**: uma IA consegue extrair o suficiente para gerar specs e implementar sem lacunas
- **Autossuficiente**: não depende de contexto externo para ser compreendida
- **Compatível com o editor rich text do Jira**: sem markdown puro (`##`, `**`) — usa formatação Atlassian

Esta skill opera em dois modos:
- **Modo Entrevista**: o usuário não tem um ticket ainda ou tem apenas uma ideia. A skill conduz a conversa para extrair as informações necessárias.
- **Modo Reformatação**: o usuário passa o ID de um ticket existente com descrição vaga ou desorganizada. A skill lê, extrai o que aproveitável e reformata.

---

## Etapa 1 — Identificação do Modo

Analise o argumento recebido:

- **URL ou ID do Jira** (ex.: `PROJ-123`) → **Modo Reformatação**
- **Vazio ou descrição inline** → **Modo Entrevista**

---

## Modo Entrevista

Conduza a conversa em rodadas. Não despeje todas as perguntas de uma vez — faça uma rodada por vez e avance conforme as respostas chegarem.

**Rodada 1 — Tipo e resumo:**

Pergunte:
1. Qual o tipo de trabalho? (`feature`, `bug` ou `refactor`)
2. Em uma frase: o que precisa ser feito?

**Rodada 2 — Contexto e problema:**

Com base no tipo respondido:

*Feature:*
- Por que isso precisa ser feito? Qual o problema ou oportunidade?
- Quem é impactado (usuário, sistema, time)?

*Bug:*
- O que está acontecendo de errado?
- Qual o comportamento esperado?
- Como reproduzir?

*Refactor:*
- O que está problemático no estado atual?
- Qual o estado alvo desejado?

**Rodada 3 — Requisitos e critérios:**

*Feature:*
- Quais são os requisitos funcionais principais? (o que o sistema deve fazer)
- Existem regras de negócio explícitas?
- Como validar que está correto? (critérios de aceitação no formato Dado/Quando/Então)

*Bug:*
- Qual componente ou camada está envolvida?
- O que a correção não pode quebrar?
- Como provar que o bug foi corrigido?

*Refactor:*
- Quais interfaces ou contratos não podem mudar?
- Como provar que o comportamento externo não mudou?

**Rodada 4 — Escopo e restrições:**

Para todos os tipos:
- O que explicitamente **não** será feito neste ticket?
- Há dependências com outros tickets ou sistemas?
- Há restrições técnicas relevantes?

**Rodada 5 — ID do ticket:**

Ao final, pergunte:
- Qual o ID ou URL do ticket onde a descrição deve ser publicada?

Se o ticket ainda não existir, informe que precisará ser criado antes e que a skill pode publicar assim que o ID estiver disponível.

---

## Modo Reformatação

**Passo 1 — Buscar o ticket:**

Use o MCP do Jira para buscar o ticket pelo ID ou URL fornecido:
- Summary (título)
- Descrição atual
- Tipo
- Critérios de Aceitação (se existirem como campo separado)
- Labels / Componentes

**Passo 2 — Avaliar a descrição existente:**

Analise o que foi encontrado e classifique:

| Situação | Ação |
|----------|------|
| Descrição vaga mas com intenção clara | Extraia o que for aproveitável e preencha os gaps com perguntas diretas |
| Descrição com informações dispersas e desorganizadas | Reorganize no template sem perder informação |
| Descrição quase vazia ou apenas um título | Trate como Modo Entrevista usando os dados do ticket como ponto de partida |

**Passo 3 — Preencher gaps:**

Se houver informações ausentes que impactam a clareza ou a capacidade de uma IA gerar specs a partir da descrição, agrupe em um único bloco e pergunte antes de gerar:

> "A descrição atual do ticket `[ID]` tem lacunas. Para gerar uma descrição completa, preciso saber:
> - [pergunta 1]
> - [pergunta 2]"

Só avance quando tiver informações suficientes.

---

## Etapa 2 — Geração da Descrição

Com todas as informações coletadas, gere a descrição no template abaixo.

**Regras de formatação para o editor rich text do Jira:**
- Títulos de seção: use `h2.` para seções principais e `h3.` para subseções
- Listas: use `*` para itens sem ordem e `#` para itens numerados
- Negrito: use `*texto*`
- Separador: use `----`
- Tabelas: use `||cabeçalho||cabeçalho||` e `|valor|valor|`
- Não use markdown puro (`##`, `**`, `---`)

---

### Template: Feature

```
h2. Contexto

[Por que este trabalho existe. Qual o problema ou oportunidade que motivou este ticket. Quem é impactado e qual o valor esperado. Duas a quatro frases.]

----

h2. Escopo

h3. O que será feito
* [item 1]
* [item 2]

h3. O que não será feito
* [item — e por quê, se relevante]

----

h2. Requisitos Funcionais

* *RF-001*: O sistema deve [comportamento esperado]
* *RF-002*: O sistema deve [comportamento esperado]

----

h2. Regras de Negócio

* *RN-001*: [regra explícita]
* *RN-002*: [regra explícita]

[Remover esta seção se não houver regras de negócio identificadas]

----

h2. Critérios de Aceitação

* *AC-001*: Dado [contexto], Quando [ação], Então [resultado esperado]
* *AC-002*: Dado [contexto], Quando [ação], Então [resultado esperado]

----

h2. Restrições & Dependências

* [restrição técnica ou de negócio relevante]
* *Depende de*: [ticket ou sistema — omitir se não houver]
* *Fora de escopo*: [o que não será tratado aqui]

[Remover esta seção se não houver restrições ou dependências relevantes]
```

---

### Template: Bug

```
h2. Problema

[Descrição objetiva do comportamento incorreto e seu impacto. Quem é afetado e qual a severidade percebida. Duas a quatro frases.]

----

h2. Comportamento Atual vs. Esperado

||Aspecto||Atual||Esperado||
|[ex.: resposta da API]|[o que acontece]|[o que deveria acontecer]|

----

h2. Passos para Reproduzir

*Pré-condições*: [estado inicial necessário]

# [Passo 1]
# [Passo 2]
# [Resultado observado]

*Ambiente*: [produção / staging / local]

----

h2. Critérios de Aceitação

* *AC-001*: Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
* *AC-002*: Os cenários existentes de [funcionalidade relacionada] continuam funcionando

----

h2. Restrições

* A correção não deve alterar [contrato / interface existente]
* [outras restrições relevantes]

[Remover esta seção se não houver restrições relevantes]
```

---

### Template: Refactor

```
h2. Contexto

[O que motivou este refactor. Qual o problema técnico atual e qual o ganho esperado. Duas a quatro frases.]

----

h2. Estado Atual vs. Estado Alvo

||Aspecto||Atual||Alvo||
|[ex.: estrutura de pacotes]|[como está]|[como ficará]|

----

h2. Escopo

h3. O que será refatorado
* [item 1]
* [item 2]

h3. O que não será alterado
* [item — seja explícito sobre o que permanece intacto]

----

h2. Contratos Preservados

[Interfaces, APIs e comportamentos externos que não podem mudar sob nenhuma circunstância.]

* [contrato 1]
* [contrato 2]

----

h2. Critérios de Aceitação

* *AC-001*: Todos os testes existentes passam sem modificação de comportamento
* *AC-002*: [critério técnico de melhoria]
* *AC-003*: A interface pública de [componente] permanece compatível com todos os consumidores

----

h2. Restrições & Dependências

* [restrição técnica relevante]
* *Depende de*: [ticket ou sistema — omitir se não houver]

[Remover esta seção se não houver restrições ou dependências relevantes]
```

---

## Etapa 3 — Revisão e Publicação

**Passo 1 — Apresentar para revisão:**

Antes de publicar, exiba a descrição gerada ao usuário e pergunte:

> "Descrição pronta para o ticket `[ID]`. Revise e confirme para publicar, ou indique o que ajustar."

Aguarde confirmação ou ajustes. Repita quantas vezes for necessário até o usuário confirmar.

**Passo 2 — Publicar via MCP:**

Após confirmação, use o MCP do Jira para atualizar o campo de descrição do ticket com o conteúdo gerado.

Não altere nenhum outro campo do ticket — apenas a descrição.

**Passo 3 — Confirmar:**

Após publicar, informe:

> "Descrição publicada no ticket `[ID]`: [URL do ticket]"

Se a publicação falhar, informe o erro retornado pelo MCP e exiba o conteúdo gerado para que o usuário possa copiar e colar manualmente.

---

## Boas Práticas

- Nunca publique sem confirmação explícita do usuário.
- Nunca invente requisitos, regras de negócio ou critérios de aceitação — se a informação não foi fornecida, sinalize com `⚠️ requer definição` no campo correspondente.
- Critérios de aceitação devem sempre estar no formato Dado/Quando/Então. Se o usuário fornecer em outro formato, converta antes de incluir.
- A descrição deve ser autossuficiente — uma IA deve conseguir extrair specs a partir dela sem consultar nenhuma outra fonte.
- No Modo Reformatação, preserve toda informação útil da descrição original mesmo que mal formatada. Nada se perde — apenas reorganiza.
- Seções marcadas como "remover se não houver" devem ser removidas de fato, não mantidas com conteúdo vazio ou placeholder.