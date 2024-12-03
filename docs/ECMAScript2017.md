# ECMAScript 8 (ES8 or ECMAScript 2017)

## 목차

- [`Object.values`/`Object.entries`](#objectvaluesobjectentries)
- [String padding](#string-padding)
- [`Object.getOwnPropertyDescriptors`](#objectgetownpropertydescriptors)
- [Trailing commas in function parameter lists and calls](#trailing-commas-in-function-parameter-lists-and-calls)
- [Async Functions](#async-functions)
- [Shared Memory and Atomics](#shared-memory-and-atomics)

## `Object.values`/`Object.entries`

```javascript
const author = {
  name: "Dayong",
  location: "South Korea",
  age: 0,
  [Symbol("isAuthor")]: true,
};

Object.values(author);
// ["Dayong", "South Korea", 0]
Object.entries(author);
// [["name", "Dayong"], ["location", "South Korea"], ["age", 0]]
```

객체가 가지고 있는 값을 가져와 사용하는 경우가 많고, 실제로 다양한 라이브러리에서 이미 `values` 메서드를 제공하고 있어요.

> lodash와 같은 유틸리티 라이브러리에서 자주 사용되는 메서드를 언어 차원에서 지원하기 위해 제안하는 경우가 많이 보여요.

`Object.values`는 주어진 객체의 열거 가능한 속성 값을 배열로 반환하고, `Object.entries`는 주어진 객체의 열거 가능한 속성을 `[key, value]` 형태의 배열로 반환해요.

기존에 있던 `Object.keys`와 일관된 동작을 위해, 반환되는 속성들은 키가 문자열인 속성만 포함되며, `Symbol` 키로 정의된 속성은 포함되지 않아요.

> 참고: [MDN - Object.values()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values), [MDN - Object.entries()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries), [Github Repo - tc39/proposal-object-values-entries](https://github.com/tc39/proposal-object-values-entries?tab=readme-ov-file)

## String padding

```javascript
"Dayong".padStart(10); // "    Dayong"
"Dayong".padEnd(10); // "Dayong    "
"Dayong".padStart(10, "0"); // "0000Dayong"
"Dayong".padStart(10, "😄"); // "😄😄Dayong"
```

이 또한 자주 사용되는 메서드를 언어 차원에서 지원하기 위해 제안된 메서드에요.

`padStart`와 `padEnd` 메서드는 문자열의 길이를 지정한 길이로 맞추고, 부족한 부분을 지정된 문자로 채워줘요. 두 번째 인수를 생략하면 공백으로 채워지고요. 두 메서드의 차이점은 `padStart`는 문자열의 앞쪽을 채우는 반면, `padEnd`는 문자열의 뒷쪽을 채운다는 점이에요.

이 메서드들은 문자열의 길이를 기준으로 채우기 때문에, 이모지와 같은 surrogate pair 문자는 2글자로 처리되어 채워져요.

참고로 자바스크립트는 UTF-16을 사용하기 때문에, 큰 문자를 표현하기 위해 surrogate pair라는 방식으로 2개의 16비트 문자를 사용해요. 이로 인해 이모지 같은 문자는 2글자로 취급돼요.

> 참고: [MDN - String.prototype.padStart()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/padStart), [MDN - String.prototype.padEnd()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/padEnd), [Github Repo - tc39/proposal-string-pad-start-end](https://github.com/tc39/proposal-string-pad-start-end?tab=readme-ov-file)

## `Object.getOwnPropertyDescriptors`

```javascript
const obj = {
  name: "Dayong",
  get age() {
    return 0;
  },
};

Object.getOwnPropertyDescriptors(obj);
/*
  {
  name: {
    value: 'Dayong',
    writable: true,
    enumerable: true,
    configurable: true
  },
  age: {
    get: [Function: get age],
    set: undefined,
    enumerable: true,
    configurable: true
  }
}
*/
```

복잡한 애플리케이션에서 함수형 프로그래밍과 불변성을 지향하려면, 속성을 적절히 복사하는 것이 중요해요. 하지만 기존의 `Object.assign`은 속성의 descriptor가 아닌 값에 직접적으로 접근하기 때문에 예상치 못한 동작이 발생할 수 있어요. 이러한 문제를 해결하기 위해 `Object.getOwnPropertyDescriptors`가 도입되었어요.

`Object.getOwnPropertyDescriptors`는 객체의 모든 속성에 대한 descriptor를 반환해요. [descriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)는 속성의 `value`, `writable`, `enumerable`, `configurable`뿐만 아니라, `get`과 `set` 메서드와 같은 정보를 포함하고 있어요.

### 활용: 얕은 복사

```javascript
const obj = {
  name: "Dayong",
  age: 0,
  get nextAge() {
    return this.age + 1;
  },
};

// Object.assign
const copy = Object.assign({}, obj);
// { name: 'Dayong', age: 0, nextAge: 1 }
copy.age = 1;
copy.nextAge; // 1

// Object.getOwnPropertyDescriptors
const copy2 = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
// { name: 'Dayong', age: 0, nextAge: [Getter] }

copy2.age = 1;
copy2.nextAge; // 2
```

`Object.assign`은 `getter`와 `setter`를 호출하기만 하고, 이를 복사할 수 없어요. 하지만 `Object.create`와 `Object.getOwnPropertyDescriptors`를 사용하면 `getter`와 `setter`를 복사할 수 있어요. 이를 통해 객체의 동작을 유지하면서 복사를 수행할 수 있어요.

다만, 이 방법은 `Object.assign`보다 다소 성능이 떨어질 수 있어요. 따라서 descriptor를 고려할 필요가 없는 경우에는 여전히 `Object.assign`을 사용하는 것이 더 효율적이에요.

> 참고: [MDN - Object.getOwnPropertyDescriptors()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors), [Github Repo - tc39/proposal-object-getownpropertydescriptors](https://github.com/tc39/proposal-object-getownpropertydescriptors)

## Trailing commas in function parameter lists and calls

```javascript
// prettier-ignore
function foo(
  a,
  b,
  c,
) {
  return a + b + c;
}

// prettier-ignore
foo(
  1,
  2,
  3,
); // 6
```

함수 시그니처 정의 또는 함수 호출 시, 마지막 매개변수 뒤에 쉼표를 허용하는 문법이에요.

이전에는 함수 매개변수를 추가하거나 삭제할 때, 마지막 쉼표를 추가하거나 삭제해야 했어요. 하지만 이 문법으로 마지막 쉼표를 허용하면서 코드 수정이 간편해지고, git 기록에서 불필요한 변경 이력을 줄일 수 있게 되었어요. 참고로, 파이썬과 같은 언어들은 이미 이 기능을 지원하고 있었어요.

다만, Prettier와 같은 코드 포맷터에서 `trailingComma` 옵션을 `all`로 설정하지 않으면, Prettier가 자동으로 마지막 쉼표를 제거할 수 있으니 설정을 확인하는 것이 좋아요.

> 참고: [Github Repo - tc39/proposal-trailing-function-commas](https://github.com/tc39/proposal-trailing-function-commas)

## Async Functions

```javascript
// Promise
function fetchUser() {
  return fetch("https://api.github.com/users/dayong-dev").then((res) =>
    res.json()
  );
}

// Async Functions
async function fetchUser() {
  const res = await fetch("https://api.github.com/users/dayong-dev");
  return res.json();
}
```

기존의 `Promise`를 사용한 비동기 처리 방식은 callback hell을 유발하거나 가독성이 떨어지는 등의 문제가 있었어요. 이를 해결하기 위해 도입된 Async Functions는 비동기 코드를 더욱 간결하고 가독성 있게 작성할 수 있도록 도와줘요.

Async Functions는 `Promise`를 기반으로 작동하지만, `await` 키워드를 사용해 비동기 작업을 순차적으로 실행할 수 있게 해줘요. 이를 통해 동기 코드와 유사하게 작성할 수 있어 일관성 있는 코드를 작성할 수 있어요.

> 참고: [MDN - Async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

## Shared Memory and Atomics

자바스크립트는 기본적으로 단일 스레드로 동작하지만, 웹 워커(Web Workers)를 통해 멀티 스레드 환경으로 확장할 수 있어요. 기존에는 스레드 간 데이터를 공유하기 위해 `postMessage`를 사용했는데, 이 방식은 데이터를 복사하기 때문에 메모리 사용량이 늘어나고 성능이 저하되는 문제가 있었어요.

이 문제를 해결하기 위해 도입된 것이 `SharedArrayBuffer`예요. `SharedArrayBuffer`는 데이터를 복사하지 않고 메모리를 공유할 수 있어 성능이 크게 개선돼요.

> 주의: `SharedArrayBuffer`는 복사 없이 메모리를 공유하여 성능이 향상되는 것이지, `postMessage` 자체의 직렬화 동작을 변경하지는 않아요. 즉, `postMessage`가 복사 대신 참조를 공유할 수 있기 때문에 성능이 좋아지는 거예요.

```javascript
// main.js
const worker = new Worker(import.meta.resolve("./worker.js"), {
  type: "module",
});
const buffer = new ArrayBuffer(1024);
const sharedBuffer = new SharedArrayBuffer(1024);

const view = new Int32Array(buffer);
const sharedView = new Int32Array(sharedBuffer);

worker.postMessage({ buffer, sharedBuffer });
worker.onmessage = (e) => {
  console.log(e.data);
  console.log(view[0], sharedView[0]);
};
```

```javascript
// worker.js
onmessage = ({ data }) => {
  const { buffer, sharedBuffer } = data;
  const view = new Int32Array(buffer);
  const sharedView = new Int32Array(sharedBuffer);

  view[0] = 1;
  sharedView[0] = 1;

  postMessage("done");
};
```

```bash
# 출력 결과
done
0 1
```

위 코드는 `ArrayBuffer`와 `SharedArrayBuffer`의 차이를 보여줘요.

`ArrayBuffer`는 복사된 데이터를 전달하므로 워커 스레드에서 변경된 값이 메인 스레드에 반영되지 않지만, `SharedArrayBuffer`는 복사 없이 메모리를 공유하므로 워커에서 변경된 값이 메인 스레드에도 반영돼요.

따라서 `SharedArrayBuffer`를 사용할 때는 경쟁 상태(race condition)에 주의해야 해요.

> 경쟁 상태는 두 개 이상의 스레드가 동시에 동일한 자원에 접근할 때, 그 결과가 접근 순서에 따라 달라질 수 있는 상태를 말해요.

이러한 문제는 `Atomics` 객체를 사용해서 해결할 수 있어요. `Atomics` 객체는 여러 메서드를 제공해 경쟁 상태를 방지할 수 있는데, 그중 가장 보편적인 `Atomics.wait`와 `Atomics.notify` 메서드를 소개할게요.

```javascript
// main.js
const worker = new Worker(import.meta.resolve("./worker.js"), {
  type: "module",
});

const sharedBuffer = new SharedArrayBuffer(1024);
const sharedView = new Int32Array(sharedBuffer);

const previous = sharedView[0];

worker.postMessage({ sharedBuffer });
console.log(sharedView[0]);
Atomics.wait(sharedView, 0, previous);
console.log("Done", sharedView[0]);
```

```javascript
// worker.js
onmessage = ({ data }) => {
  const { sharedBuffer } = data;
  const sharedView = new Int32Array(sharedBuffer);

  sharedView[0] = 1;
  console.log("Worker thread", sharedView[0]);
  Atomics.notify(sharedView, 0, 1);
};
```

```bash
# 출력 결과
0
Worker thread 1
Done 1
```

`Atomics.wait`는 특정 값이 될 때까지 기다리는 기능을 제공해요. 예를 들어, `Atomics.wait(sharedView, 0, previous)`는 `sharedView[0]`의 값이 `previous`와 같을 때까지 대기해요.

반대로, `Atomics.notify`는 대기 중인 스레드를 깨우는 역할을 해요. 예를 들어, `Atomics.notify(sharedView, 0, 1)`은 `sharedView[0]` 값을 기다리고 있는 하나의 스레드를 깨워요.

> 참고: [HTML spec - 2.7 Safe passing of structured data](https://html.spec.whatwg.org/multipage/structured-data.html), [MDN - Atomics](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics), [MDN - SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)
