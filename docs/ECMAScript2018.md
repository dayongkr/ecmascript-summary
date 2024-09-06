# ECMAScript 9 (ES9 or ECMAScript 2018)

## 목차

- [Lifting template literal restriction](#lifting-template-literal-restriction)
- [`s` (`dotAll`) flag for regular expressions](#s-dotall-flag-for-regular-expressions)
- [RegExp named capture groups](#regexp-named-capture-groups)
- [Object Rest/Spread Properties](#object-restspread-properties)
- [RegExp Lookbehind Assertions](#regexp-lookbehind-assertions)
- [RegExp Unicode Property Escapes](#regexp-unicode-property-escapes)
- [`Promise.prototype.finally`](#promiseprototypefinally)
- [Asynchronous Iteration](#asynchronous-iteration)

## Lifting template literal restriction

```typescript
// ES2017
`\u00FF`; // 정상
`\u{2F804}`; // 정상
`\xFF`; // 정상
`\251`; //정상
`\unicode`; // SyntaxError
```

ECMAScript에서는 템플릿 리터럴에 `\u00FF` 또는 `\xFF`와 같은 형태로 작성하는 것을 강제하고 있었습니다. 이는 DSL(Domain Specific Languages)에서 템플릿 리터럴을 사용할 때 불편함을 야기했습니다.

```typescript
function tag(strs) {
  strs[0] === undefined; // true
  strs.raw[0] === "\\unicode and \\u{55}"; // true
}

tag`\unicode and \u{55}`;
`unicode and u{55}`; // SyntaxError
```

이 제한을 해결하기 위해 Tagged Templates에서는 구문 제한을 없애고, 유효하지 않은 escape sequence가 포함되면 cooked value (`strs[0]`)는 `undefined`로 설정되며, raw value (`strs.raw[0]`)는 그대로 유지됩니다. 다만, 일반적인 템플릿 리터럴은 여전히 구문 제한을 적용받습니다.

> 참고: [Github Repo - tc39/proposal-template-literal-revision](https://github.com/tc39/proposal-template-literal-revision), [MDN - Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## `s` (`dotAll`) flag for regular expressions

정규 표현식에서 `.`은 문자 하나를 의미하지만, 두 가지 예외가 있습니다.

1. `.`은 `𐍈`와 같은 astral 문자를 매칭하지 않습니다. 다만, u 플래그를 설정하면 astral 문자도 매칭할 수 있습니다.
2. 개행 문자(`\n`, `\r`, `\u2028`, `\u2029`)는 매칭되지 않습니다.

```javascript
/./.test("\n"); // false
/[\s\S]/.test("\n"); // true
/[^]/.test("\n"); // true
/./.test("\v"); // true
```

두 번째가 문제가 되는데, 첫 번째 문제는 상황에 따라 `\v`을 개행 문자로 인식할 수 있을 때 일관성이 없어지고, 두 번째 문제는 `[\s\S]` 또는 `[^]` 등의 방법으로 해결할 수 있지만 가독성이 떨어진다는 문제가 있습니다.

두 번째가 문제가 되는데, 먼저 상황에 따라 `\v`와 같은 특정 문자를 개행 문자로 간주하고 있을 때 일관성이 떨어집니다. 이 문제는 [\s\S] 또는 [^]와 같은 방식으로 해결할 수 있지만, 이러한 방법은 코드의 가독성을 떨어뜨린다는 단점이 있습니다.

```javascript
/./s.test("\n"); // true
```

이 문제를 해결하기 위해 s 플래그가 도입되었습니다. s 플래그를 사용하면 개행 문자를 포함한 모든 문자를 매칭할 수 있습니다.

> 참고: [MDN - RegExp.prototype.dotAll](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/dotAll), [Github Repo - tc39/proposal-regexp-dotall-flag](https://github.com/tc39/proposal-regexp-dotall-flag?tab=readme-ov-file)

## RegExp named capture groups

정규 표현식에는 capture group이라는 개념이 있습니다. 이는 `()`로 묶인 부분을 매칭한 결과를 저장하는 역할을 합니다.

```javascript
const before = /(\d{4})-(\d{2})-(\d{2})/;
const beforeResult = before.exec("2024-09-04");

beforeResult[1]; // "2024"
```

하지만 위와 같이 순서에 따라 매칭된 결과에 접근하는 방식은 가독성이 떨어지고, 유지보수가 어려울 수 있습니다.

이 문제를 해결하기 위해 `(?<name>...)` 형식으로 이름을 지정할 수 있는 named capture groups가 도입되었습니다.

```javascript
const after = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const afterResult = after.exec("2024-09-04");

afterResult.groups.year; // "2024"
```

이를 통해 매칭된 결과에 이름을 부여해 더 직관적으로 접근할 수 있습니다.

### Named Backreferences

정규 표현식에는 backreference라는 개념도 있습니다. 이는 이전에 매칭된 그룹을 다시 참조하는 기능입니다.

```javascript
const before = /(')(.*)\1/;
const beforeResult = before.exec("'Hello'");

beforeResult[2]; // "Hello"
```

이 역시 번호로 접근하는 방식은 가독성이 떨어지기 때문에, `\k<name>` 형식으로 이름을 지정할 수 있는 named backreferences가 도입되었습니다.

```javascript
const after = /(?<quote>'(?<content>.*)'\k<quote>)/;
const afterResult = after.exec("'Hello'");

afterResult.groups.content; // "Hello"
```

이처럼 backreference에도 이름을 지정해 더 직관적으로 접근할 수 있습니다.

> 참고: [Github Repo - tc39/proposal-regexp-named-groups](https://github.com/tc39/proposal-regexp-named-groups), [MDN - Named capturing group](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Named_capturing_group), [MDN - Named backreference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Named_backreference)

## Object Rest/Spread Properties

```javascript
// Rest elements
const [first, ...rest] = [1, 2, 3, 4, 5];
first; // 1
rest; // [2, 3, 4, 5]

// Spread elements
const arr = [1, 2, 3];
const arr2 = [...arr, 4, 5];
arr2; // [1, 2, 3, 4, 5]
```

ES6에 위 예시와 같이 배열을 위한 Rest/Spread elements가 도입되었습니다.

ES9에서는 이 기능을 객체를 대상으로도 사용할 수 있도록 Object Rest/Spread properties가 도입되었습니다.

```javascript
// Rest properties
const { first, ...rest } = { first: 1, second: 2, third: 3 };
first; // 1
rest; // { second: 2, third: 3 }

// Spread properties
const obj = { first: 1, second: 2 };
const obj2 = { ...obj, third: 3 };
obj2; // { first: 1, second: 2, third: 3 }
```

위 예시와 같이 배열과 비슷한 방식으로 객체에 대한 Rest/Spread properties를 사용할 수 있습니다.

객체의 특정 속성만 분리하거나 새로운 속성을 추가할 때 매우 유용합니다.

## RegExp Lookbehind Assertions

ECMAScript는 Lookahead Assertions를 지원합니다. 이 기능은 특정 패턴이 뒤에 오는 경우에만 매칭을 허용하며, 조건으로 지정한 패턴은 매칭 결과에 포함되지 않습니다.

```javascript
const regex = /\d+(?=\₩)/;
"100000₩".match(regex); // ["100000"]
```

위 예시에서, Lookahead는 숫자 뒤에 ₩ 기호가 있을 때만 숫자를 매칭합니다. 매칭 결과에는 Lookahead 조건으로 지정한 ₩ 기호는 포함되지 않습니다.

> 해당 예시는 긍정형 Lookahead입니다. 부정형 Lookahead는 `(?!)`를 사용하여 특정 패턴이 존재하지 않을 때 매칭할 수 있습니다.

Lookbehind Assertions는 Lookahead와 반대 방향으로 동작하여, 특정 패턴이 앞에 있을 때만 매칭을 허용합니다.

```javascript
const regex = /(?<=\$)\d+/;
"$100000".match(regex); // ["100000"]
```

위 예시에서 Lookbehind는 `$` 기호 앞에 있는 숫자만 매칭하고, 매칭 결과에는 Lookbehind 조건으로 지정한 `$` 기호는 포함되지 않습니다.

부정형 Lookbehind는 `(?<!)`로 표현하며, 특정 패턴이 존재하지 않는 경우에만 매칭할 수 있습니다.

## RegExp Unicode Property Escapes

## `Promise.prototype.finally`

## Asynchronous Iteration
