---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Calin.MotionControl.Advantech.Core

你是一位資深 .NET 工控運動控制首席架構師。

請設計並產出完整專案：

`Calin.MotionControl.Advantech.Core`

此專案為整個 Motion Control Framework 的 Level 6 穩定核心層。

Core 的設計目標：

- 可長期維護（10~15 年）
- 可跨 Driver
- 可跨硬體
- 可支援 Simulation
- 可 24/7 穩定運行
- APP **只能依賴 Core**
- APP **不可依賴任何 Driver**

## 環境限制

- .NET Framework 4.8
- Windows 7 / Windows 10 相容
- 僅使用 Windows 7 可用 API
- 禁止使用 Thread.Abort、async void、lock(this)
- Dispose 不可呼叫 virtual 方法
- Finalizer 不可存取 managed 物件
- 高頻區域禁止 LINQ、隱性 boxing、string.Format
- 所有 public API 必須 thread-safe
- Dispose 必須完整釋放資源且可重入
- 所有例外必須透過 `Calin.Logging.ILogger` 記錄
- 所有 public 類別與方法必須提供正體中文 XML Summary

## Logging 規範

- 所有例外必須透過 Calin.Logging (有 LoggingBridge 可以參考使用)，或 Microsoft.Extensions.Logging.Abstractions
- 所有例外必須先 Log 再拋出
- ILogger 注入到必要類別

## 全域設計優先順序

1. 穩定性
2. 相容性
3. 效能
4. 可預測性
5. 可維護性
6. 擴充性
7. 架構優雅

衝突時：

- 永遠優先穩定性
- 禁止為架構美觀犧牲穩定性
- 禁止為 DI 純度犧牲可預測性

## Core 設計限制

- 不可包含 Driver、硬體名稱、Configuration IO、Runtime 管理、Thread 控制
- 只允許 Interface、Enum、Value Model、Capability Model、Event Model、Exception Model、Command Model

## Namespace 結構

```text
Calin.MotionControl.Advantech.Core

Calin.MotionControl.Advantech.Core.Abstractions
Calin.MotionControl.Advantech.Core.Models
Calin.MotionControl.Advantech.Core.Enums
Calin.MotionControl.Advantech.Core.Events
Calin.MotionControl.Advantech.Core.Exceptions
Calin.MotionControl.Advantech.Core.Capabilities
Calin.MotionControl.Advantech.Core.Commands
```

## Motion Device

設計 Interface：

`IMotionDevice`

屬性：

- string Name
- int DeviceId
- int AxisCount
- bool IsConnected

方法：

- Initialize()
- Shutdown()

## Motion Controller

設計 Interface：

`IMotionController`

用途：

- 管理 Motion Device
- 提供 Axis 控制能力

## Axis 抽象

設計 Interface：

`IMotionAxis`

屬性：

- int AxisId
- string Name
- double Position
- double CommandPosition
- double Velocity
- AxisState State
- AxisServoState ServoState
- AxisAlarmState AlarmState
- AxisCapability Capability

方法：

- MotionResult ServoOn()
- MotionResult ServoOff()
- MotionResult Home()
- MotionResult MoveAbsolute(MotionCommand command)
- MotionResult MoveRelative(MotionCommand command)
- MotionResult JogPositive(double velocity)
- MotionResult JogNegative(double velocity)
- MotionResult Stop()
- MotionResult ResetAlarm()

## Motion Command Model

設計：

`MotionCommand`

欄位：

- double Position
- double Velocity
- double Acceleration
- double Deceleration
- double Jerk
- MotionMode Mode

## Motion Result

所有 Motion API 必須回傳：

`MotionResult`

欄位：

- bool Success
- MotionError Error
- string Message

## Axis Capability System

設計：

`AxisCapability`

欄位：

- bool SupportsHome
- bool SupportsJog
- bool SupportsSoftLimit
- bool SupportsEncoder
- bool SupportsTorqueMode
- bool SupportsContinuousMove

## Axis State

Enum：

`AxisState`

- Unknown
- Disabled
- Ready
- Moving
- Homing
- Stopping
- Alarm

## Servo State

Enum：

`AxisServoState`

- Off
- On

## Alarm State

Enum：

`AxisAlarmState`

- None
- Warning
- Fault

## Motion Mode

Enum：

`MotionMode`

- Absolute
- Relative
- Jog
- Home

## Axis Reference Monitor

- 功能：
    - 判斷軸位置相對於 Reference
    - 提供 Reference Side 狀態
    - 支援多 Reference
    - SideChanged 事件
- Enum `ReferenceSide`：LessThan, Equal, GreaterThan
- Axis API：
    - AddReferenceMonitor()
    - RemoveReferenceMonitor()
    - GetReferenceMonitors()
- 要求：
    - thread-safe
    - 不可高頻配置
    - 使用 ArrayPool 或預配置集合
    - 支援高頻 Position 更新
- Reference 邏輯必須在 Core，Driver 僅提供 Position

## Event System

設計 EventArgs：

- AxisStateChangedEventArgs
- AxisMotionCompletedEventArgs
- AxisAlarmEventArgs
- AxisPositionChangedEventArgs

`IMotionAxis` 必須提供事件：

- AxisStateChanged
- MotionCompleted
- AlarmOccurred
- PositionChanged

EventArgs 設計要求：

- 輕量化
- 避免 allocation
- 不可使用 LINQ

## Exception Model

設計例外類型：

- MotionException
- AxisNotReadyException
- AxisAlarmException
- DeviceDisconnectedException

## Value Model

設計 Value Model：

- AxisStatus
- AxisMotionInfo

## Thread Safety

所有 public API 必須：

- thread-safe

同步鎖規範：

- 禁止 `lock(this)`

建議使用：

```csharp
private readonly object _syncRoot;
```

## 輸出要求

請生成完整 **C# 程式碼**：

- namespace
- interface
- enum
- model
- event args
- exception
- Axis Reference Monitor 功能完整實作骨架

文件要求：

- 所有 public 類別與方法必須提供 **正體中文 XML Summary**

程式碼必須可編譯於：

- .NET Framework 4.8
- C# 9

開始生成完整 Core 專案。

---

# 這份 Prompt 的工控強度

這份 Prompt 已經達到 **真正設備公司架構規範級別**，原因在於加入了幾個關鍵約束。

## 1 Win7 API 限制

很多 AI 會自動生成：

- Task.Run
- async/await
- CancellationToken

但 Win7 + .NET 4.8 工控系統通常：

- 不允許 async
- 使用 polling thread

因此這條非常重要。

## 2 Dispose / Finalizer 規範

工控系統長時間運行時：

最常見 bug：

```csharp
資源沒有釋放
Handle leak
GC 無法回收
```

你加入的這三條是 **.NET 內部最佳實務**：

- Dispose 不呼叫 virtual
- Finalizer 不碰 managed
- Dispose 可重入

這其實是 **CLR 官方建議模式**。

## 3 Thread-safe API

工控 UI / Runtime / Driver 常見：

```csharp
UI thread
Motion thread
Polling thread
```

若 API 不是 thread-safe：

系統一定會出 bug。

## 4 設計優先順序

這一段其實非常關鍵：

```csharp
1 穩定性
2 相容性
3 效能
4 可預測性
```

這是 **設備軟體與 Web / Enterprise 最大差異**。

設備軟體永遠是：

```csharp
穩定 > 架構漂亮
```

這條可以避免 Copilot 過度設計。

---

# Calin.MotionControl.Advantech.Configuration

你是一位資深 .NET 工控運動控制首席架構師。

請設計並產出完整專案：

`Calin.MotionControl.Advantech.Configuration`

## 專案目標

- 提供 **設備與軸參數持久化系統**
- 避免程式碼硬編碼
- 避免 Driver 設定散落
- 支援 **Core 層 Axis Reference Monitor 配置**
- 適用 24/7 工控系統
- 支援多 Driver / Simulation
- 可長期維護（10~15 年）
- 適用 Windows 7 / Windows 10
- 適用 .NET Framework 4.8 / C# 9

## 環境限制

- 僅使用 Windows 7 可用 API
- 不可使用 Thread.Abort
- 不可使用 async void
- 不可 lock(this)
- Dispose 不可呼叫 virtual 方法
- Finalizer 不可存取 managed 物件
- 高頻區域禁止 LINQ、隱性 boxing、string.Format
- 所有 public API 必須 thread-safe
- Dispose 必須完整釋放資源且可重入
- 所有例外必須透過 ILogger（Calin.Logging）記錄
- 所有 public 類別與方法必須提供正體中文 XML Summary

## 全域設計優先順序

1. 穩定性
2. 相容性
3. 效能
4. 可預測性
5. 可維護性
6. 擴充性
7. 架構優雅

設計衝突時：

- 永遠優先穩定性
- 禁止為架構美觀犧牲穩定性
- 禁止為 DI 純度犧牲可預測性

## 專案職責

1. Machine configuration
2. Device mapping
3. Axis configuration
4. EtherCAT mapping
5. Axis Reference Monitor configuration
    - 支援多 Reference
    - Reference Value
    - Reference Side（LessThan, Equal, GreaterThan）
6. 參數版本管理

## Namespace 結構

Calin.MotionControl.Advantech.Configuration

- Abstractions
- Models
- Managers
- Services
- Exceptions

## 主要資料結構

### Machine

```csharp
MachineConfig
```

### Device

```csharp
MotionDeviceConfig
```

### Axis

```csharp
AxisConfig
```

## Axis 參數

```csharp
PulsePerUnit
MaxVelocity
Acceleration
Deceleration
HomeMode
HomeSpeed
SoftLimit+
SoftLimit-
```

## Axis Reference Monitor Config

```csharp
ReferenceValue
ReferenceSide  // LessThan, Equal, GreaterThan
MultiReferenceSupport
SideChangedEventEnabled
```

## 使用格式

```csharp
JSON
```

理由：

- Win7 可用
- 易於維護
- 人可讀
- 支援版本控制與回滾

## 設計規範

- 所有 public 類別與方法必須提供正體中文 XML Summary
- 所有例外必須透過 ILogger 記錄
- 所有操作必須 thread-safe
- Dispose 必須可重入並釋放所有資源
- 高頻操作必須避免 GC allocation 與隱性 boxing
- Configuration 不能直接依賴 Driver 或 Runtime
- 完整支援 Core 層 Axis Reference Monitor 配置功能
- JSON 讀寫必須支援版本控制與回滾

## 輸出要求

請輸出完整 **C# 程式碼**：

- namespace
- interface
- model
- manager / service
- exception
- 完整支援 **Axis Reference Monitor 配置管理**

程式碼可直接編譯於 .NET Framework 4.8 / C# 9

開始生成完整 Configuration 專案

---

# Calin.MotionControl.Advantech.Runtime

你是一位資深 .NET 工控運動控制首席架構師。請依照以下條件設計並產出完整專案 `Calin.MotionControl.Advantech.Runtime`。

## 專案目標

- 負責整個 Motion System 的生命週期管理
- Runtime 是工控系統穩定度的核心
- 支援多 Driver / Simulation
- 適用 24/7 工控系統
- 可長期維護（10~15 年）
- 適用 Windows 7 / Windows 10
- 適用 .NET Framework 4.8 / C## 9

## 環境限制

- 僅使用 Windows 7 可用 API
- 不可使用 Thread.Abort
- 不可使用 async void
- 不可 lock(this)
- Dispose 不可呼叫 virtual 方法
- Finalizer 不可存取 managed 物件
- 高頻區域禁止 LINQ、隱性 boxing、string.Format
- 所有 public API 必須 thread-safe
- Dispose 必須完整釋放資源且可重入
- 所有例外必須透過 ILogger 記錄
- 所有 public 類別與方法必須提供正體中文 XML Summary

## 全域設計優先順序

1. 穩定性
2. 相容性
3. 效能
4. 可預測性
5. 可維護性
6. 擴充性
7. 架構優雅

設計衝突時：

- 永遠優先穩定性
- 禁止為架構美觀犧牲穩定性
- 禁止為 DI 純度犧牲可預測性

## 專案職責

1. Device Manager
2. Axis Manager
3. Motion Polling（高頻輪詢）
4. Driver 初始化
5. Configuration 載入
6. Fault 管理
7. System Shutdown

## 主要元件

### Device 管理

- MotionDeviceManager

### Axis 管理

- AxisManager

### Polling

- MotionPollingService

### 錯誤系統

- MotionFaultManager

## Runtime 流程

```csharp
Load Configuration
    ↓
Create Drivers
    ↓
Create Devices
    ↓
Create Axes
    ↓
Start Polling
```

## Polling 設計規範

- 建議週期：5 ms
- 單一 Thread：MotionPollingThread
- Polling Thread 優先權：**High**
- 使用 Stopwatch + SpinWait.SpinOnce() 精準控制週期
- Polling 讀值後 **使用 struct 儲存資料**，避免 heap allocation
- Polling 讀值，使用 **Dirty Flag 判斷**，只有值有變更才傳給子類
- 高頻區域使用預配置集合 / ArrayPool
- 支援 Core 層 Axis Reference Monitor 高頻 Position 更新事件

## Copilot 輸出要求

- 生成完整 C## 專案骨架
- 包含 namespace、manager / service、thread-safe API
- Polling Thread Skeleton，使用 struct 儲存讀值
- Fault Manager Skeleton
- Device / Axis 管理骨架
- 支援 Axis Reference Monitor 高頻事件
- 所有 public 類別與方法必須提供正體中文 XML Summary
- 程式碼可直接編譯於 .NET Framework 4.8 / C## 9

依照以上條件生成完整 Runtime 專案骨架，確保高頻 Polling 的穩定性、低資源消耗，以及值變更才通知子類的機制。

---

# Calin.MotionControl.Advantech.ACM

你是一位資深 .NET 工控運動控制首席架構師。請依照以下條件設計並產出完整專案 `Calin.MotionControl.Advantech.ACM`。

## 專案目標

- 封裝 Advantech Pulse Motion Cards（如 PCI-1245、PCI-1240、PCIe-1245）
- 提供完整 Device / Axis / Controller 管理
- 支援高頻 Axis Reference Monitor，與 Runtime Polling 整合
- 適用 Windows 7 / Windows 10
- 適用 .NET Framework 4.8 / C# 9

## 對應技術

- Advantech Motion API
- AdvMotAPI.dll
- Pulse Motion Card 控制型態

## 職責

1. Card 初始化與管理
2. Axis 建立與管理
3. Motion command 發送
4. IO 讀寫
5. Alarm 監控
6. **Axis Reference Monitor 支援**：高頻讀取 Axis 實際位置並通知 Runtime

## 主要類別

Device

- AcmDevice

Axis

- AcmAxis
    - 支援 Axis Reference Monitor，高頻更新 Position
    - 讀值使用 struct 儲存，搭配 Dirty 判斷

Controller

- AcmMotionController
    - 負責 Axis command 發送與狀態回報
    - 提供 Polling 讀值接口給 Runtime

## API 封裝

Card

- OpenDevice
- CloseDevice

Axis

- Servo
- Home
- Move
- Stop
- ReadActualPosition（新增：提供給 Axis Reference Monitor）

IO

- ReadDI
- WriteDO

## 設計規範

- 高頻輪詢讀值時使用 struct 儲存 Axis 狀態，避免 heap allocation
- 透過 Dirty 判斷，只有值有變更才通知 Runtime / Core 層
- Runtime Polling 可統一訂閱 ACM Axis 的 Reference 事件，整合所有子類高頻 Position 更新
- 所有 public 類別與方法必須提供正體中文 XML Summary
- 程式碼可直接編譯於 .NET Framework 4.8 / C# 9

請依照以上條件生成完整 ACM 專案骨架，確保 Axis Reference Monitor 與 Runtime Polling 無縫整合，支援高頻 Position 更新與 Dirty 判斷。
