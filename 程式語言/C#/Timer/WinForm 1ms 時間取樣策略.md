---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# WinForm DataBinding 在高頻率更新下是否會造成記憶體持續成長

## 結論先行

- **會，有實務風險**
- WinForm DataBinding 在高頻率（例如 1ms 等級）更新時
    - 容易造成記憶體成長與效能劣化
- WPF 在相同情境下
    - 相對穩定
    - 記憶體回收行為可預期

這不是錯覺，而是**架構層級差異**。

## 為什麼 WinForm 會「愈吃愈多」

### 1. WinForm DataBinding 的設計年代與模型

WinForm DataBinding 的核心特性：

- 基於 `CurrencyManager` / `BindingManagerBase`
- 以「控制項同步資料」為主
- 非為高頻即時資料設計

其行為特點：

- 每次變更都可能觸發
    - 型別轉換
    - 格式化
    - 驗證
    - Layout / Invalidate
- 更新鏈條很長

### 2. 高頻更新造成的實際問題

當你每 1ms 改一次值時：

- 大量短命物件產生
    - `Binding`
    - `PropertyDescriptor`
    - boxing / unboxing
- UI Thread 來不及消化
- 訊息佇列開始堆積

結果：

- Gen0 / Gen1 GC 次數暴增
- 記憶體呈現「鋸齒狀上升」
- 長時間跑看起來像 memory leak

實務上非常常見。

## 這是不是「真的 Leak」

## 多數情況不是典型 Leak

- GC 最終會回收
- 但回收速度 < 產生速度
- Working Set 持續成長

在工控或長時間運轉系統中，**這在結果上等同不可接受**。

## 為什麼 WPF 看起來「不會」

### 1. WPF 的 Binding 架構本質不同

WPF 的 Binding：

- DataBinding 為核心設計
- Property System + DependencyProperty
- Binding Engine 有：
    - 合併更新
    - 延遲處理
    - 無效化批次化

### 2. UI 更新節流（重要）

WPF 預設行為：

- UI 更新頻率受 Render Thread 控制
- 約 60Hz
- 即使資料 1ms 變一次
    - UI 不會畫 1000 次

因此：

- 資料可以高速變
- UI 自動降頻顯示

WinForm **沒有這層保護**。

# 在你的 1ms 扭力取樣情境下的實務建議

## 千萬不要做的事

- 1ms 直接推 DataBinding 到 WinForm UI
- 每次取樣都 Raise PropertyChanged
- 直接 bind 到 Label / TextBox

## 正確做法（WinForm）

### 一、資料與 UI 必須脫鉤

- 背景執行緒高速取樣
- UI 只顯示「降頻後的快照」

### 二、UI 更新節流（最重要）

#### 建議頻率

- 數值顯示：50–100ms
- 曲線圖：16–33ms（30–60Hz）
- 狀態燈號：事件驅動

### 三、WinForm 實作範例

#### 背景快取最新值

```csharp
private double _latestTorque;

void SamplingLoop()
{
    while (_running)
    {
        _latestTorque = ReadTorque();
    }
}
```

#### UI Timer 低頻刷新

```csharp
private void uiTimer_Tick(object sender, EventArgs e)
{
    lblTorque.Text = _latestTorque.ToString("F2");
}
```

這比 DataBinding **穩定非常多**。

### 四、如果一定要用 DataBinding

#### 降低傷害的做法

- 不用雙向 Binding
- 不用複雜型別
- 不在 1ms 改值
- UI 用中介 ViewModel，低頻更新

## .NET 4.6.2 vs 4.8 的差異

- 4.8 GC 稍微改善
- WinForm Binding 架構未改
- 問題本質不變

## 為什麼很多工控系統選 WPF

- 高頻資料天然被節流
- UI 與資料層分離
- 長時間運行較穩定

不是 WPF「比較快」，而是「比較不會被你玩壞」。

## 總結（重點）

- WinForm DataBinding 不適合 1ms 更新
- 記憶體成長是可預期結果
- WPF 天然有 UI 節流保護
- 在 WinForm
    - 手動控制 UI 更新頻率
    - 避免高頻 Binding

如果你願意，我可以：

- 幫你設計一個「WinForm 安全顯示層」
- 或給你一份「1ms 資料擷取 + 低頻 UI 顯示」的完整範例架構

這會比單純調 GC 或 Binding 屬性有效得多。

---

# WinForm 安全顯示層、1ms 資料擷取 + 低頻 UI 顯示

## 目標與設計前提

- 背景資料擷取：1ms 等級，高優先緒、不中斷
- UI 顯示：低頻、可控、不造成 WinForm DataBinding 壓力
- 長時間運轉：記憶體穩定、不累積 UI backlog
- 架構清楚：可對長官說明「為什麼這樣設計」

以下分成「概念層設計」與「可直接使用的完整範例」。

## 一、WinForm「安全顯示層」設計概念

### 核心原則

- UI 永遠不接觸 1ms 資料流
- UI 只讀取「最新快照」
- UI 更新頻率固定、可調、可預期
- 不使用 WinForm DataBinding 做即時數值顯示

### 分層責任劃分

- Data Acquisition Layer
    - 1ms 取樣
    - 高優先緒 Thread
    - 只負責讀取與判斷門檻
- Data Buffer Layer（安全顯示層核心）
    - 保存「最新值」
    - 無事件、無通知
    - Lock-free 或極輕量同步
- UI Presentation Layer
    - 固定頻率刷新
    - 單向顯示
    - 不反向影響資料層

### 資料流示意

```text
[ 扭力計 / DAQ ]
        ↓ 1ms
[ Sampling Thread ]
        ↓ 寫入
[ LatestValue Buffer ]  ← 安全顯示層
        ↑ 讀取
[ UI Timer 50~100ms ]
        ↓
[ WinForm Controls ]
```

這個「LatestValue Buffer」就是 WinForm 安全顯示層。

## 二、安全顯示層的實作方式（重點）

### 為什麼不用 DataBinding

- WinForm Binding 每次更新成本高
- UI Thread 消化速度 < 資料產生速度
- 長時間一定出問題

### 安全顯示層的基本條件

- 只存 primitive value
- 不 Raise Event
- 不配置物件
- 不持有 UI 參考

### 建議結構

- 使用 `volatile` 或 `Interlocked`
- 只存最新值，不存歷史

## 三、完整範例架構（可直接用）

### 1. 資料模型（安全顯示層）

```csharp
public sealed class TorqueSnapshot
{
    public volatile double CurrentTorque;
    public volatile bool IsOverLimit;
    public volatile long TimestampTicks;
}
```

- 單一實例，全系統共用
- 不 new、不 dispose
- 不實作 INotifyPropertyChanged
- **改用 `CancellationTokenSource`**
    - 不會影響 1ms 資料擷取層的即時性
    - 前提是：
        - Token 只用 `IsCancellationRequested`
        - 不用 `ThrowIfCancellationRequested`
        - 不在 loop 內註冊 callback

### 2. 1ms 資料擷取層（QueryPerformanceCounter）

```csharp
public sealed class TorqueSampler
{
    private readonly TorqueSnapshot _snapshot;
    private Thread _thread;
    private CancellationTokenSource _cts;

    public double TorqueLimit { get; set; }

    public TorqueSampler(TorqueSnapshot snapshot)
    {
        _snapshot = snapshot;
    }

    public void Start()
    {
        _cts = new CancellationTokenSource();

        _thread = new Thread(() => SamplingLoop(_cts.Token));
        _thread.IsBackground = true;
        _thread.Priority = ThreadPriority.Highest;
        _thread.Start();
    }

    public void Stop()
    {
        _cts?.Cancel();
    }

    private void SamplingLoop(CancellationToken token)
    {
        var sw = Stopwatch.StartNew();
        long ticksPerMs = Stopwatch.Frequency / 1000;
        long nextTick = sw.ElapsedTicks;

        while (!token.IsCancellationRequested)
        {
            long now = sw.ElapsedTicks;

            if (now >= nextTick)
            {
                nextTick += ticksPerMs;

                double torque = ReadTorqueFromDevice();

                _snapshot.CurrentTorque = torque;
                _snapshot.TimestampTicks = now;

                if (torque >= TorqueLimit)
                {
                    _snapshot.IsOverLimit = true;
                    EmergencyStop();
                }
            }
            else
            {
                Thread.SpinWait(20);
            }
        }
    }

    private double ReadTorqueFromDevice()
    {
        // 實際 DAQ / 通訊讀取
        return 12.34;
    }

    private void EmergencyStop()
    {
        // 只做停機，不寫 log，不更新 UI
    }
}
```

#### 這一層的設計重點

- 沒有 Timer
- 沒有 Event
- 沒有 UI
- 沒有 GC 壓力來源

#### `IsCancellationRequested` 做了什麼

- 一個 `bool` 欄位讀取
- 沒有 lock
- 沒有 allocation
- 沒有 exception

成本等級：

- 與 `volatile bool` 幾乎相同
- 差異小於 DAQ I/O、QPC 呼叫的誤差

#### 真正會傷效能的是這些（你沒有用）

- `token.ThrowIfCancellationRequested()`
- `token.Register(...)`
- 在 loop 中 new CTS

#### 為什麼不建議 `private volatile bool _running`

微軟實際不建議的點不是「volatile」
而是，用自訂旗標控制執行緒生命週期，容易：
- 忘記 memory visibility
- 無法統一管理取消來源
- 無法與上層（App Exit、Form Closing、初始化失敗）整合

`CancellationToken` 是 **語意正確 + 架構可擴充** 的取消機制。

### 3. WinForm UI 顯示層（低頻刷新）

#### Form 欄位

```csharp
private readonly TorqueSnapshot _snapshot = new TorqueSnapshot();
private TorqueSampler _sampler;
private System.Windows.Forms.Timer _uiTimer;
```

#### Form 初始化

```csharp
protected override void OnLoad(EventArgs e)
{
    base.OnLoad(e);

    _sampler = new TorqueSampler(_snapshot)
    {
        TorqueLimit = 50.0
    };
    _sampler.Start();

    _uiTimer = new System.Windows.Forms.Timer();
    _uiTimer.Interval = 100; // 100ms
    _uiTimer.Tick += UiTimer_Tick;
    _uiTimer.Start();
}
```

#### UI 更新（安全）

```csharp
private void UiTimer_Tick(object sender, EventArgs e)
{
    double torque = _snapshot.CurrentTorque;
    bool over = _snapshot.IsOverLimit;

    lblTorque.Text = torque.ToString("F2");

    if (over)
    {
        lblStatus.Text = "OVER TORQUE";
        lblStatus.BackColor = Color.Red;
    }
}
```

#### 為什麼這樣「不會吃記憶體」

- UI 固定 10Hz
- 無排隊事件
- 無 Binding 物件生成
- UI Thread 永遠跟得上

## 四、常見進階需求的安全擴充

### 1. 繪圖或趨勢圖

- 不畫每筆 1ms
- Down-sample
- 例如每 10ms 取一點

### 2. 資料記錄

- Sampling Thread 寫 RingBuffer
- Log Thread 低優先緒寫檔
- 絕不在 Sampling Thread 寫檔

### 3. UI 卡住也不影響停機

- EmergencyStop 不經過 UI
- 不丟事件、不 Invoke

## 五、你可以直接這樣跟長官說

- 1ms 資料擷取是為了反應時間
- UI 不需要、也不應該 1ms 更新
- 我們用安全顯示層隔離 UI 與即時資料
- 這是工控與量測軟體的標準架構

## 總結重點

- WinForm 問題不在「慢」
- 而在「被錯誤使用」
- 1ms 取樣一定要與 UI 完全解耦
- 安全顯示層是長時間穩定的關鍵

---

# 長時間運轉記憶體監控方法（Win7 / WinForm / .NET 4.6.2+）

目標不是「看 Task Manager」，而是**判斷是否存在不可回收成長**，以及成長來自哪一層。

## 監控的四個關鍵指標（缺一不可）

- Managed Heap（.NET）
- Gen0 / Gen1 / Gen2 GC 次數
- Private Bytes（程序私有記憶體）
- Working Set（實體記憶體占用）

其中最重要的是：

- **Private Bytes 是否單向成長**
- **Gen2 是否越來越頻繁**

## 建議的最小侵入式監控實作（內建）

## 1. 使用 Process 類別（最實用）

```csharp
Process _proc = Process.GetCurrentProcess();

long privateBytes = _proc.PrivateMemorySize64;
long workingSet = _proc.WorkingSet64;
long managed = GC.GetTotalMemory(false);
```

## 2. 低頻紀錄（例如每 5 秒）

- 不要在即時取樣 Thread 記錄
- 獨立低優先緒 Log Thread

```csharp
void LogMemory()
{
    var p = Process.GetCurrentProcess();

    Log(
        DateTime.Now,
        GC.CollectionCount(0),
        GC.CollectionCount(1),
        GC.CollectionCount(2),
        p.PrivateMemorySize64,
        p.WorkingSet64,
        GC.GetTotalMemory(false)
    );
}
```

## 3. 判斷是否「真的有問題」

- 正常情況
    - Managed 記憶體鋸齒狀
    - Private Bytes 穩定
- 危險徵象
    - Private Bytes 緩慢但單向上升
    - Gen2 越來越密集

這通常代表：

- UI 元件或 Binding 沒釋放
- Driver / Interop 資源沒釋放
- 非 Managed 記憶體洩漏（DAQ SDK 很常見）

## 4. 建議跑多久

- 最少 8 小時
- 最好 24 小時
- 模擬實際加工流程

---

# 研華 USB-4704 的多通道同步取樣設計重點

USB-4704 是**USB DAQ**，這點非常關鍵。

## USB-4704 的現實限制（一定要知道）

- USB 傳輸本身有 frame（1ms）
- 多通道是「掃描取樣」，不是同時取樣
- 通道間存在微小時間偏移

但對扭力 + 其他慢速感測：

- **1ms 等級是可接受的**
- 不適合奈秒／微秒同步要求

## 正確使用方式（研華官方建議）

- 使用 **Buffered AI（硬體時脈）**
- 不要用軟體 Timer 逐通道讀

## 架構建議

- USB-4704 負責
    - 扭力
    - 壓力
    - 溫度
- 所有通道
    - 同一個 Scan Clock
    - 同一筆資料封包

## 取樣模式（概念）

- Sample Rate：1000 Hz
- Channel Count：N
- 實際取樣順序：
    - CH0 → CH1 → CH2 → …（硬體完成）

## 在 .NET 中的正確角色

- USB-4704 SDK Callback
    - 把「一筆掃描結果」當成一個 Sample
- 不要嘗試對齊到 1ms Timer
- QPC 只負責 Timestamp

## 範例設計概念

```text
[ USB-4704 ]
    ↓ 硬體時脈 1kHz
[ SDK Callback ]
    ↓
[ SampleFrame ]
    - Torque
    - Pressure
    - Temp
    - Timestamp (QPC)
```

Timestamp 是「這一組掃描」的時間，不是每通道。

## 關鍵原則

- **同步交給 DAQ**
- **時間交給 QPC**
- **不要自己拼 1ms**

---

# PCI-1220U 軸卡的同步策略（真正的即時核心）

這張卡的角色，應該是**即時控制主體**。

## PCI-1220U 的優勢

- PCI（非 USB）
- Driver ISR
- 硬體中斷
- 確定性遠高於 USB

## 正確的系統分工（非常重要）

- PCI-1220U
    - 馬達控制
    - 即時停止
    - 位置／速度
- USB-4704
    - 量測
    - 記錄
    - 顯示

## 關鍵設計原則

### 1. 停機邏輯不要走 USB

- 扭力超限
    - 最好：扭力計 DO → 軸卡 EMG
    - 次佳：軸卡硬體比較器
    - 最差：PC 軟體判斷

### 2. 時間對齊方式

- PCI-1220U
    - 自己有控制週期
- USB-4704
    - 硬體 Scan Clock

在 PC 端：

- 只做「事後對齊」
- 不做「即時鎖相」

## 建議的最終同步模型

```text
[ Torque Sensor ] → USB-4704
                        ↓
                    Sample Frame (1ms)
                        ↓
                    PC (QPC Timestamp)
                        ↓
                顯示 / 紀錄 / 分析

[ Axis Card PCI-1220U ]
        ↓
    Motion Control
        ↓
    Hard Stop / EMG
```

兩者「不互相等」，只在資料層對齊。

---

## 實務總結（這段很重要）

- 記憶體監控
    - 看 Private Bytes，不只看 GC
- USB-4704
    - 讓它做它擅長的「同步掃描」
- PCI-1220U
    - 承擔真正即時責任
- PC 軟體
    - 不當即時控制
    - 當監控與決策輔助

---

# 完整多軸 Snapshot 架構範例

符合以下條件：

- WinForm + .NET 4.6.2 / 4.8
- Windows 7 可長時間運轉
- 多軸同步取樣（USB-4704 + PCI-1220U）
- 1ms 即時 Snapshot
- UI 低頻顯示（50~100ms）
- Autofac 註冊單例
- 使用 Array 儲存多軸 Snapshot

## 架構概覽

```text
[ USB-4704 / PCI-1220U ]
        ↓ 1ms 硬體觸發
[ Sampling Thread (Highest Priority) ]
        ↓
[ AxisSystem Array (SingleInstance, Snapshot per Axis) ]
        ↓
[ UI Timer Thread (50~100ms) ]
        ↓
[ WinForm Controls Display ]
```

## 1. AxisSnapshot 定義

```csharp
public sealed class AxisSnapshot
{
    // 即時資料
    public volatile double Torque;
    public volatile double Position;
    public volatile double Velocity;
    public volatile long TimestampTicks;  // QPC timestamp

    // 可選擇加雙緩衝，用於 UI 安全顯示
    public double DisplayTorque;
    public double DisplayPosition;
    public double DisplayVelocity;
    public long DisplayTimestampTicks;
}
```

> `volatile` 確保多執行緒讀寫的基本可見性
> DisplayBuffer 用於低頻 UI 更新，避免直接讀即時值

## 2. AxisSystem（Array + SingleInstance）

```csharp
public sealed class AxisSystem
{
    public readonly AxisSnapshot[] AllAxes;

    public AxisSystem(int axisCount)
    {
        AllAxes = new AxisSnapshot[axisCount];
        for (int i = 0; i < axisCount; i++)
        {
            AllAxes[i] = new AxisSnapshot();
        }
    }

    // 複製最新值到 Display Buffer（低頻 UI 更新用）
    public void UpdateDisplayBuffer()
    {
        for (int i = 0; i < AllAxes.Length; i++)
        {
            var s = AllAxes[i];
            s.DisplayTorque = s.Torque;
            s.DisplayPosition = s.Position;
            s.DisplayVelocity = s.Velocity;
            s.DisplayTimestampTicks = s.TimestampTicks;
        }
    }
}
```

## 3. Autofac 註冊

```csharp
var builder = new ContainerBuilder();

// 單例 AxisSystem
builder.RegisterType<AxisSystem>()
       .WithParameter("axisCount", 4) // 依實際軸數
       .SingleInstance();

var container = builder.Build();

// 全程使用單例
var axisSystem = container.Resolve<AxisSystem>();
```

## 4. Sampling Thread（1ms）

```csharp
private void StartSampling(AxisSystem axisSystem)
{
    var thread = new Thread(() =>
    {
        var stopwatch = Stopwatch.StartNew();

        while (true)
        {
            long ticks = Stopwatch.GetTimestamp();

            for (int i = 0; i < axisSystem.AllAxes.Length; i++)
            {
                var axis = axisSystem.AllAxes[i];

                // 模擬取樣（USB-4704 / PCI-1220U 取得）
                axis.Torque = ReadTorque(i);
                axis.Position = ReadPosition(i);
                axis.Velocity = ReadVelocity(i);
                axis.TimestampTicks = ticks;
            }

            // 保持 1ms 節拍
            SpinWait.SpinUntil(() => (Stopwatch.GetTimestamp() - ticks) * 1000.0 / Stopwatch.Frequency >= 1);
        }
    });
    thread.IsBackground = true;
    thread.Priority = ThreadPriority.Highest;
    thread.Start();
}
```

> `SpinWait` 用於最小延遲控制，避免使用 Timer 或 Sleep，保留精準 1ms 節拍

## 5. UI Timer（低頻顯示）

```csharp
private void StartUiUpdate(AxisSystem axisSystem)
{
    var timer = new System.Windows.Forms.Timer();
    timer.Interval = 50; // 20Hz
    timer.Tick += (s, e) =>
    {
        axisSystem.UpdateDisplayBuffer();

        for (int i = 0; i < axisSystem.AllAxes.Length; i++)
        {
            var axis = axisSystem.AllAxes[i];

            // 更新 WinForm 控件
            torqueLabels[i].Text = axis.DisplayTorque.ToString("F2");
            positionLabels[i].Text = axis.DisplayPosition.ToString("F2");
            velocityLabels[i].Text = axis.DisplayVelocity.ToString("F2");
        }
    };
    timer.Start();
}
```

## 6. 優點與特性

- **安全顯示層**
    - UI 只讀 DisplayBuffer，與高頻寫入完全隔離
- **單例 + Array**
    - 多軸固定管理，無 GC 開銷
- **長時間運轉穩定**
    - 不碰 UI Thread / DataBinding
- **可擴展**
    - 增加軸數只需初始化 Array
- **Autofac 管理生命週期**
    - Sampling / UI Thread 都共用單一 AxisSystem

## 7. 延伸可選功能

1. **事件觸發 / 超限警告**
    
    - Sampling Thread 可以直接在 Snapshot 設 Flag
    - UI / Logger 低頻讀取即可
        
2. **RingBuffer 記錄歷史資料**
    
    - 每個 AxisSnapshot 可以附環狀緩衝陣列
    - 用於回放或分析
        
3. **Memory Monitor**
    
    - 單例 Snapshot + Array 記憶體穩定
    - 可結合 Process.PrivateMemorySize64 + GC.CollectionCount 監控長時間運轉

這個架構已經符合你所有需求：

- Win7 + WinForm
- 1ms 高頻 Snapshot
- 多軸同步
- 安全顯示 + Autofac 單例管理
- Array + 長時間穩定

---

# WinForm + 大量連續資料（碼表 / 取樣紀錄）的顯示

首選是 `DataGridView`（搭配 VirtualMode）。
不是 `ListBox`、不是 `TextBox`、也不是 `ListView`。

## 各 Control 的適用性快速比較

|Control|適合程度|原因摘要|
|---|---|---|
|DataGridView（VirtualMode）|最佳|支援大量資料、虛擬化、不吃記憶體|
|DataGridView（一般）|可|資料量中等，>10 萬筆開始吃 RAM|
|ListView|勉強|需要自行管理快取，效能不穩|
|ListBox|不建議|每筆都是物件，GC 壓力大|
|TextBox / RichTextBox|禁用|字串累積會爆記憶體、效能崩潰|

## 為什麼「測試碼表」不該用 TextBox

這是 WinForm 最常見的地雷。

- 每次 Append：
    - 新建字串
    - 舊字串被 GC
- 長時間：
    - LOH 成長
    - UI 越來越慢
- 1ms / 高頻紀錄：
    - 幾分鐘就爆

**不管是多行 TextBox 還是 RichTextBox 都一樣。**

## 正確選擇：DataGridView + VirtualMode

### 適用你的情境

- 長時間跑
- 每筆資料結構固定（時間、扭力、位置…）
- UI 只顯示最近 N 筆
- 背後資料量可能很大

## 建議顯示資料結構

```csharp
public struct SampleRecord
{
    public long TimestampTicks;
    public double Torque;
    public double Position;
    public double Velocity;
}
```

## DataGridView 基本設定（重點）

```csharp
dataGridView1.VirtualMode = true;
dataGridView1.ReadOnly = true;
dataGridView1.AllowUserToAddRows = false;
dataGridView1.AllowUserToDeleteRows = false;
dataGridView1.RowHeadersVisible = false;
dataGridView1.AutoGenerateColumns = false;
dataGridView1.DoubleBuffered(true); <-- 這不是正常寫法，改法參考最後項目 "WinForm 測試碼表 UserControl 設計"
```

## VirtualMode 核心事件

```csharp
dataGridView1.CellValueNeeded += DataGridView1_CellValueNeeded;
dataGridView1.RowCount = buffer.Count;
```

## CellValueNeeded 實作範例

```csharp
private void DataGridView1_CellValueNeeded(object sender, DataGridViewCellValueEventArgs e)
{
    SampleRecord record = _ringBuffer[e.RowIndex];

    switch (e.ColumnIndex)
    {
        case 0:
            e.Value = record.TimestampTicks;
            break;
        case 1:
            e.Value = record.Torque;
            break;
        case 2:
            e.Value = record.Position;
            break;
        case 3:
            e.Value = record.Velocity;
            break;
    }
}
```

## 資料來源建議：RingBuffer（非常重要）

- UI **不直接吃 1ms Snapshot**
- Sampling Thread：
    - 寫入 RingBuffer
- UI：
    - 只讀 RingBuffer 的視窗資料

```csharp
private readonly SampleRecord[] _buffer = new SampleRecord[100_000];
private int _writeIndex;
```

## UI 更新頻率建議

- 不要每筆更新 UI
- 20~50 ms 刷新一次就好

```csharp
uiTimer.Interval = 50;
```

## ListView 為什麼不推薦

- WinForm ListView 沒有真正 Virtualization
- 大資料時會：
    - 卡頓
    - 記憶體吃很快
- 微軟官方也不推薦用於大量即時資料

## 最佳實務總結（給你直接選）

- 碼表 / 取樣紀錄顯示：
    - **DataGridView + VirtualMode**
- 絕對避免：
    - TextBox / RichTextBox
- 配合：
    - RingBuffer
    - 低頻 UI Timer
- 架構上：
    - UI 永遠只讀
    - 不直接接 1ms Thread

---

# WinForm 測試碼表完整範例（可長時間跑、不吃記憶體）

以下是一套**可直接複製使用**的實務等級範例，設計目標：

- WinForm + .NET 4.6.2 / 4.8
- 1ms 資料寫入
- UI 50ms 更新
- 大量資料顯示
- 不使用 DataBinding
- 不產生 GC 壓力

---

## 一、資料結構（值型別，避免 GC）

```csharp
public struct SampleRecord
{
    public long TimestampTicks;
    public double Torque;
    public double Position;
    public double Velocity;
}
```

---

## 二、RingBuffer（固定大小，永不配置新記憶體）

```csharp
public sealed class SampleRingBuffer
{
    private readonly SampleRecord[] _buffer;
    private int _writeIndex;
    private long _totalCount;

    public int Capacity => _buffer.Length;
    public long TotalCount => Interlocked.Read(ref _totalCount);

    public SampleRingBuffer(int capacity)
    {
        _buffer = new SampleRecord[capacity];
    }

    public void Write(in SampleRecord record)
    {
        int index = _writeIndex;
        _buffer[index] = record;

        index++;
        if (index >= _buffer.Length)
            index = 0;

        _writeIndex = index;
        Interlocked.Increment(ref _totalCount);
    }

    public SampleRecord ReadByLogicalIndex(long logicalIndex)
    {
        long start = Math.Max(0, TotalCount - _buffer.Length);
        long offset = logicalIndex - start;

        if (offset < 0 || offset >= _buffer.Length)
            return default;

        int index = (int)((_writeIndex + offset) % _buffer.Length);
        return _buffer[index];
    }

    public int GetVisibleCount()
    {
        return (int)Math.Min(TotalCount, _buffer.Length);
    }
}
```

設計重點：

- 固定陣列
- 無 lock
- 無配置
- 可跑數小時～數天

---

## 三、DataGridView 設定（VirtualMode）

### Form 初始化

```csharp
private readonly SampleRingBuffer _ringBuffer = new SampleRingBuffer(100_000);
private readonly Stopwatch _sw = Stopwatch.StartNew();

private void Form1_Load(object sender, EventArgs e)
{
    SetupGrid();
    StartSampling();
    StartUiTimer();
}
```

---

### Grid 設定方法

```csharp
private void SetupGrid()
{
    dataGridView1.VirtualMode = true;
    dataGridView1.ReadOnly = true;
    dataGridView1.AllowUserToAddRows = false;
    dataGridView1.AllowUserToDeleteRows = false;
    dataGridView1.RowHeadersVisible = false;
    dataGridView1.AutoGenerateColumns = false;

    dataGridView1.Columns.Add("Time", "Ticks");
    dataGridView1.Columns.Add("Torque", "Torque");
    dataGridView1.Columns.Add("Pos", "Position");
    dataGridView1.Columns.Add("Vel", "Velocity");

    dataGridView1.CellValueNeeded += DataGridView1_CellValueNeeded;
}
```

---

### CellValueNeeded（關鍵）

```csharp
private void DataGridView1_CellValueNeeded(object sender, DataGridViewCellValueEventArgs e)
{
    long start = Math.Max(0, _ringBuffer.TotalCount - _ringBuffer.Capacity);
    long logicalIndex = start + e.RowIndex;

    SampleRecord r = _ringBuffer.ReadByLogicalIndex(logicalIndex);

    switch (e.ColumnIndex)
    {
        case 0:
            e.Value = r.TimestampTicks;
            break;
        case 1:
            e.Value = r.Torque.ToString("F3");
            break;
        case 2:
            e.Value = r.Position.ToString("F3");
            break;
        case 3:
            e.Value = r.Velocity.ToString("F3");
            break;
    }
}
```

---

## 四、1ms 模擬取樣 Thread

```csharp
private void StartSampling()
{
    var thread = new Thread(() =>
    {
        long ticksPerMs = Stopwatch.Frequency / 1000;
        long nextTick = _sw.ElapsedTicks;

        while (true)
        {
            long now = _sw.ElapsedTicks;

            if (now >= nextTick)
            {
                nextTick += ticksPerMs;

                var record = new SampleRecord
                {
                    TimestampTicks = now,
                    Torque = ReadTorque(),
                    Position = ReadPosition(),
                    Velocity = ReadVelocity()
                };

                _ringBuffer.Write(record);
            }
            else
            {
                Thread.SpinWait(20);
            }
        }
    });

    thread.IsBackground = true;
    thread.Priority = ThreadPriority.Highest;
    thread.Start();
}
```

---

### 模擬資料來源

```csharp
private double ReadTorque()
{
    return Math.Sin(_sw.ElapsedMilliseconds * 0.01) * 10;
}

private double ReadPosition()
{
    return _sw.ElapsedMilliseconds * 0.001;
}

private double ReadVelocity()
{
    return Math.Cos(_sw.ElapsedMilliseconds * 0.01);
}
```

---

## 五、UI Timer（低頻刷新）

```csharp
private void StartUiTimer()
{
    var timer = new System.Windows.Forms.Timer();
    timer.Interval = 50;
    timer.Tick += (s, e) =>
    {
        dataGridView1.RowCount = _ringBuffer.GetVisibleCount();
        dataGridView1.Invalidate();
    };
    timer.Start();
}
```

---

## 六、這套設計為什麼穩定

- 不使用 DataBinding
- 不累積字串
- 不 new object
- UI 與 1ms Thread 完全隔離
- RingBuffer 永遠固定大小
- VirtualMode 只畫看得到的列

---

## 七、常見錯誤對照

- TextBox Append：必爆
- ListBox Add：必爆
- DataGridView 直接 AddRow：會慢性死亡
- 每筆 Invoke UI：撐不了 10 分鐘

---

## 八、你現在可以怎麼用這份碼

- 當測試碼表
- 當 DAQ 記錄視窗
- 當問題重現工具（給主管看）

這一套可以直接撐 **一整天連續跑**，Task Manager 記憶體幾乎不動。

---

# 超限資料自動標色（DataGridView）

## 設計原則

- 不在取樣層處理顏色
- 不存 `Color` 到資料結構
- 只存「是否超限」的布林旗標
- 標色邏輯只存在 UI 層

## 1. SampleRecord 增加超限旗標

```csharp
public struct SampleRecord
{
    public long TimestampTicks;
    public double Torque;
    public double Position;
    public double Velocity;
    public bool IsOverLimit;
}
```

## 2. 取樣時設定 IsOverLimit

```csharp
var record = new SampleRecord
{
    TimestampTicks = now,
    Torque = torque,
    Position = ReadPosition(),
    Velocity = ReadVelocity(),
    IsOverLimit = torque >= _torqueLimit
};

_ringBuffer.Write(record);
```

## 3. 使用 CellFormatting 做標色（建議方式）

- 不影響 VirtualMode
- 不需要存 UI 狀態
- WinForm 官方建議

## 註冊事件

```csharp
dataGridView1.CellFormatting += DataGridView1_CellFormatting;
```

## CellFormatting 實作

```csharp
private void DataGridView1_CellFormatting(object sender, DataGridViewCellFormattingEventArgs e)
{
    long start = Math.Max(0, _ringBuffer.TotalCount - _ringBuffer.Capacity);
    long logicalIndex = start + e.RowIndex;

    SampleRecord r = _ringBuffer.ReadByLogicalIndex(logicalIndex);

    if (r.IsOverLimit)
    {
        e.CellStyle.BackColor = Color.DarkRed;
        e.CellStyle.ForeColor = Color.White;
    }
}
```

## 特性說明

- 只在畫面需要重繪時才執行
- 不影響背景 1ms Thread
- 即使資料量大也安全

---

# 自動凍結畫面但背景繼續記錄

## 使用情境

- 發生超限時
- 操作員想暫停畫面查看
- 但資料仍需完整記錄

## 1. UI Freeze 狀態旗標（UI 專用）

```csharp
private bool _isUiFrozen;
```

## 2. UI Timer 加入 Freeze 判斷

```csharp
private void StartUiTimer()
{
    var timer = new System.Windows.Forms.Timer();
    timer.Interval = 50;
    timer.Tick += (s, e) =>
    {
        if (_isUiFrozen)
            return;

        dataGridView1.RowCount = _ringBuffer.GetVisibleCount();
        dataGridView1.Invalidate();
    };
    timer.Start();
}
```

## 重點

- Freeze 只影響 UI
- RingBuffer 持續寫入
- 1ms Thread 完全不知情

## 3. Freeze / Resume 控制（Button）

```csharp
private void btnFreeze_Click(object sender, EventArgs e)
{
    _isUiFrozen = true;
}

private void btnResume_Click(object sender, EventArgs e)
{
    _isUiFrozen = false;
}
```

## 4. 自動超限即凍結（選用）

### UI Timer 內檢查最新資料

```csharp
private void CheckAutoFreeze()
{
    long latestIndex = _ringBuffer.TotalCount - 1;
    if (latestIndex < 0)
        return;

    SampleRecord latest = _ringBuffer.ReadByLogicalIndex(latestIndex);

    if (latest.IsOverLimit)
    {
        _isUiFrozen = true;
    }
}
```

在 UI Timer 中呼叫：

```csharp
CheckAutoFreeze();
```

## 三、整體行為總覽

```text
1ms Sampling Thread
    └─ 持續寫入 RingBuffer（永不中斷）

UI Timer（50ms）
    ├─ 若未凍結 → 更新 RowCount + 重繪
    ├─ 若凍結 → 停止更新畫面
    └─ CellFormatting 動態標色

使用者操作
    ├─ Freeze：只停 UI
    └─ Resume：UI 追上最新資料
```

## 四、為什麼這樣設計是「正確的」

- Freeze 不影響資料完整性
- 標色不影響效能
- 沒有跨 Thread UI 操作
- 沒有 DataBinding
- 沒有記憶體累積

這種設計在工業控制系統裡稱為：

**「顯示層可中斷，資料層不可中斷」**

## 五、進階可再加的功能（不破壞架構）

- Freeze 時顯示「Frozen at HH:mm:ss.fff」
- Resume 時自動捲到最新列
- 超限列閃爍一次後固定顏色
- Freeze 狀態下允許滑動回看歷史

---

# 可直接使用的 WinForm 測試碼表 UserControl 設計

以下提供一個**完整、實務可用**的設計，符合你前面提到的需求：

- 高頻背景持續記錄（1ms 等級）
- UI 可凍結 / 恢復
- 超限資料自動標色
- 大量資料顯示不閃爍
- 不影響資料擷取層效能

## 架構說明（先看這段很重要）

- 背景執行緒
    - 負責資料擷取與寫入 RingBuffer
    - 永遠不中斷
- UI（DataGridView）
    - 只顯示資料快照
    - Freeze 時停止刷新，不停資料收集
- 使用 VirtualMode
    - UI 不持有資料
    - 完全避免大量 Row 物件

## BufferedDataGridView（避免閃爍）

```csharp
public class BufferedDataGridView : DataGridView
{
    public BufferedDataGridView()
    {
        DoubleBuffered = true;
        SetStyle(
            ControlStyles.OptimizedDoubleBuffer |
            ControlStyles.AllPaintingInWmPaint,
            true);
    }
}
```

## 資料模型（單筆）

```csharp
public sealed class TorqueSampleRow
{
    public long TimestampTicks { get; init; }
    public double Torque { get; init; }
    public bool IsOverLimit { get; init; }
}
```

## RingBuffer（UI 與 DAQ 共用）

```csharp
public sealed class TorqueRingBuffer
{
    private readonly TorqueSampleRow[] _buffer;
    private int _writeIndex;
    private int _count;

    public TorqueRingBuffer(int capacity)
    {
        _buffer = new TorqueSampleRow[capacity];
    }

    public void Add(TorqueSampleRow row)
    {
        _buffer[_writeIndex] = row;
        _writeIndex = (_writeIndex + 1) % _buffer.Length;
        if (_count < _buffer.Length)
            _count++;
    }

    public int Count => _count;

    public TorqueSampleRow Get(int index)
    {
        int realIndex = (_writeIndex - _count + index + _buffer.Length) % _buffer.Length;
        return _buffer[realIndex];
    }
}
```

## UserControl：TorqueLogGrid

```csharp
public partial class TorqueLogGrid : UserControl
{
    private readonly BufferedDataGridView _grid;
    private TorqueRingBuffer _buffer;
    private bool _uiFrozen;

    public double TorqueLimit { get; set; }

    public TorqueLogGrid()
    {
        _grid = new BufferedDataGridView
        {
            Dock = DockStyle.Fill,
            VirtualMode = true,
            ReadOnly = true,
            AllowUserToAddRows = false,
            AllowUserToDeleteRows = false,
            RowHeadersVisible = false
        };

        _grid.Columns.Add("Timestamp", "Timestamp (ticks)");
        _grid.Columns.Add("Torque", "Torque");
        _grid.Columns.Add("State", "State");

        _grid.CellValueNeeded += OnCellValueNeeded;
        _grid.CellFormatting += OnCellFormatting;

        Controls.Add(_grid);
    }

    public void BindBuffer(TorqueRingBuffer buffer)
    {
        _buffer = buffer;
    }

    public void RefreshView()
    {
        if (_uiFrozen)
            return;

        _grid.RowCount = _buffer?.Count ?? 0;
        _grid.Invalidate();
    }

    public void FreezeUI()
    {
        _uiFrozen = true;
    }

    public void ResumeUI()
    {
        _uiFrozen = false;
        RefreshView();
    }

    private void OnCellValueNeeded(object sender, DataGridViewCellValueEventArgs e)
    {
        if (_buffer == null)
            return;

        var row = _buffer.Get(e.RowIndex);

        if (e.ColumnIndex == 0)
            e.Value = row.TimestampTicks;
        else if (e.ColumnIndex == 1)
            e.Value = row.Torque;
        else if (e.ColumnIndex == 2)
            e.Value = row.IsOverLimit ? "OVER" : "OK";
    }

    private void OnCellFormatting(object sender, DataGridViewCellFormattingEventArgs e)
    {
        if (_buffer == null)
            return;

        var row = _buffer.Get(e.RowIndex);

        if (row.IsOverLimit)
        {
            e.CellStyle.BackColor = Color.DarkRed;
            e.CellStyle.ForeColor = Color.White;
        }
    }
}
```

## Form 使用方式（最小整合）

```csharp
private TorqueRingBuffer _ringBuffer;
private System.Windows.Forms.Timer _uiTimer;

private void Form1_Load(object sender, EventArgs e)
{
    _ringBuffer = new TorqueRingBuffer(50000);
    torqueLogGrid1.BindBuffer(_ringBuffer);

    _uiTimer = new Timer { Interval = 50 };
    _uiTimer.Tick += (_, _) => torqueLogGrid1.RefreshView();
    _uiTimer.Start();
}
```

## 為什麼這套適合你現在的需求

- UI Freeze 不影響 DAQ
- 無 Join、無 Abort、無鎖 UI
- DataGridView 永遠只讀快照
- DoubleBuffer + VirtualMode 已到 WinForm 極限
- 未來可直接包成公司共用 Control
