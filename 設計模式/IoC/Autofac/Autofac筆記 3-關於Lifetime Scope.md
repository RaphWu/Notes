---
aliases:
date: 2013-11-03
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-3-lifetime-scope/
tags:
  - CSharp
  - IoC
  - Autofac
---

# Autofac 筆記 3- 關於 Lifetime Scope

在使用 IoC 設計模式時，有一個有點難懂卻不能迴避的問題 -- 如何妥善管理物件生命週期，避免記憶體洩漏 (Memory Leak)?

要了解此議題，先大推一篇關於 Autofac 物件生命週期的 [經典文章](http://nblumhardt.com/2011/01/an-autofac-lifetime-primer/)，其中有頗詳細的闡述，這篇筆記只簡短摘要我實際應用的心得，關於完整說明推薦大家參考原文。

## 問題從何而來?

基本上，純 .NET 世界的資源 (Managed Resource，例如: 儲存 .NET 物件所用的記憶體) 有 GC (Garbage Collection) 機制把關，它能精準掌握物件是否仍在有效範圍，當物件已不可能再被使用，便會在必要時 (例如: 剩餘記憶體不足) 回收這些已消滅物件所耗用的記憶體。但程式要運行，多少會涉及一些非 .NET 所掌管的資源 (Unmanged Resource，例如: 網路連線、磁碟機上的檔案... 等等)，當.NET 物件使用到這些 Unmanaged Resource，建議的做法是實作 `IDispose` 介面，在 `Dispose()` 方式中確實釋放所有動用到的資源，以免.NET 物件消失後佔著茅坑不拉屎，阻礙其他 Process 使用。

一般來說，若物件有實作 `IDispose`，我們可寫成 `using (var boo = new SomeDisposableClass()) { … }`，確保範圍結束後一定呼叫 `Dispose()` 釋放資源。但 `using` 只適用單一 Method 內部，若物件建立好要交給其他程式應用，便不能任意 `Dispose()`，以免別人要用時已成廢物；但是，若不確實在物件使用完畢後呼叫 `Dispose()`，又會造成資源不當佔用。這是個難題，且沒有什麼神奇解法，設計實務多半靠 " **建立物件者必須負責善後** " 原則作為解決方案。

應用 Autofac 時，物件都是透過 `Container.Resolve<SomeType>()` 方式取得，換言之，物件是由 Autofac 建立，那麼 `Dispose()` 也會由 Autofac 呼叫嗎? 做個實驗吧! 寫個實作 `IDisposable` 的類別:

```csharp
using System;
 
class ResourceMonster : IDisposable
{
    public string Name = "Anonymous";
    public void Test()
    {
        Console.WriteLine("{0}: Hi there.", Name);
    }
    public void Dispose()
    {
        Console.WriteLine("{0}: Disposed.", Name);
    }
}
```

測試程式如下:

```csharp
using Autofac;
using System;
 
namespace Lifetime
{
    class Program
    {
        static void Main(string[] args)
        {
            ContainerBuilder builder = new ContainerBuilder();
            builder.RegisterType<ResourceMonster>();
            IContainer container = builder.Build();
 
            var monster = container.Resolve<ResourceMonster>();
            monster.Test();
            Console.WriteLine("Before IContainer Dispose");
            container.Dispose();
            Console.WriteLine("After IContainer Dispose");
 
            Console.ReadLine();
        }
    }
}
```

```plantext
Anonymous: Hi there.
Before IContainer Dispose
Anonymous: Disposed.
After IContainer Dispose
```

執行結果如上，`IContainer` 有實作 `IDisposable`，當我們呼叫 `container.Dispose()`，`container` 會盡責地善後 -- 呼叫它所建立 `monster` 物件的 `Dispose()` 方法。

測試程式只用於示範，因此在 `Main()` 單一方法內註冊型別、建立 `IContainer`，用完就馬上把 `IContainer.Dispose()`，但這不符合應用實。`IContainer` 建立要註冊所有列管型別，需要耗費資源、時間，不可能每次使用前才建立，用完就丟，下次要用再重建。因此 `IContainer` 整個 Process 多半只會建一份，通常安排在程式 (或網站) 啟動事件中建立好，並以 `static` 屬性方式供整個 Process 共用。在某個 Method 呼叫 `Resolve<T>` 建立的物件，不可能依賴 `IContainer.Dispose()` 善後 (因為其他人還要繼續用它建立物件)，為此，Autofac 提供了 `ILifetimeScope` 提供較短的生命週期應用。

運作原理是先透過 `IContainer.BeginLifetimeScope()` 建立 `ILifetimeScope` 取代 `IContainer`，它具有跟 `IConatiner` 幾乎一致的介面，如此我們便可改用 `ILifetimeScope.Resolve<T>` 建立物件，而 `ILifetimeScope` 算是為特定目所建的獨立容器，在使用完畢後可任意 `Dispose()` 而不會影響 `IContainer`。另外，`ILifetimeScope` 還可以透過 `.BeginLifetimeScope()` 再建立子 `ILifetimeScope` 形成巢狀結構，上層容器 `Dispose()` 時會一併呼叫下層容器進行 `Dispose()`，可依不同需求彈性應用。

以下是簡單的 `ILifetimeScope` 範例，先透過 `container.BeginLifetimeScope()` 建立 `ILifetimeScope`，再改用它來 `Resolve<ResourceMonster>()` 取得物件，透過 `using` 自動終結 `scope` (`using` 結束時背後會呼叫 `scope.Dispose()`)，由執行結果可觀察到兩個 `ResourceMonster` 物件也自動被 `Dispose()`:

```csharp
using Autofac;
using System;
 
namespace Lifetime
{
    class Program
    {
        static IContainer container = null;
        static void AutofacConfig()
        {
            ContainerBuilder builder = new ContainerBuilder();
            builder.RegisterType<ResourceMonster>();
            container = builder.Build();
        }
        static void Test()
        {
            using (var scope = container.BeginLifetimeScope())
            {
                var monster1 = scope.Resolve<ResourceMonster>();
                monster1.Name = "No1";
                monster1.Test();
                var monster2 = scope.Resolve<ResourceMonster>();
                monster2.Name = "No2";
                monster2.Test();
            }
        }
 
        static void Main(string[] args)
        {
            AutofacConfig();
            Test();
            Console.ReadLine();
        }
    }
}
```

執行結果:

```plantext
No1: Hi there.
No2: Hi there.
No2: Disposed.
No1: Disposed.
```

另外，先前介紹過 [Autofac Singleton](http://blog.darkthread.net/post-2013-11-02-autofac-notes-2.aspx)，`ILifetimeScope` 也可當作 `Instance` 共用的單位，例如: 指定只建一個 `Instance` 供全 `ILifetimeScope` 共用，做法是在 `RegisterType<T>` 時加上 `InstancePerLifetimeScope()`，例如:

```csharp
static IContainer container = null;
static void AutofacConfig()
{
	ContainerBuilder builder = new ContainerBuilder();
	builder.RegisterType<ResourceMonster>();
	builder.RegisterType<TheNewOne>().InstancePerLifetimeScope();
	container = builder.Build();
}

static void Test2()
{
	var scope1 = container.BeginLifetimeScope();
	var scope2 = container.BeginLifetimeScope();
	var one1 = scope1.Resolve<TheNewOne>();
	Console.WriteLine("1->");
	one1.ShowUniqueKey();
	var one2A = scope2.Resolve<TheNewOne>();
	var one2B = scope2.Resolve<TheNewOne>();
	Console.WriteLine("2A->");
	one2A.ShowUniqueKey();
	Console.WriteLine("2B->");
	one2B.ShowUniqueKey();
}

static void Main(string[] args)
{
	AutofacConfig();
	Test2();
	Console.ReadLine();
}
```

執行結果如下，如預期兩個 `ILifetimeScope` 所取得的 `TheNewOne` Unique Key 不同，而第二個 `ILifetimeScope` 取得的兩個 `TheNewOne` Unique Key 相同，證明為同一個 Instance。

```plaantext
Constructor Executed
1->
Unique Key=de3fdf54-a67e-4896-98b4-d7ab1314dbe8
Constructor Executed
2A->
Unique Key=7dc4ac10-aafa-4630-8a64-b517cde8fc82
2B->
Unique Key=7dc4ac10-aafa-4630-8a64-b517cde8fc82
```

最後提一下 Owned Instance，指 Autofac 在 `Resolve<T>` 時先產生新的 `ILifetimeScope`，再用它建立物件並傳回 `ILifetimeScope`。寫法為 `var ownedService = conatiner.Resolve<Owned<SomeType>>()`，而 `ownedService.Value` 即為所建立的 SomeType Instance，而 `ownedService.Dispose()` 則可用來結束該 `ILifetimeScope`。

## 結論

在大多數應用情境下，建議改用 `ILifetimeScope` 取代 `IContainer` 物件 `Resolve<T>()`，並在作業結束後執行 `ILifetimeScope.Dispose()` 以落實資源回收，減少記憶體洩漏的風險。

## 延伸閱讀

- [Nicholas Blumhardt  An Autofac Lifetime Primer](http://nblumhardt.com/2011/01/an-autofac-lifetime-primer/ "Nicholas Blumhardt  An Autofac Lifetime Primer")
- [Autofac Instance Scope](https://code.google.com/p/autofac/wiki/InstanceScope)
