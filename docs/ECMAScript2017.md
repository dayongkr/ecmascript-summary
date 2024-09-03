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

`Object.values` 메서드는 객체의 열거 가능한 속성 값들로 이루어진 배열을 반환하고, `Object.entries` 메서드는 객체의 열거 가능한 속성 키-값 쌍들로 이루어진 배열을 반환합니다.

해당 메소드들은 키가 문자열인 속성만 반환하며, `Symbol` 키인 속성은 반환하지 않습니다. 이전부터 존재하던 `Object.keys` 메서드 또한 마찬가지입니다.

> 참고: [MDN - Object.values()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values), [MDN - Object.entries()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries), [Github Repo - tc39/proposal-object-values-entries](https://github.com/tc39/proposal-object-values-entries?tab=readme-ov-file)

## String padding

```javascript
"Dayong".padStart(10); // "    Dayong"
"Dayong".padEnd(10); // "Dayong    "
"Dayong".padStart(10, "0"); // "0000Dayong"
"Dayong".padStart(10, "😄"); // "😄😄Dayong"
```

`padStart` 메서드는 문자열의 길이를 지정한 길이로 맞추고, 부족한 부분을 지정한 문자로 채웁니다. `padEnd` 메서드는 문자열의 끝에 채워넣습니다. 이때 두 번째 인수를 생략하면 공백으로 채웁니다.

문자열의 길이를 기준으로 채우기 때문에, 이모지와 같은 `surrogate pair` 문자는 2글자로 취급되어 채워집니다.

> `surrogate pair`: 자바스크립트는 `UTF-16`을 사용하기 때문에, 좀 더 큰 문자를 표현하기 위해 `surrogate pair`라는 방법을 사용합니다. 이 방법은 2개의 16비트 문자로 큰 문자를 표현하는 방법입니다. 따라서 이모지와 같은 문자는 2개의 문자로 취급됩니다.

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

`Object.getOwnPropertyDescriptors` 메서드는 객체의 모든 속성의 `descriptor`를 반환합니다.

> `descriptor`는 속성의 속성을 의미합니다. 객체의 속성은 `value`, `writable`, `enumerable`, `configurable`, `get`, `set` 등의 속성을 가질 수 있습니다. 자세한 내용은 [MDN - Property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)를 참고하세요.

### 얕은 복사

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

`Object.assign` 메서드는 `getter`와 `setter`를 호출하기 때문에 `getter`와 `setter`를 복사할 수 없습니다. 반면 `Object.create` 메서드와 `Object.getOwnPropertyDescriptors` 메서드를 사용하면 `getter`와 `setter`를 복사할 수 있습니다. 다만 후자는 비교적 성능이 떨어지기 때문에 상황에 따라 적절한 방법을 선택하는 것이 좋습니다.

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

함수의 파라미터 리스트와 호출 시 마지막에 쉼표를 사용할 수 있습니다.

이전에는 함수 파리미터를 추가하거나 삭제할 때, 마지막 쉼표를 추가하거나 삭제하는 작업이 필요하여 불필요한 부분도 `code history`에 남았습니다. 이를 방지하기 위해 파이썬과 같은 언어들은 이미 해당 기능을 지원하고 있었습니다.

다만 `prettier`와 같은 코드 포맷터에서 `trailingComma` 옵션을 `all`로 설정하지 않으면 `prettier`가 자동으로 제거할 수 있습니다.

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

기존 `Promise`를 사용한 비동기 처리 방식은 `callback hell`을 유발하고 가독성이 떨어지는 등의 문제가 있었습니다. `Async Functions`는 `Promise`를 사용한 비동기 처리를 보다 간결하고 가독성이 좋게 작성할 수 있도록 도와줍니다.

> 참고: [MDN - async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

## Shared Memory and Atomics

자바스크립트 엔진은 기본적으로 단일 스레드로 동작하지만, 웹 워커를 사용하면 멀티 스레드로 동작할 수 있습니다. 각 워커는 독립적인 공간을 가지고 있어, 데이터를 공유하기 위해서는 `postMessage`를 사용해야 합니다. 하지만 `postMessage`는 일반적으로 데이터를 복사해서 직렬화(serialize)하고 역직렬화(deserialize)하기 때문에 성능이 떨어지고 메모리를 많이 사용합니다.

이러한 문제를 해결하기 위해 `SharedArrayBuffer`가 등장했습니다. `SharedArrayBuffer`는 serialize 과정에서 복사를 하지 않기 때문에, 성능이 향상되고 메모리를 공유할 수 있습니다.

> 종종 `SharedArrayBuffer`를 사용하면 직렬화를 하지 않아 성능이 향상된다고 알려져 있지만, `postMessage`는 항상 직렬화를 수행하기 때문에 이는 사실이 아닙니다. 복사가 핵심입니다.

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

> 웹 워커와 `ArrayBuffer`에 대한 내용은 주제를 벗어나므로 생략하겠습니다.

위 코드는 `ArrayBuffer`와 비교했을 때 어떠한 차이가 있는지 보여주는 예제입니다. `ArrayBuffer`는 복사를 하기 때문에, 워커 스레드에서 값을 변경해도 메인 스레드에서는 변경되지 않습니다. 반면 `SharedArrayBuffer`는 복사를 하지 않고 공유하기 때문에, 워커 스레드에서 값을 변경하면 메인 스레드에서도 변경됩니다.

이처럼 서로 메모리를 공유하면 신경 써야 할 부분이 생기는데, 그 중 하나는 경쟁상태(race condition)입니다.

> 경쟁상태: 두 개 이상의 스레드가 동시에 같은 자원에 접근하려고 할 때, 그 결과가 접근 순서에 따라 달라지는 상태를 말합니다.

이러한 문제에서 보호하기 위해 `Atomics` 객체도 같이 등장했습니다. `Atomics` 객체는 다양한 메소드를 제공하는데, 이런 경쟁상태를 해결하는 가장 보편적인 예제를 보여줄 수 있는 `Atomics.wait` 메서드와 `Atomics.notify` 메서드 위주로 설명하겠습니다.

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

`Atomics.wait` 메서드는 첫 번째 인수로 `Int32Array`, 두 번째 인수로 인덱스, 세 번째 인수로 기대하는 값이 전달됩니다. 즉 `Atomics.wait(sharedView, 0, previous)`는 `sharedView[0]`가 `previous`와 같을 때까지 기다리는 것을 의미합니다.

`Atomics.notify` 메서드는 첫 번째 인수로 `Int32Array`, 두 번째 인수로 인덱스, 세 번째 인수로 몇 개의 스레드를 깨울지 전달됩니다. 즉 `Atomics.notify(sharedView, 0, 1)`는 `sharedView[0]`가 변경되길 기다리고 있는 스레드 중 하나를 깨우는 것을 의미합니다.

> 참고: [HTML spec - 2.7 Safe passing of structured data](https://html.spec.whatwg.org/multipage/structured-data.html), [MDN - Atomics](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics), [MDN - SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)
