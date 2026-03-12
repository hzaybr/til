# Git Submodules + npm Workspaces

_Date: 2026-03-12_

## Context

當共用程式碼是獨立的 git repo（別的團隊也在用），但你想在主專案裡直接 `import` 它，可以用 git submodule 拉程式碼 + npm workspaces 處理模組解析。

## The Learning

### 兩者的分工

- **Git Submodule** — 管「程式碼從哪來」，追蹤外部 repo 的特定 commit
- **npm Workspaces** — 管「怎麼 import」，建立 symlink 讓模組解析生效

### 設定步驟

```bash
# 把外部 repo 加為 submodule
git submodule add git@github.com:org/common-lib.git packages/common-lib
```

根目錄 `package.json` 把 submodule 路徑納入 workspaces：

```json
{
  "private": true,
  "workspaces": ["packages/*"],
  "dependencies": {
    "@org/common-lib": "*"
  }
}
```

跑 `npm install` 後就能直接 `import { utils } from '@org/common-lib'`。

### 日常操作

**Clone 專案（第一次）：**

```bash
git clone --recurse-submodules git@github.com:org/my-app.git
# 忘了加的話：
git submodule update --init --recursive
```

**在 submodule 裡開發：**

```bash
cd packages/common-lib
git checkout main                # submodule 預設是 detached HEAD，先切 branch
# ... 修改、commit、push ...
cd ../..
git add packages/common-lib      # 記錄新的 commit 參照
git commit -m "chore: update common-lib"
```

**拉取 submodule 最新版：**

```bash
git submodule update --remote packages/common-lib
git add packages/common-lib
git commit -m "chore: update common-lib"
```

### 常見踩坑

- **Clone 後 submodule 是空的** — `git clone` 預設不拉 submodule，要加 `--recurse-submodules` 或事後跑 `git submodule update --init`
- **npm install 報錯找不到套件** — 通常是 submodule 還沒初始化，資料夾裡沒有 `package.json`
- **Detached HEAD** — submodule checkout 的是特定 commit 不是 branch，要先 `git checkout main` 才能開發
- **CI/CD 漏了 submodule** — GitHub Actions 要加 `submodules: recursive`

### 什麼時候不需要這個組合

- 所有程式碼都在同一個 repo → 純 workspaces 就夠
- Submodule 不是 Node.js 套件（如 proto 定義、靜態資源）→ 純 submodule 就夠
- 共用程式碼已發布到 npm → 直接 `npm install` 就好

## References

- [Git Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [npm Workspaces](https://docs.npmjs.com/cli/using-npm/workspaces)
