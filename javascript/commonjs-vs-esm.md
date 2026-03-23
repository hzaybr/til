# CommonJS vs ES Module

## 背景

JavaScript 原本沒有模組系統。2009 年以前，瀏覽器裡所有 `<script>` 共用一個全域 scope，變數互相污染。

---

## CommonJS（CJS）

2009 年 Node.js 誕生時自己發明的模組系統。**不是 JS 語言標準**。

```js
// 匯出
module.exports = { formatDate }

// 匯入
const { formatDate } = require('./utils')
```

特性：
- `require()` 是普通函式，runtime 執行
- 可以放在 `if` 裡條件載入（動態）
- 同步讀檔 → 適合 Node.js（檔案在本地磁碟）
- 瀏覽器**不支援**

---

## ES Module（ESM）

2015 年 ES6 定義的語言標準模組系統。

```js
// 匯出
export function formatDate() {}
export default App

// 匯入
import { formatDate } from './utils'
import App from './App'
```

特性：
- `import`/`export` 是語法關鍵字，編譯期靜態分析
- 必須寫在最頂層，不能放在 `if` 裡
- 因為靜態 → 工具可做 **tree-shaking**（移除沒用到的 export）
- 瀏覽器**原生支援**（`<script type="module">`）

---

## 為什麼 CJS 還到處看得到

npm 上大量套件是 2015 年前寫的，或為了相容性繼續提供 CJS。很多套件同時發布兩種格式：

```json
{
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs"
}
```

---

## 跟 Vite 的關係

Vite dev 靠瀏覽器原生 ESM 運作，但 `node_modules` 裡很多套件只提供 CJS。所以 Vite 啟動時的 pre-bundling 會把 CJS 轉成 ESM，讓瀏覽器能用 `import` 載入。
