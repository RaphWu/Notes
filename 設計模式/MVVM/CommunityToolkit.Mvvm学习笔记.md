---
aliases:
date:
update:
author: MelonSuika
language: C#
sourceurl: https://so.csdn.net/so/search?q=CommunityToolkit.Mvvm%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0&t=blog&u=BadAyase
tags:
  - CSharp
  - CommunityToolkit_Mvvm
---

# Links

[CommunityToolkit.Mvvm学习笔记（1）—— 概述](https://blog.csdn.net/BadAyase/article/details/125102567)
[CommunityToolkit.Mvvm学习笔记（2）—— ObservableObject](https://blog.csdn.net/BadAyase/article/details/125112080)
[CommunityToolkit.Mvvm学习笔记（3）—— ObservableRecipient](https://blog.csdn.net/BadAyase/article/details/125116004)
[CommunityToolkit.Mvvm学习笔记（4）—— Messenger](https://blog.csdn.net/BadAyase/article/details/125128698)
[CommunityToolkit.Mvvm学习笔记（5）—— ObservableValidator](https://blog.csdn.net/BadAyase/article/details/128943859)
[CommunityToolkit.Mvvm学习笔记（6）—— RelayCommand](https://blog.csdn.net/BadAyase/article/details/125158210)
[CommunityToolkit.Mvvm学习笔记（7）—— Ioc控制反转](https://blog.csdn.net/BadAyase/article/details/125954336)

# CommunityToolkit.Mvvm 学习笔记（1）—— 概述

于 2022-06-03 07:12:03 发布

## 一、前言

先简单谈谈为什么选择学习使用这个 MVVM 框架。
接触.NET 和 WPF 有一阵子了，用过 WPF 的应该都听说过 MVVM 模式，它是一种分离前后端、松耦合的模式。某种程度上来说是 MVC 和 MVP 的升级版，也一定程度上解决了前两者存在的一些问题。因为是模式，所以不仅限于 WPF 这一种开发，许多涉及 UI 的开发都会用到它，比如 Vue。
既然它这么优秀，我自然想使用它。但由于我接触.NET 和 WPF 时间都不长，很多基础知识掌握不好，所以我决定先使用原生的 MVVM，即不用任何框架，自己先徒手实现试试。这样做或许比较费力，最终做出来的东西结构设计上也不完美，但对于整个过程和实现中的一些难点上的体会会更深，在后续使用框架开发时候应该会有一些额外收获。
确实，第一个项目做出来了，有点不伦不类，就自己硬建了几个文件夹 View、Model、ViewModel，然后开始套模式，开始绑定变量，开始绑定命令。就是在绑定命令这一步上，让我觉得 WPF 在原生的 MVVM 上支持太复杂了，因为你需要自己重写有关 ICommand 的一切。而反观几种主流的 MVVM 框架（MvvmLight、Prism 等）在命令的使用上都非常简洁。于是，我的第二个 WPF 项目也决定使用框架。
使用框架，必然少不了选择框架。起初打算用 Prism，因为相对来说，它功能最强大，支持最好。于是从 NuGet 下载添加进项目，然后准备开始学习，翻了翻博客，翻了翻 GitHub，感觉它的学习方式主要是零零散散的博客&GitHub 上的样例，说实话，不太系统（主要是我太菜，看样例太累）。于是转 MvvmLight，网上都说这个框架比较轻量级，且简单易上手。确实，我搜 Mvvm 的时候，它出来的词条最多。于是从 NuGet 搜了下，好家伙！

![[学习笔记_01_01.png]]

似乎几年前就停更了，当然想用它，还是一点问题都没有的。但紧接着，一行小字吸引了我。

![[学习笔记_01_02.png]]

这是它的替代品么？怀着好奇，我点了进去。它是一个微软官方提供支持的 Mvvm 包，而且有着详细的文档，被称为 MvvmLight 的继承者。在这机缘巧合之下，我选择了它，当然好不好用还不知道，先用用看，不好用再换。总结一下，我选它是因为微软官方的 + 文档详细 + 新。

## 二、MVVM Toolkit 概述

首先，放一下它的 [文档链接](https://docs.microsoft.com/zh-cn/windows/communitytoolkit/mvvm/introduction)，喜欢啃文档的自己去看。
Microsoft.Toolkit.Mvvm 包（也称 MVVM Toolkit）是一个现代、快速且模块化的 MVVM 库。它是 Windows 社区工具包的一部分并且围绕着以下几点原则构建的：

- 平台和运行时库相独立 - .NET 2.0 标准和.NET 5（与 UI 框架无关）
- 简单易用 - 在程序结构和编码范式上没有严格要求，即使用灵活
- 自由组件 - 自由选择要使用的组件
- 参考实现 - 精炼且性能好，提供了基础库的接口实现，但缺乏直接使用它们的具体类型

这个包针对是.NET 标准的，所以它能应用于任何平台：UWP，WinForms，WPF，Xamarin，Uno 等；以及任何运行环境下：.NET Native, .NET Core, .NET Framework, or Mono。它能在它们之上运行。且 API 在所有情况下都是相同的，这使得它非常适合构建共享库。

此外，MVVM 工具包还有专门针对.NET 5 的优化点，使得在.NET 5 运行时会有更多内部优化。当然，两种情况下的 API 是相同的，NuGet 总是会解析出包的最佳版本，而不需要用户担心 API 在平台上是否可用。

在 VS 当中安装包：

1. 在解决方案管理器中，右击工程，然后选择管理 NuGet 包。搜索 Microsoft.Toolkit.Mvvm 并安装它。
   ![[学习笔记_01_03.png]]
   
2. 添加一个 using 指令来使用 API:

```csharp
using Microsoft.Toolkit.Mvvm;
```

3. 代码示例可以在 MVVM Toolkit 的其他文档页面和项目的单元测试中找到

### 2.1. 何时使用 MVVM 工具包

使用该包可以访问一组标准的、独立的、轻量级的类型，这些类型为使用 MVVM 模式的程序提供了起始实现。通常，这些类型本身就足以满足用户的需求，而不需要额外的外部引用。

包括的类型有：

- Microsoft.Toolkit.Mvvm.ComponentModel
	- `ObservableObject`
	- `ObservableRecipient`
	- `ObservableValidator`
- Microsoft.Toolkit.Mvvm.DependencyInjection
	- `Ioc`
- Microsoft.Toolkit.Mvvm.Input
	- `RelayCommand`
	- `RelayCommand<T>`
	- `AsyncRelayCommand`
	- `AsyncRelayCommand<T>`
	- `IRelayCommand`
	- `IRelayCommand<in T>`
	- `IAsyncRelayCommand`
	- `IAsyncRelayCommand<in T>`
- Microsoft.Toolkit.Mvvm.Messaging
	- `IMessanger`
	- `WeakReferenceMessenger`
	- `StrongReferenceMessenger`
	- `IRecipient<TMessage>`
	- `MessageHandler<TRecipient, TMessage>`
- Microsoft.Toolkit.Mvvm.Messaging.Messages
	- `PropertyChangedMessage<T>`
	- `RequestMessage<T>`
	- `AsyncRequestMessage<T>`
	- `CollectionRequestMessage<T>`
	- `AsyncCollectionRequestMessage<T>`
	- `ValueChangedMessage<T>`

这个包旨在提供更多的灵活性，使得开发者能自由选择使用哪些组件。所有的类型都是松耦合的（loosely-coupled），因此只需要包含你需要用到的。当使用它们来构建程序时，没有必要完全使用特定的 API，也没有一套强制性的模式要遵循。

这节的内容和标题好像没有太强烈的关系，不过硬理解一下就是如果你的程序要用 Mvvm 模式，你可以用这个包。且包中的众多类型你用得着的就用，用不着的就不用。

### 2.2. 更多资源

示例应用程序
更多单元测试中的示例

## 三、小结

这节主要是大概讲讲 MVVM Toolkit 是什么以及如何加入到项目中。接下来开始正式学习。

---

# CommunityToolkit.Mvvm 学习笔记（2）—— ObservableObject

已于 2022-06-07 09:06:20 修改

## 一、前言

这一节在文档中是属于 MVVM- 组件模型里的一个分支，

![[学习笔记_02_01.png]]

但是它的上级条目并没有信息，所以直接来看它的内容吧。

## 二、ObservableObject

`ObservableObject` 是一个基类，通过实现 `INotifyPropertyChanged` 和 `INotifyPropertyChanging` 接口可以使它的对象可被监视。**它可以作为所有需要支持==属性更改通知功能==的对象的起点。** 这句话很拗口，因为是直译的，同时也很重要。我来解读一下，MVVM 模式中，哪个部分是需要支持属性更改通知的功能的？显然不是 View，那是 ViewModel 还是 Model 呢？如果 Model 通知了 View，那不如叫 MV 模式好了，还需要 ViewModel 做什么？所以这句话意思是它是 ViewModel 部分中重要的成员。可以说，每个 View 有对应的 ViewModel，而 ViewModel 都要继承这个类来使得属性能得以暴露给 View。

### 2.1. 它是怎么起作用的

知道了 `ObservableObject` 的作用之后，我们来看看它是通过什么方式来让程序员可以实现各种功能的。

`ObservableObject` 有着以下主要特性：

- 提供了 `INotifyPropertyChanged` 和 `INotifyPropertyChanging` 的基本实现，暴露了 `PropertyChanged` 和 `PropertyChanging` 事件。
- 提供一系列 `SetProperty` 方法，能轻松地设置继承于 `ObservableObject` 的类型的属性值，并且自动激发相应的事件。
- 提供了 `SetPropertyAndNotifyOnCompletion` 方法，该方法与 `SetProperty` 类似，但可以设置 `Task` 属性，并在分配任务完成时自动激发通知事件。
- 暴露了 `OnPropertyChanged` 和 `OnPropertyChanging` 方法，这些方法能在派生类中被重写，以自定义如何激发通知事件。

### 2.2. 简单属性的通知

下面有一个如何实现支持一个自定义属性的通知的例子：

```csharp
public class User : ObservableObject
{
	private string name;
	public string Name
	{
		get => name;
		set => SetProperty(ref name, value);
	}
}
```

它提供了 `SetProperty(ref T, T, string)` 方法来检查当前属性的值，并更新它是否变化，接着会自动激发相关事件。属性名通过使用 `[CallerMemberName]` 属性自动会被捕捉，所以并不需要手动指定要更新的属性。

原生的写法应该是这样，从代码量上来看有所简化，MVVM 工具包的优点是属性名会自动捕捉。

```csharp
public class A : INotifyPropertyChanged
{
	public event PropertyChangedEventHandler PropertyChanged;
	private Type _p;
	public Type P
	{
		get => _p;
		set
		{
			_p = value;
			if (PropertyChanged != null)
				PropertyChanged.Invoke(this, new PropertyChangedEventArgs(nameof(P)));
		}
	}
}
```

### 2.3. 封装一个不具备监视功能的模型

在一般场景下，例如，当处理数据库条目时，创建一个封装的可绑定的模型，该模型会传递数据库模型的属性，并在需要时激发属性的变化通知。

有些特殊场景，这样的功能也是被需要的，当想要将支持通知的功能注入到没有实现 `INotifyPropertyChanged` 接口的模型中时。`ObservableObject` 提供了一个专门的方法以简化该过程。在下面的例子中，`User` 是一个数据模型，它直接映射了一张数据表，并且没有继承自 `ObservableObject`：

```csharp
public class ObservableUser : ObservableObject
{
	private readonly User user;
	public ObservableUser(User user) => this.user = user;
	public string Name
	{
		get => user.Name;
		set => SetProperty(user.Name, value, user, (u, n) => u.Name = n);
	}
}
```

在本例中，我们使用了 `SetProperty<TModel, T>(T, T, TModel, Action<TModel, T>, string)` 的重载。这里的函数签名比上个例子复杂许多，不过为了让代码依旧高效地在这种场景下访问一个字段这是有必要的。我们可以详细分析这个方法签名的每个部分，以了解不同组件的作用：

- `TModel` 是一个类型参数，指明了要封装的模型的类型。例子中，它是 `User` 类。值得注意的是，我们不需要显式地指明它——C#编译器将根据我们调用 `SetProperty` 的方式自动的推测出类型。很显然，例子中我们填的是 `user` 对象而不是 `User` 类型。
- `T` 是想要设置的属性的类型。和 `TModel` 类似，它也会被自动推断出来。
- `T oldValue` 指的是第一个参数，本例中我们用的是 `user.Name` 来传递我们要封装的属性的当前值。（类似于原生 `set` 中的 `p = value` 的 `p`）
- `T newValue` 是我们要设置属性的新的值，在这儿我们传了 `value` 进去，它是属性 `setter` 中的输入值。
- `TModel model` 是我们要封装的目标模型，本例中我们传了 `user`。
- `Action<TModel, T> callback` 是一个方法，若属性的新值与当前不同，该方法就会被调用，并且属性会被设置。这些功能将有这个回调函数来完成，它将目标模型和新的属性值作为输入。在本例中，我们分配了一个叫做 `n` 的输入值给 `Name` 属性（通过 `u.Name= n` 来赋值）。这儿有几点非常重要，就是**要避免从当前作用域中获取值，并且只与作为回调输入的值进行交互，因为这可以让 C#编译器缓存回调函数并执行一些性能优化。** 正因为如此，我们不是直接访问这里的 `user` 字段或 `setter` 的 `value` 值，而是只使用 `lambda` 表达式中的输入参数。

`SetProperty<TModel, T>(T, T, TModel, Action<TModel, T>, string)` 方法使得创建封装属性格外简单，因为它负责检索并设置目标属性，同时提供了一个紧凑的 API（指的这个 API 内部帮我们做了大量工作，我们只需要填几个参数就好，从代码量上来看确实极简，但从个人角度来讲，这种多参数的 API 我不是很喜欢）。

> Note：
> 与用 LINQ 表达式实现该方法相比，具体指的是通过 Expression\<Func\<T\>\> 代替状态和回调参数，这种方式的性能改进非常显著。
> 特别地，这个版本比使用 LINQ 表达式的版本快了约 200 倍，并且没有用到任何的内存分配。

### 2.4. 处理 Task\<T\>属性

如果要用在 `Task` 类型的属性上，在 `Task` 完成时激发通知事件以便适时更新绑定（Binding，也可叫关联）。例如，在其它任务操作时，显示一个加载条或其他的状态信息。`ObservableObject` 有 API 用于这种场景：

```csharp
public class MyModel : ObservableObject
{
	private TaskNotifier<int>? requestTask;
	public Task<int>? RequestTask
	{
		get => requestTask;
		set => SetPropertyAndNotifyOnCompletion(ref requestTask, value);
	}
	public void RequestValue()
	{
		RequestTask = WebService.LoadMyValueAsync();
	}
}
```

这里 `SetPropertyAndNotifyOnCompletion<T>(ref TaskNotifier<T>, Task<T>, string)` 方法会负责更新目标字段，监视新任务，并在任务完成时激发通知事件。这样，就可以绑定到任务属性，并在状态发生变化时收到通知。 `TaskNotifier<T>` 是一个由 `ObservableObject` 暴露的特殊类型，它封装了一个 `Task<T>` 实例，为该方法启用必要的通知逻辑。如果你只有一个普通的任务，`TaskNotifier` 类型也可以直接使用。

> Note：
> SetPropertyAndNotifyOnCompletion 是被用来替换 Microsoft.Toolkit 包中的 NotifyTaskCompletion\<T\> 类型的。
> 如果该类型正在被使用，它可以用内部的 Task（或 Task\<TResult\>）属性替换，接着 SetPropertyAndNotifyOnCompletion 能被用于设置它的值并且激发通知改变。
> 在 Task 实例中，所有由 NotifyTaskCompletion\<T\> 暴露的属性都是直接可用的。

## 三、小结

这节内容在 WPF 原生的 MVVM 中也不难实现，更多的是熟悉 MVVM 工具包的里的写法。

---

# CommunityToolkit.Mvvm 学习笔记（3）—— ObservableRecipient

已于 2022-06-05 08:41:09 修改

## 一、ObservableRecipient

### 1.1. 定义

所处的位置，
命名控件：`Microsoft.Toolkit.Mvvm.ComponentModel`
程序集：`Microsoft.Toolkit.Mvvm.dll`
包：`Microsoft.Toolkit.Mvvm`

`ObservableRecipient` 类型是可监视对象（Observable objects）的一个基类，这些对象扮演着消息接收者的角色。`ObservableRecipient` 类是 `ObservableObject` 的拓展，它也提供了使用 `IMessenger` 类型的内置支持。

![[学习笔记_03_01.png]]

继承关系：
Object→ObservableObject→ObservableRecipient

相关的平台 API：

- `ObservableRecipient`
- `ObservableObject`
- `IMessenger`
- `WeakReferenceMessenger`
- `IRecipient<TMessage>`
- `PropertyChangedMessage<T>`

### 1.2. 它是如何工作的

`ObservableRecipient` 设计出来是用来作为 `viewmodel` 的基础（或者说基对象）的，它也使用了 `IMessenger` 的特性，因为它为 `IMessenger` 提供了内置支持。特别地：

- 它有一个无参的构造函数和一个接受 `IMessenger` 实例的构造函数，被用作依赖注入。
  它还暴露了一个 `Messenger` 属性，可用来在 viewmodel 中收发消息。如果使用了无参构造函数，`WeakReferenceMessenger.Default` 实例将会被分配给 `Messenger` 属性。
- 它暴露了 `IsActive` 属性以激活（Activate）/ 禁用（Deactivate）viewmodel。在这种情况下，激活意味着 viewmodel 被标记为正在使用，例如，它将开始监听已注册的消息，执行其他的设置操作等。有两个相关的方法 `OnActivated` 和 `OnDeactivated`，在属性改变值时，它们会被调用。默认情况下，`OnDeactivated` 会自动从所有已注册的消息中注销当前实例。为了获得最佳效果和避免内存泄漏，建议使用 `OnActivate` 来注册消息，使用 `OnDeactivate` 来做清理操作。这种模式允许 viewmodel 启停多次，同时可以安全地收集，而不会在每次停用的时候产生内存泄漏的风险。默认情况下，`OnActivate` 会自动注册所有通过 `IRecipient<TMessage>` 接口定义的消息处理器。
- 它暴露了 `Broadcast<T>(T, T, string)` 方法，该方法通过 `IMessenger` 实例（可以从 `Messenger` 属性获得）发送 `PropertyChangedMessage<T>` 消息。这可以用来轻松传播 viewmodel 中的变化，而不需要手动地检索 `Messenger` 实例来使用。该方法通过各种 `SetProperty` 方法的重载来使用，这些方法还有一个附加的 `bool broadcast` 属性，用于指示是否也发送消息。

下面有一个 viewmodel 的示例，它在激活时接收 `LoggedInUserRequestMessage` 消息：

```csharp
public class MyViewModel : ObservableRecipient, IRecipient<LoggedInUserRequestMessage>
{
	public void Receive(LoggedInUserRequestMessage message)
	{
		// 处理消息的代码
	}
}
```

在以上例子中，`OnActivated` 自动注册实例作为 `LoggedInUserRequestMessage` 消息的接收者，使用该方法作为要调用的动作。使用 `IRecipient<TMessage>` 接口并不是强制的，注册也可以手动完成（甚至只使用内联 lambda 表达式）：

```csharp
public class MyViewModel : ObservableRecipient
{
	protected override void OnActivated()
	{
		// Using a method group...
        Messenger.Register<MyViewModel, LoggedInUserRequestMessage>(this, (r, m) => r.Receive(m));

        // ...or a lambda expression
        Messenger.Register<MyViewModel, LoggedInUserRequestMessage>(this, (r, m) =>
        {
            // Handle the message here
        });
	}
	private void Receive(LoggedInUserRequestMessage message)
    {
        // Handle the message here
    }
}
```

---

# CommunityToolkit.Mvvm 学习笔记（4）—— Messenger

已于 2023-09-14 12:54:23 修改

## Messenger 概述

如果你对 WPF 有一定了解，你应该知道 WPF 中的命令是一个实现了 `ICommand` 接口的类。同样本文虽然标题是 `Messenger`，但也要从 `IMessenger` 接口说起。至于 `Messenger` 的中文名，我觉得就叫它的直译“信使”好了，毕竟传递消息就是信使的能力嘛。

### 1 IMessenger 接口

命名空间：`Microsoft.Toolkit.Mvvm.Messaging`
程序集：`Microsoft.Toolkit.Mvvm.dll`
包：`Microsoft.Toolkit.Mvvm`

`IMessenger` 接口提供了不同对象间交换消息的类应该具有的能力，这对解耦程序的不同模块非常有用，而不必持有引用类型的强引用。

> 强引用：
> 当应用程序的代码能够访问到程序正在使用的对象时，GC（.NET 平台的垃圾收集器）就不能回收该对象，就称程序对该对象进行了强引用。
> 强引用和这有什么关系呢？
> 如果结合“解耦程序”来理解就会明了许多。
> 不关心代码能不能访问到该对象，我一样可以交换消息。

你也可以将消息发送到指定通道（通过令牌唯一标识，uniquely identified by a token），让不同的信使呆在程序的不同部分。

要使用 `IMessenger` 功能，首先定义一个消息类，像这样：

```csharp
// 定义一个登陆完成的消息
//（这里为了说明步骤而不是谈具体实现，所以只是定义了一个普通的 class，
// 实际上它应该继承并实现 IMessenger 接口，不过 MVVM 工具包为你提供现成的类，后面会提到）
public sealed class LoginCompletedMessage { }
// sealed 关键词会防止其它类继承该类
// 至于为什么加这个关键词，因为例子里加了
```

接着，为该消息注册一个接收者（Recipient）：

```csharp
Messenger.Default.Register<MyRecipientType, LoginCompletedMessage>(this, (r, m) =>
{
    // Handle the message here...
});
```

这里的消息处理器是一个有两个参数的 lambda 表达式：接收者 `(r，recipient)` 和消息 `(m，message)`。这么做是为了避免分配闭包（这个词可以百度了解一下，在 lambda 表达式中很常见），如果表达式捕获了当前实例，就会生成闭包。将接收者作为参数，是为了能在处理程序中直接访问它，而不需要手动进行类型转换。

```csharp
xx_handler(object sender, Args e)
{
	// 这类用法大家一定不陌生，需要你手动将 object 转换类型
	(sender as xxType).Method();
}
```

将接收者作为参数能使代码冗余更少，更可靠，因为所有的检查在构建时就完成了。

如果处理器 (handler) 定义在与接收者相同的类中，那它也可以直接访问私有成员。这允许消息处理器是一个静态方法，这使得 C#编译器能执行一系列额外的内存优化（例如缓存委托，避免不必要的内存分配）。最后，在需要时发送消息，就像这样：

```csharp
Messenger.Default.Send<LoginCompletedMessage>();
```

此外，若当前作用域内有可用的带有正确签名的方法，方法组语法还可用于指定在接收消息时调用的消息处理器。这有助于使注册和处理逻辑分离。在上个例子的基础上，考虑一个带有以下方法的类：

```csharp
private static void Receive(MyRecipientType recipient, LoginCompletedMessage message)
{
    // Handle the message there
}
```

以下代码进行注册：

```csharp
Messenger.Default.Register(this, Receive);
```

C#编译器自动将表达式转化为一个与 `Register<TRecipient,TMessage>(IMessenger, TRecipient, MessageHandler<TRecipient,TMessage>)` 兼容的 `MessageHandler<TRecipient,TMessage>` 实例。如果该方法有多个重载可用，每个重载处理着不同的消息类型：C#编译器将自动根据当前消息类选择一个合适的。也可以显式注册使用 `IRecipient<TMessage>` 接口的消息处理器。若这样做，接收者只需要实现该接口，然后调用 `RegisterAll(IMessenger, Object)` 扩展，该拓展将会自动注册由接收者类型所声明的所有处理器。当然，只有一个处理器也支持。

下面是 `IMessenger` 接口，

![[学习笔记_04_01.png]]

`IMessenger` 有两个派生类：

- `Microsoft.Toolkit.Mvvm.Messaging.WeakReferenceMessenger`
- `Microsoft.Toolkit.Mvvm.Messaging.StrongReferenceMessenger`

这两个类是 MVVM 工具包提供的两种即用的实现（就是已经给你实现好了 `IMessenger`），

- 前者在内部使用的是弱引用，为接收者提供自动内存管理。
- 后者使用强引用，要求开发者（在不再需要接收者时）手动取消接收者的订阅。但作为交换，它提供了更好的性能和更少的内存使用。

相关的平台 API：`IMessenger`, `WeakReferenceMessenger`, `StrongReferenceMessenger`, `IRecipient<TMessage>`, `MessageHandler<TRecipient, TMessage>`, `ObservableRecipient`, `RequestMessage<T>`, `AsyncRequestMessage<T>`, `CollectionRequestMessage<T>`, `AsyncCollectionRequestMessage<T>`.

### 2 信使的工作原理

实现了 `IMessenger` 接口的类负责维护消息接收者（recipient）和注册消息类之间的链接（link），以及相关的消息处理器（handlers）。任何对象都可以使用消息处理器注册为指定消息类的接收者，每当使用 `IMessenger` 实例发送该类的消息时，消息处理器会被调用。也可以通过特定通信通道（由唯一令牌标识）发送消息，这样多个模块间可以交换相同类型的消息，而不会引起冲突。不使用令牌发送的消息使用默认共享通道。

有两种注册消息的方式：

一种是使用 `IRecipient<TMessage>` 接口，另一种是用 `MessageHandler<TRecipient, TMessage>` 委托来作为消息处理器。

![[学习笔记_04_02.png]]

第一种方法，允许你用 `RegisterAll` 扩展方法来注册所有处理器，它会自动注册所有声明消息处理器的接收者。而后者在你需要更灵活或者简单的 lambda 表达式作为消息处理器时更适用。

`WeakReferenceMessenger` 和 `StrongReferenceMessenger` 均暴露了 `Default` 属性，该属性提供了一个内置于包中的线程安全实现。如果需要，也可以创建多个 `messenger` 实例，例如，将不同的实例用 DI 服务提供者（Service provider）注入到程序不同的模块中（如，多个窗口运行在同一个进程中时）。

> 注意：
> 由于 WeakReferenceMessenger 用起来更简单，并与 MvvmLight 库中的 messenger 的行为更匹配，所以它是 MVVM Toolkit 中 ObservableRecipient 默认的使用类型。
> 通过传递实例给该类的构造函数，StrongReferenceMessenger 也仍然可用。

### 3 收发消息

考虑以下代码：

```csharp
// Create a message
public class LoggedInUserChangedMessage : ValueChangedMessage<User>
{
    public LoggedInUserChangedMessage(User user) : base(user)
    {
    }
}

// Register a message in some module
WeakReferenceMessenger.Default.Register<LoggedInUserChangedMessage>(this, (r, m) =>
{
    // Handle the message here, with r being the recipient and m being the
    // input message. Using the recipient passed as input makes it so that
    // the lambda expression doesn't capture "this", improving performance.
});

// Send a message from some other module
WeakReferenceMessenger.Default.Send(new LoggedInUserChangedMessage(user));
```

假设在一个简单的消息通信程序（聊天软件）中使用该消息类，该程序显示一个带有当前登录用户的用户名和简介图像的标题栏，一个带有聊天列表的面板，以及另一个带有当前聊天消息的面板（选中了其中一个）。假设这三个部分分别由 `HeaderViewModel` , `ConversationsListViewModel` 和 `ConversationViewModel` 支持。在该场景中，登录操作完成后， `LoggedInUserChangedMessage` 消息会由 `HeaderViewModel` 发送，并且其他两个 viewmodel 会为该消息注册处理器。例如，`ConversationsListViewModel` 将为新用户加载对话列表，而 `ConversationViewModel` 将关闭当前对话（如果存在的话）。

`IMessenger` 实例负责将消息分发给所有已注册的接收者。注意，接收者可以订阅指定类型的消息。还有一点要注意，继承的消息类没有在 MVVM Toolkit 提供的默认 `IMessenger` 实现中注册。

当不再需要某个接收者时，你应该注销它，使其停止接收消息。你可以通过消息类、注册令牌或接收者来注销它：

```csharp
// Unregisters the recipient from a message type
WeakReferenceMessenger.Default.Unregister<LoggedInUserChangedMessage>(this);

// Unregisters the recipient from a message type in a specified channel
WeakReferenceMessenger.Default.Unregister<LoggedInUserChangedMessage, int>(this, 42);

// Unregister the recipient from all messages, across all channels
WeakReferenceMessenger.Default.UnregisterAll(this);
```

> 警告：
> 正如前面所述，当使用 WeakReferenceMessenger 时，上面的注销操作不是严格需要的，因为使用弱引用来追踪接收者意味着不用的接收者即使仍然有激活的消息处理程序，它们仍会被 GC 清理。不过，取消订阅它们仍然是一个好的做法，这可以提高性能。
>
> 另一方面，StrongReferenceMessenger 实现使用了强引用来跟踪注册的接收者。这样做是出于性能考虑，这意味着每个注册的接收者应该手动被注销以避免内存泄漏。也就是说，只要注册了一个接收者，使用中 StrongReferenceMessenger 实例就会保活对它的引用，这将防止 GC 回收该实例。你可以手动处理它，也可以从 ObservableRecipient 继承，当它被禁用时，默认情况下会自动删除所有接收者的消息注册。

你也可以使用 `IRecipient<TMessage>` 接口来注册消息处理器。这种情况下，每个接收者需要实现给定消息类的接口，并提供一个 `Receive(TMessage)` 方法，该方法会在接收消息时被调用，如下所示：

```csharp
// Create a message
public class MyRecipient : IRecipient<LoggedInUserChangedMessage>
{
    public void Receive(LoggedInUserChangedMessage message)
    {
        // Handle the message here...
    }
}

// Register that specific message...
WeakReferenceMessenger.Default.Register<LoggedInUserChangedMessage>(this);

// ...or alternatively, register all declared handlers
WeakReferenceMessenger.Default.RegisterAll(this);

// Send a message from some other module
WeakReferenceMessenger.Default.Send(new LoggedInUserChangedMessage(user));
```

### 4 使用请求消息

`messenger` 实例另一个有用的特性是，它可以用于从一个模块向另一个模块请求值。要做到这点，该包包含了一个 `RequestMessage<T>` 基类，如下使用：

```csharp
// Create a message
public class LoggedInUserRequestMessage : RequestMessage<User>
{
}

// Register the receiver in a module
WeakReferenceMessenger.Default.Register<MyViewModel, LoggedInUserRequestMessage>(this, (r, m) =>
{
    // Assume that "CurrentUser" is a private member in our viewmodel.
    // As before, we're accessing it through the recipient passed as
    // input to the handler, to avoid capturing "this" in the delegate.
    m.Reply(r.CurrentUser);
});

// Request the value from another module
User user = WeakReferenceMessenger.Default.Send<LoggedInUserRequestMessage>();
```

`RequestMessage<T>` 类包含了一个隐式转换器，能使会话从 `LoggedInUserRequestMessage` 到其包含的 User 对象成为可能。这也将检查是否收到了消息的响应，若没有收到，则抛出异常。也可以在没有强制响应保证的情况下发送请求消息：只需将返回的消息存储在本地变量中，然后手动检查响应值是否可用。如果 Send 方法返回时没有收到响应，那么这样做不会触发自动异常。

该命名空间还包括用于其他场景的基础请求消息：
`AsyncRequestMessage<T>`, `CollectionRequestMessage<T>` 和 `AsyncCollectionRequestMessage<T>` 。下面是一个使用异步请求消息的例子：

```csharp
// Create a message
public class LoggedInUserRequestMessage : AsyncRequestMessage<User>
{
}

// Register the receiver in a module
WeakReferenceMessenger.Default.Register<MyViewModel, LoggedInUserRequestMessage>(this, (r, m) =>
{
    m.Reply(r.GetCurrentUserAsync()); // We're replying with a Task<User>
});

// Request the value from another module (we can directly await on the request)
User user = await WeakReferenceMessenger.Default.Send<LoggedInUserRequestMessage>();
```

---

# CommunityToolkit.Mvvm 学习笔记（5）—— ObservableValidator

已于 2023-02-10 09:44:01 修改

## 一、引言

第 5 节的内容是验证（validation）相关的，上次看到时还是初学，内容没看太懂，快速过一遍官方文档，有点囫囵吞枣的意思。

这次回过头来学呢，是因为我需要实现一个功能，又因为我当时瞄了一眼这节内容是与验证相关的，还有点印象，所以我直接就决定写篇博客，系统学习下。

我要实现的功能很常见，就是过滤掉一些前端的错误输入，然后给出一定反馈。

废话不多说，直接进入主题。

## 二、ObservableValidator

这节的标题—— `ObservableValidator` （可监视的验证器），一看这个单词就知道，它与 `ObservableObject` 有关，能监视到属性值更新，并且与验证有关。

`ObservableValidator` 是实现了 `INotifyDataErrorInfo` 接口的基类，为验证暴露给其他程序模块的属性提供支持。它也继承自 `ObservableObject` ，所以它也实现了 `INotifyPropertyChanged` 和 `INotifyPropertyChanging` 。它可用作需要同时支持属性更改通知和属性验证的所有类型的起点。

![[学习笔记_05_01.png]]

> 它继承自 ObservableObject 意味着它是个自动通知类。
> 实现了 INotifyDataErrorInfo 接口：
> 这里的接口是代码层面的接口，而非更高层的模块间的接口。代码上的接口意味着，继承它的类都要实现这个接口，意味着这些类需要有这个功能（尽管实现起来各异）。就从这个接口的名称可以看出来，它是用于通知数据错误信息的，很明显，与验证前端输入这点非常符合。
> 总之，若一个类继承了 ObservableValidator ，那就说明该类存在一些希望进行验证的属性。

### 2.1. 它是如何工作的？

`ObservableValidator` 有以下主要特性：

- 提供了 `INotifyDataErrorInfo` 的基本实现，暴露了 `ErrorsChanged` 事件和其他必要的 api。
- 提供了一系列额外的 `SetProperty` 重载（在基本 `ObservableObject` 类提供的重载之上），这些重载提供了在更新值之前自动验证属性和引发必要事件的能力。
- 暴露了许多 `TrySetProperty` 重载，这些重载与 `SetProperty` 类似，但只有在验证成功时才能更新目标属性，并返回生成的错误（若有）以供进一步检查。
- 还暴露了 `ValidateProperty` 方法，如果某个属性值没有更新，但它的验证依赖于已更新的另一个属性的值，那么该方法对于手动触发特定属性的验证非常有用。
- 暴露了 `ValidateAllProperties` 方法，该方法自动执行当前实例中所有公有实例属性的验证，前提是它们至少有一个 `[ValidationAttribute]`。
- 暴露了一个 `ClearAllErrors` 方法，该方法在重置绑定到用户可能想要再次填写的表单的 model 时非常有用。
- 还提供了许多构造函数，允许传递不同的参数来初始化 `ValidationContext` 实例，该实例将用于验证属性。当使用可能需要额外服务或选项才能正 - 常工作的自定义验证属性时，这尤其有用。

### 2.2. 简单属性

> 简单属性就是单个的属性，不是复合的。

下面有个例子，演示了如何实现一个同时支持更改通知和验证的属性：

```csharp
public class RegistrationForm : ObservableValidator
{
	private string name;
	[Required]
	[MinLength(2)]
	[MaxLength(100)]
	public string Name
	{
		get => name;
		set => SetProperty(ref name, value, true);
	}
}
```

这里调用了 `ObservableValidator` 暴露的 `SetProperty<T>(ref T, T, bool, string)` 方法，

![[学习笔记_05_02.png]]

额外参数 `bool` 设置为 `true` 时表示在属性更新时会验证该属性。`ObservableValidator` 将自动在每个新值（属性上方应用了 attribute 指定的）上进行验证。接着，其他组件（如 UI 控件）可以与 viewmodel 交互，并修改状态来反映当前存在于 viewmodel 中的错误，方法是通过注册到 `ErrorsChanged` 并使用 `GetErrors(string)` 方法检索已被修改的每个属性的错误列表。

### 2.3. 自定义验证方法

有时验证一个属性需要 viewmodel 去访问其他的服务、数据或 API。这边提供了多种方法向属性添加自定义验证，使用哪种方法取决于场景和灵活度需求。下面有个示例，它说明如何使用 `[CustomValidationAttribute]` 类型来指示调用特定方法进行属性的额外验证：

```csharp
public class RegistrationForm : ObservableValidator
{
    private readonly IFancyService service;

    public RegistrationForm(IFancyService service)
    {
        this.service = service;
    }

    private string name;

    [Required]
    [MinLength(2)]
    [MaxLength(100)]
    [CustomValidation(typeof(RegistrationForm), nameof(ValidateName))]
    public string Name
    {
        get => this.name;
        set => SetProperty(ref this.name, value, true);
    }

    public static ValidationResult ValidateName(string name, ValidationContext context)
    {
        RegistrationForm instance = (RegistrationForm)context.ObjectInstance;
        bool isValid = instance.service.Validate(name);

        if (isValid)
        {
            return ValidationResult.Success;
        }

        return new("The name was not validated by the fancy service");
    }
}
```

在本例中，有一个静态的 `ValidateName` 方法，它通过注入到 viewmodel 中的服务对 `Name` 属性执行验证（依赖注入章节有介绍）。该方法接收 `name` 属性值和使用中的 `ValidationContext` 实例为参数，其中包含 viewmodel 实例、正在验证的属性的名称、可选的服务提供者和一些我们使用或设置的自定义标志。在本例中，我们从 validation 上下文中检索 `RegistrationForm` 实例，然后从那使用注入的服务来验证属性。注意，该验证被执行，在其他 attribute 中指定的验证之后，所以我们可以自由组合自定义验证方法和现有的验证 attribute。

> 上面代码示例中的 Name 属性上有一串 [] attribute，验证可以一个个排下去执行。

### 2.4. 自定义验证 attribute

另一种自定义验证的方式就是实现一个自定义的 `[ValidationAttribute]` ，然后将验证逻辑插入重写的 `IsValid` 方法中。与上面的方法相比，这提供了额外的灵活性，因为它可以很容易地在多个地方重用相同的 attribute。

假设我们希望根据属性关于同一 viewmodel 中的另一个属性的相对值来验证一个属性。首先应该定义一个自定义 `[GreaterThanAttribute]` ，如下所示：

```csharp
public sealed class GreaterThanAttribute : ValidationAttribute
{
    public GreaterThanAttribute(string propertyName)
    {
        PropertyName = propertyName;
    }

    public string PropertyName { get; }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        object
            instance = validationContext.ObjectInstance,
            otherValue = instance.GetType().GetProperty(PropertyName).GetValue(instance);

        if (((IComparable)value).CompareTo(otherValue) > 0)
        {
            return ValidationResult.Success;
        }

        return new("The current value is smaller than the other one");
    }
}
```

现在，我们可以将该 attribute 添加到 viewmodel 中了：

```csharp
public class ComparableModel : ObservableValidator
{
    private int a;

    [Range(10, 100)]
    [GreaterThan(nameof(B))]
    public int A
    {
        get => this.a;
        set => SetProperty(ref this.a, value, true);
    }

    private int b;

    [Range(20, 80)]
    public int B
    {
        get => this.b;
        set
        {
            SetProperty(ref this.b, value, true);
            ValidateProperty(A, nameof(A));
        }
    }
}
```

本例中，有两个数字属性，它们必须在指定范围内，并且彼此间具有特定的关系（A 需要大于 B）。我们已经在第一个属性上添加了新的 `[GreaterThanAttribute]` ，并且在 B 的 `Setter` 中添加了对 `ValidateProperty` 的调用，这样每当 B 改变时，a 都会再次被验证（因为它的验证依赖于 b）。我们只需要 viewmodel 中的这两行代码来启动这个自定义验证，并且我们还获得了一个可重用的自定义验证 attribute 的好处，这个 attribute 在应用程序的其他 viewmodel 中也有用。这种方法还有助于代码模块化，因为验证逻辑现在完全与 viewmodel 定义本身解耦了。

## 三、结尾

验证器的代码并不复杂，属于应用型的内容，上面几小节中的示例直接拷贝出来改一改就能起作用，相当于是对 `ObservableObject` 的扩展。

---

# CommunityToolkit.Mvvm 学习笔记（6）—— RelayCommand

已于 2023-02-07 13:25:59 修改

## 一、前言

由于项目时间比较紧，所以先拣使用频繁的模块学习了。这篇先看命令 `RelayCommand`，毕竟 WPF 中命令与变量的绑定是两大主要绑定。如果说属性绑定是向 UI 暴露数据，那 `Command` 就是向 UI 暴露方法（或者说逻辑）。

> Notes:
> 这边说的暴露是解耦的，不管你绑定的变量或者命令是否存在，界面都能独立运行。

![[学习笔记_06_01.png]]

## 二、RelayCommand

### 2.1. 概述

MVVM Toolkit 中的 `RelayCommand` 和 `RelayCommand <T>` 都是实现了 `ICommand` 的类，

![[学习笔记_06_02.png]]

它们能够将方法或者委托暴露给 View（MVVM 中的 V，UI 界面）。这些类型作为在 viewmodel 和 UI 元素之间绑定命令的一种方式。

平台相关 APIs: `RelayCommand`, `RelayCommand<T>`, `IRelayCommand`, `IRelayCommand<T>`

### 2.2. 它们是如何工作的

`RelayCommand` 和 `RelayCommand<T>` 有以下主要特性：

- 提供了实现了 `ICommand` 接口的基类
- 同时也实现了 `IRelayCommand`（和 `IRelayCommand<T>`）接口，这使得它们暴露了 `NotifyCanExecuteChanged` 方法用以激发 `CanExecuteChanged` 事件
- 暴露了接收 `Action` 和 `Func<T>` 等委托的构造函数，这些构造函数允许封装标准方法和 lambda 表达式。
  ![[学习笔记_06_03.png]]

### 2.3. 简单使用无参命令

以下示例展示了如何使用一个简单的 `Command`：

```csharp
public class MyViewModel : ObservableObject
{
    public MyViewModel()
    {
        IncrementCounterCommand = new RelayCommand(IncrementCounter);
    }

    private int counter;

    public int Counter
    {
        get => counter;
        private set => SetProperty(ref counter, value);
    }

    public ICommand IncrementCounterCommand { get; }

    private void IncrementCounter() => Counter++;
}
```

关联的 UI 部分代码是这样的（使用 WinUI XAML）：

```xml
<Page
    x:Class="MyApp.Views.MyPage"
    xmlns:viewModels="using:MyApp.ViewModels">
    <Page.DataContext>
        <viewModels:MyViewModel x:Name="ViewModel"/>
    </Page.DataContext>

    <StackPanel Spacing="8">
        <TextBlock Text="{x:Bind ViewModel.Counter, Mode=OneWay}"/>
        <Button
            Content="Click me!"
            Command="{x:Bind ViewModel.IncrementCounterCommand}"/>
    </StackPanel>
</Page>
```

`Button` 绑定了 ViewModel 中的 `ICommand`，它封装了私有的 `IncrementCounter` 方法。`TextBlock` 显示 `Counter` 属性的值，并在每次属性值更改时更新它。

稍微分析一下上面的示例：

一个 MyViewModel 的类继承了 `ObservableObject`，从前面章节可知（甚至从命名上也可知），这个类就是用在 ViewModel 模块中，向 UI 提供数 `ObservableObject` 就是充当了通知类的基类）。
ViewModel 类的 `Counter` 属性封装了一个 `int`，前台（指 XAML 写的 UI 部分）的 `TextBlock` 的 `Text` 绑定了该属性。
`MyViewModel` 类中还暴露了一个 `IncrementCounterCommand` 命令，在构造函数中以 `IncrementCounter` 为参数进行初始化，并由前台 `Button` 的 `Command` 进行绑定。

### 2.4. 使用带参命令

以下示例展示了如何使用带参的 `Command` 来实现一个简单的登录界面：
ViewModel 中的主要代码：

```csharp
namespace LoginDemo
{
    internal class MainWindowViewModel : ObservableObject
    {
        private string _userName;
        private RelayCommand<PasswordBox> _loginCommand;
        public MainWindowViewModel()
        {
            UserName = "admin";
        }
        public string UserName
        {
            get => _userName;
            set => SetProperty(ref _userName, value);
        }
        private void Login(PasswordBox p)
        {
            MessageBox.Show(string.Format("UserName:{0}\nPassword:{1}", UserName, p.Password));
        }
        public RelayCommand<PasswordBox> LoginCommand
        {
            get
            {
                if (_loginCommand == null)
                {
                    _loginCommand = new RelayCommand<PasswordBox>(Login, (p) =>
                    {
                        return true;
                    });
                }
                return _loginCommand;
            }
        }
    }
}
```

在 viewmodel 中你要做的就是让类继承 `ObservableObject`，使它具有通知能力。然后我在里面暴露了一个 `UserName` 属性，来供前台绑定。并且公开了一个带参数（PasswordBox）的命令。
以下是 XAML 中的代码：

```xml
<Window x:Class="LoginDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:LoginDemo"
        mc:Ignorable="d"
        Title="用户登录" Height="250" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <StackPanel Orientation="Horizontal" VerticalAlignment="Center" HorizontalAlignment="Center">
            <TextBlock Text="用户名:"/>
            <TextBox Width="200" Text="{Binding UserName}"/>
        </StackPanel>
        <StackPanel Orientation="Horizontal" VerticalAlignment="Center" HorizontalAlignment="Center" Grid.Row="1">
            <TextBlock Text="密码:"/>
            <PasswordBox x:Name="passwordBox" Width="200"/>
        </StackPanel>
        <Button Grid.Row="2" Width="200" Content="登录" VerticalAlignment="Center"
                Command="{Binding LoginCommand}" CommandParameter="{Binding ElementName=passwordBox}"/>
    </Grid>
</Window>
```

看似繁杂，其实核心内容很简单，就是一个 `grid` 中放了三个控件，一个 `TextBox` 作用户名输入框，一个 `PasswordBox` 作密码输入框，和一个确定登录的按钮。剩下的代码都是起修饰样式和绑定的作用的。
XAML 的后台（.cs）：

```csharp
namespace LoginDemo
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            this.DataContext = new MainWindowViewModel();
        }
    }
}
```

这是一个标准的 XAML 后台，给 `DataContext` 赋上 viewmodel 对象即可，其它什么都不用做。
运行效果：

![[学习笔记_06_04.png]]

现在主要来关心以下部分，

```csharp
_loginCommand = new RelayCommand<PasswordBox>(Login, (p) =>
                    {
                        return true;
                    });
```

该 `RelayCommand` 命令初始化时用到了两个参数的构造函数，

![[学习笔记_06_05.png]]

第一个参数是个 `Action`，当然它本质上是个委托，一般表示着执行业务逻辑的方法，显然我在此处也传了符合该场景的 `Login` 函数进去（虽然其内容只是打印了用户名和密码，实际使用中你可以在里面添加一切你想要的逻辑，比如查询数据库，能否找到相匹配的用户名和密码）。第二个参数同样本质是个委托，用以检验控件是否可用，这边我使用了 lambda 表达式，常返回 `true`。而我们通过给 `<T>` 泛型赋值来使得命令支持传参。如果你需要使用多个参数，你可以给 `<T>` 赋复合类型值，将通过复合值来传递多个参数。

接着，你需要在 XAML 中绑定 viewmodel 暴露的命令，

```xml
Command="{Binding LoginCommand}" CommandParameter="{Binding ElementName=passwordBox}"
```

只要你有绑定基础，我相信这一点都不陌生。与不带参命令不同的是，这里多了一个 `CommandParameter` 的属性，并且绑定了 `passwordBox` 整个对象。

剩下的，你可以在 XAML 中对界面进行一些美化，而不需要修改后台的代码。

![[学习笔记_06_06.png]]

## 三、小结

`RelayCommand` 的使用还是比较简单的，本文只是做了一个应用上的简单介绍。但其内部机制却挺复杂，有兴趣的可以看看 WPF 原生的 `ICommand` 是如何实现自定义的。各种框架本质上就是实现了一个自定义 `ICommand` 来使用。

---

# CommunityToolkit.Mvvm 学习笔记（7）——Ioc 控制反转

已于 2023-02-06 09:29:13 修改

## 一、引言

由于项目比较忙，所以很久没有学习 Mvvm 工具包了。
那为什么突然又学了呢，因为项目中遇到了一个问题——不同的 ViewModel 间该如何通信？
不过我并不确定 Ioc 与这个问题有什么关系，只是感觉可能有关系。在学之前，我稍微搜了下 Ioc 是什么，看到了它的一个齿轮模型，觉得很有意思，所以还是偷了个闲来学习了。

## 二、Ioc（Inversion of control）

### 0. Ioc 的形象理解

我先来解释一下 Ioc 是啥，然后再去学习官方文档的东西。
Ioc 是 Inversion Of Control 的缩写，直译过来就是控制反转。
这是什么奇怪的名字。
从一个齿轮例子说起，

> 为什么是齿轮呢？可能是大家都喜欢造轮子吧。
> 如果把程序看成一个现实中运转的机械系统，那你写的代码模块就像系统中转动的齿轮一样。

![[学习笔记_07_01.png]]

假设这三个齿轮组成了一个系统,显然 A 齿轮的转动会带动小齿轮 B、C。
如果把该齿轮系统看作程序，这说明什么？
说明 A 和 B、C 之间存在**依赖关系**。再稍微专业一点就是，模块间存在**耦合**。
要使整个系统运转起来，就要一个齿轮转动来带动其他齿轮转动。先转动的那个齿轮掌握着主动权，即控制权。
模块间存在耦合不可避免，但实际开发中，我们往往更喜欢低耦合，这就得解耦。
所以我们能不能使这几个齿轮分开，达到解耦呢？
如果直接分开，一个齿轮转动就带动不了其他齿轮了，那系统就运转不起来了。
现在，我们可以在这几个齿轮之间再加入一个齿轮。

![[学习笔记_07_02.png]]

所以现在是 A 转动带动 D，再由 D 带动 B 和 C？
如果是这样，那还不如 A 直接带动 B 和 C 呢。
新加入的齿轮 D 其实是第三方的东西，在你的程序里你可以认为它是一个框架提供的机制或功能。
它才是一直在转动的一方。
D 一直在转，你需要 A 时，就把 A 装到 D 上去，需要 B 时就把 B 装上去。这样你需要的功能就会跟着“转起来”。（要啥装啥，且不需要三者直接相关了，这样 A、B、C 之间不就解耦了么？不过这只是一个粗略的理解，具体怎样还得往后看）
对比第一张图的情况，原来是由你的齿轮先转动，带动其他几个齿轮。
现在你的齿轮都是被动的一方，都是由这个第三方的齿轮所带动的。
到这里，控制反转的意思应该呼之欲出了。主动权（或者说控制权）发生了转变。你由主动方变成了被动方。
有了上面的基本理解之后，来正式学习下 Ioc。

### 1. Ioc

Ioc 是一种常见的模式，用来增强（使用 MVVM 模式的）程序的模块性。

> 增加模块化程度的，并且是适用于使用 MVVM 的程序的。不要啥程序都 Ioc。

Ioc 最常见的实现方式是用依赖注入（DI，Dependency Injection），实现思路是创建大量的服务，并将服务注入到后端类中（如，作为参数传给 ViewModel 构造函数）——这使得使用这些服务的代码不再依赖于服务的实现细节，并能很容易替换服务的具体实现。

> 使用 DI，模块间不再直接依赖，它们之间的代码不再耦合在一起。你更换一侧服务的细节实现，那调用侧并不用做更改，因为对它来说，我还是调用这个服务，至于内部发生了什么变化并不关心。

这种模式还可以很容易地将平台特有的特性提供给后端代码，方法是通过服务抽象它们，然后注入到需要的地方。

MVVM 工具包不提供内置的 API 来促进该模式使用，因为已经存在针对这种模式的专用库了，比如 `Microsoft.Extensions.DependencyInjection` 包，该包提供了一个功能齐全、功能强大的 DI API 集，并通过一个易于安装和使用的工具—— `IServiceProvider` 起作用。下面介绍会引用该库，并提供一系列示例来说明如何将它集成到 MVVM 模式的程序中。

> API 集：[Ioc](https://docs.microsoft.com/en-us/dotnet/api/Microsoft.Toolkit.Mvvm.DependencyInjection.Ioc?view=win-comm-toolkit-dotnet-7.0)

### 2. 配置和解析服务

第一步是声明一个 `IServiceProvider` 实例，并初始化所有需要的服务，通常是在程序启动时做这些事。例如，在 UWP 上（类似设置也可用于其他框架）：

```csharp
public sealed partial class App : Application
{
    public App()
    {
        Services = ConfigureServices();

        this.InitializeComponent();
    }

    /// <summary>
    /// Gets the current <see cref="App"/> instance in use
    /// </summary>
    public new static App Current => (App)Application.Current;

    /// <summary>
    /// Gets the <see cref="IServiceProvider"/> instance to resolve application services.
    /// </summary>
    public IServiceProvider Services { get; }

    /// <summary>
    /// Configures the services for the application.
    /// </summary>
    private static IServiceProvider ConfigureServices()
    {
        var services = new ServiceCollection();

        services.AddSingleton<IFilesService, FilesService>();
        services.AddSingleton<ISettingsService, SettingsService>();
        services.AddSingleton<IClipboardService, ClipboardService>();
        services.AddSingleton<IShareService, ShareService>();
        services.AddSingleton<IEmailService, EmailService>();

        return services.BuildServiceProvider();
    }
}
```

本例中，`Services` 属性在启动时被初始化，并且所有的应用程序服务和 viewmodel 都被注册。还有一个新的 `Current` 属性，该属性可用于从程序的其他 view 中轻松访问 `Services` 属性。例如：

```csharp
IFilesService filesService = App.Current.Services.GetService<IFilesService>();

// Use the files service here...
```

这里关键的点是，每个服务都能很好地使用特定于平台的 API，但由于这些 API 都是通过我们代码使用的接口抽象出来的，在解析实例和用它执行操作时，所以我们不需要关心它们。

### 3. 构造函数注入

还有一个强大的特性是“构造函数注入（constructor injection）”，这意味着 DI（依赖注入）服务提供程序能在创建被请求类的实例时自动解析已注册服务之间的间接依赖关系。考虑以下服务：

```csharp
public class FileLogger : IFileLogger
{
    private readonly IFilesService FileService;
    private readonly IConsoleService ConsoleService;

    public FileLogger(
        IFilesService fileService,
        IConsoleService consoleService)
    {
        FileService = fileService;
        ConsoleService = consoleService;
    }

    // Methods for the IFileLogger interface here...
}
```

这里我们有一个实现了 `IFileLogger` 接口的 `FileLogger` 类，并且需要 `IFilesService` 和 `IConsoleService` 实例。构造函数注入意味着 DI 服务提供程序会自动收集所有必要的服务，就像这样：

```csharp
/// <summary>
/// Configures the services for the application.
/// </summary>
private static IServiceProvider ConfigureServices()
{
    var services = new ServiceCollection();

    services.AddSingleton<IFilesService, FilesService>();
    services.AddSingleton<IConsoleService, ConsoleService>();
    services.AddSingleton<IFileLogger, FileLogger>();

    return services.BuildServiceProvider();
}

// Retrieve a logger service with constructor injection
IFileLogger fileLogger = App.Current.Services.GetService<IFileLogger>();
```

DI 服务提供程序会自动检查是否已经注册所有需要的服务，然后会检索它们并调用已注册的 IFileLogger 具体类的构造函数，来获取返回的实例。

### 4. ViewModel

服务提供程序的名称中有“服务”两字，但实际上，它可以用来解析任何类的实例，包括 ViewModel！上文提到的概念仍然适用，包括构造函数注入。假设我们有一个 `ContactsViewModel` 类，通过它的构造函数使用一个 `IContactsService` 和 `IPhoneService` 实例。我们可以用一个 `ConfigureServices` 方法，就像这样：

```csharp
/// <summary>
/// Configures the services for the application.
/// </summary>
private static IServiceProvider ConfigureServices()
{
    var services = new ServiceCollection();

    // Services
    services.AddSingleton<IContactsService, ContactsService>();
    services.AddSingleton<IPhoneService, PhoneService>();

    // Viewmodels
    services.AddTransient<ContactsViewModel>();

    return services.BuildServiceProvider();
}
```

接着在 `ContactsView` 中，分配如下的数据上下文：

```csharp
public ContactsView()
{
    this.InitializeComponent();
    this.DataContext = App.Current.Services.GetService<ContactsViewModel>();
}
```

## 三、结尾

本文中应用型的内容不多，几个基本用法的示例很难让你对（Ioc）DI 的应用场景及具体用法有较清晰的认识。
因为在 MVVM Toolkit 的官方文档里对 DI 的描述就是这么简略（这本来就是讲 Ioc 的章节），想要更深入了解 DI 的应用，得看 DI 的专门章节。
所以我觉得本文把 Ioc 的概念给弄懂就好，并且几个基本示例可以再结合网上别人的例子看看具体如何使用，不至于之后正式用时感到陌生。
