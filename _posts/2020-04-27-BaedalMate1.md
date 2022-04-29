---
title: [배달메이트] docker와 jenkins를 이용한 스프링부트 프로젝트 배포
description:
categories:
 - baedalmate, spring
tags:
---

# [배달메이트] docker와 jenkins를 이용한 스프링부트 프로젝트 배포
이번 포스트에서는 Docker 환경에서의 Jenkins 빌드 자동화 구축에 대해 알아보겠습니다.  
다음의 글을 참고하며 작성하였습니다.  
[https://dev-overload.tistory.com/40](https://dev-overload.tistory.com/40)  

## 1. Docker 설치  
미리 만들어둔 aws 인스턴스에 docker를 설치합니다.  
```
$ sudo apt-get update 

// 의존성 패키지 설치 
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common 

// Docker 패키지 인증 키 추가 
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 
// Docker 저장소 추가 
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" 

// 저장소 업데이트 
$ sudo apt-get update

// Docker 설치
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

// 사용자를 docker 그룹에 포함
$ sudo usermod -aG docker $USER
```

## 2. Jenkins 컨테이너 준비

```
// jenkins 이미지 확보
$ docker pull jenkins/jenkins:lts

// 이미지 확인
$ docker images

// 컨테이너 적재
$ docker run --name jenkins-docker -d -p 8085:8080 -p 50000:50000 -v /home/ubuntu/docker/jenkins:/var/jenkins_home -u root jenkins/jenkins:lts
```

실행 명령어 옵션의 의미는 각각 아래와 같습니다.  
- -d: 백그라운드 실행
- -p: 컨테이너와 호스트간 포트 연결
- -v: 이미지의 /var/jenkins_home 디렉토리를 호스트 내에 마운트


`docker ps` 명령어를 통해 잘 실행됐는지 확인합니다.  

aws 퍼블릭 ip(`{ip}:8085`로 접속하여 설치화면을 확인합니다.  
<img alt="설치화면" src="/assets/images/jenkins-install.png" />  

password는 마운트 된 디렉토리에서 확인합니다.  

```
$ sudo cat /home/ubuntu/docker/jenkins/secrets/initialAdminPassword
```

Install Suggested Plugins를 눌러 설치를 진행합니다.  

<img alt="플러그인 설치" src="/assets/images/jenkins-plugin.png" />

설치 후 계정을 생성하고 로그인합니다.  

서비스를 배포할 서버에 접근하기 위해서 Publish Over SSH 플러그인을 설치합니다.  

- Jenkins 관리 > Plugin 관리

<img alt="ssh plugin" src="/assets/images/jenkins-ssh.png" />

## Jenkins, Github - SSH 키 생성 등록

Jenkins에서 깃허브에 접근하기 위해서 SSH 키를 깃허브에 등록합니다.  
Docker에서 Jenkins를 적재할 때, -v 옵션으로 /home/ubuntu/docker/jenkins/에 마운트를 해둔 상태이므로 호스트 PC에서 ssh키 생성을 대행할 수 있습니다.  

아래 명령어를 통해 ssh 키를 생성합니다.  

```
$ sudo mkdir /home/ubuntu/docker/jenkins/.ssh 
$ sudo chmod 700 /home/ubuntu/docker/jenkins/.ssh 
$ sudo ssh-keygen -t rsa
```
진행하면 아래의 저장 경로와 키파일 이름을 설정하는 구간이 나옵니다. 다음과 같이 지정합니다. Enter file in which to save the key (/root/.ssh/id_rsa): /home/ubuntu/docker/jenkins/.ssh/id_rsa


## GitHub에 Jenkins의 Public SSH Key 등록

이제 발급받은 ssh key를 깃허브에 등록합니다.  

```
$ cat /home/jenkins/.ssh/id_rsa.pub
```

## Jenkins에 Service Server SSH 접근 설정

1. Jenkins 대시보드에서 Jenkins 관리 > 시스템 설정에 들어갑니다. 
2. 스크롤을 아래로 내려 Publish over SSH 항목을 찾습니다.

<img alt="public over ssh" src="/assets/images/jenkins-public-over-ssh.png" />
위와 같이 배포 서버 정보를 입력합니다.  
하단의 `고급`을 눌러 서버에 접속할 수 있는 key나 계정정보를 입력할 수 있습니다.  





