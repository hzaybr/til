# TypeScript Generic 裡的 Rest Parameters 與 never

_Date: 2026-04-27_

## Context

看到這種型別：

```ts
F extends (...args: never[]) => string
```

一開始容易卡在三件事：

- `F extends ...` 是什麼
- `...args` 是什麼
- `never[]` 為什麼可以放在 function 參數裡

## The Learning

`F extends (...args: never[]) => string` 的意思是：

```txt
F 必須是一個 function，而且它的回傳值必須是 string。
```

`F extends ...` 是 generic constraint，限制 `F` 不能是任意型別，必須符合右邊的形狀。

```ts
type StringFunction<F extends (...args: never[]) => string> = F;
```

這些都符合：

```ts
type A = StringFunction<() => string>;
type B = StringFunction<(name: string) => string>;
type C = StringFunction<(id: number, active: boolean) => string>;
```

這個不符合：

```ts
type D = StringFunction<() => number>;
```

因為回傳值不是 `string`。

### Rest parameters

`...args` 是 JavaScript / TypeScript 的 rest parameters 語法。

```ts
function sum(...numbers: number[]) {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3); // 6
```

在 function 裡，`numbers` 會是一個 array：

```ts
[1, 2, 3];
```

所以：

```ts
(...args: string[]) => string
```

意思是：

```txt
這是一個 function。
它可以接一串 string 參數。
它最後會回傳 string。
```

Rest parameters 跟「直接傳 array」不同：

```ts
function withRest(...numbers: number[]) {}
function withArray(numbers: number[]) {}

withRest(1, 2, 3);
withArray([1, 2, 3]);
```

### never 是什麼

`never` 是 TypeScript 裡的「不可能存在的值」。

```ts
const value: never = "hello"; // error
```

常見在永遠不會正常回傳的 function：

```ts
function fail(message: string): never {
  throw new Error(message);
}
```

所以 `never[]` 表面上是「元素型別是 `never` 的 array」。

但在這種 constraint 裡：

```ts
F extends (...args: never[]) => string
```

重點通常不是「這個 function 真的要吃 `never` 參數」，而是：

```txt
我只想限制 F 是某個會回傳 string 的 function。
我不打算在這裡呼叫它，所以不關心它實際吃什麼參數。
```

如果真的要呼叫這個 function，就應該把參數也建模出來：

```ts
function callWithArgs<Args extends unknown[]>(
  fn: (...args: Args) => string,
  ...args: Args
) {
  return fn(...args);
}

callWithArgs((name: string) => `Hello ${name}`, "Ada");
```

### any、unknown、never

`any` 是關掉型別檢查：

```ts
let value: any = "hello";

value.toUpperCase(); // ok
value.not.exists(); // also ok
```

`unknown` 是「我不知道型別，但使用前要先檢查」：

```ts
let value: unknown = "hello";

value.toUpperCase(); // error

if (typeof value === "string") {
  value.toUpperCase(); // ok
}
```

`never` 是「不可能有這個值」：

```ts
let value: never;

value = "hello"; // error
value = 123; // error
```

簡短記法：

- `any`：什麼都可以放，怎麼用都可以，最不安全
- `unknown`：什麼都可以放，但用之前要檢查，安全
- `never`：什麼都不能放，代表不可能

## Rule of Thumb

看到：

```ts
F extends (...args: never[]) => string
```

可以先讀成：

```txt
F 是某個回傳 string 的 function。
這裡不關心它的參數長什麼樣。
```

