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

`Object.values`는 주어진 객체의 열거 가능한 속성을 배열로 반환하고, `Object.entries`는 주어진 객체의 열거 가능한 속성을 `[key, value]` 형태의 배열로 반환합니다.

이때 키가 문자열인 속성만 반환하며, `Symbol` 키인 속성은 반환하지 않습니다. 기존에 존재하던 `Object.key`도 마찬가지입니다.

> 참고: [MDN - Object.values()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values), [MDN - Object.entries()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries), [Github Repo - tc39/proposal-object-values-entries](https://github.com/tc39/proposal-object-values-entries?tab=readme-ov-file)

## String padding

```javascript
"Dayong".padStart(10); // "    Dayong"
"Dayong".padEnd(10); // "Dayong    "
"Dayong".padStart(10, "0"); // "0000Dayong"
"Dayong".padStart(10, "😄"); // "😄😄Dayong"
```

`padStart와` `padEnd` 메서드는 문자열의 길이를 지정한 길이로 맞추고, 부족한 부분을 지정된 문자로 채워줍니다. 두 번째 인수를 생략하면 공백으로 채웁니다. 두 메서드의 차이점은 `padStart`는 문자열의 앞쪽을 채우고, `padEnd`는 문자열의 뒷쪽을 채운다는 점입니다.

문자열의 길이를 기준으로 채우기 때문에, 이모지와 같은 surrogate pair 문자는 2글자로 처리되어 채워집니다.

> 자바스크립트는 UTF-16을 사용하기 때문에, 큰 문자를 표현하기 위해 surrogate pair라는 방식으로 2개의 16비트 문자를 사용합니다. 이로 인해 이모지 같은 문자는 2글자로 취급됩니다.
>
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

`Object.getOwnPropertyDescriptors`는 객체의 모든 속성의 descriptor를 반환합니다. descriptor는 속성의 value, writable, enumerable, configurable, get, set 등의 정보를 포함합니다.

> 자세한 내용은 [MDN - Property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)를 참고해주세요.

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

`Object.assign`은 `getter와` `setter를` 호출하기 때문에 이를 복사할 수 없습니다. 반면, `Object.create`와 `Object.getOwnPropertyDescriptors`를 사용하면 `getter`와 `setter`를 복사할 수 있습니다.

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

함수의 시그니처와 호출 시, 마지막 매개변수 뒤에 쉼표를 허용하는 문법입니다.

이전에는 함수 매개변수를 추가하거나 삭제할 때, 마지막 쉼표를 추가 또는 삭제하는 작업이 필요했으나, 쉼표가 허용됨으로써 코드 수정이 간편해지고 git 기록에서 불필요한 변경이 줄어듭니다. 이를 방지하기 위해 파이썬과 같은 언어들은 이미 해당 기능을 지원하고 있었습니다.

다만, prettier와 같은 코드 포맷터에서 `trailingComma` 옵션을 `all`로 설정하지 않으면 prettier가 자동으로 쉼표를 제거할 수 있습니다.

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

Async Functions는 기존의 callback으로 처리하던 비동기 처리보다 더 간결하고 가독성 있는 코드를 작성할 수 있게 해줍니다. await 키워드를 사용하면 비동기 처리를 순차적으로 실행할 수 있어 코드 구조가 더욱 직관적입니다.

> 참고: [MDN - async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

## Shared Memory and Atomics

자바스크립트는 기본적으로 단일 스레드로 동작하지만, 웹 워커를 통해 멀티 스레드로 확장할 수 있습니다. 기존에는 스레드간 데이터를 공유하려면 `postMessage`를 사용하여 직렬화 및 역직렬화를 해야 했으나, 이는 성능이 떨어지고 메모리 사용량이 증가하는 문제를 야기합니다.

이를 해결하기 위해 `SharedArrayBuffer`가 도입되었습니다. `SharedArrayBuffer`는 데이터를 복사하지 않고 메모리를 공유할 수 있어 성능이 개선됩니다.

> 종종 `SharedArrayBuffer`를 사용하면 직렬화 과정을 건너뛰어 성능이 향상된다고 설명하지만, `postMessage`는 항상 직렬화를 수행하므로 실제로는 복사를 하지 않아 성능이 개선됩니다.

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

위 코드는 `ArrayBuffer`와 `SharedArrayBuffer`의 차이를 보여줍니다. `ArrayBuffer`는 복사된 데이터를 전달하므로 워커 스레드에서 변경된 값이 메인 스레드에 반영되지 않지만, `SharedArrayBuffer`는 복사 없이 메모리를 공유하므로 워커에서 변경된 값이 메인 스레드에도 반영됩니다.

이처럼 메모리를 공유할 때는 경쟁 상태(race condition)에 주의해야 합니다.

> 경쟁상태는 두 개 이상의 스레드가 동시에 동일한 자원에 접근할 때, 그 결과가 접근 순서에 따라 달라질 수 있는 상태를 말합니다.

이 문제를 해결하기 위해 `Atomics` 객체가 도입되었습니다. `Atomics`는 여러 메서드를 제공하여 경쟁 상태를 방지하는데, 이중 가장 보편적인 `Atomics.wait`와 `Atomics.notify` 메서드를 소개하겠습니다.

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

`Atomics.wait`는 특정 값이 될 때까지 기다리는 기능을 제공합니다. `Atomics.wait(sharedView, 0, previous)`는 `sharedView[0]`이 `previous`와 같을 때까지 기다립니다.

반대로, `Atomics.notify`는 대기 중인 스레드를 깨우는 역할을 합니다. `Atomics.notify(sharedView, 0, 1)`은 `sharedView[0]`를 기다리고 있는 하나의 스레드를 깨웁니다.

> 참고: [HTML spec - 2.7 Safe passing of structured data](https://html.spec.whatwg.org/multipage/structured-data.html), [MDN - Atomics](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics), [MDN - SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)
