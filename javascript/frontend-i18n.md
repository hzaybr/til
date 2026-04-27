# 前端 i18n 一般怎麼做

_Date: 2026-04-26_

## Context

review 一個 SvelteKit + Paraglide 的 i18n PR，順便整理前端 i18n 的通用心智模型——不限框架，不限函式庫。

## The Learning

任何前端 i18n 方案，本質上都在解決四件事：訊息儲存、查詢、locale 狀態、格式化。

### 1. 訊息儲存（Message catalog）

- 通常用 JSON / YAML / PO 檔，按 locale 分檔（`en.json`、`zh-TW.json`）
- Key 命名空間化：`settings.appearance.theme.title`，避免撞名
- 帶 placeholder（`Hello, {name}`）與複數形（ICU MessageFormat、Fluent）

PO 檔是 gettext 工具鏈常見的翻譯格式。gettext 是 GNU / Linux 生態很早就有的 i18n 系統：程式碼用 `_('Hello')` 標記要翻譯的字串，工具把它抽成 `msgid`，翻譯者填 `msgstr`。

```po
msgid "Hello, {name}"
msgstr "你好，{name}"
```

### 2. 訊息查詢（Translation function）

提供 `t(key, params)` 或 `m.key(params)` 給應用程式呼叫。兩條路線：

**Runtime lookup**（`i18next`、`vue-i18n`、`react-intl`）

- 字典物件 import 或 fetch，執行期查表
- 容易動態載入各 locale
- 缺點：如果直接 import 整包 catalog，bundle 會包含整包字典；key 是字串時，typo 編譯器抓不到（除非另外生 `d.ts` 或 typed helper）

`d.ts` 是 TypeScript declaration file。i18n 專案常用 codegen 讀 base locale 的翻譯檔，產生合法 key 的 union type：

```ts
type TranslationKey =
  | "settings.appearance.theme.title"
  | "nav.home";

function t(key: TranslationKey): string;
```

這樣 `t("settings.apperance.theme.title")` 的 typo 才會被 TypeScript 擋下來。更進階的 codegen 也會把 placeholder 參數一起產生型別。

**Compile-time / extraction tooling**（Paraglide、Lingui、`@formatjs/cli`）

- Paraglide 這類 compiler-based 方案會把每個訊息變成 typed JS function：`m.greeting({ name })`
- 編譯器產出型別與 placeholder 簽章，key 或參數寫錯立刻報錯
- Function-per-message 的輸出通常更 tree-shake 友善：沒用到的訊息不進 bundle
- FormatJS 則偏向 React / ICU MessageFormat 工具鏈：標記、抽取、編譯、驗證訊息，最後常由 `react-intl` 在 runtime format
- 缺點：build pipeline 多一步；要小心生成檔在 git / CI 的處理

### 3. Locale 狀態管理

一個全域、reactive 的「目前語系」。

- 偵測來源優先序：URL → cookie / localStorage → `navigator.language` → 預設 baseLocale
- 持久化：登入用戶存 server，匿名用戶存 localStorage / cookie
- 切換 locale 時要同步 `<html lang>`、`dir`（RTL 語言）、日期數字格式

### 4. 格式化（i18n ≠ 只翻字串）

數字、貨幣、日期、相對時間、列表 → 用 `Intl.*`（瀏覽器原生）。不要自己拼 `${date} ${time}`，會撞到 locale 的順序與分隔慣例。

```ts
// ❌ 硬寫
`${date.getMonth() + 1}/${date.getDate()}/${date.getFullYear()}`;

// ✅ Intl
new Intl.DateTimeFormat(locale, { dateStyle: "short" }).format(date);
```

複數和條件文案通常用 ICU MessageFormat、Fluent 或 i18n library 處理。ICU 是 International Components for Unicode；前端講 ICU 時，常是在講 ICU MessageFormat 這種可以描述 placeholder、plural、select 的訊息語法。

```txt
{count, plural,
  one {# item}
  other {# items}
}
```

### Reactivity 規則（最容易踩雷）

「翻譯後的字串」會在切換語系時失效，所以**不能**存進長生命週期的常數、map、closure。要存「資料 / key / enum」，render 時才轉文案。

```ts
// ❌ 切語言不會更新
const navItems = [{ path: "/settings", label: m.nav_settings() }];

// ✅ 在 reactive scope 裡計算
const navItems = $derived.by(() => [
  { path: "/settings", label: m.nav_settings() },
]);
```

這裡的問題是 `m.nav_settings()` 會立刻執行。假設當下 locale 是英文，`label` 就已經變成普通字串 `"Settings"`；之後切成中文，這個常數不會自動重算。

`map` 也一樣：

```ts
// ❌ 太早把翻譯結果存起來
const errorMessages = {
  INVALID_EMAIL: m.error_invalid_email(),
  PASSWORD_TOO_SHORT: m.error_password_too_short(),
};
```

`closure` 是 function 把舊的翻譯字串包住：

```ts
// ❌ label 在外面已經算好
const label = m.save();

function getButtonLabel() {
  return label;
}
```

比較好的規則是：長生命週期的地方存 key、enum 或原始資料；翻譯函式盡量靠近 render 時才呼叫。

同理，後端錯誤不要直接顯示英文字串，要在前端 map 成 i18n key。

### 常見地雷

1. **把 `t('Today')` 拿來當 key 用**（用英文字串去 switch）→ 哪天文案改寫就斷
2. **後端錯誤訊息直接 render** → 在前端 map 成 i18n key
3. **拼字串**：`"Hello " + name` → 用 placeholder
4. **日期格式硬寫** → `Intl.DateTimeFormat(locale)`
5. **沒處理 plural**：`${n} item(s)` → 用 ICU plural
6. **沒同步 `<html lang>`** → 螢幕閱讀器念錯語言
7. **首屏閃爍（FOUC of locale）**：先用 base locale 渲染再切 → 用 SSR / cookie / 同步初值解
8. **生成檔誤入 git**：codegen 路線要 `.gitignore`
9. **覆蓋面不夠**：placeholder、`title`、`aria-label`、錯誤訊息、toast、原生 `confirm/alert`、SEO meta、e-mail 全部要包

## References

- [Paraglide JS](https://inlang.com/m/gerre34r/library-inlang-paraglideJs)
- [MDN: Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)
- [ICU MessageFormat](https://unicode-org.github.io/icu/userguide/format_parse/messages/)
- [Project Fluent](https://projectfluent.org/)
