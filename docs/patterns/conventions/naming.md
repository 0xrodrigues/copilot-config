# Nomenclatura e Clean Code — PSCC Team

## Nomes revelam intenção

Quem lê o nome não deve precisar de contexto adicional.

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

**Regras:**
- Sem abreviações: `customerId`, não `custId` ou `cId`
- Sem sufixos genéricos: evitar `data`, `info`, `manager`, `util`
- Sem nomes de um caractere, exceto em loops (`i`, `j`) e lambdas de escopo mínimo
- Booleanos com prefixo descritivo: `isEligible`, `hasProduct`, `canOperate`
- Métodos são verbos: `findCustomer`, `validateEligibility`, `buildResponse`

---

## Responsabilidade Única

Cada classe e cada método tem uma única razão para mudar.

```java
// ❌ Errado — validação, busca e montagem no mesmo método
public EligibilityResponse check(EligibilityRequest request) {
    if (request.customerId() == null) throw new BusinessException(CustomerError.CUSTOMER_NOT_FOUND);
    var customer = jdbc.query(...);
    // lógica misturada com montagem de response
}

// ✅ Correto
public EligibilityResponse check(EligibilityRequest request) {
    var customer = customerRepository.findById(request.customerId());
    var product = productRepository.findById(request.productId());
    return buildResponse(customer, product, evaluate(customer, product));
}
```

---

## DRY — Sem lógica duplicada

Se o mesmo bloco aparece em dois lugares, ele vira método ou classe.

```java
// ❌ Errado — validação duplicada em múltiplos services
if (customer.status().equals("BLOCKED")) throw new BusinessException("Cliente bloqueado");

// ✅ Correto — extraído para o lugar certo
customerValidationHelper.assertNotBlocked(customer);
```

---

## Tratamento de Erros

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

**Regras:**
- Nunca capturar `Exception` genérica sem relançar ou logar com contexto
- Nunca retornar `null` para sinalizar erro — lançar exceção ou retornar `Optional`
- Mensagens de erro em português, com contexto suficiente para diagnóstico

→ Para os tipos de exceção e mapeamento HTTP, ver [`../error-handling.md`](../error-handling.md)
