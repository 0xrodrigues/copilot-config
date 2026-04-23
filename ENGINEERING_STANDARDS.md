# ENGINEERING_STANDARDS.md — PSCC Team

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

## Convenções de Idioma

| Contexto | Idioma |
|---|---|
| Código (classes, métodos, variáveis, constantes) | Inglês |
| Documentação (Javadoc, Markdown, `@DisplayName`) | Português |

---

## Clean Code

### 1. Nomenclatura Significativa

Nomes devem revelar intenção. Quem lê o nome não deve precisar de contexto adicional para entender o que aquilo representa.

```java
// ❌ Errado
int d;
String nm;
List<Object> lst;
boolean flag;

// ✅ Correto
int elapsedDays;
String customerName;
List<Product> eligibleProducts;
boolean isEligible;
```

Regras:
- Sem abreviações — `customerId`, não `custId` ou `cId`
- Sem prefixos ou sufixos genéricos sem contexto — evitar `data`, `info`, `manager`, `util`
- Sem nomes de um caractere, exceto em loops (`i`, `j`) e lambdas de escopo mínimo
- Booleanos com prefixo descritivo: `isEligible`, `hasProduct`, `canOperate`
- Métodos são verbos: `findCustomer`, `validateEligibility`, `buildResponse`

---

### 2. Responsabilidade Única

Cada classe e cada método tem uma única razão para mudar.

```java
// ❌ Errado — método faz validação, busca e montagem de resposta ao mesmo tempo
public EligibilityResponse check(EligibilityRequest request) {
    if (request.customerId() == null) throw new BusinessException("Id obrigatório");
    var customer = jdbc.query(...);
    var product = jdbc.query(...);
    // lógica de elegibilidade misturada com montagem de response
}

// ✅ Correto — cada método tem uma responsabilidade clara
public EligibilityResponse check(EligibilityRequest request) {
    var customer = customerRepository.findById(request.customerId());
    var product = productRepository.findById(request.productId());
    return buildResponse(customer, product, evaluate(customer, product));
}
```

---

### 3. DRY — Don't Repeat Yourself

Nenhuma lógica duplicada. Se o mesmo bloco aparece em dois lugares, ele vira um método ou classe.

```java
// ❌ Errado — validação duplicada em múltiplos services
if (customer.status().equals("BLOCKED")) throw new BusinessException("Cliente bloqueado");

// ✅ Correto — extraído para o lugar certo
customerValidationHelper.assertNotBlocked(customer);
```

---

### 4. Tratamento de Erros

Erros são tratados explicitamente — nunca ignorados, nunca engolidos.

```java
// ❌ Errado
try {
    return repository.findById(id);
} catch (Exception e) {
    return null;
}

// ✅ Correto
try {
    return repository.findById(id);
} catch (DataAccessException e) {
    log.error("Falha ao buscar cliente com id {}", id, e);
    throw new BusinessException("Erro ao consultar cliente");
}
```

Regras:
- Nunca capturar `Exception` genérica sem relançar ou logar com contexto
- Nunca retornar `null` para sinalizar erro — lançar exceção ou retornar `Optional`
- Mensagens de erro em português, com contexto suficiente para diagnóstico

---

### 5. Testes Limpos

Testes seguem o mesmo padrão de qualidade do código de produção.

```java
@Test
@DisplayName("deve retornar elegível quando cliente está ativo e produto disponível")
void deveRetornarElegivel() {
    // dado
    var customer = buildActiveCustomer();
    given(customerRepository.findById(customer.id())).willReturn(Optional.of(customer));

    // quando
    var response = eligibilityService.check(buildRequest(customer.id(), "PRODUTO_X"));

    // então
    assertThat(response.eligible()).isTrue();
}
```

Regras:
- Nome do teste descreve o comportamento esperado, não o método testado
- `@DisplayName` em português
- Um assert por comportamento — não misturar verificações não relacionadas no mesmo teste
- Sem lógica condicional dentro do teste (`if`, `for`)

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

## Lombok

Uso obrigatório para reduzir boilerplate. Nunca escrever construtores, getters ou setters manualmente quando o Lombok resolve.

**Injeção de dependência — sempre `@RequiredArgsConstructor` com campos `final`**

```java
// ❌ Errado — @Autowired
@Service
public class EligibilityService {

    @Autowired
    private EligibilityRepository repository;
}

// ❌ Errado — construtor manual
@Service
public class EligibilityService {

    private final EligibilityRepository repository;

    public EligibilityService(EligibilityRepository repository) {
        this.repository = repository;
    }
}

// ✅ Correto
@Service
@RequiredArgsConstructor
public class EligibilityService {

    private final EligibilityRepository repository;
}
```

Regras:
- `@RequiredArgsConstructor` em toda classe com injeção de dependência
- Todos os campos injetados devem ser `final`
- Nunca usar `@Autowired`
- `@Getter` em classes de model quando necessário
- `@Builder` em classes de response quando a montagem tem muitos campos
- Nunca usar `@Data` em classes de domínio — gera `equals`, `hashCode` e `toString` que podem causar problemas não intencionais

---

## Configuração

**Usar sempre `application.properties`** — nunca `application.yml`.

```properties
# Servidor
server.port=8080

# Datasource
spring.datasource.url=jdbc:oracle:thin:@${DB_HOST}:${DB_PORT}/${DB_SERVICE}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# Propriedades customizadas — prefixo do nome do serviço
eligibility.cache.ttl=60
eligibility.external-service.url=${EXTERNAL_SERVICE_URL}
```

Regras:
- Nunca commitar credenciais — sempre variáveis de ambiente com `${VAR}`
- Propriedades customizadas com prefixo do nome do serviço: `eligibility.`
- Nomes em lowercase com hífen: `external-service.url`, não `externalServiceUrl`

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

## Repository

### Convenções

**1. Sempre usar `NamedParameterJdbcTemplate` com parâmetros nomeados**

```java
// ❌ Errado — parâmetros posicionais com "?"
jdbc.query("SELECT * FROM customers WHERE id = ?", new Object[]{id}, rowMapper);

// ✅ Correto — parâmetros nomeados
var params = new MapSqlParameterSource("customerId", customerId);
jdbc.query(CustomerQueries.FIND_BY_ID, params, rowMapper);
```

**2. `RowMapper` sempre em classe separada**

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

**3. Retornar `Optional` em consultas que podem retornar vazio**

```java
// ❌ Errado — lança EmptyResultDataAccessException
return jdbc.queryForObject(sql, params, rowMapper);

// ✅ Correto
return jdbc.query(sql, params, rowMapper).stream().findFirst();
```

**4. SQL em constantes nomeadas**

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

**5. `@Transactional` nunca no Repository**

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

---

## Logs

API de uso interno — dados sensíveis (CPF, e-mail, etc.) podem ser logados, seguindo o padrão já adotado pelas demais APIs do time.

### Pontos obrigatórios de log

| Ponto | Nível | O quê registrar |
|---|---|---|
| Entrada no Controller | `INFO` | Payload recebido com campos relevantes para rastreamento |
| Antes de operação no banco | `INFO` | Parâmetros da query |
| Antes de integração com serviço externo | `INFO` | Serviço chamado e parâmetros relevantes |
| Antes de publicar no Kafka | `INFO` | Tópico e identificadores da mensagem |
| Desvios de negócio | `WARN` | Motivo do desvio com contexto |
| Erros | `ERROR` | Exceção com contexto suficiente para diagnóstico |

### Exemplos

```java
// Controller — entrada do fluxo
log.info("Requisição recebida: {}", request);

// Repository — antes de operação no banco
log.info("Buscando elegibilidade para customerId={} e productId={}", customerId, productId);

// Integração externa
log.info("Chamando serviço de cadastro para customerId={}", customerId);

// Kafka
log.info("Publicando evento no tópico {} para customerId={}", topicName, customerId);

// Desvio de negócio
log.warn("Cliente {} não elegível para produto {}: {}", customerId, productId, reason);

// Erro
log.error("Falha ao consultar elegibilidade para customerId={}", customerId, e);
```

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
| `controller/api` | Contrato/interface da API exposta |
| `service` | Lógica de negócio |
| `model` | Representação dos dados e entidades do domínio |
| `helper` | Lógica auxiliar reutilizável sem acoplamento de negócio |
| `repository` | Acesso ao banco de dados |
| `repository/queries` | Constantes SQL |
| `repository/mapper` | Mapeamento entre `ResultSet` e model |
| `listener` | Consumo de eventos (Kafka, fila, etc.) |
| `publisher` | Publicação de eventos |