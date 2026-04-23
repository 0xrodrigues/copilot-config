# ENGINEERING_STANDARDS.md — PSCC Team

> Convenções de como escrever código. Para decisões estruturais (stack, pacotes, camadas, tratamento de erros), ver [ARCHITECTURE.md](ARCHITECTURE.md).

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
    if (request.customerId() == null) throw new BusinessException(CustomerError.CUSTOMER_NOT_FOUND);
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
    throw new BusinessException(CustomerError.CUSTOMER_NOT_FOUND);
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

## `record` vs `class`

**`record`** para tudo que cruza fronteira de camada — `Request`, `Response`, `ErrorResponse`.

**`class`** para tudo que vive dentro do serviço — `Model`, exceções, componentes Spring.

```java
// ✅ Request e Response — record (imutável, contrato de entrada/saída)
public record CreateCustomerRequest(
        @NotNull(message = "Id do cliente é obrigatório")
        Long customerId
) {}

public record CreateCustomerResponse(Long customerId, String status) {}

// ✅ Model — class (dado interno, construção pode ser complexa)
public class CustomerEligibility {
    private Long id;
    private String status;
}
```

Regras:
- `record` em `Request`, `Response` e `ErrorResponse` — imutáveis por natureza, `equals`/`hashCode`/`toString` gerados ajudam nos testes
- `class` em `Model` — RowMapper e builders precisam de flexibilidade; a regra "nunca `@Data` em domínio" continua valendo
- `class` em exceções — herança obriga
- `class` em components Spring (`@Service`, `@Repository`, `@Component`) — gerenciados pelo container

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

## Controller

### ResponseEntity

Toda resposta do controller deve ser envolvida em `ResponseEntity`.

```java
// ❌ Errado — retorna o objeto diretamente
@PostMapping("/customers")
public CreateCustomerResponse create(@RequestBody CreateCustomerRequest request) {
    return customerService.create(request);
}

// ✅ Correto
@PostMapping("/customers")
public ResponseEntity<CreateCustomerResponse> create(@Valid @RequestBody CreateCustomerRequest request) {
    return ResponseEntity.ok(customerService.create(request));
}
```

### Validação de entrada

Bean Validation é usado para validar o conteúdo do request. Todo `@RequestBody` recebe `@Valid`.

```java
// ❌ Errado — sem validação de entrada
public ResponseEntity<CreateCustomerResponse> create(@RequestBody CreateCustomerRequest request) {

// ✅ Correto
public ResponseEntity<CreateCustomerResponse> create(@Valid @RequestBody CreateCustomerRequest request) {
```

As anotações de validação ficam nos campos do record de request:

```java
public record CreateCustomerRequest(

        @NotNull(message = "Id do cliente é obrigatório")
        Long customerId,

        @NotBlank(message = "Produto é obrigatório")
        String productId
) {}
```

Erros de validação são tratados automaticamente pelo `GlobalExceptionHandler` via `MethodArgumentNotValidException`, retornando `400 Bad Request`.

---

## Repository

**1. Sempre usar `NamedParameterJdbcTemplate` com parâmetros nomeados**

```java
// ❌ Errado — parâmetros posicionais com "?"
jdbc.query("SELECT * FROM customers WHERE id = ?", new Object[]{id}, rowMapper);

// ✅ Correto — parâmetros nomeados
var params = new MapSqlParameterSource("customerId", customerId);
jdbc.query(CustomerQueries.FIND_BY_ID, params, rowMapper);
```

**2. Retornar `Optional` em consultas que podem retornar vazio**

```java
// ❌ Errado — lança EmptyResultDataAccessException
return jdbc.queryForObject(sql, params, rowMapper);

// ✅ Correto
return jdbc.query(sql, params, rowMapper).stream().findFirst();
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
