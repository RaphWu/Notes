---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Analog Input Test

Click the "Analog Input" tab in the "Device Test" interface . All the AI physical channels of the device will be listed on the left. Values of AI channels are updated periodically. Users are able to edit the value range of AI channels at any time by clicking the value range type in the list.
在「裝置測試」介面中，點選「類比輸入」標籤。設備的所有類比輸入實體通道將顯示在左側。類比輸入通道的值會定期更新。使用者可以隨時點擊清單中的值範圍類型來編輯類比輸入通道的值範圍。

Item 1: Shows the number of a corresponding analog input channel. Check the checkbox and the corresponding analog input channel will be selected.
顯示對應的類比輸入通道編號。勾選此複選框即可選取對應的類比輸入通道。

The list box list value range type for each channel. The user can select the value range type in the list box.
列錶框列出了每個通道的值範圍類型。使用者可以在列錶框中選擇值範圍類型。

Item 2: Shows the sample data in graphics of each channel.
以圖形方式顯示每個通道的取樣資料。

Item 3: Shows Analog input function supported by device. USB-4704 supports instant AI and buffered AI.
顯示設備支援的類比輸入功能。 USB-4704 支援即時類比輸入和緩衝類比輸入。

Item 4: Shows frequency of analog input sampling. Configure the sampling rate on the scroll bar or edit the textbox. The textbox is used to show current frequency.
顯示類比輸入取樣頻率。使用者可以透過捲軸配置取樣率，也可以在文字方塊中編輯。文字方塊用於顯示目前頻率。

Item 5: Press this button to start the AI function, then press it again will stop the running funtion.
按下此按鈕啟動 AI 功能，再按下即可停止運作功能。

---

# Analog Output Test

Click the "Analog Output" tab in the "Device Test" interface. All the AO physical channels of the device will be listed on the left.
在「裝置測試」介面中，點選「類比輸出」標籤。設備的所有類比輸出實體通道將顯示在左側。

Item 1: Shows the number of a corresponding analog output channel. Check the checkbox and the corresponding analog output channel will be selected.
顯示對應的類比輸出通道編號。選取複選框即可選取對應的類比輸出通道。

The list box list value range type for each channel. The user can select the value range type in the list box.
列錶框顯示每個通道的值範圍類型。使用者可以在列錶框中選擇值範圍類型。

Item 2: Shows the waveforms to be output analog output channel.
顯示要輸出到類比輸出通道的波形。

Item 3: Check domain Up Down button to set amplitude and DC offset value of waveform.
勾選「上下」按鈕可設定波形的振幅和直流偏移值。

Item 4: Shows analog output waveform in the graph.
以圖形方式顯示類比輸出波形。

Item 5: Shows analog output function supported by device. USB-4704 just supports instant AO.
顯示設備支援的類比輸出功能。 USB-4704 僅支援瞬時類比輸出。

Item 6: Shows frequency of analog output waveform. Scroll bar and textbox are used to adjust frequency and textbox is used to show current frequency.
顯示類比輸出波形的頻率。捲軸和文字方塊用於調整頻率，文字方塊用於顯示目前頻率。

Item 7: "Start" button is used to start analog output.
「開始」按鈕用於啟動模擬輸出。

---

# Digital Input Test

Click the "Digital Input" tab in the "Device Test" interface. All DI ports of the device will be listed. The values of DI port are updated automatically.
點選「裝置測試」介面中的「數位輸入」標籤。設備的所有數位輸入 (DI) 連接埠將列出。 DI 連接埠的值會自動更新。

Item 1: Shows the port number of a corresponding group of lights on the right.
顯示右側對應指示燈組的連接埠號碼。

Item 2: Shows input status of the ports.
顯示連接埠的輸入狀態。

Item 3: Shows the hexadecimal port value of a corresponding group of lights on the left.
顯示左側對應指示燈組的十六進位連接埠值。

---

# Digital Output Test

Click the "Digital Output" tab in the "Device Test" interface. These pages will present all DO ports of devices in a list. Users could flip the state of bit by clicking the buttons and editing the hex value of DO port.
點選「裝置測試」介面中的「數位輸出」標籤。這些頁面將以清單形式顯示裝置的所有數位輸出 (DO) 連接埠。使用者可以透過點擊按鈕切換位元狀態並編輯 DO 連接埠的十六進位值。

Item 1: Shows the port number of a corresponding group of buttons on the right.
顯示右側對應按鈕組的連接埠號碼。

Item 2: The user can click the button to output a value to corresponding channel, then this zone can show output state of the ports.
使用者可以點選按鈕向對應的頻道輸出一個值，然後此區域將顯示連接埠的輸出狀態。

Item 3: Shows the hexadecimal port value of a corresponding group of buttons on the left.
顯示左側對應按鈕組的十六進位連接埠值。第四項：根據設備特性，手動讀取 DO 連接埠的值。

Item 4: According to the device's features, read back DO ports value manually.
根據設備的特性，手動讀取 DO 埠的值。

---

# Counter Test

Click the "Counter" tab in the "Device Test" dialog box. All physical channels of the counter will be listed. Each channel could work in one of the modes it supports. Users could edit the mode and corresponding parameters of each channel.
在「裝置測試」對話方塊中，按一下「計數器」標籤。此時將列出計數器的所有實體通道。每個通道都可以在其支援的模式之一中工作。使用者可以編輯每個通道的模式及其對應的參數。

Item 1: Shows the number of a corresponding counter on the right. The list box list the function supported by this counter. Check the checkbox and the counter function which is selected in the list box will run. Event counting is selected.
在右側顯示對應計數器的編號。列錶框列出了此計數器支援的功能。選取複選框，列錶框中選擇的計數器功能將運作。已選擇事件計數。

Item 2: If the event EvtCntEcOverCompare is supported by the device, you can set the over compare value of counter.
如果裝置支援 EvtCntEcOverCompare 事件，則可以設定計數器的過比較值。

Item 3: Shows current counting value of event counting.
顯示事件計數的目前計數值。

Item 4: The count of EvtCntEcOverCompare event occurrence.
EvtCntEcOverCompare 事件的發生次數。

Item 1: Shows the number of a corresponding counter on the right. The list box list the function supported by this counter. Check the checkbox and the counter function which is selected in the list box will run. Frequency Meter is selected.
在右側顯示對應計數器的編號。列錶框列出了此計數器支援的功能。選取複選框，列錶框中選擇的計數器功能將運作。已選擇頻率計。

Item 2: Shows the method that used for frequency measurement.
顯示頻率測量方法。

Item 3: Set the period that used to instruct the device to detect the frequency automatically.
設定設備自動偵測頻率的周期。

Item 4:Shows current measured frequency value of counter.
顯示計數器目前測量的頻率值。
