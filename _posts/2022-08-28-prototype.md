---
title: "Javascript의 prototype"
date: 2022-08-28 00:00:00 +0900
categories: [knowledge, frontend]
tags: [javascript, prototype]
---

> [Archive] 이 글은 2022년에 다른 플랫폼에서 작성한 글을 이전하며 재구성한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 프로토타입(prototype)

이 글은 JavaScript의 객체 모델 관점에서의 프로토타입에 집중한다.

### 1) prototype 이란?

JavaScript는 전통적인 class 기반 객체지향 언어와는 다른 방식의 객체 모델을 채택하고 있다.  
ES6 이전 class 개념이 존재하지 않았고, 객체와 객체 사이의 연결 관계를 통해 동작을 공유하는 방식을 사용한다.

JavaScript에서 원시 타입을 제외한 대부분의 값은 객체이며, 
각 객체는 내부적으로 자신을 생성한 객체를 참조하는 [[Prototype]]이라는 내부 슬롯을 가진다.

![](https://poiemaweb.com/img/printout_student_obj_from_chrome.png){: width="75%"}

일반적으로 우리가 접근하는 __proto__는 [[Prototype]]에 접근하기 위한 접근자(accessor)이며,  
이 연결을 통해 프로퍼티 탐색이 위임(delegate) 된다.  
이때 이 연결의 대상이 되는 객체를 흔히 프로토타입 객체라고 부른다.

[[Prototype]]의 값은 null 또는 객체이며 상속을 구현하는데 사용된다.  
[[Prototype]] 객체의 데이터 프로퍼티는 get 액세스를 위해 상속되어 자식 객체의 프로퍼티처럼 사용할 수 있다.

![](https://poiemaweb.com/img/object_literal_prototype_chaining.png){: width="75%"}

```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

Person.prototype.sayHello = function () {
  return `Hello, my name is ${this.name}`;
};

const person = new Person("Lee", "male");

// 핵심 관계들
Object.getPrototypeOf(person) === Person.prototype;        // ①
Object.getPrototypeOf(Person.prototype) === Object.prototype; // ④
Object.getPrototypeOf(Person) === Function.prototype;      // ③
person.constructor === Person;                             // ②
```

프로토타입 기반 객체 모델은 객체를 “복사”하지 않는다.  
객체는 다른 객체에 대한 참조를 통해 동작을 위임받는다.

<br/>

### 2) Javascript에서 prototype이 채택된 배경

class 기반 객체지향 프로그래밍은 대상을 추상화하고,  
이를 분류(classification)한 뒤 인스턴스를 생성하는 방식을 기본으로 한다.

반면 JavaScript가 영향을 받은 Self, Kevo와 같은 언어들은  
대상을 고정된 속성의 집합으로 보기보다는 유사성에 기반해 객체를 확장해 나가는 방식을 택했다.

이러한 배경에서 JavaScript는 “클래스 → 인스턴스” 구조보다는 객체 → 객체 간의 연결을 통한 모델을 채택하게 되었다.

이 방식은 전통적인 class 기반 OOP와 철학적으로는 차이가 있지만, 실제 사용 관점에서는 유사한 형태로 수렴하기도 한다.

<br/>

### 3) 위임(Delegation)이라는 핵심 차이

프로토타입 기반 OOP는 class 기반 OOP와 비슷해 보이지만
가장 큰 차이는 복사가 아닌 위임이라는 점이다.

객체에서 프로퍼티를 찾지 못했을 경우, JavaScript 엔진은 해당 객체의 [[Prototype]]이 가리키는 객체로 탐색을 위임한다.  
이 과정이 반복되며, 이를 프로토타입 체인이라고 부른다.

즉, 객체 간에는 상속이라기보다는 연결된 체인을 통한 동작 공유가 이루어진다.

<br/>

### 4) ES5 시절의 상속 패턴과 문제점

ES6 이전에는 class 문법이 존재하지 않았기 때문에 생성자 함수와 prototype을 직접 다뤄야 했다.

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.hello = function () {
  return `hello, my name is ${this.name}`;
};

function WeirdPerson(name, label) {
  Person.call(this, name);
  this.label = label;
}

WeirdPerson.prototype = Object.create(Person.prototype);
```

위 코드에서 Object.create(Person.prototype)를 사용한 이유는  
상위 생성자를 호출하지 않고 프로토타입 체인만 연결하기 위함이다.

하지만 이 방식에는 한 가지 문제가 있다.

이는 기존의 WeirdPerson.prototype 객체를 수정한 것이 아니라, 완전히 새로운 객체로 교체했기 때문이다.  
그 결과 새 객체에는 constructor 프로퍼티가 존재하지 않는다.

이로 인해 constructor에 의존하는 코드가 있다면 해당 프로퍼티를 직접 다시 정의해야 했다.

```javascript
Object.defineProperty(WeirdPerson.prototype, "constructor", {
  enumerable: false,
  writable: true,
  configurable: true,
  value: WeirdPerson,
});
```

#### 다른 방법들과 그 한계

```javascript
WeirdPerson.prototype = new Person();
```

이 방식은 프로토타입 체인을 연결할 수는 있지만,  
Person 생성자를 호출하게 되어 불필요한 초기화나 side effect가 발생할 수 있다.

결국 당시의 최선의 선택지는 Object.create를 사용하고 필요한 프로퍼티를 수동으로 보정하는 방식이었지만  
이는 개발자에게 상당한 부담을 주었다.

<br/>

### 5) ES6 이후의 변화

ES6에서는 class 문법이 도입되며 이러한 패턴들이 언어 차원에서 정리되었다.

또한 Object.setPrototypeOf가 추가되어 기존 객체의 프로토타입을 변경할 수 있게 되었다.

다만 이 메서드는 객체 생성 이후에 프로토타입을 변경하기 때문에 실무에서는 권장되지 않는 경우가 많다.  
결과적으로 ES6의 변화는 프로토타입 기반 객체 모델을 유지한 채 개발자 경험을 개선한 것에 가깝다고 볼 수 있다.

<br/>