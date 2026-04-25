---
name: ticket-update
description: 'Sincroniza TASK.md e especificações com o Jira em dois modos: (1) Publish — pré-implementação, cria subtarefas com specs; (2) Update — pós-implementação, atualiza status e publica no Confluence. Use invocando: /ticket-update [caminho TASK.md] [publish|update]'
---

# Sincronizar com Jira

Seu objetivo é manter o TASK.md e as especificações sincronizadas com o Jira em dois momentos distintos do fluxo.

Esta skill nunca infere nem assume dados — tudo que ela precisa está no `TASK.md`. Se qualquer informação estiver ausente ou inconsistente, ela para e informa antes de executar qualquer ação.

---

## Modos de Operação

A skill opera em dois modos exclusivos:

- **`publish`** — executado **após create-specification**, antes da implementação. Cria as subtarefas no Jira com o conteúdo das especificações.
- **`update`** — executado **após a implementação**, quando o TASK.md tem status `completed`. Atualiza subtarefas, ticket pai e publica no Confluence.

Invoque com:
```
/ticket-update [caminho/TASK.md] publish
/ticket-update [caminho/TASK.md] update
```

---

## Etapa 1 — Localizar, Validar e Interpretar o Modo

**Localização:**

Analise o argumento recebido:

- Se for um caminho de arquivo (ex.: `specs/features/minha-feature/TASK.md`), leia esse arquivo.
- Se for vazio, procure um `TASK.md` no diretório atual. Se houver mais de um, liste os encontrados e pergunte qual usar antes de continuar.

**Determinação do modo:**

O modo é passado explicitamente como segundo argumento (`publish` ou `update`). Se não for informado, pergunte ao usuário qual modo executar.

**Validações obrigatórias — Aplicáveis a ambos os modos:**

1. Se `jira_ticket` estiver vazio no frontmatter, pare: "O TASK.md não tem `jira_ticket` preenchido. Impossível sincronizar com Jira sem ID do ticket pai."

2. Se `jira_ticket_url` estiver vazio, pare: "O TASK.md não tem `jira_ticket_url` preenchido. Informe a URL do ticket antes de continuar."

3. A tabela **Especificações Técnicas** deve existir e ter pelo menos uma linha. Se vazia, pare: "Nenhuma especificação foi declarada no TASK.md."

---

## Modo PUBLISH — Pré-implementação

Executado imediatamente após gerar as specs, **antes de implementar**. Cria as subtarefas no Jira.

**Validações específicas para Publish:**

1. O campo `status` no frontmather é `planned`?
   - Se não for, informe: "Este TASK.md já foi publicado ou está em outro estágio. Para criar novas subtarefas, verifique se o status é `planned`."

2. As colunas **Jira ID** e **Jira URL** na tabela estão vazias?
   - Este é o estado esperado antes de publicar. Se já houver IDs, pergunte: "Este TASK.md já tem subtarefas no Jira. Deseja criar novas, atualizar as existentes ou cancelar?"

3. Leia cada arquivo de especificação referenciado na tabela (ex.: `spec-01-{titulo}.md`). Se algum não existir, pare: "O arquivo de especificação `[caminho]` não foi encontrado."

**Passo 1 — Apresentar Plano:**

Liste o que será criado:

> "Pronto para publicar `[name]` no Jira.
>
> **Ações:**
> - Criar [N] subtarefas a partir das especificações
> - Para cada subtarefa:
>   - Título: `[spec-NN] Título`
>   - Descrição: [primeiros 100 caracteres da spec]
>   - Status: To Do
>
> **Ticket pai:** `[jira_ticket]`
>
> Confirma?"

Aguarde confirmação antes de continuar.

**Passo 2 — Criar subtarefas:**

Para cada linha da tabela **Especificações Técnicas**, na ordem numérica:

1. Leia o arquivo da especificação (ex.: `spec-01-{titulo}.md`).

2. Extraia:
   - **Título**: dos metadados frontmatter `title`
   - **Descrição**: todo o conteúdo do arquivo
   - **Implementação**: número sequencial (1, 2, 3...)

3. Crie a subtarefa no Jira via MCP com os seguintes campos:

   - **Issue Type**: Sub-task
   - **Parent**: ticket pai (`jira_ticket`)
   - **Summary**: `[spec-NN] [Título da Spec]` — extraído do campo `title` no frontmatter da spec
   - **Description**: **cópia literal e integral do arquivo spec em markdown**, do início ao fim, exatamente como está no arquivo `spec-NN-{titulo}.md`. Nenhuma reformatação, nenhuma omissão. Preserve todo o frontmatter, todas as seções, todo o conteúdo.
   - **Status**: To Do
   - **Custom fields**: se houver campos como `implementation_order`, popule-os com o valor do frontmatter

4. Guarde o **Jira ID** retornado (ex.: `PROJ-456`).

5. Se a criação falhar, pause e informe o motivo. Não continue para a próxima subtarefa.

**Passo 3 — Atualizar o TASK.md localmente:**

Após todas as subtarefas serem criadas com sucesso:

1. Para cada linha da tabela, preencha:
   - **Jira ID**: o ID retornado (ex.: `PROJ-456`)
   - **Jira URL**: a URL completa (ex.: `https://empresa.atlassian.net/browse/PROJ-456`)

2. Atualize o campo `status` no frontmatter de `planned` para `in-progress`.

3. Atualize `last_updated` com a data atual.

4. Salve o arquivo localmente.

**Passo 4 — Relatório de Publicação:**

Informe ao usuário:

> "Publicação concluída para `[name]`.
>
> **Subtarefas criadas:**
> - [N] subtarefas
> - [Lista de IDs e URLs]
>
> **Arquivo atualizado:** `[caminho/TASK.md]` (status: `in-progress`)
>
> Próximo passo: Implemente as specs e execute o modo `update` quando terminar."

---

## Modo UPDATE — Pós-implementação

Executado **após a implementação estar concluída**. Atualiza subtarefas, ticket pai e publica no Confluence.

**Validações específicas para Update:**

1. O campo `status` no frontmather é `completed`?
   - Se não for, pare e informe: "O TASK.md ainda não está com status `completed`. Conclua a implementação antes de executar o update."

2. Todas as specs na tabela estão com status `done`?
   - Se alguma estiver `pending` ou `in-progress`, liste quais e pare: "As seguintes specs ainda não foram concluídas: [lista]. Finalize-as antes de continuar."

3. A seção **Resultado Final** está preenchida e não contém placeholders?
   - Se vazia ou só com brackets, pare: "A seção Resultado Final não foi preenchida."

4. Todas as linhas da tabela têm **Jira ID** e **Jira URL** preenchidos?
   - Se alguma estiver vazia, pare: "Linhas faltando IDs do Jira — execute o modo `publish` primeiro."

**Passo 1 — Apresentar Plano:**

> "Pronto para atualizar `[name]`.
>
> **Ações:**
> - Atualizar [N] subtarefas para Done
> - Adicionar resumo de implementação em cada subtarefa
> - Atualizar ticket pai com Resultado Final
> - Publicar no Confluence
>
> Confirma?"

Aguarde confirmação.

**Passo 2 — Atualizar Subtarefas:**

Para cada linha da tabela **Especificações Técnicas**, na ordem numérica:

1. Use o **Jira ID** para localizar a subtarefa.

2. Atualize o status para **Done** via MCP.

3. Leia a seção correspondente em **Resumos de Implementação** (ex.: resumo da `spec-01` para a linha 01).

4. Adicione um comentário na subtarefa com o resumo:
   - Se o resumo estiver vazio ou contiver apenas placeholders, sinalize: "Resumo da spec-NN vazio. Deseja deixar comentário em branco, pular ou pausar?"

5. Se a atualização falhar, pause e informe. Não continue.

**Passo 3 — Atualizar Ticket Pai:**

1. Acesse o ticket pai pela `jira_ticket_url`.

2. Atualize o campo **Description** do ticket com o seguinte conteúdo — **copiado literalmente do `TASK.md`, na ordem exata aqui descrita, sem reformatação ou omissão:**

```
# [Título do TASK.md - cópia literal]

[Resumo inicial do TASK.md - parágrafo de duas a quatro frases - cópia literal]

---

## Contexto
[Seção "Contexto" do TASK.md - conteúdo completo copiado literalmente]

## Escopo
[Seção "Escopo" do TASK.md - ambos os subítens "O que será feito" e "O que não será feito" - conteúdo completo copiado literalmente]

## Requisitos Funcionais
[Seção "Requisitos Funcionais" do TASK.md - lista completa copiada literalmente]

## Regras de Negócio
[Seção "Regras de Negócio" do TASK.md - omitir apenas se a seção não existir no TASK.md. Caso contrário, copiar literalmente]

## Critérios de Aceitação
[Seção "Critérios de Aceitação" do TASK.md - lista completa copiada literalmente]

---

## Resultado Final

**O que foi entregue**
[Campo "O que foi entregue" da seção Resultado Final - cópia literal]

**Fora do Escopo**
[Campo "Fora do escopo" da seção Resultado Final - cópia literal]

**Débitos Técnicos**
[Campo "Débitos técnicos" da seção Resultado Final - omitir seção inteira apenas se este campo estiver vazio. Caso contrário, copiar literalmente]

---

## Decisões e Desvios
[Seção "Decisões e Desvios" do TASK.md - omitir seção inteira apenas se não houver entradas além dos placeholders originais. Caso contrário, copiar literalmente]

---

Specs implementadas:
- [Para cada linha da tabela Especificações Técnicas, na ordem]: spec-NN: [escopo de uma linha - copiado literalmente]
```

3. Atualize o status do ticket para **Done** via MCP.

**Passo 4 — Publicar no Confluence:**

Este passo é opcional. Pergunte ao usuário:

> "Deseja publicar a documentação no Confluence? (sim/não)"

Se sim, continue. Se não, pule para Passo 5.

**Passo 4a — Identificar destino:**

Verifique se o ticket pai tem alguma página do Confluence linkada:
- Se sim, pergunte: "Encontrei a página `[título]` linkada. Deseja atualizar ou criar uma nova?"
- Se não, pergunte: "Informe o espaço e página pai para publicar a documentação."

Aguarde a resposta.

**Passo 4b — Montar conteúdo:**

Publique na página do Confluence o seguinte conteúdo — **copiado literalmente do `TASK.md`, mantendo a estrutura em markdown, sem reformatação ou omissão:**

```
# [Título do TASK.md - cópia literal]

[Resumo do TASK.md - cópia literal]

---

## Contexto
[Seção "Contexto" do TASK.md - conteúdo completo copiado literalmente]

## Escopo
[Seção "Escopo" do TASK.md - ambos os subítens - conteúdo completo copiado literalmente]

## Requisitos Funcionais
[Seção "Requisitos Funcionais" do TASK.md - lista completa copiada literalmente]

## Regras de Negócio
[Seção "Regras de Negócio" do TASK.md - omitir apenas se a seção não existir. Caso contrário, copiar literalmente]

## Critérios de Aceitação
[Seção "Critérios de Aceitação" do TASK.md - lista completa copiada literalmente]

## O que foi entregue
[Campo "O que foi entregue" da seção Resultado Final - cópia literal]

## Fora do Escopo
[Campo "Fora do escopo" da seção Resultado Final - cópia literal]

## Débitos Técnicos
[Campo "Débitos técnicos" da seção Resultado Final - omitir seção apenas se vazio. Caso contrário, copiar literalmente]

## Decisões e Desvios
[Seção "Decisões e Desvios" do TASK.md - omitir apenas se vazia. Caso contrário, copiar literalmente]

---
Ticket: [jira_ticket_url]
Publicado em: [data atual DD/MM/YYYY]
```

**Passo 4c — Publicar:**

- Se for **nova página**: crie no espaço e página pai indicados.
- Se for **atualização**: atualize a página existente.

Adicione o link da página ao ticket pai no Jira.

**Passo 5 — Relatório Final:**

Informe ao usuário:

> "Update concluído para `[name]`.
>
> **Jira:**
> - ✓ [N] subtarefas atualizadas para Done
> - ✓ Comentários adicionados
> - ✓ Ticket pai atualizado
>
> **Confluence:**
> - ✓ Página [criada/atualizada]: [link]
>
> Implementação concluída e sincronizada."

Se qualquer passo falhar, informe exatamente qual falhou, o motivo e o que ainda precisa ser feito.

---

## Boas Práticas

- Nunca execute ações parciais da automação sem informar o usuário. Se uma subtarefa falhar, pause antes de continuar.
- **No modo `publish`**: as especificações são copiadas integralmente como descrição das subtarefas — cópia literal, completa, em markdown, sem reformatação, interpretação ou omissão.
- **No modo `update`**: todo conteúdo vem do `TASK.md` — cópia literal das seções, preservando formatação markdown original, sem adição ou suposição.
- Se o `TASK.md` tiver campos com `⚠️ requer definição`, sinalize antes de publicar.
- O status do `TASK.md` é um indicador crítico: `planned` → `in-progress` (após publish) → `completed` (após implementação).
- Erros de integração com Jira ou Confluence devem sempre informar o MCP error, não tentar contornar.
- Markdown é preservado em todas as uploads: negrito, itálico, listas, tabelas, blocos de código, tudo mantém a formatação original.
