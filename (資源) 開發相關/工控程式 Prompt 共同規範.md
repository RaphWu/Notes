# 目錄

- [目錄](#目錄)
- [Industrial Control Software Global Requirements (Level 5) – Copilot Shared Prompt](#industrial-control-software-global-requirements-level-5--copilot-shared-prompt)
  - [全域設計優先順序（強制遵守）](#全域設計優先順序強制遵守)
  - [系統穩定性要求](#系統穩定性要求)
  - [Thread 與並行安全](#thread-與並行安全)
  - [Background Task 與 Polling 規範](#background-task-與-polling-規範)
  - [Fire-and-Forget 任務規範（Critical）](#fire-and-forget-任務規範critical)
  - [時間可預測性（Deterministic Behavior）](#時間可預測性deterministic-behavior)
  - [生命週期管理（Lifecycle）](#生命週期管理lifecycle)
  - [錯誤處理與復原](#錯誤處理與復原)
  - [記憶體與 GC 控制](#記憶體與-gc-控制)
  - [輕量化設計（Lightweight Industrial Architecture）](#輕量化設計lightweight-industrial-architecture)
  - [Logging 與診斷（已整合 Calin.Logging）](#logging-與診斷已整合-calinlogging)
  - [Library 設計限制](#library-設計限制)
  - [Application 設計](#application-設計)
  - [Configuration 規範](#configuration-規範)
  - [Simulation Mode](#simulation-mode)
  - [API 設計一致性](#api-設計一致性)
  - [相容性要求](#相容性要求)
  - [文件要求](#文件要求)
  - [最終設計原則](#最終設計原則)
- [這麼長一串丟進 GitHub Copilot 的結果](#這麼長一串丟進-github-copilot-的結果)
  - [Copilot 的實際行為（你要注意的重點）](#copilot-的實際行為你要注意的重點)
    - [1. Token 使用確實會增加](#1-token-使用確實會增加)
    - [2. 更大的問題：**規範過長 → 模型抓不到重點**](#2-更大的問題規範過長--模型抓不到重點)
    - [3. Inline Copilot 特別容易失效](#3-inline-copilot-特別容易失效)
  - [工控架構的最佳實務（重點）](#工控架構的最佳實務重點)
  - [建議架構（實務可行）](#建議架構實務可行)
    - [1. Global Prompt（精簡版，只留核心）](#1-global-prompt精簡版只留核心)
    - [2. 模組級 Prompt（按專案拆）](#2-模組級-prompt按專案拆)
    - [3. 任務級 Prompt（最重要）](#3-任務級-prompt最重要)
    - [4. 規範文件 ≠ Prompt](#4-規範文件--prompt)
  - [精準建議（結論版）](#精準建議結論版)
  - [一句話結論](#一句話結論)
- [精簡 → 分割](#精簡--分割)
  - [1. Core Prompt（全域核心規範）](#1-core-prompt全域核心規範)
  - [2. Runtime Prompt（系統核心 / Runtime / Polling Manager / 系統調度）](#2-runtime-prompt系統核心--runtime--polling-manager--系統調度)
  - [3. Driver / Library Prompt（硬體封裝 / 通訊層）](#3-driver--library-prompt硬體封裝--通訊層)
  - [4. App / Application Prompt（應用層 / UI 整合）](#4-app--application-prompt應用層--ui-整合)
- [最關鍵差異（你現在 vs 優化後）](#最關鍵差異你現在-vs-優化後)
- [Copilot 自動審查 Prompt（Level 5 工控專用）](#copilot-自動審查-promptlevel-5-工控專用)
  - [審查 Prompt（完整版｜建議主用）](#審查-prompt完整版建議主用)
  - [快速審查版（輕量｜日常用）](#快速審查版輕量日常用)
  - [Fire-and-Forget 專項審查 Prompt（強烈建議常用）](#fire-and-forget-專項審查-prompt強烈建議常用)

---

# Industrial Control Software Global Requirements (Level 5) – Copilot Shared Prompt

請在設計與產生程式碼前，**完整遵守以下工控軟體共同規範**。此規範適用於：

- Industrial Library
- Industrial Runtime
- Industrial Application

若規範僅適用於特定類型，會以標記表示：

- `[Core]`：Library 與 App 必須遵守
- `[Library]`：僅 Library
- `[App]`：僅 Application
- `[WhenApplicable]`：僅在元件具備該特性時適用

目標為建立 **Industrial Control Level 5 穩定架構**，可長期運行於工控設備與自動化系統。

## 全域設計優先順序（強制遵守）

1. 穩定性（Stability）
2. 相容性（Compatibility）
3. 效能（Performance）
4. 可預測性（Deterministic Behavior）
5. 可維護性（Maintainability）
6. 擴充性（Extensibility）
7. 優雅設計（Clean Design）

設計原則：

- 永遠優先穩定性、效能與可預測性
- 禁止為了架構美觀犧牲穩定性
- 禁止為了 DI 純度犧牲系統可預測性
- 工控系統設計應以 **長時間穩定運行** 為核心

## 系統穩定性要求

[Core]

- 支援 **24/7 長時間運作**
- 不允許單一元件錯誤導致整個系統停止
- 不可依賴不穩定外部狀態
- 所有 Background 工作必須可安全停止
- 所有 Background Task 必須支援 `CancellationToken`
- 禁止使用 `Thread.Abort`
- 避免不可控制的 Thread 建立
- 不可使用未受控的 Busy Loop

## Thread 與並行安全

[Core]

- 所有 `public` 方法必須 **thread-safe**
- 事件不可阻塞 I/O 或 Polling 執行緒
- 不可在事件回呼中執行長時間操作
- Background Loop 必須捕捉所有例外
- 不允許例外逃逸到 Polling Thread
- 禁止在 hot path 使用 `lock` 造成長時間阻塞
- 所有 async API 必須支援 `CancellationToken`
- 禁止 `async void`

## Background Task 與 Polling 規範

[Core]

- 必須具備 **安全停止機制**
- 必須支援 `CancellationToken`
- 必須捕捉所有例外
- 必須避免 CPU Busy Loop
- Polling Interval 必須可設定
- Timer 必須可安全 Dispose
- Background 工作不可無限制建立

建議：

- 使用集中式 Polling 管理
- 避免 Driver 自行建立 Thread

## Fire-and-Forget 任務規範（Critical）

[Core]

基本原則：

- **預設禁止使用 Fire-and-Forget**
- 僅允許在「非關鍵、可容忍失敗」的場景使用
- 不可用於：
  - 控制流程（Motion Control / IO Control）
  - 狀態同步
  - 設備通訊
  - Initialization / Shutdown 流程

強制規範：

- 不可使用裸 `Task.Run()` 作為 fire-and-forget
- 所有 fire-and-forget 任務必須：
  - 捕捉所有例外
  - 回報 Logging（Calin.Logging）
  - 支援 CancellationToken（若可控）
- 不可忽略 Task（禁止未觀察 Task）

建議實作模式（標準化），必須使用統一封裝：

```csharp
public static class SafeTask
{
    public static void Run(
        Func<Task> taskFactory,
        ILogger logger,
        CancellationToken ct = default)
    {
        Task.Run(async () =>
        {
            try
            {
                await taskFactory().ConfigureAwait(false);
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "Fire-and-forget task failed.");
            }
        }, ct);
    }
}
```

進階要求（Level 5 建議）：

- fire-and-forget 任務應：
  - 可被統計（數量 / 狀態）
  - 可被監控（optional）
- Runtime 可提供：
  - Background Task Registry（進階）

優先使用以下模式取代 fire-and-forget：

- 明確 Background Service（可 Stop / Dispose）
- Polling Loop
- Producer-Consumer Queue
- Channel / Buffer（受控）

## 時間可預測性（Deterministic Behavior）

[Core]

- 不可依賴不穩定時間來源
- Polling Loop 必須具有固定 Interval
- 禁止 CPU Spin
- 避免 GC Pause 影響控制流程
- 高頻控制流程應避免 Allocation

建議使用：

- `Stopwatch`
- 明確 Polling Interval 控制

## 生命週期管理（Lifecycle）

[Core]

- 建構子不得執行 I/O
- `Initialize()` 必須可安全失敗，並可重入
- `Start()` 必須安全啟動
- `Stop()` 必須安全停止背景工作
- `Dispose()` 必須完整釋放資源，並可重入

## 錯誤處理與復原

[Core]

- 所有例外必須捕捉
- 例外必須透過 **Calin.Logging** 回報（依賴 `Microsoft.Extensions.Logging.Abstractions`）
- 不可讓未處理例外終止系統
- 錯誤必須包含足夠診斷資訊

[WhenApplicable]

- 若涉及設備或通訊，必須具備錯誤復原策略
- 設備斷線應支援重新初始化
- 不可因單一設備失敗導致系統停止

## 記憶體與 GC 控制

[Core]

- 避免高頻 Allocation
- 不可在 Hot Path 使用 LINQ 或 `string.Format`
- 不可在 Hot Path 建立 closure
- 高頻資料結構應使用：`ArrayPool`、`RingBuffer`、結構型資料

目標：

- 降低 GC Pause
- 保持控制流程穩定

## 輕量化設計（Lightweight Industrial Architecture）

[Core]

- 系統應維持 **簡單可預測** 的架構
- 避免過度模組化與深層架構
- 避免 Runtime 複雜度增加
- 避免 Thread 過多、Module 過度拆分、過多背景服務
- 降低 GC 壓力
- **簡單比功能多更重要**
- **可預測性比抽象設計更重要**

## Logging 與診斷（已整合 Calin.Logging）

[Core / Library]

- Library 層依賴 **Calin.Logging**（底層使用 Serilog）
- 對外僅暴露 `ILogger<T>` 介面
- 不直接操作 File / Console / UI
- Logging 不可阻塞控制流程
- 支援等級控制（LogLevel）
- 高頻迴圈不可直接寫 Log

[App / Runtime]

- 注入 `Calin.Logging` 實例
- 管理 Log Storage / Output / Level
- 可統一整合 Serilog / ETW / NLog

> 統一使用 **Calin.Logging** 作為標準 Logging 層，底層可使用 Serilog，對外只暴露 `ILogger` 抽象。

## Library 設計限制

[Library]

- Library 必須保持 **Headless 設計**
- 不得引用 WinForms / WPF
- 不得依賴 UI Thread
- 不可直接操作 UI
- 不應管理 UI 或控制 UI 執行緒

## Application 設計

[App]

- 可管理 UI、Runtime、Logging System、Configuration 載入
- 仍需遵守 Thread Safety、例外處理、Background Task 管理

## Configuration 規範

[WhenApplicable]

- 必須支援外部 Configuration
- 啟動時可載入
- 配置錯誤不得導致系統崩潰
- 必須可驗證

## Simulation Mode

[WhenApplicable]

- 提供 **Simulation Mode**，不依賴實體設備
- 可模擬設備行為，用於開發與測試

## API 設計一致性

[Core]

- 命名一致
- API 行為可預測
- I/O 操作提供 Timeout
- async API 支援 `CancellationToken`
- 不允許 `async void`
- 不隱藏長時間操作或產生不可預測延遲

## 相容性要求

[Core]

- 僅使用 Windows 7 / Windows 10 相容 API
- 避免依賴高版本 OS 或不穩定 Framework

## 文件要求

[Core]

- 所有公開型別提供完整 XML Summary（正體中文）
- 包含每個 `class`、`interface`、`public method`、`public property`

## 最終設計原則

- 穩定性優先
- 可預測性優先
- 長時間運作優先
- 不因設計美觀犧牲系統可靠性

---

# 這麼長一串丟進 GitHub Copilot 的結果

**Token 消耗不只是「多一點」，而是可能顯著影響 Copilot 的效能與輸出品質**。

但真正的問題不只是 token，而是：

- **上下文污染（context dilution）**
- **關鍵指令被稀釋**
- **生成結果不穩定**

## Copilot 的實際行為（你要注意的重點）

### 1. Token 使用確實會增加

- Copilot（無論是 Chat 或 Inline）都有 context window 限制
- 你這份規範屬於「長系統提示（system-like prompt）」
- 每次生成時可能會：
  - 重複帶入（特別是 Chat）
  - 或被部分截斷（Inline）

結果：

- 有效資訊比例下降
- 模型「注意力分散」

### 2. 更大的問題：**規範過長 → 模型抓不到重點**

這是實務上最常見的問題：

- Copilot 不會「完整遵守規範」
- 它會「抓關鍵詞 + 模糊理解」

當規範過長時：

- Thread safety、Polling、Logging、Lifecycle…
- 全部同時出現 → 權重被平均化

結果：

- 該嚴格的地方反而不嚴格
- 該避免的（例如 fire-and-forget）還是會出現

### 3. Inline Copilot 特別容易失效

在你這種開發模式（VS + 工控架構）：

- Inline Suggestion 通常只吃「附近幾百行 context」
- 太長的 global prompt：
  - 很可能**根本沒被吃進去**
  - 或只吃到前半段

## 工控架構的最佳實務（重點）

不要把這份當：

> 「一次丟給 Copilot 的完整規範」

而要拆成：

> 「分層注入的規範系統」

## 建議架構（實務可行）

### 1. Global Prompt（精簡版，只留核心）

只保留**最高優先級規則（約 10~15 條）**

例如：

- 禁止 async void
- 所有 Background Task 必須可停止
- 必須支援 CancellationToken
- 禁止 fire-and-forget（未封裝）
- public API 必須 thread-safe
- 不可讓例外逃逸 polling thread

👉 控制在 **~300–500 tokens**

### 2. 模組級 Prompt（按專案拆）

例如：

- Runtime 專用規範
- Driver 專用規範
- Library 專用規範

👉 只在需要時提供

### 3. 任務級 Prompt（最重要）

每次叫 Copilot 時：

- 明確說：
  - 「這段是 polling loop」
  - 「這段是 driver」
  - 「這段是 fire-and-forget 禁止區」

👉 這比長規範有效 10 倍

### 4. 規範文件 ≠ Prompt

你現在這份：

> ✅ 非常適合當「設計標準文件」
> ❌ 不適合直接當「Copilot Prompt」

## 精準建議（結論版）

你應該這樣做：

- 保留這份完整規範 → 當 **Architecture Spec**
- 抽出一份：
  - **Copilot Core Rules（短版）**
- 再建立：
  - **場景 Prompt（Runtime / Driver / Polling）**

## 一句話結論

- 是的，會吃 token
- 但更嚴重的是：**會讓 Copilot 變笨，而不是變聰明**

  ***

# 精簡 → 分割

Copilot 專用 Markdown Prompt Pack（工控 Level 5）
這會比現在這份直接丟進去，效果穩定很多。

以下套件依專案層級整理，直接可當作 Copilot 的 System / Chat Prompt 使用，每個 Prompt 已整合前述核心規範與層級限制。

## 1. Core Prompt（全域核心規範）

```markdown
# Copilot Core Rules - 工控 Level 5

你正在設計工業控制系統，請嚴格遵守以下規範。

## 設計優先順序
1. 穩定性（Stability）
2. 相容性（Compatibility）
3. 效能（Performance）
4. 可預測性（Deterministic Behavior）
5. 可維護性（Maintainability）
6. 擴充性（Extensibility）
7. 優雅設計（Clean Design）

## 設計原則
- 架構簡潔、輕量，避免過度設計
- 支援長期運作（24/7）、大量設備並行（100+）
- 不可有記憶體洩漏
- 減少物件分配與 GC 壓力
- 遵守 Interface Segregation 原則
- 使用 ILogger（Calin.Logging）記錄錯誤
- 每個類別與介面必須有 XML Summary（正體中文）
- Dispose 必須完整釋放資源且可重入
- APP 不依賴 Driver
- 禁止 Thread.Abort
- 避免 race condition / deadlock / event re-entrancy

## Thread 與非同步安全
- 所有 public API 必須 thread-safe
- 事件不可阻塞 I/O 執行緒
- 禁止使用 async void（UI Handler 除外）
- 所有 Task / ValueTask 必須支援 CancellationToken
- 背景 Task 必須可安全停止並捕捉所有例外
- 禁止未受控 fire-and-forget
- 公開 API 或長時間 Task 使用 async Task
- 內部短期、頻繁非同步可使用 async ValueTask

## Polling 與 Background Task
- 採 Centralized Polling / Single Motion Thread
- 固定週期，不可漂移
- 不可阻塞主循環或事件執行緒
- 捕捉所有例外，不可中斷整體 loop
- 禁止 Busy Loop，必須有間隔控制
- 所有背景任務必須可被 Runtime 控制

## I/O 與錯誤處理
- 建構子不得執行 I/O
- Initialize / Start / Stop / Dispose 必須安全且可重入
- 所有 I/O 必須可 Timeout
- 必須處理設備異常（斷線、無回應）
- 不可拋出未處理例外至上層
- 單一設備或任務失敗不得影響整體系統
- Logging 不可阻塞主流程，高頻迴圈不可直接寫 Log
```

## 2. Runtime Prompt（系統核心 / Runtime / Polling Manager / 系統調度）

```markdown
# Copilot Runtime Rules - 工控 Level 5

遵守 Core Rules。

## 核心職責
- 統一管理 Polling，禁止 Driver 自行建立 Thread
- 所有背景任務必須可被 Runtime 控制
- Task 使用受控 Task Registry（進階）

## Polling 規範
- 固定週期，不可漂移
- 必須支援 CancellationToken
- 必須捕捉所有例外
- 不可因例外中斷整體 loop

## 錯誤隔離
- 單一設備錯誤不可影響其他設備
```

## 3. Driver / Library Prompt（硬體封裝 / 通訊層）

```markdown
# Copilot Driver / Library Rules - 工控 Level 5

遵守 Core Rules。

## 設計限制
- 不可建立 Thread 或管理 Polling
- 不依賴 UI 或 SynchronizationContext
- 必須支援 Simulation Mode（若適用）

## I/O 規範
- 所有 I/O 必須可 Timeout
- 必須處理設備異常（斷線、無回應）

## 錯誤處理
- 不可拋出未處理例外至上層
- 必須回報錯誤（ILogger）
```

## 4. App / Application Prompt（應用層 / UI 整合）

```markdown
# Copilot App Rules - 工控 Level 5

遵守 Core Rules。

## 核心職責
- 管理 UI / Runtime / Logging / Configuration
- 系統初始化與組裝

## UI 規範
- 不可阻塞 UI Thread
- 長時間操作需非同步處理（受控）

## 錯誤處理
- 必須顯示或記錄錯誤
- 不可讓應用程式崩潰
```

這套 Prompt Pack 可直接套用：

- 新專案初始化時使用對應 Prompt
- Copilot 生成程式碼自動遵守工控 Level 5 標準
- 避免不必要的重複規範，易於維護

我可以再幫你做一版 **範例套用範本**，每個層級附上**C# 專案目錄與初始類別範例**，讓 Copilot 直接生成符合規範的程式碼。

你要我直接生成這個範本嗎？

# 最關鍵差異（你現在 vs 優化後）

你原本：

- 一份超長規範 → Copilot 吃不乾淨

現在：

- **短核心 + 場景化規範 → 高命中率 + 穩定輸出**

---

# Copilot 自動審查 Prompt（Level 5 工控專用）

這一組 Prompt 用於：**讓 Copilot 在產生程式碼後，自動進行「工控規範審查」**

使用方式：
👉 先讓 Copilot 產生程式碼
👉 再貼上這段 Prompt + 程式碼
👉 要求「審查 + 修正」

## 審查 Prompt（完整版｜建議主用）

````markdown
# 工控程式碼審查（Level 5）

請針對 (XXX 專案 / XXX 程式碼) 進行「嚴格工控規範審查」，並輸出：

1. 問題清單（依嚴重程度排序：Critical / High / Medium / Low）
2. 每個問題的原因（對應工控風險）
3. 修正建議
4. 修正後完整程式碼

## 審查規範（必須逐條檢查）

### 穩定性
- 是否可能因例外導致系統中斷？
- 是否存在單點失敗？

### Thread 安全
- public API 是否 thread-safe？
- 是否存在競態條件？

### async / await
- 是否使用 async void（禁止）？
- 是否缺少 CancellationToken？
- 是否有未觀察 Task？

### Fire-and-Forget（重點）
- 是否存在 Task.Run 未封裝？
- 是否未捕捉例外？
- 是否用於控制流程（禁止）？

### Background Task
- 是否可安全停止？
- 是否有 Busy Loop？
- 是否缺少例外保護？

### Polling / Deterministic
- 是否固定週期？
- 是否有時間漂移或不可預測延遲？

### Logging
- 是否有錯誤未記錄？
- 是否在高頻路徑中寫 Log？

### GC / 效能
- Hot path 是否有 Allocation？
- 是否使用 LINQ / closure（高頻區域）？

### Lifecycle
- 是否違反 Initialize / Start / Stop / Dispose 原則？
- 是否有資源未釋放？

---

## 輸出格式（強制）

### 問題清單
- [Critical] ...
- [High] ...
- [Medium] ...
- [Low] ...

### 修正建議
（條列）

### 修正後程式碼
```csharp
// 完整修正版
````

## 快速審查版（輕量｜日常用）

當你只想快速檢查：

```markdown
請檢查 (XXX 專案 / XXX 程式碼) 是否違反工控 Level 5 規範，特別關注：

- async void（禁止）
- fire-and-forget（是否未封裝）
- 未捕捉例外
- Background Task 是否可停止
- 是否有 Busy Loop
- public API 是否 thread-safe

列出問題並提供修正後程式碼。
```

## Fire-and-Forget 專項審查 Prompt（強烈建議常用）

這是你架構中**最容易出事的點**：

```markdown
請檢查 (XXX 專案 / XXX 程式碼) 中的 fire-and-forget 使用是否安全：

檢查項目：
- 是否使用 Task.Run 但未封裝（違規）
- 是否未捕捉例外
- 是否未記錄 logging
- 是否無法取消（缺少 CancellationToken）
- 是否用於控制流程（嚴重違規）

若有問題：
1. 指出風險
2. 說明為何在工控系統中危險
3. 提供安全替代寫法（含完整程式碼）
```
