---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 官方 Wiki

- [Available sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks)
- [Enrichment](https://github.com/serilog/serilog/wiki/Enrichment)

# Serilog Sink 與 Enricher 對照用途表

|類型|名稱|用途 / 功能|使用場合 / 建議搭配|
|---|---|---|---|
|**Sink**|Console|將日誌輸出到主控台|開發、測試環境即時查看|
|**Sink**|File|將日誌寫入檔案，可設定滾動策略|永久存檔、歷史紀錄分析|
|**Sink**|RollingFile|自動依日期或大小切分檔案|長期運行程式、維護檔案大小|
|**Sink**|Seq|發送日誌到 Seq 伺服器|微服務、企業集中化日誌分析|
|**Sink**|Elasticsearch / ELK|寫入 Elasticsearch，搭配 Kibana 可視化|大型系統集中管理、搜尋分析|
|**Sink**|SQL Server|寫入資料庫表格|方便結構化查詢、報表分析|
|**Sink**|MongoDB|寫入 NoSQL 資料庫|高度彈性儲存、非結構化日誌|
|**Sink**|Azure Table / Blob / EventHub|發送到 Azure 雲端儲存或事件中心|雲端服務集中管理|
|**Sink**|Email / SMTP|發送日誌通知|異常告警、重大事件通知|
|**Sink**|Debug|輸出到 Visual Studio 輸出視窗|開發階段調試|
|**Sink**|Graylog / GELF|發送到 Graylog 系統|集中化日誌管理、搜尋|

|類型|名稱|功能 / 加入的資訊|使用場合 / 建議搭配|
|---|---|---|---|
|**Enricher**|FromLogContext|從 LogContext 加入額外屬性|結合 LogContext 的上下文資訊|
|**Enricher**|ThreadId|加入執行緒 ID|多執行緒程式追蹤|
|**Enricher**|MachineName|加入主機名稱|分散式系統、多主機辨識|
|**Enricher**|EnvironmentUserName|加入當前使用者名稱|需要追蹤使用者操作的場景|
|**Enricher**|ExceptionDetails|加入例外完整資訊|錯誤分析、除錯|
|**Enricher**|ProcessId / ProcessName|加入進程資訊|背景服務、進程監控|
|**Enricher**|CorrelationId / ActivityId|加入分散式追蹤 ID|微服務鏈路追蹤、Request Flow 分析|

|類型|名稱|說明|
|---|---|---|
|**Event Type (Level)**|Verbose|非常詳細的診斷訊息|
|**Event Type (Level)**|Debug|開發階段使用資訊|
|**Event Type (Level)**|Information|一般運行資訊|
|**Event Type (Level)**|Warning|警告事件，可能存在風險|
|**Event Type (Level)**|Error|錯誤事件，需要處理|
|**Event Type (Level)**|Fatal|致命錯誤，可能導致程式崩潰|

## 常用搭配範例

- 開發環境：`Console Sink` + `Debug / Information Level` + `ThreadId / MachineName`
- 正式環境：`File / Seq / Elasticsearch Sink` + `Information / Warning / Error Level` + `MachineName / EnvironmentUserName`
- 微服務：`Seq / Elasticsearch / Graylog Sink` + `CorrelationId / ExceptionDetails`
- 警告通知：`SMTP Sink` + `Error / Fatal Level` + `ExceptionDetails`
