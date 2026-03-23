---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - Prompt
---

這裡列出用過的 GitHub Copilot 提示詞範本，方便未來參考與使用。

```text
- 產生 README.md 檔，內容包含簡介、架構、使用說明、使用範例及其他相關事項
是舊的，準備要刪除，保留只是為了編譯能PASS，請勿做為參考。
```

---

Calin.TaskPulse.Entity 的 Prompt 依序整段複製使用。每一段都是獨立 PROMPT，請勿合併一次送出。
這些是最早的 Prompt，所以還保留了 Entity 名稱，沒有泛用化。

# Calin.TaskPulse.Entity 第一段：資料模型層（Entity / Fluent API）

## 整體架構與責任劃分

- 這是一個 .NET Framework 4.6.2 的 WinForm/WPF 桌面程式
- 使用 SQLite + EF6 + Code First
- 連接字串定義於 Calin.TaskPulse.App.config

## DbContext 定義如下

- CoreContext
- Calin.TaskPulse.Entity.Core.EmployeeEntity
- Calin.TaskPulse.Entity.Core.MachineEntity
- Calin.TaskPulse.Entity.Core.ModelEntity
- MaintiFlowContext
- Calin.TaskPulse.Entity.MaintiFlow.WorkOrderEntity

## 各 Entity 之間的關聯已寫在各 Entity 檔案開頭註解中

- 嚴格依照註解實作
- 不自行推論未註明關聯或鍵值

## SQLite 與 EF6 使用限制

- 不使用隱式多對多中介表
- 所有多對多關係必須明確定義中介 Entity
- 不使用 Cascade Delete
- 外鍵關係需明確指定

## 請產生

- Entity 類別重新排版與優化，可修改註解使其更合理清楚
- Fluent API 關聯設定
- 必要索引與 Constraint
- IEquatable 實作（以主鍵作為等值判斷，覆寫 Equals 與 GetHashCode）

## Fluent API 與關聯設定類別

- 放置於 namespace Calin.TaskPulse.Core.DB
- 僅包含結構與 Constraint
- 不包含商業邏輯
- 儘量使用 Fluent API 完成設定，避免使用 Data Annotation，並刪除 Entity 上重覆的 Attribute

## 請勿產生

- Repository、DTO、Service、ViewModel、UI 程式碼
- Autofac 註冊程式碼
- Serilog 設定或日誌程式碼

# Calin.TaskPulse.Entity 第二段：資料存取與流程層（Repository / DTO / Service / Autofac）

## 以下 Entity 與 Fluent API 已確定，請勿修改

- EmployeeEntity、MachineEntity、ModelEntity、WorkOrderEntity、所有中介 Entity

## 請在 namespace Calin.TaskPulse.Core.DB 下產生

- Repository Pattern（每個 Entity 專屬 Repository）
- Service 層
- DTO
- Mapper（手寫，不使用 AutoMapper）
- Autofac Module（註冊 Repository、Service、ILogger\<T>）
- Serilog 設定範例（最小日誌等級、Sink 範例）

## Repository 規範

- CRUD 與查詢
- 回傳 IEnumerable，不暴露 IQueryable
- 不包含交易與流程控制
- 建構子可注入 DbContext 與 ILogger
- 日誌：新增、更新、刪除成功或失敗都記錄

## DTO 規範

- 僅供 Service 與 ViewModel 使用
- 命名後綴 Dto
- 不包含 EF Attribute

## Mapper 規範

- 集中管理
- 將 Entity 與 DTO 相互轉換
- 不散落在 Service 或 ViewModel

## Service 層

- 協調多個 Repository
- 管理 DbContext 生命週期
- 處理交易與流程
- 提供唯一資料入口給 ViewModel
- 建構子可注入 ILogger
- 日誌：交易開始、完成、例外捕捉

## Autofac Module 規範

- 檔案名稱：Calin.TaskPulse.Core.CoreModule.cs
- Repository: InstancePerDependency
- Service: InstancePerLifetimeScope
- Logger: InstancePerDependency（或預設 ILogger\<T>）
- 生成範例 Module 類別供專案直接註冊
- ViewModel 依 Service 注入，僅建構子注入

## 請勿產生

- WinForm/WPF 控制項
- 直接操作 DbContext 的 UI 程式碼

# Calin.TaskPulse.Entity 第三段：ViewModel 與 WinForm DataBinding（Autofac 注入）

## 以下 Service 與 DTO 已存在

- 所有 TaskPulse Core Service
- 對應 DTO

## 請生成 ViewModel

- 放置於 namespace Calin.TaskPulse.Core.ViewModels
- 一個 ViewModel 可供多個 View 共用
- 建構子透過 Autofac 注入 Service 與 ILogger

## ViewModel 規範

- 實作 INotifyPropertyChanged
- 支援 WinForm DataBinding
- 使用 BindingSource
- 不直接參考 Entity 或 DbContext

## DataBinding 要求

- 屬性變更正確通知
- 清單型資料支援新增、刪除、重新整理
- Service 呼叫後更新狀態
- 日誌：狀態更新與操作記錄

## 示範

- 單一物件 Binding
- 清單資料 Binding
- Service 呼叫後更新 ViewModel 的流程

## 請勿產生

- Form Designer
- 實際 UI 佈局
- 直接 new Service 或 Repository

---

2026/01/10
第二版，主要做

- 高度通用、可 NuGet 化、語意解耦，不依賴實作細節而仍然嚴謹。
- Sink 規劃、Category、Filter、初始化流程完整
- LoggingModule 與 LoggingBootstrapper 初始化順序策略
- Autofac 注入安全，不依賴 Module 註冊順序

# 日誌服務 PROMPT

## 請生成一個「Logging Infrastructure」模組（Serilog 為內建實作）

## 目標環境

- 本 Logging 模組將部署於 **WinForm/WPF 桌面應用程式**
- 說明：
    - Logging Infrastructure 本身 **不依賴 WinForm/WPF 或任何 UI 元件**
    - 設計需兼容 **.NET Framework 4.6.2** 桌面環境
    - 可跨模組、跨層重用，不受 UI 框架限制

## 專案前提

- DI 容器：Autofac
- 日誌內建實作：Serilog
- Logging 核心程式碼必須集中於命名空間 `Calin.Infrastructure.Logging`
- 應用程式啟動專案不得直接建立 `LoggerConfiguration`
- 所有 Serilog 設定責任必須由 Logging Infrastructure 統一管理
- 本模組不依賴 UI 或任何特定 Host

## 設計總原則（必須遵守）

- 本模組定位為「Logging Infrastructure」，不是單純的 Serilog 封裝
- 以 Serilog 為唯一內建實作，不要求支援多種 Logger
- 設計上必須避免將 Serilog 設定責任擴散至應用層或業務模組
- 使用端只關心可注入、可使用的 Logger，不關心其建立細節
- 禁止過度抽象，不得自行定義替代 ILogger 的封裝介面

## 架構與責任劃分

### 核心元件

- **LoggingBootstrapper**
    - 統一初始化 Logging Infrastructure
    - 建立與設定 Serilog Logger
    - 對外只暴露單一初始化入口
    - 設計為 **Thread-safe、可重入**
- **LoggingModule（Autofac Module）**
    - 註冊所有 Logging 相關服務（`ILogger<T>`、`ILoggerFactory`）
    - 不包含 Sink、Filter、路徑或 Rolling 策略
    - 不依賴 Module 註冊順序，初始化安全
- **LogCategories**
    - 定義穩定跨模組共用的日誌分類契約
- 內部輔助類別
    - Sink 組裝、Filter 判斷、Pipeline 組合
    - 僅供 Logging Infrastructure 內部使用

### 初始化與使用流程規範

1. **App 啟動時**：
    - 先呼叫 `LoggingBootstrapper.Initialize()`，確保 Logger 實體先建立
2. **Module 註冊時**：
    - LoggingModule 註冊 `ILogger<T>`、`ILoggerFactory` 到 DI 容器
3. **其他模組注入 ILogger**：
    - 可以安全使用，無需擔心初始化順序
4. **設計要求**：
    - 初始化行為可重入
    - 只產生一次有效設定
    - 行為明確且可預期

## 日誌等級策略

- Global MinimumLevel：Debug
- Override：
    - Microsoft → Warning
    - System → Warning

## 日誌分類（Category）

### 定義規範

- 靜態類別 `LogCategories`
- 使用 `public const string`
- Category 為穩定 API 與日誌路由契約，不得隨意變更
- 至少包含：
    - UserActivity
    - Database
    - Business
    - System

### 使用規範

- Category 必須透過 `LogContext.PushProperty("Category", LogCategories.Xxx)` 設定
- 嚴禁直接使用字串指定 Category
- 嚴禁將分類資訊寫入 Message
- Category 僅用於 Sink Routing，不得作為模組名稱或技術判斷

### 語意定義

- UserActivity：使用者登入、登出、權限變更、關鍵操作與其結果
- Database：資料存取、交易、連線、效能量測與錯誤
- Business：業務流程狀態、規則判斷與決策結果
- System：系統啟動、關閉、背景服務與生命週期事件

## Sink 設計原則

- Sink 代表責任類型，不代表功能模組
- Category 僅負責路由，不代表 Namespace
- Text Sink：人工閱讀與除錯
- JSON Sink：系統查詢、分析、稽核

### Sink 與行為規範

- **一般資訊事件**：Debug 以上，Text，排除 UserActivity/Database，logs/information.txt，每日一檔，保留 15 天
- **錯誤與例外事件**：Error 以上，Text，logs/error_log_.txt，每日一檔，保留 90 天
- **使用者操作事件**：UserActivity，Compact JSON，logs/user_activity.json，每日一檔，保留 90 天
- **資料庫事件**：Database，Compact JSON，logs/database_event.json，每日一檔，保留 90 天
- **系統生命週期事件**：System，Text，logs/system_lifecycle.txt，每日一檔，保留 30 天
- **致命錯誤事件**：Fatal，Text，logs/fatal.txt，每日一檔，保留 180 天

## Filter 實作要求

- Filter 僅依據 Category 與 Level 判斷
- 不解析 Message
- 判斷 Category 時：
    - 不得使用 `ToString()` 比對
    - 必須正確處理 `ScalarValue`
- Filter 邏輯集中於 Logging Infrastructure 內部

## Autofac 整合需求

- 註冊：
    - `Serilog.ILogger`（SingleInstance）
    - `Microsoft.Extensions.Logging.ILoggerFactory`
    - `Microsoft.Extensions.Logging.ILogger<T>`
- 使用 `SerilogLoggerProvider`
- Logger 不隨 Autofac Container Dispose
- LoggingModule 不包含 Serilog 設定細節

## 程式碼與輸出要求

- 完全相容 .NET Framework 4.6.2
- 命名清楚、責任單一、可維護、可擴充
- 不使用過度抽象或多餘設計模式
- 需提供：
    - 命名空間與類別結構
    - 各核心類別責任描述
    - 初始化呼叫順序說明
- 程式碼可直接編譯
- 不得包含 TODO 或假設性實作

---

# 導航服務 PROMPT

請生成頁面導航的服務模組，包含以下內容：

## 專案與技術前提

- 專案類型為 WinForm/WPF 桌面應用程式
- 目標框架為 .NET Framework 4.6.2
- 導航相關核心程式碼統一放置於命名空間 `Calin.Infrastructure.Navigation`
- 依賴注入容器使用 AutoFac
- 日誌系統使用 Serilog

## 整體設計目標

- 建立可擴充、可測試、低耦合的導航架構
- 同時支援多重視窗、多分頁（Tab）與區域（Region）導向的頁面管理
- 導航邏輯與 UI 容器（Form、UserControl）解耦

## Region 導向架構（RegionManager）

- 管理所有 Region 的生命週期與註冊
- Region 為邏輯上的 UI 容器，可對應 Form、Panel、TabPage 等
- 支援同時存在多個 Region
- 支援 Region 內 View 的注入與切換
- 支援多重視窗與分頁情境下的 Region 管理
- Region 與 View 採用 AutoFac 進行依賴注入
- 每次 Region 指定時需傳遞一個 `int` 作為頁面識別碼
- 頁面識別碼需被記錄，可用於查詢目前各 Region 的活動頁面

## 導航服務架構（NavigationService）

- 提供跨 Region 的統一頁面導航介面
- 負責實際的頁面切換、紀錄與回溯
- 與 RegionManager 協作，僅在指定 Region 內進行導航
- 支援以下導航能力
    - 導航至指定頁面
    - 前進（Forward）與後退（Back）
    - 傳遞參數至目標頁面
    - 查詢目前活動頁面與其識別碼
- 導航行為需支援多重視窗與分頁結構
- 所有 View 實例由 AutoFac 建立與管理
- 導航歷史需以頁面識別碼為核心進行管理

## 頁面生命週期管理

- 導航時需支援 AutoFac Lifetime Scope
- 每個 View 需支援 `IsAlive` 設定
    - `true`：View 為全域生命週期，導航離開後不釋放，需手動釋放
    - `false`：View 為區域生命週期，Region 被釋放時 View 自動釋放
- NavigationService 需依 `IsAlive` 決定是否重用或重建 View 實例

## 導航相關介面設計

- `INavigationAware`
    - 提供頁面導航前與導航後的事件通知
    - 用於初始化、狀態保存與資源釋放
- `INavigationParameters`
    - 用於頁面之間傳遞參數
    - 支援 Key-Value 型式存取
- `INavigationJournal`
    - 負責管理導航歷史紀錄
    - 支援 Back / Forward 操作
    - 以頁面識別碼為核心索引

## 設計原則與實作要求

- 導航邏輯不可直接依賴具體 UI 類型
- RegionManager 與 NavigationService 職責需明確分離
- 所有 View 與 Service 均透過 AutoFac 解析
- 架構需便於未來擴充至其他 UI 容器或導航策略

---

# Dialog 服務 PROMPT

## 專案與技術前提

- 專案類型：WinForm/WPF 桌面應用程式
- 目標框架：.NET Framework 4.6.2
- 對話框核心程式碼命名空間：`Calin.Infrastructure.Dialogs`
- 依賴注入容器：AutoFac
- 日誌系統：Serilog

## 整體設計目標

- 建立可擴充、可測試、低耦合的對話框管理架構
- 支援多種對話框類型：模態、非模態、自定義
- 對話框邏輯與 UI 容器（Form / UserControl）解耦

## 對話框服務架構（DialogService）

- 提供統一對話框管理介面
- 負責對話框的建立、顯示、關閉
- 支援：
    - 顯示模態對話框 (`ShowDialog`)
    - 顯示非模態對話框 (`ShowModeless`)
    - 傳遞參數至對話框
    - 從對話框接收結果
- 對話框實例由 AutoFac 建立與管理
- 支援自定義樣式與行為
- 所有操作需加入 Serilog 日誌與例外處理

## 對話框介面設計

- `IDialogAware`：
    - 提供對話框顯示前與關閉後事件通知
    - 用於初始化、狀態保存、資源釋放
- `IDialogParameters`：
    - Key-Value 型參數傳遞
    - 支援基本型別、物件、集合
    - 支援資料驗證與轉換
    - 可用於傳遞複雜物件及回傳結果
- `IDialogResult`：
    - 封裝對話框返回結果
    - 支援多種結果類型：確認、取消、自定義

## 設計原則與實作要求

- 對話框邏輯不可直接依賴具體 UI 類型
- DialogService 職責需明確分離
- 所有對話框與 Service 均透過 AutoFac 解析
- 架構需便於未來擴充至其他 UI 容器或對話框策略
- 強調可測試性與低耦合設計

## 範例與使用提示

- 請提供一個簡單模態對話框與非模態對話框的使用範例：
    - 透過 `DialogService.ShowDialog<TDialog>(IDialogParameters)` 顯示模態對話框
    - 透過 `DialogService.ShowModeless<TDialog>(IDialogParameters)` 顯示非模態對話框
- 範例中需展示參數傳遞與返回結果的完整流程

## 擴充性提示

- DialogService 與介面設計需支持未來擴充多種對話框策略
- 對話框實例的建立與管理應可延伸至其他 UI 容器（如 WPF、Avalonia）
- 提供清楚的接口與抽象類，方便替換實作

---

2026/01/10
這是從 CacheManager 修改過來的。
此 PROMPT 目標是產出一個**可獨立封裝為 NuGet 套件的泛型流程協調框架核心**，任何特定用途僅是其使用情境，而非框架責任本身。

# 泛型化「流程協調框架元件」PROMPT（可重用、NuGet 化、Autofac）

請建立一個**泛型化、可重用、可 NuGet 化的「流程協調（Coordination / Orchestration）框架元件」**。
此元件為**純框架層**，不具任何業務、UI、資料存取或特定領域語意，並以 **Autofac** 作為唯一 DI 容器。

## 一、整體定位與設計目標

- 本元件是一個「流程協調框架」，不是業務 Service
- 目標框架：.NET Framework 4.6.2
- 核心責任僅包含：
    - 接收一個協調請求
    - 計算需執行的一組工作單元
    - 協調其執行流程、狀態與完成條件
    - 在流程完成後，透過單一出口發布結果
- 本框架**完全不知道**：
    - UI / ViewModel
    - 資料庫或基礎建設
    - 特定業務名詞或用途
- 本框架可被重用於多種情境，例如：
    - 多模組非同步工作協調
    - 系統初始化 / Warm-up
    - ViewModel Refresh Pipeline
    - 任意跨模組流程編排

## 二、命名空間與模組邊界

- 框架所有程式碼必須放在單一根命名空間：`Calin.Infrastructure.Coordination`
- 框架層不得引用任何：
    - UI
    - Infrastructure
    - Domain / Entity
    - 專案專屬型別
- **Autofac 註冊模組（Module）必須一併定義於此框架命名空間中**
- 框架不得要求或假設：
    - 使用者將註冊寫入根 Module
    - 使用者自行掃描型別來完成核心註冊

## 三、核心概念抽象（泛型化）

### 協調工作單元（Task）

- 框架中的最小執行單位統一稱為 Task
- 每一個 Task：
    - 以唯一 TaskKey 識別
    - 實際執行邏輯由外部提供
    - 不得知道其他 Task 是否存在
    - 不得控制流程或發布結果

### 協調請求（Request）

- 呼叫端只能描述「發生了什麼」
- Request 為不可變描述物件
- 呼叫端**不得**：
    - 指定要執行哪些 Task
    - 判斷流程或執行順序

### 協調 Session / 狀態機

- 每一次協調請求必須建立一個獨立 Session
- Session 至少需表達：
    - SessionId
    - 本次需執行的 TaskKey 集合
    - 已完成的 TaskKey
    - 尚未完成的 TaskKey
    - 是否完成整體流程
- Session 必須可用於：
    - Debug
    - Log
    - Trace
- 框架不得假設 Task 的順序、數量或意義

## 四、角色與責任劃分（嚴格）

### 協調者（Coordinator）

- 僅負責流程協調
- 不實作任何實際工作
- 不得包含 switch / if-else 判斷特定 Task
- 不得知道 Task 的業務意義
- 不得直接呼叫 UI 或使用者邏輯

### Task Handler（由使用者提供）

- 每一個 Task 對應一個 Handler
- Handler：
    - 僅負責自身 Task 的執行
    - 必須為非同步
    - 不得發布結果
    - 不得控制流程
- Handler 以 Autofac 註冊方式加入系統

### Task 映射策略（Mapping Strategy）

- 所有「Request → TaskKey」的決策邏輯必須集中於策略物件
- 可同時存在多個策略
- 協調者只負責：
    - 找出可處理該 Request 的策略
    - 合併其產生的 TaskKey
    - 去除重複 Task
- 策略必須可被單元測試
- 不得將映射邏輯放在：
    - 呼叫端
    - Task Handler

### 執行策略（Execution Policy）

- 執行模式必須為可插拔策略，例如：
    - Immediate
    - Batch / Deferred
    - Parallel
    - Retry / Timeout
- 協調者不得寫死任何執行模式
- 新增執行策略不得修改協調者核心程式碼

### 結果發布（Result Publisher）

- 協調完成後，僅能透過單一出口發布結果
- 框架僅定義抽象介面
- 發布結果的用途由使用者專案決定
- 框架層不得知道結果如何被使用

## 五、執行與流程硬性規則

- 同一個 TaskKey 在同一次 Session 中最多執行一次
- 同一個 Session 中，結果發布最多發生一次
- 所有流程必須為非同步
- 所有決策必須集中於 Coordinator 與策略物件
- 禁止以下行為：
    - Handler 判斷或影響其他 Task
    - 呼叫端指定 Task
    - 協調者以條件判斷 Task 類型
    - Handler 或 Task 直接發布 UI 或通知

## 六、Autofac DI 與 Module 規範

- 框架必須提供一個 Autofac Module
- Module：
    - 定義於框架命名空間內
    - 負責註冊所有框架核心元件
    - 不得註冊任何使用者實作
- 使用者專案僅需：
    - 引用框架 NuGet
    - 註冊該 Module
    - 另外註冊自己的 Handler、策略與 Publisher
- 框架不得假設存在「根 Module」或全域註冊點

## 七、Open-Closed Principle

- 框架必須符合：
    - 對擴充開放
    - 對修改封閉
- 新增以下項目時：
    - Task Handler
    - Mapping Strategy
    - Execution Policy
    - Result Publisher
- 均不得修改既有框架核心程式碼

## 八、產出內容要求

- 協調者核心介面與實作
- Task Handler 抽象介面
- 協調請求物件設計
- Session / 狀態機設計
- Mapping Strategy 抽象
- Execution Policy 抽象
- Result Publisher 抽象
- Autofac Module 定義與註冊示意
- 不需實作任何實際業務、UI 或資料存取細節
