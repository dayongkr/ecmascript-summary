# ECMAScript 요약 (ES7 ~ ES15) - 한국어

해당 문서는 Stage 4 단계 즉 최종 승인된 ECMAScript의 주요 기능들을 버전별로 정리한 문서입니다. 혼란을 줄이기 위해 각 기능의 제목은 영어로 작성하였으나, 설명은 한국어로 작성하였습니다.

> ES6는 [JAVASCRIPT.INFO](https://javascript.info) 등 잘 정리된 사이트가 많아서 생략하였습니다.

## 목차

- [ECMAScript 요약 (ES7 ~ ES15)](#ecmascript-요약-es7--es15)
  - [목차](#목차)
  - [ECMAScript 개요](#ecmascript-개요)
  - [ES7 (ES2016)](#es7-es2016)
    - [`Array.prototype.includes`](#arrayprototypeincludes)
    - [Exponentiation Operator](#exponentiation-operator)
  - [ES8 (ES2017)](#es8-es2017)
    - [`Object.values`/`Object.entries`](#objectvaluesobjectentries)
    - [String padding](#string-padding)
    - [`Object.getOwnPropertyDescriptors`](#objectgetownpropertydescriptors)
    - [Trailing commas in function parameter lists and calls](#trailing-commas-in-function-parameter-lists-and-calls)
    - [Async Functions](#async-functions)
    - [Shared Memory and Atomics](#shared-memory-and-atomics)
  - [ES9 (ES2018)](#es9-es2018)
    - [Lifting template literal restriction](#lifting-template-literal-restriction)
    - [`s` (`dotAll`) flag for regular expressions](#s-dotall-flag-for-regular-expressions)
    - [RegExp named capture groups](#regexp-named-capture-groups)
    - [Rest/Spread Properties](#restspread-properties)
    - [RegExp Lookbehind Assertions](#regexp-lookbehind-assertions)
    - [RegExp Unicode Property Escapes](#regexp-unicode-property-escapes)
    - [`Promise.prototype.finally`](#promiseprototypefinally)
    - [Asynchronous Iteration](#asynchronous-iteration)
  - [ES10 (ES2019)](#es10-es2019)
    - [Optional `catch` binding](#optional-catch-binding)
    - [JSON superset](#json-superset)
    - [`Symbol.prototype.description`](#symbolprototypedescription)
    - [`Function.prototype.toString` revision](#functionprototypetostring-revision)
    - [`Object.fromEntries`](#objectfromentries)
    - [Well-formed `JSON.stringify`](#well-formed-jsonstringify)
    - [`String.prototype.{trimStart,trimEnd}`](#stringprototypetrimstarttrimend)
    - [`Array.prototype.{flat,flatMap}`](#arrayprototypeflatflatmap)
  - [ES11 (ES2020)](#es11-es2020)
    - [`String.prototype.matchAll`](#stringprototypematchall)
    - [`import()`](#import)
    - [`BigInt`](#bigint)
    - [`Promise.allSettled`](#promiseallsettled)
    - [`globalThis`](#globalthis)
    - [`for-in` mechanics](#for-in-mechanics)
    - [Optional Chaining](#optional-chaining)
    - [Nullish coalescing Operator](#nullish-coalescing-operator)
    - [`import.meta`](#importmeta)
  - [ES12 (ES2021)](#es12-es2021)
    - [`String.prototype.replaceAll`](#stringprototypereplaceall)
    - [`Promise.any`](#promiseany)
    - [WeakRefs](#weakrefs)
    - [Logical Assignment Operators](#logical-assignment-operators)
    - [`Numeric separators`](#numeric-separators)
  - [ES13 (ES2022)](#es13-es2022)
    - [Class Fields](#class-fields)
    - [Regex Match Indices](#regex-match-indices)
    - [Top-level `await`](#top-level-await)
    - [Ergonomic brand checks for private fields](#ergonomic-brand-checks-for-private-fields)
    - [`at()`](#at)
    - [Accessible `Object.prototype.hasOwnProperty`](#accessible-objectprototypehasownproperty)
    - [Class Static Block](#class-static-block)
    - [Error Cause](#error-cause)
  - [ES14 (ES2023)](#es14-es2023)
    - [Array find from last](#array-find-from-last)
    - [Hashbang Grammar](#hashbang-grammar)
    - [Symbols as WeakMap keys](#symbols-as-weakmap-keys)
    - [Change Array by Copy](#change-array-by-copy)
  - [ES15 (ES2024)](#es15-es2024)
    - [Well-Formed Unicode Strings](#well-formed-unicode-strings)
    - [`Atomics.waitAsync`](#atomicswaitasync)
    - [RegExp v flag with set notation + properties of strings](#regexp-v-flag-with-set-notation--properties-of-strings)
    - [Resizable and growable ArrayBuffers](#resizable-and-growable-arraybuffers)
    - [Array Grouping](#array-grouping)
    - [`Promise.withResolvers`](#promisewithresolvers)
    - [ArrayBuffer transfer](#arraybuffer-transfer)

## ECMAScript 개요

ECMAScript는 1996년 11월에 개발되기 시작했고 1997년 6월에 ECMA General Assembly에 승인되었습니다. 해당 표준은 ISO/IEC JTC 1에 제출되었고 ISO/IEC 16262로 표준화되었습니다.

1997년에 첫 번째 버전이 나온 이후, ECMAScript는 가장 널리 사용되는 범용 프로그래밍 언어 중 하나로 성장해왔습니다. ECMAScript는 웹 브라우저에서 실행되는 스크립트 언어로 시작했지만, 이제는 서버 그리고 임베디드 애플리케이션에서도 사용되고 있습니다.

> 참고: [ECMAScript Language Specification - Introduction](https://tc39.es/ecma262/#sec-intro), [TC39 Process Document](https://tc39.es/process-document/)

## ES7 (ES2016)

ECMAScript 2016은 Ecma TC39의 매년 새로운 버전을 발표하는 정책과 공개적인 프로세스를 따르는 첫 번째 버전입니다. 따라서 해당 버전부터는 작업 기록과 소스 파일을 [TC39 GitHub 레포지토리](https://github.com/tc39)에서 찾을 수 있습니다.

### `Array.prototype.includes`

```javascript
// indexOf
[1, 2, 3].indexOf(3) !== -1; // true
[1, 2, 3].indexOf(6) !== -1; // false
[1, 2, NaN].indexOf(NaN) !== -1; // false

// includes
[1, 2, 3].includes(2); // true
[1, 2, 3].includes(4); // false

[1, 2, NaN].includes(NaN); // true

[1, 2, -0].includes(+0); // true
[1, 2, +0].includes(-0); // true

["a", "b", "c"].includes("a"); // true
["a", "b", "c"].includes("a", 1); // false
```

`includes` 메서드는 배열에 특정 요소가 포함되어 있으면 `true`를 반환하고, 그렇지 않으면 `false`를 반환합니다. 두 번째 인수로 시작 인덱스(Zero-based index)를 전달하면 해당 인덱스부터 검색을 시작합니다.

이전에는 `indexOf` 메서드를 사용하여 요소의 인덱스를 찾은 다음, 해당 인덱스가 `-1`이 아닌지 확인해야 했습니다. 해당 표현식은 의미가 명확하지 않고, `indexOf`는 엄격한 동일성 비교(`===`)을 하기 때문에 `[NaN].indexOf(NaN) === -1`는 `false`를 반환하는 등의 문제가 있었습니다. `includes` 메서드는 `SameValueZero` 알고리즘을 사용하여 동일성 비교를 하기 때문에 `NaN`을 찾을 수 있는 등 더 직관적이고 유용한 메서드입니다.

원래는 `constains` 메서드로 제안되었으나, `DOMStringList` 그리고 `DOMTokenList`와 같은 클래스들에 이미 `contains` 메서드가 존재하기 때문에 웹 호환성을 위해 `includes`로 변경되었습니다.

> 참고: [MDN - Array.prototype.includes()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes), [Github Repo - tc39/proposal-Array.prototype.includes](https://github.com/tc39/proposal-Array.prototype.includes?tab=readme-ov-file)

### Exponentiation Operator

```javascript
// Math.pow
Math.pow(2, 3); // 8

// Exponentiation Operator
2 ** 3; // 8
2 ** NaN; // NaN
1 ** Infinity; // NaN
```

거듭제곱 연산자 (`**`)는 왼쪽 피연산자를 밑, 오른쪽 피연산자를 지수로 한 값을 구합니다. `Math.pow` 메서드와 동일한 동작(`Number::exponentiate`)을 하지만 보다 간결하고 가독성이 좋고 BigInt를 지원한다는 장점이 있습니다.

`IEEE 754-2019` 표준에서는 `1 ** Infinity`, `1 ** NaN`등의 연산 결과를 1로 정의하였으나 ECMAScript의 첫 번째 버전에서 NaN으로 정의했기 때문에 호환성을 위해 NaN으로 유지되었습니다. 실제로 파이썬과 같은 다른 언어들은 1을 반환합니다.

> 참고: [MDN - Exponentiation Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation), [Github Repo - tc39/proposal-exponentiation-operator](https://github.com/tc39/proposal-exponentiation-operator)

## ES8 (ES2017)

### `Object.values`/`Object.entries`

### String padding

### `Object.getOwnPropertyDescriptors`

### Trailing commas in function parameter lists and calls

### Async Functions

### Shared Memory and Atomics

## ES9 (ES2018)

### Lifting template literal restriction

### `s` (`dotAll`) flag for regular expressions

### RegExp named capture groups

### Rest/Spread Properties

### RegExp Lookbehind Assertions

### RegExp Unicode Property Escapes

### `Promise.prototype.finally`

### Asynchronous Iteration

## ES10 (ES2019)

### Optional `catch` binding

### JSON superset

### `Symbol.prototype.description`

### `Function.prototype.toString` revision

### `Object.fromEntries`

### Well-formed `JSON.stringify`

### `String.prototype.{trimStart,trimEnd}`

### `Array.prototype.{flat,flatMap}`

## ES11 (ES2020)

### `String.prototype.matchAll`

### `import()`

### `BigInt`

### `Promise.allSettled`

### `globalThis`

### `for-in` mechanics

### Optional Chaining

### Nullish coalescing Operator

### `import.meta`

## ES12 (ES2021)

### `String.prototype.replaceAll`

### `Promise.any`

### WeakRefs

### Logical Assignment Operators

### `Numeric separators`

## ES13 (ES2022)

### Class Fields

### Regex Match Indices

### Top-level `await`

### Ergonomic brand checks for private fields

### `at()`

### Accessible `Object.prototype.hasOwnProperty`

### Class Static Block

### Error Cause

## ES14 (ES2023)

### Array find from last

### Hashbang Grammar

### Symbols as WeakMap keys

### Change Array by Copy

## ES15 (ES2024)

### Well-Formed Unicode Strings

### `Atomics.waitAsync`

### RegExp v flag with set notation + properties of strings

### Resizable and growable ArrayBuffers

### Array Grouping

### `Promise.withResolvers`

### ArrayBuffer transfer
