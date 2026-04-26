# AGENTS.md — PSCC Team

Este arquivo é o ponto de entrada. Não contém padrões — aponta para onde eles estão.
Leia apenas os documentos relevantes para a tarefa em execução.

---

## Visão Geral do Stack

→ [`patterns/stack.md`](patterns/stack.md)
Tecnologias, versões e ferramentas. Leia antes de qualquer implementação.

---

## Estrutura de Pacotes

→ [`patterns/package-structure.md`](patterns/package-structure.md)
Organização de diretórios, responsabilidade de cada pacote e regras de nomeação de classes.

---

## Convenções de Código

→ [`patterns/conventions/language.md`](patterns/conventions/language.md)
Idioma do código (EN) vs documentação e logs (PT).

→ [`patterns/conventions/naming.md`](patterns/conventions/naming.md)
Nomes de classes, métodos, variáveis e booleanos. Clean code e DRY.

→ [`patterns/conventions/lombok.md`](patterns/conventions/lombok.md)
Quais anotações usar, quando e o que nunca usar.

→ [`patterns/conventions/records-vs-classes.md`](patterns/conventions/records-vs-classes.md)
Quando usar `record` e quando usar `class`.

→ [`patterns/conventions/configuration.md`](patterns/conventions/configuration.md)
Padrões de `application.properties` e variáveis de ambiente.

---

## Camadas

→ [`patterns/layers/controller.md`](patterns/layers/controller.md)
Padrões de Controller, interface `api/`, ResponseEntity e validação de entrada.

→ [`patterns/layers/service.md`](patterns/layers/service.md)
Responsabilidades do Service e uso de `@Transactional`.

→ [`patterns/layers/repository.md`](patterns/layers/repository.md)
JDBC com `NamedParameterJdbcTemplate`, queries SQL, RowMapper e retorno com `Optional`.

→ [`patterns/layers/model.md`](patterns/layers/model.md)
Estrutura de Model, builders e regras de nomeação.

---

## Tratamento de Erros

→ [`patterns/error-handling.md`](patterns/error-handling.md)
Interface `ApiError`, enums de erro, exceções de domínio, `GlobalExceptionHandler` e mapeamento HTTP.

---

## Logs e Observabilidade

→ [`patterns/logging.md`](patterns/logging.md)
Pontos obrigatórios de log, níveis e exemplos por camada.

---

## Testes

→ [`patterns/testing.md`](patterns/testing.md)
Padrões de teste unitário e de integração, estrutura e convenções de nomenclatura.

---

## Decisões Arquiteturais (ADRs)

→ [`decisions/`](decisions/)
Registro de decisões tomadas pelo time. Consulte antes de propor mudanças estruturais.
