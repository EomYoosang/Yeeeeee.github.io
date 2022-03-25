---
title: 스프링 프로젝트 생성
description:
categories:
 - spring-basic
tags:
---
## 스프링 프로젝트 생성
### 사전 준비
1. Java 11 설치
2. IDE: **IntelliJ** or Eclipse 설치

### 1. 스프링부트 	스타터 사이트에서 스프링 프로젝트 생성
[https://start.spring.io](https://start.spring.io/)  
<img alt="StartSpring" src="/assets/images/spring-initializer.png" />  

설정 후 **GENERATE**  
다운받아 압축해제
### 2. 인텔리제이 프로젝트 오픈
인텔리제이 프로젝트에 **build.gradle** 추가  
<img alt="OpenProject" src="/assets/images/open-project.png" />  


### build.gradle
```java
// 스프링 버전
plugins {
	id 'org.springframework.boot' version '2.6.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}
// 그룹
group = 'eys'
version = '0.0.1-SNAPSHOT'
// 자바 버전
sourceCompatibility = '11'

// mavenCentral에서 라이브러리를 다운받음
repositories {
	mavenCentral()
}

// https://start.spring.io에서 추가한 dependencies
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

```
### **스프링부트란?**
**스프링을 더 쉽게 이용하기 위한 도구**
스프링을 이용하여 개발할 때, 설정해야할 것들이 많아 진입 장벽이 높지만, 스프링부트를 이용하면 간단하게 프로젝트를 생성할 수 있다.