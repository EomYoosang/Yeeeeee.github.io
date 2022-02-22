---
title: 빌드&실행하기
description:
categories:
 - spring-basic
tags:
---
## 빌드&실행하기

### 빌드
```bash
$ ls -rlth
total 56
drwxr-xr-x@  4 eomyoosang  staff   128B Dec 24 07:00 src
-rw-r--r--@  1 eomyoosang  staff    34B Dec 24 07:00 settings.gradle
-rw-r--r--@  1 eomyoosang  staff   2.7K Dec 24 07:00 gradlew.bat
-rwxr-xr-x@  1 eomyoosang  staff   7.9K Dec 24 07:00 gradlew
drwxr-xr-x@  3 eomyoosang  staff    96B Dec 24 07:00 gradle
-rw-r--r--@  1 eomyoosang  staff   496B Dec 24 07:00 build.gradle
-rw-r--r--@  1 eomyoosang  staff   1.4K Dec 24 07:00 HELP.md
-rw-r--r--@  1 eomyoosang  staff   444B Dec 24 07:00 .gitignore
drwxr-xr-x   3 eomyoosang  staff    96B Dec 30 22:53 out
drwxr-xr-x  10 eomyoosang  staff   320B Dec 31 00:18 build
```
프로젝트 디렉토리는 위와 같이 구성되어있음
```bash
$ ./gradlew build
```
위 명령어를 실행하면 빌드됨
```bash
$ cd build/libs
$ ls -rlth
```
위 명령어를 실행하면 `build/libs` 디렉토리로 가 빌드된 파일을 확인할 수 있음
```bash
total 37240
-rw-r--r--  1 eomyoosang  staff    18M Dec 31 00:18 hello-spring-0.0.1-SNAPSHOT.jar
-rw-r--r--  1 eomyoosang  staff   2.6K Dec 31 00:18 hello-spring-0.0.1-SNAPSHOT-plain.jar
```

### 실행

빌드된 파일은 아래 명령어로 실행 가능
```bash
$ java -jar hello-spring-0.0.1-SNAPSHOT.jar
```
**주의** 인텔리제이에서 프로젝트를 실행하고 있을 경우 사용하는 포트가 겹쳐 오류 발생

### 빌드가 안된다면?
```bash
$./gradlew clean build
```