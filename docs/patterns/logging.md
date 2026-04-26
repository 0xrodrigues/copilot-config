# Logs e Observabilidade — PSCC Team

API de uso interno — dados sensíveis (CPF, e-mail, etc.) podem ser logados, seguindo o padrão das demais APIs do time.

---

## Pontos obrigatórios de log

| Ponto | Nível | O quê registrar |
|---|---|---|
| Entrada no Controller | `INFO` | Payload recebido com campos relevantes para rastreamento |
| Antes de operação no banco | `INFO` | Parâmetros da query |
| Antes de integração com serviço externo | `INFO` | Serviço chamado e parâmetros relevantes |
| Antes de publicar no Kafka | `INFO` | Tópico e identificadores da mensagem |
| Desvios de negócio | `WARN` | Motivo do desvio com contexto |
| Erros | `ERROR` | Exceção com contexto suficiente para diagnóstico |

---

## Exemplos por camada

```java
// Controller — entrada do fluxo
log.info("Requisição recebida: {}", request);

// Repository — antes de operação no banco
log.info("Buscando elegibilidade para customerId={} e productId={}", customerId, productId);

// Integração externa
log.info("Chamando serviço de cadastro para customerId={}", customerId);

// Kafka
log.info("Publicando evento no tópico {} para customerId={}", topicName, customerId);

// Desvio de negócio
log.warn("Cliente {} não elegível para produto {}: {}", customerId, productId, reason);

// Erro
log.error("Falha ao consultar elegibilidade para customerId={}", customerId, e);
```

---

## Configuração

Logs em JSON estruturado via `logstash-logback-encoder` para ingestão no Dynatrace.

`@Slf4j` (Lombok) em toda classe que emite logs.
