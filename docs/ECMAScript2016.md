# ECMAScript 7 (ES7 or ECMAScript 2016)

ECMAScript 2016은 Ecma TC39의 매년 새로운 버전을 발표하는 정책과 공개적인 프로세스를 따르는 첫 번째 버전입니다. 해당 버전부터는 작업 기록과 소스 파일을 [TC39 GitHub 리포지토리](https://github.com/tc39)에서 찾을 수 있습니다.

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

`includes`는 배열에 특정 요소가 포함되어 있으면 `true`를, 그렇지 않으면 `false`를 반환합니다. 두 번째 인수로 시작 인덱스(Zero-based index)를 전달하면 해당 인덱스부터 검색을 시작합니다.

이전에는 `indexOf`를 사용하여 요소의 인덱스를 찾은 후, 해당 인덱스가 `-1`이 아닌지 확인해야 했습니다. 그러나 `indexOf`는 엄격한 동일성 비교(`===`)를 사용하기 때문에 `[NaN].indexOf(NaN) === -1`과 같이 `NaN`을 찾지 못하는 문제가 있었습니다.

하지만 `includes`는 SameValueZero 알고리즘을 사용하여 `NaN`도 찾을 수 있으며, 보다 직관적이고 명확한 메서드입니다.

원래 `contains`로 제안되었으나, `DOMStringList`와 `DOMTokenList`에서 이미 `contains`를 사용하고 있어 웹 호환성을 위해 `includes`로 변경되었습니다.

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

거듭제곱 연산자(`**`)는 왼쪽 피연산자를 밑으로, 오른쪽 피연산자를 지수로 사용하여 값을 계산합니다. 이는 Math.pow와 동일한 동작을 하지만, 코드가 더 간결하고 가독성이 좋습니다. 또한, BigInt 타입도 지원합니다.

비록 IEEE 754-2019 표준에서는 `1 ** Infinity` 와 `1 ** NaN`의 결과를 `1`로 정의했지만, ECMAScript는 첫 번째 버전에서 이 결과를 `NaN`으로 정의했으므로 호환성을 유지하기 위해 이를 따릅니다.

> 참고: [MDN - Exponentiation Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation), [Github Repo - tc39/proposal-exponentiation-operator](https://github.com/tc39/proposal-exponentiation-operator)
