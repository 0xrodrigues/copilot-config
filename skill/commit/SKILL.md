---
name: commit
description: 'Analisa o diff do repositório, identifica agrupamentos semânticos e propõe commits atômicos seguindo Conventional Commits. Permite executar todos de uma vez ou um por vez, com confirmação do usuário.'
---

### Instruções

```xml
<description>Esta skill analisa as mudanças no repositório, cruza com contexto disponível, propõe uma divisão em commits atômicos e executa conforme a escolha do usuário — todos de uma vez ou um por vez.</description>
```

### Fluxo de Trabalho

**Siga estes passos:**

1. Colete o estado atual do repositório.
2. Identifique contexto adicional disponível.
3. Analise e proponha a divisão em commits atômicos.
4. Pergunte ao usuário como deseja executar.
5. Execute conforme a escolha, com confirmação antes de cada commit.

---

### Passo 1 — Coletar estado do repositório

Execute em paralelo:
```bash
git status
git diff
git diff --cached
git branch --show-current
```

---

### Passo 2 — Identificar contexto adicional

Verifique se existem artefatos de contexto disponíveis, nesta ordem de prioridade:

1. **Mensagens da conversa atual** — o usuário pode ter descrito o que foi implementado
2. **`TASK.md`** — procure em `/specs/` recursivamente ou na raiz do projeto. Leia o arquivo completo: ele contém o escopo, os requisitos funcionais, as regras de negócio e os critérios de aceitação da tarefa em andamento
3. **`CONTEXT.md`** — no mesmo diretório do `TASK.md`. Contém os arquivos, classes e métodos mapeados durante o research — use para confirmar se o diff está alinhado com o escopo planejado

Se encontrar algum desses artefatos, use-os para entender *por que* cada arquivo foi alterado, tornando a divisão semântica em vez de puramente estrutural.

---

### Passo 3 — Propor divisão em commits atômicos

Analise o diff completo e agrupe as mudanças em commits atômicos seguindo estas regras:

- Cada commit deve ter **uma única responsabilidade** (um fix, uma feature, uma refatoração)
- Mudanças em arquivos diferentes que servem ao **mesmo propósito** ficam no mesmo commit
- Mudanças no **mesmo arquivo** que servem a propósitos diferentes devem ser separadas
- A **ordem importa**: commits que outros dependem vêm primeiro

Para cada commit proposto, defina:
- Quais arquivos/hunks fazem parte dele
- A mensagem no formato Conventional Commits (veja estrutura abaixo)
- A justificativa da separação (uma linha)

Apresente o plano ao usuário neste formato:

```
Encontrei X alterações. Sugiro dividir em N commits:

── Commit 1 ──────────────────────────────
tipo(escopo): descrição
Arquivos: arquivo-a.ts, arquivo-b.ts
Motivo: <por que estão juntos>

── Commit 2 ──────────────────────────────
tipo(escopo): descrição
Arquivos: arquivo-c.ts
Motivo: <por que está separado>

...

Como deseja executar?
  [1] Todos de uma vez
  [2] Um por vez (com confirmação a cada commit)
```

---

### Passo 4 — Verificar branch e definir PR target

Execute:
```bash
git branch --show-current
```

**Cenário A — Branch de integração (`main`, `master`, `develop`, `hom`, `homologacao`, `release`, `staging`):**

O usuário está iniciando uma nova implementação. Criação de nova branch é **obrigatória**.

1. Registre a branch atual como **PR target** (ex: `release`)
2. Sugira um nome para a nova branch no padrão `tipo/descricao-curta-em-kebab-case`, derivado dos commits propostos
3. Confirme com o usuário antes de criar:
   > Você está na branch `release`. Vou criar a branch `feat/descricao` a partir dela e abrir o PR para `release`. Confirma?
4. Após confirmação:
```bash
git checkout -b tipo/descricao-curta
```

**Cenário B — Branch de feature (qualquer outra):**

O usuário já está na branch correta. Não pergunte sobre criar nova branch — apenas prossiga com os commits.

Detecte a branch base (PR target) executando para cada branch de integração conhecida:
```bash
git log --oneline origin/release..HEAD 2>/dev/null | wc -l
git log --oneline origin/main..HEAD 2>/dev/null | wc -l
# ... demais branches de integração
```
A branch de integração com mais commits exclusivos no HEAD é a base. Registre-a como **PR target**.

Guarde o **PR target** identificado — será usado na sugestão final do Passo 5.

---

### Passo 5 — Executar commits

**Opção 1 — Todos de uma vez:**

Mostre o resumo completo dos N commits e peça confirmação final antes de executar. Execute cada commit em sequência sem novas interrupções.

**Opção 2 — Um por vez:**

Para cada commit, antes de executar:
- Mostre a mensagem e os arquivos
- Aguarde confirmação do usuário (`confirmar` / `pular` / `editar mensagem`)
- Se `editar mensagem`: solicite a nova mensagem e aplique
- Se `pular`: registre como pulado e avance para o próximo

Ao final, informe quantos commits foram criados e quais foram pulados (se houver).

Em seguida, verifique se já existe um PR aberto para a branch atual:
```bash
gh pr list --head $(git branch --show-current) --json number,url,title
```

- **Se não houver PR aberto:** sugira ao usuário usar `/pull-request` para criar o PR
- **Se já houver PR aberto:** informe o número e a URL do PR existente e oriente o usuário a solicitar nova revisão

---

### Estrutura da Mensagem de Commit

```xml
<commit-message>
	<type>feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert</type>
	<scope>()</scope>
	<description>Um resumo curto e imperativo da mudança</description>
	<body>(opcional: explicação mais detalhada)</body>
	<footer>(opcional: ex. BREAKING CHANGE: detalhes, referências a issues, ou Refs: caminho/do/TASK.md)</footer>
</commit-message>
```

### Exemplos

```xml
<examples>
	<example>feat(parser): adicionar suporte para análise de arrays</example>
	<example>fix(ui): corrigir alinhamento do botão</example>
	<example>docs: atualizar README com instruções de uso</example>
	<example>refactor: melhorar desempenho do processamento de dados</example>
	<example>chore: atualizar dependências</example>
	<example>feat!: enviar e-mail no cadastro (BREAKING CHANGE: serviço de e-mail obrigatório)</example>
</examples>
```

### Validação

```xml
<validation>
	<type>Deve ser um dos tipos permitidos. Veja <reference>https://www.conventionalcommits.org/en/v1.0.0/#specification</reference></type>
	<scope>Opcional, mas recomendado para maior clareza.</scope>
	<description>Obrigatório. Use o modo imperativo (ex.: "adicionar", não "adicionado").</description>
	<body>Opcional. Use para contexto adicional.</body>
	<footer>Use para breaking changes, referências a issues ou ao TASK.md quando aplicável.</footer>
</validation>
```