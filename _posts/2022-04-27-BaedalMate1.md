---
title: Docker&Jenkins
description:
categories:
 - baedalmate
tags:
---

# docker와 jenkins를 이용한 스프링부트 프로젝트 배포

이번 포스트에서는 Docker 환경에서의 Jenkins 빌드 자동화 구축에 대해 알아보겠습니다.  
다음의 글을 참고하며 작성하였습니다.  
[https://dev-overload.tistory.com/40](https://dev-overload.tistory.com/40)  

## 1. Docker 설치 및 Jenkins 컨테이너 준비

로컬 환경에 docker를 설치합니다.  
**로컬 환경**

- m1 맥북 에어 13인치
- os: mac os moterey 12.3.1

> 기존 aws 프리티어 인스턴스에 설치하려했으나 빌드과정에서 컨테이너가 터졌습니다. 참고하세요.

homebrew를 사용하여 살치해도 되지만 [도커 사이트](https://www.docker.com/get-started/)에서 설치하였습니다.  

설치 후 터미널에 아래 명령어를 입력해 jenkins 이미지를 pull하고 컨테이너를 적재합니다.  

```
// jenkins 이미지 확보
$ docker pull jenkins/jenkins:lts

// 이미지 확인
$ docker images

// 컨테이너 적재
$ docker run --name jenkins-docker -d -p 8085:8080 -p 50000:50000 -v /users/eomyoosang/jenkins:/var/jenkins_home -u root jenkins/jenkins:lts
```

실행 명령어 옵션의 의미는 각각 아래와 같습니다.  

- -d: 백그라운드 실행
- -p: 컨테이너와 호스트간 포트 연결
- -v: 이미지의 /var/jenkins_home 디렉토리를 호스트 내에 마운트

`docker ps` 명령어를 통해 잘 실행됐는지 확인합니다.  

`localhost:8085`로 접속하여 설치화면을 확인합니다.  
<img alt="설치화면" src="/assets/images/jenkins-install.png" />  

password는 마운트 된 디렉토리에서 확인합니다.  

```
$ cat /users/eomyoosang/jenkins/secrets/initialAdminPassword
```

Install Suggested Plugins를 눌러 설치를 진행합니다.  

<img alt="플러그인 설치" src="/assets/images/jenkins-plugin.png" />

설치 후 계정을 생성하고 로그인합니다.  

서비스를 배포할 서버에 접근하기 위해서 Publish Over SSH 플러그인을 설치합니다.  

- Jenkins 관리 > Plugin 관리

<img alt="ssh plugin" src="/assets/images/jenkins-ssh.png" />

## 2. Jenkins, Github - SSH 키 생성 등록

Jenkins에서 깃허브에 접근하기 위해서 SSH 키를 깃허브에 등록합니다.  
Docker에서 Jenkins를 적재할 때, -v 옵션으로 /users/eomyoosang/jenkins/에 마운트를 해둔 상태이므로 호스트 PC에서 ssh키 생성을 대행할 수 있습니다.  

아래 명령어를 통해 ssh 키를 생성합니다.  

```
$ sudo mkdir /users/eomyoosang/jenkins/.ssh 
$ sudo chmod 700 /users/eomyoosang/jenkins/.ssh 
$ sudo ssh-keygen -t rsa
```

진행하면 아래의 저장 경로와 키파일 이름을 설정하는 구간이 나옵니다. 다음과 같이 지정합니다. Enter file in which to save the key (/root/.ssh/id_rsa): /users/eomyoosang/jenkins/.ssh/id_rsa

## 3. GitHub에 Jenkins의 Public SSH Key 등록

이제 발급받은 ssh key를 깃허브에 등록합니다.  

```
$ cat /users/eomyoosang/jenkins/.ssh/id_rsa.pub
```

## 4. Jenkins에 Service Server SSH 접근 설정

1. Jenkins 대시보드에서 Jenkins 관리 > 시스템 설정에 들어갑니다. 
2. 스크롤을 아래로 내려 Publish over SSH 항목을 찾습니다.

<img alt="public over ssh" src="/assets/images/jenkins-public-over-ssh.png" />  

위와 같이 배포 서버 정보를 입력합니다.  
하단의 `고급`을 눌러 서버에 접속할 수 있는 key나 계정정보를 입력할 수 있습니다.  

## 5. GitHub 저장소 추가

<img alt="깃허브" src="/assets/images/github-ssh.png" />  

깃허브 리포지토리의 `CODE`를 눌러 SSH 저장소 주소를 복사합니다.  

## 6. Jenkins에 프로젝트 등록

<img alt="젠킨스 대시보드" src="/assets/images/jenkins-dashboard.png" />  

젠킨스 대시보드 좌측에서 `새로운 Item`을 눌러 프로젝트를 등록합니다.  

<img alt="등록" src="/assets/images/jenkins-new-item.png" />  

`Freestyle Project`를 선택하고 하단의 OK를 누릅니다.  

<img alt="소스코드관리" src="/assets/images/소스코드관리.png" />  

소스코드관리 항목에서 git을 체크하고 위에서 복사한 SSH 저장소 주소를 입력합니다.  
Credentials의 Add 버튼을 눌러 접속 계정을 세팅합니다.  

<img alt="add-credentials" src="/assets/images/add-credentials.png" />  

ID, username, private key를 필수적으로 입력해야합니다.  
ID는 Jenkins 컨테이너의 유저 계정입니다. (기본적으로 jenkins입니다.)  
username은 아무 이름이나 지정해도 상관 없습니다.  

private 키는 `/users/eomyoosang/jenkins/.ssh/id_rsa`의 키값을 복사해 붙여넣습니다.  

<img alt="ssh설정" src="/assets/images/jenkins-ssh설정.png" />  

ssh접속이므로 name과 RefSpec, Branch Specifier에 위와 같이 입력합니다.  

- Name: origin
- RefSpec: +refs/pull/*:refs/remotes/origin/pr/*
- Branch Specifier: */**

다음은 Build 항목을 설정합니다. 

<img alt="빌드환경설정1" src="/assets/images/jenkins-build설정1.png" />  
<img alt="빌드환경설정2" src="/assets/images/jenkins-build설정2.png" />

- **Source files**: 전송할 파일의 위치

- **Remote Directory**: 해당 위치로 빌드 파일이 전송된다. /baedalmate/deploy로 설정하였고 위에서 /home/ubuntu 경로를 설정하여 /home/ubuntu/baedalmate/deploy 경로로 전송된다. 배포서버에 미리 디렉토리를 생성한다.  

- **exec command**: 빌드파일 전송 후 실행할 명령어이다. /home/ubuntu/baedalmate/script/init_server.sh를 실행하게 하였고 해당 파일의 내용은 아래와 같다.  
  
  ```
  echo "PID Check..."
  CURRENT_PID=$(ps -ef | grep java | grep jenkins_test_spring* | awk '{print $2}')
  echo "Running PID: {$CURRENT_PID}"
  if "$CURRENT_PID" [ -z CURRENT_PID ] ; then
        echo "Project is not running"
  else
        kill -9 $CURRENT_PID
  sleep 10
  fi
  echo "Deploy Project...."
  nohup java -jar /home/serve/spring_project/target/jenkins_test_spring-0.0.1-SNAPSHOT.jar >> /home/serve/spring_project/logs/jenkins_test_spring.log &
  echo "Done"
  ```

## 8. 프로젝트 빌드&배포

저장 후 프로젝트에 들어간 후 좌측의 Build Now를 클릭해 배포를 진행합니다.  
