---
aliases:
date:
update:
author:
language:
sourceurl: https://www.modbustools.com/modbus.html
tags:
  - CSharp
  - Modbus
  - ModbusRTU
  - ModbusASCII
---

# Modbus Protocol Description

MODBUS© Protocol is a messaging structure, widely used to establish master-slave communication between intelligent devices. A MODBUS message sent from a master to a slave contains the address of the slave, the 'command' (e.g. 'read register' or 'write register'), the data, and a check sum (LRC or CRC).
MODBUS© 協定是一種訊息傳遞結構，廣泛用於在智慧型裝置之間建立主從通訊。主裝置發送給從裝置的 MODBUS 訊息包含從裝置的位址、「命令」（例如「讀取暫存器」或「寫入暫存器」）、資料以及校驗和（LRC 或 CRC）。

Since Modbus protocol is just a messaging structure, it is independent of the underlying physical layer. It is traditionally implemented using RS232, RS422, or RS485
由於 Modbus 協定本身就是一種訊息結構，因此它獨立於底層物理層。傳統上，它使用 RS232、RS422 或 RS485 介面實現。

## The Request

The function code in the request tells the addressed slave device what kind of action to perform. The data bytes contains any additional information that the slave will need to perform the function. For example, function code 03 will request the slave to read holding registers and respond with their contents. The data field must contain the information telling the slave which register to start at and how many registers to read. The error check field provides a method for the slave to validate the integrity of the message contents.
請求中的功能代碼指示被定址的從設備執行何種操作。資料位元組包含從裝置執行該功能所需的任何附加資訊。例如，功能代碼 03 將請求從裝置讀取保持暫存器並回應其內容。資料欄位必須包含指示從裝置從哪個暫存器開始以及讀取多少個暫存器的資訊。錯誤校驗欄位為從裝置提供了驗證訊息內容完整性的方法。

## The Response

If the slave makes a normal response, the function code in the response is an echo of the function code in the request. The data bytes contain the data collected by the slave, such as register values or status. If an error occurs, the function code is modified to indicate that the response is an error response, and the data bytes contain a code that describes the error. The error check field allows the master to confirm that the message contents are valid.
如果從站做出正常回應，則回應中的功能代碼將與請求中的功能代碼相呼應。數據位元組包含從站收集的數據，例如暫存器值或狀態。如果發生錯誤，則功能代碼將被修改以指示回應為錯誤回應，資料位元組包含描述錯誤的代碼。錯誤校驗欄位可讓主站確認訊息內容是否有效。

Controllers can be setup to communicate on standard Modbus networks using either of two transmission modes: ASCII or RTU.
可以設定控制器使用兩種傳輸模式之一在標準 Modbus 網路上進行通訊：ASCII 或 RTU。

## ASCII Mode

When controllers are setup to communicate on a Modbus network using ASCII (American Standard Code for Information Interchange) mode, each eight-bit byte in a message is sent as two ASCII characters. The main advantage of this mode is that it allows time intervals of up to one second to occur between characters without causing an error.
當控制器設定為使用 ASCII（美國資訊交換標準代碼）模式在 Modbus 網路上通訊時，訊息中的每個八位元組將作為兩個 ASCII 字元發送。此模式的主要優點是允許字元之間出現最多一秒的時間間隔而不會導致錯誤。

**Coding System**
Hexadecimal ASCII printable characters 0 ... 9, A ... F
**Bits per Byte**
1 start bit
7 data bits, least significant bit sent first
1 bit for even / odd parity-no bit for no parity
1 stop bit if parity is used-2 bits if no parity
**Error Checking**
Longitudinal Redundancy Check (LRC)

## RTU Mode

When controllers are setup to communicate on a Modbus network using RTU (Remote Terminal Unit) mode, each eight-bit byte in a message contains two four-bit hexadecimal characters. The main advantage of this mode is that its greater character density allows better data throughput than ASCII for the same baud rate. Each message must be transmitted in a continuous stream.
當控制器設定為使用 RTU（遠端終端單元）模式在 Modbus 網路上通訊時，訊息中的每個八位元組包含兩個四位十六進位字元。此模式的主要優勢在於，在相同波特率下，其更高的字元密度可實現比 ASCII 更高的資料吞吐量。每條訊息必須以連續流的形式傳送。

**Coding System**
Eight-bit binary, hexadecimal 0 ... 9, A ... F
Two hexadecimal characters contained in each eight-bit field of the message
**Bits per Byte**
1 start bit
8 data bits, least significant bit sent first
1 bit for even / odd parity-no bit for no parity
1 stop bit if parity is used-2 bits if no parity
**Error Check Field**
Cyclical Redundancy Check (CRC)

## ASCII Framing

In ASCII mode, messages start with a colon ( : ) character (ASCII 3A hex), and end with a carriage return-line feed (CRLF) pair (ASCII 0D and 0A hex).
在 ASCII 模式下，訊息以冒號 (:) 字元（ASCII 3A 十六進位）開頭，以回車換行 (CRLF) 對（ASCII 0D 和 0A 十六進位）結尾。

The allowable characters transmitted for all other fields are hexadecimal 0 ... 9, A ... F. Networked devices monitor the network bus continuously for the colon character. When one is received, each device decodes the next field (the address field) to find out if it is the addressed device.
所有其他欄位允許傳輸的字元為十六進位 0 ... 9、A ... F。連網設備會持續監控網路匯流排，尋找冒號字元。收到冒號字元後，每個裝置都會解碼下一個欄位（位址欄位），以確定自己是否是被定址的裝置。

Intervals of up to one second can elapse between characters within the message. If a greater interval occurs, the receiving device assumes an error has occurred. A typical message frame is shown below.
訊息中字元之間的間隔最多為一秒。如果間隔超過一秒，接收裝置將假定發生錯誤。典型的訊息幀如下所示。

| Start | Address | Function | Data    | LRC     | End   |
| ----- | ------- | -------- | ------- | ------- | ----- |
| :     | 2 Chars | 2 Chars  | N Chars | 2 Chars | CR LF |
![[Modbus/images/ASCII Framing.jpg]]

## RTU Framing

In RTU mode, messages start with a silent interval of at least 3.5 character times. This is most easily implemented as a multiple of character times at the baud rate that is being used on the network (shown as T1-T2-T3-T4 in the figure below). The first field then transmitted is the device address.
在 RTU 模式下，訊息以至少 3.5 個字元時間的靜默間隔開始。最容易實現的方式是，以網路所用波特率的倍數（如下圖所示，T1-T2-T3-T4）發送訊息。然後傳輸的第一個欄位是設備位址。

The allowable characters transmitted for all fields are hexadecimal 0 ... 9, A ... F. Networked devices monitor the network bus continuously, including during the silent intervals. When the first field (the address field) is received, each device decodes it to find out if it is the addressed device.
所有字段允許傳輸的字元均為十六進位 0 ... 9、A ... F。連網設備會持續監控網路匯流排，包括在靜默期間。當接收到第一個欄位（位址欄位）時，每個設備都會對其進行解碼，以確定自己是否是被尋址的設備。

Following the last transmitted character, a similar interval of at least 3.5 character times marks the end of the message. A new message can begin after this interval.
在最後一個字元傳輸之後，會有一段至少 3.5 個字元時間的類似間隔來標記該訊息的結束。在此間隔之後，可以開始新的訊息。

The entire message frame must be transmitted as a continuous stream. If a silent interval of more than 1.5 character times occurs before completion of the frame, the receiving device flushes the incomplete message and assumes that the next byte will be the address field of a new message.
整個訊息幀必須以連續流的形式傳送。如果在訊框完成之前出現超過 1.5 個字元時間的靜默間隔，接收裝置將刷新未完成的訊息，並假定下一個位元組將是新訊息的位址欄位。

Similarly, if a new message begins earlier than 3.5 character times following a previous message, the receiving device will consider it a continuation of the previous message. This will set an error, as the value in the final CRC field will not be valid for the combined messages. A typical message frame is shown below.
類似地，如果新訊息在前一則訊息之後的 3.5 個字元時間以內開始，接收裝置將認為它是前一則訊息的延續。這將引發錯誤，因為最終 CRC 欄位中的值對於合併後的訊息無效。典型的訊息幀如下所示。

| Start         | Address | Function | Data      | CRC    | End           |
| ------------- | ------- | -------- | --------- | ------ | ------------- |
| 3.5 Char time | 8 Bit   | 8 Bit    | N * 8 Bit | 16 Bit | 3.5 Char time |
![[Modbus/images/RTU Framing.jpg]]

## Address Field

The address field of a message frame contains two characters (ASCII) or eight bits (RTU). The individual slave devices are assigned addresses in the range of 1 ... 247.
訊息訊框的位址欄位包含兩個字元（ASCII）或八位元（RTU）。各個從站設備的位址分配範圍為 1 ... 247。

## Function Field

The Function Code field tells the addressed slave what function to perform.
The following functions are supported by Modbus Poll.
功能代碼欄位告訴被定址的從站要執行什麼功能。
Modbus Poll 支援以下功能。

| DEC     | HEX           | Description                               |
| ------- | ------------- | :---------------------------------------- |
| 01      | (0x01)        | Read Coils                                |
| 02      | (0x02)        | Read Discrete Inputs                      |
| 03      | (0x03)        | Read Holding Registers                    |
| 04      | (0x04)        | Read Input Registers                      |
| 05      | (0x05)        | Write Single Coil                         |
| 06      | (0x06)        | Write Single Register                     |
| 08      | (0x08)        | Diagnostics (Serial Line only)            |
| 11      | (0x0B)        | Get Comm Event Counter (Serial Line only) |
| 15      | (0x0F)        | Write Multiple Coils                      |
| 16      | (0x10)        | Write Multiple Registers                  |
| 17      | (0x11)        | Report Server ID (Serial Line only)       |
| 22      | (0x16)        | Mask Write Register                       |
| 23      | (0x17)        | Read/Write Multiple Registers             |
| 43 / 14 | (0x2B / 0x0E) | Read Device Identification                |

The data field contains the requested or send data.
資料欄位包含請求或傳送的資料。

## Contents of the Error Checking Field

Two kinds of error-checking methods are used for standard Modbus networks. The error checking field contents depend upon the method that is being used.
標準 Modbus 網路使用兩種錯誤校驗方法。錯誤校驗欄位的內容取決於所使用的方法。

### ASCII

When ASCII mode is used for character framing, the error-checking field contains two ASCII characters. The error check characters are the result of a Longitudinal Redundancy Check (LRC) calculation that is performed on the message contents, exclusive of the beginning colon and terminating CRLF characters.
當使用 ASCII 模式進行字元訊框傳輸時，錯誤校驗欄位包含兩個 ASCII 字元。錯誤校驗字元是對訊息內容（不包括起始冒號和終止 CRLF 字元）進行縱向冗餘校驗 (LRC) 計算的結果。

The LRC characters are appended to the message as the last field preceding the CRLF characters.
LRC 字元作為 CRLF 字元前的最後一個欄位附加到訊息中。

[[##LRC Example Code]]

### RTU

When RTU mode is used for character framing, the error-checking field contains a 16-bit value implemented as two eight-bit bytes. The error check value is the result of a Cyclical Redundancy Check calculation performed on the message contents.
當使用 RTU 模式進行字元訊框傳輸時，錯誤校驗欄位包含一個 16 位元值，該值由兩個 8 位元位元組組成。錯誤校驗值是對訊息內容進行循環冗餘校驗計算的結果。

The CRC field is appended to the message as the last field in the message. When this is done, the low-order byte of the field is appended first, followed by the high-order byte. The CRC high-order byte is the last byte to be sent in the message.
CRC 欄位作為訊息的最後一個欄位附加到訊息中。執行此操作時，首先附加該欄位的低位元組，然後附加高位元組。 CRC 高位元組是訊息中要傳送的最後一個位元組。

[[##CRC Example Code]]

---

## Function 01 (01hex) Read Coils

Reads the ON/OFF status of discrete coils in the slave.
讀取從站中離散線圈的開/關狀態。

### Request

The request message specifies the starting coil and quantity of coils to be read.
請求訊息指定了起始線圈和要讀取的線圈數量。

Example of a request to read 13 coils address 10...22 (Coil 11 to 23) from slave device address 4:
從從設備位址 4 讀取 13 個線圈位址 10...22（線圈 11 至 23）的請求範例：

| Field Name           | RTU (hex) | ASCII Characters |
| :------------------- | :-------- | :--------------- |
| Header               | None      | : (Colon)        |
| Slave Address        | 04        | 0 4              |
| Function             | 01        | 0 1              |
| Starting Address Hi  | 00        | 0 0              |
| Starting Address Lo  | 0A        | 0 A              |
| Quantity of Coils Hi | 00        | 0 0              |
| Quantity of Coils Lo | 0D        | 0 D              |
| Error Check Lo       | DD        | LRC (E 4)        |
| Error Check Hi       | 98        |                  |
| Trailer              | None      | CR LF            |
| Total Bytes          | 8         | 17               |

### Response

The coil status response message is packed as one coil per bit of the data field. Status is indicated as: 1 is the value ON, and 0 is the value OFF. The LSB of the first data byte contains the coil addressed in the request. The other coils follow toward the high-order end of this byte and from low order to high order in subsequent bytes. If the returned coil quantity is not a multiple of eight, the remaining bits in the final data byte will be padded with zeroes (toward the high-order end of the byte). The byte count field specifies the quantity of complete bytes of data.
線圈狀態回應訊息以資料欄位每位一個線圈的形式打包。狀態指示如下：1 表示值為 ON，0 表示值為 OFF。第一個資料位元組的 LSB 包含請求中指定的線圈位址。其他線圈緊跟著該位元組的高位，並在後續位元組中從低位到高位依次排列。如果返回的線圈數量不是 8 的倍數，則最後一個資料位元組的剩餘位元將用零填充（朝向位元組的高位）。位元組數字段指定完整資料的位元組數。

Example of a response to the request:

| Field Name           | RTU (hex) | ASCII Characters |
| :------------------- | :-------- | :--------------- |
| Header               | None      | : (Colon)        |
| Slave Address        | 04        | 0 4              |
| Function             | 01        | 0 1              |
| Byte Count           | 02        | 0 2              |
| Data (Coils 18...11) | 0A        | 0 A              |
| Data (Coils 23...19) | 11        | 1 1              |
| Error Check Lo       | B3        | LRC (D E)        |
| Error Check Hi       | 50        | None             |
| Trailer              | None      | CR LF            |
| Total Bytes          | 7         | 15               |

---

## Function 02(02hex) Read Discrete Inputs

Reads the ON/OFF status of discrete inputs in the slave.
讀取從站中離散輸入的開/關狀態。

### Request

The request message specifies the starting input and quantity of inputs to be read.
請求訊息指定起始輸入和要讀取的輸入數量。

Example of a request to read 13 inputs address 10...22 (input 10011 to 10023) from slave device address 4:
從從屬裝置位址 4 讀取 13 個輸入位址 10...22（輸入 10011 至 10023）的請求範例：

| Field Name            | RTU (hex) | ASCII Characters |
| :-------------------- | :-------- | :--------------- |
| Header                | None      | : (Colon)        |
| Slave Address         | 04        | 0 4              |
| Function              | 02        | 0 2              |
| Starting Address Hi   | 00        | 0 0              |
| Starting Address Lo   | 0A        | 0 A              |
| Quantity of Inputs Hi | 00        | 0 0              |
| Quantity of Inputs Lo | 0D        | 0 D              |
| Error Check Lo        | 99        | LRC (E 3)        |
| Error Check Hi        | 98        | None             |
| Trailer               | None      | CR LF            |
| Total Bytes           | 8         | 17               |

### Response

The input status response message is packed as one input per bit of the data field. Status is indicated as: 1 is the value ON, and 0 is the value OFF. The LSB of the first data byte contains the input addressed in the request. The other inputs follow toward the high-order end of this byte and from low order to high order in subsequent bytes. If the returned input quantity is not a multiple of eight, the remaining bits in the final data byte will be padded with zeroes (toward the high-order end of the byte). The byte count field specifies the quantity of complete bytes of data.
輸入狀態回應訊息以資料欄位每位一個輸入的形式打包。狀態指示如下：1 表示值為 ON，0 表示值為 OFF。第一個資料位元組的 LSB 包含請求中指定的輸入位址。其他輸入依序向該位元組的高位元方向排列，並在後續位元組中從低位元到高位元依序排列。如果傳回的輸入數量不是 8 的倍數，則最後一個資料位元組的剩餘位元將用零填充（向位元組的高位方向排列）。位元組計數欄位指定完整資料位元組的數量。

Example of a response to the request:

| Field Name            | RTU (hex) | ASCII Characters |
| :-------------------- | :-------- | :--------------- |
| Header                | None      | : (Colon)        |
| Slave Address         | 04        | 0 4              |
| Function              | 02        | 0 2              |
| Byte Count            | 02        | 0 2              |
| Data (Inputs 18...11) | 0A        | 0 A              |
| Data (Inputs 23...19) | 11        | 1 1              |
| Error Check Lo        | B3        | LRC (D D)        |
| Error Check Hi        | 14        | None             |
| Trailer               | None      | CR LF            |
| Total Bytes           | 7         | 15               |

---

## Function 03 (03hex) Read Holding Registers

Read the binary contents of holding registers in the slave.
讀取從站中保持暫存器的二進位內容。

### Request

The request message specifies the starting register and quantity of registers to be read.
請求訊息指定了要讀取的起始寄存器和寄存器數量。

Example of a request to read 0...1 (register 40001 to 40002) from slave device 1:
從從裝置 1 讀取 0...1（暫存器 40001 至 40002）的請求範例：

| Field Name               | RTU (hex) | ASCII Characters |
| :----------------------- | :-------- | :--------------- |
| Header                   | None      | : (Colon)        |
| Slave Address            | 01        | 0 1              |
| Function                 | 03        | 0 3              |
| Starting Address Hi      | 00        | 0 0              |
| Starting Address Lo      | 00        | 0 0              |
| Quantity of Registers Hi | 00        | 0 0              |
| Quantity of Registers Lo | 02        | 0 2              |
| Error Check Lo           | C4        | LRC (F A)        |
| Error Check Hi           | 0B        | None             |
| Trailer                  | None      | CR LF            |
| Total Bytes              | 8         | 17               |

### Response

The register data in the response message are packed as two bytes per register, with the binary contents right justified within each byte. For each register the first byte contains the high-order bits, and the second contains the low-order bits.
回應訊息中的暫存器資料會依每個暫存器兩個位元組打包，每個位元組內的二進位內容右對齊。每個暫存器的第一個位元組包含高位，第二個位元組包含低位。

Example of a response to the request:

| Field Name     | RTU (hex) | ASCII Characters |
| :------------- | :-------- | :--------------- |
| Header         | None      | : (Colon)        |
| Slave Address  | 01        | 0 1              |
| Function       | 03        | 0 3              |
| Byte Count     | 04        | 0 4              |
| Data Hi        | 00        | 0 0              |
| Data Lo        | 06        | 0 6              |
| Data Hi        | 00        | 0 0              |
| Data Lo        | 05        | 0 5              |
| Error Check Lo | DA        | LRC (E D)        |
| Error Check Hi | 31        | None             |
| Trailer        | None      | CR LF            |
| Total Bytes    | 9         | 19               |

---

## Function 04 (04hex) Read Input Registers

Read the binary contents of input registers in the slave.
讀取從站中輸入暫存器的二進位內容。

### Request

The request message specifies the starting register and quantity of registers to be read.
請求訊息指定了要讀取的起始寄存器和寄存器數量。

Example of a request to read 0...1 (register 30001 to 30002) from slave device 1:
從從裝置 1 讀取 0...1（暫存器 30001 至 30002）的請求範例：

| Field Name               | RTU (hex) | ASCII Characters |
| :----------------------- | :-------- | :--------------- |
| Header                   | None      | : (Colon)        |
| Slave Address            | 01        | 0 1              |
| Function                 | 04        | 0 4              |
| Starting Address Hi      | 00        | 0 0              |
| Starting Address Lo      | 00        | 0 0              |
| Quantity of Registers Hi | 00        | 0 0              |
| Quantity of Registers Lo | 02        | 0 2              |
| Error Check Lo           | 71        | LRC (F 9)        |
| Error Check Hi           | CB        | None             |
| Trailer                  | None      | CR LF            |
| Total Bytes              | 8         | 17               |

### Response

The register data in the response message are packed as two bytes per register, with the binary contents right justified within each byte. For each register the first byte contains the high-order bits, and the second contains the low-order bits.
回應訊息中的暫存器資料會依每個暫存器兩個位元組打包，每個位元組內的二進位內容右對齊。每個暫存器的第一個位元組包含高位，第二個位元組包含低位。

Example of a response to the request:

| Field Name     | RTU (hex) | ASCII Characters |
| :------------- | :-------- | :--------------- |
| Header         | None      | : (Colon)        |
| Slave Address  | 01        | 0 1              |
| Function       | 04        | 0 4              |
| Byte Count     | 04        | 0 4              |
| Data Hi        | 00        | 0 0              |
| Data Lo        | 06        | 0 6              |
| Data Hi        | 00        | 0 0              |
| Data Lo        | 05        | 0 5              |
| Error Check Lo | DB        | LRC (E C)        |
| Error Check Hi | 86        | None             |
| Trailer        | None      | CR LF            |
| Total Bytes    | 9         | 19               |

---

## Function 05 (05hex) Write Single Coil

Writes a single coil to either ON or OFF.
將單一線圈寫入 ON 或 OFF。

### Request

The request message specifies the coil reference to be written. Coils are addressed starting at zero-coil 1 is addressed as 0.
請求訊息指定了要寫入的線圈編號。線圈的尋址從零開始，例如線圈 1 的尋址為 0。

The requested ON / OFF state is specified by a constant in the request data field. A value of FF 00 hex requests the coil to be ON. A value of 00 00 requests it to be OFF. All other values are illegal and will not affect the coil.
請求的 ON/OFF 狀態由請求資料欄位中的常數指定。 FF 00（十六進位）值請求線圈處於 ON 狀態。 00 00 值請求線圈處於 OFF 狀態。所有其他值均為非法值，不會影響線圈。

Here is an example of a request to write coil 173 ON in slave device 17:
以下是在從屬設備 17 中將線圈 173 寫入 ON 的請求範例：

| Field Name      | RTU (hex) | ASCII Characters |
| :-------------- | :-------- | :--------------- |
| Header          | None      | : (Colon)        |
| Slave Address   | 11        | 0 1              |
| Function        | 05        | 0 5              |
| Coil Address Hi | 00        | 0 0              |
| Coil Address Lo | AC        | A C              |
| Write Data Hi   | FF        | 0 0              |
| Write Data Lo   | 00        | F F              |
| Error Check Lo  | 4E        | LRC (3 F)        |
| Error Check Hi  | 8B        | None             |
| Trailer         | None      | CR LF            |
| Total Bytes     | 8         | 17               |

### Response

The normal response is an echo of the request, returned after the coil state has been written.
正常回應是請求的回顯，在寫入線圈狀態後返回。

Example of a response to the request:

| Field Name      | RTU (hex) | ASCII Characters |
| :-------------- | :-------- | :--------------- |
| Header          | None      | : (Colon)        |
| Slave Address   | 11        | 1 1              |
| Function        | 05        | 0 5              |
| Coil Address Hi | 00        | 0 0              |
| Coil Address Lo | AC        | A C              |
| Write Data Hi   | FF        | 0 0              |
| Write Data Lo   | 00        | F F              |
| Error Check Lo  | 4E        | LRC (3 F)        |
| Error Check Hi  | 8B        | None             |
| Trailer         | None      | CR LF            |
| Total Bytes     | 8         | 17               |

---

## Function 06 (06hex) Write Single Register

Writes a value into a single holding register.
將值寫入單一保持暫存器。

### Request

The request message specifies the register reference to be Written. Registers are addressed starting at zero-register 1 is addressed as 0.
請求訊息指定要寫入的寄存器引用。暫存器的尋址從零開始，暫存器 1 的尋址為 0。

The requested Write value is specified in the request data field. Here is an example of a request to Write register 40002 to 00 03 hex in slave device 17.
請求的寫入值在請求資料欄位中指定。以下範例請求將 40002 暫存器寫入從裝置 17 中的 00 03 十六進位值。

| Field Name          | RTU (hex) | ASCII Characters |
| :------------------ | :-------- | :--------------- |
| Header              | None      | : (Colon)        |
| Slave Address       | 11        | 1 1              |
| Function            | 06        | 0 6              |
| Register Address Hi | 00        | 0 0              |
| Register Address Lo | 01        | 0 1              |
| Write Data Hi       | 00        | 0 0              |
| Write Data Lo       | 03        | 0 3              |
| Error Check Lo      | 9A        | LRC (E 5)        |
| Error Check Hi      | 9B        | None             |
| Trailer             | None      | CR LF            |
| Total Bytes         | 8         | 17               |

### Response

The normal response is an echo of the request, returned after the register contents have been written.
正常回應是請求的回顯，在寫入暫存器內容後返回。

| Field Name          | RTU (hex) | ASCII Characters |
| :------------------ | :-------- | :--------------- |
| Header              | None      | : (Colon)        |
| Slave Address       | 11        | 1 1              |
| Function            | 06        | 0 6              |
| Register Address Hi | 00        | 0 0              |
| Register Address Lo | 01        | 0 1              |
| Write Data Hi       | 00        | 0 0              |
| Write Data Lo       | 03        | 0 3              |
| Error Check Lo      | 9A        | LRC (E 5)        |
| Error Check Hi      | 9B        | None             |
| Trailer             | None      | CR LF            |
| Total Bytes         | 8         | 17               |

---

## Function 15 (0Fhex) Write Multiple Coils

Writes each coil in a sequence of coils to either ON or OFF.
將線圈序列中的每個線圈寫入 ON 或 OFF。

### Request

The request message specifies the coil references to be written. Coils are addressed starting at zero-coil 1 is addressed as 0.
請求訊息指定了要寫入的線圈編號。線圈的尋址從零開始，例如線圈 1 的尋址為 0。

The requested ON / OFF states are specified by contents of the request data field. A logical 1 in a bit position of the field requests the corresponding coils to be ON. A logical 0 requests it to be OFF.
請求的 ON / OFF 狀態由請求資料欄位的內容指定。字段中某個位元為邏輯 1 時，請求對應線圈處於 ON 狀態。邏輯 0 時，請求對應線圈處於 OFF 狀態。

Below is an example of a request to write a series of ten coils starting at coil 20 (addressed as 19, or 13 hex) in slave device 17.
以下是一個請求範例，該範例要求在從屬設備 17 中寫入從線圈 20（尋址為 19 或十六進位 13）開始的一系列十個線圈。

The request data contents are two bytes: CD 01 hex (1100 1101 0000 0001 binary). The binary bits correspond to the coils in the following way:
請求資料內容為兩個位元組：CD 01（十六進制，1100 1101 0000 0001 二進位）。二進位位與線圈的對應關係如下：

**Bit**: 1 1 0 0 1 1 0 1 0 0 0 0 0 0 0 1
**Coil**: 27 26 25 24 23 22 21 20 - - - - - - 29 28

The first byte transmitted (CD hex) addresses coils 27 ... 20, with the least significant bit addressing the lowest coil (20) in this set.
傳輸的第一個位元組（CD 十六進位）尋址線圈 27 ... 20，其中最低有效位元尋址此組中的最低線圈（20）。

The next byte transmitted (01 hex) addresses coils 29 and 28, with the least significant bit addressing the lowest coil (28) in this set. Unused bits in the last data byte should be zero-filled.
下一個傳輸的位元組（十六進位 01）表示線圈 29 和 28 的位址，其中最低有效位元表示該組中最低的線圈（28）。最後一個資料位元組中未使用的位元應以零填充。

| Field Name           | RTU (hex) | ASCII Characters |
| :------------------- | :-------- | :--------------- |
| Header               | None      | : (Colon)        |
| Slave Address        | 11        | 1 1              |
| Function             | 0F        | 0 F              |
| Coil Address Hi      | 00        | 0 0              |
| Coil Address Lo      | 13        | 1 3              |
| Quantity of Coils Hi | 00        | 0 0              |
| Quantity of Coils Lo | 0A        | 0 A              |
| Byte Count           | 02        | 0 2              |
| Write Data Hi        | CD        | C D              |
| Write Data Lo        | 01        | 0 1              |
| Error Check Lo       | BF        | LRC (F 3)        |
| Error Check Hi       | 0B        | None             |
| Trailer              | None      | CR LF            |
| Total Bytes          | 11        | 23               |

### Response

The normal response returns the slave address, function code, starting address, and number of coils written. Here is an example of a response to the request shown above
正常響應返回從站位址、功能碼、起始位址以及寫入的線圈數量。以下是上述請求的回應範例

| Field Name           | RTU (hex) | ASCII Characters |
| :------------------- | :-------- | :--------------- |
| Header               | None      | : (Colon)        |
| Slave Address        | 11        | 1 1              |
| Function             | 0F        | 0 F              |
| Coil Address Hi      | 00        | 0 0              |
| Coil Address Lo      | 13        | 1 3              |
| Quantity of Coils Hi | 00        | 0 0              |
| Quantity of Coils Lo | 0A        | 0 A              |
| Error Check Lo       | 26        | LRC (C 3)        |
| Error Check Hi       | 99        | None             |
| Trailer              | None      | CR LF            |
| Total Bytes          | 8         | 17               |

---

## Function 16 (10hex) Write Multiple Registers

Writes values into a sequence of holding registers
將值寫入一系列保持暫存器

### Request

The request message specifies the register references to be written. Registers are addressed starting at zero-register 1 is addressed as 0.
請求訊息指定要寫入的寄存器引用。暫存器的尋址從零開始，暫存器 1 的尋址為 0。

The requested write values are specified in the request data field. Data is packed as two bytes per register.
請求的寫入值在請求資料欄位中指定。每個暫存器的資料以兩個位元組打包。

Here is an example of a request to write two registers starting at 40002 to 00 0A and 01 02 hex, in slave device 17:
以下是從屬裝置 17 中將從 40002 開始的兩個暫存器寫入十六進位 00 0A 和 01 02 的請求範例：

| Field Name               | RTU (hex) | ASCII Characters |
| :----------------------- | :-------- | :--------------- |
| Header                   | None      | : (Colon)        |
| Slave Address            | 11        | 1 1              |
| Function                 | 10        | 1 0              |
| Starting Address Hi      | 00        | 0 0              |
| Starting Address Lo      | 01        | 0 1              |
| Quantity of Registers Hi | 00        | 0 0              |
| Quantity of Registers Lo | 02        | 0 2              |
| Byte Count               | 04        | 0 4              |
| Data Hi                  | 00        | 0 0              |
| Data Lo                  | 0A        | 0 A              |
| Data Hi                  | 01        | 0 1              |
| Data Lo                  | 02        | 0 2              |
| Error Check Lo           | C6        | LRC (C B)        |
| Error Check Hi           | F0        | None             |
| Trailer                  | None      | CR LF            |
| Total Bytes              | 13        | 23               |

### Response

The normal response returns the slave address, function code, starting address, and quantity of registers written. Here is an example of a response to the request shown above.
正常回應會傳回從機位址、功能碼、起始位址以及寫入的暫存器數量。以下是針對上述請求的回應範例。

| Field Name               | RTU (hex) | ASCII Characters |
| :----------------------- | :-------- | :--------------- |
| Header                   | None      | : (Colon)        |
| Slave Address            | 11        | 1 1              |
| Function                 | 10        | 1 0              |
| Starting Address Hi      | 00        | 0 0              |
| Starting Address Lo      | 01        | 0 1              |
| Quantity of Registers Hi | 00        | 0 0              |
| Quantity of Registers Lo | 02        | 0 2              |
| Error Check Lo           | 21        | LRC (D C)        |
| Error Check Hi           | 98        | None             |
| Trailer                  | None      | CR LF            |
| Total Bytes              | 8         | 17               |

---

## LRC Example Code

This function is an example how to calculate a LRC BYTE using the C language.
此函數是使用 C 語言計算 LRC BYTE 的範例。

```c
BYTE LRC(BYTE* nData, WORD wLength)
{
    BYTE nLRC = 0; // LRC char initialized

    for (int i = 0; i < wLength; i++)
        nLRC += *nData++;

    return (BYTE)(-nLRC);
} // End: LRC
```

---

## CRC Example Code

This function is an example how to calculate a CRC word using the C language.
此函數是使用 C 語言計算 CRC 字的範例。

```c
WORD CRC16(const BYTE* nData, WORD wLength)
{
    static const WORD wCRCTable[] = {
		0X0000, 0XC0C1, 0XC181, 0X0140, 0XC301, 0X03C0, 0X0280, 0XC241,
		0XC601, 0X06C0, 0X0780, 0XC741, 0X0500, 0XC5C1, 0XC481, 0X0440,
		0XCC01, 0X0CC0, 0X0D80, 0XCD41, 0X0F00, 0XCFC1, 0XCE81, 0X0E40,
		0X0A00, 0XCAC1, 0XCB81, 0X0B40, 0XC901, 0X09C0, 0X0880, 0XC841,
		0XD801, 0X18C0, 0X1980, 0XD941, 0X1B00, 0XDBC1, 0XDA81, 0X1A40,
		0X1E00, 0XDEC1, 0XDF81, 0X1F40, 0XDD01, 0X1DC0, 0X1C80, 0XDC41,
		0X1400, 0XD4C1, 0XD581, 0X1540, 0XD701, 0X17C0, 0X1680, 0XD641,
		0XD201, 0X12C0, 0X1380, 0XD341, 0X1100, 0XD1C1, 0XD081, 0X1040,
		0XF001, 0X30C0, 0X3180, 0XF141, 0X3300, 0XF3C1, 0XF281, 0X3240,
		0X3600, 0XF6C1, 0XF781, 0X3740, 0XF501, 0X35C0, 0X3480, 0XF441,
		0X3C00, 0XFCC1, 0XFD81, 0X3D40, 0XFF01, 0X3FC0, 0X3E80, 0XFE41,
		0XFA01, 0X3AC0, 0X3B80, 0XFB41, 0X3900, 0XF9C1, 0XF881, 0X3840,
		0X2800, 0XE8C1, 0XE981, 0X2940, 0XEB01, 0X2BC0, 0X2A80, 0XEA41,
		0XEE01, 0X2EC0, 0X2F80, 0XEF41, 0X2D00, 0XEDC1, 0XEC81, 0X2C40,
		0XE401, 0X24C0, 0X2580, 0XE541, 0X2700, 0XE7C1, 0XE681, 0X2640,
		0X2200, 0XE2C1, 0XE381, 0X2340, 0XE101, 0X21C0, 0X2080, 0XE041,
		0XA001, 0X60C0, 0X6180, 0XA141, 0X6300, 0XA3C1, 0XA281, 0X6240,
		0X6600, 0XA6C1, 0XA781, 0X6740, 0XA501, 0X65C0, 0X6480, 0XA441,
		0X6C00, 0XACC1, 0XAD81, 0X6D40, 0XAF01, 0X6FC0, 0X6E80, 0XAE41,
		0XAA01, 0X6AC0, 0X6B80, 0XAB41, 0X6900, 0XA9C1, 0XA881, 0X6840,
		0X7800, 0XB8C1, 0XB981, 0X7940, 0XBB01, 0X7BC0, 0X7A80, 0XBA41,
		0XBE01, 0X7EC0, 0X7F80, 0XBF41, 0X7D00, 0XBDC1, 0XBC81, 0X7C40,
		0XB401, 0X74C0, 0X7580, 0XB541, 0X7700, 0XB7C1, 0XB681, 0X7640,
		0X7200, 0XB2C1, 0XB381, 0X7340, 0XB101, 0X71C0, 0X7080, 0XB041,
		0X5000, 0X90C1, 0X9181, 0X5140, 0X9301, 0X53C0, 0X5280, 0X9241,
		0X9601, 0X56C0, 0X5780, 0X9741, 0X5500, 0X95C1, 0X9481, 0X5440,
		0X9C01, 0X5CC0, 0X5D80, 0X9D41, 0X5F00, 0X9FC1, 0X9E81, 0X5E40,
		0X5A00, 0X9AC1, 0X9B81, 0X5B40, 0X9901, 0X59C0, 0X5880, 0X9841,
		0X8801, 0X48C0, 0X4980, 0X8941, 0X4B00, 0X8BC1, 0X8A81, 0X4A40,
		0X4E00, 0X8EC1, 0X8F81, 0X4F40, 0X8D01, 0X4DC0, 0X4C80, 0X8C41,
		0X4400, 0X84C1, 0X8581, 0X4540, 0X8701, 0X47C0, 0X4680, 0X8641,
		0X8201, 0X42C0, 0X4380, 0X8341, 0X4100, 0X81C1, 0X8081, 0X4040 };

    BYTE nTemp;
    WORD wCRCWord = 0xFFFF;

    while (wLength--)
    {
        nTemp = *nData++ ^ wCRCWord;
        wCRCWord >>= 8;
        wCRCWord ^= wCRCTable[nTemp];
    }
    return wCRCWord;
} // End: CRC16
```
