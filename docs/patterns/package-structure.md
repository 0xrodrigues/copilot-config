# Estrutura de Pacotes — PSCC Team

Como cada microsserviço representa um único domínio, a estrutura de pacotes é direta — sem quebra por domínio interno.

```
com.nubank.{servico}/
├── controller/
│   ├── request/
│   ├── response/
│   └── api/
├── service/
├── model/
│   └── builder/
├── helper/
├── repository/
│   ├── queries/
│   └── mapper/
├── listener/       (se aplicável)
└── publisher/      (se aplicável)
```

## Responsabilidades

| Pacote | Responsabilidade |
|---|---|
| `controller` | Recebe a requisição HTTP e delega ao service |
| `controller/request` | Contratos de entrada da API |
| `controller/response` | Contratos de saída da API |
| `controller/api` | Interface da API com anotações OpenAPI |
| `service` | Lógica de negócio |
| `model` | Representação dos dados e entidades do domínio |
| `model/builder` | Builders para models com construção complexa |
| `helper` | Lógica auxiliar reutilizável sem acoplamento de negócio |
| `repository` | Acesso ao banco de dados |
| `repository/queries` | Constantes SQL |
| `repository/mapper` | Mapeamento entre `ResultSet` e model |
| `listener` | Consumo de eventos (Kafka) |
| `publisher` | Publicação de eventos |

## Convenção de Nomeação de Classes

| Pacote | Sufixo | Exemplo |
|---|---|---|
| `controller` | `Controller` | `EligibilityController` |
| `controller/api` | `Api` | `EligibilityApi` |
| `controller/request` | `Request` | `CheckEligibilityRequest` |
| `controller/response` | `Response` | `ProductEligibilityResponse` |
| `service` | `Service` | `EligibilityService` |
| `model` | sem sufixo | `CustomerEligibility` |
| `model/builder` | `Builder` | `EligibilityFilterBuilder` |
| `helper` | `Helper` | `CustomerValidationHelper` |
| `repository` | `Repository` | `EligibilityRepository` |
| `repository/queries` | `Queries` | `EligibilityQueries` |
| `repository/mapper` | `Mapper` | `CustomerEligibilityMapper` |
| `listener` | `Listener` | `EligibilityEventListener` |
| `publisher` | `Publisher` | `EligibilityEventPublisher` |

Nunca criar interfaces para implementações únicas. Se há apenas um `EligibilityService`, não existe `EligibilityServiceImpl`.
