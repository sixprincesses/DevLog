## 1. 실습 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok

## 2. Custom Exception

### Custom Error Code

1. Global 폴더
```java
public interface ErrorCode {

    int getStatus();

    String getCode();

    String getMessage();
}

```
2. Domain 별 개별 폴더
```java
@Getter
public enum AuthErrorCode implements ErrorCode {

    REFRESH_NOT_FOUND(400, "Auth_001", "refresh is not found"),
    REFRESH_NOT_VALID(401, "Auth_002", "refresh is not valid");

    private final int status;
    private final String code;
    private final String message;

    AuthErrorCode(final int status, final String code, final String message) {
        this.status = status;
        this.code = code;
        this.message = message;
    }
}
```

`** code는 BE에서 남기기 위한 용도입니다`

### Custom Exception

1. Global 폴더
```java
@Getter
public class CustomException extends RuntimeException {

    private ErrorCode errorCode;

    public CustomException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
}
```
2. Domain 별 개별 폴더
```java
public class AuthException extends CustomException {
    public AuthException(ErrorCode errorCode) {
        super(errorCode);
    }
}
```

### Error Response

1. Global 폴더
```java
@Data
@Builder
public class ErrorResponse {
    private int status;
    private String message;

    public static ResponseEntity<ErrorResponse> toResponse(ErrorCode errorCode) {

        return ResponseEntity
                .status(errorCode.getStatus())
                .body(ErrorResponse.builder()
                        .status(errorCode.getStatus())
                        .message(errorCode.getMessage())
                        .build());
    }
}
```

### Custom Exception Handler

CustomException이 throw되면 인터셉트합니다

1. Global 폴더
```java
@RestControllerAdvice
public class CustomExceptionHandler {
    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ErrorResponse> customExceptionHandle(CustomException e) {

        return ErrorResponse.toResponse(e.getErrorCode());
    }
}
```