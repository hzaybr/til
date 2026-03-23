# Vite 運作原理與 Plugin

## Vite 是什麼

不只是「打包工具」，更精確的說是 **dev server + build tool**：

- **Dev 模式**（`vite dev`）：不打包，靠瀏覽器原生 ES Module 按需 serve
- **Build 模式**（`vite build`）：用 Rollup 打包，做 tree-shaking、code splitting、minification

---

## Dev 模式怎麼運作

### 瀏覽器原生 ESM 的流程

1. HTML 載入 `<script type="module" src="/src/main.ts">`
2. 瀏覽器發 `GET /src/main.ts` 給 Vite dev server
3. Vite 即時用 esbuild 把 TS → JS，回傳給瀏覽器
4. 瀏覽器解析 JS 裡的 `import`，再發請求拿下一個檔案
5. 每個檔案都是獨立的 HTTP 請求，獨立轉譯

在 DevTools Network tab 可以看到每個 `.ts`、`.svelte` 檔案都是獨立的請求，不是一個大 bundle。

### HMR（Hot Module Replacement）

改了一個檔案 → Vite 只重新編譯那一個 → WebSocket 通知瀏覽器 → 瀏覽器只替換該 module。

### `node_modules` 的 pre-bundling

`node_modules` 裡的套件可能是 CJS 格式或散成幾百個小檔案。Vite 啟動時用 esbuild 預先打包成少數幾個 ESM 檔案，快取在 `node_modules/.vite/`。

只有 `src/` 原始碼才是真正「不打包、按需 serve」。

`optimizeDeps.exclude` 可以排除特定依賴不做 pre-bundling（例如 workspace 裡的 local package）。

### 為什麼 production 不能不打包

使用者網路有延遲，幾百個小檔案 = 幾百個 HTTP round trip。加上沒有 tree-shaking 和 minification，所以 `vite build` 仍用 Rollup 打包。

---

## Vite Plugin

### 本質

一個回傳物件的函式，物件裡包含各種 lifecycle hook：

```ts
function myPlugin(): Plugin {
  return {
    name: 'my-plugin',
    transform(code, id) {
      if (id.endsWith('.css')) {
        return code.replace('red', 'blue')
      }
    }
  }
}
```

### 主要 Hook

- **resolve** — 解析 import 路徑（`import 'foo'` → 實際檔案位置）
- **load** — 讀取檔案內容
- **transform** — 轉換檔案內容（最常用）
- 其他：設定 dev server、注入 HTML、build 前後處理等

### 常見範例

```ts
// vite.config.ts
export default defineConfig({
  plugins: [
    sveltekit(),     // .svelte → Svelte compiler → JS + CSS
    tailwindcss()    // 掃描 class → 產生 utility CSS
  ]
})
```

### 為什麼 Tailwind v4 從 PostCSS plugin 改成 Vite plugin

- PostCSS 只能做 CSS → CSS 轉換，要另外設定 `content` 路徑掃描 class
- Vite plugin 直接整合進 module graph，自動知道哪些檔案改了
- 不需要手動設定 `content`
- HMR 更精確（只重新產生用到的 CSS）

### Plugin 相容性

Vite plugin 相容 Rollup plugin 格式（build 階段就是用 Rollup），很多 Rollup 社群 plugin 可直接使用。
