---
aliases:
date:
update:
author: 精致码农
language:
sourceurl:
tags:
  - CSharp
  - 非同步程式設計
---

# 为什么要小心使用 Task.Run 原创

精致码农 2021-09-14 15:26:33
https://blog.51cto.com/u_12495983/3880472

昨天在博客园有园友问了我一个问题，是这样的：
![[小心使用Task.Run_圓友問題.webp]]

先是半个月前 @碧水青荷 童鞋的一句话“大家都说不要随便 `Task.Run(()=>{})`

这样写”，当时没有想太多，这句话并没有引起我注意，只顾着回答他“不想在代码中加 `async`/`await` 该怎么做”的问题。

然后这句话被 @裤兜 童鞋注意到，昨天问了我为什么。我当时也很纳闷，`Task.Run` 在并行场景中很常见啊，为什么大家会有不要随便使用的说法。很遗憾，当时我脑海里认为这种说法只是空穴来风，并没有细究。

我有个习惯，就是下班路上在地铁上快速复盘一下今天发生的事情。当时这个问题刚好就在脑海里闪现了一下，“为什么大家都说不要随便使用 `Task.Run`”。突然想起了多年前的一个晚上……哦，难道是“Ta”？

对，应该就是它，**内存泄露**，除了这个原因我再也想不到其它原因了。因为我隐约记得多年前我确实踩过一次这个坑，也可能是两次。

没错，`Task.Run` 使用不当，一不留意就会有内存泄露的问题。

我们先来看一段代码：

```csharp
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;

    public MyClass(Logger<MyClass> logger)
    {
        _logger = logger;
    }

    public Task Foo(Logger<MyClass> logger)
    {
        return Task.Run(() =>
        {
            _logger.LogInformation($"Executing job with ID {_id}");
            // do sth.
        });
    }
}
```

在这段代码中，私有成员 `_id` 被 `Task.Run` 的匿名方法捕获使用，进而导致 `MyClass` 实例被引用。当外部使用完 `MyClass` 实例时，本该由 GC 回收的时候却发现它还被其它资源引用着，所以 GC 认为该实例不应用被回收，也就永远失去了被回收的机会。

道理很简单，我就不再用示例演示了。解决办法也很简单，想必很多人都知道，就是使用本地变量。

```csharp
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;

    public MyClass(Logger<MyClass> logger)
    {
        _logger = logger;
    }

    public Task Foo(Logger<MyClass> logger)
    {
        var localId = _id;
        return Task.Run(() =>
        {
            _logger.LogInformation($"Executing job with ID {localId}");
            // do sth.
        });
    }
}
```

通过将值分配给一个本地变量，类就没有成员被捕获，即避免了潜在的内存泄漏。

内存泄漏问题在 `Task.Run` 身上发生很常见，容易被大家记住，容易提高警觉。其实不光是 `Task.Run`，其它地方使用了匿名方法也同样要小心，比如这个示例：

```csharp
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;
    private JobQueue _jobQueue;

    public MyClass(Logger<MyClass> logger, JobQueue jobQueue)
    {
        _logger = logger;
        _jobQueue = jobQueue;
    }

    public void Foo()
    {
        _jobQueue.EnqueueJob(() =>
        {
             _logger.LogInformation($"Executing job with ID {_id}");
            // do sth.
        });
    }
}
```

也有内存泄漏的问题。

总之，==任何使用**匿名方法**的地方都要**避免捕获类的成员**，小心内存泄漏。==

---

# 小心使用 Task.Run 续篇 原创

精致码农 2021-09-14 15:24:45
https://blog.51cto.com/u_12495983/3880447

关于前两天发布的文章：文中演示的示例到底会不会导致内存泄露，给很多人带来了疑惑。这点我必须向大家道歉，是我对导致内存泄漏的原因没描述和解释清楚，也没用实际的示例证实，是我的错。

但是，文中示例演示的 `Task.Run` 捕获类成员的情况，确实会有内存泄漏的风险，我将在本文演示给大家看。

如果一个对象（或数据）不需要再使用了，但依然还一直占据内存空间，则视为内存泄漏。这一点大家观点是一致的吧，那如何来检测对象有没有被回收呢？

我们知道，在 C# 中，实例对象被释放回收，必然会执行析构函数。所以我们可以对一个类重写其析构函数，如果该类的实例对象使用完后，强制执行 GC 回收，其析构函数依然不被执行，则说明 GC 没有回收该对象。若 GC 后面一直不回收这个对象，则说明存在内存泄漏。

手动强制执行 GC 回收的代码如下：

```csharp
GC.Collect();
GC.WaitForPendingFinalizers();
GC.Collect();
```

这三句代码可以确保 GC 把所有能搜索到的可回收对象清理干净。注意：不推荐在生产环境这样写。

我们还是用 为什么要小心使用 Task.Run 这篇文章用到的示例，只是为了测试稍加修改了一下：

```csharp
class Program
{
    static void Main(string[] args)
    {
        Test();

        // 对不需要再使用的资源强制回收
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();

        // 程序保活
        while (true)
        {
            Thread.Sleep(100);
        }
    }

    static void Test()
    {
        var myClass = new MyClass();
        myClass.Foo();
        // 到这，myClass对象不需要再使用了
    }
}

public class MyClass
{
    private int _id;
    private List<string> _list;

    public Task Foo()
    {
        return Task.Run(() =>
        {
            Console.WriteLine($"Task.Run is executing with ID {_id}");
            Thread.Sleep(100); // 模板耗时操作
        });
    }

    ~MyClass()
    {
        Console.WriteLine("MyClass instance has been colleted.");
    }
}
```

我们在 `myClass` 对象使用完后，手动强制执行 GC 回收，运行结果如下：
![[小心使用Task.Run_result01.webp]]

我们看到 `MyClass` 的析构函数一直没有执行，也就意味着它的实例一直没有被回收。

现在我们修改 `MyClass` 类的 `Foo` 方法，改用本地（局部）变量试一试：

```csharp
...
public Task Foo()
{
    var localId = _id;
    return Task.Run(() =>
    {
        Console.WriteLine($"Task.Run is executing with ID {localId}");
    });
}
...
```

再运行看看效果：
![[小心使用Task.Run_result02.webp]]

这次我们可以看到，`MyClass` 的析构函数执行了，说明实例对象被回收了。

前后唯一区别是，前者在 `Task.Run` 的匿名方法中捕获了类的成员，而后者使用了本地变量。前者出现了内存泄漏，后者避免了内存泄漏。

所以，在 `Task.Run` 的匿名方法中捕获类的成员，确实有可能导致内存泄漏（注意是有可能而不是一定）。

那背后的原因是什么呢？我在上一篇文章是这样解释的：

> 私有成员 `_id` 被 `Task.Run` 的匿名方法捕获使用，进而导致 `MyClass` 实例被引用。当外部使用完 `MyClass` 实例时，本该由 GC 回收的时候却发现它还被其它资源引用着，所以 GC 认为该实例不应该被回收，也就可能永远失去了被回收的机会。

这个解释有很大的问题，至少给广大读者带来了两大疑惑：

1. 由于值类型是拷贝的方式赋值，所以捕获的本地变量和类成员指向的是各自的值，对本地变量的捕获不会影响到整个类。但如果把 `_id` 改为引用类型（如 `String`），那两者指向的就是同一个对象值，那是不是意味着即便使用本地变量也还是无法避免内存泄漏的问题？
2. GC 第一次回收时发现 `myClass` 实例存在被捕获的成员，则认为它不应该被回收。那当 `Task.Run` 执行完后， 被捕获的成员也使用完了，GC 再次搜索时不就可以回收 `myClass` 对象吗？只是晚了一些时间回收而已嘛。

感谢善于思考提出疑惑的读者们，为你们点赞。

这两大疑惑该如何解释？后半部分我还没写完，大家可以先思考一下，我将在下一篇给大家解惑，望大家见谅。当然，我的解释也不一定会是对的，希望大家带着怀疑的态度和批判性思维来看我的文章，也请大家分享自己的理解和观点。

---

# 小心使用 Task.Run 终篇解惑 原创

精致码农 2021-09-14 15:22:58
https://blog.51cto.com/u_12495983/3880349

继 上一篇文章之后，这篇文章主要解答以下两个疑惑：(參考上方)

为了方便理解，我再把昨天的关键代码贴出来：

```csharp
public class MyClass
{
    private int _id;
    private List<string> _list;

    public Task Foo()
    {
        var localId = _id;
        return Task.Run(() =>
        {
            Console.WriteLine($"Task.Run is executing with ID {localId}");
            Thread.Sleep(100); // 模拟耗时操作
        });
    }
}
```

先来看第一个疑惑。经实测，把 `_id` 改为 `String` 类型运行结果是和 `int` 一样的，说明和值类型或引用类型无关。我的理解是这样的：

我们知道，引用类型的变量在声明的时候就会在栈中分配一个空间，用来存放地址引用，而给它的赋值则存储在托管堆中。虽然本地变量 `localId` 和类的成员 `_id` 的地址都指向的是托管堆中同一块空间，但他们在栈中的地址却分属不同的作用域。==所谓被捕获就是被作用域捕获==，当一个作用域结束时，该作用域内的成员的地址空间都会随着一起被释放。至于==地址指向的托管堆中的字符串值，则不是作用域关心的事情==。当该字符串值所在的空间没有地址指向它时，就会被 GC 回收。有点抽象，但应该还好理解。

再来看第二个疑惑。在此之前，我们先来了解一下 GC 的分代算法。

当 CLR 试图搜索不再使用的对象的时，它需要遍历托管堆上的对象。随着程序的持续运行，托管堆可能越来越大，如果要对整个托管堆进行垃圾回收，势必会严重影响性能。所以，为了优化这个过程，CLR 中使用了分代算法。

[記憶體回收的基本概念](https://learn.microsoft.com/zh-tw/dotnet/standard/garbage-collection/fundamentals)

简单来说，分代算法就是把内存中的资源划分为三代：Gen 0、Gen 1、Gen 2，它们被 GC 遍历的频率依次从高到低。所有新创建的对象属于 Gen 0，GC 扫描它的频率最高。进行一次扫描后，处于 Gen 0 的不可回收对象就会被标记为 Gen 1。类似的，GC 扫描 Gen 1 时，如果 Gen 1 的对象依然不可回收，就会标记为 Gen 2。有点像马太效应，资源停留在内存时间越长，就越不容易被回收。
![[小心使用Task.Run_分代算法.webp]]

Gen 2 的回收被称为 Full GC。而 Full GC 只有在满足一定的条件才会执行，具体请阅读这篇官方文档：

[完整垃圾回收](https://learn.microsoft.com/zh-tw/dotnet/standard/garbage-collection/notifications#full-garbage-collection)

也就是说，进入 Gen 2 的资源，若条件没有达到，就会一直不被回收。

理解了分代算法和 Full GC，第二个疑惑就迎刃而解了。第二个疑惑关键在三个时间点上：

1. `myClass` 对象作用域结束的时间点
2. GC 执行回收的时间点
3. `Task.Run` 匿名方法执行完成的时间点

如果程序执行的时间点顺序是：1、3、2，那么不会有内存漏泄的问题，这点很容易理解。

由于实际情况 `Task.Run` 一般为耗时操作（非耗时任务一般没有必要使用 `Task.Run`），所以时间点的顺序极有可能是：1、2、3。如果是此执行的顺序，那么 GC 在回收时就会因为 `myClass` 对象存在成员被引用而把它标记为 Gen 1。如果 `Task.Run` 耗时足够长，`myClass` 就可能会进入 Gen 2，进而可能很难被回收，甚至可能永远不被回收。

其实大部分场景，我们也不必过于小心，即使在 `Task.Run` 匿名方法捕获了类的成员使该类的实例进入了 Gen 2，Gen 2 中留存的不再使用的资源也是有限的。根据官方文档对 Full GC 的介绍（地址在前文），当 Gen 2 积累到一定的量时便满足了执行回收的条件，在 GC 下一次回收时便会回收 Gen 2 中不再使用的资源。当然，作为一个优秀的程序员，我们还是得养成好的编码习惯，不要在
`Task.Run` 中的匿名方法捕获类的成员。

最后，郑重声明，最近三篇关于小心使用 `Task.Run` 的文章皆属我个人理解，知识水平有限，难免存在遗漏和错误。若有发现，请大家不吝指正。
