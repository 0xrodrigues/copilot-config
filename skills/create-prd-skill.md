---
name: create-prd
description: 'Cria um PRD estruturado para feature, bug ou refactor a partir de uma descrição livre do usuário. O PRD gerado é compatível com a skill create-specification.'
---

# Criar PRD

Seu objetivo é criar um PRD (Product Requirements Document) bem estruturado com base nas informações fornecidas pelo usuário.

- **Tipo de trabalho**: `${input:PRDType}` — deve ser `feature`, `bug` ou `refactor`
- **Descrição**: `${input:Description}` — descrição livre do que o usuário deseja

## Etapa 1 — Validação do Tipo

Verifique se `${input:PRDType}` é um dos valores válidos: `feature`, `bug` ou `refactor`.

- Se for válido, prossiga para a Etapa 2 (leitura do contexto do projeto).
- Se não for válido ou estiver ambíguo, informe o usuário e peça para escolher entre `feature`, `bug` ou `refactor` antes de continuar.

## Etapa 2 — Leitura do Contexto do Projeto

Antes de analisar a descrição do usuário, leia o arquivo `ENGINEERING_STANDARDS.md` na raiz do projeto para extrair:

- A stack tecnológica do projeto (linguagem, frameworks, banco de dados, mensageria)
- Padrões arquiteturais e restrições relevantes

Use essas informações para garantir que o PRD gerado seja coerente com o projeto real. **Não invente stack ou restrições técnicas — use apenas o que está documentado.**

Se o `ENGINEERING_STANDARDS.md` não existir, prossiga sem contexto de stack e deixe as seções técnicas com `⚠️ requer detalhamento`.

## Etapa 3 — Análise da Descrição

Analise `${input:Description}` e extraia:

- O **nome do trabalho**: slug em kebab-case, descritivo e conciso (ex.: `customer-eligibility`, `fix-payment-timeout`, `extract-auth-service`)
- As **informações fornecidas pelo usuário** que preencherão o PRD — **não adicione requisitos, regras de negócio ou restrições que não estejam explícitos na descrição**
- As **lacunas de informação**: seções do template que não podem ser preenchidas com a descrição fornecida

## Etapa 4 — Planejamento (obrigatório antes de criar o arquivo)

Verifique se já existe um arquivo no caminho `/docs/prds/{tipo}/prd-{nome}.md`:

- Se **não existir**, prossiga normalmente.
- Se **já existir**, informe o usuário e pergunte se deseja sobrescrever, renomear ou cancelar. Aguarde a decisão antes de continuar.

Antes de criar o arquivo, apresente ao usuário:

1. O caminho do arquivo que será criado: `/docs/prds/{tipo}/prd-{nome}.md`
2. As seções que serão preenchidas com base na descrição
3. As seções que ficarão com placeholder por falta de informação, sinalizadas com `⚠️ requer detalhamento`

Aguarde a confirmação do usuário antes de prosseguir para a Etapa 5.

## Etapa 5 — Geração do PRD

Após a confirmação, crie o arquivo usando o template correspondente ao tipo informado.

Se o diretório `/docs/prds/{tipo}/` não existir, crie-o antes de gerar o arquivo. Não solicite confirmação adicional para a criação do diretório — ela já foi obtida na Etapa 4.

### Convenção de nomenclatura e diretório

- **Feature**: `/docs/prds/features/prd-{nome}.md`
- **Bug**: `/docs/prds/bugs/prd-{nome}.md`
- **Refactor**: `/docs/prds/refactor/prd-{nome}.md`
- **Nome do arquivo**: `prd-` seguido do slug em kebab-case (ex.: `prd-customer-eligibility.md`)

---

## Templates por Tipo de Trabalho

### Template: Feature

```md
---
type: feature
name: [slug em kebab-case]
date_created: [YYYY-MM-DD]
status: [draft | review | approved]
owner: [Equipe/Indivíduo responsável]
ticket: [Opcional: referência ao ticket no sistema de rastreamento, ex.: GH-123, JIRA-456, LINEAR-789]
---

# PRD — [Feature] [Título Descritivo da Feature]

## Visão Geral

[Descrição objetiva da feature em duas a quatro frases. O que ela entrega, para quem e qual problema resolve.]

## Problema & Motivação

[Descreva o problema atual que justifica a criação desta feature. Inclua o impacto para o usuário ou negócio se o problema não for resolvido.]

## Usuários Afetados

[Quem se beneficia desta feature. Liste os perfis de usuário ou sistemas impactados.]

- **[Perfil 1]**: [como é afetado]
- **[Perfil 2]**: [como é afetado]

## Requisitos Funcionais

[O que a feature deve fazer. Use linguagem direta e mensurável.]

- **RF-001**: [Comportamento esperado do sistema]
- **RF-002**: [Comportamento esperado do sistema]

## Requisitos Não Funcionais

[Restrições de qualidade: performance, segurança, disponibilidade, escalabilidade.]

- **RNF-001**: [Requisito de qualidade]
- **RNF-002**: [Requisito de qualidade]

## Regras de Negócio

[Regras que governam o comportamento da feature. Seja explícito sobre condições, exceções e limites.]

- **RN-001**: [Regra de negócio]
- **RN-002**: [Regra de negócio]

## Fora de Escopo

[O que explicitamente não será entregue nesta feature, para evitar scope creep.]

- [Item fora de escopo]
- [Item fora de escopo]

## Critérios de Sucesso

[Como saberemos que a feature foi bem-sucedida? Métricas, comportamentos observáveis ou KPIs.]

- [Critério mensurável]
- [Critério mensurável]

## Dependências & Riscos

[Sistemas, times ou decisões dos quais esta feature depende. Riscos conhecidos que podem impactar a entrega.]

| Tipo | Descrição | Impacto |
|------|-----------|---------|
| Dependência | [ex.: API de terceiro] | [ex.: bloqueia RF-002] |
| Risco | [ex.: volume de dados] | [ex.: pode impactar RNF-001] |

## Referências

[Links para documentos, tickets, designs ou discussões relacionadas.]

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
owner: [Equipe/Indivíduo responsável]
ticket: [Opcional: referência ao ticket no sistema de rastreamento, ex.: GH-123, JIRA-456, LINEAR-789]
---

# PRD — [Bug] [Título Descritivo do Bug]

## Descrição do Problema

[Descrição objetiva do bug em duas a quatro frases. O que está acontecendo de errado e qual o impacto para o usuário ou sistema.]

## Comportamento Atual

[Descreva exatamente o que acontece hoje. Seja específico — inclua mensagens de erro, estados inesperados ou dados corrompidos se aplicável.]

## Comportamento Esperado

[Descreva o que deveria acontecer. Base de comparação para validar a correção.]

## Passos de Reprodução

[Lista ordenada e precisa para reproduzir o bug. Inclua pré-condições e dados de entrada.]

**Pré-condições**: [estado inicial necessário para reproduzir]

1. [Passo 1]
2. [Passo 2]
3. **Resultado**: [o que acontece de errado]

**Ambiente onde foi reproduzido**: [produção / staging / local — versão, configuração relevante]

## Impacto

[Qual o impacto real do bug? Quantos usuários afeta, qual funcionalidade bloqueia, há risco de perda de dados ou segurança?]

- **Usuários afetados**: [estimativa ou perfil]
- **Funcionalidade bloqueada**: [o que o usuário não consegue fazer]
- **Risco associado**: [ex.: perda de dados, exposição de informação, indisponibilidade]

## Hipótese de Causa Raiz

[Se houver hipótese sobre a causa, descreva aqui. Inclua componentes suspeitos, condições de corrida, dados inválidos ou mudanças recentes que possam ter introduzido o bug.]

- **Componente suspeito**: [módulo, serviço ou camada]
- **Hipótese**: [descrição técnica da causa provável]

## Restrições da Correção

[O que a correção não pode quebrar. Contratos, interfaces ou comportamentos que devem ser preservados.]

- [Interface ou contrato que deve ser mantido]
- [Comportamento relacionado que não pode ser afetado]

## Critérios de Aceitação da Correção

[Como validar que o bug foi corrigido sem introduzir regressões.]

- Dado [contexto de reprodução], Quando [ação], Então [comportamento correto]
- Os cenários existentes de [funcionalidade relacionada] continuam funcionando

## Referências

[Links para tickets, logs, screenshots ou discussões relacionadas.]

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
owner: [Equipe/Indivíduo responsável]
ticket: [Opcional: referência ao ticket no sistema de rastreamento, ex.: GH-123, JIRA-456, LINEAR-789]
---

# PRD — [Refactor] [Título Descritivo do Refactor]

## Descrição do Refactor

[Descrição objetiva do que será refatorado e qual o ganho esperado, em duas a quatro frases.]

## Motivação Técnica

[Por que este refactor é necessário agora? Descreva o problema técnico atual com evidências objetivas sempre que possível.]

- **Problema atual**: [ex.: alto acoplamento, duplicação de lógica, dificuldade de testar]
- **Consequência**: [ex.: risco elevado de bugs, lentidão no desenvolvimento, dificuldade de onboarding]
- **Gatilho**: [o que tornou este refactor prioritário agora]

## Estado Atual

[Descrição do estado atual do código ou sistema. Inclua o que está funcionando, o que está problemático e métricas se disponíveis.]

- **Componentes envolvidos**: [lista dos módulos, serviços ou arquivos afetados]
- **Métricas atuais** *(se disponíveis)*: [ex.: cobertura de testes, complexidade ciclomática, número de dependências]

## Estado Alvo

[Como o código ou sistema deve ficar após o refactor. Seja específico sobre a estrutura desejada.]

- **Estrutura esperada**: [ex.: separação em camadas, extração de serviço, padronização de interfaces]
- **Métricas alvo** *(se aplicável)*: [ex.: cobertura acima de X%, redução de dependências]

## Em Escopo

[O que será alterado neste refactor. Seja explícito para evitar scope creep.]

- [Componente ou comportamento que será refatorado]
- [Componente ou comportamento que será refatorado]

## Fora de Escopo

[O que não será tocado neste refactor, mesmo que relacionado.]

- [Item que não será alterado]
- [Item que não será alterado]

## Restrições & Preservações

[O que absolutamente não pode mudar após o refactor. Contratos, interfaces públicas, comportamentos observáveis.]

- **O comportamento externo observável deve permanecer idêntico**
- [Interface pública que deve ser preservada]
- [Contrato ou integração que não pode ser quebrado]

## Riscos Identificados

[Pontos de atenção que podem complicar o refactor ou gerar impacto inesperado.]

| Risco | Probabilidade | Mitigação |
|-------|--------------|-----------|
| [ex.: componente muito acoplado] | [alta/média/baixa] | [ex.: extrair interface primeiro] |

## Critérios de Sucesso

[Como validar que o refactor atingiu seu objetivo sem quebrar nada.]

- Todos os testes existentes passam sem alteração de comportamento
- [Métrica técnica de melhoria atingida]
- [Interface pública permanece compatível com consumidores existentes]

## Referências

[Links para discussões técnicas, ADRs, tickets ou documentação relacionada.]

- [Título](url-ou-caminho) — [descrição]
```

---

## Boas Práticas ao Gerar o PRD

- Sinalize claramente com `⚠️ requer detalhamento` as seções que não puderam ser preenchidas com base na descrição fornecida.
- Use linguagem direta, sem ambiguidades — o PRD será lido pela skill `create-specification` para gerar specs técnicas.
- O campo `type` no front matter deve sempre corresponder ao `${input:PRDType}` informado.
- O título deve seguir o padrão `# PRD — [Feature|Bug|Refactor] Título` para que a skill `create-specification` identifique o tipo automaticamente.
