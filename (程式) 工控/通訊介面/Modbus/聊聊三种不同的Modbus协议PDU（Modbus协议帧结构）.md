---
aliases:
date: 2024-06-25
author: Hello工控
language:
sourceurl: https://cloud.tencent.com/developer/article/2431674
tags:
  - Modbus
---

我们这期主要讨论 Mobus 协议帧内部的结构（PDU 和 ADU）。

# Modubs PDU

MODBUS 协议定义了一个简单的协议数据单元（PDU），这个定义是独立于底层通信层的。

Protocol data unit ，缩写为 PDU，即协议数据单元，结构如下图所示：
![[聊聊三种不同的Modbus协议PDU_1.jpg]]

一个 PDU 单元主要由功能码和相应的数据两部分组成。

将 MODBUS 协议映射到特定的总线或网络会在协议数据单元上引入一些额外的字段。发起 MODBUS 事务的客户端构建 MODBUS PDU，然后添加字段以构建适当的通信 PDU。下图即在串行总线上的 PDU 通信帧结构。
![[聊聊三种不同的Modbus协议PDU_2.jpg]]

当然也有将上述结构简称为 ADU（Application Data Unit 应用数据单元），如下图所示：
![[聊聊三种不同的Modbus协议PDU_3.jpg]]

需要说明的以下几点：

MODBUS 应用数据单元（ADU）由发起 MODBUS 事务的客户端构建。功能码指示服务器要执行的操作类型。MODBUS 应用协议建立了客户端发起请求的格式。

MODBUS 数据单元的功能码字段以一字节编码。有效的代码范围是 1 到 255 的十进制数（128 到 255 的范围是保留的，用于异常响应）。当从客户端发送消息到服务器设备时，功能码字段告诉服务器要执行什么操作。功能码 "0" 是无效的。

某些功能码会添加子功能码以定义多个操作。客户端发送到服务器设备的消息的数据字段包含服务器用来执行功能码定义的操作的额外信息。这可能包括离散和寄存器地址、要处理的项目数量以及字段中实际数据字节的计数。

在某些类型的请求中，数据字段可能不存在（长度为零），在这种情况下，服务器不需要任何额外信息。功能码单独指定操作。

如果与正确接收到的 MODBUS ADU 中请求的 MODBUS 功能相关的没有发生错误，服务器对客户端的响应的数据字段包含请求的数据。对于正常响应，服务器简单地向请求回响原始的功能码。
![[聊聊三种不同的Modbus协议PDU_4.jpg]]
![[聊聊三种不同的Modbus协议PDU_5.jpg]]

如果发生与请求的 MODBUS 功能相关的错误，该字段包含一个异常代码，服务器应用程序可以使用它来确定下一步要采取的操作。
![[聊聊三种不同的Modbus协议PDU_6.jpg]]

例如，客户端可以读取一组离散输出或输入的开/关状态，或者它可以读写一组寄存器的数据内容。当服务器响应客户端时，它使用功能码字段来指示是正常（无错误）响应还是发生了某种错误（称为异常响应）。

需要特别注意的是：超时处理机制是必要的。可以来避免不无限期地等待可能永远不会到来的回复。

# RTU、ASCII 和 TCP 协议帧

我们先通过内部的 PDU 结构图来看看：

Modbus RTU 协议帧：
![[聊聊三种不同的Modbus协议PDU_7.jpg]]

Modbus ASCII 协议帧：
![[聊聊三种不同的Modbus协议PDU_8.jpg]]

Modbus TCP 协议帧：
![[聊聊三种不同的Modbus协议PDU_9.jpg]]

MODBUS PDU 的大小受到从最初的串行线路网络（最大 RS485 ADU = 256 字节）继承的大小限制。

因此：串行线路通信的 MODBUS PDU = 256 - 服务器地址（1 字节）CRC（2 字节）= 253 字节。

RS232 / RS485 ADU = 253 字节 + 服务器地址（1 字节）+ CRC（2 字节）= 256 字节。

TCP MODBUS ADU = 253 字节 + MBAP（7 字节）= 260 字节。

# 三种不同类型的 PDU

MODBUS 协议定义了三种 PDUs（协议数据单元），它们是：

- MODBUS 请求 PDU，mb_req_pdu
- MODBUS 响应 PDU，mb_rsp_pdu
- MODBUS 异常响应 PDU，mb_excep_rsp_pdu

这三种具体的定义如下：

`mb_req_pdu = {function_code, request_data}`

其中：function_code = [1 字节] MODBUS 功能码， request_data = [n 字节] 这个字段依赖于功能码，通常包含诸如变量引用、 变量计数、数据偏移量、子功能码等信息。

`mb_rsp_pdu = {function_code, response_data}`

其中：function_code = [1 字节] MODBUS 功能码 response_data = [n 字节] 这个字段依赖于功能码，通常包含诸如变量引用、变量计数、数据偏移量、子功能码等信息。

`mb_excep_rsp_pdu = {exception-function_code, request_data}`

其中：

exception-function_code = [1 字节] MODBUS 功能码 + 0x80

exception_code = [1 字节] 定义在“MODBUS 异常代码”表中的 MODBUS 异常代码，后期会单独说明，敬请持续关注。
