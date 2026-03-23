---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 重構目標總覽

在**同仁尚未開始使用**的前提下，對 Calin.Logging 進行一次「**簡化但不犧牲未來彈性**」的重構，同時完成各專案對 Logging Infrastructure 的**依賴鬆綁**。
此重構以「**降低理解成本、裝了就能用、未來仍可抽換**」為核心。

# 設計總原則（讓 Copilot 先對齊思考）

- 內部 NuGet 使用者不需要理解抽象、DI、Adapter
- 預設一定有 Logging 行為
- Logging 對 Core 而言是可選的橫切關注點
- 抽象存在，但對大多數人是隱形的
- 不追求多 Logger，只保留「將來可換」的出口

# Calin.Logging vNext 重構方向（高層描述）

- Calin.Logging 仍是**唯一預設 Logging 實作**
- 新增一個**極薄的 Logging 抽象**
- Core / Domain 僅依賴抽象
- App 不做任何設定即可使用 Calin.Logging
- 進階使用者才需要替換

# Copilot 可直接執行的完整 Prompt

## 一、Logging 抽象層（降低複雜度版）

1. 在專案 `Calin.Abstractions` 中定義一個介面 `ICalinLogger`：
    - 僅包含最基本的方法：
        - Debug(string message)
        - Information(string message)
        - Warning(string message)
        - Error(string message, Exception ex = null)
2. 不使用 LogLevel enum
3. 不暴露 Serilog、Microsoft.Extensions.Logging、Autofac 或任何第三方型別
4. 該介面定位為「Core 可安全依賴的最小 API」
5. 提供一個 `NoOpLogger` 實作：
    - 當系統未注入任何 Logger 時，不輸出任何 Log
    - 僅作內部保護，不對使用者顯示或要求設定

## 二、Calin.Logging（Infrastructure）調整

1. 保留 `Calin.Logging` 現有所有功能與行為：
    - LoggingBootstrapper
    - LoggingModule
    - Category、Filter、Sink、LogContextExtensions
2. 在 `Calin.Logging` 中新增一個 `CalinLogger`（Adapter 類別）：
    - 實作 `ICalinLogger`
    - 內部委派給現有的 `Serilog.ILogger`
    - 不改變任何日誌分類、路由、輸出格式
3. `Calin.Logging` 預設提供 `ICalinLogger` 的實作，不需要使用者額外註冊
4. 為 `LoggingModule` 新增 Autofac 註冊：
    - 註冊 `ICalinLogger` 為 Singleton
    - 默認使用 `CalinLogger`
    - 可選注入：
        - 注入自訂 Logger（ICalinLogger 實作）
        - 或注入 `NoOpLogger`，完全不輸出 Log
    - 使所有注入此介面的專案可直接使用
5. 不要求使用者理解或操作 Logging 抽象

## 三、預設行為與簡化策略

1. 若應用程式未進行任何 Logger 設定：
    - 系統自動使用 `CalinLogger`（內建 Calin.Logging）
    - Logging 行為與現有完全一致
2. Autofac 容器載入後：
    - 可直接解析 `ICalinLogger`
    - 使用者可自由選擇：內建 Logger、自訂 Logger、或 NoOpLogger
3. `NoOpLogger` 僅作內部保護，不對外曝光或干擾普通使用者

## 四、各專案解除依賴

1. Core / Domain / Application 專案：
    - 移除對 `Calin.Logging`、Serilog、MEL 的 reference
    - 改為僅依賴 `Calin.Logging.Abstractions`
2. 將原本注入 `Serilog.ILogger` 或 `ILogger<T>` 的地方：
    - 改為注入 `ICalinLogger`
3. Core 專案不得出現 Logging 初始化、Module 註冊、或第三方 Logger 型別

## 五、App / Composition Root

1. App 啟動流程保持原樣：
    - 仍呼叫 `LoggingBootstrapper.Initialize()`
    - 仍註冊 `LoggingModule`
2. Autofac 容器載入後可直接解析 `ICalinLogger`
3. 不要求新增任何設定或 DI 註冊程式碼
4. 進階使用者可選擇：
    - 替換 `ICalinLogger` 為自訂 Logger
    - 或注入 `NoOpLogger` 達到完全不輸出 Log

## 六、明確禁止事項（避免過度設計）

1. 不引入 Microsoft.Extensions.Logging.Abstractions 到 Core
2. 不強迫使用者理解 Hook / Adapter / Clean Architecture
3. 不為抽象新增額外初始化步驟
4. 不改變 README 中「裝了就能用」的核心體驗

## 七、驗收條件

1. 新專案只引用 Calin.Logging，即可直接輸出日誌
2. Core 專案可獨立編譯，且不依賴任何 Logging Infrastructure
3. Logging 行為、分類、檔案結構與目前設計完全一致
4. Autofac 載入後，專案可自由選擇：
    - 使用內建 `CalinLogger`
    - 注入自訂 `ICalinLogger`
    - 注入 `NoOpLogger` 完全不輸出
5. 抽象層對 90% 使用者完全不可見

# 這版 Prompt 的定位

- **不是**為了多 Logger
- **不是**為了展示架構技巧
- 是為了：
    - 讓今天好用
    - 讓明天不被綁死

# 最後的架構師判斷

在目前這個時間點做這個版本的重構：

- 幾乎沒有組織阻力
- 幾乎沒有學習成本
- 卻替未來保留關鍵彈性

這是「內部 Infrastructure 套件」最理想的演進節奏。

# Copilot-ready PROMPT

請幫我重構 Calin.Logging 專案與相關專案，依照以下規則生成程式碼：

1. **Logging 抽象層（Calin.Abstractions）**
    - 定義介面 `ICalinLogger`：
        - Debug(string message)
        - Information(string message)
        - Warning(string message)
        - Error(string message, Exception ex = null)
    - 不使用 LogLevel enum
    - 不暴露 Serilog、MEL、Autofac 或其他第三方型別
    - 提供 `NoOpLogger` 實作：
        - 完全不輸出任何 Log
        - 僅作內部保護，不對使用者顯示

2. **Calin.Logging（Infrastructure）**
    - 保留現有所有功能與行為：
        - LoggingBootstrapper
        - LoggingModule
        - Category、Filter、Sink、LogContextExtensions
    - 新增 `CalinLogger` 類別：
        - 實作 `ICalinLogger`
        - 內部委派給現有 `Serilog.ILogger`
        - 保留現有分類、路由、輸出格式
    - `LoggingModule` Autofac 註冊：
        - 註冊 `ICalinLogger` 為 Singleton
        - 默認使用 `CalinLogger`
        - 可選注入自訂 `ICalinLogger` 或 `NoOpLogger`
    - 不要求使用者理解 Logging 抽象或額外初始化

3. **Core / Domain / Application 專案**
    - 移除對 `Calin.Logging`、Serilog、MEL 的 reference
    - 僅依賴 `Calin.Logging.Abstractions`
    - 注入 `ICalinLogger`，不得出現 Logging 初始化、Module 註冊或第三方 Logger 型別

4. **App / Composition Root**
    - 呼叫 `LoggingBootstrapper.Initialize()`
    - 註冊 `LoggingModule`
    - Autofac 容器載入後：
        - 可直接解析 `ICalinLogger`
        - 可選擇使用內建 `CalinLogger`、自訂 Logger、或 `NoOpLogger`

5. **驗收條件**
    - 新專案只引用 Calin.Logging 即可直接輸出日誌
    - Core 專案可獨立編譯且不依賴 Logging Infrastructure
    - Logging 行為、分類、檔案結構與現有完全一致
    - 抽象層對一般使用者不可見
    - Autofac 載入後可自由選擇 Logger 實作或不輸出

---

# Calin.Logging 重構與解除直接依賴的 Copilot PROMPT（低複雜度版本）

以下 PROMPT 設計目標：
降低心智負擔
不強迫導入複雜抽象或框架
符合大型專案常見實務，但保留「內部 NuGet 易用性」
未注入 Logger 時，系統自然靜默，不做任何輸出

## PROMPT 1：重構 Calin.Logging（核心庫）

用途：
請 GitHub Copilot 協助你重構 Calin.Logging，使其成為「可選 Logger Hook」而非強制實作

PROMPT 內容：

```csharp
請重構 Calin.Logging 專案，目標如下：

1. Calin.Logging 不再直接依賴任何具體 Logging Framework（例如 Serilog、NLog、Microsoft.Extensions.Logging）。
2. 在 Calin.Logging 中定義一個極簡的 Logger 抽象介面（例如 ILoggingHook 或 IAppLogger），僅包含必要的方法（如 Debug / Info / Warn / Error）。
3. 提供一個預設的 NoOpLogger（或 NullLogger）實作：
    - 當系統未注入任何 Logger 時，所有 Log 呼叫都不產生任何輸出
    - 不丟例外、不寫 Console、不寫檔
4. Calin.Logging 對外僅暴露一個靜態或單例入口（例如 Logging.Current 或 Logging.Hook）：
    - 預設值為 NoOpLogger
    - 允許應用程式在啟動時自行指定實際 Logger
5. Calin.Logging 本身不得引用 DI Container（如 Autofac、Microsoft DI）
6. 確保現有 Calin.Logging 的公開 API 變動最小，避免影響尚未使用的專案
7. 加入簡單的 XML 註解，說明：
    - 此 Logging Hook 為「可選」
    - 若未設定 Logger，系統將自動靜默

請優先考量可讀性、低學習成本、內部 NuGet 使用情境
```

## PROMPT 2：調整現有專案，解除對 Calin.Logging.csproj 的直接依賴

用途：
讓 Copilot 協助你在各應用層專案中，改為「可選接入 Logger」

PROMPT 內容：

```csharp
請調整目前專案中對 Calin.Logging 的使用方式，目標如下：

1. 移除專案對 Calin.Logging.csproj 的直接實作耦合假設
2. 專案僅在啟動階段（如 Program.cs / Bootstrapper）決定是否要設定 Logger
3. 若專案未設定任何 Logger：
    - 不需要額外設定
    - 系統應保持靜默，不輸出 Log
4. 若專案有使用既有 Logging Framework（如 Serilog）：
    - 僅在啟動時建立 Adapter，實作 Calin.Logging 定義的 Logger Hook
    - 不允許在業務邏輯中直接引用具體 Logging Framework
5. 確保修改後：
    - 既有功能行為不變
    - 未使用 Logging 的專案無需新增任何設定或程式碼

請避免引入過度設計（如多層抽象、Factory、Service Locator）
```

## PROMPT 3：提供給「不在同一方案內」專案的修改指引（對外溝通用）

用途：
這一段是你可以直接丟給 Copilot，產出 README 或內部技術說明文件

PROMPT 內容：

```csharp
請產出一份「Calin.Logging 使用與遷移說明」，讀者為公司內部開發同仁，內容需包含：

1. 為什麼 Calin.Logging 改為可選 Logger Hook（背景與好處，避免技術術語）
2. 舊專案若沒有使用 Logging，需要做什麼？
    - 答案應為：不需要做任何事
3. 舊專案若有使用 Logging Framework，需要做什麼？
    - 僅需在啟動時設定一次 Logger Hook
4. 提供一個最小可用範例（Pseudo Code 即可）：
    - 設定 Logger
    - 未設定 Logger 的情境
5. 明確說明：
    - Calin.Logging 不會強制輸出 Log
    - 不會因未設定 Logger 而拋出例外

請使用內部技術文件常見語氣，避免框架推銷感
```

## 為什麼「現在改」是好時機（給你決策用）

重構成本最低的時點：
尚未被大量專案引用
尚未形成「大家以為一定要用某 Logger」的錯誤共識

這個版本的設計：
符合大型專案常見模式（Optional Infrastructure）
但對同仁來說：
不用學 DI
不用懂 Adapter Pattern
不用理解 Logging 架構史

一句話總結給同仁：
「你要 Log，就在啟動時接一個；你不要 Log，就什麼都不用做。」

如果你需要：
下一步我可以直接幫你「設計 Calin.Logging 的最小介面範例」或「幫你把 PROMPT 再縮成一段給 Copilot Chat 用」。
