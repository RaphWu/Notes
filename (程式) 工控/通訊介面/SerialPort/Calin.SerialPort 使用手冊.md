---
aliases:
date: 2025-12-17
update:
author: Raphael Wu
language:
sourceurl:
tags:
  - SerialPort
---

# Calin.SerialPort 使用手冊

## 簡介

`Calin.SerialPort` 是一個 Thread-safe，用於管理 SerialPort 的 .NET 函式庫，提供了高效能、易於擴展的 API，支援多設備管理，並支援多執行緒操作、自動重連、心跳檢測和完整的狀態管理。

## 功能特色

1. **多 SerialPort 管理器** - 可同時管理多個串列埠設備。
2. **Thread-safe 設計** - 所有操作都是執行緒安全的。
3. **支援 ASCII 與 UTF-8 編碼** - 可根據需求選擇編碼方式。
4. **自動斷線重連機制** - 當連線中斷時自動嘗試重新連線。
5. **心跳/保持連線** - 定期發送心跳訊息確保連線正常。
6. **非同步背景監控** - 使用背景執行緒進行自動重連和心跳檢測。
7. **讀取迴圈 + 事件模型** - 持續讀取資料並透過事件通知。
8. **設備狀態機** - 完整的狀態管理（Disconnected / Connecting / Ready / Fault）。
9. **完整的例外處理** - 包含大部分可能的錯誤情況處理。
10. **支援 Autofac 依賴注入** - 方便應用層整合。
11. **設定檔持久化** - 使用 JSON 格式記錄各設備參數值。

## 相依套件

- **RJCP.SerialPortStream** ( [NuGet](https://www.nuget.org/packages/RJCP.SerialPortStream/3.0.4) ) ( [GitHub](https://github.com/jcurl/serialportstream) )
- **Newtonsoft.Json**
- **Autofac**

### 延伸閱讀

- [If you *must* use .NET System.IO.Ports.SerialPort](https://sparxeng.com/blog/software/must-use-net-system-io-ports-serialport)。

## 目標框架

- .NET Framework 4.6.2
- .NET 8.0

## 檔案結構

```markdown
Calin.SerialPort/
├── SerialPortConfig.cs                 // 參數設定類別
├── ISerialPortService.cs               // 單一 Port 介面
├── SerialPortService.cs                // 單一 Port 服務
├── ISerialPortManager.cs               // 多 Port 介面
├── SerialPortManager.cs                // 多 Port 管理器
├── SerialPortState.cs                  // 狀態機定義
├── SerialPortStateChangedEventArgs.cs  // 狀態變更事件參數
├── SerialPortEventArgs.cs              // 事件參數
├── JsonFileHelper.cs                   // JSON 檔案輔助
├── TextEncoding.cs                     // 文字通訊編碼選項
├── SerialPortModule.cs                 // Autofac 模組
└── README.md                           // 說明文件
```

## 類別與介面

### SerialPortConfig

- 用於配置 SerialPort 的類別，包含所有 SerialPort 參數。
- 自動重連、心跳檢測相關設定。
- 提供 `Clone()` 方法用於複製設定。

| 參數                      | 型別         | 說明                                                               | 預設值 | 單位  |
| ------------------------- | ------------ | ------------------------------------------------------------------ | ------ |:-----:|
| `PortName`                | string       | SerialPort 名稱                                                    | "COM1" |       |
| `BaudRate`                | int          | 鮑率                                                               | 9600   |       |
| `DataBits`                | int          | 資料位元數                                                         | 8      |       |
| `Parity`                  | Parity       | 同位位元                                                           | None   |       |
| `StopBits`                | StopBits     | 停止位元                                                           | One    |       |
| `Handshake`               | Handshake    | 交握模式                                                           | None   |       |
| `ReadTimeout`             | int          | 讀取逾時                                                           | 1000   |  ms   |
| `WriteTimeout`            | int          | 寫入逾時                                                           | 1000   |  ms   |
| `RtsEnable`               | bool         | 啟用 RTS 信號                                                      | false  |       |
| `DtrEnable`               | bool         | 啟用 DTR 信號                                                      | false  |       |
| `EnableAutoReconnect`     | bool         | 啟用自動重連                                                       | true   |       |
| `ReconnectInterval`       | int          | 重連間隔                                                           | 5000   |  ms   |
| `EnableHeartbeat`         | bool         | 啟用心跳檢測                                                       | false  |       |
| `HeartbeatInterval`       | int          | 心跳間隔                                                           | 30000  |  ms   |
| `HeartbeatMessage`        | string       | 心跳訊息                                                           | ""     |       |
| `HeartbeatTimeout`        | int          | 心跳逾時                                                           | 10000  |  ms   |
| `TextEncoding`            | TextEncoding | 取得或設定 ASCII 模式的文字編碼                                    | Ascii  |       |
| `AsciiLineTerminator`     | string       | 取得或設定 ASCII 行結尾字串                                        | "\r\n" |       |
| `EnableAsciiLineMode`     | bool         | 取得或設定接收時是否依 AsciiLineTerminator<br>切成一行一行觸發事件 | true   |       |
| `AsciiReceiveBufferLimit` | int          | 取得或設定 ASCII 接收緩衝的最大字元數                              | 8192   | bytes |
| `HeartbeatTimeout`        | int          | 心跳逾時                                                           | 10000  |  ms   |

- 行模式：收到資料後會依 AsciiLineTerminator 分割成多行，每一行都會觸發一次 DataReceived。
- 非行模式：維持「收到一段就觸發一次」(不切行)。

| 方法                                       | 說明                         |
| ---------------------------------------- | -------------------------- |
| `System.Text.Encoding GetTextEncoding()` | 取得對應的 System.Text.Encoding |

### TextEncoding

- 文字通訊編碼選項。

| 列舉      | 說明          |
| ------- | ----------- |
| `Ascii` | 使用 ASCII 編碼 |
| `Utf8`  | 使用 UTF-8 編碼 |

### SerialPortService (實作 ISerialPortService)

- 用於管理單一 SerialPort 的類別。
- Thread-safe 的單一 SerialPort 服務。
- 支援自動重連機制。
- 支援心跳檢測。
- 持續讀取資料（Read Loop）。
- 完整的狀態機管理。
- 配置持久化方法，儲存或載入配置到 JSON 檔案。

| 方法                                 | 說明                                     |
| ---------------------------------- | -------------------------------------- |
| `bool Open()`                      | 開啟 SerialPort                          |
| `void Close()`                     | 關閉 SerialPort                          |
| `bool SendData(string data)`       | 傳送字串資料                                 |
| `bool SendData(byte[] data)`       | 傳送位元組資料                                |
| `bool SendAscii(string data)`      | 傳送 ASCII 字串資料                          |
| `bool SendAsciiLine(string line)`  | 傳送 ASCII 字串資料並加上 `AsciiLineTerminator` |
| `bool SaveConfig(string filePath)` | 儲存目前配置到 JSON 檔案                        |
| `bool LoadConfig(string filePath)` | 從 JSON 檔案載入配置並更新設定                     |

| 事件              | 說明        |
| --------------- | --------- |
| `StateChanged`  | 當狀態改變時觸發  |
| `DataReceived`  | 當接收到資料時觸發 |
| `ErrorOccurred` | 當發生錯誤時觸發  |

### SerialPortManager (實作 ISerialPortManager)

- 用於管理多個 SerialPort 的類別。
- Thread-safe，使用 ConcurrentDictionary。
- 統一管理多個 SerialPortService。
- 支援廣播功能。
- 配置管理方法，儲存或載入所有已註冊的 Port 配置。

| 方法                                                          | 說明                                                         |
| ----------------------------------------------------------- | ---------------------------------------------------------- |
| `bool RegisterPort(SerialPortConfig config)`                | 註冊並開啟一個新的 SerialPort                                       |
| `bool UnregisterPort(string portName`                       | 取消註冊並關閉指定的 SerialPort                                      |
| `SerialPortService GetPort(string portName)`                | 取得指定的 SerialPort 服務                                        |
| `bool SendData(string portName, string data)`               | 傳送字串資料到指定的 SerialPort                                      |
| `int BroadcastData(string data)`                            | 廣播字串資料到所有已就緒的 SerialPort                                   |
| `bool SaveAllConfigs(string filePath)`                      | 儲存所有已註冊的 Port 配置到 JSON 檔案                                  |
| `int LoadAllConfigs(string filePath, bool autoOpen = true)` | 從 JSON 檔案載入並註冊多個 Port 配置                                   |
| `bool SendAscii(string portName, string data)`              | 傳送 ASCII 字串資料到指定的 SerialPort                               |
| `bool SendAsciiLine(string portName, string line)`          | 傳送 ASCII 字串資料到指定的 SerialPort，<br>並加上 `AsciiLineTerminator` |
| `bool SavePortConfig(string portName, string filePath)`     | 儲存特定 Port 的配置到 JSON 檔案                                     |
| `bool LoadPortConfig(string portName, string filePath)`     | 載入特定 Port 的配置並套用                                           |

### SerialPortModule

- 用於 Autofac 依賴注入的模組。
- 簡化應用層整合。

| 屬性                                      | 說明                      |
| --------------------------------------- | ----------------------- |
| `bool RegisterManager`                  | 是否註冊 SerialPortManager  |
| `bool RegisterService`                  | 是否註冊 SerialPortService  |
| `SerialPortConfig DefaultServiceConfig` | SerialPortService 的預設設定 |

### SerialPortState (狀態機)

| 狀態             | 說明  |
| -------------- | --- |
| `Disconnected` | 已斷線 |
| `Connecting`   | 連線中 |
| `Ready`        | 已就緒 |
| `Fault`        | 故障  |

## 流程

1. **初始化**: 使用 `SerialPortConfig` 配置 SerialPort。
2. **建立服務**: 使用 `SerialPortService` 或 `SerialPortManager` 管理 SerialPort。
3. **訂閱事件**: 訂閱 `StateChanged`、`DataReceived` 和 `ErrorOccurred` 事件。
4. **開啟連線**: 呼叫 `Open()` 方法開啟 SerialPort。
5. **傳輸資料**: 使用 `SendData()` 方法傳送資料。
6. **關閉連線**: 呼叫 `Close()` 方法關閉 SerialPort。

## 快速開始

### 1. 基本使用 - 單一 SerialPort

```csharp
using Calin.SerialPort;

// 建立設定
var config = new SerialPortConfig
{
    PortName = "COM1",
    BaudRate = 9600,
    DataBits = 8,
    Parity = Parity.None,
    StopBits = StopBits.One,
    EnableAutoReconnect = true,
    ReconnectInterval = 5000
};

// 建立 SerialPort 服務
using (var service = new SerialPortService(config))
{
    // 訂閱事件
    service.StateChanged += (sender, e) =>
    {
        Console.WriteLine($"狀態變更: {e.OldState} -> {e.NewState}");
    };

    service.DataReceived += (sender, e) =>
    {
        Console.WriteLine($"接收資料: {e.Data}");
    };

    service.ErrorOccurred += (sender, e) =>
    {
        Console.WriteLine($"錯誤: {e.ErrorMessage}");
    };

    // 開啟連線
    if (service.Open())
    {
        Console.WriteLine("連線成功");

        // 傳送資料
        service.SendData("Hello SerialPort!");

        Console.ReadKey();
    }
}
```

### 2. 進階使用 - 多 SerialPort 管理

```csharp
using Calin.SerialPort;

// 建立管理器
using (var manager = new SerialPortManager())
{
    // 訂閱全域事件
    manager.StateChanged += (sender, e) =>
    {
        Console.WriteLine($"[{e.PortName}] 狀態: {e.NewState}");
    };

    manager.DataReceived += (sender, e) =>
    {
        Console.WriteLine($"[{e.PortName}] 資料: {e.Data}");
    };

    // 註冊多個 SerialPort
    manager.RegisterPort(new SerialPortConfig
    {
        PortName = "COM1",
        BaudRate = 9600,
        EnableAutoReconnect = true
    });

    manager.RegisterPort(new SerialPortConfig
    {
        PortName = "COM2",
        BaudRate = 115200,
        EnableAutoReconnect = true
    });

    // 傳送資料到特定 Port
    manager.SendData("COM1", "Hello COM1!");
    
    // 廣播資料到所有 Port
    manager.BroadcastData("Hello All!");

    // 取得所有 Port 的狀態
    var states = manager.GetAllPortStates();
    foreach (var kvp in states)
    {
        Console.WriteLine($"{kvp.Key}: {kvp.Value}");
    }

    Console.ReadKey();
}
```

簡易版：

```csharp
var manager = new SerialPortManager();
manager.RegisterPort(new SerialPortConfig { PortName = "COM1", BaudRate = 9600 });
manager.RegisterPort(new SerialPortConfig { PortName = "COM2", BaudRate = 115200 });
```

ASCII 範例：

```csharp
var config = new SerialPortConfig
{
    PortName = "COM1",
    BaudRate = 9600,
    TextEncoding = TextEncoding.Ascii,
    EnableAsciiLineMode = true,
    AsciiLineTerminator = "\r\n"
};

var mgr = new SerialPortManager();
mgr.RegisterPort(config);

// 送出一行（自動加 \r\n）
mgr.SendAsciiLine("COM1", "READ?");

// 收到時（行模式）每行觸發一次
mgr.DataReceived += (s, e) =>
{
    Console.WriteLine($"[{e.PortName}] ASCII Line: {e.Data}");
};
```

### 3. 心跳檢測 / Keep-alive

```csharp
var config = new SerialPortConfig
{
    PortName = "COM1",
    BaudRate = 9600,
    EnableHeartbeat = true,
    HeartbeatInterval = 30000,      // 30 秒發送一次
    HeartbeatMessage = "PING\r\n",  // 心跳訊息
    HeartbeatTimeout = 10000        // 10 秒內沒收到回應視為異常
};

using (var service = new SerialPortService(config))
{
    service.Open();
    // 自動發送心跳訊息並監控回應
}
```

### 4. 自動斷線重連

```csharp
var config = new SerialPortConfig
{
    EnableAutoReconnect = true,
    ReconnectInterval = 5000  // 5 秒重連一次
};
```

### 5. Thread-safe 設計

- 所有公開方法都使用 `lock` 保護
- 使用 `ConcurrentDictionary` 管理多個 Port
- 安全的事件觸發機制

### 6. 非同步背景監控

- 自動重連監控 (ReconnectMonitor)
- 心跳檢測 (Heartbeat)
- 讀取迴圈 (ReadLoop)
- 使用 `CancellationToken` 安全停止

### 7. Read Loop + Event Model

- 持續讀取資料
- 透過事件通知應用層
- 提供原始 byte[] 和 UTF-8 字串

### 8. 設備狀態機

- 完整的狀態轉換
- 狀態變更事件通知
- 錯誤訊息記錄

### 9. 完整的例外處理

- 所有公開方法都有 try-catch
- 錯誤透過事件通知
- 不會因例外而中斷服務

## Autofac 依賴注入

### 註冊方式 1 - 使用 Module

```csharp
using Autofac;
using Calin.SerialPort;

var builder = new ContainerBuilder();

// 註冊 SerialPort Module
builder.RegisterModule<SerialPortModule>();

var container = builder.Build();

// 解析 Manager
var manager = container.Resolve<ISerialPortManager>();
```

### 註冊方式 2 - 手動註冊

```csharp
var builder = new ContainerBuilder();

// 註冊 Manager
builder.RegisterType<SerialPortManager>()
    .As<ISerialPortManager>()
    .SingleInstance();

// 註冊特定設定的 Service
builder.Register(c => new SerialPortService(new SerialPortConfig
{
    PortName = "COM1",
    BaudRate = 9600
}))
.As<ISerialPortService>()
.SingleInstance();

var container = builder.Build();
```

### 在應用程式中使用

```csharp
public class BarcodeService
{
    private readonly ISerialPortManager _portManager;

    public BarcodeService(ISerialPortManager portManager)
    {
        _portManager = portManager;
    }

    public void Initialize()
    {
        _portManager.RegisterPort(new SerialPortConfig
        {
            PortName = "COM3",
            BaudRate = 9600
        });

        _portManager.DataReceived += OnBarcodeReceived;
    }

    private void OnBarcodeReceived(object sender, SerialPortDataReceivedEventArgs e)
    {
        if (e.PortName == "COM3")
        {
            // 處理條碼資料
            ProcessBarcode(e.Data);
        }
    }
}
```

## 工廠自動化應用範例

```csharp
var builder = new ContainerBuilder();

// 註冊基礎設施
builder.RegisterModule<SerialPortModule>();

// 註冊應用服務
builder.RegisterType<BarcodeService>()
    .AsSelf()
    .SingleInstance();

builder.RegisterType<DisplacementSensorService>()
    .AsSelf()
    .SingleInstance();

builder.RegisterType<ProductionLineController>()
    .AsSelf()
    .SingleInstance();

var container = builder.Build();

// 啟動應用
using (var scope = container.BeginLifetimeScope())
{
    var controller = scope.Resolve<ProductionLineController>();
    controller.Start();

    Console.WriteLine("生產線控制器已啟動");
    Console.WriteLine("按任意鍵停止...");
    Console.ReadKey();

    controller.Stop();
}
```

## 測試建議

1. **單元測試**: 測試狀態轉換邏輯。
2. **整合測試**: 使用虛擬 COM Port 測試。
3. **壓力測試**: 模擬多設備同時運作。
4. **斷線測試**: 測試自動重連機制。
5. **心跳測試**: 驗證心跳逾時處理。

## 效能考量

- 使用 `ConcurrentDictionary` 提升多執行緒效能。
- 讀取迴圈使用小延遲 (10ms) 避免 CPU 過載。
- 事件處理建議非同步執行，避免阻塞讀取迴圈。

## 注意事項

- 請確保安裝了 `RJCP.SerialPortStream` 套件。
- 使用心跳檢測時，請設定適當的 `HeartbeatInterval` 和 `HeartbeatTimeout`。
  心跳訊息需根據實際設備協定設定。
- 使用完畢後請記得呼叫 `Dispose()` 或使用 `using` 語句。
- 事件處理器中不應執行長時間的操作，建議使用非同步處理。
- 自動重連啟用後，手動 Close 仍會嘗試重連。

## 未來擴充方向

1. 支援更多通訊協定 (Modbus, etc.)。
2. 資料緩衝與解析機制。
3. 通訊記錄功能。
4. 效能監控與統計。

---

# SerialPortManager 管理 SerialPort 數量上限

根據 SerialPortManager 的設計，該類別使用了 ConcurrentDictionary<string, SerialPortService> 來管理 SerialPort。以下是對其管理 SerialPort 數量上限的分析：

## 1. 主要限制來源

### 系統資源限制

•	每個 SerialPort 的管理需要分配一定的系統資源（如記憶體、執行緒、檔案描述符等）。
•	Windows 系統對於開啟的 SerialPort 數量並沒有明確的限制，但會受到以下因素影響：
•	可用記憶體：每個 SerialPortService 物件會佔用一定的記憶體。
•	執行緒數量：如果每個 SerialPort 都啟用了背景執行緒，則會受到系統執行緒數量的限制。
•	檔案描述符限制：Windows 對開啟的檔案描述符（File Handles）有上限，通常是幾千到幾萬。

### ConcurrentDictionary 的限制

•	ConcurrentDictionary 本身並沒有明確的容量限制，但它的容量會受到系統記憶體的限制。
•	每個 SerialPortService 物件的鍵值（string）和對應的物件都會佔用記憶體。

### SerialPortService 的開啟限制

•	SerialPortService 的實現可能會限制同時開啟的 SerialPort 數量（例如，底層驅動程式的限制）。

## 2. 估算上限

假設以下條件：

•	每個 SerialPortService 物件佔用約 1 MB 記憶體（包括內部緩衝區、執行緒等）。
•	系統可用記憶體為 2 GB。
•	不考慮其他應用程式的資源佔用。

在這種情況下，理論上可以管理的 SerialPort 數量為：

> 2 GB / 1 MB ≈ 2000 個 SerialPort

## 3. 實際測試建議

由於上限會受到多種因素影響，建議進行以下測試：
1.	逐步增加 SerialPort 數量：使用 RegisterPort(SerialPortConfig) 方法註冊 SerialPort，觀察系統資源的使用情況。
2.	監控系統資源：使用工具（如 Windows 資源監視器）檢查記憶體、CPU 和檔案描述符的使用情況。
3.	檢查穩定性：在高負載下測試應用程式的穩定性。

## 4. 結論

根據設計，SerialPortManager 的理論上限主要取決於系統資源，而非程式內部的邏輯限制。在一般情況下，管理 數百到數千個 SerialPort 是可行的，但具體上限需要根據實際系統資源和 SerialPortService 的實現進行測試。
