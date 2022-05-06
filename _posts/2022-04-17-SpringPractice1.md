---
title: 스프링 프로젝트 생성
description:
categories:
 - spring-practice
tags:
---

# 프로젝트 생성

김영한님의 인프런 강의 `실전! 스프링 부트와 JPA활용 1편`을 보고 공부한 내용을 정리한 자료입니다.  

## 목차

- 프로젝트 생성
- 라이브러리
- View 환경설정
- H2 데이터베이스 설치
- JPA와 DB 설정, 동작확인

## 프로젝트 생성

- 스프링부트 스타터([https://start.spring.io/](https://start.spring.io/))
- 사용 기능: web, thymeleaf, jpa, h2, lomlok, validation
  - groupId: jpabook
  - artifactId: jpashop

**Gradle 전체 설정**

```groovy
plugins {
    id 'org.springframework.boot' version '2.6.6'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'jpabook'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    //JUnit4 추가
    testImplementation("org.junit.vintage:junit-vintage-engine") {
        exclude group: "org.hamcrest", module: "hamcrest-core"
    }
}

test {
    useJUnitPlatform()
}
```

> 테스트 라이브러리로 JUnit4를 사용합니다. 위의 Gradle처럼 코드를 추가해야합니다.

```
//JUnit4 추가
testImplementation("org.junit.vintage:junit-vintage-engine") {
    exclude group: "org.hamcrest", module: "hamcrest-core"
}
```

**lombok 적용**  

1. Prefrences > plugin > lombok 검색 실행 (재시작)
2. Prefrences > Annotation Processors 검색 > Enable annotation processing 체크 (재시작)
3. 임의의 테스트 클래스를 만들고 @Getter, @Setter 확인

**IntelliJ Gradle 대신에 자바 직접 실행**  
최근 IntelliJ 버전은 Gradle로 실행을 하는 것이 기본 설정이다. 이렇게 하면 실행속도가 느리다. 다음과 같이 변경하면 자바로 바로 실행해서 실행속도가 더 빠르다.  

- Preferences -> Build, Execution, Deployment Build -> Tools Gradle  
- Build and run using: Gradle -> IntelliJ IDEA  
- Run tests using: Gradle -> IntelliJ IDEA  

## View 환경설정

**thymeleaf 템플릿 엔진**  

- 스프링 부트 thymeleaf viewName 매핑  
  
  - `resources:templates/` +{ViewName}+ `.html`  

- jpabook.jpashop/controller/HelloController.java
  
  ```
  @Controller
  public class HelloController {
  @GetMapping("hello")
  public String hello(Model model) {
  model.addAttribute("data", "hello!!");
  return "hello";
  }
  }
  ```

thymeleaf 템플릿엔진 동작 확인(hello.html)

- resources/templates/hello.html  
  
  ```
  <!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <title>Hello</title>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
  <p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
  </body>
  </html>
  ```

- static/index.html  
  
  ```
  <!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
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

## h2 데이터베이스 설치

매우 작고 빠른 관계형 데이터베이스.  
주로 메모리에 데이터를 저장하는 용도로 쓰인다.  
개발이나 테스트 용도로 가볍고 편리하며 웹 콘솔을 제공한다.  

- [https://www.h2database.com](https://www.h2database.com)  
- 데이터베이스 파일 생성 방법
  - [http://localhost:8082](http://localhost:8082) 접속
  - `jdbc:h2:~jpashop`(최소 한 번, 세션키 유지한 상태로 실행)
  - `~/jpashop.mv.db` 파일 생성 확인
  - 이후 부터는 `jdbc:h2:tcp://localhost/~jpashop` 접속

## JPA와 DB 설정, 동작확인

h2 데이터베이스에 연결하기 위한 값

- application.yml

```
spring:
datasource:
  url: jdbc:h2:tcp://localhost/~/jpashop
  username: sa
  password:
  driver-class-name: org.h2.Driver
jpa:
  hibernate:
    ddl-auto: create
  properties:
    hibernate:
      # show_sql: true
      format_sql: true

logging.level:
  org.hibernate.SQL: debug
  #org.hibernate.type: trace
```

멤버 엔티티

- jpabook.jpashop/domain/Member.java 

```
@Entity
@Getter @Setter
public class Member {
 @Id
// @Column(name = "id", nullable = false)
 @GeneratedValue
 private Long id;
 private String username;
}
```

멤버 리포지토리

- jpabook.jpashop/repository/MemberRepository.java

```
@Repository
public class MemberRepository {
 @PersistenceContext
 EntityManager em;
 public Long save(Member member) {
 em.persist(member);
 return member.getId();
 }
 public Member find(Long id) {
 return em.find(Member.class, id);
 }
}
```

동작을 확인하기 위해 테스트코드를 작성합니다.

- 단축키 `cmd`+`shift`+`t`로 MemberRepositoryTest 작성

- MemberRepositoryTest.java
  
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class MemberRepositoryTest {
  
      @Autowired MemberRepository memberRepository;
      @Test
      @Transactional
      @Rollback(false)
      public void testMember() throws Exception {
        // given
        Member member = new Member();
        member.setUsername("memberA");
      
        // when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(savedId);
      
        // then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
      
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        Assertions.assertThat(findMember).isEqualTo(member); //JPA 엔티티 동일성 보장
      }
  
    }

### 쿼리 파라미터 로그 남기기

1. application.yml에 다음과 같이 추가

```
logging:
 level:
 org.hibernate.SQL: debug
 org.hibernate.type: trace
```

2. build.gradle에 다음을 추가

```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6
```

> 참고: 쿼리 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하므로, 개발 단계에서는 편하
> 게 사용해도 된다. 하지만 운영시스템에 적용하려면 꼭 성능테스트를 하고 사용하는 것이 좋다.