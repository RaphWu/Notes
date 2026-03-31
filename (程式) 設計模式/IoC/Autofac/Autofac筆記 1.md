---
aliases:
date: 2011-06-07
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-1/
tags:
  - CSharp
  - IoC
  - Autofac
---

# Autofac 筆記 1

決定在專案引進 IoC，端午假期在網路上划龍舟，做了些研究。忘性愈來愈大，故筆記不可少，就順便跟大家分享。(對於 IoC，我仍在初學摸索階段，諸位先進如發現有誤入歧途之處，還請指正)

## **【IoC?】**

IoC 是什麼? DI 又是什麼? 很多.NET 領域的老師及同學已深入研究並有專文介紹，此處不再班門弄斧:

- 保哥的文章: [Unity Application Block 與 ASP.NET MVC 學習資源整理](http://blog.miniasp.com/post/2009/09/28/Unity-Application-Block-and-ASPNET-MVC-Learning-Resources.aspx)
- 91 的介紹: [Software Architecture]IoC and DI](http://www.dotblogs.com.tw/hatelove/archive/2009/10/02/10894.aspx) (Spring.NET) , [[ASP.NET]重構之路系列v4 – 簡單使用interface之『你也會IoC』](http://www.dotblogs.com.tw/hatelove/archive/2011/05/26/refactoring-using-interface-to-ioc.aspx)
- 黃忠成老師的 Object Builder Application(Unity 的前身) 剖析: [Inside ObjectBuilder Part1](http://www.dotblogs.com.tw/code6421/archive/2008/05/07/3837.aspx), [Inside ObjectBuilder Part 2](http://www.dotblogs.com.tw/code6421/archive/2008/05/07/3838.aspx), [Inside ObjectBuilder Part 3](http://www.dotblogs.com.tw/code6421/archive/2008/05/07/3839.aspx), [Inside ObjectBuilder 范例](http://www.dotblogs.com.tw/code6421/archive/2008/05/07/3841.aspx)
- [可抽換元件設計模式- _IoC_ Pattern - Pete.NET- _點部落_](http://www.google.com.tw/url?sa=t&source=web&cd=1&ved=0CCgQFjAA&url=http%3A%2F%2Fwww.dotblogs.com.tw%2Fpetedotnet%2Farchive%2F2011%2F04%2F01%2Finversion_of_control_pattern.aspx&ei=dijsTdr9E4LyvwPK-YjHDw&usg=AFQjCNEd7xFDrEhLvMOMJbZ2qE6aWTq0ZQ)
- [[Object-oriented] : 控制反轉- 昏睡領域- _點部落_](http://www.google.com.tw/url?sa=t&source=web&cd=5&ved=0CEEQFjAE&url=http%3A%2F%2Fwww.dotblogs.com.tw%2Fclark%2Farchive%2F2010%2F11%2F29%2F19772.aspx&ei=dijsTdr9E4LyvwPK-YjHDw&usg=AFQjCNGk7jDEc3ZvyrKXDQvtZAXUOWk_9Q)

## **【比較】**

.NET 可用的 IoC Framework 百家爭鳴，但弱水三千也只能飲一瓢，頭痛的抉選時刻來了。

- [Ninject, StructureMap, Unity, Spring.NET, Windsor, Autofac的精簡比較](http://stackoverflow.com/questions/411660/enterprise-library-unity-vs-other-ioc-containers)
- [Comparing .NET DI (IoC) Frameworks, Part 1](http://blog.ashmind.com/2008/08/19/comparing-net-di-ioc-frameworks-part-1/)
- [Comparing .NET DI (IoC) Frameworks, Part 2](http://blog.ashmind.com/2008/09/08/comparing-net-di-ioc-frameworks-part-2/)
    (作者結論傾向 Autofac，註冊遞迴錯誤處理是一大原因。Ninject 在當時的 Build 版本不穩、Unity 輸在建構式選取 ' 及 Property 注入，StructureMap 少了遞迴錯誤處理及建構式選取，Spring.NET 的學習曲線太陡，功能強大，但常會陷入一堆 XML 手工。)
- Scott Hanselman 列舉的 IoC 容器清單: [List of .NET Dependency Injection Containers (IOC)](http://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx)
- 系列文章: [IoC in .NET part 1: Autofac](http://geekswithblogs.net/Sharpoverride/archive/2009/08/15/ioc-in-.net-part-1-autofac.aspx), [IoC in .NET part2: StructureMap](http://geekswithblogs.net/Sharpoverride/archive/2009/08/16/ioc-in-.net-part2-structuremap.aspx), [IoC in .NET part 3: Ninject 2 beta](http://geekswithblogs.net/Sharpoverride/archive/2009/08/17/ioc-in-.net-part-3-ninject-2-beta.aspx), [IoC in .NET part4: Spring.NET](http://geekswithblogs.net/Sharpoverride/archive/2009/08/18/ioc-in-.net-part4-spring.net.aspx), [IoC in .NET part 5: Using CastleWindsor container](http://geekswithblogs.net/Sharpoverride/archive/2009/08/17/ioc-in-.net-part-5-using-castlewindsor-container.aspx), [IoC Containers in .NET part 6: Unity Container](http://geekswithblogs.net/Sharpoverride/archive/2009/08/20/ioc-containers-in-.net-part-6-unity-container.aspx)
- 對岸 MVP 的 [Autofac心得](http://www.cnblogs.com/shanyou/category/232884.html)

看過一些分析後，我對 Unity 與 Autofac 比較感興趣。Unity 是微軟主推的 IoC 解決方案，有很豐富的教學資源 (甚至有教學影片)、Autofac 的特色則在全力發揮 C#語言的特性，許多地方沿襲了 LINQ/Lambda 風格，簡潔扼要，很合我的胃口。(例如: `builder.RegisterAssemblyTypes(typeof(MyModel).Assembly).Where(t => t.Name.EndsWith(“Repository”).OnActivating(e => ((IRepository)e.Instance).Init()); )` 而 Autofac 的開發社群挺活躍的，愛好者也不少，看來是可以安心選用的工具。

對 " 簡潔語法 " 總是有說不出的著迷，一如過去對 jQuery/LINQ 的一見鍾情，這回決定以 Autofac 當成 IoC Container 的學習入口，若發現有什麼重大缺失再轉向也不遲。

## 【術語】

1. **Component**: 要註冊到 Container 的類別，當其他 Component 需要某項 Service 時，Container 會依需要建立 Component。
2. **Service**: Component 所提供的功能，多半以 Interface 方式定義，Component 會實作這些定義
3. **Autowiring**: Container 多半有能力自動找到 Dependency Injection 的注射點，例如: 比對建構式的參數型別，提供所需的 Service。
4. **Transient Component**: 與 Singleton 成對比，指每次 Container 被要求時都新建一個 Instance 提供服務。(各 Framework 用的名詞可能有出入)
5. **Automatic Registration**: 找出組件或環境裡的 Component，自動註冊。
6. **CommonServiceLocator**: 把 IoC Container 再抽象化，呼叫時透過統一的介面取得服務，不必去管底層使用何種 IoC Container([中文參考](http://www.dotblogs.com.tw/wadehuang36/archive/2011/01/14/commonservicelocator.aspx))。目前支援 Castle Windsor, Spring.NET, Unity, StructureMap, Autofac, MEF, LinFu。不過，要求彈性就要付出代價，異中求同代表只能使用各 Container 都支援的功能，無法將特定 Container 的優勢發揮到極致，如果不需要切換 Container，

## 【初體驗】

我讀的第一篇入門文章是 CodeProject 網站的一篇教學: [http://www.codeproject.com/KB/architecture/di-with-autofac.aspx](http://www.codeproject.com/KB/architecture/di-with-autofac.aspx)。它用一個備忘事項提醒的類別當範例，剛好涵蓋常見的基本用法，所以我也實地跟著做一回，順便加上自己的一些詮釋。

先設計一個備忘事項檢查器類別，建構時傳入備忘事項資料來源及通知服務，`CheckNow()` 會檢查是否已有到期事項，若有，就呼叫通知服務送出通知。

```csharp
using System;
using System.Linq;
using System.IO;
 
namespace AutofacLab
{
    //備忘事項類別
    class Memo
    {
        public string Title;
        public DateTime DueAt;
    }
 
    #region 通知備忘事項
    //產生通知動作的呼叫介面
    interface IMemoDueNotifier
    {
        void MemoIsDue(Memo memo);
    }
    //實作由TextWriter輸出的通知服務
    class PrintingNotifier : IMemoDueNotifier
    {
        TextWriter _writer;
        // 傳入TextWriter的建構式
        public PrintingNotifier(TextWriter writer)
        {
            _writer = writer;
        }
        // 輸出通知訊息到TextWriter
        public void MemoIsDue(Memo memo)
        {
            _writer.WriteLine("Memo '{0}' is due!", memo.Title);
        }
    }
    #endregion
 
 
    #region 檢查備忘錄
    class MemoChecker
    {
        IQueryable<Memo> _memos;
        IMemoDueNotifier _notifier;
 
        // 建立備忘錄檢查器，顯示到期待辦事項
        public MemoChecker(IQueryable<Memo> memos, IMemoDueNotifier notifier)
        {
            _memos = memos;
            _notifier = notifier;
        }
 
        // 依目前日期找出已到期項目
        public void CheckNow()
        {
            var overdueMemos = _memos.Where(memo => memo.DueAt < DateTime.Now);
 
            foreach (var memo in overdueMemos)
                _notifier.MemoIsDue(memo);
        }
    }
    #endregion
}
```

接著，我們寫一段 Console 程式來呼叫它。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
 
namespace AutofacLab
{
    class Program
    {
        //在記憶體中建立一個IQueryable<Memo>資料來源
        static IQueryable<Memo> GenSomeMemos()
        {
            IQueryable<Memo> memos = new List<Memo>() {
                new Memo { Title = "Release Autofac 1.0", 
                           DueAt = new DateTime(2007, 12, 14) },
                new Memo { Title = "Write CodeProject Article", 
                           DueAt = DateTime.Now },
                new Memo { Title = "End of The World", 
                           DueAt = new DateTime(2012, 12, 21) }
            }.AsQueryable();
            return memos;
        }
 
        static void Main(string[] args)
        {
            NoDI();
            Console.WriteLine("Done! press any key to exit...");
            Console.Read();
        }
 
 
        static void NoDI()
        {
            //傳統寫法，物件的產生是寫死的
            MemoChecker chkr = new MemoChecker(GenSomeMemos(),
                new PrintingNotifier(Console.Out));
            chkr.CheckNow();
        }
 
    }
}
```

建構 `MemoChecker` 的地方很簡單，用一個函數產生 `IQueryable<Memo>` 作為第一個參數，當場傳入 `Console.Out` 建立 `PrintingNotifier` 當成第二個參數，簡單明瞭。如果不考慮未來調整的彈性，沒什麼不好。

```csharp
MemoChecker chkr = new MemoChecker(GenSomeMemos(),
	new PrintingNotifier(Console.Out));
chkr.CheckNow();
```

下一步，用 VS2010 在專案中用 NuGet 加入 Autofac (沒錯，又是 [NuGet](http://blog.darkthread.net/post-2011-03-12-nuget.aspx)，大家有沒有開始覺得它很好用?):

![](https://blog.darkthread.net/Photos/1191-f714-l.jpg)

接著改寫程式讓 Autofac 來管理物件的建構，我將說明加在註解裡，直接看程式:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Autofac;
using System.IO;
 
namespace AutofacLab
{
    class Program
    {
        //在記憶體中建立一個IQueryable<Memo>資料來源
        static IQueryable<Memo> GenSomeMemos()
        {
            //...略...
        }
 
        static ContainerBuilder builder;
 
        static void Main(string[] args)
        {
            builder = new ContainerBuilder();
            //註冊MemoChecker，建構式的參數也由Container產生
            builder.Register(c => new MemoChecker(
                c.Resolve<IQueryable<Memo>>(),
                c.Resolve<IMemoDueNotifier>())
                );
            //將PrintingNotifier註冊成IMemoDueNotifier的預設來源
            builder.Register(c => new PrintingNotifier(
                c.Resolve<TextWriter>())).As<IMemoDueNotifier>();
            IQueryable<Memo> memos = GenSomeMemos();
            //直接註冊現有物件是很方便的玩法，在此註冊IQueryable<Memo>的Instance
            //當需要IQueryable<Memo>時，就會用它
            builder.RegisterInstance(memos);
            //Console.Out有實作IDisposable，但不應該由Container來終結
            //所以要加上ExternallyOwned
            builder.RegisterInstance(Console.Out).As<TextWriter>().ExternallyOwned();
 
            InAutofacWay();
            Console.WriteLine("Done! press any key to exit...");
            Console.Read();
        }
 
        static void InAutofacWay()
        {
            //使用using包住Container
            //當程式結束時Autofac會自動處理需要Dispose的物件
            using (var container = builder.Build())
            {
                container.Resolve<MemoChecker>().CheckNow();
            }
        }
    }
}
```

在 `Main()` 裡，我們花了不少精力註冊介面、型別，在註冊時，連建構式參數都試圖不寫死，例如: `new MemoChecker(c.Resolve<IQueryable<Memo>>()`, `c.Resolve(IMemoDueNotifier))`、`new PrintingNotifier(c.Resolve<TextWriter>())`，意思是 `IQueryable<Memo>`、`IMemoDueNotifier`、`TextWriter` 要傳入什麼，都可以用 `Container` 全權決定，保留最大彈性。而呼叫程式只剩下 `container.Resolve<MemoChecker>().CheckNow()`，沒用到任何建構式，也不用指定任何參數。

但，明明程式執行結果與 `NoDI()` 相同，為什麼要多寫了這麼多程式嗎? 如此大費周章只為了換來一個好處，未來要改變資料來源、換掉 `TextWriter` 的輸出對象、甚至改成 `MessageBox` 彈出訊息，`InAutofacWay()` 的程式完全不用做任何更動。在這個例子裡，搞了大半天只為求不想修改 `InAutofacWay()`，沒什麼了不起；但想像一下，如果是在一個龐大且複雜的系統中要將資料來源由查詢資料庫改成從讀檔案取資料，會影響系統中數百段用到資料查詢來源的程式碼，現在只需要調整 `builder.Register()` 就瞬間搞定，IoC Container 的威力就顯現了。

另外還有一點很重要，在進行單元測試時，我們亦可透過 IoC Container 抽換非測試重點的相關元件，讓測試得以順利進行。例如: 檢查庫存方法 `CheckStock()` 原本需要一個 `IStockDataProvider` 從 SQL 資料庫取得存貨資料，但單元測試期間，資料庫根本連個影子都沒有，此時在 IoC Container 註冊測試用的假 `IStockDataProvider` 物件，模擬提供必要的資料。`CheckStock()` 中的 `container.Resolve<IStockDataProvider>().GetInventory(…)` 就可自動建構測試用假物件，順利執行測試。

## 【抽換實驗】

再來，準備來玩一下抽換遊戲，驗證 IoC Container 的功效，確認我們多寫的程式碼不是寫心酸的。

### 實驗 1

將 `TextWriter` 換成 `StreamWriter`，輸出到 `yyyyMMdd.log`。執行後可以在檔案中看到原本輸出到 `Console.Out` 的內容。

因採用附加文字內容的做法，`StreamWriter` 每次使用完畢可 `Dispose()`，故不用像 `Console.Out` 加註 `ExternallyOwned()`，在 `using (var container = …)` 結束後，`StreamWriter` 就會被 `Dispose()`。

```csharp
//...省略...
	//很方便的做法，直接註冊一個IQueryable<Memo>的Instance
	//當需要IQueryable<Memo>時，就會用它
	builder.RegisterInstance(memos);
	//將內容附加在yyyyMMdd.log(註: 此處未考慮Multi-Threading問題)
	builder.Register<TextWriter>(c => new StreamWriter(
		string.Format("F:\\{0:yyyyMMdd}.log", DateTime.Today), true));

	InAutofacWay();
//...省略...
```

### 實驗 2

新增一個 `MsgBoxNotifier` (專案要參考 `System.Windows.Form.dll`)

```csharp
class MsgBoxNotifier : IMemoDueNotifier
{
	public void MemoIsDue(Memo memo)
	{
		System.Windows.Forms.MessageBox.Show(
			string.Format("Memo '{0}' is due!", memo.Title)
			);
	}
}
```

接著註冊 `MsgBoxNotifier` 為預設通知服務，此時結果會以 `MessageBox` 方式呈現。

```csharp
//...省略...
	builder.Register(c => new MemoChecker(
		c.Resolve<IQueryable<Memo>>(),
		c.Resolve<IMemoDueNotifier>())
		);
	//將MsgBoxNotifier註冊成IMemoDueNotifier的預設來源
	  //這裡用AsImplementedInterfaces()，因MsgBoxNotifier實作了IMemoDueNotifier
	//等同於As<IMemoDueNotifier>()
	builder.Register(c => new MsgBoxNotifier()).AsImplementedInterfaces();

	IQueryable<Memo> memos = GenSomeMemos();
//...省略...
```

在不更動 `InAutofacWay()` 半行程式的前提下，我們可以任意改變其串接的服務，這就是費心導入 IoC Conatiner 所追求的。

Autofac 還有不少細節，後篇再續。
