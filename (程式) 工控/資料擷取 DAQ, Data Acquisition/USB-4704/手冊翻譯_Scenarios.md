---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Scenarios

The function of a data acquisition card is to covert physical quantity in nature to digital signal for computer processing. Its applications are wide but complex, while its usages can vary enormously limited by components of a data acquisition card, design differences and sensor response. Therefore, Advantech has proposed a concept and design of "Scenarios" for its data acquisition cards. Scenarios can classify some given usages into certain operation process models, and illustrate the operation method and process of data acquisition card through examples, which will help the user to foresee the possible results before programming and also to select the specific code snippet to be applied to user's program development.
資料擷取卡的功能是將自然界的實體量轉換為數位訊號，供電腦處理。其應用範圍廣泛但複雜，且由於資料擷取卡的組件、設計差異和感測器響應的限制，其具體應用可能千差萬別。為此，研華科技針對其數據採集卡提出了「場景」的概念和設計。場景可以將一些給定的應用場景歸類到特定的操作流程模型中，並透過範例說明資料擷取卡的操作方法和流程，幫助使用者在程式設計之前預見可能的結果，並選擇適用於使用者程式開發的特定程式碼片段。

Scenarios include many abstracted card application concept. Actually, there is no specific standard about the scenarios that the data acquisition system should cover. Based on the current known application cases and the functions Advantech data acquisition cards support, we have designed the following scenarios for users. We believe that these scenarios can not cover all possible application cases, but they will be continuously expanded and rectified with the development of new product functions and the accumulation of application examples.
場景涵蓋了許多抽象的卡片應用概念。實際上，對於資料採集系統應涵蓋的場景並沒有具體的標準。基於目前已知的應用案例和研華資料擷取卡支援的功能，我們為使用者設計了以下場景。我們相信這些場景無法涵蓋所有可能的應用場景，但隨著新產品功能的開發和應用案例的積累，這些場景將不斷擴展和完善。

# Analog Input Programming Flowcharts

## Instant AI

Instant AI emphasizes on real time response. When Instant AI function is called, driver will return current analog value as soon as possible. Generally speaking, if an application is used as a command for next step after a certain analog data is processed, Instant AI command will be used, such as PID control, SCADA control, and etc. In DAQNavi, the call flow of Instant AI is so simple that one command can complete all actions. As for some setting functions of ADC, they may not always be executed during run time, and driver will directly use settings in device configuration page.
Instant AI 強調**即時回應**。當呼叫 Instant AI 函數時，驅動程式會盡快傳回目前的類比值。一般來說，==如果應用程式在處理特定類比資料後需要發出下一步指令，例如 PID 控制、SCADA 控制等，則會使用 Instant AI 指令。==在 DAQNavi 中，Instant AI 的呼叫流程非常簡單，一條指令即可完成所有操作。至於 ADC 的某些設定功能，它們可能並非始終在執行時執行，驅動程式會直接使用裝置設定頁面中的設定。

![[AdvantechDAQNaviSdk_Flowcharts_Instant AI.png|Instant AI]]

## Synchronous One Buffered AI

One buffered AI supports reading back a finite amount of analog data, the number of which may be more than one. Each ADC conversion requires a specific trigger signal from either the card itself or an external clock to start. Synchronous One Buffered AI means that call return can both complete data acquisition and save it into the buffer, and directly move to next step data processing without any additional codes to decide whether to complete multi-tasking synchronous actions. Since the conversion efficiency of ADC has been constantly improved, the time of acquisition of a specific amount of data may be short when high sampling rate is used. If this time requirement is acceptable by program operation (for example, 1000 pieces of huge data acquisition can be completed within 1 ms with a sampling rate of 1 MHz), the scenario of Synchronous One Buffered AI can be used so as to make the whole call process is as easy as getting one piece of data.
單緩衝 AI 支援讀取**有限數量**的類比數據，該數量可能不只一個。每次 ADC 轉換都需要來自卡片本身或外部時脈的**特定觸發訊號**才能啟動。同步單緩衝 AI 意味著**呼叫返回後，既可以完成資料採集並將其保存到緩衝區，又可以直接進入下一步資料處理，無需任何額外的程式碼來決定是否完成多任務同步操作。** 由於 ADC 的轉換效率不斷提高，在高取樣率下，採集特定數量資料所需的時間可能很短。==如果程式運作可以接受這種時間要求（例如，在 1 MHz 的取樣率下，1 ms 內可以完成 1000 個大型資料擷取），則可以使用同步單緩衝 AI 方案==，使整個呼叫過程如同取得一條資料一樣簡單。

![[AdvantechDAQNaviSdk_Flowcharts_Synchronous One Buffered AI.png|Synchronous One Buffered AI]]

## Asynchronous One Buffered AI

One buffered AI supports reading back a finite amount of analog data at one time, the number of which may be more than one. Each ADC conversion requires a specific trigger signal from either the card itself or an external clock to start. Asynchronous One Buffered AI can only complete start action during call return, and then driver will send `DataReady` and `Stop` events to inform the caller that a section of data has been sampled and the sampling is finished so as to enable the program to realize synchronous multi-tasking. In function operation process, the caller can stop the operation of One Buffered AI at any time according to the operation status. The usage of multi-tasking synchronous call may be comparatively complex, but can realize high efficiency of the whole program operation. If the entire sampling time is longer (such as, longer than 100 ms) which may affect the operation fluency, it is suggested to use Asynchronous One Buffered AI.
單緩衝 AI 支援**一次性**讀取**有限數量**的類比數據，讀取數量可以不只一個。每次 ADC 轉換都需要來自卡片本身或外部時脈的**特定觸發訊號**才能啟動。非同步單緩衝 AI **只能在呼叫返回期間完成啟動操作**，之後驅動程式會發送 `DataReady` 和 `Stop` 事件來通知呼叫者一段資料已被取樣且採樣已完成，從而使程式能夠實現同步多任務處理。在函數執行過程中，呼叫者可以根據操作狀態隨時停止單一緩衝 AI 的運作。多任務同步呼叫的使用可能相對複雜，但可以實現整個程式運行的高效性。==如果整個採樣時間較長（例如超過 100 毫秒），可能會影響操作流暢性，建議使用非同步單緩衝 AI。==

![[AdvantechDAQNaviSdk_Flowcharts_Asynchronous One Buffered AI.png|Asynchronous One Buffered AI]]

## Streaming AI

Streaming AI means after data acquisition is started, ADC will convert the analog signals to digital values according to a trigger clock. Then the data will be continuously sent to call program by the driver until it is demanded to stop. Each ADC conversion requires a specific trigger signal from either the card itself or an external clock to start, depending on the actual requirements of the application.
串流式 AI 是指資料擷取開始後，ADC 會**根據觸發時脈將類比訊號轉換為數位值**。然後，**驅動程式會將資料持續傳送給呼叫程序，直到收到停止指令。** 每次 ADC 轉換都需要來自卡片本身或外部時脈的特定觸發訊號才能啟動，這取決於應用的實際需求。

Streaming AI has no specific stop mechanism, thus belongs to synchronous operation mode. Only start action can be completed during call return, then driver will save data to the buffer and send out `DataReady` events of a specified number so as to enable the program to realize synchronous multi-tasking. Since data is continuously sent out, the driver must know the processing situation of the program to maintain the whole operation status. If an event of "whether some data is missing" is sent out, the driver will require the caller to send a hand shaking call as a response to clear the event, then the whole operation can be smooth and correct.
串流式 AI **沒有特定的停止機制**，因此屬於**同步運作模式**。在呼叫返回期間，只能完成啟動操作，然後驅動程式會將資料保存到緩衝區，並發送指定數量的 `DataReady` 事件，從而使程式能夠實現同步多任務處理。**由於資料會持續發送，驅動程式必須了解程式的處理情況，以維護整體運作狀態。** 如果發送了「資料是否缺失」的事件，驅動程式會要求呼叫方發送握手呼叫作為回應來清除該事件，從而確保整個運行的流暢性和正確性。

![[AdvantechDAQNaviSdk_Flowcharts_Streaming AI.png|Streaming AI]]

## Triggered AI

Triggered AI means after data acquisition is started, ADC will convert, or stop the analog signals to digital values according to a trigger clock. For One buffered AI, there are two ways for triggered AI, such as starting and stopping the ADC. For streaming AI, there is only one way for triggered AI, such as starting the ADC.
觸發式 AI 是指資料擷取開始後，類比數位轉換器 (ADC) 會根據觸發時脈將類比訊號轉換為數位值或停止轉換。對於 One buffered AI，觸發式 AI 有兩種方式，即啟動和停止 ADC。對於 streaming AI，觸發式 AI 只有一種方式，即啟動 ADC。

![[AdvantechDAQNaviSdk_Flowcharts_Triggered AI.png|Triggered AI]]

# Analog Output Programming Flowcharts

## Static AO

During the call, data on analog output channel should be updated in real-time to complete the update of output data. During call return, the level on AO channel may not reach the target level. Since different DAC supports different conversion efficiencies and the conversion efficiency is also related to the level difference before and after conversion, resulting in different output delays, **it is not suitable for the driver to lock the operation of the whole program to wait for the end of conversion and thus return the caller when the output command is completed.** A common system design is usually equipped with a feedback loop to know the operation status, such as PID control. If there is no feedback loop, time requirements of delay should be paid special attention during program development.
在呼叫過程中，類比輸出通道上的資料需要即時更新，以完成輸出資料的更新。呼叫返回時，AO 通道上的電平可能無法達到目標電平。由於不同的 DAC 支援不同的轉換效率，而轉換效率又與轉換前後的電平差相關，從而導致**不同的輸出延遲**，因此**驅動程式不應該為了等待轉換結束而鎖定整個程式的運行，從而在輸出命令完成後返回呼叫者。** 常見的系統設計通常會配備回饋迴路來了解運作狀態，例如 PID 控制。**如果沒有回饋迴路，則在程式開發過程中應特別注意延遲的時間要求。**

![[AdvantechDAQNaviSdk_Flowcharts_Static AO.png|Static AO]]

## Synchronous One Waveform AO

One Waveform AO means the output will be automatically stopped after a section of waveform has been output. Synchronous One Waveform AO means all data has been sent to data acquisition card when function call returns which does not mean all the waveforms have been output, thus time wait method or "`Stopped`" event can be used to confirm whether the waveform has been output completely. Regarding to the design of the device, buffer should be designed in order to ensure a continuous data output. When data has been saved into the buffer, it will be then updated to DAC one by one according to the predefined clock frequency. When all waveform data has been completely output to the buffer of the device, call function of Synchronous One Waveform AO will return caller, but some data may not be output to the channel. Thanks to the improvement of the computer performance, the time required to send a section of waveform to the buffer of the device may be very short. For example, if there is a buffer of 2 K on the device and a waveform of any kind including 1 K data should be output, the required time is very short no matter what the requirements of DAC's update frequency are. In such cases, the scenario of Synchronous One Waveform AO can be used so as to make the whole call process is as easy as an AO and even no additional design of multi-tasking synchronous actions will not affect the fluency of program operation.
單波形輸出 (One Waveform AO) 表示在輸出一段波形後自動停止輸出。同步單波形輸出 (Synchronous One Waveform AO) 表示函數呼叫返回時所有資料已傳送至資料擷取卡，但**這並不表示所有波形都已輸出**，因此可以使用時間等待方法或「`Stopped`」事件來確認波形是否已完全輸出。在設備設計方面，**應設計 buffer 以確保資料連續輸出**。資料儲存到緩衝區後，將根據預設的時脈頻率逐一更新到數位類比轉換器 (DAC)。當所有波形資料都已完全輸出到裝置緩衝區時，同步單波形輸出的呼叫函數將會傳回，但部分資料可能尚未輸出到通道。由於電腦效能的提升，將一段波形傳送到裝置緩衝區所需的時間可能非常短。例如，如果裝置上有 2K 的緩衝區，並且需要輸出包含 1K 資料的任意波形，那麼無論 DAC 的更新頻率要求如何，所需時間都非常短。在這種情況下，可以使用同步單波形 AO 方案，使整個呼叫過程像 AO 一樣簡單，即使不進行額外的多任務同步操作設計，也不會影響程式運行的流暢性。

![[AdvantechDAQNaviSdk_Flowcharts_Synchronous One Waveform AO.png|Synchronous One Waveform AO]]

## Asynchronous One Waveform AO

One Waveform AO means the output will be automatically stopped after a section of waveform has been output. Asynchronous One Waveform AO means to return caller after all transmission settings are completed and Buffered AO function is started, and then the driver will send out events based on data output status for the caller to carry out multi-tasking synchronous operations. Regarding to the design of the device, buffer should be designed in order to ensure a continuous data output. When data has been saved into the buffer, it will be then updated to DAC one by one according to the predefined clock frequency. In scenario of Asynchronous One Waveform AO, it is supposed that when all data has been sent to the device, this operation can be terminated and a new AO operation can be started; however, the device at this moment may not stop running completely and there may be still some output data remained in the buffer to wait for being sent to ADC through clock trigger. Even if the driver has stopped running, it will still monitor the running status of the device to see whether all data has been output, so as to send out an event of output completed. Though the operation mode of Asynchronous is more complicated compared with that of Synchronous, when multi-tasking operation becomes the only choice because the complete time is too long or uncertain and the fluency of program operation should be ensured, Asynchronous will be the only ideal option.
單波形 AO 是指在輸出一段波形後自動停止輸出。非同步單波形 AO 是指在所有傳輸設定完成後，緩衝 AO 功能啟動，然後驅動程式會**根據資料輸出狀態傳送事件**，供呼叫者執行多任務同步操作。在設備設計方面，**應設計 buffer 以確保資料連續輸出**。資料儲存到緩衝區後，將根據預先定義的時脈頻率逐條更新至 DAC。在非同步單波形 AO 場景下，假設所有資料都已傳送到設備，則可以終止此操作並啟動新的 AO 操作；但是，此時設備可能不會完全停止運行，緩衝區中可能仍有一些輸出資料等待透過時脈觸發傳送到 ADC。即使驅動程式已停止運行，它仍會監控裝置的運作狀態，以查看所有資料是否已輸出，從而發送輸出完成事件。雖然非同步操作模式比同步操作模式更複雜，但當多任務操作成為唯一選擇（因為完成時間太長或不確定，並且需要保證程式運行的流暢性）時，非同步將是唯一理想的選擇。

![[AdvantechDAQNaviSdk_Flowcharts_Asynchronous One Waveform AO.png|Asynchronous One Waveform AO]]

## Streaming AO

Streaming AO will continuously update digital data in the buffer to analog output channels according to the predefined clock signal frequency until the caller demands to stop. Streaming AO has no specific stop mechanism, thus is an asynchronous operation mechanism in applications. The driver will always monitor the buffer's status during operation and send an event requiring new data to the caller in appropriate time, then the caller should prepare data of a new section and send it to the driver after receiving the event so as to let the driver complete transmission before the buffer is depleted. Another possible aim is to output a specific waveform constantly and repeatedly. The common way is to load all data into the buffer of the computer, to let the driver continuously read and transmit data from the buffer of the computer to that of the device. Buffered AO provides two output status to satisfy the demands of different applications: One is to immediately stop data update on analog channels no matter whether there is still unsent data in the buffer of the computer or that of the device; The other is to stop AO channel update when data on all computer's channels has been sent to AO channels.
串流類比輸出 (Streaming AO) 會根據預先定義的時脈訊號頻率，持續地將緩衝區中的數位資料更新到類比輸出通道，直到呼叫方要求停止為止。串流式類比輸出**沒有特定的停止機制**，因此在應用程式中是一種**非同步操作機制**。驅動程式會在運行期間持續監控緩衝區的狀態，並在適當的時候向呼叫方發送需要新資料的事件。呼叫方在收到事件後，應準備好新資料段並將其傳送給驅動程序，以便驅動程式在緩衝區耗盡之前完成傳輸。另一個可能的目標是持續重複地輸出特定的波形。**常見的做法是將所有資料載入到電腦的緩衝區中，然後讓驅動程式持續地從電腦緩衝區讀取資料並將其發送到裝置的緩衝區。** 緩衝類比輸出 (Buffered AO) 提供兩種輸出狀態以滿足不同應用程式的需求：一種是無論電腦緩衝區或裝置緩衝區中是否仍有未發送的數據，都立即停止類比通道上的數據更新；另一種是在電腦所有通道上的數據都已發送到​​類比輸出通道後，停止類比輸出通道的更新。

![[AdvantechDAQNaviSdk_Flowcharts_Streaming AO.png|Streaming AO]]

# Digital Input/Output Programming Flowcharts

## Static DI

During the call, the device should immediately get the specified DI port value.
呼叫期間，設備應立即取得指定的 DI 連接埠值。

![[AdvantechDAQNaviSdk_Flowcharts_Static DI.png|Static DI]]

## DI Interrupt

Di snap can get status of Di ports when DI interrupt event occurs. When DI bit meets a pre-defined edge change (rising or falling), an interrupt is generated.
Di snap 可以在 DI 中斷事件發生時取得 Di 連接埠的狀態。當 DI 位元遇到預先定義的邊緣變化（上升沿或下降沿）時，會產生中斷。

![[AdvantechDAQNaviSdk_Flowcharts_DI Interrupt.png|DI Interrupt]]

## DI Pattern Match interrupt

DI snap can get status of DI ports when pattern match interrupt event occurs. When the value of certain specified channel on DI port is equal to a certain predefined match value, the driver will immediately read the value of the specified digital channel and send out an event to inform the caller.
當模式比對中斷事件發生時，DI snap 可以取得 DI 連接埠的狀態。當 DI 連接埠上某個指定通道的值等於某個預先定義的符合值時，驅動程式會立即讀取指定數位通道的值，並傳送事件通知呼叫者。

![[AdvantechDAQNaviSdk_Flowcharts_DI Pattern Match interrupt.png|DI Pattern Match interrupt]]

## DI Status Change interrupt

DI Status Change Interrupt can get status of DI ports when status change event occurs. When the level of certain specified channel on DI port changes, the driver will immediately read the value of the specified digital channel and send out an event to inform the caller.
DI 狀態變更中斷可以在 DI 連接埠發生狀態變更事件時取得其狀態。當 DI 連接埠上某個指定通道的電平發生變化時，驅動程式會立即讀取該指定數位通道的值，並發送事件通知呼叫方。

Whether a DI port supports status change detection function depends on the design of the device.
DI 連接埠是否支援狀態變化偵測功能取決於設備的設計。

![[AdvantechDAQNaviSdk_Flowcharts_DI Status Change interrupt.png|DI Status Change interrupt]]

## Static DO

During the call, the digital value should be immediately output to DO port of the device. Relay and isolated DO operations also belong to this scenario. Because the response of these terminals are usually far slower than the function call of the program and the driver will not stop the operation of the program for these terminals, response delay requirements should be paid extra attention in the process of program designing so as to avoid missing the due status change.
呼叫期間，數位值應立即輸出到裝置的 DO 連接埠。繼電器和隔離式 DO 操作也屬於這種情況。由於這些終端的響應速度通常遠慢於程式的函數調用，且驅動程式不會因這些終端而停止程式運行，因此**在程式設計過程中應格外注意響應延遲要求，以免錯過必要的狀態變更。**

![[AdvantechDAQNaviSdk_Flowcharts_Static DO.png|Static DO]]

# Counter Programming Flowcharts

## Event Counter

Continuously count the pulse number of signal from counter input.
持續計數來自計數器輸入訊號的脈衝數。

![[AdvantechDAQNaviSdk_Flowcharts_Event Counter.png|Event Counter]]

## Frequency Measurement

This scenario can calculate the current pulse frequency of digital signals.
此方案可以計算數位訊號的當前脈衝頻率。

![[AdvantechDAQNaviSdk_Flowcharts_Frequency Measurement.png|Frequency Measurement]]

## Delayed Pulse Generation

Delayed Pulse Generation function of counter provides user the ability to output pulse on give interval on the output pin of counter. A certain pin on the device can be used to detect an external pulse. When an external pulse has been detected, another pin will output a pulse after a certain buffer pulse account.
計數器的延遲脈衝產生功能允許使用者在計數器的輸出引腳上按給定的時間間隔輸出脈衝。裝置上的特定引腳可用於偵測外部脈衝。當偵測到外部脈衝時，另一個引腳會在經過一定的緩衝脈衝間隔後輸出脈衝。

![[AdvantechDAQNaviSdk_Flowcharts_Delayed Pulse Generation.png|Delayed Pulse Generation]]

## Pulse Output with Timer Interrupt

Pulse Output with Timer Interrupt means two function, which is pulse output and timer interrupt. Pulse output provides user the ability to output signals with specified frequencies. Timer interrupt means the device will send out a pulse at a predefined interval; meanwhile the driver will send out an event.
脈衝輸出帶定時中斷功能包含脈衝輸出和定時中斷兩種功能。脈衝輸出允許使用者輸出指定頻率的訊號。定時中斷功能是指設備會依照預設的時間間隔發送脈衝訊號；同時，驅動程式會發出事件訊號。

![[AdvantechDAQNaviSdk_Flowcharts_Pulse Output with Timer Interrupt.png|Pulse Output with Timer Interrupt]]

## Pulse Width Measurement

This scenario can measure the duration time of high level and low level of a continuous pulse.
此方案可以測量連續脈衝高電平和低電平的持續時間。

![[AdvantechDAQNaviSdk_Flowcharts_Pulse Width Measurement.png|Pulse Width Measurement]]

## PWM Output

Generate PWM (Pulse Width Modulation) signal.
產生 PWM（脈衝寬度調變）訊號。

![[AdvantechDAQNaviSdk_Flowcharts_PWM Output.png|PWM Output]]

## UpDown counter

UpDown Counter function input consists of two square wave inputs. The counter will increment or decrement depend on the counting type and the phase of input channel A and B.
上下計數器功能輸入由兩個方波輸入組成。計數器會根據計數類型以及輸入通道 A 和 B 的相位進行遞增或遞減。

Chooses counting type of quadruple AB phase encoder counter. Counts the times of pulse produced by external digital signals.
選擇四路 AB 相位編碼器計數器的計數類型。對外部數位訊號產生的脈衝次數進行計數。

![[AdvantechDAQNaviSdk_Flowcharts_UpDown counter.png|UpDown counter]]

## Snap Counter

Snap counter is the function to use a trigger signal to latch the current counter value.
快照計數器是利用觸發訊號鎖定目前計數器值的功能。

Chooses snap event ID. Start the snap counter function, when specified event occurs, immediately read back the data from the specified counter channel memory to return buffer.
選擇捕捉事件 ID。啟動捕捉計數器功能，當指定事件發生時，立即從指定的計數器通道記憶體讀取資料到返回緩衝區。

![[AdvantechDAQNaviSdk_Flowcharts_Snap Counter.png|Snap Counter]]

## Continue compare

Continue Compare function can issue event when the count value matches the compare value set in advance .
當計數值與預先設定的比較值相符時，繼續比較功能可以發出事件。

Sets compare values and start up-down counter. When counter value and compare value matches, sends out an event.
設定比較值並啟動遞增遞減計數器。當計數器值與比較值相符時，發出事件。

![[AdvantechDAQNaviSdk_Flowcharts_Continue compare.png|Continue compare]]
