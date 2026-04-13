---
name: create-specification
description: 'Gera especificações prontas para IA a partir de um Design Document ou de uma descrição inline (modo express). Aceita o caminho de um arquivo existente ou inicia uma coleta rápida de informações diretamente na conversa. Compatível com os tipos feature, bug e refactor.'
---

# Criar Especificações

Seu objetivo é gerar todas as especificações necessárias para que a IA possa implementar o trabalho com máximo controle de contexto e rastreabilidade.

Esta skill opera em dois modos:

- **Modo arquivo**: recebe o caminho de um Design Document existente via `${input:DesignDocPath}` e gera as specs a partir dele.
- **Modo express**: quando nenhum caminho é fornecido, coleta as informações mínimas diretamente na conversa e gera as specs sem precisar de arquivo.

---

## Contexto do Projeto

Antes de qualquer outra ação, tente ler o arquivo `ENGINEERING_STANDARDS.md` na raiz do projeto.

- Se existir, extraia a stack tecnológica (linguagem, frameworks, banco de dados, mensageria) e os padrões arquiteturais relevantes.
- Use essas informações para preencher as seções de estratégia de testes e dependências nas specs geradas.
- Se não existir, prossiga normalmente — deixe as seções dependentes de stack com `[definir conforme stack do projeto]`.

---

## Etapa 1 — Identificação do Modo

Verifique o valor de `${input:DesignDocPath}`:

- Se for um caminho de arquivo válido (ex.: `docs/design-docs/features/minha-feature.md`), ative o **Modo Arquivo**.
- Se estiver vazio, em branco, ou o usuário informar que não tem um Design Document (ex.: "não tenho", "express", "-"), ative o **Modo Express**.

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

Se nenhum caminho foi fornecido, inicie a coleta inline. Faça as perguntas abaixo em grupos naturais — não todas de uma vez.

**Identificação:**
- Qual o tipo de trabalho? (`feature`, `bug` ou `refactor`)
- Descreva brevemente o que precisa ser feito.

**Para Feature:**
- Quais são os requisitos funcionais principais?
- Existe alguma restrição técnica ou regra de negócio explícita?
- Como validar que está correto? (critérios de aceitação no formato Dado/Quando/Então)

**Para Bug:**
- Qual o comportamento atual e o esperado?
- Qual componente ou camada está envolvido?
- O que a correção não pode quebrar?
- Como provar que o bug foi corrigido?

**Para Refactor:**
- O que está problemático e qual o estado alvo?
- Quais interfaces ou contratos não podem mudar?
- Como provar que o comportamento externo não mudou?

Após coletar as respostas, prossiga para o Planejamento (Etapa 2) usando as informações coletadas no lugar do Design Document.

---

## Etapa 2 — Análise e Planejamento

Com as informações disponíveis (do arquivo ou da coleta inline), identifique:

- O **tipo de trabalho**: `feature`, `bug` ou `refactor`
- O **nome do trabalho**: slug em kebab-case para o nome da pasta (ex.: `customer-eligibility`, `fix-payment-timeout`, `extract-auth-service`)
- As **responsabilidades distintas** que precisam de specs separadas

> Uma spec por responsabilidade. Se duas responsabilidades compartilham o mesmo contrato de dados e critérios de aceitação, podem ser agrupadas. Se não, devem ser separadas.

Verifique se a pasta `/specs/{tipo}/{nome}/` já existe:

- Se **não existir**, prossiga normalmente.
- Se **já existir**, informe o usuário e pergunte se deseja sobrescrever o conteúdo existente, usar um nome diferente ou cancelar. Aguarde a decisão antes de continuar.

Apresente ao usuário o plano com:

1. O tipo de trabalho identificado
2. O caminho base que será criado: `/specs/{tipo}/{nome}/`
3. A lista de specs planejadas:
   - `spec-01-{titulo}.md` — [descrição de uma linha do escopo]
   - `spec-02-{titulo}.md` — [descrição de uma linha do escopo]
4. A estrutura do `IMPLEMENTATION.md` que será gerado

Aguarde a confirmação do usuário antes de prosseguir para a Etapa 3.

---

## Etapa 3 — Geração dos Arquivos

Após a confirmação, crie todos os arquivos na seguinte estrutura:

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

---

## IMPLEMENTATION.md — Índice e Registro de Implementação

```md
---
type: [feature | bug | refactor]
name: [nome do trabalho]
design_doc: [caminho do Design Document de origem — omitir se modo express]
date_created: [YYYY-MM-DD]
status: [planned | in-progress | completed]
---

# [Nome do Trabalho]

[Resumo de uma a três frases do que este trabalho entrega ou corrige.]

## Especificações

| # | Arquivo | Escopo | Status |
|---|---------|--------|--------|
| 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending |
| 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending |

> Atualize o campo Status para `in-progress` ao iniciar e `done` ao concluir cada spec.

## Ordem de Implementação Sugerida

1. `spec-01-{titulo}.md` — [motivo]
2. `spec-02-{titulo}.md` — [depende de spec-01]

## Decisões e Desvios

[Registre aqui decisões tomadas em runtime que divergiram do plano original, com justificativa.]

- **[data]**: [decisão ou desvio] — [motivo]

## Resultado Final

[Preenchido após a conclusão. O que foi entregue, o que ficou fora do escopo e débitos técnicos gerados.]
```

---

## Templates de Especificação por Tipo de Trabalho

### Template: Feature

```md
---
title: [Título Conciso Descrevendo o Foco da Especificação]
type: feature
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
```

---

### Template: Bug

```md
---
title: [Título Conciso Descrevendo o Bug]
type: bug
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
```

---

### Template: Refactor

```md
---
title: [Título Conciso Descrevendo o Refactor]
type: refactor
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
```

---

## Boas Práticas Gerais

- Use linguagem precisa, explícita e sem ambiguidades.
- Distinga claramente entre requisitos, restrições e recomendações.
- Defina todas as siglas e termos específicos do domínio.
- Garanta que cada spec seja autossuficiente — não exija que o leitor consulte outras specs para entender o escopo.
- Cada spec deve cobrir uma única responsabilidade. Se o escopo exigir mais de três grupos de requisitos não relacionados, divida em specs separadas.
- Ao referenciar specs internas, use sempre o caminho relativo (`./spec-NN-{titulo}.md`).
- Se o Design Document de origem contiver `⚠️ requer definição`, sinalize a lacuna na spec — nunca preencha com suposições.
- No modo express, se as respostas coletadas forem insuficientes para gerar critérios de aceitação testáveis, pergunte antes de gerar — specs sem AC são inutilizáveis para implementação.
