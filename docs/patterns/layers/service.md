# Camada Service — PSCC Team

## Responsabilidades

O Service é o único lugar onde vive lógica de negócio. Não acessa `HttpServletRequest`, não monta `ResponseEntity`, não executa queries diretamente.

Fluxo esperado:
1. Recebe o request (ou parâmetros extraídos pelo controller)
2. Busca dados via repository
3. Aplica regras de negócio
4. Monta e retorna response

---

## `@Transactional`

Controle de transação é responsabilidade exclusiva do Service. Nunca no Repository, nunca no Controller.

```java
// ❌ Errado — @Transactional no Repository
@Transactional
public Optional<Customer> findById(Long id) { ... }

// ✅ Correto — @Transactional no Service
@Transactional(readOnly = true)
public ProductEligibilityResponse check(CheckEligibilityRequest request) {
    var customer = customerRepository.findById(request.customerId());
    var product = productRepository.findById(request.productId());
    return buildResponse(evaluate(customer, product));
}
```

**Regras:**
- Operações somente leitura usam `@Transactional(readOnly = true)`
- Operações de escrita usam `@Transactional` sem atributos adicionais
- Métodos que não acessam banco não recebem `@Transactional`
