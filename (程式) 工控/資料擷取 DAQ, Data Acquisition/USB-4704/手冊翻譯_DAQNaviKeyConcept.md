---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Scenarios 場景

參考 [[手冊翻譯_Scenarios]]。

# Configure and Run 配置和運行

"Configure and Run" refers to configuration and run, which includes two layers of meanings:
「配置和運行」指的是配置和運行，其中包含兩層含義：

1. Device layer: The parameters of Configure and Run in this layer can be configured either through device manager or through dynamic settings in the program (also called runtime parameter setting).
   設備層：此層中的配置和運作參數可以透過裝置管理員或透過程式中的動態設定（也稱為執行時間參數設定）進行設定。
2. Scenario layer: This layer instructs you to choose or set parameters through prompt message on each page of the scenario.
   場景層：此層透過場景每一頁上的提示訊息，引導您選擇或設定參數。

The function of data acquisition card is to convert physical quantities in nature to digital signals that can be processed by computer, while the applications of it are both wide and complex. Since card device components have differences in design and the sensor has limited response, the operation modes of the cards for different applications varied a lot. Advantech's continuous innovation of DAQ device has made DAQ device different from general stereotyped devices. Differences exist in function or run mode of each device, so we hope you can know function differences between each device through Property. Then you can know functions the device supports, features of the device as well as configurations before running of specific application and scenario and parameter settings during running, which forms the concept of Configure and Run. To be specific, Configure and Run includes two layers of meanings:
資料擷取卡的功能是**將自然界中的物理量轉換為電腦可處理的數位訊號**，其應用範圍廣泛且複雜。由於卡片設備組件的設計各不相同，且感測器的**響應速度有限**，因此不同應用場景下卡片的運作模式差異很大。研華科技在資料擷取設備上的持續創新，使其有別於傳統的刻板設備。每款設備的功能或運作模式都有差異，因此我們希望您能透過「屬性」了解各款設備的功能差異。這樣，您就能了解設備支援的功能、特性，以及在特定應用場景和運行過程中的配置和參數設置，這構成了「配置與運行」的概念。具體來說，「配置與運行」包含兩層意義：

- **Device layer**: The parameters of Configure and Run in this layer can be configured either through device manager or through dynamic settings in the program which is also called runtime parameter setting. We provide a friendly graphical configuration interface. Each device driver is equipped with a graphical configuration dialog box which can help you to deploy a executable setting, so as to release you from setting many complicated parameters through API in application development. About more information about configurations of Advantech data acquisition card, please refer to Device Configuration in this manual.
  設備層：此層中的設定和運作參數可透過裝置管理員或程式中的動態設定（也稱為執行時間參數設定）進行設定。我們提供了一個友善的圖形化配置介面。每個裝置驅動程式都配備了一個圖形化配置對話框，可協助您部署可執行設置，從而避免在應用程式開發中透過 API 設定許多複雜的參數。有關研華數據採集卡配置的更多信息，請參閱本手冊中的“設備配置”部分。

- **Scenario layer**: In this layer, scenarios have classified some known use methods into operation flows and standards, so as to illustrate scenario-related parameters, parameter configurations, applied flows and relative SDK interfaces. DAQNavi Wizard is an Advantech configuration wizard tool with scenario as its core concept. It can be used to carry out configuration and management when you create your own project, instructing you to complete scenario configurations, parameter settings, parameter modifications and checking the descriptions of other parameters step by step in real applications. DAQNavi Wizard supports to automatically detect the available devices in your system according to current scenario and collect device features, as well as provides support items and ranges when you set a parameter. To your convenience, DAQNavi Wizard has been integrated into .NET Device Control and ActiveX Control in DAQNavi SDK, which will greatly facilitate and simply your development of application programs. So far, for DAQNavi Wizard supported scenarios, please refer to Scenarios. For information about DAQNavi Wizard, please refer to DAQNavi Wizard.
  場景層：本層將一些已知的使用方法歸類為操作流程和標準，以說明與場景相關的參數、參數配置、應用流程和對應的 SDK 介面。 DAQNavi Wizard 是研華科技以場景為核心概念的設定精靈工具。它可用於建立專案時進行設定和管理，引導您逐步完成場景配置、參數設定、參數修改，並在實際應用中查看其他參數的說明。 DAQNavi Wizard 支援根據目前場景自動偵測系統中的可用裝置並收集裝置特性，並在您設定參數時提供支援項和範圍。為了方便您的使用，DAQNavi Wizard 已整合到 DAQNavi SDK 的.NET 裝置控制項和 ActiveX 控制項中，這將大大簡化您的應用程式開發。目前，DAQNavi Wizard 支援的場景請參閱「場景」部分。有關 DAQNavi Wizard 的更多信息，請參見“DAQNavi Wizard”部分。

# DAQNavi Wizard/Assistant

Advantech DAQ Device development tool supports many programming languages and tools on Windows and Linux. Before, different development tools should be adopted for different programming languages or even the same programming language; besides the usages and configuration methods and procedures of those tools are totally different. Therefore, developers should spend too much time and energy on studying and being familiar with tools so as to support them. Now, DAQNavi has introduced DAQNavi Wizard/Assistant to unify the configuration methods of Advantech DAQ Device development tool on various platforms and development tool. This unified, friendly corss-platform and cross-IDE configuration interface can significantly improve developers' work efficiency and shorten development cycle.
研華資料擷取設備開發工具支援 Windows 和 Linux 平台上的多種程式語言和工具。以往，不同的程式語言，甚至同一種程式語言，都需要使用不同的開發工具；而且這些工具的使用方法、設定步驟也各不相同。因此，開發人員需要花費大量時間和精力去學習和熟悉這些工具。現在，DAQNavi 推出了 DAQNavi 嚮導/助手，統一了研華資料採集設備開發工具在不同平台和開發工具上的設定方法。這種統一、友善的跨平台、跨 IDE 配置介面能夠顯著提高開發人員的工作效率，縮短開發週期。

DAQNavi Wizard/Assistant, a DAQNavi CSCL-based interface, is an Advantech DAQ Device design-time configuration wizard tool targeted at compatibility with DAQNavi Driver. Through correspondence between device functions and different scenarios, you can use DAQNavi Wizard/Assistant to configure each scenario in a complete and unified manner, no matter using C++ or VB or directly calling DAQNavi CSCL or through DAQNavi Control. The tool can guide you with design-time configurations ranging from functions, scenarios, devices to device features step by step. During configurations, different configuration pages have be divided based on the importance and correlation of each parameter, making you understand the relations and functions of each device feature more easily. In order to help you understand more easily, DAQNavi Wizard/Assistant can display vivid illustrations to explain working principles under different parameter configurations, as well as provide detailed meanings and concept descriptions, notes and related parameter impacts of parameters to be configured. When a parameter is being configured, DAQNavi Wizard/Assistant can automatically read the features of the device based on the device you have selected, then check current input and point out reasonable input range when parameter input is invalid. DAQNavi Wizard/Assistant can release you from previous a lot of repetitive code configuration and configuration verification, rather you can wholly focus on how to realize a specific function. When configuration is completed, the settings will be automatically imported to your project and applied to real device along with the program execution, to finish scenario tasks via an established way.
DAQNavi Wizard/Assistant 是一款基於 DAQNavi CSCL 的介面，是研華資料擷取裝置設計時配置精靈工具，設計為與 DAQNavi Driver 相容。透過裝置功能與不同場景的對應關係，無論使用 C++ 或 VB 編寫程式碼，或是直接呼叫 DAQNavi CSCL 或透過 DAQNavi Control，您都可以使用 DAQNavi Wizard/Assistant 以統一的方式完整配置每個場景。該工具可引導您逐步完成從功能、場景、設備到設備特性等各個方面的設計時配置。在配置過程中，不同的配置頁面根據每個參數的重要性和關聯性進行劃分，使您更容易理解每​​個設備特性之間的關係和功能。為了幫助您更輕鬆地理解，DAQNavi Wizard/Assistant 會顯示生動的圖示來解釋不同參數配置下的工作原理，並提供待配置參數的詳細含義、概念描述、註釋以及相關參數影響。在設定參數時，DAQNavi Wizard/Assistant 可以根據您選擇的裝置自動讀取裝置的特性，然後檢查目前輸入，並在參數輸入無效時指出合理的輸入範圍。 DAQNavi Wizard/Assistant 可以幫助您擺脫以往大量重複的程式碼設定和設定驗證工作，讓您可以專注於如何實現特定功能。配置完成後，設定將自動匯入您的專案中，並隨著程式的執行應用到實際裝置上，從而以既定的方式完成場景任務。

DAQNavi Wizard/DAQNavi Assistant has been integrated in .NET Device Control, ActiveX Device Control and DAQNavi Labview Driver Express Vi. About how to call DAQNavi Wizard/Assistant in Visual Studio, please refer to DAQNavi_User_Interface\\Using DAQNavi\\Creating Your Application and Getting Started\\For C#.
DAQNavi Wizard/DAQNavi Assistant 已整合到 .NET Device Control、ActiveX Device Control 和 DAQNavi Labview Driver Express Vi 中。有關如何在 Visual Studio 中呼叫 DAQNavi Wizard/Assistant 的信息，請參閱 DAQNavi_User_Interface\\Using DAQNavi\\Creating Your Application and Getting Started\\For C#。

# Component Style Class Library 元件式類別庫

The concept of "Component" originates from COM. The design of DAQNavi has fully used this concept but not completely based on the interfaces standard of Component in COM, so the SDK of DAQNavi is called "Component Style Class Library "(The abbreviation is CSCL). Component Style Class library can make you not care about internal implementation details while programming and developing APP and Advantech DAQ card communications, rather you just need to include related head files or DLLs in your APP based on our DLL and introduce parameters according to defined interfaces, to minimize the programming difficulty of data acquisition card.
「組件」的概念源自於 COM 協定。 DAQNavi 的設計充分利用了這個概念，但並非完全基於 COM 協定中的元件介面標準，因此 DAQNavi 的 SDK 被稱為「元件風格類別庫」（簡稱 CSCL）。元件風格類別庫使您在編寫和開發 APP 與研華數據採集卡通訊程式時無需關注內部實現細節，只需基於我們的 DLL 在您的 APP 中包含相關的頭文件或 DLL，並根據定義的接口引入參數即可，從而最大限度地降低數據採集卡的編程難度。

Component Style Class Library (CSCL) has applied component-oriented concept to encapsulate a lot of APIs we provide to users based on their functions, forming many components with focused functions, which compose a component library. Each component has its properties, methods and events, all conforming to the using habit of natural language. The system diagram of CSCL is shown as below:
元件風格類別庫 (CSCL) 應用了元件導向的概念，根據 API 的功能進行封裝，形成許多具有特定功能的元件，這些元件構成了一個元件庫。每個元件都有其屬性、方法和事件，所有這些都符合自然語言的使用習慣。 CSCL 的系統圖如下：

![[AdvantechDAQNaviSdk_AutomationBDaq Namespace.png]]

From system diagram you can see, CSCL has encapsulated the common functions of data acquisition card into 17 components, which include:
從系統圖中可以看出，CSCL 將資料擷取卡的常用功能封裝成了 17 個元件，其中包括：

- **Analog input** functions: `InstantAiCtrl`, `WaveformAiCtrl`.
- **Analog output** functions: `InstantAoCtrl`, `BufferedAoCtrl`.
- **Digital input** and **output** functions: `InstantDiCtrl`, `InstantDoCtrl`.
- **Counter related** functions: `EventCounterCtrl`, `FreqMeterCtrl`, `OneShotCtrl`, `PwMeterCtrl`, `PwModulatorCtrl`, `TimerPulseCtrl`, `UdCounterCtrl`.

Each component has function related Property, Method and Event. In terms of property and method, since properties and specific methods of each component have been classified during encapsulation, it's much easier to use. Parameters in methods should be as simple as possible, which will be convenient for user to understand and use. Speaking of event handling, you should refer to .NET event handling mechanism. You do not have to manage events and what you should do is to pay attention to your own event handling, so development will become easier. Meanwhile, component library of CSCL supports multiple languages. SDK that supports C++ is called C++ CSCL. SDK that supports Java is called Java CSCL. SDK that supports .NET is called .NET Device Control.
每個元件都包含與功能相關的屬性、方法和事件。在屬性和方法方面，由於每個元件的屬性和具體方法在封裝過程中已經過分類，因此使用起來更加便捷。方法中的參數應盡可能簡潔，以便使用者理解和使用。關於事件處理，您可以參考 .NET 事件處理機制。您無需管理事件，只需專注於您自己的事件處理即可，這樣開發工作將更加輕鬆。同時，CSCL 元件庫支援多種語言。支援 C++ 的 SDK 稱為 C++ CSCL，支援 Java 的 SDK 稱為 Java CSCL，支援 .NET 的 SDK 稱為 .NET Device Control。

Multiple language support of the component library has unified use flows and interface names of various language environments and system platforms to offer high portability to your code, so you do not have to know different SDKs for different platforms, which will greatly save your time and improve your work efficiency. Below shows consistency of property, method and event handling of four platforms:
此元件庫支援多種語言，統一了不同語言環境和系統平台的使用流程和介面名稱，為您的程式碼提供高度可移植性，您無需了解不同平台的 SDK，這將大大節省您的時間並提高工作效率。以下展示了四個平台在屬性、方法和事件處理上的一致性：

```cpp title="C++"
#include "BDaqCtrl.h"
using namespace Automation::BDaq;
// This function is used to deal with 'StatusChange' Event.
void BDAQCALL OnDiSnapEvent(void * sender, DiSnapEventArgs * args, void * userParam)
{
   ...   
};
// Event: Register our event handler.
instantDiCtrl->addChangeOfStateHandler(OnDiSnapEvent, NULL); 
// Property: Get Property of PortCount
int32 portCount = instantDiCtrl->getPortCount();
// Method: Start StatusChangeInterrupt
ret = instantDiCtrl->SnapStart();
// Method: Stop StatusChangeInterrupt
ret = instantDiCtrl->SnapStop();
```

```java title="Java" 
import Automation.BDaq.*;
...
// Property: Get Property of PortCount
portCount = instantDiCtrl.PortCount;
// Event: Event handler of Status change event
instantDiCtrl.addChangeOfStateListener(new ChangeOfStateEventListener());
// Method: Start DI snap
ret = instantDiCtrl.SnapStart();
...
// Method: Stop DI snap
ret = instantDiCtrl.SnapStop();
...
// Definition of class ChangeOfStatusEventListener.
class ChangeOfStateEventListener implements DiSnapEventListener
{
    public void DiSnapEvent(Object sender, DiSnapEventArgs args) 
    {
        ....
    }
    ...
}
```

```csharp title="C#"
using Automation.BDaq;
// Event: Event handler of Status change event.
instantDiCtrl.ChangeOfState += new EventHandler<DiSnapEventArgs>(instantDiCtrl_ChangeOfState);
static void instantDiCtrl_ChangeOfState(object sender, DiSnapEventArgs e)
{
    ...
}
// Property: Get Property of PortCount
int portCount = instantDiCtrl.PortCount;
// Method: Start StatusChangeInterrupt
ret = instantDiCtrl.SnapStart();
// Method: Stop StatusChangeInterrupt
ret = instantDiCtrl.SnapStop();
```

```vb title="VB.NET"
Imports Automation.BDaq
...
'Property: Get Property of PortCount
portCount = instantDiCtrl1.PortCount;
'Method: Start DI snap
ret = InstantDiCtrl1.SnapStart()
...
'Method: Stop DI snap
ret = InstantDiCtrl1.SnapStop()
...
'Event: Event handler of Status change event
Private Sub InstantDiCtrl1_DiIntCosPortX(ByVal src As Object, ByVal args As DiSnapEventArgs) Handles InstantDiCtrl1.ChangeOfState'Using 'Invoke' method to update the UI elements
    Try
        Me.Invoke(New UpdateListview(AddressOf UpdateListviewMethod), New Object() {args.SrcNum, args.PortData})
    Catch ex As System.Exception
    End Try
End Sub
```

After compared the above language code fragments of four platform, you can find out:
透過比較上述四個平台的語言程式碼片段，可以發現：

The operations to property `PortCount` are all the same as below:
屬性 `PortCount` 的操作均與下列相同：

```cpp
// Property: Get Property of PortCount
int portCount = instantDiCtrl.PortCount;
```

The operations to method `SnapStart()` are all the same as below:
`SnapStart()` 方法的操作與下面的操作完全相同：

```cpp
// Method: Start StatusChangeInterrupt
ret = instantDiCtrl.SnapStart();
```

The operation method for events in DAQNavi adopt the event delegation, you just need to attach the handler and wait for the event happening, thus helps the users focus on the event processing rather than event management.
DAQNavi 中的事件操作方法採用事件委託，只需附加處理程序並等待事件發生，從而幫助使用者專注於事件處理而不是事件管理。
