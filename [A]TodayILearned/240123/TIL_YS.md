# 1. Custom Exception

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

# 1. Custom Annotation

## 1. 들어가며

모든 Controller에 /member-service 경로를 적용해야 했습니다

## 2. @RestController 커스텀

1. Global 폴더

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
@RequestMapping("/member-service")
@CrossOrigin("*")
public @interface RestWondooController {
    @AliasFor(
            annotation = Controller.class
    )
    String value() default "";
}
```

`@Target({ElementType.TYPE})`: 클래스 레벨에서만 사용할 수 있음을 나타냅니다

`@Retention(RetentionPolicy.RUNTIME)`: 런타임에도 접근 가능하게 합니다

`@Documented`: javadoc 같은 문서에 포함될 수 있도록 합니다

`@Controller`: 이 클래스가 스프링 MVC 컨트롤러임을 나타냅니다

`@ResponseBody`: 컨트롤러의 메소드가 반환하는 객체를 HTTP Response Body로 변환합니다

`@RequestMapping("/member-service")`: 처리하는 요청의 URL 경로를 /member-service로 지정합니다

`@CrossOrigin("*")`: 모든 도메인에서의 CORS 요청을 허용합니다

2. 사용
```java
@RestWondooController
public class AuthController {
}
```