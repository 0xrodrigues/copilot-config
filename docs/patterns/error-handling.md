# Tratamento de Erros — PSCC Team

## Formato de resposta de erro

Toda API retorna erros no mesmo formato:

```json
{
  "messageError": "CUSTOMER_CLOSED_FRAUD",
  "code": 2,
  "message": "Cliente fechado por fraude",
  "moment": "2026-04-22T14:30:00"
}
```

---

## Mapeamento HTTP

| Situação | HTTP Status |
|---|---|
| Erro de negócio | `422 Unprocessable Entity` |
| Recurso não encontrado | `404 Not Found` |
| Erro de validação do request | `400 Bad Request` |
| Erro inesperado | `500 Internal Server Error` |

---

## Interface `ApiError`

Todos os enums de erro implementam `ApiError`:

```java
public interface ApiError {
    int getCode();
    String getMessage();
    String name();
}
```

---

## Enums de erro

Cada contexto define seu próprio enum. Os códigos são **incrementais por serviço** — não por enum. Antes de adicionar um novo código, verificar se já está em uso em outro enum do mesmo serviço.

```java
public enum CustomerError implements ApiError {

    CUSTOMER_NOT_FOUND(1, "Cliente não encontrado"),
    CUSTOMER_CLOSED_FRAUD(2, "Cliente fechado por fraude");

    private final int code;
    private final String message;

    CustomerError(int code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override public int getCode() { return code; }
    @Override public String getMessage() { return message; }
}
```

---

## `ErrorResponse`

```java
@Builder
public record ErrorResponse(
        String messageError,
        int code,
        String message,
        LocalDateTime moment
) {
    public static ErrorResponse of(ApiError error) {
        return ErrorResponse.builder()
                .messageError(error.name())
                .code(error.getCode())
                .message(error.getMessage())
                .moment(LocalDateTime.now())
                .build();
    }
}
```

---

## Exceções de domínio

```java
public class BusinessException extends RuntimeException {
    private final ApiError error;

    public BusinessException(ApiError error) {
        super(error.getMessage());
        this.error = error;
    }

    public ApiError getError() { return error; }
}

public class NotFoundException extends RuntimeException {
    private final ApiError error;

    public NotFoundException(ApiError error) {
        super(error.getMessage());
        this.error = error;
    }

    public ApiError getError() { return error; }
}
```

---

## `GlobalExceptionHandler`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException ex) {
        log.warn("Erro de negócio: {}", ex.getError().name());
        return ResponseEntity.unprocessableEntity()
                .body(ErrorResponse.of(ex.getError()));
    }

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(NotFoundException ex) {
        log.warn("Recurso não encontrado: {}", ex.getError().name());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ErrorResponse.of(ex.getError()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        log.warn("Erro de validação: {}", ex.getMessage());
        return ResponseEntity.badRequest()
                .body(ErrorResponse.builder()
                        .messageError("VALIDATION_ERROR")
                        .code(0)
                        .message(ex.getBindingResult().getFieldErrors().stream()
                                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                                .collect(Collectors.joining(", ")))
                        .moment(LocalDateTime.now())
                        .build());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Erro inesperado", ex);
        return ResponseEntity.internalServerError()
                .body(ErrorResponse.builder()
                        .messageError("INTERNAL_ERROR")
                        .code(0)
                        .message("Erro interno. Tente novamente.")
                        .moment(LocalDateTime.now())
                        .build());
    }
}
```
