---
name: explore-codebase
description: 'Recebe uma pergunta ou cenário em linguagem de negócio e varre o codebase para produzir um mapa técnico completo: classes, fluxos, queries, regras de negócio encontradas no código, glossário de termos e pontos de acoplamento. Salva o resultado em /docs/exploration/{yyyymmddhhmm}/EXPLORATION.md para reuso em sessões futuras e como contexto para o create-specification.'
---

# Explorar Codebase

Seu objetivo é receber uma pergunta ou cenário em linguagem de negócio e produzir um mapa técnico preciso do que está acontecendo no código — sem inventar, sem assumir. Tudo que aparecer no `EXPLORATION.md` deve ter sido encontrado no codebase.

O resultado é salvo em `/docs/exploration/{yyyymmddhhmm}/EXPLORATION.md` e pode ser reutilizado em sessões futuras ou passado como contexto para o `create-specification`.

---

## Etapa 1 — Entender a Pergunta

Leia o argumento recebido. Ele pode ser:

- **Uma pergunta de negócio**: "Como funciona a convivência de informações quando há alteração de taxa?"
- **Um cenário**: "O que acontece quando um pagamento é estornado?"
- **Um componente ou fluxo**: "Me explica o fluxo de emissão de nota fiscal."
- **Um módulo com escopo aberto**: "Quero entender o módulo de conciliação."

Se o argumento estiver vazio, pergunte antes de continuar:
> "Qual cenário ou fluxo você quer explorar? Descreva em linguagem de negócio — pode ser uma pergunta, um processo ou um componente."

Se o argumento for muito amplo (ex.: "me explica o sistema inteiro"), proponha um recorte:
> "O escopo é amplo. Sugiro focar em [sugestão baseada no que foi dito]. Posso explorar isso ou você prefere um recorte diferente?"

Aguarde confirmação antes de continuar.

---

## Etapa 2 — Extrair Termos de Busca

A partir da pergunta, extraia:

1. **Entidades de negócio** — os substantivos principais (ex.: taxa, pagamento, nota fiscal, cliente, contrato)
2. **Ações / verbos de processo** — o que acontece com essas entidades (ex.: alterar, estornar, emitir, conciliar)
3. **Qualificadores** — condições ou estados relevantes (ex.: inadimplente, parcelado, retroativo)

Registre esses termos — eles guiam todas as buscas da Etapa 3.

---

## Etapa 3 — Varredura do Codebase

Execute as buscas em ordem crescente de profundidade. A cada nível, avalie se encontrou o suficiente para responder a pergunta antes de ir ao próximo.

### Nível 1 — Localização inicial

Use grep e find para localizar onde os termos de busca aparecem no código:

- Busque por cada entidade e ação extraída na Etapa 2
- Inclua variações de nomenclatura: camelCase, snake_case, abreviações comuns
- Procure em nomes de classes, métodos, tabelas, tópicos Kafka, endpoints REST
- Inclua arquivos de configuração, migrations e schemas se relevantes para o cenário

Para cada resultado relevante encontrado, anote o caminho exato e o contexto da ocorrência.

### Nível 2 — Leitura dos arquivos relevantes

Para cada arquivo identificado no Nível 1:

- Leia as partes relevantes ao cenário (não o arquivo inteiro se for grande)
- Identifique: responsabilidade da classe/função, como ela se conecta às entidades de negócio, quais dependências ela chama
- Anote métodos específicos que implementam o comportamento descrito na pergunta

### Nível 3 — Rastreamento do fluxo

A partir dos arquivos lidos, siga as chamadas e dependências para mapear o fluxo completo:

- Quem chama o quê, em qual ordem
- Onde decisões de negócio são tomadas (condicionais, regras, validações)
- Onde dados são lidos e escritos (queries, repositórios, chamadas externas)
- Pontos de entrada (controllers, consumers, jobs, listeners)
- Pontos de saída (respostas HTTP, eventos publicados, gravações em banco)

### Nível 4 — Queries e schemas

Se o cenário envolver dados persistidos:

- Identifique as queries relevantes (HQL, SQL, métodos de repositório)
- Mapeie as tabelas e colunas envolvidas
- Note índices, constraints ou triggers relevantes para o cenário
- Identifique se há cache envolvido e em qual camada

### Limite de profundidade

Pare quando conseguir responder à pergunta original com precisão. Não explore além do necessário. Se chegar a um ponto onde o rastro some (ex.: chamada externa, configuração ausente, código gerado), registre explicitamente como lacuna — não invente o que está além.

---

## Etapa 4 — Montar o EXPLORATION.md

Calcule o timestamp no formato `yyyymmddhhmm` usando a data e hora atuais. Crie o diretório `/docs/exploration/{timestamp}/` e escreva o arquivo `EXPLORATION.md` com a seguinte estrutura:

```md
---
question: [pergunta ou cenário original, exatamente como foi informado]
date: [YYYY-MM-DD HH:MM]
codebase_root: [diretório raiz do projeto explorado]
---

# Exploração: [título derivado da pergunta]

[Resumo de duas a quatro frases do que foi encontrado: qual é o fluxo central, quais são as peças principais e o que ficou fora do alcance da exploração.]

---

## Glossário de Termos

> Mapeamento entre o vocabulário de negócio da pergunta e os identificadores reais no código.

| Termo de Negócio | Nome no Código | Tipo | Localização |
|------------------|----------------|------|-------------|
| [termo] | [ClassName / methodName / table_name] | [classe / método / tabela / tópico / endpoint] | [caminho:linha] |

---

## Arquivos Relevantes

| Arquivo | Responsabilidade no contexto desta exploração |
|---------|-----------------------------------------------|
| [caminho/exato/Arquivo.ext] | [o que este arquivo faz em relação ao cenário] |

---

## Fluxo Principal

[Descreva o fluxo em linguagem clara, do ponto de entrada ao ponto de saída. Use numeração para indicar sequência. Para cada passo, indique qual classe/método é responsável e o caminho do arquivo.]

1. **[Ponto de entrada]** — `CaminhoDoArquivo.ext` > `NomeDoMetodo()`
   [O que acontece aqui]

2. **[Próximo passo]** — `CaminhoDoArquivo.ext` > `NomeDoMetodo()`
   [O que acontece aqui]

...

N. **[Ponto de saída]** — `CaminhoDoArquivo.ext`
   [O que é produzido: resposta HTTP, evento publicado, gravação em banco]

---

## Regras de Negócio Encontradas no Código

> Regras identificadas a partir da lógica implementada — não da documentação.

- **RN-001**: [regra] — encontrada em `caminho/Arquivo.ext:linha`
- **RN-002**: [regra] — encontrada em `caminho/Arquivo.ext:linha`

---

## Queries e Dados

> Preencher apenas se o cenário envolver dados persistidos.

### Tabelas Envolvidas

| Tabela | Papel no cenário | Operação (leitura / escrita) |
|--------|-----------------|------------------------------|
| [nome_tabela] | [o que ela guarda relevante ao cenário] | [R / W / RW] |

### Queries Relevantes

```sql
-- [descrição do que esta query faz no contexto do cenário]
-- Localização: caminho/Arquivo.ext:linha
[query ou pseudo-query representativa]
```

---

## Pontos de Acoplamento

> Dependências externas, integrações ou contratos que o fluxo toca.

- **[Nome do serviço / sistema]**: [o que é chamado, quando e por quê] — `caminho/Arquivo.ext`

---

## Lacunas e Pontos Cegos

> O que não foi encontrado ou ficou ambíguo durante a exploração. Nunca omita — lacunas são informação.

- **[Lacuna 1]**: [o que não foi encontrado e por quê — ex.: chamada externa sem código disponível, configuração ausente, código gerado]
- **[Lacuna 2]**: [ambiguidade identificada — dois caminhos possíveis sem evidência de qual é usado]
```

---

## Etapa 5 — Relatório Final

Após salvar o arquivo, informe ao usuário com o resumo da exploração e como usar o resultado:

```
Exploração concluída. Arquivo salvo em `docs/exploration/{timestamp}/EXPLORATION.md`.

**Resumo:**
- [N] arquivos relevantes encontrados
- [N] regras de negócio identificadas no código
- [N] lacunas registradas

**Próximo passo:** Use este arquivo como contexto ao invocar o create-specification:
  /create-specification [ticket] --context docs/exploration/{timestamp}/EXPLORATION.md
```

---

## Boas Práticas

- Tudo que aparecer no `EXPLORATION.md` deve ter sido encontrado no codebase com caminho e linha precisos. Nunca preencha com suposições.
- Se um termo de negócio não tiver correspondência no código, registre como lacuna — não invente um mapeamento.
- O Glossário de Termos é a seção mais crítica para projetos legados: é onde a distância entre linguagem de negócio e nomenclatura do código fica explícita.
- Se o fluxo for muito longo, priorize os nós de decisão de negócio — os pontos onde o comportamento muda com base em regras.
- Explore o suficiente para responder a pergunta. Profundidade excessiva aumenta o custo da sessão sem benefício para quem vai usar o mapa.
- Se durante a exploração aparecer algo inesperado que muda o entendimento do cenário (ex.: um módulo legado completamente separado que também processa o mesmo fluxo), sinalize antes de continuar e pergunte se deve incluí-lo na exploração.
