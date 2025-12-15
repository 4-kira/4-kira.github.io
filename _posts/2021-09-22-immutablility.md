---
title: "불변성"
date: 2021-09-22 00:00:00 +0900
categories: [knowledge, frontend]
tags: [react, immutability]
---

> [Archive] 이 글은 2021년에 다른 플랫폼에서 작성한 글을 이전하며 재구성한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 리액트에서 불변성이 중요한 이유

React는 상태 변경 자체를 감지하는 것이 아니라,  
이전 렌더 트리와 다음 렌더 트리를 비교(reconciliation) 하여 참조 동일성이 깨진 지점부터 다시 렌더링을 수행한다.

이 과정에서 불변성은 최소 변경 단위를 식별하기 위한 전제 조건으로 작동한다.

### 불변성에 관하여

JavaScript에서 원시 타입은 값 자체가 불변이지만,  
객체 타입은 참조를 통해 공유되기 때문에 불변성을 의식적으로 다뤄야 한다.

자바스크립트의 원시 타입(primitive type)

- Boolean
- String
- Number
- Null
- undefined
- Symbol

값은 메모리영역 안에서 변경이 불가능하며 변수에 할당할 때 완전히 새로운 값이 만들어져 재할당된다.

```javascript
let name = "foo";
let newName = name;
name = "bar";

console.log(newName); // foo
console.log(name); // bar
```

```javascript
let x = {
  name: "sam",
};

let y = x;
x.name = "kim";

console.log(y.name); // kim
console.log(x === y); // true
```

가변 객체를 여러 곳에서 공유한 상태에서 mutation이 발생했다.  
객체를 불변으로 다룰 것이라는 암묵적 가정이 코드 레벨에서 보장되지 않는 상황.

#### - const

const는 재선언 및 재할당이 불가능하다.

이 때 const로 선언한 변수는 값이 불변한다고 생각할 수 있는데,  
**const는 값에 대한 참조(가리키는 것)가 한번 변수에 할당되면 변할 수 없음을 의미하는 것**일 뿐  
**const 변수가 참조하고 있는 '값'이 불변한다는 것을 의미하지 않는다**.

이 단계에서 리액트에서는 더이상 변화를 적절히 감지할 수 없기 때문에 적절하게 불변성을 지켜줘야 한다.  
그 방법 중 하나는 spread operator(...)를 사용하는 것이다.

#### - spread operator

```javascript
const y = { ...x };
```

얕은 복사인 spread operator는 모든 객체에 대해 불변성을 보장하지 않는다.  
단지 새로운 최상위 객체를 생성할 뿐이며, 중첩 구조에 대해서는 여전히 참조를 공유한다.

얕은 복사는 최상위 객체의 참조만 분리하고, 중첩된 객체는 기존 참조를 그대로 공유하는 복사 방식이다.

하지만 객체의 계층이 깊어지면 여전히 문제가 발생한다.

```javascript
const person = {
  name: "sangmin",
  age: 25,
  friends: ["junzzi", "junseo", "sungho"],
};
const expert = { ...person };
/* {...person, friends: [...person.friends]}로 불변성 유지 가능 */

person.name = "lee";
person.friends.push("kang");
/* user = { name: 'lee', age: 25
, friends: ['junzzi', 'junseo', 'sungho', 'kang'] } 
otherUser = { name: 'sangmin', age: 25
, friends: ['junzzi', 'junseo', 'sungho'] } */

person === expert; // false
person.friends === expert.friends; // true
```

완전히 다른 객체를 만들고 싶다면 '깊은 복사'를 해야한다.  
깊은 복사는 데이터 자체를 통째로 복사하며, 복사된 두 객체는 완전히 독립적인 메모리를 차지한다.

<br/>

### React에서의 불변성

React에서는 값을 비교할 때, '얕은 비교'를 실행하여 성능 최적화를 이룬다.  
React에서 중요한 것은 깊은 복사가 아니라 변경이 발생한 경로의 참조가 새로워졌는지 여부이다.

React는 상태 업데이트 시 이전 상태와 다음 상태를 Object.is를 통해 참조 동일성으로 비교한다.  
**이 비교는 값의 내용이 아니라 참조가 동일한지 여부를 기준으로 이루어진다**.

#### - 의미

따라서 React에서 불변성을 지킨다는 것은 다음을 의미한다.

- 변경이 발생한 객체는 반드시 새로운 참조를 가져야 한다
- 변경되지 않은 객체는 기존 참조를 유지해야 한다

이러한 패턴을 구조적 공유(structural sharing) 라고 부르며, React의 렌더링 모델은 이 전제를 기반으로 설계되어 있다.

#### - 목표

모든 상태를 깊은 복사하는 것은 성능적으로도, 개념적으로도 적절하지 않다.

React 애플리케이션의 목표는  
완전히 독립적인 상태 트리를 만드는 것이 아니라, **변경된 부분만 새로운 참조를 갖도록 명확히 표현하는 것**이다.

깊은 복사는 불변성을 달성하는 하나의 방법일 수는 있지만, React가 기대하는 불변성의 형태는 아니다.

```javascript
import { useState } from "react";

export default function Counter() {
  const [state, setState] = useState({
    count: 0,
  });
  console.log("mounted or updated");
  return (
    <div>
      <p> {state.count} </p>
      <button
        onClick={() => {
          state.count += 1;
          setState(state);
        }}
      >
        +1
      </button>
      <button
        onClick={() =>
          setState((prevState) => ({ count: prevState.count - 1 }))
        }
      >
        -1
      </button>
    </div>
  );
}
```

위 코드에서는 상태 값이 변경되었음에도 불구하고 Counter 컴포넌트는 다시 렌더링되지 않는다.  
이는 state 내부의 값을 직접 변경했지만, React에 전달된 상태 객체의 참조는 이전과 동일하기 때문이다.

```javascript
import { useState } from "react";

export default function Counter() {
  const [state, setState] = useState({ count: 0 });
  console.log("mounted or updated");
  return (
    <div>
      <p>{state.count}</p>
      <button
        onClick={() => {
          setState((prevState) => ({ count: prevState.count + 1 }));
        }}
      >
        +1
      </button>
      <button
        onClick={() =>
          setState((prevState) => ({ count: prevState.count - 1 }))
        }
      >
        -1
      </button>
    </div>
  );
}
```

반면 updater 함수를 통해 새로운 객체를 반환하면 이전 상태와는 다른 참조가 생성된다.  
이 경우 React는 상태가 변경되었다고 판단하고 컴포넌트를 다시 렌더링한다.

중요한 점은, React가 상태를 비교할 때 깊은 비교를 수행하지 않는다는 사실이다.  
이 설계는 성능을 위한 선택이며 그 전제로 개발자가 상태를 불변적으로 다룰 것을 기대한다.

즉, 불변성은 React의 렌더링 모델을 성립시키기 위한 계약에 가깝다.
