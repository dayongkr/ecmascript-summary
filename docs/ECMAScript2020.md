# ECMAScript 11 (ES11 or ECMAScript 2020)

## 목차

- [`String.prototype.matchAll`](#stringprototypematchall)
- [`import()`](#import)
- [`BigInt`](#bigint)
- [`Promise.allSettled`](#promiseallsettled)
- [`globalThis`](#globalthis)
- [`for-in` mechanics or `for-in` order](#for-in-mechanics-or-for-in-order)
- [Optional Chaining](#optional-chaining)
- [Nullish coalescing Operator](#nullish-coalescing-operator)
- [`import.meta`](#importmeta)

## `String.prototype.matchAll`

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

str.match(regexp); // ["test1", "test2"]

// 캡처 그룹에 대한 정보를 얻기 위해서는 다음과 같이 해야 했어요.
const matches = [];
str.replace(regexp, (match, p1, p2, p3, offset, string) => {
  matches.push({ match, p1, p2, p3, offset, string });
});

matches; // [{ match: "test1", p1: "e", p2: "st1", p3: "1", offset: 0, string: "test1test2" }, { match: "test2", p1: "e", p2: "st2", p3: "2", offset: 5, string: "test1test2" }]
```

기존의 `String.prototype.match()` 메서드는 캡처 그룹에 대한 정보를 제공하지 않아서 위와 같은 불편함이 있었어요.

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

const matches = [...str.matchAll(regexp)];

matches; // [ { 0: "test1", 1: "e", 2: "st1", 3: "1", index: 0, input: "test1test2" }, { 0: "test2", 1: "e", 2: "st2", 3: "2", index: 5, input: "test1test2" } ]
```

이를 해결하기 위해 `String.prototype.matchAll()` 메서드가 추가됐어요. 이 메서드는 `RegExp` 객체의 `g` 플래그와 함께 사용하여 문자열에서 일치하는 모든 캡처 그룹을 반환해요.

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

for (const match of str.matchAll(regexp)) {
  console.log(match);
}
/*
[ "test1", "e", "st1", "1", index: 0, input: "test1test2", groups: undefined ]
[ "test2", "e", "st2", "2", index: 5, input: "test1test2", groups: undefined ]
*/
```

반환된 값은 Iterator 객체이며, `for...of` 구문을 사용하여 순회할 수 있어요.

> 참고: [GitHub: tc39/proposal-string-matchall](https://github.com/tc39/proposal-string-matchall), [MDN: String.prototype.matchAll()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll)

## `import()`

웹 환경에서는 성능상의 이유등으로 동적으로 모듈을 가져와야 할 때가 있어요. 하지만 이전의 구문 형태로는 동적으로 모듈을 가져오는 것이 불가능했어요.

```javascript
import("./module.js")
  .then((module) => {
    // module을 가져오는데 성공했을 때
  })
  .catch((err) => {
    // module을 가져오는데 실패했을 때
  });

// 또는 async/await를 사용할 수 있어요.
try {
  const module = await import("./module.js");
} catch (err) {
  // module을 가져오는데 실패했을 때
}
```

이를 해결하기 위해 `import()` 함수가 추가됐어요. CJS(CommonJS)의 `require()`와 달리 별칭(alias) 생성을 문법적으로 막았기 때문에 보다 안전하게 사용할 수 있다고 생각해요.

### 일반적인 `import`와 다른 점

- `import()`는 동적으로 모듈을 가져올 수 있어요.
- 호이스팅이 되지 않아요.
- 스크립트에도 사용할 수 있어요. (`<script>` 태그에서 사용 가능)
- 템플릿 리터럴 등 인자로 동적인 값을 넣을 수 있어요.

> 참고: [GitHub: tc39/proposal-dynamic-import](https://github.com/tc39/proposal-dynamic-import)

## `BigInt`

자바스크립트에서는 `2^53 - 1`보다 큰 정수를 표현할 수 없어요.

```javascript
const max = Number.MAX_SAFE_INTEGER; // 9007199254740991
max + 1; // 9007199254740991

const bigIntMax = BigInt(Number.MAX_SAFE_INTEGER); // 9007199254740991n
bigIntMax + 1n; // 9007199254740992n
```

그렇다보니 나노초 단위의 타임스탬프를 표현 하거나 64비트의 정수를 다루는데 어려움이 있었고, 이를 해결하기 위해 `BigInt`가 추가됐어요.

```JavaScript
1n / 2n; // 0n
```

`BigInt`는 `Number`와 다르게 `n`을 붙여서 표현해요.다만, `BigNumber`가 아니기 때문에 나눗셈 연산을 할 때 소수점 이하를 버린다는 점을 주의해야 해요.

> 참고: [GitHub: tc39/proposal-bigint](https://github.com/tc39/proposal-bigint?tab=readme-ov-file)

## `Promise.allSettled`

기존의 `Promise.all`, `Promise.race`, `Promise.any`는 단락 평가(short-circuit evaluation)를 사용해요. 즉, 하나라도 실패(rejected)하거나 성공(fulfilled)하면 즉시 반환해요.

이러한 특성 때문에 모든 프로미스가 처리될 때까지 기다리는 방법이 없었어요. 이를 해결하기 위해 `Promise.allSettled`가 추가됐어요.

```javascript
const promises = [fetch("index.html"), fetch("https://does-not-exist/")];
const results = await Promise.allSettled(promises);
const successfulPromises = results.filter((p) => p.status === "fulfilled");
```

`Promise.allSettled`는 모든 프로미스가 처리될 때까지 기다리고, `status`, `value`, `reason` 프로퍼티를 가진 객체를 반환해요.

- `status`: 프로미스의 상태를 나타내는 문자열. `"fulfilled"` 또는 `"rejected"`가 될 수 있어요.
- `value`: 프로미스가 성공했을 때의 결과 값. `status`가 `"fulfilled"`일 때만 존재해요.
- `reason`: 프로미스가 실패했을 때의 이유. `status`가 `"rejected"`일 때만 존재해요.

> 참고: [GitHub: tc39/proposal-promise-allSettled](https://github.com/tc39/proposal-promise-allSettled), [MDN: Promise.allSettled()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

## `globalThis`

브라우저에서는 `window`, `self`, `this`, `frames`으로 전역 객체에 접근할 수 있어요. 하지만 Node.js에서는 `global`, `this`으로 접근이 가능한데, 여러 제한 사항이 있어요. 이외에도 `Function('return this')()` 등으로 전역 객체에 접근할 수 있었지만, 이는 CSP(Content Security Policy)에 위배되는 문제가 있어서 사용을 권장하지 않았어요.

이처럼 환경에 따라 전역 객체에 접근하는 방법이 다르고 그 안에서도 방법이 다양해 혼란이 있었어요.

```javascript
globalThis.setTimeout === window.setTimeout; // true
globalThis.setTimeout === self.setTimeout; // true
globalThis.setTimeout === this.setTimeout; // true
```

이를 해결하기 위해 어떤 환경에서든 전역 객체에 접근할 수 있는 `globalThis`가 추가됐어요. 단, 브라우저에서는 `iframe` 그리고 cross-window 보안 문제로 실제 전역 객체를 Proxy로 감싸서 제공하기 때문에, 이런 경우엔 `globalThis`는 실제 전역 객체가 아닌 `Proxy` 객체를 반환해요.

> 참고: [GitHub: tc39/proposal-global](https://github.com/tc39/proposal-global?tab=readme-ov-file), [MDN: globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)

## `for-in` mechanics or `for-in` order

`for-in`의 순서는 대게 명세화되어 있지 않았어요. 하지만 주요 엔진에서는 특정 조건에서 이미 일관된 동작을 보여주기 때문에, 이러한 동작을 명세화하기로 결정했어요.

해당 제안에서는 아래 3가지의 경우가 발생하기 이전까진 동일한 동작을 보여주도록 명세화됐어요.

1. 프로토타입 체인이 변경된 경우
2. 속성이 삭제된 경우
3. 프로토타입 체인에 속성이 추가되거나 열거 가능 속성이 변경된 경우

해당 부분은 `EnumerableOwnPropertyNames`의 동작과 관련이 있으며, 이는 `Object.keys()`, `Object.values()`, `Object.entries()`, `JSON.stringify()`, `JSON.parse()`에 영향이 있어요.

> 참고: [GitHub: tc39/proposal-for-in-order](https://github.com/tc39/proposal-for-in-order?tab=readme-ov-file)

## Optional Chaining

```javascript
obj && obj.a && obj.a.b && obj.a.b.c;
```

객체의 중첩된 프로퍼티에 접근할 때, 중간에 `null` 또는 `undefined`가 있을 때 에러가 발생해요. 따라서 이를 확인하기 위해 위와 같이 코드를 작성해야 했어요.

```javascript
obj?.a?.b?.c;
```

이러한 경우를 해결하기 위해 `?.` 연산자가 추가됐어요. `?.` 연산자는 좌항이 `null` 또는 `undefined`일 때 에러를 발생시키지 않고 `undefined`를 반환해요.

```javascript
const obj = {
  a: () => "hello",
};

obj.a?.(); // "hello"
obj.b?.(); // undefined

const arr = [1, 2, 3];

arr?.[0]; // 1
arr?.[3]; // undefined
```

`?.` 연산자는 함수 호출 그리고 배열 접근에도 사용할 수 있어요.

> 참고: [GitHub: tc39/proposal-optional-chaining](https://github.com/tc39/proposal-optional-chaining), [MDN: Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)

## Nullish coalescing Operator

프로퍼티에 접근할 때, 프로퍼티가 `null` 또는 `undefined`일 때 대체값을 지정하기 위해 기존에는 `||` 연산자를 사용했어요.

```javascript
const obj = {
  a: 0,
  b: "",
  c: null,
  d: undefined,
  e: "hello",
};

obj.a || "default"; // "default"
obj.b || "default"; // "default"
obj.c || "default"; // "default"
obj.d || "default"; // "default"
obj.e || "default"; // "hello"
```

하지만 이는 falsy한 값에 대해서도 대체값을 지정해버리는 문제가 있었어요.

```javascript
obj.a ?? "default"; // 0
obj.b ?? "default"; // ""
obj.c ?? "default"; // "default"
obj.d ?? "default"; // "default"
obj.e ?? "default"; // "hello"
```

이를 해결하기 위해 `??` 연산자가 추가됐어요. `??` 연산자는 `null` 또는 `undefined`일 때만 대체값을 지정해요.

> 참고: [GitHub: tc39/proposal-nullish-coalescing](https://github.com/tc39/proposal-nullish-coalescing)

## `import.meta`

라이브러리를 만들 때 사용자가 전달한 값에 따라 동작을 다르게 할 수 있으면 유연성이 높아져요.

CJS(CommonJS)에서는 현재 파일의 경로 또는 모듈의 경로를 알 수 있는 `__dirname`과 `__filename`이 있어서 사용자가 전달한 값을 가져올 수 있었어요.

```html
<script data-option="value" src="script.js"></script>
<script>
  console.log(document.currentScript.dataset.option); // "value"
</script>
```

그리고 위처럼 모듈이 아닌 스크립트에서는 `document.currentScript`를 사용하여 사용자가 전달한 값을 가져올 수 있어요.

하지만 ESM에서는 위와 같은 방법을 사용할 수 없었고, 이를 해결하기 위해 `import.meta`가 추가됐어요.

```javascript
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

먼저 위처럼 `import.meta.url`을 사용하여 현재 모듈의 URL을 가져올 수 있어요. 이를 `url.fileURLToPath()`, `path.dirname()`와 함께 사용하여 현재 파일의 경로와 디렉토리 경로를 가져올 수 있어요.

> Node.js 20.11.0 버전부터 `import.meta.dirname`와 `import.meta.filename`이 추가되어 `import.meta.url`을 사용하지 않아도 돼요.

```html
<script type="module">
  import "./module.js?query=1";
</script>
```

```javascript
// module.js
new URL(import.meta.url).searchParams.get("query"); // "1"
```

그리고 위처럼 쿼리 스트링을 사용하여 사용자가 전달한 값을 가져올 수 있어요. 이를 통해 `document.currentScript`와 같이 사용자가 전달한 값에 따라 동작을 다르게 할 수 있어요.

> 참고: [GitHub: tc39/proposal-import-meta](https://github.com/tc39/proposal-import-meta), [MDN: import.meta](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta)
