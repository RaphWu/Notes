---
aliases:
date: 2011-01-21
update:
author: Nicholas Blumhardt
language: C#
sourceurl: https://nblumhardt.com/2011/01/an-autofac-lifetime-primer/
tags:
  - CSharp
  - IoC
  - Autofac
---

# An Autofac Lifetime Primer 《Autofac 生命周期入門》

_Or, “Avoiding Memory Leaks in Managed Composition”
或，「避免在受控組合中出現記憶體洩漏」_

Understanding lifetime can be pretty tough when you’re new to IoC. Even long-time users express vague fears and misgivings when it comes to this subject, and disconcerting issues – components not being disposed, steadily climbing memory usage or an `OutOfMemoryException` – have bitten many of us at [one time](http://matthamilton.net/autofac-and-transient-objects) or [another](http://davybrion.com/blog/2010/02/avoiding-memory-leaks-with-nservicebus-and-your-own-castle-windsor-instance/).
對於剛接觸 IoC 的人來說，理解生命週期可能相當困難。即使是長期使用者，在談到這個主題時也會表達模糊的擔憂和疑慮，而令人困擾的問題——組件未被正確釋放、記憶體使用量穩定上升或 `OutOfMemoryException` ——也曾在某個時候讓許多人遭遇過。

Avoiding lifetime issues when using an IoC container is generally straightforward, but doing so successfully is more a question of application design rather than just container API usage. There’s lots of good advice out there, but very little of it tells the complete story from beginning to end. I’ve attempted to do that with this rather long article, and hope that with a bit of feedback from you it can be shaped into a useful reference.
使用 IoC 容器時避免生命週期問題通常很直觀，但成功實現則更多是應用程式設計的問題，而非僅僅是容器 API 的使用。網路上有很多好的建議，但很少有能完整從頭到尾說明的。我嘗試在這篇相當長的文章中做到這一點，並希望透過您的回饋，能將其塑造成一個有用的參考資料。

This article is about Autofac, but the broad issues are universal – even if you’re not an Autofac user, chances are there’s something to learn about your container of choice.
本文是關於 Autofac 的，但廣泛的問題是普遍的——即使您不是 Autofac 的使用者，很可能您的容器選擇也有值得學習的地方。

## What Leaks?  什麼會洩漏？

Let’s begin with the issue that in all likelihood brought you here. _Tracking_ containers like Autofac hold references to the disposable components they create.
讓我們從最可能讓您到這裡的問題開始。像 Autofac 這樣的容器追蹤會持有它們所創建的可處理組件的參考。

By _disposable_ we mean any component that implements the BCL’s `IDisposable` interface (or is configured with an `OnRelease()` [action](http://code.google.com/p/autofac/wiki/OnActivatingActivated)):
所謂可釋出，是指任何實現了 BCL 的 `IDisposable` 接口（或配置了 `OnRelease()` 動作）的元件：

```csharp
interface IMyResource { … }

class MyResource : IMyResource, IDisposable { … }
```

When an instance of `MyResource` is created in an Autofac container, the container will hold a reference to it even when it is no longer being used by the rest of the program. This means that the garbage collector will be unable to reclaim the memory held by that component, so the following program will eventually exhaust all available memory and crash:
當 `MyResource` 的實例在 Autofac 容器中建立時，==即使它不再被程式的其他部分使用，容器也會持有對它的參考。==這意味著垃圾收集器將無法回收該元件所持有的記憶體，因此以下程式最終會耗盡所有可用記憶體並崩溃：

```csharp
var builder = new ContainerBuilder();
builder.RegisterType<MyResource>().As<IMyResource>();
using (var container = builder.Build())
{
    while (true)
        var r = container.Resolve<IMyResource>(); // Out of memory!
}
```

This is a far cry from typical CLR behaviour, which is one more reason why it is good to get away from thinking about an IoC container as a replacement for `new`. If we’d just created `MyResource` directly in the loop, there wouldn’t be any problem at all:
這與典型的 CLR 行為相去甚遠，這也是為什麼==我們應該避免將 IoC 容器視為 `new` 的替代品==的一個原因。如果我們直接在迴圈中直接建立 `MyResource` ，根本不會有任何問題：

```csharp
while (true)
    var r = new MyResource(); // Fine, feed the GC
```

### Transitive Dependencies  傳遞性依賴

Looking at the code above you might be tempted to think that the problem only surfaces when disposable components are resolved directly from the container. That’s not really the case - _every_ disposable component created by the container is tracked, even those created indirectly to satisfy dependencies.
看上面的程式碼，你可能會誤以為只有在直接從容器解析可拋棄元件時才會出現問題。事實上並非如此 - ==容器所創建的每一個可拋棄元件都會被追蹤，即使那些為了滿足依賴而間接創建的元件也是如此。==

```csharp
interface IMyService { }

class MyComponent : IMyService
{
    // Dependency on a service provided by a disposable component
    public MyComponent(IMyResource resource) { … }
}
```

If a second component is resolved that depends on a service provided by a disposable component, the memory leak still occurs:
如果解析第二個依賴於可拋棄元件所提供服務的元件，記憶體洩漏仍然會發生：

```csharp
while (true)
    // Still holds instances of MyResource
    var s = container.Resolve<IMyService>();
```

### Results Returned from Delegate Factories 來自委派工廠的結果

Rather than calling `Resolve()` directly on an `IContainer` we might instead take a dependency on an Autofac delegate factory type like `Func<T>`:
與其在 `IContainer` 上直接呼叫 `Resolve()` ，我們或許可以相依於 Autofac 委派工廠類型，例如 `Func<T>` ：

```csharp
interface IMyService2
{
    void Go();
}

class MyComponent2 : IMyService2
{
    Func _resourceFactory;

    public MyComponent(Func<IMyResource> resourceFactory)
    {
        _resourceFactory = resourceFactory;
    }

    public void Go()
    {
        while (true)
            var r = _resourceFactory(); // Still out of memory.
    }
}
```

Now in the main loop we only resolve one component instance and call the `Go()` method.
現在在主迴圈中，我們只解析一個元件實例並呼叫 `Go()` 方法。

```csharp
using (var container = builder.Build())
{
    var s = container.Resolve<IMyService2>();
    s.Go();
}
```

Even though we’ve only called the container once directly, the leak is still there.
即使我們僅僅直接調用過容器一次，泄漏仍然存在。

## Why does Autofac behave this way? 為什麼 Autofac 會這樣行為？

It might seem that there are several traps here, though there’s really only one – and it is worth reiterating:
儘管看起來這裡有幾個陷阱，但實際上只有一個——值得重申：

**Autofac will track every disposable component instance that it creates, no matter how that instance is requested.**
**Autofac 會追蹤它所創建的每一個可處理元件實例，無論該實例是如何請求的。**

This isn’t, of course, the end of the road. Autofac is very carefully designed to make resource management easier than programming without a container. Notice I slipped in the word _resource_ there? The ‘memory leaks’ we’re talking about are a result of preventing another kind of ‘leak’ – the resources that are managed through `IDisposable`.
這當然不是終點。Autofac 的設計非常謹慎，讓資源管理比沒有容器進行程式設計更容易。注意到我剛才滑進了「資源」這個詞嗎？我們所說的「洩漏」是因為防止另一種「洩漏」——透過 `IDisposable` 管理的資源。

### What are Resources?  資源是什麼？

Traditionally, a _resource_ might be defined as anything with finite capacity that must be shared between its users.
傳統上，*資源*可能被定義為任何具有有限容量且必須在用戶之間共享的東西。

In the context of components in an IoC container, a resource is anything with _acquisition_ and _release_ phases in its lifecycle. Many low-level examples exist, such as components that rely on items from the list below, but it is also very common to find high-level application constructs with similar semantics.
在 IoC 容器中組件的語境下，資源是任何在其生命週期中有*獲取*和*釋放*階段的東西。存在許多低層級的例子，例如依賴以下列表中項目的組件，但它也很常見找到具有相似語義的高層級應用構造。

- Locks (e.g. `Monitor.Enter()` and `Monitor.Exit()`)
    鎖 (例如 `Monitor.Enter()` 和 `Monitor.Exit()` )
- Transactions (`Begin()` and `Commit()`/`Abort()`)
    交易 ( `Begin()` 和 `Commit()` / `Abort()` )
- Event subscriptions (`+=` and `-=`)
    事件訂閱 ( `+=` 和 `-=` )
- Timers (`Start()` and `Dispose()`)
    計時器 ( `Start()` 和 `Dispose()` )
- Machine resources like sockets and files (usually `Open()`/`Create()` and `Close()`)
    機器資源，如插座和檔案（通常為 `Open()` / `Create()` 和 `Close()` ）
- Waiting worker threads (`Create()` and `Signal()`)
    等待工作線程（ `Create()` 和 `Signal()` ）

In .NET there’s a standard way to represent resource semantics on a component by implementing `IDisposable`. When such a component is no longer required, the `Dispose()` method must be called to complete the ‘release’ phase.
在 .NET 中，有一種標準方法來表示元件上的資源語義，透過實作 `IDisposable` 來完成。當這樣的元件不再需要時，必須呼叫 `Dispose()` 方法來完成「釋出」階段。

### Not Calling Dispose() is most often a Bug 不呼叫 Dispose() 通常是一個 Bug

You can read some interesting discussions via [Kim Hamilton’s](http://blogs.msdn.com/b/kimhamil/archive/2008/11/05/when-to-call-dispose.aspx) and [Joe Duffy’s](http://www.bluebytesoftware.com/blog/PermaLink.aspx?guid=88e62cdf-5919-4ac7-bc33-20c06ae539ae) articles on the topic of when `Dispose()` must be called.
您可以透過 [Kim Hamilton](http://blogs.msdn.com/b/kimhamil/archive/2008/11/05/when-to-call-dispose.aspx) 和 [Joe Duffy 的](http://www.bluebytesoftware.com/blog/PermaLink.aspx?guid=88e62cdf-5919-4ac7-bc33-20c06ae539ae) 文章閱讀有關何時必須調用 `Dispose()` 主題的一些有趣的討論。

There’s a fairly strong consensus that more often than not, failing to call `Dispose()` will lead to a bug, regardless of whether or not the resource in question is protected by a finalizer (or `SafeHandle`).
人們普遍認為，無論相關資源是否受到終結器（或 `SafeHandle` ）的保護，未能呼叫 `Dispose()` 都會導致錯誤。

What is less clear is how applications and APIs should be structured so that `Dispose()` can be called reliably and at the correct time.
不太清楚的是，應用程式和 API 應該如何構建，以便能夠可靠地在正確的時間呼叫 `Dispose()` 。

#### IDisposable and Ownership IDisposable 和所有權

Before IoC (assuming you use it now) there were probably two approaches you could apply to calling `Dispose()`:
在 IoC 之前（假設您現在使用它），可能有兩種方法來呼叫 `Dispose()` ：

1. The C# `using` statement
2. Ad-hoc  特別指定

`IDisposable` and `using` are a match made in heaven, but they only apply when a resource’s lifetime is within a single method call.
`IDisposable` 和 `using` 是天作之合，但它們僅適用於資源的生命週期在單一方法呼叫內的情況。

For everything else, you need to find a strategy to ensure resources are disposed when they’re no longer required. The most widely-attempted one is based around the idea that **whatever object _acquires_ the resource should also _release_ it.** I pejoratively call it “ad-hoc” because it doesn’t work consistently. Eventually you’ll come up against one (and likely more) of the following issues:
對於其他所有情況，你需要找到一種策略來確保資源在不再需要時被釋放。最廣泛嘗試的策略是基於這樣一種理念：**任何物件 * 獲取 * 資源時，也應該 * 釋放 * 它。** 我貶義地稱之為「特別的」策略，因為它的工作原理不一致。最終，你會遇到以下一個（甚至多個）問題：

**Sharing**: When multiple independent components share a resource, it is very hard to figure out when none of them requires it any more. Either a third party will have to know about all of the potential users of the resource, or the users will have to collaborate. Either way, things get hard fast.
**共用** ：當多個獨立元件共用一個資源時，很難確定它們何時不再需要該資源。要么第三方必須了解該資源的所有潛在用戶，要么用戶必須協作。無論哪種方式，事情都會很快變得棘手。

**Cascading Changes**: Let’s say we have three components – `A` uses `B` which uses `C`. If no resources are involved, then no thought needs to be given to how resource ownership or release works. But, if the application changes so that `C` must now own a disposable resource, then both `A` and `B` will probably have to change to signal appropriately (via disposal) when that resource is no longer needed. The more components involved, the nastier this one is to unravel.
**級聯變更** ：假設我們有三個元件－A 使用 `B` ，而 `B` 又使用 `C` 如果不涉及任何資源，則無需考慮資源所有權或釋放機制。但是，如果應用程式發生變更，導致 `C` 現在必須擁有一個可釋放的資源，那麼 `A` 和 `B` 可能都必須進行相應更改，以便在不再需要該資源時（透過處置）發出適當的訊號。涉及的組件越多，這個問題就越難解決。

**Contract vs. Implementation**: In .NET we’re encouraged to program to a contract, not an implementation. Well-designed APIs don’t usually give details of the implementation type, for example an Abstract Factory could be employed to create caches of different sizes:
**契約 vs. 實作** ：.NET 鼓勵我們根據契約進行編程，而不是根據實作。設計良好的 API 通常不會提供實作類型的細節，例如，可以使用抽象工廠來創建不同大小的快取：

```csharp
public ICache CreateCache(int maximumByteSize);  // Abstract Factory

interface ICache // : IDisposable?
{
    void Add(object key, object value, TimeSpan ttl);
    bool TryLookup(object key, out object value);
} 
```

The initial implementation may return only `MemoryCache` objects, that have no resource management requirements. However, because we may in future create a `FileCache` implementation, does that mean that `ICache` should be disposable?
初始實作可能只會傳回 `MemoryCache` 對象，這些物件不需要資源管理。但是，由於我們將來可能會建立一個 `FileCache` 實現，這是否意味著 `ICache` 應該是一次性的？

The root of the problem here is that for _any_ contract, it is conceivable that there will one day be an implementation that is disposable.
問題的根源在於，對於 *任何* 合約來說，可以想像有一天會有一個可丟棄的實現。

This particular problem is exacerbated by loosely-coupled systems like those built with IoC. Since components only know about each other through contracts (services) and never their implementation types, there really isn’t any way for one component to determine whether it should try to dispose another.
像使用 IoC 建構的鬆散耦合系統會加劇這個問題。由於元件之間只能透過契約（服務）相互了解，而無法了解它們的實作類型，因此一個元件實際上無法確定是否應該嘗試釋放另一個元件。

#### IoC to the Rescue IoC 來救援

There is a viable solution out there. As you can guess, a) **in typical enterprise and web applications** and b) **at a certain level of granularity** IoC containers provide a good solution to the resource management problem.
確實存在一個可行的解決方案。正如你所猜測的，a) **在典型的企業和 Web 應用程式中** ；b) **在一定的粒度層級上，** IoC 容器為資源管理問題提供了一個很好的解決方案。

To do this, they need to take ownership of the disposable components that they create.
為了做到這一點，他們需要對自己所創造的一次性組件負責。

But this is only part of the story – they also need to be told about the units of work that the application performs. This is how the container will know to dispose components and release references, avoiding the memory leak problems that will otherwise occur.
但這只是故事的一部分——還需要告知它們應用程式執行的工作單元。這樣容器才能知道如何處理元件並釋放引用，從而避免可能發生的記憶體洩漏問題。

**Why not have the container use WeakReference just to be safe?** A solution that relies on `WeakReference` is afflicted by the same problems as not calling `Dispose()` at all. Allowing components to be garbage collected before the enclosing unit of work is complete can lead to a whole class of subtle loadand environment-dependent bugs. Many higher-level resources are also unable to be properly released within a finalizer.
**為什麼不讓容器使用弱引用來確保安全呢？** 依賴 `WeakReference` 的解決方案與根本不呼叫 `Dispose()` 有相同的問題。允許組件在封閉工作單元完成之前被垃圾回收，可能會導致一系列與負載和環境相關的細微錯誤。許多高級資源也無法在終結器中正確釋放。

### Units of Work  工作單元

As they run, enterprise and web applications tend to perform tasks with a defined beginning and end. These tasks might be things like responding to a web request, handling an incoming message, or running a batch process over some data.
企業和 Web 應用程式在運行時往往會執行一些有明確開始和結束的任務。這些任務可能包括回應 Web 請求、處理傳入訊息或對某些資料執行批次處理。

These are tasks in the abstract sense – not to be confused with something like `Task` or any of the asynchronous programming constructs. To make this clearer, we’ll co-opt the term ‘unit of work’ to describe this kind of task.
這些是抽象意義上的任務——不要與 `Task` 或任何非同步程式設計結構混淆。為了更清晰地理解，我們將使用「工作單元」一詞來描述這類任務。

Determining where units of work begin and end is the key to using Autofac effectively in an application.
確定工作單元的開始和結束位置是應用程式中有效使用 Autofac 的關鍵。

### Lifetime Scopes: Implementing Units of Work with Autofac 生命週期範圍：使用 Autofac 實現工作單元

Autofac caters to units of work through _lifetime scopes_. A lifetime scope (`ILifetimeScope`) is just what it sounds like – a scope at the completion of which, the lifetime of a set of related components will end.
Autofac 透過 *生命週期作用域* 來滿足工作單元的需求。生命週期作用域（ `ILifetimeScope` ）顧名思義，就是當一組相關元件的生命週期結束時，該作用域的生命週期也就結束了。

Component instances that are resolved during the processing of a unit of work get associated with a lifetime scope. By tracking instantiation order and ensuring that dependencies can only be satisfied by components in the same or a longer-lived lifetime scope, Autofac can take responsibility for disposal when the lifetime scope ends.
在工作單元處理過程中解析的元件實例會與生命週期範圍關聯。透過追蹤實例化順序並確保依賴項只能由相同或更長生命週期範圍內的組件滿足，Autofac 可以在生命週期範圍結束時負責處理。

Going back to our original trivial example, the leak we observed can be eliminated by creating and disposing a new lifetime scope each time we go through the loop:
回到我們最初的簡單範例，我們觀察到的洩漏可以透過在每次循環時創建和處置新的生命週期範圍來消除：

```csharp
// var container = …
while (true)
{
    using (var lifetimeScope = container.BeginLifetimeScope())
    {
        var r = lifetimeScope.Resolve<IMyResource>();
        // r, all of its dependencies and any other components
        // created indirectly will be released here
    }
}
```

This is all it takes to avoid memory and resources leaks with Autofac.
這就是使用 Autofac 避免記憶體和資源洩漏所需的全部內容。

**Don’t resolve from the root container. Always resolve from and then release a lifetime scope.
不要從根容器解析。始終從生命週期範圍解析，然後釋放。**

Lifetime scopes are extremely flexible and can be applied to a huge range of scenarios. There are a few handy things to know that might not be immediately obvious.
生命週期作用域非常靈活，可以應用於各種各樣的場景。有一些實用的小技巧可能不太明顯，但需要注意。

#### Lifetime Scopes can Exist Simultaneously 生命週期作用域可以同時存在

Many lifetime scopes can exist simultaneously on the same container. This is how incoming HTTP requests are handled in the Autofac ASP.NET integration, for example. Each request causes the creation of a new lifetime scope for handling it; this gets disposed at the end of the request.
多個生命週期作用域可以同時存在於同一個容器中。例如，Autofac ASP.NET 整合就是這樣處理傳入的 HTTP 請求的。每個請求都會建立一個新的生命週期作用域來處理它；該生命週期作用域會在請求結束時被釋放。

![[Concurrent-Scopes.png]]

Note that we call the root of the lifetime scope hierarchy the ‘Application’ scope, because it will live for the duration of an application run.
請注意，我們將生命週期範圍層次結構的根稱為「應用程式」範圍，因為它將在應用程式運行期間存在。

#### Lifetime Scopes can be Nested 生命週期作用域可以嵌套

The container itself is the root lifetime scope in an Autofac application. When a lifetime scope is created from it, this sets up a parent-child relationship between the nested scope and the container.
容器本身是 Autofac 應用程式中的根生命週期作用域。當從其建立生命週期作用域時，這將在巢狀作用域和容器之間建立父子關係。

When dependencies are resolved, Autofac first attempts to satisfy the dependency with a component instance in the scope in which the request was made. In the diagram below, this means that the dependency on an NHibernate session is satisfied within the scope of the HTTP request and the session will therefore be disposed when the HTTP request scope ends.
當依賴關係解析完成後，Autofac 首先嘗試在請求發起的範圍內使用元件實例來滿足依賴關係。在下圖中，這意味著對 NHibernate 會話的依賴關係在 HTTP 請求的範圍內得到滿足，因此當 HTTP 請求範圍結束時，該會話將被釋放。

![[Dep-res-process.png]]

When the dependency resolution process hits a component that cannot be resolved in the current scope – for example, a component configured as a Singleton using `SingleInstance()`, Autofac looks to the parent scope to see if the dependency can be resolved there.
當依賴關係解析過程遇到目前範圍內無法解析的元件時 - 例如，使用 `SingleInstance()` 配置為單例的元件，Autofac 會查看父範圍以查看是否可以在那裡解析依賴關係。

![[Dep-res-process2.png]]

When the resolve operation has moved from a child scope to the parent, any further dependencies will be resolved in the parent scope. If the `SingleInstance()` component `Log` depends on `LogFile` then the log file dependency will be associated with the root scope, so that even when the current HTTP request scope ends, the file component will live on with the log that depends on it.
當解析操作從子作用域移至父作用域時，任何進一步的依賴項都會在父作用域中解析。如果 `SingleInstance()` 元件 `Log` 依賴 `LogFile` ，則日誌檔案依賴項將與根作用域關聯，因此即使目前 HTTP 請求作用域結束，檔案元件仍將與依賴它的日誌一起繼續存在。

Lifetime scopes can be nested to arbitrary depth:
生命週期範圍可以嵌套任意深度：

```csharp
var ls1 = container.BeginLifetimeScope();
var ls2 = ls1.BeginLifetimeScope();
// ls1 is a child of the container, ls2 is a child of ls1
```

#### Sharing is driven by Lifetime Scopes 共享由生命週期範圍驅動

At this point in our discussion of lifetime management (you thought we were going to talk about memory leaks but that’s just how I roped you in!) it is worth mentioning the relationship between lifetime scopes and how Autofac implements ‘sharing’.
在我們討論生命週期管理的這一點上（您以為我們將要討論記憶體洩漏，但這只是我吸引您的方式！）值得一提的是生命週期範圍之間的關係以及 Autofac 如何實現「共享」。

By default, every time an instance of a component is needed (either requested directly with the `Resolve()` method or as a dependency of another component) Autofac will create a new instance and, if it is disposable, attach it to the current lifetime scope. All IoC containers that I know of support something like this kind of ‘transient lifestyle’ or ‘instance per dependency.’
預設情況下，每次需要元件實例時（無論是直接使用 `Resolve()` 方法請求，還是作為其他元件的依賴項），Autofac 都會建立一個新實例，如果該實例是可釋放的，則將其附加到當前生命週期範圍。據我所知，所有 IoC 容器都支援這種「瞬態生命週期」或「每個依賴項一個實例」的機制。

Also universally supported is ‘singleton’ or ‘single instance’ sharing, in which the same instance is used to satisfy all requests. In Autofac, `SingleInstance()` sharing will associate an instance with the root node in the tree of active lifetime scopes, i.e. the container.
同樣普遍支援的是「單例」或「單一實例」共享，即使用同一個實例來滿足所有請求。在 Autofac 中， `SingleInstance()` 共用會將實例與活躍生命週期作用域樹中的根節點（即容器）關聯。

There are two other common sharing modes that are used with Autofac.
Autofac 也使用了另外兩種常見的共享模式。

#### Matching Scope Sharing  匹配範圍共享

When creating a scope, it is possible to give it a name in the form of a ‘tag’:
建立範圍時，可以以「標籤」的形式為其命名：

```csharp
// For each session concurrently managed by the application
var sessionScope = container.BeginLifetimeScope("session");

// For each message received in the session:
var messageScope = sessionScope.BeginLifetimeScope("message");
var dispatcher = messageScope.Resolve<IMessageDispatcher>();
dispatcher.Dispatch(…);
```

Tags name the levels in the lifetime scope hierarchy, and they can be used when registering components in order to determine how instances are shared.
標籤命名生命週期範圍層次中的級別，並且可以在註冊元件時使用它們來確定如何共用實例。

```csharp
builder.RegisterType<CredentialCache>()
    .InstancePerMatchingLifetimeScope("session");
```

In this scenario, when processing messages in a session, even though we always `Resolve()` dependencies from the `messageScope`s, all components will share a single `CredentialCache` at the session level.
在這種情況下，在會話中處理訊息時，即使我們始終從 `messageScope` 中 `Resolve()` 依賴項，所有元件仍將在會話層級共用單一 `CredentialCache` 。

#### Current Scope Sharing  目前範圍共享

While occasionally units of work are nested multiple layers deep, most of the time a single executable will deal with only one kind of unit of work.
雖然有時工作單元會嵌套多層，但大多數情況下，單一可執行檔只會處理一種工作單元。

In a web application, the ‘child’ scopes created from the root are for each HTTP request and might be tagged with `"httpRequest"`. In a back-end process, we may also have a two-level scope hierarchy, but the child scopes may be per-`"workItem"`.
在 Web 應用程式中，從根建立的「子」作用域針對每個 HTTP 請求，並可能帶有 `"httpRequest"` 標記。在後端程序中，我們可能也有兩級作用域層次結構，但子作用域可能是針對每個 `"workItem"` 。

Many kinds of components, for example those related to persistence or caching, need to be shared per unit of work, regardless of which application model the component is being used in. For these kinds of components, Autofac provides the `InstancePerLifetimeScope()` sharing model.
許多類型的元件，例如與持久性或快取相關的元件，需要在每個工作單元中共享，無論該元件在哪種應用程式模型中使用。對於這些類型的元件，Autofac 提供了 `InstancePerLifetimeScope()` 共享模型。

If a component is configured to use an `InstancePerLifetimeScope()` then at most a single instance of that component will be shared between all other components with the same lifetime, regardless of the level or tag associated with the scope they’re in. This is great for reusing components and configuration between different applications or services in the same solution.
如果將元件配置為使用 `InstancePerLifetimeScope()` ，則最多會在具有相同生命週期的所有其他元件之間共用該元件的一個實例，而不管它們所處範圍關聯的層級或標籤為何。這對於在同一解決方案中的不同應用程式或服務之間重複使用元件和配置非常有用。

#### Lifetime Scopes don’t have any Context Dependency 生命週期範圍沒有任何上下文依賴

It is completely acceptable to create multiple independent lifetime scopes on the same thread:
在同一個執行緒上創建多個獨立的生命週期範圍是完全可以接受的：

```csharp
var ls1 = container.BeginLifetimeScope();
var ls2 = container.BeginLifetimeScope();
// ls1 and ls2 are completely independent of each other
```

It is also perfectly safe to share a single lifetime scope between many threads, and to end a lifetime scope on a thread other than the one that began it.
在多個執行緒之間共享單一生命週期範圍，以及在啟動生命週期範圍的執行緒之外的執行緒上結束生命週期範圍也是非常安全的。

Lifetime scopes don’t rely on thread-local storage or global state like the HTTP context in order to do their work.
生命週期範圍不依賴執行緒本地儲存或 HTTP 上下文等全域狀態來完成其工作。

#### Components can Create Lifetime Scopes 元件可以建立生命週期作用域

The container can be used directly for creating lifetime scopes, but most of the time you won’t want to use a global container variable for this.
容器可以直接用於建立生命週期範圍，但大多數時候您不想為此使用全域容器變數。

All a component needs to do to create and use lifetime scopes is to take a dependency on the `ILifetimeScope` interface:
元件建立和使用生命週期範圍所需要做的就是依賴 `ILifetimeScope` 介面：

```csharp
class MessageDispatcher : IMessageDispatcher
{
    ILifetimeScope _lifetimeScope;

    public MessageDispatcher(ILifetimeScope lifetimeScope)
    {
        _lifetimeScope = lifetimeScope;
    }

    public void OnMessage(object message)
    {
        using (var messageScope = _lifetimeScope.BeginLifetimeScope())
        {
            var handler = messageScope.Resolve<IMessageHandler>();
            handler.Handle(message);
        }
    }
}
```

The `ILifetimeScope` instance that is injected will be the one in which the component ‘lives.’
注入的 `ILifetimeScope` 實例將是元件「生存」的實例。

### Owned Instances  自有實例

[Owned instances](http://code.google.com/p/autofac/wiki/OwnedInstances) (`Owned<T>`) are the last tale in Autofac’s lifetime story.
[自有實例](http://code.google.com/p/autofac/wiki/OwnedInstances) （ `Owned<T>` ）是 Autofac 一生故事中的最後一個故事。

An owned instance is a component that comes wrapped in its own lifetime scope. This makes it easier to keep track of how a component should be released, especially if it is used outside of a `using` statement.
自有實例是指被包裝在其自身生命週期範圍內的組件。這使得追蹤元件的釋放方式變得更加容易，尤其是在 `using` 語句之外使用元件時。

To get an owned instance providing service `T`, you can resolve one directly from the container:
若要取得提供服務 `T` 的自有實例，您可以直接從容器中解析一個：

```csharp
var ownedService = container.Resolve<Owned<IMyService>>();
```

In lifetime terms, this is equivalent to creating a new lifetime scope from the container, and resolving `IMyService` from that. The only difference is that the `IMyService` and the lifetime scope that holds it come wrapped up together in a single object.
從生命週期的角度來看，這相當於從容器中建立一個新的生命週期作用域，並從中解析 `IMyService` 。唯一的區別是， `IMyService` 和包含它的生命週期作用域被包裝在一個物件中。

The service implementation is available through the `Value` property:
服務實作可透過 `Value` 屬性取得：

```csharp
// Value is IMyService
ownedService.Value.DoSomething();
```

When the owned instance is no longer needed, it and all of its disposable dependencies can be released by disposing the `Owned<T>` object:
當不再需要所擁有的實例時，可以透過處置 `Owned<T>` 物件來釋放它及其所有可處置依賴項：

```csharp
ownedService.Dispose();
```

#### Combining with Func Factories 與函數工廠結合

Autofac [automatically provides](http://code.google.com/p/autofac/wiki/RelationshipTypes) `Func<T>` as a service when `T` is registered. As we saw in an earlier example, components can use this mechanism to create instances of other components on the fly.
當 `T` 註冊時，Autofac [會自動提供](http://code.google.com/p/autofac/wiki/RelationshipTypes) `Func<T>` 作為服務。正如我們在前面的範例中看到的，元件可以使用此機制動態建立其他元件的實例。

```csharp
class MyComponent2 : IMyService2
{
    Func<IMyResource> _resourceFactory;

    public MyComponent(Func<IMyResource> resourceFactory)
    {
        _resourceFactory = resourceFactory;
    }

    public void Go()
    {
        while (true)
            var r = _resourceFactory(); // Out of memory.
    }
}
```

Components that are returned from `Func<T>` delegates are associated with the _same lifetime scope_ as the delegate itself. If that component is itself contained in a lifetime scope, and it creates a finite number of instances through the delegate, then there’s no problem – everything will be cleaned up when the scope completes.
從 `Func<T>` 委託返回的組件與委託本身俱有 _ 相同的生命週期範圍 _ 。如果該元件本身包含在生命週期範圍內，並且它透過委託創建了有限數量的實例，那麼就沒有問題——當生命週期範圍完成時，所有內容都會被清除。

If the component calling the delegate is a long-lived one, then the instances returned from the delegate will need to be cleaned up eagerly. To do this, `Owned<T>` can be used as the return value of a factory delegate:
如果呼叫委託的元件是長生命週期元件，則需要立即清理委託傳回的實例。為此，可以使用 `Owned<T>` 作為工廠委託的返回值：

```csharp
class MyComponent2 : IMyService2
{
    Func<Owned<IMyResource>> _resourceFactory;

    public MyComponent(Func<Owned<IMyResource>> resourceFactory)
    {
        _resourceFactory = resourceFactory;
    }

    public void Go()
    {
        while (true)
            using (var r = _resourceFactory())
                // Use r.Value before disposing it 
    }
}
```

`Func<T>` and `Owned<T>` can be combined with other [relationship types](https://nblumhardt.com/2010/01/the-relationship-zoo/) to define relationships with a clearer intent than just taking a dependency on `ILifetimeScope`.
`Func<T>` 和 `Owned<T>` 可以與其他 [關係類型](https://nblumhardt.com/2010/01/the-relationship-zoo/) 結合使用，定義比僅依賴 `ILifetimeScope` 具有更清晰意圖的關係。

### Do I really have to think about all this stuff? 我真的需要考慮所有這些事情嗎？

That concludes our tour of lifetime management with Autofac. Whether you use an IoC container or not, and whether you use Autofac or another IoC container, non-trivial applications have to deal with the kinds of issues we’ve discussed.
至此，我們結束了 Autofac 生命週期管理的探索。無論你是否使用 IoC 容器，也無論你使用的是 Autofac 還是其他 IoC 容器，一些重要的應用程式都必須處理我們討論過的這些問題。

Once you’ve wrapped your mind around a few key concepts, you’ll find that lifetime rarely causes any headaches. On the plus side, a huge range of difficult application design problems around resource management and ownership will disappear.
一旦你了解幾個關鍵概念，你會發現生命週期幾乎不會帶來任何麻煩。好的一面是，圍繞資源管理和所有權的大量棘手的應用程式設計問題將會消失。

Just remember:  請記住：

1. Autofac **holds references** to all the disposable components it creates
    Autofac 對其創建的所有一次性組件**保存引用**
2. To prevent memory and resource leaks, always resolve components from **lifetime scopes** and dispose the scope at the conclusion of a task
    為了防止記憶體和資源洩漏，請始終從**生命週期範圍**解析組件，並在任務結束時處置範圍
3. Effectively using lifetime scopes requires that you clearly map out the **unit of work** structure of your applications
    有效地使用生命週期範圍需要你清楚地規劃出應用程式的**工作單元**結構
4. **Owned instances** provide an abstraction over lifetime scopes that can be cleaner to work with
    **自有實例**提供了生命週期範圍的抽象，可以更清晰地使用

Happy programming!  程式設計愉快！
