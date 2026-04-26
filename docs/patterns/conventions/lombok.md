# Lombok — PSCC Team

Uso obrigatório para reduzir boilerplate. Nunca escrever construtores, getters ou setters manualmente quando o Lombok resolve.

## Injeção de dependência

Sempre `@RequiredArgsConstructor` com campos `final`. Nunca `@Autowired`.

```java
// ❌ Errado
@Service
public class EligibilityService {
    @Autowired
    private EligibilityRepository repository;
}

// ❌ Errado
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

## Referência rápida

| Anotação | Quando usar |
|---|---|
| `@RequiredArgsConstructor` | Toda classe com injeção de dependência |
| `@Getter` | Models que precisam expor campos |
| `@Builder` | Responses com muitos campos ou construção complexa |
| `@Slf4j` | Toda classe que faz log |

## O que nunca usar

| Anotação | Motivo |
|---|---|
| `@Data` | Gera `equals`, `hashCode` e `toString` que causam problemas em classes de domínio |
| `@Autowired` | Substituído por `@RequiredArgsConstructor` |
| `@AllArgsConstructor` | Expõe construtor público sem controle — preferir `@RequiredArgsConstructor` ou builder |
