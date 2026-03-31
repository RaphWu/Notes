---
aliases:
date: 2024-04-09
update: 2024-09-29
author: Jason Curl
language: C#
sourceurl: https://github.com/jcurl/RJCP.DLL.SerialPortStream
tags:
  - CSharp
  - SerialPort
---

# Serial Port Stream

3.0.4

SerialPortStream is an independent implementation of `System.IO.Ports.SerialPort` and `SerialStream` for better reliability and maintainability, and now for portability to Mono on Linux systems.
SerialPortStream 是 `System.IO.Ports.SerialPort` 和 `SerialStream` 的獨立實現，以提高可靠性和可維護性，現在為了在 Linux 系統上 Mono 的可移植性而進行改進。

The `SerialPortStream` is a ground up implementation of a `Stream` that buffers data to and from a serial port. It uses low level Win32API (on Windows) for managing events and asynchronous I/O, using a programming model as in the MSDN [PipeServer](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365603.aspx) example.
`SerialPortStream` 是一種從底層重新設計的 `Stream` 實現，用來在串列埠上緩衝資料的傳輸。它使用低階 Win32 API（在 Windows 上）來管理事件和非同步 I/O，其程式設計模型與 MSDN 的 PipeServer 範例相似。

On Linux, it uses a support library to interface with Posix OS calls for an event loop.
在 Linux 上，它使用支援庫與 Posix OS 呼叫事件循環進行互動。

These notes are for version 3.x, which is a refactoring of v2.x for better maintainability. See the end of these notes for differences.
這些說明是為 3.x 版本所準備的，此版本是基於 v2.x 進行重構，以提高可維護性。請參閱這些說明的末尾以了解差異。

# 1. Why another Serial Port implementation 為什麼要再實作一個串列埠庫

Microsoft and Mono already provided a reasonable implementation for accessing the serial port. Today the main goal is to provide a buffered solution that can be used on various operating systems, the the ability to also abstract hardware. Along the way, various issues with the original implementation in .NET Framework are resolved in this library (see the next section).
Microsoft 和 Mono 已經提供了存取序列埠的合理實作。目前的主要目標是提供一個可在各種作業系統上使用的緩衝解決方案，並具備抽象硬體的能力。在此過程中，.NET Framework 中原始實作的各種問題都已在此程式庫中解決（請參閱下一節）。

# 2. Goals 目標

This project tries to achieve the following:
該專案試圖實現以下目標：

- An implementation similar to the MS implementation of `SerialPort`. It's not meant to be 100% compatible, but instead provide similar functionality
  類似微軟對 `SerialPort` 的實作。它並非百分之百相容，而是提供類似的功能。
- Abstract the driver implementation and provide for a more reliable transport, by making writing serial data completely buffered. With the MS implementation, one can write data, but subsequently needs to check if all data is written or not. If it isn't written, then it needs to be retried. The `SerialPortStream` makes this easier.
  抽象驅動程式實作，提供更可靠的傳輸方式，讓寫入序列資料完全緩衝。在微軟實作中，可以寫入資料，但隨後需要檢查所有資料是否都已寫入。若未寫入，則需重試。 SerialPortStream 使這變得更容易。
- Provide for reliable and consistent behaviour. See the next section.
  提供可靠且一致的行為。請參閱下一節。

## 2.1. Issues with MS Serial Port 微軟序列埠的問題

The `SerialPortStream` tries to solve the following issues observed:
`SerialPortStream` 嘗試解決觀察到的以下問題：

- Zach Saw describes issues regarding behaviour of the `fAbortError` flag in the Serial `DCB`. The `SerialPortStream` defines this flag.
  Zach Saw 描述了 `SerialPortStream` `DCB` 中 `fAbortError` 標誌的行為問題。 SerialPortStream 定義了此標誌。
- Closing a serial port, then reopening it generally causes problems. The `SerialPortStream` shouldn't have this issue. Note, there are some cases observed where the Operating System hangs, and this can't be avoided.
  關閉串行端口然後重新打開它通常會導致問題。 `SerialPortStream` 應該不會有這個問題。請注意，在某些情況下，作業系統會掛起，這是無法避免的。
- The `ReadTo()` implementation can subtly change the byte stream buffer, when one switches from characters to bytes. This problem occurs because the MS implementation actually converts the characters back to bytes into its buffer. So if you have UTF8, decoded some invalid characters and then have a timeout, this results in the invalid characters being converted back to bytes, resulting in "lost" data. I take some care when decoding bytes to characters to ensure a seamless and accurate transition between bytes and characters.
  當從字元切換到位元組時， `ReadTo()` 實作可能會巧妙地改變位元組流緩衝區。出現此問題的原因是，微軟的實作實際上會將字元轉換回位元組並存入緩衝區。因此，如果您使用的是 UTF8 編碼，解碼了一些無效字符，然後發生逾時，這會導致無效字元被轉換回字節，從而導致資料「遺失」。我在將位元組解碼為字元時會格外小心，以確保位元組和字元之間的無縫準確轉換。
- `Write()` gives the data to the serial port. If the operation is asynchronous, the call back results in the number of bytes that were actually transferred to the driver. You need to check yourself if this is valid or not. In the synchronous case, the data is simply thrown away. The `SerialPortStream` method simply copies data to a local buffer and uses asynchronous writes in a different thread. It works in the background to send out the data you provided. If the data can't be sent, then you get a `TimeoutException` without any data being buffered at all. So you can implement reliable protocols and your code is simpler.
  `Write()` 將資料寫入串列埠。如果操作是異步的，回呼函數會傳回實際傳輸到驅動程式的位元組數。您需要自行檢查該值是否有效。如果是同步操作，資料將被丟棄 `SerialPortStream` 方法只是將資料複製到本地緩衝區，並使用非同步寫入 不同的線程。它在後台運行，將您 提供。如果資料無法傳送，則會拋出 `TimeoutException` ，且不會緩衝任何資料。因此，您可以實現可靠的協議，並且程式碼更簡潔。
- Disposing or Closing the serial port during a blocking write operation will not abort the write operation. This implementation will abort with an `System.IO.IOException` type.
  在阻塞寫入操作期間處置或關閉序列埠將 不會中止寫入操作。此實作將以 `System.IO.IOException` 類型。

## 2.2. Differences to the MS Serial Port 與 MS 序列埠的差異

The goal is to provide a `Stream`, not an API compatible replacement to the `SerialPort`.
目標是提供一個 `Stream` ，而不是一個 API 相容的替代品 `SerialPort` .

All data is buffered internally in memory, captured using an I/O thread. The extra buffering adds delays by reading the bytes, then performing a context switch for the user code to read the buffer. This can slow down your software.
所有資料都緩存在記憶體中，並透過 I/O 線程捕獲。額外的快取會讀取字節，然後執行上下文切換，讓使用者程式碼讀取緩衝區，從而增加延遲。這會降低軟體的運作速度。

Buffering solves the problem however, that data is read from the serial port in an arbitrary sized memory buffer, and not dependent on the driver, so a likelihood of driver underruns and overruns are reduced. This was an important aspect when writing this library.
然而，緩衝解決了這個問題，即資料從串列埠讀取到任意大小的記憶體緩衝區中，並且不依賴驅動程序，因此降低了驅動程式欠載和過載的可能性。這是編寫此程式庫時的一個重要方面。

# 3. System Requirements 系統要求

## 3.1. Testing** **測試

Software has been tested and developed using:
軟體已使用以下方式進行測試和開發：

- .NET 6.0 and 8.0 on Windows 11 Pro x64, .NET SDK 9.x
  Windows 11 Pro x64 上的 .NET 6.0 和 8.0，.NET SDK 9.x
- .NET 4.8.1 on Windows 11 Pro x64
  Windows 11 Pro x64 上的 .NET 4.8.1
- Mono 6.x from Xamarin on Ubuntu 22.04 (64-bit)
  Ubuntu 22.04（64 位元）上的 Xamarin Mono 6.x

See later in these notes for known issues and changes.
請參閱本說明後面的已知問題和變更。

## 3.2. Compatibility 相容性

### 3.2.1. .NET Frameworks (Windows) .NET 框架（Windows）

The software is written originally for .NET 4.0 and should work on those platforms. It is extended for .NET 4.5 features. A version targets .NET Core 6.0 and 8.0.
該軟體最初是為 .NET 4.0 編寫的，應該可以在這些平台上運行。它已針對 .NET 4.5 功能進行了擴充。此外，還有一個版本是針對 .NET Core 6.0 和 8.0。

Windows XP SP3 and later should work when using .NET 4.0. It's not possible to run the unit tests on Windows XP since the unit tests have migrated to NUnit 4.x, but was working fine prior to that with NUnit 2.x.
使用 .NET 4.0 時，Windows XP SP3 及更高版本應該可以正常運作。由於單元測試已遷移到 NUnit 4.x，因此無法在 Windows XP 上執行單元測試，但在先前的 NUnit 2.x 版本中運作良好。

### 3.2.2. Mono Framework (Linux Only) Mono 框架（僅限 Linux）

The `SerialPortStream` should work on Linux, and it should be possible to import the assembly into your code when running on Linux.
`SerialPortStream` 應該可以在 Linux 上運行，並且在 Linux 上運行時應該可以將組件匯入到您的程式碼中。

When using the Mono Framework, you should reference the .NET 4.0 or .NET 4.6.2 projects.
使用 Mono Framework 時，您應該參考 .NET 4.0 或 .NET 4.6.2 專案。

It has been tested to compile and unit test cases pass with the `dotnet` command on Linux.
它已經過測試，可以在 Linux 上使用 `dotnet` 命令進行編譯並通過單元測試案例。

# 4. Installation 安裝

## 4.1. Windows

On Windows, just reference the assembly in your project installing the NuGet version.
在 Windows 上，只需在安裝 NuGet 版本的專案中引用組件。

## 4.2. Linux

You first need to compile the support library `libnserial.so` for your platform. To do that, you'll need a compiler (e.g. GCC 4.8 or later) and `cmake`. The binaries for Linux are not part of the distribution, as it's operating system specific.
首先，您需要為您的平台編譯支援函式庫 l `libnserial.so` 。為此，您需要一個編譯器（例如 GCC 4.8 或更高版本）和 `cmake` 。 Linux 的二進位檔案不包含在發行版中，因為它特定於作業系統。

After cloning the repository, execute the following:
克隆儲存庫後，執行以下操作：

```text
git clone https://github.com/jcurl/serialportstream.git
cd serialportstream/dll/serialunix
./build.sh
```

Binaries are built and put in the `bin` folder from where you ran the build script. You can add a reference to `LD_LIBRARY_PATH` to the library:
二進位檔案建置完成後，會放入你執行建置腳本的 `bin` 資料夾中。你可以加入 `LD_LIBRARY_PATH` 函式庫的參考：

```text
export LD_LIBRARY_PATH=`pwd`/bin/usr/local/lib:$LD_LIBRARY_PATH
```

and then run your Mono program from there.
然後從那裡運行你的 Mono 程式。

Or you can build and install in your system:
或者您可以在系統中建置並安裝：

```text
cd serialportstream/dll/serialunix
mkdir mybuild
cd mybuild
cmake .. && make
sudo make install
```

# 5. Extra Features 額外功能

The following features are in addition to the `System.IO.Ports.SerialPort` implementation:
除了 `System.IO.Ports.SerialPort` 之外，還提供了以下功能 執行：

- You can obtain the `RingIndicator` pin status.
  您可以取得 `RingIndicator` 引腳狀態。
- The `Read()` and `Write()` buffers are completely independent of the low level Windows driver.
  `Read()` 和 `Write()` 緩衝區完全獨立於低級 Windows 驅動程式。
    - For those concerned, the buffering means that a copy must always be made on every `Read()` and `Write()` method.
      對於那些關心的人來說，緩衝意味著必須始終在每個 `Read()` 和 `Write()` 方法上複製。

# 5.1. Reading and Writing - Buffering 讀取和寫入 - 緩衝

Why is it interesting to perform buffering? A driver might be configured to be 4096 or 8192 bytes (which is quite typical). Testing with older PL2303 chipset, one can't write more than about 12KB with a single write operation. Newer drivers seem to allow much larger buffers.
為什麼執行緩衝操作如此重要？驅動程式可能配置為 4096 或 8192 位元組（這很常見）。使用較舊的 PL2303 晶片組進行測試，單次寫入操作最多只能寫入 12KB 的資料。較新的驅動程式似乎允許更大的緩衝區。

A Write buffer may be 128KB, which one writes to. The thread in the background will write the data and issue as many write calls as is necessary to get the job done. A Read buffer may be 5MB. The background thread will read from the serial port when ever data arrives and buffers into the 5MB.
寫入緩衝區可能為 128KB，用於寫入資料。後台執行緒將寫入數據，並根據需要發出盡可能多的寫入呼叫以完成任務。讀取緩衝區可能為 5MB。每當數據到達時，後台執行緒都會從串列埠讀取數據，並將其緩衝到 5MB 的緩衝區中。

So long as the I/O thread in .NET can execute every 100-200ms, it can continue to read data from the driver. Your own application doesn't need to keep to such difficult time constraints. Such issues typically arise in Automation type environments where a computer has many different peripherals. So long as the process doesn't block, your main application might sleep for 10 seconds and you've still lost no data. The MS implementation wouldn't be so simple, you have to make sure that you perform frequent read operations else the driver itself might overflow (resulting in lost data).
只要 .NET 中的 I/O 執行緒能夠每 100-200 毫秒執行一次，它就可以繼續從驅動程式讀取資料。您自己的應用程式無需遵守如此嚴格的時間限制。此類問題通常出現在自動化環境中，其中電腦具有許多不同的周邊。只要進程不阻塞，您的主應用程式即使休眠 10 秒也不會遺失任何資料。 MS 的實作就沒那麼簡單了，您必須確保頻繁執行讀取操作，否則驅動程式本身可能會溢位（導致資料遺失）。

As the writes are buffered, they tend to return immediately to the application, instead of waiting for the write to complete. Use the new `Flush()` method to ensure that the writes are completed.
由於寫入操作被緩衝，它們往往會立即返回給應用程序，而不是等待寫入完成。使用新的 `Flush()` 方法可確保寫入作業完成。

# 6. Developer Notes 開發者筆記

## 6.1. Logging 日誌記錄

If you come across a problem using this library, you may be asked to provide additional debug logs. This section describes how to obtain those logs for .NET Framework and .NET Core.
如果您在使用此程式庫時遇到問題，可能會要求您提供其他偵錯日誌。本節介紹如何取得 .NET Framework 和 .NET Core 的日誌。

## 6.2. The LogSource abstraction LogSource 抽象

Logging for `SerialPortStream` 3.x uses my `RJCP.Diagnostics.Trace` library that provides an implementation called `LogSource`. This is a wrapper around `TraceSource`, and can provide where necessary a `TraceListener` for .NET Core. For .NET Core, it provides a factory method as a singleton as an alternative to dependency injection. The following sections provide more details.
`SerialPortStream` 3.x 的日誌記錄使用我的 `RJCP.Diagnostics.Trace` 函式庫，該函式庫提供了一個名為 `LogSource` 的實作。這是一個包裝器 `TraceSource` ，並可在必要時為 .NET Core 提供 `TraceListener` 。對於 .NET Core，它提供了一個工廠方法作為單例，作為依賴注入的替代方案。以下部分提供了更多詳細資訊。

## 6.3. .NET Framework (.NET 4.0 to .NET 4.8) .NET Framework（.NET 4.0 至 .NET 4.8）

The library uses the `TraceSource` object, so you can add tracing to your project in the normal way. You should use the switch name `RJCPIO.Ports.SerialPortStream`. An example of an `app.config` file that you can use to enable logging:
該庫使用 `TraceSource` 對象，因此您可以將追蹤新增至您的 以正常方式進行專案。您應該使用開關名稱 `RJCPIO.Ports.SerialPortStream` 。可用於啟用日誌記錄的 `app.config` 檔案範例：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
	<system.diagnostics>
		<sources>
			<source name="RJCP.IO.Ports.SerialPortStream" switchValue="Verbose">
				<listeners>
					<clear/>
					<add name="myListener"/>
				</listeners>
			</source>
		</sources>
		<sharedListeners>
			<add name="myListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="logfile.txt"/>
		</sharedListeners>
	</system.diagnostics>
</configuration>
```

Please note, for SerialPortStream 3.x and later, the name of the trace source has changed to include the full namespace, to be compatible with my other projects.
請注意，對於 SerialPortStream 3.x 及更高版本，追蹤來源的名稱已變更為包含完整的命名空間，以便與我的其他專案相容。

## 6.4. .NET Core .NET 核心

.NET Core has an implementation of `TraceListener` and `TraceSource`, but it doesn't load the `app.config` on start up, nor provide a singleton for applications to use for tracing. There are two ways to enable logging for `SerialPortStream` on .NET Core.
.NET Core 有一個 `TraceListener` 和 `TraceSource` 的實現，但它不會在啟動時載入 `app.config` ，也不會為 用於追蹤的應用程式。有兩種方法可以啟用 .NET Core 上的 `SerialPortStream` 。

### 6.4.1. Dependency Injection 依賴注入

The `SerialPortStream` has a constructor where you can provide your `ILogger` object for tracing.
`SerialPortStream` 有一個建構函數，您可以在其中提供 `ILogger` 追蹤的對象。

Internally, the `ILogger` is wrapped around a minimal `TraceListener` implementation to keep the code common between .NET Framework and .NET Core.
在內部， `ILogger` 被包裹在一個最小的 `TraceListener` 中 實作以保持 .NET Framework 和 .NET Core 之間的程式碼通用。

### 6.4.2. Singleton via LogSource 透過 LogSource 實現單例

When upgrading from .NET Framework to .NET Core, it can be quite difficult to refactor software from using a singleton pattern, to dependency injection, and may require large swaths of code to be refactored.
從 .NET Framework 升級到 .NET Core 時，將軟體從使用單例模式重構為依賴注入可能非常困難，並且可能需要重構大量程式碼。

To avoid this, the `LogSource` classes provide a mechanism to get an `ILogger` using a factory method you provide.
為了避免這種情況， `LogSource` 類別提供了一個獲取 `ILogger` 機制 使用您提供的工廠方法。

You may add in your code the following:
您可以在程式碼中加入以下內容：

```csharp
using Microsoft.Extensions.Logging;
using RJCP.CodeQuality.NUnitExtensions.Trace;
using RJCP.Diagnostics.Trace;

internal static class GlobalLogger {
    static GlobalLogger() {
        ILoggerFactory factory = LoggerFactory.Create(builder => {
            builder
              .AddFilter("Microsoft", LogLevel.Warning)
              .AddFilter("System", LogLevel.Warning)
              .AddFilter("RJCP", LogLevel.Debug)
              .AddNUnitLogger();
        });
        LogSource.SetLoggerFactory(factory);
    }

    public static void Initialize() {
        /* Intentially empty. By calling this method, the static constructor
           will be automatically called */
    }
}
```

The example above is from the unit test cases for `SerialPortStream`, but can be easily adapted for your own projects. The important part is that:
上面的範例來自 `SerialPortStream` 的單元測試案例，但可以輕鬆調整以用於您自己的專案。重要的是：

- In your production code, you assign the static factory with `LogSource.SetLoggerFactory()`, which is not called as part of your unit tests. It will likely be different to the example given above, as you'd want to instead, get the logging configuration (log levels) via a configuration file.
  在您的生產代碼中，您使用 `LogSource.SetLoggerFactory()` ，它不會在單元測試中被呼叫。它可能與上面給出的範例不同，因為您可能希望透過設定檔取得日誌配置（日誌等級）。
- Your test code would set the `LogSource.SetLoggerFactory()` similarly as in the example provided above (the example given above is good for NUnit 3.x with the `.AddNUnitLogger()` provided by my `RJCP.CodeQuality` library).
  您的測試程式碼將設定 `LogSource.SetLoggerFactory()` ，類似於上面提供的範例（上面給出的範例適用於 NUnit 3.x，其中使用我的 `RJCP.CodeQuality` 庫提供的 `.AddNUnitLogger()` ）。
- Setting the factory is only needed for .NET Core. On .NET Framework, the `TraceSource` class provides the singleton functionality and will instantiate for you the correct `TraceSource` from the application configuration file.
  僅在 .NET Core 中才需要設定工廠。在 .NET Framework 中， `TraceSource` 類別提供單例功能，並將從應用程式設定檔中為您實例化正確的 `TraceSource` 。

The code works by requesting to the `ILoggerFactory.CreateLogger` with the name `RJCP.IO.Ports.SerialPortStream`. This is also the motivation why the trace source was changed to include the full namespace. The code above shows that it will create this object and log all debug level events to the NUnit logger.
該程式碼透過向 `ILoggerFactory.CreateLogger` 請求名稱來運作 `RJCP.IO.Ports.SerialPortStream` 。這也是將追蹤來源更改為包含完整命名空間的原因。上面的程式碼顯示它將建立此物件並將所有偵錯等級事件記錄到 NUnit 記錄器中。

# 7. Known Issues 已知問題

## 7.1. General Issues 一般問題

### 7.1.1. ReadTo

The implementation of `ReadTo` and other character based read events with the `SerialPortStream` is slow. It tries to calculate the size (in bytes) of each individual character, in case the user decides to read bytes in-between. The purpose of this was to prevent possible data loss in when a read of bytes is mixed with a read of characters. It's recommended not to use the character based APIs.
`ReadTo` 和其他基於字元的讀取事件的實現 `SerialPortStream` 速度較慢。它會嘗試計算每個字元的大小（以位元組為單位），以防使用者決定在中間讀取位元組。這樣做的目的是防止在讀取位元組和字元時混合讀取時可能造成的資料遺失。建議不要使用基於字元的 API。

Some test cases are failing and needs to be investigated (although it's not clear if this is a bug in the library or in the test case).
一些測試案例失敗，需要進行調查（儘管不清楚這是庫中的錯誤還是測試案例中的錯誤）。

## 7.2. Windows

The following issues are known:
已知問題如下：

- This is not an issue in the library, but when using the `Com0Com` for running unit tests, some specific test cases for Parity will fail. That is because Com0Com doesn't emulate data at a bit level. Using real serial hardware with a NULL modem adapter works as expected.
    這在庫中不是問題，但當使用 `Com0Com` 執行單元測試時，某些特定的 Parity 測試案例會失敗。這是因為 Com0Com 無法以位元等級模擬資料。使用具有 NULL 數據機適配器的真實串行硬體可以正常工作。
- .NET 4.0 to .NET 4.8.1 and .NET Core has a minor issue in `System.Text.Decoder` that in a special circumstance it will consume too many bytes. The `PeekChar()` method is therefore a slower implementation than what it could be. Please refer to the Xamarin bug [40002](https://bugzilla.xamarin.com/show_bug.cgi?id=40002). Found against Mono 4.2.3.4 and later tested to be present since .NET 4.0 on Windows XP also.
    .NET 4.0 到 .NET 4.8.1 和 .NET Core 存在一個小問題 `System.Text.Decoder` 在特殊情況下會消耗過多的位元組。因此， `PeekChar()` 方法的實現速度比 有可能。請參閱 Xamarin 錯誤 [40002.](https://bugzilla.xamarin.com/show_bug.cgi?id=40002) 在 Mono 4.2.3.4 中發現，後來測試發現自 Windows XP 上的 .NET 4.0 起也存在此問題。

### 7.2.1. Driver Specific Issues on Windows Windows 上的驅動程式特定問題

#### 7.2.1.1. Flow Control 流量控制

Using the FTDI chipset on Windows 10 x64 (FTDI 2.12.16.0 dated 09/Mar/2016) flow control (RTS/CTS) doesn't work as expected. For writing small amounts of data (1024 bytes) with CTS off, the FTDI driver will still send data. See the test case `ClosedWhenFlushBlocked`, change the buffer from 8192 bytes to 1024 and the test case now fails. This problem is not observable with com0com 3.0. You can see the effect in logs, there is a TX-EMPTY event that occurs, which should never be there if no data is ever sent.
在 Windows 10 x64 上使用 FTDI 晶片組（FTDI 2.12.16.0，發布日期：2016 年 3 月 9 日）時，流量控制 (RTS/CTS) 無法正常運作。在關閉 CTS 的情況下寫入少量資料（1024 位元組）時，FTDI 驅動程式仍會傳送資料。請參閱測試案例 `ClosedWhenFlushBlocked` ，將緩衝區大小從 8192 位元組變更為 1024 位元組後，該測試案例會失敗。此問題在 com0com 3.0 中未觀察到。您可以在日誌中看到影響，其中發生了 TX-EMPTY 事件，如果沒有發送任何數據，該事件不應該出現。

#### 7.2.1.2. BytesToWrite 待寫入位元組數

On Windows, the `SerialPortStream` returns the bigger of either the internal write buffer, or the amount of data in the output queue of the driver. Drivers don't report the number of bytes that are in the output queue before the next write begins, and may return sooner. This leads to the effects:
在 Windows 上， `SerialPortStream` 會傳回內部寫入緩衝區或驅動程式輸出佇列中較大的資料量。驅動程式不會報告下一次寫入開始前輸出佇列中的位元組數，因此可能會提前傳回。這會導致以下結果：

##### 7.2.1.2.1. CP2101 Driver CP2101 驅動程式

This driver indicates more bytes are in the output queue than what it will return from the current ongoing write operation. This can cause some jumps in the returned value.
此驅動程式指示輸出佇列中的位元組數多於目前正在進行的寫入操作將傳回的位元組數。這可能會導致返回值出現一些跳躍。

[CP210x Universal Windows Driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) v10.1.10 1/13/2021.
[CP210x 通用 Windows 驅動程式](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) v10.1.10 2021 年 1 月 13 日。

For example:  例如：

```text
BytesToWrite = 40960 (driver 12288)
RJCP.IO.Ports.SerialPortStream Verbose: 0 : COM5: SerialThread: ProcessWriteEvent: 1024 bytes
BytesToWrite = 40412 (driver 40412)
RJCP.IO.Ports.SerialPortStream Verbose: 0 : COM5: SerialThread: DoWriteEvent: WriteFile(736, 312385272, 39936, ...) == False
BytesToWrite = 40387 (driver 40387)
BytesToWrite = 40386 (driver 40386)
```

The internal buffer is 40kB, the driver returned it wrote 1024 bytes, but the queue still has 40412 bytes (which is more than the 39936 bytes it should be).
內部緩衝區為 40kB，驅動程式返回它寫入了 1024 個位元組，但佇列仍然有 40412 個位元組（超過了應有的 39936 個位元組）。

It can also fluctuate without writes without calls to the OS in between.
它也可以在沒有寫入或呼叫作業系統的情況下波動。

```text
BytesToWrite = 40418 (driver 40418)
BytesToWrite = 40393 (driver 40393)
BytesToWrite = 40392 (driver 40392)
BytesToWrite = 40391 (driver 40391)
BytesToWrite = 40390 (driver 40390)
BytesToWrite = 40389 (driver 40389)
BytesToWrite = 40428 (driver 40428)
BytesToWrite = 40427 (driver 40427)
BytesToWrite = 40426 (driver 40426)
```

##### 7.2.1.2.2. PL2303 RA

Generally this driver reports that it has zero bytes in the output queue, but may sometimes report the number of bytes in the last `WriteFile()` call. This is not a problem, but the number of bytes in the output queue is less than what is still to be written, so a user may think it is complete, when it is not.
通常情況下，此驅動程式會報告輸出佇列中位元組數為零，但有時也會報告上次 `WriteFile()` 呼叫中的位元組數。這本身並沒有問題，但輸出佇列中的位元組數小於待寫入的位元組數，因此使用者可能會認為寫入已完成，但實際上並非如此。

The drivers only work when flushing if the `WriteTotalTimeoutConstant` is zero. That means doing something like:
只有當 `WriteTotalTimeoutConstant` 為零時，驅動程式才會在刷新時工作。這意味著執行以下操作：

```csharp
var serialPort = new RJCP.IO.Ports.WinSerialPortStream(com, 115200, 8, RJCP.IO.Ports.Parity.None, IO.Ports.StopBits.One);
serialPort.Settings.WriteTotalTimeoutMultiplier = 0;
serialPort.Settings.WriteTotalTimeoutConstant = 500;
serialPort.Open();
```

will cause flush to come back too soon. In version 3.0.0 - 3.0.2, the default was 500 and was changed to zero as part of [Issue #154](https://github.com/jcurl/RJCP.DLL.SerialPortStream/issues/154).
會導致 flush 回傳過早。在 3.0.0 - 3.0.2 版本中，預設值為 500，並已在 [問題 #154](https://github.com/jcurl/RJCP.DLL.SerialPortStream/issues/154) 中變更為零。

## 7.3. Linux

The `SerialPortStream` was tested on Ubuntu 14.04 to 22.04. Feedback welcome for other distributions!
`SerialPortStream` 已在 Ubuntu 14.04 至 22.04 上測試。歡迎針對其他發行版提供回饋！

The main functionality on Linux is provided by a support C library that abstracts the Posix system call `select()`.
Linux 上的主要功能由抽象 Posix 系統呼叫 `select()` 支援 C 函式庫提供。

Issues in the current implementation are:
目前實施中存在的問題有：

- Custom baud rates are not supported. To know what baud rates are supported on your system, look at the file `config.h` after building.
  不支援自訂波特率。若要了解您的系統支援的波特率，請在建置後查看 `config.h` 檔案。
- DSR and DTR handshaking is not supported. You can still set and clear the pins though.
  不支援 DSR 和 DTR 握手。不過您仍然可以設定和清除引腳。

Patches are welcome to implement these features!
歡迎使用補丁來實現這些功能！

### 7.3.1. Mono on non-Windows Platforms 非 Windows 平台上的 Mono

Use the currently supported versions of Mono provided by the Mono project for your Linux distribution. For example, Ubuntu 14.04 ships with Mono 3.2.8 which is known to not work.
請使用 Mono 專案為您的 Linux 發行版提供的目前支援的 Mono 版本。例如，Ubuntu 14.04 附帶的 Mono 3.2.8 已知無法正常運作。

- [[Mono-Dev] Mono 3.2.8 incompatibility with .NET 4.0 on Windows 7-10](http://lists.ximian.com/pipermail/mono-devel-list/2015-December/043423.html). The System.Text implementation for converting bytes to UTF8 don't work. If you don't use the character based methods, it may work. But the software has not been tested against this framework.
  [[Mono-Dev] Mono 3.2.8 與 Windows 7-10 上的 .NET 4.0 不相容](http://lists.ximian.com/pipermail/mono-devel-list/2015-December/043423.html) 。 System.Text 將位元組轉換為 UTF8 的實作不起作用。如果您不使用基於字元的方法，它可能有效。但該軟體尚未針對此框架進行測試。
- The `DataReceived` event doesn't fire for the EOF character (0x1A). On Windows it does, as this is managed by the driver itself and not emulated by the C-Library.
  當遇到 EOF 字元 (0x1A) 時， `DataReceived` 事件不會觸發。但在 Windows 上會觸發，因為 EOF 字元由驅動程式本身管理，而不是由 C 語言庫模擬。
- Linux doesn't implement DSR.
  Linux 沒有實作 DSR。

### 7.3.2. Driver Specific Issues on Linux Linux 上的驅動程式特定問題

Tests have been done using FTDI, PL2303H, PL2303RA and 16550A (some still do exist!). The following has been observed:
已使用 FTDI、PL2303H、PL2303RA 和 16550A（其中一些仍然存在！）進行了測試。觀察到以下情況：

#### 7.3.2.1. Parity Errors 奇偶校驗錯誤

Some chipsets do not report properly parity errors. The 16550A chipset works as expected. Issues observed with FTDI, PL2303H, PL2303RA. In particular, on a parity error, more bytes are reported as having parity errors than there are in the stream. Tested using loopback devices with `comptest`.
某些晶片組無法正確報告奇偶校驗錯誤。 16550A 晶片組工作正常。 FTDI、PL2303H 和 PL2303RA 也存在此問題。尤其是在發生奇偶校驗錯誤時，報告的奇偶校驗錯誤位元組數比實際資料流中的位元組數要多。使用環回設備進行測試，並使用 `comptest` 進行測試。

```text
$ ./nserialcomptest /dev/ttyUSB0 /dev/ttyUSB1`
  [ RUN      ] SerialParityTest.Parity7E1ReceiveError
/home/jcurl/Programming/serialportstream/dll/serialunix/libnserial/comptest/SerialParityTest.cpp:221: Failure
Value of: comparison
  Actual: false
Expected: true
Unexpected byte received with Even Parity
[  FAILED  ] SerialParityTest.Parity7E1ReceiveError (585 ms)
[ RUN      ] SerialParityTest.Parity7O1ReceiveError
/home/jcurl/Programming/serialportstream/dll/serialunix/libnserial/comptest/SerialParityTest.cpp:373: Failure
Value of: comparison
  Actual: false
Expected: true
Unexpected byte received with Even Parity
[  FAILED  ] SerialParityTest.Parity7O1ReceiveError (584 ms)
[ RUN      ] SerialParityTest.Parity7O1ReceiveErrorWithReplace
/home/jcurl/Programming/serialportstream/dll/serialunix/libnserial/comptest/SerialParityTest.cpp:427: Failure
Value of: comparison
  Actual: false
Expected: true
Unexpected byte received with Even Parity
[  FAILED  ] SerialParityTest.Parity7O1ReceiveErrorWithReplace (572 ms)
```

#### 7.3.2.2. Garbage Data on Open 開啟時的垃圾數據

On Linux Kernel with Ubuntu 14.04 and Ubuntu 16.04, we observe that some USB-SER drivers provide extra data depending on what a previous process was doing. It shows itself as garbage zero's appearing at the beginning of a stream when reading, and may be visible in your application also. There's a test case `comptest/kernelbug` that shows this behaviour on a Lenovo T61p. Affected is PL2303H and FTDI chipsets. Chipsets that don't show this behaviour are 16550A and PL2303RA chipsets. Invocate the test program twice and you'll see the error. This is reported to [Ubuntu](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1542862)
在 Ubuntu 14.04 和 Ubuntu 16.04 的 Linux 核心上，我們觀察到一些 USB-SER 驅動程式會根據前一個進程正在執行的操作提供額外的資料。它 當出現以下情況時，它表現為出現在流開頭的垃圾零 閱讀，也可能在你的應用程式中可見。有一個測試用例 `comptest/kernelbug` 在聯想 T61p 上發現了這個行為。受影響的是 PL2303H 和 FTDI 晶片組。沒有出現此現象的晶片組是 16550A 和 PL2303RA 晶片組。呼叫測試程式兩次，您就會看到錯誤。 據報道 [Ubuntu](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1542862)

```text
$ kernelbug /dev/ttyUSB0 /dev/ttyUSB1
Offset: 4
Flushing...
Writing Complete...
Reading complete...
Comparison MATCH                    <---- PASS
Flushing...
Reading complete...
Complete...

$ kernelbug /dev/ttyUSB0 /dev/ttyUSB1
Offset: 108
Flushing...
Flush 2 bytes
Writing Complete...
Reading complete...
ERROR: Comparison mismatch           <---- ERROR
Flushing...
Flush 510 bytes
Reading complete...
Complete...
```

#### 7.3.2.3. Monitoring Pins and Timing Resolution 監控引腳和時序分辨率

Monitoring of pins CTS, DSR, RI and DCD is not 100% reliable for some chipsets and workarounds are in place. In particular, the chips PL2303H, PL2303RA do not support the `ioctl(TIOCGICOUNT)`, so on a pin toggle, we cannot reliably detect if they have changed if the pulse is too short. For 16550A and FTDI chips, this `ioctl()` does work and so we can always detect a change. To check if your driver supports the `ioctl(TIOCGICOUNT)` call, run the small test program `comptest/icount`.
對於某些晶片組，對 CTS、DSR、RI 和 DCD 引腳的監控並非 100% 可靠，因此需要採取一些變通措施。特別是 PL2303H 和 PL2303RA 晶片不支援 `ioctl(TIOCGICOUNT)` ，因此在引腳切換時，我們無法可靠地檢測到 如果脈衝太短，它們是否發生了變化。對於 16550A 和 FTDI 晶片，這 `ioctl()` 確實有效，因此我們總是可以偵測到變化。若要檢查您的驅動程式是否支援 `ioctl(TIOCGICOUNT)` 調用，請執行小型測試程式 `comptest/icount` 。

```text
$ ./icount /dev/ttyS0
Your driver supports TIOCGICOUNT
ocounter.cts=0
ocounter.dsr=0
ocounter.rng=3
ocounter.dcd=0
```

or in the case it's not supported:
或在不支持的情況下：

```text
$ ./icount /dev/ttyUSB0
Your driver doesn't support TIOCGICOUNT
Error: 25 (Inappropriate ioctl for device)
```

#### 7.3.2.4. Close Times with Flow Control 有流量控制的關閉時間

Some times closing the serial port may take a long time (observed from 5s to 21s) if it is write blocked due to hardware flow control. In particular, the C-library function `serial_close()` appears to take an excessive time when calling `close(handle->fd)` on Ubuntu 16.04. This issue appears related to the Linux driver and not the MONO framework.
如果由於硬體流控制導致序列埠寫入阻塞，關閉序列埠有時可能需要很長時間（觀察結果顯示為 5 秒到 21 秒）。具體來說，在 Ubuntu 16.04 上，C 函式庫函數 `serial_close()` 在呼叫 `close(handle->fd)` 時似乎會花費過多的時間。此問題似乎與 Linux 驅動程式有關，而非 MONO 框架。

The .NET Test Cases that show this behaviour are (blocking on write):
顯示此行為的 .NET 測試案例是（寫入時阻塞）：

- `ClosedWhenBlocked`
- `CloseWhenFlushBlocked`
- `DisposeWhenBlocked`
- `DisposeWhenFlushBlocked`

This issue is not reproducible with the 16550A UART when it is write blocked. In this case, the times for closing are usually not more than 20ms.
當 16550A UART 被設定為寫入阻止模式時，此問題無法重現。在這種情況下，關閉時間通常不超過 20ms。

#### 7.3.2.5. Opening Ports (and some unit test case failures) 開啟連接埠（以及一些單元測試案例失敗）

When testing continuously to open a port, send data, and then receive on another port using a NULL-modem cable, minor issues can occur that result in test case failures. These issues would also be visible in real-world programs and are driver dependent.
使用零調製解調器線連續測試開啟一個連接埠、發送數據，然後在另一個連接埠接收資料時，可能會出現一些小問題，導致測試案例失敗。這些問題在實際程式中也會出現，並且與驅動程式相關。

The test scenario is on Linux (Ubuntu 20.04) with various USB serial port devices. The test case `ReadToWithMbcs` from SerialPortStreamNativeTest is modified to run 2000 times with `[Repeat(2000)]`. The command to run the test after building is then:
測試場景在 Linux（Ubuntu 20.04）上，包含各種 USB 串列裝置。 SerialPortStreamNativeTest 中的測試案例 `ReadToWithMbcs` 已修改為執行 2000 次，並使用 `[Repeat(2000)]` 進行測試。建置後執行測試的命令如下：

```text
dotnet test RJCP.SerialPortStreamNativeTest.dll --filter Name=ReadToWithMbcs
```

(please note, not only this test case is affected, but it is easy to reproduce.)
（請注意，不僅這個測試案例受到影響，而且很容易重現。）

- _FTDI_: After opening the serial port, in about 1% of the cases, data is not sent (observed using .NET Mono 4.0 and 4.5). It is confirmed that the system call `write()` was called, and all data was given to the kernel via the library `libnserial`. However, on the other serial port, data is never received. Waiting 15ms after opening would resolve the problem - this workaround will not be part of the `SerialPortStream` as it appears to be very specific to this driver and similar behaviour is not observed on other drivers.
  _FTDI_ ：開啟串列埠後，大約有 1% 的情況資料未傳送（使用 .NET Mono 4.0 和 4.5 觀察）。已確認系統調用 `write()` 已被調用，並且所有資料均透過函式庫 `libnserial` 傳遞給核心。然而，在另一個串列埠上，數據始終未收到。打開後等待 15 毫秒即可解決問題——此解決方法不屬於 `SerialPortStream` 的一部分，因為它似乎是此驅動程式特有的，並且在其他驅動程式上未觀察到類似的行為。
- _PL2303H_: Sometimes on connecting the serial port, a spurios 0xFF is sent. This causes the test case to fail, as data that was not sent is received and affects the output.
  _PL2303H_ ：連接串列埠時，有時會傳送 0xFF 的雜散資料。這會導致測試案例失敗，因為接收了未發送的資料並影響輸出。
- _PL2303RA_: The test cases appear to run about 10x slower than any other driver, but no errors were observed.
  _PL2303RA_ ：測試案例的運行速度似乎比其他驅動程式慢 10 倍，但沒有觀察到錯誤。
- _CP2101_: Seems to work flawlessly.
  _CP2101_ ：似乎運行完美。

## 7.4. Guidelines on Serial Protocols 串行協定指南

Given the issues listed in this section, one can come up with the following recommendations for protocol design over the serial port:
鑑於本節列出的問題，可以針對串行埠協定設計提出以下建議：

- Assume that at Layer 2 (the serial port bus), data can be inserted, modified, or deleted.
  假設在第 2 層（序列埠匯流排），可以插入、修改或刪除資料。
- Define data as frames. There should be a marker byte indicating the start of a frame, a length to know how much data should be received, and a checksum (at least a CRC16) that can be used to check the integrity of the frame.
  將資料定義為幀。應該有一個標記位元組指示訊框的開始，一個長度用於確定應接收的資料量，以及一個校驗和（至少是 CRC16），用於檢查訊框的完整性。
- Define the protocol to be able to resend data in case of lost data if needed, or can continue if data is lost.
  定義協定以便在資料遺失時能夠重新發送資料（如果需要），或在資料遺失時可以繼續。

I recommend to not use hardware or software flow control, but define in the serial protocol frames, like a link control protocol (LCP) that can manage this. Do not use parity, and instead opt to use checksum bytes within a frame.
我建議不要使用硬體或軟體流控制，而是在串行協定幀中定義，例如可以管理流控制的鏈路控制協定 (LCP)。不要使用奇偶校驗，而是選擇在訊框中使用校驗和位元組。

- Hardware flow control can lead to deadlocks in software. No flow control just means data can be lost, and can be replaced using a LCP. Software flow control can also cause complications in the protocol, which can be more generically handled using a LCP.
  硬體流控制可能導致軟體死鎖。沒有流控制意味著資料可能遺失，可以使用 LCP 進行替換。軟體流控制也可能導致協定複雜化，使用 LCP 可以更通用地處理這種情況。
- Parity can insert arbitrary bytes and corrupted data, especially with USB serial devices. Use frame checksums (FCS) instead.
  奇偶校驗可能會插入任意位元組和損壞的數據，尤其是在 USB 序列裝置上。請使用訊框校驗和 (FCS)。

Allow bundling of frames one after the other, and decode separately. Lots of small frames that need to be acknowledged can lead to delays between frames, and longer transmission times for an already "slow" bus speed. The `SerialPortStream` is buffered, so performance is impacted by lots of context switches between sending data, and waiting for a response, as there is a buffer thread used in-between.
允許將幀逐個捆綁在一起，然後分別解碼。許多 需要確認的小幀可能會導致幀之間的延遲，並且 對於已經很慢的總線速度來說，傳輸時間更長。 `SerialPortStream` 是緩衝的，因此效能會受到發送資料和等待回應之間的大量上下文切換的影響，因為其間使用了緩衝執行緒。
