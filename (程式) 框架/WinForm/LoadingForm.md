---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - WinForm
---

# ✅ 常見需求：WinForm 中執行耗時工作 + 顯示等待動畫

在 WinForm 做「讀寫資料庫、計算、匯入匯出」這類耗時工作時，如果畫面沒有回應，使用者會以為程式死掉，所以顯示等待動畫是非常必要的。

以下是幾種 **業界常用且實用的做法**，並分析優缺點。

## ✅ 方法一：使用獨立的「等待中」小視窗（推薦）

✔ 開一個小 Form 或 Dialog，只顯示進度文字或 GIF 動畫
✔ 背景用 `Task.Run` 或 `async/await` 執行耗時工作
✔ 工作完成後關閉視窗

```csharp
using (var loading = new LoadingForm())
{
    loading.Show();       // 顯示等待視窗
    loading.Refresh();    // 確保顯示

    await Task.Run(() =>
    {
        // 耗時工作
        DoDatabaseWork();
    });
}
```

**優點：**
✅ 使用者明確知道「正在處理」
✅ 主畫面仍可鎖定或不鎖定（你可選擇 ShowDialog 或 Show）
✅ UI/邏輯分離、乾淨

**缺點：**
⚠ 需要另外設計一個 LoadingForm（但只做一次即可重用）

## ✅ 方法二：在主畫面上蓋一個 Panel + GIF

✔ Panel 半透明、置中 GIF/文字
✔ `Task.Run` 執行背景工作
✔ 工作完成後隱藏 Panel

**優點：**
✅ 不需要額外視窗
✅ 整體體驗像 Web App

**缺點：**
⚠ 需要處理控制項層級、Disable 背後控制項
⚠ GIF 有時不會動（需要使用 PictureBox + Enabled=true）

## ✅ 方法三：內建 ProgressBar + BackgroundWorker (舊方法，不建議)

BackgroundWorker 是 WinForm 時代的官方寫法，但現在已被 `async/await` 取代。

## ✅ 方法四：第三方 Loading 套件

例如：

- DevExpress WaitForm
- Telerik RadWaitingBar
- Guna UI loading form

**優點：**
✅ 外觀漂亮、動畫順暢
✅ 幾乎不用寫 UI 程式碼

**缺點：**
⚠ 套件需付費或增加專案體積

## ✅ 方法五：不開畫面，用 Cursor = Cursors.WaitCursor (最簡單，但最爛)

```csharp
this.Cursor = Cursors.WaitCursor;
DoWork(); // ⚠ 同執行緒 -> UI 卡死
this.Cursor = Cursors.Default;
```

**缺點嚴重**：UI 仍會停住，使用者看起來像程式當掉。

## ✅ 最推薦做法（現代 WinForms）

👉 **LoadingForm + async/await + Task.Run**

這是目前最安全、最易維護、最容易重用、最不會造成 UI 卡住的方式。

---

# 各種選擇

## ✅ 好的！下一步請告訴我你想要哪一種實作方式

我可以直接幫你寫範例，你只要選一個：

### ✅ 1. 另外開一個 LoadingForm（小視窗顯示等待動畫）→ **最推薦**

- 外觀乾淨、容易重用
- async/await + Task.Run 不會卡 UI

### ✅ 2. 在主畫面上蓋一個半透明 Panel + GIF

- 不開新視窗
- 感覺像 Web 的 loading 遮罩

### ✅ 3. 使用內建 ProgressBar + BackgroundWorker（傳統方式）

- 舊專案可能用這種

### ✅ 4. 給你一個工具方法（自動顯示等待視窗 + 執行工作）

- 只需要呼叫一行，最省事

👉 請直接回覆：
**1 或 2 或 3 或 4**
我就會依你選的方式寫出完整實作程式碼給你！

## ✅ 選擇 1：LoadingForm 小視窗等待動畫（最推薦）

我會幫你做一個 **可重用的 LoadingForm**，並示範如何在背景執行耗時工作。
接下來我會提供兩個部分：

### ✅ 第一部分：設計 LoadingForm

- 只顯示「請稍候…」或 GIF 動畫
- 無邊框、置中顯示
- 顯示時避免卡住（用 `Show()` + `Refresh()`）

### ✅ 第二部分：主畫面如何呼叫

- 使用 `async/await`
- 用 `Task.Run()` 執行真正的耗時工作
- 完成後關閉 LoadingForm

在我開始寫程式碼前，想確認兩個小細節：

### ❓ A. LoadingForm 外觀你希望是

1. 只顯示文字「請稍候…」
2. 文字 + 進度環 或 GIF 動畫
3. 任意簡單即可，我之後自己美化

### ❓ B. LoadingForm 顯示方式

1. Show()（非模態，主視窗可選擇要不要鎖住）
2. ShowDialog()（模態，主視窗無法操作）

👉 只要回覆：
- A 選 1/2/3
- B 選 1/2
我就會直接貼完整可執行範例！

## ✅ 確認設定如下

### ✅ 外觀 (A2)

✔ 文字 + 進度環 或 GIF 動畫

### ✅ 顯示方式 (B1)

✔ `Show()`（非模態顯示，但我會示範如何鎖住主畫面避免亂點）

接下來我會提供完整可執行範例，內容包含：

### ✅ 1. LoadingForm 設計

- 無邊框、置中
- PictureBox 放 GIF 或 ProgressRing (如使用 WinForms 支援的動畫)
- Label 顯示「請稍候…」

### ✅ 2. 主畫面呼叫方式（重點示範）

```csharp
var loading = new LoadingForm();
loading.Show();
loading.Refresh();

await Task.Run(() =>
{
    // 耗時作業
});

loading.Close();
```

### ✅ 3. 如何避免主畫面被操作（可選）

- 暫時 Disable 主畫面
- 完成後再 Enable

我會依照這個方向寫程式碼。

👉 如果你已經準備好，我現在就會貼出完整實作！
（包含設計、程式碼、使用範例）

---

# LoadingForm + 背景執行完整範例（可直接使用）

## ✅ Step 1：建立 LoadingForm

```csharp
public partial class LoadingForm : Form
{
    public LoadingForm()
    {
        InitializeComponent();
        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;
    }

    private void LoadingForm_Load(object sender, EventArgs e)
    {
        // 讓動畫正常播放，避免凍結
        this.Refresh();
    }
}
```

### Designer 可以這樣配置

- Form 大小約 200x100
- 加入 PictureBox (SizeMode = Zoom) 放 GIF
- 加入 Label 顯示「請稍候…」
- PictureBox 和 Label 置中

👉 GIF 可放在 Resources，PictureBox.Image = Properties.Resources.YourGif

## ✅ Step 2：主畫面使用方式 (async/await)

```csharp
private async void button1_Click(object sender, EventArgs e)
{
    var loading = new LoadingForm();

    // 選擇性：鎖住主畫面
    this.Enabled = false;

    loading.Show();
    loading.Refresh();   // 確保顯示出來

    try
    {
        await Task.Run(() =>
        {
            // 模擬耗時工作 (例如資料庫操作)
            System.Threading.Thread.Sleep(3000);
            // DoDatabaseWork();
        });
    }
    finally
    {
        loading.Close();
        this.Enabled = true;   // 恢復主畫面
    }
}
```

## ✅ 為什麼這種寫法正確？

✅ 使用 Task.Run → 耗時工作在背景執行
✅ 使用 async/await → 不會卡 UI
✅ loading.Show() + Refresh() → 動畫能動
✅ try/finally → 即使發生例外也會關閉視窗
✅ this.Enabled = false → 防止使用者亂點主畫面（可選）

## ✅ 想要更進階？我可以再幫你做

- ✅ 自動置中於主 Form (多螢幕也正確)
- ✅ 支援取消 (加入取消按鈕 + CancellationToken)
- ✅ 支援顯示進度百分比
- ✅ 做成 Helper：只要呼叫一行程式即可顯示 + 執行 + 關閉

## ✅ 下一步

如果你希望我提供 **完整設計（含 Designer 佈局程式碼）** 或 **改造成可重用的 Helper 方法**
請告訴我：

👉 是否需要 **完整 Designer 程式碼**？
👉 或是希望我幫你包成 **通用的 LoadingHelper 類別**？

---

# 通用 LoadingHelper 類別

使用方式會像這樣，非常簡單：

```csharp
await LoadingHelper.RunAsync(this, () =>
{
    // 耗時工作，如：讀寫資料庫
});
```

✅ 自動顯示等待視窗 (LoadingForm)
✅ 自動鎖定主畫面
✅ 自動背景執行
✅ 自動關閉視窗
✅ 自動處理例外狀況（不會卡視窗）
✅ 可重複使用，不用每次寫一堆重複程式碼

## ✅ 我會提供以下內容

### 1️⃣ LoadingForm (簡單含動畫/文字)

### 2️⃣ LoadingHelper 類別 (核心邏輯)

### 3️⃣ 使用範例 (主畫面呼叫方式)

我會直接貼出完整可用程式碼，你只要 Copy 到專案即可。

✅ 如果你沒有其它特別需求，我會先用：

- `FormBorderStyle.None`
- 置中顯示
- PictureBox + GIF + Label「請稍候…」
- Show() + async/await + Task.Run

👉 若你要顯示不同文字或傳入自訂內容，也可以在 Helper 中加入參數，我也能幫你做。

---

# 完整可重用方案：LoadingForm + LoadingHelper

此方案讓你只要寫一行程式，就能在執行耗時工作時自動顯示等待動畫、背景執行、關閉畫面並解鎖 UI。

## ✅ 1️⃣ 建立 LoadingForm (簡單等候畫面)

```csharp
public partial class LoadingForm : Form
{
    public LoadingForm()
    {
        InitializeComponent();

        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;
    }

    private void LoadingForm_Load(object sender, EventArgs e)
    {
        this.Refresh(); // 確保動畫正常開始
    }
}
```

### 🔹 Designer 建議配置

- Form Size：200x100
- 加入 PictureBox（SizeMode = Zoom）→ 顯示 GIF
- 加入 Label，文字「請稍候…」
- 控件都置中排版

## ✅ 2️⃣ 建立 LoadingHelper 類別（核心）

```csharp
public static class LoadingHelper
{
    public static async Task RunAsync(Form owner, Action action)
    {
        var loading = new LoadingForm();
        try
        {
            owner.Enabled = false;
            loading.Show(owner);
            loading.Refresh();

            await Task.Run(action);
        }
        finally
        {
            loading.Close();
            owner.Enabled = true;
        }
    }

    public static async Task<TResult> RunAsync<TResult>(Form owner, Func<TResult> func)
    {
        var loading = new LoadingForm();
        try
        {
            owner.Enabled = false;
            loading.Show(owner);
            loading.Refresh();

            return await Task.Run(func);
        }
        finally
        {
            loading.Close();
            owner.Enabled = true;
        }
    }
}
```

✅ 提供 Action 與 Func 兩種版本
✅ 自動處理 Show / Close / Refresh / Enabled
✅ 即使發生例外也保證關閉等待畫面

## ✅ 3️⃣ 主畫面使用範例 (超簡單)

```csharp
private async void button1_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, () =>
    {
        // 模擬耗時工作（寫資料庫、讀檔等）
        System.Threading.Thread.Sleep(3000);
    });

    MessageBox.Show("完成！");
}
```

或取得回傳值：

```csharp
private async void button2_Click(object sender, EventArgs e)
{
    int result = await LoadingHelper.RunAsync(this, () =>
    {
        Thread.Sleep(2000);
        return 123;
    });

    MessageBox.Show($"結果：{result}");
}
```

## ✅ 4️⃣ 優點整理

✅ 一行呼叫 → 自動顯示 + 背景執行 + 關閉
✅ 不會卡 UI
✅ 可重用全專案
✅ 支援回傳值
✅ 例外狀況也安全

## ✅ 5️⃣ 下一步建議

目前這個版本是「固定文字 + 固定版型」。
如果你希望更進階，我可以幫你：

✅ 傳入自訂訊息 (例：「正在匯入資料…」)
✅ 支援取消按鈕 + CancellationToken
✅ 支援顯示進度百分比
✅ 自動置中於主畫面 (支援多螢幕)
✅ 使用委派更新 UI 訊息

---

# 在 LoadingHelper 中加入傳入自訂訊息、支援 CancellationToken、支援顯示進度百分比

這代表我們需要設計一個具備互動能力的 LoadingForm，並且 LoadingHelper 要能：
- 建立 CancellationTokenSource 並傳給執行任務的邏輯
- 提供 IProgress 或回呼事件讓背景任務回報進度
- 顯示訊息與進度
- 處理使用者按下取消後的流程（取消 token + UI 顯示）

## 我會這樣設計

### ✅ 一、LoadingForm (有以下元件)

- Label：顯示狀態訊息（可自訂）
- ProgressBar 或 Label 顯示百分比
- Cancel 按鈕（按下後觸發取消事件）
- 無邊框、置中、無法操作背景視窗（或 Disable owner）

### ✅ 二、LoadingHelper.RunAsync(...)

```csharp
await LoadingHelper.RunAsync(
    owner: this,
    message: "正在匯入資料…",
    work: async (token, progress) =>
    {
        for (int i = 0; i < 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50);
            progress.Report(i + 1);
        }
    });
```

### ✅ 三、Helper 自動處理

- 顯示 LoadingForm
- 建立 CancellationTokenSource
- 建立 Progress
- 綁定事件：Progress.Report → 更新 UI
- 綁定事件：Cancel button → CancellationTokenSource.Cancel()
- 等待工作完成或被取消
- 關閉 LoadingForm

## ✅ 在我開始編寫完整程式碼前，最後確認 2 件事

### 1️⃣ 進度條顯示方式

你要用哪一種？
- ProgressBar 控制項（標準 WinForms）
- 文字顯示「50%」
- 兩者一起

👉 **我建議：ProgressBar + 百分比文字**
如：██████▒▒▒▒▒▒ 60%

### 2️⃣ LoadingForm 外觀

- 保持簡潔（白底 + 小動畫 + 文字 + 進度 + 取消）
- 還是你要 Sunny.UI 樣式？
    （你前面 code 有使用 Sunny.UI，如果要一致，我可以用 Sunny.UI 的 UIForm / UIProgress / UIButton）

---

# 一套完整、可重複使用的通用 Loading 系統

包含：

## ✅ 一、LoadingForm (Sunny.UI 版)

- UIForm 或 UIPanel
- UILabel 顯示自訂訊息
- UIProgressBar 顯示進度
- UILabel 顯示百分比
- UIButton（取消）

## ✅ 二、LoadingHelper

- `RunAsync(owner, message, work)`
- 建立 CancellationTokenSource
- 建立 Progress
- 綁定取消按鈕事件
- 自動處理顯示 / 關閉視窗
- 自動 Disable / Enable owner
- 把 token + progress 傳給 work

## ✅ 三、使用方式（超簡潔）

```csharp
await LoadingHelper.RunAsync(
    this,
    "正在匯入資料…",
    async (token, progress) =>
    {
        for (int i = 0; i < 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50);
            progress.Report(i + 1);
        }
    });
```

我會確保以下特性：

- ✅ Thread-safe UI 更新 (Invoke 處理)
- ✅ 使用 async/await，不會卡 UI
- ✅ 可取消（ThrowIfCancellationRequested 即可）
- ✅ 完成或取消後自動關閉
- ✅ 可重用，不依賴特定 Form
- ✅ 可擴充（未來要顯示文字、圖片、Spinner 也可以）

# Sunny.UI 通用 LoadingHelper 完整實作範例（Progress + 百分比 + 取消）

以下程式碼包含三個部分：

1. **LoadingForm**（UIForm）
2. **LoadingHelper**（通用方法）
3. **使用範例**

## 1️⃣ LoadingForm（Sunny.UI 版）

```csharp
using Sunny.UI;
using System;
using System.Threading;
using System.Windows.Forms;

public partial class LoadingForm : UIForm
{
    private CancellationTokenSource _cts;
    private IProgress<int> _progress;

    public CancellationToken Token => _cts?.Token ?? CancellationToken.None;
    public IProgress<int> Progress => _progress;

    public LoadingForm(string message, CancellationTokenSource cts, IProgress<int> progress)
    {
        InitializeComponent();

        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;

        lblMessage.Text = message;
        _cts = cts;
        _progress = progress;

        btnCancel.Click += BtnCancel_Click;
    }

    private void BtnCancel_Click(object sender, EventArgs e)
    {
        btnCancel.Enabled = false; // 防止重複點擊
        _cts?.Cancel();
        lblMessage.Text = "正在取消…";
    }

    public void ReportProgress(int value)
    {
        if (InvokeRequired)
        {
            this.Invoke(new Action(() => ReportProgress(value)));
            return;
        }
        progressBar.Value = value;
        lblPercent.Text = $"{value}%";
    }
}
```

### Designer 建議配置（Sunny.UI 元件）

- `UILabel lblMessage`：顯示自訂訊息
- `UIProgressBar progressBar`：顯示進度
- `UILabel lblPercent`：顯示百分比
- `UISymbolButton btnCancel`：取消按鈕
- 控件置中排版，UIForm 尺寸約 300x120

## 2️⃣ LoadingHelper（通用方法）

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public static class LoadingHelper
{
    public static async Task RunAsync(Form owner, string message, Func<CancellationToken, IProgress<int>, Task> work)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) =>
        {
            loadingForm?.ReportProgress(value);
        };

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            await work(cts.Token, progress);
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }

    public static async Task<TResult> RunAsync<TResult>(Form owner, string message, Func<CancellationToken, IProgress<int>, Task<TResult>> work)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) =>
        {
            loadingForm?.ReportProgress(value);
        };

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            return await work(cts.Token, progress);
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
            return default;
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }
}
```

## 3️⃣ 使用範例（主畫面）

```csharp
private async void btnImport_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, "正在匯入資料…", async (token, progress) =>
    {
        for (int i = 0; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50); // 模擬耗時工作
            progress.Report(i);
        }
    });

    MessageBox.Show("匯入完成！");
}
```

或取得回傳值：

```csharp
private async void btnCalculate_Click(object sender, EventArgs e)
{
    int result = await LoadingHelper.RunAsync(this, "計算中…", async (token, progress) =>
    {
        int sum = 0;
        for (int i = 0; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(30);
            sum += i;
            progress.Report(i);
        }
        return sum;
    });

    MessageBox.Show($"計算結果：{result}");
}
```

## ✅ 特性總結

- 自訂訊息
- 進度條 + 百分比文字
- 支援取消（CancellationToken）
- async/await 執行背景工作
- 自動 Disable / Enable 主視窗
- 完成或取消後自動關閉等待畫面
- 可重複使用，整個專案通用

---

# 動態更新訊息版本（Sunny.UI LoadingHelper）

這個版本允許你在背景工作中，不只回報進度百分比，還可以更新 Label 文字，例如：「正在匯入第 10 筆…」。

## 1️⃣ LoadingForm 改版

```csharp
using Sunny.UI;
using System;
using System.Threading;
using System.Windows.Forms;

public partial class LoadingForm : UIForm
{
    private CancellationTokenSource _cts;
    private IProgress<int> _progress;
    private IProgress<string> _messageProgress;

    public CancellationToken Token => _cts?.Token ?? CancellationToken.None;
    public IProgress<int> Progress => _progress;
    public IProgress<string> MessageProgress => _messageProgress;

    public LoadingForm(string message, CancellationTokenSource cts, IProgress<int> progress, IProgress<string> messageProgress)
    {
        InitializeComponent();

        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;

        lblMessage.Text = message;
        _cts = cts;
        _progress = progress;
        _messageProgress = messageProgress;

        btnCancel.Click += BtnCancel_Click;
        _messageProgress.ProgressChanged += (s, msg) =>
        {
            if (InvokeRequired)
            {
                this.Invoke(new Action(() => lblMessage.Text = msg));
            }
            else
            {
                lblMessage.Text = msg;
            }
        };
    }

    private void BtnCancel_Click(object sender, EventArgs e)
    {
        btnCancel.Enabled = false;
        _cts?.Cancel();
        lblMessage.Text = "正在取消…";
    }

    public void ReportProgress(int value)
    {
        if (InvokeRequired)
        {
            this.Invoke(new Action(() => ReportProgress(value)));
            return;
        }
        progressBar.Value = value;
        lblPercent.Text = $"{value}%";
    }
}
```

## 2️⃣ LoadingHelper 改版

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public static class LoadingHelper
{
    public static async Task RunAsync(Form owner, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task> work)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) => loadingForm?.ReportProgress(value);

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress, messageProgress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            await work(cts.Token, progress, messageProgress);
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }

    public static async Task<TResult> RunAsync<TResult>(Form owner, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task<TResult>> work)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) => loadingForm?.ReportProgress(value);

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress, messageProgress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            return await work(cts.Token, progress, messageProgress);
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
            return default;
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }
}
```

## 3️⃣ 使用範例（動態更新訊息 + 進度）

```csharp
private async void btnImport_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, "正在匯入資料…", async (token, progress, messageProgress) =>
    {
        for (int i = 1; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50); // 模擬耗時工作

            progress.Report(i);
            messageProgress.Report($"正在匯入第 {i} 筆資料…");
        }
    });

    MessageBox.Show("匯入完成！");
}
```

## ✅ 特性總結

- 自訂訊息可動態更新
- 支援取消按鈕 + CancellationToken
- 支援進度條 + 百分比文字
- async/await 執行背景工作，不卡 UI
- 自動 Disable / Enable 主視窗
- 可重複使用、全專案通用

---

# 自動取消超時功能

這個版本可以在指定時間（例如 30 秒）自動取消耗時任務，並提示使用者「操作超時」。

## 1️⃣ LoadingHelper 新增 Timeout 支援

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public static class LoadingHelper
{
    public static async Task RunAsync(Form owner, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task> work, int timeoutMs = 0)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) => loadingForm?.ReportProgress(value);

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress, messageProgress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            Task workTask = work(cts.Token, progress, messageProgress);

            if (timeoutMs > 0)
            {
                var timeoutTask = Task.Delay(timeoutMs, cts.Token);
                var completed = await Task.WhenAny(workTask, timeoutTask);
                if (completed == timeoutTask)
                {
                    cts.Cancel();
                    MessageBox.Show(owner, "操作超時，已自動取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                }
                await workTask; // 確保例外 propagate
            }
            else
            {
                await workTask;
            }
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }

    public static async Task<TResult> RunAsync<TResult>(Form owner, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task<TResult>> work, int timeoutMs = 0)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        LoadingForm loadingForm = null;

        progress.ProgressChanged += (s, value) => loadingForm?.ReportProgress(value);

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress, messageProgress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            Task<TResult> workTask = work(cts.Token, progress, messageProgress);

            if (timeoutMs > 0)
            {
                var timeoutTask = Task.Delay(timeoutMs, cts.Token);
                var completed = await Task.WhenAny(workTask, timeoutTask);
                if (completed == timeoutTask)
                {
                    cts.Cancel();
                    MessageBox.Show(owner, "操作超時，已自動取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                }
                return await workTask; // 確保結果或例外 propagate
            }
            else
            {
                return await workTask;
            }
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
            return default;
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }
}
```

## 2️⃣ 使用範例（30 秒超時）

```csharp
private async void btnImport_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, "正在匯入資料…", async (token, progress, messageProgress) =>
    {
        for (int i = 1; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(500); // 模擬耗時工作

            progress.Report(i);
            messageProgress.Report($"正在匯入第 {i} 筆資料…");
        }
    }, timeoutMs: 30000); // 30秒超時
}
```

## ✅ 特性總結

- 動態更新訊息
- 支援進度條 + 百分比文字
- 支援取消按鈕
- 支援自動超時取消
- async/await 背景執行
- 自動 Disable / Enable 主視窗
- 完成或取消後自動關閉 LoadingForm
- 可重複使用，整個專案通用

---

# 簡潔美觀 LoadingForm 範例（白底 + 小動畫 + 文字 + 進度 + 取消）

這個版本設計保持簡單、清爽，可以直接用於對話框，也可以改放在 StatusBar 或 Panel 中使用。

## 1️⃣ LoadingForm Designer 配置（Sunny.UI）

- **Form 屬性**
    - UIForm 或 Form
    - FormBorderStyle = None
    - StartPosition = CenterParent
    - ShowInTaskbar = false
    - TopMost = true
    - BackColor = 白色
- **控件**
    - **PictureBox / UISymbolButton（小動畫）**
        - 顯示 GIF 或簡單旋轉圖示
        - SizeMode = Zoom
        - 置中
    - **UILabel lblMessage**
        - 顯示訊息文字，如「正在處理…」
        - 字體稍大、黑色
        - 置中於 PictureBox 下方
    - **UIProgressBar progressBar**
        - 顯示進度
        - Style = Continuous 或 Sunny.UI 預設樣式
        - 寬度可稍微比文字寬
    - **UILabel lblPercent**
        - 顯示百分比文字，例如「50%」
        - 置於 progressBar 右側或上方
    - **UISymbolButton btnCancel**
        - 顯示「取消」
        - Anchor = BottomRight 或右下
        - 鼠標手形

## 2️⃣ LoadingForm 程式碼範例

```csharp
using Sunny.UI;
using System;
using System.Threading;
using System.Windows.Forms;

public partial class LoadingForm : UIForm
{
    private CancellationTokenSource _cts;
    private IProgress<int> _progress;
    private IProgress<string> _messageProgress;

    public CancellationToken Token => _cts?.Token ?? CancellationToken.None;
    public IProgress<int> Progress => _progress;
    public IProgress<string> MessageProgress => _messageProgress;

    public LoadingForm(string message, CancellationTokenSource cts, IProgress<int> progress, IProgress<string> messageProgress)
    {
        InitializeComponent();

        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;
        this.BackColor = System.Drawing.Color.White;

        lblMessage.Text = message;
        _cts = cts;
        _progress = progress;
        _messageProgress = messageProgress;

        btnCancel.Click += BtnCancel_Click;
        _messageProgress.ProgressChanged += (s, msg) =>
        {
            if (InvokeRequired)
            {
                this.Invoke(new Action(() => lblMessage.Text = msg));
            }
            else
            {
                lblMessage.Text = msg;
            }
        };
    }

    private void BtnCancel_Click(object sender, EventArgs e)
    {
        btnCancel.Enabled = false;
        _cts?.Cancel();
        lblMessage.Text = "正在取消…";
    }

    public void ReportProgress(int value)
    {
        if (InvokeRequired)
        {
            this.Invoke(new Action(() => ReportProgress(value)));
            return;
        }
        progressBar.Value = value;
        lblPercent.Text = $"{value}%";
    }
}
```

## 3️⃣ 使用方式（範例）

```csharp
private async void btnStart_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, "正在匯入資料…", async (token, progress, messageProgress) =>
    {
        for (int i = 1; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50);

            progress.Report(i);
            messageProgress.Report($"正在匯入第 {i} 筆資料…");
        }
    });
}
```

## ✅ 特性

- 白底、簡潔、清爽
- 小動畫（GIF 或旋轉符號）
- 自訂文字 + 進度百分比
- 取消按鈕 + CancellationToken
- async/await 背景執行，不卡 UI
- 可用於 StatusBar、Panel 或彈出視窗
- 方便整合到任何 Sunny.UI 專案

---

# 簡潔美觀 LoadingForm（純 WinForms 版）

這個版本完全使用標準 WinForms 控制項，不依賴 Sunny.UI。可以直接用於對話框或 StatusBar，支援文字、進度條、百分比和取消按鈕。

## 1️⃣ LoadingForm Designer 配置（WinForms）

- **Form 屬性**
    - FormBorderStyle = None
    - StartPosition = CenterParent
    - ShowInTaskbar = false
    - TopMost = true
    - BackColor = 白色
    - Size 約 300x120
- **控件**
    - **PictureBox picAnimation**
        - 顯示 GIF 或簡單旋轉圖示
        - SizeMode = Zoom
        - 置中
    - **Label lblMessage**
        - 顯示訊息文字，如「正在處理…」
        - 字體稍大、黑色
        - 置中於 PictureBox 下方
    - **ProgressBar progressBar**
        - 顯示進度條
        - 寬度比文字略寬
    - **Label lblPercent**
        - 顯示百分比文字，例如「50%」
        - 置於 progressBar 右側
    - **Button btnCancel**
        - 顯示「取消」
        - Anchor = BottomRight

## 2️⃣ LoadingForm 程式碼

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public partial class LoadingForm : Form
{
    private CancellationTokenSource _cts;
    private IProgress<int> _progress;
    private IProgress<string> _messageProgress;

    public CancellationToken Token => _cts?.Token ?? CancellationToken.None;
    public IProgress<int> Progress => _progress;
    public IProgress<string> MessageProgress => _messageProgress;

    public LoadingForm(string message, CancellationTokenSource cts, IProgress<int> progress, IProgress<string> messageProgress)
    {
        InitializeComponent();

        this.FormBorderStyle = FormBorderStyle.None;
        this.StartPosition = FormStartPosition.CenterParent;
        this.ShowInTaskbar = false;
        this.TopMost = true;
        this.BackColor = System.Drawing.Color.White;

        lblMessage.Text = message;
        _cts = cts;
        _progress = progress;
        _messageProgress = messageProgress;

        btnCancel.Click += BtnCancel_Click;

        _messageProgress.ProgressChanged += (s, msg) =>
        {
            if (InvokeRequired)
            {
                this.Invoke(new Action(() => lblMessage.Text = msg));
            }
            else
            {
                lblMessage.Text = msg;
            }
        };
    }

    private void BtnCancel_Click(object sender, EventArgs e)
    {
        btnCancel.Enabled = false;
        _cts?.Cancel();
        lblMessage.Text = "正在取消…";
    }

    public void ReportProgress(int value)
    {
        if (InvokeRequired)
        {
            this.Invoke(new Action(() => ReportProgress(value)));
            return;
        }
        progressBar.Value = value;
        lblPercent.Text = $"{value}%";
    }
}
```

## 3️⃣ LoadingHelper（WinForms 版）

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public static class LoadingHelper
{
    public static async Task RunAsync(Form owner, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task> work, int timeoutMs = 0)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        LoadingForm loadingForm = null;
        progress.ProgressChanged += (s, value) => loadingForm?.ReportProgress(value);

        try
        {
            owner.Enabled = false;

            loadingForm = new LoadingForm(message, cts, progress, messageProgress);
            loadingForm.Show(owner);
            loadingForm.Refresh();

            Task workTask = work(cts.Token, progress, messageProgress);

            if (timeoutMs > 0)
            {
                var timeoutTask = Task.Delay(timeoutMs, cts.Token);
                var completed = await Task.WhenAny(workTask, timeoutTask);
                if (completed == timeoutTask)
                {
                    cts.Cancel();
                    MessageBox.Show(owner, "操作超時，已自動取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                }
                await workTask;
            }
            else
            {
                await workTask;
            }
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(owner, "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        finally
        {
            loadingForm?.Close();
            owner.Enabled = true;
        }
    }
}
```

## 4️⃣ 使用範例

```csharp
private async void btnStart_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(this, "正在匯入資料…", async (token, progress, messageProgress) =>
    {
        for (int i = 1; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50);

            progress.Report(i);
            messageProgress.Report($"正在匯入第 {i} 筆資料…");
        }
    }, timeoutMs: 30000); // 30 秒自動取消
}
```

## ✅ 特性

- 白底、簡潔、清爽
- 可使用 GIF 或旋轉圖示作動畫
- 自訂文字 + 進度百分比
- 支援取消按鈕
- 支援自動超時取消
- async/await 背景執行
- 可用於對話框或 StatusBar / Panel
- 完成或取消後自動關閉 Form

---

# StatusBar 內嵌 LoadingPanel（WinForms 版）

這個版本把原本的 LoadingForm 改成 **UserControl / Panel**，可以直接嵌入 StatusBar 或任何 Panel 上，不會彈出視窗，仍然支援文字、進度、百分比和取消。

## 1️⃣ LoadingPanel 設計

- **UserControl 或 Panel**
    - BackColor = 白色
    - Size 高度可自訂（例如 40~50px）
- **控件**
    - **PictureBox picAnimation**：顯示小動畫（GIF）
    - **Label lblMessage**：顯示訊息文字
    - **ProgressBar progressBar**：顯示進度
    - **Label lblPercent**：顯示百分比文字
    - **Button btnCancel**：取消按鈕

控件水平排列，置中，保持簡潔美觀。

## 2️⃣ LoadingPanel 程式碼

```csharp
using System;
using System.Threading;
using System.Windows.Forms;

public partial class LoadingPanel : UserControl
{
    private CancellationTokenSource _cts;
    private IProgress<int> _progress;
    private IProgress<string> _messageProgress;

    public CancellationToken Token => _cts?.Token ?? CancellationToken.None;
    public IProgress<int> Progress => _progress;
    public IProgress<string> MessageProgress => _messageProgress;

    public event EventHandler Cancelled;

    public LoadingPanel(string message, CancellationTokenSource cts, IProgress<int> progress, IProgress<string> messageProgress)
    {
        InitializeComponent();

        this.BackColor = System.Drawing.Color.White;

        lblMessage.Text = message;
        _cts = cts;
        _progress = progress;
        _messageProgress = messageProgress;

        btnCancel.Click += BtnCancel_Click;

        _messageProgress.ProgressChanged += (s, msg) =>
        {
            if (InvokeRequired)
                this.Invoke(new Action(() => lblMessage.Text = msg));
            else
                lblMessage.Text = msg;
        };
    }

    private void BtnCancel_Click(object sender, EventArgs e)
    {
        btnCancel.Enabled = false;
        _cts?.Cancel();
        lblMessage.Text = "正在取消…";
        Cancelled?.Invoke(this, EventArgs.Empty);
    }

    public void ReportProgress(int value)
    {
        if (InvokeRequired)
        {
            this.Invoke(new Action(() => ReportProgress(value)));
            return;
        }
        progressBar.Value = value;
        lblPercent.Text = $"{value}%";
    }
}
```

## 3️⃣ LoadingHelper 改成 StatusBar / Panel 使用

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

public static class LoadingHelper
{
    public static async Task RunAsync(Control hostPanel, string message, Func<CancellationToken, IProgress<int>, IProgress<string>, Task> work, int timeoutMs = 0)
    {
        var cts = new CancellationTokenSource();
        var progress = new Progress<int>();
        var messageProgress = new Progress<string>();

        var loadingPanel = new LoadingPanel(message, cts, progress, messageProgress)
        {
            Dock = DockStyle.Fill
        };

        progress.ProgressChanged += (s, value) => loadingPanel.ReportProgress(value);

        hostPanel.Controls.Add(loadingPanel);
        loadingPanel.BringToFront();
        loadingPanel.Refresh();

        try
        {
            Task workTask = work(cts.Token, progress, messageProgress);

            if (timeoutMs > 0)
            {
                var timeoutTask = Task.Delay(timeoutMs, cts.Token);
                var completed = await Task.WhenAny(workTask, timeoutTask);
                if (completed == timeoutTask)
                {
                    cts.Cancel();
                    MessageBox.Show(hostPanel.FindForm(), "操作超時，已自動取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                }
                await workTask;
            }
            else
            {
                await workTask;
            }
        }
        catch (OperationCanceledException)
        {
            MessageBox.Show(hostPanel.FindForm(), "操作已取消", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
        finally
        {
            hostPanel.Controls.Remove(loadingPanel);
            loadingPanel.Dispose();
        }
    }
}
```

## 4️⃣ 使用範例（StatusBar / Panel 上嵌入）

假設你有一個 `Panel pnlStatusBar` 放在 Form 底部：

```csharp
private async void btnStart_Click(object sender, EventArgs e)
{
    await LoadingHelper.RunAsync(pnlStatusBar, "正在匯入資料…", async (token, progress, messageProgress) =>
    {
        for (int i = 1; i <= 100; i++)
        {
            token.ThrowIfCancellationRequested();
            await Task.Delay(50);

            progress.Report(i);
            messageProgress.Report($"正在匯入第 {i} 筆資料…");
        }
    }, timeoutMs: 30000);
}
```

## ✅ 特性

- 完全內嵌在 Panel / StatusBar，不彈出新視窗
- 白底、簡潔美觀，可放動畫 GIF
- 支援動態文字、進度條和百分比
- 支援取消按鈕 + CancellationToken
- 支援自動超時取消
- async/await 執行，不阻塞 UI
- 結束後自動移除控制項

這個版本非常適合 **長耗時任務** 或 **狀態列提示**，UI 整潔、可重複使用。

---

# EF6 背景操作資料庫顯示進度條的問題

EF6 本身 **沒有內建進度回報機制**，也就是說它的 `SaveChanges()` 或 LINQ 查詢都是同步操作，執行過程中無法直接得知「進行到哪裡了」。要顯示進度條，需要自己拆分操作並手動回報進度。

## 常用做法

- **拆批處理（Batch Processing）**
    - 將大量資料分批（例如 100~500 筆）做 `SaveChanges()`
    - 每批完成後回報進度
    - 範例：

```csharp
int total = entities.Count;
int batchSize = 100;
for (int i = 0; i < total; i += batchSize)
{
    var batch = entities.Skip(i).Take(batchSize).ToList();
    context.Entities.AddRange(batch);
    context.SaveChanges();
    int progress = (i + batch.Count) * 100 / total;
    progressReporter.Report(progress); // 回報進度
}
```

- **背景執行（BackgroundWorker / Task）**
    - 使用 `Task.Run()` 或 `BackgroundWorker` 來執行資料庫操作
    - 主 UI 綁定進度回報，避免卡住界面
- **自訂進度回報**
    - EF 無法回報單筆 Insert/Update 進度
    - 只能自己在程式層面計算「批次完成比例」

## 注意

- **單次 SaveChanges() 無法得知進度**
    - 例如 10000 筆資料一次 SaveChanges()，EF 內部是同步提交 SQL，無法逐筆回報。
- **進度條只能粗略顯示批次完成比例**
    - 若希望非常精確，需要自行拆成更小批次，甚至一筆筆處理。
- **UI 不要直接呼叫 SaveChanges()**
    - 避免卡住界面，要放在 Task / BackgroundWorker 執行，透過 `IProgress<int>` 回報進度。

---

# EF6 批次 SaveChanges + 進度回報範例（可搭配 LoadingHelper/LoadingPanel）

這個範例示範如何 **將大量資料分批處理**，並使用 `IProgress<int>` 回報進度到 UI，支援 LoadingPanel 顯示百分比。

## 1️⃣ 批次儲存資料方法

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Data.Entity;

public static class EfBatchHelper
{
    /// <summary>
    /// 將資料分批插入資料庫，並回報進度
    /// </summary>
    public static async Task InsertEntitiesAsync<T>(DbContext context, List<T> entities, IProgress<int> progress, CancellationToken token, int batchSize = 100) where T : class
    {
        int total = entities.Count;
        for (int i = 0; i < total; i += batchSize)
        {
            token.ThrowIfCancellationRequested();

            var batch = entities.Skip(i).Take(batchSize).ToList();
            context.Set<T>().AddRange(batch);
            await context.SaveChangesAsync();

            int percent = (i + batch.Count) * 100 / total;
            progress?.Report(percent);
        }
    }
}
```

## 2️⃣ 在 LoadingHelper 中呼叫

假設有一個 `pnlStatusBar` 或 LoadingPanel 可用於顯示進度：

```csharp
private async void btnImport_Click(object sender, EventArgs e)
{
    using (var context = new MyDbContext())
    {
        List<MyEntity> data = GetDataToInsert(); // 取得要插入的資料

        await LoadingHelper.RunAsync(pnlStatusBar, "正在匯入資料…", async (token, progress, messageProgress) =>
        {
            await EfBatchHelper.InsertEntitiesAsync(context, data, progress, token, batchSize: 200);
        }, timeoutMs: 60000); // 60秒自動取消
    }
}
```

## 3️⃣ 特性說明

- 將大量資料 **分批提交**，避免一次 SaveChanges 卡 UI
- **IProgress** 進度回報可直接綁定 LoadingPanel
- 支援 **CancellationToken** 取消
- 支援 LoadingHelper/LoadingPanel 顯示百分比進度
- 可設定批次大小（batchSize），平衡效能與 UI 回報流暢度

這種方式適合：
- 資料量大時避免 UI 凍結
- 希望進度條顯示粗略進度
- 希望支援取消操作

---

# EF6 讀取 + 寫入雙階段進度範例（LoadingPanel + CancellationToken）

這個範例示範如何在 UI 上顯示 **讀取資料進度** 和 **寫入資料進度**，並支援取消操作。

## 1️⃣ 雙階段進度方法

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Data.Entity;

public static class EfBatchHelper
{
    public static async Task ReadAndInsertAsync<T>(DbContext context, Func<DbContext, Task<List<T>>> readFunc, IProgress<int> progress, IProgress<string> messageProgress, CancellationToken token, int batchSize = 100) where T : class
    {
        // 讀取資料階段
        messageProgress?.Report("正在讀取資料…");
        List<T> data = await readFunc(context);
        int total = data.Count;

        messageProgress?.Report("讀取完成，開始寫入…");

        // 寫入資料階段（分批）
        for (int i = 0; i < total; i += batchSize)
        {
            token.ThrowIfCancellationRequested();

            var batch = data.Skip(i).Take(batchSize).ToList();
            context.Set<T>().AddRange(batch);
            await context.SaveChangesAsync();

            int percent = (i + batch.Count) * 100 / total;
            progress?.Report(percent);
            messageProgress?.Report($"寫入第 {i + 1} ~ {i + batch.Count} 筆資料…");
        }

        messageProgress?.Report("完成！");
        progress?.Report(100);
    }
}
```

## 2️⃣ 使用 LoadingHelper / LoadingPanel 呼叫

```csharp
private async void btnImport_Click(object sender, EventArgs e)
{
    using (var context = new MyDbContext())
    {
        await LoadingHelper.RunAsync(pnlStatusBar, "初始化中…", async (token, progress, messageProgress) =>
        {
            await EfBatchHelper.ReadAndInsertAsync(context,
                async db =>
                {
                    // 模擬讀取資料，或實際讀取資料庫
                    await Task.Delay(500); 
                    return db.SourceEntities.ToList();
                },
                progress,
                messageProgress,
                token,
                batchSize: 200);
        }, timeoutMs: 60000); // 60秒自動取消
    }
}
```

## 3️⃣ 特性說明

- **雙階段進度**：讀取階段 + 寫入階段
- **進度百分比**：寫入批次完成後回報
- **文字訊息更新**：讀取/寫入狀態可動態顯示
- **取消支援**：點擊取消按鈕即可中斷操作
- **async/await**：UI 不會被卡住
- **批次大小可調**：平衡效能與進度回報流暢度
- 適合大量資料操作，需要 UI 即時反饋的情況

### 這個模式也可以延伸

- 多個資料表同時匯入，每個表單獨 LoadingPanel
- 讀取大檔案 + 寫入資料庫，分段回報進度
- 支援不同 UI，例如 StatusBar、Panel 或獨立 LoadingForm
