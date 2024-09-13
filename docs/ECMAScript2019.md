# ECMAScript 10 (ES10 or ECMAScript 2019)

## 목차

- [Optional `catch` binding](#optional-catch-binding)
- [JSON superset](#json-superset)
- [`Symbol.prototype.description`](#symbolprototypedescription)
- [`Function.prototype.toString` revision](#functionprototypetostring-revision)
- [`Object.fromEntries`](#objectfromentries)
- [Well-formed `JSON.stringify`](#well-formed-jsonstringify)
- [`String.prototype.{trimStart,trimEnd}`](#stringprototypetrimstarttrimend)
- [`Array.prototype.{flat,flatMap}`](#arrayprototypeflatflatmap)

## Optional `catch` binding

```javascript
try {
  throw new Error("Whoops!");
} catch (unused) {}
```

위 코드처럼 종종 `catch` 블록에 있는 Catch Parameter를 사용하지 않는 경우가 있습니다. 이러한 경우에 편의를 위해 `catch` 블록에 Catch Parameter를 생략할 수 있게 되었습니다.

```javascript
try {
  throw new Error("Whoops!");
} catch {}
```

단순히 문법적인 부분만 변경된 것이기 때문에, 이 변경은 기존 코드와의 호환성을 해치지 않습니다.

> 참고: [Github - proposal-optional-catch-binding](https://github.com/tc39/proposal-optional-catch-binding), [tc39 - Optional catch binding](https://tc39.es/proposal-optional-catch-binding/)

## JSON superset

## `Symbol.prototype.description`

## `Function.prototype.toString` revision

## `Object.fromEntries`

## Well-formed `JSON.stringify`

## `String.prototype.{trimStart,trimEnd}`

## `Array.prototype.{flat,flatMap}`
