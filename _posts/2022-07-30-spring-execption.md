```
title: 스프링 예외처리
description:
categories:
 - springboot
tags:
```

# 스프링 예외처리

스프링 프레임워크는 매우 다양한 에러 처리 방법을 제공합니다 . 어떤 방법이 있고 각각의 장단점, 어떤 방법이 가장 좋은 방법인지 알아봅시다.

## Spring의 기본적인 예외 처리 방법(BasicErrorController)
스프링은 만들어질 때 부터 에러 처리를 위한 BasicErrorController를 구현해두었습니다. 만약 html 요청에서 에러가 발생한다면 errorHtml()을 거쳐 ViewResolver를 통해 에러 페이지가 반환되고, 이외의 요청에서 에러가 발생하면 error()를 거쳐 에러 메시지를 반환합니다. 에러의 경로는 기본적으로 `/error`로 정의되어 있으며 properties에서 server.error.path로 변경할 수 있습니다.

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    private final ErrorProperties errorProperties;
    ...

    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        ...
    }

    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ...
        return new ResponseEntity<>(body, status);
    }

    @ExceptionHandler(HttpMediaTypeNotAcceptableException.class)
    public ResponseEntity<String> mediaTypeNotAcceptable(HttpServletRequest request) {
        HttpStatus status = getStatus(request);
        return ResponseEntity.status(status).build();
    }
}
```

errorHtml()과 error()는 모두 getErrorAttributeOptions를 호출해 반환할 에러 속성을 얻는데, 기본적으로 `DefaultErrorAttributes`로부터 반환할 정보를 가져옵니다. DefaultErrorAttributes는 다음과 같은 전체 항목들에서 프로퍼티 설정에 따라 불필요한 속성들을 제거합니다. 
- timestamp: 에러 발생 시간
- status: 에러의 HTTP 상태
- error: 에러 코드
- path: 에러가 발생한 uri
- exception: 최상위 예외 클래스의 이름(설정 필요)
- message: 에러에 대한 내용(설정 필요)
- errors: BindingException에 의해 생긴 에러 목록(설정 필요)
- trace: 에러 스택 트레이스(설정 필요)

```
{
    "timestamp": "2022-07-30T03:35:44.675+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/product/5000"
}
```

위는 기본 설정으로 받는 에러 응답입니다. 클라이언트 입장에서는 message가 없어 에러의 내용을 확인하기 어렵고 status, error의 내용이 크게 유용하지 않습니다.
> 스프링 2.3부터 클라이언트에게 너무 많은 정보가 노출되는 것을 방지하기 위해 message가 기본적으로 제외됨

## ExceptionResolver
자바에서는 예외 처리를 위해 try-catch를 사용해야 하지만 이 경우 가독성을 떨어뜨린다. Spring은 이러한 문제를 해결하기 위해 에러 처리라는 공통관심사(cross-cutting concerns)를 메인 로직으로부터 분리하는 예외 처리 방식을 고안하였고, 전략 패턴을 사용하여 HandlerExceptionResolver 인터페이스가 만들어졌습니다.

```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```

컨트롤러에서 예외가 발생하면 dispatcher servlet까지 전달됩니다. dispatcher servlet은 상황에 맞는 적합한 예외 처리 전략을 위해 HandlerExceptionResolver 구현체들을 빈으로 등록해서 리스트로 관리합니다. 그리고 적용 가능한 예외 처리기(구현체)를 찾아 예외 처리를 하는데, 기본적으로 아래 네 가지 구현체들이 빈으로 등록되어 있습니다.

- DefaultErrorAttributes: 에러 속성을 저장하며 직접 예외를 처리하지는 않는다.
- DefaultHandlerExeptionResolver: 스프링의 예외들을 처리한다.
- ResponseStatusExceptionResolver: `@ResponseStatus` 또는 `@ResponseStatusException`에 의한 예외를 처리한다.
- ExceptionHandlerExceptionResolver: Controller나 ControllerAdvice에 있는 ExceptionHandler에 의한 예외를 처리한다.

DefaultErrorAttributes를 제외한 세 가지 ExceptionResolver들을 HandlerExceptionResolverComposite로 모아서 관리합니다. (컴포지트 패턴)

이제 각각의 예외 처리 방식에 대해 알아봅시다.

### ResponseStatus

`@ResponseStatus`는 에러 HTTP 상태를 변경하도록 도와주는 어노테이션입니다. `@ResponseStatus`는 다음과 같은 경우들에 적용할 수 있습니다.
- Exception 클래스 자체에 적용
- 메서드에 `@ExceptionHandler`와 함께 적용
- 클래스에 `@RestControllerAdvice`와 함께 적용

예를 들어 아래 CustomException 클래스에 다음과 같이 응답 상태를 지정할 수 있습니다.
```
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class CustomException extends RuntimeException {
    ...
}
```

위 예외 상황 발생시 status 404를 반환합니다.

하지만 `@ResposneStatus`는 다음과 같은 한계점을 가지고 있습니다.
- 응답 내용(Payload)를 수정할 수 없다.
- 예외 상황마다 예외 클래스를 추가해야 한다.
- 예외 클래스와 강하게 결합되어 모든 예외에 대해 동일한 상태와 에러 메세지를 반환하게 한다

### ExceptionHandler
`@ExceptionHandler`는 유연하게 에러를 처리할 수 있는 방법을 제공합니다. `@ExceptionHandler`는 컨트롤러의 메서드나  `@ControllerAdvice`또는 `@RestControllerAdvice`가 있는 클래스의 메서드와 함께 사용합니다.

아래는 컨트롤러 메서드에 `@ExceptionHandler`를 추가한 예시입니다.

```java
@RestController
@RequiredArgsConstructor
public class ProductController {

  private final ProductService productService;
  
  @GetMapping("/product/{id}")
  public Response getProduct(@PathVariable String id){
    return productService.getProduct(id);
  }

  @ExceptionHandler(NoSuchElementFoundException.class)
  public ResponseEntity<String> handleNoSuchElementFoundException(NoSuchElementFoundException exception) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(exception.getMessage());
  }
}
```

@ExceptionHandler는 Exception 클래스들을 속성으로 받아 처리할 예외를 지정할 수 있습니다. @ResponseStatus와도 결합가능한데, ResponseEntity에서도 status를 지정하고 @ResponseStatus도 있다면 ResponseEntity가 우선순위를 갖습니다.
ExceptionHandler는 @ResponseStatus와 달리 에러 응답(payload)을 자유롭게 다룰 수 있다는 점에서 유연하다는 장점이 있습니다. 다음 항목을 정해주면 좋습니다.

- code: 어떠한 종류의 에러가 발생하는지에 대한 에러 코드
- message: 왜 에러가 발생했는지에 대한 설명
- erros: 어느 값이 잘못되어 @Valid에 의한 검증이 실패한 것인지를 위한 에러 목록

`@ExceptionHandler`를 사용 시에 주의할 점은 `@ExceptionHandler`에 등록된 예외 클래스와 파라미터로 받는 예와 클래스가 동일해야 한다는 점입니다. 만약 값이 다르다면 스프링은 컴파일 시점에 에러를 내지 않다가 런타임 시점에 에러를 발생시킵니다.

**더 좋은 예외처리 방법의 필요성**  
@ExceptionHandler는 컨트롤러에 구현하므로 특정 컨트롤러에서만 발생하는 예외를 처리하는데 유용합니다. 하지만 여러 컨트롤러에서 발생하는 예외라면 에러 처리 코드가 중복될 가능성이 매우 높습니다. 이러한 문제를 해결하기 위해 @ExceptionHandler가 구현된 공통 에러 처리 컨트롤러를 만들어 해결할 수도 있지만 컨트롤러에 상속이 사용된다는 점에서 더 좋은 방법을 찾을 필요가 있습니다.

### ControllerAdvice와 RestControllerAdvice

Spring은 @ExceptionHandler의 한계를 극복하고자 전역적으로 예외를 처리할 수 있는 @ControllerAdvice와 @RestControllerAdvice 어노테이션을 각각 Spring3.2, Spring4.3부터 제공합니다. 두 어노테이션의 차이는 응답이 json인지 여부입니다.  

아래는 ControllerAdvice 코드입니다.  
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {
    ...
}
```
Component가 붙어있어 @ControllerAdvice가 붙어있는 클래스는 스프링 빈에 등록되는 것을 알 수 있습니다. 아래는 사용 예시입니다.  

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementFoundException.class)
    protected ResponseEntity<?> handleNoSuchElementFoundException(NoSuchElementFoundException e) {
        final ErrorResponse errorResponse = ErrorResponse.builder()
                .code("Item Not Found")
                .message(e.getMessage()).build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
}
```

ControllerAdvice는 기본적으로 모든 컨트롤러에서 동일하게 적용됩니다. 특정 클래스에서만 제한적으로 사용하고 싶다면 @RestControllerAdvice의 basePackages 등을 설정함으로써 제한할 수 있습니다.  
스프링은 예외를 미리 처리해둔 ResponseEntityExceptionHandler를 추상 클래스로 제공하고 있습니다. ResponseEntityExceptionHandler는 스프링 예외에 대한 ExceptionHandler를 모두 구현하므로 ControllerAdvice 클래스가 이를 상속받게 하면 됩니다. 하지만 에러 메시지를 반환하지 않으므로 에러 응답을 보내려면 아래 메서드를 오버라이딩 해야합니다.  
```java
public abstract class ResponseEntityExceptionHandler {
    ...

    protected ResponseEntity<Object> handleExceptionInternal(
        Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {
            
        ...
    }
}
```

만약 위의 추상 클래스 ResponseEntityExceptionHandler를 상속받지 않는다면 예외들은 ModelAndView 객체를 반환하는 DefaultHandlerExceptionResolver로 리다이렉트됩니다. 이 경우 클라이언트가 일관된 에러 응답을 받지 못하므로 ResponseEntityExceptionHandler를 상속받는 것이 좋습니다.  

ControllerAdvice를 이용할 경우 다음과 같은 장점이 있습니다.
- 하나의 클래스로 모든 컨트롤러에 대해 전역적으로 예외 처리가 가능함
- 직접 정의한 에러 응답을 일관성있게 클라이언트에게 내려줄 수 있음
- 별도의 try-catch문이 없어 코드의 가독성이 높아짐

위의 장점 덕에 ControllerAdvice를 이용한 예외 처리 방법이 일반적으로 가장 좋다고 평가받습니다. 하지만 ControllerAdvice를 사용할 때는 다음의 내용을 주의해야합니다.
- 한 프로젝트당 하나의 ControllerAdvice만 관리하는 것이 좋다.
- 만약 여러 ControllerAdvice가 필요하다면 basePackages나 annotations 등을 지정해야 한다.
- 직접 구현한 Exception 클래스들은 한 공간에서 관리한다.

여러 ControllerAdvice가 있을 때 `@Order` 어노테이션으로 ControllerAdvice에 대한 순서(우선순위)를 지정하지 않는다면 의도하지 않은 방식으로 예외가 처리될 수 있습니다. 

### ResponseStatusException
Spring5에서는 `@ResponseStatus`의 프로그래밍적 대안으로서 기본 에러 포맷(DefaultErrorAttribute) 기반으로 빠르게 에러를 반환할 수 있는 ResponseStatusException을 추가하였습니다. ResponseStatusException는 HttpStatus와 함께 선택적으로 reason과 cause를 추가할 수 있으며 언체크 예외(RuntimeException)을 상속받고 있어 명시적으로 에러를 처리해주지 않아도 됩니다. ResponseStatusException은 다음과 같이 사용할 수 있으며 우리가 만든 예외 클래스에 이를 상속받게 구현하면 원하는 status와 message를 설정할 수도 있습니다.  

```java
@GetMapping("/product/{id}")
public ResponseEntity<Product> getProduct(@PathVariable String id) {
    try {
        return ResponseEntity.ok(productService.getProduct(id));
    } catch (NoSuchElementFoundException e) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Item Not Found");
    }
}
```

ResponseStatusException을 사용하면 다음과 같은 장점이 있습니다.  
- 기본적인 예외 처리를 빠르게 적용할 수 있으므로 손쉽게 프로토타이핑할 수 있다.
- HttpStatus를 설정할 수 있고, 예외와의 결합도를 낮출 수 있다.
- 불필요하게 많은 별도의 예외 클래스를 만들지 않아도 된다.
- 프로그래밍 방식으로 예외를 직접 생성하므로 예외를 더욱 잘 제어할 수 있다.

다음과 같은 장점 또한 존재합니다.  
- 전역적인 @ControllerAdvice와 달리 일관되게 예외 처리하는 것이 어렵다.
- 예외 처리 코드가 중복될 수 있다.
- Spring 예외를 처리하는 것이 어렵다.

ResponseStatusException은 DefaultErrorAttributes를 사용하므로 DefaultErrorAttributes를 상속받는 클래스를 만들어 빈으로 등록하면 반환하는 에러메시지를 수정할 수 있습니다.  

```java
@Component
public class CustomTempErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
//        Throwable throwable = getError(webRequest);
//        Throwable cause = throwable.getCause();
        Map<String, Object> errorAttributes = new HashMap<String, Object>();
        errorAttributes.put("code", "NOT_FOUND");
        errorAttributes.put("message", "Item not found");
        return errorAttributes;
    }
}
```

## 스프링 예외처리 흐름

앞서 설명하였듯 다음과 같은 예외 처리기들은 스프링의 빈으로 등록되어 있고, 예외가 발생하면 순차적으로 다음의 Resolver들이 처리가능한지 판별한 후에 예외가 처리됩니다.  

1. ExceptionHandlerExceptionResolver: Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리
2. ResponseStatusExceptionResolver: @ResponseStatus 또는 ResponseStatusException를 처리
3. DefaultHandlerExceptionResolver: 스프링의 예외들을 처리 ex) HttpMediaTypeNotSupportedException, NoHandlerFoundException 등

![](2022-08-04-20-09-15.png)

위 그림처럼 Controller, ControllerAdvice에서 ExceptionHandler가 있는지 확인하여 처리하며 없다면 @ResponseStatus 또는 ResponseStatusException가 있는지 확인하고 모두 해당하지 않는다면 BasicErrorController를 거쳐 에러가 처리됩니다.  

이번 포스트에서 ControllerAdvice를 사용하는 편이 일반적으로 가장 좋은 방법임을 알아봤습니다. 다음 포스트에서는 ControllerAdvice를 효과적으로 사용하는 방법을 다뤄보겠습니다.