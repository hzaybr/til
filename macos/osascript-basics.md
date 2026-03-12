# macOS Shell Scripts with osascript

_Date: 2026-03-12_

## Context

macOS 內建 `osascript`，可以從 shell script 呼叫 OSA (Open Scripting Architecture) 語言，實現桌面通知、偵測前景 app 等功能，不需要額外安裝工具。

## The Learning

### OSA 是什麼

OSA (Open Scripting Architecture) 是 macOS 讓腳本操控 app 的系統：

- **Apple Events** — macOS app 之間的溝通協定（像 HTTP 之於 web）
- **OSA** — 讓你用高階語言發送 Apple Events 的框架
- **osascript** — CLI 入口，選擇用哪個語言引擎來跑腳本

內建兩種語言，用 `-l` 切換：AppleScript（預設，1993）、JXA（JavaScript，2014）。

### 操控 App — tell application

每個 app 有一份 scripting dictionary，列出支援的操作：

```bash
osascript -e 'tell application "Finder" to empty the trash'
```

### Standard Additions — 不需要 app 的通用指令

不屬於任何 app，直接呼叫系統服務：

```bash
osascript -e 'display notification "body" with title "title"'           # 桌面通知
osascript -e 'display dialog "確定？" buttons {"取消", "OK"}'            # 對話框
osascript -e 'say "Hello" using "Mei-Jia"'                              # 語音朗讀
osascript -e 'choose file with prompt "選一個檔案"'                      # 檔案選擇器
```

### 安全傳遞變數

用 `on run argv` 避免引號問題：

```bash
osascript - "$MSG" <<'APPLESCRIPT'
on run argv
  display notification (item 1 of argv) with title "My Script"
end run
APPLESCRIPT
```

### 偵測前景 App / Window Title

```bash
# 前景 app 名稱
osascript -e 'tell application "System Events" to get name of first process whose frontmost is true'

# 特定 app 的 window title（需要輔助使用權限）
osascript -e 'tell application "System Events" to tell process "ghostty" to get title of front window'
```

### 生態定位

不算主流，但在 shell script 裡塞一行發通知或偵測前景 app 這種「黏合劑」用法很常見。Raycast、Alfred 底層也常用。現代替代方案有 Shortcuts（GUI 自動化）和 Hammerspoon（Lua 腳本，強項是視窗管理）。

## References

- [ss64 osascript reference](https://ss64.com/mac/osascript.html)
- [Open Scripting Architecture - Apple Developer](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptX/Concepts/osa.html)
- `man osascript`
