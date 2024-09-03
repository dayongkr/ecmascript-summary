# ECMAScript 7 (ES7 or ECMAScript 2016)

ECMAScript 2016은 Ecma TC39의 매년 새로운 버전을 발표하는 정책과 공개적인 프로세스를 따르는 첫 번째 버전입니다. 따라서 해당 버전부터는 작업 기록과 소스 파일을 [TC39 GitHub 레포지토리](https://github.com/tc39)에서 찾을 수 있습니다.

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

`includes` 메서드는 배열에 특정 요소가 포함되어 있으면 `true`를 반환하고, 그렇지 않으면 `false`를 반환합니다. 두 번째 인수로 시작 인덱스(Zero-based index)를 전달하면 해당 인덱스부터 검색을 시작합니다.

이전에는 `indexOf` 메서드를 사용하여 요소의 인덱스를 찾은 다음, 해당 인덱스가 `-1`이 아닌지 확인해야 했습니다. 해당 표현식은 의미가 명확하지 않고, `indexOf`는 엄격한 동일성 비교(`===`)을 하기 때문에 `[NaN].indexOf(NaN) === -1`는 `false`를 반환하는 등의 문제가 있었습니다. `includes` 메서드는 `SameValueZero` 알고리즘을 사용하여 동일성 비교를 하기 때문에 `NaN`을 찾을 수 있는 등 더 직관적이고 유용한 메서드입니다.

원래는 `constains` 메서드로 제안되었으나, `DOMStringList` 그리고 `DOMTokenList`와 같은 클래스들에 이미 `contains` 메서드가 존재하기 때문에 웹 호환성을 위해 `includes`로 변경되었습니다.

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

거듭제곱 연산자 (`**`)는 왼쪽 피연산자를 밑, 오른쪽 피연산자를 지수로 한 값을 구합니다. `Math.pow` 메서드와 동일한 동작(`Number::exponentiate`)하지만 보다 간결하고 가독성이 좋고 `BigInt`를 지원한다는 장점이 있습니다.

`IEEE 754-2019` 표준에서는 `1 ** Infinity`, `1 ** NaN`등의 연산 결과를 1로 정의하였으나 ECMAScript의 첫 번째 버전에서 `NaN`으로 정의했기 때문에 호환성을 위해 `NaN`으로 유지되었습니다. 실제로 파이썬과 같은 다른 언어들은 `1`을 반환합니다.

> 참고: [MDN - Exponentiation Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Exponentiation), [Github Repo - tc39/proposal-exponentiation-operator](https://github.com/tc39/proposal-exponentiation-operator)
