---
aliases:
date: 2013-11-02
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-2/
tags:
  - CSharp
  - IoC
  - Autofac
  - 設計模式
  - 單例模式_Singleton
---

# Autofac 筆記 2- 淺談 Singleton

開始前先聲明 (坦白從寬~)，我對 Design Pattern 的研究十分淺薄，寫起 IoC、Singleton 的題材有種越級打怪的心虛感，我知道本部落格有不少讀者深諳此道，如筆記有疑或有誤之處，懇請十方大德不吝指正。

[Singleton](http://openhome.cc/Gossip/DesignPattern/SingletonPattern.htm) 是挺常見的設計模式，旨在確保該型別於 `Process` 中只會產生單一 Instance (執行個體)。在.NET 實現 Singleton 慣用的做法是將建構式設成 `private`，另外宣告一個 `static` 屬性命名為 `Instance`，在第一次 get 時建立物件，之後每次要取用該類別時不再重新建構，而是直接取用 `Instance` 屬性，如下例:

```csharp
using System;
using System.Threading;
 
public class TheOne
{
    private Guid UniqueKey = Guid.NewGuid();
 
    private static TheOne instance = null;
    public static TheOne Instance
    {
        get
        {
            if (instance == null)
            {
                instance = new TheOne();
            }
            return instance;
        }
    }
 
    /// <summary>
    /// 建構式
    /// </summary>
    private TheOne()
    {
        Thread.Sleep(2000);
        Console.WriteLine("Constructor Executed");
    }
 
    public void ShowUniqueKey()
    {
        Console.WriteLine("Unique Key={0}", UniqueKey);
    }
}
```

應用時透過 `TheOne.Instance` 取得唯一的執行個體:

```csharp
static void Test1()
{
	for (int i = 0; i < 3; i++)
	{
		TheOne theOne = TheOne.Instance;
		theOne.ShowUniqueKey();
	}
}
```

執行後可驗證 `TheOne` 只被建構了一次，三次使用的都是同一 `Instance`。

```plantext
Constructor Executed
Unique Key=571842aa-5037-43db-9341-6e82f0ebe6d0
Unique Key=571842aa-5037-43db-9341-6e82f0ebe6d0
Unique Key=571842aa-5037-43db-9341-6e82f0ebe6d0
```

不過，老鳥們都知道上述寫法未考慮 Thread-Safe，在多執行緒下肯定破功。以下我們就來踢爆這個 " 黑心 Singleton"(誤):

```csharp
static void Test2()
{
	for (int i = 0; i < 3; i++)
	{
		ThreadPool.QueueUserWorkItem((o) =>
		{
			TheOne theOne = TheOne.Instance;
			theOne.ShowUniqueKey();
		});
	}
}
```

當改用 ThreadPool 以三個執行緒同時存取 TheOne。很好! 建構式跑了三次，生出三個 TheOne…

```plantext
Constructor Executed
Constructor Executed
Unique Key=a3f52f2f-6097-4a90-8a26-5cb91b12c484
Constructor Executed
Unique Key=a06da108-c38b-43c9-aa01-04f5e261c121
Unique Key=37063ff3-990c-44a0-ac0a-798ea0bcf1ae
```

微軟有一篇很棒的 [文章](http://msdn.microsoft.com/en-us/library/ff650316.aspx?WT.mc_id=DOP-MVP-37580) 詳細討論了.NET Singleton 實作，如果要做到 Thread-Safe，`TheOne` 最好加上雙重檢查鎖定 (_Double-Check Locking)_ 機制並改寫如下:

```csharp
private static TheOne instance = null;
private static object syncRoot = new Object();
public static TheOne Instance
{
	get
	{
		if (instance == null)
		{
			lock (syncRoot)
			{
				if (instance == null)
					instance = new TheOne();
			}
		}
		return instance;
	}
}
```

如此就能確保多執行緒下也只產生唯一的 `Instance`。

Autofac 提供了另一種實現 Singleton 的選擇，做法是在 `ContainerBuilder` 註冊型別時呼叫 `SngleInstance()`，任何類別，不需特殊設計都能實現 Singletone。我們另外宣告一個 `TheNewOne` 類別，功能與 `TheOne` 類似，但直接提供 `public` 的建構式，省去 `static Instace` 屬性，Singleton 需求交給 Autofac 處理:

```csharp
using System;
using System.Threading;
 
public class TheNewOne
{
    private Guid UniqueKey = Guid.NewGuid();
 
    public TheNewOne()
    {
        Thread.Sleep(2000);
        Console.WriteLine("Constructor Executed");
    }
 
    public void ShowUniqueKey()
    {
        Console.WriteLine("Unique Key={0}", UniqueKey);
    }
}
```

測試時先宣告 `ContainerBuilder`，註冊 `TheNewOne` 型別並宣告 `SingleInstane()`，後續使用時只需透過 `ResolveType<TheNewOne>()` 取得 `TheNewOne`，就是 Singleton 了。

```csharp
static void Test3()
{
	ContainerBuilder builder = new ContainerBuilder();
	//註冊時加註SingleInstance()，Autofac便會以Singleton方式提供物件
	builder.RegisterType<TheNewOne>().SingleInstance();
	IContainer container = builder.Build();

	for (int i = 0; i < 3; i++)
	{
		ThreadPool.QueueUserWorkItem((o) =>
		{
			TheNewOne theOne = container.Resolve<TheNewOne>();
			theOne.ShowUniqueKey();
		});
	}
}
```

測試結果，類別不用特別加入 Singleton 邏輯就實現了多執行緒下的 Singleton。

```plantext
Constructor Executed
Unique Key=972def1e-8788-45aa-bd15-2aef15870514
Unique Key=972def1e-8788-45aa-bd15-2aef15870514
Unique Key=972def1e-8788-45aa-bd15-2aef15870514
```

【結論】透過 IoC(Autofac) 實現 Singleton，相形之下比自己 DIY 簡便許多，但有個缺點: 由於類別建構式公開，無法禁止開發人員繞過 Autofac 自行另建 Instance。但==在實務上，若架構啟用 Autofac 或任何 IoC，多會另外宣告 Interface，開發人員依賴的是 Interface 而非 Class 本身，對底層究竟使用何 Class 理應處於 " 無知 " 狀態，越過 Interface 直接存取類別可視為 " 違法亂紀 " 的罪行，視為人員管理問題而非架構缺陷。==依此前題，建構式公開就不算嚴重缺失，可安心服用。
