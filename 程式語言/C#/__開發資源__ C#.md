---
aliases:
created: 
update:
author: Raphael Wu
language:
sourceurl:
tags:
date:
---

# 名詞

| 英文             | 中文  | 其他常見翻譯 |
| -------------- | --- | ------ |
| Abstraction    | 抽象  |        |
| Encapsulation  | 封裝  |        |
| Inheritance    | 繼承  |        |
| Polymorphism   | 多型  | 多態性    |
| Implementation | 實現  |        |
| Association    | 關聯  |        |
| Dependency     | 依賴  |        |
| Aggregation    | 聚合  |        |
| Composition    | 組合  |        |
| Base           | 基底  |        |
| Derived        | 衍生  | 派生     |
| override       | 覆寫  | 覆蓋     |
| virtual        | 虛擬  |        |

---

# .NET / C#

- [C# 的歷史](https://learn.microsoft.com/zh-tw/dotnet/csharp/whats-new/csharp-version-history)
- [.NET API 瀏覽器](https://learn.microsoft.com/zh-tw/dotnet/api/)
- [Language Feature Status (此連結網址中有空白字元，Markdown 好像不支援，須自行複製網址)](https://github.com/dotnet/roslyn/blob/main/docs/Language%20Feature%20Status.md)
- [C#版本与.NET 版本对应关系以及各版本的特性](https://www.cnblogs.com/MingsonZheng/p/11273700.html)
- [線上編輯工具 .NET Fiddle](https://dotnetfiddle.net/)
- [C# at Google Style Guide](https://google.github.io/styleguide/csharp-style.html)

## Source Code

- [.NET Runtime Libraries (GitHub)](https://github.com/dotnet/runtime/tree/main/src/libraries/System.Private.CoreLib/src/System)
- [.NET Source Browser](https://source.dot.net/)
- [.NET Framework source code online (mscorlib)](https://referencesource.microsoft.com/)

## 規範

- [C# at Google Style Guide](https://google.github.io/styleguide/csharp-style.html)
- [C# Coding Style](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md)

## XML Documentation

- [C# 檔批註的建議 XML 標籤](https://learn.microsoft.com/zh-tw/dotnet/csharp/language-reference/xmldoc/recommended-tags)
- [Sandcastle XML Comments Guide](http://ewsoftware.github.io/XMLCommentsGuide/html/4268757F-CE8D-4E6D-8502-4F7F2E22DDA3.htm)
- [C#使用 XML 註解](http://johnniebooks.blogspot.com/2009/07/cxml.html)
- [XML Documentation 常見陷阱一則](https://blog.darkthread.net/blog/xml-doc-char-pitfall/)

### Sandcastle Help File Builder

- [GitHub](https://github.com/EWSoftware/SHFB)
- [Sandcastle Help File Builder Documentation](http://ewsoftware.github.io/SHFB/html/bd1ddb51-1c4f-434f-bb1a-ce2135d3a909.htm)
- [C#类库生成 API 文档！](https://www.dongchuanmin.com/csharp/2565.html)
- [使用 Sandcastle Help File Builder 產生 Library 文件](https://blog.givemin5.com/shi-yong-sandcastle-help-file-builder-chan-sheng-library-wen-jian/)
- [使用 Sandcastle Help File Builder 產生 .NET Library 說明文件](https://dotblogs.com.tw/wuanunet/2015/12/17/get_csharp_documentation_by_sandcastle_generator)

## Collections

- [集合和資料結構](https://learn.microsoft.com/zh-tw/dotnet/standard/collections/)

## 多執行緒

- [C# 學習筆記：多執行緒](https://www.huanlintalk.com/2013/04/csharp-notes-multithreading-1.html)

## 非同步程式設計

- [Async/Await 非同步程式設計中的最佳做法](https://learn.microsoft.com/zh-tw/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)

## 設計模式

- [一文讀懂七大設計原則及 GoF 23 種設計模式](https://codingnote.cc/zh-tw/p/83149/)
- [C#设计模式之一单例模式（Singleton Pattern）【创建型】](https://www.cnblogs.com/PatrickLiu/p/8250985.html)
- [淺談 C#單例模式的實現和效能對比](https://www.php.cn/csharp-article-379465.html)
- [Implementing the Singleton Pattern in C#](https://csharpindepth.com/Articles/Singleton)

## DEBUG 相關

- [C# 前置處理器指示詞](https://learn.microsoft.com/zh-tw/dotnet/csharp/language-reference/preprocessor-directives)
- [Debug 和 Release 的不同](http://johnniebooks.blogspot.com/2009/05/debug-release.html)
- [在 Debug 或 Release 時執行特定程式碼](https://dotblogs.com.tw/joysdw12/2014/03/14/asp-net-debug-release-if-else-conditional)

## 綜合

- [huanlin/csharp-notes: 學習筆記，主要是 C# 與 .NET 技術](https://github.com/huanlin/csharp-notes)

---

# 專題

- [[C#] 自訂例外處理 (Exception)](https://dotblogs.com.tw/atowngit/2009/12/06/12298)
- [[C#.NET] 使用自訂例外 Exception](https://dotblogs.com.tw/yc421206/2013/05/13/103536)
- [C# 技巧：用列舉及 nameof 取代字串常數提高可維護性](https://blog.darkthread.net/blog/enum-nameof-instead-of-string-constant/)
- [Difference between SortedList and SortedDictionary in C#](https://www.geeksforgeeks.org/difference-between-sortedlist-and-sorteddictionary-in-c-sharp/)

## 序列化 Serializer

- [.NET 性能优化-是时候换个序列化协议了](https://blog.csdn.net/mzl87/article/details/127751662)
- [MemoryPack](https://github.com/Cysharp/MemoryPack)
- [MessagePack for C# (.NET, .NET Core, Unity, Xamarin)](https://github.com/MessagePack-CSharp/MessagePack-CSharp)
- [使用.NET7 和 C#11 打造最快的序列化程序-以 MemoryPack 为例](https://www.cnblogs.com/InCerry/p/how-to-make-the-fastest-net-serializer-with-net-7-c-11-case-of-memorypack.html)

## 數值運算

### Math.NET Numerics

Math.NET Numerics 是 Math.NET 計畫的數值基礎，旨在為科學、工程和日常應用中的數值計算提供方法和演算法。涵蓋的主題包括特殊函數、線性代數、機率模型、隨機數、統計、內插、積分、迴歸、曲線擬合、積分變換（FFT）等等。
除了完全以 C# 編寫的核心 .NET 套件之外，Numerics 還專門支援 F#，並提供慣用的擴充模組，同時保留了源自 F# PowerPack 的 BigRational 等數學資料結構。如果需要提升效能，可以將支援其線性代數例程和分解的託管程式碼提供者替換為針對最佳化原生實作（例如 Intel MKL）的包裝器。
Math.NET Numerics 採用 [MIT 授權](https://github.com/mathnet/mathnet-numerics/blob/master/LICENSE.md) 。因此，您可以連結到它，並在開源和專有軟體專案中使用它。我們歡迎您的貢獻！

- [官方網站](https://numerics.mathdotnet.com/)
- [Github](https://github.com/mathnet/mathnet-numerics)
- [Documentation](https://numerics.mathdotnet.com/)
- [API Reference](https://numerics.mathdotnet.com/api/)
- [math.net](https://www.math.net/) ( 數學知識 )
- 網路文章
- [C#使用 Math.Net 库进行矩阵运算](https://blog.csdn.net/kenjianqi1647/article/details/88869077)
- [Geolocation](https://github.com/scottschluer/geolocation)~

一個 C# 類別庫，它將計算兩組座標之間的距離和基本方向，並提供原點座標周圍的緯度/經度邊界，允許在給定半徑內對位置進行簡單的 SQL 或 LINQ 選擇。

## Obfuscar

- [GitHub](https://github.com/obfuscar/obfuscar)
- [介紹好用工具：使用 Obfuscar 混淆你的 .NET 組件](https://blog.miniasp.com/post/2023/08/10/Useful-tool-Obfuscar-Open-source-obfuscation-tool-for-NET-assemblies)

## QR Code

- [QRCoder](https://github.com/codebude/QRCoder/)
- [如何使用 .NET 的 QRCoder 套件產生 QRCode 圖片](https://blog.miniasp.com/post/2023/08/30/How-to-use-QRCoder-generates-QR-Code-using-dotNet)

---

# 控制反轉 (IoC)、相依性注入 (DI)

- [Microsoft.Extensions.DependencyInjection](https://learn.microsoft.com/zh-tw/dotnet/core/extensions/dependency-injection)
- [控制反轉 (IoC) 與 依賴注入 (DI)](https://notfalse.net/3/ioc-di)
- [小菜学习设计模式（五）—控制反转（Ioc）](https://www.cnblogs.com/xishuai/p/3666276.html)
- [IoC 依赖注入容器 Unity](https://www.cnblogs.com/gaochundong/archive/2013/04/10/ioc_container_unity.html)
- [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)

## Autofac

- [官方網站](https://autofac.org/)
- [Github](https://github.com/autofac/Autofac)
- [Documentation](https://autofac.readthedocs.io/en/latest/)

### 網路文章

- [黑暗執行緒 分類檢視：autofac](https://blog.darkthread.net/blog/category/autofac)
- [Dependency Injection with Autofac](https://www.codeproject.com/articles/Dependency-Injection-with-Autofac)

## Unity

- [官方網站](http://unitycontainer.org/)
- [Github](https://github.com/unitycontainer)

### 網路文章

- [C# Unity 依赖注入](https://www.cnblogs.com/tuyile006/p/6929796.html)
- [Unity 依赖注入使用详解](https://www.cnblogs.com/xishuai/p/3670292.html)

---

# Prism

- [GitHub (含主原始碼、範例、Templates)](https://github.com/PrismLibrary/)
- [官方文件](https://prismlibrary.com/docs/)
- [官方文件簡體翻譯](https://csharpshare.com/articles/framework/prism-doc/index.html)
- [微軟官方文件 (舊的 Prism 5.0)](<https://learn.microsoft.com/en-us/previous-versions/msp-n-p/gg406140(v=pandp.10)>)
- [Brian Lagunas's Blog](https://brianlagunas.com/)
- [DryIoc](https://prismplugins.com/containers/dryioc/)

## 官方沒在說明檔內的更新資料

- [A New IDialogService for WPF](https://github.com/PrismLibrary/Prism/releases/tag/v7.2.0.1367)

# 網路文章

- [Prism 8.0 入门（上）：Prism.Core](https://www.cnblogs.com/dino623/p/using_prism_core.html)
- [Prism 8.0 入门（下）：Prism.Wpf 和 Prism.Unity](https://www.cnblogs.com/dino623/p/using_prism_wpf_and_prism_unity.html)
- [WPF NET5 Prism8.0 的升级指南](https://cloud.tencent.com/developer/article/1776684)
- [When should I use an event handler over an event aggregator?](https://stackoverflow.com/questions/13565774/when-should-i-use-an-event-handler-over-an-event-aggregator)
- [Prism 之 InvokeCommandAction 的 TriggerParameterPath 和 CommandParameter 的用法](https://blog.csdn.net/jiuzaizuotian2014/article/details/104845775)
- [Prism 合集](https://www.cnblogs.com/zh7791/category/1893907.html)
- [Using IActiveAware and INavigationAware](https://not-now-nigel.blogspot.com/2011/09/using-iactiveaware-and-inavigationaware.html)
- [Blog - 用心爱你的邻居](https://www.cnblogs.com/hicolin/tag/Prism/)
- [Blog - 包建强的无线技术空间](https://www.cnblogs.com/Jax/category/213017.html)
- [Stack Overflow 問答 - View 及 ViewModel 註冊](https://stackoverflow.com/questions/8617277/prism-connecting-views-and-viewmodels-with-unity-trying-to-understand-it)
- [Prism の Prism Full App (.NET Core) テンプレートを見てみよう](https://qiita.com/okazuki/items/cfbf5c9eaea6c5aed4e1)

## 舊版參考資料

- [Developer's Guide to Microsoft Prism Library 5.0 for WPF](<https://learn.microsoft.com/en-us/previous-versions/msp-n-p/gg406140(v=pandp.10)>)
- [Blog - My musings on .Net Technology](https://rohiton.wordpress.com/category/prism/)
- [PrismNew](https://prismnew.readthedocs.io/en/latest/)
- [WPF Step By Step 系列-Prism 框架在项目中使用](https://www.cnblogs.com/hegezhou_hot/archive/2012/12/21/2828162.html)

---

# FP 函數式程式設計

- [Functional Programming 簡介](https://old-oomusou.goodjack.tw/fp/intro/)

---

# Logger (Serilog)

- [Serilog 官方網站](https://serilog.net/)
- [Github](https://github.com/serilog/serilog)
- [官方文件](https://github.com/serilog/serilog/wiki)
- [Serilog 2.10 中文文档](https://blog.csdn.net/catshitone/article/details/121565967)
- [Message Templates](https://messagetemplates.org/)
- [Serilog.Sinks.RichTextBox.Wpf](https://github.com/serilog-contrib/serilog-sinks-richtextbox)
- [Serilog.Sinks.WPF](https://github.com/umairsyed613/Serilog.Sinks.WPF)
- [Serilog.Sinks.TextWriter](https://github.com/serilog/serilog-sinks-textwriter)
- [Serilog.Sinks.Memory](https://github.com/pmiossec/serilog-sinks-memory)

---

# 文檔

## CsvHelper

- [官方網站](https://joshclose.github.io/CsvHelper/)
- [GitHub](https://github.com/JoshClose/CsvHelper)
- [Documentation](https://joshclose.github.io/CsvHelper/)
- [使用 C# 與 CsvHelper 套件解析《臺北市政府行政機關辦公日曆表》公開資料 | Will 保哥](https://blog.miniasp.com/post/2022/11/12/Parsing-Taipei-Gov-OpenData-Taiwan-Holiday-using-CsvHelper)
- [使用 CsvHelper 讀取 Csv 檔案](https://blog.ite2.com/using-csvhelper-to-read-csv-files/)
- [[C#][.NET Core] CsvHelper : 透過 C# 讀寫 csv 檔案](https://dog0416.blogspot.com/2019/11/aspnet-core-csvhelper-c-csv.html)

## MiniExcel

- [MiniExcel](https://github.com/mini-software/MiniExcel)

## PDFsharp & MigraDoc 6

- [GitHub](https://github.com/empira/PDFsharp)
- [主頁](https://docs.pdfsharp.net/)
- [.NET 小技巧 - 使用 PdfSharp / PdfSharpCore 合併 PDF、加浮水印](https://blog.darkthread.net/blog/pdfsharp)

## PdfSharpCore

- [GitHub](https://github.com/ststeiger/PdfSharpCore)
- [Documentation](https://github.com/ststeiger/PdfSharpCore/blob/master/docs/index.md)

---

# 外接介面

## TouchSocket

- [主頁](https://rrqm_home.gitee.io/touchsocket/)
- [Gitee](https://gitee.com/rrqm_home/touchsocket)
- [GitHub](https://github.com/RRQM/TouchSocket)
- [文档](https://rrqm_home.gitee.io/touchsocket/docs/current/)
- [作者的教學視頻](https://space.bilibili.com/94253567/video)
- [作者的 Blog](https://blog.csdn.net/qq_40374647?type=blog)

## IoTClient

- [IoTClient - Gitee [农码一生]](https://gitee.com/zhaopeiym/IoTClient)
- [IoTClient - GitHub [zhaopeiym]](https://github.com/zhaopeiym/IoTClient)
- [IoTClient Tool](https://gitee.com/zhaopeiym/IoTClient.Examples)
- [物联网基础组件 IoTClient 开发系列 [农码一生]](https://www.cnblogs.com/zhaopei/p/11651790.html)

## Serial Port

- [System.IO.Ports.SerialPort](https://learn.microsoft.com/zh-tw/dotnet/api/system.io.ports.serialport)
- [Serial Port Stream](https://github.com/jcurl/RJCP.DLL.SerialPortStream)
- [If you _must_ use .NET System.IO.Ports.SerialPort](https://www.sparxeng.com/blog/software/must-use-net-system-io-ports-serialport)
- [.Net Core 跨平台应用研究-CustomSerialPort(增强型跨平台串口类库)](https://www.cnblogs.com/flyfire-cn/p/10434171.html)

---

# 測試

- [Bogus](https://github.com/bchavez/Bogus)
- [Cat Facts API](https://catfact.ninja/)

---

# Blog

- [InCerry](https://www.cnblogs.com/InCerry/)

---

# Visual Studio

## 延伸模組

- XAML Styler
- ResXManager
- Prism Template Pack

## Costura.Fody

- [.NET 将多个程序集合并成单一程序集的 4+3 种方法](https://blog.walterlv.com/post/how-to-merge-dotnet-assemblies.html)
- [.NET(C#) 使用 Costura.Fody 将程序发布成单个 exe 文件](https://www.cjavapy.com/article/2696/)
- [Use Costura.Fody to bundle all assemblies into one](http://dalechang.blogspot.com/2016/04/use-costurafody-to-bundle-all.html)

## 程式目標版本選擇

- [跨平台目標設定](https://learn.microsoft.com/zh-tw/dotnet/standard/library-guidance/cross-platform-targeting)
- [微軟停止更新.NET Standard 並由.NET 5 取代](https://www.ithome.com.tw/news/140028)
- [淺談 .NET 類別程式庫跨平台開發](https://blog.darkthread.net/blog/netstandard-class-library/)

## 網路文章

- [如何更精准地设置 C# / .NET Core 项目的输出路径？](https://blog.walterlv.com/post/the-properties-that-affetcs-project-output-path.html)
- [[.NET] 將共用的元件庫放在指定的目錄中](https://dotblogs.com.tw/maduka/2017/08/20/121309)
- [.NET 知識高裝檢 - .pdb 檔、編譯最佳化與偵錯](https://blog.darkthread.net/blog/about-dotnet-pdb/)

---

# Visual Studio Code

## 延伸模組

- C# for Visual Studio Code

## Markdown

## 其他文章

- [必備的 Visual Studio Code 套件](https://chiahsien.github.io/post/visual-studio-code-extensions/)

https://www.cnblogs.com/taogeli/p/16046892.html
