# ECMAScript 7 (ES7 or ECMAScript 2016)

ECMAScript 2016은 ECMA TC39에서 매년 새로운 버전을 발표하는 정책과 공개적인 프로세스를 따르는 첫 번째 버전이에요. 이 버전부터는 작업 기록과 소스 파일을 [TC39 GitHub 리포지토리](https://github.com/tc39)에서 찾을 수 있어요.

## 목차

- [`Array.prototype.includes`](#arrayprototypeincludes)
- [Exponentiation Operator](#exponentiation-operator)

## `Array.prototype.includes`

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

`includes`는 배열에 특정 요소가 포함되어 있으면 `true`, 그렇지 않으면 `false`를 반환해요. 두 번째 인수로 시작 인덱스(Zero-based index)를 전달하면 해당 인덱스부터 검색을 시작할 수 있어요.

이전에는 `indexOf`를 사용해서 요소의 인덱스를 찾은 후, 그 인덱스가 `-1`이 아닌지 확인해야 했어요. 하지만 `indexOf`는 엄격한 동일성 비교(`===`)를 사용하기 때문에 `[NaN].indexOf(NaN) === -1`처럼 `NaN`을 찾지 못하는 문제가 있었어요.

`includes`는 SameValueZero 알고리즘을 사용해서 `NaN`도 찾을 수 있고, 보다 직관적이고 명확한 메서드예요.

원래는 `contains`라는 이름으로 제안되었지만, `DOMStringList`와 `DOMTokenList`에서 이미 `contains`를 사용하고 있었기 때문에 웹 호환성을 고려해 `includes`로 이름이 변경됐어요.

> 참고: [MDN - Array.prototype.includes()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes), [Github Repo - tc39/proposal-Array.prototype.includes](https://github.com/tc39/proposal-Array.prototype.includes?tab=readme-ov-file)

## Exponentiation Operator

```javascript
// Math.pow
Math.pow(2, 3); // 8

// Exponentiation Operator
2 ** 3; // 8
2 ** NaN; // NaN
1 ** Infinity; // NaN
```

거듭제곱 연산자(`**`)는 왼쪽 피연산자를 밑으로, 오른쪽 피연산자를 지수로 사용해 값을 계산해요. 이는 `Math.pow`와 동일한 동작을 하지만, 코드가 더 간결하고 가독성이 좋아요. 또한, `BigInt` 타입도 지원해요.

IEEE 754-2019 표준에서는 `1 ** Infinity`와 `1 ** NaN`의 결과를 `1`로 정의하지만, ECMAScript 초기 버전에서는 이를 `NaN`으로 정의했어요. 호환성을 유지하기 위해 이 정의를 그대로 따르고 있어요.

> 참고: [MDN - Exponentiation Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation), [Github Repo - tc39/proposal-exponentiation-operator](https://github.com/tc39/proposal-exponentiation-operator)
