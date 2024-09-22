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

```javascript
const LS = "\u2028"; // Line separator
const PS = "\u2029"; // Paragraph separator

JSON.parse(`{"lineSeparator":"${LS}"}`); // SyntaxError
JSON.parse(`{"paragraphSeparator":"${PS}"}`); // SyntaxError
```

ECMAScript는 JSON을 `JSON.parse` 하위집합(Subset)으로 주장하고 있었지만, 실제로는 JSON에서 지원하는 모든 유니코드 이스케이프 시퀀스를 지원하지 않았습니다. 예를 들어, 위 코드에서는 JSON에서 지원하는 유니코드 이스케이프 시퀀스인 Line separator(`\u2028`)와 Paragraph separator(`\u2029`)를 사용했지만, ECMAScript에서는 SyntaxError가 발생합니다.

따라서 ECMAScript 2019에서는 ECMA-262의 구문을 JSON의 상위집합(Superset)으로 확장하기 위해, 위와 같은 두 개의 유니코드 이스케이프 시퀀스를 지원하게 되었습니다.

> 참고: [Github - proposal-json-superset](https://github.com/tc39/proposal-json-superset?tab=readme-ov-file), [tc39 - JSON superset](https://tc39.es/proposal-json-superset/)

## `Symbol.prototype.description`

## `Function.prototype.toString` revision

## `Object.fromEntries`

## Well-formed `JSON.stringify`

## `String.prototype.{trimStart,trimEnd}`

## `Array.prototype.{flat,flatMap}`
