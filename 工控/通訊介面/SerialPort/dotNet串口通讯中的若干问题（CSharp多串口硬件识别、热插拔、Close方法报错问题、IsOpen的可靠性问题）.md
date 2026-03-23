---
aliases:
date: 2019-05-06
update:
author: 卖雨伞的小男孩
language: C#
sourceurl: https://www.cnblogs.com/xietianjiao/p/10820928.html
tags:
  - CSharp
  - SerialPort
---

# .Net 串口通讯中的若干问题（C#多串口硬件识别、热插拔、Close 方法报错问题、IsOpen 的可靠性问题）

## 一、需求场景

最近有时间静下心来研究 SDK，串口通讯的。要求实现识别 cp210x 和 cp2303 驱动的两款硬件，并且 2303 的优先级高，即有 2303 识别之，没有再识别 210x；要求实现热插拔，拔掉自动断开，插上自动连接。

## 二、问题一：如何实现串口硬件的识别呢？

1. 如果方便的话，SerialPort 的 Handshake 这个字段值得深入研究，可以利用这个实现；
2. 添加自定义的握手协议，本人用一个 5 字节的串进行校验（第三位是硬件版本标识），校验算法如下：

```csharp
for (int i = 0; i < btData.Length; i++)
{
	if ((i % 5) == 0) commit = 0;
	if (btData[i].ToString().Equals(byteMitt[i % 5]) || i % 5 == 3)
		commit++;
	if (commit == 5)
	{
		SendMessage(EnumDataConverter.GetCommandStatus(SendCommandType.Test) +
					intDeviceCount.ToString().PadLeft(2, '0'));
		tpVersion = btData[3].ToString();
		IsConnected = true;
		strDeviceName = tempPort.Value;
		return true;
	}
} //校验完毕没有成功，进入下次循环
```

## 三、热插拔的监听问题

经过实践，cp210x 可以在串口设备表中查询到，cp2303 不可不发现，故而采用查询计算机设备表的方式

备注：

`System.IO.Ports.SerialPort.GetPortNames()` 只能获取设备名，不能获取详细信息（类型等），我下面用来进行一些比对的逻辑操作

```csharp
private void CreateUSBWatcher()
{
    Ports = System.IO.Ports.SerialPort.GetPortNames();
    //建立监听
    ManagementScope scope = new ManagementScope("root\\CIMV2");
    scope.Options.EnablePrivileges = true;
    //建立插入监听
    try
    {
        WqlEventQuery USBInsertQuery = new WqlEventQuery("__InstanceCreationEvent", "TargetInstance ISA 'Win32_PnPEntity'");
        USBInsertQuery.WithinInterval = new TimeSpan(0, 0, 2);
        USBInsert = new ManagementEventWatcher(scope, USBInsertQuery);
        USBInsert.EventArrived += USBInsert_EventArrived;
        USBInsert.Start();
    }
    catch (Exception ex)
    {
        if (USBInsert != null)
        {
            USBInsert.Stop();
        }
        throw ex;
    }
    //建立拔出监听
    try
    {
        WqlEventQuery USBRemoveQuery = new WqlEventQuery("__InstanceDeletionEvent", "TargetInstance ISA 'Win32_PnPEntity'");
        USBRemoveQuery.WithinInterval = new TimeSpan(0, 0, 2);
        USBRemove = new ManagementEventWatcher(scope, USBRemoveQuery);
        USBRemove.EventArrived += USBRemove_EventArrived;
        USBRemove.Start();
    }
    catch (Exception ex)
    {
        if (USBRemove != null)
        {
            USBRemove.Stop();
        }
        throw ex;
    }
}

/// <summary>
/// USB设备插入
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void USBInsert_EventArrived(object sender, EventArrivedEventArgs e)
{
    string[] tempPorts = System.IO.Ports.SerialPort.GetPortNames();
    if (tempPorts.Count() == Ports.Count())
        return;
    else
        Ports = tempPorts;

    if (IsConnected)
        return;

    if (blnDesireConnected && Open())
        commExecuteInterface?.DeviceArrivaled();
}

/// <summary>
/// USB设备拔出
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void USBRemove_EventArrived(object sender, EventArrivedEventArgs e)
{
    string[] tempPorts = System.IO.Ports.SerialPort.GetPortNames();
    if (tempPorts.Count() == Ports.Count())
        return;
    else
        Ports = tempPorts;

    if (!IsConnected)
        return;

    IsConnected = false;
    spUSB.Close();
    commExecuteInterface?.DeviceRemoved();
}
```

## 四、调用 Close 方法会提示“ unsafe handler 已关闭”

微软的方法有问题的，微软的 `SerialPort` 的这个类有问题，一但断开硬件的连接，会触发部分对象资源的释放（`SerialPort` 不是单纯的托管资源，非托管资源被释放了），因此调用 `Close` 会出问题，除非此进程完全退出，所以 `SerialPort` 的资源需要重置；`SerialPort` 的初始化方法中有一个是有 `IContainer` 参数的，`IContainer` 提供了非托管资源的管理方法，因此，把 `SerialPort` 对象放到一个容器中可解决 `Close` 问题；即，想实现热插拔，必须把 `SerialPort` 放到容器中！

 ```csharp
spUSB = new SerialPort(container);
KeyValuePair<string, string> tempPort = queueSerialPorts.Dequeue();
spUSB.BaudRate = 115200;
spUSB.PortName = tempPort.Key;

spUSB.Open();
SendMessage(EnumDataConverter.GetCommandStatus(SendCommandType.ECC));  //尝试让主控机复位

byte[] btData = ReadData();
```

## 五、IsOpen 属性并不可靠，尽量少用
