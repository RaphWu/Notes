---
aliases:
date: 2022-02-24
update:
author: 虚梦年华
language:
sourceurl: https://blog.csdn.net/XUMENGCAS/article/details/122062266
tags:
  - Modbus
  - ModbusRTU
  - CSharp
---

# C#实现 ModbusRTU 详解【一】—— 简介及仿真配置

> 本文介绍 ModbusRTU 通信协议，一种广泛应用的工业标准。它采用串口传输报文，支持一主多从架构，允许主站对从站进行读写操作。文中还详细讲解了如何使用 VirtualSerialPortDriver、ModbusSlave 和 ModbusPoll 软件搭建 ModbusRTU 仿真环境。

# ModbusRTU 简介

ModbusRTU 是公开的 Modbus 通讯协议中的一种，一般以串口作为通讯介质来传输标准格式的报文，实现主从站之间的数据交互，广泛应用于工业设备的通信中。

ModbusRTU 是一种一对多的通信协议，所有的数据读写请求由主站发送，指定的从站接收到正确的报文后，则会被动响应主站的请求，返回对应的响应报文。

以下引用自 [百度百科](https://baike.baidu.com/item/Modbus%E9%80%9A%E8%AE%AF%E5%8D%8F%E8%AE%AE/5972462)：

>Modbus 是一种串行通信协议，是 Modicon 公司（现在的施耐德电气 Schneider Electric）于 1979 年为使用可编程逻辑控制器（PLC）通信而发表。Modbus 已经成为工业领域通信协议的业界标准（De facto），并且现在是工业电子设备之间常用的连接方式。Modbus 比其他通信协议使用的更广泛的主要原因有：
>
> 1. 公开发表并且无版权要求
> 2. 易于部署和维护
> 3. 对供应商来说，修改移动本地的比特或字节没有很多限制
>
>==Modbus 允许多个 (大约 240 个) 设备连接在同一个网络上进行通信==，举个例子，一个由测量温度和湿度的装置，并且将结果发送给计算机。在数据采集与监视控制系统（SCADA）中，Modbus 通常用来连接监控计算机和远程终端控制系统（RTU）。

==实际上一个 ModbusRTU 通讯网络允许用户组建的从站数最多只能是 31 个从站==，在这 31 个从站中允许你用 1~247 的数据为每一个从站编辑系统中唯一的一个标号。==当需要组件的从站设备超过 31 个的时候，由于信号会衰减，所以需要添加中继，进而将允许添加的从站数量增加到 120 个以上。==

# ModbusRTU 主站与从站

**主站**是 ModbusRTU 的请求报文发送方以及响应报文接收方，==在一个 ModbusRTU 通讯网络中，仅允许存在一个主站。==它的定位类似于 TCP/IP 中的服务端，但它不同于服务端的是，在同一个 ModbusRTU 网络中，==主站拥有通讯的绝对主动权，即主站能对从站进行任意读写请求==，但是不支持从站向主站进行数据读写。

**从站**是 ModbusRtu 的请求报文接收方以及响应报文发送方，在一个 ModbusRTU 通讯网络中，允许存在多个从站。它的定位类似于 TCP/IP 中的客户端，但它不同于客户端的是，在同一个 ModbusRtu 网络中，==从站不具有读写权限，它仅能被动响应主站的请求，即从站不能成为请求的发送方。==

需要注意的是，当一个设备同时存在于两个 ModbusRTU 网络时，它可以作为 A 网络中的主站的同时也可以作为 B 网络中的从站，即==同一个设备在不同网络中可以扮演不同的角色。==

形象一点来说，一个 ModbusRTU 通信网络可以类比成一个单人玩的单机的电脑游戏，玩游戏的玩家是主站，而游戏中的被操作的角色是从站，鼠标和键盘则是通讯介质。

对于这个游戏来说，玩家只能有一个，但是他可以操纵的角色可以是一个，也可以是很多个（**一主多从**）。

游戏中的角色并不会自己就去推进游戏的剧情或者打怪升级，而是需要玩家进行操作，发出诸如移动、战斗、对话等的指令后，角色响应玩家的指令，才会按照指令的要求做出指定的动作（**主站发送请求，从站被动响应**）。

如果玩家想要让角色前往游戏中不允许到达的地方，则会有类似于“前面的区域，以后再来探索吧~”的对应的错误提示（**主站发送的操作请求错误，如想要读取不存在的寄存器或线圈**）。

如果玩家右下角的 QQ 或者微信突然间接收到了消息，他打开了消息窗需要回复他人的消息，虽然玩家通过鼠标和键盘输入了指令（打开窗口，打字），但是游戏中的角色却不会有响应，因为那个消息不是角色能识别的或者根本就不是发送给角色的（**主站发送错误的请求报文或者主站与从站之间通讯中断**）。

当然玩家也不会永远都是发送消息的一方，玩游戏玩到忘乎所以的玩家突然间听到妈妈喊他去吃饭，为了不惹妈妈生气，就只能暂停自己的游戏去吃饭了。这时，妈妈就成为了通讯中的主站，玩家就成了从站，通讯的介质是空气等物质，搭载着“吃饭啦”的这条消息。（**主站在另一个 ModbusRTU 网络中可以成为从站**）

# ModbusRTU 仿真配置

ModbusRTU 可通过 Virtual Serial Port Driver、Modbus Slave 和 Modbus Poll 三个工具实现通讯仿真。

## 软件下载

以下为官网下载链接：

[Virtual Serial Port Driver](https://www.eltima.com/products/vspdxp/)
[Modbus Slave 和 Modbus Poll](https://www.modbustools.com/download.html)

其中 Virtual Serial Port Driver 的使用可以参考这篇文章：

[C# 串口通讯](https://blog.csdn.net/XUMENGCAS/article/details/121990702?spm=1001.2014.3001.5501)

## Modbus Slave 使用

Modbus Slave 可以仿真出 ModbusRTU 中的从站。

打开 Modbus Slave，可以看到如下界面：
![[CSharp实现ModbusRTU详解_01_ModbusSlave_MainScreen.png]]

点击上方的 Connection 按钮，然后在展开的列表中点击 Connect，则会弹出通讯设置窗口：
![[CSharp实现ModbusRTU详解_01_ModbusSlave_Connection.png]]

在这里可以选择通过 Virtual Serial Port Driver 仿真出来的端口或者真实的端口，然后设置需要的波特率、数据位、奇偶校验和停止位。Connection 选择 Serial Port，Mode 选择 RTU，点击 OK 后则会仿真出一个 ModbusRTU 的从站。

成功仿真之后，选择仿真出的从站窗口，按下 F8，则会弹出 Slave Definition 窗口：
![[CSharp实现ModbusRTU详解_01_ModbusSlave_SlaveDefinition.png]]

在这里可以设置该从站的站地址、寄存器或线圈的起始地址和数量，以及寄存器或线圈的类型。

## Modbus Poll 使用

Modbus Poll 则可以仿真出 ModbusRTU 中的主站。

Modbus Poll 的使用与 Slave 相似，都是打开 Connection 进行配置，成功后则上方的消息窗口则不会出现红色的错误提示。配置如下图所示：
![[CSharp实现ModbusRTU详解_01_ModbusPoll_Connection.png]]

连接成功后可以点击 F8 来设置读写模式：
![[CSharp实现ModbusRTU详解_01_ModbusPoll_RWDefinition.png]]

## 使用示例

双击 Modbus Poll 中地址 0 的值，可以打开值设置窗口：
![[CSharp实现ModbusRTU详解_01_Sample1.png]]

修改值为 20，然后点击 Send，可以看到，Slave 中从站的地址 0 的值也被更改为 20：
![[CSharp实现ModbusRTU详解_01_Sample2.png]]

打开 Modbus Poll，点击 Display，选择 Commuaction，可以查看发送的报文：
![[CSharp实现ModbusRTU详解_01_Sample3.png]]

![[CSharp实现ModbusRTU详解_01_Sample4.png]]

# 结尾

本文简单介绍了 ModbusRTU 这种通信协议与三个仿真软件。至此，我们需要的 ModbusRTU 仿真环境已经搭建完成，下一篇则会详细介绍 ModbusRTU 读取报文的格式及代码实现。

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
