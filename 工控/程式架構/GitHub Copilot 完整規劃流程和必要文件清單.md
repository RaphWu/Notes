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

# GitHub Copilot 完整規劃流程和必要文件清單

好的，你想做一個工控專案，並且利用 **GitHub Copilot*- 來生成程式碼。這類專案通常涉及硬體控制、即時性要求、安全性要求，因此除了單純的 prompt，你的規劃流程和必要文件需要很完整。以下我整理一個完整、可落地的流程，並考慮到各種可能的狀況：

## **1. 專案需求分析 (Requirements Analysis)**

**目標**：明確專案的範圍與功能需求，這是 Copilot 生成程式碼的基礎。

必要文件 / 說明：

- **功能需求文件 (FRD, Functional Requirements Document)**
    - 具體列出每個功能，如輸入/輸出控制、通訊協議、數據處理。
    - 例如：
        - “當溫度傳感器 T1 > 80°C，啟動冷卻風扇 F1。”
        - “PLC 需與 SCADA 系統透過 Modbus TCP 通訊，每秒更新一次數據。”
- **非功能需求文件 (NFR, Non-Functional Requirements)**
    - 即時性需求：反應時間 < 100ms
    - 安全性需求：錯誤時安全停機
    - 資料記錄：日誌、歷史數據保存
- **硬體清單與接口說明**
    - 控制器、感測器、致動器型號
    - 通訊協議 (Modbus, CAN, EtherCAT)
    - IO Pin 定義與電氣規格

> 💡 **提示**：Copilot 對硬體細節了解有限，明確規格越清楚，生成程式碼越準確。

## **2. 系統架構設計 (System Architecture Design)**

**目標**：定義軟硬體整體架構，並讓 Copilot 知道程式分層與責任。

必要文件 / 說明：

- **系統架構圖**
    - 控制器、感測器、HMI/SCADA、資料庫的關係
    - 通訊線路、協議層
- **模組設計文件**
    - 定義程式各模組：SensorManager, ActuatorController, Logger, Communication
    - 模組之間的接口（function signature / data structures）
- **時序圖 / 狀態圖**
    - 例如：設備啟動、運作、警報、停機流程
    - Copilot 可依據這些規則生成狀態機程式碼

## **3. 軟體規格定義 (Software Specification)**

**目標**：把系統需求轉成程式化描述，方便 Copilot 生成正確程式碼。

必要文件 / 說明：

- **程式規格文件 (SRS, Software Requirement Specification)**
    - 功能列表 → 函式、類別、模組
    - 輸入/輸出格式（數據結構）
    - 錯誤處理策略
- **程式流程圖 / Pseudocode**
    - Copilot 對明確流程圖生成程式碼效果最佳
- **API 規格**
    - 例如：PLC 通訊函式接口
    - Copilot 可以直接生成呼叫 API 的程式碼

## **4. Prompt 設計 (Prompt Engineering)**

Copilot 的生成能力受 prompt 影響極大。

建議流程：

1. **明確指定語言與平台**
    - 例如 Python + Raspberry Pi GPIO，或 C++ + STM32 HAL
2. **描述功能與約束條件**
    - “請生成 C++ 代碼，用 STM32 HAL 控制 GPIO，當溫度 > 80°C 啟動風扇，每秒讀取一次傳感器”
3. **提供上下文**
    - 給 Copilot 目標模組的範例程式碼、類別定義
4. **小步生成 + 驗證**
    - 先生成單一模組，測試可行，再組合成完整系統

> 💡 **技巧**：Copilot 在有明確函數/類定義和注釋時生成結果最精準。

## **5. 測試計畫 (Testing Plan)**

**目標**：工控程式必須保證可靠性。

必要文件 / 說明：

- **單元測試清單**
    - 每個函式或模組的測試案例
- **集成測試流程**
    - 模組之間資料流、通訊驗證
- **硬體在環測試 (HIL, Hardware-in-the-loop)**
    - 例如模擬傳感器數據，測試控制器行為
- **安全/故障測試**
    - 斷電、傳感器異常、通訊中斷時的系統反應

## **6. 文件與說明準備 (Documentation for Copilot & Team)**

- **README / Docstring**：清楚說明每個函式用途與參數
- **代碼風格指南**：一致化有助於 Copilot 生成整齊程式碼
- **硬體配置文件**：Pin 定義、連線示意圖

## **7. 版本控制與部署規劃 (Version Control & Deployment)**

- **Git 分支策略**：feature branch, dev, main
- **自動化部署**：CI/CD 測試腳本，確保生成程式碼能編譯運行
- **硬體版本對應**：不同控制器或感測器型號可能需要不同代碼

### **小結：必要文件列表**

1. 功能需求文件 (FRD)
2. 非功能需求文件 (NFR)
3. 硬體清單與接口說明
4. 系統架構圖 / 模組設計
5. 狀態圖 / 時序圖
6. 軟體需求規格 (SRS)
7. API / 函式接口規格
8. 測試計畫 (單元、集成、HIL、安全)
9. 代碼風格指南與 README
10. Prompt 設計文檔 (範例程式碼 + 指令)

> ⚠️ 注意：工控系統涉及安全與即時性，Copilot 生成程式碼需人工嚴格審核與測試，不能直接上線。

# 工控專案 Copilot 範例模板（C# 版）

## 1. Copilot Prompt 範例（Human-in-the-Loop）

```text
# 專案目標：
# 這是一個輸送帶控制系統的工控專案，使用 C# 編寫 PLC/控制模組程式碼。
# 所有 Safety Logic 由人工撰寫，Copilot 僅生成非安全核心邏輯。
# 遵守以下規則：
# 1. 所有 I/O 變數使用資料字典定義的命名。
# 2. 禁止生成不帶 Timeout 檢查的迴圈。
# 3. 每個硬體通訊必須包含錯誤檢查。
# 4. 逐模組生成程式碼，每段生成後需人工驗證。

# 任務：
# 生成「溫度感測器讀取模組」，要求：
# - 每秒讀取一次感測器 T1
# - 超過 80°C 時觸發 g_alarmHighTemp
# - 使用資料字典中的變數命名
# - 包含錯誤檢查與 Timeout
# - 模組化函式，易於單元測試
```

## 2. 模組化範例模板（C#）

### **A. Sensor 模組**

```csharp
using System;
using System.Threading;

namespace IndustrialControl
{
    // SensorManager.cs
    // 功能：讀取感測器數值並檢查異常
    public class SensorManager
    {
        // 資料字典變數
        public double g_TempValue { get; private set; }
        public bool g_alarmHighTemp { get; private set; }

        private readonly int timeoutMs = 1000;

        public bool ReadTemperature(Func<double> readSensor)
        {
            bool success = false;
            var startTime = DateTime.Now;

            try
            {
                while ((DateTime.Now - startTime).TotalMilliseconds < timeoutMs)
                {
                    g_TempValue = readSensor();
                    if (g_TempValue > 80.0)
                        g_alarmHighTemp = true;
                    else
                        g_alarmHighTemp = false;

                    success = true;
                    break;
                }
            }
            catch (Exception ex)
            {
                // 硬體錯誤處理
                Console.WriteLine($"Sensor read error: {ex.Message}");
                success = false;
            }

            return success;
        }
    }
}
```

### **B. Actuator 模組**

```csharp
using System;

namespace IndustrialControl
{
    // MotorController.cs
    // 功能：控制馬達啟停，包含安全聯鎖
    public class MotorController
    {
        public bool g_MotorRunning { get; private set; }

        public void ExecuteControl(bool di_StartCmd, bool di_StopCmd, bool di_SafetyOk)
        {
            if (!di_SafetyOk)
            {
                g_MotorRunning = false; // 強制停機
                return;
            }

            if (di_StartCmd)
                g_MotorRunning = true;
            else if (di_StopCmd)
                g_MotorRunning = false;
        }
    }
}
```

### **C. 通訊模組**

```csharp
using System;
using System.Net.Sockets;
using System.Text;

namespace IndustrialControl
{
    // ModbusComm.cs
    // 功能：與 SCADA 系統通訊，包含錯誤檢查與重試
    public class ModbusComm
    {
        public bool g_DataReceived { get; private set; }
        public bool g_CommError { get; private set; }

        public void SendRequest(string ip, int port, byte[] request)
        {
            try
            {
                using (TcpClient client = new TcpClient(ip, port))
                {
                    NetworkStream stream = client.GetStream();
                    stream.Write(request, 0, request.Length);

                    byte[] buffer = new byte[256];
                    int bytes = stream.Read(buffer, 0, buffer.Length);

                    g_DataReceived = bytes > 0;
                    g_CommError = !g_DataReceived;
                }
            }
            catch (Exception ex)
            {
                g_CommError = true;
                Console.WriteLine($"Comm error: {ex.Message}");
            }
        }
    }
}
```

### **D. 主控制迴圈範例**

```csharp
using System;
using System.Threading;

namespace IndustrialControl
{
    class Program
    {
        static void Main(string[] args)
        {
            var sensor = new SensorManager();
            var motor = new MotorController();
            var comm = new ModbusComm();

            while (true)
            {
                // 讀取感測器
                sensor.ReadTemperature(() =>
                {
                    // TODO: 讀取硬體感測器數值
                    return 75.0; // 範例數值
                });

                // 控制馬達
                motor.ExecuteControl(di_StartCmd: true, di_StopCmd: false, di_SafetyOk: true);

                // 與 SCADA 系統通訊
                comm.SendRequest("192.168.0.10", 502, Encoding.ASCII.GetBytes("request"));

                // 每秒迴圈
                Thread.Sleep(1000);
            }
        }
    }
}
```

## 3. Copilot 使用建議流程（C#）

1. **建立資料字典與 HAL**：定義所有 I/O、變數、寄存器
2. **逐模組生成程式碼**：Sensor → Actuator → Comm → 主迴圈
3. **人工驗證**：單元測試、模擬器/HIL 測試
4. **安全審核**：Safety Logic、報警處理必須人工撰寫
5. **版本控制**：Feature branch → Merge → Master/Main

## 4. 專案文件建議

- `README.md`：專案背景、模組用途、命名規則
- `.github/copilot-instructions.md`：生成規則與禁令
- `DataDictionary.xlsx`：I/O 與變數命名、範圍、型態
- `FSM_Diagram.drawio`：狀態機圖
- `SRS.docx`：軟體需求規格
- `TestPlan.xlsx`：單元、集成、HIL 測試計畫
