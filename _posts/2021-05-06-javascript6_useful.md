---
title: 자바스크립트6 - 알아두면 유용한 문법
description:
categories:
 - javascript
tags:
---

# 알아두면 유용한 문법

## 삼항연산자



ES6 문법은아닙니다.

```javascript
const array = [];
let text = '';
if (array.length === 0) {
  text = '배열이 비어있습니다.';
} else {
  text = '배열이 비어있지 않습니다.';
}
console.log(text);
```

요런 if-else문은 삼항연산자로 쓰기 좋습니다.

```javascript
const array = [];
let text = array.length === 0 ? '배열이 비어있습니다' : '배열이 비어있지 않습니다.';

console.log(text);
```

사용법

```javascript
조건 ? true일 때 : false일 때
```

줄바꿈 가능

```javascript
const array = [];
let text = array.length === 0 
  ? '배열이 비어있습니다' 
  : '배열이 비어있지 않습니다.';

console.log(text);
```

중첩 가능

```javascript
const condition1 = false;
const condition2 = false;

const value = condition1 
  ? '와우!' 
  : condition2 
    ? 'blabla' 
    : 'foo';

console.log(value);
```

가독성은 안좋으니까 쓰지 않도록

## Truthy and Falsy

Falsy: 대충 거짓인 값들 ex) null, undefined, 0, NaN, ''

Trythy: 대충 참인 값들

```javascript
console.log(!undefined);
console.log(!null);
console.log(!0);
console.log(!'');
console.log(!NaN);
```

전부 false 가 출력됨

## 단축 평가 (short-circuit evaluation) 논리 계산법

이전에 배운 연산자로 코드를 단축시킬 수 있습니다.

### &&연산자로 코드 단축

```javascript
return A && B
```

A가 true 라면 B가 결과 값

A가 false 라면 A가 결과 값

아래는 예시

```javascript
console.log(true && 'hello'); // hello
console.log(false && 'hello'); // false
console.log('hello' && 'bye'); // bye
console.log(null && 'hello'); // null
console.log(undefined && 'hello'); // undefined
console.log('' && 'hello'); // ''
console.log(0 && 'hello'); // 0
console.log(1 && 'hello'); // hello
console.log(1 && 1); // 1
```

### ||연산자로 코드 단축

```javascript
return A || B
```

A가 false 라면 B가 결과 값

A가 true 라면 A가 결과 값

```javascript
const namelessDog = {
  name: ''
};

function getName(animal) {
  const name = animal && animal.name;
  return name || '이름이 없는 동물입니다.';
}

const name = getName(namelessDog);
console.log(name); // 이름이 없는 동물입니다.
```

위와 같이 사용 가능

## 함수 기본 파라미터

원의 넓이를 구하는 함수

```javascript
function calculateCircleArea(r) {
  return Math.PI * r * r;
}
```

여기서 r을 안넣으면?

```javascript
const area = calculateCircleArea();
console.log(area); // NaN
```

결과는 NaN

ES6부터 다음과 같이 가능

```javascript
function calculateCircleArea(r = 1) {
  return Math.PI * r * r;
}
```

화살표 함수에서도 사용 가능

```javascript
const calculateCircleArea = (r = 1) => Math.PI * r * r;
```

## 조건문 똑똑하게 쓰기

비교해야할 값이 많아진다면?

```javascript
function isAnimal(text) {
  return (
    text === '고양이' || text === '개' || text === '거북이' || text === '너구리'
  );
}

console.log(isAnimal('개')); // true
console.log(isAnimal('노트북')); // false
```

코드는 길어진다.

배열의 includes 함수를 사용하여 간단하게 만들 수 있다.

```javascript
function isAnimal(name) {
  const animals = ['고양이', '개', '거북이', '너구리'];
  return animals.includes(name);
}

console.log(isAnimal('개')); // true
console.log(isAnimal('노트북')); // false
```

만약 값에 따라 리턴값이 달라진다면 switch 문을 사용하여도 좋다.

또한 return이므로 break를 생략한다.

```javascript
function getSound(animal) {
  switch (animal) {
    case '개':
      return '멍멍!';
    case '고양이':
      return '야옹~';
    case '참새':
      return '짹짹';
    case '비둘기':
      return '구구 구 구';
    default:
      return '...?';
  }
}

console.log(getSound('개')); // 멍멍!
console.log(getSound('비둘기')); // 구구 구 구
```

앞에 배운 단축 평가 논리 계산법을 이용해 다음과 같이 단순화 가능하다.

```javascript
function getSound(animal) {
  const sounds = {
    개: '멍멍!',
    고양이: '야옹~',
    참새: '짹짹',
    비둘기: '구구 구 구'
  };
  return sounds[animal] || '...?';
}

console.log(getSound('개')); // 멍멍!
console.log(getSound('비둘기')); // 구구 구 구
```

값에 따라 실행하는 코드가 다를 경우에는 객체에 코드를 넣는다.

```javascript
function makeSound(animal) {
  const tasks = {
    개() {
      console.log('멍멍');
    },
    고양이() {
      console.log('고양이');
    },
    비둘기() {
      console.log('구구 구 구');
    }
  };
  if (!tasks[animal]) {
    console.log('...?');
    return;
  }
  tasks[animal]();
}

getSound('개');
getSound('비둘기');
```