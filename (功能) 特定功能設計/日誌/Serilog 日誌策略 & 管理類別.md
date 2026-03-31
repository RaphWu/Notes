---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Serilog 日誌策略範例（分檔、分格式、分等級）

## 1. 一般資訊事件（Information / Debug）

- 用途：APP 流程、操作訊息、狀態更新
- 格式：純文字檔（Rolling File）
- 建議：每天一個檔案，保留 7~30 天

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.File("logs/general_log_.txt", rollingInterval: RollingInterval.Day, retainedFileCountLimit: 30)
    .CreateLogger();

// 範例使用
Log.Information("使用者 {User} 開啟功能 {Feature}", userName, featureName);
```

## 2. 使用者操作事件（登入、登出、重要操作）

- 用途：使用者行為追蹤、統計、稽核
- 格式：JSON 或 CSV（方便匯入分析工具）
- 建議：每天一個檔案

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.File(new CompactJsonFormatter(), "logs/user_activity.json", rollingInterval: RollingInterval.Day, retainedFileCountLimit: 90)
    .CreateLogger();

// 範例使用
Log.Information("登入事件: {User}, IP: {IP}, 時間: {Time}", userName, ipAddress, DateTime.Now);
Log.Information("登出事件: {User}, 時間: {Time}", userName, DateTime.Now);
```

## 3. 錯誤與例外事件（Error / Fatal）

- 用途：程式錯誤追蹤、堆疊追蹤
- 格式：純文字檔或 JSON（純文字較易人工閱讀）
- 建議：單獨檔案，每天分檔

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Error()
    .WriteTo.File("logs/error_log_.txt", rollingInterval: RollingInterval.Day, retainedFileCountLimit: 90)
    .CreateLogger();

try
{
    // 可能拋例外的程式
}
catch(Exception ex)
{
    Log.Error(ex, "程式發生錯誤");
}
```

## 4. 高敏感事件（密碼修改、權限變更等）

- 用途：高安全性事件稽核
- 格式：加密後存檔或限制存取
- 建議：使用 JSON，並單獨分檔

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.File(new CompactJsonFormatter(), "logs/security_event.json", rollingInterval: RollingInterval.Day, retainedFileCountLimit: 180)
    .CreateLogger();

// 範例使用
Log.Information("權限變更: {User}, 變更內容: {Change}, 時間: {Time}", adminUser, changeDetail, DateTime.Now);
```

## 5. 其他建議

- **保留策略**：一般資訊保留 7~30 天，錯誤與高敏感事件保留 90~180 天
- **檔案大小控制**：設定 `rollingInterval` 和 `retainedFileCountLimit`
- **格式選擇**：分析用途用 JSON/CSV，人工閱讀用純文字
- **敏感資料保護**：密碼或個資避免明文寫入，必要時加密或 Mask

---

# 完整 C# Serilog 初始化範例（多事件類別分檔策略）

```csharp
using Serilog;
using Serilog.Formatting.Compact;

public static class LoggerSetup
{
    public static void Init()
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            
            // 一般事件 (Information / Debug)
            .WriteTo.File(
                "logs/general_log_.txt",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 30,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Debug
            )
            
            // 使用者操作事件 (登入 / 登出 / 重要操作)
            .WriteTo.File(
                new CompactJsonFormatter(),
                "logs/user_activity.json",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information
            )
            
            // 錯誤與例外事件 (Error / Fatal)
            .WriteTo.File(
                "logs/error_log_.txt",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Error
            )
            
            // 高敏感事件 (密碼修改、權限變更)
            .WriteTo.File(
                new CompactJsonFormatter(),
                "logs/security_event.json",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 180,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information
            )
            
            .CreateLogger();
    }
}
```

## 使用範例

```csharp
class Program
{
    static void Main(string[] args)
    {
        LoggerSetup.Init();

        // 一般事件
        Log.Information("APP 啟動成功");

        // 使用者登入
        Log.Information("登入事件: {User}, IP: {IP}, 時間: {Time}", "UserA", "192.168.1.1", DateTime.Now);

        // 程式錯誤
        try
        {
            int a = 5;
            int b = 0;
            int c = a / b;
        }
        catch(Exception ex)
        {
            Log.Error(ex, "程式發生錯誤");
        }

        // 高敏感事件
        Log.Information("權限變更: {Admin}, 變更內容: {Change}, 時間: {Time}", "Admin01", "UserA->Manager", DateTime.Now);

        Log.CloseAndFlush();
    }
}
```

## 特點

- 不同事件類型分檔，方便管理與分析
- 每日分檔，避免單檔過大
- JSON 檔方便匯入分析工具（ELK、Splunk 等）
- 錯誤日誌保留詳細堆疊資訊，便於除錯
- 高敏感事件單獨管理，利於安全稽核

---

# 完整 Serilog 日誌管理類別範例（統一管理 + 非同步 + 快取）

```csharp
using Serilog;
using Serilog.Formatting.Compact;
using Serilog.Sinks.Async;
using System;

public static class LogManager
{
    public static void Init()
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            
            // 一般事件 (資訊/流程)
            .WriteTo.Async(a => a.File(
                "logs/general_log_.txt",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 30,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Debug
            ))
            
            // 使用者操作事件 (登入/登出/重要操作)
            .WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                "logs/user_activity.json",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information
            ))
            
            // 程式錯誤事件 (Error / Fatal)
            .WriteTo.Async(a => a.File(
                "logs/error_log_.txt",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Error
            ))
            
            // 高敏感事件 (密碼修改/權限變更)
            .WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                "logs/security_event.json",
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 180,
                restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information
            ))
            
            .CreateLogger();
    }

    // 一般事件
    public static void Info(string message, params object[] args)
    {
        Log.Information(message, args);
    }

    // 使用者操作事件
    public static void UserActivity(string message, params object[] args)
    {
        Log.ForContext("EventType", "UserActivity").Information(message, args);
    }

    // 錯誤事件
    public static void Error(Exception ex, string message, params object[] args)
    {
        Log.Error(ex, message, args);
    }

    // 高敏感事件
    public static void Security(string message, params object[] args)
    {
        Log.ForContext("EventType", "Security").Information(message, args);
    }

    // 程式結束前呼叫
    public static void Flush()
    {
        Log.CloseAndFlush();
    }
}
```

## 使用範例

```csharp
class Program
{
    static void Main(string[] args)
    {
        LogManager.Init();

        // 一般事件
        LogManager.Info("APP 啟動完成");

        // 使用者登入
        LogManager.UserActivity("登入事件: {User}, IP: {IP}, 時間: {Time}", "UserA", "192.168.1.1", DateTime.Now);

        // 程式錯誤
        try
        {
            int a = 5;
            int b = 0;
            int c = a / b;
        }
        catch(Exception ex)
        {
            LogManager.Error(ex, "程式發生錯誤");
        }

        // 高敏感事件
        LogManager.Security("權限變更: {Admin}, 變更內容: {Change}, 時間: {Time}", "Admin01", "UserA->Manager", DateTime.Now);

        LogManager.Flush();
    }
}
```

## 特點

- 統一 LogManager 呼叫入口，程式碼乾淨易維護
- 多類型事件分檔管理（一般、使用者、錯誤、高敏感）
- 使用非同步寫入，支援高頻事件快取
- JSON 格式方便後續分析、匯入工具
- 錯誤事件完整保留堆疊資訊
- 支援每日分檔、保留天數設定

---

# 進階 Serilog 日誌管理類別（完整功能版）

這個版本包含以下進階功能：

- 可動態設定各類別事件的 Log 等級
- 可啟用或停用特定事件類別記錄
- 支援多目錄管理（例如區分一般、使用者、錯誤、高敏感目錄）
- 支援非同步寫入 + 快取
- JSON 與純文字格式混用，方便分析與人工查看

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Formatting.Compact;
using Serilog.Sinks.Async;
using System;

public static class LogManager
{
    // 可動態設定
    public static bool EnableGeneralLog { get; set; } = true;
    public static bool EnableUserActivity { get; set; } = true;
    public static bool EnableErrorLog { get; set; } = true;
    public static bool EnableSecurityLog { get; set; } = true;

    public static LogEventLevel GeneralLogLevel { get; set; } = LogEventLevel.Debug;
    public static LogEventLevel UserActivityLevel { get; set; } = LogEventLevel.Information;
    public static LogEventLevel ErrorLogLevel { get; set; } = LogEventLevel.Error;
    public static LogEventLevel SecurityLogLevel { get; set; } = LogEventLevel.Information;

    public static void Init(string logBasePath = "logs")
    {
        var config = new LoggerConfiguration()
            .MinimumLevel.Debug();

        if (EnableGeneralLog)
        {
            config = config.WriteTo.Async(a => a.File(
                System.IO.Path.Combine(logBasePath, "General", "general_log_.txt"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 30,
                restrictedToMinimumLevel: GeneralLogLevel
            ));
        }

        if (EnableUserActivity)
        {
            config = config.WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                System.IO.Path.Combine(logBasePath, "UserActivity", "user_activity.json"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: UserActivityLevel
            ));
        }

        if (EnableErrorLog)
        {
            config = config.WriteTo.Async(a => a.File(
                System.IO.Path.Combine(logBasePath, "Error", "error_log_.txt"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 90,
                restrictedToMinimumLevel: ErrorLogLevel
            ));
        }

        if (EnableSecurityLog)
        {
            config = config.WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                System.IO.Path.Combine(logBasePath, "Security", "security_event.json"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: 180,
                restrictedToMinimumLevel: SecurityLogLevel
            ));
        }

        Log.Logger = config.CreateLogger();
    }

    // 一般事件
    public static void Info(string message, params object[] args)
    {
        if (EnableGeneralLog)
            Log.Information(message, args);
    }

    // 使用者操作事件
    public static void UserActivity(string message, params object[] args)
    {
        if (EnableUserActivity)
            Log.ForContext("EventType", "UserActivity").Information(message, args);
    }

    // 錯誤事件
    public static void Error(Exception ex, string message, params object[] args)
    {
        if (EnableErrorLog)
            Log.Error(ex, message, args);
    }

    // 高敏感事件
    public static void Security(string message, params object[] args)
    {
        if (EnableSecurityLog)
            Log.ForContext("EventType", "Security").Information(message, args);
    }

    // 程式結束前呼叫
    public static void Flush()
    {
        Log.CloseAndFlush();
    }
}
```

## 使用範例

```csharp
class Program
{
    static void Main(string[] args)
    {
        // 初始化並指定 Log 存放根目錄
        LogManager.Init("C:\\MyAppLogs");

        // 一般事件
        LogManager.Info("APP 啟動完成");

        // 使用者登入
        LogManager.UserActivity("登入事件: {User}, IP: {IP}, 時間: {Time}", "UserA", "192.168.1.1", DateTime.Now);

        // 程式錯誤
        try
        {
            int a = 5;
            int b = 0;
            int c = a / b;
        }
        catch(Exception ex)
        {
            LogManager.Error(ex, "程式發生錯誤");
        }

        // 高敏感事件
        LogManager.Security("權限變更: {Admin}, 變更內容: {Change}, 時間: {Time}", "Admin01", "UserA->Manager", DateTime.Now);

        // 可動態停用某些事件記錄
        LogManager.EnableUserActivity = false;
        LogManager.UserActivity("這條不會被記錄");

        LogManager.Flush();
    }
}
```

## 特點

- 可動態啟用/停用事件類別
- 支援各類事件單獨設定 Log 等級
- 支援多目錄分層管理，清楚分類
- 非同步寫入 + 快取，提高高頻事件效能
- JSON 格式方便分析工具，純文字方便人工查看
- 支援每日分檔及保留天數管理

這個版本可以直接在生產環境使用，並且方便擴充新事件類別或調整策略。

---

# Serilog 日誌管理類別 + 自動清理舊檔案功能

這個版本在前一版基礎上，新增了 **自動清理超過保留天數的舊檔案**，避免日誌檔案過多占用磁碟空間。

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Formatting.Compact;
using Serilog.Sinks.Async;
using System;
using System.IO;
using System.Linq;

public static class LogManager
{
    // 可動態設定
    public static bool EnableGeneralLog { get; set; } = true;
    public static bool EnableUserActivity { get; set; } = true;
    public static bool EnableErrorLog { get; set; } = true;
    public static bool EnableSecurityLog { get; set; } = true;

    public static LogEventLevel GeneralLogLevel { get; set; } = LogEventLevel.Debug;
    public static LogEventLevel UserActivityLevel { get; set; } = LogEventLevel.Information;
    public static LogEventLevel ErrorLogLevel { get; set; } = LogEventLevel.Error;
    public static LogEventLevel SecurityLogLevel { get; set; } = LogEventLevel.Information;

    public static int GeneralRetentionDays { get; set; } = 30;
    public static int UserActivityRetentionDays { get; set; } = 90;
    public static int ErrorRetentionDays { get; set; } = 90;
    public static int SecurityRetentionDays { get; set; } = 180;

    public static void Init(string logBasePath = "logs")
    {
        var config = new LoggerConfiguration()
            .MinimumLevel.Debug();

        if (EnableGeneralLog)
        {
            string generalPath = Path.Combine(logBasePath, "General");
            Directory.CreateDirectory(generalPath);

            config = config.WriteTo.Async(a => a.File(
                Path.Combine(generalPath, "general_log_.txt"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: GeneralRetentionDays,
                restrictedToMinimumLevel: GeneralLogLevel
            ));

            CleanupOldFiles(generalPath, GeneralRetentionDays, "*.txt");
        }

        if (EnableUserActivity)
        {
            string userPath = Path.Combine(logBasePath, "UserActivity");
            Directory.CreateDirectory(userPath);

            config = config.WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                Path.Combine(userPath, "user_activity.json"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: UserActivityRetentionDays,
                restrictedToMinimumLevel: UserActivityLevel
            ));

            CleanupOldFiles(userPath, UserActivityRetentionDays, "*.json");
        }

        if (EnableErrorLog)
        {
            string errorPath = Path.Combine(logBasePath, "Error");
            Directory.CreateDirectory(errorPath);

            config = config.WriteTo.Async(a => a.File(
                Path.Combine(errorPath, "error_log_.txt"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: ErrorRetentionDays,
                restrictedToMinimumLevel: ErrorLogLevel
            ));

            CleanupOldFiles(errorPath, ErrorRetentionDays, "*.txt");
        }

        if (EnableSecurityLog)
        {
            string securityPath = Path.Combine(logBasePath, "Security");
            Directory.CreateDirectory(securityPath);

            config = config.WriteTo.Async(a => a.File(
                new CompactJsonFormatter(),
                Path.Combine(securityPath, "security_event.json"),
                rollingInterval: RollingInterval.Day,
                retainedFileCountLimit: SecurityRetentionDays,
                restrictedToMinimumLevel: SecurityLogLevel
            ));

            CleanupOldFiles(securityPath, SecurityRetentionDays, "*.json");
        }

        Log.Logger = config.CreateLogger();
    }

    private static void CleanupOldFiles(string folderPath, int retentionDays, string searchPattern)
    {
        try
        {
            var files = Directory.GetFiles(folderPath, searchPattern)
                                 .Select(f => new FileInfo(f))
                                 .Where(f => f.CreationTime < DateTime.Now.AddDays(-retentionDays));

            foreach (var file in files)
            {
                file.Delete();
            }
        }
        catch
        {
            // 若清理失敗，避免影響程式啟動，可記錄到一般事件或忽略
        }
    }

    // 一般事件
    public static void Info(string message, params object[] args)
    {
        if (EnableGeneralLog)
            Log.Information(message, args);
    }

    // 使用者操作事件
    public static void UserActivity(string message, params object[] args)
    {
        if (EnableUserActivity)
            Log.ForContext("EventType", "UserActivity").Information(message, args);
    }

    // 錯誤事件
    public static void Error(Exception ex, string message, params object[] args)
    {
        if (EnableErrorLog)
            Log.Error(ex, message, args);
    }

    // 高敏感事件
    public static void Security(string message, params object[] args)
    {
        if (EnableSecurityLog)
            Log.ForContext("EventType", "Security").Information(message, args);
    }

    // 程式結束前呼叫
    public static void Flush()
    {
        Log.CloseAndFlush();
    }
}
```

## 使用範例

```csharp
class Program
{
    static void Main(string[] args)
    {
        // 初始化並指定 Log 存放根目錄
        LogManager.Init("C:\\MyAppLogs");

        LogManager.Info("APP 啟動完成");

        LogManager.UserActivity("登入事件: {User}, IP: {IP}, 時間: {Time}", "UserA", "192.168.1.1", DateTime.Now);

        try
        {
            int a = 5;
            int b = 0;
            int c = a / b;
        }
        catch(Exception ex)
        {
            LogManager.Error(ex, "程式發生錯誤");
        }

        LogManager.Security("權限變更: {Admin}, 變更內容: {Change}, 時間: {Time}", "Admin01", "UserA->Manager", DateTime.Now);

        LogManager.Flush();
    }
}
```

## 特點

- 支援 **非同步寫入 + 快取**，減少高頻事件對效能影響
- 各類別事件可動態啟用/停用
- 各事件可單獨設定 **Log 等級、保留天數、檔案目錄**
- 自動清理超過保留天數的舊檔案
- JSON 與純文字混用，分析與人工查看皆方便
- 可直接套用於生產環境，易於擴充與維護

這個版本已經是一個 **完整、可直接使用的 Serilog 日誌管理方案**。

---

# 放置位置

以 TaskPulse 架構詢問 ChatGPT 的結論：

**Serilog「初始化與 Sink 設定」應放在「程式啟動位置」**
**Core 只保留「日誌使用抽象（ILogger）」與共用慣例，不負責建立 Logger**

這樣的分工最符合你目前的架構與 Autofac 使用方式。

## 為什麼不建議把 Serilog 初始化放在 Core

以你目前的依賴關係來看：

- Core 被所有模組參考
- Core 理論上應是「可重用、無部署語意」的層
- Serilog 的 Sink 設定本質上是「部署與執行環境決策」

若放在 Core，會產生下列問題：

- Core 被迫知道 Console、File、Seq 等輸出目標
- 不同 Host 想用不同 Sink 會變得困難
- Core 會隱含「一定要用 Serilog」的實作依賴
- 未來若換日誌框架，Core 需要一起動

結論一句話：
**Core 不應該知道「日誌寫到哪裡」**

## 建議的責任切分

### 程式啟動專案（Calin.TaskPulse）

負責：

- Serilog LoggerConfiguration
- Sink、Level、Async、環境判斷
- Autofac 與 Serilog 的整合
- 全域 Log.Logger 初始化與關閉

這一層「知道自己是誰、跑在哪、要把 log 丟到哪」

### Core 專案（Calin.TaskPulse.Core）

只負責：

- 使用 `ILogger<T>` 或 Serilog `ILogger`
- 定義共用日誌慣例（Property 名稱、CorrelationId 使用方式）
- 不建立、不初始化 Logger

這一層「只寫 log，不決定 log 怎麼輸出」

## 建議架構示意（語意層級）

- Entity
    - 完全不知道日誌
- Core
    - 依賴 ILogger（抽象）
    - 不含 Sink、不含 LoggerConfiguration
- 各功能模組（MaintiFlow / MechaTrack / ToolQuest）
    - 注入 ILogger
    - 照 Core 定義的慣例寫 log
- 主程式（Calin.TaskPulse）
    - 初始化 Serilog
    - 註冊到 Autofac
    - 決定 Console / File / Seq / Async

## Autofac + Serilog 正確整合位置

### 啟動時建立 Serilog

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

### Autofac 註冊 ILogger

```csharp
builder.RegisterInstance(Log.Logger)
       .As<Serilog.ILogger>()
       .SingleInstance();

builder.RegisterGeneric(typeof(Logger<>))
       .As(typeof(ILogger<>))
       .SingleInstance();
```

### Core 中的使用方式

```csharp
public class TaskService
{
    private readonly ILogger<TaskService> _logger;

    public TaskService(ILogger<TaskService> logger)
    {
        _logger = logger;
    }

    public void Execute()
    {
        _logger.Information("Task started");
    }
}
```

## 什麼情況下「Core 可以碰到 Serilog」

只限以下情況：

- 定義共用 Enricher Key 名稱
    - 例如 CorrelationId、ModuleName
- 提供 LogContext Helper
- 提供 ILogger 擴充方法

但仍然遵守原則：

- 不 new LoggerConfiguration
- 不指定 Sink
- 不讀設定檔

## 實務建議總結

- 初始化 Serilog：程式啟動專案
- 設定 Sink / Level / Async：程式啟動專案
- Core：只依賴 ILogger 抽象
- Autofac：在 Host 層整合 Serilog

這樣你的架構在以下方面會非常乾淨：

- 模組可重用
- 日誌策略可替換
- 部署差異不污染 Core
- 交接與維護成本低
