# ECMAScript 11 (ES11 or ECMAScript 2020)

## 목차

- [`String.prototype.matchAll`](#stringprototypematchall)
- [`import()`](#import)
- [`BigInt`](#bigint)
- [`Promise.allSettled`](#promiseallsettled)
- [`globalThis`](#globalthis)
- [`for-in` mechanics](#for-in-mechanics)
- [Optional Chaining](#optional-chaining)
- [Nullish coalescing Operator](#nullish-coalescing-operator)
- [`import.meta`](#importmeta)

## `String.prototype.matchAll`

기존의 `String.prototype.match()` 메서드는 캡처 그룹에 대한 정보를 제공하지 않았다. 따라서 아래와 같은 불편함이 있었다.

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

str.match(regexp); // ["test1", "test2"]

// 캡처 그룹에 대한 정보를 얻기 위해서는 다음과 같이 해야 했다.
const matches = [];
str.replace(regexp, (match, p1, p2, p3, offset, string) => {
  matches.push({ match, p1, p2, p3, offset, string });
});

matches; // [{ match: "test1", p1: "e", p2: "st1", p3: "1", offset: 0, string: "test1test2" }, { match: "test2", p1: "e", p2: "st2", p3: "2", offset: 5, string: "test1test2" }]
```

이를 해결하기 위해 `String.prototype.matchAll()` 메서드가 추가되었다. 이 메서드는 `RegExp` 객체의 `g` 플래그를 사용하여 문자열에서 일치하는 모든 결과를 반환한다.

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

const matches = [...str.matchAll(regexp)];

matches; // [ { 0: "test1", 1: "e", 2: "st1", 3: "1", index: 0, input: "test1test2" }, { 0: "test2", 1: "e", 2: "st2", 3: "2", index: 5, input: "test1test2" } ]
```

`matchAll()` 메서드는 `RegExp` 객체의 `g` 플래그를 사용하여 문자열에서 일치하는 모든 결과를 반환한다. 반환된 값은 `Iterator` 객체이며, `for...of` 구문을 사용하여 순회할 수 있다.

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = "test1test2";

for (const match of str.matchAll(regexp)) {
  console.log(match);
}
```

> [GitHub: tc39/proposal-string-matchall](https://github.com/tc39/proposal-string-matchall), [MDN: String.prototype.matchAll()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll)

## `import()`

웹 환경에서는 성능상의 이유등으로 동적으로 모듈을 가져와야 할 때가 있다. 하지만 이전의 구문 형태로는 동적으로 모듈을 가져오는 것이 불가능했다. 이를 해결하기 위해 `import()` 함수가 추가되었다.

```javascript
import("./module.js")
  .then((module) => {
    // module을 사용한다.
  })
  .catch((err) => {
    // module을 가져오는데 실패했을 때
  });

// 또는 async/await를 사용할 수 있다.
try {
  const module = await import("./module.js");
} catch (err) {
  // module을 가져오는데 실패했을 때
}
```

CJS(CommonJS)의 `require()`와 달리 별칭(alias) 생성을 문법적으로 막았기 때문에 보다 안전하게 사용할 수 있다고 생각한다.

### 일반적인 `import`와 다른 점

- `import()`는 동적으로 모듈을 가져올 수 있다.
- 호이스팅이 되지 않는다.
- 스크립트에도 사용할 수 있다. (`<script>` 태그에서 사용 가능)
- 템플릿 리터럴 등 인자로 동적인 값을 넣을 수 있다.

> [GitHub: tc39/proposal-dynamic-import](https://github.com/tc39/proposal-dynamic-import)

## `BigInt`

자바스크립트에서는 2^53 - 1보다 큰 정수를 표현할 수 없었다.

그렇다보니 나노초 단위의 타임스탬프를 표현 하거나 64비트의 정수를 다루는데 어려움이 있었고, 이를 해결하기 위해 `BigInt`가 추가되었다.

```javascript
const max = Number.MAX_SAFE_INTEGER; // 9007199254740991
max + 1; // 9007199254740991

const bigIntMax = BigInt(Number.MAX_SAFE_INTEGER); // 9007199254740991n
bigIntMax + 1n; // 9007199254740992n
```

`BigInt`는 `Number`와 다르게 `n`을 붙여서 표현한다.

조심해야 할점은 `BigNumber`가 아니기 때문에 나눗셈 연산을 할 때 소수점 이하를 버린다.

```JavaScript
1n / 2n; // 0n
```

> [GitHub: tc39/proposal-bigint](https://github.com/tc39/proposal-bigint?tab=readme-ov-file)

## `Promise.allSettled`

기존의 `Promise.all`, `Promise.race`, `Promise.any`는 단락 평가(short-circuit evaluation)를 사용한다. 즉, 하나라도 실패(rejected)하거나 성공(fulfilled)하면 즉시 반환한다.

이러한 특성 때문에 모든 프로미스가 처리될 때까지 기다리는 방법이 없었다. 이를 해결하기 위해 `Promise.allSettled`가 추가되었다.

`Promise.allSettled`는 모든 프로미스가 처리될 때까지 기다린다.

```javascript
const promises = [fetch("index.html"), fetch("https://does-not-exist/")];
const results = await Promise.allSettled(promises);
const successfulPromises = results.filter((p) => p.status === "fulfilled");
```

`Promise.allSettled`는 다른 메서드와 다르게 `status`, `value`, `reason` 프로퍼티를 가진 객체를 반환한다.

- `status`: 프로미스의 상태를 나타내는 문자열. `"fulfilled"` 또는 `"rejected"`가 될 수 있다.
- `value`: 프로미스가 성공했을 때의 결과 값. `status`가 `"fulfilled"`일 때만 존재한다.
- `reason`: 프로미스가 실패했을 때의 이유. `status`가 `"rejected"`일 때만 존재한다.

> [GitHub: tc39/proposal-promise-allSettled](https://github.com/tc39/proposal-promise-allSettled), [MDN: Promise.allSettled()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

## `globalThis`

## `for-in` mechanics

## Optional Chaining

## Nullish coalescing Operator

프로퍼티에 접근할 때, 프로퍼티가 `null` 또는 `undefined`일 때 대체값을 지정할 때 기존에는 `||` 연산자를 사용했다. 하지만 이는 falsy한 값에 대해서도 대체값을 지정해버리는 문제가 있었다.:

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

이를 해결하기 위해 `??` 연산자가 추가되었다. `??` 연산자는 `null` 또는 `undefined`일 때만 대체값을 지정한다.

```javascript
obj.a ?? "default"; // 0
obj.b ?? "default"; // ""
obj.c ?? "default"; // "default"
obj.d ?? "default"; // "default"
obj.e ?? "default"; // "hello"
```

> [GitHub: tc39/proposal-nullish-coalescing](https://github.com/tc39/proposal-nullish-coalescing)

## `import.meta`

일반적인 경우엔 사용할 일이 없지만, 라이브러리를 만들 때 사용자가 전달한 값에 따라 동작을 다르게 하여 유연성을 높일 수 있다.

CJS(CommonJS)에서는 현재 파일의 경로 또는 모듈의 경로를 알 수 있는 `__dirname`과 `__filename`이 있었다.

```html
<script data-option="value" src="script.js"></script>
```

```javascript
// script.js
document.currentScript.dataset.option; // "value"
```

그리고 위처럼 모듈이 아닌 스크립트에서는 `document.currentScript`를 사용하여 사용자가 전달한 값을 가져올 수 있었다.

하지만 ESM에서는 위와 같은 방법을 사용할 수 없었고, 이를 해결하기 위해 `import.meta`가 추가되었다.

```javascript
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

먼저 위처럼 `import.meta.url`을 사용하여 현재 모듈의 URL을 가져올 수 있다. 이를 `url.fileURLToPath()`, `path.dirname()`와 함께 사용하여 현재 파일의 경로와 디렉토리 경로를 가져올 수 있게 됐다.

> Node.js 20.11.0 버전부터 `import.meta.dirname`와 `import.meta.filename`이 추가되어 `import.meta.url`을 사용하지 않아도 된다.

```html
<script type="module">
  import "./module.js?query=1";
</script>
```

```javascript
// module.js
new URL(import.meta.url).searchParams.get("query"); // "1"
```

그리고 위처럼 쿼리 스트링을 사용하여 사용자가 전달한 값을 가져올 수 있다. 이를 통해 사용자가 전달한 값을 통해 동작을 다르게 할 수 있다.

> [GitHub: tc39/proposal-import-meta](https://github.com/tc39/proposal-import-meta), [MDN: import.meta](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta)
