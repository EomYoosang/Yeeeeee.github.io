---
title: 자바스크립트8 - 비동기
description:
categories:
 - javascript
tags:
---

# 비동기 작업

## Promise

프로미스는 비동기 작업을 조금 더 편하게 처리할 수 있다록 ES6에 도입.

프로미스는 다음과 같이 만듦.

```javascript
const myPromise = new Promise((resolve, reject) => {
  // 구현..
})
```

프로미스가 성공하면 resolve를 호출, 실패하면 reject를 호출

```javascript
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 1000);
});

myPromise.then(n => {
  console.log(n);
});
```

작업이 끝나고 또 다른 작업을 해야할 때는 Promise뒤에 `.then()`을 붙여서 사용

1초 뒤에 실패하는 코드

```javascript
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error());
  }, 1000);
});

myPromise
  .then(n => {
    console.log(n);
  })
  .catch(error => {
    console.log(error);
  });
```

```javascript
function increaseAndPrint(n) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const value = n + 1;
      if (value === 5) {
        const error = new Error();
        error.name = 'ValueIsFiveError';
        reject(error);
        return;
      }
      console.log(value);
      resolve(value);
    }, 1000);
  });
}

increaseAndPrint(0)
  .then(increaseAndPrint)
  .then(increaseAndPrint)
  .then(increaseAndPrint)
  .then(increaseAndPrint)
  .then(increaseAndPrint)
  .catch(e => {
    console.error(e);
  });
```

위 코드처럼 프로미스를 사용하면 비동기 작업의 개수가 많아져도 코드의 깊이가 깊어지지 않음

## async/await

ES8에 해당하는 문법. Promise를 더욱 쉽게 사용할 수 있게 해줌.

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function process() {
  console.log('안녕하세요!');
  await sleep(1000); // 1초쉬고
  console.log('반갑습니다!');
}

process();
```

함수를 선언할 때 함수의 앞부분에 `async`키워드 사용

promise 앞에 await을 넣어주면 해당 프로미스가 끝날 때까지 기다렸다가 다음 작업을 수행

### Error Detection

`async` 함수에서 에러를 발생 시킬때에는 `throw`를 사용하고, 에러를 잡아낼 때에는 `try`/`catch`문 사용

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function makeError() {
  await sleep(1000);
  const error = new Error();
  throw error;
}

async function process() {
  try {
    await makeError();
  } catch (e) {
    console.error(e);
  }
}

process();
```

`promise.all`사용시 등록된 프로미스 중 하나라도 실패시 모두 실패

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const getDog = async () => {
  await sleep(1000);
  return '멍멍이';
};

const getRabbit = async () => {
  await sleep(500);
  return '토끼';
};
const getTurtle = async () => {
  await sleep(3000);
  return '거북이';
};

async function process() {
  const [dog, rabbit, turtle] = await Promise.all([
    getDog(),
    getRabbit(),
    getTurtle()
  ]);
  console.log(dog);
  console.log(rabbit);
  console.log(turtle);
}

process();
```

`promise.race` 사용시 가장 빨리 끝난 프로미스의 결과값만 가져옴

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const getDog = async () => {
  await sleep(1000);
  return '멍멍이';
};

const getRabbit = async () => {
  await sleep(500);
  return '토끼';
};
const getTurtle = async () => {
  await sleep(3000);
  return '거북이';
};

async function process() {
  const first = await Promise.race([
    getDog(),
    getRabbit(),
    getTurtle()
  ]);
  console.log(first);
}

process();
```