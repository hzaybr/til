# zPages: Google 內部的診斷頁面命名慣例

_Date: 2026-03-23_

## Context

在設定 OTel Collector 的 extensions 時看到 `zpages` 這個選項，好奇 "z" 是什麼意思、是不是某種正式標準。

## The Learning

zPages 不是正式的標準或 convention，而是源自 **Google 內部分散式追蹤系統（Dapper/Census）** 的命名慣例。做法是每個服務自己開一個 HTTP 端點，用 `z` 結尾的路徑名提供診斷頁面：

```
/debug/tracez     ← trace 的診斷
/debug/rpcz       ← RPC 呼叫統計
/debug/statsz     ← 內部指標
```

**z 就是這些診斷頁面的命名慣例**，所以統稱 zPages。

這個概念隨著 Google Census → OpenCensus → OpenTelemetry 的演變路徑被保留下來。在 OTel Collector 裡，它是一個 extension，打開後可以用瀏覽器直接查看 Collector 正在處理的 Span、錯誤、延遲分佈，不需要接任何後端。

```yaml
extensions:
  zpages:
    endpoint: 0.0.0.0:55679
```

生產環境通常不對外開放這個端口，只在內網或 debug 時使用。

## References

- [OpenTelemetry Collector - zPages Extension](https://github.com/open-telemetry/opentelemetry-collector/blob/main/extension/zpagesextension/README.md)
- [Google Dapper Paper](https://research.google/pubs/pub36356/)
