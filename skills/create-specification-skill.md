---
name: create-specification
description: 'Lê um PRD e gera um conjunto de especificações prontas para IA, organizadas por tipo de trabalho (feature, bug, refactor), com IMPLEMENTATION.md como índice.'
---

# Criar Especificações a partir de um PRD

Seu objetivo é ler o arquivo `${input:PRDPath}` e gerar todas as especificações necessárias para que a IA possa implementar o trabalho com máximo controle de contexto e rastreabilidade.

## Etapa 1 — Leitura e Análise do PRD

Leia o arquivo informado em `${input:PRDPath}` na íntegra antes de qualquer outra ação.

Identifique:
- O **tipo de trabalho**: `feature`, `bug` ou `refactor`
- O **nome do trabalho**: slug em kebab-case que será usado como nome da pasta (ex.: `customer-eligibility`, `fix-payment-timeout`, `extract-auth-service`)
- As **responsabilidades distintas** que precisam de specs separadas

> Uma spec por responsabilidade. Se duas responsabilidades compartilham o mesmo contrato de dados e critérios de aceitação, podem ser agrupadas. Se não, devem ser separadas.

## Etapa 2 — Planejamento (obrigatório antes de criar arquivos)

Antes de criar qualquer arquivo, apresente ao usuário o plano com:

1. O tipo de trabalho identificado (`feature` / `bug` / `refactor`)
2. O caminho base que será criado: `/specs/{tipo}/{nome}/`
3. A lista de specs planejadas no formato:
   - `spec-01-{titulo}.md` — [descrição de uma linha do escopo]
   - `spec-02-{titulo}.md` — [descrição de uma linha do escopo]
4. A estrutura do `IMPLEMENTATION.md` que será gerado

Aguarde a confirmação do usuário antes de prosseguir para a Etapa 3.

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
- **Nome da pasta**: kebab-case, descritivo, sem versão (ex.: `customer-eligibility`, `fix-payment-timeout`)
- **Specs**: `spec-NN-{titulo}.md` onde `NN` é o número sequencial com zero à esquerda (01, 02, 03...)
- **Título**: kebab-case, descritivo do escopo da spec (ex.: `spec-01-eligibility-rules.md`)

---

## IMPLEMENTATION.md — Índice e Registro de Implementação

O `IMPLEMENTATION.md` é um documento vivo. Crie-o junto com as specs, preenchendo as seções de planejamento. As seções de registro serão preenchidas durante e após a implementação.

```md
---
type: [feature | bug | refactor]
name: [nome do trabalho]
prd: [caminho do PRD de origem]
date_created: [YYYY-MM-DD]
status: [planned | in-progress | completed]
---

# [Nome do Trabalho]

[Resumo de uma a três frases do que este trabalho entrega ou corrige, extraído do PRD.]

## Especificações

| # | Arquivo | Escopo | Status |
|---|---------|--------|--------|
| 01 | [spec-01-{titulo}.md](./spec-01-{titulo}.md) | [escopo de uma linha] | pending |
| 02 | [spec-02-{titulo}.md](./spec-02-{titulo}.md) | [escopo de uma linha] | pending |

> Atualize o campo Status para `in-progress` ao iniciar e `done` ao concluir cada spec.

## Ordem de Implementação Sugerida

[Liste as specs na ordem recomendada de implementação, com justificativa quando a ordem não for óbvia.]

1. `spec-01-{titulo}.md` — [motivo]
2. `spec-02-{titulo}.md` — [depende de spec-01]

## Decisões e Desvios

[Seção preenchida durante a implementação. Registre aqui decisões tomadas em runtime que divergiram do plano original das specs, com justificativa.]

- **[data]**: [decisão ou desvio] — [motivo]

## Resultado Final

[Seção preenchida após a conclusão. Descreva o que foi entregue, o que ficou de fora do escopo e qualquer débito técnico gerado.]
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
tags: [lista de tags relevantes, ex.: `schema`, `process`, `design`]
---

# Introdução

[Breve introdução à especificação e ao objetivo que ela pretende atingir, em relação ao PRD de origem.]

## 1. Propósito & Escopo

[Descrição clara do que esta spec cobre e do que está fora do escopo. Indique o público-alvo e as premissas assumidas.]

## 2. Definições

[Liste e defina siglas, abreviações e termos específicos do domínio usados nesta spec.]

## 3. Requisitos, Restrições & Diretrizes

[Liste explicitamente todos os requisitos, restrições, regras e diretrizes. Use marcadores ou tabelas para maior clareza.]

- **REQ-001**: Requisito funcional
- **SEC-001**: Requisito de segurança
- **CON-001**: Restrição técnica ou de negócio
- **GUD-001**: Diretriz recomendada
- **PAT-001**: Padrão a seguir

## 4. Interfaces & Contratos de Dados

[Descreva interfaces, APIs, contratos de dados e pontos de integração. Use tabelas ou blocos de código para esquemas e exemplos.]

## 5. Critérios de Aceitação

[Critérios claros e testáveis usando o formato Dado-Quando-Então.]

- **AC-001**: Dado [contexto], Quando [ação], Então [resultado esperado]
- **AC-002**: O sistema deve [comportamento] quando [condição]

## 6. Estratégia de Automação de Testes

[Defina a abordagem de testes adequada à stack do projeto — não copie exemplos de outras stacks.]

- **Níveis de Teste**: Unitário, Integração, Ponta a Ponta
- **Frameworks**: [frameworks compatíveis com a stack do projeto]
- **Gestão de Dados de Teste**: [criação e limpeza de dados de teste]
- **Integração CI/CD**: [pipeline de CI/CD do projeto]
- **Requisitos de Cobertura**: [limites mínimos de cobertura]
- **Testes de Performance**: [abordagem para carga e desempenho, se aplicável]

## 7. Justificativa & Contexto

[Explique o raciocínio por trás dos requisitos e decisões de design. Referencie o PRD quando relevante.]

## 8. Dependências & Integrações Externas

[Foque no **o quê** é necessário, não no **como** será implementado. Evite versões de pacotes exceto quando forem restrições arquiteturais.]

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

[Use a linguagem real do projeto. Demonstre ao menos um caso principal e um caso extremo ou de erro.]

```plaintext
// Substitua pelo exemplo real na linguagem do projeto (ex.: typescript, python, json, yaml, sql).
// Caso principal:
//   <exemplo>
//
// Caso extremo ou de erro:
//   <comportamento esperado>
```

## 10. Critérios de Validação

[Critérios que devem ser satisfeitos para que esta spec seja considerada implementada corretamente.]

## 11. Especificações Relacionadas / Leitura Adicional

[Specs internas pelo caminho relativo; documentação externa pela URL completa.]

- [Título da Spec Relacionada](./spec-NN-{titulo}.md) — [descrição do relacionamento]
- [Título da Documentação Externa](https://exemplo.com/docs) — [o que pode ser encontrado]
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

[Descrição objetiva do bug: o que está acontecendo de errado e qual o impacto para o usuário ou sistema.]

## 1. Propósito & Escopo

[O que esta spec cobre para corrigir o bug. Delimite o que está e o que não está no escopo desta correção.]

## 2. Definições

[Liste e defina termos específicos do domínio usados nesta spec.]

## 3. Causa Raiz

[Descreva a causa raiz identificada ou hipótese mais provável. Inclua referências a arquivos, funções ou fluxos envolvidos.]

- **Componente afetado**: [nome do módulo, serviço ou camada]
- **Causa identificada**: [descrição técnica da causa]
- **Hipótese alternativa**: [se houver, liste outras causas possíveis]

## 4. Comportamento Atual vs. Esperado

| Aspecto | Comportamento Atual | Comportamento Esperado |
|---------|--------------------|-----------------------|
| [ex.: resposta da API] | [o que acontece] | [o que deveria acontecer] |
| [ex.: estado do banco] | [o que acontece] | [o que deveria acontecer] |

## 5. Passos de Reprodução

[Liste os passos exatos para reproduzir o bug. Inclua pré-condições, dados de entrada e ambiente.]

**Pré-condições**: [estado inicial necessário]

1. [Passo 1]
2. [Passo 2]
3. [Resultado observado]

**Ambiente**: [produção / staging / local — versão, configuração relevante]

## 6. Requisitos da Correção

[Liste o que a correção deve satisfazer, incluindo restrições para não introduzir regressões.]

- **REQ-001**: A correção deve [comportamento esperado]
- **CON-001**: A correção não deve alterar [contrato / interface existente]
- **SEC-001**: [Requisito de segurança se o bug tiver implicação de segurança]

## 7. Interfaces & Contratos de Dados Afetados

[Descreva os contratos de dados, APIs ou interfaces impactados pela correção.]

## 8. Critérios de Aceitação

[Critérios que provam que o bug foi corrigido e nenhuma regressão foi introduzida.]

- **AC-001**: Dado [contexto de reprodução], Quando [ação], Então [comportamento esperado, não o bugado]
- **AC-002**: Os cenários existentes de [funcionalidade relacionada] continuam funcionando corretamente

## 9. Estratégia de Testes

- **Teste de regressão**: [como garantir que o bug não volta]
- **Frameworks**: [frameworks compatíveis com a stack do projeto]
- **Cenários a cobrir**: [lista de cenários críticos a testar]

## 10. Dependências & Contexto Técnico

[Componentes, serviços ou configurações que precisam ser considerados na correção.]

- **DEP-001**: [Dependência ou componente relacionado] - [como impacta a correção]

## 11. Exemplos & Dados de Teste

```plaintext
// Dados ou payloads que reproduzem o bug.
// Entrada que causa o bug:
//   <exemplo>
//
// Entrada que deve funcionar corretamente após a correção:
//   <exemplo>
```

## 12. Especificações Relacionadas / Leitura Adicional

- [Spec Relacionada](./spec-NN-{titulo}.md) — [descrição do relacionamento]
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

[Descrição objetiva do que será refatorado e qual o ganho esperado: legibilidade, performance, manutenibilidade, separação de responsabilidades, etc.]

## 1. Propósito & Escopo

[O que será refatorado e o que não será tocado neste trabalho. Seja explícito sobre os limites para evitar scope creep.]

**Em escopo**: [lista do que será refatorado]
**Fora de escopo**: [lista do que não será alterado neste refactor]

## 2. Definições

[Liste e defina termos específicos usados nesta spec.]

## 3. Motivação Técnica

[Explique por que o refactor é necessário agora. Inclua evidências objetivas sempre que possível.]

- **Problema atual**: [descrição do problema técnico — ex.: acoplamento, duplicação, complexidade ciclomática]
- **Impacto do problema**: [ex.: dificuldade de testar, risco de bugs, lentidão de desenvolvimento]
- **Métricas atuais** *(se disponíveis)*: [ex.: cobertura de testes, tempo de build, número de dependências]

## 4. Estado Atual vs. Estado Alvo

| Aspecto | Estado Atual | Estado Alvo |
|---------|-------------|-------------|
| [ex.: estrutura de módulos] | [como está] | [como ficará] |
| [ex.: cobertura de testes] | [% atual] | [% alvo] |
| [ex.: número de responsabilidades] | [quantidade] | [quantidade] |

## 5. Requisitos & Restrições

[O que o refactor deve garantir. Especialmente importante: o que NÃO pode mudar.]

- **REQ-001**: O comportamento externo observável deve permanecer idêntico após o refactor
- **REQ-002**: [Requisito técnico específico do refactor]
- **CON-001**: A interface pública de [componente] não deve ser alterada
- **CON-002**: [Restrição de compatibilidade, se houver]

## 6. Interfaces & Contratos Preservados

[Liste explicitamente os contratos, APIs e interfaces que devem ser preservados sem alteração.]

## 7. Estratégia de Refactor

[Descreva a abordagem técnica. Inclua a ordem de execução quando há dependências entre etapas.]

1. [Primeiro passo — ex.: extrair interface]
2. [Segundo passo — ex.: mover implementação]
3. [Terceiro passo — ex.: atualizar dependentes]

**Riscos identificados**: [ex.: pontos de acoplamento que podem quebrar, dados em produção afetados]

## 8. Estratégia de Testes

[Como garantir que o refactor não quebrou nada.]

- **Abordagem**: [ex.: garantir cobertura antes de refatorar, rodar suite existente a cada etapa]
- **Frameworks**: [frameworks compatíveis com a stack do projeto]
- **Testes de regressão**: [o que deve ser validado ao final]
- **Métricas de melhoria**: [como medir que o refactor atingiu seu objetivo]

## 9. Critérios de Aceitação

- **AC-001**: Todos os testes existentes passam sem modificação de comportamento
- **AC-002**: [Critério técnico de melhoria — ex.: cobertura acima de X%, complexidade reduzida]
- **AC-003**: A interface pública de [componente] permanece compatível com os consumidores existentes

## 10. Dependências & Componentes Afetados

[Liste todos os componentes, serviços ou times que serão impactados pelo refactor.]

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

- [Spec Relacionada](./spec-NN-{titulo}.md) — [descrição do relacionamento]
- [Documentação Externa](https://exemplo.com/docs) — [contexto relevante]
```

---

## Boas Práticas Gerais

- Use linguagem precisa, explícita e sem ambiguidades.
- Distinga claramente entre requisitos, restrições e recomendações.
- Defina todas as siglas e termos específicos do domínio.
- Garanta que cada spec seja autossuficiente — não exija que o leitor consulte outras specs para entender o escopo.
- Cada spec deve cobrir uma única responsabilidade. Se o escopo exigir mais de três grupos de requisitos não relacionados, divida em specs separadas e vincule-as na seção de specs relacionadas.
- Ao referenciar specs internas, use sempre o caminho relativo (`./spec-NN-{titulo}.md`).
