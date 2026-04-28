# PSCC Team — Copilot Instructions


## Idioma

- Código (classes, métodos, variáveis, constantes): **inglês**
- Logs, documentação, comentários, `@DisplayName`, mensagens de erro da API: **português**

---

## Lombok

Uso obrigatório. Nunca escrever construtores, getters ou setters manualmente.

- Injeção de dependência: sempre `@RequiredArgsConstructor` com campos `final`. Nunca `@Autowired`.
- Logs: `@Slf4j` em toda classe que emite logs.
- **Nunca usar** `@Data`, `@Autowired` ou `@AllArgsConstructor`.

```java
// ✅ Correto
@Service
@RequiredArgsConstructor
public class EligibilityService {
    private final EligibilityRepository repository;
}
```

---

## Nomenclatura

- Sem abreviações: `customerId`, não `custId`
- Sem sufixos genéricos: evitar `data`, `info`, `manager`, `util`
- Booleanos com prefixo descritivo: `isEligible`, `hasProduct`, `canOperate`
- Métodos são verbos: `findCustomer`, `validateEligibility`, `buildResponse`

---

## Camada Repository

**1. Sempre `NamedParameterJdbcTemplate` com parâmetros nomeados — nunca `?`.**

**2. SQL em constantes `static final` na classe `[Domain]Queries`. Nunca SQL inline.**

```java
public final class CustomerQueries {
    private CustomerQueries() {}

    public static final String FIND_BY_ID = """
            SELECT id, status, risk_profile
            FROM   customers
            WHERE  id = :customerId
            """;
}
```

**3. `RowMapper` sempre em classe separada anotada com `@Component`. Nunca inline.**

**4. Consultas que podem retornar vazio: usar `jdbc.query(...).stream().findFirst()`. Nunca `queryForObject`.**

**5. `@Transactional` nunca no Repository — somente no Service.**

---

## Tratamento de Erros

- Nunca capturar `Exception` genérica sem relançar ou logar com contexto
- Nunca retornar `null` para sinalizar erro — lançar exceção ou retornar `Optional`
- Mensagens de erro em português com contexto suficiente para diagnóstico

---

## Logs

Pontos obrigatórios:

| Ponto | Nível |
|---|---|
| Entrada no Controller | `INFO` |
| Antes de operação no banco | `INFO` |
| Antes de integração externa ou publicação Kafka | `INFO` |
| Desvio de negócio | `WARN` |
| Erro | `ERROR` (sempre com exceção como argumento final) |

---

## Testes

- `@DisplayName` em português descrevendo o comportamento
- Estrutura obrigatória: `// dado / quando / então`
- Um assert por comportamento
- Sem lógica condicional dentro do teste

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