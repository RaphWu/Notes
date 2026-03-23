---
aliases:
date: 2014-05-07
update:
author: Ben Voigt
language:
sourceurl: https://sparxeng.com/blog/software/must-use-net-system-io-ports-serialport
tags:
  - CSharp
  - SerialPort
---

# If you *must* use .NET System.IO.Ports.SerialPort

As an embedded developer who writes desktop software mostly for configuration of, and data download from, peripheral devices, I use serial data streams a lot. Mostly USB virtual serial posts [from FTDI](http://www.ftdichip.com/Products/ICs.htm), but also the USB Communication Device Class and real 16550-compatible UARTs on the PCI bus. Since looking at data through an in-circuit emulator debug interface is generally a miserable experience, getting serial data communication with a custom PC application is essential to analyzing data quality and providing feedback on hardware designs. C# and the .NET Framework provide a rapid application development that is ideal for early development that needs to track changing requirements as hardware designs evolve. Ideal in most respects, I should say.
作為嵌入式開發人員，我編寫的桌面軟體主要用於配置外圍設備並從外圍設備下載數據，因此我經常使用串行數據流。主要是來自 FTDI 的 USB 虛擬串行端口，但也使用 USB 通信設備類和 PCI 總線上兼容 16550 的真正 UART。由於透過線上模擬器調試介面查看數據通常體驗不佳，因此使用自訂 PC 應用程式進行串行數據通訊對於分析數據品質和提供硬體設計反饋至關重要。 C# 和 .NET Framework 提供了一種快速的應用程式開發方法，非常適合需要追蹤硬體設計演進過程中需求變化的早期開發。應該說，它在大多數方面都堪稱理想之選。

The [System.IO.Ports.SerialPort](https://learn.microsoft.com/zh-tw/dotnet/api/system.io.ports.serialport) class which ships with .NET is a glaring exception. To put it mildly, it was designed by computer scientists operating far outside their area of core competence. They neither understood the characteristics of serial communication, nor common use cases, and it shows. Nor could it have been tested in any real world scenario prior to shipping, without finding flaws that litter both the documented interface and the undocumented behavior and make reliable communication using System.IO.Ports.SerialPort (henceforth IOPSP) a real nightmare. (Plenty of evidence on StackOverflow attests to this, from devices that work in Hyperterminal but not .NET because [IOPSP makes setting certain parameters mandatory, although they aren’t applicable to virtual ports, and closes the port on failure.](https://stackoverflow.com/questions/14885288/i-o-exception-error-when-using-serialport-open) There’s no way to bypass or ignore failure of these settings during IOPSP initialization.)
.NET 隨附的 `System.IO.Ports.SerialPort` 類別是一個明顯的例外。委婉地說，它是由一些電腦科學家設計的，他們的工作遠遠超出了他們的核心能力範圍。他們既不了解串行通訊的特性，也不了解常見的用例，這一點顯而易見。在發布之前，它也無法在任何實際場景中進行測試，因為沒有發現任何缺陷，這些缺陷不僅存在於已記錄的介面中，也存在於未記錄的行為中，使得使用 `System.IO.Ports.SerialPort`（以下簡稱 IOPSP）進行可靠的通訊成為一場真正的噩夢。 （StackOverflow 上有大量證據證明了這一點，這些證據來自在超級終端中工作但不在 .NET 中運行的設備，因為 IOPSP 強制設置某些參數，儘管這些參數不適用於虛擬端口，並且在端口失敗時會關閉。在 IOPSP 初始化期間，無法繞過或忽略這些設置的失敗。）

What’s even more astonishing is that this level of failure occurred when the underlying kernel32.dll APIs are immensely better (I’ve used the WinAPI before working with .NET, and still do when I want to use a function that .NET doesn’t have a wrapper for, which notably includes device enumeration). The .NET engineers not only failed to devise a reasonable interface, they chose to disregard the WinAPI design which was very mature, nor did they learn from two decades of kernel team experience with serial ports.
更令人震驚的是，在底層 kernel32.dll API 效能大幅提升的情況下，這種程度的失敗竟然發生了（在使用 .NET 之前，我曾使用過 WinAPI，現在仍然在使用 .NET 尚未封裝的函數時使用，尤其是設備枚舉）。 .NET 工程師不僅未能設計出合理的接口，還選擇無視於已經非常成熟的 WinAPI 設計，也沒有汲取核心團隊二十年串口開發經驗的教訓。

A future series of posts will present the design and implementation of a rational serial port interface built upon, and preserving the style of, the WinAPI serial port functions. It fits seamlessly into the .NET event dispatch model, and multiple coworkers have expressed that it’s exactly how they want a serial-port class to work. But I realize that external circumstances sometimes prohibit using a C++/CLI mixed-mode assembly. The C++/CLI solution is incompatible with:
後續的一系列文章將介紹如何設計和實現一個合理的串行端口接口，該接口基於 WinAPI 串行端口函數構建，並保留其風格。它與 .NET 事件調度模型無縫銜接，許多同事也表示，這正是他們希望串行埠類別能夠運作的方式。但我意識到，外部環境有時會限制使用 C++/CLI 混合模式彙編。 C++/CLI 解決方案與以下程式碼不相容：

- Partial trust (not really a factor, since IOPSP’s Open method also demands Unmanaged Code permission)
  部分信任（不是一個真正的因素，因為 IOPSP 的 Open 方法也需要非託管程式碼權限）
- Single-executable deployment (there may be workarounds involving ILMerge or using netmodules to link the C# code into the C++/CLI assembly)
  單一可執行部署（可能有涉及 ILMerge 或使用 netmodules 將 C# 程式碼連結到 C++/CLI 組件的解決方法）
- Development policies that prohibit third-party projects
  禁止第三方專案的開發政策
- .NET Compact Framework (no support for mixed-mode assemblies)
  .NET Compact Framework（不支援混合模式組件）
The public license (as yet undetermined) might also present a problem for some users.
公共許可證（尚未確定）也可能給一些用戶帶來問題。

Or maybe you are responsible for improving IOPSP code that is already written, and the project decision-maker isn’t ready to switch horses. (This is not a good decision, the headaches IOPSP will cause in future maintenance far outweigh the effort of switching, and you’ll end up switching in the end to get around the unfixable bugs.)
或者，你負責改進已經編寫好的 IOPSP 程式碼，而專案決策者還沒準備好更換專案。 （這不是一個好的決定，IOPSP 在未來維護中帶來的麻煩遠遠超過更換專案的努力，你最終還是會為了解決那些無法修復的錯誤而更換專案。）

So, if you fall into one of these categories and using the Base Class Library is mandatory, you don’t have to suffer the worst of the nightmare. There are some parts of IOPSP that are a lot less broken that the others, but that you’ll never find in MSDN samples. (Unsurprisingly, these correspond to where the .NET wrapper is thinnest.) That isn’t to say that all the bugs can be worked around, but if you’re lucky enough to have hardware that doesn’t trigger them, you can get IOPSP to work reliably in limited ways that cover most usage.
因此，如果您屬於上述類別之一，並且必須使用基類庫，那麼您無需承受最嚴重的噩夢。 IOPSP 中有些部分的問題比其他部分少得多，但您在 MSDN 範例中絕對找不到它們。 （不出所料，這些部分對應於 .NET 包裝器最薄的部分。）這並不是說所有錯誤都可以解決，但如果您足夠幸運，擁有不會觸發這些錯誤的硬件，那麼您可以讓 IOPSP 在有限的幾種情況下可靠地工作，從而滿足大多數用途。

I planned to start with some guidance on how to recognize broken IOPSP code that needs to be reworked, and thought of giving you a list of members that should not be used, ever. But that list would be several pages long, so instead I’ll list just the most egregious ones and also the ones that are safe.
我原本計劃先提供一些關於如何識別需要返工的 IOPSP 代碼的指導，並考慮列出一些永遠不應該使用的成員。但這份清單會長達數頁，所以我只列出最嚴重的問題以及安全的問題。

**The worst offending System.IO.Ports.SerialPort members**, ones that not only should not be used but are signs of a deep code smell and the need to rearchitect all IOPSP usage:
最嚴重的問題在於 `System.IO.Ports.SerialPort` 成員，它們不僅不應該被使用，而且是嚴重程式碼異味的標誌，需要重新建構所有 IOPSP 的使用：

- The `DataReceived` event (100% redundant, also completely unreliable 100% 冗餘，也完全不可靠)
- The `BytesToRead` property (completely unreliable)
- The `Read`, `ReadExisting`, `ReadLine` methods (handle errors completely wrong, and are synchronous 處理錯誤完全錯誤，並且是同步的)
- The `PinChanged` event (delivered out of order with respect to every interesting thing you might want to know about it 你可能想知道的每一件有趣的事情都是亂序傳遞的)

**Members that are safe to use:**

- The mode properties: `BaudRate`, `DataBits`, `Parity`, `StopBits`, but only before opening the port. And only for standard baud rates.
- Hardware handshaking control: the `Handshake` property
- Port selection: constructors, `PortName` property, `Open` method, `IsOpen` property, `GetPortNames` method
And the one member that no one uses because MSDN gives no example, but is **absolutely essential** to your sanity:
由於 MSDN 沒有提供範例，因此沒有人使用這個成員，但這對於您的理智來說絕對是必不可少的：
- The `BaseStream` property

The only serial port read approaches that work correctly are accessed via BaseStream. Its implementation, the System.IO.Ports.SerialStream class (which has internal visibility; you can only use it via Stream virtual methods) is also home to the few lines of code which I wouldn’t choose to rewrite.
唯一能正常運作的串行埠讀取方法是透過 `BaseStream` 存取的。它的實現，`System.IO.Ports.SerialStream` 類別（具有內部可見性；只能透過 `Stream` 虛方法使用它）也包含幾行我不想重寫的程式碼。

Finally, some code.

Here’s the (wrong) way the examples show to receive data:

```csharp
port.DataReceived += port_DataReceived;

// (later, in DataReceived event)
try {
    byte[] buffer = new byte[port.BytesToRead];
    port.Read(buffer, 0, buffer.Length);
    raiseAppSerialDataEvent(buffer);
}
catch (IOException exc) {
    handleAppSerialError(exc);
}
```

Here’s the right approach, which matches the way the underlying Win32 API is intended to be used:
這是正確的方法，它與底層 Win32 API 的使用方式相符：

```csharp
byte[] buffer = new byte[blockLimit];
Action kickoffRead = null;
kickoffRead = delegate {
    port.BaseStream.BeginRead(buffer, 0, buffer.Length, delegate (IAsyncResult ar) {
        try {
            int actualLength = port.BaseStream.EndRead(ar);
            byte[] received = new byte[actualLength];
            Buffer.BlockCopy(buffer, 0, received, 0, actualLength);
            raiseAppSerialDataEvent(received);
        }
        catch (IOException exc) {
            handleAppSerialError(exc);
        }
        kickoffRead();
    }, null);
};
kickoffRead();
```

It looks like a little bit more, and more complex code, but it results in far fewer p/invoke calls, and doesn’t suffer from the unreliability of the `BytesToRead` property. (Yes, the `BytesToRead` version can be adjusted to handle partial reads and bytes that arrive between inspecting `BytesToRead` and calling `Read`, but those are only the most obvious problems.)
它的程式碼看起來稍微多了一些，也更複雜了，但它可以顯著減少 p/invoke 呼叫次數，並且不會受到 `BytesToRead` 屬性不可靠性的影響。 （沒錯，`BytesToRead` 版本可以進行調整，以處理部分讀取以及在檢查 `BytesToRead` 和調用 `Read` 之間到達的字節數，但這些只是最明顯的問題。）

Starting in .NET 4.5, you can instead call ReadAsync on the `BaseStream` object, which calls `BeginRead` and `EndRead` internally.
從 .NET 4.5 開始，您可以改為在 `BaseStream` 物件上呼叫 `ReadAsync`，它會在內部呼叫 `BeginRead` 和 `EndRead`。

Calling the Win32 API directly we would be able to streamline this even more, for example by reusing a kernel event handle instead of creating a new one for each block. We’ll look at that issue and many more in future posts exploring the C++/CLI replacement.
直接呼叫 Win32 API 可以進一步簡化這個過程，例如，透過重複使用核心事件句柄，而不是為每個區塊建立一個新的句柄。我們將在後續探討 C++/CLI 替代方案的文章中探討這個問題以及更多其他問題。

---

Ben Voigt says:
September 25, 2014 at 12:57 pm
Reading from the `BaseStream` will get the data to your application with low overhead. But that’s generally a small part of the processing cost; ultimately how efficiently your serial processing is depends on your code that buffers, packetizes, and parses it.

從 `BaseStream` 讀取資料可以以較低的開銷將資料傳送到您的應用程式。但這通常只佔處理成本的一小部分；最終，==串行處理的效率取決於緩衝、打包和解析資料的程式碼。==

---

dominikjeske says:
October 6, 2014 at 9:25 pm
Can You give full example on sending a message and receiving a response using BaseStream. I have tried this but it is not working for me? Sample would be very helpful
Can You give full example on sending a message and receiving a response using BaseStream. I have tried this but it is not working for me? Sample would be very helpful

您能否舉出有關使用 BaseStream 發送消息和接收響應的完整示例。我已經嘗試過這個，但它對我不起作用？樣品會很有幫助

Marya says:
October 6, 2014 at 9:47 pm
This might be useful:

for writing, you can use

```csharp
await sp.BaseStream.WriteAsync(buffer ,0, buffer.Length);
```

for reading,

```csharp
byte[] buffer = new byte [7];
int read = 0;
while (read < 7)
	read += await sp.BaseStream.ReadAsync(buffer , read, 7 – read);
```

these commands should be used within a function that uses "async" as a modifier. For example:

```csharp
private async void DoSomething()
{
	// Read or write like explained earlier
}
```

---

Marya says:
October 21, 2014 at 7:50 pm
Is there any way the we read from the buffer without knowing how long is the buffer? Thanks

我們有什麼辦法在不知道緩衝區有多長的情況下從緩衝區讀取？謝謝

Ben Voigt says:
October 21, 2014 at 9:24 pm
Sure, if you set a read timeout of zero and issue a read request, you’ll get as much data transferred to your buffer as is already received (up to the limit of the size you made your buffer). This is a much better approach than querying the buffer size. In general, just getting the data is more efficient and involves fewer race conditions than asking questions about the data and then retrieving it.

當然，如果您將讀取逾時設定為零並發出讀取請求，您將獲得傳輸到緩衝區的資料量與已收到的資料量一樣多（最多是您為緩衝區設定的大小限制）。這是比查詢緩衝區大小更好的方法。一般而言，僅取得資料比詢問有關資料然後擷取資料更有效率，而且涉及的競爭條件更少。

---

Marya says:
October 22, 2014 at 5:40 pm
As far as I know, readAsync or WriteAsync does not support Readtimeout or Writetimeout. Even creating a manual timeout and setting a condition would not work since await operator is sitting there to get more data, although reading task is complete.

據我所知，readAsync 或 WriteAsync 不支援 Readtimeout 或 Writetimeout。即使創建手動超時並設置條件也不起作用，因為等待運算子坐在那裡獲取更多數據，儘管讀取任務已完成。

Ben Voigt says:
October 22, 2014 at 5:53 pm
“As far as I know” leads to false assumptions. You can avoid bad assumptions by either testing, or inspecting the implementation logic (JetBrains dotPeek and Red Gate .NET Reflector are good options here). It seems like you did neither.

「據我所知」會導致錯誤的假設。您可以透過測試或檢查實作邏輯來避免錯誤的假設 （JetBrains dotPeek 和 Red Gate .NET Reflector 是不錯的選擇）。看來你兩者都沒有做。

There are no serial port functions in the Win32 API that don’t respect timeout. The only way the .NET wrapper would not finish on timeout is if it detected a partial read and looped to read the rest. But I used dotPeek to verify that there is no such logic.

Win32 API 中沒有不遵守逾時的序列埠函式。.NET 包裝函式不會在逾時完成的唯一方式是偵測到部分讀取並迴圈讀取其餘部分。但我用 dotPeek 驗證了沒有這樣的邏輯。

Stream’s ReadAsync is just a wrapper around BeginRead/EndRead, and that returns once a Win32 ReadFile operation completes. There’s no loop to fill the entire buffer. A timeout will cause the ReadFile operation to complete, which means SerialStream.EndRead is called, and Stream.ReadAsync’s task completes also.

Stream 的 ReadAsync 只是 BeginRead/EndRead 的包裝函式，一旦 Win32 ReadFile 作業完成，就會傳回。沒有循環來填充整個緩衝區。逾時會導致 ReadFile 作業完成，這表示會呼叫 SerialStream.EndRead，而且 Stream.ReadAsync 的工作也會完成。

Now, it is true that SerialStream.ReadTimeout is not as flexible as Win32 SetCommTimeouts. But it does support the desired combination “A (ReadIntervalTimeout) value of MAXDWORD, combined with zero values for both the ReadTotalTimeoutConstant and ReadTotalTimeoutMultiplier members, specifies that the read operation is to return immediately with the bytes that have already been received, even if no bytes have been received.” You get that from setting SerialStream.ReadTimeout=0.

現在，SerialStream.ReadTimeout 確實不如 Win32 SetCommTimeouts 靈活。但它確實支援所需的組合「MAXDWORD 的 （ReadIntervalTimeout） 值，結合 ReadTotalTimeoutConstant 和 ReadTotalTimeoutMultiplier 成員的零值，指定讀取作業會立即傳回已收到的位元組，即使尚未收到任何位元組也一樣。」您可以透過設定 SerialStream.ReadTimeout=0 來取得此值。

---

Michael A Katz says:
March 7, 2019 at 11:34 pm
When you say `DataReceived` and `BytesToRead` are “completely unreliable” — can you detail what you mean by that. I see you say lower that you can call BytesToRead and then when you actually go to read the bytes there are more in the buffer than BytesToRead reported (because they came in between the BytesToRead call and the data retrieve call). But besides that, what are the other gotchas?
當您說 `DataReceived` 和 `BytesToRead` “完全不可靠”時，您能詳細說明一下您的意思嗎？我看到你說你可以調用 `BytesToRead`，然後當你真正去讀取位元組時，緩衝區中的位元組比 `BytesToRead` 報告的要多（因為它們介於 `BytesToRead` 呼叫和資料檢索呼叫之間）。但除此之外，還有什麼其他問題呢？

Ben Voigt says:
March 8, 2019 at 9:21 pm
Michael, the first and most severe is that DataReceived fires on a threadpool thread, and ==can be fired again without waiting for the previous event handler to return. So it leads you into a race condition==, where when you go to read the buffer there are **fewer** than `BytesToRead` promised you, because another instance of the event handler read them in the meantime. The application programmer can overcome that by explicitly synchronizing.
Michael，第一個也是最嚴重的問題是，==`DataReceived` 事件在線程池線程上觸發，並且可以在不等待前一個事件處理程序返回的情況下再次觸發。因此，這會導致競爭條件==，當您讀取緩衝區時，實際的 `BytesToRead` 數量會少於承諾的數量，因為與此同時，另一個事件處理程序實例正在讀取它們。應用程式設計師可以透過明確同步來解決這個問題。

But synchronization in the application won’t solve the race condition that exists within the implementation itself. `BytesToRead` calls `ClearCommError` to get the buffer level, and discards everything else. But `ClearCommError` is an atomic exchange on a number of error flags — you get one shot at seeing them then they’re gone. And other code in the framework is looking at those flags to trigger `PinChanged` and `ErrorReceived` events. Because `BytesToRead` ignores them, events get lost. In fact, the MSDN page for the `ErrorReceived` event says that “Because the operating system determines whether to raise this event or not, not all parity errors may be reported.” This is an outright lie — the loss of events happens inside the `getter` functions for `BytesToRead` and `BytesToWrite`.
但是，應用程式中的同步並不能解決實作本身存在的競爭條件。 `BytesToRead` 會呼叫 `ClearCommError` 來取得緩衝區級別，並丟棄其他所有內容。但 `ClearCommError` 是對多個錯誤標誌進行原子交換——您只有一次機會看到它們，然後它們就消失了。框架中的其他程式碼正在查看這些標誌以觸發 `PinChanged` 和 `ErrorReceived` 事件。由於 `BytesToRead` 忽略了它們，事件就會遺失。事實上，`ErrorReceived` 事件的 MSDN 頁面寫道：「由於作業系統決定是否觸發此事件，因此並非所有奇偶校驗錯誤都會被報告。」這完全是謊言——事件的遺失發生在 `BytesToRead` 和 `BytesToWrite` 的 `getter` 函數內部。

Its been a few years and my notes on the bugs are not handy, so there are probably a couple more problems with both `BytesToRead` and `DataReceived` that I am not remembering today.
已經過去好幾年了，我之前關於這些 bug 的記錄不太方便，所以 `BytesToRead` 和 `DataReceived` 可能還存在一些我今天想不起來了的問題。
