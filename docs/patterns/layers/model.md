# Camada Model — PSCC Team

## O que é um Model

Classes do pacote `model/` representam qualquer estrutura de dados interna do serviço: retorno de queries, filtros, agregados para insert ou update. Não existem classes genéricas de entidade. Cada model é nomeado pelo contexto ou operação que representa.

```
model/
├── CustomerEligibility      ← retorno de query de elegibilidade
├── Product                  ← dado interno de produto
├── EligibilityFilter        ← parâmetros de filtro para query
└── builder/
    ├── CustomerEligibilityBuilder
    └── EligibilityFilterBuilder
```

---

## Exemplos

```java
// Model simples — sem builder
public class CustomerEligibility {
    private Long id;
    private String status;
    private String riskProfile;
}

// Model complexo — com builder no sub-pacote model/builder/
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

---

## Regras

- Nomeado pelo contexto: `CustomerEligibility`, não `Customer`
- Usar builder quando o model tem muitos campos ou campos opcionais
- Um model por contexto/operação — não reutilizar models entre operações distintas
- Nunca usar `@Data` — ver [`../conventions/lombok.md`](../conventions/lombok.md)
- Model não é Request nem Response — não cruza fronteira de camada

→ Para a distinção entre `record` e `class`, ver [`../conventions/records-vs-classes.md`](../conventions/records-vs-classes.md)
