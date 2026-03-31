---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# ADC

Analog to Digital Converter (ADC), also called A/D Converter, mainly converts analog signal to digital signal, which is the hardware infrastructure for AI function.
類比數位轉換器（ADC），又稱為 A/D 轉換器，主要用於將類比訊號轉換為數位訊號，是 AI 功能的硬體基礎架構。

# Ai Filter

A digitizer or ADC might sample signals containing frequency components above the Nyquist limit. The undesirable effect of the digitizer modulating out-of-band components into the Nyquist bandwidth is aliasing. The greatest danger of aliasing is that you cannot determine whether aliasing occurred by looking at the ADC output. If an input signal contains several frequency components or harmonics, some of these components might be represented correctly while others contain aliased artifacts.
數位轉換器或類比數位轉換器 (ADC) 可能會對包含高於奈奎斯特頻率範圍的頻率成分的訊號進行取樣。數位轉換器將帶外分量調製到奈奎斯特頻寬內會產生混疊效應。混疊效應最大的危害在於，無法透過觀察 ADC 輸出來判斷是否發生了混疊。如果輸入訊號包含多個頻率分量或諧波，則其中一些分量可能被正確表示，而另一些分量則包含混疊偽影。

Low-pass filtering to eliminate components above the Nyquist frequency either before or during the digitization process can guarantee that the digitized data set is free of aliased components.
在數位化過程之前或期間進行低通濾波，以消除奈奎斯特頻率以上的分量，可以確保數位化資料集中沒有混疊分量。

[取樣定理](https://zh.wikipedia.org/zh-tw/%E9%87%87%E6%A0%B7%E5%AE%9A%E7%90%86)
[奈奎斯特頻率](https://zh.wikipedia.org/zh-tw/%E5%A5%88%E5%A5%8E%E6%96%AF%E7%89%B9%E9%A2%91%E7%8E%87)

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - FilterTypes
- AiChannel
    - FilterType
    - FilterCutoffFreq
    - FilterCutoffFreq1

# Channel

## Physical Channel

Physical channel refers to visible input/output terminals or pins that can connect wires on a data acquisition card. This channel can be used to input or output a real signal, including digital signal or analog signal.
實體通道是指資料擷取卡上可見的輸入/輸出端子或接腳，可用於連接導線。此通道可用於輸入或輸出實際訊號，包括數位訊號或類比訊號。

Range of physical channel: \[0, total count of actual physical channels - 1]. For physical channel count, you can get it from hardware manual or via property `ChannelCountMax` in the program. If the count you get via property `ChannelCountMax` is 16, the device has 16 physical channels in all, numbered from 0 to 15.
實體通道數的範圍為：\[0, 實際實體通道總數 - 1]。實體通道數可以從硬體手冊中取得，也可以透過程式中的 `ChannelCountMax` 屬性取得。如果透過 `ChannelCountMax` 屬性取得的頻道數為 16，則表示該裝置共有 16 個實體頻道，編號從 0 到 15。

The input parameter "ch" of Read (int ch, out double dataScaled) method in InstantAiCtrl calss means the physical channel number, and its valid range is \[0,ChannelCountMax-1]. This method is used to read the data from a single channel.
InstantAiCtrl 類別中 Read (int ch, out double dataScaled) 方法的輸入參數「ch」表示物理通道號，其有效範圍為 \[0, ChannelCountMax-1]。此方法用於從單一通道讀取資料。

## Logical Channel

Logic channel is a concept on software layer. It refers to interfaces that you can actually use , which is composed of one or several physical channels.
邏輯通道是軟體層的一個概念。它指的是你可以實際使用的接口，這些接口由一個或多個實體通道組成。

Range of logic channel: a differential logic channel is made of two physical channels. If a device has 16 differential physical channels, its logic channel count is 8 numbered from 0 to7. If all 16 channels are single-ended, the same as physical channel count, its logic channel count is 16 numbered from 0 to 15.
邏輯通道範圍：差分邏輯通道由兩個實體通道組成。如果一個設備有 16 個差分實體通道，則其邏輯通道數為 8，編號從 0 到 7。如果所有 16 個通道都是單端的，與實體通道數相同，則其邏輯通道數為 16，編號從 0 到 15。

Remarks: In `AiCtrlBase` of DAQNavi, property `ChannelCount` indicates the number of channels. For example, if the real physical channel is 16 and all are single-ended, this value is 16; if all are differential, this value is 8.
備註：在 DAQNavi 的 `AiCtrlBase` 中，`ChannelCount` 屬性表示通道數。例如，如果實際物理通道數為 16，且全部為單端訊號，則該值為 16；如果全部為差分訊號，則該值為 8。

The input parameter "`chStart`" of Read `(int chStart, int chCount, int[] dataRaw, double[] dataScaled)` method means the physical channel number, and its valid range is \[0,ChannelCountMax-1]. The input parameter "`chCount`" means the the logical channel number, and its valid range is \[1, ChannelCount].
Read `(int chStart, int chCount, int[] dataRaw, double[] dataScaled)` 方法的輸入參數「`chStart`」表示實體通道號，其有效範圍為 \[0, ChannelCountMax-1]。輸入參數「`chCount`」表示邏輯頻道號，其有效範圍為 \[1, ChannelCount]。

## Operations to AI Channel in DAQNavi - DAQNavi 中 AI 頻道的操作

In DAQNavi, all physical channels of AI got by property `Channels` of `AiCtrlBase` compose `AnalogInputChannel` set. Each object (`AnalogInputChannel`) in the set represents a physical channel. You can set the properties of AI through an object, such as `ValueRange` and Signal Connection Mode (Single-ended or other modes). If Disconnection Detection function is supported, you can also set return value type of the channel that supports the capability of detecting disconnection.
在 DAQNavi 中，透過 `AiCtrlBase` 的 `Channels` 屬性所取得的所有 AI 實體通道構成 `AnalogInputChannel` 集合。集合中的每個物件 (`AnalogInputChannel`) 代表一個實體通道。您可以透過物件設定 AI 的屬性，例如 `ValueRange` 和訊號連接模式（單端或其他模式）。如果支援斷開偵測功能，您也可以設定支援斷開偵測功能的通道的回傳值類型。

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - ChannelCountMax
    - ConnectionTypes
    - OverallValueRange
    - ThermoSupported
    - ValueRanges
    - BurnoutReturnTypes
- AiChannel
    - Channel
    - LogicalNumber
    - ValueRange
    - SignalType
    - BurnoutRetType
    - BurnoutRetValue

# Single-Ended and Differential

## Signal Type

Analog signal has two types: Single-ended and Differential. In DAQNavi, enumeration `AiSignalType` corresponds to signal type.
類比訊號有兩種類型：單端訊號和差分訊號。在 DAQNavi 中，枚舉值 `AiSignalType` 對應於訊號類型。

## Signal Connection Mode

AI signal connection has four types: `AllSingleEnded`, `AllDifferential`, `AllSeDiffAdj` and `MixedSeDiffAdj`.
AI 訊號連接有四種類型：`AllSingleEnded`、`AllDifferential`、`AllSeDiffAdj` 和 `MixedSeDiffAdj`。

Single-ended connection measures the voltage difference between input signal and ground. Differential connection measures the voltage difference between two input signals. When measuring a single-ended signal, you just need to connect the signal to the input port through a wire. The measured input voltage is referenced to common ground, and no grounded signal source is called "floating" signal source. In this mode, the device provides a reference ground to external floating signal. Standard connection method for measuring single-ended analog signal input is shown as below:
單端連接測量輸入訊號與接地之間的電壓差。差分連接測量兩個輸入訊號之間的電壓差。測量單端訊號時，只需以導線將訊號連接到輸入埠即可。被測輸入電壓以公共地為參考，沒有接地的訊號源稱為「floating」訊號源。在這種模式下，設備會為外部浮地訊號提供參考地。單端類比訊號輸入的標準連接方式如下所示：

![[AdvantechDAQNaviSdk_Single-ended.png]]

For differential input, you need to connect two wires to two input channels, measuring the voltage difference between two inputs. Standard connection method for measuring differential analog signal input is shown as below:
對於差分輸入，需要將兩根導線連接到兩個輸入通道，以測量兩個輸入通道之間的電壓差。差分類比訊號輸入的標準連接方式如下所示：

![[AdvantechDAQNaviSdk_Differential.png]]

Not all the devices support differential channel, please refer to related hardware manual for detailed information or get signal connection modes supported by AI function of the device through property `ChannelType`. You should set an effective connection mode for each channel through property `SignalType`.
並非所有裝置都支援差分通道，詳細資訊請參閱相關硬體手冊，或透過屬性 `ChannelType` 取得裝置 AI 功能支援的訊號連線模式。您需要透過屬性 `SignalType` 為每個頻道設定有效的連線模式。

## Effect of Single-ended and Differential on Physical Channel and Logic Channel - 單端和差分對物理通道和邏輯通道的影響

Since the mixed use of both single-ended and differential will change logic channel (Single-ended connection only needs one physical channel, while differential connection needs two physical channels), the execution result of setting scanning channel range will be different.
由於單端和差分混合使用會改變邏輯通道（單端連接只需要一個實體通道，而差分連接需要兩個實體通道），因此設定掃描通道範圍的執行結果會有所不同。

Below example shows the changing relationship between single-ended or differential setting and physical channel or logic channel.
下面的例子顯示了單端或差分設定與物理通道或邏輯通道之間不斷變化的關係。

![[AdvantechDAQNaviSdk_Channel.png]]

As shown in above figure, physical channel 2 and 3, 6 and 7 are configured as two sets of differential input channels.
如上圖所示，實體通道 2 和 3、6 和 7 配置為兩組差分輸入通道。

Note: In practice, unused channel will be connected to ground (AIGND) to avoid its interference to measurement.
注意：實際上，未使用的通道將接地（AIGND），以避免其對測量造成干擾。

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - ConnectionTypes
- AiChannel
    - Channel
    - SignalType

# Data Resolution and LSB

For AI and AO functions, analog to digital converter (ADC) and digital to analog converter (DAC) should be used to realize the conversion between analog signal and digital signal and the correspondence between data of unlimited range and data of limited range. In this process, the concepts of resolution and LSB may be involved.
對於 AI 和 AO 功能，需要使用類比數位轉換器（ADC）和數位類比轉換器（DAC）來實現類比訊號和數位訊號之間的轉換，以及無限範圍資料和有限範圍資料之間的對應關係。在此過程中，可能涉及分辨率和最低有效位元（LSB）的概念。

## Data Resolution (ADC data resolution)

Data resolution refers to the number of the bit that is used to realize analog to digital conversion. The bigger the bit number of ADC is, the higher the resolution will be and the smaller the distinguished min. voltage value will be. The resolution should be high enough so that the digital signal can have the ability to distinguish voltage and better restore the original signal. Currently, Advantech DAQ Device supports the resolution of 8-bit, 12-bit, 14-bit and 16-bit, etc. Resolution is expressed by the bit number of the output binary number. If the resolution is 8-bit, the analog signal can be divided into 256 parts; if the resolution is 12-bit, it can be divided into 4096 parts. Therefore, the more the bit number is, the smaller the error will be and the higher the sampling precision will be. You can get ADC resolution of the device through property `Resolution` of `AiFeatures`.
資料解析度是指用於實現類比數位轉換的位數。 **ADC 的位數越大，解析度越高，可區分的最小電壓值就越小**。**分辨率應足夠高，以使數位訊號能夠區分電壓並更好地還原原始訊號**。目前，研華資料採集設備支援 8 位元、12 位元、14 位元和 16 位元等解析度。分辨率以輸出二進制數的位數表示。如果解析度為 8 位，則類比訊號可以被分成 256 份；如果解析度為 12 位，則可以分成 4096 份。因此，**位數越多，誤差越小，採樣精度越高**。您可以透過 `AiFeatures` 的 `Resolution` 屬性來取得裝置的 ADC 解析度。

## LSB

LSB refers to the analog voltage variation needed when the digital quantity changes a unit. LSB refers to the smallest change of input signal that can be detected by measuring equipment, i.e. coding width. The smaller the coding width is, the higher the measuring precision will be.
LSB（最低有效位元）是指當數位量變化一個單位時所需的類比電壓變化量。 LSB 指的是測量設備能夠偵測到的輸入訊號最小變化量，即編碼寬度。編碼寬度越小，測量精度越高。

For example, if the resolution is 8-bit, there will be 2^8=256 coding. If the input voltage range is 0～10 V, the below formula can be applied.
例如，如果解析度為 8 位，則編碼數為 2^8=256。如果輸入電壓範圍為 0～10 V，則可使用下列公式。

$\frac{10}{2^8} = \frac{10}{256} \approx 0.039065 V$

Then, LSB represents 0.039065V.Below figure shows a coding and LSB comparison diagram of a 0 ~ 10 V sinusoidal signal sampled by both 3-bit and 2-bit ADC.
那麼，LSB 代表 0.039065V。下圖顯示了 3 位元和 2 位元 ADC 對 0 ~ 10 V 正弦訊號進行取樣的編碼和 LSB 比較圖。

![[AdvantechDAQNaviSdk_LSB.png]]

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - Resolution
    - DataSize

# Data Type

DAQNavi has two data types: one is raw data, the other is scaled data. Although major conversion functions are completed by parser, the data conversion function of most DAQ devices are basically the same since the meaning of conversion is almost fixed. For example, the conversion from raw data to voltage or current is normally linear conversion.
DAQNavi 有兩種資料類型：原始資料和縮放後的資料。雖然主要的轉換功能由解析器完成，但由於轉換的含義幾乎固定，大多數 DAQ 設備的資料轉換功能基本上相同。例如，從原始資料到電壓或電流的轉換通常是線性轉換。

In DAQNavi, `Read` (method for reading data) of Instant AI provides multiple interfaces, through which you can get raw data or scaled data. In Buffered AI, method `GetData` can be used to get acquired AI data. Similar with Instant AI, it also provides multiple interfaces, through which you can get acquired raw data via specified data type or scaled data converted according to device features.
在 DAQNavi 中，Instant AI 的 `Read` 方法（用於讀取資料）提供了多種接口，您可以透過這些接口獲取原始資料或縮放後的資料。在 Buffered AI 中，可以使用 `GetData` 方法取得擷取的 AI 資料。與 Instant AI 類似，它也提供了多種接口，您可以透過這些接口獲取指定資料類型的原始資料或根據設備特徵轉換後的縮放資料。

## Raw Data

Raw Data refers to source data, i.e. binary data stream.
原始資料指的是來源數據，即二進位資料流。

## Scaled Data

Scaled Data refers to data converted and calculated from Raw Data (binary data stream).
縮放資料是指由原始資料（二進位資料流）轉換和計算得到的資料。

## DAQNavi References

- InstantAiCtrl
    - Read
- WaveformAiCtrl
    - GetData

# Data Conversion

In AI and AO functions there exist a conversion between digital value and analog value, which is a conversion table in the driver. For AI, an analog-to-digital conversion table is needed to calculate the analog value by the digital value got from ADC and `ValueRange` set in AI function. For AO, an analog-to-digital conversion table is needed to calculate the digital value by `ValueRange` and the analog value specifically output. So far, since Advantech devices support excellent linearity, there is only one conversion table and the conversion relation is a first-order polynomial. DAQNavi supports the method of setting conversion table and getting current conversion table. This conversion table allows you to set several tables to realize fitting with a piecewise curve, while each curve can be a polynomial of arbitrary order. Here we will not discuss it in detail. For more information, please consult Advantech AE.
在 AI 和 AO 功能中，存在數位值和類比值之間的轉換，這體現在驅動程式中的一個轉換表中。對於 AI，需要一個類比數位轉換表，根據從 ADC 獲得的數位值和 AI 功能中設定的 `ValueRange` 值來計算類比值。對於 AO，需要一個類比數位轉換表，根據 `ValueRange` 值和具體輸出的類比值來計算數字值。目前，由於研華裝置具有出色的線性度，因此只有一個轉換錶，且轉換關係為一階多項式。 DAQNavi 支援設定轉換表和取得目前轉換表的方法。此轉換表可讓您設定多個轉換錶，以實現分段曲線擬合，其中每條曲線可以是任意階的多項式。此處不再贅述。更多信息，請諮詢研華 AE。

# Data Size

Data Size refers to the number of bytes occupied by raw data of a sampling point, the unit is byte. Data Size is decided by specific hardware specifications of the data acquisition card, but not limited to ADC resolution of the card. For example, ADC resolution of PCI-1712 is 12-bit, so Data Size is 2 bytes; ADC resolution of PCM-3813I is 12-bit, so Data Size is 4 bytes.
資料大小是指採樣點原始資料所佔用的位元組數，單位為位元組。資料大小取決於資料擷取卡的具體硬體規格，但不限於該卡的 ADC 解析度。例如，PCI-1712 的 ADC 解析度為 12 位元，因此資料大小為 2 位元組；PCM-3813I 的 ADC 解析度為 12 位元，因此資料大小為 4 位元組。

The definition of Data Size is aimed to let users assign a buffer memory of enough length for storing raw sampling data. For example, Data Size of PCM-3813I is 4, so at least 4 bytes are needed for storing a raw data, among which bit0~bit4 are used to store channel number while bit20~bit32 are used to store effective ADC data. Below is the structure of PCM-3813I register (high effective).
資料大小的定義旨在讓使用者分配足夠長度的緩衝區來儲存原始採樣資料。例如，PCM-3813I 的資料大小為 4，因此至少需要 4 個位元組來儲存原始數據，其中 bit0~bit4 用於儲存通道號，bit20~bit32 用於儲存有效 ADC 資料。下圖是 PCM-3813I 暫存器的結構（高有效值）。

![[AdvantechDAQNaviSdk_DataSize_1.png]]

Below is the data storage format of PCI-1712 and the structure of the register with a data size of 2 bytes (low effective). Here, bit0~bit11 are used to store effective ADC data, while other bits are invalid.
下圖展示了 PCI-1712 的資料儲存格式以及資料大小為 2 位元組（低有效位元）的暫存器結構。其中，bit0~bit11 用於儲存有效的 ADC 數據，其餘位無效。

![[AdvantechDAQNaviSdk_DataSize_2.png]]

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - Resolution
    - DataSize

# Data Mask

Data Mask is used to define the effective bits of AD data got from device register.
資料遮罩用於定義從裝置暫存器取得的 AD 資料的有效位元。

Register data got from each sampling point may contain other relative information except AD data. You can logically calculate register data and Data mask to get AD data in this register data.
從每個採樣點取得的暫存器資料可能包含 AD 資料以外的其他相關資訊。您可以邏輯地計算暫存器資料和資料遮罩，從而從這些暫存器資料中獲得 AD 資料。

Some devices are low effective, while others are high effective. Here, PCM-3813I (high effective) and PCI-1712 (low effective) are taken as examples to illustrate the functions of Data Mask.
有些裝置的效率較低，而有些元件的效率較高。這裡以 PCM-3813I（效率較高）和 PCI-1712（效率較低）為例，說明資料遮罩的功能。

In the below figure, high effective PCM-3813I is taken as an example. The figure shows the bit distribution structure of data register whose data size is 4, its corresponding data mask as well as the calculation result of Data Mask and calculation (AND).
下圖以高效 PCM-3813I 為例。圖中展示了資料大小為 4 的資料暫存器的位元分佈結構、其對應的資料遮罩以及資料遮罩與運算（AND）的結果。

![[AdvantechDAQNaviSdk_DataMask_1.png]]

In the below figure, low effective PCI-1712 is taken as an example. The figure shows the bit distribution structure of data register whose data size is 2, its corresponding data mask as well as the calculation result of Data Mask and calculation (AND).
下圖以低效率的 PCI-1712 為例。圖中展示了資料大小為 2 的資料暫存器的位元分佈結構、其對應的資料遮罩以及資料遮罩與運算（AND）的結果。

![[AdvantechDAQNaviSdk_DataMask_2.png]]

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - Resolution
    - DataSize

# Convert Clock Source

## Clock

Periodical digital edge can be used as an clock. Clock, except conversion (sampling) clock, is unlike trigger that can cause a certain action. DAQNavi has the following clocks:
週期性數位邊緣可用作時脈。除轉換（取樣）時鐘外，時鐘與觸發器不同，觸發器可以觸發特定動作。 DAQNavi 具有以下時鐘：

- AI conversion clock
- AO conversion clock
- DI conversion clock
- DO conversion clock
- Counter conversion clock

When the digital edge used as an trigger is periodical, clock is trigger. Conversion clock is an good example.
當用作觸發訊號的數位邊緣具有週期性時，時脈訊號就是觸發訊號。轉換時鐘就是一個很好的例子。

## Conversion Clock Source

Conversion clock source is the clock signal that drives ADC or DAC module to convert analog value to digital value or digital value to analog value. The device makes use of conversion clock to control the generation speed of sampling data or the speed of data output.
轉換時脈源是驅動類比數位轉換器 (ADC) 或數位類比轉換器 (DAC) 模組進行類比值與數位值或數位值與類比值轉換的時脈訊號。本設備利用轉換時脈來控制採樣資料的產生速度或資料輸​​出速度。

On the design of AI hardware, if no time-division multiplexer is used which means one ADC corresponds to one channel, the conversion clock will control all the ADCs to work at the same time. If a time-division multiplexer is used which means one ADC corresponds to multiple channels, the conversion clock will switch among several channels with the help of the multiplexer, to make sure each channel can sample the data at a certain speed. Therefore, in AI, conversion clock refers to both the clock that triggers ADC to work and the clock that controls the switch of time-division multiplexer.
在 AI 硬體設計中，如果不使用分時多工器（即一個 ADC 對應一個通道），轉換時脈將控制所有 ADC 同時工作。如果使用分時多工器（即一個 ADC 對應多個通道），轉換時脈將藉助分時多工器在多個通道之間切換，以確保每個通道都能以一定的速度取樣資料。因此，在 AI 中，轉換時鐘既指觸發 ADC 工作的時鐘，也指控制分時多工器切換的時鐘。

AI conversion clock source (Convert Clock Source) refers to clock signal source that drives ADC module to realize the analog-to-digital conversion. The device uses conversion clock to control the generation speed of the samples.
AI 轉換時脈源（轉換時脈源）是指驅動 ADC 模組實現類比數位轉換的時脈訊號源。本設備利用轉換時脈來控制取樣頻率的產生速度。

In DAQNavi, two available clock sources of conversion clock source are:
在 DAQNavi 中，轉換時鐘來源有兩種可用時鐘來源：

1. Internal clock source. Internal clock is built-in on the hardware of the data acquisition card, which can use the signal generated by its own circuit as the clock source, therefore there is no need of extra connection for the signal of clock source.
   內部時鐘源。資料擷取卡的硬體內建內部時鐘，可利用自身電路產生的訊號作為時脈源，因此無需額外連接時脈源訊號。

2. External clock source. External clock can connect an external signal to the clock pin (CLK pin) to serve as the trigger signal source.
   外部時鐘源。外部時脈可以將外部訊號連接到時脈引腳（CLK 引腳），作為觸發訊號源。

Structure `SignalDrop` in DAQNavi lists all possible clock sources, including multiple frequencies of internal clock source, AI external clock source, DIO external clock source and Counter external clock source.
DAQNavi 中的 `SignalDrop` 結構列出了所有可能的時脈來源，包括內部時脈來源的多個頻率、AI 外部時脈來源、DIO 外部時脈來源和 Counter 外部時脈來源。

If an internal clock is used, the clock frequency is limited. Different data acquisition cards provide different available clock frequencies. If the clock frequency needed by your sample signal is not available in internal clock, the external clock will become very useful. You can set frequency or other parameters of the external clock source according to your requirements.
如果使用內部時鐘，時鐘頻率會受到限制。不同的資料擷取卡提供的可用時脈頻率也不同。如果內部時脈無法提供取樣訊號所需的頻率，外部時脈就非常有用。您可以根據需要設定外部時鐘來源的頻率或其他參數。

Remarks: On the design of AO hardware, no time-division multiplexer is applied, so one DAC corresponds to one channel.
備註：在 AO 硬體設計中，沒有採用時分複用器，因此一個 DAC 對應一個通道。

## Reference Clock Source

The frequency timebase is generated by a phase-locked loop (PLL) whose input is driven by the reference clock. This frequency timebase feeds a direct digital synthesis (DDS) chip, which is used to generate the other on-board timing signals such as sampling clock.
頻率時基由鎖相環 (PLL) 生成，其輸入由參考時脈驅動。此頻率時基饋送至直接數位合成 (DDS) 晶片，該晶片用於產生其他板載定時訊號，例如取樣時脈。

In DAQNavi, two available reference clock sources are:
在 DAQNavi 中，有兩個可用的參考時鐘來源：

1. Internal reference clock source. Internal reference clock is built-in on the hardware of the data acquisition card so that there is no need of extra connection for the signal of reference clock source.
   內部參考時鐘來源。內部參考時脈內建於資料擷取卡的硬體中，因此無需額外連接參考時脈來源訊號。

2. External reference clock source. User can connect an external reference clock to the clock pin (CLK pin). When synchronizing multiple devices, each device must share a common sample clock timebase so that each is able to generate a phase-aligned version of the ADC oversample clocks. This allows for tight synchronization between multiple devices.
   外部參考時鐘來源。使用者可以將外部參考時脈連接到時脈引腳（CLK 引腳）。同步多個裝置時，每個裝置必須共用一個公共的取樣時脈時基，以便每個裝置都能產生相位對齊的 ADC 過取樣時脈。這可以實現多個裝置之間的精確同步。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - ConvertClockSources
    - ConvertClockRange
- WaveformAiCtrl
    - Conversion
- Conversion
    - ClockSource
    - ClockRate

# Convert Clock Rate

Convert Clock Rate refers to the sampling frequency of each AI channel during high-speed sampling (Buffered AI). Sampling frequency (also called sampling speed or sampling rate) defines sampling count per second, the unit is Hz. The reciprocal of sampling frequency is sampling period or sampling time, which is the time interval between the samplings. The number of sampling points in a specified time period can be decided by sampling frequency. For example, when an analog signal of more than 1 second is continuously sampled and its sampling frequency is 1 M, on the time axis a point will be sampled every 1 us. Then totally 1 M points will be sampled in 1 second, and 10 M points will be sampled in 10 seconds.
轉換時脈頻率是指在高速取樣（緩衝 AI）期間每個 AI 通道的取樣頻率。取樣頻率（也稱為取樣速度或取樣率）定義了每秒的取樣次數，單位為赫茲（Hz）。取樣頻率的倒數是取樣週期或取樣時間，即取樣之間的時間間隔。在指定的時間段內，取樣點的數量由取樣頻率決定。例如，當對一個持續超過 1 秒的類比訊號進行連續取樣，且其取樣頻率為 1MHz 時，在時間軸上每 1 微秒取樣一個點。那麼，1 秒內總共會採樣 100 萬個點，10 秒內會採樣 1000 萬個點。

Sampling theorem indicates that the sampling frequency should be more than twice of the bandwidth of the sampled signal. Another equivalent statement is Nyquist frequency should be greater than the bandwidth of the sampled signal. The sampling frequency should be at least twice as the frequency of the component of the max frequency in the signal, otherwise the original signal can not be restored from signal sampling. If the signal width is 100 Hz, the sampling frequency should be greater than 200 Hz in order to avoid aliasing.
取樣定理表明，取樣頻率應大於被取樣訊號頻寬的兩倍。換句話說，奈奎斯特頻率應大於被取樣訊號的頻寬。取樣頻率至少應為訊號中最大頻率成分頻率的兩倍，否則無法透過訊號取樣恢復原始訊號。例如，如果訊號寬度為 100 Hz，則取樣頻率應大於 200 Hz 以避免混疊。

In DAQNavi, property `Rate` of `ConvertClock` can be used to set the sampling frequency of each channel.
在 DAQNavi 中，可以使用 `ConvertClock` 的 `Rate` 屬性來設定每個頻道的取樣頻率。

Note: It should be noted that, the value got by property `ConvertClockRange` of AiFeatures is the range of sampling rate this card supports. When the sampling method (got by property `SamplingMethod` of AiFeatures) of the card is `EqualTimeSwitch`, the range of sampling rate that can be set for each channel is relevant to the number of sampling channel (property `ChannelCount` of `ScanChannel`) set currently. If the range got from `ConvertClockRange` is \[n,m], the range of sampling rate that can be set for each channel is \[n/Scan Channel Count, m/Scan Channel Count]. When the sampling method of the card is `Simultaneous`, the range of sampling rate that can be set for each channel is \[n,m] and it's not relevant to the number of sampling channel set currently.
注意：需要注意的是，AiFeatures 的 `ConvertClockRange` 屬性的值表示該音效卡支援的取樣率範圍。當音效卡的取樣方式（由 AiFeatures 的 `SamplingMethod` 屬性取得）為 `EqualTimeSwitch` 時，每個頻道可設定的取樣率範圍與目前設定的取樣頻道數（`ScanChannel` 的 `ChannelCount` 屬性）相關。如果 `ConvertClockRange` 的值為 \[n,m]，則每個通道可設定的取樣率範圍為 \[n/Scan Channel Count, m/Scan Channel Count]。當音效卡的取樣方式為 `Simultaneous` 時，每個頻道可設定的取樣率範圍也是 \[n,m]，與目前設定的取樣頻道數無關。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - ConvertClockSources
    - ConvertClockRange
- WaveformAiCtrl
    - Conversion
- Conversion
    - ClockSource
    - ClockRate

# Sampling Method

For high-speed AI data acquisition of multiple channels, there are two implementation approaches on the device hardware:
對於多通道高速 AI 資料擷取，設備硬體上有兩種實作方法：

1. One channel has one analog-to-digital conversion chip (ADC), such as AI hardware design of PCI-1714.
   一個通道配備一個類比數位轉換晶片（ADC），例如 PCI-1714 的 AI 硬體設計。

![[AdvantechDAQNaviSdk_OneChannelHasOneAdc.png]]

3. Multiple channels share one ADC, which can be distributed to each channel through a time-division multiplexer.
   多個通道共用一個 ADC，可以透過分時多工器將 ADC 分配給每個通道。

![[AdvantechDAQNaviSdk_MultipleChannelsShareOneAdc.png]]

According to two different designs, the commonly seen signal sampling methods of multiple channels are Simultaneous Sampling and EqualTimeSwitch Sampling.
根據兩種不同的設計，常見的多通道訊號取樣方法是同時取樣和等時切換取樣。

**Simultaneous Sampling**: Synchronous trigger signal (Convert Clock Source of the convert clock) is used to trigger each ADC to implement analog-to-digital conversion. One trigger signal can get data on all target channels.
**同步取樣**：使用同步觸發訊號（轉換時脈的轉換時脈源）觸發每個 ADC 進行類比數位轉換。一個觸發訊號可以獲得所有目標通道的資料。

Below is a set of diagrams for Simultaneous sampling, in which Scan Channel Count (In DAQNavi, this is set by property ChannelCount of ScanChannel) is respectively 1, 2 and N.
下面一組圖表用於同時取樣，其中掃描通道計數（在 DAQNavi 中，這是由 ScanChannel 的 ChannelCount 屬性設定的）分別為 1、2 和 N。

![[AdvantechDAQNaviSdk_SimultaneousSampling.png]]

**EqualTimeSwitch Sampling**: All channels share one ADC. Each signal that triggers a conversion will switch the ADC to another channel for sampling. Multiple channels can share one ADC by the way of time-division.
**等時切換取樣**：所有通道共用一個 ADC。每次觸發轉換的訊號都會將 ADC 切換到另一個通道進行取樣。多個通道可以透過時分復用的方式共用一個 ADC。

Below is a set of diagrams for EqualTimeSwitch sampling, in which Scan Channel Count (In DAQNavi, this is set by property `ChannelCount` of `ScanChannel`) is respectively 1, 2 and N.
下面一組是 EqualTimeSwitch 採樣的圖表，其中掃描通道計數（在 DAQNavi 中，這是由 `ScanChannel` 的 `ChannelCount` 屬性設定的）分別為 1、2 和 N。

![[AdvantechDAQNaviSdk_EqualTimeSwitchSampling.png]]

Remarks: No matter the device uses Simultaneous or EqualTimeSwitch method, the sampling rate of each channel is always set through property `Rate` of `ScanClock` in DAQNavi.
備註：無論裝置採用同步切換或等時切換方法，每個通道的取樣率始終透過 DAQNavi 中的 `ScanClock` 屬性 `Rate` 來設定。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - ConvertClockSources
    - ConvertClockRange
- WaveformAiCtrl
    - Conversion
- Conversion
    - ClockSource
    - ClockRate

# Burst Scan

Currently, the sampling methods of AD are Simultaneous and EqualTimeSwitch. In order to realize a more flexible sampling method, a scan clock has been added in the functional design of A/D, which is called Burst Scan.
目前，類比數位轉換器（AD）的取樣方式主要有同步取樣和等時切換取樣兩種。為了實現更靈活的取樣方式，類比數位轉換器的功能設計中增加了一個掃描時鐘，稱為突發掃描（Burst Scan）。

Burst scan of AI means in Buffered AI, when a scan clock signal comes, ADC will sample s specified amount of data according to the signal of convert clock. When the number of the samples has reached the number defined by scan count, the sampling will stop until next scan clock signal comes. The process, from the time a scan clock signal comes to start sampling to the time the sampling is stopped, is called a Acquisition Window. Convert clock in the acquisition window is an active clock that can trigger ADC to work. Scan Clock can be visualized as Gate (an enabling switch) to control Convert Clock. When Scan Clock is active, the clock of Convert Clock will be active to trigger sampling; conversely, when Scan Clock is not active, the sampling will not be triggered even if Convert Clock exists.
在緩衝式 AI（Buffered AI）中，突發掃描是指當掃描時脈訊號到達時，類比數位轉換器（ADC）會根據轉換時脈訊號取樣指定數量的資料。當取樣數達到掃描計數設定的值時，取樣將停止，直到下一個掃描時脈訊號到達。從掃描時脈訊號到達開始取樣到取樣停止的這段時間稱為擷取視窗 (Acquisition Window)。擷取視窗中的轉換時鐘是能夠觸發 ADC 工作的有效時鐘。掃描時鐘可以看作是控制轉換時鐘的門（啟用開關）。當掃描時脈有效時，轉換時脈也會觸發取樣；反之，當掃描時脈無效時，即使轉換時脈存在，也不會觸發取樣。

When the sampling method is Simultaneous, the schematic diagram of Burst Scan is shown as below:
當取樣方式為同步取樣時，突發掃描的示意圖如下：

![[AdvantechDAQNaviSdk_BurstScan_Simultaneous.png]]

When the sampling method is EqualTimeSwitch, the schematic diagram of Burst Scan is shown as below:
當採樣方法為等時切換時，突發掃描的原理圖如下所示：

![[AdvantechDAQNaviSdk_BurstScan_EqualTimeSwitch.png]]

The aim of designing scan clock is for instant sampling of large amount of data periodically.
掃描時脈的設計目的是為了週期性地對大量資料進行即時取樣。

Burst Scan has two important elements:
突發掃描包含兩個重要要素：

(1) **Scan Clock**: The clock signal that can trigger the hardware to start a large amount data acquisition.
(1) **掃描時鐘**：可觸發硬體開始大量資料擷取的時脈訊號。

(2) **Scan Count**: The size of the data to be acquired by each channel when a scan clock signal comes. The minimum value of Scan Count is 1.
(2) **掃描計數**：當接收到掃描時脈訊號時，每個通道需要擷取的資料量。掃描計數的最小值為 1。

All scan clock signals will be ignored before Burst Scan reaches the acquisition number defined by Scan Count. After the acquisition number defined by Scan Count is reached, next Scan Clock will start next Burst Scan.
在突發掃描達到由掃描計數定義的擷取次數之前，所有掃描時脈訊號都將被忽略。達到由掃描計數定義的擷取次數後，下一個掃描時脈將啟動下一次突發掃描。

Note: When the speed of Scan Clock is the same as that of Convert Clock and Scan Count is 1, the effect will be the same as that only has Clock but no Burst Clock.
注意：當掃描時鐘的速度與轉換時鐘的速度相同，且掃描計數為 1 時，其效果與只有時鐘而沒有突發時鐘的情況相同。

In DAQNavi, Burst Scan related areas are as followings:
在 DAQNavi 中，突發掃描相關區域如下：

(1) Property of Features of BufferedAiCtrl - BufferedAiCtrl 的特徵屬性

`BurstScanSupported` can get whether this device supports Burst Scan function.
`BurstScanSupported` 可取得該裝置是否支援連拍掃描功能。

`ScanClockSource` can get the clock source that supports Burst Scan.
`ScanClockSource` 可以取得支援突發掃描的時鐘來源。

`ScanClockRange` can get the range of Scan Clock Rate.
`ScanClockRange` 可以取得掃描時脈速率的範圍。

`ScanCountMax` can get the max sampling count of each channel when Scan Clock comes each time that this device supports. For example, if the return value of `getScanCountMax()` is 1000, the value you set for Scan Count can not exceed 1000.
`ScanCountMax` 函數可取得裝置支援的每個掃描時脈週期內各通道的最大取樣次數。例如，如果 `getScanCountMax()` 的回傳值為 1000，則您設定的掃描計數值不能超過 1000。

(2) Property of ScanClock of BufferedAiCtrl - BufferedAiCtrl 的 ScanClock 屬性

Source is used to set or read the clock source of Burst Scan. When `ScanClockSource` is set to None, Burst Scan function will not be enabled.
`ScanClockSource` 用於設定或讀取突發掃描的時鐘來源。當 `ScanClockSource` 設定為 `None` 時，突發掃描功能將不會啟用。

`Rate` is used to set or read Clock Rate of Burst Scan.
`Rate` 用於設定或讀取突發掃描的時脈速率。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - BurstScanSupported
    - ScanClockSources
    - ScanClockRange
    - ScanCountMax
- BufferedAiCtrl
    - ScanClock
- ScanClock
    - Source
    - Rate
    - ScanCount

# Temperature Measurement

Temperature Measurement is an common application of DAQ devices. Two read-out values from AI channels are necessary for measuring a temperature value. These two values will be converted into temperature values using different conversion functions according to different types of thermocouples.
溫度測量是數據採集設備常見的應用之一。測量溫度需要從 AI 通道取得兩個讀數。根據熱電偶的類型，這兩個讀數將使用不同的轉換函數轉換為溫度值。

The measuring principle of thermocouple makes use of the thermal voltage effect of the metal. Different metals support different thermal voltages under different temperatures. One of the ends of two metals can be coupled as temperature sensing terminal while the other ends of two metals are separate as two measuring terminals, so when the temperature on sensing terminal is different from that on measuring terminal, there will be a voltage difference, which can be used to convert temperature difference between sensing and measuring terminal. Different metal combinations will have different curves for temperature and thermal voltage, and different thermocouples will have different fitting curves for voltage difference and temperature difference.
熱電偶的測量原理是利用金屬的熱電壓效應。不同的金屬在不同溫度下會產生不同的熱電壓。兩根金屬的一端可以連接成溫度感測端，另一端則分別作為兩個測量端。當感測端和測量端的溫度不同時，就會產生電壓差，該電壓差可用於轉換感測端和測量端之間的溫差。不同的金屬組合具有不同的溫度 - 熱電壓曲線，不同的熱電偶也具有不同的電壓差 - 溫差擬合曲線。

Voltage difference can only help to convert temperature difference. If you want to measure the temperature of a measuring point, you need to know the temperature value of one terminal to calculate the temperature value of the other terminal, which is the temperature of cold junction (CJC).
電壓差只能用於轉換溫差。如果要測量某個測量點的溫度，就需要知道一個端子的溫度值才能計算出另一個端子的溫度值，也就是冷端溫度（CJC）。

Here will introduce some DAQNavi related parameters temperature measurement function uses, including Thermocouple Type, cold junction (CJC) temperature, AI Channel (CJC Source), temperature measuring frequency (CJC update frequency), etc.
這裡將介紹一些與 DAQNavi 溫度測量功能相關的參數，包括熱電偶類型、冷端 (CJC) 溫度、AI 通道（CJC 源）、溫度測量頻率（CJC 更新頻率）等。

How to decide whether the device supports temperature measurement function? How to set related parameters in the program?
如何判斷設備是否支援溫度測量功能？如何在程式中設定相關參數？

You can decide whether the device supports temperature measurement function by referring to hardware manual or through property CJC of InstantAiCtrl. You can also get channels that support this function through property CjcChannels of AiFeatures. No channel means this device does not support temperature measurement function.
您可以查閱硬體手冊或查看 InstantAiCtrl 的 CJC 屬性來判斷設備是否支援溫度測量功能。您也可以透過 AiFeatures 的 CjcChannels 屬性來取得支援此功能的通道。如果沒有通道，則表示該設備不支援溫度測量功能。

If the device supports this function, you can follow the below steps to complete temperature related configurations.
如果設備支援此功能，您可以按照以下步驟完成與溫度相關的配置。

1. Get temperature measurement related channels through property `CjcChannels` of AiFeatures.
   透過 AiFeatures 的 `CjcChannels` 屬性取得溫度測量相關通道。

2. Get a `CjcSetting` object through property `Cjc` of InstantAi, then set Cjc related properties through the object of CjcSetting, such as `Cjc Channel`, `CjcValue` and etc.
   透過 InstantAi 的 `Cjc` 屬性取得 `CjcSetting` 對象，然後透過 CjcSetting 物件設定與 Cjc 相關的屬性，例如 `Cjc Channel`、`CjcValue` 等。

3. Set or read the channels measuring CJC temperature of the device through property `Channel` of `CjcSetting`, i.e. which supported channels can be used for measuring temperature or which channels are currently used for measuring temperature.
   透過 `CjcSetting` 的 `Channel` 屬性設定或讀取測量設備 CJC 溫度的通道，即哪些支援的通道可用於測量溫度，或目前哪些通道用於測量溫度。

4. Set or read the temperature value of CJC that has been set through property `Value` of `CjcSetting`.
   設定或讀取透過 `CjcSetting` 的 `Value` 屬性設定的 CJC 的溫度值。

5. Set update frequency of CJC value through a property. In SDK, this property will not be introduced and currently this property is only supported in "Device Configuration" page.
   透過屬性設定 CJC 值的更新頻率。此屬性在 SDK 中不會引入，目前僅在「裝置配置」頁面中支援。

6. Read the temperature value of Celsius temperature scale (either Raw Data or Scaled Data) through property `Read` of InstantAiCtrl.
   透過 InstantAiCtrl 的 `Read` 屬性讀取攝氏溫度標度的溫度值（原始資料或縮放資料）。

The above information shows you how to complete configurations through properties. Besides, you can also complete parameter configurations through "Device Configuration" or "Device Setting" of Navigator. Here "Device Setting" of PCI-1710HG is taken as an example to illustrate how to complete configurations related to temperature measurement.
以上資訊展示如何透過屬性完成配置。此外，您也可以透過導覽器的「裝置配置」或「裝置設定」完成參數配置。這裡以 PCI-1710HG 的「設備設定」為例，說明如何完成與溫度測量相關的配置。

Navigator 使用設定請參見手冊。

For more information, you can refer to user manual of other data acquisition cards that support temperature measurement, such as PCI-1710 and USB-4718.
有關更多信息，您可以參考其他支援溫度測量的數據採集卡的用戶手冊，例如 PCI-1710 和 USB-4718。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - CjcChannels
- InstantAiCtrl
    - Cjc
- CjcSetting
    - Channel
    - Value

# Burn Out Detected

Burn Out Detected is used to detect whether the signal channel loop of AI channel is normally connected. If a burn out is detected, the specified data type that has been set will be returned. The data types include:
「燒毀檢測」用於檢測 AI 通道的訊號通道環路是否正常連接。如果偵測到燒毀，則會傳回已設定的指定資料類型。資料類型包括：

1. Do not detect burn out, but directly return read-out value;
   不檢測燒毀情況，直接回傳讀取值；
2. Set a particular value and return it;
   設定一個特定值並傳回該值；
3. Return the maximum value of current gain;
   傳回當前增益的最大值；
4. Return the minimum value of current gain;
   傳回當前增益的最小值；
5. Return the last effective value.
   傳回最後一個有效值。

The above data types corresponds to the content in enumeration `BurnoutRetType` {Current = 0, `ParticularValue`, `UpLimit`, `LowLimit`, `LastCorrectValue`} in DAQNavi.
上述資料型態對應於 DAQNavi 中枚舉 `BurnoutRetType` {Current = 0, `ParticularValue`, `UpLimit`, `LowLimit`, `LastCorrectValue`} 的內容。

In DAQNavi:

You can get the supported return data types of the device through property `BurnoutReturnTypes` of `AiFeatures`.
您可以透過 `AiFeatures` 的 `BurnoutReturnTypes` 屬性來取得裝置支援的回傳資料類型。

You can set or read current return data types through property `BurnoutRetType` of `AnalogInputChannel`.
您可以透過 `AnalogInputChannel` 的 `BurnoutRetType` 屬性設定或讀取目前回傳資料類型。

You can set or get the burn out value when burn out is detected through property `BurnoutRetValue` of `AnalogInputChannel`.
您可以透過 `AnalogInput Channel` 的 `Burnout GetValue` 屬性，在偵測到燒毀時設定或取得燒毀值。

The above information shows you how to complete configurations through properties. For data acquisition card that supports burn out detection function, you can also complete parameter configurations through "Device Configuration" or "Device Setting" of Navigator. Here "Device Setting" of USB-4718 is taken as an example to illustrate how to complete configurations related to burn out detection.
以上資訊展示如何透過屬性完成配置。對於支援燒毀偵測功能的資料擷取卡，您也可以透過導航器的「裝置配置」或「裝置設定」完成參數配置。這裡以 USB-4718 的「裝置設定」為例，說明如何完成與燒毀偵測相關的設定。

Navigator 使用設定請參見手冊。

For more information, you can refer to user manual of USB-4718 or other data acquisition card that supports burn out detection function.
有關更多信息，您可以參考 USB-4718 或其他支援燒毀檢測功能的數據採集卡的用戶手冊。

## DAQNavi References

- AiCtrlBase
    - Features
- AiFeatures
    - BurnoutReturnTypes
- AiCtrlBase
    - Channels
- AiChannel
    - BurnoutRetType
    - BurnoutRetValue

# IEPE

IEPE stands for "integrated electronics piezoelectric".
IEPE 代表「集成電子壓電元件」。

If you attach an IEPE accelerometer or microphone that requires excitation to an AI channel, you must enable the IEPE excitation circuitry for that channel to generate the required excitation current. You can independently configure IEPE signal conditioning on a per channel basis.
如果將需要激勵的 IEPE 加速度計或麥克風連接到 AI 通道，則必須啟用該通道的 IEPE 激勵電路以產生所需的激磁電流。您可以針對每個通道獨立設定 IEPE 訊號調理。

A DC voltage offset is generated equal to the product of the excitation current and sensor impedance when IEPE signal conditioning is enabled. To remove the unwanted offset, enable AC coupling. DC coupling can be used with IEPE excitation enabled without a loss of signal integrity only if the offset plus the peak of the AC signal of interest does not exceed the voltage range of the channel.
**啟用 IEPE 訊號調理時，會產生一個直流電壓偏移**，其大小等於激磁電流與感測器阻抗的乘積。若要消除此偏移，請啟用**交流耦合**。只有當偏移量加上目標交流訊號的峰值不超過通道的電壓範圍時，才能在啟用 IEPE 激磁的情況下使用直流耦合而不會損失訊號完整性。

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - IepeTypes
- AiChannel
    - IepeType

# Coupling

Some devices can be configured as either AC or DC coupling. If DC coupling is selected, any DC offset present in the source signal is passed to the ADC. The DC-coupling configuration is usually best if the signal source has only small amounts of offset voltage or if the DC content of the acquired signal is important. If the source has a significant amount of unwanted offset, select AC coupling to take full advantage of the input dynamic range.
某些設備可以配置為交流耦合或直流耦合。如果選擇直流耦合，則來源訊號中存在的任何直流偏移都會傳遞給類比數位轉換器 (ADC)。如果訊號源的偏移電壓很小，或者擷取訊號的直流成分很重要，則直流耦合配置通常是最佳選擇。如果訊號源存在較大的非預期偏移，則應選擇交流耦合，以充分利用輸入動態範圍。

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - CouplingTypes
- AiChannel
    - CouplingType

# Impedance

In General users use measuring equipment with high impedance to avoid measurement error. i.e. the 1MΩ input impedance when probing a circuit.
通常情況下，使用者會**使用高阻抗的測量設備來避免測量誤差**，例如，在偵測電路時使用 1MΩ的輸入阻抗。

However, if you are connecting to an equivalent 50 ohm output impedance of a source, 50 ohm low impedance input mode is needed. For example a signal generator with a 50 ohm output or an RF signal generator with 50 ohm characteristic output impedance. When using the 50 ohm low impedance input mode, you will need to use a 50Ω coax too. We always want our source, line and load to be impedance matched to minimize reflections that will add or subtract from the true signal we are measuring.
但是，如果您要連接到等效 50 歐姆輸出阻抗的訊號源，則需要使用 50 歐姆低阻抗輸入模式。例如，輸出阻抗為 50 歐姆的訊號產生器或特性輸出阻抗為 50 歐姆的射頻訊號產生器。使用 50 歐姆低阻抗輸入模式時，您還需要使用 50Ω 的同軸電纜。我們始終希望訊號源、傳輸線和負載的阻抗匹配，以最大限度地減少反射，**避免反射對我們所測量的真實訊號產生影響**。

Furthermore, the high impedance mode adds a lot of capacitance to our circuit (parasitic capacitance) at the point of probing, and this excess capacitance certainly can and will distort our circuit, especially at high frequencies. You will need to consider this when deciding to use high input impedance or the other.
此外，**高阻抗模式會在探測點處為電路增加大量電容（寄生電容）**，而這種額外的電容必然會使電路失真，尤其是在高頻下。在決定使用高輸入阻抗模式還是其他模式時，您需要考慮這一點。

In short, most work with high-frequency will want to use the lower impedance, lower capacitance mode with a matched impedance signal chain.
簡而言之，大多數高頻應用都需要使用低阻抗、低電容模式以及匹配阻抗的訊號鏈。

## DAQNavi References

- AiCtrlBase
    - Channels
    - Features
- AiFeatures
    - ImpedanceTypes
- AiChannel
    - ImpedanceType

# Gain

Gain refers to the ability to increase the signal power or amplitude. Gain is mainly used to amplify the attenuated signal before the signal is digitalized.
增益是指增強訊號功率或幅度的能力。增益主要用於在訊號數位化之前放大衰減的訊號。

![[AdvantechDAQNaviSdk_Gain.png]]

Gain can help to equivalently reduce A/D input range to divide the signal into as many equal parts as possible, basically covering the full range so as to better restore the signal. For the same voltage input range, the quantization error of large signal is small while that of small signal is big. When the input signal does not cover full range, the quantization error will be relatively big. Advantech DAQ devices realize gain through setting Value Range (the output range of voltage and current), and automatically choose the size of gain through choosing different value ranges.
增益可以等效地縮小 A/D 輸入範圍，將訊號盡可能分成多個相等的部分，基本上覆蓋整個範圍，從而更好地還原訊號。對於相同的電壓輸入範圍，大訊號的量化誤差較小，而小訊號的量化誤差較大。當輸入訊號未覆蓋整個範圍時，量化誤差會相對較大。研華 DAQ 裝置透過設定數值範圍（電壓和電流的輸出範圍）來實現增益，並根據選擇不同的數值範圍自動選擇適當的增益大小。

The resolution, range and gain of a data acquisition card decides the distinguishable minimum voltage, i.e. LSB.
資料擷取卡的分辨率、範圍和增益決定了可區分的最小電壓，即 LSB。

## DAQNavi References

- AiCtrlBase
    - Features
    - Channels
- AiFeatures
    - ValueRanges
    - OverallValueRange
- AiChannel
    - ValueRange

# Analog Input Events

Event refers to the response that driver gives to actively inform the user of some specified situations or the response to the executed detection. Below will introduce several important concepts related to events of AI.
事件是指驅動程式主動向使用者告知特定情況或對已執行偵測的回應。以下將介紹與 AI 事件相關的幾個重要概念。

## 1. DataReady

In the process of Buffered AI data acquisition, when the number of the sample data reaches the number defined by `IntervalCount`, driver will send out this event to inform the user that User APP has finished the data acquisition of a Section and the user can start to read the data. This process is shown as below:
在緩衝式 AI 資料收集過程中，當樣本資料數量達到 `IntervalCount` 設定的值時，驅動程式會發出此事件通知用戶，用戶 APP 已完成一個區塊的資料收集，用戶可以開始讀取資料。過程如下所示：

![[AdvantechDAQNaviSdk_AnalogInputEvents_DataReady.png]]

1. When AI acquisition starts, data will be filled into buffer. The set `IntervalCount` can divide buffer into a number of sections.
   AI 採集開始時，資料將被填入緩衝區。設定的 `IntervalCount` 參數可以將緩衝區分割成若干個區間。

2. When the first Section in the buffer has been fully filled with data, driver will send out `DataReady` event to User APP to inform the user that new data has arrived and can be read.
   當緩衝區中的第一個分割區完全被資料填滿時，驅動程式將向使用者應用程式發送 `DataReady` 事件，以通知使用者新資料已到達並可讀取。

3. Then, the acquisition continues and you can get the data in previous section. When next section is fully filled with data, `DataReady` event will be sent out again to inform the user that new data has arrived and can be read. This process will continue until the acquisition is complete.
   然後，數據採集繼續進行，您可以獲得上一部分的數據。當下一部分資料完全填入後，系統會再次傳送 `DataReady` 事件，通知使用者新資料已到達，可以讀取。此過程將持續進行，直到資料採集完成。

In DAQNavi, event `DataReady` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `DataReady` 事件來處理此事件。

## 2. Overrun

Driver will send out this event to inform User APP that unprocessed data has been overwritten.
驅動程式將發送此事件以通知使用者 APP 未處理的資料已被覆寫。

If new acquired data overwrites unprocessed data in asynchronous high-speed AI acquisition, the processing speed of User APP may be too slow.
在非同步高速 AI 採集過程中，如果新採集的數據覆蓋了未處理的數據，則用戶 APP 的處理速度可能會太慢。

![[AdvantechDAQNaviSdk_AnalogInputEvents_Overrun.png]]

1. When AI acquisition starts, data will be filled into buffer.
   AI 採集開始時，數據將被填充到緩衝區中。

2. When the first section has been fully filled with data, `DataReady` event will be sent out. Then some data has not been got by the user or not been got in time due to high acquisition speed.
   當第一個資料部分完全填入後，會發出 `DataReady` 事件。此時，可能由於資料擷取速度過快，導致部分資料尚未被使用者取得或未能及時取得。

3. Data is continuously filled into next section.
   數據將持續填充到下一部分。

4. When the buffer is fully filled with data and the user still does not get the data: if the data acquisition is non cyclical, AI acquisition stops; if the acquisition is cyclical, driver will continue to fill data into the first section.
   當緩衝區已完全被資料填滿，而使用者仍然沒有獲得資料時：如果資料擷取是非循環的，則 AI 擷取停止；如果擷取是循環的，則驅動程式將繼續向第一部分填入資料。

5. When data is filled into the first section again and the old data has been overwritten by the data that has not been got by the user, driver will send out `Overrun` event to User APP to inform the user that some unprocessed data has been overwritten.
   當資料再次填入第一部分，而舊資料已被使用者尚未取得的資料覆蓋時，驅動程式會向使用者應用程式發送 `Overrun` 事件，以通知使用者某些未處理的資料已被覆寫。

In DAQNavi, event `Overrun` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `Overrun` 事件來處理此事件。

## 3. CacheOverflow

Driver will send out this event to inform User APP that buffer data overflows on-board.
驅動程式將發送此事件以通知用戶 APP 板上緩衝區資料溢位。

If asynchronous high-speed AI acquisition is adopted, overflow occurs when data in register for Buffered AI overflows on the device. The reason is that sampling speed is so faster than data transmission speed that the amount of accumulated data has exceed the buffer size. Since this kind of data loss occurs on hardware, unlike `Overrun`, you will not know how much data has been lost.
如果採用非同步高速 AI 採集，當裝置上用於緩衝 AI 的暫存器中的資料溢位時，就會發生溢位。這是因為取樣速度遠快於資料傳輸速度，導致累積的資料量超過了緩衝區大小。由於這種資料遺失發生在硬體層面，與 `Overrun` 不同，您無法得知遺失了多少資料。

As shown in the figure below, normally an interrupt will be sent out when half FIFO on the device has been fully filled. After driver receives this interrupt, data will be transmitted from FIFO to user buffer for further analysis.
如下圖所示，通常情況下，當裝置上的 FIFO 緩衝區一半被填滿時，會發出中斷。驅動程式收到此中斷後，會將資料從 FIFO 緩衝區傳輸到用戶緩衝區以進行進一步分析。

![[AdvantechDAQNaviSdk_AnalogInputEvents_CacheOverflow_1.png]]

However, if sampling speed is faster than transmission speed, data will be accumulated in FIFO. When the amount of accumulated data exceeds buffer size of FIFO, driver will send out `CacheOverflow` event, which is shown as below:
但是，如果取樣速度快於傳輸速度，資料就會累積在 FIFO 緩衝區中。當累積的資料量超過 FIFO 緩衝區的大小時，驅動程式會發出 `CacheOverflow` 事件，如下所示：

![[AdvantechDAQNaviSdk_AnalogInputEvents_CacheOverflow_2.png]]

In DAQNavi, event `CacheOverflow` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `CacheOverflow` 事件來處理此事件。

## 4. Stopped

Driver will send out this event to inform User APP that Buffered AI data acquisition has been finished.
驅動程式將會傳送此事件以通知使用者 APP 緩衝 AI 資料擷取已完成。

This event will be sent out when Buffered AI has been executed or has been stopped by an interrupt.
當 Buffered AI 執行完畢或因中斷而停止時，將發出此事件。

In DAQNavi, event `Stopped` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `Stopped` 事件來處理此事件。

## 5. BurnOut

Driver will send out this event to inform User APP that BurnOut has occurred.
驅動程式將發送此事件以通知使用者 APP 已發生「BurnOut」事件。

`BurnOut` Event is a notification indicating that HW self-protection mechanism has been launched. When BufferedAi is running, if the input voltage is greater then a certain multiple of the measurement range, the device will launch self-protection mechanism to disconnect the measured signal to avoid hardware damage.
`BurnOut` 事件是硬體自保護機制啟動的通知。當 BufferedAi 運作時，如果輸入電壓大於測量範圍的某個倍數，設備將啟動自保護機制，斷開被測訊號，以避免硬體損壞。

In DAQNavi, event `BurnOut` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `BurnOut` 事件來處理此事件。

## 6. TimeStampOverrun

Driver will send out this event to inform User APP that unprocessed timeStamp information has been overwritten.
驅動程式將發送此事件以通知使用者 APP 未處理的時間戳記資訊已被覆寫。

When performing retrigger function, if very small amount of data is included in each record and sampling rate is too fast, it will cause that timeStamp information is generated too fast which led to overrun.
執行重新觸發功能時，如果每筆記錄中包含的資料量非常小，且取樣率過快，則會導致時間戳資訊產生過快，從而導致溢位。

In DAQNavi, event `TimeStampOverrun` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `TimeStampOverrun` 事件來處理此事件。

## 7. TimeStampCacheOverflow

Driver will send out this event to inform User APP that timestamp buffer overflows on-board.
驅動程式將發送此事件以通知用戶 APP 車載時間戳緩衝區溢位。

The root cause of timeStampCacheOverflow is that the timeStamp information is generated too fast and the transmitted speed of bus is not fast enough.
timeStampCacheOverflow 的根本原因是 timeStamp 資訊產生速度過快，而匯流排傳輸速度不夠快。

In DAQNavi, event `TimeStampCacheOverflowof` BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用事件 `TimeStampCacheOverflowof` BufferedAiCtrl 來處理此事件。

## 8. MarkOverrun

Driver will send out this event to inform User APP that unprocessed mark information has been overwritten.
驅動程式將發送此事件通知用戶 APP 未處理的標記訊息已被覆蓋。

If the frequency of Mark trigger is too high, the mark information will be generated too fast and cause the mark buffer overwritten.
如果標記觸發頻率過高，標記資訊產生速度過快，會導致標記緩衝區被覆蓋。

In DAQNavi, event `MarkOverrun` of BufferedAiCtrl is used to handle this event.
在 DAQNavi 中，使用 BufferedAiCtrl 的 `MarkOverrun` 事件來處理此事件。

## DAQNavi References

- WaveformAiCtrl
    - DataReady
    - Overrun
    - CacheOverflow
    - Stopped
    - BurnOut
    - TimeStampOverrun
    - TimeStampCacheOverflow
    - MarkOverrun

# Instant and Buffered

Data transmission modes are divided into "Instant" and "Buffered". Instant is the real-time reflection of current phenomenon. From data perspective, Instant data is a single data, which is a real-time data. Buffered means data is firstly stored in memory temporarily, then will be transmitted to computer in a large amount for processing. From data feature perspective, Buffered data is an array, which is a time series. In function description, Instant refers to low speed AI. For example, the sampling frequency of Instant AI is comparatively low, accordingly the sampled signal frequency will also be low. Buffered AI refers to high speed AI. The design of Buffered is more complex. Data will be firstly stored in buffer, therefore many problems will be encountered, such as the transmission definition and method of previous and later section of data in Buffer, trigger, interrupt, etc. The following will give you a detailed introduction of AI and Buffered.
資料傳輸模式分為「即時」和「緩衝」兩種。即時模式是對當前現象的即時反映。從數據角度來看，即時數據是單一數據，即即時數據。緩衝模式是指資料先暫時儲存在記憶體中，然後再批次傳送到電腦進行處理。從資料特徵來看，緩衝資料是一個數組，即時間序列。在功能描述中，即時模式指的是低速人工智慧。例如，即時模式人工智慧的取樣頻率相對較低，因此取樣訊號的頻率也較低。緩衝模式人工智慧指的是高速人工智慧。緩衝模式的設計更為複雜。資料首先儲存在緩衝區中，因此會遇到許多問題，例如緩衝區中前後資料段的傳輸定義和方法、觸發、中斷等。以下將詳細介紹人工智慧和緩衝模式。

## Instant AI

### What is Instant AI?

Instant AI refers to sampling mode of the software for analog data. The software sends out a command to trigger the data transmission so as to immediately read the data on AD channel. The signal frequency of Instant AI sampling is comparatively low, therefore it is also called low speed AI which is used for low speed sampling of less data.
即時 AI 是指軟體對模擬資料的一種採樣模式。軟體會發出指令觸發資料傳輸，以便立即讀取 AD 通道上的資料。即時 AI 採樣的訊號頻率相對較低，因此也稱為低速 AI，用於對少量資料進行低速取樣。

### Instant AI Trigger Mode

In terms of trigger mode, Buffered AI adopts hardware trigger, while Instant AI adopts software trigger.
在觸發模式方面，緩衝式 AI 採用硬體觸發，而即時式 AI 則採用軟體觸發。

The operation flow of Instant AI is shown as below:
Instant AI 的運作流程如下所示：

![[AdvantechDAQNaviSdk_Flowcharts_Instant AI.png]]

### DAQNavi References

- InstantAiCtrl

## Buffered AI

### What is Buffered AI?

Buffered AI refers to the high speed sampling mode of analog data in buffer which is used for high speed sampling of a large amount of data. Since Buffered AI uses buffer, you should set the sampling count, i.e. the size of sampling buffer, before you starts high speed sampling. The acquired data will be firstly transmitted to sampling buffer. You can further process the data according to your requirements. Buffered AI can be divided into synchronous data transmission, asynchronous data transmission, one buffered data acquisition and streaming data acquisition.

### Buffered AI Trigger Mode

In terms of trigger mode, Buffered AI belongs to hardware trigger.

### How to Decide Data Buffer?

Data Buffer is the buffer where the acquired data is stored. Data Buffer is assigned from the system buffer by the driver, so User APP can use it directly.

### Buffered AI Types

According to the work mode between User APP and the driver, Buffered AI can be divided into two types: Asynchronous and Synchronous. According to the data size that has been acquired, Buffered AI can be divided into One Buffered and Streaming. In conclusion, Buffered AI has the following types: Synchronous One Buffered AI, Asynchronous One Buffered AI and Streaming AI.

#### Asynchronous

Asynchronous means User APP runs asynchronously with Driver. When User APP calls a service provided by Driver, after this function is started User APP will immediately return and continue to execute other tasks. After the service is completed, Driver will send out an Event or set a Flag to inform the caller of the operation status of the service. This operation mode allows User APP to execute other tasks while waiting for execution results, so as to give full play to the advantages of a multi-task operation system. Below shows the working principle of Asynchronous.

![[AdvantechDAQNaviSdk_BufferedAiType_Asynchronous.png]]

#### Synchronous

Synchronous means User APP runs synchronously with Driver. When User APP calls a service provided by Driver, User APP will return after the service is completed, then continue to perform other User APP tasks. Below shows the working principle of Synchronous.

![[AdvantechDAQNaviSdk_BufferedAiType_Synchronous.png]]

Below are use flows of Synchronous One Buffered AI, Asynchronous One Buffered AI and Streaming AI.

![[Synchronous]]

Below are use flows of Synchronous One Buffered AI and Asynchronous One Buffered AI.

Below is use flow of Streaming AI.

DAQNavi References:

WaveformAiCtrl
