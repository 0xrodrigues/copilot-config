---
name: update-from-task
description: 'Lê um TASK.md com status completed e executa todas as atualizações externas: atualiza sub-tarefas e ticket pai no Jira, e publica documentação no Confluence. Deve ser executada apenas após a implementação estar concluída.'
---

# Update from TASK.md

Seu objetivo é ler o `TASK.md` de uma implementação concluída e executar todas as atualizações externas: Jira e Confluence.

Esta skill nunca infere nem assume dados — tudo que ela precisa está no `TASK.md`. Se qualquer informação estiver ausente ou inconsistente, ela para e informa antes de executar qualquer ação.

---

## Etapa 1 — Localizar e Validar o TASK.md

**Localização:**

Analise o argumento recebido:

- Se for um caminho de arquivo (ex.: `specs/features/minha-feature/TASK.md`), leia esse arquivo.
- Se for vazio, procure um `TASK.md` no diretório atual. Se houver mais de um, liste os encontrados e pergunte qual usar antes de continuar.

**Validações obrigatórias antes de qualquer ação:**

1. O campo `status` no frontmatter é `completed`?
   - Se não for, pare e informe: "O TASK.md ainda não está com status `completed`. Conclua a implementação antes de executar o update."

2. Todas as specs na tabela estão com status `done`?
   - Se alguma estiver `pending` ou `in-progress`, liste quais e pare: "As seguintes specs ainda não foram concluídas: [lista]. Finalize-as antes de continuar."

3. A seção **Resultado Final** está preenchida?
   - Se estiver vazia ou contiver apenas os placeholders originais, pare: "A seção Resultado Final não foi preenchida. Preencha-a antes de continuar."

4. Se `jira_ticket` estiver preenchido, `jira_ticket_url` também deve estar. Se estiver ausente, pergunte antes de continuar.

Se todas as validações passarem, apresente ao usuário o resumo do que será executado e aguarde confirmação:

> "Pronto para executar o update do TASK.md `[name]`.
>
> **Jira:**
> - Atualizar [N] sub-tarefas para Done
> - Adicionar comentário em cada sub-tarefa com o resumo da implementação
> - Atualizar ticket pai `[jira_ticket]` com o Resultado Final
>
> **Confluence:**
> - [Criar nova página / Atualizar página existente — conforme Etapa 3]
>
> Confirma?"

---

## Etapa 2 — Atualizar Jira

Execute apenas se `jira_ticket` estiver preenchido no frontmatter. Se não estiver, pule esta etapa.

**Passo 1 — Atualizar sub-tarefas:**

Para cada linha da tabela **Especificações Técnicas**, na ordem numérica:

1. Use o **Jira ID** da linha para localizar a sub-tarefa.
2. Atualize o status da sub-tarefa para **Done** via MCP.
3. Adicione um comentário na sub-tarefa com o conteúdo do resumo correspondente da seção **Resumos de Implementação**.
   - O resumo da `spec-01` vai para a sub-tarefa da linha 01, e assim por diante.
   - Se o resumo de uma spec estiver vazio ou contiver apenas o placeholder original, sinalize e pergunte se deseja pular o comentário ou pausar.

**Passo 2 — Atualizar ticket pai:**

1. Acesse o ticket pai pela `jira_ticket_url`.
2. Adicione um comentário com o seguinte conteúdo, extraído diretamente do `TASK.md`:

```
[Título do TASK.md] — Implementação concluída

[conteúdo completo da seção Resultado Final]

Specs implementadas:
- spec-01: [escopo de uma linha da tabela]
- spec-02: [escopo de uma linha da tabela]

[Se houver Decisões e Desvios com conteúdo além dos placeholders:]
Decisões e desvios registrados:
- [lista de decisões]
```

3. Atualize o status do ticket pai para **Done** via MCP.

---

## Etapa 3 — Publicar no Confluence

**Passo 1 — Identificar destino:**

Verifique se o ticket pai no Jira tem alguma página do Confluence linkada:
- Se sim, pergunte ao usuário: "Encontrei a página `[título]` linkada no ticket. Deseja atualizar essa página ou criar uma nova?"
- Se não houver link, pergunte: "Onde deseja publicar no Confluence? Informe o espaço e a página pai."

Aguarde a resposta antes de continuar.

**Passo 2 — Montar o conteúdo da página:**

Monte a página com o seguinte conteúdo, extraído integralmente do `TASK.md`:

```
# [Título do TASK.md]

[Resumo do TASK.md — parágrafo de duas a quatro frases]

---

## Contexto

[Conteúdo da seção Contexto]

## Escopo

[Conteúdo da seção Escopo]

## Requisitos Funcionais

[Conteúdo da seção Requisitos Funcionais]

## Regras de Negócio

[Conteúdo da seção Regras de Negócio — omitir se a seção foi removida]

## Critérios de Aceitação

[Conteúdo da seção Critérios de Aceitação]

## O que foi entregue

[Conteúdo do campo "O que foi entregue" do Resultado Final]

## Fora do Escopo

[Conteúdo do campo "Fora do escopo" do Resultado Final]

## Débitos Técnicos

[Conteúdo do campo "Débitos técnicos" do Resultado Final — omitir se vazio]

## Decisões e Desvios

[Conteúdo da seção Decisões e Desvios — omitir se não houver entradas além dos placeholders]

---
Ticket: [jira_ticket_url]
Gerado em: [data atual no formato DD/MM/YYYY]
```

**Passo 3 — Publicar via MCP:**

- Se for **nova página**: crie no espaço e página pai informados pelo usuário.
- Se for **atualização**: atualize a página existente preservando o histórico de versões do Confluence.

Após publicar, adicione o link da página ao ticket pai no Jira via MCP.

---

## Etapa 4 — Confirmação Final

Após todas as ações, apresente o resumo do que foi executado:

> "Update concluído para `[name]`.
>
> **Jira:**
> - ✓ [N] sub-tarefas atualizadas para Done
> - ✓ Comentários adicionados em cada sub-tarefa
> - ✓ Ticket pai `[jira_ticket]` atualizado com o Resultado Final
>
> **Confluence:**
> - ✓ Página [criada / atualizada]: [link da página]"

Se qualquer passo falhar, informe exatamente qual falhou, o motivo retornado pelo MCP e o que ainda precisa ser feito manualmente.

---

## Boas Práticas

- Nunca execute atualizações parciais sem informar o usuário. Se uma sub-tarefa falhar, pause e informe antes de continuar para a próxima.
- Nunca reescreva ou interprete o conteúdo do `TASK.md` — copie-o literalmente para os comentários e para a página do Confluence.
- Se o `TASK.md` tiver campos com `⚠️ requer definição` ainda não resolvidos, sinalize antes de publicar e pergunte se deseja prosseguir mesmo assim.
- O conteúdo publicado no Confluence deve ser legível por qualquer pessoa do time — não adicione jargões técnicos além do que está no `TASK.md`.