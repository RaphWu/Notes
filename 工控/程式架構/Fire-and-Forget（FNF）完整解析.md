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

# Fire-and-Forget（FNF）完整解析（工控/高可靠系統觀點）

Fire-and-Forget 指的是「觸發一個非同步工作後，不等待結果、不追蹤完成狀態」的執行模式。在 .NET 中通常對應 **未被 await 的 Task / Thread / Queue WorkItem**。

這種模式在 UI 或一般 Web 系統中尚可接受，但在**工控（Industrial Control）或高可靠系統**中屬於**高風險設計**。

## 一、核心定義

Fire-and-Forget 的本質特徵：

- 呼叫後立即返回（non-blocking）
- 呼叫端不持有控制權（no ownership）
- 不保證成功/失敗被處理
- 通常無 retry / timeout / cancellation

常見寫法：

```csharp
_ = Task.Run(() => DoWork());
```

或更危險：

```csharp
DoWorkAsync(); // 未 await
```

## 二、底層行為（.NET Runtime）

Fire-and-Forget 在 .NET 的實際運作：

- Task 仍會進入 ThreadPool 排程
- Exception 若未觀察（unobserved）：
    - .NET Framework → 可能 Crash（取決於設定）
    - .NET Core / .NET 5+ → 預設吞掉，但仍會觸發 `UnobservedTaskException`
- GC 可能在未完成前回收相關資源（若無引用）
- 無法保證執行順序與完成時間

## 三、使用場景（允許條件）

在**工控系統**中，只允許以下「低風險、可容忍遺失」場景：

### 1. Telemetry / Logging（可丟失）

```csharp
_ = Task.Run(() => logger.Log(...));
```

條件：

- 記錄失敗不影響主流程
- 有 fallback（例如本地 buffer）

### 2. 非關鍵 UI 更新

例如：

- 狀態顯示
- 非關鍵動畫

### 3. Cache Warmup / Lazy Init（可重做）

### 4. 監控性質（非控制性）

例如：

- 心跳記錄（非控制）
- Metrics 上報

## 四、嚴格禁止場景（工控級）

以下在工控系統中 **禁止使用 Fire-and-Forget**：

### 1. 硬體控制

```csharp
// ❌ 危險：可能未完成就結束
_ = axis.MoveAsync(position);
```

風險：

- 動作未完成
- 無法確認成功
- 無法 Stop / Abort

### 2. 狀態變更（State Mutation）

例如：

- Axis 狀態
- Device 狀態
- Safety 狀態

### 3. 資料寫入（DB / File / PLC）

可能導致：

- 寫入失敗無法察覺
- 資料不一致（Data Corruption）

### 4. 交易流程（Transaction）

例如：

- 生產流程
- 工單狀態

### 5. Initialization / Dispose

```csharp
// ❌ 錯誤
_ = driver.InitializeAsync();
```

## 五、常見錯誤模式

### 錯誤 1：未 await async

```csharp
public void Start()
{
    InitAsync(); // ❌
}
```

問題：

- Exception 消失
- 初始化未完成

### 錯誤 2：吞掉 Exception

```csharp
_ = Task.Run(async () =>
{
    await DoWorkAsync();
});
```

問題：

- 無 logging
- 無 retry

### 錯誤 3：假裝非同步

```csharp
Task.Run(() => BlockingIO());
```

問題：

- ThreadPool 被耗盡
- 延遲不可控

## 六、安全替代設計（強烈建議）

### 1. 明確 await（首選）

```csharp
await DoWorkAsync();
```

### 2. Controlled Background Worker（工控推薦）

建立**受控背景執行器**

```csharp
public interface IBackgroundWorker
{
    Task Enqueue(Func<CancellationToken, Task> work);
}
```

優點：

- 可追蹤
- 可取消
- 可統一 exception handling

### 3. Channel / Queue 模式

```csharp
Channel<T>
BlockingCollection<T>
```

適合：

- Motion command queue
- IO processing

### 4. Task + Tracking Registry

```csharp
ConcurrentDictionary<Guid, Task>
```

可用於：

- 任務監控
- Debug
- Shutdown 等待

### 5. Hosted Service（類似 ASP.NET Core）

即使在工控系統，也可設計：

- Runtime Background Service
- Polling Engine

## 七、工控級設計規範（建議納入你的 Level 5 規範）

### 規範 1：禁止未受控 FNF

- 所有 Task 必須：
    - await
    - 或納入管理（Registry / Worker）

### 規範 2：所有背景任務必須具備

- CancellationToken
- Timeout
- Exception Logging
- Retry Policy（必要時）

### 規範 3：關鍵流程必須可追蹤

- Motion
- IO
- Device State

### 規範 4：Shutdown 必須可收斂

```csharp
await Task.WhenAll(_runningTasks);
```

## 八、進階：安全 Fire-and-Forget 模式（有限度使用）

若真的需要 FNF，必須包裝：

```csharp
public static class TaskExtensions
{
    public static void SafeFireAndForget(this Task task, ILogger logger)
    {
        _ = task.ContinueWith(t =>
        {
            logger.LogError(t.Exception, "FNF Exception");
        }, TaskContinuationOptions.OnlyOnFaulted);
    }
}
```

使用：

```csharp
DoWorkAsync().SafeFireAndForget(logger);
```

## 九、總結（工控觀點）

Fire-and-Forget 本質是：

- **放棄控制權**
- **放棄可觀測性**
- **放棄可靠性**

在工控系統中，這三點通常是不可接受的。

因此結論：

- 可以存在，但必須**極度受限**
- 預設應視為 **anti-pattern**
- 應以「受控背景處理模型」取代

---

# Fire-and-Forget 工控級規範 + Runtime 落地設計（Copilot 專用 Prompt）

以下內容可**直接貼入 GitHub Copilot** 作為全域規範，已整合你目前的 Level 5 / Runtime 架構思維。

## Prompt：Fire-and-Forget 控制規範（強制版）

````markdown
# Fire-and-Forget（FNF）工控級嚴格規範

你正在開發的是「工控等級（Industrial Control Level 5）」系統。

請**嚴格遵守以下規範**：

## 一、基本原則

Fire-and-Forget（未 await 的 Task）在本系統中預設為：

- ❌ 不安全
- ❌ 不可追蹤
- ❌ 不可用於控制流程

除非符合「允許場景」，否則**一律禁止使用**

---

## 二、禁止事項（必須遵守）

以下情境「絕對禁止」Fire-and-Forget：

### 1. 硬體控制（Motion / IO）

- Axis Move
- Homing
- Stop / Emergency
- IO 寫入

```csharp
// ❌ 嚴禁
_ = axis.MoveAsync(...);
````

### 2. 系統狀態變更

- Device State
- Axis State
- Runtime State

### 3. 資料寫入

- Database
- File
- PLC Register

### 4. 初始化 / 釋放

```csharp
// ❌ 嚴禁
_ = driver.InitializeAsync();
```

### 5. 交易流程 / 生產流程

## 三、允許場景（極度限制）

僅允許以下用途：

### 1. Logging（可遺失）

### 2. Telemetry / Metrics

### 3. 非關鍵 UI 更新

## 四、替代方案（強制使用）

禁止直接使用 FNF，必須改為以下模式：

### 1. await（優先）

```csharp
await DoWorkAsync();
```

### 2. Background Worker（標準方案）

所有背景任務必須透過：

```csharp
IBackgroundWorker.Enqueue(...)
```

### 3. Channel Queue（高頻控制）

用於：

- Motion Command Queue
- IO Queue

## 五、背景任務強制要求

所有背景任務必須具備：

- CancellationToken
- Exception Handling
- Logging
- Timeout（必要時）
- Retry（必要時）

## 六、例外處理（最低要求）

禁止：

```csharp
_ = Task.Run(...)
```

允許（但仍需審核）：

```csharp
task.SafeFireAndForget(logger);
```

## 七、Shutdown 規範

系統關閉時必須：

```csharp
await Task.WhenAll(runningTasks);
```

確保：

- 無殘留任務
- 無未完成控制

## 八、設計目標

本規範的核心目標：

- 可預測（Predictable）
- 可追蹤（Traceable）
- 可中止（Abortable）
- 可恢復（Recoverable）

任何違反上述原則的設計，視為錯誤設計。

## Runtime 落地實作（關鍵元件）

以下是你系統中**應該實際存在的元件設計**

## 1. IBackgroundWorker（核心介面）

```csharp
public interface IBackgroundWorker
{
    Task<Guid> Enqueue(
        Func<CancellationToken, Task> work,
        string name,
        CancellationToken cancellationToken = default);
}
````

## 2. BackgroundWorker 實作（受控 FNF）

```csharp
public class BackgroundWorker : IBackgroundWorker
{
    private readonly ConcurrentDictionary<Guid, Task> _tasks = new();

    public Task<Guid> Enqueue(
        Func<CancellationToken, Task> work,
        string name,
        CancellationToken cancellationToken = default)
    {
        var id = Guid.NewGuid();

        var task = Task.Run(async () =>
        {
            try
            {
                await work(cancellationToken);
            }
            catch (Exception ex)
            {
                // TODO: logging
            }
            finally
            {
                _tasks.TryRemove(id, out _);
            }
        }, cancellationToken);

        _tasks[id] = task;

        return Task.FromResult(id);
    }

    public Task WhenAllAsync()
    {
        return Task.WhenAll(_tasks.Values);
    }
}
```

## 3. SafeFireAndForget（最低安全封裝）

```csharp
public static class TaskExtensions
{
    public static void SafeFireAndForget(this Task task, Action<Exception> onError)
    {
        _ = task.ContinueWith(t =>
        {
            onError?.Invoke(t.Exception);
        }, TaskContinuationOptions.OnlyOnFaulted);
    }
}
```

## 4. Runtime 整合（你的架構重點）

在：

```csharp
Calin.MotionControl.Advantech.Runtime
```

加入：

### 核心元件

- BackgroundWorker（唯一入口）
- Task Registry
- Shutdown Manager

### Runtime 啟動

```csharp
await backgroundWorker.Enqueue(async ct =>
{
    await pollingService.RunAsync(ct);
}, "Polling");
```

### Runtime 關閉

```csharp
await backgroundWorker.WhenAllAsync();
```

## 5. Motion / Driver 層強制規範

在：

```csharp
Driver / Controller 層
```

必須：

- ❌ 不得出現 Task.Run
- ❌ 不得出現 Fire-and-Forget
- ✅ 必須回傳 Task
- ✅ 必須由上層控制生命週期

## 六、設計結論（關鍵觀念）

你的整體架構應該轉為：

### ❌ 錯誤模型

```csharp
Controller → Fire-and-Forget
```

### ✅ 正確模型

```csharp
Controller → return Task
Runtime → 管理 Task（BackgroundWorker）
```

## 七、如果你要再往上升級（Level 6）

可以再加：

- Task Priority Queue（Motion > IO > Log）
- Watchdog（任務超時監控）
- Execution Pipeline（可插 Retry / Circuit Breaker）
- 狀態機整合（State Machine + Task）
