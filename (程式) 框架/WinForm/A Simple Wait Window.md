---
aliases:
date: 2010-03-05
update:
author: The Man from U.N.C.L.E.
language: C#
sourceurl: https://www.codeproject.com/articles/A-Simple-Wait-Window
tags:
  - CSharp
  - WinForm
---

# Introduction  介紹

Back in the days when I used FoxPro, we would always pop up a wait window while running any long process. Come .NET, there is no built in wait window, so you have to roll your own. This has become an increasing issue as we issue calls to web services that may be fast or slow. In the mean time, our users need to know the application has not crashed. They should also not be able to click on any other part of the application until the process has finished, in the same manner as the old FoxPro wait window.
以前我用 FoxPro 的時候，運行任何耗時較長的進程都會彈出一個等待視窗。但到了 .NET，它就沒有內建的等待視窗功能了，所以你得自己實作。隨著我們呼叫的 Web 服務速度忽快忽慢，這個問題變得越來越突出。用戶需要知道應用程式沒有崩潰，而且在進程完成之前，他們也應該像以前的 FoxPro 等待視窗一樣，無法點擊應用程式的任何其他部分。

![waitwindow.png](https://www.codeproject.com/_next/image?url=https%3A%2F%2Fcloudfront.codeproject.com%2Fprogress%2Fwaitwindow%2Fwaitwindow.png&w=1080&q=75)

# Background  背景

There are a few details that need to be covered for a good wait window.
要確定一個合理的等待時間，需要考慮一些細節。

During our process, we want to provide an animation on the wait window to ensure the user believes the application has not crashed. For the purposes of this article, I have used a `ProgessBar`, but you could equally use an animated GIF, or some custom animation scheme of your own devising. The side effect is that the worker method must execute on a secondary thread or it will block the GUI updating the animation.
在我們的流程中，我們希望在等待視窗上添加動畫，以確保用戶相信應用程式沒有崩潰。本文中我使用了 `ProgessBar` ，但您也可以使用動畫 GIF 或您自己設計的自訂動畫方案。需要注意的是，工作方法必須在輔助執行緒中執行，否則會阻塞 GUI 更新動畫。

Also, we cannot use a `BackgroundWorker` for this as we want the execution to occur inline. For example, we want to call the wait window specifying the method to execute, then expect it to have finished by the time we hit the next line of code. Therefore we will have to roll our own threading code.
此外，我們不能使用 `BackgroundWorker` ，因為我們希望程式碼內聯執行。例如，我們希望呼叫等待視窗並指定要執行的方法，然後期望在執行下一行程式碼時該方法已經完成。因此，我們需要自己編寫線程程式碼。

# Using the Code  使用程式碼

I have built the sample code with the wait window in a separate assembly so you can just reference the assembly if you want. However, I expect most people will copy the classes into their own project structure, which is fine.
我已經將包含等待視窗的範例程式碼放在一個單獨的程式集中，您可以根據需要直接引用該程式集。不過，我估計大多數人會將這些類別複製到自己的專案結構中，這也沒問題。

Before anyone asks, the sample code includes the project in C# and in VB.NET, and while I have compiled it against the 3.5 framework, it should be fine built against any .NET Framework version, though you would have to manually copy the code into a Visual Studio 2005 project if you wanted to use the older solution format.
在有人問之前，範例程式碼包括 C# 和 VB.NET 項目，雖然我是針對 3.5 框架編譯的，但它應該可以針對任何 .NET Framework 版本構建，不過如果您想使用舊版解決方案格式，則必須手動將程式碼複製到 Visual Studio 2005 專案中。

First, you will need to write your method to execute. Remember this is running on a separate thread, so pass everything in and return the results, or ensure any calls you make are guaranteed thread safe!
首先，你需要寫要執行的方法。記住，該方法運行在單獨的線程中，所以要傳入所有參數並返回結果，或者確保你進行的任何呼叫都是線程安全的！

To make things easier, the event args for the method include a collection of arguments that you can pass in, and a reference to the `WaitWindow` object so that the message displayed can be updated during the process. The code below shows most of the things you may want to do. For more examples, see the demo project.
為了簡化操作，此方法的事件參數包含一個可傳入的參數集合，以及對 `WaitWindow` 物件的引用，以便在處理過程中更新顯示的訊息。以下程式碼展示了您可能需要執行的大部分操作。更多範例，請參閱演示項目。

CS

```cs
private void WorkerMethod(object sender, Jacksonsoft.WaitWindowEventArgs e){
	// Do something
	for (int progress = 1; progress <= 100; progress++){
		System.Threading.Thread.Sleep(20);
		
		// Update the wait window message
		e.Window.Message = string.Format
		("Please wait ... {0}%", progress.ToString().PadLeft(3));
	}
	
	// Use the arguments sent in
	if (e.Arguments.Count> 0){
		// Set the result to return
		e.Result = e.Arguments[0].ToString();
	} else {
		// Set the result to return
		e.Result = "Hello World";
	}
}
```

Next, you need to call the `WaitWindow` with the method you have created. The example below just uses the default, "Please wait..." message, but you can also pass in the message to display, and any arguments for the worker method. Again, there are more examples in the demo project.
接下來，你需要使用你所建立的方法來呼叫 `WaitWindow` 。下面的範例使用了預設的「請稍候…」訊息，但你也可以傳入要顯示的訊息以及任何工作方法的參數。演示項目中還有更多範例。

CS

```cs
object result = WaitWindow.Show(this.WorkerMethod);
```

There are two methods exposed by the `WaitWindow` object that can be called from within the worker method. These are the methods, `Cancel()` and the property, `Message`. These cancel the execution immediately, and update the message displayed in the `WaitWindow` respectively.
`WaitWindow` 物件公開了兩個方法，可以在 worker 方法內部呼叫。這兩個方法分別是 `Cancel()` 方法和 `Message` 屬性。它們分別用於立即取消執行和更新 `WaitWindow` 中顯示的訊息。

# Points of Interest  景點

In the first release, I did some clever stuff to make a modeless window appear modal, however it was pointed out that I did not actually need this at all.
在第一個版本中，我做了一些巧妙的事情，讓非模態視窗看起來像模態窗口，但是有人指出我實際上根本不需要這樣做。

If you do want to do this for some other reason, you have to capture the mouse and then capture all the windows messages and ensure that only the ones you want get passed on to the rest of the application.
如果你出於其他原因想要這樣做，則必須捕獲滑鼠，然後捕獲所有視窗訊息，並確保只有你想要的訊息才能傳遞給應用程式的其餘部分。

We pass the `Form` into the `AddMessageFilter` method to hook up the form to receive all the windows messages first.
我們將 `Form` 傳遞給 `AddMessageFilter` 方法，以便將窗體連接到接收所有 Windows 訊息的介面。

CS

```cs
System.Windows.Forms.Application.AddMessageFilter(this._GUI);
```

Inside the code for the window, we can now implement the `IMessageFilter` interface giving us a `PreFilterMessage` method. All that remains is to return `true` for the mouse and keyboard events, rather than the default `false` and the window will now act as if it were modeless when in fact it is modal.
現在，在視窗的程式碼中，我們可以實作 `IMessageFilter` 接口，從而獲得 `PreFilterMessage` 方法。接下來，只需將滑鼠和鍵盤事件的反應值從預設的 `false` 改為 `true` ，視窗就會像非模態視窗一樣運行，而實際上它是模態的。

A shame that I ditched that rather neat piece of code because it was a pointless waste of time, but that's development for you, a constant learning process. 
很可惜我放棄了那段相當簡潔的程式碼，因為它毫無意義，純屬浪費時間。但這就是開發，一個不斷學習的過程。

# History  歷史

- First release - March 2010
    首次發布 - 2010 年 3 月
- Release 2 - Updated for simpler threading model (Many thanks to [John Brett](http://www.codeproject.com/script/Membership/View.aspx?mid=1006534) for his input)
    版本 2 - 更新為更簡單的線程模型（非常感謝 [John Brett 的](http://www.codeproject.com/script/Membership/View.aspx?mid=1006534) 貢獻）
- Release 3 - Updated to support cancellation and better exception handling
    版本 3 - 更新以支援取消操作和改進異常處理

# License  執照

This article, along with any associated source code and files, is licensed under [The Code Project Open License (CPOL)](http://www.codeproject.com/cpol)
本文及其相關原始碼和文件均根據 [Code Project 開源許可證 (CPOL)](http://www.codeproject.com/cpol) 授權。

[Download source - 23.04 KB](https://api-main.codeproject.com/v1/article/waitwindow/downloadAttachment?src=waitwindow/waitwindow.zip)
