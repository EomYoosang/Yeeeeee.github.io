---
title: 자바스크립트7 - 알아두면 유용한 문법
description:
categories:
 - javascript
tags:
---

# 알아두면 유용한 문법

## 비구조화 할당

객체 안에 있는 값을 추출해 변수 혹은 상수로 바로 선언

```javascript
const object = { a: 1, b: 2 };

const { a, b } = object;

console.log(a); // 1
console.log(b); // 2
```

함수의 파라미터에도 비구조화 할당

```javascript
const object = { a: 1, b: 2 };

function print({ a, b }) {
  console.log(a);
  console.log(b);
}

print(object);
```

### 비구조화 할당시 기본값 설정

```javascript
const object = { a: 1 };

function print({ a, b = 2 }) {
  console.log(a);
  console.log(b);
}

print(object);
// 1
// 2
```

b = 2가 없을 경우 `undefined` 출력

### 비구조화 할당시 이름 변경

``` javascript
const animal = {
  name: '멍멍이',
  type: '개'
};

// const nickname = animal.name;
const { name: nickname } = animal
console.log(nickname); // 멍멍이
```

### 배열 비구조화 할당

```javascript
const array = [1];
const [one, two = 2] = array;

console.log(one);
console.log(two);
```

### 깊은 값 비구조화 할당

다음의 예시

```javascript
const deepObject = {
  state: {
    information: {
      name: 'velopert',
      languages: ['korean', 'english', 'chinese']
    }
  },
  value: 5
};
```

name, languages, value 값들을 밖으로 꺼내고 싶다면?

1. 비구조화 할당 문법을 두 번 사용

``` javascript
const deepObject = {
  state: {
    information: {
      name: 'velopert',
      languages: ['korean', 'english', 'chinese']
    }
  },
  value: 5
};

const { name, languages } = deepObject.state.information;
const { value } = deepObject;

const extracted = {
  name,
  languages,
  value
};

console.log(extracted); // {name: "velopert", languages: Array[3], value: 5}
```

이 때 extracted 객체 선언은 다음과 같음

```javascript
const extracted = {
  name: name,
  languages: languages,
  value: value
}
```

2. 한번에 모두 추출

```javascript
const deepObject = {
  state: {
    information: {
      name: 'velopert',
      languages: ['korean', 'english', 'chinese']
    }
  },
  value: 5
};

const {
  state: {
    information: { name, languages }
  },
  value
} = deepObject;

const extracted = {
  name,
  languages,
  value
};

console.log(extracted);
```

## spread 와 rest

ES6에서 도입

### spread

객체 혹은 배열을 펼칠 수 있음

```javascript
const slime = {
  name: '슬라임'
};

const cuteSlime = {
  ...slime,
  attribute: 'cute'
};

const purpleCuteSlime = {
  ...cuteSlime,
  color: 'purple'
};

console.log(slime);
console.log(cuteSlime);
console.log(purpleCuteSlime);
```

여기서 `...`가 spread 연산자

아래는 배열에서 사용

```
const animals = ['개', '고양이', '참새'];
const anotherAnimals = [...animals, '비둘기'];
console.log(animals);
console.log(anotherAnimals);
```

### rest

```javascript
const purpleCuteSlime = {
  name: '슬라임',
  attribute: 'cute',
  color: 'purple'
};

const { color, ...rest } = purpleCuteSlime;
console.log(color);
console.log(rest);
```

`const { color, ...rest } = purpleCuteSlime`에서 color 값을 제외한 값이 들어감

__추출한 값의 이름이 꼭 rest일 필요는 없음__

여러 값을 없애고 싶다면 여러번 사용

```javascript
const purpleCuteSlime = {
  name: '슬라임',
  attribute: 'cute',
  color: 'purple'
};

const { color, ...cuteSlime } = purpleCuteSlime;
console.log(color);
console.log(cuteSlime);

const { attribute, ...slime } = cuteSlime;
console.log(attribute);
console.log(slime);
```

### 배열에서의 rest

```javascript
const numbers = [0, 1, 2, 3, 4, 5, 6];

const [one, ...rest] = numbers;

console.log(one);
console.log(rest);
```

원하는 값을 밖으로 꺼내고, 나머지 값을 rest 안에 넣음

### 함수 파라미터에서의 rest

```javascript
function sum(...rest) {
  return rest.reduce((acc, current) => acc + current, 0);
}

const result = sum(1, 2, 3, 4, 5, 6);
console.log(result); // 21
```

함수의 파라미터가 몇 개가 될 지 모를 때 유용

```javascript
function sum(...rest) {
  return rest.reduce((acc, current) => acc + current, 0);
}

const numbers = [1, 2, 3, 4, 5, 6];
const result = sum(...numbers);
console.log(result);
```

위처럼 함수 인자에 spread를 사용 가능

## Scope

scope란 변수 혹은 함수를 선언하게 될 때 해당 변수 또는 함수가 유요한 범위를 의미

- **Global Scope**: 코드의 모든 범위에서 사용 가능
- **Function Scope**: 함수 안에서만 사용 가능
- **Block Scope**: if, for, switch 등 특정 블록 내부에서만 사용이 가능

### Hoisting

자바스크립트에서 아직 선언되지 않은 함수/변수를 "끌어올려서" 사용할 수 있는 자바스크립트의 작동 방식

```javascript
myFunction();

function myFunction() {
  console.log('hello world!');
}
```

`myFunction`함수를 선언하기 전에 `myFunction()`을 호출

자바스크립트 엔진은 다음과 같이 받아들임

```javascript
function myFunction() {
  console.log('hello world!');
}

myFunction();
```

**Hoisting이 발생하는 코드는 이해하기 어려우며 유지보수도 힘듦으로 방지해야함**

**`var`대신 `const`,`let`을 위주로 사용하기**

