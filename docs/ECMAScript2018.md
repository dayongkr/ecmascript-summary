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

ECMA Script의 문법은 템플릿 리터럴에서 `\u00FF` 또는 `\xFF`와 같은 형태로 작성하는 것을 강제하고 있습니다. 따라서 DSLs(Domain Specific Languages)와 같은 특정한 문법을 사용할 때 불편함을 겪을 수 있습니다.

```typescript
function tag(strs) {
  strs[0] === undefined; // true
  strs.raw[0] === "\\unicode and \\u{55}"; // true
}

tag`\unicode and \u{55}`;
`unicode and u{55}`; // SyntaxError
```

따라서 해당 제안은 `Tagged Templates`에서는 해당 구문 제한을 없애는 대신, 허락되지 않은 `escape sequence`가 포함되면 `cooked value (strs[0])`는 `undefined`로 설정하고 `raw value(strs.raw[0])`로는 전달 받은 값을 그대로 유지하도록 변경되었습니다. 다만, 일반적인 경우로 템플릿 리터럴을 사용할 때는 여전히 해당 제한이 적용됩니다.

> 참고: [Github Repo - tc39/proposal-template-literal-revision](https://github.com/tc39/proposal-template-literal-revision), [MDN - Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## `s` (`dotAll`) flag for regular expressions

정규 표현식 패턴에서, `.`은 문자 하나를 의미합니다. 하지만 두 가지 예외가 있습니다.

1. `.`은 `𐍈`와 같은 astral 문자는 매칭되지 않습니다. 다만, `u` 플래그를 설정하면 매칭되게 할 수 있습니다.
2. 개행 문자(`\n`, `\r`, `\u2028`, `\u2029`)은 매칭되지 않습니다.

```javascript
/./.test("\n"); // false
/[\s\S]/.test("\n"); // true
/[^]/.test("\n"); // true
/./.test("\v"); // true
```

두 번째가 문제가 되는데, 첫 번째 문제는 상황에 따라 `\v`을 개행 문자로 인식할 수 있을 때 일관성이 없어지고, 두 번째 문제는 `[\s\S]` 또는 `[^]` 등의 방법으로 해결할 수 있지만 가독성이 떨어진다는 문제가 있습니다.

```javascript
/./s.test("\n"); // true
```

이를 해결하기 위해 `s` 플래그가 도입되었습니다. `s` 플래그를 사용하면 개행 문자를 포함한 모든 문자를 매칭할 수 있습니다.

> 참고: [MDN - RegExp.prototype.dotAll](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/dotAll), [Github Repo - tc39/proposal-regexp-dotall-flag](https://github.com/tc39/proposal-regexp-dotall-flag?tab=readme-ov-file)

## RegExp named capture groups

## Rest/Spread Properties

## RegExp Lookbehind Assertions

## RegExp Unicode Property Escapes

## `Promise.prototype.finally`

## Asynchronous Iteration
