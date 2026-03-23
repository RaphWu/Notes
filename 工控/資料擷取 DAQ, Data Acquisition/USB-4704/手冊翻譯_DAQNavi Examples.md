---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

DAQNavi examples included in DAQNavi SDK package is programming examples, aiming to help you get started developing an application with DAQNavi SDK. You can modify the example code and save it in an application. Also you can use the examples to develop a new application.
DAQNavi SDK 軟體包中包含的 DAQNavi 範例是程式設計範例，旨在協助您快速上手使用 DAQNavi SDK 開發應用程式。您可以修改範例程式碼並將其儲存到應用程式中。此外，您還可以使用這些範例開發新的應用程式。

First, please install DAQNavi SDK from the CD-ROM. Examples for DAQNavi SDK are in the System disk\Advantech\DAQNavi\Examples directory. For detailed information about DAQNavi examples, please refer to DAQNavi SDK manual. DAQNavi SDK provides two kinds of examples: DAQNavi Class Library Examples and DAQNavi Control Examples.
首先，請從 CD-ROM 安裝 DAQNavi SDK。 DAQNavi SDK 的範例位於系統磁碟\Advantech\DAQNavi\Examples 目錄中。有關 DAQNavi 範例的詳細信息，請參閱 DAQNavi SDK 手冊。 DAQNavi SDK 提供兩種類型的範例：DAQNavi 類別庫範例和 DAQNavi 控制項範例。

Here is the list of the examples supported by USB-4704:

| Example Name                 | Description                                                                                                                                           |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| AI_Instant                   | Retrieves data of several AI channel inputs through Instant method.<br>透過 Instant 方法檢索多個 AI 通道輸入的資料。                                                  |
| AI_AsynchronousOneBufferedAI | Retrieves finite massive data of several AI channel inputs repeatedly through asynchronous buffered method.<br>透過非同步緩衝方法重複檢索多個 AI 通道輸入的有限海量資料。        |
| AI_SynchronousOneBufferedAI  | Retrieves massive data of several AI channel inputs repeatedly through synchronous buffered method.<br>透過同步緩衝方式重複讀取多個 AI 通道輸入的大量資料。                   |
| AI_StreamingAI               | Retrieves infinite massive data of several AI channel inputs repeatedly through asynchronous buffered method.<br>透過非同步緩衝方式重複讀取多個 AI 通道輸入的無限海量資料。      |
| AO_StaticAO                  | Signal output to AO channel through single data by software method.<br>透過軟體方式，以單一資料形式向 AO 頻道輸出訊號。                                                     |
| DI_StaticDI                  | Reads a DI port input repeatedly through Instant method and shows the result.<br>以即時方式重複讀取 DI 連接埠輸入並顯示結果。                                             |
| DO_StaticDO                  | Writes the output state value of a DO port according to the hex value input by the user through Instant method<br>透過即時方式，根據使用者輸入的十六進位值，寫入 DO 埠的輸出狀態值。 |
| Counter_EventCounter         | Counts the times of pulse produced by external digital signals.<br>統計外部數位訊號產生的脈衝次數。                                                                   |
| Counter_FrequencyMeasurement | Counts the frequency value of input signals.<br>統計輸入訊號的頻率值。                                                                                           |
