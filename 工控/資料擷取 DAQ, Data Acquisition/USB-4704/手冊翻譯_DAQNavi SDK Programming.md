---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

DAQNavi SDK has a series of application programming interfaces, which is a library of classes, methods, and properties for creating applications for your device. For information about how to program with DAQNavi SDK, please refer to DAQNavi Interface Manual Programming with CSCL.
DAQNavi SDK 包含一系列應用程式介面 (API)，它是一個包含類別、方法和屬性的程式庫，用於建立適用於您裝置的應用程式。有關如何使用 DAQNavi SDK 進行程式設計的信息，請參閱《DAQNavi 介面手冊：使用 CSCL 進行程式設計》。

# Property List

Device properties are stored in the system and retrieved by the driver when it is loaded, and then the device will be programmed according to these loaded values. Different devices have different properties because of the different hardware they use. Property in DAQNavi can be divided into two category, Feature and Property. Feature describes the device's characteristics, so that it is read only. In DAQNavi, you will find the features only has a Get method. Property can be changed by programming, at runtime. You can also configure the properties using the Configuration Dialog in the system's Device Manager. Before changing any of the Property's value, you'd best check the corresponding features for details.
設備屬性儲存在系統中，驅動程式載入時會檢索這些屬性，然後根據這些已載入的值對設備進行編程。由於硬體不同，不同的設備具有不同的屬性。 DAQNavi 中的屬性可以分為兩類：特性和屬性值。特性描述設備的特徵，因此是唯讀的。在 DAQNavi 中，特性只有 Get 方法。屬性可以在運行時透過編程進行更改。您也可以使用系統設備管理員中的配置對話方塊來配置屬性。在更改任何屬性值之前，最好先查看相應的特性以了解詳細資訊。

## Analog Input

### AiCtrlBase

#### Channels

If you set SignalType for one channel, please note that two corresponding channels should be set to Differential, and the two channels should have the same ValueRange when you set ValueRange for the channel.
如果為一個通道設定訊號類型，請注意，兩個對應的通道應設定為差分訊號類型，並且在為該通道設定值範圍時，這兩個通道的值範圍應相同。

# Method List

# Event ID List

|              | Component Name | Events          | Description                                                                                                                                                                                      |
| ------------ | -------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Device       | DeviceCtrl     | PropertyChanged | Event that occurs when the low-level device's properties changes. It is only effective for the device which selected with 'ModeRead'.<br>底層設備屬性改變時所發生的事件。僅對使用“ModeRead”模式選擇的設備有效。                |
|              |                | Reconnected     | This event occurs when the selected device reconnect to the system after breaking for a while. USB series devices often supports this event.<br>當選定的裝置斷開連線一段時間後重新連線到系統時，會發生此事件。 USB 系列裝置通常支援此事件。 |
|              |                | Removed         | This event occurs when the selected device removes from the system. USB series devices often supports this event.<br>當選定的設備從系統中移除時，會發生此事件。 USB 系列裝置通常支援此事件。                                      |
| Analog Input | WaveformAiCtrl | DataReady       | Event that occurs when the Buffered AI operation has finished transferring one section of data.<br>當緩衝 AI 操作完成一個資料段的傳輸時發生的事件。                                                                    |
|              |                | Overrun         | Event that occurs when the Buffered AI operation overruns.<br>緩衝 AI 操作溢位時發生的事件。                                                                                                                  |
|              |                | CacheOverflow   | Event that occurs when the cache for the Buffered AI operation overflows.<br>當緩衝 AI 操作的快取溢出時發生的事件。                                                                                               |
|              |                | Stopped         | Event that occurs when the Buffered AI operation has stopped.<br>緩衝 AI 操作停止時發生的事件。                                                                                                               |
