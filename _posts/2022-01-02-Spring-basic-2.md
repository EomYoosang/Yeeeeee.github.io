---
title: View 생성
description:
categories:
 - spring-basic
tags:
---

## View 생성

### 메인 페이지 생성
```html
<!DOCTYPE HTML>
<html>
<head>
	<title>Hello</title>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```
- 스프링부트는 `static/index.html`을 메인 페이지로 제공  
2.6.2버전 기준 공식문서 중: [https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web)

### thymeleaf 템플릿 엔진
nodejs의 nunjucks나 ejs를 떠올리자  
thymeleaf외에도 FreeMarker, Mustache등 사용  

thymeleaf 공식문서: [https://www.tymeleaf.org/](https://www.tymeleaf.org/)

#### Controller
```java:HelloController.java
package eys.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello(Model model){
        model.addAttribute("data", "hello!");
        return "hello";
    }
}

```
`controller` 패키지에 `HelloController`클래스 생성

#### Template
```html:hello.html
<!DOCTYPE HTML>
<html xmlns:th="http://thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```

동작 방식은 아래와 같다.  
<img alt="스프링 동작방식" src="/assets/images/springboot-thymeleaf.png" />

### 참고
`spring-boot-devtools`라이브러리를 추가하면 `html`파일을 컴파일만 해주면 서버 재시작 없이 View파일 변경이 가능하다.  
스크립트 언어는 소스코드 변경도 적용해주었는데 컴파일언어는 그게 안되니 불편하다...