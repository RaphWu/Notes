---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

Device in this manual refers in particular to DAQ (Data Acquisition) device, which is also called data acquisition card. According to the installation consequence of driver and data acquisition card, driver installation can be divided into "PreInstall" and "Install". This section will introduce these two concepts in detail. Besides, this section will also introduce you how to represent a data acquisition card in DAQNavi; when multiple cards are used at the same time, how to label and distinguish different cards. On the way to find answers to above questions, you will encounter some important concepts, such as Board ID, Product ID and Terminal Board, etc.
本手冊中的「設備」特別指資料收集（DAQ）設備，也稱為資料擷取卡。根據驅動程式和資料擷取卡的安裝順序，驅動程式的安裝可分為「預先安裝」和「安裝」兩個階段。本節將詳細介紹這兩個概念。此外，本節也將介紹如何在 DAQNavi 中表示資料擷取卡；以及同時使用多張卡片時，如何標記和區分不同的卡片。在尋找上述問題的答案的過程中，您將會遇到一些重要的概念，例如 Board ID、Product ID 和端板等。

# Driver Install

詳見原手冊。

# Add/Remove/Update Device

詳見原手冊。

# Device Product ID

Product ID in driver is used to represent a DAQ Device model and distinguish devices of different models. Each device has an unique Product ID, for example:
驅動程式中的 Product ID 用於表示資料擷取設備型號，並區分不同型號的設備。每個設備都有一個唯一的 Product ID，例如：

Product ID of PCI1710 is BD_PCI1710.
Product ID of PCI1710HG is BD_PCI1710HG.
Product ID of PCI1756 is BD_PCI1756.
Product ID of PCM3718 is BD_PCM3718.
Product ID of USB4718 is BD_USB4718.
Product ID of MIC3718 is BD_MIC3718.

# Device Board ID

Product ID in driver is used to distinguish different devices, while Board ID is used to distinguish different cards of the same model.
驅動程式中的 Product ID 用於區分不同的設備，而 Board ID 用於區分相同型號的不同卡片。

For example, if two DAQ devices (one is PCI-1712, the other is PCI-1710) have been installed in your computer, driver will distinguish two cards by Product ID; if two PCI-1710s have been installed in your computer, driver will distinguish them by Board ID.
例如，如果您的電腦中安裝了兩個 DAQ 裝置（一個是 PCI-1712，另一個是 PCI-1710），驅動程式將透過 Product ID 區分這兩個卡；如果您的電腦中安裝了兩個 PCI-1710，驅動程式將透過板 ID 區分它們。

Settings of Board ID for different DAQ devices can be sorted by the following situations:
不同資料擷取設備的 Board ID 設定可依下列情況排序：

## 1. For PCI Device

For PCI devices, you can set their respective Board ID via the dial switch on the card.
對於 PCI 設備，您可以透過卡片上的撥動開關設定其各自的板 ID。

Generally, Board ID of a PCI device is 0 by default. If you do change it, Board ID of this PCI device will be 0. When you change change a slot or insert a new PCI device, Device Number may change. Under this circumstances, if you just open a device by its device number, the device you open may not exist or you may open other wrong devices, therefore it is strongly recommended you set a non-zero Board ID for different devices of the same model.
通常情況下，PCI 裝置的板級 ID 預設為 0。如果您變更了板級 ID，則該 PCI 裝置的板級 ID 仍為 0。當您更換插槽或插入新的 PCI 裝置時，裝置編號可能會變更。在這種情況下，如果您僅憑設備編號開啟設備，則開啟的設備可能不存在，或者您可能會開啟其他錯誤的設備。因此，強烈建議您為相同型號的不同裝置設定非零的板級 ID。

Remarks: For early PCI devices, there is no dial switch on a data acquisition card to set Board ID. The below figure shows the location of PCI1758UDI Board ID and how to set Board ID.
備註：早期的 PCI 設備資料擷取卡上沒有撥盤開關來設定 Board ID。下圖顯示了 PCI1758UDI Board ID 的位置以及如何設定 Board ID。

![[AdvantechDAQNaviSdk_DeviceBoardId_1.png]]

Notes:

- Board ID is 0 by default.
- For devices that support Board ID, we recommend you to use a non-zero Board ID, to avoid the change of device number due to slot change or other situations.
  對於支援板 ID 的設備，我們建議您使用非零板 ID，以避免因插槽變更或其他情況而導致設備編號變更。
- When you use a non-zero Board ID, Board IDs of several cards of one model should not be the same.
  使用非零板 ID 時，相同型號的多張卡片的板 ID 不應相同。
- New device will only be installed when its Board ID is different from Board IDs of other devices of same type.
  只有當新設備的 Board ID 與其他同類型設備的 Board ID 不同時，才會安裝該新設備。

> Note: It is strongly recommended that you dial the dial switch to set Board ID of your card to a non-zero value when you use the card for the first time, otherwise the card may not be found due to slot change or insertion of other cards which leads to the change of device number.
> 注意：強烈建議您在首次使用該卡時，撥動撥號開關將卡的板 ID 設為非零值，否則由於插槽更改或插入其他卡，導致設備編號更改，可能無法找到該卡。

## 2. For ISA Device

For ISA devices, Base Address can be used to distinguish them, so Base Address can serve as Board ID of ISA devices. The below figure shows the location of PCM3718H Board ID and how to set Board ID.
對於 ISA 裝置，可以使用基底位址來區分它們，因此基底位址可以作為 ISA 元件的板 ID。下圖顯示了 PCM3718H 板 ID 的位置以及如何設定板 ID。

![[AdvantechDAQNaviSdk_DeviceBoardId_2.png]]

## 3. For USB Device

For USB devices, you can set Board ID through program setting and you can save the settings to USB devices permanently. The below figure shows how to set Board ID of USB4761.
對於 USB 設備，您可以透過程式設定來設定 Board ID，並且可以將設定永久儲存到 USB 裝置。下圖展示如何設定 USB4761 的卡牌 ID。

![[AdvantechDAQNaviSdk_DeviceBoardId_3.png]]

For PCI devices, if Board ID is 0, the device will be recognized as a new device when its slot changes and DeviceNumber will change at the same time; if Board ID is not 0, the device will not be recognized as a new device when its slot changes and DeviceNumber will not change at the same time.
對於 PCI 設備，如果板 ID 為 0，則當其插槽發生變化時，該設備將被識別為新設備，並且設備編號也會同時發生變化；如果板 ID 不為 0，則當其插槽發生變化時，該設備不會被識別為新設備，並且設備編號也不會同時發生變化。

For USB devices, Board ID is only used to distinguish different cards of the same model. Board ID of an USB device is set through software, so you will still have no idea about which USB device this configuration page corresponds to when you open Device Manager even if you have set different Board IDs for different USB devices in your system.
對於 USB 設備，Board ID 僅用於區分相同型號的不同卡。 USB 裝置的 Board ID 是透過軟體設定的，因此即使您為系統中不同的 USB 裝置設定了不同的 Board ID，開啟裝置管理員後您仍然無法確定此設定頁面對應的是哪個 USB 裝置。

To users' convenience, Advantech USB device driver provides a "Locate" function. When you open a USB configuration page (Device Configuration), click "Locate device" button on the page, then LED indicators on the corresponding device will start to flicker quickly which may help you to find the correct USB device.
為了方便用戶，研華 USB 設備驅動程式提供了一個「定位」功能。當您開啟 USB 設定頁面（裝置設定）時，點選頁面上的「定位裝置」按鈕，對應裝置上的 LED 指示燈將開始快速閃爍，這有助於您找到正確的 USB 裝置。

The below figure shows "Device Configuration" page of USB-4711A.

![[AdvantechDAQNaviSdk_DeviceBoardId_4.png]]

# Base Address & IRQ

Base address is the I/O port address reserved in the system. Data input and output of ISA device are also from this first address. In other word, this is the first address of a range of consecutive I/O port addresses that ISA device uses.
基底位址是系統中保留的 I/O 連接埠位址。 ISA 設備的資料輸入和輸出也都從這個初始位址開始。換句話說，這是 ISA 設備使用的一系列連續 I/O 連接埠位址中的第一個位址。

IRQ stands for interrupt request, which is used to execute hardware interrupt request action. Some cards do not need interrupt, while some cards may need more than one interrupts. IRQ of ISA device is set via hardware jumper (jumper settings).
IRQ 代表中斷要求，用於執行硬體中斷請求操作。有些卡片不需要中斷，而有些卡片可能需要多個中斷。 ISA 設備的 IRQ 透過硬體跳線（跳線設定）進行設定。

IRQ and interrupt of ISA device should be set manually on hardware, so we strongly recommend you use some unoccupied addresses or interrupts in your system and make them consistent with your hardware settings.
ISA 裝置的 IRQ 和中斷應該在硬體上手動設置，因此我們強烈建議您在系統中使用一些未佔用的位址或中斷，並使其與您的硬體設定保持一致。

# Device Number & Description

## Device Number

When a new data acquisition device is detected, driver will assign a device number to it. Device number is based on the the sequence of Adavantech DAQ device searched by Pnp. Device number is the basis of follow-up operations after the device is open. You can set Board ID of the device to non-zero to fix its device number, which will not be changed due to slot change or the insertion of another device. It should be pointed out that device number is not the unique identifier for the driver to distinguish a device. For devices of different models, Product ID is the unique identifier, while for different devices of the same model, Board ID is the unique identifier.
當偵測到新的資料擷取設備時，驅動程式會為其指派一個設備編號。設備編號是基於 PnP 搜尋到的研華資料擷取設備序列。設備編號是設備開機後後續操作的基礎。您可以將設備的 Board ID 設為非零值來固定其設備編號，這樣即使更換插槽或插入其他設備，設備編號也不會改變。需要注意的是，裝置編號並非驅動程式區分裝置的唯一識別碼。**對於不同型號的設備，Product ID 才是唯一識別碼**；而**對於同一型號的不同設備，Board ID 才是唯一識別碼**。

After device and driver have been successfully installed, please open Device Manager, double-click device name under "Advantech DAQ Devices", then choose "Device Configuration" tab, select "Device" to see the device number of the device.
裝置和驅動程式安裝成功後，請開啟裝置管理員，雙擊「研華資料擷取裝置」下的裝置名稱，然後選擇「裝置設定」標籤，選擇「裝置」以檢視裝置的裝置編號。

In DAQNavi, `SelectedDevice` property of `DeviceCtrlBase` is used to open the device with specified information which is introduced by `DeviceInformation`, including Device Number, Module Index, Device Access Mode (AccessMode) and device Description.
在 DAQNavi 中，`DeviceCtrlBase` 的 `SelectedDevice` 屬性用於開啟設備，並顯示 `DeviceInformation` 引入的指定訊息，包括設備編號、模組索引、設備存取模式 (AccessMode) 和設備描述。

## Device Description

The device will be given an unique name when the driver is installed. The name should be a character string, the length of which should not be exceed 64 characters. Generally, the name consists of device name and Board ID. For example, for PCI-1710, its default device description is "PCI-1710, BID#0". You can modify the character string of device description to define an unique description as its device identifier in order to open the device. We recommend that after you have set an unique identifier for the device, please use device description to open it to avoid incorrect opening of other devices due to the change of device number.
安裝驅動程式時，設備將被賦予一個唯一的名稱。該名稱應為字串，長度不得超過 64 個字元。通常，名稱由裝置名稱和 Board ID 組成。例如，PCI-1710 的預設裝置描述為「PCI-1710，BID#0」。您可以修改裝置描述的字串，將其定義為唯一的裝置標識符，以便開啟裝置。我們建議您在為設備設定唯一識別碼後，使用設備描述來開啟設備，以避免因設備編號變更而錯誤地開啟其他設備。

## DAQNavi References

- DaqCtrlBase
    - SelectedDevice
- DeviceInformation
    - DeviceNumber
    - DeviceMode
    - ModuleIndex
    - Description

# Device Identification

Advantech provides many data acquisition cards of different models. Several cards may be installed in the same system and they may change slot when necessary, so how to identify and distinguish these cards. To solve this problem, Advantech has introduced "**Fix Device Number**" mechanism, which will be combined with Board ID setting (Board ID is set to non-zero) of the hardware, so a Device Number will be generated when the device is firstly inserted into the system, which can be used to distinguish, search and operate the device. This device number will not change due to slot change or the insertion of another card, so you can feel free to use this number to open the device, get device information and configuration, run it supported functions, etc.
研華提供多種不同型號的數據採集卡。在同一系統中可能安裝多張擷取卡，並且必要時可能需要更換插槽，那麼如何識別和區分這些擷取卡呢？為了解決這個問題，研華引入了「**固定設備編號**」機制，該機制結合硬體的 Board ID 設定（Board ID 設定為非零值），在設備首次插入系統時產生一個設備編號，可用於識別、查找和操作設備。此設備編號不會因插槽更換或插入其他採集卡而改變，因此您可以放心使用該編號開啟設備、取得設備資訊和配置、運行其支援的功能等。

When a new data acquisition card is firstly inserted into the system, a device number will be generated for the device, which is based on the sequence of Advantech DAQ Device searched by Pnp.
當新的資料擷取卡首次插入系統時，將為該設備產生一個設備編號，該編號基於 Pnp 搜尋到的研華資料擷取設備序列。

Note: Board ID is 0 by default when it is shipped from factory. It is strongly recommended that you dial the dial switch to set Board ID of your card to a non-zero value when you use the card for the first time, otherwise the card may not be opened due to slot change or insertion of other cards which leads to the change of device number.
注意：出廠時，Board ID 預設為 0。**強烈建議您在首次使用該卡時，撥動撥碼開關將 Board ID 設為非零值**，否則由於更換卡槽或插入其他卡，可能導致裝置編號變更，從而使該卡無法開啟。

# Access Mode

In DAQNavi, no matter which functions of the device you will use, the procedure is to select device by property `SelectedDevice` and then run the function.
在 DAQNavi 中，無論要使用設備的哪個功能，步驟都是透過 `SelectedDevice` 屬性選擇設備，然後執行該功能。

Access right management of the device refers to how to access the device when `SelectedDevice` is selected. Different access modes provide different rights for operation to the device. There are three access modes:
設備的存取權限管理是指在選取 `SelectedDevice` 時如何存取該設備。不同的存取模式賦予設備不同的操作權限。共有三種訪問模式：

**Read Only** (**Read**): If the device is opened via this mode (Read), any properties related to the device can not be changed and its supported functions will be limited, such as write action of DO. Read only (Read) mode allows multiple users to open the device only with read permission.
唯讀（讀取）：如果裝置以唯讀模式打開，則無法變更與裝置相關的任何屬性，且其支援的功能將受到限制，例如 DO 的寫入操作。唯讀（讀取）模式允許多個使用者僅以讀取權限開啟裝置。

**Write**: If the device is opened via this mode, users will have full permission. Each time only one user can open the device with write permission.
寫入：如果透過此模式開啟設備，使用者將擁有完全權限。**每次只能有一個使用者以寫入權限開啟設備。**

In DAQNavi, enumeration of `AccessMode` lists these three access modes: `ModeRead`, `ModeWrite`.
在 DAQNavi 中，`AccessMode` 枚舉列出了以下三種存取模式：`ModeRead`、`ModeWrite`。

## DAQNavi References

- DaqCtrlBase
    - SelectedDevice
- DeviceInformation
    - DeviceNumber
    - DeviceMode
    - ModuleIndex
    - Description

# Module Index

In Advantech DAQ devices, functions of some data acquisition cards can be divided into modules according to hardware or some other factors, so a certain function of the device has multiple modules. You can define to run this function of a module. In order to distinguish these modules of a certain function, you need to number them starting from 0. The modules are numbered as Module0, 1, 2...n, which is called Module Index.
在研華資料擷取設備中，某些資料擷取卡的功能會根據硬體或其他因素被劃分為多個模組，因此設備的某個功能可能對應多個模組。您可以指定執行某個模組的對應功能。為了區分相同功能的不同模組，需要從 0 開始對它們進行編號。模組編號為 Module0、1、2……n，稱為模組索引。

Counter function of PCI-1780U has two modules: ModuleIndex 0 and ModuleIndex 1. ModuleIndex 0 supports Event Counter, Frequency Measurement, Delayed Pulse Generation, Pulse Output with Timer Interrupt, Pulse Width Measurement and PWM Output, while ModuleIndex 1 only supports Pulse Output with Timer Interrupt.
PCI-1780U 的計數器功能有兩個模組：模組索引 0 和模組索引 1。模組索引 0 支援事件計數器、頻率測量、延遲脈衝產生、帶定時器中斷的脈衝輸出、脈衝寬度測量和 PWM 輸出，而模組索引 1 僅支援帶定時器中斷的脈衝輸出。

It should be noted that, if the function of the card has several modules, you should firstly choose module index to run a certain module before you run this function. Here PCI-1780U is taken as an example. Two modules of PCI-1780U both support Pulse Output with Timer Interrupt function, therefore you should select a module in wizard shown as below before you run this function.
需要注意的是，如果卡片的功能包含多個模組，則在運行此功能之前，應先選擇模組索引以運行特定模組。這裡以 PCI-1780U 為例。 PCI-1780U 的兩個模組都支援帶有定時器中斷的脈衝輸出功能，因此在運行此功能之前，應在如下所示的精靈中選擇一個模組。

圖片參考手冊。

Of course, if you do not select module through wizard, you can also set `ModuleIndex` of `DeviceInformation` through Selected Device, then introduce the module you want to run.
當然，如果您沒有透過嚮導選擇模組，也可以透過“`ModuleIndex`”設定“`DeviceInformation`”的“模組索引”，然後引入您想要執行的模組。

## DAQNavi References

- DaqCtrlBase
    - SelectedDevice
- DeviceInformation
    - DeviceNumber
    - DeviceMode
    - ModuleIndex
    - Description

# Terminal Board

Terminal Board is a wiring board that connects to external signal and data acquisition card. It has two types:
接線板是一種連接外部訊號和資料擷取卡的接線板。它有兩種類型：

## 1. Terminal board that does not support any signal processing functions. 不支援任何訊號處理功能的接線板

This is the simplest terminal board, which only connects wires from main board to wiring board which can be easily connected to an external signal. There is no signal processing. This kind of terminal boards are called wiring board.
這是最簡單的接線板，它僅用於連接主機板和接線板，方便連接外部訊號。它不進行信號處理。這種接線板稱為接線板。

## 2. Terminal board that supports signal processing functions. 支援訊號處理功能的接線板

This kind of terminal boards support signal processing functions, such as over current protection, filter, signal amplification, current detection conversion and CJC measure current, etc. Since a terminal board with signal processing functions may affect the features of the whole device, a whole device is made of main board and terminal board. This kind of terminal boards support signal processing without the problem of channel number addressing.
這類接線端子板支援訊號處理功能，例如過電流保護、濾波、訊號放大、電流偵測轉換和 CJC 電流測量等。由於具有訊號處理功能的接線端子板可能會影響整個設備的性能，因此整個設備由主機板和接線端子板組成。這類接線端子板支援的訊號處理不存在通道號碼尋址的問題。

Terminal board with signal amplification function may affect the options of Value Range Type. Terminal board that can convert voltage to current may change the options of Value Range Type. Terminal board with built-in CJC current may change some temperature-related options and may have different setting values.
具有訊號放大功能的接線端子板可能會影響數值範圍類型的選項。能夠將電壓轉換為電流的接線端子板可能會改變數值範圍類型的選項。內建 CJC 電流的接線端子板可能會改變某些與溫度相關的選項，並且可能具有不同的設定值。

PCLD-8115/8710 (With CJC) belongs to terminal board with signal processing functions.
PCLD-8115/8710（含 CJC）屬於具有訊號處理功能的終端板。

# Power Management

Power management of hardware design refers to how power can be effectively allocated to different components in system. Power management here, mainly about OS layer (i.e. power management of software layer), means the OS management of power utilization. For example, it allows you to set after how many seconds the monitor will be turned off in idle status and after how many seconds the system will enter stand-by status.
硬體設計的電源管理是指如何有效地將電源分配給系統中的不同組件。這裡的電源管理主要指作業系統層面的電源管理（即軟體層面的電源管理），指的是作業系統對電源利用的管理。例如，它可以設定顯示器在空閒狀態下經過多少秒後關閉，以及系統在多少秒後進入待機狀態。

Driver of Advantech DAQ device can effectively handle power request. Assumed that your DAQ device is running a certain function, the power management solution of entering sleep mode after the device has finished executing current functions when the system issues a sleep request: if you can ensure the finish time of current function is within the limit, then enter sleep mode after the function is executed; if you are not sure about the time requirement or the finish time of current function is out of the limit, the function will stop. After system is restarted, the device should be reset to status before entering sleep mode and send out `DeviceReconnected` event.
研華資料擷取設備驅動程式能夠有效處理電源請求。假設您的資料擷取裝置正在運作某個功能，當系統發出睡眠要求時，裝置執行完目前功能後進入睡眠模式的電源管理方案如下：如果能夠確保目前功能的完成時間在規定範圍內，則在功能執行完畢後進入睡眠模式；如果無法確定時間需求或目前功能的完成時間超出規定範圍，則停止執行該功能。系統重新啟動後，裝置應重設為進入睡眠模式前的狀態，並傳送 `DeviceReconnected` 事件。

For remote devices, such as USB devices, the device may be disconnected temporarily when the function is running. When the device is reconnected, operation system (OS) will issue an notification, while driver will just wait. For devices that do not support hot swap function, driver will try to query the device periodically. Driver of Advantech DAQ device supports the capability of disconnection detection. After disconnection, you should decide whether it is a temporary disconnection or a mandatory remove. If it is an temporary disconnection and the device has been reconnected, driver will send out `DeviceReconnected` event; if the device has been removed or the reconnection has been failed, driver will send out `DeviceRemoved` event.
對於遠端裝置（例如 USB 裝置），在功能運作時裝置可能會暫時中斷連線。裝置重新連接後，作業系統 (OS) 會發出通知，而驅動程式只需等待即可。對於不支援熱插拔功能的設備，驅動程式會定期嘗試查詢設備狀態。研華資料採集 (DAQ) 裝置的驅動程式支援斷開連接偵測功能。斷開連接後，您需要判斷是暫時斷開還是強制移除。如果是暫時斷開且裝置已重新連接，驅動程式會傳送 `DeviceReconnected` 事件；如果裝置已移除或重新連線失敗，驅動程式會傳送 `DeviceRemoved` 事件。

## DAQNavi References

- DeviceCtrl
    - Reconnected
    - Removed

# Device Reconnected

For devices that support hot-swap function, such as USB devices, the device may be disconnected temporarily when it is running. Driver of Advantech DAQ device has the capability of detecting disconnection. When an USB device is temporarily disconnected and then reconnected to the system, driver will send out `DeviceReconnected` event to inform users that the device has been reconnected to the system.
對於支援熱插拔功能的裝置（例如 USB 裝置），裝置在運作過程中可能會暫時中斷連線。研華資料採集設備的驅動程式能夠偵測到這種斷開連接的情況。當 USB 裝置暫時斷開連線後重新連線至系統時，驅動程式會傳送 `DeviceReconnected` 事件，通知使用者裝置已重新連線至系統。

Whether the device is disconnected or removed is decided by the time interval between disconnected and reconnected. You can set the longest waiting time of auto reconnection after disconnection of remote devices (such as USB devices) , but this property is reserved for now. For more about this, please consult Advantech AE.
設備是斷開連接還是被移除，取決於斷開連接和重新連接之間的時間間隔。您可以設定遠端裝置（例如 USB 裝置）斷開連線後自動重新連線的最長等待時間，但此屬性目前尚未啟用。有關更多信息，請諮詢研華 AE。

## DAQNavi References

- DeviceCtrl
    - Reconnected
    - Removed

# Device Removed

When a hot-swap device or a running device is removed from the system, driver will send out `DeviceRemoved` event to inform users that the device has been removed from the system.
當熱插拔裝置或正在執行的裝置從系統中移除時，驅動程式將發出 `DeviceRemoved` 事件，以通知使用者該裝置已從系統中移除。

## DAQNavi References

- DeviceCtrl
    - Reconnected
    - Removed

# Demo Device

Demo card (Demo Device) is a simulation card with multiple functions, which is a virtual device. This card does not need hardware support.
演示卡（演示設備）是具有多種功能的類比卡，它是一種虛擬設備。此卡無需硬體支援。

Installation, un-installation and configuration of demo device are similar with that of PCI card. Since it does not need hardware support, the concept of pre-install and install is not involved. Demo device has covered almost all acquisition functions, including:
演示設備的安裝、卸載和配置與 PCI 卡類似。由於無需硬體支持，因此不涉及預先安裝和安裝的概念。演示設備涵蓋了幾乎所有採集功能，包括：

Most AI functions and Thermo function. You can set the resolution of the device to 12, 14 or 16-bit in "Configuration Dialog" of the driver. Buffer AI of demo device supports all trigger functions. Any AI channel can be set as trigger source. Burst scan function is also supported.
大多數 AI 功能和熱成像功能均已支援。您可以在驅動程式的「設定對話框」中將裝置的解析度設定為 12 位元、14 位元或 16 位元。演示設備的緩衝 AI 支援所有觸發功能。任何 AI 通道均可設定為觸發來源。此外，也支援突發掃描功能。

Most AO functions. Eight AO channels support user-defined Value Range Type and Scale Table. The resolution of AO channel can be set to 12, 14 or 16. Buffer AO with Trigger function is also supported.
支援大多數 AO 功能。八個 AO 通道支援使用者自訂數值範圍類型和比例表。 AO 通道的解析度可設定為 12、14 或 16。同時支援帶有觸發功能的緩衝 AO。

DIO functions. Two 8-bit DI/DO ports support Static DI, Static DO, DI Interrupt, DI Pattern Match Interrupt and DI Status ChangeInterrupt.
DIO 功能。兩個 8 位元 DI/DO 連接埠支援靜態 DI、靜態 DO、DI 中斷、DI 模式匹配中斷和 DI 狀態變更中斷。

Demo device provides two 32-bit counters, supporting most counter functions, including Event Counting, One Shot, Frequency Measurement, Timer Pulse, Pulse Width Measurement and Pulse Width Modulation. Each counter has a built-in clock.
演示設備提供兩個 32 位元計數器，支援大多數計數器功能，包括事件計數、單次觸發、頻率測量、定時脈衝、脈衝寬度測量和脈衝寬度調變。每個計數器都內建時鐘。

# Fake Device

Fake device is the device still existing in registry information when data acquisition card has been removed. The cause is that the device has been removed or the device does not exist after the driver of a certain card has been installed in the system.
「偽設備」是指資料擷取卡移除後，註冊表資訊中仍然存在的設備。原因可能是設備已被移除，或在系統中安裝了特定採集卡的驅動程式後，該設備已不存在。

In Advantech Navigator, icons for physical device and fake device under "Devices" are different, which are shown as below:
在研華導航器中，「設備」下的實體設備和類比設備的圖示不同，如下所示：

圖片見手冊。

# Device Configuration

Device configuration refers to the configurations of parameters of the device and for device running. With Advantech DAQ device, you can complete complex parameter settings via simple UI operations or configure parameters via the method provided by DAQNavi SDK.
設備配置是指對設備參數進行設置，並用於設備運作。使用研華資料擷取設備，您可以透過簡單的使用者介面操作完成複雜的參數設置，也可以透過 DAQNavi SDK 提供的方法配置參數。

其餘見手冊。
