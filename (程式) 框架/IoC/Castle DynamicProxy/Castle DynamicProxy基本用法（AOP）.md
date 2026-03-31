---
aliases:
date: 2019-06-02
update: 2019-07-24
author: 拓荒者IT
language: CSharp
sourceurl: https://www.cnblogs.com/youring2/p/10962573.html
tags:
  - CSharp
  - IoC
  - AOP
  - Autofac
  - DynamicProxy
---

# Castle DynamicProxy 基本用法（AOP）

本文介绍 AOP 编程的基本概念、Castle DynamicProxy（DP）的基本用法，使用第三方扩展实现对异步（async）的支持，结合 Autofac 演示如何实现 AOP 编程。

## AOP

百科中关于 AOP 的解释：

> AOP 为 Aspect Oriented Programming 的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP 是 OOP 的延续，是软件开发中的一个热点……是函数式编程的一种衍生范型。利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

在 AOP 中，我们关注横切点，将通用的处理流程提取出来，我们会提供系统通用功能，并在各业务层中进行使用，例如日志模块、异常处理模块等。通过 AOP 编程实现更加灵活高效的开发体验。

## DynamicProxy 的基本用法

动态代理是实现 AOP 的一种方式，即在开发过程中我们不需要处理切面中（日志等）的工作，而是在运行时，通过动态代理来自动完成。Castle DynamicProxy 是一个实现动态代理的框架，被很多优秀的项目用来实现 AOP 编程，EF Core、Autofac 等。

我们来看两段代码，演示 AOP 的好处。在使用 AOP 之前：

```csharp
public class ProductRepository ： IProductRepository
{
    private readonly ILogger logger;
    
    public ProductRepository(ILogger logger)
    {
        this.logger = logger;
    }
        
    public void Update(Product product)
    {
        //执行更新操作
        //......
 
        //记录日志
        logger.WriteLog($"产品{product}已更新");
    }
}
```

在使用 AOP 之后：

```csharp
public class ProductRepository ： IProductRepository
{
    public void Update(Product product)
    {
        //执行更新操作
        //......
    }
}
```

可以明显的看出，在使用之前我们的 ProductRepository 依赖于 ILogger，并在执行 Update 操作以后，要写出记录日志的代码；而在使用之后，将日志记录交给动态代理来处理，降低了不少的开发量，即使遇见略微马虎的程序员，也不耽误我们日志的记录。

那该如何实现这样的操作呢？

- 首先，引用 `Castle.Core`
- 然后，定义拦截器，实现 `IInterceptor` 接口

```csharp
public class LoggerInterceptor : IInterceptor
{
    private readonly ILogger logger;
 
    public LoggerInterceptor(ILogger logger)
    {
        this.logger = logger;
    }
 
    public void Intercept(IInvocation invocation)
    {
        //获取执行信息
        var methodName = invocation.Method.Name;
 
        //调用业务方法
        invocation.Proceed();
 
        //记录日志
        this.logger.WriteLog($"{methodName} 已执行");
    }
}
```

- 最后，添加调用代码

```csharp
static void Main(string[] args)
{
    ILogger logger = new ConsoleLogger();
    Product product = new Product() { Name = "Book" };
    IProductRepository target = new ProductRepository();
 
    ProxyGenerator generator = new ProxyGenerator();
 
    IInterceptor loggerIntercept = new LoggerInterceptor(logger);
    IProductRepository proxy = generator.CreateInterfaceProxyWithTarget(target, loggerIntercept);
    
    proxy.Update(product);
}
```

至此，我们已经完成了一个日志拦截器，其它业务需要用到日志记录的时候，也可通过创建动态代理的方式来进行 AOP 编程。

但是，调用起来还是比较复杂，需要怎么改进呢？当然是使用依赖注入（DI）了。

## Autofac 的集成

Autofac 集成了对 DynamicProxy 的支持，我们需要引用 `Autofac.Extras.DynamicProxy`，然后创建容器、注册服务、生成实例、调用方法，我们来看下面的代码：

```csharp
ContainerBuilder builder = new ContainerBuilder();
//注册拦截器
builder.RegisterType<LoggerInterceptor>().AsSelf();
 
//注册基础服务
builder.RegisterType<ConsoleLogger>().AsImplementedInterfaces();
 
//注册要拦截的服务
builder.RegisterType<ProductRepository>().AsImplementedInterfaces()
    .EnableInterfaceInterceptors()                  //启用接口拦截
    .InterceptedBy(typeof(LoggerInterceptor));      //指定拦截器
 
var container = builder.Build();
 
//解析服务
var productRepository = container.Resolve<IProductRepository>();
 
Product product = new Product() { Name = "Book" };
productRepository.Update(product);
```

对这段代码做一下说明：

- 注册拦截器时，需要注册为 `AsSelf`，因为服务拦截时使用的是拦截器的实例，这种注册方式可以保证容器能够解析到拦截器。
- 开启拦截功能：注册要拦截的服务时，需要调用 `EnableInterfaceInterceptors` 方法，表示开启接口拦截；
- 关联服务与拦截器：`InterceptedBy` 方法传入拦截器，指定拦截器的方式有两种，一种是我们代码中的写法，对服务是无入侵的，因此推荐这种用法。另一种是通过 `Intercept` 特性来进行关联，例如我们上面的代码可以写为 `ProductRepository` 类上添加特性 `[Intercept(typeof(LoggerInterceptor))]`
- 拦截器的注册，可以注册为类型拦截器，也可以注册为命名的拦截器，使用上会有一些差异，主要在拦截器的关联上，此部分可以参考 Autofac 官方文档。我们示例用的是类型注册。
- 拦截器只对公共的接口方法、类中的虚方法有效，使用时需要特别注意。

## DynamicProxy 的基本原理

上面我们说到动态代理只对公共接口方法、类中的虚方法生效，你是否想过为什么？

其实，动态代理是在运行时为我们动态生成了一个代理类，通过 `Generator` 生成的时候返回给我们的是代理类的实例，而只有接口中的方法、类中的虚方法才可以在子类中被重写。

如果不使用动态代理，我们的代理服务应该是什么样的呢？来看下面的代码，让我们手工创建一个代理类：

_以下是我对代理类的理解，请大家辩证的看待，如果存在不正确的地方，还望指出。_

**为接口使用代理：**

```csharp
public class ProductRepositoryProxy : IProductRepository
{
    private readonly ILogger logger;
    private readonly IProductRepository target;
 
    public ProductRepositoryProxy(ILogger logger, IProductRepository target)
    {
        this.logger = logger;
        this.target = target;
    }
 
    public void Update(Product product)
    {
        //调用IProductRepository的Update操作
        target.Update(product);
 
        //记录日志
        this.logger.WriteLog($"{nameof(Update)} 已执行");
    }
}
 
//使用代理类
IProductRepository target = new ProductRepository();
ILogger logger = new ConsoleLogger();
IProductRepository productRepository = new ProductRepositoryProxy(logger, target);
```

**为类使用代理：**

```csharp
public class ProductRepository : IProductRepository
{
    //改写为虚方法
    public virtual void Update(Product product)
    {
        //执行更新操作
        //......
    }
}
 
public class ProductRepositoryProxy : ProductRepository
{
    private readonly ILogger logger;
 
    public ProductRepositoryProxy(ILogger logger)
    {
        this.logger = logger;
    }
 
    public override void Update(Product product)
    {
        //调用父类的Update操作
        base.Update(product);
        //记录日志
        this.logger.WriteLog($"{nameof(Update)} 已执行");
    }
}
 
//使用代理类
ILogger logger = new ConsoleLogger();
ProductRepository productRepository = new ProductRepositoryProxy(logger);
```

## 异步（async/await）的支持

如果你站在应用程序的角度来看，异步只是微软的一个语法糖，使用异步的方法返回结果为一个 Task 或 Task 的对象，这对于 DP 来说和一个 int 类型并无差别，但是如果我们想要在拦截中获取到真实的返回结果，就需要添加一些额外的处理。

`Castle.Core.AsyncInterceptor` 是帮我们处理异步拦截的框架，通过使用该框架可以降低异步处理的难度。

我们本节仍然结合 Autofac 进行处理，首先对代码进行改造，将 `ProductRepository.Update` 方法改为异步的。

```csharp
public class ProductRepository : IProductRepository
{
    public virtual Task<int> Update(Product product)
    {
        Console.WriteLine($"{nameof(Update)} Entry");
 
        //执行更新操作
        var task = Task.Run(() =>
        {
            //......
            Thread.Sleep(1000);
 
            Console.WriteLine($"{nameof(Update)} 更新操作已完成");
            //返回执行结果
            return 1;
        });
 
        //返回
        return task;
    }
}
```

接下来定义我们的异步拦截器：

```csharp
public class LoggerAsyncInterceptor : IAsyncInterceptor
{
    private readonly ILogger logger;
 
    public LoggerAsyncInterceptor(ILogger logger)
    {
        this.logger = logger;
    }
 
    /// <summary>
    /// 同步方法拦截时使用
    /// </summary>
    /// <param name="invocation"></param>
    public void InterceptSynchronous(IInvocation invocation)
    {
        throw new NotImplementedException(); 
    }
 
    /// <summary>
    /// 异步方法返回Task时使用
    /// </summary>
    /// <param name="invocation"></param>
    public void InterceptAsynchronous(IInvocation invocation)
    {
        throw new NotImplementedException();
    }
 
    /// <summary>
    /// 异步方法返回Task<T>时使用
    /// </summary>
    /// <typeparam name="TResult"></typeparam>
    /// <param name="invocation"></param>
    public void InterceptAsynchronous<TResult>(IInvocation invocation)
    {
        //调用业务方法
        invocation.ReturnValue = InternalInterceptAsynchronous<TResult>(invocation);
    }
 
    private async Task<TResult> InternalInterceptAsynchronous<TResult>(IInvocation invocation)
    {
        //获取执行信息
        var methodName = invocation.Method.Name;
 
        invocation.Proceed();
        var task = (Task<TResult>)invocation.ReturnValue;
        TResult result = await task;
 
        //记录日志
        this.logger.WriteLog($"{methodName} 已执行，返回结果：{result}");
 
        return result;
    }
}
```

`IAsyncInterceptor` 接口是异步拦截器接口，它提供了三个方法：

- `InterceptSynchronous`：拦截同步执行的方法
- `InterceptAsynchronous`：拦截返回结果为 Task 的方法
- `InterceptAsynchronous<TResult>`：拦截返回结果为 Task 的方法

在我们上面的代码中，只实现了 `InterceptAsynchronous<TResult>` 方法。

由于 `IAsyncInterceptor` 接口和 DP 框架中的 `IInterceptor` 接口没有关联，所以我们还需要一个同步拦截器，此处直接修改旧的同步拦截器：

```csharp
public class LoggerInterceptor : IInterceptor
{
    private readonly LoggerAsyncInterceptor interceptor;
    public LoggerInterceptor(LoggerAsyncInterceptor interceptor)
    {
        this.interceptor = interceptor;
    }
 
    public void Intercept(IInvocation invocation)
    {
        this.interceptor.ToInterceptor().Intercept(invocation);
    }
}
```

从代码中可以看到，异步拦截器 `LoggerAsyncInterceptor` 具有名为 `ToInterceptor()` 的扩展方法，该方法可以将 `IAsyncInterceptor` 接口的对象转换为 `IInterceptor` 接口的对象。

接下来我们修改 DI 的服务注册部分：

```csharp
ContainerBuilder builder = new ContainerBuilder();
//注册拦截器
builder.RegisterType<LoggerInterceptor>().AsSelf();
builder.RegisterType<LoggerAsyncInterceptor>().AsSelf();
 
//注册基础服务
builder.RegisterType<ConsoleLogger>().AsImplementedInterfaces();
 
//注册要拦截的服务
builder.RegisterType<ProductRepository>().AsImplementedInterfaces()
    .EnableInterfaceInterceptors()                  //启用接口拦截
    .InterceptedBy(typeof(LoggerInterceptor));      //指定拦截器
 
var container = builder.Build();
```

以上便是通过 `IAsyncInterceptor` 实现异步拦截器的方式。除了使用这种方式，我们也可以在在动态拦截器中判断返回结果手工处理，此处不再赘述。

## 探讨：ASP.NET MVC 中的切面编程

通过上面的介绍，我们已经了解了 AOP 的基本用法，但是如何用在 `ASP.NET Core` 中呢？

1. MVC 控制器的注册是在 Services 中完成的，而 Services 本身不支持 DP。这个问题可以通过整合 Autofac 重新注册控制器来完成，但是这样操作真的好吗？
2. MVC 中的控制器是继承自 ControllerBase，Action 方法是我们自定义的，不是某个接口的实现，这对实现 AOP 来说存在一定困难。这个问题可以通过将 Action 定义为虚方法来解决，但是这样真的符合我们的编码习惯吗？

我们知道，AOP 的初衷就是对使用者保持黑盒，通过抽取切面进行编程，而这两个问题恰恰需要我们对使用者进行修改，违背了 SOLID 原则。

那么，如果我们要在 MVC 中使用 AOP，有什么方法呢？其实 MVC 已经为我们提供了两种实现 AOP 的方式：

1. 中间件（Middleware），这是 MVC 中的大杀器，提供了日志、Cookie、授权等一系列内置的中间件，从中可以看出，MVC 并不想我们通过 DP 实现 AOP，而是要在管道中做文章。
2. 过滤器（Filter），Filter 是 ASP.NET MVC 的产物，曾经一度帮助我们解决了异常、授权等逻辑，在 Core 时代我们仍然可以采用这种方式。

这两种方式更符合我们的编码习惯，也体现了 MVC 框架的特性。

综上，不建议在 MVC 中对 Controller 使用 DP。如果采用 NLayer 架构，则可以在 Application 层、Domain 层使用 DP，来实现类似数据审计、SQL 跟踪等处理。

虽然不推荐，但还是给出代码，给自己多一条路：

- MVC 控制器注册为服务

```csharp
services.AddMvc()
    .AddControllersAsServices();
```

- 重新注册控制器，配置拦截

```csharp
builder.RegisterType<ProductController>()
    .EnableClassInterceptors()
    .InterceptedBy(typeof(ControllerInterceptor));
```

- 控制器中的 Action 定义为虚方法

```csharp
[HttpPost]
public virtual Task<int> Update(Product product)
{
    return this.productRepository.Update(product);
}
```

## 补充内容

- **2019 年 7 月 24 日补充**

在创建代理类时（无论是 class 或 interface），都有两种写法：WithTarget 和 WithoutTarget，这两种写法有一定的区别，withTarget 需要传入目标实例，而 withoutTarget 则不用，只需要传入类型即可。

## 参考文档

- [Castle Dynamic 官方文档](https://github.com/castleproject/Core/blob/master/docs/dynamicproxy.md)
- [Autofac Type Interceptors 官方文档](https://autofac.readthedocs.io/en/latest/advanced/interceptors.html)
- [使用Castle DynamicProxy（AOP）](https://www.cnblogs.com/chunjin/p/6801335.html)
- [框架学习与探究之AOP--Castle DynamicProxy](https://www.cnblogs.com/DjlNet/p/7603654.html)
- [异步支持库 Castle.Core.AsyncInterceptor](https://github.com/JSkimming/Castle.Core.AsyncInterceptor)
- [Cross Cutting Concerns in .NET Applications](https://www.codeproject.com/Articles/1059998/WebControls/)
