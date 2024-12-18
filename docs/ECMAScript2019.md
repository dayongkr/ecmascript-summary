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

`catch` 블록에서 종종 Catch Parameter를 사용하지 않을 때가 있어요. 이런 경우, 편의를 위해 Catch Parameter를 생략할 수 있도록 변경되었어요.

```javascript
try {
  throw new Error("Whoops!");
} catch {}
```

이는 기존 코드와의 호환성을 해치지 않아요.

> 참고: [Github - proposal-optional-catch-binding](https://github.com/tc39/proposal-optional-catch-binding), [tc39 - Optional catch binding](https://tc39.es/proposal-optional-catch-binding/)

## JSON superset

ECMAScript는 JSON을 `JSON.parse`의 subset(부분집합)으로 주장하고 있었지만, 실제로는 JSON에서 지원하는 모든 유니코드 이스케이프 시퀀스를 지원하지 않았어요.

```javascript
const LS = "\u2028"; // Line separator
const PS = "\u2029"; // Paragraph separator

JSON.parse(`{"lineSeparator":"${LS}"}`); // SyntaxError
JSON.parse(`{"paragraphSeparator":"${PS}"}`); // SyntaxError
```

예를 들어, 위 코드에서는 JSON에서 지원하는 유니코드 이스케이프 시퀀스인 Line separator(`\u2028`)와 Paragraph separator(`\u2029`)를 사용했지만, ECMAScript에서는 `SyntaxError`가 발생해요.

따라서 ECMAScript의 구문을 JSON의 상위집합(Superset)으로 확장하기 위해, 위와 같은 두 개의 유니코드 이스케이프 시퀀스를 지원하게 됐어요.

> 참고: [Github - proposal-json-superset](https://github.com/tc39/proposal-json-superset?tab=readme-ov-file), [tc39 - JSON superset](https://tc39.es/proposal-json-superset/)

## `Symbol.prototype.description`

```javascript
const sym = Symbol("foo");

console.log(sym); // Symbol(foo)
console.log(sym.description); // foo
```

심볼(Symbol)을 생성할 때 선택적으로 설명(description)을 정의할 수 있어요. 기존에는 `Symbol.prototype.toString`을 통해 설명을 추출했지만, 설명만 추출하려면 추가적인 처리가 필요했어요. 이제 `Symbol.prototype.description`을 통해 명시적으로 설명을 추출할 수 있어요.

예를 들어 `Symbol("foo").toString().slice(7, -1); // foo` 와 같은 방법으로 설명을 추출할 수 있었어요. 하지만 이제는 `Symbol("foo").description; // foo` 와 같이 명시적으로 설명을 추출할 수 있어요.

> 참고: [Github - proposal-Symbol-description](https://github.com/tc39/proposal-Symbol-description), [MDN - Symbol.prototype.description](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/description)

## `Function.prototype.toString` revision

해당 제안은 전방 호환성(Future-compatibility)가 없는 요구사항을 제거하고, 반환값을 명시적으로 정의하는 것을 목표로 해요.

### 1. 전방 호환성(Future-compatibility)가 없는 요구사항 제거

이전에는 `toString` 메서드가 구조적으로 유효한 ECMAScript를 반환하지 못했다면, `eval` 함수를 통해 해당 문자열을 실행하면 `SyntaxError`가 발생해야 한다는 요구사항이 있었어요. 이러한 요구사항은 전방 호환성을 제공하지 못하기 때문에, 해당 요구사항은 제거되었고 `[native code]`와 같은 문자열을 반환하는 것으로 대체되었어요.

### 2. 반환값을 명시적으로 정의

1. 바운드된 익명 함수이거나 내장 함수인 경우, 함수 바디에 `[native code]`를 넣어 반환
   a. 익명 함수가 아닌 경우, 잘 알려진 내장 객체(Well-known intrinsic object; `Array.prototype.push`)인 경우, 함수 이름을 함께 반환
2. 함수 내부에 `[[SourceText]]` 내부 슬롯이 있는 경우, 해당 속성을 반환
3. 호출 가능한 객체인 경우, NativeFunction 형태 즉, 바디에 `[native code]`를 넣어 반환

예외 상황에는 `TypeError`를 던지도록 명시되어 있어요.

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

```javascript
obj = Object.fromEntries([
  ["foo", "bar"],
  ["baz", 42],
]); // { foo: "bar", baz: 42 }
```

`Object.fromEntries` 메서드는 배열을 객체로 변환하는 메서드예요. 이는 `Object.entries` 메서드의 반대 역할을 해요. 이전에는 배열을 객체로 변환하려면 복잡한 방법이 필요했지만, 이제 간단히 처리할 수 있어요.

### 활용: Map 객체를 객체로 변환

```javascript
const map = new Map([
  ["foo", "bar"],
  ["baz", 42],
]);

JSON.stringify(map); // "{}"
JSON.stringify(Object.fromEntries(map)); // {"foo":"bar","baz":42}
```

Map 객체는 `JSON.stringify` 메서드를 통해 직렬화할 수 없어요. 하지만 `Object.fromEntries` 메서드를 통해 Map 객체를 객체로 변환함으로써 직렬화할 수 있어요.

> 참고: [Github - proposal-object-from-entries](https://github.com/tc39/proposal-object-from-entries),[MDN - Object.fromEntries](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)

## Well-formed `JSON.stringify`

[RFC 8529 섹션 8.1](https://datatracker.ietf.org/doc/html/rfc8259#section-8.1)에 따르면, JSON 문자열은 UTF-8 인코딩을 사용해야 한다고 명시되어 있어요. 하지만 이전의 `JSON.stringify` 메서드는 UTF-8로 표현할 수 없는 코드 포인트를 포함하는 문자열을 반환할 수 있었어요.

```javascript
JSON.stringify("\uD800"); // '"�"'
```

특히, 위와 같은 surrogate 코드 포인트(U+D800 ~ U+DFFF)를 포함했어요.

```javascript
JSON.stringify("\uD800"); // '"\\ud800"'
```

이러한 문제를 해결하기 위해 짝이 맞지 않는 surrogate 코드 포인트를 포함하는 문자열을 반환하지 않는 대신, JSON 이스케이프 시퀀스로 표현하게 됐어요

> 참고: [Github - proposal-well-formed-stringify](https://github.com/tc39/proposal-well-formed-stringify?tab=readme-ov-file)

## `String.prototype.{trimStart,trimEnd}`

ECMAScript 2016에서 `String.prototype.trim`만 표준화 했지만, `String.prototype.{trimLeft,trimRight}`는 표준화하지 않았어요.

이를 표준화 했는데, `padStart`와 `padEnd` 메서드와 일관성을 유지하기 위해 `String.prototype.trimStart`와 `String.prototype.trimEnd`로 메서드를 표준화하게 됐어요.

```javascript
String.prototype.trimStart === String.prototype.trimLeft; // true
String.prototype.trimEnd === String.prototype.trimRight; // true
```

하지만 웹 호환성을 위해 `String.prototype.{trimLeft,trimRight}`을 두 메서드의 별칭으로 유지하게 됐어요.

> 참고: [Github - proposal-string-trim-start-end](https://github.com/tc39/proposal-string-left-right-trim?tab=readme-ov-file)

## `Array.prototype.{flat,flatMap}`

```javascript
[1, 2, [3, 4]].flat(); // [1, 2, 3, 4]
[1, 2, [3, [4]]].flat(2); // [1, 2, 3, 4]
```

`Array.prototype.flat` 메서드는 중첩된 배열을 평탄화(flatten)하는 메서드이에요. 이 메서드는 기본적으로 1단계의 중첩 배열만 평탄화하며, 인수를 통해 평탄화할 깊이를 지정할 수 있어요.

```javascript
[1, 2, 3].flatMap((x) => [x, x * 2]); // [1, 2, 2, 4, 3, 6]
[1, 2, 3].map((x) => [x, x * 2]).flat(); // [1, 2, 2, 4, 3, 6]
```

`Array.prototype.flatMap` 메서드는 `map` 메서드를 수행한 후 `flat` 메서드를 수행하는 메서드이에요. 즉, `map` 메서드와 `flat` 메서드를 한 번에 더 효율적으로 수행할 수 있어요.

> 참고: [Github - proposal-flatMap](https://tc39.es/proposal-flatMap/), [MDN - Array.prototype.flat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat), [MDN - Array.prototype.flatMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap)
