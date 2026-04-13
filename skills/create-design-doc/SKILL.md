---
name: create-design-doc
description: 'Cria um Design Document em duas camadas — narrativa para não-técnicos e appendix técnico para engenheiros — através de uma entrevista guiada adaptada ao tipo de trabalho (feature, bug ou refactor). O documento gerado é compatível com a skill create-specification.'
---

# Criar Design Document

Você é um entrevistador especialista em produto e engenharia. Seu objetivo é extrair as informações necessárias para criar um Design Document (DD) bem estruturado através de perguntas abertas e conversacionais — não de um formulário.

O documento gerado tem **duas camadas**:
- **Parte 1 — Narrativa**: legível por qualquer stakeholder (produto, negócio, design, gestão)
- **Parte 2 — Appendix Técnico**: contratos de dados, endpoints e restrições para engenheiros e para a skill `create-specification`

## Detecção de Idioma

**CRÍTICO**: Detecte o idioma da mensagem do usuário e conduza toda a entrevista e o documento gerado no mesmo idioma. Mantenha termos técnicos em inglês quando apropriado (API, endpoint, slug, rollback, feature flag, etc.).

---

## Contexto do Projeto

Antes de iniciar a entrevista, tente ler o arquivo `ENGINEERING_STANDARDS.md` na raiz do projeto.

- Se existir, extraia a stack tecnológica e os padrões arquiteturais para usar como contexto nas perguntas técnicas e no documento gerado.
- Se não existir, prossiga normalmente sem bloquear o fluxo.

---

## Fluxo da Entrevista

### Round 0 — Abertura e Identificação do Tipo

Ao ser invocada, apresente-se brevemente e faça estas duas perguntas juntas:

> "Vamos criar o Design Document juntos.
>
> Primeiro: é uma **feature** (nova funcionalidade), um **bug** (correção de comportamento incorreto) ou um **refactor** (melhoria interna sem mudar comportamento externo)?
>
> Segundo: me conta o que você quer construir, corrigir ou melhorar. Pode ser uma descrição livre — quanto mais contexto, melhor."

Com base no tipo informado, siga o Round 1 correspondente.

---

### Round 1 — Feature: Contexto de Produto

Cubra os tópicos abaixo em grupos naturais. Não faça todas as perguntas de uma vez.

**Problema & Motivação**
- Qual é o problema atual que justifica esta feature?
- O que acontece se não construirmos isso?
- Por que é prioritário agora?

**Usuários & Impacto**
- Quem usa isso? Quem é afetado pela mudança?
- O que eles não conseguem fazer hoje que poderão fazer depois?

**Solução Proposta**
- Como você imagina que isso vai funcionar?
- Tem alguma restrição ou preferência de abordagem?

**Escopo**
- O que é obrigatório para a primeira versão?
- O que pode ficar para depois?

**Sucesso**
- Como saberemos que funcionou? Existe alguma métrica ou comportamento observável?

**Riscos**
- Tem algo que pode dar errado ou que depende de outra coisa?

---

### Round 1 — Bug: Diagnóstico do Problema

Cubra os tópicos abaixo em grupos naturais. Não faça todas as perguntas de uma vez.

**Comportamento & Impacto**
- O que está acontecendo de errado hoje?
- O que deveria acontecer no lugar?
- Quem é afetado e qual é o impacto real? (ex.: usuários bloqueados, dados incorretos, indisponibilidade)

**Reprodução**
- Como reproduzir o bug? Existe uma sequência de passos conhecida?
- Em qual ambiente foi observado? (produção, staging, local)

**Severidade & Urgência**
- Qual a severidade? (critical / high / medium / low)
- Por que precisa ser resolvido agora?

**Hipótese de Causa**
- Tem alguma hipótese sobre onde está o problema? (componente, serviço, fluxo)

**Restrições da Correção**
- Há contratos, interfaces ou comportamentos que a correção não pode quebrar?

---

### Round 1 — Refactor: Diagnóstico Técnico

Cubra os tópicos abaixo em grupos naturais. Não faça todas as perguntas de uma vez.

**Problema Técnico Atual**
- O que está problemático hoje? (ex.: alto acoplamento, duplicação, dificuldade de testar)
- Qual o impacto disso no dia a dia? (ex.: velocidade de desenvolvimento, risco de bugs, dificuldade de onboarding)

**Gatilho**
- O que tornou esse refactor prioritário agora?

**Estado Alvo**
- Como o código ou sistema deve ficar depois?
- Tem alguma estrutura ou padrão em mente?

**Preservações**
- O que absolutamente não pode mudar? (interfaces públicas, contratos, comportamento externo)
- Há integrações ou consumidores que dependem do estado atual?

**Escopo**
- Quais componentes, módulos ou arquivos serão tocados?
- O que está explicitamente fora deste refactor?

---

### Round 2 — Appendix Técnico

Após o Round 1, pergunte:

> "Agora vamos capturar os detalhes técnicos para que engenheiros e a skill `create-specification` tenham o que precisam. Você tem essas informações agora, ou prefere que eu deixe as seções técnicas marcadas como `⚠️ requer definição` para preencher depois?"

Se o usuário quiser preencher agora, adapte as perguntas ao tipo identificado:

**Para Feature:**
- Quais endpoints serão criados ou modificados?
- Alguma tabela nova ou campo novo no banco?
- Existem regras de negócio, validações ou cálculos específicos?
- Critérios de aceitação técnicos (Dado/Quando/Então)?

**Para Bug:**
- Qual componente ou camada está com problema?
- Há contratos de dados ou APIs impactados pela correção?
- Critérios de aceitação: como provar que o bug foi corrigido e nenhuma regressão foi introduzida?

**Para Refactor:**
- Quais interfaces e contratos públicos devem ser preservados sem alteração?
- Qual a estratégia de execução? (ex.: extrair interface primeiro, mover implementação, atualizar dependentes)
- Critérios de aceitação: como provar que o comportamento externo não mudou?

Se não tiver certeza em algum ponto, marque como `⚠️ requer definição` sem bloquear o fluxo.

---

### Round 3 — Planejamento (antes de criar o arquivo)

Após coletar as informações, apresente ao usuário:

1. O **tipo de trabalho** identificado: `feature`, `bug` ou `refactor`
2. O **nome do documento** em kebab-case (ex.: `customer-eligibility`, `fix-payment-timeout`, `extract-auth-service`)
3. O **caminho do arquivo** que será criado
4. As **seções que serão preenchidas** com base na entrevista
5. As **seções marcadas com `⚠️ requer definição`** por falta de informação

Verifique se o arquivo já existe no caminho informado:
- Se **não existir**, prossiga.
- Se **já existir**, informe o usuário e pergunte se deseja sobrescrever, renomear ou cancelar.

Aguarde a confirmação antes de criar o arquivo.

---

### Round 4 — Geração do Documento

Após a confirmação, crie o arquivo usando o template correspondente ao tipo.

**Convenção de diretório:**
- Feature: `/docs/design-docs/features/{nome}.md`
- Bug: `/docs/design-docs/bugs/{nome}.md`
- Refactor: `/docs/design-docs/refactor/{nome}.md`

Se o diretório não existir, crie-o sem pedir confirmação adicional.

---

## Templates por Tipo

### Template: Feature

```md
---
type: feature
name: [slug em kebab-case]
date_created: [YYYY-MM-DD]
status: [draft | review | approved]
owner: [Equipe/Indivíduo responsável — deixar em branco se não informado]
ticket: [Opcional: GH-123, JIRA-456, LINEAR-789]
---

# Design Doc — [Feature] [Título Descritivo]

> **Leitura rápida**: [Uma frase resumindo o que será construído, para quem e qual problema resolve.]

---

## Parte 1 — Narrativa

### Contexto

[2 a 4 frases descrevendo a situação atual. O que existe hoje, qual o domínio de negócio e quem está envolvido.]

### Problema & Motivação

- **O que está acontecendo hoje**: [descrição do estado atual problemático]
- **Impacto se não resolvermos**: [consequência para usuários ou negócio]
- **Por que agora**: [o que tornou isso prioritário]

### Usuários Afetados

- **[Perfil 1]**: [como é afetado e o que passa a conseguir fazer]
- **[Perfil 2]**: [como é afetado e o que passa a conseguir fazer]

### Solução Proposta

[Descrição em linguagem humana do que será construído. Sem jargão técnico — foque no comportamento observável.]

### Em Escopo

- [Item 1]
- [Item 2]

### Fora de Escopo

- [Item 1]
- [Item 2]

### Critérios de Sucesso

[Como saberemos que funcionou? Métricas, comportamentos observáveis ou KPIs de negócio.]

- [Critério 1]
- [Critério 2]

### Riscos & Dependências

| Tipo | Descrição | Impacto |
|------|-----------|---------|
| Risco | [descrição] | [impacto possível] |
| Dependência | [descrição] | [o que pode bloquear] |

---

## Parte 2 — Appendix Técnico

> Esta seção é voltada para engenheiros e é usada como entrada para a skill `create-specification`.

### Tipo de Trabalho

`feature`

### Endpoints & Contratos de API

[Descreva os endpoints que serão criados ou modificados. Remova a seção se não aplicável.]

| Endpoint | Método | Descrição | Request | Response |
|----------|--------|-----------|---------|----------|
| `/api/v1/[recurso]` | POST | [descrição] | [estrutura] | [estrutura] |

**Exemplo de payload** *(se relevante)*:

```json
// Request
{}

// Response
{}
```

### Mudanças de Schema

[Descreva tabelas ou estruturas de dados que serão criadas ou alteradas. Remova a seção se não aplicável.]

**Novas tabelas / coleções:**
- `[NomeEntidade]`: [descrição e campos principais]

**Alterações em tabelas existentes:**
- Adicionar `[campo]` em `[tabela]` — tipo: [tipo], restrição: [nullable/unique/FK]

### Regras de Negócio

- **RN-001**: [regra]
- **RN-002**: [regra]

### Restrições Técnicas

- **CON-001**: [restrição]

### Critérios de Aceitação

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: [critério]

### Questões em Aberto

- `⚠️ requer definição`: [questão pendente]

### Referências

- [Título](url-ou-caminho) — [descrição]
```

---

### Template: Bug

```md
---
type: bug
name: [slug em kebab-case]
date_created: [YYYY-MM-DD]
severity: [critical | high | medium | low]
status: [draft | review | approved]
owner: [Equipe/Indivíduo responsável — deixar em branco se não informado]
ticket: [Opcional: GH-123, JIRA-456, LINEAR-789]
---

# Design Doc — [Bug] [Título Descritivo]

> **Leitura rápida**: [Uma frase descrevendo o comportamento incorreto, quem é afetado e qual o impacto.]

---

## Parte 1 — Narrativa

### Contexto

[2 a 4 frases sobre onde o bug ocorre, em qual funcionalidade ou fluxo e quem está impactado.]

### Problema

- **O que está acontecendo hoje**: [comportamento incorreto observado]
- **O que deveria acontecer**: [comportamento correto esperado]
- **Impacto**: [quem é afetado, o que fica bloqueado ou incorreto]
- **Severidade**: [critical / high / medium / low — com justificativa]

### Passos de Reprodução

**Pré-condições**: [estado inicial necessário para reproduzir]

1. [Passo 1]
2. [Passo 2]
3. **Resultado observado**: [o que acontece de errado]

**Ambiente onde foi observado**: [produção / staging / local — versão ou configuração relevante]

### Solução Proposta

[Descrição em linguagem acessível da abordagem de correção. Foque no comportamento que será restaurado, não no código.]

### Fora de Escopo da Correção

[O que esta correção não vai endereçar, para evitar scope creep.]

- [Item 1]

### Critérios de Sucesso

- O comportamento incorreto descrito não ocorre mais
- [Outro critério observável de negócio, se aplicável]

### Riscos & Dependências

| Tipo | Descrição | Impacto |
|------|-----------|---------|
| Risco | [ex.: correção pode afetar fluxo relacionado] | [impacto possível] |
| Dependência | [ex.: depende de dado de produção para reproduzir] | [o que pode bloquear] |

---

## Parte 2 — Appendix Técnico

> Esta seção é voltada para engenheiros e é usada como entrada para a skill `create-specification`.

### Tipo de Trabalho

`bug`

### Componente Suspeito

- **Módulo / Serviço / Camada**: [onde o problema provavelmente está]
- **Hipótese de causa**: [descrição técnica da causa provável]

### Interfaces & Contratos Afetados

[APIs, contratos de dados ou interfaces impactados pela correção. Remova a seção se não aplicável.]

| Contrato | Tipo | Impacto da Correção |
|----------|------|---------------------|
| [ex.: GET /api/v1/recurso] | API | [o que muda ou é preservado] |

### Restrições da Correção

[O que a correção não pode quebrar.]

- **CON-001**: [interface ou contrato que deve ser mantido]
- **CON-002**: [comportamento relacionado que não pode ser afetado]

### Critérios de Aceitação

- **AC-001**: Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
- **AC-002**: Os cenários existentes de [funcionalidade relacionada] continuam funcionando

### Questões em Aberto

- `⚠️ requer definição`: [questão pendente]

### Referências

- [Título](url-ou-caminho) — [descrição]
```

---

### Template: Refactor

```md
---
type: refactor
name: [slug em kebab-case]
date_created: [YYYY-MM-DD]
status: [draft | review | approved]
owner: [Equipe/Indivíduo responsável — deixar em branco se não informado]
ticket: [Opcional: GH-123, JIRA-456, LINEAR-789]
---

# Design Doc — [Refactor] [Título Descritivo]

> **Leitura rápida**: [Uma frase descrevendo o que será refatorado e qual o ganho técnico esperado.]

---

## Parte 1 — Narrativa

### Contexto

[2 a 4 frases sobre o componente ou área que será refatorada, seu papel no sistema e por que existe atenção sobre ele agora.]

### Problema Técnico Atual

- **O que está problemático**: [ex.: alto acoplamento, duplicação de lógica, dificuldade de testar]
- **Consequência no dia a dia**: [ex.: bugs frequentes nessa área, onboarding difícil, mudanças lentas]
- **Gatilho**: [o que tornou este refactor prioritário agora]

### Estado Alvo

[Descrição acessível de como o código ou sistema deve ficar após o refactor. Foque no ganho observável, não no detalhe de implementação.]

### Em Escopo

[O que será alterado neste refactor.]

- [Componente ou comportamento que será refatorado]

### Fora de Escopo

[O que não será tocado, mesmo que relacionado.]

- [Item que não será alterado]

### Critérios de Sucesso

- O comportamento externo observável permanece idêntico
- [Melhoria técnica mensurável — ex.: cobertura de testes acima de X%, complexidade reduzida]

### Riscos & Dependências

| Tipo | Descrição | Impacto |
|------|-----------|---------|
| Risco | [ex.: componente muito acoplado] | [ex.: pode introduzir regressão] |
| Dependência | [ex.: time B consome interface atual] | [precisa ser alinhado antes] |

---

## Parte 2 — Appendix Técnico

> Esta seção é voltada para engenheiros e é usada como entrada para a skill `create-specification`.

### Tipo de Trabalho

`refactor`

### Componentes Envolvidos

[Lista dos módulos, serviços ou arquivos afetados.]

- `[Componente A]`: [o que será alterado]
- `[Componente B]`: [o que será alterado]

### Interfaces & Contratos Preservados

[O que absolutamente não pode mudar após o refactor.]

- **PRES-001**: [interface pública ou contrato que deve ser mantido]
- **PRES-002**: [comportamento externo que não pode mudar]

### Estratégia de Execução

[Ordem das etapas do refactor, quando há dependências entre elas.]

1. [Primeiro passo]
2. [Segundo passo — depende do anterior]
3. [Terceiro passo]

### Métricas Atuais vs. Alvo *(se disponíveis)*

| Métrica | Estado Atual | Estado Alvo |
|---------|-------------|-------------|
| [ex.: cobertura de testes] | [%] | [%] |
| [ex.: número de dependências] | [n] | [n] |

### Restrições Técnicas

- **CON-001**: [restrição]

### Critérios de Aceitação

- **AC-001**: Todos os testes existentes passam sem alteração de comportamento
- **AC-002**: [Critério técnico de melhoria]
- **AC-003**: A interface pública de [componente] permanece compatível com consumidores existentes

### Questões em Aberto

- `⚠️ requer definição`: [questão pendente]

### Referências

- [Título](url-ou-caminho) — [descrição]
```

---

## Boas Práticas ao Conduzir a Entrevista

- Faça **uma ou duas perguntas por vez** — conduza uma conversa, não aplique um formulário.
- Se a resposta cobrir espontaneamente um tópico futuro, **não pergunte de novo** — use o que já foi dito.
- Se uma pergunta não se aplica ao contexto (ex.: sem API para um refactor interno), **pule-a**.
- Sinalize claramente com `⚠️ requer definição` as seções sem informação suficiente — nunca invente dados técnicos.
- A Parte 1 deve ser legível por alguém sem contexto técnico. Se uma frase exigir conhecimento de engenharia para ser entendida, reescreva.
- A Parte 2 deve ter precisão suficiente para que a `create-specification` gere specs sem ambiguidade.
- Se `ENGINEERING_STANDARDS.md` foi lido, use o contexto da stack para fazer perguntas técnicas mais precisas no Round 2 (ex.: já saber o banco de dados ou o framework evita perguntas óbvias).
