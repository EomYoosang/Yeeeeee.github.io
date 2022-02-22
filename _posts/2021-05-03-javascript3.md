---
title: 자바스크립트3
description:
categories:
 - javascript
tags:
---
# Javascript3

## 조건문

특정 조건을 만족시킬 때 코드를 실행시킵니다.

### if 문



기본적인 조건문

```javascript
const a = 1;
if (a + 1 === 2) {
  console.log('a + 1 이 2 입니다.');
}
```

a를 0으로 바꾼다면?

```javascript
const a = 0;
if (a + 1 === 2) {
  console.log('a + 1 이 2 입니다.');
}
```

출력X

### if-else if 문

```javascript
if (a === 5) {
  console.log('5입니다!');
} else if (a === 10) {
  console.log('10입니다!');
} else {
  console.log('5도 아니고 10도 아닙니다.');
}
```

결과는

```
a = 5 -> "5입니다!"
a = 7 -> "5도 아니고 10도 아닙니다."
a = 10 -> "10입니다!"
```

### Switch/case 문

특정 값이 무엇이냐에 따라 다른 작업

```javascript
const device = 'iphone';

switch (device) {
  case 'iphone':
    console.log('아이폰!');
    break;
  case 'ipad':
    console.log('아이패드!');
    break;
  case 'galaxy note':
    console.log('갤럭시 노트!');
    break;
  default:
    console.log('모르겠네요..');
}
```

device의 값에 따라 다른 결과 출력

`default:`는 준비한 값이 없을 때 실행

## 함수

```javascript
function add(a,b){
    return a + b;
}
const sum = add(1,2);
```

함수는 `function`키워드를 사용합니다. 함수에서 어떤 값을 받아올지 정해주는데 이를 파라미터(매개변수)라고 합니다.

함수의 결과물은 `return`을 사용하여 지정합니다. 또한 `return` 사용 시 함수가 끝납니다.



## 객체

객체는 우리가 변수 혹은 상수를 사용할 때 하나의 이름에 여러 종류의 값을 넣을 수 있게 해줍니다.

```javascript
const dog = {
    name: '멍멍이',
    age: 2
};
```

객체를 선언할 때는 `{}` 안에 원하는 값들을 넣어주며 값들은 `키: 원하는 값`으로 이루어집니다.

키에 해당하는 부분은 공백이 없어야합니다. 공백이 있어야 하는 상황엔 `''`로 감싸 문자열로 넣어줍니다.

```javascript
const sample = {
  'key with space': true
};
```

### 함수에서 객체를 파라미터로 받기

```javascript
function print(animal) {
    const test = '${animal.name}는 ${animal.age}살';
    console.log(test);
}
print(dog)
```

결과는 `멍멍이는 2살`입니다.

함수 내부 값에는 `.`을 통해 접근합니다.

매번 `animal`을 통해 접근하기 귀찮다면 다음과 같이 바꿀 수 있습니다.

```javascript
function print(animal) {
    const {name, age} = dog;
    const test = '${name}는 ${age}살';
    console.log(test);
}
```

다음과 같이도 가능합니다.

```javascript
function print({name,age}) {
    const test = '${name}는 ${age}살';
    console.log(test);
}
```

### 객체 안에 함수 넣기

객체 안에 함수를 넣을 수 있습니다.

```javascript
const dog = {
  name: '멍멍이',
  sound: '멍멍!',
  say: function say() {
    console.log(this.sound);
  }
};

dog.say();
```

이때 `this`는 자신이 속한 객체를 가르킵니다.

객체 안에 함수를 넣을 때, 화살표 함수로 선언한다면 제대로 작동하지 않습니다.

### Getter / Setter 함수

객체를 만들고 객체 안의 값을 수정할 수 있습니다. Getter함수와 Setter함수를 사용하게 되면 특정 값을 바꾸려고 하거나, 특정 값을 조회하려고 할 때 우리가 우너하는 코드를 실행 시킬 수 있습니다.

#### Getter 함수

```javascript
const numbers = {
  a: 1,
  b: 2,
  get sum() {
    console.log('sum 함수가 실행됩니다!');
    return this.a + this.b;
  }
};

console.log(numbers.sum);
numbers.b = 5;
console.log(numbers.sum);
```

결과는 다음과 같습니다.

```
sum 함수가 실행됩니다!
3
sum 함수가 실행됩니다!
6
```

getter 함수 안의 console.log가 실행되었습니다.

#### Setter 함수

```javascript
const numbers = {
  _a: 1,
  _b: 2,
  sum: 3,
  calculate() {
    console.log('calculate');
    this.sum = this._a + this._b;
  },
  get a() {
    return this._a;
  },
  get b() {
    return this._b;
  },
  set a(value) {
    console.log('a가 바뀝니다.');
    this._a = value;
    this.calculate();
  },
  set b(value) {
    console.log('b가 바뀝니다.');
    this._b = value;
    this.calculate();
  }
};

console.log(numbers.sum);
numbers.a = 5;
numbers.b = 7;
numbers.a = 9;
console.log(numbers.sum);
console.log(numbers.sum);
console.log(numbers.sum);
```

결과는 다음과 같습니다.

```
3
a가 바뀝니다.
calculate
b가 바뀝니다.
calculate
a가 바뀝니다.
calculate
16
16
16
```

값을 변경할 때 자동으로 sum값을 연산합니다.

## 배열

배열은 여러개의 항목들이 들어있는 리스트입니다.

배열 안에는 어떤 값이던지 넣을 수 있습니다.

```javascript
const array = [1, 2, 3, 4, 5];
const objects = [{name: '멍멍이'}, {name: '야옹이'}];
```

n 번째 항목은 `obejcts[n]` 으로 조회할 수 있습니다.

### 새 항목 추가하기

`push`함수를 사용하여 배열에 내용을 추가할 수 있습니다.

```javascript
const objects = [{ name: '멍멍이' }, { name: '야옹이' }];
objects.push({
    name: '멍뭉이'
})
```

### 배열의 크기

`length`를 사용해 배열의 크기를 알 수 있습니다.

```javascript
const objects = [{ name: '멍멍이' }, { name: '야옹이' }];
console.log(objects.length);
```

```
2
```

## 반복문

### for 문

``` javascript
for (초기 구문; 조건 구문; 변화 구문) {
    코드
}
```

```javascript
for (let i = 10; i > 0; i--){
    console.log(i);
}
```

```
10
9
8
7
6
5
4
3
2
1
```

배열과 for문을 함께 활용할 수 있습니다.

```javascript
const names = ['멍멍이', '야옹이', '멍뭉이'];

for (let i = 0; i < names.length; i++) {
  console.log(names[i]);
}
```

```
멍멍이
야옹이
멍뭉이
```

### while

조건이 참이라면 반복

``` 
let i = 0;
while (i < 10) {
  console.log(i);
  i++;
}
```

### for...of

```javascript
let numbers = [10, 20, 30, 40, 50];
for (let number of numbers) {
  console.log(number);
}
```

### for...in

객체에 사용합니다.

```javascript
const doggy = {
  name: '멍멍이',
  sound: '멍멍',
  age: 2
};

for (let key in doggy) {
  console.log(`${key}: ${doggy[key]}`);
}
```

### break & continue

break가 실행되면 반복을 종료하고

continue가 실행되면 반복을 계속합니다.

```javascript
for (let i = 0; i < 10; i++) {
  if (i === 2) continue; // 다음 루프를 실행
  console.log(i);
  if (i === 5) break; // 반복문을 끝내기
}
```

## Reference

https://learnjs.vlpt.us/basics/08-loop.html