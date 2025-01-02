# ECMAScript 15 (ES15 or ECMAScript 2024)

## 목차

- [Well-Formed Unicode Strings](#well-formed-unicode-strings)
- [`Atomics.waitAsync`](#atomicswaitasync)
- [RegExp v flag with set notation + properties of strings](#regexp-v-flag-with-set-notation--properties-of-strings)
- [Resizable and growable ArrayBuffers](#resizable-and-growable-arraybuffers)
- [Array Grouping](#array-grouping)
- [`Promise.withResolvers`](#promisewithresolvers)
- [ArrayBuffer transfer](#arraybuffer-transfer)

## Well-Formed Unicode Strings

ECMAScript 문자열은 16비트 부호 없는 정수 시퀀스이지만, UTF-16 기준으로 잘못된 코드 포인트를 포함할 수 있어요. 이러한 문제는 유효한 문자열이 필요한 WebAssembly와 같은 환경에서 특히 문제가 될 수 있어요. 이러한 문제를 해결하기 위해 `isWellFormed`와 `toWellFormed` 메서드가 추가됐어요.

```javascript
const badString = "hello\uD800";
const goodString = "hello";

console.log(badString.isWellFormed()); // false
console.log(goodString.isWellFormed()); // true

console.log(badString.toWellFormed()); // hello�
console.log(goodString.toWellFormed()); // hello
```

`isWellFormed` 메서드는 문자열이 유효한지 여부를 확인하고, `toWellFormed` 메서드는 유효하지 않은 코드 포인트를 대체 문자(`U+FFFD` or `�`)로 대체해요.

> 참고: [GitHub - tc39/proposal-well-formed-unicode-strings](https://github.com/tc39/proposal-is-usv-string)

## `Atomics.waitAsync`

`Atomics.wait` 메서드는 호출한 스레드를 블록하고, 다른 스레드가 해당 메모리 위치에 값을 쓸 때까지 기다리는 메서드에요. 따라서 블록을 허용하지 않는 환경에서는 사용할 수 없어요. 이러한 문제를 해결하기 위해 `Atomics.waitAsync` 메서드가 추가됐어요.

```javascript
const buffer = new SharedArrayBuffer(4);
const view = new Int32Array(buffer);

console.log("Waiting...");
Atomics.waitAsync(view, 0, 0).then((result) => {
  console.log(result); // "ok" or "timed-out"
});

setTimeout(() => {
  console.log("Notifying...");
  Atomics.notify(view, 0, 1);
}, 1000);

console.log("Running...");

// Waiting...
// Running...
// Notifying...
```

`Atomics.waitAsync` 메서드는 `Atomics.wait`와 동일한 기능을 제공하지만, `Promise` 기반 비동기 방식으로 동작해요. 이 덕분에 블록을 허용하지 않는 환경에서도 사용할 수 있게 되었어요.

> 참고: [GitHub - tc39/proposal-atomics-wait-async](https://github.com/tc39/proposal-atomics-wait-async/blob/master/PROPOSAL.md)

## RegExp v flag with set notation + properties of strings

UTS(유니코드 기술 표준)에서는 집합 연산자 지원을 추천하고 있지만 ECMAScript에서는 지원하지 않아요. 해당 연산자를 도입한다면 복잡한 정규식을 유니코드 속성 문자와 함께 사용하여 더 간단하게 작성할 수 있게 되어요.

다만, 후방 호환성을 고려하여 `v` 플래그를 사용하여 활성화해야 해당 기능을 사용할 수 있어요.

```javascript
const regex = /[\p{Decimal_Number}--[0-9]]/v;
console.log(regex.test("٢")); // true (아랍어 숫자)
console.log(regex.test("9")); // false (ASCII 숫자)
```

위 예시는 0부터 9까지의 숫자를 제외한 모든 숫자를 찾는 정규식이에요. 즉, `--` 연산자는 차집합 기능을 제공해요. 추가로 합집합(`++`), 교집합(`&&`) 연산자도 지원돼요.

> 참고: [GitHub - tc39/proposal-regexp-set-notation](https://github.com/tc39/proposal-regexp-v-flag?tab=readme-ov-file)

## Resizable and growable ArrayBuffers

기존의 `ArrayBuffer`는 고정 크기로 변경할 수 없어요. 이 때문에 크기를 변경하려면 새로운 `ArrayBuffer`를 생성하고 기존 데이터를 복사해야 했어요. 이러한 문제를 해결하기 위해 `ArrayBuffer`를 크기를 변경할 수 있게 되었어요.

```javascript
const buffer = new ArrayBuffer(4, { maxByteLength: 8 });
const view = new Int32Array(buffer);

console.log(view.byteLength); // 4

buffer.resize(8);

console.log(view.byteLength); // 8

buffer.resize(16); // 오류 발생: 최대 크기 초과
```

위처럼 `ArrayBuffer`의 생성자에 `maxByteLength` 옵션을 사용하여 최대 크기를 지정한 후 `resize` 메서드를 사용하여 `ArrayBuffer`의 크기를 변경할 수 있게 되었어요. 이때 in-place로 크기를 변경하기 때문에 메모리를 효율적으로 사용할 수 있어요. 또한, 위 예시처럼 `TypedArray`는 `ArrayBuffer`의 변경된 크기를 자동으로 반영해요.

참고로 `SharedArrayBuffer` 보안 문제로 인해 크기를 키우는 것만 허용돼요.

> 참고: [GitHub - tc39/proposal-resizable-arraybuffer](https://github.com/tc39/proposal-resizablearraybuffer)

## Array Grouping

```javascript
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const grouped = Object.groupBy(arr, (value) =>
  value % 2 === 0 ? "even" : "odd"
);

console.log(grouped);
// {
//   odd: [1, 3, 5, 7, 9],
//   even: [2, 4, 6, 8, 10],
// }
```

배열을 그룹화하는 것은 매우 일반적인 작업이지만, 기존에는 직접 구현해야 했어요. 이러한 유즈 케이스를 위해 `Object.groupBy`와 `Map.groupBy` 메서드가 추가됐어요.

먼저 `Object.groupBy` 메서드는 배열을 그룹화하여 프로토타입이 `null`인 객체를 반환해요. 그리고 `Map.groupBy` 메서드는 일반적인 `Map`을 반환해요.

> 참고: [GitHub - tc39/proposal-array-grouping](https://github.com/tc39/proposal-array-grouping)

## `Promise.withResolvers`

```javascript
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});

resolve("ok");
reject("error");
```

기존에는 외부에서 `resolve`와 `reject` 메서드를 사용하려면 executor 함수를 사용해야 했어요. 이러한 작업은 너무 번거롭기 때문에 명시적으로 `resolve`와 `reject` 메서드를 반환하는 `Promise.withResolvers` 메서드가 추가됐어요.

```javascript
const { promise, resolve, reject } = Promise.withResolvers();
```

위처럼 `Promise.withResolvers` 메서드를 사용하면 `promise`, `resolve`, `reject` 메서드를 명시적으로 반환받을 수 있어요.

> 참고: [GitHub - tc39/proposal-promise-with-resolvers](https://github.com/tc39/proposal-promise-with-resolvers)

## ArrayBuffer transfer

```javascript
const buffer = new ArrayBuffer(4);
const owned = buffer.transfer();

console.log(buffer.detached); // true
console.log(owned.detached); // false
console.log(owned.byteLength); // 4
```

`ArrayBuffer`를 파일 시스템이나 워커 등으로 안전하게 전송하기 위해서는 소유권 이전이 필요해요. 이러한 작업에서 `ArrayBuffer.prototype.slice` 메서드를 사용하면 모든 데이터를 복사해야 해서 비효율적이었어요. 이러한 문제를 해결하기 위해 `ArrayBuffer.prototype.transfer` 메서드가 추가됐어요. `transfer` 메서드는 zero-copy 이동 또는 `realloc`을 통해 구현될 것이기 때문에 매우 효율적이에요.

> 참고: [GitHub - tc39/proposal-arraybuffer-transfer](https://github.com/tc39/proposal-arraybuffer-transfer)
