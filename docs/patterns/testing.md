# Testes — PSCC Team

⚠️ Este documento está em construção. As seções abaixo representam o escopo a ser definido.

---

## Convenções (definidas)

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

**Regras:**
- `@DisplayName` em português descrevendo o comportamento, não o método
- Estrutura `// dado / quando / então` obrigatória
- Um assert por comportamento
- Sem lógica condicional dentro do teste (`if`, `for`)

---

## A definir

- [ ] Frameworks: JUnit 5 + Mockito + AssertJ? Confirmar versões
- [ ] Estratégia de teste de Repository (H2 in-memory vs Testcontainers Oracle?)
- [ ] Estratégia de teste de integração (contexto completo vs slices?)
- [ ] Cobertura mínima exigida pelo time
- [ ] Padrão de nomenclatura de classes de teste (`CustomerServiceTest` vs `CustomerServiceSpec`?)
- [ ] Onde ficam os builders de teste (fixtures compartilhados?)
- [ ] Estratégia para testes de Kafka consumer/producer
