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
