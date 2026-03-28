# C## 中 struct 與 class 全面比較

以下以 .NET CLR 實作模型說明，包含記憶體配置、GC 行為、語意差異與實務建議。

## 一、本質差異

class

- 參考型別（Reference Type）
- 預設可為 null
- 具繼承（單一繼承）
- 比較時預設比對參考（Reference Equality）

struct

- 實值型別（Value Type）
- 不可為 null（除非 Nullable）
- 不可繼承 class，只能實作 interface
- 比較時預設為值比較（欄位逐一比較）

## 二、記憶體分配方式

### class

- 實體配置在 Heap
- 變數儲存在 Stack 上的是「參考指標」
- 由 GC 管理生命週期

範例

```csharp
class Foo { public int X; }

Foo a = new Foo();
Foo b = a;
b.X = 10;
```

a 與 b 指向同一物件。

特性

- 需要 GC 回收
- 有配置成本
- 共享同一實例

### struct

- 預設配置在 Stack（但不絕對）
- 若為 class 成員，則存在於該 class 的 Heap 區塊內
- 不由 GC 單獨追蹤

範例

```csharp
struct Foo { public int X; }

Foo a = new Foo();
Foo b = a;
b.X = 10;
```

a 與 b 是不同實體（複製語意）

特性

- 無額外 GC 負擔
- 複製語意
- 大 struct 會造成複製成本

## 三、初始化差異與常見異常

### struct 必須完全初始化

```csharp
struct Foo
{
    public int X;
    public int Y;
}
```

錯誤：

```csharp
Foo f;
Console.WriteLine(f.X); // 編譯錯誤：未初始化
```

必須：

```csharp
Foo f = new Foo();
```

或

```csharp
Foo f;
f.X = 1;
f.Y = 2;
```

### struct 不允許無參數建構子（C## 10 以前）

C## 10+ 可定義，但仍有規則限制。

### class 初始化

```csharp
Foo f;                  // OK（null）
Console.WriteLine(f.X); // NullReferenceException
```

class 未 new 是 null，不是未初始化。

## 四、記憶體結構對照

### class 佈局

Stack

- 參考指標

Heap

- Method Table Pointer
- Sync Block
- 欄位資料

### struct 佈局

Stack

- 直接欄位資料

若 struct 是 class 成員：

Heap

- class header
- struct 內容（內嵌）

## 五、效能比較

### 小型 struct（< 16 bytes）

優勢

- 無 Heap 分配
- 無 GC 壓力
- Cache locality 好

適合

- 數值型資料
- 座標
- 狀態封包
- 高頻資料（例如工控 IO snapshot）

### 大型 struct

缺點

- 每次傳遞都會複製
- 當作方法參數時產生拷貝成本

解法

```csharp
void Process(in BigStruct data)
```

使用 in 參考傳遞避免複製。

### class 效能特性

優勢

- 傳遞成本固定（8 bytes pointer）
- 適合大型物件

缺點

- Heap allocation
- GC 壓力
- 記憶體碎片

## 六、Equals / Boxing 行為

### struct 會發生 boxing

```csharp
object obj = myStruct; // boxing
```

會產生：

- Heap 分配
- GC 負擔

避免：

- 使用泛型
- 實作 IEquatable

## 七、執行語意差異（極重要）

### class

- Reference semantics
- 多處共享同一實例
- 修改會影響所有持有者

### struct

- Value semantics
- Assignment = copy
- 修改副本不影響原始

工控常見錯誤：

```csharp
var state = axisStates[0];
state.Position = 100;
```

如果 axisStates 是 struct array，這只改副本。

## 八、使用場景建議

### 使用 struct 當

- 不可變資料（immutable）
- 小型資料結構
- 數值型資料
- DTO snapshot
- 高頻運算

例如：

```csharp
struct AxisState
{
    public double Position;
    public double Velocity;
    public bool IsMoving;
}
```

### 使用 class 當

- 需要繼承
- 需要共享狀態
- 物件生命週期長
- 複雜物件圖
- 需要 null 表示不存在

例如：

```csharp
class AxisController
{
    public AxisState CurrentState { get; set; }
}
```

## 九、工控專案實務建議

### 狀態資料 → struct

- 減少 GC
- Snapshot 比較差異快
- 適合 polling thread

### 控制器 / Service → class

- 共享實例
- 管理生命週期
- DI 容器管理

## 十、常見陷阱

1. 大 struct (> 32 bytes) 效能反而變差
2. struct 放入 List 會 boxing
3. 修改 struct property 可能改的是副本
4. struct 不能繼承 class
5. 不可變 struct 才是最佳實踐

## 十一、簡化決策表

資料小 + 不可變 + 高頻 → struct
需要共享 + 複雜行為 + DI → class
超過 32 bytes → 優先 class
需要繼承 → 只能 class

## 十二、效能結論

小型 struct 在高頻場景下通常優於 class
大型 struct 可能比 class 慢
真正瓶頸通常不是 struct vs class，而是：

- GC 壓力
- Lock contention
- 記憶體配置頻率

---

# 工控軸狀態資料流設計：struct vs class 架構實戰比較

情境假設

- 4 張軸卡
- 每張 8 軸
- 10ms polling
- 每秒 100 次更新
- UI 顯示座標 / 速度 / Alarm / Servo / InPosition

目標

- 低 GC 壓力
- 高頻更新穩定
- UI 不被淹沒

## 一、struct 版本架構（推薦用於狀態資料）

### 1️⃣ AxisState 設計（不可變）

```csharp
public readonly struct AxisState
{
    public readonly double Position;
    public readonly double Velocity;
    public readonly bool ServoOn;
    public readonly bool Alarm;
    public readonly bool InPosition;

    public AxisState(
        double position,
        double velocity,
        bool servoOn,
        bool alarm,
        bool inPosition)
    {
        Position = position;
        Velocity = velocity;
        ServoOn = servoOn;
        Alarm = alarm;
        InPosition = inPosition;
    }
}
```

特點

- readonly 避免誤改
- 約 8+8+1+1+1 ≈ 24 bytes（安全範圍內）
- 無 heap allocation

### 2️⃣ Polling Thread（Shadow Buffer 比較）

```csharp
private AxisState[] _shadowStates;
private AxisState[] _currentStates;

void PollLoop()
{
    while (true)
    {
        for (int i = 0; i < _axisCount; i++)
        {
            var newState = ReadAxis(i);

            if (!newState.Equals(_shadowStates[i]))
            {
                _shadowStates[i] = newState;
                OnAxisChanged(i, newState);
            }
        }

        Thread.Sleep(10);
    }
}
```

優勢

- 比較成本低（值比較）
- 無 GC
- 無 boxing
- Cache locality 佳

### 3️⃣ UI 更新（只推差異）

```csharp
void OnAxisChanged(int axisNo, AxisState state)
{
    _dispatcher.Post(() =>
    {
        _viewModel.UpdateAxis(axisNo, state);
    });
}
```

### GC 壓力分析（struct 版本）

每次 polling：

- 無 new
- 無 GC
- 無 boxing
- 僅 stack copy

即使 32 軸 × 100Hz
每秒資料搬移約：

32 × 100 × 24 bytes ≈ 76,800 bytes/s

極小。

## 二、class 版本架構

### 1️⃣ AxisState 使用 class

```csharp
public class AxisState
{
    public double Position;
    public double Velocity;
    public bool ServoOn;
    public bool Alarm;
    public bool InPosition;
}
```

### 2️⃣ Polling

```csharp
var newState = new AxisState();
```

每次 new：

32 軸 × 100Hz = 3200 objects / sec

1 分鐘 = 192,000 objects

這會產生：

- Gen0 壓力
- GC 暫停
- UI jitter

## 三、效能比較表

struct 小型 immutable

- 無 GC
- 無 allocation
- 高頻最佳
- 適合 snapshot

class

- 有 GC
- allocation 成本
- 傳遞固定成本（pointer）
- 適合大型物件

## 四、進階最佳模型（工控專用）

### 架構分層

Core Layer

- struct AxisState
- Polling thread
- Shadow compare

Service Layer

- class AxisService
- 事件派送

UI Layer

- class AxisViewModel
- Binding

這種混合模式最佳。

## 五、常見錯誤示範（struct 陷阱）

### ❌ 修改副本

```csharp
var state = _shadowStates[i];
state.Position = 100;  // 若非 readonly struct 會修改副本
```

必須整個覆蓋：

```csharp
_shadowStates[i] = new AxisState(...);
```

### ❌ 放進 object 容器

```csharp
List<object> list;
list.Add(axisState);  // boxing
```

避免。

## 六、極致優化版本（Span + in 參數）

```csharp
bool HasChanged(in AxisState a, in AxisState b)
```

避免 struct 複製。

## 七、結論

高頻狀態資料 → 小型 readonly struct
控制器 / Service → class
大於 32 bytes → 優先 class
需要共享可變狀態 → class

## 八、給你目前專案的建議

根據你之前的設計風格：

AxisState → struct
AxisConfig → class
AxisController → class
ShadowBuffer → struct array
UI ViewModel → class

這樣：

- GC 幾乎為 0
- Polling 穩定
- UI 不抖動
- 架構清晰

---

# struct 對 CPU Cache 與 GC 行為的實際影響（工控高頻場景解析）

以下用「32 軸 × 100Hz polling」為例，做實務層分析。

前提假設

- AxisState ≈ 24 bytes
- 每秒更新 3200 次
- 連續執行 8 小時

## 一、CPU Cache 行為差異

### 1️⃣ struct（連續陣列）

```csharp
AxisState[] states = new AxisState[32];
```

記憶體佈局：

- 連續排列
- 24 bytes × 32 = 768 bytes
- 完整落在 L1/L2 Cache 內

優勢：

- 順序讀取 → Prefetch 有效
- 無指標跳躍
- Cache Miss 率低
- CPU pipeline 穩定

這種模式非常適合 polling。

### 2️⃣ class（陣列存指標）

```csharp
AxisState[] states = new AxisState[32];
```

實際佈局：

- 陣列存 32 個 reference
- 每個物件分散在 Heap
- 記憶體非連續

缺點：

- Pointer chasing
- Cache line 無法預測
- Miss 機率高
- TLB 壓力上升

在高頻讀寫下差異會逐漸放大。

## 二、GC 行為分析

### struct 版本

每次更新：

- 無 new
- 無 allocation
- 無 GC tracking

結果：

- GC 幾乎只處理 UI / 其他物件
- Polling thread 不干擾 GC

長時間運作穩定。

### class 版本（每次 new）

每秒 3200 個物件
1 分鐘 ≈ 192,000
10 分鐘 ≈ 1,920,000

GC 會發生：

- Gen0 高頻觸發
- Gen1 提升
- 偶發 Gen2

影響：

- STW（Stop-the-world）
- UI 卡頓
- Poll jitter

工控場景最忌諱這種抖動。

## 三、大 struct 何時會變慢？

假設：

```csharp
struct BigAxisState
{
    double Position;
    double Velocity;
    double Acc;
    double Jerk;
    double Torque;
    double CmdPos;
    double CmdVel;
    double CmdAcc;
}
```

8 個 double = 64 bytes

若：

- 以值傳遞
- 每次方法呼叫都複製

成本會上升。

解法：

```csharp
void Process(in BigAxisState state)
```

使用 in 傳參考避免複製。

## 四、Benchmark 思維模型

簡化模型：

struct array loop

- 線性記憶體存取
- 幾乎無 GC

class array loop

- 間接存取
- GC 週期性干擾

當更新頻率 > 50Hz 時
struct 優勢開始明顯。

## 五、極端情況測試建議

你可以用 BenchmarkDotNet 測：

1. 32 軸 100Hz
2. 64 軸 200Hz
3. 128 軸 500Hz

觀察：

- Allocation rate
- GC Count
- Mean latency
- StdDev

你會看到：

struct + shadow compare
幾乎沒有 allocation。

## 六、真正影響效能的排序

在工控專案中：

1. GC allocation frequency
2. Lock contention
3. Thread context switch
4. Cache miss
5. struct vs class 差異

重點永遠是 allocation。

## 七、最佳實務模型（穩定版）

AxisState → readonly struct
ShadowStates → AxisState[]
Polling → 單一 thread
UI → 只收變動事件
跨 thread → ConcurrentQueue 或 Channel

不要：

- 每次 new class
- 每次 boxing
- 在 polling 裡 Raise Event 直接觸 UI

## 八、何時一定要用 class？

- 物件 > 64 bytes
- 需要多處共享可變狀態
- 需要繼承
- 生命週期長

## 九、總結（給你專案等級建議）

你的工控架構屬於：

高頻資料流 + 長時間穩定運行

最佳組合：

狀態資料 → 小型 readonly struct
控制器 → class
配置資料 → class
事件參數 → struct 或 readonly struct
高頻方法參數 → in

這樣可以做到：

- 幾乎 0 GC 干擾
- CPU cache 最佳化
- 長時間 24/7 穩定
