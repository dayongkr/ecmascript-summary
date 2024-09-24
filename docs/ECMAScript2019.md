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

```javascript
const sym = Symbol("foo");

console.log(sym); // Symbol(foo)
console.log(sym.description); // foo
```

심볼(Symbol)은 생성할 때 선택적으로 설명(description)을 정의할 수 있습니다. 이러한 설명은 `Symbol.prototype.toString` 메서드를 통해 확인할 수 있었습니다. 하지만 해당 메서드는 설명만 반환하지 않고 `Symbol()`과 같은 형태로 반환되기 때문에, 설명만 추출하기 위해서는 추가적인 처리가 필요했습니다.

> 예를 들어 `Symbol("foo").toString().slice(7, -1); // foo` 와 같은 방법으로 설명을 추출할 수 있습니다.

ECMAScript 2019에서는 이러한 설명을 추출하기 위한 `Symbol.prototype.description` 프로퍼티를 추가하게 되었습니다. 이를 통해 명시적으로 설명을 추출할 수 있게 되었습니다.

> 참고: [Github - proposal-Symbol-description](https://github.com/tc39/proposal-Symbol-description), [MDN - Symbol.prototype.description](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/description)

## `Function.prototype.toString` revision

해당 제안은 전방 호환성(Future-compatibility)가 없는 요구사항을 제거하고, 반환값을 명시적으로 정의하는 것을 목표로 합니다.

### 1. 전방 호환성(Future-compatibility)가 없는 요구사항 제거

이전에는 `toString` 메서드가 구조적으로 유효한 ECMAScript를 반환하지 못했다면, `eval` 함수를 통해 해당 문자열을 실행하면 `SyntaxError`가 발생해야 한다는 요구사항이 있었습니다. 이러한 요구사항은 전방 호환성을 제공하지 못하기 때문에, 해당 요구사항은 제거되었고 `[native code]`와 같은 문자열을 반환하는 것으로 대체되었습니다.

### 2. 반환값을 명시적으로 정의

1. 바운드된 익명 함수이거나 내장 함수인 경우, 함수 바디에 `[native code]`를 넣어 반환
   a. 익명 함수가 아닌 경우, 잘 알려진 내장 객체(Well-known intrinsic object; `Array.prototype.push`)인 경우, 함수 이름을 함께 반환
2. 함수 내부에 `[[SourceText]]` 내부 슬롯이 있는 경우, 해당 속성을 반환
3. 호출 가능한 객체인 경우, NativeFunction 형태 즉, 바디에 `[native code]`를 넣어 반환

예외 상황에는 `TypeError`를 던지도록 명시되어 있습니다.

### 예시

```javascript
// 바운드된 익명 함수
(() => {}).bind({}).toString(); // "() => { [native code] }"

// 내장 함수
Array.prototype.push.toString(); // "function push() { [native code] }"

// 함수 내부에 [[SourceText]] 내부 슬롯이 있는 경우
(function foo() {
  console.log("foo");
}).toString(); // 'function foo() {\n  console.log("foo");\n}'

// arrow function
(() => {}).toString(); // "() => {}"

// generator function
(function* () {}).toString(); // "function*() {}"

// async function
(async function () {}).toString(); // "async function() {}"
```

> 참고: [tc39 - Function.prototype.toString revision](https://tc39.es/Function-prototype-toString-revision)

## `Object.fromEntries`

`Object.fromEntries` 메서드는 `Object.entries` 메서드의 반대 역할을 수행합니다. 즉, `Object.entries` 메서드는 객체를 배열로 변환하는 반면, `Object.fromEntries` 메서드는 배열을 객체로 변환합니다.

```javascript
obj = Object.fromEntries([
  ["foo", "bar"],
  ["baz", 42],
]); // { foo: "bar", baz: 42 }
```

Map 같은 경우에는 이전부터 `new Map(Object.entries(obj))`와 같은 방법으로 배열을 Map으로 변환할 수 있었지만, 객체는 `Object.entries(obj).reduce((acc, [key, val]) => ({ ...acc, [key]: val }), {})`와 같은 방법으로만 배열을 객체로 변환할 수 있었습니다. 이러한 번거로움을 해소하기 위해 `Object.fromEntries` 메서드가 추가되었습니다.

### `Object.fromEntries` 사용 예시

```javascript
const map = new Map([
  ["foo", "bar"],
  ["baz", 42],
]);

JSON.stringify(map); // "{}"
JSON.stringify(Object.fromEntries(map)); // {"foo":"bar","baz":42}
```

Map 객체는 `JSON.stringify` 메서드를 통해 직렬화할 수 없습니다. 하지만 `Object.fromEntries` 메서드를 통해 Map 객체를 객체로 변환한 후 직렬화할 수 있습니다.

> 참고: [Github - proposal-object-from-entries](https://github.com/tc39/proposal-object-from-entries),[MDN - Object.fromEntries](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)

## Well-formed `JSON.stringify`

## `String.prototype.{trimStart,trimEnd}`

## `Array.prototype.{flat,flatMap}`
