---
aliases:
date:
update:
author:
language:
sourceurl: https://www.cnblogs.com/zhaotianff/p/18233083
tags:
---

# [WPF界面反编译神器Snoop使用介绍](https://www.cnblogs.com/zhaotianff/p/18233083 "发布于 2024-06-05 16:40")

# **Snoop 介绍**

Snoop 是一款开源的 WPF 监视工具，它能够监视或浏览任何正在运行的 WPF 应用程序的可视化、逻辑和自动化树（无需调试器），还可以更改属性值、查看触发器、在属性更改时设置断点等。

项目地址：[https://github.com/snoopwpf/snoopwpf](https://github.com/snoopwpf/snoopwpf)

# **下载运行**

可以到 [github release](https://github.com/snoopwpf/snoopwpf/releases) 下载已经编译的二进制文件。

不同的 snoop 版本对应不同的.NET/.Net Framework 版本，根据自己的需求下载对应的版本使用

|Snoop|.NET Framework|.NET|
|---|---|---|
|3.0|4.0|3.0|
|4.0|4.5.1|3.0|
|5.0|4.5.2|3.1|
|6.0|4.6.2|6.0|

也可以直接导入代码编译。

**注意，这里有一个限制：不支持自包含的单个文件应用程序，因为没有可靠的方法来获取 .NET 运行时的句柄**

# **软件原理**

关于软件原理，我没有仔细的去读软件的源码，最近也比较忙。

当我用 Snoop spy 一个 WPF 程序时，在 ProcessMonitor 里查看进程模块时，会看到 Snoop.Core.dll，如下 所示：

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605143528112-1067935667.png)

所以我猜想也是利用注入的原理。

在 win32 里，可以通过 SetWindowHookEx/注入 +HookApi 之类的方式捕获调用进行 spy。

但是 WPF 没有句柄，所以这里确实是有点强大了，不知道内部是如何实现的，等后面有时间的时候再去深入了解原理。

# **软件界面介绍**

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605154352478-917811012.png)

# **如何 Spy WPF 程序**

我们这里以开源软件 [WindowsX](https://github.com/zhaotianff/WindowsX) 为例

首先我们运行 ScreenToGif 和 Snoop，然后通过上图 6 的按钮（或者在下列列表选中，再单击上图 5 的按钮），将鼠标光标拖动到 WindowsX 的界面上

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606142007731-1056829502.gif)

**Snoop 的主界面主要由如下几部分组成**

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605162644921-280716362.png)

左边是树列表，可以切换显示**Visual Tree/LogicalTree**

右边上半部分是控件的一些参数，包括**属性/数据上下文/事件/触发器**等。

右边下半部分可以看到**诊断日志和控件预览**。

# **如何快速定位到控件**

例如在 WindowsX 界面上有一个按钮，我们想快速定位到它，并查看相关参数。

我们只需要按住**Ctrl+Shift**键，然后移动鼠标到控件上即可，Snoop 的可视化树会自动选中对应的控件

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605163228465-1108902870.gif)

# **如何查看/保存控件缩略图**

把鼠标放在可视化树上，可以看到控件的缩略图

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605163537895-1797501985.gif)

如果我们想保存缩略图，在可视化树上选中后，在右下角的**Preview 页**，单击**【保存】**按钮即可

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605163657819-520556997.png)

# **查看/编辑控件属性**

在左侧的可视化树中选择后，在右边的**Properties** Tab 页可以看到控件的属性

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605164424353-1666313618.png)

**搜索栏可以快速搜索属性**

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605164736227-352947224.png)

**属性过滤**可以将属性进行过滤，例如，我想查看颜色相关的属性，就切换为 Color，想查看布局相关的属性，就切换为 Layout。

还可以自己创建属性过滤器

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240605164644501-1878857382.png)

选中属性后，可以对属性值进行**编辑**，编辑后界面会实时更新。

右键菜单选择**Delve/或者双击**可以查看属性详细值。

通过这个小三角按钮可以返回上一级

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606144218968-338109868.png)

# **查看控件 DataContext**

在**DataContext** Tab 页，可以看到当前在左侧选中控件绑定的**DataContext**，

使用 MVVM 模式开发的 WPF 程序，在这个标签页可以快速查看 ViewModel 中的属性和命令等。

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606142242577-288414657.png)

可以通过上面的链接定位到 ViewModel 类所在的文件，再借用 ILSpy/.Net Reflector 之类的反编译工具，可以查看 ViewModel 代码

右键选择 Delve 或者双击属性，可以查看属性值 

例如这里的 StudentList 是一个 ObservableCollection<T>类型，我们用 delve 功能，可以看到集合内部的数据

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606143109595-2056272159.gif)

 还可以对这些值进行编辑，编辑后，界面会实时更新

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606143253400-299851727.gif)

# **查看控件事件**

Events Tab 页下可以列出程序的所有路由事件，这样我们就可以查看这些事件是如何路由的，以及这些事件是否被处理，被处理的路由事件会标识为绿色。

**注意：这个标签页是针对整个应用程序的，你无法单独查看某个控件的事件。**

g![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606144643683-121354922.png)

# **查看控件触发器**

在**Triggers**页，可以查看当前选中控件的触发器，如下所示

这里只能查看，不能修改

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606144922973-126965198.png)

# **查看控件的行为**

这里指的是使用了**Microsoft.Xaml.Behaviors.Wpf**包里的**Behaviour**相关的功能。

例如窗口 XAML 如下

![复制代码](https://assets.cnblogs.com/images/copycode.gif)

 1 <Window x:Class="WpfApp1.MainWindow" ... 2 xmlns:behaviour="http://schemas.microsoft.com/xaml/behaviors"
 4 mc:Ignorable="d"
 5 Title="MainWindow" Height="450" Width="800">
 6 <behaviour:Interaction.Behaviors>
 7 <behaviour:MouseDragElementBehavior></behaviour:MouseDragElementBehavior>
 8 </behaviour:Interaction.Behaviors>
 9 <Grid>
10 </Grid>

![复制代码](https://assets.cnblogs.com/images/copycode.gif)

在 Snoop 里面 spy 后，在 Behaviours 标签页显示如下

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606150045100-1324849412.png)

如果不了解 WPF 的行为，可以查看 [https://github.com/Microsoft/XamlBehaviors/wiki](https://github.com/Microsoft/XamlBehaviors/wiki)

XAML Behaviors 是一种简单易用的方法，能以最少的代码为应用程序添加常用和可重复使用的交互性。

# **调用函数**

**Methods**标签页允许你快速调用当前选中的控件的函数，例如，可以调用**Hide()**函数，隐藏当前窗口。

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606151631311-849051916.png)

 或者调用**RaiseEvent()**手动触发事件

![](https://img2024.cnblogs.com/blog/826348/202406/826348-20240606151748928-542206269.png)

# **总结** 

当我们在网上看到一个好看的 WPF 程序，想学习而没有源码时，就可以用到 Snoop。

利用 Snoop 可以看一下界面是如何布局的，控件样式是怎么样的，还可以查看某个界面对应的 ViewModel 叫什么，并定位到 dll 文件。

最后再借助反编译工具，可以直接提取 WPF 里的资源，包括 XAML 和图片等。

Snoop 最初是由 [Pete Blois](https://github.com/peteblois) 创建，现在交由 [Bastian Schmidt](https://github.com/batzen) 维护，感谢两位大佬贡献这么强大的项目。

life runs on code

作者： [zhaotianff](http://www.cnblogs.com/zhaotianff/)

转载请注明出处
