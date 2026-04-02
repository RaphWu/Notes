---
aliases:
date:
update:
author:
language:
sourceurl: https://github.com/microsoft/perfview
tags:
---

# PerfView GitHub

PerfView is a free performance-analysis tool that helps isolate CPU and memory-related performance issues. It is a Windows tool, but it also has some support for analyzing data collected on Linux machines. It works for a wide variety of scenarios, but has a number of special features for investigating performance issues in code written for the .NET runtime.
PerfView 是一款免費的效能分析工具，可協助使用者找出與 CPU 和記憶體相關的效能問題。它是一款 Windows 工具，但也支援分析在 Linux 機器上收集的資料。 PerfView 適用於各種應用場景，並針對 .NET 執行時間程式碼的效能問題提供了一些特殊功能。

If you are unfamiliar with PerfView, there are [PerfView video tutorials](http://channel9.msdn.com/Series/PerfView-Tutorial). Also, [Vance Morrison's blog](https://docs.microsoft.com/en-us/archive/blogs/vancem/) gives overview and getting started information.
如果您不熟悉 PerfView，可以觀看 [PerfView 的影片教學](http://channel9.msdn.com/Series/PerfView-Tutorial) 。此外， [Vance Morrison 的部落格](https://docs.microsoft.com/en-us/archive/blogs/vancem/) 也提供了概述和入門資訊。

# PerfView vs. TraceEvent

Not sure which you should use? This document aims to point you in the right direction.
不確定該使用哪一個？本文檔旨在為您指明正確的方向。

## Start With PerfView If Your Goal Is To. 如果您的目標是…，請從 PerfView 開始

- Collect an adhoc Event Tracing for Windows (ETW) trace to analyze program behavior or a performance issue.
  收集 Windows 事件追蹤 (ETW) 臨時追蹤訊息，以分析程式行為或效能問題。
- Collect an adhoc heap snapshot to analyze a managed memory issue such as a managed memory leak.
  收集臨時堆快照，以分析託管記憶體問題，例如託管記憶體洩漏。
- Use the flight recorder mode to capture an ETW trace of hard to reproduce behavior.
  使用飛行記錄儀模式捕捉難以重現行為的 ETW 軌跡。
- Perform adhoc analysis of a previously collected performance trace.
  對先前收集的性能軌跡進行臨時分析。
- Diff two performance traces to or managed memory heap snapshots to root cause a performance issue.
  比較兩個效能追蹤或託管記憶體堆快照，以找出效能問題的根本原因。
- Use a GUI-based performance analysis tool.
  使用基於圖形使用者介面的效能分析工具。

## Start With TraceEvent If Your Goal Is To. 如果您的目標是…，請從 TraceEvent 開始

- Have programmatic access to trace collection and/or trace processing and analysis.
  擁有對痕跡收集和/或痕跡處理和分析的程式化存取權。
- Implement a service that captures or processes traces at scale.
  實現大規模捕獲或處理追蹤資訊的服務。
- Build collection and/or processing capabilities into an existing application.
  在現有應用程式中建構資料採集和/或處理功能。

## PerfView Limitations  PerfView 的局限性

- PerfView is not designed to be used as a capture or processing agent in services. It is designed for use in user-interactive sessions.
  PerfView 並非設計用於服務中的擷取或處理代理程式。它設計用於使用者互動會話。
- PerfView is not supported on operating system SKUs such as nanoserver that do not have GUI libraries installed. See PerfViewCollect if you need to capture adhoc traces on these SKUs.
  PerfView 不支援未安裝 GUI 函式庫的作業系統 SKU，例如 nanoserver。如果您需要在這些 SKU 上捕獲臨時追蹤訊息，請參閱 PerfViewCollect。
