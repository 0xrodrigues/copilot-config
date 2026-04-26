# Camada Repository — PSCC Team

## 1. `NamedParameterJdbcTemplate` com parâmetros nomeados

Nunca usar parâmetros posicionais com `?`.

```java
// ❌ Errado
jdbc.query("SELECT * FROM customers WHERE id = ?", new Object[]{id}, rowMapper);

// ✅ Correto
var params = new MapSqlParameterSource("customerId", customerId);
jdbc.query(CustomerQueries.FIND_BY_ID, params, rowMapper);
```

---

## 2. SQL em constantes — classe `[Domain]Queries`

Todo SQL fica em constantes `static final` na classe de queries do domínio. Nunca SQL inline no método.

```java
// ❌ Errado
jdbc.query("SELECT id, status FROM customers WHERE id = :customerId", params, rowMapper);

// ✅ Correto
public final class CustomerQueries {

    private CustomerQueries() {}

    public static final String FIND_BY_ID = """
            SELECT id,
                   status,
                   risk_profile
            FROM   customers
            WHERE  id = :customerId
            """;
}
```

---

## 3. `RowMapper` sempre em classe separada

Nunca mapper inline na chamada JDBC.

```java
// ❌ Errado
jdbc.query(sql, params, (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("name")));

// ✅ Correto
@Component
public class CustomerMapper implements RowMapper<Customer> {
    @Override
    public Customer mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Customer(rs.getLong("id"), rs.getString("name"));
    }
}
```

---

## 4. `Optional` em consultas que podem retornar vazio

Nunca `queryForObject` para consultas que podem não retornar resultado — lança `EmptyResultDataAccessException`.

```java
// ❌ Errado
return jdbc.queryForObject(sql, params, rowMapper);

// ✅ Correto
return jdbc.query(sql, params, rowMapper).stream().findFirst();
```

---

## 5. `@Transactional` nunca no Repository

O Repository apenas executa queries. Controle de transação é responsabilidade do Service.

→ Ver [`service.md`](service.md)
