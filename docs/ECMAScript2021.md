# ECMAScript 12 (ES12 or ECMAScript 2021)

## 목차

- [`String.prototype.replaceAll`](#stringprototypereplaceall)
- [`Promise.any`](#promiseany)
- [WeakRefs](#weakrefs)
- [Logical Assignment Operators](#logical-assignment-operators)
- [`Numeric separators`](#numeric-separators)

## `String.prototype.replaceAll`

```javascript
const str = "안녕 안녕";
console.log(str.replace("안녕", "Hello")); // Hello 안녕
console.log(str.replaceAll("안녕", "Hello")); // Hello Hello
```

`String.prototype.replace`메소드는 첫 번째 매칭된 문자열만 치환해요. 따라서 모든 매칭된 문자열을 치환하려면 정규표현식을 사용하거나, `split` 과`join` 메소드를 함께 사용해야 했어요. `replaceAll` 메소드는 이러한 불편함을 해소해요.

```javascript
const str = "안녕";
console.log(str.replace("", "-")); // -안녕
console.log(str.replaceAll("", "-")); // -안-녕-
```

`replaceAll` 메소드는 `replace` 메소드와 마찬가지로 빈 문자열을 매칭시키면 UCS-2/UTF-16 코드 단위로 문자열을 분리해 치환해요. 즉, 빈 문자열을 매칭시키면 각 문자 사이에 치환 문자열이 삽입돼요.

> 참고: [GitHub: tc39/proposal-string-replaceall](https://github.com/tc39/proposal-string-replaceall)

## `Promise.any`

```javascript
try {
  const result = await Promise.any([
    fetch("https://api.example.com/endpoint1"),
    fetch("https://api.example.com/endpoint2"),
    fetch("https://api.example.com/endpoint3"),
  ]);
  console.log(result);
} catch (error) {
  console.error(error);
}
```

`Promise.any` 메소드는 여러 개의 프로미스 중 하나라도 성공하면 해당 프로미스를 반환해요. 만약 모든 프로미스가 실패하면 `AggregateError`가 반환돼요.

이 기능은 이미 여러 프로미스 라이브러리에서 활발히 제공되고 있었고, 이를 언어 차원에서 표준화하기 위해 제안됐어요.

> 참고: [GitHub: tc39/proposal-promise-any](https://github.com/tc39/proposal-promise-any)

## WeakRefs

`WeakRef`와 `FinalizationRegistry` 클래스는 메모리 누수를 방지하기 위해 도입됐어요. 다만, 가비지 컬렉션의 타이밍은 JS 엔진마다 다르기 때문에 가능하면 사용을 피하는 것을 권장하고 있어요. 심지어 같은 엔진이라도 버전에 따라 다를 수 있기 때문에 주의해야 해요.

```javascript
let target = { name: "target" };
const weakRef = new WeakRef(target);
const finalizationRegistry = new FinalizationRegistry((value) => {
  console.log("FinalizationRegistry", value);
});

finalizationRegistry.register(target, "FinalizationRegistry");

target = null;
weakRef.deref(); // { name: "target" }
// 가비지 컬렉션 이후에 "FinalizationRegistry FinalizationRegistry"가 출력돼요.
weakRef.deref(); // undefined
```

`WeakRef` 클래스는 `deref` 메소드를 통해 약한 참조된 객체를 반환하며, `FinalizationRegistry` 클래스는 `register` 메소드를 통해 객체와 콜백을 등록해요. 강한 참조가 없을 경우, `deref` 메소드는 `undefined`를 반환하며 가비지 컬렉션 후에 `FinalizationRegistry`의 콜백이 실행돼요.

이 기능은 이터러블 WeakMap 또는 파일 누수 탐지 등에 활용될 수 있어요.

> 참고: [GitHub: tc39/proposal-weakrefs](https://github.com/tc39/proposal-weakrefs)

## Logical Assignment Operators

```javascript
const obj = {
  a: "a",
};

obj.a = obj.a ?? "default"; // 'a'
obj.b ?? (obj.b = "default"); // 'default'
```

조건에 따라 값을 할당하거나 변경할 때, 조건문을 사용하거나 위와 같은 표현을 활용해 간결하게 작성하곤 했어요. 하지만 첫 번째 방법은 setter를 항상 호출하게 되어 성능에 영향을 줄 수 있고, 두 번째 방법은 setter를 항상 호출하지는 않지만, 가독성이 떨어질 수 있어요.

```javascript
const obj = {
  a: "a",
};

obj.a ??= "default"; // 'a'
```

이 문제를 해결하기 위해 `??=`, `||=`, `&&=` 논리 할당 연산자가 추가됐어요. 이를 사용하면 setter를 호출하지 않으면서도 간결하게 값을 할당하거나 변경할 수 있어요.

> 참고: [GitHub: tc39/proposal-logical-assignment](https://github.com/tc39/proposal-logical-assignment)

## `Numeric separators`

```javascript
const num = 1_000_000;
console.log(num); // 1000000

const float = 1_000_000.0_1;
console.log(float); // 1000000.01

const hex = 0xFF_FF_FF_FF;
console.log(hex); // 4294967295

const binary = 0b1010_1010;
console.log(binary); // 170

const octal = 0o7_7;
console.log(octal); // 63

const bigInt = 1_000_000n;
console.log(bigInt); // 1000000n


_1_000_000; // SyntaxError
1_000_000_; // SyntaxError
1__000_000; // SyntaxError
0x_FF_FF_FF_FF; // SyntaxError
```

숫자 리터럴에서 `_`로 구분해 가독성을 높일 수 있어요. 이는 숫자 값에 영향을 미치지 않아요. 단, 아래와 같은 경우에는 SyntaxError가 발생해요:

- `_`로 시작하거나 끝나는 경우 (e.g. `_1_000_000`, `1_000_000_`)
- `_`가 연속으로 사용된 경우 (e.g. `1__000_000`)

> 참고: [GitHub: tc39/proposal-numeric-separator](https://github.com/tc39/proposal-numeric-separator)
