---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

aliases:
date:
update:
author:
language:
sourceurl:
tags:

# Thread 與 Timer 分工更新 UI

**不要讓 Timer 來「判斷要不要更新哪一顆燈」**
這個判斷應該已經在 Thread 做完
Timer 只做一件事：**把「已標記為變更」的燈更新到 UI**

## 正確責任切分（50 顆燈的關鍵）

- Worker Thread
    - 讀 IO
    - 計算邏輯
    - 判斷狀態是否改變
    - 標記「哪些燈變了」
- UI Thread（Timer）
    - 只更新「被標記的燈」
    - 不重新判斷業務邏輯

## 關鍵概念：Dirty Flag（變更標記）

每一顆燈都有一個「是否需要更新 UI」的旗標

### 狀態模型範例（概念）

- SignalId
- CurrentState
- IsDirty

Worker Thread 的責任

- 新值 ≠ CurrentState
- 更新 CurrentState
- IsDirty = true

UI Thread 的責任

- 只處理 IsDirty == true 的項目
- 更新 Control
- IsDirty = false

## Timer 如何「過濾不要更新的燈」

### 原則

- 不比對 IO
- 不重新算邏輯
- 不掃 50 顆燈做全量判斷

### 實際流程

- Timer Tick
- 取得「已變更清單」
- 逐一更新對應的 UI 控制項
- 清除 Dirty Flag

## 兩種常見實作策略

### 策略一：Dirty List（推薦）

- Thread 維護一個「變更清單」
- 只放「狀態改變的燈」

優點

- UI 更新量最小
- Timer 很輕
- 50 顆、500 顆都撐得住

### 策略二：Dirty Flag Array

- 每顆燈一個 Flag
- Timer 掃描一次找 Dirty

優點

- 結構簡單
    缺點
    
- 每 Tick 還是要掃 50 次

## 為什麼不要讓 Timer 自己比對

- Timer 跑在 UI Thread
- 50 顆燈 × 每 Tick 比對
- 邏輯一膨脹就回到卡頓風險

## 實務建議數值

- 50 顆燈
    - Thread 判斷變更
    - UI 只更新 0~3 顆是常態
- Timer Interval
    - 100ms ~ 200ms 足夠
    - UI 感受仍是即時

## 進階但很實用的優化

- 多顆燈更新前先 SuspendLayout
- 同一種類燈用同一 Update 方法
- UI 更新前先比對 Control 當前值（防重設）

## 一句話總結

- **Thread 決定「哪些燈該動」**
- **Timer 只負責「把動過的燈畫出來」**
- Timer 不該再做判斷，否則只是換地方卡

---

# 50 顆信號燈「最小成本 Dirty 更新模型」（WinForms 工控）

以下設計目標

- IO / 判斷全部在背景 Thread
- UI Thread 不做業務判斷
- Timer 工作量與「實際變更數量」成正比
- 適合老舊工控電腦

## 一、核心資料模型（State Layer）

### SignalState

- Id：信號燈識別
- Value：目前邏輯狀態（On / Off / Fault）
- Dirty：是否需要更新 UI

說明

- Value 代表「真實設備狀態」
- Dirty 代表「UI 是否還沒反映」

## 二、背景 Thread 的責任（IO + 判斷）

### 背景流程

- 輪詢設備
- 計算新狀態
- 與目前 Value 比對
- 有變更才寫入
- 標記 Dirty

### 設計重點

- 不關心 UI
- 不 Invoke
- 不知道 Control 存在

### 偽流程

- foreach 設備信號
    - newValue = ReadAndParse()
    - if newValue != state.Value
        - state.Value = newValue
        - state.Dirty = true

## 三、變更集合（Dirty Collection）

### 建議方式：Dirty List

- 背景 Thread 將「變更的 SignalId」加入集合
- 可使用 thread-safe 結構

好處

- UI 不需要掃 50 顆
- 只處理「真的有變的」

## 四、UI Thread（Timer）的唯一工作

### Timer Tick 流程

- 取得 Dirty 清單
- 對應到 UI 控制項
- 更新外觀
- 清除 Dirty

### UI 不該做的事

- 不重新比對 Value
- 不判斷邏輯
- 不讀設備

## 五、Timer 為什麼還需要

- 節流 UI 更新
- 合併多筆變更
- 保持畫面更新節奏穩定

### 建議 Interval

- 100ms ~ 200ms
- 即使 IO 很快，也不需要 UI 同步那麼快

## 六、UI 更新最佳化細節

- 多顆燈同時更新時
    - SuspendLayout
    - ResumeLayout
- 更新前比對 Control 當前狀態
- 避免重複設定相同屬性
- 不使用 Refresh

## 七、50 顆燈的實際效益

- 平均 Tick 更新數量：0~3 顆
- UI Thread 負擔極低
- 即使瞬間 50 顆全變
    - 也只是一個 Tick 處理完

## 八、擴充性

- 50 → 200 顆燈
- 不改 UI 架構
- 只增加 State 數量

## 九、常見錯誤對照

- Timer 掃全部燈再比對
- Timer 裡有業務判斷
- 背景 Thread 直接 Invoke UI
- IO Thread 和 UI Thread 同時寫同一資料

## 十、一句話總結

- **狀態是否改變，只判斷一次**
- **UI 永遠只畫「被標記的變更」**
- **Timer 是節流器，不是決策者**

---

# thread-safe Dirty List 的具體做法（推薦實務版）

## 設計目標

- 背景 Thread 可安全加入變更
- UI Thread 可快速取出並清空
- 不重複加入同一顆燈
- 不需要鎖整個 State 陣列

## 資料結構設計

- SignalState
    - Id
    - Value
- DirtyQueue：只放「變更的 SignalId」

## 為什麼只放 Id

- 降低共享資料量
- UI 再依 Id 讀取最新狀態
- 避免複製 Value 造成不同步

## thread-safe Dirty List（實作）

```csharp
using System.Collections.Concurrent;

public class SignalStateStore
{
    private readonly SignalState[] _states;

    private readonly ConcurrentQueue<int> _dirtyQueue = new();
    private readonly ConcurrentDictionary<int, byte> _dirtySet = new();

    public SignalStateStore(int count)
    {
        _states = new SignalState[count];
        for (int i = 0; i < count; i++)
            _states[i] = new SignalState(i);
    }

    public void UpdateFromWorker(int id, SignalValue newValue)
    {
        var state = _states[id];

        if (state.Value == newValue)
            return;

        state.Value = newValue;

        if (_dirtySet.TryAdd(id, 0))
            _dirtyQueue.Enqueue(id);
    }

    public bool TryDequeueDirty(out int id)
    {
        if (_dirtyQueue.TryDequeue(out id))
        {
            _dirtySet.TryRemove(id, out _);
            return true;
        }

        return false;
    }

    public SignalValue GetValue(int id)
        => _states[id].Value;
}
```

## 這個設計的關鍵點

- ConcurrentQueue
    - 負責順序與 thread-safe
- ConcurrentDictionary 當 Set
    - 防止同一顆燈重複入列
- Worker Thread
    - 永遠只呼叫 UpdateFromWorker
- UI Thread
    - 只呼叫 TryDequeueDirty

## UI Timer 使用方式（搭配此模型）

```csharp
private void uiTimer_Tick(object sender, EventArgs e)
{
    while (_store.TryDequeueDirty(out int id))
    {
        UpdateLampUI(id, _store.GetValue(id));
    }
}
```

---

# 不用 Timer：事件 + UI 節流（進階穩定版）

## 適用情境

- UI 只在「有變更」時更新
- 不需要固定刷新率
- 希望完全移除 WinForms.Timer

## 核心概念

- Worker Thread 發出「狀態已變更」事件
- UI Thread 收到後排程更新
- 用節流機制避免 Invoke 風暴

## 節流策略（關鍵）

- 最多同時只允許一個 UI 更新排程
- 多次變更合併成一次 UI 更新

## 實作步驟

### 1. Worker Thread 發事件（不碰 UI）

```csharp
public event Action StateChanged;

private void OnStateChanged()
{
    StateChanged?.Invoke();
}
```

在 UpdateFromWorker 最後加入：

```csharp
OnStateChanged();
```

## 2. UI 端節流排程器

```csharp
private int _uiUpdateScheduled = 0;

private void OnWorkerStateChanged()
{
    if (Interlocked.Exchange(ref _uiUpdateScheduled, 1) == 1)
        return;

    BeginInvoke(new Action(ProcessDirtyStates));
}
```

## 3. UI 更新實作

```csharp
private void ProcessDirtyStates()
{
    try
    {
        while (_store.TryDequeueDirty(out int id))
        {
            UpdateLampUI(id, _store.GetValue(id));
        }
    }
    finally
    {
        Interlocked.Exchange(ref _uiUpdateScheduled, 0);
    }
}
```

## 這個版本為什麼穩

- 不用 Timer
- UI 只在有資料時動
- 多次變更自動合併
- 不會連續 BeginInvoke

## 事件 + 節流 vs Timer 對照

- Timer
    - 固定頻率
    - 可能空跑
    - 適合動畫、趨勢圖
- 事件 + 節流
    - 完全資料驅動
    - UI 負擔最小
    - 適合信號燈、狀態顯示

## 三、實務選擇建議（50 顆燈）

- 純狀態燈
    - 事件 + 節流（最佳）
- 有閃爍、動畫
    - 事件 + UI Timer 混用
- 老舊工控電腦
    - 優先移除高頻 Timer

## 一句話總結

- **Dirty 判斷只做一次**
- **UI 只處理「已確認變更」**
- **Timer 能不用就不用，事件更乾淨**

---

# 連續量（Streaming Value）

你描述的是**連續量（Streaming Value）**
它和前面講的「信號燈（離散狀態）」**本質不同**

如果用同一套 Dirty Queue
**一定會塞爆或浪費更新**

結論先講：

- 座標這種資料
    - **不應該用 Dirty Queue**
    - **也不應該保證每一筆都被 UI 看到**
- 正確做法是：
    - **UI 只看「最新值」**
    - 中間值直接被覆蓋、丟棄

## 問題本質拆解

### 你的實際情境

- 軸移動中
    - Thread 每 loop 更新座標
    - 可能 1ms 一次
- UI
    - 可能 50ms ~ 100ms 才更新一次

### 如果硬要「每筆都排隊」

- 1 秒：
    - Thread 產生 1000 筆
    - UI 顯示 10 筆
- 結果
    - Queue 永遠清不完
    - UI 顯示的是「過去」

這不是效能問題
是**模型選錯**

## 關鍵觀念：Latest Wins（只要最新）

### UI 對座標真正的需求

- 不需要每個中間點
- 只需要：
    - 目前位置
    - 視覺上連續即可

### 工控 UI 的鐵律

- **狀態類**：不能漏
- **連續值**：可以跳

座標 = 連續值

## 正確模型一：Overwrite Buffer（覆寫式）

### 核心概念

- Thread 永遠寫最新值
- UI 讀的時候，只拿「當下那一筆」

### 資料結構

- PositionState
    - X
    - Y
    - Z
    - Version 或 DirtyFlag

### Worker Thread

```csharp
public class AxisPositionState
{
    public volatile double X;
    public volatile double Y;
    public volatile double Z;
}
```

```csharp
_position.X = newX;
_position.Y = newY;
_position.Z = newZ;
```

### UI Thread（Timer 或事件）

```csharp
UpdatePositionUI(_position.X, _position.Y, _position.Z);
```

特性

- 沒有 Queue
- 沒有堆積
- 永遠顯示最新

## 正確模型二：Version Gate（推薦）

### 為什麼還要 Version

- 避免 UI 重複畫一樣的值
- 不用比較 double 誤差

### State 定義

```csharp
public class AxisPositionState
{
    public double X;
    public double Y;
    public double Z;
    public int Version;
}
```

### Worker Thread

```csharp
Interlocked.Increment(ref _state.Version);
_state.X = newX;
_state.Y = newY;
_state.Z = newZ;
```

### UI Thread

```csharp
if (_lastVersion != _state.Version)
{
    _lastVersion = _state.Version;
    UpdatePositionUI(_state.X, _state.Y, _state.Z);
}
```

效果

- UI 自動跳過中間資料
- 永遠是最新

## 正確模型三：Ring Buffer（只有在需要軌跡時）

### 什麼時候才用

- 繪製運動軌跡
- 趨勢圖
- 回放

### 設計原則

- 固定大小
- 滿了就覆寫
- UI 慢也不會爆

這和信號燈模型是**完全不同系統**

## 為什麼 Dirty Queue 不適合座標

- Dirty Queue 假設：
    - 每次變更都要處理
- 座標實際需求：
    - **最後一筆最重要**

## 工控專案實務分流建議（非常重要）

### 資料類型分級

- Level 1：狀態 / 告警
    - Dirty Queue
    - 不能漏
- Level 2：即時數值（座標、速度）
    - Latest Wins
    - 可跳值
- Level 3：歷史 / 曲線
    - Ring Buffer
    - 固定容量

## UI 更新策略建議

- 座標顯示
    - 50ms ~ 100ms 更新
- 軸實際控制
    - 不受 UI 影響
- UI 慢 ≠ 控制慢

## 一句話總結（這句很關鍵）

- **Queue 是給「不能漏的資料」用的**
- **座標是「只要最新」**
- **UI 不該背歷史包袱**

---

# 把「信號燈 + 座標 + 趨勢圖」三種資料整合成一個清楚、可維護的模型圖

下面這套是**實務上穩定、可長期維護**的分流設計
專門解你現在遇到的「信號燈 + 座標連續更新」問題

## 一、先定義三種資料「等級」

這一步非常關鍵，之後所有設計都會變得清楚

### Level 1：狀態 / 告警（不能漏）

特性

- 離散值
- 變更次數低
- 每一次變更都重要

例子

- 信號燈 On / Off
- Alarm 發生 / 清除
- Ready / Busy / Error

策略

- Dirty Queue
- 事件驅動
- UI 必須顯示每一次最終狀態

### Level 2：即時數值（只要最新）

特性

- 連續變化
- 更新頻率高
- 中間值不重要

例子

- 軸座標
- 速度
- 電流、壓力即時值

策略

- Overwrite（覆寫）
- Version Gate
- UI 固定頻率或事件節流

### Level 3：歷史 / 趨勢（有限保留）

特性

- 用來畫曲線
- 只保留最近一段

例子

- Position Trend
- Torque 曲線

策略

- Ring Buffer
- 固定容量
- 滿了就覆寫

## 二、整體架構分層（文字版架構圖）

```csharp
IO / Motion Thread
    ├── Read Device
    ├── Parse Frame
    ├── Update State Store
    │       ├── Level 1: Dirty Queue
    │       ├── Level 2: Latest State
    │       └── Level 3: Ring Buffer
    └── Raise StateChanged Event（只一次）

UI Thread
    ├── Event / Timer
    ├── Process Dirty Queue（狀態燈）
    ├── Read Latest Position（座標）
    └── Draw Trend（必要時）
```

## 三、實作範例整合

## 1️⃣ Level 1：信號燈（不能漏）

### Worker Thread

```csharp
if (newLamp != state.Lamp)
{
    state.Lamp = newLamp;
    MarkDirty(lampId);
}
```

### UI Thread

```csharp
while (TryDequeueDirty(out int id))
{
    UpdateLampUI(id, GetLampState(id));
}
```

行為

- 不漏
- 不重複
- UI 更新量極小

## 2️⃣ Level 2：座標（只要最新）

### State 定義

```csharp
public class AxisPositionState
{
    public double X;
    public double Y;
    public double Z;
    public int Version;
}
```

### Worker Thread（每 loop）

```csharp
_state.X = x;
_state.Y = y;
_state.Z = z;
Interlocked.Increment(ref _state.Version);
```

### UI Thread（Timer 或節流事件）

```csharp
if (_lastVersion != _state.Version)
{
    _lastVersion = _state.Version;
    UpdatePositionUI(_state.X, _state.Y, _state.Z);
}
```

行為

- UI 永遠顯示最新
- 中間值自動被跳過
- 不可能塞爆

## 3️⃣ Level 3：趨勢圖（Ring Buffer）

### Worker Thread

```csharp
_ringBuffer.Write(new PositionSample(x, y, z));
```

### UI Thread

```csharp
var snapshot = _ringBuffer.Snapshot();
DrawTrend(snapshot);
```

行為

- 固定記憶體
- UI 慢也不會影響 IO

## 四、UI 更新節流（整合三種資料）

### 節流核心（共用）

```csharp
private int _uiScheduled = 0;

void OnWorkerStateChanged()
{
    if (Interlocked.Exchange(ref _uiScheduled, 1) == 1)
        return;

    BeginInvoke(ProcessUIUpdate);
}
```

### UI Update

```csharp
void ProcessUIUpdate()
{
    try
    {
        ProcessLampDirty();
        UpdatePositionIfChanged();
        UpdateTrendIfVisible();
    }
    finally
    {
        Interlocked.Exchange(ref _uiScheduled, 0);
    }
}
```

效果

- 多次事件合併
- 不會 Invoke 風暴
- UI Thread 穩定

## 五、正確答案（重點）

> THREAD 每次 LOOP 更新一次，UI 還沒讀就塞一堆

**正解是**

- 座標不是用 Queue
- 是用「最後寫入勝出」
- UI 永遠只看最新

這不是妥協
而是**正確的工控 UI 設計**

## 六、快速判斷表（之後不會再搞錯）

- 要不要 Queue？
    - 會不會漏 = 不能漏 → Queue
    - 中間值重要嗎？→ 不重要 → Overwrite
- UI 慢會不會壞事？
    - 會 → 架構錯
    - 不會 → 架構對

## 一句話總結

- **控制系統跑在 Thread**
- **狀態語意跑在 Model**
- **UI 只是看結果，不是歷史**
