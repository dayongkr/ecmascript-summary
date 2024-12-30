# ECMAScript 13 (ES13 or ECMAScript 2022)

## 목차

- [Class Fields](#class-fields)
- [Regex Match Indices](#regex-match-indices)
- [Top-level `await`](#top-level-await)
- [Ergonomic brand checks for private fields](#ergonomic-brand-checks-for-private-fields)
- [`at()`](#at)
- [Accessible `Object.prototype.hasOwnProperty`](#accessible-objectprototypehasownproperty)
- [Class Static Block](#class-static-block)
- [Error Cause](#error-cause)

## Class Fields

```javascript
class Person {
  constructor() {
    this.name = "이다용";
  }
}
```

클래스 필드를 선언하기 위해서는 클래스의 생성자 내부에 `this`를 사용하여 필드를 선언해야 했어요. 이제는 클래스 필드를 클래스의 최상위 레벨에서 선언할 수 있어요.

```javascript
class Person {
  name = "이다용";
}
```

이렇게 선언하면, 생성자 내부에 필드를 선언하는 것보다 더 간결하게 클래스 필드를 선언할 수 있어요.

```javascript
class Person {
  name = "이다용";
  #age = 24;
  job; // undefined
}
```

추가로 `#`을 사용하여 private으로 선언할 수도 있어요.

주의할 점은 최상위에서만 선언할 수 있어요. 그리고 초기화를 하지 않으면, `undefined`가 할당돼요.

> 참고: [GitHub - tc39/proposal-class-fields](https://github.com/tc39/proposal-class-fields?tab=readme-ov-file)

## Regex Match Indices

원래의 `RegExp`는 매칭된 문자열의 내용만 반환했고 인덱스는, 반환하지 않았어요. 이제는 `d` 플래그를 사용하여 매칭된 문자열의 인덱스를 반환할 수 있어요.

> 성능을 위해서 `d` 플래그를 , 인덱스를 반환하니 주의해야 해요.

```javascript
const regex = /a+(?<Z>z)?/d;

const match = regex.exec("aaaz");
console.log(match.indices); // [ [ 0, 4 ], [ 3, 4 ], groups: [Object: null prototype] { Z: [ 3, 4 ] } ]
console.log(match.indices.groups.Z); // [ 3, 4 ]
console.log(match.indices[0]); // [ 0, 4 ]
```

인덱스 정보는 배열로 반환되며, `indices` 프로퍼티에 저장돼요. 추가로 `groups` 프로퍼티에는 이름이 있는 캡처 그룹의 인덱스 정보가 저장돼요.

> 참고: [GitHub - tc39/proposal-regexp-match-indices](https://github.com/tc39/proposal-regexp-match-indices)

## Top-level `await`

기존에는 Top-level에서 `await`를 사용할 수 없었어요. 따라서 아래와 같은 두 가지 방법으로 해결해야 했어요.

- 익명 함수 안에 `async` 함수를 선언하고 즉시 호출하기
- `Promise`를 반환하고 사용하는 쪽에서 `await`를 사용하기

첫 번째 방법은 코드를 복잡하게 만들고, 두 번째 방법은 `await`를 사용하는 것을 잊으면 race condition이 발생할 수 있어요.

```javascript
const response = await fetch("https://api.example.com");
const data = await response.json();
console.log(data);
```

이러한 문제를 해결하기 위해, Top-level에서 `await`를 사용할 수 있게 됐어요.

> 참고: [GitHub - tc39/proposal-top-level-await](https://github.com/tc39/proposal-top-level-await)

## Ergonomic brand checks for private fields

```javascript
class Person {
  #name = "이다용";
  static whoIs(person) {
    try {
      console.log(person.#name);
    } catch (error) {
      console.log("이름이 없어요.");
    }
  }
}

const daYong = new Person();

Person.whoIs(daYong); // 이다용
Person.whoIs({}); // 이름이 없어요.
```

private 필드에 접근했는데, 해당 필드가 존재하지 않으면 예외가 발생했어요. 그래서 `try-catch` 문을 사용하여 예외를 처리해야 했어요. 하지만 이러한 방식은 코드를 이해하기 어렵게 만들어요.

```javascript
class Person {
  #name = "이다용";
  static whoIs(person) {
    if (#name in person) {
      console.log(person.#name);
    } else {
      console.log("이름이 없어요.");
    }
  }
}
```

이제는 `in` 연산자를 사용하여 명시적으로 private 필드가 존재하는지 확인할 수 있어요.

> 참고: [GitHub - tc39/proposal-private-fields-in-in](https://github.com/tc39/proposal-private-fields-in-in)

## `at()`

자바스크립트의 `[]` 문법은 배열과 문자열만을 위한 것이 아니라 객체에도 사용할 수 있어요. 그래서 음수 인덱싱(`arr[-1]`)을 쉽사리 지원하기 어려웠어요. 이 때문에 마지막 요소를 가져오기 위해 `arr[arr.length - 1]`과 같은 방법을 사용해야 했어요.

```javascript
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(-1)); // 5
console.log(arr.at(-2)); // 4
console.log(arr.at(-3)); // 3
console.log(arr.at(-4)); // 2
console.log(arr.at(-5)); // 1
console.log(arr.at(-6)); // undefined
console.log(arr.at(5)); // undefined
```

이를 해결하기 위해 배열, 문자열 그리고 `TypedArray`에 `at()` 메소드를 추가했어요. `at()` 메소드는 음수 인덱싱을 지원하며, 인덱스가 범위를 벗어나면 `undefined`를 반환해요.

> 참고: [GitHub - tc39/proposal-array-at](https://github.com/tc39/proposal-relative-indexing-method)

## Accessible `Object.prototype.hasOwnProperty`

```javascript
const obj1 = Object.create(null);
obj1.hasOwnProperty("name"); // Uncaught TypeError: ...

let obj2 = {
  name: "이다용",
  hasOwnProperty() {
    throw new Error("gotcha!");
  },
};

obj2.hasOwnProperty("name"); // Uncaught Error: gotcha!
console.log(Object.prototype.hasOwnProperty.call(obj2, "name")); // true
```

`Object.create(null)`로 생성된 객체는 `Object.prototype`으로부터 상속받지 않아서 `hasOwnProperty` 메소드를 사용할 수 없었어요. 또한 `hasOwnProperty` 메소드는 오버라이딩이 가능하기 때문에 예상치 못한 결과가 발생할 수 있어요. 그래서 `Object.prototype.hasOwnProperty.call`을 사용하여 이러한 문제를 해결해야 했어요.

```javascript
const obj1 = { name: "이다용" };
const obj2 = Object.create(null);

console.log(Object.hasOwn(obj1, "name")); // true
console.log(Object.hasOwn(obj2, "name")); // false
```

하지만 코드가 복잡해지고 가독성이 떨어지는 문제가 있었어요. 이제는 `Object.hasOwn` 메소드를 제공하여 더 간결하게 `hasOwnProperty` 메소드를 사용할 수 있어요.

> 참고: [GitHub - tc39/proposal-accessible-object-hasownproperty](https://github.com/tc39/proposal-accessible-object-hasownproperty)

## Class Static Block

```javascript
class Person {
  static x;
  static y;
  static #z;
}

try {
  Person.x = 1;
  Person.y = 2;
  Person.#z = 3; // 에러 발생
} catch (error) {
  console.log("에러 발생");
}
```

기존에는 정적 필드를 선언하고 이후 초기화하기 위해서는 클래스 밖에서 초기화해야 했어요. 이러한 방식은 정적 private 필드를 초기화할 수 없었어요. 물론 private 필드 선언을 외부로 노출하면 초기화할 수 있지만, 이는 private 필드의 의미를 퇴색시키는 문제가 있어요.

```javascript
class Person {
  static x;
  static y;
  static #z;

  static {
    Person.x = 1;
    Person.y = 2;
    Person.#z = 3;
  }
}
```

정적 필드에 대한 유연성을 높이기 위해 클래스 정적 블록을 도입했어요. 이처럼 `static { ... }` 블록을 사용하여 정적 필드를 초기화할 수 있어요.

> 참고: [GitHub - tc39/proposal-class-static-block](https://github.com/tc39/proposal-class-static-block)

## Error Cause

```javascript
function foo() {
  try {
    throw new Error("초기 에러");
  } catch (error) {
    throw new Error("에러 다시 던지기", { cause: error });
  }
}

try {
  foo();
} catch (error) {
  console.error(error); // Error: 에러 다시 던지기
  console.error(error.cause); // Error: 초기 에러
}
```

기존에는 내부 메서드에서 발생한 오류를 다룰 때, 원인을 설명하기 위한 추가적인 정보를 제공하기 어려웠어요. 이를 해결하기 위해 `cause` 프로퍼티를 추가하여 오류의 원인을 설명할 수 있게 됐어요. 이처럼 추가 정보를 담아서 다시 던지는 것은 에러를 다루는 패턴 중 흔한 접근 방식이에요.

> 참고: [GitHub - tc39/proposal-error-cause](https://github.com/tc39/proposal-error-cause)
