---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# GC.GetTotalMemory(false) 會變動是正常的嗎？

是，**完全正常**。
而且在長時間壓力測試中「一定會波動」。

但關鍵在於：

你要判斷的是「趨勢」，不是「瞬間數值」。

## GC.GetTotalMemory(false) 在量什麼？

它回傳的是：

> 目前 Managed Heap 中「仍被存活物件佔用」的位元組數

注意幾點：

1. 不包含 native 記憶體
2. 不包含 GC 尚未回收但可回收的垃圾（因為 false 不會強制 GC）
3. 會受 GC 觸發時機影響
4. 會受 LOH (Large Object Heap) 影響

## 為什麼會上下波動？

在你的測試中：

- 10 個裝置
- 持續產生訊息
- Task / async / await
- Timer + Delay
- AutoReconnect

這會產生：

1. 短生命週期物件（字串、buffer、lambda）
2. Gen0 / Gen1 回收
3. 偶發 Gen2 回收
4. LOH 分配（如果 buffer > 85KB）

典型波動模式會像：

```csharp
120 MB
125 MB
130 MB
118 MB  ← GC 發生
122 MB
128 MB
135 MB
121 MB  ← GC 發生
```

這是健康狀態。

## 什麼才是不正常？

### 不正常模式 1：單向成長

```csharp
120
125
130
135
140
145
150
...
```

即使 GC 後也回不來。

### 不正常模式 2：GC 後仍然持續墊高

```csharp
120
130
140
128  ← GC
150
160
145  ← GC
170
```

這代表：

- 有物件被意外持有引用
- Event 未解除
- 靜態集合持續增加
- Task 沒結束

## 你的測試方式是正確的

你在最後有：

```csharp
GC.Collect();
GC.WaitForPendingFinalizers();
GC.Collect();
```

然後：

```csharp
GC.GetTotalMemory(true)
```

這是正確的「最終基準值」。

這個數值才是你判斷是否洩漏的依據。

## 更專業一點的觀察方式

### 不要只看 GC.GetTotalMemory

同時看：

1. Process.PrivateMemorySize64
2. Process.WorkingSet64
3. HandleCount
4. ThreadCount

因為：

SerialPort 類型（尤其是 RJCP）會使用 native handle。

如果：

- GC 穩定
- 但 Handle 持續增加

那是 unmanaged resource leak，不是 GC leak。

## 你現在這段程式邏輯有一個小盲點

這行：

```csharp
var avgMemoryGrowthPerHour = memoryDelta / stopwatch.Elapsed.TotalHours;
```

如果測試 5 分鐘：

```csharp
TotalHours = 0.0833
```

那 1MB 差異會被放大成：

```csharp
12 MB/hr
```

短測試時這個數字會嚴重失真。

建議：

- 少於 1 小時不要用 MB/hr 判斷
- 至少跑 2~4 小時

48 小時測試才是真實長時間行為。

## 在 .NET Framework 4.7.2 下的 GC 特性

你現在是 net472：

- Workstation GC（預設）
- Server GC（若 config 開啟）

Workstation GC 本來就會有較明顯波動。

如果這是工控長時間運行程式，建議：

在 app.config 開啟：

```xml
<runtime>
  <gcServer enabled="true"/>
</runtime>
```

Server GC 對長時間服務更穩定。

## 給你一個判斷標準（實務）

如果：

- 強制 GC 後
- 記憶體回到初始 ±5%
- Handle 不持續增加
- Thread 不持續增加

那就是健康。

如果：

- 每小時淨增加 > 2~3MB
- 且持續線性成長

那才是洩漏。

## 最專業的做法

不要用 GC.GetTotalMemory 來判斷 leak。

改用：

- dotMemory
- PerfView
- Visual Studio Diagnostic Tools
- 或 Windows Performance Monitor (Process\Private Bytes)

因為：

GC.GetTotalMemory 只能看 managed heap。

SerialPort 類型（尤其像 `RJCP.SerialPortStream`）
很可能洩漏的是 native buffer。

## 結論

你的問題答案：

✔ 會變動是正常
✔ 波動是健康現象
✔ 單向成長才是異常
✔ 最後強制 GC 的數值才是判斷基準

---

# 加入 GC.CollectionCount 與 PrivateMemory 監控

你現在的監控只看：

- GC.GetTotalMemory(false)
- ThreadCount
- HandleCount

這只能看到 managed heap 表層變化。

如果要判斷：

- 是否 GC 壓力異常
- 是否有 unmanaged memory leak
- 是否 Gen2 過度成長

需要再加兩個維度：

1. GC.CollectionCount(0/1/2)
2. Process.PrivateMemorySize64

## 如何解讀數據（這才是重點）

### 正常健康模式

- Gen0 很頻繁增加
- Gen1 偶爾增加
- Gen2 很少增加
- PrivateMemory 穩定
- 強制 GC 後 Managed 回到基準

這代表：

✔ 正常分配 / 正常回收

### Managed Leak

- Gen2 持續增加
- 強制 GC 後 Managed 不下降
- PrivateMemory 同步增加

代表：

有物件被長期持有引用

常見原因：

- event 沒解除
- static list
- 未完成 Task

### Unmanaged Leak（SerialPort 常見）

- Managed Memory 穩定
- PrivateMemory 持續上升
- HandleCount 持續增加
- Gen2 沒明顯增加

代表：

native buffer / handle 沒釋放

這通常發生在：

- SerialPort
- FileStream
- SafeHandle 未 Dispose

### GC 壓力過大

- Gen2 非常頻繁增加
- CPU 升高
- 記憶體呈鋸齒狀

代表：

配置太多大物件或長生命週期物件。

## 為什麼 PrivateMemory 很關鍵？

GC.GetTotalMemory 只看 managed heap。

但：

RJCP.SerialPortStream 是使用 native buffer。

如果它洩漏，你的 GC 會顯示「很健康」，
但 PrivateMemory 會慢慢上升。

這是很多人誤判的地方。

## 專業建議（長時間工控）

如果這是長時間工業環境程式：

建議同時記錄：

- GC.CollectionCount
- PrivateMemory
- HandleCount
- ThreadCount
- 每小時最大值 / 最小值

然後匯出 CSV 分析趨勢。

不要只看 Console。

## 最重要結論

加入這兩個監控後：

你就可以精準區分：

- Managed leak
- Unmanaged leak
- 正常 GC 波動
- 壓力過大

---

# 自動判定 Leak 類型（規則式診斷模型）

目標：
讓壓測結束後，程式自動輸出：

- ✔ 正常
- ⚠ Managed Leak
- ⚠ Unmanaged Leak
- ⚠ GC 壓力過高
- ⚠ Thread / Handle 洩漏

這裡給你一個「工程實務可用」的判定模型。

## 一、先定義分析輸入資料

假設你已經有：

- Managed Memory（GC.GetTotalMemory）
- PrivateMemory
- Gen0 / Gen1 / Gen2 CollectionCount
- ThreadCount
- HandleCount

並且在「強制 GC 後」取得 final sample。

## 二、建立分析模型

新增一個分析方法：

```csharp
static void AnalyzeLeak(LongRunningStatistics statistics, TimeSpan duration)
{
    if (statistics.Samples.Count < 2)
        return;

    var first = statistics.Samples.First();
    var last = statistics.Samples.Last();

    double managedDelta = last.MemoryMB - first.MemoryMB;
    double privateDelta = last.PrivateMemoryMB - first.PrivateMemoryMB;

    int gen0Delta = last.Gen0Collections - first.Gen0Collections;
    int gen1Delta = last.Gen1Collections - first.Gen1Collections;
    int gen2Delta = last.Gen2Collections - first.Gen2Collections;

    int threadDelta = last.ThreadCount - first.ThreadCount;
    int handleDelta = last.HandleCount - first.HandleCount;

    double hours = duration.TotalHours;
    if (hours <= 0) hours = 0.01;

    double managedPerHour = managedDelta / hours;
    double privatePerHour = privateDelta / hours;

    System.Console.WriteLine();
    System.Console.WriteLine("=== 自動診斷結果 ===");

    // 1️⃣ Managed Leak
    if (managedPerHour > 3 && gen2Delta > 0)
    {
        System.Console.WriteLine("⚠ 判定：Managed Memory Leak");
        System.Console.WriteLine("   特徵：Gen2 增加 + Managed 持續成長");
        return;
    }

    // 2️⃣ Unmanaged Leak
    if (privatePerHour > 5 && managedPerHour < 1)
    {
        System.Console.WriteLine("⚠ 判定：Unmanaged Memory Leak");
        System.Console.WriteLine("   特徵：PrivateMemory 上升但 Managed 穩定");
        return;
    }

    // 3️⃣ Handle Leak
    if (handleDelta > 10)
    {
        System.Console.WriteLine("⚠ 判定：Handle 洩漏");
        System.Console.WriteLine("   特徵：HandleCount 持續增加");
        return;
    }

    // 4️⃣ Thread Leak
    if (threadDelta > 5)
    {
        System.Console.WriteLine("⚠ 判定：Thread 洩漏");
        System.Console.WriteLine("   特徵：ThreadCount 持續增加");
        return;
    }

    // 5️⃣ GC 壓力過高
    if (gen2Delta > duration.TotalMinutes / 5)
    {
        System.Console.WriteLine("⚠ 判定：GC 壓力過高");
        System.Console.WriteLine("   特徵：Gen2 回收頻繁");
        return;
    }

    // 正常
    System.Console.WriteLine("✔ 判定：系統穩定，未發現明顯洩漏");
}
```

然後在測試結束時呼叫：

```csharp
AnalyzeLeak(statistics, stopwatch.Elapsed);
```

## 三、判定邏輯說明（專業解讀）

### 1️⃣ Managed Leak 判定條件

條件：

- Managed 每小時 > 3MB
- Gen2 有增加

原因：

Gen2 代表長生命週期物件
若 Gen2 增加且 Managed 成長 → 很可能物件被持有引用

常見原因：

- event 未解除
- static list
- Task 未結束
- 快取未清除

### 2️⃣ Unmanaged Leak 判定條件

條件：

- PrivateMemory 每小時 > 5MB
- Managed 幾乎不變

這是 SerialPort 常見狀況。

代表：

- native buffer
- SafeHandle 未 Dispose
- Win32 Handle 未釋放

你的專案若使用：

RJCP SerialPortStream

這種情況尤其需要看 PrivateMemory。

### 3️⃣ Handle Leak

HandleCount 持續增加：

- File handle
- Socket
- SerialPort
- WaitHandle

通常表示：

Dispose 未正確呼叫。

### 4️⃣ Thread Leak

ThreadCount 緩慢增加：

- Task 未結束
- Timer 未 Dispose
- Background thread 未停止

### 5️⃣ GC 壓力過高

如果：

- Gen2 很頻繁
- Managed 呈鋸齒型
- CPU 升高

代表：

分配太多大物件或高 churn。

## 四、專業門檻值說明

這些門檻不是理論值，是實務經驗值：

- Managed > 3MB/hr 才算異常
- Private > 5MB/hr 才算嚴重
- Handle > 10 才值得警告
- Thread > 5 才算 leak

短時間測試不要用 MB/hr 判斷。

建議至少 2 小時以上。

## 五、如果你要更精準（進階版）

可以再加：

- LOH 監控
- GC.GetGCMemoryInfo()
- 高水位統計
- 每小時 slope regression

那就變成「準生產級診斷框架」。

## 六、總結

加入這套規則後，你的壓測可以：

自動區分：

- Managed Leak
- Unmanaged Leak
- Handle Leak
- Thread Leak
- GC 壓力
- 正常

---

# Handle 是什麼？

在 Windows 上，**Handle 是作業系統層級的資源識別碼**。
常見類型包括：

- File handle
- Serial port handle
- Socket handle
- Event / Mutex / WaitHandle
- Thread handle
- Registry handle

只要你呼叫 Win32 API 或使用封裝 native 資源的 .NET 類別，都會產生 Handle。

你的情境（SerialPort）屬於：

- Win32 `CreateFile` 開啟 COM Port
- 產生 native file handle
- 由 SafeHandle 包裝

## 為什麼 Handle 會洩漏？

Handle 洩漏的本質是：

> 有 native 資源建立，但沒有正確 Close/Dispose

在你使用 RJCP SerialPortStream 的情境下，常見原因有：

1. Open 多次但未 Dispose
2. AutoReconnect 建立新 instance
3. 事件未解除導致物件無法被 GC
4. 背景 Task 持有 stream 參考
5. 例外發生時未走到 Dispose

## 正常 vs 異常 Handle 行為

### 正常情況

- 啟動時增加
- 穩定維持
- 結束時回到接近初始值

例如：

```csharp
啟動：120
連線後：140
穩定維持：140~142
結束後：121
```

這是健康狀態。

### 洩漏情況

```csharp
120
135
150
165
180
200
...
```

每次重連都增加，永遠不下降。

這是典型：

> SerialPort instance 未正確釋放

## 如何精準判斷 Handle Leak？

### 1️⃣ 記錄每次 Open/Close 前後

```csharp
var proc = Process.GetCurrentProcess();
Console.WriteLine($"Before Open: {proc.HandleCount}");

await port.OpenAsync();

Console.WriteLine($"After Open: {proc.HandleCount}");
```

然後：

```csharp
await port.CloseAsync();
port.Dispose();
GC.Collect();
GC.WaitForPendingFinalizers();
GC.Collect();

Console.WriteLine($"After Dispose: {proc.HandleCount}");
```

如果：

- Open +1~3 → 正常
- Dispose 後沒有回來 → 有 leak

### 2️⃣ 使用 Process Explorer（專業做法）

用 Sysinternals Process Explorer：

- 查看你的 process
- 切到 Handles
- 篩選 "File"
- 找到 `\Device\Serial...`

如果你看到：

- 多個 Serial handle 存在
- 而理論上只應該有 1 個

那就是 leak。

## 正確釋放 SerialPortStream 模式

### 必須確保

```csharp
await stream.CloseAsync();
stream.Dispose();
```

不要只 Close，不 Dispose。

### 更安全寫法

```csharp
if (stream != null)
{
    try
    {
        await stream.CloseAsync();
    }
    catch { }

    stream.Dispose();
    stream = null;
}
```

## AutoReconnect 的陷阱

很多人會這樣寫：

```csharp
if (!IsConnected)
{
    stream = new SerialPortStream(...);
    await stream.OpenAsync();
}
```

但舊的 stream 沒 Dispose。

這會：

- 每次斷線增加 1 handle
- PrivateMemory 緩慢上升
- 最終達到 handle 上限（約 16K）

## 最安全的設計（工業級）

### 方案 A：單一實例重用

- 一個 SerialPortStream
- 不重建 instance
- 只 Close/Open

### 方案 B：完全重建但確保釋放

```csharp
CloseAsync
Dispose
等待 50~200ms
建立新 instance
```

不要立刻重建。

## 進階：如何確認是否真的修好了？

壓測 2 小時以上，觀察：

- HandleCount 是否呈現「平台型」而不是「線性上升」
- PrivateMemory 是否穩定
- 測試結束後是否回到初始 ±5

如果符合：

✔ 沒有 unmanaged leak
✔ SerialPort 使用正確

## 實務經驗結論

在 SerialPort 類型專案中：

- 90% 的 leak 是 Dispose 問題
- 5% 是背景 Task 未停止
- 5% 是第三方套件 bug
