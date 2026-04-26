# Stack — PSCC Team

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 21 |
| Framework | Spring Boot 3.x |
| Banco de Dados | Oracle |
| Acesso a Dados | `NamedParameterJdbcTemplate` (JDBC puro) |
| Build | Maven |
| Configuração | `application.properties` |

Sem JPA, sem Hibernate, sem Spring Data. Acesso a dados é feito exclusivamente via JDBC com `NamedParameterJdbcTemplate`.
