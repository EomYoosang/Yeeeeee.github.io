---
title: 회원 관리 예제2
description:
categories:
 - spring-basic
tags:
---

## 회원 관리 예제2
### 스프링 빈과 의존관계
#### 스프링 빈이란?
스프링 IoC 컨테이너가 관리하는 자바 객체를 빈(Bean)이라는 용어로 부른다.
어떤 객체를 new 연산자로 생성하면 그 객체는 빈이 아니고, **ApplicationContext.getBean()** 으로 얻어질 수 있는 객체는 빈이다.  
즉, Bean은 ApplicationContext가 만들어서 그 안에 담고있는 객체를 의미한다.
#### 컴포넌트 스캔과 자동 의존관계 설정
SpringBoot에서는 **Annotation** 을 이용하여 편리하게 빈을 생성한다.

**MemberController**
```java:MemberController.java
package eys.hellospring.controller;

import eys.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
MemberController 클래스 위의 **@Controller** 가 Annotation이다.  
Annotation이 지정된 클래스는 자동으로 빈이 생성된다.

**Annotation**
- `@Component`: 이 Annotation이 있으면 스프링 빈에 자동으로 등록된다.  
아래 Annotation들은 `@Component`를 포함하여 스프링 빈에 자동으로 등록된다.
   - `@Controller`
   - `@Service`
   - `@Repository`

- `@Autowired`: 객체 생성 시점에 스프링 컨테이너에서 해당 스프링 빈을 찾아서 주입한다.


#### 자바 코드로 직접 스프링 빈 등록하기
회원 서비스와 회원 리포지토리의 @Service, @Repository, @Autowired Annotation을 제거하고 진행한다.
```java:SpringConfig.java
@Configuration
public class SpringConfig {
	@Bean
	public MemberService memberService() {
		return new MemberService(memberRepository());
	}

	@Bean
	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}
}

```

#### MemberController의 의존관계
MemberController가 MemberService를, MemberService가 MemberRepository를 이용하므로 각 클래스의 빈이 생성되어야 한다.
```java:MemberController.java

@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
```java:MemberService.java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
	...
}
```

```java:MemberRepository.java
@Repository
public interface MemberRepository {
	...
}
```
```java:MemoryMemberRepository.java

@Repository
public class MemoryMemberRepository implements MemberRepository {
	...
}
```
위 클래스들의 의존관계는 아래와 같다.  
<img alt="의존관계" src="/assets/images/springContainer" width="80%" />  


> **참고** : 스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, 기본으로 싱글톤으로 등록한다.  
따라서 같은 스프링 빈이면 모두 같은 인스턴스이다. 싱글톤이 아니도록 설정할 수 있지만, 대부분의 경우 싱글톤을 사용한다.
> **참고** : XML로 설정하는 방식도 있지만 최근에는 잘 사용하지 않는다.
> **참고** : DI에는 필드 주입, setter 주입, 생성자 주입 3가지 방법이 있다. 의존관계가 실행 중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장한다.
> **참고** : 실무에서는 정형화된 컨트롤러, 서비스, 리포지토리와 같은 코드는 컴포넌트 스캔을 사용하고, 정형화되지 않거나 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다.
> **참고** : `@Autowired`를 통한 DI는 스프링이 관리하는 객체에서만 동작한다. 스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않는다.
> 스프링 입문 공부 기록이므로 DI관련된 내용은 따로 작성할 예정!