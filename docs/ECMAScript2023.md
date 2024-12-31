# ECMAScript 14 (ES14 or ECMAScript 2023)

## 목차

- [Array find from last](#array-find-from-last)
- [Hashbang Grammar](#hashbang-grammar)
- [Symbols as WeakMap keys](#symbols-as-weakmap-keys)
- [Change Array by Copy](#change-array-by-copy)

## Array find from last

```javascript
const arr = [1, 2, 3, 4, 5];
console.log(arr.find((value) => value > 2)); // 3
console.log(arr.findIndex((value) => value > 2)); // 2
```

`lastIndexOf` 메서드와 달리 `find`와 `findIndex` 메서드는 배열의 앞에서부터 조건에 맞는 요소를 찾아요. 이 때문에 배열의 끝에서부터 조건에 맞는 요소를 찾으려면 배열을 뒤집어야 했어요.

```javascript
const arr = [1, 2, 3, 4, 5];
console.log(arr.findLast((value) => value > 2)); // 5
console.log(arr.findLastIndex((value) => value > 2)); // 4
```

이러한 문제를 해결하기 위해 `findLast`, `findLastIndex` 메소드를 추가했어요. `findLast`, `findLastIndex`를 사용하면 배열의 끝에서부터 조건에 맞는 요소를 찾을 수 있어요.

> 참고: [GitHub - tc39/proposal-array-find-from-last](https://github.com/tc39/proposal-array-find-from-last)

## Hashbang Grammar

```javascript
#!/usr/bin/env node
console.log("Hello, World!");
```

사용자가 인터프리터를 지정할 필요 없이 스크립트를 실행할 수 있도록 해주는 해시뱅 혹은 쉬뱅(shebang)은 `#!`로 시작하는 특수한 문자열이에요. 이 문자열은 주로 셸 스크립트에서 사용되지만 자바스크립트에서도 사용할 수 있어요.

해당 기능은 CLI(Command Line Interface) 라이브러리에서 사용하는 것을 종종 볼 수 있어요.

> 참고: [GitHub - tc39/proposal-hashbang-grammar](https://github.com/tc39/proposal-hashbang)

## Symbols as WeakMap keys

```javascript
const key = Symbol("key");
const value = "value";

const map = new WeakMap();
map.set(key, value);

console.log(map.get(key)); // value
```

`WeakMap`의 키로 객체가 사용되는 이유는 객체가 고유한 성질을 가지고 있기 때문이에요. `Symbol`도 고유한 값을 가지기 때문에 `WeakMap`의 키로 사용할 수 있게 되었어요.

```javascript
const key = Symbol("key");
const value = { data: "value" };
const map = new WeakMap();
map.set(key, value);

const record = #{
  // 아직 제안 중인 레코드 및 튜플 리터럴 문법이에요.
  data: key,
};

console.log(map.get(record.data)); // { data: "value" }
```

`Symbol`은 객체와 달리 원시값이기 때문에, 객체 및 비원시값을 사용하지 못하는 상황에서 `WeakMap`에 키는 `Symbol` 그리고 값은 객체로 둔 후 `Symbol`을 사용하여 값을 가져오는 방법으로 해결할 수 있게 되었어요.

> 참고: [GitHub - tc39/proposal-symbols-as-weakmap-keys](https://github.com/tc39/proposal-symbols-as-weakmap-keys?tab=readme-ov-file)

## Change Array by Copy

```javascript
const arr = [5, 4, 3, 2, 1];

arr.sort((a, b) => a - b);

console.log(arr); // [1, 2, 3, 4, 5]
```

기존의 `sort` 메서드는 배열을 직접 변경하기 때문에 원본 배열이 변경되는 문제가 있었어요.

```javascript
const arr = [5, 4, 3, 2, 1];

const reversed = arr.toReversed();
const sorted = arr.toSorted((a, b) => a - b);
const spliced = arr.toSpliced(0, 2);
const withed = arr.with(0, 3);

console.log(arr); // [5, 4, 3, 2, 1]
console.log(reversed); // [1, 2, 3, 4, 5]
console.log(sorted); // [1, 2, 3, 4, 5]
console.log(spliced); // [3, 2, 1]
console.log(withed); // [3, 4, 3, 2, 1]
```

이러한 문제를 해결하기 위해, 배열을 변경하지 않고 복사 후 변경하는 `toReversed`, `toSorted`, `toSpliced`, `with` 메소드를 추가했어요. 각 메서드는 아래와 같은 기능을 제공해요.

- `toReversed()`: 배열을 뒤집어요.
- `toSorted(compareFn)`: 배열을 compareFn에 따라 정렬해요.
- `toSpliced(start, deleteCount, ...items)`: 배열의 start부터 deleteCount만큼 제거하고 items를 추가해요. (items가 없으면 제거만 해요)
- `with(index, value)`: index에 있는 값을 value로 변경해요.

> 참고: [GitHub - tc39/proposal-change-array-by-copy](https://github.com/tc39/proposal-change-array-by-copy)
