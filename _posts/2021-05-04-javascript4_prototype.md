---
title: 자바스크립트4 - 프로토타입
description:
categories:
 - javascript
tags:
---
# Prototype

## 머릿말

https://learnjs.vlpt.us/의 프로토타입 설명이 잘 이해가지 않아 찾아보며 정리해보았습니다!

## 객체 생성자

객체 생성자는 함수를 통해서 새로운 객체를 만들고 그 안에 넣고 싶은 값 혹은 함수를 구현 할 수 있게 해줍니다.

```javascript

function Animal(type, name, sound) {
  this.type = type;
  this.name = name;
  this.sound = sound;
  this.say = function() {
    console.log(this.sound);
  };
}

const dog = new Animal('개', '멍멍이', '멍멍');
const cat = new Animal('고양이', '야옹이', '야옹');

dog.say();
cat.say();

```

이 때 dog 가 가지고 있는 say 함수와 cat 이 가지고 있는 say 함수가 같은 코드를 가지고 있지만 객체 생성시 함수도 새로 만들어짐!

프로토타입은 같은 객체 생성자 함수를 사용할 때, 특정 함수 또는 값을 재사용할 수 있게 해줍니다.

## Prototype

`.prototype.[원하는키] =` 를 이용하여 설정

```javascript
function Animal(type, name, sound) {
  this.type = type;
  this.name = name;
  this.sound = sound;
}

Animal.prototype.say = function() {
  console.log(this.sound);
};

const dog = new Animal('개', '멍멍이', '멍멍');
const cat = new Animal('고양이', '야옹이', '야옹');

dog.say();
cat.say();
```

기존 객체 생성자 안에 있던 say함수를 Prototype으로 만들어 재사용합니다.

더 자세히 들어가서 Prototype은 **Prototype Link**와 **Prototype Object**를 말합니다.

### Prototype Object

함수가 정의될 때는 두 가지 일이 이루어집니다. 1. 해당 함수에 Constructor 자격 부여  
 -Constructor 자격이 부여되면 new를 통해 객체를 만들어낼 수 있게 됩니다.

1.	해당 함수의 Prototype Object 생성 및 연결  
	-function Animal()을 생성하고 prototype 속성을 통해 Prototype Obejct에 접근합니다.  

```javascript
function Animal(type, name, sound) {
    this.type = type;
    this.name = name;
}

Animal.prototype
```

Prototype Object는 일반적인 객체와 같으며 속성으로 **constructor**와 **\_\_proto\_\_** 를 가집니다.

**constructor**는 Prototype Object와 같이 생성되었던 함수를 가리키고 있습니다(위 코드의 function Animal())  
**\_\_proto\_\_** 는 Prototype Link입니다.

위 코드 실행 시<img src = ./result.png width="60%"><br> Animal Prototype Object에 say()가 추가되었습니다.

이번엔 dog를 Animal함수를 통해 생성 후 dog.\_\_proto\_\_ 를 확인해보겠습니다.  
<img src = ./result3.png width="60%"/>

### 마무리

<center><img src=./prototype_chain.png width="90%" /></center><br>

Prototype이 값을 재사용하는 것 뿐만 아니라 객체들이 프로토타입 체인으로 연결되어있음을 알아보았습니다.<br><br><br><br>

### Reference

[https://learnjs.vlpt.us/basics/10-prototype-class.htmlhttps://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67
https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67](https://learnjs.vlpt.us/basics/10-prototype-class.htmlhttps://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67
https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)