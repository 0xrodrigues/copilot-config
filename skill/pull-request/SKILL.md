---
name: pull-request
description: 'Analisa commits e contexto do repositório, gera a descrição completa do PR seguindo os padrões do projeto e cria o PR automaticamente via GitHub CLI após confirmação do usuário.'
---

### Instruções

```xml
<description>Esta skill coleta commits, diff e contexto disponível (conversa, TASK.md, CONTEXT.md), gera um rascunho completo da descrição do PR, apresenta para revisão e cria o PR via gh CLI após confirmação — sem que o usuário precise preencher ou executar nada.</description>
```

### Fluxo de Trabalho

**Siga estes passos:**

1. Colete o estado atual do repositório.
2. Identifique contexto adicional disponível.
3. Gere o rascunho completo da descrição.
4. Apresente o rascunho ao usuário para revisão.
5. Crie o PR via GitHub CLI após confirmação.

---

### Passo 1 — Coletar estado do repositório

Execute em paralelo:
```bash
git branch --show-current
gh auth status
```

**Detectar a branch base (PR target):**

Teste cada branch de integração conhecida e identifique de qual a branch atual divergiu:
```bash
git log --oneline origin/release..HEAD 2>/dev/null | wc -l
git log --oneline origin/main..HEAD 2>/dev/null | wc -l
git log --oneline origin/master..HEAD 2>/dev/null | wc -l
git log --oneline origin/develop..HEAD 2>/dev/null | wc -l
git log --oneline origin/hom..HEAD 2>/dev/null | wc -l
git log --oneline origin/staging..HEAD 2>/dev/null | wc -l
```

A branch de integração que existir no remote e tiver o maior número de commits exclusivos no HEAD é a branch base. Use-a como PR target — não assuma `main`.

Com a branch base identificada, execute:
```bash
git log <branch-base>..HEAD --oneline
git diff <branch-base>..HEAD --stat
```

Identifique:
- Branch atual (será a branch do PR)
- Branch base detectada (será o target do PR)
- Lista de commits incluídos no PR
- Arquivos alterados e volume de mudanças

---

### Passo 2 — Identificar contexto adicional

Verifique se existem artefatos de contexto, nesta ordem de prioridade:

1. **Mensagens da conversa atual** — o usuário pode ter descrito o que foi implementado
2. **`TASK.md`** — procure em `/specs/` recursivamente ou na raiz do projeto. Leia o arquivo completo: ele contém o escopo, os requisitos funcionais, as regras de negócio e os critérios de aceitação da tarefa em andamento
3. **`CONTEXT.md`** — no mesmo diretório do `TASK.md`. Contém os arquivos, classes e métodos mapeados durante o research — use para confirmar se o diff está alinhado com o escopo planejado

Use esses artefatos para tornar a descrição semântica — explicando o *porquê* das mudanças, não apenas o *o quê*.

---

### Passo 3 — Verificar checklist automaticamente

Antes de gerar o rascunho, verifique ativamente:

- **Testes:** há arquivos de teste no diff? (`*test*`, `*spec*`, `__tests__`)
- **Breaking changes:** algum commit contém `!` ou `BREAKING CHANGE` no footer?
- **TASK.md:** existe `TASK.md` com status `in-progress` ou `completed` relacionado ao diff?
- **CONTEXT.md:** existe `CONTEXT.md` no mesmo diretório do `TASK.md`?

Pré-marque os itens verificáveis. Deixe em aberto apenas o que não pode ser verificado automaticamente.

---

### Passo 4 — Gerar rascunho da descrição

Monte a descrição completa usando as informações coletadas:

```markdown
## O que foi alterado
<!-- Derivado dos commits e diff — liste objetivamente as mudanças realizadas -->

## Motivação
<!-- Derivado do contexto (conversa, TASK.md) — por que essa mudança foi necessária -->

## Como testar
<!-- Derive dos arquivos alterados e tipo de mudança — passos para validar o comportamento -->

## Impacto em outros fluxos
<!-- Analise o diff em busca de dependências, exports alterados ou breaking changes -->

## Checklist
- [x] Testes adicionados ou atualizados
- [ ] Sem breaking changes (ou documentados abaixo)
- [ ] Logs e documentação atualizados
- [ ] `TASK.md` atualizado com status e resumo da implementação (se aplicável)
```

Apresente ao usuário:

```
Rascunho gerado. Revise antes de criar o PR:

Título: tipo(escopo): descrição derivada do commit principal
Branch: feature/nome → <branch-base detectada>

--- DESCRIÇÃO ---
<conteúdo gerado>
-----------------

Deseja criar o PR com esta descrição?
  [1] Criar agora
  [2] Editar título
  [3] Editar descrição
  [4] Cancelar
```

Se o usuário escolher editar, colete os ajustes e apresente o rascunho atualizado antes de prosseguir.

---

### Passo 5 — Criar o PR via GitHub CLI

Após confirmação, execute:

```bash
gh pr create \
  --title "<título>" \
  --body "<descrição completa>" \
  --base <branch-base>
```

Se o repositório tiver reviewers padrão configurados ou o usuário mencionar revisores, adicione `--reviewer usuario1,usuario2`.

Após a criação, exiba:
- A URL do PR criado
- O número do PR (útil para usar com `/pr-plan` futuramente)

---

### Título do PR

Siga o mesmo padrão dos commits — derive do commit mais representativo ou do conjunto:

```
feat(order): adicionar suporte para reembolso parcial
fix(payment): dedução correta do saldo em reembolsos de múltiplos itens
```

---

### Processo de Revisão

```xml
<review>
	<rule>Responda a todos os comentários antes de solicitar uma nova revisão</rule>
	<rule>Use "Resolver conversa" somente após a correção ser implementada</rule>
	<rule>Prefira commits de correção separados em vez de force-push durante a revisão</rule>
	<rule>Registre decisões tomadas a partir de comentários de revisão na seção "Decisões e Desvios" do TASK.md</rule>
</review>
```