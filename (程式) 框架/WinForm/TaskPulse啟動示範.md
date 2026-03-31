---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 程式碼

```csharp
internal static class Program
 {
     // P/Invoke 讓已存在程式跳到前景
     private static class NativeMethods
     {
         [DllImport("user32.dll")]
         public static extern bool SetForegroundWindow(IntPtr hWnd);

         [DllImport("user32.dll")]
         public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
     }
     private static Mutex _mutex;

     /// <summary>
     /// 應用程式的主要進入點。
     /// </summary>
     [STAThread]
     static void Main()
     {
         // 確保單一實例執行
         _mutex = new Mutex(true, "Calin.TaskPulse.UniqueAppMutex", out bool createdNew);
         if (!createdNew)
         {
             // 找到現有程式，BringToFront
             var current = Process.GetCurrentProcess();
             var existing = Process.GetProcessesByName(current.ProcessName)
                                   .FirstOrDefault(p => p.Id != current.Id);
             if (existing != null)
             {
                 MessageBox.Show("TaskPulse 已在執行中，將切換到已開啟的視窗。", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
                 NativeMethods.ShowWindow(existing.MainWindowHandle, 5); // SW_SHOW
                 NativeMethods.SetForegroundWindow(existing.MainWindowHandle);
             }
             return;
         }

         Application.EnableVisualStyles();
         Application.SetCompatibleTextRenderingDefault(false);

         using (var cts = new CancellationTokenSource())
         {
             Application.Run(new AppStartupContext(cts));
         }

         _mutex.Dispose();
     }
 }
```

```csharp
    /// <summary>
    /// 表示應用程式的啟動上下文，負責初始化應用程式並管理其生命週期。
    /// </summary>
    public sealed class AppStartupContext : ApplicationContext
    {
        private readonly CancellationTokenSource _cts;
        private readonly SplashScreen _splash;
        private IContainer _container;

        /// <summary>
        /// 初始化 <see cref="ApplicationContext"/> 類別的新執行個體。
        /// </summary>
        /// <param name="cts">用於取消操作的 <see cref="CancellationTokenSource"/>。</param>
        public AppStartupContext(CancellationTokenSource cts)
        {
            _cts = cts;

            // 顯示啟動畫面
            _splash = new SplashScreen();
            _splash.Show();
            _splash.Refresh();

            SplashMessenger.Post("程式初始化開始...");
            using (LogContextExtensions.WithSystem())
            {
                Log.Information("TaskPulse 啟動");
            }
            StartInitialization();
        }

        /// <summary>
        /// 開始初始化應用程式的核心邏輯。
        /// </summary>
        private void StartInitialization()
        {
            _ = Task.Run(() =>
            {
                try
                {
                    InitializeCore(_cts.Token);

                    if (_cts.IsCancellationRequested)
                        return;

                    SplashMessenger.Post("初始化完成，準備啟動主畫面...");
                    Thread.Sleep(1200);

                    // 回到 UI Thread 顯示 MainForm
                    _splash.BeginInvoke(new Action(() =>
                    {
                        var mainForm = _container.Resolve<MainForm>();
                        mainForm.FormClosed += MainForm_FormClosed;

                        MainForm = mainForm;
                        _splash.Close();
                        mainForm.Show();
                        mainForm.Activate();
                    }));
                }
                catch (OperationCanceledException)
                {
                    Log.Information("應用程式初始化已取消");
                    using (LogContextExtensions.WithSystem())
                    {
                        Log.Information("TaskPulse 由 CancellationTokenSource 結束");
                    }

                    ExitOnUiThread();
                }
                catch (Exception ex)
                {
                    Log.Fatal(ex, "應用程式初始化失敗");
                    using (LogContextExtensions.WithSystem())
                    {
                        Log.Information($"TaskPulse異常結束!\n\n{ex.Message}");
                    }

                    ShowErrorAndExit(ex.Message);
                }
            }, _cts.Token);
        }

        /// <summary>
        /// 初始化應用程式的核心模組和依賴項。
        /// </summary>
        /// <param name="token">用於取消操作的 <see cref="CancellationToken"/>。</param>
        private void InitializeCore(CancellationToken token)
        {
            token.ThrowIfCancellationRequested();

            SplashMessenger.Post("初始化日誌服務..."); // 必須最先執行
            LoggingBootstrapper.Initialize();

            // 初始化 DI 容器
            var builder = new ContainerBuilder();
            builder.RegisterType<MainForm>();
            builder.RegisterModule<CoreModule>();
            builder.RegisterModule<MainModule>();
            builder.RegisterModule<MaintiFlowModule>();
            builder.RegisterModule<MechaTrackModule>();
            builder.RegisterModule<ToolQuestMoudle>();

            token.ThrowIfCancellationRequested();

            _container = builder.Build();
        }

        /// <summary>
        /// 處理主視窗關閉事件，並執行應用程式的關閉邏輯。
        /// </summary>
        /// <param name="sender">事件的來源。</param>
        /// <param name="e">包含事件資料的 <see cref="FormClosedEventArgs"/>。</param>
        private void MainForm_FormClosed(object sender, FormClosedEventArgs e)
        {
            using (LogContextExtensions.WithSystem())
            {
                Log.Information("TaskPulse正常結束");
            }

            Shutdown();
        }

        /// <summary>
        /// 關閉應用程式，釋放資源並停止所有操作。
        /// </summary>
        private void Shutdown()
        {
            _cts.Cancel();

            _container?.Dispose();
            LoggingBootstrapper.Shutdown();

            ExitThread();
        }

        /// <summary>
        /// 在 UI 執行緒上退出應用程式。
        /// </summary>
        private void ExitOnUiThread()
        {
            _splash.BeginInvoke(new Action(ExitThread));
        }

        /// <summary>
        /// 顯示錯誤訊息並退出應用程式。
        /// </summary>
        /// <param name="message">要顯示的錯誤訊息。</param>
        private void ShowErrorAndExit(string message)
        {
            _splash.BeginInvoke(new Action(() =>
            {
                MessageBox.Show(message, "啟動失敗", MessageBoxButtons.OK, MessageBoxIcon.Error);
                Shutdown();
            }));
        }
    }
```

```csharp
public partial class MainForm : UIForm
{
    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        base.OnFormClosing(e);

        _messageTimer?.Stop();
        _messageTimer?.Dispose();

        _idleTimer?.Stop();
        _idleTimer?.Dispose();

        WeakReferenceMessenger.Default.UnregisterAll(this);
    }

    protected override void OnFormClosed(FormClosedEventArgs e)
    {
        if (_keyboardHookId != IntPtr.Zero)
            UnhookWindowsHookEx(_keyboardHookId);

        if (_mouseHookId != IntPtr.Zero)
            UnhookWindowsHookEx(_mouseHookId);
            
        base.OnFormClosed(e);
    }
}
```

# Program 中的 CancellationTokenSource 會在什麼時候觸發?

## 結論先行

在你目前這套設計中：

**Program 裡建立的 `CancellationTokenSource` 本身「不會自動觸發」**
它只會在 **`AppStartupContext` 主動呼叫 `_cts.Cancel()` 時才會觸發**。

如果程式碼中沒有任何地方呼叫 `Cancel()`，那麼：

- Token 永遠是 `IsCancellationRequested == false`
- `Task.Run(..., token)` **不會自己停止**
- CancellationToken 只是「存在」，但不會發生任何效果

## 你的程式目前的實際觸發點

### Program.cs

```csharp
using (var cts = new CancellationTokenSource())
{
    Application.Run(new AppStartupContext(cts));
}
```

這段程式碼只做三件事：

- 建立 CTS
- 把 CTS 傳給 AppStartupContext
- 等到 `Application.Run()` 結束後才 Dispose CTS

**這裡不會觸發 Cancel()**

Dispose ≠ Cancel
Dispose 只是釋放資源，不會發送取消訊號。

## 目前唯一會觸發 Cancel 的地方

### AppStartupContext.Shutdown()

```csharp
private void Shutdown()
{
    _cts.Cancel();

    _container?.Dispose();
    LoggingBootstrapper.Shutdown();

    ExitThread();
}
```

### 也就是說，只有在以下情境之一才會觸發

- MainForm 關閉（FormClosed）
- 初始化失敗呼叫 `Shutdown()`
- 你未來主動加上其他取消邏輯

## 目前設計下「實際會取消初始化的時機」

|情境|是否呼叫 Cancel|
|---|---|
|初始化正常完成|否|
|MainForm 關閉|是|
|初始化中發生例外|是|
|使用者關閉 Splash（目前未實作）|否|
|Program 結束 using 區塊|否|

## 為什麼這樣設計是「合理的」

CancellationToken 在這個架構中是：

- **「協調背景初始化與應用程式關閉」**
- 而不是「自動中斷機制」

這樣的好處：

- 初始化流程可單向完成
- MainForm 關閉時可安全通知所有背景工作停止
- 不會因 UI Dispose 而產生未預期例外

## 常見誤解澄清（重要）

### 1. Dispose CTS 會不會觸發取消？

不會。

```csharp
cts.Dispose(); // 不會觸發 Cancel
```

### 2. Task.Run(token) 會自動中斷嗎？

不會。

```csharp
Task.Run(() => DoWork(), token);
```

只有在以下情況 Task 才會取消：

- Token 在 **Task 尚未啟動前** 已被 Cancel
- 或 Task 內部主動檢查 token 並拋出 `OperationCanceledException`

## 你現在「其實少了一個可用的取消情境」

### 建議補強（但不是必須）

如果你希望：

- 使用者在 Splash 顯示期間按 X
- 或系統強制關閉
- 初始化立刻中斷

可以加上：

```csharp
_splash.FormClosing += (_, __) =>
{
    _cts.Cancel();
};
```

並在 InitializeCore 中確保：

```csharp
token.ThrowIfCancellationRequested();
```

## 一句話總結

> Program 中的 CancellationTokenSource **只是一個傳遞取消能力的容器**
> 真正決定「什麼時候取消」的是 AppStartupContext
> 而不是 Program，也不是 using 區塊

目前這個架構已經是「可控、可關閉、不中毒」的穩定版本，後續你可以依需求再加的延伸點只有三類，彼此互不衝突：

- Splash 期間允許使用者中斷啟動（X 關閉或取消按鈕 → `_cts.Cancel()`）
- 初始化失敗後提供 Retry（重建 Container + 重新 StartInitialization）
- 初始化完成後才啟動長駐背景服務（由 MainForm 或 AppContext 負責）

---

# CTS 放在 Program 的意義是什麼？

## 簡短結論

在你目前這個架構下：

**CTS 放在 Program「幾乎沒有實質技術必要性」，主要是「架構語意上的選擇」。**

## 放在 Program 的典型意義

以下是「放在 Program 才有意義」的幾種情境，但你目前**沒有用到**：

- Program 需要在 `Application.Run` 之外主動取消整個 App
- 有多個 ApplicationContext 共用同一個取消來源
- 單元測試時由 Program 控制啟動 / 中止流程
- 有「非 UI 條件」需要終止初始化
    - 命令列參數
    - 單一實例檢查失敗
    - 授權檢查失敗

## 你現在實際發生的事

```csharp
using (var cts = new CancellationTokenSource())
{
    Application.Run(new AppStartupContext(cts));
}
```

- Program **從未呼叫 `Cancel()`**
- Program **完全不關心取消時機**
- CTS 只是被「轉交」給 AppStartupContext

在這個狀態下：

> CTS 放在 Program
> 與
> CTS 放在 AppStartupContext
> 行為是 **100% 相同的**

---

# 改成 AppStartupContext 自己 new CTS 會有什麼問題？

## 結論先說

**在你目前的設計中：完全沒有問題，反而更乾淨。**

## 為什麼「沒有問題」

因為：

- ApplicationContext 本來就是整個 App 的生命週期擁有者
- CTS 的用途只在：
    - 中斷初始化 Task
    - 在關閉時通知背景工作
- Program 並沒有參與取消決策

## 改寫後的自然版本

```csharp
public sealed class AppStartupContext : ApplicationContext
{
    private readonly CancellationTokenSource _cts = new();
}
```

```csharp
Application.Run(new AppStartupContext());
```

這樣的好處：

- 職責單一
- Program 只負責「是否啟動 App」
- Context 負責「App 什麼時候結束」

## 為什麼一開始會「有人建議 CTS 放 Program」？

這通常來自 **Console / Service / WPF** 的慣例，而不是 WinForms。

### 在這些場景中

- Program（Main）本身就是「主控流程」
- UI 只是其中一個階段
- CTS 放在最外層很合理

### 但在 WinForms 中

- `Application.Run()` 本身就是主流程
- ApplicationContext 是「真正的 Main」
- Program 只是 bootstrapper

## 什麼時候「一定要放在 Program」？

只有以下情況才值得：

- 你要支援 **多 ApplicationContext**
- 你要在 `Application.Run()` 外部控制中斷
- 你有 headless / batch 模式（不顯示 UI）
- 你要在 Main 裡做長時間非 UI 工作

你目前 **全部都沒有**。

## 實務建議（直接給結論）

### 對你現在的專案

- **改成 AppStartupContext 內部 new CTS**
- Program 不傳 CTS
- 程式碼更直覺、更不容易被誤用

### 你現在的 Program 可以簡化成

```csharp
[STAThread]
static void Main()
{
    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);
    Application.Run(new AppStartupContext());
}
```

## 一句話總結（可寫成註解）

> 在 WinForms 中，ApplicationContext 是 App 的生命週期核心
> 若取消只影響 UI 與初始化流程，CTS 應由 Context 自行管理

---

# 關於 CTS

## 1. CTS 是否要 Dispose？什麼時候？

### 結論

- **是的，應該要 Dispose**
- **時機**：App 不再使用 CancellationToken，也就是 **ApplicationContext 退出前**

### 原因

`CancellationTokenSource` 實作了 `IDisposable`，內部有 **Timer / 等待資源**
不 Dispose 會：

- 微量資源沒釋放（一般不會馬上造成問題）
- 但長期累積可能增加 GC 負擔

### 在你的 AppStartupContext 中，建議方式

```csharp
private void Shutdown()
{
    _cts.Cancel();
    _container?.Dispose();
    LoggingBootstrapper.Shutdown();

    _cts.Dispose();    // <- 這裡可以安全釋放
    ExitThread();
}
```

- Shutdown 一次就處理所有資源
- 包含 CTS / Container / Logging
- 確保 Process 正確釋放

> 也可以寫成 `try { _cts.Cancel(); } finally { _cts.Dispose(); }` 保險

## 2. 初始化完成後 CTS 是否還需要？

### 現在流程分析

1. CTS 主要用途：

- 讓初始化 Task 可取消
- 通知 App 關閉時取消 Task / 背景工作

2. 初始化完成後：

- Task 已經完成
- MainForm 開始顯示
- 如果沒有其他 Task 需要監聽這個 CTS，**CTS 就不再必要了**

### 建議做法

- **初始化完成後可以保留 CTS 一直到 App 關閉**
    - 保證未來若 MainForm 有背景工作需要取消，可共用這個 CTS
- **如果沒有後續用途，也可以在初始化完成後立即 Dispose**
    - 不過在 UI Thread 期間最好保留，避免未來擴展的取消需求

### 總結

|情境|是否還需要 CTS|
|---|---|
|初始化 Task 完成|不再必要，但保留方便未來取消背景任務|
|MainForm 關閉 / App 關閉|必須 Cancel + Dispose|
|初始化期間用戶可取消|必須保留直到初始化 Task 完全結束|

### 實務建議（對你現在專案最簡單安全做法）

```csharp
private void Shutdown()
{
    // 取消初始化或背景 Task
    _cts.Cancel();

    // Dispose DI 與 Logging
    _container?.Dispose();
    LoggingBootstrapper.Shutdown();

    // 最後釋放 CTS
    _cts.Dispose();

    ExitThread();
}
```

- 確保 Process 不殘留
- 不管初始化完成與否，都能安全釋放資源
