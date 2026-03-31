---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 是否應在 Library 內直接包入 Serilog

不建議在「通用 Library」中直接相依 Serilog，原因如下。

- 破壞呼叫端的日誌策略
    Library 會強制使用 Serilog，使應用程式無法自由選擇 NLog、log4net、ETW、Application Insights 等
- 日誌生命週期不在 Library 掌控內
    Sink 初始化、Flush、Close 通常應由最上層 Application 負責
- NuGet 相依性污染
    Library 使用者即使不需要 logging，也被迫引入 Serilog 與相關 Sink
- 設定衝突風險
    Library 若自行設定 Logger，容易覆蓋或干擾 Host Application 的設定

適合直接使用 Serilog 的情境。

- 專案屬於「公司內部整套系統」
- Library 與 Host App 一起部署、一起維護
- 明確規範 Serilog 為標準 logging 框架

## Library 正確的例外與錯誤處理原則

核心原則只有一個。
- Library「回報錯誤」，不「決定怎麼記錄」

### 1. 直接丟出例外（最推薦）

- 發生不可恢復錯誤時，直接 throw
- 保留完整 StackTrace
- 不在 Library 內吞掉例外

適合情境。

- 邏輯錯誤
- 狀態不一致
- 呼叫端必須知道錯誤

### 2. 使用 .NET 抽象介面 ILogger（Microsoft.Extensions.Logging）

這是最通用、最推薦的折衷方案。

- Library 只相依抽象
- Host App 決定是否用 Serilog、NLog 或其他 Provider
- 幾乎是現代 .NET Library 的標準作法

建議做法。

- 在 Library 建構子注入 ILogger
- 不做任何 LoggerFactory 初始化
- 不設定 Sink

### 3. 提供 Logging Hook（進階選項）

適合「不想相依任何 logging 套件」的極簡 Library。

- 提供 Action 或介面讓呼叫端掛接
- Library 只負責呼叫，不關心實作

缺點。

- 標準化不足
- 呼叫端需要自行包裝

## 不同方案的比較

- 直接使用 Serilog
    使用方便，但彈性最低，僅適合內部專案
- 丟例外，不記 log
    最乾淨、最安全，呼叫端完全掌控
- Microsoft.Extensions.Logging
    通用性最佳，與 Serilog 高度相容
- 自訂 Logging Hook
    相依最小，但維護成本高

## 實務建議（結論）

- 公開或可重用 Library
    不要直接引用 Serilog
- 需要記錄「診斷資訊」
    注入 ILogger
- 需要回報錯誤
    丟出明確、語意化的 Exception
- 記錄行為
    永遠交給最上層 Application

---

# 不相依 Serilog、但能無縫接 Serilog 的 Library 架構範例

## 範例目標與前提

- Library 不直接相依 Serilog
- 支援例外回拋
- 可由 Host App 自由接 Serilog
- 適用 .NET Framework 4.6.2（WinForm / Console 皆可）
- 使用 Microsoft.Extensions.Logging 抽象層

## Library 專案設計

### NuGet 相依

- Microsoft.Extensions.Logging.Abstractions

Library 只引用 Abstractions，不引用任何實作。

### Library 類別實作範例

```csharp
using System;
using Microsoft.Extensions.Logging;

namespace MyCompany.MyLibrary
{
    public class DeviceController
    {
        private readonly ILogger<DeviceController> _logger;

        public DeviceController(ILogger<DeviceController> logger)
        {
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public void Connect(string portName)
        {
            if (string.IsNullOrWhiteSpace(portName))
                throw new ArgumentException("Port name is required.", nameof(portName));

            _logger.LogInformation("Connecting to device on port {PortName}", portName);

            try
            {
                // 模擬實際連線邏輯
                if (portName == "COM0")
                    throw new InvalidOperationException("Invalid port.");

                _logger.LogInformation("Device connected successfully.");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Device connection failed.");
                throw;
            }
        }
    }
}
```

設計重點。

- Library 不建立 LoggerFactory
- 不設定任何 Sink
- 發生錯誤仍然 throw
- Log 僅作為「診斷輔助」

## Host App（WinForm / Console）整合 Serilog

### NuGet 相依（Host App）

- Serilog
- Serilog.Extensions.Logging
- Serilog.Sinks.File（或其他 Sink）
- Microsoft.Extensions.Logging

### Serilog 初始化

```csharp
using Microsoft.Extensions.Logging;
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.File("app.log", rollingInterval: RollingInterval.Day)
    .CreateLogger();

ILoggerFactory loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddSerilog(dispose: true);
});
```

### 建立 Library 物件

```csharp
using MyCompany.MyLibrary;
using Microsoft.Extensions.Logging;

ILogger<DeviceController> logger =
    loggerFactory.CreateLogger<DeviceController>();

var controller = new DeviceController(logger);

controller.Connect("COM1");
```

關鍵結果。

- Library 自動寫入 Serilog
- Library 完全不知道 Serilog 存在
- Host 可隨時改成 NLog 或其他 Provider

## 不使用 ILogger 的極簡替代方案（可選）

### Library 定義 Hook

```csharp
public class SimpleService
{
    public Action<string, Exception> OnError;

    public void Run()
    {
        try
        {
            throw new InvalidOperationException("Something failed.");
        }
        catch (Exception ex)
        {
            OnError?.Invoke("Run failed.", ex);
            throw;
        }
    }
}
```

缺點。

- 無 LogLevel
- 無結構化日誌
- 不建議大型專案使用

## 實務結論整理

- 可重用 Library
    使用 Microsoft.Extensions.Logging.Abstractions
- 不可恢復錯誤
    一律 throw
- 日誌框架選擇權
    完全交給 Host App
- Serilog
    只存在於最上層 Application

---

## 大型專案下的 Logging 與 Exception 規劃目標

在規模較大的系統中，重點不只是「能記錄」，而是。

- Logging 行為一致
- 例外語意清楚、可追蹤
- 可跨模組整合與分析
- 不影響系統可替換性與可測試性

以下以「多 Library + 多應用程式」為前提說明。

## 分層責任劃分（最重要）

- Domain / Core Library
    不做 logging，只定義例外型別
- Application / Service Layer
    記錄「流程與狀態」的 log
- Infrastructure / Host App
    決定 logging framework、Sink、格式、留存策略

這個劃分是大型專案能長期維護的關鍵。

## 例外設計策略

### 1. 自訂語意化 Exception

避免直接丟出 Exception、InvalidOperationException。

```csharp
public class DeviceConnectionException : Exception
{
    public string PortName { get; }

    public DeviceConnectionException(string portName, Exception inner)
        : base($"Failed to connect device on port '{portName}'.", inner)
    {
        PortName = portName;
    }
}
```

原則。

- 一個「錯誤情境」對應一個 Exception 型別
- Exception 本身即是規格的一部分
- 不在 Exception 內做 logging

### 2. Domain 不 catch，Application 才轉譯

- Domain 層
    只 throw
- Application 層
    catch 並轉成「可理解的例外」或 Result

## Logging 架構規劃（推薦標準）

### 統一使用 Microsoft.Extensions.Logging

- 所有非 Domain 層皆可注入 ILogger
- 不允許自行 new LoggerFactory
- 不允許在 Library 設定 Sink

### 定義 Logging 分類與層級規範

建議在公司或專案層級定義規則。

- LogTrace
    極高頻、除錯細節
- LogDebug
    開發用狀態
- LogInformation
    業務流程節點（開始、完成、狀態轉換）
- LogWarning
    可恢復但需注意
- LogError
    操作失敗、功能中斷
- LogCritical
    系統無法繼續運作

避免問題。

- 所有錯誤都用 LogError
- catch(Exception) 就寫 log

## 專案結構範例

```csharp
/src
    /MyProduct.Domain
        - 不引用 Microsoft.Extensions.Logging
        - 定義 Entity / ValueObject / DomainException
    /MyProduct.Application
        - 引用 Logging.Abstractions
        - 協調流程、記錄 log
    /MyProduct.Infrastructure
        - 實作 I/O、裝置、DB
        - 可記錄技術性 log
    /MyProduct.WinFormApp
        - 初始化 Serilog
        - 設定 Sink / Enricher
```

## Logging Context（大型系統必備）

一定要規劃「關聯識別」。

- CorrelationId
- OperationId
- UserId
- DeviceId

實作方式。

- 使用 BeginScope
- 或在 Serilog 加 Enricher

效果。

- 可跨模組追蹤一次操作
- Log 聚合工具可完整還原流程

## 反例（大型專案常見災難）

- 每個 Library 各自包 Serilog
- Domain Entity 裡寫 log
- Exception 丟出前先記一次 Error，最上層又記一次
- 靠 log 訊息字串判斷錯誤型別

## 建議落地做法（實戰）

- 寫一份「Logging & Exception Coding Guideline」
- Code Review 明確檢查
    有沒有在錯的層寫 log
- 封裝共用的 OperationScope
- Log 是診斷資料，不是控制流程

## 總結

- 大型專案
    架構一致性比工具選擇重要
- Logging
    一定抽象化，集中化
- Exception
    是 API 契約的一部分
- Serilog
    只負責輸出，不負責設計

---

## Logging 與 Exception 設計規範（大型專案版）

本文件適用於「多 Library、多 Application」的中大型 .NET 專案，目標是確保可維護性、可追蹤性與技術一致性。

## 一、核心設計原則

- Logging 是診斷工具，不是控制流程
- Exception 是 API 契約的一部分
- Library 不決定日誌框架與輸出方式
- 所有記錄行為必須可被 Host App 統一管理

## 二、分層責任規範

### Domain Layer（核心領域）

- 不引用任何 logging 套件
- 不使用 ILogger
- 不 catch 再 throw
- 只負責定義語意化 Exception
- Exception 不做 logging

允許內容。

- Entity
- ValueObject
- Domain Service
- DomainException

禁止事項。

- LogXXX 呼叫
- try/catch 僅為了記 log

### Application Layer（流程協調）

- 可引用 Microsoft.Extensions.Logging.Abstractions
- 負責流程型 log
- 負責例外轉譯與封裝
- 不設定 Sink、不建立 LoggerFactory

典型責任。

- UseCase / Service
- Workflow
- Command / Query Handler

### Infrastructure Layer（技術實作）

- 可使用 ILogger
- 記錄技術細節 log
- 不吞掉例外
- 不決定最終錯誤呈現方式

包含。

- DB
- File
- Device
- Network
- External API

### Host Application（WinForm / Service / Web）

- 初始化 logging framework（如 Serilog）
- 設定 Sink、Format、Retention
- 設定 Enricher
- 捕捉最外層未處理例外

## 三、Logging 使用規範

### Logger 抽象

- 一律使用 Microsoft.Extensions.Logging
- 僅注入 ILogger
- 禁止直接引用 Serilog、NLog 於 Library

### Log Level 定義

- LogTrace
    極高頻率、細節追蹤，不預設開啟
- LogDebug
    開發除錯資訊
- LogInformation
    業務流程節點（開始、完成、狀態轉換）
- LogWarning
    非預期但可恢復狀況
- LogError
    操作失敗、功能中斷
- LogCritical
    系統無法繼續運作

### Logging 行為準則

- 不在每一層都記同一個錯誤
- 有 throw 的地方不一定要 log
- 有 log 的地方不一定要 throw
- 禁止使用 log 訊息文字作為邏輯判斷依據

## 四、Exception 設計規範

### 語意化 Exception

- 一個錯誤情境一個型別
- Exception 命名必須具體
- 必須保留 InnerException

範例命名。

- DeviceConnectionException
- ConfigurationMissingException
- DataIntegrityViolationException

### Catch 與轉譯原則

- Domain 層
    不 catch
- Infrastructure 層
    可 catch 技術例外後包成語意化例外
- Application 層
    決定是否轉為可顯示錯誤或回傳 Result

### 禁止事項

- catch(Exception) 只為了吞掉
- throw ex
- 使用 Exception 當作一般流程控制

## 五、關聯追蹤（大型專案必備）

### 必須具備的 Context

- CorrelationId
- OperationId
- UserId（若有）
- DeviceId（若有）

### 實作方式

- 使用 ILogger.BeginScope
- 或由 Host App 透過 Serilog Enricher 注入

目的。

- 可跨模組還原一次操作完整流程
- 支援集中式 Log 分析工具

## 六、專案結構建議

```csharp
/src
    /Product.Domain
        - 無 logging
        - DomainException
    /Product.Application
        - ILogger<T>
        - 流程 log
    /Product.Infrastructure
        - 技術 log
    /Product.HostApp
        - Serilog 設定
        - Sink / Enricher
```

## 七、Code Review 檢查清單

- Domain 是否出現 logging
- Library 是否直接引用 Serilog
- 是否重複記錄同一個例外
- Exception 是否具備語意
- Logging 是否符合層級責任

## 八、總結原則

- Logging 架構先於工具選擇
- Exception 是設計，不是臨時補救
- Library 永遠保持中立
- Host App 擁有最終決定權
