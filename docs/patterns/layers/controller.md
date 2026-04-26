# Camada Controller — PSCC Team

## Interface `api/`

O pacote `controller/api/` contém a interface que o controller implementa, com todas as anotações OpenAPI. Isso separa o contrato da implementação.

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

**Regras:**
- Anotações OpenAPI ficam **somente na interface** — nunca no controller
- `@RestController`, `@RequiredArgsConstructor` e `@Slf4j` ficam no controller, não na interface
- O log de entrada fica no controller
- Toda resposta envolvida em `ResponseEntity`
- Nenhuma lógica de negócio no controller — só delega ao service

---

## ResponseEntity

```java
// ❌ Errado
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

---

## Validação de Entrada

Todo `@RequestBody` recebe `@Valid`. As anotações de validação ficam nos campos do record de request.

```java
public record CreateCustomerRequest(

        @NotNull(message = "Id do cliente é obrigatório")
        Long customerId,

        @NotBlank(message = "Produto é obrigatório")
        String productId
) {}
```

Erros de validação são tratados automaticamente pelo `GlobalExceptionHandler` → retorna `400 Bad Request`.

→ Para detalhes do tratamento de erro, ver [`../error-handling.md`](../error-handling.md)
