---
name: pr-plan
description: 'Lê os comentários de um Pull Request do GitHub e gera um documento markdown com o resumo do feedback e um plano de implementação das correções/sugestões.'
---

### Instruções

```xml
<description>Esta skill lê todos os comentários de um PR (inline e gerais) via MCP do GitHub e produz um arquivo markdown em docs/review-pr/ com o resumo de cada comentário e um plano concreto para implementar as mudanças sugeridas.</description>
```

### Fluxo de Trabalho

**Siga estes passos:**

1. Receba o número ou URL do PR como argumento.
2. Use o MCP do GitHub para buscar os dados do PR e seus comentários.
3. Analise cada comentário e identifique o que precisa ser alterado.
4. Crie as pastas `docs/review-pr/` na raiz do projeto caso não existam.
5. Gere o arquivo do plano dentro de `docs/review-pr/`.

### Passo 1 — Identificar o PR

Extraia do argumento recebido — o argumento pode vir em qualquer um destes formatos:

- **URL completa:** `https://github.com/owner/repo/pull/123` → extrair owner, repo e número
- **Linguagem natural com repo:** `"pr #34 do repo org/repo_name"` ou `"PR 34 org/repo_name"` → extrair owner/repo e número do texto
- **Apenas número:** usar `git remote get-url origin` para inferir o repositório

Sempre que `owner/repo` estiver explícito no argumento (qualquer formato), use-o diretamente sem consultar o git remote.

### Passo 2 — Buscar dados via MCP do GitHub

Busque as seguintes informações:
- Metadados do PR: título, descrição, autor, branch base
- Comentários inline (em código) com arquivo e linha
- Comentários gerais na thread do PR

### Passo 3 — Analisar os comentários

Para cada comentário identifique:
- **Tipo:** correção de bug, qualidade de código, nomenclatura, refatoração, testes, documentação, arquitetura
- **Arquivo e linha** (se for comentário inline)
- **O que precisa mudar** (o pedido central do revisor)
- **Esforço estimado:** pequeno (renomear/formatar) | médio (refatorar/adicionar lógica) | grande (redesenho/nova feature)

### Passo 4 — Criar diretório e gerar o plano

Antes de escrever o arquivo, garanta que as pastas existam:
```bash
mkdir -p docs/review-pr
```

O nome do arquivo deve ser `PR-PLAN-<número>.md` (ex: `PR-PLAN-123.md`), salvo em `docs/review-pr/`.

Use a estrutura abaixo:

```markdown
# Plano do PR: <título do PR>

**PR:** <URL>
**Autor:** <autor>
**Branch base:** <branch>

---

## Resumo do Feedback

Visão geral em 2-3 frases dos principais temas levantados pelos revisores.

---

## Comentários e Plano de Implementação

### 1. <Título curto para o comentário ou grupo>

**Revisor:** @username
**Arquivo:** `caminho/para/arquivo.ts` (linha N) — omitir se for comentário geral
**Tipo:** correção de bug | sugestão | nomenclatura | refatoração | testes | documentação | arquitetura
**Esforço:** pequeno | médio | grande

**Resumo do comentário:**
> Uma ou duas frases resumindo o que o revisor disse.

**Plano de implementação:**
- Passo concreto para resolver o comentário
- Passo adicional se necessário

---

### 2. ...

---

## Ordem de Execução

Lista priorizada: bloqueadores primeiro, depois por esforço (pequeno → grande).

1. Item N — motivo da prioridade
2. ...
```

### Resultado

Após gerar o arquivo, informe ao usuário:
- O caminho do arquivo gerado (ex: `docs/review-pr/PR-PLAN-123.md`)
- Quantos comentários foram processados
- Quantos itens estão no plano de implementação

### Próximos passos sugeridos

Oriente o usuário sobre o fluxo natural após o plano gerado:

1. Implemente as correções seguindo a **Ordem de Execução** do plano
2. Use `/conventional-commit` para commitar as alterações — a skill vai detectar o `PR-PLAN` automaticamente e ajudar a dividir os commits por item do plano
3. Como o PR já existe, não é necessário criar um novo — solicite nova revisão após o push
