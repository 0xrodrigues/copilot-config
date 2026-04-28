---
name: research-task
description: Lê o TASK.md de uma demanda, explora o projeto em profundidade para identificar arquivos, classes, métodos e linhas exatas relevantes, impactos, riscos e pontos de integração, e gera o CONTEXT.md. Usar após create-task e antes de create-specs, ou quando o usuário menciona "fazer research", "explorar o projeto", "mapear impactos", "criar CONTEXT.md" ou "onde preciso alterar".
---

# Research do Projeto

Lê o `TASK.md`, explora o projeto em profundidade e produz o `CONTEXT.md` — o mapa completo de onde e como a tarefa será implementada, com classes, métodos, linhas exatas, riscos, impactos e pontos de atenção.

O `CONTEXT.md` é a entrada para a skill `create-specs`. Quanto mais preciso, mais certeiras serão as especificações.

---

## Etapa 1 — Localizar o TASK.md

Se o usuário informou um caminho, use-o diretamente.

Caso contrário, procure por `TASK.md` em:
- Raiz do projeto
- `/specs/` recursivamente

Se houver mais de um `TASK.md`, liste-os e peça ao usuário para escolher antes de continuar.

Leia o arquivo completamente. Extraia e memorize: `type`, `size`, `name`, escopo (o que será e o que não será feito), requisitos funcionais, regras de negócio e critérios de aceitação.

---

## Etapa 2 — Mapa de Padrões do Time

Leia `docs/AGENTS.md` se existir — use como índice dos padrões do time. Não leia todos os arquivos referenciados agora; navegue até os específicos conforme a exploração revelar necessidade.

Se `docs/AGENTS.md` não existir, prossiga e registre essa ausência nas lacunas do `CONTEXT.md`.

---

## Etapa 3 — Exploração do Projeto

Esta é a etapa principal. Explore o projeto com profundidade suficiente para responder com precisão: **onde** a tarefa deve ser implementada e **o que** será impactado.

### 3.1 Módulos e arquivos relevantes

Baseado no escopo e requisitos do `TASK.md`, identifique:
- Diretórios e pacotes diretamente afetados
- Arquivos que precisarão ser **criados**
- Arquivos que precisarão ser **modificados** — com justificativa e linhas de interesse

### 3.2 Classes, métodos e linhas

Esta é a parte mais importante da exploração. Para cada arquivo relevante, **leia o arquivo** e mapeie com precisão:

- **Classe** — nome, linha exata de declaração e responsabilidade atual
- **Métodos a modificar** — nome, assinatura completa, **linha exata de declaração** e o que precisa mudar
- **Métodos a criar** — nome, assinatura esperada e responsabilidade
- **Pontos de interesse** — linhas onde há lógica diretamente relevante para a task: condicionais de negócio, chamadas a serviços externos, mapeamentos, validações, configurações

> **Regra:** leia os arquivos — nunca infira ou estime números de linha. Registre apenas o que foi verificado diretamente.

### 3.3 Contratos e interfaces públicas

Identifique:
- Endpoints REST expostos que serão criados ou modificados (path, método HTTP, request/response)
- Eventos Kafka publicados ou consumidos — tópico, payload esperado
- Interfaces e abstrações que outros módulos dependem

### 3.4 Dependências e integrações

Identifique:
- Serviços internos ou externos chamados pela funcionalidade
- Clients HTTP, SDKs ou integrações de terceiros envolvidos
- Tabelas de banco de dados lidas ou escritas — inclua colunas relevantes se o schema for acessível
- Migrations necessárias

### 3.5 Impactos

Rastreie o que pode ser afetado **além** do escopo direto:
- Outros módulos que consomem as classes ou interfaces que serão alteradas
- Testes existentes que precisarão ser atualizados ou que podem quebrar
- Configurações, feature flags ou propriedades de ambiente impactadas

### 3.6 Riscos

Com base em tudo que encontrou, identifique:
- Acoplamentos que tornam a mudança arriscada
- Comportamentos existentes que podem ser alterados inadvertidamente
- Dados em produção que podem ser afetados (migrations, transformações)
- Complexidade ou incerteza que aumenta a chance de retrabalho

### 3.7 Padrões aplicáveis

Para os módulos identificados, verifique quais arquivos de `docs/patterns/` se aplicam:
- Padrão da camada correspondente (controller, service, repository)
- Padrão de testes — `docs/patterns/testing.md`
- Padrão de log — obrigatório para `feature` e `bug` (`docs/patterns/logging.md`)
- Convenção de idioma — `docs/patterns/conventions/language.md`
- Stack e estrutura de pacotes — `docs/patterns/stack.md`, `docs/patterns/package-structure.md`

---

## Etapa 4 — Gerar CONTEXT.md

Crie `/specs/{tipo}/{nome}/CONTEXT.md` com tudo que foi encontrado na exploração:

~~~md
---
task: [nome do trabalho]
task_file: ./TASK.md
date_created: [YYYY-MM-DD]
---

# Contexto de Implementação: [Nome do Trabalho]

## Módulos e Arquivos

### Criar
| Arquivo | Responsabilidade |
|---------|-----------------|
| [caminho] | [o que este arquivo vai fazer] |

### Modificar
| Arquivo | Linhas de interesse | Motivo da modificação |
|---------|--------------------|-----------------------|
| [caminho] | L42, L87–L95 | [o que precisa mudar e por quê] |

## Classes e Métodos

### [NomeClasse] — `[caminho/do/arquivo]` — declaração: L[N]

**Métodos a modificar:**
- `nomeDoMetodo(params)` — L[N] — [comportamento atual] → [o que precisa mudar]

**Métodos a criar:**
- `nomeDoMetodo(params): ReturnType` — [responsabilidade esperada]

**Pontos de interesse:**
- L[N] — [lógica relevante: condicional de negócio, chamada externa, mapeamento, validação]

> Repita este bloco para cada classe relevante. Apenas linhas verificadas por leitura direta do arquivo.

## Contratos e Interfaces Públicas

[Endpoints REST, eventos Kafka, interfaces e contratos que serão criados ou alterados. Inclua assinaturas conhecidas.]

## Dependências e Integrações

[Serviços, clients, tabelas e schemas envolvidos. Inclua nomes de tabelas e colunas relevantes quando acessíveis.]

## Impactos

[O que pode ser afetado além do escopo direto — consumidores das interfaces alteradas, testes que podem quebrar, configurações impactadas.]

## Riscos

[Acoplamentos perigosos, comportamentos que podem mudar inadvertidamente, dados em produção afetados, incertezas que aumentam o risco de retrabalho.]

## Padrões a Seguir

[Liste os arquivos de `docs/patterns/` aplicáveis com uma frase de relevância para esta tarefa. Não resuma o conteúdo — apenas referencie para que o agente saiba quais abrir durante a implementação.]

## Lacunas e Pontos de Atenção

[O que não foi encontrado no código. Ambiguidades que podem gerar dúvidas na implementação. Decisões que precisarão ser tomadas em runtime. Sinalize com clareza para que apareçam nas specs.]
~~~

---

## Etapa 5 — Confirmação

Após criar o `CONTEXT.md`, informe ao usuário:
- Caminho do arquivo gerado
- Quantos arquivos e classes foram mapeados
- Riscos críticos identificados (se houver)
- Lacunas que precisam de decisão humana antes de gerar as specs

Oriente o usuário a rodar a skill `create-specs` para gerar as especificações técnicas a partir do `TASK.md` e do `CONTEXT.md`.

---

## Boas Práticas

- **Leia os arquivos — nunca estime linhas.** Números de linha só entram no `CONTEXT.md` após leitura direta do arquivo.
- Seja específico: nomes reais de classes, métodos e arquivos — nunca genéricos.
- Registre pontos de interesse mesmo que não sejam alterados — eles orientam o desenvolvedor sobre o contexto ao redor da mudança.
- Lacunas são informação valiosa. Registre tudo que está ausente ou incerto.
- Não gere o `CONTEXT.md` com informações insuficientes — explore mais antes.
