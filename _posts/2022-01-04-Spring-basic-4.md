---
title: 스프링 웹 개발 기초
description:
categories:
 - spring-basic
tags:
---

## 스프링 웹 개발 기초
- 정적 컨텐츠  
고정된 페이지를 보냄
- MVC와 템플릿 엔진  
페이지를 서버에서 수정하여 보냄
- API  
브라우저가 아닌 다른 플랫폼에 JSON등의 방식으로 데이터를 보냄

### 정적 컨텐츠
- 스프링 부트 정적 컨텐츠 기능  
- src/resoureces/static 폴더의 html파일 제공  
http://localhost:8080/static.html => static.html을 보냄   

<img alt="static" src="/assets/images/static.png" style="width:80%" />  

### MVC와 템플릿 엔진
- MVC: Model, View, Controller

**Controller**

```java:HelloController
@Controller
public class HelloController {
	@GetMapping("hello-mvc")
	public String helloMvc(@RequestParam("name") String name, Model model) {
		model.addAttribute("name", name);
		return "hello-template";
	}
}
```
**View**  
`resoureces/template/hello-template.html`
```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```  
<img alt="template" src="/assets/images/template.png" stype="width:80%" />  

### API
```java:HelloController
@Controller
public class HelloController {
	@GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

`@ResponseBody`를 사용하고, 객체를 반환하면 객체가 자동으로 JSON으로 변환됨  

- http://localhost:8080/hello-api?name=spring  
- 결과: { name: spring }  

**@ResponseBody 사용 원리**  
<img alt="responseBody" src="/assets/images/responseBody.png" style="width:80%"  />
- `ResponseBody`를 사용
   * HTTP의 BODY에 문자 내용을 직접 반환
   * `viewResolver`대신에 `HttpMessageConverter`가 동작
   * 기본 문자처리: `StringHttpmessageConverter`
   * 기본 객체처리: `MappingJackson2HttpMessageConverter`
   * byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

**참고**: 클라이언트의 HTTP Accept헤더와 서버의 컨트롤러 반호나 타입 정보 둘을 조합해서 `HttpMessageConverter`가 선택된다.
