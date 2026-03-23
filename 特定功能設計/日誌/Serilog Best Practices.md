---
aliases:
date: 2020-04-13
update:
author: Ben Foster
language:
sourceurl: https://benfoster.io/blog/serilog-best-practices/
tags:
  - CSharp
  - 日誌
  - Serilog
---

# Serilog Best Practices  Serilog 最佳實踐

溪源 More [翻譯](https://www.cnblogs.com/xiyuanMore/p/15106494.html)

[Serilog](https://serilog.net/) is a structured logging library for Microsoft .NET and has become the preferred logging library for .NET at [Checkout.com.](http://checkout.com/). It supports a variety of logging destinations, referred to as [Sinks](https://github.com/serilog?q=sinks&type=&language=), from standard console and files based sinks to logging services such as Datadog.
[Serilog](https://serilog.net/) 是 Microsoft .NET 的結構化日誌庫，已成為 [Checkout.com](http://checkout.com/) 首選的 .NET 日誌庫。它支援各種日誌目標（稱為 [接收器）](https://github.com/serilog?q=sinks&type=&language=) ，從標準的控制台和檔案接收器到 Datadog 等日誌服務。

This guide started off as an article in our engineering handbook and after receiving positive feedback internally, I decided to release it on my blog.
本指南最初是我們工程手冊中的一篇文章，在收到內部的正面回饋後，我決定將其發佈到我的部落格上。

# Contents  內容

- Standard log properties 標準日誌屬性
- Logging fundamentals 日誌記錄基礎知識
    - Log key events 記錄關鍵事件
    - Choose an appropriate logging level 選擇合適的日誌級別
    - Timing operations 計時操作
    - Source Context 來源上下文
    - HTTP Logging HTTP 記錄
    - Avoid excessive production logging 避免過度生產日誌記錄
    - Create log specific objects when destructuring 解構時建立日誌特定對象
- Enrich your logs 豐富您的日誌
    - Standard Serilog enrichers 標準 Serilog 濃縮器
    - Enrich from global properties 從全域屬性中豐富
- Correlating logs 相關日誌
- Message Templating 訊息模板
    - Message Template Recommendations 郵件範本推薦
- Log and Diagnostic Context 日誌和診斷上下文
    - Log Context 日誌上下文
    - Diagnostic Context 診斷背景
- Configuration 配置
- Production logging 生產紀錄
    - When logs become more than logs 當日誌不再只是日誌時
- Other tools and utilities 其他工具和實用程式
    - Use Seq locally 本地使用 Seq
    - Event Type Enricher 事件類型增強器
    - Property Bag Enricher 屬性袋增強器
    - Request Log Enricher 請求日誌增強器

# Standard log properties  標準日誌屬性

Standardising log event properties enables you to get the most out of log search and analysis tools. Use the following Properties where applicable:
規範日誌事件屬性有助於您充分利用日誌搜尋和分析工具。請在適用情況下使用下列屬性：

|`ApplicationName`|The name of the application generating the log events  <br>產生日誌事件的應用程式名稱|
|---|---|
|`ClientIP`|The IP address of the client from which a request originated  <br>發起請求的客戶端的 IP 位址|
|`CorrelationId`|An ID that can be used to trace the request across multiple application boundaries  <br>可用於跨多個應用程式邊界追蹤請求的 ID|
|`Elapsed`|The time in milliseconds an operation took to complete  <br>完成一項操作所花費的時間（以毫秒為單位）|
|`EventType`|A hash of the message template used to determine the message type  <br>用於確定訊息類型的訊息模板哈希值|
|`MachineName`|The name of the machine on which the application is running  <br>應用程式運行所在機器的名稱|
|`Outcome`|The outcome of an operation  <br>手術結果|
|`RequestMethod`|The HTTP request method e.g. `POST`  <br>HTTP 請求方法，例如 `POST`|
|`RequestPath`|The HTTP request path  HTTP 請求路徑|
|`SourceContext`|The name of component/class from which the log originated  <br>日誌來源的元件/類別的名稱|
|`StatusCode`|The HTTP response status code  <br>HTTP 回應狀態碼|
|`UserAgent`|The HTTP user agent  HTTP 使用者代理|
|`Version`|The version of the running application  <br>正在運行的應用程式版本|

Many of the above attributes come from Serilog’s own extensions, for example [Serilog Timings](https://github.com/nblumhardt/serilog-timings) (used to time operations) and [Serilog request logging](https://github.com/serilog/serilog-aspnetcore).
上述許多屬性都來自 Serilog 本身的擴展，例如 [Serilog Timings](https://github.com/nblumhardt/serilog-timings) （用於計時操作）和 [Serilog 請求日誌記錄](https://github.com/serilog/serilog-aspnetcore) 。

# Logging fundamentals  日誌記錄基礎知識

## Log key events  記錄關鍵事件

In general, log key events that provide insight into your application and user behaviour, for example:
一般來說，應記錄能夠幫助您深入了解應用程式和使用者行為的關鍵事件，例如：

- Major branching points in your code
    程式碼中的主要分支點
- When errors or unexpected values are encountered
    當遇到錯誤或意外值時
- Any IO or resource intensive operations
    任何 I/O 或資源密集型操作
- Significant domain events
    重大領域事件
- Request failures and retries
    請求失敗和重試
- Beginning and end of time-consuming batch operations
    耗時批次操作的開始和結束

## Choose an appropriate logging level

選擇合適的日誌級別

Be generous with your logging but be strict with your logging levels. In almost all cases the level of your logs should be `Debug`. Use `Information` for log events that would be needed in production to determine the running state or correctness of your application and `Warning` or `Error` for unexpected events, such as Exceptions.
日誌記錄要盡可能多，但日誌等級要嚴格控制。幾乎所有情況下，日誌等級都應該設定為 `Debug` 。對於生產環境中用於判斷應用程式運作狀態或正確性的事件，請使用 `Information` 進行記錄；對於意外事件（例如異常），請使用 `Warning` 或 `Error` 進行記錄。

Note that the `Error` level should be reserved for events that you intend to act on. If something becomes normal application behaviour (e.g. a request failing input validation) you should downgrade the log level to reduce log “noise”.
請注意， `Error` 等級應僅用於您需要採取相應措施的事件。如果某些情況成為應用程式的正常行為（例如，請求輸入驗證失敗），則應降低日誌等級以減少日誌「雜訊」。

## Timing operations  計時操作

Log every resource-intensive operation such as IO, in your applications, alongside your metrics code. This is very useful when running applications locally to see application bottlenecks or what is eating into response time. The [Serilog Timings library](https://github.com/nblumhardt/serilog-timings) provides a convenient way to do this:
在應用程式的指標代碼旁邊，記錄所有資源密集型操作（例如 I/O）。這在本地運行應用程式時非常有用，可以幫助您發現應用程式瓶頸或影響回應時間的原因。 Serilog [Timings 函式庫](https://github.com/nblumhardt/serilog-timings) 提供了一種便捷的方式來實現這一點：

```csharp
using (_logger.TimeDebug("Sending notification to Slack channel {Channel} with {WebhookUrl}", _slackOptions.Channel, _slackOptions.WebhookUrl))
using (_metrics.TimeIO("http", "slack", "send_message"))
{

}
```

## Source Context  來源上下文

The `SourceContext` property is used to track the source of the log event, typically the C# class from where the logger is being used. It’s common to inject Serilog’s `ILogger` into classes using Dependency Injection. To ensure the `SourceContext` is set correctly, use the `ForContext` extension:
`SourceContext` 屬性用於追蹤日誌事件的來源，通常是呼叫日誌記錄器的 C# 類別。通常使用依賴注入將 Serilog 的 `ILogger` 注入到類別中。為了確保正確設定 `SourceContext` ，請使用 `ForContext` 擴充：

```csharp
public TheThing(ILogger logger)
{
    _logger = logger?.ForContext<TheThing>() ?? throw new ArgumentNullException(nameof(_logger));
}
```

## HTTP Logging  HTTP 記錄

Use the [Serilog request logging middleware](https://github.com/serilog/serilog-aspnetcore) to log HTTP requests. This automatically includes many of the HTTP attributes listed above and produces the following log message:
使用 [Serilog 請求日誌中間件](https://github.com/serilog/serilog-aspnetcore) 來記錄 HTTP 請求。這會自動包含上面列出的許多 HTTP 屬性，並產生以下日誌訊息：

```text
HTTP POST /payments responded 201 in 1348.6188 ms
```

Add the following to your application startup to add the middleware:
將以下內容新增至應用程式啟動項目中，以新增中間件：

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseHealthAndMetricsMiddleware();
    app.UseSerilogRequestLogging();
    app.UseAuthentication();
    app.UseMvc();
}
```

Note that the Serilog middleware is added _after_ the health and metrics middleware. This is to avoid generating logs every time your health check endpoints are hit by AWS load balancers.
請注意，Serilog 中間件是在運行狀況和指標中間件 _ 之後 _ 添加的。這樣做是為了避免每次 AWS 負載平衡器存取執行狀況檢查端點時都會產生日誌。

### Logging HTTP Resources  記錄 HTTP 資源

The Serilog middleware by default logs the request path. If you do have a need to see all requests for a particular endpoint in your application you may have a challenge if the path contains dynamic parameters such as identifiers.
Serilog 中間件預設會記錄請求路徑。如果您需要查看應用程式中特定端點的所有請求，而該路徑包含動態參數（例如識別碼），則可能會遇到一些挑戰。

To get around this, log the resource name which by convention in our applications is the `Name` attribute given to the corresponding route. This is retrieved like so:
為了解決這個問題，我們需要記錄資源名稱，按照我們應用程式的慣例，資源名稱是賦予對應路由的 `Name` 屬性。取得方式如下：

```csharp
public static string GetMetricsCurrentResourceName(this HttpContext httpContext)
{
    if (httpContext == null)
        throw new ArgumentNullException(nameof(httpContext));

    Endpoint endpoint = httpContext.Features.Get<IEndpointFeature>()?.Endpoint;

#if NETCOREAPP3_1
    return endpoint?.Metadata.GetMetadata<EndpointNameMetadata>()?.EndpointName;
#else
    return endpoint?.Metadata.GetMetadata<IRouteValuesAddressMetadata>()?.RouteName;
#endif
}
```

## Avoid excessive production logging 避免過度生產日誌記錄

Logs help to reason about your application and diagnose issues. You should review your production logs regularly to ensure they provide value. It’s all too easy to add additional logging during the investigation of complex issues and not clean it up afterwards.
日誌有助於分析應用程式並診斷問題。您應該定期查看生產日誌，確保它們發揮價值。在調查複雜問題時，很容易添加額外的日誌，但事後卻忘記清理。

Bad actors logging excessively can also impact the health of logging systems, which are often shared between teams. It’s helpful to establish standards on what events should be logged at INFO level. Remember, you can always dial up your logging to DEBUG if you need.
惡意行為者過度記錄日誌也會影響日誌系統的運作狀況，而這些系統通常是團隊之間共享的。因此，制定關於哪​​些事件應該記錄為 INFO 等級的日誌的標準很有幫助。請記住，如有需要，您可以隨時將日誌等級提升到 DEBUG。

Finally, rather than adding additional log events to log information at different points in the request lifecycle, favour using Serilog’s diagnostic context feature (discussed below) to collapse contextual information into a single completion log.
最後，與其在請求生命週期的不同階段添加額外的日誌事件來記錄訊息，不如使用 Serilog 的診斷上下文功能（如下所述）將上下文資訊合併到單一完成日誌中。

## Create log specific objects when destructuring 解構時建立日誌特定對象

Serilog supports [_destructuring_](https://github.com/serilog/serilog/wiki/Structured-Data#preserving-object-structure), allowing complex objects to be passed as parameters in your logs.
Serilog 支援 [_解構賦值_](https://github.com/serilog/serilog/wiki/Structured-Data#preserving-object-structure) ，允許將複雜物件作為參數傳遞到日誌中。

**This feature should be used with caution**. It’s not uncommon for sensitive information to be included in logs because an object that is defined outside of the scope of the logging code is extended.
**應謹慎使用此功能** 。由於擴展了在日誌記錄程式碼作用域之外定義的對象，敏感資訊被包含在日誌中的情況並不少見。

When destructuring, be explicit about the data that is logged by creating log specific objects (anonymous objects work great):
在進行解構時，要明確指定要記錄的數據，方法是建立日誌專用物件（匿名物件效果很好）：

For example, instead of:  例如，而不是：

```csharp
_logger.Information("Processing HTTP request with data {@Request}", httpContext.Request);
```

use:  使用：

```csharp
var requestData = new { Path = httpContext.Request.Uri.AbsolutePath, User = httpContext.Request.User.GetUserId()  };
_logger.Information("Processing HTTP request with data {@Request}", requestData);
```

# Enrich your logs  豐富您的日誌

Pushing additional information into your logs can help provide additional context about a specific event.
在日誌中添加更多資訊可以幫助提供有關特定事件的更多背景資訊。

## Standard Serilog enrichers 標準 Serilog 濃縮器

You can use enrichers to enrich all log events generated by your application. We recommend use of the following Serilog enrichers:
您可以使用 Serilog 增強器來豐富應用程式產生的所有日誌事件。我們建議使用以下 Serilog 增強器：

- Log Context enricher - Built in to Serilog, this enricher ensures any properties added to the [Log Context](https://github.com/serilog/serilog/wiki/Enrichment) are pushed into log events
    日誌上下文增強器 - 此增強器內建於 Serilog 中，可確保新增至 [日誌上下文](https://github.com/serilog/serilog/wiki/Enrichment) 中的任何屬性都會被推送到日誌事件中。
- [Environment enrichers](https://github.com/serilog/serilog-enrichers-environment) - Enrich logs with the machine or current user name
    [環境增強器](https://github.com/serilog/serilog-enrichers-environment) - 使用機器名稱或目前使用者名稱豐富日誌

Enrichers can be specified using the `Enrich.With` fluent API of the Serilog `LoggerConfiguration` or via your `appsettings.json` file (recommended):
可以使用 Serilog `LoggerConfiguration` 的 `Enrich.With` 流暢 API 指定增強器，也可以透過 `appsettings.json` 檔案指定（建議）：

```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console"
    ],
    "MinimumLevel": {
      "Default": "Information"
    },
    "WriteTo": [
      {
        "Name": "Console"
      }
    ],
    "Enrich": [
      "FromLogContext",
      "WithMachineName"
    ],
    "Properties": {
      "ApplicationName": "Gateway API"
    }
  }
}
```

## Enrich from global properties 從全域屬性中豐富

You can also specify properties globally. The above snippet from `appsettings.json` demonstrates how we commonly set the `ApplicationName` property. In some cases, we need to calculate properties on startup, which can be done using the Fluent API:
您也可以全域指定屬性。以上來自 `appsettings.json` 的程式碼片段示範了我們通常如何設定 `ApplicationName` 屬性。在某些情況下，我們需要在啟動時計算屬性，這可以使用 Fluent API 來實現：

```csharp
loggerConfiguration.ReadFrom.Configuration(hostContext.Configuration)
    .EnrichWithEventType()
    .Enrich.WithProperty("Version", ReflectionUtils.GetAssemblyVersion<Program>());
```

# Correlating logs  相關日誌

In order to correlate logs that belong to the same request, even across multiple applications, add a `CorrelationId` property to your logs.
為了關聯屬於相同請求的日誌（即使跨越多個應用程式），請在日誌中新增 `CorrelationId` 屬性。

In HTTP applications we typically map this from the `HttpContext.TraceIdentifier` property. This is passed between internal APIs using the `Cko-Correlation-Id` header. We use the following extension to obtain the _current_correlation ID:
在 HTTP 應用中，我們通常透過 `HttpContext.TraceIdentifier` 屬性來對應此資訊。此資訊透過 `Cko-Correlation-Id` 標頭在內部 API 之間傳遞。我們使用以下擴充功能來取得 _current_correlation ID：

```csharp
public static string GetCorrelationId(this HttpContext httpContext)
{
    httpContext.Request.Headers.TryGetValue("Cko-Correlation-Id", out StringValues correlationId);
    return correlationId.FirstOrDefault() ?? httpContext.TraceIdentifier;
}
```

**Note that if the application is public facing, you should not rely on the provided correlation ID header.
請注意，如果應用程式是面向公眾的，則不應依賴提供的關聯 ID 標頭。**

To ensure that the correlation ID is pushed into every log event we use the following middleware that uses Serilog’s `LogContext` (discussed in more detail later in this article):
為了確保將關聯 ID 推送到每個日誌事件中，我們使用以下中間件，該中間件使用 Serilog 的 `LogContext` （本文稍後將詳細討論）：

```csharp
public class RequestLogContextMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLogContextMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public Task Invoke(HttpContext context)
    {
        using (LogContext.PushProperty("CorrelationId", context.GetCorrelationId()))
        {
            return _next.Invoke(context);
        }
    }
}
```

# Message Templating  訊息模板

Log messages should provide a short description of the event. We commonly see developers create overly verbose messages as means of including additional data in the event, for example:
日誌訊息應提供事件的簡要描述。我們經常看到開發人員為了在事件中包含額外資料而創建過於冗長的訊息，例如：

```csharp
_logger.Information("Storing payment state in Couchbase for Payment ID {PaymentId} and current state {State}", paymentId, state);
```

Instead you can use `ForContext` (or the property bag enricher at the bottom of this article) to still include the data but have terser messages:
或者，您可以使用 `ForContext` （或本文末尾的屬性包增強器）來包含數據，但訊息會更簡潔：

```csharp
_logger
    .ForContext("PaymentId", paymentId)
    .ForContext("State", state)
    .Information("Storing payment state in Couchbase");
```

## Message Template Recommendations 郵件範本推薦

### Fluent Style Guideline  Fluent 風格指南

Good Serilog events use the names of properties as content within the message to improve readability and make events more compact, for example:
優秀的 Serilog 事件會使用屬性名稱作為訊息內容，以提高可讀性並使事件更加簡潔，例如：

```csharp
_logger.Information("Processed {@Position} in {Elapsed:000} ms.", position, elapsedMs);
```

### Sentences vs. Fragments  句子與句子片段

Log event messages are fragments, not sentences; for consistency with other libraries that use Serilog, avoid a trailing period/full stop when possible.
日誌事件訊息是片段，而不是句子；為了與其他使用 Serilog 的庫保持一致，請盡可能避免在末尾添加句號。

### Templates vs. Messages  範本與訊息

Serilog events have a message template associated, _not_ a message. Internally, Serilog parses and caches every template (up to a fixed size limit). Treating the string parameter to log methods as a message, as in the case below, will degrade performance and consume cache memory. For example, avoid:
Serilog 事件關聯的是訊息模板， _ 而不是 _ 訊息本身。 Serilog 內部會解析並快取每個模板（快取大小有限制）。如果像下面的例子一樣，將字串參數作為訊息傳遞給日誌方法，會降低效能並消耗快取記憶體。例如，應避免：

```csharp
Log.Information("The time is " + DateTime.Now);
```

Instead use message properties:
請改用訊息屬性：

```csharp
Log.Information("The time is {Now}", DateTime.Now);
```

Aside from the performance overhead of using string concatenation/interpolation within your log messages it also means that a consistent event type cannot be calculated (see Event Type Enricher) making it impossible to find all the logs of a particular type.
除了在日誌訊息中使用字串連接/插值會帶來效能開銷之外，這也意味著無法計算一致的事件類型（請參閱事件類型豐富器），因此無法找到特定類型的所有日誌。

# Log and Diagnostic Context 日誌和診斷上下文

Serilog supports two context-aware features that can be used to enhance your logs.
Serilog 支援兩種情境感知功能，可用於增強您的日誌。

## Log Context  日誌上下文

The `LogContext` can be used to dynamically add and remove properties from the ambient “execution context”; for example, all messages written during a transaction might carry the id of that transaction, and so-on.
`LogContext` 可用於動態地向環境「執行上下文」新增和刪除屬性；例如，在事務期間寫入的所有訊息都可能攜帶該交易的 ID，依此類推。

The `RequestLogContextMiddleware` presented above demonstrates how we push the `CorrelationId` of the request into the `LogContext` at the beginning of the request. This ensures that all logs within that request _include_ that property.
上面展示的 `RequestLogContextMiddleware` 示範如何在請求開始時將請求的 `CorrelationId` 推送到 `LogContext` 。這可確保該請求中的所有日誌都 _ 包含 _ 該屬性。

More information can be found on the [Serilog wiki](https://github.com/serilog/serilog/wiki/Enrichment#the-logcontext).
更多資訊請參考 [Serilog wiki](https://github.com/serilog/serilog/wiki/Enrichment#the-logcontext) 。

## Diagnostic Context  診斷背景

One challenge with logging is that context is not always known upfront. For example, during the processing of a HTTP request, additional context is _gained_ as we progress through the HTTP pipeline such as knowing the identity of the user. Whilst `LogContext` would all us to create new contexts as additional information becomes available, this information would only be available in _subsequent_log entries. This commonly leads to an increase in the number of logs, just to capture everything about the overall request or operation.
日誌記錄面臨的一個挑戰是，上下文資訊並非總是預先已知的。例如，在處理 HTTP 請求的過程中，隨著 HTTP 流程的推進，我們會 _ 獲得 _ 更多上下文訊息，例如使用者身分。雖然 `LogContext` 允許我們在獲得額外資訊時建立新的上下文，但這些資訊只能在後續的日誌條目中記錄。這通常會導致日誌數量增加，只是為了記錄整個請求或操作的所有資訊。

The diagnostic context is provides an execution context (similar to `LogContext`) with the advantage that it can be enriched throughout its lifetime. The request logging middleware then uses this to enrich the final “log completion event”. This allows us to collapse many different log operations into a single log entry, containing information from many points in the request pipeline, for example:
診斷上下文提供了一個執行上下文（類似於 `LogContext` ），其優勢在於可以在其整個生命週期中不斷豐富。請求日誌中間件隨後使用此上下文來豐富最終的「日誌完成事件」。這使我們能夠將許多不同的日誌操作合併到單一日誌條目中，其中包含來自請求管道中多個點的信息，例如：

![Enriched properties in SEQ](https://benfoster.io/blog/serilog-best-practices/seq-properties.png)

Here you can see that not only have the HTTP properties emitted by the middleware but also application data such as `AcquirerId`, `MerchantName` and `ResponseCode`. These data points come from different points in the request but are pushed into the diagnostic context via the `IDiagnosticContext` interface:
這裡可以看到，不僅有中間件發出的 HTTP 屬性，還有應用程式數據，例如 `AcquirerId` 、 `MerchantName` 和 `ResponseCode` 。這些資料點來自請求中的不同位置，但透過 `IDiagnosticContext` 介面推送到診斷上下文：

```csharp
public class HomeController : Controller
{
    readonly IDiagnosticContext _diagnosticContext;

    public HomeController(IDiagnosticContext diagnosticContext)
    {
        _diagnosticContext = diagnosticContext ?? throw new ArgumentNullException(nameof(diagnosticContext));
    }

    public IActionResult Index()
    {
        // The request completion event will carry this property
        _diagnosticContext.Set("CatalogLoadTime", 1423);

        return View();
    }
```

### Using Diagnostic Context in non-HTTP applications 在非 HTTP 應用程式中使用診斷上下文

Diagnostic Context is not limited to usage within ASP.NET Core. It can also be used in non-HTTP applications in a very similar way to the request logging middleware. For example, we are using it to generate a completion log event in our SQS consumers.
診斷上下文的使用並不限於 ASP.NET Core。它也可以用於非 HTTP 應用程序，其使用方式與請求日誌中間件非常相似。例如，我們使用它在 SQS 用戶中產生完成日誌事件。

# Configuration  配置

Serilog can be configured using either a Fluent API or via the Microsoft Configuration system. We recommend using the configuration system since the logging configuration can be changed without releasing a new version of your application.
Serilog 可以透過 Fluent API 或 Microsoft 設定係統進行設定。我們建議使用配置系統，因為這樣無需發布新版本的應用程式即可更改日誌配置。

To do so, add the [Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration) package and configure Serilog as below:
為此，請新增 [Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration) 套件並如下設定 Serilog：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((hostContext, loggerConfiguration) =>
        {
            loggerConfiguration.ReadFrom.Configuration(hostContext.Configuration);
        })
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            config
                .AddEnvironmentVariables(prefix: "FLOW_")
                .AddAwsSecrets();
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();  
        });
```

You can now configure Serilog via any supported configuration provider. Typically we use `appsettings.json` for global settings and configure the actual sinks via environment variables in production (since we don’t want to use our remote logging services when running locally):
現在您可以透過任何受支援的配置提供者設定 Serilog。通常，我們使用 `appsettings.json` 進行全域設置，並在生產環境中透過環境變數配置實際的日誌接收器（因為我們不希望在本地運行時使用遠端日誌服務）：

```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.Datadog.Logs"
    ],
    "MinimumLevel": {
      "Default": "Information"
    },
    "WriteTo": [
      {
        "Name": "Console"
      }
    ],
    "Enrich": [
      "FromLogContext",
      "WithMachineName"
    ],
    "Properties": {
      "ApplicationName": "Flow API"
    }
  }
}
```

# Production logging  生產紀錄

When deploying your applications in production, ensure that logging is configured accordingly:
在生產環境中部署應用程式時，請確保已正確配置日誌記錄：

- Console logging should be restricted to `Error`. In .NET writing to Console is a blocking call and can have a [significant performance impact](https://weblog.west-wind.com/posts/2018/Dec/31/Dont-let-ASPNET-Core-Default-Console-Logging-Slow-your-App-down).
    控制台日誌記錄應僅限於 `Error` 。在 .NET 中，寫入控制台是一個阻塞調用，可能會對 [效能產生顯著影響](https://weblog.west-wind.com/posts/2018/Dec/31/Dont-let-ASPNET-Core-Default-Console-Logging-Slow-your-App-down) 。
- Global logging should be configured for `Information` and above.
    `Information` 以上應配置全域日誌記錄。

It’s understood that during the release of a new project you may need additional information to build confidence in the solution or to diagnose any expected teething issues. Rather than upgrading your log entries to `Information` for this, consider enabling `Debug` level for a limited period of time.
我們瞭解，在發布新專案期間，您可能需要額外資訊來增強對解決方案的信心，或診斷任何預期的初期問題。為此，與其將日誌條目升級到 `Information` ，不如考慮在一段時間內啟用 `Debug` 級別。

A common question heard from developers is how they can dynamically switch logging levels at runtime. Whilst this is possible, it can also be achieved using blue/green deployments. Configure and deploy the non-active environment with the reduced log level and then switch a portion or all of the traffic via weighted target groups.
開發人員經常會問到如何在運行時動態切換日誌等級。雖然這可以實現，但也可以使用藍綠部署來實現。先配置並部署一個日誌等級較低的非活動環境，然後透過加權目標群組將部分或全部流量切換到該環境。

## When logs become more than logs 當日誌不再只是日誌時

Logs can provide a lot of insight into an application and in many cases be sufficient to handle day-to-day support requests or troubleshooting. There are however cases where logs are not the _right tool for the job_, with a number of warning signs:
日誌可以提供很多關於應用程式的信息，在許多情況下足以處理日常支援請求或故障排除。然而，在某些情況下，日誌並非 _ 最佳工具 _ ，以下是一些警告信號：

- You find yourself opening up application logs to non-technical users
    你發現自己正在向非技術用戶開放應用程式日誌。
- Logs are being used to generate application metrics
    日誌用於產生應用程式指標。
- More information is being “stuffed” into logs to satisfy common support requests or reporting requirements
    為了滿足常見的支援請求或報告要求，越來越多的信息被「塞」進日誌中。

In these cases, you may need to consider dedicated tooling for your product. A number of teams have developed “Inspector” like applications that aggregate key system and business data together to handle BAU requests that can be provided to non-technical stakeholders. Additionally you may find the need to push data from your application into reporting and analytics tools.
在這些情況下，您可能需要考慮為您的產品開發專用工具。許多團隊已經開發出類似“Inspector”的應用程序，這些應用程式可以將關鍵的系統和業務資料聚合在一起，以處理日常業務請求，並提供給非技術利益相關者。此外，您可能還需要將應用程式中的資料推送到報告和分析工具中。

The effectiveness of your logs is both what you log and what you _don’t_ log.
日誌的有效性既取決於你記錄了什麼，也取決於你 _ 沒有 _ 記錄什麼。

# Other tools and utilities 其他工具和實用程式

## Use Seq locally  本地使用 Seq

[Seq](https://datalust.co/seq) is a free (for local use) logging tool created by the author of Serilog. It provides advanced search and filtering capabilities as well as full access to structured log data. Whilst our logging requirements now extend beyond what Seq can offer, it is still a great option for testing locally.
[Seq](https://datalust.co/seq) 是 Serilog 作者開發的免費（僅限本機使用）日誌記錄工具。它提供高級搜尋和過濾功能，並可完全存取結構化日誌資料。雖然我們現在的日誌記錄需求超出了 Seq 的功能範圍，但它仍然是本地測試的絕佳選擇。

We usually spin up Seq in docker as part of a separate docker-compose file (`docker-compose-logging.hml`):
我們通常會在單獨的 docker-compose 檔案（ `docker-compose-logging.hml` ）中啟動 docker 中的 Seq：

```yaml
version: "3.5"

services:

seq:
    image: datalust/seq:latest
    container_name: seq
    ports:
    - '5341:80'
    environment:
    - ACCEPT_EULA=Y
    networks:
    - gateway-network

networks:
gateway-network:
    name: gateway-network
```

And configure our `appsettings.Development.json` file to use the Seq sink:
並將 `appsettings.Development.json` 檔案配置為使用 Seq sink：

```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.Seq"
    ],
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "Microsoft": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://localhost:5341",
          "apiKey": "none"
        }
      }
    ]
  }
}
```

## Event Type Enricher  事件類型增強器

Often we need to uniquely identify logs of the same type. Some sinks such as [Seq](https://datalust.co/seq) do this automatically by hashing the message template. To replicate the same behaviour in other sinks we created the following enricher which uses the [Murmerhash algorithm](https://github.com/darrenkopp/murmurhash-net):
我們經常需要唯一標識相同類型的日誌。一些接收器（例如 [Seq）](https://datalust.co/seq) 透過對訊息範本進行雜湊處理來自動完成此操作。為了在其他接收器中實現相同的行為，我們創建了以下使用 [Murmerhash 演算法](https://github.com/darrenkopp/murmurhash-net) 的增強器：

```csharp
internal class EventTypeEnricher : ILogEventEnricher
{
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        if (logEvent is null)
            throw new ArgumentNullException(nameof(logEvent));

        if (propertyFactory is null)
            throw new ArgumentNullException(nameof(propertyFactory));

        Murmur32 murmur = MurmurHash.Create32();
        byte[] bytes = Encoding.UTF8.GetBytes(logEvent.MessageTemplate.Text);
        byte[] hash = murmur.ComputeHash(bytes);
        string hexadecimalHash = BitConverter.ToString(hash).Replace("-", "");
        LogEventProperty eventId = propertyFactory.CreateProperty("EventType", hexadecimalHash);
        logEvent.AddPropertyIfAbsent(eventId);
    }
}
```

## Property Bag Enricher  屬性袋增強器

In cases where you want to add multiple properties to your log events, use the `PropertyBagEnricher`:
如果要為日誌事件新增多個屬性，請使用 `PropertyBagEnricher` ：

```csharp
public class PropertyBagEnricher : ILogEventEnricher
{
    private readonly Dictionary<string, Tuple<object, bool>> _properties;

    /// <summary>
    /// Creates a new <see cref="PropertyBagEnricher" /> instance.
    /// </summary>
    public PropertyBagEnricher()
    {
        _properties = new Dictionary<string, Tuple<object, bool>>(StringComparer.OrdinalIgnoreCase);
    }

    /// <summary>
    /// Enriches the <paramref name="logEvent" /> using the values from the property bag.
    /// </summary>
    /// <param name="logEvent">The log event to enrich.</param>
    /// <param name="propertyFactory">The factory used to create the property.</param>
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        foreach (KeyValuePair<string, Tuple<object, bool>> prop in _properties)
        {
            logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty(prop.Key, prop.Value.Item1, prop.Value.Item2));
        }
    }

    /// <summary>
    /// Add a property that will be added to all log events enriched by this enricher.
    /// </summary>
    /// <param name="key">The property key.</param>
    /// <param name="value">The property value.</param>
    /// <param name="destructureObject">
    /// Whether to destructure the value. See https://github.com/serilog/serilog/wiki/Structured-Data
    /// </param>
    /// <returns>The enricher instance, for chaining Add operations together.</returns>
    public PropertyBagEnricher Add(string key, object value, bool destructureObject = false)
    {
        if (string.IsNullOrEmpty(key)) throw new ArgumentNullException(nameof(key));

        if (!_properties.ContainsKey(key)) _properties.Add(key, Tuple.Create(value, destructureObject));

        return this;
    }
}
```

### Usage:  用法

```csharp
_logger
    .ForContext(
      new PropertyBagEnricher()
        .Add("ResponseCode", response?.ResponseCode)
        .Add("EnrollmentStatus", response?.Enrolled)
    )
    .Warning("Malfunction when processing 3DS enrollment verification");
```

## Request Log Enricher  請求日誌增強器

The Serilog request logging middleware allows a function to be provided that can be used to add additional information from the HTTP request to the completion log event. We use this to log the `ClientIP`, `UserAgent` and `Resource` properties:
Serilog 請求日誌中間件允許提供一個函數，用於將 HTTP 請求中的附加資訊新增至完成日誌事件。我們使用此函數來記錄 `ClientIP` 、 `UserAgent` 和 `Resource` 屬性：

```csharp
public static class LogEnricher
{
    /// <summary>
    /// Enriches the HTTP request log with additional data via the Diagnostic Context
    /// </summary>
    /// <param name="diagnosticContext">The Serilog diagnostic context</param>
    /// <param name="httpContext">The current HTTP Context</param>
    public static void EnrichFromRequest(IDiagnosticContext diagnosticContext, HttpContext httpContext)
    {
        diagnosticContext.Set("ClientIP", httpContext.Connection.RemoteIpAddress.ToString());
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"].FirstOrDefault());
        diagnosticContext.Set("Resource", httpContext.GetMetricsCurrentResourceName());
    }
}
```

### Usage  用法

```csharp
app.UseSerilogRequestLogging(opts
  => opts.EnrichDiagnosticContext = LogEnricher.EnrichFromRequest);
```
