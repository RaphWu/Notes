---
aliases:
date:
update:
author:
language:
sourceurl: https://blog.csdn.net/qq_44695769/article/details/136224344
tags:
---

前言
WPF 虽然自己本身自带调试工具，但是那个调试工具相对来说功能有点少，我这里会对 Visual Studio 原生的调试工具和第三方调试工具 Snoop 进行比较

运行环境
window10
visual studio 2022
.net core 8.0
简单的 WPF 代码
我这里用了 CommunityToolkit.MVVM

实现一个简单的 ListBox
ViewModel

 public class TestViewModel
 {
     public record Person(int Id,string Name,string Descirption);

     public List<Person> People => new List<Person>()
     {
         new Person(1,"小王","王哥"),
         new Person(2,"小帅","大帅比"),
         new Person(3,"小美","美美的")
     };


     public TestViewModel() { }
 }
AI 写代码
csharp
运行

1
2
3
4
5
6
7
8
9
10
11
12
13
14
<UserControl x:Class="WpfSnoopDemo.Views.TestView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:local="clr-namespace:WpfSnoopDemo.Views"
             mc:Ignorable="d"
             xmlns:viewModels="clr-namespace:WpfSnoopDemo.ViewModels"
             d:DesignHeight="450" d:DesignWidth="800">
    <UserControl.DataContext>
        <viewModels:TestViewModel />
    </UserControl.DataContext>
    <Grid>
        <ListBox ItemsSource="{Binding People}">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Vertical">
                        <!--这个是一种仿CSS的写法-->
                        <StackPanel.Resources>
                            <Style TargetType="StackPanel">
                                <Setter Property="Orientation"
                                        Value="Horizontal" />
                            </Style>
                        </StackPanel.Resources>
                        <StackPanel>
                            <TextBlock Text="Id:" />
                            <TextBlock Text="{Binding Id}" />
                        </StackPanel>
                        <StackPanel>
                            <TextBlock Text="Name:" />
                            <TextBlock Text="{Binding Name}" />
                        </StackPanel>
                        <StackPanel>
                            <TextBlock Text="Descirption:" />
                            <TextBlock Text="{Binding Descirption}" />
                        </StackPanel>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Grid>
</UserControl>

AI 写代码
xml

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43

Visual Studio 自带代码调试

热重置功能测试
热重置的意思是编译运行之后修改源代码，能通过重载而不用重新编译就能看到新修改的效果。

实时可视化树

序号	用途
1	显示 WPF Debug 运行调试工具
2	选择元素
3	显示装饰器，就是个十字坐标定位，显示盒子模型
4	跟踪具有焦点的元素，暂时不知道有啥用
5	显示绑定问题
6	辅助扫描，没啥用
7	预览选定项，不知道有啥用
8	活动文档查找元素。就是你鼠标选择了哪个，点这个可以跳转到鼠标选中的对应结构
9	显示对应元素属性
10	展开树结构
11	压缩树结构
12	只显示自己的代码
WPF Debug 窗口是部分工具展示，这里就不展开说明了。

查找窗口元素

显示属性

这里面会显示所有对应的属性

也可以看 DataContext

Snoop 调试使用
WPF Snoop Github 地址

Snoop 下载地址

下载好了直接安装

双击运行之后出现这个界面

Snoop 简单使用
关于 Snoop 的用法

打开 Snoop 我们可以看到这几个按钮

序号	含义	使用情况
1	选择正在运行的 WPF 窗口	一般不用
2	刷新找到的 WPF 窗口	一般不用
3	在【1】选择好对象后，创建一个 Snoop 克隆	一般不用
4	拖动准星，选择 WPF 窗口，实现【3】效果	一般不用
5	创建一个 Snoop 可克隆对象并添加【调试控制台】	常用
6	在【4】的基础上面创建【调试控制台】	常用
7	设置	一般默认即可
8	缩小
9	关闭
调试控制台
我们在使用【5】/【6】的时候，会生成如下的调试窗口

序号	功能
1	结构树
2	配置文件
3	设置
4	窗口元素追踪，快捷键：Ctrl+Shift+ 鼠标移动
5	断点调试
6	版本
7	主题，有黑色和白色
元素追踪

使用 Ctrl+Shift 选中元素，由于我 GIF 录屏会有窗口遮挡，所以有点不连贯。

有时候选择多了会出现这个 Bug，我们点击清空即可

结构树

我们先选中一个元素

鼠标停留在对应的树节点上面，会显示对应的可视化元素值

Visual/可视化结构树
可视化结构树就是里面所有的基础控件元素，和我们 F12 跳出来的 Html 控制台的结果差不多

Logical/本地代码可视化树

AutoMation/自动识别结构树
自动识别处于两种之间，自动识别我们自己本地的代码

WPF 元素控制台

序号	用处
1	元素属性
2	元素上下文
3	元素 Event 事件，一般是鼠标事件
4	元素触发器
5	元素行为
6	元素方法
7	Debug 监听器
这里用法太多了，我们就不展开说明了

我们也可以实时修改对应的元素，但是感觉用处不是很大，因为 WPF 已经支持热重载了。

结论
Snoop 算是 Visual Studio 的补充，Visual Studio 本身的代码调试就已经是非常的惊艳了。其它的玩法我也在摸索当中。Snoop 算是浏览器的 F12，你是用来查看元素的，不是直接改 Html 结果的。这个是一个很好的代码调试的作用和对元素 Visual Studio 的补充。
