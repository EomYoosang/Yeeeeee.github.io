---
title: 스프링 웹 개발 기초
description:
categories:
 - spring-basic
tags:
---

# 스프링 웹 개발 기초
- 정적 컨텐츠  
고정된 페이지를 보냄
- MVC와 템플릿 엔진  
페이지를 서버에서 수정하여 보냄
- API  
브라우저가 아닌 다른 플랫폼에 JSON등의 방식으로 데이터를 보냄

## 정적 컨텐츠
- 스프링 부트 정적 컨텐츠 기능  
- src/resoureces/static 폴더의 html파일 제공  
http://localhost:8080/static.html => static.html을 보냄   

<img alt="static" src="/assets/images/static.png" style="width:80%" />  

## MVC와 템플릿 엔진
- MVC: Model, View, Controller  
MVC 패턴의 주 목적은 Business logic과 Presentation logic을 분리하기 위함  

**Model**
- 데이터 저장소(ex. db)와 연동하여 사용자가 입력한 데이터나 사용자에게 출력할 데이터를 다루는 일을 함  
- 여러 개의 데이터 변경 작업을 하나의 작업으로 묶는 트랜잭션을 다루는 일도 함  
- DAO클래스, Service클래스가 Model에 해당  

**View**
- 모델이 처리한 데이터나 결과를 가지고 사용자에게 출력할 화면을 만듦.  
**Controller**  
- 클라이언트의 요청을 받았을 때 그 요청에 대해 실제 업무를 수행하는 모델 컴포넌트를 호출하는 일을 함
- 클라이언트가 보낸 데이터가 있다면, 모델을 호출하기 전 데이터를 가공
- 모델의 결과물을 뷰에 전달

## 일반적인 웹 애플리케이션 계층 구조
<img alt="기본 웹 구조" src="/assets/images/basicWeb.png" style="width:80%;" />

- 컨트롤러: 웹 MVC의 컨트롤러 역할
- 서비스: 핵심 비즈니스 로직 구현
- 리포지토리: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리
- 도메인: 비즈니스 도메인 객체, 예) 회원, 주문, 쿠폰 등등 주로 데이터베이스에 저장하고 관리됨  

## 예제 코드
- HelloController.java

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
- resoureces/template/hello-template.html
```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```  
<img alt="template" src="/assets/images/template.png" stype="width:80%" />  

** api version
- HelloController.java
```java
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


