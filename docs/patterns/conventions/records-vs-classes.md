# `record` vs `class` — PSCC Team

## Regra

**`record`** para tudo que cruza fronteira de camada.
**`class`** para tudo que vive dentro do serviço.

| Tipo | Usar | Motivo |
|---|---|---|
| `Request` | `record` | Imutável, contrato de entrada |
| `Response` | `record` | Imutável, contrato de saída |
| `ErrorResponse` | `record` | Imutável, contrato de erro |
| `Model` | `class` | RowMapper e builders precisam de flexibilidade |
| Exceções | `class` | Herança obriga |
| Components Spring | `class` | Gerenciados pelo container |

## Exemplos

```java
// ✅ Request e Response — record
public record CheckEligibilityRequest(
        @NotNull(message = "Id do cliente é obrigatório")
        Long customerId,

        @NotBlank(message = "Produto é obrigatório")
        String productId
) {}

public record ProductEligibilityResponse(boolean eligible, List<String> rejections) {}

// ✅ Model — class
public class CustomerEligibility {
    private Long id;
    private String status;
    private String riskProfile;
}
```

`record` gera `equals`, `hashCode` e `toString` automaticamente — útil em testes de assertions sobre responses.
A regra "nunca `@Data` em domínio" do Lombok continua válida para `class`.
