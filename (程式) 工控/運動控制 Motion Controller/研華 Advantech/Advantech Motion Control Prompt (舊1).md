---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Prompt 1：Calin.MotionControl.Advantech Level 5 核心父類庫（純 Core Library 版）

專案名稱：Calin.MotionControl.Advantech.Core

你現在是一位 **資深 .NET 工控運動控制首席架構師**，請重新設計並產生一個 **可直接作為 24/7 工控量產基礎父類庫的 Level 5 核心運動控制架構**。

**前提：原有 `Calin.MotionControl.Advantech.csproj` 已刪除，必須重新建立為真正的核心基礎庫（Core Library）。**

此專案的定位不是 PCI 驅動庫、不是 EtherCAT 驅動庫、不是 Standalone 控制器驅動庫。
此專案是所有後續硬體子類庫的共同父層。

後續硬體子類庫將包含但不限於：

- `Calin.MotionControl.Advantech.PCI`
- `Calin.MotionControl.Advantech.PCBased`
- `Calin.MotionControl.Advantech.EtherCAT`
- `Calin.MotionControl.Advantech.Standalone`

本專案必須作為這些子類庫的穩定共同基礎。

Calin.MotionControl.Advantech.Core 是父類核心庫，應用程式不可直接依賴此專案作為實際硬體控制入口；應改依賴後續硬體子類庫或其組裝模組。

## 一、專案定位

本專案為：

- 長期穩定核心層（Long-term Stable Core Layer）
- 硬體無關（Hardware-agnostic）
- 平台抽象層（Abstraction-first）
- 不包含任何特定硬體 SDK 實作
- 不包含任何 Native Driver 綁定
- 不包含任何 PCI / EtherCAT / Standalone 專屬邏輯

本專案提供：

- 統一抽象介面
- 共用資料模型
- 共用狀態機
- 共用命令模型
- 共用診斷模型
- 共用安全策略抽象
- 共用組態與參數抽象
- 共用工廠 / 模組 / 擴充點
- Fake / Simulator
- 與 App 解耦的 logging / diagnostics / lifecycle 基礎能力

## 二、必須同時滿足

- 工控成熟度模型 Level 5
- 長期穩定核心層
- 多設備 / 多控制器 / 多卡 / 多軸抽象
- 支援後續 PCI / PC-Based / EtherCAT / Standalone 子類庫擴充
- 外部組態 / 參數持久化支援
- 完整可編譯
- 所有 public API thread-safe
- 所有 public 類別與方法提供 **正體中文 XML Summary**

生成程式碼必須：

- 完整可編譯
- 不得包含 TODO
- 不得包含 Stub
- 不得包含未完成方法
- 不得產生示範式架構
- 不得簡化核心分層

## 三、絕對禁止

### 1. 禁止硬體綁定

本 Core Library **不可直接依賴或參考任何硬體 SDK**，包括但不限於：

- `AdvMotAPI.dll`
- MotionNavi API
- EtherCAT Master SDK
- 任何控制卡 vendor DLL
- 任何 Native Handle / Native Struct / Native Enum
- 任何 `DllImport`

### 2. 禁止出現下列內容

- PCI 專屬邏輯
- EtherCAT 專屬邏輯
- Standalone 專屬通訊協定
- Polling
- UI
- 商業邏輯
- Application 層行為
- PLC/HMI 行為
- `Thread.Abort`
- `async void`
- `lock(this)`

### 3. Core Library 不可做的事

- 不可直接開硬體
- 不可直接控制驅動 DLL
- 不可直接做 EtherCAT PDO/SDO
- 不可直接做 PCI 卡號掃描
- 不可直接做控制器通訊協定實作

## 四、環境限制

- .NET Framework 4.8
- Windows 7 / Windows 10 相容
- 僅使用 Win7 可用 API
- Dispose 不可呼叫 virtual
- Finalizer 不可存取 managed 物件
- 高頻區域禁止 LINQ
- 高頻區域禁止隱性 boxing
- 高頻區域禁止 string.Format
- Dispose 必須完整釋放資源且可重入
- 所有例外必須透過 ILogger 記錄
- 所有 public API 必須 thread-safe

## 五、Logging 規範（必須遵守 Calin.Logging.Abstractions.LoggingBridge）

本專案必須使用：

- `Calin.Logging.Abstractions.LoggingBridge`

已知規則如下，必須完全遵守：

### LoggingBridge 使用限制

- Core Library / Device Library / NuGet Library **不可呼叫** `LoggingBridge.Initialize(...)`
- `LoggingBridge.Initialize(...)` **只能由 App 的 Composition Root 呼叫一次**
- 本專案只能使用：
  - `LoggingBridge.CreateLogger<T>()`
  - `LoggingBridge.CreateLogger(Type)`
  - `LoggingBridge.CreateLogger(string)`

### 必須保證

- 若 App 尚未初始化 `LoggingBridge`，本專案仍可正常執行
- 未初始化時，允許使用 `NullLogger`
- 不可因 logger 未初始化而拋例外
- 所有例外、狀態轉移失敗、命令失敗、模組故障、資源釋放錯誤，都必須經過 `ILogger` 記錄

## 六、核心抽象介面（必須建立）

本專案必須提供完整、穩定、可擴充的抽象介面：

### 控制器與設備抽象

- `IMotionSystem`
- `IMotionController`
- `IDevice`
- `IMotionCard`
- `IAxis`
- `IMotionGroup`
- `IIO`
- `INetworkController`

### 工廠與模組抽象

- `IMotionControllerFactory`
- `IDeviceFactory`
- `ICardFactory`
- `IAxisFactory`
- `IControllerModule`
- `IHardwareProvider`

### 診斷與時間抽象

- `ITimeProvider`
- `IDiagnosticsCollector`
- `ICommandScheduler`
- `ICommandQueue`

### 組態與參數抽象

- `IAxisConfiguration`
- `ICardConfiguration`
- `IDeviceConfiguration`
- `IControllerConfiguration`
- `INetworkConfiguration`
- `IParameterStore`

## 七、核心資料模型（必須建立）

本專案必須建立硬體無關的共用模型：

- `MotionCommand`
- `MotionCommandType`
- `MotionCommandPriority`
- `MotionState`
- `AxisState`
- `ControllerState`
- `IOStatus`
- `AxisStatus`
- `MotionResult`
- `MotionResult<T>`
- `MotionErrorCode`
- `MotionException`
- `StateViolationException`
- `SafetyPolicyViolationException`

## 八、狀態機（Atomic）

狀態機必須為核心共用能力，不依賴任何硬體實作。

### 控制器/卡/軸共用狀態

至少支援：

- `Disconnected`
- `Initializing`
- `Ready`
- `Enabled`
- `Faulted`
- `Disposed`

### 要求

- 使用 `Interlocked.CompareExchange`
- 所有狀態轉移 atomic
- 非法轉移拋出 `StateViolationException`
- 保存 `FaultReason`
- 記錄來源與目標狀態
- thread-safe

## 九、命令模型與排程模型

Core 必須建立統一命令模型，不得依賴硬體。

### 必須建立

- `MotionCommand`
- `CommandQueue`
- `ICommandScheduler`
- `CommandExecutionContext`
- `CommandResult`

### 命令優先權

- `Stop > Home > Move > Read`

### 規則

- `Stop` 永遠最高優先
- `Move` 可覆蓋 `Move`
- `Home` 必須可被 `Stop` 中止
- 指令佇列與排程邏輯屬於 Core，可供子類庫重用

## 十、安全策略層

建立：

- `IAxisSafetyPolicy`

規則抽象必須包含：

- Servo 未 ON 禁止 Move
- Alarm 未清除禁止 Enable
- RDY 未成立禁止動作
- Home 未完成禁止 Move（可設定）

但：

- 不可依賴特定硬體訊號格式
- 不可包含商業邏輯
- 只定義核心判斷規則與抽象輸入模型

## 十一、Capability（Immutable）

Capability 必須為 immutable，供不同子類庫實作時回報能力。

至少包含：

- `SupportsRdySignal`
- `SupportsSoftLimit`
- `SupportsEncoderFeedback`
- `SupportsInterpolation`
- `SupportsNetwork`
- `SupportsIoModule`
- `MaxAxisCount`
- `MaxVelocity`
- `MaxAcceleration`

不可 runtime 修改。

## 十二、Axis Reference Monitor

核心層必須提供：

- `AxisReferenceMonitor`

功能：

- 判斷軸位置相對於 Reference
- 提供 Side 狀態
- `SideChanged` 事件
- 支援多 Reference

### ReferenceSide

- `LessThan`
- `Equal`
- `GreaterThan`

### Axis API

- `AddReferenceMonitor`
- `RemoveReferenceMonitor`
- `GetReferenceMonitors`

要求：

- thread-safe
- 不可高頻配置
- 使用 `ArrayPool` 或預配置集合
- 支援高頻 Position 更新
- 位置由硬體子類庫提供，但 Reference 判斷邏輯必須位於 Core

## 十三、同步 / 插補預留

建立抽象：

- `IMotionGroup`
- `IInterpolationController`
- `ITrajectoryPlanner`

僅建立抽象與共用模型，不實作特定硬體插補。

## 十四、時間來源抽象

建立：

- `ITimeProvider`

內部 timeout / latency / diagnostics 必須使用：

- `Stopwatch`

禁止使用：

- `DateTime.Now`

作為 timeout 判斷。

## 十五、診斷模式

提供：

- 命令耗時統計
- 平均命令延遲
- 最長命令耗時
- Queue 深度
- 最近 N 筆記錄（RingBuffer）
- 狀態轉移記錄
- 故障記錄

診斷模式不得顯著影響正常模式效能。

## 十六、GC 控制

要求：

- `ArrayPool`
- `RingBuffer`
- `readonly struct` 事件模型
- 禁止高頻 new
- 禁止高頻 string allocation

## 十七、DI

預設 DI：

- `Autofac`

提供：

- `Autofac Module`

限制：

- 不可強依賴 Container
- Factory 不可持有 Container
- Core 必須可在無 DI 情境下使用
- DI 只作為組裝工具，不可成為核心依賴

## 十八、Fake / Simulator

本專案必須提供：

- `FakeMotionSystem`
- `FakeMotionController`
- `FakeMotionCard`
- `FakeAxis`
- `FakeIO`
- `FakeNetworkController`

要求：

- 不引用任何硬體 SDK
- 不引用任何 Native namespace
- 可模擬 Position 更新
- 可觸發 `AxisReferenceMonitor`
- 可用於單元測試與整合測試
- 行為需符合 Core 抽象契約

## 十九、子類庫擴充規則（非常重要）

本專案必須明確設計成供下列專案擴充：

- `Calin.MotionControl.Advantech.PCI`
- `Calin.MotionControl.Advantech.PCBased`
- `Calin.MotionControl.Advantech.EtherCAT`
- `Calin.MotionControl.Advantech.Standalone`

因此 Core 必須保證：

- 不依賴 `AdvMotAPI.dll`
- 不依賴任何 Native SDK
- 不依賴任何硬體型別
- 僅透過抽象介面與模型與子類庫互動

## 二十、專案結構要求

必須重建為清楚分層，例如：

```text
Calin.MotionControl.Advantech
 ├─ Abstractions
 ├─ Models
 ├─ States
 ├─ Commands
 ├─ Diagnostics
 ├─ Safety
 ├─ Monitoring
 ├─ Configuration
 ├─ Factories
 ├─ DependencyInjection
 ├─ Simulation
 ├─ Utilities
 └─ Exceptions
```

禁止在 Core 專案中出現：

- `Native`
- `DllImport`
- `AdvMotAPI`
- `PCI`
- `EtherCAT`
- `StandaloneDriver`

## 二十一、csproj 要求

`Calin.MotionControl.Advantech.csproj` 必須符合：

- SDK-style project
- `TargetFramework=net48`
- `LangVersion=9.0`
- `GenerateDocumentationFile=true`
- 可引用：
  - `Autofac`
  - `System.Buffers`
  - `Microsoft.Extensions.Logging.Abstractions`
  - `Calin.Logging`
- 不可引用：
  - `AdvMotAPI.dll`
  - 任何 Native DLL
  - 任何 PCI / EtherCAT / Standalone 專案

## 二十二、輸出要求

Copilot 必須：

- 生成完整可編譯專案
- 所有 public API 提供正體中文 XML Summary
- 不產生任何 Native 封裝
- 不產生任何硬體專屬邏輯
- 不產生任何 `DllImport`
- 不產生 Polling
- 不產生 UI
- 不產生商業邏輯
- 使用 segmented generation
- 不要一次生成整個專案
- 優先先生成：
  1. `csproj`
  2. `Abstractions`
  3. `Models`
  4. `Exceptions`
  5. `States`
  6. `Diagnostics`
  7. `Simulation`
  8. `DependencyInjection`
