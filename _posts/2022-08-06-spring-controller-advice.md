---
title: RestControllerAdvice를 이용한 예외처리
description:
categories:
 - springboot
tags:
---

# ControllerAdvice와 RestControllerAdvice
스프링은 예외를 전역적으로 처리할 수 있는 ControllerAdvice와 RestControllerAdvice를 제공하고 있습니다. RestControllerAdvice는 ControllerAdvice에 @ResponseBody가 붙어있다는 점이 다릅니다. 

```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@ControllerAdvice 
@ResponseBody 
public @interface RestControllerAdvice {
    ... 
} 

@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@Component 
public @interface ControllerAdvice {
    ... 
}
```

ControllerAdvice는 여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용해줍니다.

> 위 내용들은 이전 포스트에서 설명했습니다:)

# RestControllerAdvice를 이용한 예외처리 방법

## 에러코드 정의하기
먼저 클라이언트에게 보내줄 에러 코드를 정의해봅시다. 기본적으로 에러 이름과 HTTP 상태 및 메세지를 가지고 있는 에러 코드 클래스를 만듭니다. 에러 코드는 애플리케이션에서 전역적으로 사용되는 CommonErrorCode와 특정 도메인에 대해 구체적으로 내려가는 UserErrorCode로 나누고, 인터페이스를 이용해 추상화합니다.
아래는 CommonErrorCode와 UserErrorCode의 공통 메서드로 추상화 할 ErrorCode 인터페이스입니다.

```java
public interface ErrorCode {

    String name();
    HttpStatus getHttpStatus();
    String getMessage();

}
```

그리고 발생할 수 있는 에러코드를 다음과 같이 정의할 수 있습니다.

```java
@Getter
@RequiredArgsConstructor
public enum CommonErrorCode implements ErrorCode {

    INVALID_PARAMETER(HttpStatus.BAD_REQUEST, "Invalid parameter included"),
    RESOURCE_NOT_FOUND(HttpStatus.NOT_FOUND, "Resource not exists"),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "Internal server error");

    private final HttpStatus httpStatus;
    private final String message;
}

@Getter
@RequiredArgsConstructor
public enum UserErrorCode implements ErrorCode {

    INACTIVE_USER(HttpStatus.FORBIDDEN, "User is inactive");

    private final HttpStatus httpStatus;
    private final String message;
}
```

## 예외 클래스 생성

에러 코드 추가 후에는 예외를 처리해줄 예외 클래스를 추가해주어야 합니다. 예를 들어 런타임 예외(언체크 예외)를 상속받는 예외 클래스를 다음과 같이 추가해줄 수 있습니다. 여기서 언체크 예외를 상속받는 이유는 일반적인 비즈니스 로직들은 catch해서 처리할 것이 없으므로 체크 예외로 한다면 불필요하게 throws가 전파될 것이기 때문입니다. 또한 체크 예외는 자동으로 롤백되지 않습니다.

## 에러 응답 클래스 생성

우리는 클라이언트로 다양한 포맷의 에러를 던져줄 수 있어야합니다. 이를 위해 아래와 같은 에러 응답 클래스를 추가할 수 있습니다.
```java
@Getter
@Builder
@RequiredArgsConstructor
public class ErrorResponse {

    private final String code;
    private final String message;

    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    private final List<ValidationError> errors;

    @Getter
    @Builder
    @RequiredArgsConstructor
    public static class ValidationError {
    
        private final String field;
        private final String message;

        public static ValidationError of(final FieldError fieldError) {
            return ValidationError.builder()
                    .field(fieldError.getField())
                    .message(fieldError.getDefaultMessage())
                    .build();
        }
    }
}
```

위의 예외는 @Valid를 사용했을 때 에러가 발생한 경우 어느 필드에서 에러가 발생했는지 응답을 위한 ValidationError를 내부 정적 클래스로 추가했습니다. 또한 만약 errors가 없다면 응답으로 내려가지 않도록 @JsonInclude 어노테이션을 추가하였습니다.

## @RestControllerAdvice 구현하기

이제 전역적으로 에러를 처리해주는 @RestControllerAdvice 클래스를 추가합니다. 스프링 예외를 처리하기 위해 스프링 예외를 미리 처리해둔 ResponseEntityExceptionHandler 추상 클래스를 상속받게하고 아래 메서드를 오버라이딩시켜 Json형식의 에러를 리턴할 수 있도록 합니다.
```java
public abstract class ResponseEntityExceptionHandler {
    ...

    protected ResponseEntity<Object> handleExceptionInternal(
        Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request){
            
        ...
    }
}
```

이제 우리가 만든 RestApiException 예외와 @Valid에 의한 유효성 검증에 실패했을 때 발생하는 IllegalArgumentException 예외와 마지막으로 잘못된 파라미터를 넘겼을 경우 발생하는 IllegalArgumentException 에러를 처리하겠습니다. 
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(RestApiException.class)
    public ResponseEntity<Object> handleCustomException(RestApiException e) {
        ErrorCode errorCode = e.getErrorCode();
        return handleExceptionInternal(errorCode);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Object> handleIllegalArgument(IllegalArgumentException e) {
        log.warn("handleIllegalArgument", e);
        ErrorCode errorCode = CommonErrorCode.INVALID_PARAMETER;
        return handleExceptionInternal(errorCode, e.getMessage());
    }

    @Override
    public ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException e,
            HttpHeaders headers,
            HttpStatus status,
            WebRequest request) {
        log.warn("handleIllegalArgument", e);
        ErrorCode errorCode = CommonErrorCode.INVALID_PARAMETER;
        return handleExceptionInternal(e, errorCode);
    }

    @ExceptionHandler({Exception.class})
    public ResponseEntity<Object> handleAllException(Exception ex) {
        log.warn("handleAllException", ex);
        ErrorCode errorCode = CommonErrorCode.INTERNAL_SERVER_ERROR;
        return handleExceptionInternal(errorCode);
    }

    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode));
    }

    private ErrorResponse makeErrorResponse(ErrorCode errorCode) {
        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(errorCode.getMessage())
                .build();
    }

    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode, String message) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode, message));
    }

    private ErrorResponse makeErrorResponse(ErrorCode errorCode, String message) {
        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(message)
                .build();
    }

    private ResponseEntity<Object> handleExceptionInternal(BindException e, ErrorCode errorCode) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(e, errorCode));
    }

    private ErrorResponse makeErrorResponse(BindException e, ErrorCode errorCode) {
        List<ErrorResponse.ValidationError> validationErrorList = e.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(ErrorResponse.ValidationError::of)
                .collect(Collectors.toList());

        return ErrorResponse.builder()
                .code(errorCode.name())
                .message(errorCode.getMessage())
                .errors(validationErrorList)
                .build();
    }
}
```

RestApiException 예외와 IllegalArgumentException의 경우에는 이를 캐치해서 핸들링하는 @ExceptionHandler를 구현해주면 됩니다. 하지만 @Valid에 의한 MethodArgumentNotValidException의 경우에는 에러 필드와 메세지를 추가해주어야 하는데, 관련 정보는 MethodArgumentNotValidException의 getBindingResult를 통해서 얻을 수 있습니다.  

## 에러 응답 확인
아래와 같은 예시 컨트롤러를 통해 에러 응답을 확인해봅니다.

```java
@RestController
@RequiredArgsConstructor
public class UserController {

    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser() {
        throw new RestApiException(UserErrorCode.INACTIVE_USER);
    }
}
```

해당 api를 호출해보면 원하는 에러가 나오는 것을 확인할 수 있습니다.  