# ENGINEERING_STANDARDS.md

> Documento de padrões de engenharia para microserviços.  
> Válido para todos os projetos independentemente de arquitetura específica.  
> Stack base: **Java 21 · Spring Boot 3.x · Oracle · JDBC puro · Kafka**

| | |
|---|---|
| **Autor** | Edson Rodrigues de Paulo |
| **Equipe** | PSCC — Produtos e Serviços do Cliente \| Cadastro |
| **Data** | 11/04/25 |
| **Versão** | 0.2 |

---

## Índice

- [1. Linguagem](#1-linguagem)
- [2. Estrutura e Organização](#2-estrutura-e-organização)
  - [2.1 Organização de pacotes](#21-organização-de-pacotes)
  - [2.2 Injeção de dependência](#22-injeção-de-dependência)
  - [2.3 Interfaces](#23-interfaces)
  - [2.4 Uso obrigatório do Lombok](#24-uso-obrigatório-do-lombok)
  - [2.5 Nomenclatura de classes](#25-nomenclatura-de-classes)
- [3. Banco de dados (JDBC)](#3-banco-de-dados-jdbc)
  - [3.1 Template padrão](#31-template-padrão)
  - [3.2 Separação de queries SQL](#32-separação-de-queries-sql)
  - [3.3 RowMapper](#33-rowmapper)
  - [3.4 Transações](#34-transações)
  - [3.5 Mapeamento de colunas Oracle](#35-mapeamento-de-colunas-oracle)
  - [3.6 Tratamento de ausência de dados](#36-tratamento-de-ausência-de-dados)
  - [3.7 Mapeamento de parâmetros com `MapSqlParameterSource`](#37-mapeamento-de-parâmetros-com-mapsqlparametersource)
- [4. Logging](#4-logging)
  - [4.1 Formato de saída](#41-formato-de-saída)
  - [4.2 Níveis de log](#42-níveis-de-log)
  - [4.3 O que nunca logar](#43-o-que-nunca-logar)
- [5. Java 21](#5-java-21)
  - [5.1 Records para DTOs imutáveis](#51-records-para-dtos-imutáveis)
  - [5.2 Virtual Threads (Project Loom)](#52-virtual-threads-project-loom)
- [6. Tratamento de Exceções](#6-tratamento-de-exceções)
  - [6.1 Hierarquia de exceções de domínio](#61-hierarquia-de-exceções-de-domínio)
  - [6.2 Handler global com `ProblemDetail`](#62-handler-global-com-problemdetail)
  - [6.3 Onde lançar exceções](#63-onde-lançar-exceções)
  - [6.4 Mensagens de erro](#64-mensagens-de-erro)

---

## 1. Linguagem

- **Código**: 100% inglês — classes, métodos, variáveis e pacotes.
- **Documentação**: 100% português — JavaDoc, comentários inline, mensagens de log e `@DisplayName` em testes.

```java
// ✅ Correto
public class PaymentService {

    /**
     * Processa o pagamento e notifica o cliente.
     */
    public void process(PaymentRequest request) {
        log.info("Iniciando processamento do pagamento id={}", request.getId());
    }
}

// ❌ Errado — nome de classe em português
public class ServicoPagemento { ... }

// ❌ Errado — documentação em inglês
/** Processes the payment and notifies the customer. */
```

---

## 2. Estrutura e Organização

### 2.1 Organização de pacotes

Pacotes são organizados por **fatia funcional (feature slice)**, não por camada técnica.

```
# ✅ Correto — por domínio/funcionalidade
com.empresa.servico.payment
com.empresa.servico.payment.repository
com.empresa.servico.payment.queries
com.empresa.servico.payment.mapper

# ❌ Errado — por camada técnica
com.empresa.servico.repositories
com.empresa.servico.mappers
com.empresa.servico.services
```

### 2.2 Injeção de dependência

- **Sempre** usar `@RequiredArgsConstructor` do Lombok com campos `final`.
- **Nunca** escrever construtores explícitos para injeção.
- **Nunca** usar `@Autowired` em campo ou método.

```java
// import lombok.RequiredArgsConstructor;
// import lombok.extern.slf4j.Slf4j;
// import lombok.Value;
// import lombok.Builder;
// import lombok.Data;

// ✅ Correto
@Service
@RequiredArgsConstructor
public class PaymentService {

    private final PaymentRepository paymentRepository;
    private final NotificationService notificationService;
}

// ❌ Errado — construtor explícito desnecessário
@Service
public class PaymentService {

    private final PaymentRepository paymentRepository;

    public PaymentService(PaymentRepository paymentRepository) {
        this.paymentRepository = paymentRepository;
    }
}

// ❌ Errado — @Autowired em campo
@Service
public class PaymentService {

    @Autowired
    private PaymentRepository paymentRepository;
}
```

### 2.3 Interfaces

- **Não criar interfaces para implementações únicas.** Interfaces existem para abstrair comportamento quando há mais de uma implementação real ou para definir contrato de módulo externo.
- Toda interface criada deve ter justificativa explícita no código ou neste documento.

```java
// ❌ Errado — interface sem propósito de substituição
public interface PaymentService {
    void process(Payment payment);
}

@Service
public class PaymentServiceImpl implements PaymentService { ... }

// ✅ Correto — classe concreta direta
@Service
@RequiredArgsConstructor
public class PaymentService {
    public void process(Payment payment) { ... }
}
```

### 2.4 Uso obrigatório do Lombok

Lombok é obrigatório para reduzir boilerplate. Anotações de uso padrão:

| Contexto | Anotação |
|---|---|
| Injeção de dependência | `@RequiredArgsConstructor` |
| DTO imutável sem herança | `record` (Java 21) — preferido |
| DTO imutável com herança | `@Value` |
| DTOs mutáveis | `@Data` |
| Builder pattern | `@Builder` |
| Herança com Builder | `@SuperBuilder` |
| Logging | `@Slf4j` |

- **Preferir `record`** a `@Value` para DTOs imutáveis sem herança — é nativo do Java 21 e não depende do Lombok.
- **Não** combinar `@Data` com `@Builder` em entidades de domínio (gera conflito em `equals`/`hashCode`). Preferir `@Value` + `@Builder`.

### 2.5 Nomenclatura de classes

| Tipo | Sufixo | Exemplo |
|---|---|---|
| Repositório JDBC | `Repository` | `PaymentRepository` |
| Classe de queries SQL | `Queries` | `PaymentQueries` |
| RowMapper | `RowMapper` | `PaymentRowMapper` |
| Serviço de domínio | `Service` | `PaymentService` |
| Controller REST | `Controller` | `PaymentController` |
| DTO de entrada | `Request` | `CreatePaymentRequest` |
| DTO de saída | `Response` | `PaymentResponse` |
| Exceção de domínio | `Exception` | `PaymentNotFoundException` |
| Exceção base de domínio | `Exception` | `DomainException` |
| Handler global de exceções | `GlobalExceptionHandler` | `GlobalExceptionHandler` |
| Classe de configuração Spring | `Config` | `KafkaConfig`, `SecurityConfig` |
| Consumer de evento Kafka | `EventListener` | `PaymentEventListener` |
| Publisher de evento Kafka | `EventPublisher` | `PaymentEventPublisher` |

---

## 3. Banco de dados (JDBC)

### 3.1 Template padrão

Usar **exclusivamente** `NamedParameterJdbcTemplate`. O `JdbcTemplate` posicional (`?`) é proibido.

```java
// ✅ Correto — named parameters com MapSqlParameterSource
var params = new MapSqlParameterSource("id", id);
jdbc.query(sql, params, rowMapper);

// ❌ Errado — positional
jdbc.queryForObject(sql, rowMapper, id);
```

### 3.2 Separação de queries SQL

SQL **não vive dentro do Repository**. Cada domínio tem uma classe `[Domain]Queries` — uma classe utilitária com construtor privado, responsável por centralizar todas as queries daquele domínio como constantes estáticas.

- **Não** anotar com `@Component` — a classe não é um bean Spring.
- Acesso sempre via nome da classe: `PaymentQueries.FIND_BY_ID`.
- Construtor privado para impedir instanciação.

```java
// ✅ Correto — classe utilitária com constantes estáticas
public final class PaymentQueries {

    private PaymentQueries() {}

    public static final String FIND_BY_ID = """
            SELECT p.id,
                   p.amount,
                   p.status,
                   p.created_at
              FROM payments p
             WHERE p.id = :id
            """;

    public static final String LIST_BY_STATUS = """
            SELECT p.id,
                   p.amount,
                   p.status,
                   p.created_at
              FROM payments p
             WHERE p.status = :status
            """;
}

// ✅ Repository referencia as queries diretamente pela classe
@Repository
@RequiredArgsConstructor
public class PaymentRepository {

    private final NamedParameterJdbcTemplate jdbc;
    private final PaymentRowMapper rowMapper;

    public Optional<Payment> findById(Long id) {
        var params = new MapSqlParameterSource("id", id);
        return jdbc.query(PaymentQueries.FIND_BY_ID, params, rowMapper)
                   .stream()
                   .findFirst();
    }
}

// ❌ Errado — @Component em classe de constantes estáticas
@Component
public class PaymentQueries {
    public static final String FIND_BY_ID = "...";
}

// ❌ Errado — acesso por instância injetada
private final PaymentQueries queries;
jdbc.query(queries.FIND_BY_ID, params, rowMapper); // acessa membro estático via instância
```

> **Por que não injetar como bean?** Constantes estáticas não têm estado e não precisam de ciclo de vida gerenciado pelo Spring. Anotar com `@Component` e injetar cria a aparência de dependência onde não existe — e acessar um campo `static final` via referência de instância é um erro semântico que IDEs reportam como warning.

### 3.3 RowMapper

- `RowMapper` **sempre** em classe separada, **nunca** inline ou como lambda dentro do Repository.
- Um `RowMapper` por entidade de domínio.
- Nomeado como `[Entity]RowMapper`.
- Registrado como `@Component` para ser injetado onde necessário.

```java
// ✅ Correto
@Component
public class PaymentRowMapper implements RowMapper<Payment> {

    @Override
    public Payment mapRow(ResultSet rs, int rowNum) throws SQLException {
        return Payment.builder()
                .id(rs.getLong("id"))
                .amount(rs.getBigDecimal("amount"))
                .status(PaymentStatus.valueOf(rs.getString("status")))
                .createdAt(rs.getTimestamp("created_at").toLocalDateTime())
                .build();
    }
}

// ❌ Errado — RowMapper inline no Repository
public Optional<Payment> findById(Long id) {
    return jdbc.query(SQL, params, (rs, n) -> Payment.builder()
            .id(rs.getLong("id"))
            // ...
            .build())
            .stream().findFirst();
}
```

### 3.4 Transações

- `@Transactional` vive **exclusivamente na camada de Service**, nunca no Repository.
- Métodos de leitura devem usar `@Transactional(readOnly = true)`.
- Transações não são gerenciadas no Controller.

```java
// ✅ Correto
@Service
@RequiredArgsConstructor
public class PaymentService {

    @Transactional(readOnly = true)
    public PaymentResponse find(Long id) { ... }

    @Transactional
    public void process(CreatePaymentRequest request) { ... }
}

// ❌ Errado — @Transactional no Repository
@Repository
public class PaymentRepository {

    @Transactional  // nunca aqui
    public void save(Payment payment) { ... }
}
```

### 3.5 Mapeamento de colunas Oracle

- Sempre usar alias explícito no SQL para evitar dependência de ordem de coluna ou case sensitivity do Oracle.
- Colunas `TIMESTAMP` e `DATE` sempre convertidas explicitamente no `RowMapper` via `rs.getTimestamp(...).toLocalDateTime()`.
- **Nunca** usar índice posicional — sempre por nome de coluna.

```java
// ✅ Correto — nome de coluna explícito
rs.getLong("id")
rs.getString("status")
rs.getTimestamp("created_at").toLocalDateTime()

// ❌ Errado — índice posicional
rs.getLong(1)
rs.getString(2)
```

### 3.6 Tratamento de ausência de dados

- Usar `jdbc.query(...)` + `.stream().findFirst()` para queries que podem retornar zero linhas.
- **Nunca** usar `queryForObject` quando ausência de resultado é possível — lança `EmptyResultDataAccessException`.

```java
// ✅ Correto
public Optional<Payment> findById(Long id) {
    var params = new MapSqlParameterSource("id", id);
    return jdbc.query(PaymentQueries.FIND_BY_ID, params, rowMapper)
               .stream()
               .findFirst();
}

// ❌ Errado — estoura exceção quando não encontra
public Payment findById(Long id) {
    var params = new MapSqlParameterSource("id", id);
    return jdbc.queryForObject(PaymentQueries.FIND_BY_ID, params, rowMapper);
}
```

### 3.7 Mapeamento de parâmetros com `MapSqlParameterSource`

Usar **exclusivamente** `MapSqlParameterSource` para montar os parâmetros das queries. O uso de `Map.of(...)` é proibido — `MapSqlParameterSource` oferece encadeamento fluente, suporte a tipos SQL explícitos e melhor legibilidade em queries com múltiplos parâmetros.

```java
// ✅ Correto — parâmetro único
var params = new MapSqlParameterSource("id", id);

// ✅ Correto — múltiplos parâmetros com encadeamento
var params = new MapSqlParameterSource()
        .addValue("customerId", customerId)
        .addValue("status", status.name())
        .addValue("createdAfter", createdAfter);

// ✅ Correto — tipo SQL explícito (necessário com Oracle em alguns tipos de data)
var params = new MapSqlParameterSource()
        .addValue("id", id)
        .addValue("processedAt", processedAt, Types.TIMESTAMP);

// ❌ Errado — Map.of() não deve ser usado
var params = Map.of("id", id, "status", status.name());
jdbc.query(PaymentQueries.LIST_BY_STATUS, params, rowMapper);
```

Exemplo completo no Repository:

```java
@Repository
@RequiredArgsConstructor
public class PaymentRepository {

    private final NamedParameterJdbcTemplate jdbc;
    private final PaymentRowMapper rowMapper;

    public Optional<Payment> findById(Long id) {
        var params = new MapSqlParameterSource("id", id);
        return jdbc.query(PaymentQueries.FIND_BY_ID, params, rowMapper)
                   .stream()
                   .findFirst();
    }

    public List<Payment> listByCustomerAndStatus(Long customerId, PaymentStatus status) {
        var params = new MapSqlParameterSource()
                .addValue("customerId", customerId)
                .addValue("status", status.name());
        return jdbc.query(PaymentQueries.LIST_BY_CUSTOMER_AND_STATUS, params, rowMapper);
    }

    public void save(Payment payment) {
        var params = new MapSqlParameterSource()
                .addValue("id", payment.getId())
                .addValue("customerId", payment.getCustomerId())
                .addValue("amount", payment.getAmount())
                .addValue("status", payment.getStatus().name())
                .addValue("createdAt", payment.getCreatedAt(), Types.TIMESTAMP);
        jdbc.update(PaymentQueries.INSERT, params);
    }
}
```

---

## 4. Logging

### 4.1 Formato de saída

Usar `@Slf4j` do Lombok em todas as classes que emitem logs. Nunca instanciar `Logger` manualmente.

```java
// ✅ Correto
@Service
@Slf4j
public class PaymentService {
    public void process(CreatePaymentRequest request) {
        log.info("Iniciando processamento do pagamento");
    }
}

// ❌ Errado — instanciação manual
private static final Logger log = LoggerFactory.getLogger(PaymentService.class);
```

Interpolação de valores **sempre** via placeholders `{}` — nunca por concatenação de string. A concatenação ocorre mesmo quando o nível de log está desabilitado, gerando alocação desnecessária.

```java
// ✅ Correto — placeholder avaliado apenas se o nível estiver ativo
log.debug("Processando pagamento id={} para customerId={}", id, customerId);

// ❌ Errado — concatenação sempre executada
log.debug("Processando pagamento id=" + id + " para customerId=" + customerId);
```

### 4.2 Níveis de log

| Nível | Quando usar |
|---|---|
| `INFO` | Início e fim de operações relevantes, eventos de negócio, transições de estado |
| `WARN` | Situações inesperadas mas recuperáveis (retry, fallback, dado ausente não-crítico) |
| `ERROR` | Falhas não recuperáveis, exceções que interrompem o fluxo |
| `DEBUG` | Detalhes de desenvolvimento — **proibido em produção** |

```java
// ✅ Correto
log.info("Pagamento aprovado id={}", id);
log.warn("Tentativa {} de {} falhou, tentando novamente", attempt, maxAttempts);
log.error("Falha ao processar pagamento id={}", id, exception);

// ❌ Errado — stacktrace perdido
log.error("Falha ao processar pagamento"); // exception ausente como argumento

// ❌ Errado — DEBUG em produção
log.debug("Payload recebido: {}", payload);
```

> `log.error` deve **sempre** receber a exception como último argumento para que o stacktrace apareça no log.

### 4.3 O que nunca logar

Dados sensíveis **nunca** devem aparecer em logs. Violação disso é um incidente de segurança.

| Proibido | Alternativa |
|---|---|
| Número do cartão (PAN) | Últimos 4 dígitos: `****1234` |
| CVV | Nunca logar |
| Senha / token de autenticação | Nunca logar |
| CPF completo | Mascarado: `***.***.678-90` |
| Payload de requisição completo | Campos individuais e relevantes |

```java
// ❌ Errado — pode conter PAN, CPF, senha, etc.
log.info("Requisição recebida: {}", request);

// ✅ Correto — campos explícitos e seguros
log.info("Requisição recebida para customerId={}", request.getCustomerId());
```

---

## 5. Java 21

### 5.1 Records para DTOs imutáveis

Java 21 oferece **records** como alternativa nativa ao `@Value` do Lombok para objetos imutáveis. Records são a escolha preferida para DTOs de entrada e saída quando não há necessidade de herança.

| Situação | Usar |
|---|---|
| DTO imutável sem herança | `record` |
| DTO imutável com herança | `@Value` + `@SuperBuilder` |
| DTO mutável | `@Data` |

```java
// ✅ Correto — record para DTO de saída
public record PaymentResponse(
        Long id,
        BigDecimal amount,
        String status,
        LocalDateTime createdAt
) {}

// ✅ Correto — record com validação via construtor compacto
public record CreatePaymentRequest(
        Long customerId,
        BigDecimal amount,
        String description
) {
    public CreatePaymentRequest {
        Objects.requireNonNull(customerId, "customerId é obrigatório");
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("amount deve ser positivo");
        }
    }
}

// ❌ Desnecessário — @Value do Lombok quando record resolve
@Value
@Builder
public class PaymentResponse {
    Long id;
    BigDecimal amount;
    String status;
    LocalDateTime createdAt;
}
```

> Records **não** devem ser usados para entidades de domínio com identidade mutável. Use-os exclusivamente para objetos de transferência de dados (DTOs, eventos, value objects sem estado mutável).

### 5.2 Virtual Threads (Project Loom)

Spring Boot 3.2+ suporta **Virtual Threads** nativamente. Para serviços com workloads predominantemente bloqueantes (JDBC, chamadas HTTP síncronas), a habilitação aumenta throughput sem alteração de código.

**Padrão de configuração por tipo de projeto:**

- **Novos projetos** — usar `application.yml`:

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

- **Projetos existentes** com `application.properties` — manter o formato já adotado:

```properties
spring.threads.virtual.enabled=true
```

> Não migrar projetos existentes de `.properties` para `.yml` apenas para esta configuração. A migração de formato deve ser feita de forma planejada e abrangente, não incremental.

Com isso, o Spring Boot configura automaticamente o executor do Tomcat e do `@Async` para usar virtual threads.

**Quando habilitar:**

| Cenário | Recomendação |
|---|---|
| Serviço com muitas threads bloqueadas em JDBC / IO | ✅ Habilitar |
| Serviço CPU-bound (processamento pesado, criptografia) | ⚠️ Avaliar — ganho marginal |
| Serviço reativo (WebFlux) | ❌ Não aplicável — já é não-bloqueante |

**Restrições importantes:**

- **Não usar `synchronized` em blocos que fazem IO** — causa *pinning* da virtual thread na carrier thread e anula o benefício. Substituir por `ReentrantLock`.
- **`ThreadLocal` ainda funciona**, mas o custo de criação de virtual threads é tão baixo que padrões baseados em pooling de threads (ex: `ThreadLocal` para conexão manual) perdem sentido — prefira escopo por requisição.
- O pool de conexões do HikariCP funciona corretamente com virtual threads a partir da versão 5.1.0.

```java
// ❌ Evitar — synchronized com IO dentro causa pinning
public synchronized Payment fetchAndProcess(Long id) {
    return repository.findById(id); // IO dentro de synchronized
}

// ✅ Correto — ReentrantLock não causa pinning
private final ReentrantLock lock = new ReentrantLock();

public Payment fetchAndProcess(Long id) {
    lock.lock();
    try {
        return repository.findById(id);
    } finally {
        lock.unlock();
    }
}
```

---

## 6. Tratamento de Exceções

### 6.1 Hierarquia de exceções de domínio

Toda exceção de negócio deve estender uma classe base comum. Isso permite tratamento centralizado e garante que o handler global consiga diferenciar erros de domínio de erros de infraestrutura.

```java
// import org.springframework.http.HttpStatus;

// Classe base — nunca lançada diretamente
public abstract class DomainException extends RuntimeException {

    private final HttpStatus status;

    protected DomainException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}

// Exceções específicas — cada uma define seu próprio status HTTP
public class PaymentNotFoundException extends DomainException {

    public PaymentNotFoundException(Long id) {
        super("Pagamento não encontrado para id=" + id, HttpStatus.NOT_FOUND);
    }
}

public class PaymentAlreadyProcessedException extends DomainException {

    public PaymentAlreadyProcessedException(Long id) {
        super("Pagamento id=" + id + " já foi processado", HttpStatus.UNPROCESSABLE_ENTITY);
    }
}

public class InsufficientBalanceException extends DomainException {

    public InsufficientBalanceException(Long customerId) {
        super("Saldo insuficiente para customerId=" + customerId, HttpStatus.UNPROCESSABLE_ENTITY);
    }
}
```

### 6.2 Handler global com `ProblemDetail`

Usar `@RestControllerAdvice` com `ProblemDetail` (RFC 9457) — suportado nativamente pelo Spring Boot 3.x. O handler centraliza toda a tradução de exceção → resposta HTTP. **Nenhum Controller trata exceções diretamente.**

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // Qualquer DomainException — status HTTP definido na própria exceção
    @ExceptionHandler(DomainException.class)
    public ProblemDetail handleDomain(DomainException ex) {
        log.warn("Erro de domínio: {}", ex.getMessage());
        return ProblemDetail.forStatusAndDetail(ex.getStatus(), ex.getMessage());
    }

    // Erros de validação de entrada — 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        var detail = ex.getBindingResult().getFieldErrors().stream()
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .collect(Collectors.joining(", "));
        log.warn("Erro de validação: {}", detail);
        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, detail);
    }

    // Fallback — erros não mapeados — 500
    @ExceptionHandler(Exception.class)
    public ProblemDetail handleUnexpected(Exception ex) {
        log.error("Erro inesperado", ex);
        return ProblemDetail.forStatusAndDetail(
                HttpStatus.INTERNAL_SERVER_ERROR,
                "Erro interno. Tente novamente mais tarde.");
    }
}
```

Habilitação do `ProblemDetail` no `application.yml`:

```yaml
spring:
  mvc:
    problemdetails:
      enabled: true
```

Exemplo de resposta gerada:

```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Pagamento não encontrado para id=42",
  "instance": "/payments/42"
}
```

### 6.3 Onde lançar exceções

| Camada | Comportamento esperado |
|---|---|
| `Repository` | Nunca lança exceção de domínio — retorna `Optional` quando ausência é válida |
| `Service` | Lança exceção de domínio quando regra de negócio é violada |
| `Controller` | Nunca trata exceção — delega ao handler global |

```java
// ✅ Correto — Service lança, Controller não trata
@Service
@RequiredArgsConstructor
public class PaymentService {

    @Transactional(readOnly = true)
    public PaymentResponse find(Long id) {
        return paymentRepository.findById(id)
                .map(paymentMapper::toResponse)
                .orElseThrow(() -> new PaymentNotFoundException(id));
    }
}

// ✅ Correto — Controller limpo, sem try/catch
@RestController
@RequiredArgsConstructor
@RequestMapping("/payments")
public class PaymentController {

    private final PaymentService paymentService;

    @GetMapping("/{id}")
    public PaymentResponse find(@PathVariable Long id) {
        return paymentService.find(id);
    }
}

// ❌ Errado — Controller tratando exceção diretamente
@GetMapping("/{id}")
public ResponseEntity<PaymentResponse> find(@PathVariable Long id) {
    try {
        return ResponseEntity.ok(paymentService.find(id));
    } catch (PaymentNotFoundException ex) {
        return ResponseEntity.notFound().build();
    }
}
```

### 6.4 Mensagens de erro

- Mensagens de exceção de domínio devem ser **descritivas e incluir o identificador** relevante para facilitar rastreio nos logs.
- **Nunca** expor detalhes de infraestrutura nas mensagens de erro retornadas ao cliente (stack trace, nome de tabela, query SQL).
- O campo `detail` do `ProblemDetail` é voltado ao consumidor da API — deve ser legível e acionável.

```java
// ✅ Correto — mensagem descritiva com contexto
throw new PaymentNotFoundException(id);
// → "Pagamento não encontrado para id=42"

// ❌ Errado — mensagem genérica sem contexto
throw new PaymentNotFoundException("Not found");

// ❌ Errado — detalhe de infraestrutura exposto ao cliente
ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR,
        "ORA-00942: table or view does not exist");
```

---

*Próximas seções a definir: Testes · API e Contratos · Configuração e Profiles*