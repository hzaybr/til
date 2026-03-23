# Callback

_Date: 2026-03-20_

## Context

Callback 是 JavaScript 最核心的模式之一，到處都在用。

## The Learning

Callback 就是「你寫的函數，交給別人呼叫」。字面意思是「回頭呼叫」（call back）。

```javascript
function doTwice(callback) {
  callback()
  callback()
}

doTwice(() => console.log('hi'))
// hi
// hi
```

### Array 方法

```javascript
const nums = [1, 2, 3, 4, 5]

nums.map(n => n * 2)              // [2, 4, 6, 8, 10] — 轉換每個元素
nums.filter(n => n > 3)           // [4, 5] — 篩選
nums.find(n => n > 3)             // 4 — 找第一個符合的
nums.some(n => n > 3)             // true — 有任何一個符合嗎
nums.every(n => n > 3)            // false — 全部都符合嗎
nums.forEach(n => console.log(n)) // 純遍歷，沒回傳值
nums.reduce((sum, n) => sum + n, 0) // 15 — 累積成一個值
nums.sort((a, b) => a - b)        // 排序
```

### 計時器

```javascript
setTimeout(() => { ... }, 1000)   // 1 秒後執行一次
setInterval(() => { ... }, 1000)  // 每 1 秒執行一次
```

### 事件

```javascript
button.addEventListener('click', () => { ... })
input.addEventListener('change', (event) => { ... })
```

### Promise

```javascript
fetch('/api/users')
  .then(res => res.json())
  .catch(err => console.error(err))
```

### Web 框架的 handler 也是 callback

```javascript
// Express
app.get('/users', (req, res) => { ... })

// Elysia
app.get('/users', ({ query }) => { ... })
```

你寫好函數交給框架，有人打這個路由時，框架幫你呼叫。
