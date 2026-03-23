---
aliases:
created: 
update:
author:
language:
sourceurl:
tags:
date:
---

# 比較

## 結論先行

在 C### 應用程式中使用 Serilog，**核心 Logging 能力應模組化（共用）**，但**實際設定與行為應由各 APP 個別決定**。這是目前企業級與多 App 解決方案中最穩定、可維護性最高的做法。

## 為什麼不應完全「每個 APP 各做一套」

- 重複實作 Sink、Enricher、Formatter，維護成本高
- 設定風格不一致，Log 可讀性與分析困難
- 修正一個 Logging Bug，需要同步修改多個專案
- 無法形成一致的 Logging 規範

## 為什麼不應完全「做成固定模組」

- 各 APP 的需求不同
    - 桌面程式 vs Service vs Console
    - 有無檔案、資料庫、Seq、Elastic
    - Log 保留天數、容量限制不同
- APP 層最清楚「哪些 Log 是重要的」
- 過度封裝會導致：
    - 設定彈性不足
    - Debug 時反而難追蹤

## 建議的最佳實務分層

- Logging.Core（共用模組 / NuGet）
    - Serilog 基礎包裝
    - 共用 Enricher
    - 共用 OutputTemplate
    - 共用 Extension Methods
- Logging.Integration（選擇性）
    - 與 Autofac / Microsoft DI 整合
    - 與 WinForm / WPF / ASP.NET Hosting 整合
- App 專層（每個 APP 自己決定）
    - Sink 組合（File / Console / Seq / DB）
    - 最低 Log Level
    - 設定來源（appsettings.json / XML / Code）
    - 檔名規則、保留策略

## 模組化時「應該放」與「不該放」的東西

- 應該放在共用模組
    - IAppLogger / ILogger Wrapper
    - 共用 Enricher（MachineName、ProcessId、ThreadId）
    - 共用 Exception 格式化策略
    - LogContext Extension
- 不該放在共用模組
    - 實際 Sink 設定值（路徑、連線字串）
    - 最終 MinimumLevel
    - APP 特有的 Log Category

## 典型結構範例

- Calin.Logging
- Calin.Logging.Serilog
- Calin.Logging.Serilog.Autofac
- AppA
- AppB

### 實務上的一句話原則

- Logging 是 Infrastructure，不是 Business
- **能力集中、決策下放**
- 模組提供「怎麼做」，APP 決定「做多少、做到哪」

### 什麼情況下可以「每個 APP 都自己來」

- 只有一個 App
- 專案壽命短
- PoC / Prototype
- 沒有 Log 聚合與統一分析需求

### 延伸建議

若你已經使用 Autofac，建議將 Logging Module 明確獨立，並讓 APP 在 Composition Root 明確註冊，避免隱性初始化與順序不明的問題。

---

# 無法有效記錄

## 結論建議

**建議重做，但不是推倒重來，而是「收斂責任、重新定義邊界」後重做一個可控的 Logging 架構。**
你現在遇到的狀況，典型不是 Serilog 本身的問題，而是早期 Logging 設計「過度抽象或初始化不可控」所造成的系統性失效。

## 從你描述推斷的關鍵問題徵象

- 多個 NuGet 依賴同一套 Logging，但實際上「沒寫進去」
- Log 在某些 App 有效、某些 App 無效
- Debug 時看不到 Log 初始化點
- 不確定 Logger 是不是被正確建立、何時建立

這通常代表以下幾件事之一（或同時存在）：

- Logger 初始化被藏在 Library 內
- 多次初始化導致 Logger 被覆蓋或關閉
- NuGet 在載入時就呼叫 Logger，但 Host 尚未建立
- 使用 static logger，卻沒有明確生命週期

## 是否「一定要重做」的判斷標準

只要符合下列任一點，就**值得重做**：

- 無法在單一位置明確看到「Logger 是怎麼被建起來的」
- NuGet 內部可以自行初始化 Logging
- 無法用最小範例驗證 Logging 是否一定會輸出
- 無法在 App 啟動初期（Main / Program）確保 Logging 已 ready

依你描述，「一直無法有效記錄」其實已經達到必須重構的門檻。

## 建議的重做策略（避免第二次失敗）

### 1. 明確定義 Logging 的唯一初始化點

- 只能在 App 的 Composition Root 初始化
- 不允許任何 NuGet / Library 呼叫 `Log.Logger = ...`

原則：

- Library 只能「使用 Logger」
- App 才能「建立 Logger」

### 2. 降低抽象層數，不再包過頭

- 不再自製完整 Logger Facade
- 直接依賴 `Serilog.ILogger` 或 `Microsoft.Extensions.Logging.ILogger`
- Wrapper 只負責補齊 Context，不重新發明 API

### 3. 重做時的模組責任切分

- Calin.Logging.Abstractions
    - 少量介面（若真的需要）
- Calin.Logging.Serilog
    - Serilog 註冊 Helper
    - 共用 Enricher
- 不提供任何「自動啟動」行為

### 4. 強制「失敗即爆」

- 初始化失敗直接丟 Exception
- 禁止 Silent Fail
- 啟動時一定要能寫出一筆「Logging initialized」

## 一個健康的最小驗證標準

每個 App 在 Main/Program 中，至少要看到以下三行流程：

- 建立 LoggerConfiguration
- 設定 Sink
- 寫入 Startup Log（且你實際看得到）

如果任何一個 App 做不到這三步，表示 Logging 架構仍然有問題。

## 什麼情況下「不需要重做」

- 問題只出在單一 App 設定錯誤
- 已能定位是 Sink 或權限問題
- Logging 初始化流程清楚可追

但依你的描述，這些條件目前大多不成立。

## 務實建議

不要試圖「修舊架構直到它變好」。
Logging 是 Infrastructure 的地基，一旦不穩，後續所有觀測、除錯、營運都會失效。

---

# NuGet

## 核心立場先講清楚

**NuGet（設備模組）一定可以「寫 Log」，但絕對不應該「決定 Log 寫到哪裡、怎麼寫」。**
這是你目前卡住、也最容易再踩雷的關鍵分界。

## 正確的責任切分（非常重要）

- App（Composition Root）
    - 建立 Logger
    - 決定 Sink、Level、格式、保存策略
    - 決定是否啟用某些設備的詳細 Log
- NuGet（設備控制模組）
    - 只負責「發出 Log 事件」
    - 不初始化、不設定、不關閉 Logger
    - 不知道 Log 最終去哪裡

一句話總結：
**NuGet 有記錄責任，但沒有 Logging 決策權。**

## NuGet 為什麼「一定要能 Log」

你的情境其實非常合理：

- 設備異常通常發生在模組內部
- 錯誤若只丟 Exception，現場難以重現
- 設備狀態、通訊封包、回應碼，都屬於模組最清楚的資訊

因此 NuGet 內「不 Log 反而是設計缺陷」。

## 正確讓 NuGet 記錄 Log 的三種模式

### 模式一：建構式注入 ILogger（最推薦）

- NuGet 類別透過 DI 取得 `ILogger`
- Logger 由 App 提供
- NuGet 完全不關心 Serilog 是否存在

特點：

- 最乾淨
- 可測試
- 不會有初始化順序問題

### 模式二：ILoggerFactory 注入（設備多實例常用）

- App 提供 `ILoggerFactory`
- NuGet 自行建立 Category Logger

適合：

- 多設備、多通道
- 需要動態分類（DeviceId、PortName）

### 模式三：Fallback Null Logger（防呆用）

- NuGet 預設使用 NullLogger
- App 若有註冊 Logging，自然會被替換

用途：

- 避免未初始化時噴 NullReference
- 絕不主動建立 Logger

## NuGet 內「可以做」與「禁止做」

- 可以做
    - 記錄設備狀態、命令、回應
    - 記錄 Retry、Timeout、CRC Error
    - 在 Catch 區塊完整寫 Exception
- 禁止做
    - `Log.Logger = new LoggerConfiguration()`
    - 決定 Sink（File、Seq、DB）
    - 在 static constructor 初始化 Logging
    - 讀 appsettings.json

## 為什麼你以前的 Logging「會記不起來」

以你描述的狀況，幾乎可以確定踩到以下其中一項：

- NuGet 在載入時就初始化 Logger，App 尚未 ready
- 多個 NuGet 各自初始化，後初始化者覆蓋前者
- static Logger 被 CloseAndFlush 過
- Logger 生命週期與設備物件不同步

這些都會造成「看起來有寫 Log，但實際沒落地」。

## 建議的設備型 NuGet Logging 標準寫法（概念）

- 設備模組：
    - 只依賴 `ILogger` 或抽象介面
    - Category 以「設備類型」或「設備名稱」
- App：
    - 統一設定 Sink
    - 用 Enricher 補 DeviceId、Port、Station
    - 決定哪些設備開 Debug / Verbose

結果是：

- 設備模組可重用
- Log 集中、可分析
- 行為可控、不失序

## 關鍵設計原則（請記這三句）

- Library 只能寫 Log，不能管 Log
- Logger 只能由 App 建立一次
- 問題不是「NuGet 要不要 Log」，而是「誰有權初始化」

## 下一步我可以幫你做的事

- 幫你定義「設備型 NuGet 的 Logging 規範」
- 幫你檢視舊 NuGet 哪裡違反初始化原則
- 幫你設計一個「第二代 Logging + 設備模組」的標準範本
- 幫你產出一份重構用的 Copilot Prompt

如果你願意，下一步可以先挑**一個最常出問題的設備 NuGet**，我們從它開始解剖，會最快看到成效。

---

# GitHub Copilot PROMPT

以下為**完整最終版 PROMPT**，已納入：

- 多 APP 共用
- 多設備型 NuGet（不硬性規定命名方式）
- Serilog 預設但可替換
- Autofac 與非 DI（new）並存
- Device.* 僅示範，不實作實際設備
- 明確禁止常見踩雷行為

---

你是一位資深 .NET 架構師，請協助我設計並產出一套 **工控 LEVEL 5 Logging 架構**，可同時被多個 APP 與多個「設備控制型 NuGet」安全共用。

## 核心限制（非常重要）

- **整個 Logging 架構只允許一個專案**
- 專案名稱固定為：`Calin.Logging`
- **只產生一個 `Calin.Logging.csproj`**
- 不得建立或假設存在：
    - Calin.Logging.Abstractions.csproj
    - Calin.Logging.Serilog.csproj
    - Calin.Logging.Integration.Autofac.csproj
- 必須透過 **Namespace、資料夾結構、組件內分層** 來區隔責任，而非多個 csproj

## 背景與使用情境

- 技術平台：
    - .NET Framework 4.8
    - 使用情境：WPF / WinForm / Console
- 專案位置：`Calin.Logging`
- Logging 屬於 Infrastructure
- 公司內有多個 APP
- 公司內有多個「設備控制 NuGet」
    - NuGet 命名以「設備型號 / 品牌 / 類型」為主
    - 每個 NuGet 負責實際設備通訊與控制
    - 設備操作與異常必須能記錄 Log
- 主要 DI 容器：Autofac
- 部分同事不使用 DI，會直接 `new` 物件
- 預設使用 Serilog，但必須 **可替換為其他 Logging 實作**
- Logging 初始化權限 **只能在 App**

## 核心設計原則（必須嚴格遵守）

- Logging 能力集中
- Logging 決策下放
- App 是唯一可以初始化 Logger 的地方
- Library / NuGet 只能寫 Log，不能管 Log
- **專案內部必須明確區分：抽象層 / Framework 實作 / DI 整合，但仍在同一 csproj**

## 絕對禁止事項（請在設計中主動避免）

- Library / NuGet 不可：
    - 建立 LoggerConfiguration
    - 設定 Sink、MinimumLevel
    - 讀取 appsettings.json
    - 使用 static constructor 初始化 Logger
    - 直接依賴 Serilog / NLog 套件
    - 偷用 Autofac Container 或 Service Locator

## 單一專案內部結構規劃（必須遵守）

`Calin.Logging` 專案內請使用以下資料夾與 Namespace 分層：

- Abstractions
    - Logging 最小必要抽象
    - 不依賴任何具體 Logging Framework
    - 設計可被所有 APP 與設備 NuGet 參考
- Serilog
    - Serilog 專用實作
    - 僅此區域可以引用 Serilog 套件
    - 包含：
        - Enricher
        - Formatter
        - LoggerFactory Adapter
        - Extension Methods
- Integration
    - Autofac 整合
    - 僅提供 Module / Extension
    - 不做隱性初始化
- Internal
    - Logger Bridge / Accessor
    - Null Logger / No-Op Logger
    - 僅供 Logging 基礎設施使用
- Samples
    - Device.Sample
    - 僅示範用途，不代表實際設備實作

## 關於 Device.Sample 的明確說明（必須體現於產出）

- Device.Sample 是角色示範，不是實際設備
- 真實設備 NuGet 將以設備型號命名
- Sample 僅示範：
    - 如何取得 Logger
    - 如何寫 Log
    - 如何在有 / 無 DI 情境下運作
- Sample 不實作任何真實設備通訊

## Logging 抽象設計要求

- 優先使用 `Microsoft.Extensions.Logging.ILogger`
- 如需自訂介面，需：
    - 極簡
    - 與 ILogger 語意一致
- Logger 必須支援：
    - Category
    - Scope / Context
    - Exception

## 同時支援 DI 與非 DI（非常重要）

## DI 使用情境

- 透過 Autofac 注入 `ILogger<T>` 或 `ILoggerFactory`
- 由 App 統一建立 LoggerFactory
- Calin.Logging 僅提供註冊輔助，不負責初始化

### 非 DI 使用情境

- 允許 `new DeviceController()` 直接使用
- 未注入 Logger 時：
    - 使用 Null Logger / No-Op Logger
    - 不丟例外、不影響流程
- 設計一個明確的 Logger Bridge / Accessor：
    - 預設為 Null Logger
    - 只能由 App 初始化後設定
    - Device NuGet 不可設定

## Enricher 與 Context 策略

- Enricher 僅能存在於 `Serilog` 區域
- DeviceId、PortName、Station 等資訊：
    - 由設備模組透過 Scope / Context 傳遞
    - 不由設備模組建立 Enricher
- 是否啟用 Enricher 由 App 決定

## 初始化與生命週期

- Logger 初始化只能發生在 App 的 Composition Root
- 初始化失敗必須明確丟出例外
- 初始化完成後必須寫出一筆 Startup Log
- 禁止 Silent Fail

## 請產出以下內容

- 單一 `Calin.Logging.csproj` 內的：
    - 資料夾結構
    - Namespace 規劃
- 各責任區塊的角色說明
- 關鍵介面與類別設計
- Autofac 註冊範例
- App 使用 Serilog 的初始化範例
- Device.Sample：
    - DI 使用範例
    - 非 DI（new）使用範例
- 清楚標示：
    - 哪些程式碼只能存在於 App
    - 哪些程式碼可存在於設備 NuGet
- README，圖表使用 Mermaid 格式

## 設計品質要求

- 可實際編譯、可落地
- 架構清楚、命名一致
- 不過度抽象、不魔法化
- 有利於單元測試
- 未來可替換 Logging Framework 而不影響設備 NuGet

---

# 架構補充說明（給你看的，不用貼給 Copilot）

這份 PROMPT 的關鍵價值在於：

- **不懂 DI 的同事不會破壞架構**
- 懂 DI 的人不會被限制
- Logging 初始化權限仍然集中在 App
- 不會再發生「NuGet 各自初始化導致 Log 消失」

如果你接下來想做得更嚴謹，我會建議再補一個：
-「禁止設備 NuGet 使用 Serilog 套件的檢查清單」

這會讓整個團隊的 Logging 使用方式開始一致、可控。

---

# GitHub Copilot PROMPT

你是一位資深 .NET 架構師，請協助我設計並產出一套 **工控 LEVEL 5 Logging NUGET 庫**，給所有 NUGET、LIB、DRIVER、APP 安全共用。

專案名稱：`Calin.Logging`

## 背景與使用情境

- 技術平台：
    - .NET Framework 4.8
    - 使用情境：WPF / WinForm / Console
- 專案位置：`Calin.Logging`
- Logging 屬於 Infrastructure
- 公司內有多個 APP
- 公司內有多個「設備控制 NuGet」
    - NuGet 命名以「設備型號 / 品牌 / 類型」為主
    - 每個 NuGet 負責實際設備通訊與控制
    - 設備操作與異常必須能記錄 Log
- 主要 DI 容器：Autofac
- DI 與非 DI 支援
- DI 預設使用 Serilog，但必須 **可替換為其他 Logging 實作**
- Logging 初始化權限 **只能在 App**

## 核心設計原則（必須嚴格遵守）

- Logging 能力集中
- Logging 決策下放
- App 是唯一可以初始化 Logger 的地方
- Library / NuGet 只能寫 Log，不能管 Log

## 絕對禁止事項（請在設計中主動避免）

- Library / NuGet 不可：
    - 建立 LoggerConfiguration
    - 設定 Sink、MinimumLevel
    - 讀取 appsettings.json
    - 使用 static constructor 初始化 Logger
    - 直接依賴 Serilog / NLog 套件
    - 偷用 Autofac Container 或 Service Locator

## Logging 抽象設計要求

- 優先使用 `Microsoft.Extensions.Logging.ILogger`
- 如需自訂介面，需：
    - 極簡
    - 與 ILogger 語意一致
- Logger 必須支援：
    - Category
    - Scope / Context
- Exception- 高效能寫法（強制）：
    - 使用 `LoggerMessage.Define` 預編譯 logging
    - 避免 boxing / template parsing / allocation

## 同時支援 DI 與非 DI（非常重要）

### DI 使用情境

- 透過 Autofac 注入 `ILogger<T>` 或 `ILoggerFactory`
- 由 App 統一建立 LoggerFactory
- Calin.Logging 僅提供註冊輔助，不負責初始化

## Enricher 與 Context 策略

- Enricher 僅能存在於 `Serilog` 區域
- DeviceId、PortName、Station 等資訊：
    - 由設備模組透過 Scope / Context 傳遞
    - 不由設備模組建立 Enricher
- 是否啟用 Enricher 由 App 決定

## 初始化與生命週期

- Logger 初始化只能發生在 App 的 Composition Root
- 初始化失敗必須明確丟出例外
- 初始化完成後必須寫出一筆 Startup Log
- 禁止 Silent Fail

## 請產出以下內容

- 單一 `Calin.Logging.csproj` 內的：
    - 資料夾結構
    - Namespace 規劃
- 各責任區塊的角色說明
- 關鍵介面與類別設計
- Autofac 註冊範例
- App 使用 Serilog 的初始化範例
- Device.Sample：
    - DI 使用範例
    - 非 DI（new）使用範例
- 清楚標示：
    - 哪些程式碼只能存在於 App
    - 哪些程式碼可存在於設備 NuGet
- README，圖表使用 Mermaid 格式

## 設計品質要求

- 可實際編譯、可落地
- 架構清楚、命名一致
- 不過度抽象、不魔法化
- 有利於單元測試
- 未來可替換 Logging Framework 而不影響設備 NuGet
