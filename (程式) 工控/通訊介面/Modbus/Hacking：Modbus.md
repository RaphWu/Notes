---
aliases:
language:
date: 2021-07-07
author: Omar Morando / SCADAsploit
sourceurl: https://scadasploit.dev/posts/2021/07/hacking-modbus/
tags:
  - Modbus
  - ModbusRTU
  - ModbusTCP
---

# Hacking: Modbus

One of the challenges of pentesting in the OT/ICS environment is given by the protocols used which can also be very different from those of IT. ICS installations use a wide variety of protocols that often have little in common with standard Ethernet and TCP/IP.
在 OT/ICS 環境中進行滲透測試的挑戰之一在於所使用的協定可能與 IT 協定大相逕庭。 ICS 安裝使用各種各樣的協議，這些協議通常與標準乙太網路和 TCP/IP 協定幾乎沒有共同之處。

This difference has been the strong point of OT installations for years, protecting them through the mechanism of “security by obscurity”. Now that these protocols are becoming more known and understood, security concerns in these facilities have been heightened.
多年來，這種差異一直是 OT 設施的優勢所在，它透過「模糊安全」機制來保護這些設施。如今，隨著這些協議越來越為人所知和理解，這些設施的安全問題也日益凸顯。

In this article we see the main characteristics of one of these protocols: the Modbus.
在本文中，我們將了解其中一種協定的主要特徵：Modbus。

## The Modbus standard

The Modbus protocol is mainly characterized by the physical connection medium, which can be on a serial port or on Ethernet. There are several variants of the protocol:
Modbus 協定的主要特點在於其實體連接介質，可以是串列埠，也可以是乙太網路。該協議有幾種變體：

- Modbus RTU
- Modbus ASCII
- Modbus-TCP
- Modbus over TCP/IP or Modbus RTU/IP
- Modbus over UDP
- Modbus Plus (Modbus+)
- Secure Modbus

In this article, I will cover the most popular ones: Modbus RTU and Modbus TCP. I will also mention Secure Modbus, a security-oriented version of the protocol that incorporates a data encryption layer.
在本文中，我將介紹最受歡迎的兩種協定：Modbus RTU 和 Modbus TCP。此外，我還將提到 Secure Modbus，這是 Modbus 協定的安全版本，其中包含一個資料加密層。

### Modbus RTU

Modbus RTU was first developed in 1979 by Modicon (now part of Schneider Electric) for their PLC and industrial automation systems. It has become the de facto industry standard. Modbus is a widely accepted public domain protocol, it is a simple and lightweight protocol intended for serial communication. It has a data limit of 253 bytes.
Modbus RTU 最初由 Modicon（現為施耐德電機的一部分）於 1979 年為其 PLC 和工業自動化系統開發。它已成為事實上的行業標準。 Modbus 是一種廣泛接受的公共協議，它是一種簡單輕量級的串行通訊協定。其資料長度限制為 253 個位元組。

Modbus operates at layer 7 of the OSI model. It is an efficient communication methodology between interconnected devices using a “request/response” model. Because it’s simple and lightweight, it requires little processing power. Suffice it to say that there are communication libraries available for practically any embedded device, starting from a simple Arduino board up to the more sophisticated Raspberry.
Modbus 運行於 OSI 模型的第 7 層。它是一種使用“請求/回應”模型在互連設備之間進行高效通訊的方法。由於其簡單輕量，因此所需的處理能力極低。可以說，幾乎所有嵌入式設備（從簡單的 Arduino 開發板到更複雜的 Raspberry）都有可用的通訊庫。

Modbus was initially implemented on the physical topology RS-232C (point-to-point) or RS-485 (multi-drop). It can have up to 32 devices communicating over a serial link with each device having a unique ID.
Modbus 最初是在 RS-232C（點對點）或 RS-485（多點）物理拓撲上實現的。它最多可容納 32 個設備透過串行鏈路進行通信，每個設備都有唯一的 ID。

![[Hacking_Modbus_mod_1.png]]

Modbus uses a master/slave (client/server) architecture where only one device can send requests. The slaves/servers provide the requested data to the master or perform the action requested by the master itself. A slave is any peripheral device (transducer, valve, network unit, etc.) that processes information and sends its output to the master via the Modbus protocol.
Modbus 採用主/從（客戶端/伺服器）架構，其中只有一個裝置可以傳送請求。從設備/伺服器將請求的資料提供給主設備，或執行主設備本身請求的操作。從設備是指任何處理資訊並透過 Modbus 協定將其輸出傳送到主設備的周邊設備（感測器、閥門、網路單元等）。

Masters can address individual slaves or broadcast a message to all slaves. Slaves return a response to all queries addressed to them individually, but do not respond to broadcast queries. Slaves do not generate messages, they can only reply to the master. A master’s query will consist of the slave address (slave ID or unit ID), a function code, all required data, and a CRC error checking field.
主設備可以尋址單一從設備，也可以向所有從設備廣播訊息。從設備會對所有單獨發送給它的查詢做出回應，但不回應廣播查詢。從設備不會產生訊息，只能回覆主設備。主設備的查詢包含從設備位址（從設備 ID 或單元 ID）、功能代碼、所有必需資料以及 CRC 錯誤校驗欄位。

Modbus communicates using Function Codes, function codes that identify a wide range of commands.
Modbus 使用功能代碼進行通信，功能代碼可以識別多種命令。

Main Function Codes for data access and diagnostics
用於資料存取和診斷的主要功能代碼

![[Hacking_Modbus_mod_2.png]]

### Modbus TCP

Modbus TCP is the encapsulated Modbus protocol for use over TCP/IP using port 502. It uses the same request/response as Modbus RTU, the same function codes, and the same 253-byte data limit. The error checking field used in Modbus RTU is eliminated as the TCP/IP link layer uses its checksum methods.
Modbus TCP 是封裝的 Modbus 協議，透過 TCP/IP 使用，使用連接埠 502。它使用與 Modbus RTU 相同的請求/回應、相同的功能代碼以及相同的 253 位元組資料限制。由於 TCP/IP 連結層使用其校驗和方法，因此 Modbus RTU 中使用的錯誤檢查欄位被取消。

Modbus TCP adds an application layer (MBAP) to the Modbus RTU frame. It is 7 bytes long with 2 bytes for the header, 2 bytes for the protocol identifier, 2 bytes for the length and 1 byte for the address (unit ID).
Modbus TCP 在 Modbus RTU 幀的基礎上增加了一個應用層 (MBAP)。該幀長 7 個位元組，其中 2 個位元組為幀頭，2 個位元組為協議標識符，2 個位元組為幀長度，1 個位元組為位址（單元 ID）。

![[Hacking_Modbus_mod_3.png]]

The use of Ethernet allows the creation of more complex architectures, even of the hybrid type, making use of special gateways.
使用乙太網路可以創造更複雜的架構，甚至是混合類型的架構，利用特殊的網關。

![[Hacking_Modbus_mod_4.png|RTU/TCP混合架構]]

### Data packet format

A Modbus frame consists of an Application Data Unit (ADU) encapsulating a Protocol Data Unit (PDU), according to this scheme:
Modbus 訊框由封裝協定資料單元 (PDU) 的應用程式資料單元 (ADU) 組成，具體方案如下：

ADU = Address + PDU + Error check
PDU = Function code + Data

The byte order for values in Modbus data frames is the most significant byte of a multi-byte value sent first. All Modbus variants use one of the following frame formats.
Modbus 資料幀中值的位元組順序是，多位元組值中最高有效位元組優先發送。所有 Modbus 變體都使用以下幀格式之一。

Modbus RTU frame format

| Name     | Length (bits) | Function                                                    |
| :------- | ------------- | :---------------------------------------------------------- |
| Start    | 28            | At least 3½ characters to start frame (with sign condition) |
| Address  | 8             | Station address                                             |
| Function | 8             | Function code, e.g. read coils/holding registers            |
| Data     | n × 8         | Data + length will be filled according to message type      |
| CRC      | 16            | Cyclic Redundancy Check                                     |
| End      | 28            | At least 3½ characters of silence between frames            |

Notes on calculating the CRC:

- Polynomial: x16 + x15 + x2 + 1 (CRC-16-ANSI also known as CRC-16-IBM, normal hexadecimal algebraic polynomial being 8005 and reversed A001).
- Initial value: 65,535.
- Example of frame in hexadecimal: 01 04 02 FF FF B8 80 (CRC-16-ANSI computed from 01 to FF generates 80B8, first the byte is transmitted** least** significant).

Modbus ASCII frame format

| Name     | Length (bytes) | Function                                               |
| :------- | -------------- | :----------------------------------------------------- |
| Start    | 1              | Start with : (ASCII value 3A)                          |
| Address  | 2              | Station address                                        |
| function | 2              | Function code, e.g. read coils                         |
| Date     | n × 2          | Data + length will be filled according to message type |
| LRC      | 2              | Checksum (Longitudinal redundancy check)               |
| End      | 2              | <CR><LF> set (ASCII values 0D, 0A)                     |

Modbus TCP frame format

| name                   | Length (bytes) | Function                                             |
| :--------------------- | -------------- | :--------------------------------------------------- |
| Transaction identifier | 2              | For synchronizing between server and client messages |
| Protocol identifier    | 2              | 0 for Modbus/TCP                                     |
| Length field           | 2              | Number of bytes remaining in this frame              |
| Unit identifier        | 1              | Slave address (255 if not used)                      |
| Function code          | 1              | Function code                                        |
| Data bytes             | n              | Data as a response or commands                       |

## Modbus protocol security 協定安全性

Modbus owes its widespread diffusion to the simplicity of the protocol and to its by now historic presence on the market. But precisely for these two factors it offers the side to different possibilities of attack, with numerous known vulnerabilities. Here’s how some attacks could be performed by exploiting the simple functions that the protocol itself provides, without dedicated enumeration tools like nmap.
Modbus 的廣泛傳播得益於其協議的簡潔性及其在市場上的歷史地位。但正是由於這兩個因素，它也為各種攻擊提供了可能性，並存在許多已知的漏洞。以下是一些攻擊方法，它們可以利用協定本身提供的簡單功能進行攻擊，而無需使用像 nmap 這樣的專用枚舉工具。

> A hacker can initiate his reconnaissance attack by scanning the network for Modbus devices using the protocol diagnostic commands: Clear Counter and Diagnostic Register. A request sent to the PLC, with function code 8 (0x08) and sub-function code 10 (0x0A), will cause the target server to clear its counters and diagnostic register. This feature is typically only implemented in serial devices.
> 駭客可以使用協定診斷指令「清除計數器」和「診斷暫存器」掃描網路中的 Modbus 設備，從而發動偵察攻擊。向 PLC 發送功能代碼為 8 (0x08) 和子功能代碼為 10 (0x0A) 的請求，將導致目標伺服器清除其計數器和診斷暫存器。此功能通常僅在串行設備中實現。

> Another diagnostic command that can be used is Read Device Identification as an attempt to gather Modbus device information: a request with Read Device Identification function code 43 will cause a Modbus server to return the vendor name , the product name and version number. Further information can also be provided in optional fields. An attacker sends the Modbus request packet with function code 43 to all systems on the network and collects information that could be useful for subsequent attacks.
> 另一個可用的診斷命令是“讀取設備標識”，用於嘗試收集 Modbus 設備資訊：帶有“讀取設備標識”功能代碼 43 的請求將導致 Modbus 伺服器傳回供應商名稱、產品名稱和版本號。還可以在可選字段中提供更多資訊。攻擊者將帶有功能代碼 43 的 Modbus 請求封包傳送到網路上的所有系統，並收集可能對後續攻擊有用的資訊。

## Modbus protocol vulnerability 協定漏洞

The Modbus TCP protocol implementation contains several vulnerabilities that could allow an attacker to perform enumeration tasks or send arbitrary commands.
Modbus TCP 協定實作包含多個漏洞，可能允許攻擊者執行枚舉任務或傳送任意命令。

1. **Lack of Confidentiality**: All Modbus messages are transmitted in the clear over the transmission medium.
   **缺乏保密性**：所有 Modbus 訊息均透過傳送媒體以明文形式傳送。
2. **Lack of Integrity**: There are no integrity checks within the protocol, and as a result, it depends on lower-level protocols to preserve data integrity.
   **缺乏完整性**：協定中沒有完整性檢查，因此，它依賴較低層級的協定來維護資料完整性。
3. **Lack of Authentication**: There is no authentication at any level of the protocol, with the possible exception of some undocumented programming commands.
   **缺乏身份驗證**：協議的任何級別都沒有身份驗證，除了一些未記錄的編程命令。
4. **Simplistic Framing**: Modbus TCP frames are sent over established TCP connections. While such connections are generally reliable, they have a significant drawback because of the next point.
   **簡化的幀結構**：Modbus TCP 幀透過已建立的 TCP 連線發送。雖然此類連接通常可靠，但由於以下幾點，它們有一個明顯的缺陷。
5. **Lack of session structure**: Like many request/response protocols (e.g. SNMP, HTTP, etc.) Modbus TCP consists of short-lived transactions where the master sends a request to the slave which results in in a single action. When combined with the lack of authentication and poor TCP Initial Sequence Number (ISN) generation in many embedded devices, it becomes possible for attackers to issue commands without knowing the existing session.
   **缺乏會話結構**：與許多請求/回應協定（例如 SNMP、HTTP 等）一樣，Modbus TCP 由短時交易組成，主設備向從設備發送請求，從設備隨後執行單一操作。由於許多嵌入式裝置缺乏驗證機制，且 TCP 初始序號 (ISN) 產生機制較差，攻擊者可以在不了解現有會話的情況下發出命令。

### “Illegal Function Exception” Vulnerability “非法函數異常”漏洞

These vulnerabilities allow an attacker to perform reconnaissance on the target network. The first vulnerability exists because a Modbus slave device can return an Illegal Function Exception for queries that contain unsupported function code. An unauthenticated remote user could exploit this vulnerability by sending crafted short codes to perform reconnaissance on the target network.
這些漏洞允許攻擊者對目標網路進行偵察。第一個漏洞的存在是因為 Modbus 從設備可能會對包含不受支援的功能代碼的查詢傳回「非法功能異常」。未經身份驗證的遠端使用者可以透過發送精心設計的短代碼來利用此漏洞對目標網路進行偵察。

### “Illegal Address Exception” Vulnerability 「非法地址異常」漏洞

An additional reconnaissance vulnerability is due to the multiple Illegal Address Exception responses generated for queries that contain an illegal slave address. An unauthenticated attacker could exploit this vulnerability by sending queries that contain invalid addresses to the target network and gleaning information about network hosts from the returned messages.
另一個偵察漏洞是由於包含非法從屬位址的查詢會產生多個非法位址異常回應。未經身份驗證的攻擊者可以透過向目標網路發送包含無效位址的查詢並從傳回的訊息中收集有關網路主機的資訊來利用此漏洞。

### Authentication Vulnerability 身份驗證漏洞

Another vulnerability is due to the lack of security controls in the implementation of the Modbus TCP protocol. The protocol specification does not include an authentication mechanism for validating communication between master and slave devices. This flaw could allow an unauthenticated user to send arbitrary commands to any slave device via an attack master.
另一個漏洞是由於 Modbus TCP 協定實作中缺乏安全控製所造成的。此協定規範未包含用於驗證主設備與從設備之間通訊的身份驗證機制。此漏洞可能允許未經身份驗證的使用者透過攻擊主設備向任何從設備發送任意命令。

### DoS Vulnerability DoS 漏洞

The Modbus TCP protocol also contains vulnerabilities that could allow an attacker to cause a Denial of Service (DoS) condition on a target system. The vulnerability is due to an implementation error in the protocol itself when processing discrete input request and response messages.
Modbus TCP 協定也存在漏洞，可能允許攻擊者在目標系統上引發拒絕服務 (DoS) 攻擊。此漏洞是由於協定本身在處理離散輸入請求和回應訊息時存在實作錯誤所造成的。

### Buffer overflow vulnerability 緩衝區溢位漏洞

Another attack on Modbus can be the data packet exceeding the maximum length. The protocol limits the PDU size to 253 bytes to allow the packet to be sent over a serial line, e.g. RS-485 interface. Modbus TCP prepends a 7 byte Modbus Application Protocol (MBAP) header to the PDU and the whole, MBAP+PDU, is encapsulated in a TCP packet. This puts an upper limit on the packet size.
Modbus 的另一種攻擊方式是封包超出最大長度。此協定將 PDU 大小限制為 253 位元組，以便資料包能夠透過串列線路（例如 RS-485 介面）發送。 Modbus TCP 在 PDU 前面附加一個 7 位元組的 Modbus 應用協定 (MBAP) 標頭，並將整個 MBAP 和 PDU 封裝在一個 TCP 封包中。這限制了資料包的大小。

An attacker creates a specially crafted packet longer than 260 bytes and sends it to a client and server. If the client or server has been programmed incorrectly this could lead to a buffer overflow or denial-of-service attack.
攻擊者會建立一個長度超過 260 位元組的特製封包，並將其傳送到客戶端和伺服器。如果用戶端或伺服器程式設計錯誤，則可能導致緩衝區溢位或拒絕服務攻擊。

### Protocol Sniffing 協定嗅探

The simplest attack to use against Modbus is to sniff network traffic, find connected devices, and then send malicious commands to the devices.
針對 Modbus 最簡單的攻擊是嗅探網路流量，找到連接的設備，然後向設備發送惡意命令。

Having no security or encryption features, it’s easy to use Wireshark to gather information from data packets going over the network to and from a Modbus port on a device and read the contents of those packets. Wireshark makes it easy to see what’s in these packets, examine IP addresses, see request short codes, and alter devices to function properly.
由於沒有安全或加密功能，Wireshark 可以輕鬆收集透過網路往返於設備 Modbus 連接埠的封包信息，並讀取這些封包的內容。 Wireshark 可以輕鬆查看這些封包中的內容、檢查 IP 位址、查看請求短代碼，並修改裝置使其正常運作。

![[Hacking_Modbus_mod_5.png|Whireshark in action]]

## Secure Modbus

The most common approach to securing OT protocols is to encapsulate them within a Transport Layer Security (TLS) protocol and use mutual authentication. Many standardization bodies publish guidelines for doing this depending on the protocol, for example:
保護 OT 協定最常見的方法是將其封裝在傳輸層安全性 (TLS) 協定中，並使用相互驗證。許多標準化機構會根據協議發布相應的指南，例如：

- ODVA specifies how to apply TLS encryption to the EtherNet/IP protocol.
  ODVA 指定如何將 TLS 加密應用於 EtherNet/IP 協定。
- Schneider Electric has recently worked to create a Secure Modbus version, which also includes the addition of the X.509 extension for defining permissions (read-only or read-write).
  - 施耐德電機最近致力於創建一個安全的 Modbus 版本，其中還包括添加用於定義權限（唯讀或讀寫）的 X.509 擴充功能。
- IEC 62351-3 defines how to use TLS for the energy industry sector over TCP-based protocols.
  IEC 62351-3 定義如何透過基於 TCP 的協定在能源產業領域使用 TLS。

![[Hacking_Modbus_mod_6.png|Modbus TCP Security]]

![[Hacking_Modbus_mod_7.png]]

# Important note

This article is intended for educational and informational purposes only. Any unauthorized action towards any control system present on a public or private network is illegal! The information contained in this and other articles are intended to make people understand how necessary it is to improve defense systems, and not to provide tools for attacking them. Violating a computer system is punishable by law and can cause serious damage to property and people, especially when it comes to ICS. All the tests that are illustrated in the tutorials have been carried out in isolated, safe, or manufacturer-authorized laboratories.
本文僅供教育資訊參考。任何未經授權針對公共或私人網路上的任何控制系統的操作均屬違法！本文及其他文章中的資訊旨在讓人們了解改善防禦系統的必要性，而非提供攻擊這些系統的工具。入侵電腦系統將受到法律制裁，並可能對財產和人員造成嚴重損害，尤其是在工業控制系統 (ICS) 方面。本教程中演示的所有測試均在獨立、安全或製造商授權的實驗室中進行。

Stay safe, stay free.
