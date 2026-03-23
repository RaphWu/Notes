---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# The main structure of this set of APIs

![[AdvantechDAQNaviSdk_AutomationBDaq Namespace.png]]

# Inheritance Hierarchy

System.Object
    Automation.BDaq.AiFeatures
    Automation.BDaq.AnalogChannel
        Automation.BDaq.AnalogInputChannel
    Automation.BDaq.AoFeatures
    Automation.BDaq.CjcSetting
    Automation.BDaq.CntrFeatures
        Automation.BDaq.CntrFmFeatures
        Automation.BDaq.EventCountFeatures
        Automation.BDaq.OneShotFeatures
        Automation.BDaq.PwmInFeatures
        Automation.BDaq.PwmOutFeatures
        Automation.BDaq.TimerPulseFeatures
    Automation.BDaq.ConvertClock
    Automation.BDaq.DiCosintPort
    Automation.BDaq.DiintChannel
    Automation.BDaq.DiNoiseFilterChannel
    Automation.BDaq.DioFeatures
        Automation.BDaq.DiFeatures
        Automation.BDaq.DoFeatures
    Automation.BDaq.DiPmintPort
    Automation.BDaq.PortDirection
    Automation.BDaq.ScanChannel
    Automation.BDaq.ScanClock
    Automation.BDaq.ScanPort
    Automation.BDaq.Trigger
    System.ComponentModel.Component
        System.MarshalByRefObject
            Automation.BDaq.DeviceCtrlBase
                Automation.BDaq.AiCtrlBase
                    Automation.BDaq.BufferedAiCtrl
                    Automation.BDaq.InstantAiCtrl
                Automation.BDaq.AoCtrlBase
                    Automation.BDaq.BufferedAoCtrl
                    Automation.BDaq.InstantAoCtrl
                Automation.BDaq.CntrCtrlBase
                    Automation.BDaq.EventCounterCtrl
                    Automation.BDaq.FreqMeterCtrl
                    Automation.BDaq.OneShotCtrl
                    Automation.BDaq.PwMeterCtrl
                    Automation.BDaq.PwModulatorCtrl
                    Automation.BDaq.TimerPulseCtrl
                Automation.BDaq.DioCtrlBase
                    Automation.BDaq.DiCtrlBase
                        Automation.BDaq.BufferedDiCtrl
                        Automation.BDaq.InstantDiCtrl
                    Automation.BDaq.DoCtrlBase
                        Automation.BDaq.BufferedDoCtrl
                        Automation.BDaq.InstantDoCtrl
    System.EventArgs
        Automation.BDaq.BfdAiEventArgs
        Automation.BDaq.BfdAoEventArgs
        Automation.BDaq.BfdDiEventArgs
        Automation.BDaq.BfdDoEventArgs
        Automation.BDaq.CntrEventArgs
        Automation.BDaq.DeviceEventArgs
        Automation.BDaq.DiSnapEventArgs
    System.ValueType
        Automation.BDaq.DeviceInformation
        Automation.BDaq.DeviceTreeNode
        Automation.BDaq.MathInterval
        Automation.BDaq.PulseWidth

# All Classes and Functions

## Classes

### Common Components

| Class               | Description                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| DaqCtrlBase         | Base class of DAQNavi device components, implements the basic functionality common to device components.<br>DAQNavi 設備元件的基類，實現了設備元件通用的基本功能。            |
| DeviceCtrl          | Provides interface to access the underlying hardware information.<br>提供存取底層硬體資訊的介面。                                                                    |
| DeviceEventArgs     | Provides data for the Device events.<br>提供設備事件的數據。                                                                                                     |
| DeviceEventListener | Defines the interface for an object that receives the device event. For JAVA language ONLY.<br>定義接收設備事件的物件的介面。僅適用於 Java 語言。                            |
| DaqException        | Represents errors that occur during DAQNavi component execution.<br>表示 DAQNavi 元件執行期間發生的錯誤。                                                            |
| Conversion          | Provides interface to manage the Buffered AI/AO conversion parameters such as start channel, channel count, etc.<br>提供管理緩衝 AI/AO 轉換參數（例如起始通道、通道數等）的介面。 |
| Record              | Provides interface to manage the data buffer parameters such as section length, section count, etc.<br>提供管理資料緩衝區參數（例如段長度、段數等）的介面。                      |
| ConvertClock        | Provides interface to manage the convert clock.<br>提供管理轉換時脈的介面。                                                                                        |
| ScanChannel         | Provides interface to manage the scan channel.<br>提供管理掃描通道的介面。                                                                                         |
| Trigger             | Provides interface to manage the trigger.<br>提供管理觸發的介面。                                                                                                |
| ScanPort            |                                                                                                                                                        |
| ScanClock           |                                                                                                                                                        |
| Array<T>            | Provides interface to access the collection the DAQNavi components returned. (C++ Only)<br>提供用於存取 DAQNavi 元件傳回的集合的介面。 （僅限 C++）                         |

### Analog Input Components

| Class              | Description                                                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| InstantAiCtrl      | Defines interface to acquire analog data in instant mode.<br>定義了以瞬時模式擷取模擬資料的介面。                                                             |
| WaveformAiCtrl     | Defines interface to acquire analog data in buffered mode.<br>定義了以緩衝模式擷取模擬資料的介面。                                                            |
| AiFeatures         | Represents a collection of read-only properties of analog input function.<br>表示類比輸入功能的唯讀屬性集合。                                               |
| AiChannel          | Represents an individual analog input channel.<br>表示單一類比輸入通道。                                                                               |
| CjcSetting         | Defines interface to manage CJC (Cold-Junction Compensation) function.<br>定義了管理 CJC（冷端補償）功能的介面。                                             |
| BfdAiEventArgs     | Provides data for the Buffered AI events.<br>提供緩衝式 AI 事件的資料。                                                                                |
| BfdAiEventListener |                                                                                                                                             |
| AiCtrlBase         | Base class of analog input components, implements the basic functionality common to analog input components.<br>類比輸入組件的基類，實現了類比輸入組件通用的基本功能。 |

### Analog Output Components

| Class              | Description                                                                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| InstantAoCtrl      | Defines interface to output analog data in instant mode.<br>定義了以瞬時模式輸出模擬資料的介面。                                                                |
| BufferedAoCtrl     | Defines interface to output analog data in buffered mode.<br>定義了以緩衝模式輸出模擬資料的介面。                                                               |
| AoFeatures         | Represents a collection of read-only properties of analog output function.<br>表示類比輸出功能的唯讀屬性集合。                                                |
| AoChannel          | Represents an individual analog output channel.<br>表示單一類比輸出通道。                                                                                |
| BfdAoEventArgs     | Provides data for the Buffered AO events.<br>提供緩衝式 AO 事件的數據。                                                                                  |
| BfdAoEventListener |                                                                                                                                               |
| AoCtrlBase         | Base class of analog output components, implements the basic functionality common to analog output components.<br>類比輸出元件的基類，實現了類比輸出元件通用的基本功能。 |

### Digital Input/Output Components

| Class               | Description                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| InstantDiCtrl       | Defines interface to acquire digital data in instant mode, and monitor the digital input events.<br>定義用於以即時模式擷取數位資料的接口，並監控數位輸入事件。                                              |
| InstantDoCtrl       | Defines interface to output digital data in instant mode.<br>定義用於以即時模式輸出數位資料的介面。                                                                                               |
| BufferedDiCtrl      | Defines interface to acquire digital data in buffered mode.<br>定義用於以緩衝模式擷取數位資料的介面。                                                                                             |
| BufferedDoCtrl      | Defines interface to output digital data in buffered mode.<br>定義用於以緩衝模式輸出數位資料的介面。                                                                                              |
| DioFeatures         | Represents a collection of read-only properties of Digital Input/Output (abbreviation: DI/O) function.<br>表示數字輸入/輸出（簡稱：DI/O）功能的唯讀屬性集合。                                         |
| DioPort             | Represents an individual Digital Input/Output port.<br>表示一個獨立的數位輸入/輸出埠。                                                                                                        |
| DiintChannel        | Represents an individual DI channel which supports the DI Interrupt function.<br>表示一個支援 DI 中斷功能的獨立 DI 通道。                                                                      |
| DiCosintPort        | Represents an individual DI port which supports the DI Status Change Interrupt function.<br>表示一個支援 DI 狀態變更中斷功能的獨立 DI 連接埠。                                                      |
| DiPmintPort         | Represents an individual DI port which supports the DI Pattern Match Interrupt function.<br>表示一個支援 DI 模式匹配中斷功能的獨立 DI 連接埠。                                                      |
| NoiseFilterChannel  | Represents an individual digital channel which supports noise filter function.<br>表示一個支援雜訊濾波功能的獨立數位通道。                                                                         |
| DiSnapEventArgs     | Provides data for the DI snap events.<br>提供 DI 捕捉事件的資料。                                                                                                                        |
| DiSnapEventListener |                                                                                                                                                                                |
| BfdDiEventArgs      |                                                                                                                                                                                |
| BfdDiEventListener  |                                                                                                                                                                                |
| BfdDoEventArgs      |                                                                                                                                                                                |
| BfdDoEventListener  |                                                                                                                                                                                |
| DioCtrlBase         | Base class of Digital Input/Output (abbreviation: DI/O) components, implements the basic functionality common to DI/O components.<br>數位輸入/輸出（簡稱：DI/O）元件的基類，實現了 DI/O 元件共有的基本功能。 |

### Counter Components

| Class                     | Description                                                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| EventCounterCtrl          | Defines interface to count pulse of the input signal.<br>定義用於計數輸入訊號脈衝的介面。                                                      |
| EcChannel                 | Represents an individual counter for Event Counting function.<br>表示事件計數功能的單一計數器。                                                |
| FreqMeterCtrl             | Defines interface to measure the frequency of input signal.<br>定義用於測量輸入訊號頻率的介面。                                                 |
| FmChannel                 | Represents an individual counter for Frequency Measurement function.<br>表示頻率測量功能的單一計數器。                                        |
| OneShotCtrl               | Defines interface to generate delay pulse output.<br>定義用於產生延遲脈衝輸出的介面。                                                           |
| OsChannel                 | Represents an individual counter for Delayed Pulse Generation (one-shot) function.<br>表示延遲脈衝產生（單次觸發）功能的單一計數器。                   |
| TimerPulseCtrl            | Defines interface to Pulse Output with Timer Interrupt function.<br>定義帶定時器中斷的脈衝輸出介面。                                           |
| TmrChannel                | Represents an individual counter for Pulse Output with Timer Interrupt function.<br>表示帶定時器中斷的脈衝輸出功能的單一計數器。                     |
| PwMeterCtrl               | Defines interface to measure the pulse width of input signal.<br>定義用於測量輸入訊號脈衝寬度的介面。                                             |
| PiChannel                 | Represents an individual counter for Pulse Width Measurement function.<br>表示脈衝寬度測量功能的單一計數器。                                     |
| PwModulatorCtrl           | Defines interface to generate pulse width modulation output.<br>定義用於產生脈衝寬度調變輸出的介面。                                             |
| PoChannel                 | Represents an individual counter for Pulse Width Modulation Output function.<br>表示脈衝寬度調變輸出功能的單一計數器。                            |
| UdCounterCtrl             | Defines interface to execute up/down counting function.<br>定義用於執行加/減計數功能的介面。                                                    |
| UdChannel                 | Represents an individual counter for Up-down Counting function.<br>表示加/減計數功能的單一計數器。                                             |
| CntrFeatures              | Represents a collection of read-only properties of the counter function.<br>表示計數器功能的唯讀屬性集合。                                     |
| CntrEventArgs             | Provides data for the counter events, except the up-down counting.<br>提供計數器事件的資料（不包括上下計數）。                                      |
| CntrEventListener         |                                                                                                                                 |
| UdCntrEventArgs           | Provides data for the up-down counting events.<br>提供上下計數事件的數據。                                                                  |
| UdCntrEventListener       |                                                                                                                                 |
| CounterIndexer<T>         | Provides interface to access the counters' features which are a 2-D matrix.<br>提供用於存取計數器特性（二維矩陣）的介面。                            |
| CounterCapabilityIndexer  | Provides interface to access the counters' capabilities.<br>提供用於存取計數器功能（或稱為效能）的介面。                                              |
| CounterClockSourceIndexer | Provides interface to access the counters' clock sources.<br>提供用於存取計數器時鐘來源的介面。                                                  |
| CounterGateSourceIndexer  | Provides interface to access the counters' gate sources.<br>提供用於存取計數器閘電路來源的介面。                                                  |
| CntrCtrlBase              | Base class of counter components, implements the basic functionality common to counter components.<br>計數器組件的基類，實現了計數器組件通用的基本功能。 |

## Structures

| Structure         | Description                                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DeviceInformation | Defines the device information used for selecting device by SelectedDevice property of DaqCtrlBase.<br>定義用於透過 DaqCtrlBase 的 SelectedDevice 屬性選擇裝置的裝置資訊。 |
| DeviceTreeNode    | Provides the information of a device.<br>提供設備資訊。                                                                                                        |
| MathInterval      | Defines a valid mathematic value range.<br>定義有效的數學值範圍。                                                                                                  |
| PulseWidth        | Defines the pulse width.<br>定義脈衝寬度。                                                                                                                     |
| DataMark          | Defines the data mark information.<br>定義資料標記資訊。                                                                                                         |

## Enumerations

### AccessMode

Defines the valid access mode used to open a device.<br>定義用於開啟設備的有效存取模式。

| Member    | Description                                                                                                                                                                                                                                              |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ModeRead  | Open the device with read-only mode. Nothing can be changed under this mode and the available functions are limited. The device can be opened in this mode by multiple users at the same time.<br>以唯讀模式開啟設備。在此模式下，任何設定都無法更改，可用功能也受到限制。多個使用者可以同時以該模式開啟設備。 |
| ModeWrite | Open the device with Read-Write mode. The APP owns full control of the device. Only one APP can open the device in this mode at the same time.<br>以讀寫模式開啟設備。在此模式下，APP 擁有設備的完全控制權。同一時間只能有一個 APP 以此模式開啟裝置。                                                 |
AccessMode 列出了支援的存取模式。一旦 APP 以 ModeRead 模式開啟設備，則所選設備的所有屬性都無法更改，它僅擁有讀取權限，某些可用功能將受到限制，例如 DO Write 或 AO Write，但此模式允許多個 APP 同時以 ModeRead 模式開啟裝置。如果 APP 以 ModeWrite 模式開啟設備，則它擁有該設備的**獨佔控制權**，其他 APP 無法同時以 ModeWrite 模式開啟設備。

### ActiveSignal

Defines the valid signal trigger edge.<br>定義有效的訊號觸發邊緣。

| Member      | Description                                                                                                                                                                            |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ActiveNone  | No effect.<br>無效果。                                                                                                                                                                     |
| RisingEdge  | Rising edge means action will happen when the signal’s level is from low to high.<br>上升沿表示訊號電平由低到高時觸發動作。                                                                               |
| FallingEdge | Falling edge means action will happen when the signal’s level is from high to low.<br>下降沿表示訊號電平由高到低時觸發動作。                                                                              |
| BothEdge    | Both edge, also called rising and falling, means action will happen when the signal's level is both from low to high and from high to low.<br>上升沿和下降沿（也稱為上升和下降沿）表示訊號電平既有由低到高也有由高到低的情況。 |
| HighLevel   | Signal trigger source is at high level.<br>訊號觸發源處於高電位。                                                                                                                                 |
| LowLevel    | Signal trigger source is at low level.<br>訊號觸發源處於低電位。                                                                                                                                  |
`ActiveNone`、`RisingEdge`、`FallingEdge` 和 `BothEdge` 用於設定觸發邊緣。

`LowLevel` 和 `HighLevel` 用於設定門限。

If the trigger type is analog, setting a specific level, the will trigger will happen once the trigger signal’s level higher or lower than the specific level. RisingEdge means action will happen when trigger signal’s level higher than the specific level. FallingEdge means action will happen when trigger signal’s level lower than the specific level. BothEdge means action will happen when trigger signal’s level higher or lower than the specific level.
如果觸發類型為模擬，設定特定電平後，當觸發訊號的電平高於或低於該特定電平時，就會觸發。 `RisingEdge` 表示當觸發訊號的電平高於該特定電平時觸發。 `FallingEdge` 表示當觸發訊號的電平低於該特定電平時觸發。 `BothEdge` 表示當觸發訊號的電平高於或低於該特定電平時都會觸發。

![[AdvantechDAQNaviSdk_ActiveSignal_1.png]]

If the trigger type is digital, RisingEdge means action will happen when the signal’s level is from low to high. FallingEdge means action will happen when the signal’s level is from high to low. BothEdge means action will happen both the signal's level is both from low to high and from high to low.
如果觸發類型為數位，`RisingEdge` 表示當訊號電平由低到高時觸發。 `FallingEdge` 表示當訊號電平由高到低時觸發。 `BothEdge` 表示當訊號電平由低到高和由高到低時都會觸發。

![[AdvantechDAQNaviSdk_ActiveSignal_2.png]]

### AiChannelType

Defines the signal connection type of AI channels supported by the device.<br>定義設備支援的 AI 通道的訊號連接類型。

| Member          | Description                                                                                                                              |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| AllSingleEnded  | All the AI channels are single-ended.<br>所有 AI 通道均為單端。                                                                                   |
| AllDifferential | All the AI channels are differential.<br>所有 AI 通道均為差分。                                                                                   |
| AllSeDiffAdj    | All the AI channels could be set to be either single ended or differential at the same time by programming.<br>所有 AI 通道均可透過編程同時設定為單端或差分。 |
| MixedSeDiffAdj  | Every AI channel could be set to be either single ended or differential respectively.<br>每個 AI 通道可單獨設定為單端或差分。                            |
In general, the ChannelType property value of the selected device will be one of the AiChannelType enumeration, but there are some special cases, for example ChannelType property value of device USB-4702 and USB-4704 is decided by the hardware jumper, it could be AllSingleEnded or AllDifferential, if it is AllSingleEnded, all the channel can only be set to single-ended and can't change to be differential by programming.
通常情況下，所選設備的 `ChannelType` 屬性值將是 `AiChannelType` 枚舉中的一個值，但也有一些特殊情況。例如，USB-4702 和 USB-4704 裝置的 `ChannelType` 屬性值由硬體跳線決定，可以是 `AllSingleEnded` 或 `AllDifferential`。如果設定為 `AllSingleEnded`，則所有頻道只能設定為單端模式，無法透過程式變更為差分模式。

Please note that the case of USB-4702 and USB-4704 is different from the conditions that the ChannelType property value of the selected device equals to AllSeDiffAdj, the latter means all the AI channels can be set to single-ended at the same time and can change all the AI channels to be differential by programming.
請注意，USB-4702 和 USB-4704 的情況與 `ChannelType` 屬性值為 `AllSeDiffAdj` 的情況不同。後者意味著所有 AI 通道可以同時設定為單端模式，並且可以透過編程將所有 AI 通道變更為差分模式。

### AiSignalType

Defines the valid signal connection types for AI channels.<br>定義 AI 通道的有效訊號連接類型。

| Member             | Description                                                                                                                                                                                                                                                                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SingleEnded        | Single-ended connection measures the voltage difference between input signal and ground. Connect an AI channel to the reference ground directly to build an electronic circuit.<br>單端連接測量輸入訊號與接地之間的電壓差。將一個 AI 通道直接連接到參考接地即可建構電子電路。                                                                                                                 |
| Differential       | Differential connection measures the voltage difference between two input signals.Connect two AI channels to build an electronic circuit. Both AI channels can be connected to a valid signal source and either one of them can be used as the reference ground.<br>差分連接測量兩個輸入訊號之間的電壓差。連接兩個 AI 通道即可建構電子電路。兩個 AI 通道都可以連接到有效的訊號源，並且其中任何一個通道都可以用作參考地。 |
| PseudoDifferential | A PseudoDifferential input is much like a differential input in that it provides common-mode voltage rejection (unlike single-ended inputs). However, PseudoDifferential inputs are all referred, but not directly tied, to a common ground.<br>偽差分輸入與差分輸入非常相似，因為它提供共模電壓抑制（與單端輸入不同）。但是，偽差分輸入都參考到公共地，但並非直接連接到公共地。                                   |

### BurnoutRetType

Defines the type of the return value which will be returned under some particular situations.<br>定義在某些特定情況下傳回的傳回值類型。

| Member           | Description                                                        |
| ---------------- | ------------------------------------------------------------------ |
| Current          | Return the value directly without any detecting.<br>直接傳回該值，無需任何檢測。 |
| LastCorrectValue | Return the last valid value.<br>傳回最後一個有效值。                         |
| LowLimit         | Return the minimum value.<br>傳回最小值。                                |
| ParticularValue  | Return a particular value.<br>傳回指定值。                               |
| UpLimit          | Return the maximum value.<br>返回最大值。                                |

### ControlState

Defines the running state of DAQNavi device components.<br>定義 DAQNavi 設備組件的運作狀態。

| Member   | Description                                                                                                                |
| -------- | -------------------------------------------------------------------------------------------------------------------------- |
| Idle     | The component is in idle state.<br>元件處於空閒狀態。                                                                      |
| Ready    | The component is ready for executing the specific function.<br>組件已準備好執行指定函數。                                  |
| Running  | The component is executing the specific function.<br>組件正在執行指定函數。                                                |
| Stopped  | The component has stopped executing the specific function.<br>組件已停止執行指定函數。                                     |
| Uninited | The component has not been initialized and cannot execute the specific function.<br>組件尚未初始化，因此無法執行指定函數。 |

### CounterCapability

Defines the capability of counter.<br>定義計數器的功能。

| Member            | Description                                        |
| ----------------- | -------------------------------------------------- |
| Primary           | Reserved.<br>預留。                                   |
| InstantEventCount | Event Counter.<br>事件計數器。                           |
| OneShot           | Delayed Pulse Generation. <br>延遲脈衝產生。              |
| TimerPulse        | Pulse Output with Timer Interrupt.<br>帶定時器中斷的脈衝輸出。 |
| InstantFreqMeter  | Frequency Measurement.<br>頻率測量。                    |
| InstantPwmIn      | Pulse Width Measurement.<br>脈衝寬度測量。                |
| InstantPwmOut     | Pulse Width Modulation.<br>脈衝寬度調變。                 |
| UpDownCount       | Up-down counting.<br>計數加減 1。                       |

### CounterCascadeGroup

Defines the counter cascade group type.<br>定義計數器級聯組類型。

| Member    | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| GroupNone | No cascade.<br>無級聯。                                                      |
| Cnt0Cnt1  | Counter 0 as first, counter 1 as second.<br>計數器 0 為第一個計數器，計數器 1 為第二個計數器。 |
| Cnt2Cnt3  | Counter 2 as first, counter 3 as second.<br>計數器 2 為第一個計數器，計數器 3 為第二個計數器。 |
| Cnt4Cnt5  | Counter 4 as first, counter 5 as second.<br>計數器 4 為第一個計數器，計數器 5 為第二個計數器。 |
| Cnt6Cnt7  | Counter 6 as first, counter 7 as second.<br>計數器 6 為第一個計數器，計數器 7 為第二個計數器。 |
Counter has the cascade using mode, it means two or more than two counters are interconnected to use, which named cascade, for example one counter which resolution is 16-bits cascade another 16-bits counter will form a 32-bits counter for using.
計數器具有級聯使用模式，這意味著兩個或兩個以上的計數器相互連接使用，這稱為級聯，例如一個分辨率為 16 位元的計數器級聯另一個 16 位元計數器，將形成一個 32 位元計數器使用。

### CounterOperationMode

Defines the counter operation mode.<br>定義計數器的工作模式。

| Member   | Description                                                                                                                    |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| C1780_MA | Mode A level & pulse out, Software-Triggered without Hardware Gating<br>模式 A：電平脈衝輸出，軟體觸發，無硬體門控                                 |
| C1780_MB | Mode B level & pulse out, Software-Triggered with Level Gating, = 8254_M0<br>模式 B：電平脈衝輸出，軟體觸發，帶電平門控，= 8254_M0                  |
| C1780_MC | Mode C level & pulse out, Hardware-triggered strobe level<br>模式 C：電平脈衝輸出，硬體觸發，選通電平                                             |
| C1780_MD | Mode D level & Pulse out, Rate generate with no hardware gating<br>模式 D：電平脈衝輸出，速率產生器，無硬體門控                                     |
| C1780_ME | Mode E level & pulse out, Rate generator with level Gating<br>模式 E：電平脈衝輸出，速率產生器，帶電平門控                                          |
| C1780_MF | Mode F level & pulse out, Non-retriggerable One-shot (Pulse type = 8254_M1)<br>模式 F：電平脈衝輸出，不可重觸發單次脈衝（脈衝類型 = 8254_M1）           |
| C1780_MG | Mode G level & pulse out, Software-triggered delayed pulse one-shot<br>模式 G：電平脈衝輸出，軟體觸發，延遲脈衝單次脈衝                               |
| C1780_MH | Mode H level & pulse out, Software-triggered delayed pulse one-shot with hardware gating<br>模式 H：電平脈衝輸出，軟體觸發，延遲脈衝單次脈衝，帶硬體門控    |
| C1780_MI | Mode I level & pulse out, Hardware-triggered delay pulse strobe<br>模式 I：電平脈衝輸出，硬體觸發，延遲脈衝選通                                     |
| C1780_MJ | Mode J level & pulse out, Variable Duty Cycle Rate Generator with No Hardware Gating<br>模式 J：電平脈衝輸出，可變佔空比速率產生器，無硬體門控           |
| C1780_MK | Mode K level & pulse out, Variable Duty Cycle Rate Generator with Level Gating<br>模式 K：電平脈衝輸出，可變佔空比速率產生器，帶電平門控                 |
| C1780_ML | Mode L level & pulse out, Hardware-Triggered Delayed Pulse One-Shot<br>模式 L：電平脈衝輸出，硬體觸發，延遲脈衝單次脈衝                               |
| C1780_MO | Mode O level & pulse out, Hardware-Triggered Strobe with Edge Disarm<br>模式模式 0：電平及脈衝輸出，硬體觸發頻閃燈，帶邊緣解除警報功能                       |
| C1780_MR | Mode R level & pulse out, Non-Retriggerbale One-Shot with Edge Disarm<br>模式 R：電平及脈衝輸出，不可重觸發單次脈衝頻閃燈，帶邊緣解除警報功能                   |
| C1780_MU | Mode U level & pulse out, Hardware-Triggered Delayed Pulse Strobe with Edge Disarm<br>模式 U：電平及脈衝輸出，硬體觸發延遲脈衝頻閃燈，帶邊緣解除警報功能       |
| C1780_MX | Mode X level & pulse out, Hardware-Triggered Delayed Pulse One-Shot with Edge Disarm<br>模式 X：電平及脈衝輸出，硬體觸發延遲脈衝單次脈衝頻閃燈，帶邊緣解除警報功能 |
| C8254_M0 | 8254 mode 0, interrupt on terminal count.<br>8254 模式 0：終端計數中斷                                                                  |
| C8254_M1 | 8254 mode 1, hardware retriggerable one-shot.<br>8254 模式 1：硬體可重觸發單次脈衝頻閃燈                                                       |
| C8254_M2 | 8254 mode 2, rate generator.<br>8254 模式 2：速率產生器                                                                                |
| C8254_M3 | 8254 mode 3, square save mode.<br>8254 模式 3：方波保存模式                                                                             |
| C8254_M4 | 8254 mode 4, software triggered strobe.<br>8254 模式 4：軟體觸發頻閃燈                                                                   |
| C8254_M5 | 8254 mode 5, hardware triggered strobe.<br>8254 模式 5：硬體觸發頻閃燈                                                                   |

### CounterValueRegister

Defines the counter register name.<br>定義計數器暫存器名稱。

| Member          | Description                                         |
| --------------- | --------------------------------------------------- |
| CntHold         | Hold register of counter.<br>保持計數器暫存器。              |
| CntLoad         | Load register of counter.<br>載入計數器暫存器。              |
| CntOverCompare  | Over compare register of counter.<br>對計數器暫存器進行過比較。  |
| CntPreset       | Preset value register of counter.<br>預設計數器暫存器的值。    |
| CntUnderCompare | Under compare register of counter.<br>對計數器暫存器進行欠比較。 |

### DioPortDir

Defines the DI/O port direction.<br>定義 DI/O 埠方向。

| Member  | Description                                                                 |
| ------- | --------------------------------------------------------------------------- |
| Input   | Eight bits are input.<br>輸入 8 位資料。                                          |
| LinHout | Higher four bits are output, lower four bits are input.<br>輸出高 4 位，輸入低 4 位。 |
| LoutHin | Higher four bits are input, lower four bits are output.<br>輸入高 4 位，輸出低 4 位。 |
| Output  | Eight bits are output.<br>輸出 8 位元數據。                                        |

### DioPortType

Defines the DI/O port type.<br>定義 DI/O 連接埠類型。

| Member        | Description                                                                                                                                                                                                                                                                                                                                     |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Port8255A     | The port number refers to a PPI port A mode DI/O port. This type means the whole port (All the eight channels of this port) should be configured as input or output at the same time.<br>連接埠號碼指的是 PPI A 型 DI/O 連接埠。這種類型表示整個連接埠（該連接埠的所有八個通道）必須同時配置為輸入或輸出。                                                                                        |
| Port8255C     | The port number refers to a PPI port C mode DI/O port. This type means that the eight channels of the port were divided into two groups names Up-Four Bits and Down-Four Bits (each group has four channels), each group can configured as input or output.<br>連接埠號碼指的是 PPI C 型 DI/O 連接埠。這種類型表示該連接埠的八個通道分為兩組，分別稱為上四位和下四位（每組四個通道），每組都可以配置為輸入或輸出。 |
| PortDi        | The port number refers to a DI port.<br>連接埠號碼指的是 DI 連接埠。                                                                                                                                                                                                                                                                                        |
| PortDio       | The port number refers to a DI port and a DO port.<br>連接埠號碼指的是 DI 連接埠和 DO 連接埠。                                                                                                                                                                                                                                                                  |
| PortDo        | The port number refers to a DO port.<br>連接埠號指的是 DO 連接埠。                                                                                                                                                                                                                                                                                         |
| PortIndvdlDio | The port number refers to a port whose each channel can be configured as input or output.<br>連接埠號碼指的是每個通道都可以配置為輸入或輸出的連接埠。                                                                                                                                                                                                                       |
DioPortType is a kind of classification description to the DI/O port from the software aspects in DAQNavi. The following table lists all the ports type According to whether supporting digital input function(DI), supporting digital output(DO) , whether supporting direction configuration:
`DioPortType` 是 DAQNavi 軟體中對 DI/O 連接埠的一種分類描述。下表列出了所有連接埠類型，分類依據為是否支援數位輸入功能 (DI)、是否支援數位輸出功能 (DO) 以及是否支援方向配置：

| Port Type                               | PortDi | PortDo | PortDio | Port8255A | Port8255C | PortIndvdlDio |
| --------------------------------------- | :----: | :----: | :-----: | :-------: | :-------: | :-----------: |
| Wether support DI?                      |   √    |   ×    |    √    |     √     |     √     |       √       |
| Wether support DO?                      |   ×    |   √    |    √    |     √     |     √     |       √       |
| Wether support direction configuration? |   ×    |   ×    |    ×    |     √     |     √     |       √       |

### ErrorCode

Defines the error code used by DAQNavi device components.<br>定義 DAQNavi 設備元件所使用的錯誤代碼。

| Member                               | Value      | Description                                                                                        |
| ------------------------------------ | ---------- | -------------------------------------------------------------------------------------------------- |
| Success                              | 0          | The operation is completed successfully.<br>操作已成功完成。                                               |
| ErrorHandleNotValid                  | 0xE0000000 | The handle is NULL or its type doesn't match the required operation.<br>句柄為空或其類型與所需操作不符。           |
| ErrorParamOutOfRange                 | 0xE0000001 | The parameter value is out of range.<br>參數值超出範圍。                                                   |
| ErrorParamNotSpted                   | 0xE0000002 | The parameter value is not supported.<br>不支援此參數值。                                                  |
| ErrorParamFmtUnexpted                | 0xE0000003 | The parameter value format is not the expected.<br>參數值格式不符合預期。                                     |
| ErrorMemoryNotEnough                 | 0xE0000004 | Not enough memory is available to complete the operation.<br>記憶體不足，無法完成操作。                         |
| ErrorBufferIsNull                    | 0xE0000005 | The data buffer is null.<br>資料緩衝區為空。                                                               |
| ErrorBufferTooSmall                  | 0xE0000006 | The data buffer is too small for the operation.<br>資料緩衝區太小，無法進行此操作。                                |
| ErrorDataLenExceedLimit              | 0xE0000007 | The data length exceeded the limitation.<br>資料長度超出限制。                                              |
| ErrorFuncNotSpted                    | 0xE0000008 | The required function is not supported.<br>不支援所需的函數。                                               |
| ErrorEventNotSpted                   | 0xE0000009 | The required event is not supported.<br>不支援所需的事件。                                                  |
| ErrorPropNotSpted                    | 0xE000000A | The required property is not supported.<br>不支援所需的屬性。                                               |
| ErrorPropReadOnly                    | 0xE000000B | The required property is read-only.<br>所需的屬性為唯讀屬性。                                                 |
| ErrorPropValueConflict               | 0xE000000C | The specified property value conflicts with the current state.<br>指定的屬性值與目前狀態衝突。                   |
| ErrorPropValueOutOfRange             | 0xE000000D | The specified property value is out of range.<br>指定的屬性值超出範圍。                                       |
| ErrorPropValueNotSpted               | 0xE000000E | The specified property value is not supported.<br>不支援指定的屬性值。                                       |
| ErrorPrivilegeNotHeld                | 0xE000000F | The handle hasn't own the privilege of the operation the user wanted.<br>句柄不擁有使用者所需操作的權限。          |
| ErrorPrivilegeNotAvailable           | 0xE0000010 | The required privilege is not available because someone else had own it.<br>所需的權限不可用，因為其他人已經擁有該權限。 |
| ErrorDriverNotFound                  | 0xE0000011 | The driver of specified device was not found.<br>未找到指定設備的驅動程式。                                     |
| ErrorDriverVerMismatch               | 0xE0000012 | The driver version of the specified device mismatched.<br>指定裝置的驅動程式版本不符。                           |
| ErrorDriverCountExceedLimit          | 0xE0000013 | The loaded driver count exceeded the limitation.<br>載入的驅動程式數量超出限制。                                 |
| ErrorDeviceNotOpened                 | 0xE0000014 | The device is not opened.<br>設備未開啟。                                                                |
| ErrorDeviceNotExist                  | 0xE0000015 | The required device does not exist.<br>所需設備不存在。                                                    |
| ErrorDeviceUnrecognized              | 0xE0000016 | The required device is unrecognized by driver.<br>驅動程式無法識別所需設備。                                    |
| ErrorConfigDataLost                  | 0xE0000017 | The configuration data of the specified device is lost or unavailable.<br>指定設備的配置資料遺失或不可用。         |
| ErrorFuncNotInited                   | 0xE0000018 | The function is not initialized and can't be started.<br>函數未初始化，無法啟動。                              |
| ErrorFuncBusy                        | 0xE0000019 | The function is busy.<br>函數正忙。                                                                     |
| ErrorIntrNotAvailable                | 0xE000001A | The interrupt resource is not available.<br>中斷資源不可用。                                               |
| ErrorDmaNotAvailable                 | 0xE000001B | The DMA channel is not available.<br>DMA 通道不可用。                                                    |
| ErrorDeviceIoTimeOut                 | 0xE000001C | Time out when reading/writing the device.<br>讀寫設備逾時。                                               |
| ErrorSignatureNotMatch               | 0xE000001D | The given signature does not match with the device current one.<br>給定的簽章與設備目前簽章不符。                 |
| ErrorFuncConflictWithBfdAi           | 0xE000001E | The function cannot be executed while the buffered AI is running.<br>緩衝 AI 運行時無法執行函數。              |
| ErrorVrgNotAvailableInSeMode         | 0xE000001F | The value range is not available in single-ended mode.<br>單端模式下，該值範圍不可用。                           |
| ErrorVrgNotAvailableIn50ohmMode      | 0xE0000020 | The value range is not available in 50omh input impedance mode.<br>50Ω 輸入阻抗模式下，該值範圍不可用。            |
| ErrorCouplingNotAvailableIn50ohmMode | 0xE0000021 | The coupling type is not available in 50omh input impedance mode.<br>50Ω 輸入阻抗模式下，此耦合類型不可用。         |
| ErrorCouplingNotAvailableInIEPEMode  | 0xE0000022 | The coupling type is not available in IEPE mode.<br>IEPE 模式下，此耦合類型不可用。                             |
| ErrorDeviceCommunicationFailed       | 0xE0000023 | Communication is failed when reading/writing the device.<br>讀寫設備時通訊失敗。                             |
| ErrorFixNumberConflict               | 0xE0000024 | The device's 'fix number' conflicted with other device's.<br>設備的「固定編號」與其他設備的「固定編號」衝突。              |
| ErrorTrigSrcConflict                 | 0xE0000025 | The Trigger source conflicted with other trigger's configuration.<br>觸發來源與其他觸發器的配置衝突。              |
| ErrorPropAllFailed                   | 0xE0000026 | All properties of a property set are failed to be written into device.<br>屬性集中的所有屬性均無法寫入設備。        |
| ErrorEthOneServiceStartedFailed      | 0xE000002A | One of the service is not started properly.<br>某項服務未正常啟動。                                          |
| ErrorEdgeNotFound                    | 0xE000002C | The specified managed edge was not found by hostname.<br>未找到指定的託管邊緣設備（主機名稱未指定）。                    |
| ErrorUndefined                       | 0xE000FFFF | Undefined error.<br>未定義錯誤。                                                                         |
|                                      |            |                                                                                                    |
| WarningIntrNotAvailable              | 0xA0000000 | The interrupt resource is not available.<br>中斷資源不可用。                                               |
| WarningParamOutOfRange               | 0xA0000001 | The parameter is out of the range.<br>參數超出範圍。                                                      |
| WarningPropValueOutOfRange           | 0xA0000002 | The property value is out of range.<br>屬性值超出範圍。                                                    |
| WarningPropValueNotSpted             | 0xA0000003 | The property value is not supported.<br>不支援此屬性值。                                                   |
| WarningPropValueConflict             | 0xA0000004 | The property value conflicts with the current state.<br>屬性值與目前狀態衝突。                                |
| WarningVrgOfGroupNotSame             | 0xA0000005 | The value range type of a group is not same.<br>組的值範圍類型不一致。                                        |
| WarningPropPartialFailed             | 0xA0000006 | Some properties of a property set are failed to be written into device.<br>屬性集中的某些屬性寫入設備失敗。        |
| WarningFuncStopped                   | 0xA0000007 | The operation had been stopped.<br>操作已停止。                                                          |
| WarningFuncTimeout                   | 0xA0000008 | The operation is time-out<br>操作超時。                                                                 |
| WarningCacheOverflow                 | 0xA0000009 | The hardware cache is overflow.<br>硬體緩存溢出。                                                         |
| WarningBurnout                       | 0xA000000A | The channel is burn-out.<br>通道燒毀。                                                                  |
| WarningRecordEnd                     | 0xA000000B | The current data record is end.<br>目前資料記錄已結束。                                                      |
| WarningProfileNotValid               | 0xA000000C | The specified profile is not valid.<br>指定的設定檔無效。                                                   |

### FreqMeasureMethod

Defines the method of frequency measurement.<br>定義頻率測量方法。

| Member                 | Description                                                                                   |
| ---------------------- | --------------------------------------------------------------------------------------------- |
| AutoAdaptive           | Intelligently select the measurement method according to the input signal.<br>根據輸入訊號智能選擇測量方法。 |
| CountingPulseByDevTime | Using system timing clock to calculate the frequency.<br>使用系統時脈計算頻率。                          |
| CountingPulseBySysTime | Using the device timing clock to calculate the frequency.<br>使用設備時鐘計算頻率。                      |
| PeriodInverse          | Calculate the frequency from the period of the signal.<br>根據訊號週期計算頻率。                         |
Frequency measurement is used for measuring the frequency of the input signal using counter. there are four kinds of measuring method,we strongly suggest using the method `CountingPulseByDevTime` and `CountingPulseBySysTime` if the frequency of the input measured signal is relatively high; using `PeriodInverse` if the frequency is relatively low;using `AutoAdaptive` when you are not sure the frequency of the measured signal.
頻率測量用於使用計數器測量輸入訊號的頻率。測量方法有四種：如果輸入測量訊號的頻率較高，我們強烈建議使用 `CountingPulseByDevTime` 和 `CountingPulseBySysTime` 方法；如果頻率較低，則使用 `PeriodInverse` 方法；如果您不確定測量訊號的頻率，則使用 `AutoAdaptive` 方法。

You must set the `CollectionPeriod` once you use the method `CountingPulseBySysTime`, it helps the driver measuring the frequency more accurate.
使用 `CountingPulseBySysTime` 方法時，必須設定 `CollectionPeriod`，這有助於驅動程式更準確地測量頻率。

### MathIntervalType

Defines the interval type in concept of algebra, such as the close set, open set, and some other sets combined of left limit and right limit.<br>定義代數概念中的區間類型，例如閉集、開集以及由左極限和右極限組合而成的其他集合。

| Member              | Description                                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Boundless           | LeftOpenSet \| RightOpenSet                                                                                                                |
| LCBRCB              | LeftClosedBoundary \| RightClosedBoundary,mathematic notation:[min,right].<br>LeftClosedBoundary \| RightClosedBoundary，數學表示法：[min,right]。 |
| LCBROB              | LeftClosedBoundary \| RightOpenBoundary,mathematic notation:[min,right).<br>LeftClosedBoundary \| RightOpenBoundary，數學表示法：[min,right]。     |
| LCBROS              | LeftClosedBoundary \| RightOpenSet,mathematic notation:[min,un-limit).<br>LeftClosedBoundary \| RightOpenSet，數學表示法：[min,un-limit]。         |
| LeftClosedBoundary  | The minimum value is included.<br>包含最小值。                                                                                                   |
| LeftOpenBoundary    | The minimum value is not included.<br>不包含最小值。                                                                                              |
| LeftOpenSet         | No minimum value limitation. Left boundary definition, define the minimum value state, use the bit 2,3.<br>無最小值限制。定義左邊界，定義最小值狀態，使用位元 2、3。  |
| LOBRCB              | LeftOpenBoundary \| RightClosedBoundary,mathematic notation:(min,right].<br>LeftOpenBoundary \|右閉邊界，數學表示法：(min,right)。                     |
| LOBROB              | LeftOpenBoundary \| RightOpenBoundary,mathematic notation:(min,right).<br>左開邊界 \| 右開邊界，數學表示法：(min,right)。                                  |
| LOBROS              | LeftOpenBoundary \| RightOpenSet,mathematic notation:(min,un-limit).<br>左開邊界 \| 右開集，數學表示法：(min,un-limit)。                                  |
| LOSRCB              | LeftOpenSet \| RightClosedBoundary,mathematic notation:(un-limit,max].<br>左開集合 \| 右閉邊界，數學表示法：(un-limit,max)。                               |
| LOSROB              | LeftOpenSet \| RightOpenBoundary,mathematic notation:(un-limit,max).<br>左開集 \| 右開邊界，數學表示法：(un-limit,max)。                                  |
| LOSROS              | LeftOpenSet \| RightOpenSet,mathematic notation:(un-limit,max).<br>左開集 \| 右開集，數學表示法：(un-limit,max)。                                        |
| RightClosedBoundary | The maximum value is included.Right boundary definition,define the maximum value state,use the bit 0,1.<br>包含最大值。右邊界定義，定義最大值狀態，使用位元 0 或 1。 |
| RightOpenBoundary   | The maximum is not included.Right boundary definition,define the maximum value state,use the bit 0,1.<br>不包含最大值。右邊界定義，定義最大值狀態，使用位元 0 或 1。  |
| RightOpenSet        | No maximum value limitation.Right boundary definition,define the maximum value state,use the bit 0,1.<br>無最大值限制。右邊界定義，定義最大值狀態，使用位元 0 或 1。  |

### ModuleType

Defines the the module type the DAQ device supported.<br>定義資料採集設備支援的模組類型。

| Member     | Description                                                                                          |
| ---------- | ---------------------------------------------------------------------------------------------------- |
| DaqGroup   | Group type, generated by the union of several DAQ device.<br>由多個資料擷取設備組合而成的群組類型。                     |
| DaqDevice  | Type of the DAQ device.<br>資料採集設備的類型。                                                                |
| DaqAi      | Type representing the analog input functionality of the DAQ device.<br>表示資料擷取設備模擬輸入功能的類型。            |
| DaqAo      | Type representing the analog output functionality of the DAQ device.<br>表示資料擷取設備類比輸出功能的類型。           |
| DaqDio     | Type representing the digital input/output functionality of the DAQ device.<br>表示資料擷取設備數位輸入/輸出功能的類型。 |
| DaqCounter | Type representing the counting functionality of the DAQ device.<br>表示資料擷取設備計數功能的類型。                  |
| DaqAny     | Unspecified type, representing any possible sub component.<br>未指定的類型，表示任何可能的子元件。                     |

### OutSignalType

Defines the output signal type of counter related functions, such as Pulse Output with Timer Interrupt, Delayed Pulse Generation and so on.<br>定義計數器相關功能的輸出訊號類型，例如帶定時器中斷的脈衝輸出、延遲脈衝產生等。

| Member          | Description                                           |
| --------------- | ----------------------------------------------------- |
| OutSignalNone   | no output or output is 'disabled'<br>無輸出或輸出已“停用”      |
| ChipDefined     | hardware chip defined<br>硬體晶片定義                       |
| NegChipDefined  | hardware chip defined, negative logical<br>硬體晶片定義，負邏輯 |
| PositivePulse   | a low-to-high pulse<br>低到高的脈衝                         |
| NegativePulse   | a high-to-low pulse<br>高到低的脈衝                         |
| ToggledFromLow  | the level toggled from low to high<br>電平由低到高切換        |
| ToggledFromHigh | the level toggled from high to low<br>電平由高到低切換        |

### ProductId

Product ID in driver is used to represent a DAQ Device model and distinguish devices of different models. Each device has an unique Product ID, for example: Product ID of PCI1710 is BD_PCI1710，Product ID of PCI1710HG is BD_PCI1710HG，Product ID of USB4718 is BD_USB4718.<br>驅動程式中的產品 ID 用於表示資料擷取設備型號，並區分不同型號的設備。每個裝置都有一個獨特的產品 ID，例如：PCI1710 的產品 ID 為 BD_PCI1710，PCI1710HG 的產品 ID 為 BD_PCI1710HG，USB4718 的產品 ID 為 BD_USB4718。

| Member         | Description  |
| -------------- | ------------ |
| BD_DEMO        | Demo device  |
| BD_PCI1706MSU  | PCI-1706MSU  |
| BD_PCI1706U    | PCI-1706U    |
| BD_PCI1706UL   | PCI-1706UL   |
| BD_PCI1710     | PCI-1710     |
| BD_PCI1710HG   | PCI-1710HG   |
| BD_PCI1710HGL  | PCI-1710HGL  |
| BD_PCI1710L    | PCI-1710L    |
| BD_PCI1711     | PCI-1711     |
| BD_PCI1711L    | PCI-1711L    |
| BD_PCI1712     | PCI-1712     |
| BD_PCI1712L    | PCI-1712L    |
| BD_PCI1713     | PCI-1713     |
| BD_PCI1713U    | PCI-1713U    |
| BD_PCI1714     | PCI-1714     |
| BD_PCI1714UL   | PCI-1714UL   |
| BD_PCI1715U    | PCI-1715U    |
| BD_PCI1716     | PCI-1716     |
| BD_PCI1716L    | PCI-1716L    |
| BD_PCI1718HDU  | PCI-1718HDU  |
| BD_PCI1718HGU  | PCI-1718HGU  |
| BD_PCI1720     | PCI-1720     |
| BD_PCI1721     | PCI-1721     |
| BD_PCI1723     | PCI-1723     |
| BD_PCI1724     | PCI-1724     |
| BD_PCI1727U    | PCI-1727U    |
| BD_PCI1730     | PCI-1730     |
| BD_PCI1730U    | PCI-1730U    |
| BD_PCI1731     | PCI-1731     |
| BD_PCI1733     | PCI-1733     |
| BD_PCI1734     | PCI-1734     |
| BD_PCI1735U    | PCI-1735U    |
| BD_PCI1736UP   | PCI-1736UP   |
| BD_PCI1737U    | PCI-1737U    |
| BD_PCI1739U    | PCI-1739U    |
| BD_PCI1741U    | PCI-1741U    |
| BD_PCI1742U    | PCI-1742U    |
| BD_PCI1747     | PCI-1747     |
| BD_PCI1750     | PCI-1750     |
| BD_PCI1751     | PCI-1751     |
| BD_PCI1752     | PCI-1752     |
| BD_PCI1752U    | PCI-1752U    |
| BD_PCI1752USO  | PCI-1752USO  |
| BD_PCI1753     | PCI-1753     |
| BD_PCI1754     | PCI-1754     |
| BD_PCI1755     | PCI-1755     |
| BD_PCI1756     | PCI-1756     |
| BD_PCI1757     | PCI-1757     |
| BD_PCI1758UDI  | PCI-1758UDI  |
| BD_PCI1758UDIO | PCI-1758UDIO |
| BD_PCI1758UDO  | PCI-1758UDO  |
| BD_PCI1760     | PCI-1760     |
| BD_PCI1760U    | PCI-1760U    |
| BD_PCI1761     | PCI-1761     |
| BD_PCI1762     | PCI-1762     |
| BD_PCI1763UP   | PCI-1763UP   |
| BD_PCI1780     | PCI-1780     |
| BD_PCI1784     | PCI-1784     |
| BD_PCIE1744    | PCIE-1744    |
| BD_PCIE1752    | PCIE-1752    |
| BD_PCIE1754    | PCIE-1754    |
| BD_PCIE1756    | PCIE-1756    |
| BD_PCL818      | PCL-818      |
| BD_PCL818H     | PCL-818H     |
| BD_PCL818HD    | PCL-818HD    |
| BD_PCL818HG    | PCL-818HG    |
| BD_PCL818L     | PCL-818L     |
| BD_PCM3712     | PCM-3712     |
| BD_PCM3718     | PCM-3718     |
| BD_PCM3718H    | PCM-3718H    |
| BD_PCM3718HG   | PCM-3718HG   |
| BD_PCM3718HO   | PCM-3718HO   |
| BD_PCM3723     | PCM-3723     |
| BD_PCM3724     | PCM-3724     |
| BD_PCM3725     | PCM-3725     |
| BD_PCM3730     | PCM-3730     |
| BD_PCM3730I    | PCM-3730I    |
| BD_PCM3753P    | PCM-3753P    |
| BD_PCM3761I    | PCM-3761I    |
| BD_PCM3780     | PCM-3780     |
| BD_PCM3784     | PCM-3784     |
| BD_PCM3810I    | PCM-3810I    |
| BD_PCM3810I_HG | PCM-3810I_HG |
| BD_PCM3813I    | PCM-3813I    |
| BD_MIC3713     | MIC-3714     |
| BD_MIC3714     | MIC-3713     |
| BD_MIC3716     | MIC-3716     |
| BD_MIC3720     | MIC-3720     |
| BD_MIC3723     | MIC-3723     |
| BD_MIC3747     | MIC-3747     |
| BD_MIC3751     | MIC-3751     |
| BD_MIC3753     | MIC-3753     |
| BD_MIC3755     | MIC-3755     |
| BD_MIC3758DIO  | MIC-3758DIO  |
| BD_MIC3761     | MIC-3761     |
| BD_MIC3780     | MIC-3780     |
| BD_CPCI3756    | CPCI-3756    |
| BD_USB4702     | USB-4702     |
| BD_USB4704     | USB-4704     |
| BD_USB4711     | USB-4711     |
| BD_USB4711A    | USB-4711A    |
| BD_USB4716     | USB-4716     |
| BD_USB4718     | USB-4718     |
| BD_USB4750     | USB-4750     |
| BD_USB4751     | USB-4751     |
| BD_USB4751L    | USB-4751L    |
| BD_USB4761     | USB-4761     |

### SamplingMethod

Defines the the sampling method.<br>定義採樣方法。

| Member          | Description                                         |
| --------------- | --------------------------------------------------- |
| EqualTimeSwitch | Sample during equal time intervals.<br>在相等的時間間隔內取樣。 |
| Simultaneous    | Sample simultaneously.<br>同時取樣。                     |
Some device hardware designing of AI is one channel has one ADC, but others are multi channel using one ADC with multiplexer, by these two kinds of designing, there are two kinds of sampling method, `EqualTimeSwitch` corresponding to the designing of one channel has one ADC and `Simultaneous` corresponding to the other.
有些人工智慧設備的硬體設計是一個通道使用一個 ADC，而有些則是多通道使用一個 ADC 和多路復用器，根據這兩種設計方式，有兩種取樣方法，`EqualTimeSwitch` 等時開關對應於一個通道使用一個 ADC 的設計，`Simultaneous` 同時取樣對應於另一個通道使用一個 ADC 的設計。

`Simultaneous` method uses Convert Clock Source to trigger all the ADC of each channel to start work. The Following group diagram show the meaning of Simultaneous, scan channel count individually equals to 1,2 and N.
`Simultaneous` 方法使用轉換時鐘來源觸發每個通道的所有 ADC 開始工作。下圖展示了同時掃描法的意義，掃描通道數分別等於 1、2 和 N。

![[AdvantechDAQNaviSdk_SamplingMethod_1.png]]

`EqualTimeSwitch` method uses convert clock source to switchover in all channels. The Following group diagram show the meaning of EqualTimeSwitch , scan channel count individually equals to 1,2 and N.
`EqualTimeSwitch` 方法使用轉換時鐘來源在所有通道上進行切換。下圖展示了 EqualTimeSwitch 的意義，掃描通道數分別等於 1、2 和 N。

![[AdvantechDAQNaviSdk_SamplingMethod_2.png]]

In DAQNavi,the property `rate` of `ConvertClock` is used for setting the real sample rate per channel no matter what the sample method the selected device is.
在 DAQNavi 中，無論所選設備的取樣方法為何，`ConvertClock` 的 `rate` 屬性都用於設定每個通道的實際取樣率。

### CountingType

Defines the counting mode of the counter while dealing with clock signal.<br>定義計數器處理時脈訊號時的計數模式。

| Member         | Description                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DownCount      | Counter value decreases on each clock.<br>每次時鐘週期計數器值遞減。                                                                                                 |
| UpCount        | Counter value increases on each clock.<br>每次時鐘週期計數器值遞增。                                                                                                 |
| PulseDirection | Counting direction is determined by two signals, one is clock, and the other is direction signal.<br>計數方向由兩個訊號決定，一個是時脈訊號，另一個是方向訊號。                      |
| TwoPulse       | Counting direction is determined by two signals, one is up-counting signal, and the other is down-counting signal.<br>計數方向由兩個訊號決定，一個是向上計數訊號，另一個是向下計數訊號。 |
| AbPhaseX1      | AB phase,1x rate up/down counting.<br>AB 相，1 倍速向上/向下計數。                                                                                                 |
| AbPhaseX2      | AB phase,2x rate up/down counting.<br>AB 相，2 倍速向上/向下計數。                                                                                                 |
| AbPhaseX4      | AB phase,4x rate up/down counting.<br>AB 相，4 倍速向上/向下計數。                                                                                                 |

### SignalDrop

Defines the connection point type of the signal which may come from a pin on the connector or be generated by a internal logic.<br>定義訊號的連接點類型，該訊號可能來自連接器上的引腳，也可能由內部邏輯產生。

| Member      | Description                                                             |
| ----------- | ----------------------------------------------------------------------- |
| InternalSig | The signal connector is a build-in signal source.<br>訊號連接器是內建訊號源。       |
| OnAmsi      | The signal source is connected to the signal connector.<br>訊號源連接到訊號連接器。 |
| OnConnector | The signal source is on the ASMI connector.<br>訊號源位於 ASMI 連接器上。         |

### TemperatureDegree

Defines of temperature scale types.<br>定義溫度標度類型。

| Member     | Description                                                                                                                                                                                                                     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Celsius    | Temperature scale that registers the freezing point of water as 0 degrees C and the boiling point as 100 degrees C under normal atmospheric pressure.<br>在標準大氣壓力下，水的冰點為攝氏 0 度，沸點為攝氏 100 度的溫度標度。                                 |
| Fahrenheit | Temperature scale that registers the freezing point of water as 32 degrees F and the boiling point as 212 degrees F at one atmosphere of pressure.<br>在標準大氣壓力下，水的冰點為華氏 32 度，沸點為華氏 212 度的溫度標度。                                   |
| Kelvin     | Temperature scale in which zero occurs at absolute zero and each degree equals one kelvin. Water freezes at 273.15 K and boils at 373.15 K.<br>零度為絕對零度的溫度標度，每度等於 1 克耳文。水的冰點為 273.15 K，沸點為 373.15 K。                             |
| Rankine    | Absolute temperature using degrees the same size as those of the Fahrenheit scale, in which the freezing point of water is 491.69 and the boiling point of water is 671.69.<br>使用與華氏度相同刻度的絕對溫度標度，其中水的冰點為 491.69 K，沸點為 671.69 K。 |

### TerminalBoard

Defines the terminal board type.<br>定義接線端子類型。

| Member      | Description                                                                 |
| ----------- | --------------------------------------------------------------------------- |
| PCLD789     | PCLD789 daughter board, belonging to expansion board<br>PCLD789 子板，屬於擴充板。   |
| PCLD8115    | PCLD8115 daughter board, belonging to expansion board<br>PCLD8115 子板，屬於擴充板。 |
| PCLD8710    | PCLD8710 daughter board, belonging to expansion board<br>PCLD8710 子板，屬於擴充板。 |
| WiringBoard | Common terminal board used as signal connector.<br>用作訊號連接器的公共端子板。           |

### TriggerAction

Defines the action which will be executed by device when the trigger condition is met.<br>定義觸發條件滿足時設備將執行的操作。

| Member       | Description                                                                                                                                                                                                                                                                      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ActionNone   | No any action will be taken even if the trigger condition is satisfied.<br>即使觸發條件滿足，也不會執行任何操作。                                                                                                                                                                |
| DelayToStart | Begin to start after the specified time is elapsed if the trigger condition is satisfied. <br>如果觸發條件滿足，則在指定時間過後開始執行。                                                                                                                                       |
| DelayToStop  | Stop execution after the specified time is elapsed if the trigger condition is satisfied.<br>如果觸發條件滿足，則在指定時間過後停止執行。                                                                                                                                        |
| Mark         | Mark the data if the trigger condition is satisfied. The data mark can be retrieved via `DataMark` buffer when calling `GetDatamethod` of `WaveformAiCtrl`.<br>如果觸發條件滿足，則標記資料。可以透過呼叫 `WaveformAiCtrl` 的 `GetData` 方法，從 `DataMark` 緩衝區檢索資料標記。 |

In Buffered AI, driver can use `TriggerAction` to control acquisition process. Likewise, In Buffered AO, driver can also use the `TriggerAction` methods to control output process.
在緩衝式 AI（Buffered AI）中，驅動程式可以使用觸發動作（`TriggerAction`）來控制擷取過程。同樣，在緩衝式 AO（Buffered AO）中，驅動程式也可以使用觸發動作（`TriggerAction`）的方法來控制輸出過程。

In Buffered AI, `DelayToStart` means that driver starts the acquisition process, but not fills data into the data buffer, Once the trigger happens, the driver will wait for samples, the number of which is set by `DelayCount`, to begin to fill in the data buffer;
在緩衝式 AI 中，`DelayToStart` 表示驅動程式啟動採集過程，但不將資料填入資料緩衝區。一旦觸發發生，驅動程式將等待由 `DelayCount` 設定的樣本數，然後開始填入資料緩衝區；

In Buffered AO, `DelayToStart` means driver starts the output process, but not writes data to device , Once the trigger happens, the driver will wait for samples, the number of which is set by `DelayCount`, to write to device from the data buffer.
在緩衝式 AO 中，`DelayToStart` 表示驅動程式啟動輸出過程，但不將資料寫入裝置。一旦觸發發生，驅動程式將等待由 `DelayCount` 設定的取樣數，然後從資料緩衝區將資料寫入裝置。

Following is a diagram for `DelayToStart`.

![[AdvantechDAQNaviSdk_DelayToStart.png]]

In Buffered AI, `DelayToStop` means that driver starts the acquisition process and keeps filling data into the data buffer, Once the trigger happens, the driver will wait for samples, the number of which is set by `DelayCount`, to stop to fill in the data buffer;
在緩衝式 AI 中，`DelayToStop` 表示驅動程式啟動採集過程並持續將資料填入資料緩衝區。一旦觸發發生，驅動程式將等待由 `DelayCount` 設定的樣本數停止填充資料緩衝區；

In Buffered AO, `DelayToStop` means driver starts the output process and writes data to device, Once the trigger happens, the driver will wait for samples, the number of which is set by `DelayCount`, to stop writing to device from the data buffer.
在緩衝式 AO 中，`DelayToStop` 表示驅動程式啟動輸出過程並將資料寫入裝置。一旦觸發發生，驅動程式將等待由 `DelayCount` 設定的取樣數，以停止從資料緩衝區寫入資料到裝置。

Following is a diagram for `DelayToStop`.

![[AdvantechDAQNaviSdk_DelayToStop.png]]

In Buffered AI, `ActionNone` means that the driver starts the acquisition process and keeps filling data into the data buffer. Whether the trigger happens or not, this processing will not be changed.
在緩衝式 AI 中，`ActionNone` 表示驅動程式啟動採集過程並持續向資料緩衝區填入資料。無論觸發事件是否發生，此處理過程都不會改變。

In Buffered AO, `ActionNone` means whether the trigger happens or not, the output processing will not be changed. It also means trigger is disabled.
在緩衝式 AO 中，`ActionNone` 表示無論觸發是否發生，輸出處理都不會改變。它也表示觸發被禁用。

Note: Here, there are two small notice you should pay attention :
注意：這裡有兩點要注意：

1. for Synchronous One Buffered AI example or application with trigger, if trigger action is `DelayToStart` and the trigger is enabled, start the application, if you always don't input trigger signal to the pin, the example or application will be same as dead, because it always waiting for the trigger signal arriving for data acquisition.
   對於帶有觸發器的同步單緩衝 AI 範例或應用程序，如果觸發器操作為 `DelayToStart` 且觸發器已啟用，則啟動應用程式；如果您始終不向引腳輸入觸發訊號，則範例或應用程式將如同死機一般，因為它始終等待觸發訊號到達以進行資料擷取。
2. for Synchronous One Buffered AI example or application with trigger, if trigger action is `DelayToStop` and trigger is enabled, start the application, the example or application will run continuously and not stop, even if the buffer has been filled up. It will be stopped only if the trigger signal coming from the pin.
   對於具有觸發器的同步單緩衝 AI 範例或應用程序，如果觸發器操作為 `DelayToStop` 且觸發器已啟用，則啟動應用程式後，即使緩衝區已滿，範例或應用程式也會持續運行而不會停止。只有當引腳接收到觸發訊號時，它才會停止。

### ValueRange

Defines the value range type the DAQ device supported.<br>定義資料採集設備支援的值範圍類型。

| Member                 | Description                                                        |
| ---------------------- | ------------------------------------------------------------------ |
| V_OMIT                 | Unknown when getting, being ignored when setting.<br>獲取時未知，設定時被忽略。 |
| V_Neg15To15            | +/15 V                                                             |
| V_Neg10To10            | +/10 V                                                             |
| V_Neg5To5              | +/5 V                                                              |
| V_Neg2pt5To2pt5        | +/2.5 V                                                            |
| V_Neg1pt25To1pt25      | +/1.25 V                                                           |
| V_Neg1To1              | +/1 V                                                              |
| V_0To15                | 0~15 V                                                             |
| V_0To10                | 0~10 V                                                             |
| V_0To5                 | 0~5 V                                                              |
| V_0To2pt5              | 0~2.5 V                                                            |
| V_0To1pt25             | 0~1.25 V                                                           |
| V_0To1                 | 0~1 V                                                              |
| mV_Neg625To625         | +/625mV                                                            |
| mV_Neg500To500         | +/500 mV                                                           |
| mV_Neg312pt5To312pt5   | +/312.5 mV                                                         |
| mV_Neg200To200         | +/200 mV                                                           |
| mV_Neg150To150         | +/150 mV                                                           |
| mV_Neg100To100         | +/100 mV                                                           |
| mV_Neg50To50           | +/50 mV                                                            |
| mV_Neg30To30           | +/30 mV                                                            |
| mV_Neg20To20           | +/20 mV                                                            |
| mV_Neg15To15           | +/15 mV                                                            |
| mV_Neg10To10           | +/10 mV                                                            |
| mV_Neg5To5             | +/5 mV                                                             |
| mV_0To625              | 0 ~ 625 mV                                                         |
| mV_0To500              | 0 ~ 500 mV                                                         |
| mV_0To150              | 0 ~ 150 mV                                                         |
| mV_0To100              | 0 ~ 100 mV                                                         |
| mV_0To50               | 0 ~ 50 mV                                                          |
| mV_0To20               | 0 ~ 20 mV                                                          |
| mV_0To15               | 0 ~ 15 mV                                                          |
| mV_0To10               | 0 ~ 10 mV                                                          |
| mA_Neg20To20           | +/20m                                                              |
| mA_0To20               | 0 ~ 20 mA                                                          |
| mA_4To20               | 4 ~ 20 mA                                                          |
| mA_0To24               | 0 ~ 24 mA                                                          |
| V_Neg2To2              | +/2 V                                                              |
| V_Neg4To4              | +/4 V                                                              |
| V_Neg20To20            | +/20 V                                                             |
|                        |                                                                    |
| Pt392_Neg50To150       | Pt392 -50~150 'C                                                   |
| Pt385_Neg200To200      | Pt385 -200~200 'C                                                  |
| Pt385_0To400           | Pt385 0~400 'C                                                     |
| Pt385_Neg50To150       | Pt385 -50~150 'C                                                   |
| Pt385_Neg100To100      | Pt385 -100~100 'C                                                  |
| Pt385_0To100           | Pt385 0~100 'C                                                     |
| Pt385_0To200           | Pt385 0~200 'C                                                     |
| Pt385_0To600           | Pt385 0~600 'C                                                     |
| Pt392_Neg100To100      | Pt392 -100~100 'C                                                  |
| Pt392_0To100           | Pt392 0~100 'C                                                     |
| Pt392_0To200           | Pt392 0~200 'C                                                     |
| Pt392_0To400           | Pt392 0~400 'C                                                     |
| Pt392_0To600           | Pt392 0~600 'C                                                     |
| Pt392_Neg200To200      | Pt392 -200~200 'C                                                  |
| Pt1000_Neg40To160      | Pt1000 -40~160 'C                                                  |
| Balcon500_Neg30To120   | Balcon500 -30~120 'C                                               |
| Ni518_Neg80To100       | Ni518 -80~100 'C                                                   |
| Ni518_0To100           | Ni518 0~100 'C                                                     |
| Ni508_0To100           | Ni508 0~100 'C                                                     |
| Ni508_Neg50To200       | Ni508 -50~200 'C                                                   |
| Thermistor_3K_0To100   | Thermistor 3K 0~100 'C                                             |
| Thermistor_10K_0To100  | Thermistor 10K 0~100 'C                                            |
| Jtype_Neg210To1200C    | T/C J type -210~1200 'C                                            |
| Ktype_Neg270To1372C    | T/C K type -270~1372 'C                                            |
| Ttype_Neg270To400C     | T/C T type -270~400 'C                                             |
| Etype_Neg270To1000C    | T/C E type -270~1000 'C                                            |
| Rtype_Neg50To1768C     | T/C R type -50~1768 'C                                             |
| Stype_Neg50To1768C     | T/C S type -50~1768 'C                                             |
| Btype_40To1820C        | T/C B type 40~1820 'C                                              |
| Jtype_Neg210To870C     | T/C J type -210~870 'C                                             |
| Rtype_Neg0To1768C      | T/C R type 0~1768 'C                                               |
| Stype_Neg0To1768C      | T/C S type  0~1768 'C                                              |
| Ttype_Neg20To135C      | T/C T type -20~135 'C                                              |
|                        |                                                                    |
| UserCustomizedVrgStart | 0xC000 ~ 0xF000 : user customized value range type                 |
| UserCustomizedVrgEnd   | 0xC000 ~ 0xF000 : user customized value range type                 |
|                        |                                                                    |
| V_ExternalRefBipolar   | External reference voltage bipolar.<br>外部參考電壓為雙極性。                 |
| V_ExternalRefUnipolar  | External reference voltage unipolar.<br>外部參考電壓為單極性。                |

### ValueUnit

Defines the unit used by DAQ device.<br>定義資料採集設備使用的單位。

| Member      | Description                                                     |
| ----------- | --------------------------------------------------------------- |
| Ampere      | Ampere, an unit of current.<br>A 安培，電流單位。                       |
| CelsiusUnit | Celsius, an unit of temperature scale.<br>°C 攝氏度，溫度單位。          |
| Kiloampere  | Kiloampere, an unit of current.<br>KA 千安培，電流單位。                 |
| Kilovolt    | Kilovolt, an unit of voltage.<br>KV 千伏，電壓單位。                    |
| Microampere | Microampere, one millionth of an ampere.<br>$\mu$A 微安培，百萬分之一安培。 |
| Microvolt   | Microvolt, one millionth of a volt.<br>$\mu$V 微伏特，百萬分之一伏特。      |
| Milliampere | Milliampere, one thousandth of an ampere.<br>mA 毫安培，千分之一安培。     |
| Millivolt   | Millivolt, one thousandth of a volt.<br>mV 毫伏特，千分之一伏特。          |
| Volt        | Volt, an unit of voltage.<br>V 伏特，電壓單位。                         |

### EventId

Defines the event ID the DAQ device used.<br>定義所用資料採集設備的事件 ID。

| Member                                          | Description                                                                    |
| ----------------------------------------------- | ------------------------------------------------------------------------------ |
| EvtDeviceRemoved                                | The device was removed from the system.<br>設備已從系統中移除。                         |
| EvtDeviceReconnected                            | The device is reconnected to the system.<br>設備已重新連接到系統。                       |
| EvtPropertyChanged                              | Some properties of the device were changed.<br>設備的某些屬性已變更。                     |
|                                                 |                                                                                |
| EvtBufferedAiDataReady                          | AI data is ready for reading.<br>AI 資料已準備好讀取。                                  |
| EvtBufferedAiOverrun                            | AI data buffer is overrun.<br>AI 資料緩衝區溢位。                                     |
| EvtBufferedAiCacheOverflow                      | Hardware data cache is overflow.<br>硬體資料快取溢出。                                  |
| EvtBufferedAiStopped                            | Buffered AI function is stopped.<br>緩衝的 AI 功能已停止。                              |
|                                                 |                                                                                |
| EvtBufferedAoDataTransmitted                    | The data has been transmitted to device.<br>資料已傳輸到設備。                          |
| EvtBufferedAoUnderrun                           | AO data buffer is underrun.<br>AO 資料緩衝區下溢。                                    |
| EvtBufferedAoCacheEmptied                       | Hardware data cache is empted.<br>硬體資料快取已清空。                                  |
| EvtBufferedAoTransStopped                       | AO data transmission is stopped.<br>AO 資料傳輸已停止。                                |
| EvtBufferedAoStopped                            | AO data conversion is stopped.<br>AO 資料轉換已停止。                                 |
|                                                 |                                                                                |
| EvtDiintChannel000 ~ EvtDiintChannel255         | DI channel 0 ~ 255 interrupt event occurred.<br>DI 通道 0~255 發生中斷事件。           |
| EvtDiCosintPort000 ~ EvtDiCosintPort031         | DI port 0 ~ 31 status change event occurred.<br>DI 連接埠 0~31 發生狀態變更事件。          |
| EvtDiPmintPort000 ~ EvtDiPmintPort031           | DI port 0 ~ 31 pattern match event occurred.<br>DI 連接埠 0~31 發生模式比對事件。          |
|                                                 |                                                                                |
| EvtCntOneShot0 ~ EvtCntOneShot7                 | Counter 0 ~ 7 one-shot event occurred.<br>計數器 0~7 發生單次觸發事件。                    |
| EvtCntTimer0 ~ EvtCntTimer7                     | Counter 0 ~ 7 timer event occurred.<br>計數器 0~7 發生定時器事件。                        |
| EvtCntPwmInOverflow0 ~ EvtCntPwmInOverflow7     | Counter 0 ~ 7 is overflow when measure the pulse width.<br>測量脈衝寬度時，計數器 0~7 溢位。 |
| EvtUdIndex0 ~ EvtUdIndex7                       | Counter 0 ~ 7 index event occurred.<br>計數器 0~7 發生索引事件。                         |
| EvtCntPatternMatch0 ~ EvtCntPatternMatch7       | Counter 0 ~ 7 pattern match event occurred.<br>計數器 0~7 發生模式比對事件。               |
| EvtCntCompareTableEnd0 ~ EvtCntCompareTableEnd0 | Counter 0 ~ 7 comparison table is end.<br>計數器 0~7 比較表已結束。                      |
|                                                 |                                                                                |
| EvtBufferedAiBurnOut                            | AI channel is burnout.<br>AI 通道已過載。                                           |
| EvtBufferedAiTimeStampOverrun                   | AI timestamp buffer is overrun.<br>AI 時間戳緩衝區已溢位。                               |
| EvtBufferedAiTimeStampCacheOverflow             | Hardware timestamp cache is overflow.<br>硬體時間戳記快取已溢出。                         |
| EvtBufferedAiMarkOverrun                        | AI data mark buffer is overrun.<br>AI 資料標記緩衝區已溢出。                              |

### CouplingType

Defines the coupling type of the input channels.<br>定義輸入通道的耦合類型。

| Member     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DCCoupling | DC coupling allows both AC and DC signals to pass through a connection. When using DC coupling, no additional capacitor is added to filter the signal.<br>直流耦合允許交流和直流訊號同時通過連接。使用直流耦合時，無需添加額外的電容器來濾波訊號。                                                                                                                                                                                                                                                                                                              |
| ACCoupling | AC coupling consists of using a capacitor to filter out the DC signal component from a signal with both AC and DC components. The capacitor must be in series with the signal. AC coupling is useful because the DC component of a signal acts as a voltage offset, and removing it from the signal can increase the resolution of signal measurements. AC coupling is also known as capacitive coupling.<br>交流耦合則是使用電容器從同時包含交流和直流分量的訊號中濾除直流分量。電容器必須與訊號串聯。交流耦合的優點在於，訊號的直流分量會造成電壓偏移，將其濾除可**提高訊號測量的解析度**。交流耦合也稱為電容耦合。 |

### ImpedanceType

Defines the impedance type of the channels.<br>定義通道的阻抗類型。

| Member   | Description                         |
| -------- | ----------------------------------- |
| Ipd1Momh | The impedance is 1 MΩ.<br>阻抗為 1 MΩ。 |
| Ipd50omh | The impedance is 50 Ω.<br>阻抗為 50 Ω。 |

### IepeType

Defines the excitation current for IEPE (Integrated Electronic Piezoelectric Excitation) sensor.<br>定義 IEPE（集成電子壓電元件）感測器的激磁電流。

| Member   | Description                                    |
| -------- | ---------------------------------------------- |
| IEPENone | No excitation current.<br>無激磁電流。               |
| IEPE4mA  | The excitation current is 4mA.<br>激磁電流為 4mA。   |
| IEPE10mA | The excitation current is 10mA.<br>激磁電流為 10mA。 |

### FilterType

Defines the filter type of the input channels.<br>定義輸入通道的濾波器類型。

| Member     | Description                                                                                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FilterNone | No filter, passes all frequencies.<br>無濾波器，所有頻率均可通過。                                                                                                                 |
| LowPass    | Passes frequencies lower than a certain cutoff frequency and rejects (attenuates) frequencies higher than the cutoff frequency.<br>允許低於特定截止頻率的頻率通過，並抑制（衰減）高於截止頻率的頻率。 |
| HighPass   | Passes frequencies higher than a certain cutoff frequency and rejects (attenuates) frequencies lower than the cutoff frequency.<br>允許高於特定截止頻率的頻率通過，並抑制（衰減）低於截止頻率的頻率。 |
