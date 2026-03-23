---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 子視窗、工具視窗實作

## 設計原則（工控 + 低效能環境）

1. 不使用複雜 UI Framework，不依賴動畫與資料繫結。
2. 明確控制生命週期（Owner、Dispose、Hide）。
3. 不讓 Form 自行 new 自己，統一由呼叫端管理。
4. 嚴格區分「阻斷流程」與「非阻斷工具視窗」。

以下為 .NET Framework 4.6.2 ~ 4.8、WinForms 可直接使用的標準寫法。

## 1️⃣ 模態視窗（阻斷流程型）

### 特性

1. 使用 `ShowDialog()`
2. `StartPosition = CenterScreen`
3. `FormBorderStyle = FixedDialog`
4. `MaximizeBox = false`
5. `MinimizeBox = false`
6. 關閉後必須 Dispose

### Form 設計

```csharp
public partial class ActionForm : Form
{
    public ActionForm()
    {
        InitializeComponent();

        this.StartPosition = FormStartPosition.CenterScreen;
        this.FormBorderStyle = FormBorderStyle.FixedDialog;
        this.MaximizeBox = false;
        this.MinimizeBox = false;
    }
}
```

### 呼叫端寫法（推薦標準寫法）

```csharp
private void OpenActionForm()
{
    using (var form = new ActionForm())
    {
        form.ShowDialog(this);
    }
}
```

說明：

1. `using` 保證 Dispose
2. `this` 指定 Owner，避免跳到背景
3. 不要保留實例，避免記憶體殘留

## 2️⃣ 工具 / 監控視窗（非阻斷型）

## 種類 1：每次關閉都 Dispose

### 特性

1. `Show()`
2. `TopMost = true`
3. 關閉後 Dispose
4. 記錄位置
5. 下次重建時還原位置

### Form 設計

```csharp
public partial class MonitorForm : Form
{
    public MonitorForm()
    {
        InitializeComponent();
        this.TopMost = true;
    }
}
```

### 主畫面管理

```csharp
private Point? _monitorLastLocation;

private void OpenMonitorForm()
{
    var form = new MonitorForm();

    if (_monitorLastLocation.HasValue)
    {
        form.StartPosition = FormStartPosition.Manual;
        form.Location = _monitorLastLocation.Value;
    }
    else
    {
        form.StartPosition = FormStartPosition.CenterParent;
    }

    form.FormClosed += (s, e) =>
    {
        _monitorLastLocation = form.Location;
        form.Dispose();
    };

    form.Show(this);
}
```

說明：

1. 每次都 new
2. 關閉即銷毀
3. 記錄位置在主畫面欄位
4. 適合畫面狀態複雜、不希望殘留資源的監控頁

## 種類 2：Show / Hide，不 Dispose

適合：

1. 即時監控
2. 頻繁開關
3. 不希望重新初始化資料

### 主畫面管理

```csharp
private MonitorForm _monitorForm;
private Point? _monitorLastLocation;

private void ToggleMonitorForm()
{
    if (_monitorForm == null || _monitorForm.IsDisposed)
    {
        _monitorForm = new MonitorForm();
        _monitorForm.TopMost = true;

        if (_monitorLastLocation.HasValue)
        {
            _monitorForm.StartPosition = FormStartPosition.Manual;
            _monitorForm.Location = _monitorLastLocation.Value;
        }

        _monitorForm.FormClosing += MonitorForm_FormClosing;
        _monitorForm.Show(this);
        return;
    }

    if (_monitorForm.Visible)
        _monitorForm.Hide();
    else
        _monitorForm.Show();
}

private void MonitorForm_FormClosing(object sender, FormClosingEventArgs e)
{
    e.Cancel = true; // 不真正關閉
    _monitorLastLocation = _monitorForm.Location;
    _monitorForm.Hide();
}
```

說明：

1. 永遠不 Dispose
2. 攔截 Close 改為 Hide
3. 適合高頻率使用場景
4. 要確保主程式結束時手動 Dispose

## 工控強化建議

### 1️⃣ 防止重複開啟多個視窗

```csharp
if (_monitorForm != null && !_monitorForm.IsDisposed)
{
    _monitorForm.BringToFront();
    return;
}
```

### 2️⃣ 防止跑出螢幕外（雙螢幕/拔掉螢幕）

```csharp
private Point ValidateLocation(Point location, Size size)
{
    var screen = Screen.FromPoint(location);
    var bounds = screen.WorkingArea;

    if (!bounds.Contains(new Rectangle(location, size)))
    {
        return new Point(bounds.Left + 50, bounds.Top + 50);
    }

    return location;
}
```

## 工控環境選擇建議

模態視窗適用：

1. 設定參數
2. 手動確認
3. 需流程鎖定的動作

工具視窗（Dispose 型）適用：

1. 偶爾查看
2. 狀態不需保留
3. 記憶體敏感設備

工具視窗（Hide 型）適用：

1. 高頻率監控
2. 需保留畫面資料
3. 低延遲即時顯示

---

# WinForms（.NET Framework 4.8）由按鍵開啟／關閉 Form 的正確寫法

在 WinForms 架構中，`Form` 是獨立視窗物件，生命週期與 `UserControl` 不同。實務上要明確管理：

- 是否允許重複開啟
- 關閉後是否釋放資源
- 是否使用模態（Modal）或非模態（Modeless）

以下提供標準設計模式。

## 一、非模態視窗（可同時操作主視窗）推薦實務寫法

適用情境：

- 工具視窗
- 監控畫面
- 可與主畫面同時操作

### 設計原則

1. Form 只允許單一實例
2. 關閉時自動清除引用
3. 不使用 Dispose 後再 Show

### 範例

```csharp
public partial class MainForm : Form
{
    private SubForm _subForm;

    public MainForm()
    {
        InitializeComponent();
    }

    private void btnToggle_Click(object sender, EventArgs e)
    {
        if (_subForm == null || _subForm.IsDisposed)
        {
            _subForm = new SubForm();
            _subForm.FormClosed += (s, args) => _subForm = null;
            _subForm.Show(this);
        }
        else
        {
            _subForm.Close();
        }
    }
}
```

重點說明：

- `IsDisposed` 必須檢查
- `FormClosed` 事件中清除引用
- 不要呼叫 Hide 當作 Close（除非你要保留狀態）

## 二、模態視窗（ShowDialog）

適用情境：

- 設定視窗
- 必須完成動作才能回主畫面
- 需要 DialogResult

### 範例

```csharp
private void btnOpen_Click(object sender, EventArgs e)
{
    using (var form = new SubForm())
    {
        var result = form.ShowDialog(this);

        if (result == DialogResult.OK)
        {
            // 處理回傳結果
        }
    }
}
```

特性：

- 視窗會阻塞主視窗
- 關閉後自動 Dispose（因為 using）
- 不需要手動管理實例

## 三、只 Hide 不 Close（保留狀態）

適用情境：

- 需要保留 UI 狀態
- 頻繁開關

### 範例

```csharp
private SubForm _subForm;

private void btnToggle_Click(object sender, EventArgs e)
{
    if (_subForm == null)
    {
        _subForm = new SubForm();
        _subForm.FormClosing += (s, args) =>
        {
            args.Cancel = true;
            _subForm.Hide();
        };
        _subForm.Show(this);
    }
    else
    {
        if (_subForm.Visible)
            _subForm.Hide();
        else
            _subForm.Show(this);
    }
}
```

關鍵：

- 攔截 FormClosing
- 改為 Hide
- 不會 Dispose

缺點：

- 記憶體常駐
- 若有非受控資源要自行處理

## 四、架構層級建議（避免常見錯誤）

1. 不要 new 之後多次 ShowDialog 同一實例
2. 不要 Dispose 後再 Show
3. 不要讓子 Form 持有主 Form 強參考造成循環依賴
4. 若有長時間背景執行緒，關閉時必須正確停止

## 簡潔總結

情境 | 建議做法
一般工具視窗 | 單一實例 + Show / Close
設定對話框 | using + ShowDialog
高頻切換 | 攔截 FormClosing 改 Hide
需要釋放資源 | Close + Dispose

---

# WinForms 多視窗管理：Form Manager + Factory 架構（.NET Framework 4.8）

適用情境：

- 工控監控畫面
- 多設備視窗
- 禁止重複開啟
- 統一生命週期管理
- 可集中做例外與釋放控制

設計目標：

- 主畫面不直接 new Form
- 不允許同類型視窗重複開啟
- 關閉自動清理引用
- 可選擇保留或釋放實例

## 架構概念

MainForm
↓
IFormManager
↓
FormManager
↓
IFormFactory
↓
Concrete Form

核心思想：

- FormManager 管理「已開啟視窗」
- Factory 負責「建立視窗」
- 主畫面只呼叫 Manager

## 一、IFormFactory

```csharp
public interface IFormFactory
{
    T Create<T>() where T : Form;
}
```

## 二、基本 Factory 實作

```csharp
public class DefaultFormFactory : IFormFactory
{
    public T Create<T>() where T : Form
    {
        return Activator.CreateInstance<T>();
    }
}
```

若未來導入 Autofac，只需替換這層。

## 三、IFormManager

```csharp
public interface IFormManager
{
    void Show<T>(Form owner = null) where T : Form;
    void Close<T>() where T : Form;
    bool IsOpen<T>() where T : Form;
}
```

## 四、FormManager 實作（核心）

```csharp
public class FormManager : IFormManager
{
    private readonly IFormFactory _factory;
    private readonly Dictionary<Type, Form> _forms = new Dictionary<Type, Form>();

    public FormManager(IFormFactory factory)
    {
        _factory = factory;
    }

    public void Show<T>(Form owner = null) where T : Form
    {
        var type = typeof(T);

        if (_forms.TryGetValue(type, out var existing))
        {
            if (!existing.IsDisposed)
            {
                existing.BringToFront();
                return;
            }

            _forms.Remove(type);
        }

        var form = _factory.Create<T>();
        form.FormClosed += (s, e) => _forms.Remove(type);

        _forms[type] = form;

        if (owner != null)
            form.Show(owner);
        else
            form.Show();
    }

    public void Close<T>() where T : Form
    {
        var type = typeof(T);

        if (_forms.TryGetValue(type, out var form))
        {
            form.Close();
        }
    }

    public bool IsOpen<T>() where T : Form
    {
        var type = typeof(T);
        return _forms.TryGetValue(type, out var form) && !form.IsDisposed;
    }
}
```

特性：

- 同類型只會存在一個實例
- 關閉後自動移除
- 不會殘留已 Dispose 物件

## 五、MainForm 使用方式

```csharp
private IFormManager _formManager;

public MainForm()
{
    InitializeComponent();

    var factory = new DefaultFormFactory();
    _formManager = new FormManager(factory);
}

private void btnDevice_Click(object sender, EventArgs e)
{
    _formManager.Show<DeviceMonitorForm>(this);
}

private void btnCloseDevice_Click(object sender, EventArgs e)
{
    _formManager.Close<DeviceMonitorForm>();
}
```

主畫面完全不需要知道 new 或管理細節。

## 六、進階：改為 Hide 模式（保留狀態）

如果希望關閉按 X 只是 Hide：

在 FormManager 建立時改為：

```csharp
form.FormClosing += (s, e) =>
{
    if (e.CloseReason == CloseReason.UserClosing)
    {
        e.Cancel = true;
        form.Hide();
    }
};
```

此時 Close 需要改為：

```csharp
form.Hide();
```

但記憶體會常駐。

## 七、工控強化建議

1. 若 Form 內有背景執行緒，必須在 FormClosing 中安全停止
2. 若含 SerialPort / Socket，Close 前確保 Dispose
3. 不要讓 Form 直接 new 設備物件，由 DI 或 Factory 注入
4. 可擴充：
   Dictionary<string, Form>
   用設備 ID 區分多實例

## 八、延伸：多設備多實例版本（依 Key 管理）

如果是：

設備 A → 一個視窗
設備 B → 一個視窗

則改為：

Dictionary<string, Form>

由：

Show(string key)

控制每台設備獨立視窗。

如果你願意，我可以幫你做「設備 ID + 泛型 + DI + 背景工作安全關閉」的完整工控等級版本。

---

# FormManager 會增加工控環境負擔嗎？

結論：**正確設計下幾乎沒有負擔，反而降低風險。**

從工控實務角度分析：

## 記憶體負擔

1. FormManager 本質只是集中管理 reference
2. 不持有大量物件、不做背景執行緒
3. 不使用事件匯流排或複雜 DI 容器
4. 常駐成本 ≈ 幾個 reference + dictionary

在 .NET Framework 4.6.2~4.8 上，這種負擔可忽略。

## CPU 負擔

1. 只在開關視窗時執行
2. 不涉及 polling / timer
3. 不做反射或動態掃描

對低階工控電腦（Atom / Celeron / 舊 i3）影響可視為 0。

### 穩定性角度（反而是優點）

沒有集中管理時常見問題：

1. 重複開多個監控視窗
2. Owner 混亂導致 Z-order 異常
3. 忘記 Dispose
4. Hide 型視窗被 GC 回收誤判
5. 位置紀錄散落在各頁

FormManager 可降低：

1. 記憶體 leak 風險
2. 視窗管理錯亂
3. 維護成本

在工控系統中，「可控性」比「多 1 個 class」重要得多。

---

# 團隊學習曲線與排斥問題

這個問題比效能更關鍵。

## 若設計成過度工程化（會被排斥）

例如：

1. 泛型 + 反射
2. DI 自動掃描
3. 事件匯流
4. 生命週期 state machine
5. 抽象過多

工控團隊通常偏好：

1. 明確
2. 可預測
3. 好追蹤
4. Debug 容易

如果做成「像 WPF 框架那樣」，一定會被排斥。

## 正確的工控版 FormManager 應該長這樣

特性：

1. 不用 DI
2. 不用反射
3. 不用泛型工廠
4. 不藏邏輯
5. 呼叫方式明確
6. 可單步除錯

例如使用方式：

```csharp
FormManager.ShowModal<ActionForm>(this);
FormManager.ShowTool<MonitorForm>(this);
FormManager.TogglePersistent<MonitorForm>(this);
```

工程師能理解為：

「只是集中管理 new / show / dispose」

這樣學習成本極低。

## 實務團隊接受度分析

### 會排斥的情況

1. 舊專案硬塞進去
2. 文件不足
3. 命名不清楚
4. 過度抽象
5. 強迫全面改寫

### 容易接受的情況

1. 新專案一開始就使用
2. 有清楚範例
3. 只解決實際痛點
4. 不影響既有程式寫法

## 工控專案真實風險比較

### 沒有管理器的長期風險

1. 多人開發後視窗生命週期混亂
2. 不一致的 Dispose 策略
3. Tool 視窗定位錯亂
4. 主畫面關閉時殘留 Form

### 有簡單 FormManager 的風險

1. 多一個 class
2. 需要團隊知道「怎麼用」

顯然第二種風險小很多。

## 建議結論

如果專案：

小型 + 單人維護 → 不一定需要
中型以上 + 多人開發 → 強烈建議使用
長期維護 5~10 年 → 幾乎必須使用

在工控環境中，**穩定與可控優先於極簡。**

---

# 視窗開關管理有那麼多問題嗎？

## 簡短結論

一般商用軟體 → 不會那麼嚴重
24/7 工控系統 → 真的會出問題

差別在「使用時間尺度」。

## 為什麼一般桌面軟體沒事？

1. 每天會關閉程式
2. 使用時間短
3. 使用者會重開
4. 不追求 0 當機

記憶體小 leak 幾乎不會被察覺。

## 為什麼工控 24/7 會出問題？

時間累積效應。

假設：

- 每次開關窗漏 1 個 GDI 物件
- 每天開關 300 次
- 一個月 ≈ 9000 次

GDI leak 9000 → 畫面開始出現異常。

這不是理論，是 WinForms 常見真實問題。

## 實務常見問題類型

### 1️⃣ GDI 漏洞（最常見）

來源：

- Bitmap 未 Dispose
- Graphics 未釋放
- 自訂繪圖
- 圖表控制項

結果：

- 視窗無法建立
- 控制項不顯示
- 程式突然當掉

### 2️⃣ Handle 累積

來源：

- 重複 new Form
- 未 Dispose
- 重複建立 UserControl

症狀：

- 開窗變慢
- 切換畫面延遲
- 最終例外

### 3️⃣ UI Freeze

來源：

- DataReceived 直接操作 UI
- Invoke 死鎖
- 同步 I/O

症狀：

- 畫面卡死
- 滑鼠可動但按鈕無反應
- 現場人員只能重開機

### 4️⃣ 記憶體慢性上升

來源：

- Persistent Form 裡保留資料
- 事件未解除
- static 引用

症狀：

- 3 個月後記憶體暴增
- 32bit 直接爆 2GB 限制

## 那是不是一定要做那麼重？

看場景。

### 如果系統條件是

1. 每天有人關機
2. 使用者是工程師
3. 畫面數量少
4. 不頻繁開關

→ 可以簡化

### 如果系統條件是

1. 24/7
2. 操作人員不會除錯
3. 多畫面
4. 高頻率切換
5. 有圖表或即時監控

→ 不做保護遲早出問題

## 真實經驗數據

在工控專案中：

3~6 個月內
若無管理機制
幾乎一定出現：

- GDI 破 8000
- 偶發 UI 卡死
- 記憶體緩慢上升

## 重點不是「視窗管理」

真正問題是：

時間 × 累積 × 無人重啟

WinForms 本身並不是為 365 天不停運轉設計的。

## 你真正要問的是

是不是「每個專案都要上全套監控」？

答案：

不用。

## 工控分級建議

### Level 1（基礎）

- 正確 Dispose
- 不重複開窗
- Owner 管理

### Level 2（穩定）

- Thread-Safe
- Persistent 管理
- 位置安全

### Level 3（24/7）

- Freeze 監測
- GDI 監控
- 記憶體監控
- 例外隔離

## 我會反問你一個實務問題

你的系統：

- 每天大概開關視窗幾次？
- 是否有圖表 / 即時影像？
- 是否 32bit？
- 是否曾發生 UI 卡死？

如果你過去從未遇過：

- GDI 過高
- UI Freeze
- 記憶體上升

那代表目前規模還沒到臨界點。

## 最務實結論

視窗開關本身不是問題。

「無紀律的視窗生命週期」
+
「長時間不重啟」
+
「資源未監控」

才是問題。

如果你願意，我可以幫你做一個：

「最小化 24/7 必要防護版本」

只保留：

1. Dispose 管理
2. UI Freeze 偵測
3. GDI 監控

其他全部拿掉。

這樣會非常輕量，但仍足夠工控安全。

---

# 自訂繪圖專案專用的 24/7 WinForms 工控框架模板

整合了：

- FormManager（簡化版，單例管理 + Dispose）
- GDI / Handle / Memory 監控
- UI Freeze 偵測
- 全域例外隔離

以下是一個可直接落地的範例框架：

## 一、FormManager（簡化版）

```csharp
public static class FormManager
{
    private static readonly Dictionary<string, Form> _forms = new Dictionary<string, Form>();
    private static readonly object _lock = new object();

    public static T ShowForm<T>(bool persistent = false) where T : Form, new()
    {
        lock (_lock)
        {
            string key = typeof(T).FullName;

            if (_forms.ContainsKey(key))
            {
                var existing = _forms[key];
                if (persistent)
                {
                    existing.Show();
                    existing.BringToFront();
                    return (T)existing;
                }
                else
                {
                    existing.Close();
                    _forms.Remove(key);
                }
            }

            var form = new T();
            if (!persistent)
            {
                form.FormClosed += (s, e) =>
                {
                    form.Dispose();
                    _forms.Remove(key);
                };
            }

            _forms[key] = form;
            form.Show();
            return form;
        }
    }

    public static void CloseForm<T>() where T : Form
    {
        lock (_lock)
        {
            string key = typeof(T).FullName;
            if (_forms.TryGetValue(key, out var form))
            {
                form.Close();
            }
        }
    }
}
```

## 二、Safe UI Invoker

```csharp
public static class SafeUIInvoker
{
    public static void Invoke(Action action)
    {
        if (action == null) return;
        if (System.Windows.Forms.Application.OpenForms.Count == 0) return;

        var form = System.Windows.Forms.Application.OpenForms[0];
        if (form.InvokeRequired)
            form.Invoke(action);
        else
            action();
    }
}
```

## 三、UI Freeze 監控

```csharp
public static class UIFreezeMonitor
{
    private static System.Threading.Timer _timer;
    private static DateTime _lastUiResponse;
    private static int _timeoutMs = 5000;
    public static Action OnFreezeDetected;

    public static void Start(int checkIntervalMs = 1000)
    {
        _lastUiResponse = DateTime.UtcNow;
        _timer = new System.Threading.Timer(_ =>
        {
            SafeUIInvoker.Invoke(() => _lastUiResponse = DateTime.UtcNow);

            if ((DateTime.UtcNow - _lastUiResponse).TotalMilliseconds > _timeoutMs)
                OnFreezeDetected?.Invoke();

        }, null, 0, checkIntervalMs);
    }

    public static void Stop() => _timer?.Dispose();
}
```

## 四、資源監控（GDI / Handle / Memory）

```csharp
using System.Diagnostics;
using System.Runtime.InteropServices;

public static class ResourceMonitor
{
    [DllImport("user32.dll")]
    private static extern int GetGuiResources(IntPtr hProcess, int uiFlags);

    private const int GR_GDIOBJECTS = 0;
    private const int GR_USEROBJECTS = 1;

    public static int GetGdiCount() => GetGuiResources(Process.GetCurrentProcess().Handle, GR_GDIOBJECTS);
    public static int GetUserHandleCount() => GetGuiResources(Process.GetCurrentProcess().Handle, GR_USEROBJECTS);
    public static int GetHandleCount() => Process.GetCurrentProcess().HandleCount;
    public static long GetPrivateMemoryMB() => Process.GetCurrentProcess().PrivateMemorySize64 / 1024 / 1024;
}
```

## 五、穩定性監控整合

```csharp
public static class StabilityMonitor
{
    private static System.Threading.Timer _timer;
    public static Action<string> OnWarning;
    public static Action<string> OnCritical;

    public static void Start(int intervalMs = 10000)
    {
        _timer = new System.Threading.Timer(_ => Check(), null, 0, intervalMs);
    }

    private static void Check()
    {
        var gdi = ResourceMonitor.GetGdiCount();
        var handle = ResourceMonitor.GetHandleCount();
        var memory = ResourceMonitor.GetPrivateMemoryMB();

        if (gdi > 8000) OnCritical?.Invoke($"GDI HIGH: {gdi}");
        else if (gdi > 5000) OnWarning?.Invoke($"GDI WARNING: {gdi}");

        if (handle > 10000) OnCritical?.Invoke($"HANDLE HIGH: {handle}");
        if (memory > 1500) OnCritical?.Invoke($"MEMORY HIGH: {memory} MB");
    }

    public static void Stop() => _timer?.Dispose();
}
```

## 六、全域例外隔離

```csharp
public static class ExceptionHandler
{
    public static void Register()
    {
        Application.ThreadException += (s, e) =>
        {
            LogError("ThreadException", e.Exception);
        };

        AppDomain.CurrentDomain.UnhandledException += (s, e) =>
        {
            LogError("UnhandledException", e.ExceptionObject as Exception);
        };
    }

    private static void LogError(string type, Exception ex)
    {
        // 這裡可寫 Log 或顯示警告
        Debug.WriteLine($"{type}: {ex}");
    }
}
```

## 七、主程式整合範例

```csharp
static class Program
{
    [STAThread]
    static void Main()
    {
        ExceptionHandler.Register();
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);

        UIFreezeMonitor.Start();
        StabilityMonitor.Start();

        UIFreezeMonitor.OnFreezeDetected = () =>
        {
            Debug.WriteLine("UI Freeze Detected!");
        };

        StabilityMonitor.OnCritical = msg =>
        {
            Debug.WriteLine($"Critical: {msg}");
        };

        Application.Run(new MainForm());
    }
}
```

### 特點

1. 適合自訂繪圖 / 即時曲線 / 即時影像
2. 24/7 長時間運行，UI Freeze、GDI、Memory 有監控
3. 簡化 FormManager 保持單例 / Dispose
4. 全域例外捕捉避免主程式崩潰
5. 可在現場非工程師操作下穩定運行
