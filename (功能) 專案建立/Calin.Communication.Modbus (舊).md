# Modbus 工控強化最終版（RTU + ASCII）

## 企業級最嚴謹完整版 PROMPT（技術條款 + 架構條款 + 約束矩陣 + Initialize 機制）

你是一位資深 .NET 工控通訊架構師與 24/7 高可靠系統設計專家。
請在既有架構：

Application
↓
Calin.SerialPort
↓
Calin.Transport.Core
↓
Calin.IO.Abstractions

之上實作企業級 `Calin.Modbus`。

不得推翻既有抽象與基礎設施。
不得修改 `Calin.IO.Abstractions`。
不得引入第三方套件。

本專案為工控等級 24/7 長時間運作核心庫。
不要簡化。不要過度抽象。不要忽略邊界條件。

## 一、設計優先順序（強制遵守）

1. 穩定性
2. 相容性
3. 效能
4. 可預測性
5. 可維護性
6. 擴充性
7. 優雅設計

若設計與穩定性或效能衝突，優先穩定性。

## 二、三層架構（強制）

1. 基礎父類庫：Calin.Modbus

- 僅負責：
  - Modbus 協定完整實作（RTU + ASCII）
  - Frame 建構與解析
  - Exception Code 精細化
  - Retry 整合
  - Health 回報
  - 並發控制
- 不包含：
  - AutoReconnect
  - Background Polling
  - Heartbeat
  - UI
  - 商業邏輯
- 不會自動生成任何設備子類

1. 子類專案（例如 Calin.Modbus.DeviceX）
   - 繼承 IModbusClient 或包裝為 Service Layer
   - 可實作：
     - 背景 Polling Task
     - 事件封裝
     - 狀態管理
     - FakeDevice
   - 不得修改父類庫

2. Application
   - 僅依賴抽象介面
   - 不直接依賴 RTU/ASCII 實作

## 三、DI 與 非 DI 雙模式（嚴格規範）

### 核心原則

- DI 僅用來取得 IModbusClient
- 註冊時不得建立實例
- 實例必須在呼叫 Initialize() 後才建立
- 非 DI 用法與 DI 完全一致（同樣需要呼叫 Initialize()）

### API 規範

```csharp
public interface IModbusClient : IDisposable
{
    Task InitializeAsync(CancellationToken ct);

    Task<ushort[]> ReadHoldingRegistersAsync(
        byte slaveId,
        ushort startAddress,
        ushort quantity,
        CancellationToken ct);

    Task<bool> WriteSingleRegisterAsync(
        byte slaveId,
        ushort address,
        ushort value,
        CancellationToken ct);
}
```

規範：

- 未呼叫 InitializeAsync 前：
  - 所有 I/O API 必須丟出 InvalidOperationException
- InitializeAsync：
  - 僅可成功一次
  - 必須 thread-safe
  - 不得阻塞呼叫執行緒
- Dispose：
  - 可重入
  - 必須安全釋放資源
- 所有 public 方法 thread-safe

### DI 註冊規範

- 提供：
  - AddModbus()
  - RegisterModbusRtuClient()
  - RegisterModbusAsciiClient()
- 註冊時：
  - 不得建立 Serial 連線
  - 不得建立背景 Task
- 僅在 InitializeAsync 才建立內部 Transport

### 非 DI 用法

```csharp
var client = new ModbusRtuClient(
    streamPort,
    retryPolicy,
    logger,
    options);

await client.InitializeAsync(ct);
```

必須與 DI 用法行為一致。

## 四、Modbus 協定支援（完整）

### 必須同時支援

1. Modbus RTU
2. Modbus ASCII

### Modbus RTU 規範

- 使用 CRC16
- 呼叫 SerialPortHelper.CalculateCrc16
- Binary Frame，不可轉字串
- 處理 Silent Interval（≥3.5 char time）
- 不假設 DataReceived 為完整 Frame
- 根據 FunctionCode + ByteCount 動態判斷長度
- 最大 ADU 256 bytes，超出拒絕

### Modbus ASCII 規範

- 使用 LRC
- 呼叫 SerialPortHelper.CalculateLrc
- 「:」開頭
- 預設 CRLF 結尾（FrameTerminator 可設定）
- Line Buffer 組裝
- 解析大小寫不敏感
- 發送固定大寫
- 最大長度檢查

## 五、Function Code（完整支援）

0x01
0x02
0x03
0x04
0x05
0x06
0x0F
0x10
0x11
0x17

RTU 支援 0x08 Diagnostics

必須支援 Broadcast（SlaveId=0）：

- 不等待回應
- 不可重試
- 直接視為成功

## 六、Exception Code 精細化（不得簡化）

必須解析：

- Illegal Function
- Illegal Data Address
- Illegal Data Value
- Slave Device Failure
- Slave Device Busy

規則：

- 僅 Slave Busy 可由 IRetryPolicy 判斷是否重試
- 其他不可重試
- 不得簡化為單一 ProtocolError

## 七、錯誤映射（強制）

CRC/LRC 錯誤 → ChecksumError
Timeout → Timeout
Exception → ProtocolError（含細分類）
I/O 失敗 → TransportError
回應長度錯誤 → ProtocolError
SlaveId 不符 → ProtocolError

不得丟出原始 IOException。

## 八、並發模型

- 每個 Slave 強制序列化請求（SemaphoreSlim(1,1)）
- 同一 Slave 不得並行
- 不同 Slave 可並行
- 避免 out-of-order response

## 九、Retry 策略

- 必須使用 IRetryPolicy
- 不得自行寫 retry loop
- 不得重試 ChecksumError
- 不得重試 ProtocolError（除 Slave Busy）
- Broadcast 不可重試

## 十、Health 整合

使用 IHealthReportProvider

HealthReport 必須包含：

- 最後成功時間
- 連續失敗次數
- 是否 Offline

規則：

- 連續 N 次 Timeout → Offline
- 成功後重置計數
- 不得自行實作 AutoReconnect / Heartbeat

## 十一、效能與記憶體要求

- 使用 BufferManager.Rent()
- 使用 Span 解析
- 不使用 LINQ
- 不產生多餘 byte[]
- ASCII 僅在封包邊界轉換
- 高頻路徑禁止字串操作
- 支援 100+ 設備並行
- 不可記憶體洩漏

## 十二、日誌分級

Trace：Hex Dump
Debug：Function 呼叫
Warning：Exception Code
Error：Timeout / CRC / I/O

高頻路徑不得寫 Info。

## 十三、測試要求

- 可注入 Fake IStreamPort
- 不依賴實體 COM
- CRC/LRC 可單元測試
- 不使用 Thread.Sleep

## 十四、子類專案生成規範

子類專案必須：

- 繼承 IModbusClient 或封裝為 Service
- 背景 Polling Task 必須可取消
- 使用 CancellationToken
- 不得阻塞 I/O 執行緒
- 事件不得造成 re-entrancy
- 可獨立版本控制
- 可加入 FakeDevice
- 不得修改父類

## 十五、README 必須包含

1. 三層架構說明
2. 父類責任邊界
3. DI vs 非 DI
4. Initialize 機制說明
5. 子類生成步驟
6. 24/7 設計原則
7. 錯誤模型說明

## 十六、約束矩陣（必須落實）

|項目|父類庫|子類專案|Application|
|||||
|協定解析|✔|✘|✘|
|Retry|✔|✘|✘|
|Health|✔|可讀取|可讀取|
|Polling|✘|✔|✘|
|UI|✘|✘|✔|
|Initialize|✔|呼叫|呼叫|
|AutoReconnect|✘|✘|外層控制|

## 十七、實作順序（強制）

1. Frame Builder
2. CRC/LRC
3. Response Validator
4. Transport
5. Client（含序列化控制 + Initialize）
6. Health
7. DI 註冊
8. README
9. 子類生成 PROMPT

這是一個工控等級 24/7 Modbus 通訊核心庫。
請生成完整專案實作。
