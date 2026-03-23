---
aliases:
language:
date: 2025-01-21
author: PicoExpert
sourceurl: https://www.picotech.com/library/knowledge-bases/oscilloscopes/modbus-serial-protocol-decoding
tags:
  - Modbus
---

# Modbus® serial protocol decoding

## Introduction

Modbus is a low-speed serial data protocol commonly used in industrial applications where a supervisory computer (master) controls or monitors multiple remote devices (slaves).
Modbus 是一種低速串列資料協議，常用於工業應用，其中監控電腦（主機）控製或監控多個遠端設備（從機）。

The specification was originally published in 1979 by Modicon (now Schneider Electric) for use with its programmable logic controllers (PLCs).
該規範最初於 1979 年由 Modicon（現為施耐德電氣）發布，用於其可程式邏輯控制器 (PLC)。

In a standard Modbus network there is one master, and up to 247 slaves each with a unique address from 1 to 247.
在標準 Modbus 網路中，有一個主站和最多 247 個從站，每個從站都有一個從 1 到 247 的唯一位址。

PicoScope (Beta) software provides support for Modbus RTU and Modbus ASCII.
PicoScope（測試版）軟體支援 Modbus RTU 和 Modbus ASCII。

## Modbus protocol versions

Several versions of Modbus have been developed to suit the transmission medium being used. Most common are:
目前已開發了多個版本的 Modbus，以適應所使用的傳輸介質。最常見的是：

- **Modbus RTU** (Remote Terminal Unit) – typically for use over RS-232 single-ended or RS-485 differential lines, uses binary coding and CRC error checking.
  **Modbus RTU**（遠端終端單元）－通常用於 RS-232 單端或 RS-485 差分線路，使用二進位編碼和 CRC 錯誤校驗。
- **Modbus ASCII** – also for use over RS-232 or RS-485 lines, uses ASCII characters instead of binary, making it more readable but less efficient, and it uses less effective LRC error checking. ASCII mode uses ASCII characters to begin and end messages whereas RTU uses time gaps of 3.5 character times for framing. Modbus ASCII messages require twice as many bytes to transmit the same content as a Modbus RTU message.
  **Modbus ASCII** – 也適用於 RS-232 或 RS-485 線路，使用 ASCII 字元而非二進制，使其更易讀但效率較低，並且使用效率較低的 LRC 錯誤校驗。 ASCII 模式使用 ASCII 字元作為訊息的開始和結束，而 RTU 使用 3.5 個字元時間的時間間隔進行訊框傳輸。與 Modbus RTU 訊息相比，Modbus ASCII 訊息需要兩倍的位元組數來傳輸相同的內容。
- **Modbus TCP** – for use over TCP/IP networks, typically Ethernet, (not currently supported by PicoScope).
  **Modbus TCP** – 用於 TCP/IP 網絡，通常是乙太網路（目前不受 PicoScope 支援）。

## Modbus frame structure

The Modbus protocol defines a Protocol Data Unit (PDU), which is independent of the underlying communication layers. Additional fields may be introduced in the Application Data Unit (ADU) depending on the type of bus or network employed.
Modbus 協定定義了一個協定資料單元 (PDU)，它獨立於底層通訊層。根據所採用的匯流排或網路類型，可以在應用程式資料單元 (ADU) 中引入其他欄位。

![[ModbusSerialProtocolDecoding_modbus-frame-structure.jpg]]

**Protocol Data Unit (PDU)** contains:
- The Function Code that indicates the kind of action to be performed.
  指示要執行的操作類型的功能代碼。
- The Data Field of frames sent from a master to slave devices that contains additional information about action defined by the function code. This can include items like discrete and register addresses, the quantity of items to be handled, and the count of actual data bytes in the field. The data field may be nonexistent (of zero length) in certain kinds of requests.
  主設備向從設備發送的幀的資料字段，包含由功能代碼定義的操作的附加資訊。這些資訊可能包括離散量和暫存器位址、待處理資料項目的數量以及欄位中實際資料位元組數。在某些類型的請求中，資料欄位可能不存在（長度為零）。

**Application Data Unit (ADU)** contains:
- The Protocol Data Unit (PDU)
- The Slave ID
- The CRC Error Check

**Error Codes** – When the server responds to the client it uses the function code field to indicate either a normal (error-free) response or that some kind of error occurred, called an exception response. For a normal response the server simply echoes the original function code and returns the data requested.
錯誤代碼 – 當伺服器回應客戶端時，它使用功能代碼欄位來指示回應是正常（無錯誤）還是發生了某種錯誤（稱為異常回應）。對於正常回應，伺服器只需回顯原始功能代碼並傳回請求的資料。

## Data storage

Data is stored in slave devices in four different tables. Two of them store on-off (1-bit) values called Coils and Discrete Inputs, and two store numerical values as 16-bit words called Registers. Each is either read-only or read/write.
資料儲存在從屬設備的四個不同表中。其中兩個表儲存開關（1 位元）值，稱為“線圈”和“離散輸入”，另外兩個表將數值儲存為 16 位元字，稱為“暫存器”。每個表都為唯讀或可讀寫表。

Each table has 9999 locations.
每個表格有 9999 個位置。

![[ModbusSerialProtocolDecoding_modbus-data-storage-table.png]]

## Function codes

There are three categories of Modbus function codes:
Modbus 功能碼有三類：

- Public Function codes – From 1 to 127 except for user-defined codes, validated by Modbus.org community, publicly documented and guaranteed unique.
  公共功能代碼 – 除使用者定義的代碼外，範圍從 1 到 127，由 Modbus.org 社群驗證，公開記錄並保證唯一。
- User-Defined Function Codes – in two ranges from 65 to 72 and from 100 to 110.
  使用者定義的功能代碼 - 有兩個範圍：65 至 72 和 100 至 110。
- Reserved Function Codes – Used by some companies for legacy products and not available for public use.
  保留功能代碼 – 有些公司將其用於舊產品，不供公眾使用。

Examples of commonly used function codes are shown in the table.
常用功能程式碼範例如下表所示。

The full specification for Modbus is freely available from www.modbus.org

![[ModbusSerialProtocolDecoding_modbus-function-codes-examples.png]]

## Decoding Modbus with Picoscope

...
