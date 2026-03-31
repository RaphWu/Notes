---
aliases:
date: 2019-03-20
update:
author: Leo_wl
language: CSharp
sourceurl: https://www.cnblogs.com/Leo_wl/p/10565012.html
tags:
  - CSharp
  - CSharp_Span/Memory
---

# 深入理解 C# 中的 IDisposable 接口

## 写在前面

在开始之前，我们需要明确什么是 C# (或者说 .NET ) 中的资源，打码的时候我们经常说释放资源，那么到底什么是资源，简单来讲， C# 中的每一种类型都是一种资源，而资源又分为托管资源和非托管资源，那这又是什么？！

托管资源：由 CLR 管理分配和释放的资源，也就是我们直接 `new` 出来的对象；

非托管资源：不受 CLR 控制的资源，也就是不属于 .NET 本身的功能，往往是通过调用跨平台程序集 (如 C++) 或者操作系统提供的一些接口，比如 Windows 内核对象、文件操作、数据库连接、socket、Win32API、网络等。

我们下文讨论的，主要也就是非托管资源的释放，而托管资源 .NET 的垃圾回收已经帮我们完成了。其实非托管资源有部分 .NET 的垃圾回收也帮我们实现了，那么如果要让 .NET 垃圾回收帮我们释放非托管资源，该如何去实现。

## 如何正确的显式释放资源

假设我们要使用 `FileStream`，我们通常的做法是将其 `using` 起来，或者是更老式的 `try`…`catch`…`finally`…这种做法，因为它的实现调用了非托管资源，所以我们必须用完之后要去显式释放它，如果不去释放它，那么可能就会造成内存泄漏。

这听上去貌似很简单，但我们编码的时候可能很多时候会忽略掉释放资源这个问题， .NET 的垃圾回收又如何帮我们释放非托管资源，接下来我们一探究竟吧，一个标准的释放非托管资源的类应该去实现 `IDisposable` 接口：

```csharp
public class MyClass:IDisposable
{
    /// <summary>执行与释放或重置非托管资源关联的应用程序定义的任务。</summary>
    public void Dispose()
    {
    }
}
```

我们实例化的时候就可以将这个类 using 起来：

```csharp
using(var mc = new MyClass())
{
}
```

看上去很简单嘛，但是，要是就这么简单的话，也没有这篇文章的必要了。如果要实现 `IDisposable` 接口，我们其实应该这样做：

1. 实现 `Dispose` 方法；
2. 提取一个受保护的 `Dispose` 虚方法，在该方法中实现具体的释放资源的逻辑；
3. 添加析构函数；
4. 添加一个私有的 `bool` 类型的字段，作为释放资源的标记

接下来，我们来实现这样的一个 Dispose 模式：

```csharp
public class MyClass : IDisposable
{
    /// <summary>
    /// 模拟一个非托管资源
    /// </summary>
    private IntPtr NativeResource { get; set; } = Marshal.AllocHGlobal(100);
    
    /// <summary>
    /// 模拟一个托管资源
    /// </summary>
    public Random ManagedResource { get; set; } = new Random();
    
    /// <summary>
    /// 释放标记
    /// </summary>
    private bool disposed;
    
    /// <summary>
    /// 为了防止忘记显式的调用Dispose方法
    /// </summary>
    ~MyClass()
    {
        //必须为false
        Dispose(false);
    }
    
    /// <summary>
    /// 执行与释放或重置非托管资源关联的应用程序定义的任务。
    /// </summary>
    public void Dispose()
    {
        //必须为true
        Dispose(true);
        //通知垃圾回收器不再调用终结器
        GC.SuppressFinalize(this);
    }
    
    /// <summary>
    /// 非必需的，只是为了更符合其他语言的规范，如C++、java
    /// </summary>
    public void Close()
    {
        Dispose();
    }
    
    /// <summary>
    /// 非密封类可重写的Dispose方法，方便子类继承时可重写
    /// </summary>
    /// <param name="disposing"></param>
    protected virtual void Dispose(bool disposing)
    {
        if (disposed)
        {
            return;
        }
        //清理托管资源
        if (disposing)
        {
            if (ManagedResource != null)
            {
                ManagedResource = null;
            }
        }
        //清理非托管资源
        if (NativeResource != IntPtr.Zero)
        {
            Marshal.FreeHGlobal(NativeResource);
            NativeResource = IntPtr.Zero;
        }
        //告诉自己已经被释放
        disposed = true;
    }
}
```

这里面每行代码都有它独自的含义，文章里不可能每句话都讲解透彻，为了突出重点，所以接下来就挑出几个重要的地方逐一解释咯，当然截止现在，我们只需要记住：

如果类型需要显式的释放资源，那么就一定要实现 `IDisposable` 接口。

实现 `IDisposable` 接口其实也是为了方便使用 `using` 这个语法糖，以方便编译器帮我们自动生成 `Dispose` 的 IL 代码：

```csharp
using(var mc = new MyClass())
{
}
```

就相当于：

```csharp
MyClass mc = null;
try
{
    mc = new MyClass();
}
finally
{
    if (mc != null)
    {
        mc.Dispose();
    }
}
```

如果要同时管理多个相同类型的对象：

```csharp
using(MyClass mc1 = new MyClass(), mc2 = new MyClass())
{
}
```

如果类型不一致：

```csharp
using(var client = new HttpClient())
{
    using(var stream = File.Create(""))
    {
    }
}
```

## 为什么需要析构方法？

在之前我们实现的更标准的 Dispose 模式中，我们注意到了，类里面包含了一个 `~` 开头的析构方法：

```csharp
~MyClass()
{
    //必须为false
    Dispose(false);
}
```

这个析构方法更规范的说法叫做终结器，它的意义在于，如果我们忘记了显式调用 `Dispose` 方法，垃圾回收器在扫描内存的时候，会作为释放资源的一种补救措施。

为什么加了析构方法就会有这种效果，我们知道在 `new` 对象的时候，CLR 会为对象创建一块内存空间，一旦对象不再被引用，就会被垃圾回收器回收掉，对于没有实现 `IDisposable` 接口的类来说，垃圾回收时将直接回收掉这片内存空间，而对于实现了 `IDisposable` 接口的类来说，由于析构方法的存在，在创建对象之初，CLR 会将该对象的一个指针放到终结器列表中，在 GC 回收内存之前，会首先将终结器列表中的指针放到一个 freachable 队列中，同时，CLR 还会分配专门的内存空间来读取 freachable 队列，并调用对象的终结器，只有在这个时候，对象才被真正的被标识为垃圾，在下一次垃圾回收的时候才回收这个对象所占用的内存空间。

> 筆記
> - 对于没有实现 `IDisposable` 接口的类：
>     1. 垃圾回收时将直接回收掉这片内存空间。
> - 对于实现了 `IDisposable` 接口的类，由于析构方法的存在：
>     1. 在创建对象之初，CLR 会将该对象的一个指针放到终结器列表中。
>     2. 在 GC 回收内存之前，会首先将终结器列表中的指针放到一个 freachable 队列中，
>     3. 同时，CLR 还会分配专门的内存空间来读取 freachable 队列，并调用对象的终结器，
> 只有在这个时候，对象才被真正的被标识为垃圾。
>     4. 在下一次垃圾回收的时候才回收这个对象所占用的内存空间。

那么，实现了 `IDisposable` 接口的对象在回收时要经过两次 GC 才能被真正的释放掉，因为 GC 要先安排 CLR 调用终结器，基于这个特点，如果我们显式调用了 `Dispose` 方法，那么 GC 就不会再进行第二次垃圾回收了，当然，如果忘记了 `Dispose`，也避免了忘记调用 `Dispose` 方法造成的内存泄漏。

提示：析构方法是在 C++ 中的一种说法，因为终结器和析构方法两者特点很像，为了沿袭 C++ 的叫法，称之为析构方法也没有什么不妥，但它们又不完全一致，所以微软后来又确定它叫终结器。

还有一点我们也注意到了，如果已经显式的调用了 `Dispose` 方法，那么隐式释放资源就再没必要运行了，GC 的 `SuppressFinalize` 方法就是通知 GC 的这一点：

```csharp
public void Dispose()
{
    //必须为true
    Dispose(true);
    //通知垃圾回收器不再调用终结器
    GC.SuppressFinalize(this);
}
```

所以在实现的 `Dispose` 方法中先调用我们正常的资源释放代码，再通知 GC 不要调用终结器了。

## 为什么需要提供一个 Dispose 虚方法？

我们注意到了，实现自 `Idisposable` 接口的 `Dispose` 方法并没有做实际的清理工作，而是调用了我们这个受保护的 `Dispose` 虚方法：

```csharp
protected virtual void Dispose(bool disposing)
{
    if (disposed)
    {
        return;
    }
    //清理托管资源
    if (disposing)
    {
        if (ManagedResource != null)
        {
            ManagedResource = null;
        }
    }
    //清理非托管资源
    if (NativeResource != IntPtr.Zero)
    {
        Marshal.FreeHGlobal(NativeResource);
        NativeResource = IntPtr.Zero;
    }
    //告诉自己已经被释放
    disposed = true;
}
```

之所以是虚方法，就是考虑到它如果被其他类继承时，子类也实现了 `Dispose` 模式，这个虚方法可以提醒子类，清理的时候要注意到父类的清理工作，即如果子类重新该方法，必须调用 `base.Dispose` 方法，假设现在我们有个子类，继承自 MyClass：

```csharp
public class MyClassChild : MyClass
{
    /// <summary>
    /// 模拟一个非托管资源
    /// </summary>
    private IntPtr NativeResource { get; set; } = Marshal.AllocHGlobal(100);
    
    /// <summary>
    /// 模拟一个托管资源
    /// </summary>
    public Random ManagedResource { get; set; } = new Random();
    
    /// <summary>
    /// 释放标记
    /// </summary>
    private bool disposed;
    
    /// <summary>
    /// 非密封类可重写的Dispose方法，方便子类继承时可重写
    /// </summary>
    /// <param name="disposing"></param>
    protected override void Dispose(bool disposing)
    {
        if (disposed)
        {
            return;
        }
        //清理托管资源
        if (disposing)
        {
            if (ManagedResource != null)
            {
                ManagedResource = null;
            }
        }
        //清理非托管资源
        if (NativeResource != IntPtr.Zero)
        {
            Marshal.FreeHGlobal(NativeResource);
            NativeResource = IntPtr.Zero;
        }
        base.Dispose(disposing);
    }
}
```

如果不是虚方法，那么就很有可能让开发者在子类继承的时候忽略掉父类的清理工作，所以，基于继承体系的原因，我们要提供这样的一个虚方法。

其次，提供的这个虚方法是一个带 `bool` 参数的，带这个参数的目的，是为了释放资源时区分对待托管资源和非托管资源，而实现自 `IDisposable` 的 `Dispose` 方法调用时，传入的是 `true`，而终结器调用的时候，传入的是 `false`，当传入 `true` 时代表要同时处理托管资源和非托管资源；而传入 `false` 则只需要处理非托管资源即可。

那为什么要区别对待托管资源和非托管资源？在这个问题之前，其实我们应该先弄明白：托管资源需要手动清理吗？不妨将 C# 的类型分为两类：一类实现了 `IDisposable`，另一类则没有。前者我们定义为非普通类型，后者为普通类型。非普通类型包含了非托管资源，实现了 `IDisposable`，但又包含有自身是托管资源，所以不普通，对于我们刚才的问题，答案就是：普通类型不需要手动清理，而非普通类型需要手动清理。

而我们的 `Dispose` 模式设计思路在于：如果显式调用 `Dispose`，那么类型就该按部就班的将自己的资源全部释放，如果忘记了调用 `Dispose`，那就假定自己的所有资源 (哪怕是非普通类型) 都交给 GC 了，所以不需要手动清理，所以这就理解为什么实现自 `IDisposable` 的 `Dispose` 中调用虚方法是传 `true`，终结器中传 `false` 了。

同时我们还注意到了，虚方法首先判断了 `disposed` 字段，这个字段用于判断对象的释放状态，这意味着多次调用 `Dispose` 时，如果对象已经被清理过了，那么清理工作就不用再继续。

但 `Dispose` 并不代表把对象置为了 `null`，且已经被回收彻底不存在了。但事实上，对象的引用还可能存在的，只是不再是正常的状态了，所以我们明白有时候我们调用数据库上下文有时候为什么会报“数据库连接已被释放”之类的异常了。

所以，`disposed` 字段的存在，用来表示对象是否被释放过。

## 如果对象包含非托管类型的字段或属性的类型应该是可释放的

这句话读起来可能有点绕啊，也就是说，如果对象的某些字段或属性是 IDisposable 的子类，比如 `FileStream`，那么这个类也应该实现 `IDisposable`。

之前我们说过 C# 的类型分为普通类型和非普通类型，非普通类型包含普通的自身和非托管资源。那么，如果类的某个字段或属性的类型是非普通类型，那么这个类型也应该是非普通类型，应该也要实现 `IDisposable` 接口。

举个栗子，如果一个类型，组合了 `FileStream`，那么它应该实现 `IDisposable` 接口，代码如下：

```csharp
public class MyClass2 : IDisposable
{
    ~MyClass2()
    {
        Dispose(false);
    }
    public FileStream FileStream { get; set; }
    /// <summary>
    /// 释放标记
    /// </summary>
    private bool disposed;
    /// <summary>执行与释放或重置非托管资源关联的应用程序定义的任务。</summary>
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    /// <summary>
    /// 非密封类可重写的Dispose方法，方便子类继承时可重写
    /// </summary>
    /// <param name="disposing"></param>
    protected virtual void Dispose(bool disposing)
    {
        if (disposed)
        {
            return;
        }
        //清理托管资源
        if (disposing)
        {
            //todo
        }
        //清理非托管资源
        if (FileStream != null)
        {
            FileStream.Dispose();
            FileStream = null;
        }
        //告诉自己已经被释放
        disposed = true;
    }
}
```

因为类型包含了 `FileStream` 类型的字段，所以它包含了非普通类型，我们仍旧需要为这个类型实现 `IDisposable` 接口。

## 及时释放资源

可能很多人会问啊，GC 已经帮我们隐式的释放了资源，为什么还要主动地释放资源，我们先来看一个例子：

```csharp
private void button6_Click(object sender, EventArgs e)
{
    var fs = new FileStream(@"C:\1.txt",FileMode.OpenOrCreate,FileAccess.ReadWrite);
}
private void button7_Click(object sender, EventArgs e)
{
    GC.Collect();
}
```

上面的代码在 WinForm 程序中，单击按钮 6，打开一个文件流，单击按钮 7 执行 GC 回收所有“代”(下文将指出代的概念) 的垃圾，如果连续单击两次按钮 6，将会抛异常：

如果单击按钮 6 再单击按钮 7，然后再单击按钮 6 则不会出现这个问题。

我们来分析一下：在单击按钮 6 的时候打开一个文件，方法已经执行完毕，fs 已经没有被任何地方引用了，所以被标记为了垃圾，那么什么时候被回收呢，或者 GC 什么时候开始工作？微软官方的解释是，当满足以下条件之一时，GC 才会工作：

1. 系统具有较低的物理内存；
2. 由托管堆上已分配的对象使用的内存超出了可接受的范围；
3. 手动调用 `GC.Collect` 方法，但几乎所有的情况下，我们都不必调用，因为垃圾回收器会自动调用它，但在上面的例子中，为了体验一下不及时回收垃圾带来的危害，所以手动调用了 `GC.Collect`，大家也可以仔细体会一下运行这个方法带来的不同。

GC 还有个“代”的概念，一共分 3 代：0 代、1 代、2 代。而这三代，相当于是三个队列容器，第 0 代包含的是一些短期生存的对象，上面的例子 fs 就是个短期对象，当方法执行完后，fs 就被丢到了 GC 的第 0 代，但不进行垃圾回收，只有当第 0 代满了的时候，系统认为此时满足了低内存的条件，才会触发垃圾回收事件。所以我们永远不知道 fs 什么时候被回收掉，在回收之前，它实际上已经没有用处了，但始终占着系统资源不放 (占着茅坑不拉屎)，这对系统来说是种极大的浪费，而且这种浪费还会干扰整个系统的运行，比如我们的例子，由于它始终占着资源，就导致了我们不能再对文件进行访问了。

不及时释放资源还会带来另外的一个问题，虽然之前我们说实现 `IDisposable` 接口的类，GC 可以自动帮我们释放，但这个过程被延长了，因为它不是在一次回收中完成所有的清理工作，即使 GC 自动帮我们释放了，那也是先调用 `FileStream` 的终结器，在下一次的垃圾回收时才会真正的被释放。

了解到危害后，我们在打码过程中，如果我们明知道它应该被 `using` 起来时，一定要 `using` 起来：

```csharp
using (var fs = new FileStream(@"C:\1.txt", FileMode.OpenOrCreate, FileAccess.ReadWrite))
{
}
```

## 需不需要将不再使用的对象置为 null

在上文的内容中，我们都提到要释放资源，但并没有说明需不需要将不再使用的对象置为 `null`，而这个问题也是一直以来争议很大的问题，有人认为将对象置为 `null` 能让 GC 更早地发现垃圾，也有人认为这并没有什么卵用。其实这个问题首先是从方法的内部被提起的，为了更好的说明这个问题，我们先来段代码来检验一下：

```csharp
private void button6_Click(object sender, EventArgs e)
{
    var mc1 = new MyClass() { Name = "mc1" };
    var mc2 = new MyClass() { Name = "mc2" };
    mc1 = null;
}
private void button7_Click(object sender, EventArgs e)
{
    GC.Collect();
}
public class MyClass
{
    public string Name { get; set; }
    ~MyClass()
    {
        MessageBox.Show(Name + "被销毁了");
    }
}
```

单击按钮 6，再单击按钮 7，我们发现：

没有置为 `null` 的 mc2 会先被释放，虽然它在 mc1 被置为 `null` 之后；

在 CLR 托管的应用程序中，有一个“根”的概念，类型的静态字段、方法参数以及局部变量都可以被作为“根”存在（值类型不能作为“根”，只有引用类型才能作为“根”）。

上面的代码中，mc1 和 mc2 在代码运行过程中分别会在内存中创建一个“根”。在垃圾回收的过程中，GC 会沿着线程栈扫描“根”(栈的特点先进后出，也就是 mc2 在 mc1 之后进栈，但 mc2 比 mc1 先出栈)，检查完毕后还会检查所有引用类型的静态字段的集合，当检查到方法内存在“根”时，如果发现没有任何一个地方引用这个局部变量的时候，不管你是否已经显式的置为 `null` 这都意味着“根”已经被停止，然后 GC 就会发现该根的引用为空，就会被标记为可被释放，这也代表着 mc1 和 mc2 的内存空间可以被释放，所以上面的代码 mc1=null 没有任何意义 (方法的参数变量也是如此)。

其实 .NET 的 JIT 编译器是一个优化过的编译器，所以如果我们代码里面将局部变量置为 `null`，这样的语句会被忽略掉：

```csharp
s = null;
```

如果我们的项目是在 Release 配置下的，上面的代码压根就不会被编译到 dll，正是由于我们上面的分析，所以很多人都会认为将对象赋值为 `null` 完全没有必要，但是，在另一种情况下，就完全有必要将对象赋值为 `null`，那就是静态字段或属性，但这斌不意味着将对象赋值为 `null` 就是将它的静态字段赋值为 null：

```csharp
private void button6_Click(object sender, EventArgs e)
{
    var mc = new MyClass() { Name = "mc" };
}
private void button7_Click(object sender, EventArgs e)
{
    GC.Collect();
}
public class MyClass
{
    public string Name { get; set; }
    public static MyClass2 MyClass2 { get; set; } = new MyClass2();
    ~MyClass()
    {
        //MyClass2 = null;
        MessageBox.Show(Name + "被销毁了");
    }
}
public class MyClass2
{
    ~MyClass2()
    {
        MessageBox.Show("MyClass2被释放");
    }
}
```

上面的代码运行我们会发现，当 mc 被回收时，它的静态属性并没有被 GC 回收，而我们将 MyClass 终结器中的 MyClass2 = null 的注释取消，再运行，当我们两次点击按钮 7 的时候，属性 MyClass2 才被真正的释放，因为第一次 GC 的时候只是在终结器里面将 MyClass 属性置为 `null`，在第二次 GC 的时候才当作垃圾回收了，之所以静态变量不被释放 (即使赋值为 `null` 也不会被编译器优化)，是因为类型的静态字段一旦被创建，就被作为“根”存在，基本上不参与 GC，所以 GC 始终不会认为它是个垃圾，而非静态字段则不会有这样的问题。

所以在实际工作当中，一旦我们感觉静态变量所占用的内存空间较大的时候，并且不会再使用，便可以将其置为 `null`，最典型的案例就是缓存的过期策略的实现了，将静态变量置为 `null` 这或许不是很有必要，但这绝对是一个好的习惯，试想一个项目中，如果将某个静态变量作为全局的缓存，如果没有做过期策略，一旦项目运行，那么它所占的内存空间只增不减，最终顶爆机器内存，所以，有个建议就是：**尽量地少用静态变量**。
