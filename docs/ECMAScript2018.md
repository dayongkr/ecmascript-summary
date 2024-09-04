# ECMAScript 9 (ES9 or ECMAScript 2018)

## 목차

- [Lifting template literal restriction](#lifting-template-literal-restriction)
- [`s` (`dotAll`) flag for regular expressions](#s-dotall-flag-for-regular-expressions)
- [RegExp named capture groups](#regexp-named-capture-groups)
- [Rest/Spread Properties](#restspread-properties)
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

## Rest/Spread Properties

## RegExp Lookbehind Assertions

## RegExp Unicode Property Escapes

## `Promise.prototype.finally`

## Asynchronous Iteration
