---
title: 회원 관리 예제3
description:
categories:
 - spring-basic
tags:
---


# 회원 관리 예제3
## 회원 웹 기능 - 홈 화면 추가
- HomeController.java  
```java:HomeController.java
@Controller
public class HomeController {
	@GetMapping("/")
	public String home() {
		return "home";
	}
}
```

- home.html  
```html:home.html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
	<div>
	<h1>Hello Spring</h1>
	<p>회원 기능</p>
	<p>
		<a href="/members/new">회원 가입</a>
		<a href="/members">회원 목록</a>
	</p>
</div>
</div> <!-- /container -->
</body>
</html>
```

`http://localhost:8080/`에 접근하면 관련된 Controller가 있는지 확인하고 없으면 static/index.html을 전송한다.



MemberController.java에 회원 등록 매핑을 추가한다.

- MemberController.java  
```java:MemberController.java
@Controller
public class MemberController {

	...

	@GetMapping(value="/members/new")
	public String createForm() {
		return "members/createMemberFrom";
	}
}
```

- members/createMemberForm.html  
```html.createMemberForm.html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <form action="/members/new" method="post">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
        </div>
        <button type="submit">등록</button>
    </form>
</div> <!-- /container -->
</body>
</html>
```

MemberController에 form의 post요청을 처리하는 메서드를 추가한다.

- MemberController.java  
```java:MemberController.java
@Controller
public class MemberController {

	...

    @PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }
}
```
폼 데이터를 받기 위한 클래스도 추가한다.

- MemberForm.java  
```java:MemberForm.java
public class MemberForm {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
`http://localhost:8080/members/new`에서 회원을 등록하면 홈 화면으로 돌아온다.  
다음은 회원 목록 확인 페이지를 만들어보자.  

- members/memberList.html  
```html:memberList.html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <table>
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```
MemberController에 GetMapping을 추가한다.  
- MemberController.java  
```java:MemberConroller.java
@Controller
public class MemberController {

	...

    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
}
```
`http://localhost:8080/members`에서 회원 목록을 확인할 수 있다.