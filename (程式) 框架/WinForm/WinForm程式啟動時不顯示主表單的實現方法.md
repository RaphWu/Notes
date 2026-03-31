---
aliases:
date: 2023-05-25
update:
author: OneBaBa
language: CSharp
sourceurl: https://blog.csdn.net/CSDNwei/article/details/72769926
tags:
  - CSharp
  - WinForm
---

# C# WinForm 程序启动时不显示主窗体的实现方法

本文介绍了一种在 C#中通过自定义 `ApplicationContext` 实现窗体应用程序的启动和关闭逻辑的方法。该方法允许在主窗体未完全加载前进行特定处理，并确保在适当时候正确地关闭程序。
首先我们需要知道 `ApplicationContext` 实质上就是一个 Application 与主窗体之间的连接器，掌管着二者之间的互动关系。
其中最主要的，就是负责在主窗体关闭时结束线程。既然如此，我们只要根据需要自定义一个 `ApplicationContext` 就可以了：
C# Code 如下

```csharp
internal class HideOnStartupApplicationContext : ApplicationContext
{
    private Form mainFormInternal;

    public HideOnStartupApplicationContext(Form mainForm)
    {
        this.mainFormInternal = mainForm;
        this.mainFormInternal.FormClosed += new FormClosedEventHandler(mainFormInternal_FormClosed);
    }

    void mainFormInternal_FormClosed(object sender, FormClosedEventArgs e)
    {
        Application.Exit();
    }
}
```

接下来在 `Main` 函数里面修改 Form 的启动方式

```csharp
Application.Run(new HideOnStartupApplicationContext(new MainForm()));
```

网上找到的资料就是这样了. 但实际上这个方法有点问题, 我也觉得是优点的部分. 那就是因为现在的 `MianForm` 实际上还未显示. 包括 `Load` 事件都还没有触发.. 这时 `MainForm` 中直接使用 `Close()` 方法根本就无法触发事件. 必须要先使用 `this.Show()` 以后才行.
所以在主窗体我们还需要一些处理. 这里要用到一个 `Load` 事件, 以及其他地方的关闭窗体时的处理.
在 `Load` 事件中做如下处理

```csharp
bool isLoaded = false;
private void MainForm_Load(object sender, EventArgs e)
{
    isLoaded = true;
}
```

我这里是 `NotifyBar` 中一个退出按钮的事件. 处理如下

```csharp
private void CloseToolStripMenuItem_Click(object sender, EventArgs e)
{
    if (isLoaded)
    {
        mainNotifyBar.Dispose();
        this.Close();
    }
    else
    {
        Application.Exit();
    }
}
```

稍微解释下. 其实就是因为窗体还未创建. 所以需要在 `Load` 事件来判断是否已经加载了. 如果还没有加载则在退出的时候直接退出程序, 当然因为我这有 `NotifyBar` 的缘故, 同时需要释放这个控件, 不然右下角这个程序的图标不会主动消失. 当然如果启动了还是老样子. 调用 `Close` 方法来退出就可以了.

这里也顺带把另一种方法写下. 如果窗口即使不显示 还是想要能先加载完成就可以按如下方法. 也比较简单
设置主窗体的如下属性就可以了.

```csharp
this.ShowInTaskbar = false;
this.WindowState = FormWindowState.Minimized;
```

当然需要显示的时候还是需要按照自己的需求设置回来.
