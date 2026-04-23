# ARCHITECTURE.md — PSCC Team

---

## Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 21 |
| Framework | Spring Boot 3.x |
| Banco de Dados | Oracle |
| Acesso a Dados | `NamedParameterJdbcTemplate` (JDBC puro) |
| Build | Maven |
| Configuração | `application.properties` |

---

## Estrutura de Pacotes

Como cada microsserviço representa um domínio, a estrutura de pacotes é direta — sem quebra por domínio interno:

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

### Responsabilidades

| Pacote | Responsabilidade |
|---|---|
| `controller` | Recebe a requisição HTTP e delega ao service |
| `controller/request` | DTOs de entrada da API |
| `controller/response` | DTOs de saída da API |
| `controller/api` | Contrato/interface da API exposta com anotações OpenAPI |
| `service` | Lógica de negócio |
| `model` | Representação dos dados e entidades do domínio |
| `model/builder` | Builders para models com construção complexa |
| `helper` | Lógica auxiliar reutilizável sem acoplamento de negócio |
| `repository` | Acesso ao banco de dados |
| `repository/queries` | Constantes SQL |
| `repository/mapper` | Mapeamento entre `ResultSet` e model |
| `listener` | Consumo de eventos (Kafka, fila, etc.) |
| `publisher` | Publicação de eventos |

---

## Tratamento de Erros

### Formato de resposta de erro

Toda API retorna erros no mesmo formato:

```json
{
  "messageError": "CUSTOMER_CLOSED_FRAUD",
  "code": 2,
  "message": "Cliente fechado por fraude",
  "moment": "2026-04-22T14:30:00"
}
```

### Mapeamento de HTTP status

| Situação | HTTP Status |
|---|---|
| Erro de negócio | `422 Unprocessable Entity` |
| Recurso não encontrado | `404 Not Found` |
| Erro de validação do request | `400 Bad Request` |
| Erro inesperado | `500 Internal Server Error` |

### Interface `ApiError`

Todos os enums de erro implementam a interface `ApiError`, permitindo que `BusinessException`, `NotFoundException` e o `GlobalExceptionHandler` funcionem independente de qual enum está sendo usado.

```java
public interface ApiError {
    int getCode();
    String getMessage();
    String name();
}
```

### Enums de erro

Cada API define seus próprios enums por contexto. Os códigos são **incrementais por serviço** — não por enum. Antes de adicionar um novo código, verificar se ele já está em uso em algum outro enum do mesmo serviço.

```java
public enum CustomerError implements ApiError {

    CUSTOMER_NOT_FOUND(1, "Cliente não encontrado"),
    CUSTOMER_CLOSED_FRAUD(2, "Cliente fechado por fraude");

    private final int code;
    private final String message;

    CustomerError(int code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override public int getCode() { return code; }
    @Override public String getMessage() { return message; }
}

public enum ProductError implements ApiError {

    PRODUCT_NOT_AVAILABLE(3, "Produto não disponível"),
    PRODUCT_NOT_FOUND(4, "Produto não encontrado");

    private final int code;
    private final String message;

    ProductError(int code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override public int getCode() { return code; }
    @Override public String getMessage() { return message; }
}
```

### Classe de resposta de erro

```java
@Builder
public record ErrorResponse(
        String messageError,
        int code,
        String message,
        LocalDateTime moment
) {
    public static ErrorResponse of(ApiError error) {
        return ErrorResponse.builder()
                .messageError(error.name())
                .code(error.getCode())
                .message(error.getMessage())
                .moment(LocalDateTime.now())
                .build();
    }
}
```

### Exceções de domínio

```java
public class BusinessException extends RuntimeException {

    private final ApiError error;

    public BusinessException(ApiError error) {
        super(error.getMessage());
        this.error = error;
    }

    public ApiError getError() { return error; }
}

public class NotFoundException extends RuntimeException {

    private final ApiError error;

    public NotFoundException(ApiError error) {
        super(error.getMessage());
        this.error = error;
    }

    public ApiError getError() { return error; }
}
```

### GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException ex) {
        log.warn("Erro de negócio: {}", ex.getError().name());
        return ResponseEntity.unprocessableEntity()
                .body(ErrorResponse.of(ex.getError()));
    }

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(NotFoundException ex) {
        log.warn("Recurso não encontrado: {}", ex.getError().name());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ErrorResponse.of(ex.getError()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        log.warn("Erro de validação: {}", ex.getMessage());
        return ResponseEntity.badRequest()
                .body(ErrorResponse.builder()
                        .messageError("VALIDATION_ERROR")
                        .code(0)
                        .message(ex.getBindingResult().getFieldErrors().stream()
                                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                                .collect(Collectors.joining(", ")))
                        .moment(LocalDateTime.now())
                        .build());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Erro inesperado", ex);
        return ResponseEntity.internalServerError()
                .body(ErrorResponse.builder()
                        .messageError("INTERNAL_ERROR")
                        .code(0)
                        .message("Erro interno. Tente novamente.")
                        .moment(LocalDateTime.now())
                        .build());
    }
}
```

---

## Model

Classes do pacote `model` representam qualquer estrutura de dados interna do serviço — retorno de queries, filtros, agregados para insert ou update, etc. Não existem classes genéricas de entidade. Cada model é nomeado pelo contexto ou operação que representa.

```
model/
├── CustomerEligibility
├── Product
├── EligibilityFilter
└── builder/
    ├── CustomerEligibilityBuilder
    └── EligibilityFilterBuilder
```

O sub-pacote `builder/` contém os builders para models que possuem construção complexa, seguindo o Builder Pattern.

```java
// Model simples
public class CustomerEligibility {
    private Long id;
    private String status;
    private String riskProfile;
}

// Model com builder para construção complexa
public class EligibilityFilter {
    private Long customerId;
    private String productId;
    private String operationType;
    private LocalDate referenceDate;

    private EligibilityFilter() {}

    public static EligibilityFilterBuilder builder() {
        return new EligibilityFilterBuilder();
    }
}
```

Regras:
- Nomeado pelo contexto: `CustomerEligibility`, não `Customer`
- Usar builder quando o model tem muitos campos ou campos opcionais
- Um model por contexto/operação — não reutilizar models entre operações distintas

---

## Request e Response

Classes dos pacotes `controller/request` e `controller/response` representam o contrato da API — o que entra e o que sai do Controller. Não usar a nomenclatura "DTO".

Nomeadas pela operação:

```
controller/
├── request/
│   ├── CreateCustomerRequest
│   └── CheckEligibilityRequest
└── response/
    ├── CreateCustomerResponse
    ├── FindCustomerResponse
    └── ProductEligibilityResponse
```

Regras:
- Uma classe por operação — não reutilizar Request ou Response entre endpoints diferentes
- Response reflete apenas o que o cliente precisa receber, não o que o model contém
- Request contém apenas os campos necessários para executar a operação

---

## Controller

### Pacote controller/api

O pacote `controller/api` contém a interface que o controller implementa, com todas as anotações OpenAPI/Swagger. Isso separa o contrato da API da implementação.

```java
// controller/api/EligibilityApi.java
@Tag(name = "Elegibilidade", description = "Operações de elegibilidade de clientes")
public interface EligibilityApi {

    @Operation(summary = "Verifica elegibilidade do cliente")
    @ApiResponse(responseCode = "200", description = "Elegibilidade verificada com sucesso")
    @ApiResponse(responseCode = "400", description = "Erro de validação do request")
    @ApiResponse(responseCode = "404", description = "Cliente ou produto não encontrado")
    @ApiResponse(responseCode = "422", description = "Erro de negócio")
    @PostMapping("/eligibility")
    ResponseEntity<ProductEligibilityResponse> check(@Valid @RequestBody CheckEligibilityRequest request);
}

// controller/EligibilityController.java
@RestController
@RequiredArgsConstructor
@Slf4j
public class EligibilityController implements EligibilityApi {

    private final EligibilityService eligibilityService;

    @Override
    public ResponseEntity<ProductEligibilityResponse> check(@Valid @RequestBody CheckEligibilityRequest request) {
        log.info("Requisição recebida: {}", request);
        return ResponseEntity.ok(eligibilityService.check(request));
    }
}
```

Regras:
- Controller implementa a interface do pacote `api`
- Anotações OpenAPI ficam **somente na interface** — nunca no controller
- `@RestController`, `@RequiredArgsConstructor` e `@Slf4j` ficam no controller, não na interface
- O log de entrada fica no controller

---

## Repository

**1. `RowMapper` sempre em classe separada**

```java
// ❌ Errado — mapper inline
jdbc.query(sql, params, (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("name")));

// ✅ Correto — classe separada no pacote repository/mapper
@Component
public class CustomerMapper implements RowMapper<Customer> {
    @Override
    public Customer mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Customer(rs.getLong("id"), rs.getString("name"));
    }
}
```

**2. SQL em constantes nomeadas**

```java
// ❌ Errado — SQL inline no método
jdbc.query("SELECT id, status FROM customers WHERE id = :customerId", params, rowMapper);

// ✅ Correto — SQL em classe de constantes no pacote repository/queries
public final class CustomerQueries {

    private CustomerQueries() {}

    static final String FIND_BY_ID = """
            SELECT id, status
            FROM customers
            WHERE id = :customerId
            """;
}
```

**3. `@Transactional` nunca no Repository**

Controle de transação é responsabilidade do Service. O Repository apenas executa queries.

```java
// ❌ Errado
@Transactional
public Optional<Customer> findById(Long id) { ... }

// ✅ Correto — @Transactional no Service
@Transactional(readOnly = true)
public EligibilityResponse check(EligibilityRequest request) {
    var customer = customerRepository.findById(request.customerId());
    ...
}
```
