---
aliases:
date: 2019-05-06
update:
author:
language:
sourceurl: https://marcus116.blogspot.com/search/label/Serilog
tags:
---

# Links

[[NETCore] 結構化日誌 Serilog 初體驗](https://marcus116.blogspot.com/2019/05/netcore-serilog-intro.html)
[[NETCore] 結構化日誌 Serilog - 配置設定](https://marcus116.blogspot.com/2019/05/about-netcore-serilog-config-settings.html)
[[NETCore] 結構化日誌 Serilog - Events Types 和 Enrichment](https://marcus116.blogspot.com/2019/05/netcore-serilog-events-types-enrichers.html)
[[NETCore] Serilog 好幫手 - Serilog Analyzer](https://marcus116.blogspot.com/2019/05/netcore-serilog-serilog-analyzer.html)
[[NETCore] 在 ASP.NET Core Web API 中使用 Serilog](https://marcus116.blogspot.com/2019/05/serilog-in-netcore-aspnet-core-web-app.html)

---

# [NETCore] 結構化日誌 Serilog - Events Types 和 Enrichment

## 前言

前兩篇分別介紹了關於 Serilog 的基礎應用與設定，這篇就來針對事件類型 Event Type 與 介紹幾個常用的 Enricher，若有問題或是錯誤的地方歡迎提出來一起討論。

## Event Type

結構化日誌的好處是可以清楚的分辨 **每一次** 紀錄的事件，舉例來說下列簡單的代碼是透過 Serilog 寫入 log 到 Console 與 File 檔案，紀錄內容是 3 筆資料

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs\\log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();
 
Log.Information("Logging Start");
var total = 1;
for (var i = 0; i < 3; ++i)
{
    total *= i;
    Log.Information("Computed iteration {Counter}, total is {Total}", i, total);
}
```

上述的代碼輸出後如下

```console
[00:35:28 INF] Logging Start
[00:35:28 INF] Computed iteration 0, total is 0
[00:35:28 INF] Computed iteration 1, total is 0
[00:35:28 INF] Computed iteration 2, total is 0
```

上述的 Log 雖然可以記錄事情，但是假設資料量大或是 Log 數量很龐大的時候，找到同一事件 (群) 新增的 Log 紀錄是件很困難的事情，要解決這問題可以在 Log 加上一串識別字串  `hashCode`，方便查詢同樣事件使用。在 Serilog 解決方案是可以透過 Erinch 解決，首先新增一個類別 `EventTypeEnricher` 實作 `ILogEventEnricher` 介面的 `Enrich` 方法，內容為自訂 `hash` 的算法

```csharp
class EventTypeEnricher : ILogEventEnricher
{
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        var murmur = MurmurHash.Create32();
        var bytes = Encoding.UTF8.GetBytes(logEvent.MessageTemplate.Text);
        var hash = murmur.ComputeHash(bytes);
        var numericHash = BitConverter.ToUInt32(hash, 0);
        var eventId = propertyFactory.CreateProperty("EventType", numericHash);
        logEvent.AddPropertyIfAbsent(eventId);
    }
}
```

接著在建立 `ILogger` 時候透過 `Enrich.With` 將產生 `hash` 字串內容，並自定義 `outputTemplate` 模板樣式

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.With<EventTypeEnricher>()
    .WriteTo.Console(outputTemplate:
        "{Timestamp:HH:mm:ss} [{EventType:x8} {Level:u3}] {Message:lj}{NewLine}{Exception}")
    .CreateLogger();
```

接著重新執行一次，可以看到 log 多了自行定義的 hash Code

```console
01:28:08 [4dd86bd1 INF] Logging Start
01:28:08 [f20ba6e0 INF] Computed iteration 0, total is 0
01:28:08 [f20ba6e0 INF] Computed iteration 1, total is 0
01:28:08 [f20ba6e0 INF] Computed iteration 2, total is 0
```

成功透過自訂 Enrich 新增 hashCode 解決此問題

## More Enrichers

在前一篇 [結構化日誌 Serilog - 配置設定](https://marcus116.blogspot.com/2019/05/about-netcore-serilog-config-settings.html) 中介紹了基本的應用，在 Serilog 中提供很多實用的 Enrichers 方便開發者使用，像是積木的概念如果有遇到需要的就透過 nuget 套用即可十分方便，這裡在舉例自己在專案上有用到推薦的 Enrich 項目，舉例來說如果想要在每筆 Log 都記錄 MachineName、Env、Application 與 RequestID 等資訊時，目的是在 log 中紀錄下圖的資訊

![[AddedRequestId.png]]

在 Log 加入這些屬性的目的是為了在 ELK 搜尋時更容易分辨，如果當 Production 與 UAT 的 Log 都存放在同一台 ELK 時候，在查詢資料就可以透過 Environment 來分辨環境找到 Log 以及 Application，並自動在建立 Logger 時定義 MachineName 資訊，不用在 parser 時另外指定。

### 加入 Application 與 Environment

首先先從簡單的開始，分別在建立 Logger 定義 `Enrich.WithProperty` 屬性 `Applicaion` 與 `Environment` 並賦予相關值，在用 Json 格式輸出

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.WithProperty("Application", "MyApplication")
    .Enrich.WithProperty("Environment", ConfigurationManager.AppSettings["Environment"])
    .WriteTo.File(new CompactJsonFormatter(), "log.txt")
    .CreateLogger();
```

以上代碼輸出如下，很簡單的定義了輸出包含 `ApplicationName` 與 `Environment` 的 Log 資訊 

```console
{"@t":"2019-05-07T04:44:40.3369156Z","@mt":"Computed iteration {Counter}, total is {Total}","Counter":0,"Total":0,"Application":"e-Commerce","Environment":"Production"}
```

### 加入 MachineName

接著，再加上 `MachineName`，需要透過 nuget 安裝 `Serilog.Enrichers.Environment`  

```console
Install-Package Serilog.Enrichers.Environment -Version 2.1.3
```

並在建立 Logger 時加入 `Enricher` 指定 Machine 

```csharp
    .Enrich.WithMachineName()
```

輸出的 Log 部分會加上 `ServerName `

```console
{"@t":"2019-05-07T10:24:50.2274022Z","@mt":"Computed iteration {Counter}, total is {Total}","Counter":0,"Total":0,"Application":"MyApplication","Environment":"Production","MachineName":"ServerName"}
```

### 加入 RequestID

在 Serilog 中可以使用 [SerilogWeb.Classic](https://github.com/serilog-web/classic) 紀錄 `RequestId`，使用方式很簡單透過 nuget 下載後使用 `WithHttpRequestId` 方法，就可以很輕鬆的達到紀錄 `RequestId` 的需求。在 Serilog 中有另外提供一個好用的物件  `LogContext`  可以讓所有的紀錄事件加入特定的屬性，使用方式是 `new LoggerConfiguration` 加上 `Enrich.FromLogContext()` 擴充方法，再透過  `LogContext.PushProperty`  將特定的屬性以 key/value 方式加入紀錄事件中，代碼如下 `Enrich.WithMachineName()` 指定 `Machine` 

```csharp
var logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithProperty("Application", "MyApplication")
    .Enrich.WithProperty("Environment", "Production")
    .WriteTo.File(new CompactJsonFormatter(), "log.txt")
    .WriteTo.Console(outputTemplate:
        "{Timestamp:HH:mm:ss} [{EventType:x8} {Level:u3}] {Message:lj}{NewLine}{Exception}")
    .CreateLogger(); 
 
var requestId = Guid.NewGuid();
using (LogContext.PushProperty("RequestId", requestId, true))
{
    // Middleware Invoke
    logger.Information("This is a book, not a pencil");
}
```

在代碼中透過 `pushProperty` 方式將 `requestId` 注入到 `LogContent` 中，值為隨機產生 GUID 其結果輸出如下

```markdown
{"@t":"2019-05-07T14:56:47.2892546Z","@mt":"This is a book, not a pencil",**"RequestId":"0e9eaf11-544f-414d-8091-76bb7797edea"**,"Application":"MyApplication","Environment":"Production"}
```

可以看到 Log 順利紀錄了指定的 `RequestId` 資訊，如果在 ASP.NET Core 中止需要將產生 `RequestID` 的方式加到 MiddleWare 中的 `Invoke` 方法，即可順利地產生出 `RequestID` 資訊，這裡就不在細說
另外在 `LogContext` 中也提供指定 source Type 方法

```csharp
var valueLog = Log.ForContext<valueController>();
```

想要了解更細節的部份可以看作者所撰寫的 [Context and correlation – structured logging concepts in .NET](https://nblumhardt.com/2016/08/context-and-correlation-structured-logging-concepts-in-net-5/)，相信可以對細節與應用情境有更深入的了解。以上介紹了幾種簡單的應用，官網也有列出 enricher package 項目，如果有興趣的也可以到 github 查看使用方式及使用。

![[serilog_enricher.png]]

希望這篇的介紹對各位有幫助，Happy Coding :)

## 參考

[Structured logging concepts in .NET Series](https://nblumhardt.com/2016/06/structured-logging-concepts-in-net-series-1/)
[Serilog](https://github.com/serilog/serilog/wiki/Configuration-Basics)
