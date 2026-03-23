---
aliases:
created: 
update:
author:
language:
sourceurl:
tags:
date:
---

# 完整的生成 Prompt

你是一位資深 .NET 架構師與企業級工控通訊框架設計專家。

請產生一個**完整可編譯**、**可直接建立為 Visual Studio 解決方案**的 .NET Framework 4.7.2 多專案架構骨架。

主體專案名稱為：

Calin.SerialPort

本版本採用：

分散式多實例模型（每個設備自行建立 SerialPortService）
不提供集中式管理器
但必須防止同一 PortName 被重複開啟

目標是：

高穩定、低資源占用、可長時間 24/7 運作、可未來擴充 TCP / Modbus / CAN / PLC 的企業級通訊基礎框架。

請直接輸出完整專案檔案內容（含 .csproj、.cs 檔、README、範例程式）。
不得只提供說明文字。

## 【整體解決方案架構】

建立以下三個專案：

1. Calin.IO.Abstractions（Class Library, .NET Framework 4.7.2）
2. Calin.Transport.Core（Class Library, .NET Framework 4.7.2）
3. Calin.SerialPort（Class Library, .NET Framework 4.7.2）

## 【架構定位】

### Calin.IO.Abstractions

純抽象層，不包含任何實作。

不得引用 RJCP。

包含：

- IConnection
- IStreamPort
- IRetryPolicy
- IHealthReportProvider
- ILogger（引用 Calin.Logging 抽象）
- TransportResult
- HealthReport
- ErrorCode enum

必須符合 Interface Segregation。

### Calin.Transport.Core

依賴 Calin.IO.Abstractions。

功能：

- RetryPolicy Base 實作
- Transport Base Class
- AutoReconnect Pipeline
- HealthReport 基礎實作
- 背景讀取迴圈模板
- CancellationToken 管理
- 統一錯誤封裝
- Channel 封裝層（避免事件阻塞讀取執行緒）

不得引用 RJCP。

不得依賴 SerialPort。

### Calin.SerialPort

依賴：

- Calin.IO.Abstractions
- Calin.Transport.Core
- RJCP.SerialPortStream
- Autofac
- Newtonsoft.Json
- Calin.Logging

僅此專案可引用 RJCP。

對外不可暴露 RJCP 型別。

## 【設計哲學】

1. 分散式多實例模型

   - 每個設備自行建立 SerialPortService
   - 不提供 SerialPortManager
2. 支援多實例並行

   - 不同 PortName 可同時運作
3. 必須防止同一 PortName 被重複開啟

   - 使用 internal PortRegistry
   - 使用 ConcurrentDictionary\<string, byte>
   - 若重複開啟 → 拋 PortAlreadyInUseException
4. 保留未來擴充為 TCP / Modbus / CAN / PLC 的能力
5. 優先確保穩定性、相容性、低資源占用

## 【狀態機】

SerialPortState：

- Disconnected
- Connecting
- Ready
- Fault

狀態轉換：

Open() → Connecting → Ready / Fault
心跳失敗 → Fault
物理斷線 → Fault
AutoReconnect 成功 → Ready
Close() → Disconnected

## 【SerialPortConfig 必須包含】

PortName
BaudRate
DataBits
Parity
StopBits
Handshake
ReadTimeout
WriteTimeout
RtsEnable
DtrEnable
EnableAutoReconnect
ReconnectInterval
EnableHeartbeat
HeartbeatInterval
HeartbeatMessage
HeartbeatTimeout
TextEncoding (Ascii / Utf8)
AsciiLineTerminator
EnableAsciiLineMode
AsciiReceiveBufferLimit
EnableTransmissionVerification
TransmissionTestMessage
TransmissionTestTimeout

必須提供 Clone()

## 【ISerialPortService】

屬性：

PortName
State
IsReady
IsOpen
IsTransmissionVerified
Config
HealthReport

方法：

Open(CancellationToken)
Close()
SendData(string)
SendData(byte[])
SendAscii(string)
SendAsciiLine(string)
SaveConfig(string path)
LoadConfig(string path)

事件：

StateChanged
DataReceived
ErrorOccurred

必須實作 IDisposable。

## 【必要企業級功能】

1. Factory Pattern
2. 狀態機設計
3. 背景讀取 Task（延遲 10ms，不可 busy-wait）
4. 心跳監控 Task
5. 自動重連 Task
6. RetryPolicy 抽象
7. HealthReport
8. 傳輸驗證機制
9. JSON 設定儲存/載入
10. DI 與非 DI 雙支援
11. FakeSerialPort（測試用）
12. CancellationToken 外部控制
13. 統一錯誤封裝（不可直接丟 RJCP 例外）

## 【效能優化層】

必須包含：

- 內部 Channel 佇列（避免 DataReceived 阻塞讀取迴圈）
- ArrayPool\<byte>
- 使用 Memory\<byte>
- 減少 buffer 複製
- 事件觸發不可阻塞讀取 Task
- 所有 Task 可安全取消
- Dispose 必須完整釋放背景任務
- 不可記憶體洩漏
- 支援大量設備並行
- 適用 Windows 7 / Windows 10
- 不使用 .NET Core API

若與穩定性或效能衝突，優先穩定性與效能。

## 【Autofac】

提供：

SerialPortModule

功能：

- 註冊 ISerialPortService
- 支援 Named Registration
- 不包含業務邏輯

## 【README 必須包含】

1. 架構說明
2. 分散式模型說明
3. 為何不提供集中式管理器
4. 多實例使用方式
5. DI 範例
6. 非 DI 範例
7. JSON 設定範例
8. 傳輸驗證說明
9. 心跳說明
10. 已知限制
11. 未來擴充方向

## 【輸出要求】

1. 輸出完整解決方案檔案結構
2. 每個專案 .csproj
3. 所有 .cs 檔完整內容
4. 每個類別必須包含 XML Summary
5. 完整 README.md
6. Console 範例程式
7. 不得省略必要實作
8. 必須可編譯
9. 不得輸出設計說明文字
10. 只輸出專案內容
11. 所有註解及 README 內容，除程式碼及專業術語，均使用正體中文，圖表使用 Mermaid 格式

請開始產生完整專案內容。

---

好的，我幫你把 **Step 1~5 的分段 Prompt**整理成一份 **Claude 4.6 可直接逐步貼入的完整執行清單**，每個 Step 都有編號、描述與 Prompt，本清單可以依序執行，最終得到完整可編譯的解決方案骨架。

---

# Claude 4.6 專案骨架分段生成清單

## Step 1：生成解決方案與專案結構

**目標**：建立 Calin.SerialPort.sln 及三個專案資料夾與 .csproj
Prompt：

```markdown
你是一位資深 .NET 架構師，請生成一個完整的 .NET Framework 4.7.2 解決方案骨架：

解決方案名稱：Calin.SerialPort

包含三個專案：
1. Calin.IO.Abstractions（Class Library）
2. Calin.Transport.Core（Class Library，依賴 Calin.IO.Abstractions）
3. Calin.SerialPort（Class Library，依賴 Calin.IO.Abstractions 與 Calin.Transport.Core，可引用 RJCP.SerialPortStream、Autofac、Newtonsoft.Json）

請輸出：
- Calin.SerialPort.sln 內容
- 每個專案完整 .csproj 文件
- 各專案 .csproj 內容
- 專案資料夾結構（建立空檔案即可，例如 Program.cs 可留空）
- 不包含任何程式碼實作
- 不可循環參考
- Abstractions 不依賴 Implementation
- Core 不依賴 UI
- 先只生成專案檔，不生成程式碼
- 可直接用 Visual Studio 開啟
```

## Step 2：生成 Calin.IO.Abstractions 專案程式碼

**目標**：建立抽象層核心介面與資料結構
Prompt：

```markdown
你是一位資深 .NET 架構師，請生成 Calin.IO.Abstractions 專案內所有 .cs 檔：

全域設計優先原則（強制遵守）
- 優先順序：
  1. 穩定性
  2. 相容性
  3. 效能
  4. 可預測性
  5. 可維護性
  6. 擴充性
  7. 優雅設計
- 必須支援 24/7 長時間運作
- 不可有記憶體洩漏
- 所有背景 Task 必須可安全取消，且使用 CancellationToken
- Dispose 必須完整釋放資源且可重入
- 所有 public 方法需 thread-safe
- 支援大量設備並行（100+）
- 事件不可阻塞 I/O 執行緒
- 避免 race condition / deadlock / event re-entrancy
- 不可使用 Thread.Abort
- 僅使用 Windows 7/10 相容 API
- 減少物件分配與 GC 壓力
- 僅建立必要抽象層，避免過度設計
- 所有例外必須捕捉並回報
- 遵守 Interface Segregation 原則
- 使用 ILogger（來自 Calin.Logging）
- 每個類別與介面必須有 XML Summary（正體中文）
- 若設計與穩定性或效能衝突，優先穩定性與效能

內容需求：
- 介面：IConnection, IStreamPort, IRetryPolicy, IHealthReportProvider
- 資料結構與類別：TransportResult, HealthReport, ErrorCode enum
- 明確 Dispose 合約
- 事件安全訂閱/取消
- 不暴露具體實作型別
- 不引用 RJCP
- 可直接編譯
- 生成完後只產生 Calin.IO.Abstractions 專案程式碼
```

## Step 3：生成 Calin.Transport.Core 專案程式碼

**目標**：建立傳輸核心與背景處理模板
Prompt：

```markdown
你是一位資深 .NET 架構師，請生成 Calin.Transport.Core 專案內所有 .cs 檔：

全域設計優先原則（強制遵守）
- 優先順序：
  1. 穩定性
  2. 相容性
  3. 效能
  4. 可預測性
  5. 可維護性
  6. 擴充性
  7. 優雅設計
- 必須支援 24/7 長時間運作
- 不可有記憶體洩漏
- 所有背景 Task 必須可安全取消，且使用 CancellationToken
- Dispose 必須完整釋放資源且可重入
- 所有 public 方法需 thread-safe
- 支援大量設備並行（100+）
- 事件不可阻塞 I/O 執行緒
- 避免 race condition / deadlock / event re-entrancy
- 不可使用 Thread.Abort
- 僅使用 Windows 7/10 相容 API
- 減少物件分配與 GC 壓力
- 僅建立必要抽象層，避免過度設計
- 所有例外必須捕捉並回報
- 遵守 Interface Segregation 原則
- 使用 ILogger（來自 Calin.Logging）
- 每個類別與介面必須有 XML Summary（正體中文）
- 若設計與穩定性或效能衝突，優先穩定性與效能

非同步規則：
- 禁止使用 async void
- 禁止 fire-and-forget：所有 Task 或 ValueTask 必須 await 或安全捕捉例外
- 可在內部短期、頻繁非同步方法使用 async ValueTask
- 外部公開 API 或長時間 Task 使用 async Task
- 背景 Task / Channel / 心跳 / 傳輸驗證 / 自動重連均必須符合此規則
- 事件觸發不可阻塞 Task，異常必須捕捉並回報
- 所有 Task 支援 CancellationToken 安全停止

內容需求：
- 依賴 Calin.IO.Abstractions
- RetryPolicy 抽象及 Base 實作
- Transport Base Class
- AutoReconnect Pipeline
- HealthReport 基礎實作
- 背景讀取 Task 模板（延遲 10ms，不使用 busy-wait）
- CancellationToken 管理
- 統一錯誤封裝策略：
    - 禁止直接丟出底層例外（如 RJCP.SerialPortStream）
    - 所有操作、背景 Task、心跳、傳輸驗證、Send / Open / Close 例外必須封裝
    - 使用 ErrorCode enum 表示錯誤類型
    - 操作結果使用 TransportResult 或 OperationResult 包含：
        - Success / Failure
        - ErrorCode
        - 描述訊息
        - 可選內部 Exception（僅內部記錄）
    - 所有失敗透過 ErrorOccurred 事件通知
    - 更新 HealthReport 記錄錯誤
    - 多實例並行，錯誤影響僅限該 PortName
    - 背景 Task 與事件觸發均需非同步安全
- Channel 封裝層要求：
    - 每個資料流使用 Channel 或 ConcurrentQueue + SemaphoreSlim
    - 支援固定容量，超出容量時行為明確
    - 提供 EnqueueData / TryDequeue / Flush / Clear
    - 背景 Task 從 Channel 非同步取資料處理，觸發事件不可阻塞
    - 支援 CancellationToken 安全停止
    - Channel 僅負責資料緩衝與排程，不包含業務邏輯
    - 可選：Priority Channel、Batch Dequeue、容量監控
- 明確定義執行緒策略
- 佇列最大容量，佇列滿時行為明確
- 狀態機完整，必須實作完整狀態轉換邏輯與合法轉換檢查，不可任意跳轉狀態。
- 記憶體管理：ArrayPool<byte>、Memory<byte>、減少 buffer 複製
- 所有 Task 可安全取消
- 不引用 RJCP 或 SerialPort
- 每個類別含 XML Summary (使用正體中文)
- 可直接編譯
```

## Step 4：生成 Calin.SerialPort 專案程式碼

**目標**：實作企業級 + 工控級安全標準的 SerialPortService，基於 RJCP.SerialPortStream
Prompt：

```markdown
你是一位資深 .NET 架構師，請生成 Calin.SerialPort 專案內所有 .cs 檔，必須完整實作且可編譯：

全域設計優先原則（強制遵守）
- 優先順序：
  1. 穩定性
  2. 相容性
  3. 效能
  4. 可預測性
  5. 可維護性
  6. 擴充性
  7. 優雅設計
- 必須支援 24/7 長時間運作
- 不可有記憶體洩漏
- 所有背景 Task 必須可安全取消，且使用 CancellationToken
- Dispose 必須完整釋放資源且可重入
- 所有 public 方法需 thread-safe
- 支援大量設備並行（100+）
- 事件不可阻塞 I/O 執行緒
- 避免 race condition / deadlock / event re-entrancy
- 不可使用 Thread.Abort
- 僅使用 Windows 7/10 相容 API
- 減少物件分配與 GC 壓力
- 僅建立必要抽象層，避免過度設計
- 所有例外必須捕捉並回報
- 遵守 Interface Segregation 原則
- 使用 ILogger（來自 Calin.Logging）
- 每個類別與介面必須有 XML Summary（正體中文）
- 若設計與穩定性或效能衝突，優先穩定性與效能

非同步規則：
- 禁止使用 async void
- 禁止 fire-and-forget：所有 Task 或 ValueTask 必須 await 或安全捕捉例外
- 可在內部短期、頻繁非同步方法使用 async ValueTask
- 外部公開 API 或長時間 Task 使用 async Task
- 背景 Task / Channel / 心跳 / 傳輸驗證 / 自動重連均必須符合此規則
- 事件觸發不可阻塞 Task，異常必須捕捉並回報
- 所有 Task 支援 CancellationToken 安全停止
- 背景 Task 不可因例外終止

依賴：
- Calin.IO.Abstractions
- Calin.Transport.Core
可引用：
- RJCP.SerialPortStream（僅此專案可引用，對外不可暴露）
- Autofac
- Newtonsoft.Json
- Calin.Logging

類別與介面：
- ISerialPortService 實作：
    - 屬性：PortName, State, IsReady, IsOpen, IsTransmissionVerified (Transmission Verification), Config, HealthReport
    - 方法：Open(CancellationToken), Close(), SendData(string/byte[]), SendAscii(string), SendAsciiLine(string), SaveConfig(path), LoadConfig(path)
    - 事件：StateChanged, DataReceived, ErrorOccurred
    - IDisposable 實作
- SerialPortConfig 完整欄位，提供 Clone()

- 背景 Task：
    - 背景讀取 Task（延遲 10ms，不使用 busy-wait）
    - 心跳監控 Task
    - AutoReconnect Task

- 狀態機強制規則：
    - 定義 SerialPortState enum：Disconnected, Connecting, Ready, Fault
    - 狀態轉換規則：
        - Open() : Disconnected -> Connecting
        - Open 成功 : Connecting -> Ready
        - Open 失敗 : Connecting -> Fault
        - 心跳失敗 : Ready -> Fault
        - 傳輸驗證失敗 : Ready -> Fault
        - 物理斷線 : Ready -> Fault
        - Read/Write IOException : Ready -> Fault
        - 裝置消失 : Ready -> Fault
        - AutoReconnect 成功 : Fault -> Ready
        - Close() : 任意非 Disconnected -> Disconnected
    - 禁止非法狀態跳轉，若違規 → 記錄 Error / 更新 HealthReport
    - 狀態變更透過 StateChanged 事件通知，事件觸發不可阻塞
    - 狀態與屬性同步：
        - IsReady = (State == Ready)
        - IsOpen = (State != Disconnected)
        - IsTransmissionVerified 與 Transmission Verification 結果同步
    - 狀態更新必須 thread-safe
    - 支援 CancellationToken 安全停止
- Open / Close / Send 不得競態：
    - 同一 PortName 同時只允許一個執行緒操作
    - 背景 Task (心跳、傳輸驗證、自動重連、Channel 出隊) 不可與 Open/Close/Send 競態
    - 建議使用 lock 或 SemaphoreSlim 包裹操作
    - 所有操作支援 CancellationToken 安全取消
    - 配合狀態機：
        - Open() 只能從 Disconnected
        - Close() 只能從非 Disconnected
        - SendData / SendAscii / SendAsciiLine 只能從 Ready
    - 確保操作完成後釋放鎖，避免死鎖
    - 禁止 async void / fire-and-forget
- 工控級硬體斷線偵測（強制）：
    - 強制持續 Read Loop：
        - 必須有常駐背景 ReadAsync Task
        - while + CancellationToken
        - 捕捉 IOException / UnauthorizedAccessException / ObjectDisposedException
        - 發生例外時立即：
            - State -> Fault
            - 更新 HealthReport
            - 觸發 ErrorOccurred
            - 啟動 AutoReconnect
        - 資料處理流程：
            - ReadAsync → Append 到 LineBuffer
            - 若 AsciiLineTerminator != "":
                - while TryExtractLine：
                    - 非阻塞觸發 DataReceived
            - 若 Terminator = "":
                - Raw 模式
                - 每次 Read 直接觸發 DataReceived
        - 不得使用：
            - ReadLine()
            - StringBuilder
            - 阻塞式讀取
    - WriteAsync 失敗即斷線：
        - WriteAsync 發生 IOException 視為連線異常
        - 立即 State -> Fault
        - 更新 HealthReport
        - 啟動 AutoReconnect
    - 心跳為補強機制：
        - 心跳失敗 -> Fault
        - 心跳不可為唯一斷線偵測手段
    - 背景 Task 不得因例外終止
- 裝置消失檢查（Device Presence Check）：
    - 定期檢查目前 PortName 是否仍存在於系統 Port 清單
    - 若不存在：
        - 視為 DeviceNotPresent
        - State -> Fault
        - 啟動 AutoReconnect
    - Open 前必須驗證 Port 存在
    - 不得依賴 OS 即時通知
- 自動重連機制：
    - Fault 狀態啟動
    - 可設定重試間隔與最大次數
    - 支援 CancellationToken
    - 重連成功 -> Ready
    - 重連失敗 -> 維持 Fault
- ASCII 尾端可變更選項（強制支援）
    - SerialPortConfig 必須新增：
        - string AsciiLineTerminator
            - 預設 "\r\n"
            - 不可為 null（若 JSON 為 null，自動修正為 "\r\n" 並記錄 Warning）
            - 允許：
                - "\r\n"
                - "\n"
                - "\r"
                - ""
                - 任意字串
        - bool AutoAppendLineTerminator
            - 預設 true
        - Clone() 必須完整複製。
        - JSON Save / Load 必須支援。
    - SendAscii(string data)：
        - 僅送 ASCII bytes
        - 不附加 Terminator
        - 使用 Encoding.ASCII
        - 不得產生多餘字串分配
    - SendAsciiLine(string data)：
        - 若 AutoAppendLineTerminator = true：
            - 傳送 data + AsciiLineTerminator
        - 若 false：
            - 等同行為於 SendAscii
        - 不可使用字串拼接
        - 建議：
            - 快取 terminator bytes
            - ArrayPool<byte>
            - stackalloc（小資料）
                - 必須 thread-safe
                - 不可造成 GC 壓力
    - Transmission Verification / Heartbeat：
        - 若訊息為 ASCII 字串
        - 是否附加 Terminator 必須依 AutoAppendLineTerminator 決定
        - 不可硬編碼 "\r\n"
- 工控級 LineBuffer（強制實作）
    - 必須新增 internal 類別：
        - LineBuffer
        - 功能：
            - Append(ReadOnlySpan<byte>)
            - TryExtractLine(out ReadOnlyMemory<byte>)
            - Clear()
            - Dispose()
    - 設計規範：
        - 內部欄位：
            - byte[] _buffer
            - int _length
            - byte[] _terminator
            - int _maxLineLength（預設 4096 或來自 Config）
        - 行為：
            - Append：
                - 累積資料
                - 空間不足時成長為 2 倍
                - 不可每次 new byte[]
            - TryExtractLine：
                - 若 terminator 為空字串 → Raw 模式 → 回傳 false
                - 搜尋 terminator（使用 Span.IndexOf）
                - 找到：
                  - 使用 ArrayPool<byte>
                  - 複製完整行
                  - Compact 內部 buffer
                  - 回傳 true
                - 找不到：
                  - 回傳 false
            - Compact：
                - 移除已取出資料
                - 將剩餘資料前移
                - 更新 _length
                - 不重新 new 陣列
    - 安全限制：
        - 若累積長度 > _maxLineLength：
            - 記錄 ProtocolError
            - 清空 buffer
            - 不直接進 Fault
        - 不可因 Terminator 設錯造成無限累積
        - 不可死循環
- 統一錯誤封裝策略：
    - 禁止直接丟出底層例外（如 RJCP.SerialPortStream）
    - 所有操作、背景 Task、心跳、傳輸驗證、Send / Open / Close 例外必須封裝
    - 使用 ErrorCode enum 表示錯誤類型
    - 操作結果使用 TransportResult 或 OperationResult 包含：
        - Success / Failure
        - ErrorCode
        - 描述訊息
        - 可選內部 Exception（僅內部記錄）
    - 所有失敗透過 ErrorOccurred 事件通知
    - 更新 HealthReport 記錄錯誤
    - 多實例並行，錯誤影響僅限該 PortName
    - 背景 Task 與事件觸發均需 thread-safe
    - 錯誤類型需區分：
        - CommunicationFault
        - DeviceNotPresent
        - ProtocolError
        - HeartbeatTimeout
        - TransmissionVerificationFailed
- PortRegistry（ConcurrentDictionary<string, byte>），防止同一 PortName 被重複開啟，若重複拋 PortAlreadyInUseException
- Public API surface 不得包含任何 RJCP 型別或例外，必須封裝轉換
- Transmission Verification 流程要求：
    - 屬性：
        - EnableTransmissionVerification (bool)
        - TransmissionTestMessage (byte[] / string)
        - TransmissionTestTimeout (ms)
    - 每次 SendData / SendAscii / SendAsciiLine 後觸發驗證
    - 發送 TransmissionTestMessage，等待設備回應（透過 LineBuffer）
    - 回應符合規則 → IsTransmissionVerified = true
    - 回應失敗或 Timeout → IsTransmissionVerified = false，觸發 ErrorOccurred
    - 支援 RetryPolicy 重試多次
    - 背景 Task 非阻塞，使用 CancellationToken 安全停止
    - 與狀態機整合，Transmission Verification 結果影響 State / IsTransmissionVerified
    - 配置可調整 Timeout、TestMessage、啟用開關
- Heartbeat 背景任務要求：
    - 屬性：
        - EnableHeartbeat (bool)
        - HeartbeatInterval (ms)
        - HeartbeatMessage (byte[] / string)
        - HeartbeatTimeout (ms)
    - 背景 Task 定時發送 HeartbeatMessage
    - 等待回應：
        - 成功 → State = Ready, IsReady = true
        - 失敗 / Timeout → State = Fault, IsReady = false, 觸發 ErrorOccurred
    - 心跳失敗可觸發自動重連 Task
    - 支援 CancellationToken 安全停止
    - 背景 Task 不阻塞資料讀取 Task
    - 多實例獨立運作，每個 PortName 擁有自己的心跳
- 多實例支援：
    - 不同 PortName 可並行
    - 每個實例獨立狀態與 HealthReport
    - 錯誤不得影響其他實例
- JSON 設定儲存/載入
- FakeSerialPort 測試支援
- IDisposable 完整釋放背景任務
    - 取消 Token
    - 等待 Task 結束
    - 清空 LineBuffer
    - 清空 Channel
    - 釋放 PortRegistry
    - 釋放 RJCP 實例
- 支援 DI 與非 DI（包含 Autofac Module、Named Registration）
- 所有類別與介面含 XML Summary (使用正體中文)

要求：
- 可直接編譯
- 請依檔案類型與用途自動將檔案分資料夾存放：
    - 介面、抽象類別、DTO/Models、核心 Service、事件、Channel、Fake 測試、配置、範例程式等
    - namespace 與資料夾保持一致
    - ROOT 僅放 README.md 與必要專案檔
    - 無需明確指定每個資料夾名稱，由 Copilot 判斷最合理的分層
- 所有功能必須完整實作
- 背景 Task 可安全取消，不可 silent crash
- 事件觸發不可阻塞讀取 Task
- 不得因例外導致服務失效
```

## Step 5：生成 README.md 與 Console 範例程式 + 壓力測試

**目標**：生成 README、範例 Console App、Mermaid 流程圖
Prompt：

```markdown
你是一位資深 .NET 架構師，請生成 Calin.SerialPort 專案的 README.md 與 Console 範例程式：

README.md 需求：
- 架構說明、分散式模型、多實例使用方式
- 為何不提供集中式管理器
- DI 與非 DI 範例
- JSON 設定範例
- 傳輸驗證說明、心跳說明
- 已知限制
- 未來擴充方向
- 使用 Mermaid 圖表表示架構
- 支援 OS 與 .NET 版本、不適用場景

Console 範例程式：
- 開啟 SerialPort
- 讀寫資料
- 訂閱事件
- 可直接編譯

壓力測試條件：
- 同時 100+ 裝置連線，高頻資料輸入
- 連續運作 48 小時
- 模擬斷線、高頻錯誤、大量 reconnect
- 驗證記憶體、CPU、Handle、執行緒穩定性
```

這樣的流程可以逐步貼入 Claude 4.6，每次只生成一個專案或 README，Token 不會爆掉，而且每個部分都是完整可編譯的專案骨架，最終得到完整、可編譯、企業級的 Calin.SerialPort 架構。

---

# 為何分段的 Prompt 比較短？

完全正常，而且這其實是刻意設計的。原因有幾點：

## **Token 控制**

- 原本完整 Prompt 一次就包含所有專案、類別、屬性、事件、Task、背景邏輯、README、範例程式……非常龐大，直接貼給 Claude 4.6 會爆 Token。
- 分段後，每段只專注一個專案或一個任務（例如 Step 2 只生成 Calin.IO.Abstractions 的程式碼），Token 消耗大幅下降。

## **細節保留策略**

- 每個 Step 都把原先完整 Prompt 裡對該專案或部分的「所有細節要求」保留下來，例如：
    - Step 2 保留了 Interface Segregation、XML Summary、禁止 RJCP 等要求
    - Step 4 保留了 SerialPortConfig 欄位、狀態機、背景 Task、PortRegistry、傳輸驗證、Dispose 等要求
    - Step 5 保留了 README 的全部內容與 Mermaid 圖表要求

換句話說，每段的內容是「針對該部分的完整規格」，所以**不會遺漏細節**。

## **可組合性**

- 分段生成後，你可以逐步在 Claude 4.6 執行，每段生成後再組合到解決方案中。
- 最終產出的專案骨架跟一次生成是等效的，只是生成流程被拆成多步。

## **優勢**

- 更容易 debug，如果某個 Step 出問題，只需重跑該 Step
- Token 消耗可控
- 每個 Step 可直接產生可編譯程式碼，不需依賴其他 Step 完全生成才看得懂

所以，**分段 Prompt 比完整 Prompt 短很多是正常的，而且不會遺漏細節**。你只要按照 Step 1~5 逐步執行，最終得到的解決方案和一次性生成的結果是等價的。

---

# 直接傳送 string 與轉成 byte[] 傳送的差別

這個問題其實涉及 **.NET SerialPort I/O 的底層行為**，字串 vs byte[] 的差異不只是型別轉換，也影響編碼與效率。以下詳細分析：

## 一、SerialPort.Write(string)

- 方法簽名：`SerialPort.Write(string text)`
- 行為：
    - 內部會依據 SerialPort 的 **Encoding 屬性** 將 string 轉成 byte[]
        - 預設是 `Encoding.Default`（依作業系統）
        - 可改成 `Encoding.ASCII` 或 `Encoding.UTF8`
    - 轉換完成後再呼叫底層 Write 送出 byte[]
- 優點：
    - 直接寫 string，程式碼簡單可讀
    - 適合 ASCII / 可預測編碼
- 缺點：
    - 依賴 SerialPort.Encoding，若 Encoding 與設備不一致可能造成亂碼
    - 每次寫入都會產生新的 byte[]，小訊息頻繁寫入會增加 GC 壓力
    - 無法完全控制 byte[] 內容（無法手動插入特定控制字元）

## 二、SerialPort.Write(byte[] buffer, int offset, int count)

- 行為：
    - 直接送 byte[]，完全由開發者控制
    - 不經 Encoding 轉換
    - 適合二進位資料或需要精準控制字元的情況
- 優點：
    - 完全控制送出的 byte 內容
    - 避免 Encoding 轉換帶來的問題
    - 在高頻 I/O 或長時間運作時，GC 壓力可降（可使用 ArrayPool）
- 缺點：
    - 程式碼可讀性較低，需要自行處理編碼

## 三、效率比較

|方法|編碼|GC 影響|精準度|適用情境|
|---|---|---|---|---|
|Write(string)|SerialPort.Encoding 轉 byte|每次轉 byte[]，高頻使用增加 GC|依 Encoding，可能出錯|ASCII/簡單文字，低頻 I/O|
|Write(byte[])|無需轉換|可重複使用 buffer，低 GC|完全精準|二進位資料、高頻 I/O、精確控制尾端字元|

## 四、實務建議（工控/長時間 RS-232 通訊）

1. **ASCII 通訊、短字串、低頻**
    - 可直接使用 `Write(string)`
    - 設定 `SerialPort.Encoding = Encoding.ASCII` 確保一致
2. **高頻率或長時間運作**
    - 建議 `Write(byte[])`
    - 自行用 `Encoding.ASCII.GetBytes` 或 ArrayPool 取得 buffer
    - 避免每次都產生新的 byte[]
3. **需要尾端控制字元**
    - 若固定 \r\n，可在 byte[] 中手動添加，避免 string 轉換不一致
4. **混合協定 / 二進位控制碼**
    - 一律用 byte[]
    - 可以更容易實作 Reconfigure / LineBuffer / Retry
