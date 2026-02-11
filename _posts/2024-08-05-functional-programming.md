
title: "fxts 라이브러리 기반 함수형 프로그래밍의 실전적 적용"
date: 2024-08-05 00:00:00 +0900
categories: [implementation, react]
tags: [functional programming, fxts]


> [Archive] 이 글은 2024년에 다른 플랫폼에서 작성한 글을 이전한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## fxts로 시작하는 실전 함수형 프로그래밍

### 함수형 프로그래밍의 핵심 원칙

함수형 프로그래밍은 "함수를 값처럼 다루며, 부수효과 없이 데이터를 변환하는" 프로그래밍 패러다임이다.

#### 1. 순수 함수 (Pure Function)

같은 입력에 항상 같은 출력을 반환하며, 외부 상태를 변경하지 않는 함수다.

```typescript
// ❌ 순수하지 않은 함수 (외부 상태 변경)
let total = 0;
function add(n: number) {
  total += n;  // 외부 변수 수정
  return total;
}

// ✅ 순수 함수
function add(a: number, b: number) {
  return a + b;  // 입력만으로 출력 결정, 외부 영향 없음
}
```

#### 2. 부수 효과 없음 (No Side Effects)

함수가 반환값 외에 외부 세계에 영향을 주지 않아야 한다.

```typescript
// ❌ 부수 효과 있음
function updateUser(user: User) {
  user.lastUpdated = new Date();  // 원본 수정
  saveToDatabase(user);  // 외부 시스템 변경
  return user;
}

// ✅ 부수 효과 없음
function updateUser(user: User): User {
  return {
    ...user,
    lastUpdated: new Date()  // 새 객체 반환
  };
}
// 데이터베이스 저장은 별도 함수에서 명시적으로 처리
```

#### 3. 1급 객체 (First-Class Function)

함수를 변수에 할당하고, 인자로 전달하고, 반환값으로 사용할 수 있다.

```typescript
// 함수를 변수에 할당
const double = (n: number) => n * 2;

// 함수를 인자로 전달
const numbers = [1, 2, 3];
numbers.map(double);  // map은 함수를 인자로 받음

// 함수를 반환
const multiplyBy = (factor: number) => (n: number) => n * factor;
const triple = multiplyBy(3);
triple(5);  // 15
```

#### 4. 참조 투명성 (Referential Transparency)

함수 호출을 그 결과값으로 대체해도 프로그램 동작이 변하지 않아야 한다.

```typescript
// ✅ 참조 투명함
const add = (a: number, b: number) => a + b;
const result = add(2, 3) + add(2, 3);
// ↓ 다음과 같이 대체 가능
const result = 5 + 5;

// ❌ 참조 투명하지 않음
let counter = 0;
const increment = () => ++counter;
const result = increment() + increment();  // 3
// increment()를 결과값으로 대체할 수 없음 (매번 다른 값)
```

<br/>

### 왜 이런 원칙이 중요한가?

함수형 프로그래밍은 "어떻게(How)" 구현할지가 아니라 "무엇을(What)" 할지에 집중한다.  
이런 관점의 전환이 코드에 어떤 차이를 만드는지 보자.

```typescript
// 명령형: 어떻게(How) 할지 기술
function getAdultNames(users: User[]): string[] {
  const result = [];
  for (let i = 0; i < users.length; i++) {
    if (users[i].age >= 20) {
      result.push(users[i].name);
    }
  }
  return result;
}

// 함수형: 무엇을(What) 할지 기술
const getAdultNames = (users: User[]) =>
  users
    .filter(u => u.age >= 20)
    .map(u => u.name);
```

명령형 코드는 반복문의 인덱스 `i`, 빈 배열 `result`, `push` 같은 구현 세부사항이 드러난다.  
함수형 코드는 "20세 이상을 필터링하고, 이름만 추출한다"는 의도만 남는다.

코드가 비즈니스 로직의 의도를 그대로 드러내기 때문에, 구현 세부사항보다 본질에 집중할 수 있다.

<br/>

## fxts란?

[fxts](https://github.com/marpple/FxTS)는 TypeScript 기반 함수형 라이브러리로, 지연 평가와 비동기 처리를 자연스럽게 지원한다.

```bash
npm install @fxts/core
```

### 핵심 기능

fxts가 제공하는 핵심 기능들을 통해 함수형 프로그래밍이 실무에서 어떤 문제를 해결하는지 살펴본다.

#### 1. 함수 합성 (Function Composition)

작은 함수들을 조합해 복잡한 로직을 만든다. pipe는 함수들을 순차적으로 연결하는 도구다.

```typescript
import { pipe, map, filter } from '@fxts/core';

// 각각의 작은 함수
const filterAdults = filter((u: User) => u.age >= 20);
const extractNames = map((u: User) => u.name);
const toUpperCase = map((name: string) => name.toUpperCase());

// 조합해서 새로운 함수 생성
const getAdultNamesUpper = pipe(
  filterAdults,
  extractNames,
  toUpperCase
);

// 재사용
const result1 = getAdultNamesUpper(users);
const result2 = getAdultNamesUpper(employees);
```

각 함수는 독립적으로 테스트 가능하고, 다른 조합으로 재사용할 수 있다.  
`filterAdults`는 다른 변환과도 조합할 수 있고, `extractNames`도 성인이 아닌 전체 사용자에게 적용할 수 있다.

#### 2. 지연 평가 (Lazy Evaluation)

필요한 만큼만 계산하는 전략.  
거대한 데이터셋에서 처음 5개만 필요할 때, 전체를 처리하지 않는다.

```typescript
import { pipe, map, filter, take, toArray } from '@fxts/core';

// 100만 개의 데이터가 있다고 가정
const hugeData = range(1, 1000000);

// ✅ 지연 평가: 5개만 실제로 처리됨
const result = pipe(
  hugeData,
  filter(n => n % 2 === 0),
  map(n => n * n),
  take(5),  // 여기서 5개가 완성되면 평가 중단
  toArray
);

// ❌ 즉시 평가: 100만 개 전부 처리
const result = pipe(
  hugeData,
  filter(n => n % 2 === 0),
  toArray,  // 전체 필터링
  arr => arr.slice(0, 5)
);
```

`take(5)` 전까지는 실제 계산이 일어나지 않는다.  
toArray나 reduce 같은 "종료 연산"을 만나야 비로소 필요한 만큼만 평가가 진행된다.  
성능 최적화가 자연스럽게 이뤄진다.

#### 3. 동시성 처리 (Concurrency)

비동기 작업을 동기 코드처럼 작성하면서도 효율적으로 제어할 수 있다.

```typescript
import { pipe, toAsync, map, concurrent, toArray } from '@fxts/core';

const fetchUser = async (id: number) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
};

// 100개의 요청을 3개씩 동시에 실행
const users = await pipe(
  range(1, 100),
  toAsync,
  map(fetchUser),
  concurrent(3),  // 동시 실행 수 제어
  toArray
);
```

`concurrent(3)`은 동시에 3개의 요청만 실행하도록 제한한다.  
서버 부하를 조절하면서도 순차 실행보다 훨씬 빠르다.  
비동기 작업을 제어하기 위한 복잡한 Promise 관리 코드가 필요 없다.

#### 4. 에러 처리 (Error Handling)

함수형 방식으로 에러를 값으로 다룬다. try-catch로 흐름을 끊지 않고 파이프라인을 유지한다.

```typescript
import { pipe, toAsync, map, filter, toArray } from '@fxts/core';

type Result<T> = { ok: true; data: T } | { ok: false; error: Error };

const safeFetch = (id: number) => async (): Promise<Result<User>> => {
  try {
    const user = await fetchUser(id);
    return { ok: true, data: user };
  } catch (error) {
    return { ok: false, error: error as Error };
  }
};

const results = await pipe(
  [1, 2, 3, 4, 5],
  toAsync,
  map(id => safeFetch(id)()),
  toArray
);

// 성공한 것과 실패한 것을 나눠서 처리
const successes = results.filter(r => r.ok);
const failures = results.filter(r => !r.ok);
```

에러가 발생해도 전체 파이프라인이 중단되지 않는다.  
에러를 데이터처럼 다루면 일괄 처리 후 성공/실패를 분류할 수 있다.

#### 5. 메서드 체이닝 (Method Chaining)

pipe 대신 fx를 사용하면 객체 메서드 체이닝 스타일로도 작성할 수 있다.

```typescript
import { fx, toArray } from '@fxts/core';

// pipe 스타일
const result1 = pipe(
  [1, 2, 3, 4, 5],
  filter(n => n % 2 === 0),
  map(n => n * n),
  toArray
);

// 메서드 체이닝 스타일
const result2 = fx([1, 2, 3, 4, 5])
  .filter(n => n % 2 === 0)
  .map(n => n * n)
  .toArray();
```

같은 결과를 내지만, 메서드 체이닝이 더 직관적일 때가 있다.

이런 기능들이 결합되면서 함수형 프로그래밍은 단순히 "코드를 깔끔하게 쓰는 방법"을 넘어선다.  
성능, 에러 처리, 비동기 제어를 자연스럽게 해결하는 도구로써 존재할 수 있다.

<br/>

## 실전 예제

### 예제 0: pipe와 go로 추상화 맞추기

`pipe`는 재사용 가능한 함수를 만들고, `go`는 데이터를 즉시 변환한다.  
같은 로직을 여러 곳에서 쓴다면 `pipe`, 한 번만 쓴다면 `go`를 선택한다.

```typescript
import { pipe, go, map, filter } from '@fxts/core';

const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 17 }
];

// pipe: 재사용 가능한 함수 생성
const getAdultNames = pipe(
  filter((u: User) => u.age >= 20),
  map(u => u.name)
);
const names1 = getAdultNames(users);
const names2 = getAdultNames(otherUsers);  // 재사용

// go: 즉시 실행
const names = go(
  users,
  filter((u: User) => u.age >= 20),
  map(u => u.name)
);
```

`pipe`는 함수를 반환해서 여러 곳에서 재사용하게 만들고,  
`go`는 첫 번째 인자로 데이터를 받아 즉시 결과를 반환해서 일회성 변환에 적합하다.  
동일한 변환 로직이 2번 이상 등장하면 `pipe`로 추출해서 중복을 제거한다.

### 예제 1: API 데이터 가공

실무에서 가장 흔한 패턴이다. API에서 받은 데이터를 필터링하고 UI에 맞게 변환한다.  
명령형으로 작성하면 for 루프와 if 문이 뒤섞이지만, 함수형으로 작성하면 각 단계가 명확히 분리된다.

```typescript
import { pipe, filter, map, toArray } from '@fxts/core';

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// 각 단계를 명확한 함수로 분리
const onlyInStock = filter((p: Product) => p.inStock);
const toDisplayName = map((p: Product) => ({
  id: p.id,
  display: `${p.name} ($${p.price})`
}));

// 추상화 수준이 일치하는 조합
const getProductList = pipe(
  onlyInStock,
  toDisplayName,
  toArray
);

const products = await fetchProducts();
const displayList = getProductList(products);
```

`onlyInStock`과 `toDisplayName`은 독립적으로 테스트하고 다른 파이프라인에서 재사용한다.  
"재고 있는 것만", "표시용 이름으로 변환"이라는 의도가 함수 이름으로 드러나서 주석 없이도 읽힌다.

### 예제 2: 폼 검증

명령형 검증은 if 문과 에러 배열 관리가 뒤섞여서 복잡해진다.  
함수형으로 작성하면 "유효하지 않은 필드를 찾아서 에러 메시지로 변환한다"는 의도가 그대로 드러난다.

```typescript
import { pipe, filter, map, toArray } from '@fxts/core';

interface Field {
  name: string;
  value: string;
  required: boolean;
}

const isInvalid = (f: Field) => f.required && !f.value.trim();
const toError = (f: Field) => ({ 
  field: f.name, 
  message: `${f.name}은(는) 필수입니다` 
});

const validateForm = pipe(
  filter(isInvalid),
  map(toError),
  toArray
);

const fields = [
  { name: '이름', value: '', required: true },
  { name: '이메일', value: 'a@b.com', required: true }
];

const errors = validateForm(fields);
// [{ field: '이름', message: '이름은(는) 필수입니다' }]
```

검증 규칙(`isInvalid`)과 에러 메시지 생성(`toError`)이 분리되어서 각각 수정하기 쉽다.  
새로운 검증 규칙을 추가할 때도 파이프라인에 함수 하나만 추가하면 된다.

### 예제 3: 비동기 데이터 처리

`Promise.all`은 모든 요청을 동시에 보내서 서버에 부담을 주고,  
순차 실행은 너무 느리다. `concurrent`로 동시 요청 수를 제어해서 둘 다 해결한다.

```typescript
import { pipe, toAsync, map, filter, concurrent, toArray } from '@fxts/core';

const fetchUser = async (id: number) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
};

const enrichUser = async (user: User) => {
  const detail = await fetch(`/api/users/${user.id}/detail`).then(r => r.json());
  return { ...user, ...detail };
};

// 비동기 파이프라인
const getActiveUsers = pipe(
  toAsync,
  map(fetchUser),
  concurrent(3),  // 동시에 3개씩 요청
  map(enrichUser),
  concurrent(3),
  filter(u => u.isActive),
  toArray
);

const userIds = [1, 2, 3, 4, 5];
const users = await getActiveUsers(userIds);
```

`concurrent(3)`이 동시 실행 수를 3개로 제한해서 서버 부하를 조절하면서도 순차 실행보다 훨씬 빠르다.  
비동기 작업도 동기 코드처럼 읽히고, Promise 체이닝이나 async/await의 중첩 없이 깔끔하다.

### 예제 4: React 컴포넌트

React의 `useMemo`와 함수형 파이프라인을 결합하면 재렌더링 최적화와 가독성을 동시에 얻는다.  
명령형으로 작성하면 변환 로직이 컴포넌트 안에 흩어지지만, 함수형은 데이터 흐름이 한눈에 들어온다.

```tsx
import { pipe, filter, map, toArray } from '@fxts/core';
import { useMemo } from 'react';

interface Task {
  id: string;
  title: string;
  status: 'todo' | 'done';
  priority: number;
}

const byStatus = (status: string) => filter((t: Task) => t.status === status);
const sortByPriority = (tasks: Task[]) => 
  [...tasks].sort((a, b) => a.priority - b.priority);
const toViewModel = map((t: Task) => ({
  ...t,
  display: `[우선순위 ${t.priority}] ${t.title}`
}));

function TaskList({ tasks }: { tasks: Task[] }) {
  const todoTasks = useMemo(() => 
    pipe(
      tasks,
      byStatus('todo'),
      toArray,
      sortByPriority,
      toViewModel,
      toArray
    ),
    [tasks]
  );

  return <div>{todoTasks.map(t => <div key={t.id}>{t.display}</div>)}</div>;
}
```

필터링, 정렬, 변환 로직이 파이프라인으로 명확히 분리되어서 각 단계를 독립적으로 수정한다.  
`useMemo`가 tasks가 바뀔 때만 재계산해서 불필요한 렌더링을 막는다.

### 예제 5: 에러 처리

try-catch는 에러가 발생하면 전체 흐름을 중단시킨다.  
에러를 값으로 다루면 일부 요청이 실패해도 성공한 것들은 정상 처리되는 안정적인 시스템을 만든다.

```typescript
import { pipe, toAsync, map, filter, toArray } from '@fxts/core';

type Result<T> = { ok: true; data: T } | { ok: false; error: Error };

const tryCatch = <T,>(fn: () => Promise<T>) => async (): Promise<Result<T>> => {
  try {
    return { ok: true, data: await fn() };
  } catch (error) {
    return { ok: false, error: error as Error };
  }
};

const isOk = <T,>(r: Result<T>): r is { ok: true; data: T } => r.ok;
const getData = <T,>(r: Result<T>) => (r.ok ? r.data : null);

const fetchUsers = async (ids: number[]) => {
  const results = await pipe(
    ids,
    toAsync,
    map(id => tryCatch(() => fetchUser(id))()),
    toArray
  );

  return {
    users: pipe(results, filter(isOk), map(getData), toArray),
    errors: pipe(results, filter(r => !r.ok), toArray)
  };
};
```

모든 요청을 실행한 후 성공과 실패를 분류해서 부분 실패를 허용한다.  
사용자는 일부 데이터라도 볼 수 있고, 개발자는 어떤 요청이 실패했는지 추적한다.

<br/>

## 추상화 수준 맞추기

함수형 코드의 가독성은 모든 단계가 동일한 추상화 수준을 유지할 때 극대화된다.  
구체적인 구현과 추상적인 의도가 섞이면 코드를 읽는 사람이 혼란스럽다.

### Tip 1: 함수 이름으로 의도 표현

```typescript
// ❌ 추상화 수준이 섞임
pipe(
  data,
  x => x.filter(item => item.age > 20),  // 구체적
  mapToViewModel,  // 추상적
  x => x.slice(0, 10)  // 구체적
);

// ✅ 모든 단계가 동일한 추상화 수준
pipe(
  data,
  filterAdults,
  mapToViewModel,
  takeFirst10
);
```

익명 함수 대신 이름 있는 함수를 사용해서 각 단계의 의도를 명확히 드러낸다.  
파이프라인을 읽을 때 구현 세부사항 없이 "무엇을 하는지"만 이해하게 된다.

### Tip 2: 하나의 함수는 한 가지 일만 수행

```typescript
// ❌ 여러 책임이 섞임
const process = map((user: User) => ({
  name: user.name.toUpperCase(),
  isAdult: user.age >= 20,
  display: `${user.name} (${user.age}세)`
}));

// ✅ 책임 분리
const normalize = map((u: User) => ({ ...u, name: u.name.toUpperCase() }));
const addAdultFlag = map((u: User) => ({ ...u, isAdult: u.age >= 20 }));
const addDisplay = map((u: User) => ({ ...u, display: `${u.name} (${u.age}세)` }));

const process = pipe(normalize, addAdultFlag, addDisplay);
```

하나의 함수가 여러 변환을 동시에 하면 재사용하기 어렵고 테스트하기 힘들다.  
단일 책임으로 쪼개면 각 함수를 독립적으로 테스트하고 다른 조합에서도 쓴다.

### Tip 3: 재사용 vs 일회성

```typescript
// ✅ 재사용 로직은 pipe로
const getActiveUsers = pipe(
  filter((u: User) => u.isActive),
  toArray
);
const active1 = getActiveUsers(users);
const active2 = getActiveUsers(admins);

// ✅ 일회성은 go로
go(
  users,
  filter(u => u.age > 20),
  map(u => u.name),
  toArray
);
```

동일한 로직이 2번 이상 쓰이면 `pipe`로 추출해서 중복을 제거하고,  
한 곳에서만 쓰인다면 `go`로 즉시 실행해서 불필요한 함수 선언을 피한다.

<br/>

## 성능 최적화

### 지연 평가 활용

fxts의 지연 평가는 필요한 만큼만 계산해서 불필요한 연산을 제거한다.

```typescript
// ✅ take(5) 덕분에 5개만 처리됨
pipe(
  hugeArray,
  map(expensiveOperation),
  filter(condition),
  take(5),
  toArray
);

// ❌ 전체를 먼저 처리
pipe(
  hugeArray,
  map(expensiveOperation),
  filter(condition),
  toArray,
  arr => arr.slice(0, 5)
);
```

`take(5)`가 5개를 얻는 순간 평가를 중단해서 거대한 배열도 빠르게 처리한다.  
즉시 평가는 100만 개 전체를 먼저 처리하지만, 지연 평가는 필요한 5개만 계산한다.

### 동시성 제어

비동기 작업의 동시 실행 수를 제어해서 서버 부하와 응답 속도의 균형을 맞춘다.

```typescript
// API 호출을 3개씩만 동시 실행
await pipe(
  userIds,
  toAsync,
  map(fetchUser),
  concurrent(3),
  toArray
);
```

`Promise.all`처럼 모든 요청을 동시에 보내지 않고, `concurrent(3)`으로 3개씩만 실행해서 과부하를 방지한다.  
순차 실행보다 훨씬 빠르면서도 서버에 부담을 주지 않는다.

<br/>

## 마치며

함수형 프로그래밍은 "무엇을 할 것인가"에 집중하게 한다.

- 순수 함수로 예측 가능한 코드
- 작은 함수 조합으로 복잡한 로직 표현
- pipe/go로 추상화 수준 일치

map, filter를 pipe로 연결하는 것부터 시작한다.  
복잡한 비즈니스 로직도 읽기 쉬운 함수 조합으로 표현할 수 있다.
