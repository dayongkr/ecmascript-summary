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

## `Promise.allSettled`

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
