# npm Workspaces

_Date: 2026-03-12_

## Context

在一個 repo 裡管理多個 Node.js 套件時，手動管理依賴和跨套件引用很痛苦。npm workspaces 是 monorepo 的原生解法。

## The Learning

### 基本設定

根目錄 `package.json` 宣告哪些資料夾是 workspace：

```json
{
  "name": "my-project",
  "private": true,
  "workspaces": ["packages/*"]
}
```

`"private": true` 防止根目錄被意外 `npm publish`。

### 三個核心機制

**依賴提升（Hoisting）** — `npm install` 會把所有 workspace 的依賴統一裝到根目錄 `node_modules/`。三個套件都用 `lodash`，只裝一份。

**自動 Symlink** — workspace 之間互相引用時，npm 會在 `node_modules/` 建立 symlink 指向本地資料夾。改了共用套件的程式碼，其他套件立刻看到變化，不用 publish 也不用 `npm link`。

```json
// web-frontend/package.json
{
  "dependencies": {
    "@my/shared-utils": "*"
  }
}
```

版本寫 `"*"` 表示「用本地的 workspace 版本」。

**統一指令** — 在根目錄操作所有或特定 workspace：

```bash
npm install                                   # 安裝全部
npm run build --workspaces                    # 全部 build
npm run dev --workspace=packages/web          # 只跑特定 workspace
npm install cors --workspace=packages/api     # 只對特定 workspace 裝套件
```

### Glob 寫法

```json
"workspaces": ["packages/*"]
"workspaces": ["packages/*", "apps/*"]
"workspaces": ["packages/shared", "packages/web"]
```

### `type: "module"` 搭配

Workspaces 裡每個套件可以獨立設定 `"type": "module"` 或 `"commonjs"`。通常整個 monorepo 統一用 ESM：

```json
{
  "type": "module",
  "main": "index.js"
}
```

ESM 的 `import` 路徑必須包含副檔名（`'./utils.js'`），沒有 `__dirname`（用 `import.meta.url` 取代）。

### npm vs pnpm vs bun

- **npm** — `package.json` 的 `"workspaces"` 欄位，依賴提升到根目錄
- **pnpm** — 用 `pnpm-workspace.yaml`，依賴不提升（content-addressable store），引用寫 `"workspace:*"`
- **bun** — 和 npm 一樣用 `"workspaces"`，行為類似但速度快很多

## References

- [npm Workspaces](https://docs.npmjs.com/cli/using-npm/workspaces)
- [Node.js Modules: type field](https://nodejs.org/api/packages.html#type)
