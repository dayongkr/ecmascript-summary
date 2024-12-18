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

ECMAScript에서는 템플릿 리터럴에 `\u00FF`나 `\xFF` 형태처럼 유효한 escape sequence를 사용해야 했어요. 이 제한은 DSL(Domain Specific Languages)에서 템플릿 리터럴을 사용할 때 불편함을 줬어요. 예를들어 `\unicode`와 같은 유효하지 않은 escape sequence를 사용할 때, SyntaxError가 발생했어요.

```typescript
function tag(strs) {
  strs[0] === undefined; // true
  strs.raw[0] === "\\unicode and \\u{55}"; // true
}

tag`\unicode and \u{55}`;
`unicode and u{55}`; // SyntaxError
```

이 문제를 해결하기 위해 Tagged Template에서는 구문 제한을 없애고, 유효하지 않은 escape sequence를 포함하면 cooked value(`strs[0]`)는 `undefined`가 되고, raw value(`strs.raw[0]`)는 그대로 유지되도록 했어요. 다만, 일반 템플릿 리터럴은 여전히 구문 제한을 적용받아요.

> 참고: [Github Repo - tc39/proposal-template-literal-revision](https://github.com/tc39/proposal-template-literal-revision), [MDN - Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## `s` (`dotAll`) flag for regular expressions

정규 표현식에서 `.`은 문자 하나를 의미하지만, 두 가지 예외가 있어요.

1. `.`은 `𐍈`와 같은 astral 문자를 매칭하지 않아요. 다만, u 플래그를 설정하면, astral 문자도 매칭할 수 있어요.
2. 개행 문자(`\n`, `\r`, `\u2028`, `\u2029`)는 매칭되지 않아요.

두 번째가 문제가 되는데, 먼저 상황에 따라 `\v`와 같은 특정 문자를 개행 문자로 간주하고 있으면, `\v`는 매칭되는데 다른 개행 문자는 매칭되지 않아 일관성을 떨어뜨려요.

```javascript
/./.test("\n"); // false
/[\s\S]/.test("\n"); // true
/[^]/.test("\n"); // true
/./.test("\v"); // true
```

위처럼 `[\s\S]` 또는 `[^]`와 같은 방식으로 해결할 수 있지만, 이러한 방법은 코드의 가독성을 떨어뜨려요.

```javascript
/./s.test("\n"); // true
```

이 문제를 해결하기 위해 `s` 플래그가 도입되었어요. `s` 플래그를 사용하면, 개행 문자를 포함한 모든 문자를 매칭할 수 있어요.

> 참고: [MDN - RegExp.prototype.dotAll](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/dotAll), [Github Repo - tc39/proposal-regexp-dotall-flag](https://github.com/tc39/proposal-regexp-dotall-flag?tab=readme-ov-file)

## RegExp named capture groups

```javascript
const before = /(\d{4})-(\d{2})-(\d{2})/;
const beforeResult = before.exec("2024-09-04");

beforeResult[1]; // "2024"
```

정규 표현식에는 캡처 그룹(capture group)이라는 개념이 있어요. 이는 `()`로 묶은 부분을 매칭한 결과를 저장하는 역할을 해요. 하지만 순서에 따라 매칭된 결과에 접근하는 방식은 가독성이 떨어지고, 유지보수가 어려울 수 있어요.

```javascript
const after = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const afterResult = after.exec("2024-09-04");

afterResult.groups.year; // "2024"
```

이를 해결하기 위해 `(?<name>...)` 형식으로 이름을 지정할 수 있는 named capture groups가 도입되었어요. 이를 통해 매칭된 결과에 이름을 부여해 더 직관적으로 접근할 수 있어요.

### Named Backreferences

정규 표현식에는 Backreference라는 이전에 매칭된 그룹을 다시 참조하는 기능이 있어요.

```javascript
const before = /(')(.*)\1/;
const beforeResult = before.exec("'Hello'");

beforeResult[2]; // "Hello"
```

이 역시 번호로 접근하는 방식은 가독성이 떨어지기 때문에, `\k<name>` 형식으로 이름을 지정할 수 있는 Named Backreferences가 도입되었어요.

```javascript
const after = /(?<quote>'(?<content>.*)'\k<quote>)/;
const afterResult = after.exec("'Hello'");

afterResult.groups.content; // "Hello"
```

이렇게 이름을 지정해 더 직관적으로 접근할 수 있어요.

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

ES6에서는 배열에 대해 Rest/Spread 요소를 사용할 수 있었어요.

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

ES9에서는 이 기능을 객체에도 적용할 수 있게 되었어요. 이를 통해 객체에서 특정 속성을 쉽게 추출하거나 새로운 속성을 추가하는 작업을 더 편리하게 할 수 있어요.

> 참고: [MDN - Spread properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals)

## RegExp Lookbehind Assertions

정규 표현식에서는 Lookahead Assertions를 통해 특정 패턴이 뒤에 오는 경우에만 매칭을 허용할 수 있어요. 여기서 조건으로 지정한 패턴은 매칭 결과에 포함되지 않아요.

```javascript
const regex = /\d+(?=\₩)/;
"100000₩".match(regex); // ["100000"]
```

위와 같은 경우, 숫자 뒤에 `₩` 기호가 있을 때만 숫자를 매칭하고 매칭
결과에는 Lookahead 조건으로 지정한 `₩` 기호는 포함되지 않아요.

```javascript
const regex = /(?<=\$)\d+/;
"$100000".match(regex); // ["100000"]
```

Lookbehind Assertions는 이와 반대로 특정 패턴이 앞에 오는 경우에만 매칭을 허용할 수 있어요. 이 역시 매칭 결과에는 조건에 해당하는 부분은 결과에 포함되지 않아요. 작성 방법은 Lookahead와 유사하지만, `(?<=...)` 형식으로 사용해요.

## RegExp Unicode Property Escapes

유니코드 표준에서는 각 유니코드 문자에 대해 다양한 속성을 할당하고 있어요. 이를 활용하면 한글 혹은 이모지만 매칭하는 정규 표현식을 쉽게 작성할 수 있어요. 이러한 기능을 지원하기 위해 ES9에서는 General_Category, Script, Script_Extensions 속성을 정규 표현식에서 사용할 수 있게 되었어요.

```javascript
const hangul = /\p{Script=Hangul}/gu;
const lower = /\p{Lowercase}/gu;
const emoji = /\p{Emoji}/gu;
const string = "안녕하세요 Hello 🌍";

string.match(hangul); // ["안", "녕", "하", "세", "요"]
string.match(lower); // ["e", "l", "l", "o"]
string.match(emoji); // ["🌍"]
```

사용 방법은 아래와 같아요.

- General_Category 속성이나 [Binary Property](https://tc39.es/ecma262/multipage/text-processing.html#table-binary-unicode-properties)는 `\p{...}`와 `\P{...}` 형식으로 사용할 수 있어요.
- [그 외의 속성들](https://tc39.es/ecma262/multipage/text-processing.html#table-nonbinary-unicode-properties)은 `\p{property=value}`와 `\P{property=value}` 형식으로 사용할 수 있어요.

참고로 `u` 플래그를 사용해야 유니코드 속성을 사용할 수 있어요.

> 참고: [MDN - Unicode character class escape: \p{...}, \P{...}](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Unicode_character_class_escape), [Github Repo - tc39/proposal-regexp-unicode-property-escapes](https://github.com/tc39/proposal-regexp-unicode-property-escapes?tab=readme-ov-file)
>
> Script 속성의 범위가 궁금하면, [Scripts.txt](https://www.unicode.org/Public/12.1.0/ucd/Scripts.txt)에서 확인할 수 있어요.
>
> 추가 학습 자료: [Unicode Character Ranges](https://jrgraphix.net/r/Unicode/), [D2 - 한글 인코딩의 이해 2편: 유니코드와 Java를 이용한 한글 처리](https://d2.naver.com/helloworld/76650)

## `Promise.prototype.finally`

기존에 존재하던 `Promise`의 메소드는 성공한 경우에만 실행되거나 실패한 경우에만 실행됐어요. 따라서 성공과 실패에 상관없이 항상 실행되는 메서드를 위해 많은 `Promise` 라이브러리에서 `finally` 메서드를 제공했어요. 이러한 `finally` 메서드는 ES9에서 `Promise.prototype.finally` 메서드로 표준화됐어요.

> 이전에는 `promise.then(onFinally, onFinally)`와 같은 방식으로 이를 구현했지만, 이는 `Promise`의 결과를 변경할 수 있어 적절하지 않았어요.

```javascript
await Promise.resolve(1).then(
  () => {},
  () => {}
); // undefined
await Promise.resolve(1)
  .then(() => {})
  .finally(() => {}); // 1
```

위 예시와 같이, `finally` 메서드는 `then` 메서드와 같이 메서드 체인 형태로 사용할 수 있으며, `Promise` 결과를 변경하지 않고 항상 실행돼요.

```javascript
Promise.resolve(1).finally(() => {
  throw 0;
}); // Uncaught (in promise) 0
Promise.resolve(1).finally(() => Promise.reject(0)); // Uncaught (in promise) 0
```

단, finally 내부에서 에러를 발생시키거나 거부된 `Promise`가 반환하면, 앞의 `Promise` 결과와 상관없이 거부된 `Promise`가 반환돼요.

> 참고: [MDN - Promise.prototype.finally()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally), [Github Repo - tc39/proposal-promise-finally](https://github.com/tc39/proposal-promise-finally)

## Asynchronous Iteration

ES6에서는 iterator를 사용해 순차적으로 데이터를 다룰 수 있었지만, 이는 동기적인 작업만 다룰 수 있었어요. ES9에서는 비동기 작업을 처리할 수 있는 비동기 이터레이션(asynchronous iteration)이 도입됐어요.

```javascript
const asyncIterable = {
  [Symbol.asyncIterator]() {
    let i = 0;
    return {
      async next() {
        if (i < 3) {
          return { value: i++, done: false };
        }
        return { done: true };
      },
      async return() {
        // Optional: if the loop is terminated early
        return { done: true };
      },
    };
  },
};

for await (const value of asyncIterable) {
  console.log(value); // 0, 1, 2
}
```

비동기 iterable 객체는 `Symbol.asyncIterator` 메서드를 구현해야 하며, `next` 메서드는 Promise를 반환해야 해요. `return` 메서드는 옵션으로, 이터레이션을 중단할 때 호출돼요.

이처럼 만든 비동기 iterable 객체는 `for await...of` 구문을 사용해 순회할 수 있어요.

```javascript
async function* asyncGenerator() {
  let i = 0;
  while (i < 3) {
    yield i++;
  }
}

for await (const value of asyncGenerator()) {
  console.log(value); // 0, 1, 2
}
```

위처럼, async generator 함수를 사용하면, 비동기 iterable 객체를 쉽게 만들 수 있어요.

```javascript
function* generator() {
  yield 1;
  yield Promise.resolve(2);
  yield 3;
}

for await (const value of generator()) {
  console.log(value); // 1, 2, 3
}

for (const value of generator()) {
  console.log(value); // 1, Promise { 2 }, 3
}
```

마지막으로 `for await...of` 구문은 동기적인 이터러블에 대해서도 사용할 수 있어요. 위 예시에서 `Promise.resolve(2)`가 해결되기 전까지 대기하고, 해결된 후에 순회를 계속해요.

따라서 `Promise.reject`와 같이 거부된 프로미스가 반환되면 `for await...of` 구문은 에러를 발생시키고 `for...of` 구문은 그대로 진행해요.

> 참고: [MDN - for await...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of), [Github Repo - tc39/proposal-async-iteration](https://github.com/tc39/proposal-async-iteration)
