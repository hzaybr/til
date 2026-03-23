# Generator Function (`function*`)

_Date: 2026-03-20_

## Context

普通函數執行後一次跑完，Generator 函數可以暫停和恢復執行。

## The Learning

用 `function*` 宣告，用 `yield` 暫停，回傳一個 iterator：

```javascript
function* count() {
  yield 1
  yield 2
  yield 3
}

const gen = count()
gen.next() // { value: 1, done: false }
gen.next() // { value: 2, done: false }
gen.next() // { value: 3, done: false }
gen.next() // { value: undefined, done: true }
```

### 惰性無限序列

不會爆掉，因為每次 `yield` 就暫停：

```javascript
function* fibonacci() {
  let a = 0, b = 1
  while (true) {
    yield a
    ;[a, b] = [b, a + b]
  }
}
```

### 可以用 `for...of` 迭代

```javascript
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i
  }
}

for (const n of range(0, 5)) {
  console.log(n) // 0, 1, 2, 3, 4
}
```

### `yield*` 委派給另一個 generator

```javascript
function* inner() {
  yield 'a'
  yield 'b'
}

function* outer() {
  yield 1
  yield* inner()
  yield 2
}
// 產出: 1, 'a', 'b', 2
```

### 防禦性分號

行首是 `[`、`(`、`` ` `` 時，前面加 `;` 避免被解析器黏到前一行：

```javascript
yield a
;[a, b] = [b, a + b]
// 不加分號的話，可能被解析為 yield a[a, b] = ...
```
