---
title: "비동기 함수의 동작과 콜백"
date: 2021-12-11 00:00:00 +0900
categories: [knowledge, frontend]
tags: [javascript, callback]
---

## 비동기 함수와 실행 시점

비동기 함수는 호출한 쪽에서 **실행 결과를 기다리지 않아도 되는 함수**.  
싱글 스레드 환경에서 동작하는 언어에서 비동기 처리는 UI 응답성을 유지하는 데 중요한 역할을 한다.

하지만 비동기 함수는 인간에게 직관적이지 않게 느껴질 수 있다.

### 비동기 함수란 무엇인가

비동기 함수는 호출되더라도 즉시 실행이 완료되지 않는다.  
대신 작업을 등록해 두고, 나중에 특정 시점에 실행된다.

이로 인해 코드의 작성 순서와 실행 순서가 항상 일치하지 않을 수 있다.

V8 엔진(Chrome 환경)에서는 다음과 같은 흐름으로 코드가 처리된다.

1. 동기 코드는 콜 스택(Call Stack)에서 즉시 실행된다.
2. setTimeout, 이벤트, 비동기 I/O와 같은 작업은 실행 요청만 등록하고 Web APIs 영역으로 위임된다.
3. 해당 작업이 완료되면 콜백은 태스크 큐(Task Queue) 로 이동한다.
4. 콜 스택이 완전히 비어 있을 때, 이벤트 루프(Event Loop)가 큐에서 작업을 하나 꺼내 실행한다.

이 구조 때문에 비동기 코드는 뒤로 밀리는 것처럼 보이지만,  
실제로는 동기 코드의 실행을 방해하지 않도록 분리되어 처리된다.

<br />

### 비동기 함수에서 return이 동작하지 않는 이유

```javascript
function findUser(id) {
  let user;

  setTimeout(() => {
    user = {
      id,
      name: "User" + id,
      email: id + "@test.com",
    };
  }, 100);

  return user;
}

const user = findUser(1);
console.log(user); // undefined
```

setTimeout 내부의 코드는 아직 실행되지 않았지만, 함수는 이미 종료되었기 때문이다.

이때 문제는 값이 아니라 실행 시점이 다르다는 점이다.  
비동기 함수에서는 함수가 끝나는 시점과 작업이 완료되는 시점이 분리된다.

<br />

### 콜백으로 시점을 전달하는 방식

문제를 해결하기 위해 비동기 작업에서는 결과를 return하지 않고 콜백 함수로 전달한다.

```javascript
function findUserAndCallback(id, cb) {
  setTimeout(() => {
    const user = {
      id,
      name: "User" + id,
      email: id + "@test.com",
    };
    cb(user);
  }, 100);
}

findUserAndCallback(1, (user) => {
  console.log(user); // { id: 1, name: "User1", email: "1@test.com" }
});
```

#### - 콜백의 한계

콜백 방식은 비동기 문제를 해결할 수 있지만,
중첩이 깊어질수록 가독성과 유지보수성이 급격히 떨어진다.

이로 인해 Promise, async/await와 같은 표현 방식이 등장하게 되었다.

#### - 현대의 대안 (Promise, async/await)

이들은 콜백을 제거하기 위한 것이 아니라,  
**비동기 작업의 실행 시점을 더 읽기 쉬운 형태로 표현하기 위한 문법**이다.

Promise와 async/await 또한 비동기 작업이 완료되는 시점을 기준으로 다음 로직을 실행한다는 점에서  
콜백과 본질적으로 동일한 구조를 가진다.

<br />