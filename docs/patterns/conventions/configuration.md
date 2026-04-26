# Configuração — PSCC Team

Usar sempre `application.properties`. Nunca `application.yml`.

## Exemplo

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

## Regras

- Nunca commitar credenciais — sempre variáveis de ambiente com `${VAR}`
- Propriedades customizadas com prefixo do nome do serviço: `eligibility.`
- Nomes em lowercase com hífen: `external-service.url`, não `externalServiceUrl`
