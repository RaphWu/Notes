# 目錄

- [目錄](#目錄)
- [UserControl 實現 Dispose 的方式](#usercontrol-實現-dispose-的方式)
- [Program, AppStartupContext](#program-appstartupcontext)
- [取得取得控制項所在的Form](#取得取得控制項所在的form)

---

# UserControl 實現 Dispose 的方式

問題點：

- 因為 Windows Forms Designer 確實會覆蓋 Designer.cs 文件的內容，這意味著在 Designer.cs 修改的內容會在下次使用設計器時被覆蓋。
- Designer.cs 內已有 `protected override void Dispose(bool disposing) {}`，再繼承 IDispose 會有重覆命名的問題。

```csharp
public partial class Monitor : UserControl
{
    public Monitor()
    {
        InitializeComponent();
        this.Disposed += OnDisposed;
    }

    #region Dispose

    /// <summary>
    /// 當 UserControl 被 Dispose 時執行的清理邏輯
    /// </summary>
    private void OnDisposed(object sender, EventArgs e)
    {
        try
        {
            // 釋放資源
        }
        catch (Exception ex)
        {
            // 記錄清理錯誤但不拋出，避免影響正常的 Dispose 流程
            log.Information("...");
        }
    }

    #endregion Dispose
}
```

[🔝](#目錄)

---

# Program, AppStartupContext

- 應用程式只有一個實例
- 攔截異常
- AppStartupContext 呼叫

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

    // 用於確保單一實例執行的 Mutex
    private static Mutex _mutex;

    /// <summary>
    /// 應用程式的主要進入點。
    /// </summary>
    [STAThread]
    static void Main()
    {
        // 確保單一實例執行
        _mutex = new Mutex(true, "Calin.ScrewFastening.UniqueAppMutex", out bool createdNew);
        if (!createdNew)
        {
            // 找到現有程式，BringToFront
            var current = Process.GetCurrentProcess();
            var existing = Process.GetProcessesByName(current.ProcessName)
                                    .FirstOrDefault(p => p.Id != current.Id);
            if (existing != null)
            {
                MessageBox.Show("ScrewFastening 已在執行中，將切換到已開啟的視窗。", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
                NativeMethods.ShowWindow(existing.MainWindowHandle, 5); // SW_SHOW
                NativeMethods.SetForegroundWindow(existing.MainWindowHandle);
            }
            return;
        }

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);

        // 攔截異常
        Application.ThreadException += (s, e) =>
        {
            File.AppendAllText("error.log", DateTime.Now + " UI: " + e.Exception + Environment.NewLine);
            MessageBox.Show(e.Exception.ToString(), "例外 (UI)");
        };
        AppDomain.CurrentDomain.UnhandledException += (s, e) =>
        {
            File.AppendAllText("error.log", DateTime.Now + " Non-UI: " + e.ExceptionObject + Environment.NewLine);
            MessageBox.Show(e.ExceptionObject?.ToString(), "例外 (Non-UI)");
        };

        Application.Run(new AppStartupContext());

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
    private readonly CancellationTokenSource _cts = new CancellationTokenSource();
    private readonly SplashScreen _splash;
    private IContainer _container = default;
    private ILogger<AppStartupContext> _logger;

    /// <summary>
    /// 初始化 <see cref="ApplicationContext"/> 類別的新執行個體。
    /// </summary>
    public AppStartupContext()
    {
        // 顯示啟動畫面
        _splash = new SplashScreen();
        _splash.AbortClicked += Splash_AbortClicked;
        _splash.Show();
        _splash.Refresh();

        if (!AppPaths.IsInitialized)
            AppPaths.Init("Calin", "ScrewFastening", isDocument: true);
        AppPaths.EnsureDirectoryExists(AppFolderNames.Config);
        AppPaths.EnsureDirectoryExists(AppFolderNames.Data);

        // 初始化日誌服務，必須最先執行
        string logDir = AppPaths.Logs;
        SplashMessenger.Post("初始化日誌服務...");
        var loggerFactory = new SerilogLoggerFactoryBuilder()
            .SetMinimumLevel(LogEventLevel.Debug)
            .AddDebug()
            .AddFile(Path.Combine(logDir, "ScrewFastening.log"))
            .AddJsonFile(Path.Combine(logDir, "ScrewFastening.json"))
            .EnableAllEnrichers("ScrewFastening")
            .Build();
        LoggingBridge.Initialize(loggerFactory);
        _logger = LoggingBridge.CreateLogger<AppStartupContext>();

        StartInitialization();
    }

    /// <summary>
    /// 開始初始化應用程式的核心邏輯。
    /// </summary>
    private void StartInitialization()
    {
        // 在背景執行緒中初始化應用程式
        _ = Task.Run(() =>
        {
            try
            {
                _logger.LogInformation("初始化核心模組");
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
                _logger.LogInformation("ScrewFastening 由 CancellationTokenSource 結束");
                ExitOnUiThread();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, $"ScrewFastening異常結束!\n\n{ex.Message}");
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

        // 初始化 DI 容器
        var builder = new ContainerBuilder();

        builder.RegisterType<MainForm>();

        // ...

        token.ThrowIfCancellationRequested();

        _container = builder.Build();
    }

    /// <summary>
    /// 處理啟動退出事件，並執行應用程式的關閉邏輯。
    /// </summary>
    private void Splash_AbortClicked(object sender, EventArgs e)
    {
        _cts.Cancel(); // 如果初始化還在跑，取消它
        Shutdown();
    }

    /// <summary>
    /// 處理主視窗關閉事件，並執行應用程式的關閉邏輯。
    /// </summary>
    private void MainForm_FormClosed(object sender, FormClosedEventArgs e)
    {
        _logger.LogInformation("ScrewFastening正常結束");
        Shutdown();
    }

    /// <summary>
    /// 關閉應用程式，釋放資源並停止所有操作。
    /// </summary>
    private void Shutdown()
    {
        // 取消初始化或背景 Task
        _cts.Cancel();

        _container?.Dispose();
        Log.CloseAndFlush();

        // 解除 Splash
        if (_splash != null && !_splash.IsDisposed)
        {
            _splash.AbortClicked -= Splash_AbortClicked;
            _splash.Close();
            _splash.Dispose();
        }

        // 釋放 CTS
        _cts.Dispose();

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

[🔝](#目錄)

---

# 取得取得控制項所在的Form

[Control.FindForm Method](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.control.findform)

```csharp
 var ownerForm = this.FindForm();
 if (ownerForm == null)
 {
     MessageBox.Show("無法取得父視窗", "錯誤", MessageBoxButtons.OK, MessageBoxIcon.Error);
     return;
 }
```

[🔝](#目錄)
