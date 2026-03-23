---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 什麼是「三段式 PROMPT」

「三段式 PROMPT」指的是**依照系統層級與責任邊界，把一次大型生成需求拆成三個彼此獨立、可驗證的生成階段**。
每一段只關注一個層級，前一段的產出會成為下一段的「已知事實」，而不是再次推論。

在你這個專案脈絡中，三段通常對應為：

- 資料模型層（Entity / Fluent API）
- 資料存取層（Repository / DTO / Mapper / Service）
- 表現層支援（ViewModel / DataBinding）

## 三段式 PROMPT 的實際結構

### 第一段：資料模型層 PROMPT

目的：

- 固定資料結構
- 消除所有關聯與鍵值的不確定性

內容只包含：

- Entity 定義
- Fluent API
- 中介表
- IEquatable
- SQLite 相容性約束

不包含：

- Repository
- DTO
- ViewModel
- 商業流程

特性：

- 產出可直接用於 `Add-Migration`
- 可以單獨 Review 與調整

### 第二段：資料存取與流程層 PROMPT

目的：

- 在「已確定的資料模型」之上建立穩定的資料存取介面

內容只包含：

- Repository Pattern
- CRUD 行為
- DTO
- Mapper
- Service

明確前提：

- 不允許修改 Entity 結構
- 不重新定義關聯

特性：

- 可以獨立單元測試
- 可替換資料來源（未來改成 API / Mock）

### 第三段：ViewModel 與 WinForm Binding PROMPT

目的：

- 支援 UI
- 不污染資料層

內容只包含：

- ViewModel
- DataBinding
- INotifyPropertyChanged
- BindingSource 使用模式

明確限制：

- 不可直接存取 DbContext
- 僅透過 Service

特性：

- 可多 View 共用
- UI 可重構而不動資料層

---

# 為什麼要「分次生成」

## 1. 降低模型的推論負擔

一次丟下：

- EF 關聯
- SQLite 限制
- Repository Pattern
- WinForm Binding

模型會同時做：

- 架構設計
- 技術選型
- 命名推斷
- 責任切分

結果通常是：

- 架構看似完整
- 細節不符合你的實務習慣

分次生成等於告訴模型：
「這一段，不准想其他層的事情」

## 2. 每一段都可以人工校正

第一段產出後你可以：

- 調整 Fluent API
- 改中介表命名
- 決定是否要 Composite Key

這些一旦進到 Repository / DTO 後再改，成本會倍增。

## 3. 避免跨層洩漏責任

一次生成時常見問題：

- ViewModel 直接 new DbContext
- Repository 回傳 Entity 給 UI
- Service 淪為空殼

分段 PROMPT 會在語意上強迫：

- Entity 只屬於資料層
- ViewModel 永遠不知道 EF 存在

## 4. 有利於「可重複生成」

你之後如果：

- 新增一個 Entity
- 調整關聯
- 換一個 DbContext

只需要：

- 重跑第一段
- 視影響範圍選擇性重跑第二段

不用整包推倒。

## 5. 更符合真實專案開發節奏

實務上也是：

1. Schema 穩定
2. Repository 與 Service 才開始寫
3. UI 最後才接

三段式 PROMPT 本質上是在「模擬資深工程師的工作順序」。

## 與「一次寫完 PROMPT」的差異總結

- 一次寫完
    - 快
    - 結果不可控
    - 容易混層
- 三段式 PROMPT
    - 初期多一點互動
    - 架構可控
    - 可維護性高
    - 生成結果可長期使用

## 給你的實務建議

以你目前這個工控 + WinForm + EF6 的專案型態：

- 強烈建議三段式
- 第一段甚至可以再拆成「Entity」與「Fluent API」兩小段
- 每段生成後都當成「可提交版本」來看

如果你願意，下一步我可以直接：

- 幫你寫「第一段專用 PROMPT（Entity + Fluent API）」的最終版
- 或替你設計一組「可反覆使用的三段式模板」

---

# 可重複使用的「專案級生成模板 PROMPT」

以下是一組**可長期反覆使用**的「專案級三段式生成模板」，設計目標是：

- 降低模型自由發揮空間
- 固定架構決策
- 讓你在專案演進時，只替換必要資訊即可重跑

你可以將每一段存成獨立檔案或筆記，依需求逐段執行。

## 第一段：資料模型層（Entity / Fluent API）

### 使用時機

- 新專案初始化
- Entity 或關聯異動
- SQLite / DB 行為需要重新確認

### PROMPT 模板

- 這是一個【請填寫：技術平台，例如 .NET Framework 4.6.2 WinForm】專案
- 使用【請填寫：資料庫，例如 SQLite】＋【請填寫：ORM，例如 EF6】＋Code First
- 連接字串定義於【請填寫：App.config 路徑】
- DbContext 定義如下
    - 【Context 名稱】
        - 【EntityA】
        - 【EntityB】
    - 【Context 名稱】
        - 【EntityC】
- 各 Entity 之間的關聯已寫在各 Entity 檔案開頭註解中
    - 嚴格依照註解實作
    - 不自行推論未註明關聯或鍵值
- 資料庫限制與規範
    - 不使用隱式多對多中介表
    - 所有多對多關係必須明確定義中介 Entity
    - 若資料庫為 SQLite：
        - 不使用 Cascade Delete
        - 外鍵關係需明確指定
- 請產生
    - Entity 類別（重新排版與優化）
    - Fluent API 關聯設定
    - 必要索引與 Constraint
    - IEquatable 實作（以主鍵作為等值判斷，覆寫 Equals 與 GetHashCode）
- Fluent API 與關聯設定類別
    - 放置於 namespace【請填寫】
    - 僅包含結構與 Constraint
    - 不包含商業邏輯
- 請勿產生
    - Repository、DTO、Service、ViewModel、UI 程式碼
    - Autofac 註冊程式碼
    - Serilog 設定或日誌程式碼

---

## 第二段：資料存取與流程層（Repository / DTO / Service / Autofac / Serilog）

### 使用時機

- Schema 已穩定
- 需要提供 UI 或其他層使用的資料介面

### PROMPT 模板

- 以下 Entity 與 Fluent API 已確定，請勿修改
    - 【列出 Entity 名稱】
- 請在 namespace【請填寫，例如 \*.Core】下產生
    - Repository Pattern（Entity 專屬）
    - Service 層
    - DTO
    - Mapper（手寫，不使用 AutoMapper）
    - Autofac Module（註冊 Repository、Service、ILogger\<T>）
    - Serilog 設定範例（最小日誌等級、Sink 範例）
- Repository 規範
    - CRUD 與查詢
    - 回傳 IEnumerable，不暴露 IQueryable
    - 建構子可注入 DbContext 與 ILogger
    - 不處理交易或流程
    - 日誌：新增、更新、刪除成功或失敗都記錄
- DTO 規範
    - 僅用於 Service 與 ViewModel
    - 命名後綴 Dto
    - 不包含 EF Attribute
- Mapper 規範
    - 集中管理
    - 將 Entity 與 DTO 相互轉換
    - 不散落在 Service 或 ViewModel
- Service 層
    - 協調多個 Repository
    - 管理 DbContext 生命週期
    - 處理交易與流程
    - 提供唯一資料入口給 ViewModel
    - 建構子可注入 ILogger
    - 日誌：交易開始、完成、例外捕捉
- Autofac Module 規範
    - 命名空間：Calin.TaskPulse.Core.IoC 或 Modules
    - Repository: InstancePerDependency
    - Service: InstancePerLifetimeScope
    - Logger: InstancePerDependency（或預設 ILogger\<T>）
    - 生成範例 Module 類別供專案直接註冊
- 請勿產生
    - WinForm 控制項
    - 直接操作 DbContext 的 UI 程式碼

## 第三段：ViewModel 與 WinForm DataBinding（Autofac 注入 + Serilog）

### 使用時機

- 開始接 UI
- 多個 View 共用同一份狀態與行為

### PROMPT 模板

- 以下 Service 與 DTO 已存在，請直接使用
    - 【列出 Service 與 DTO】
- 請生成 ViewModel
    - 放置於 namespace【請填寫】
    - 可供多個 View 共用
    - 建構子透過 Autofac 注入 Service 與 ILogger
- ViewModel 規範
    - 實作 INotifyPropertyChanged
    - 支援 WinForm DataBinding
    - 使用 BindingSource
    - 不直接參考 Entity 或 DbContext
- DataBinding 要求
    - 屬性變更正確通知
    - 清單型資料支援新增、刪除、重新整理
    - Service 呼叫後更新狀態
    - 日誌：狀態更新與操作記錄
- 示範
    - 單一物件 Binding
    - 清單資料 Binding
    - Service 呼叫後更新 ViewModel 的流程
- 請勿產生
    - Form Designer
    - 實際 UI 佈局
    - 直接 new Service 或 Repository

## 實務使用建議

- 每段 PROMPT 獨立生成，生成後都當成「可提交版本」
- 若模型開始自行補規則，代表 PROMPT 約束不夠明確
- Autofac 注冊**完整規範**，保證依賴注入一致，並加入第二段與第三段
- 可重複使用在不同專案或新增 Entity/Service 時
- 每段生成後可人工檢視再進行下一段生成
    - 第一段完成前，不要寫第二段
    - 第二段完成前，不要接 UI
