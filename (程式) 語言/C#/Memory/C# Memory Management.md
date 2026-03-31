---
aliases:
date:
update:
author: Sena Kılıçarslan
language: CSharp
sourceurl:
tags:
  - CSharp
  - CSharp_Span/Memory
---

# Links

[C# Memory Management — Part 1](https://medium.com/c-programming/c-memory-management-part-1-c03741c24e4b)
[C# Memory Management — Part 2 (Finalizer & Dispose)](https://medium.com/c-programming/c-memory-management-part-2-finalizer-dispose-d3b3e43c08d1)
[C# Memory Management — Part 3 (Garbage Collection)](https://medium.com/c-programming/c-memory-management-part-3-garbage-collection-18faf118cbf1)

---

    # C# Memory Management - Part 1 C# 内存管理 - 第一部分

In this article, I want to mention how memory management is done in .NET environment. I will try to keep it simple and short so that people with different levels of knowledge and experience can benefit from this. I believe this topic is very important in writing good-quality code and besides, it is popular in programming interviews.
在這篇文章中，我想談論 .NET 環境中的記憶體管理方式。我會盡量保持簡潔，以便不同知識和經驗水平的人都能受益。我相信這個主題在寫出高品質程式碼方面非常重要，而且它在程式設計面試中也很受歡迎。

I am planning to write this in three parts in order to make it easy to read. This part will include the topics:
我計劃將這篇文章分成三部分，以便於閱讀。這一部分將包含以下主題：

- Stack and heap  堆疊和堆
- Value types and reference types 值類型與參考類型

If you are ready, let’s start :)
如果你已準備好，讓我們開始吧 :)

## Stack and Heap  堆疊與堆

Stack and heap are portions of the memory. The **_Common Language Runtime (CLR)_** allocates memory for objects in these parts.
堆疊和堆是記憶體的一部分。通用語言執行平台 (CLR) 會在這些部分中為物件分配記憶體。

**Stack** is a simple LIFO(last-in-first-out) structure. Variables allocated on the stack are stored directly to the memory and access to this memory is very fast, and ==its allocation is done when the program is compiled==. When a method is invoked, the CLR bookmarks the top of the stack. The method then pushes data onto the stack as it executes. When the method completes, the CLR resets the stack to its previous bookmark, popping all the method’s memory allocations is one simple operation.
堆疊是一種簡單的後進先出（LIFO）結構。在堆疊上分配的變數直接儲存到記憶體中，且存取此記憶體非常快速，其分配是在程式編譯時完成的。當一個方法被呼叫時，CLR（通用語言執行平台）會標記堆疊的頂端。然後，該方法在執行時將資料推送到堆疊上。當方法完成時，CLR 會將堆疊重置到其先前的標記，彈出所有方法的記憶體分配是一個簡單的操作。

**Heap** can be viewed as a random jumble of objects. It allows objects to be allocated or deallocated in a random order. Variables allocated on the heap have their memory allocated at run time and accessing this memory is a bit slower, but the heap size is only limited by the size of virtual memory. The heap requires the overhead of a garbage collector to keep things in order.
堆疊可以看作是一堆隨機混亂的物件。它允許物件以隨機的順序進行分配或釋放。在堆疊上分配的變數在執行時分配記憶體，但存取此記憶體的速度稍慢，但堆疊的大小僅受虛擬記憶體大小的限制。堆疊需要垃圾收集器的開銷來保持秩序。

Value type variables are stored in the stack and reference type variables are stored in the heap. _(This is a very general statement based on the information_ [_here_](https://docs.microsoft.com/en-us/dotnet/api/system.valuetype?redirectedfrom=MSDN&view=netframework-4.7.2)_.)_
值類型變數存放在堆疊上，參考類型變數存放在堆疊上。（這是一個基於這裡的信息的非常普遍的陳述。）

![[CSharpMemoryManagement01_01.webp]]

## Value Type and Reference Type 值類型與參考類型

A **value type** holds the data within its own memory location.
一個值類別在它自己的記憶位置中持有資料。

Value types => _bool, byte, char, decimal, double, float, int, long, uint, ulong, ushort, enum, struct_
值類別 => bool, byte, char, decimal, double, float, int, long, uint, ulong, ushort, enum, struct

A **reference type** contains a pointer to another memory location that holds the real data.
一個參考類別包含一個指向另一個記憶位置的指標，該位置持有實際的資料。

Reference types => _class, interface, delegate, string, object, dynamic, arrays_
參考類別 => class, interface, delegate, string, object, dynamic, 陣列

If you assign a value type variable to another variable, the value is copied directly.
若將值類型變數指定給另一個變數，值會被直接複製。

Here is an example for a value type variable copy:
這是一個值類型變數複製的範例：

```csharp
static void Main(string[] args)
{
    int x = 3;
    int y = x;
    x = 5;
    Console.WriteLine($"x = {x}");
    Console.WriteLine($"y = {y}");
}
```

Output:  輸出：

![[CSharpMemoryManagement01_02.webp]]

As you see in the output, both variables work independently.
從輸出中可以看出，這兩個變數各自獨立運作。

If you assign a reference type variable to another, as reference type variables represent the address of the variable, the reference is copied and both variables point to the same location of the heap.
若將參考類型變數指定給另一個，由於參考類型變數代表變數的位址，參考會被複製，而這兩個變數都會指向堆疊中的相同位置。

Below is an example of reference variable copy:
以下是參考變數複製的範例：

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

static void Main(string[] args)
{
    Person alice = new Person { Name = "Alice", Age = 36 };
    Person bob = alice;
    bob.Name = "Bob";

    Console.WriteLine($"Alice's name: {alice.Name} \nBob's name: {bob.Name}");
}
```

Output:  輸出：

![[CSharpMemoryManagement01_03.webp]]

Have you noticed how Alice lost her name :) As these types of copies can result in unexpected behaviours in our programs, we should be careful when we copy reference type variables.
你注意到愛麗絲怎麼失去她的名字了嗎 :) 由於這類複製可能會導致程式出現不可預期的行為，我們在複製參考類型變數時應該小心。

Now we have a general idea about the stack, heap and value/reference type variables. Additionally, I want to provide two very helpful resources which explain in detail how variables are stored in the memory.
現在，我們對堆疊、堆和值/參考類型變數有了基本的了解。此外，我想提供兩個非常實用的資源，它們詳細解釋了變數如何在記憶體中儲存。

First one is [Jon Skeet](https://stackoverflow.com/users/22656/jon-skeet)‘s [blog post](http://jonskeet.uk/csharp/memory.html). Below is a citation from his post:
第一個是 Jon Skeet 的部落格文章。以下是他文章中的一段引用：

![[Memory in  .NET  - what goes where#So where are things stored? 那麼東西存放在哪裡呢？]]

Second is Wallace Kelly’s YouTube video. I strongly recommend you watch this video as he demonstrates the topic in a perfect way. After watching that, you will see how everything fall into place in your mind :)
第二是 Wallace Kelly 的 YouTube 影片。我強烈建議你觀看這個影片，因為他完美地展示了這個主題。觀看之後，你會看到所有的事情如何在你的腦中順利地組合起來 :)

![](https://www.youtube.com/watch?v=clOUdVDDzIM)

Here I conclude part one of this series. Hope you liked it and found helpful. Please let me know if you have any improvements and/or corrections.
我在這裡結束了這個系列的第一部分。希望你喜欢並發現它很有幫助。如果你有任何改進和/或修正，請告訴我。

_You can find the following posts below:
你可以找到以下帖子：_

- [Finalizer & Dispose](https://medium.com/@sena.kilicarslan/c-memory-management-part-2-finalizer-dispose-d3b3e43c08d1)
- [Garbage Collection](https://medium.com/c-programming/c-memory-management-part-3-garbage-collection-18faf118cbf1)

_If you want to find out more about value type and reference type variables, you can check the below posts:
如果您想了解更多有關值類型與參考類型變數的資訊，您可以參考以下文章：_

- [Passing Parameters in C#](https://medium.com/@sena.kilicarslan/passing-parameters-in-c-3e4ead58e384)
- [C# 7 Features — Ref Return and Ref Local](https://medium.com/@sena.kilicarslan/c-7-features-ref-return-and-ref-local-1004af8cdc3f)

Bye!  再見！

**References  參考資料**

[_https://docs.microsoft.com/tr-tr/dotnet/csharp/language-reference/keywords/value-types-table_](https://docs.microsoft.com/tr-tr/dotnet/csharp/language-reference/keywords/value-types-table)
[_https://docs.microsoft.com/tr-tr/dotnet/csharp/language-reference/keywords/reference-types_](https://docs.microsoft.com/tr-tr/dotnet/csharp/language-reference/keywords/reference-types)
[_http://net-informations.com/faq/net/stack-heap.htm_](http://net-informations.com/faq/net/stack-heap.htm)
[_http://www.albahari.com/valuevsreftypes.aspx_](http://www.albahari.com/valuevsreftypes.aspx)

---

# C# Memory Management— Part 2 (Finalizer & Dispose) C# 内存管理—第二部分

In this post, I will write about **Finalizer** and **Dispose** method and I will show how to use them with code examples.
在本篇文章中，我將會寫關於 Finalizer 和 Dispose 方法，並且會透過程式碼範例來展示如何使用它們。

Let’s start with why these are needed and used.
讓我們先從為什麼需要這些以及它們的使用開始談起。

**Garbage Collector** handles memory management in .NET framework and conducts the allocation and reclaiming of memory for the applications (_We will talk about the details of garbage collection in the_ [_third part_](https://medium.com/c-programming/c-memory-management-part-3-garbage-collection-18faf118cbf1) _of the series_). However, when we create objects that include unmanaged resources such as windows, files, network and database connections, we must explicitly release those resources after using them in our applications. This can be done by :
垃圾收集器 (Garbage Collector) 負責處理 .NET 框架中的記憶體管理，並進行應用程式的記憶體配置與回收（我們將在系列的第三部分中詳細討論垃圾收集的細節）。然而，當我們建立包含未管理資源（例如視窗、檔案、網路和資料庫連接）的物件時，必須在應用程式中使用它們之後明確釋放這些資源。這可以透過以下方式完成：

- using finalizers
  使用終結器
- implementing _Dispose_ method from the _IDisposable_ interface
  實作從 IDisposable 介面繼承的 Dispose 方法

First, let’s learn about finalizers.
首先，讓我們來學習終結器的相關知識。

## What is a finalizer?  終結器是什麼？

Finalizers (which are also called **destructors**) are used to perform any necessary final clean-up when a class instance is being collected by the garbage collector. Some important points about finalizers are:
解構子（也稱為解構子）用於在類別實例被垃圾收集器收集時執行任何必要的最終清理。有關解構子的一些重要點：

- A class can have only one finalizer.
  一個類別只能有一個解構子。
- A finalizer does not have modifiers or parameters.
  解構子沒有修飾詞或參數。
- Finalizers cannot be called explicitly, they are called by the garbage collector (GC) when the GC considers the object eligible for finalization. They are also called when the program finishes in .NET framework applications.
  解構子無法明確呼叫，它們由垃圾收集器（GC）在 GC 認為物件符合終結條件時呼叫。在 .NET 框架應用程式中，當程式結束時也會呼叫解構子。

Following is an example of a class with a finalizer:
以下是具有終結器的類別範例：

```csharp
class Person
{
    //properties
    public string Name { get; set; }
    public int Age { get; set; }

    public Person() //constructor
    {
        //initialization statements
    }

    ~Person() //finalizer (destructor)
    {
        //cleanup statements
    }
}
```

Finalizer overrides the _Finalize_ method of the base class. So the call to the finalizer is implicitly translated to the following code:
終結器會覆蓋基底類別的 `Finalize` 方法。因此，對終結器的呼叫會隱含地轉換為以下程式碼：

```csharp
    protected override void Finalize()
    {
        try
        {
            //cleanup statements
        }
        finally
        {
    
            base.Finalize();
        }
    }
```

We can see this by creating an example that includes an inheritance chain as follows:
我們可以透過建立包含以下繼承鏈的範例來看到這一點：

```csharp
class Person
{
    ~Person() //finalizer (destructor)
    {
        //cleanup statements
        Console.WriteLine("Person's finalizer is called.");
    }
}

class Employee : Person
{
    ~Employee()
    {
        //cleanup statements
        Console.WriteLine("Employee's finalizer is called.");
    }
}

class Manager : Employee
{
    ~Manager()
    {
        //cleanup statements
        Console.WriteLine("Manager's finalizer is called.");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var manager = new Manager();
    }
}
```

Output of this code is:
此程式碼的輸出為：

![[CSharpMemoryManagement02_01.webp]]

As you see, finalizers are called recursively for all instances in the inheritance chain, from the most-derived to the least-derived.
如你所見，終結器會針對繼承鏈中的所有實例進行遞迴呼叫，從最衍生到最少衍生。

## Dispose Method  Dispose 方法

We mentioned above that finalizers are called by the garbage collector or when the program finishes (_in .NET framework applications_). This means we cannot call them. ==If our application uses an expensive external resource, we should then release the resource explicitly. We can do this by implementing **_Dispose_** method from _IDisposable_ interface.== By this way, we can improve the performance of our application as well. Now, let’s see this in practice.
我們在上面提到，終結器會由垃圾收集器呼叫，或當程式結束時呼叫（在 .NET 框架應用程式中）。這表示我們無法呼叫它們。==如果我們的應用程式使用昂貴的外部資源，我們應該明確釋放這些資源。我們可以透過實作 `IDisposable` 接口中的 `Dispose` 方法來做到這一點。==這樣我們也能提升應用程式的效能。現在，讓我們來實際看看這個方法。

### How to Implement Dispose Method? 如何實作 Dispose 方法？

First, we create a class that implements **_IDisposable_** interface and then choose _Implement interface with Dispose pattern_ from the _Quick Actions_ menu as shown in the following image:
首先，我們創建一個實現 `IDisposable` 介面的類別，然後從快速動作菜單中選擇「實現介面與 Dispose 模式」選項，如下圖所示：

![[CSharpMemoryManagement02_02.webp]]

Then a skeleton code is generated as follows:
接著會產生以下範本程式碼：

```csharp
class DatabaseConnection : IDisposable
{

    #region IDisposable Support
    private bool disposedValue = false; // To detect redundant calls

    protected virtual void Dispose(bool disposing)
    {
        if (!disposedValue)
        {
            if (disposing)
            {
                // TODO: dispose managed state (managed objects).
            }

            // TODO: free unmanaged resources (unmanaged objects) and override a finalizer below.
            // TODO: set large fields to null.

            disposedValue = true;
        }
    }

    // TODO: override a finalizer only if Dispose(bool disposing) above has code to free unmanaged resources.
    // ~DatabaseConnection() {
    //   // Do not change this code. Put cleanup code in Dispose(bool disposing) above.
    //   Dispose(false);
    // }

    // This code added to correctly implement the disposable pattern.
    public void Dispose()
    {
        // Do not change this code. Put cleanup code in Dispose(bool disposing) above.
        Dispose(true);
        // TODO: uncomment the following line if the finalizer is overridden above.
        // GC.SuppressFinalize(this);
    }
    #endregion
}
```

I changed the code to make it usable and understandable as below:
我修改了程式碼使其可用且易於理解，如下所示：

```csharp
class DatabaseConnection : IDisposable
{

    #region IDisposable Support
    private bool disposedValue = false; // To detect redundant calls

    protected virtual void Dispose(bool disposing)
    {
        if (!disposedValue)
        {
            Console.WriteLine("This is the first call to Dispose. Necessary clean-up will be done!");

            if (disposing)
            {
                // TODO: dispose managed state (managed objects).
                Console.WriteLine("Explicit call: Dispose is called by the user.");
            }
            else
            {
                Console.WriteLine("Implicit call: Dispose is called through finalization.");
            }

            // TODO: free unmanaged resources (unmanaged objects) and override a finalizer below.
            Console.WriteLine("Unmanaged resources are cleaned up here.");

            // TODO: set large fields to null.

            disposedValue = true;
        }
        else
        {
            Console.WriteLine("Dispose is called more than one time. No need to clean up!");
        }
    }

    // TODO: override a finalizer only if Dispose(bool disposing) above has code to free unmanaged resources.
    ~DatabaseConnection()
    {
        // Do not change this code. Put cleanup code in Dispose(bool disposing) above.
        Dispose(false);
    }

    // This code added to correctly implement the disposable pattern.
    public void Dispose()
    {
       // Do not change this code. Put cleanup code in Dispose(bool disposing) above.
        Dispose(true);
        // TODO: uncomment the following line if the finalizer is overridden above.
        GC.SuppressFinalize(this);
    } 
    #endregion
}
```

I want to make some explanations about the variables and methods used in the pattern:
我想針對模式中使用的變數和方法做一些說明：

```csharp
private bool disposedValue = false; // To detect redundant calls
```

**_disposedValue_** boolean variable provides that clients can call the method multiple times without getting an exception.
`disposedValue` 這個布林變數提供客戶端可以多次呼叫方法而不會產生例外。

```csharp
protected virtual void Dispose(bool disposing)
```

This override of _Dispose_ method is either called by the **_client_**:
這個 `Dispose` 方法的覆寫是由客戶端呼叫：

```csharp
public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}
```

or the **_finalizer_**:  或是由完成者呼叫：

```csharp
~DatabaseConnection()
{
    Dispose(false);
}
```

As you see the distinction is provided by the **_disposing_** boolean variable.
正如你所見，這個區別是由 `disposing` 布林變數提供的。

As a final note;  作為最後的補充；

```csharp
GC.SuppressFinalize(this);
```

prevents finalization because it is not needed as the client explicitly forces the release of resources.
它會防止完成化，因為它不是必要的，客戶端明確強制釋出資源。

Now, we can use _DatabaseConnection_ class in order to see how _Dispose pattern_ acts in different scenarios.
現在，我們可以使用 `DatabaseConnection` 類別來看看 `Dispose` 模式在不同情境下的作用。

First, we call the _Dispose_ method explicitly:
首先，我們明確呼叫 `Dispose` 方法：

```csharp
class Program
{
    static void Main(string[] args)
    {
        var connection = new DatabaseConnection();
        try
        {
            //Write your operational code here
        }
        finally
        {
            connection.Dispose();
        }
    }
}
```

Output:  輸出：

![[CSharpMemoryManagement02_03.webp]]

Another and commonly used method to call _Dispose_ is using **_using_** statement:
另一種常見的方法來呼叫 `Dispose` 是使用 `using` 語句：

```csharp
class Program
{
    static void Main(string[] args)
    {
        using (var connection = new DatabaseConnection())
        {
            //Write your operational code here
        }
    }
}
```

As you see there is no call to _Dispose_ method because the _using_ statement handles that automatically.
如您所見，沒有呼叫 `Dispose` 方法，因為 `using` 語句會自動處理這件事。

Both codes above generate the same output.
上述兩段程式碼都產生相同的輸出。

Next, let’s call _Dispose_ method more than once and see the output:
接下來，讓我們呼叫 `Dispose` 方法超過一次，看看輸出結果：

```csharp
class Program
{
    static void Main(string[] args)
    {
        var connection = new DatabaseConnection();
        try
        {
            //Write your operational code here
        }
        finally
        {
            connection.Dispose();
            connection.Dispose();
            connection.Dispose();
        }
    }
}
```

Output:  輸出：

![[CSharpMemoryManagement02_04.webp]]

Finally, let’s see what happens if we don’t call the _Dispose_ method explicitly:
最後，讓我們看看如果沒有明確呼叫 Dispose 方法會發生什麼：

```csharp
class Program
{
    static void Main(string[] args)
    {
        var connection = new DatabaseConnection();
        try
        {
            //Write your operational code here
        }
        finally
        {
            //connection.Dispose();
        }
    }
}
```

Output:  輸出：

![[CSharpMemoryManagement02_05.webp]]

As you see, _Dispose_ method is called implicitly during the finalization. So adding the _finalizer_ during the implementation of the _Dispose pattern_ becomes a safeguard to clean up the resources if the client does not call the _Dispose_ method.
如你所見，`Dispose` 方法在 finalization 時間被隱含地呼叫。因此，在實作 `Dispose` 模式時加上 finalizer，變成了一道保障，以防客戶端沒有呼叫 `Dispose` 方法而清理資源。

This is the end of part two. Hope you found it helpful and easy to understand. Please let me know if you have any corrections and/or questions in the comments.
這是第二部分的結尾。希望你覺得有幫助且容易理解。若有任何修正與/或問題，請在評論中告訴我。

**References  參考資料**

[_https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/_](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
[_https://docs.microsoft.com/tr-tr/dotnet/csharp/programming-guide/classes-and-structs/destructors_](https://docs.microsoft.com/tr-tr/dotnet/csharp/programming-guide/classes-and-structs/destructors)
[_https://docs.microsoft.com/tr-tr/dotnet/standard/garbage-collection/implementing-dispose_](https://docs.microsoft.com/tr-tr/dotnet/standard/garbage-collection/implementing-dispose)
[_https://www.monitis.com/blog/improving-net-application-performance-part-5-finalize-and-dispose/_](https://www.monitis.com/blog/improving-net-application-performance-part-5-finalize-and-dispose/)

---

# C# Memory Management — Part 3 (Garbage Collection) C# 記憶管理 — 第 3 部分 (垃圾收集)

I am writing this post as the last part of the C# Memory Management ([Part 1](https://medium.com/@sena.kilicarslan/c-memory-management-part-1-c03741c24e4b) & [Part 2](https://medium.com/@sena.kilicarslan/c-memory-management-part-2-finalizer-dispose-d3b3e43c08d1)) series. This post explains what the **_Garbage Collector_** does and how it works in the .NET environment. (_Most of the information presented here is cited from Microsoft documentation_)
我寫這篇文章作為 C# 記憶管理 (第 1 部 & 第 2 部) 系列的最後一部分。這篇文章解釋了垃圾收集器的作用以及它在 .NET 環境中的工作方式。(這裡提供的許多信息引自 Microsoft 文档)

First, let’s start with _Common Language Runtime_.
首先，讓我們從通用語言執行平台開始。

## Common Language Runtime  通用語言執行平台

The .NET Framework provides a run-time environment called the **_Common Language Runtime (CLR)_,** which runs the code and provides services that make the development process easier.
.NET 框架提供一個名為通用語言執行平台 (CLR) 的執行時環境，它執行代碼並提供使開發過程更簡易的服務。

The following illustration shows the relationship of the CLR and the class library to your apps and to the overall system.
下列圖示說明了 CLR 和類別庫與您的應用程式以及整體系統的關係。

![[CSharpMemoryManagement03_01.webp]]

Source : [https://docs.microsoft.com/tr-tr/dotnet/framework/get-started/overview](https://docs.microsoft.com/tr-tr/dotnet/framework/get-started/overview)

The components of the CLR are:
CLR 的組成部分包括：

![[CSharpMemoryManagement03_02.webp]]

Source : [https://www.slideshare.net/Thenmurugeshwari/architecture-of-net-framework](https://www.slideshare.net/Thenmurugeshwari/architecture-of-net-framework)

In the CLR, the _Garbage Collector_ serves as an automatic memory manager. C# and other languages on top of the CLR are garbage collected.
在 CLR 中，垃圾收集器 (Garbage Collector) 作為自動記憶體管理器。C# 與其他在 CLR 上運行的語言都是進行垃圾收集的。

## Garbage Collector  垃圾收集器

.NET’s **_Garbage Collector (GC)_** manages the allocation and release of memory for your application.
.NET 的垃圾收集器 (GC) 負責管理您的應用程式的記憶體分配與釋出。

GC provides the following benefits:
GC 提供以下好處：

- Enables you to develop your application without having to free memory.
  讓您能夠開發應用程式，而無需釋放記憶體。
- Allocates objects on the managed heap efficiently.
  有效率地在受管堆上配置物件。
- Reclaims objects that are no longer being used, clears their memory, and keeps the memory available for future allocations.
  回收不再使用的物件，清除其記憶體，並保留記憶體供未來配置使用。
- Provides memory safety by making sure that an object cannot use the content of another object.
  確保記憶體安全性，以確保一個物件無法使用另一個物件的內容。

All processes on the same computer share the same physical memory. Each process has its own, separate virtual address space. As an application developer, you work only with virtual address space and never manipulate physical memory directly. The GC allocates and frees virtual memory for you on the **_managed heap_**.
同一台電腦上的所有進程都共享相同的物理記憶體。每個進程都有自己的、獨立的虛擬位址空間。作為應用程式開發者，您只與虛擬位址空間合作，從不直接操作物理記憶體。GC 在受管理的堆疊上為您分配和釋放虛擬記憶體。

The heap can be considered as the accumulation of two heaps: the _large object heap_ and the _small object heap_. The **_large object heap_** contains very large objects that are 85,000 bytes and larger (_The objects on the large object heap are usually arrays)._
堆疊可以考慮為兩個堆疊的積累：大物件堆疊和小物件堆疊。大物件堆疊包含非常大的物件，它們的大小為 85,000 字節及以上（大物件堆疊上的物件通常是陣列）。

Garbage collection gets triggered when one of the following conditions is true:
當符合以下任一條件時，垃圾收集會被觸發：

- The system has low physical memory.
  系統物理記憶體低落。
- The memory that is used by allocated objects on the managed heap surpasses an acceptable threshold.
  由於在受管堆上配置的物件所使用的記憶體超過了可接受的閾值。
- The [GC.Collect](https://docs.microsoft.com/en-us/dotnet/api/system.gc.collect) method is called. _(In almost all cases, you do not have to call this method, because the garbage collector runs continuously. This method is primarily used for unique situations and testing.)_
  呼叫 `GC.Collect` 方法。(幾乎在所有情況下，您不需要呼叫這個方法，因為垃圾收集器會持續運行。這個方法主要用於獨特情況和測試。)

Before a garbage collection starts, all managed threads are suspended except for the thread that triggered the garbage collection.
在垃圾收集開始之前，除了觸發垃圾收集的執行緒之外，所有受管執行緒都會暫停。

![[CSharpMemoryManagement03_03.webp]]

Source: [https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)

### Generations  世代

In 1984 David Ungar came up with a [generational hypothesis](https://people.cs.umass.edu/~emery/classes/cmpsci691s-fall2004/papers/p157-ungar.pdf) which gave birth to the generational garbage collectors:
在 1984 年，David Ungar 提出了世代假說，從而誕生了世代垃圾收集器：

> Young objects die young. Therefore reclamation algorithm should not waste time on old objects.
> 年輕對象早逝。因此，回收演算法不應浪費時間在舊對象上。
>
> ==Copying survivors is cheaper than scanning corpses.==
> ==複製倖存者比掃描屍體便宜。==

The heap is organized into generations so it can handle long-lived and short-lived objects. There are three generations of objects on the heap:
堆疊記憶體被組織成世代，以便處理長生命週期和短生命週期的物件。堆疊記憶體上有三個物件的世代：

#### Generation 0

This is the youngest generation and contains _short-lived objects_. An example of a _short-lived object_ is a temporary variable. Garbage collection occurs most frequently in this generation.
這是最年輕的世代，包含短生命週期的物件。短生命週期的物件範例是暫時變數。垃圾收集最常發生在這個世代。

Newly allocated objects form a new generation of objects and are implicitly _Gen 0_ collections unless they are large objects, in which case they go on the _large object heap_ in a _Gen 2_ collection.
新分配的物件形成一個新的世代，並且除非它們是大型物件，否則它們會被隱含地進行世代 0 的收集，在這種情況下，它們會在世代 2 的收集中放置在大型物件堆疊記憶體上。

Most objects are reclaimed for garbage collection in _Gen 0_ and do not survive to the next generation. Objects that survive a _Gen 0_ garbage collection are promoted to _Gen 1_.
大多數物件會在 Gen 0 中被回收進行垃圾收集，並不會存活到下一世代。經過 Gen 0 垃圾收集後存活的物件會被提升到 Gen 1。

#### Generation 1

This generation contains _short-lived objects_ and acts as a buffer between _short-lived objects_ and _long-lived objects_. Objects that survive a _Gen 1_ garbage collection are promoted to _Gen 2_.
這個世代包含短命物件，並作為短命物件與長命物件之間的緩衝區。經過 Gen 1 垃圾收集後存活的物件會被提升到 Gen 2。

#### Generation 2

This generation contains _long-lived objects_. An example of a _long-lived object_ is an object in a server application that contains static data that lives for the duration of the process. Objects that survive a _Gen 2_ garbage collection remain in _Gen 2_.
這個世代包含長命物件。長命物件的例子是伺服器應用程式中包含靜態資料的物件，這些資料會在整個過程中存活。經過 Gen 2 垃圾收集後存活的物件會停留在 Gen 2。

Collecting a generation means collecting objects in that generation and all its younger generations. A _Gen 2_ garbage collection is also known as a **_full garbage collection_** because it reclaims all objects in all generations.
收集一個世代是指收集該世代及其所有較輕世代的物件。Gen 2 垃圾收集也稱為完整垃圾收集，因為它會回收所有世代中的所有物件。

### How Garbage Collector Works 垃圾收集器如何運作

A garbage collection has the following phases:
垃圾收集包含以下階段：

**_Marking_** _:_ Finds and creates a list of all live objects.
標記：尋找並建立所有活躍物件的清單。

**_Relocating_** _:_ Updates the references to the objects that will be compacted.
重新定位：更新將要壓縮的物件的參考。

**_Compacting_** _:_ Reclaims the space occupied by the dead objects and compacts the surviving objects. The compacting phase moves objects that have survived a garbage collection toward the older end of the segment. _(Ordinarily, the large object heap is not compacted, because copying large objects imposes a performance penalty.)_
壓縮：收回被死亡物件佔用的空間，並壓縮存活的物件。壓縮階段會將經歷過垃圾收集後存活的物件移動到區段較老的一端。（通常，大型物件堆不會被壓縮，因為複製大型物件會造成效能損耗。）

The GC uses the following information to determine whether objects are live:
垃圾收集器使用下列資訊來判斷物件是否存活：

- **_Stack roots_**_:_ Stack variables provided by the just-in-time (JIT) compiler and stack walker.
  堆疊根：由即時編譯器 (JIT) 編譯器及堆疊追蹤器提供的堆疊變數。
- **_Garbage collection handles_**_:_ Handles that point to managed objects and that can be allocated by user code or by the CLR.
  垃圾收集處理：指向受管物件的手柄，且可由用戶代碼或 CLR 分配。
- **_Static data_**_:_ Static objects in application domains that could be referencing other objects.
  靜態資料：應用程式域中的靜態物件，可能會參考其他物件。

![[CSharpMemoryManagement03_04.webp]]

Source: [https://www.csharpstar.com/interview-questions-garbage-collection-csharp/](https://www.csharpstar.com/interview-questions-garbage-collection-csharp/)

In addition, there are unmanaged resources like file streams, network/database connections that you need to take special care of. You need to use **_Finalizer & Dispose_** in this case. You can refer to [this post](https://medium.com/@sena.kilicarslan/c-memory-management-part-2-finalizer-dispose-d3b3e43c08d1) for further information.
此外，還有不受管資源，例如檔案串流、網路/資料庫連接，這些需要特別小心處理。這種情況下需要使用 Finalizer & Dispose。您可以參考這篇文章獲得更多資訊。

Lastly, I recommend you watch the below video which explains the topic visually with a very helpful demonstration at the end.
最後，我建議您觀看下方影片，影片以視覺方式解釋此主題，並在結尾有非常實用的示範。

![](https://www.youtube.com/watch?v=-JOkkn1ET8c)

I hope you found this post helpful and easy to understand. As I mentioned in the beginning, this was the last part of the **_C# Memory Management_** series. If you are interested, you can read the other posts ([Part 1](https://medium.com/@sena.kilicarslan/c-memory-management-part-1-c03741c24e4b) & [Part 2](https://medium.com/@sena.kilicarslan/c-memory-management-part-2-finalizer-dispose-d3b3e43c08d1)) as well.
我希望您覺得這篇文章對您有幫助且容易理解。正如我在開頭提到的，這是 C#記憶管理系列的最后一部分。如果您感興趣，您也可以閱讀其他文章（第一部分和第二部分）。

THE END :)  結束囉 :)

_References  參考資料

[_https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/_](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
[_https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals_](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)
[_https://chodounsky.net/2017/05/03/garbage-collection-in-c-sharp/_](https://chodounsky.net/2017/05/03/garbage-collection-in-c-sharp/)
