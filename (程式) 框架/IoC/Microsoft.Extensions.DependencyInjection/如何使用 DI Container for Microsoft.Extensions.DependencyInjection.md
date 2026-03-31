---
aliases:
date: 2023-10-22
update:
author: 余小章
language:
sourceurl: https://dotblogs.azurewebsites.net/yc421206/2020/10/29/standard_di_container_for_microsoft_extensions_dependencyInjection
tags:
  - CSharp
  - ASPdotNET
  - IoC
---

[`Microsoft.Extensions.DependencyInjection`](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection) 是微軟實作的 DI Container，在 ASP.NET Core 大量的使用，無形之間已經成為一種開發標準，過去，還沒有使用 DI Container 時，我會使用靜態屬性來集中管理物件的生命週期，現在，集中管理物件的生命週期，我又多了一個選擇。

Microsoft.Extensions.DependencyInjection 支援 .NET Fx 4.6.1 以上

# 相依、注入

- 一個 class A 需要另外一個 class B 才能完成功能，稱為 A **依賴**於 B。
- 根據物件導向的 [依賴倒置原則 (Dependency Inversion Principle, DIP)](https://notfalse.net/1/dip)，A 不應直接依賴 B，而是==依賴抽象== (Interface、Abstract)，目的是為了降低物件彼此之間的依賴。
- 在日常生活當中常見的抽象設計，例如：電腦主機板上的 CPU 插槽、RAM 插槽、燈具、插座…等把 B 宣告成欄位或屬性、方法參數，建構函數參數，讓使用 A 的人可以改變 B 的狀態，這種可以由外部改變內部狀態的行為，稱為**注入**。

> 開一個洞，可以讓外部可以改變內部狀態，稱為：DI (Dependency Injection)

> 本來是內部控制，由於反轉，外部也可以控制了；控制流程由內部控制變成外部控制，主動變成被動，稱為 [Ioc (Inversion of Control)](https://notfalse.net/3/ioc-di)：控制反轉；ASP.NET Core 實現了 IoC，預設就允許你這麼做

常見的開洞方式

- 建構函數參數
- 屬性
- 方法參數
- 檔案、資料庫

屬性注入的範例如下：

```csharp
internal static class Program
{
    [STAThread]
    private static void Main()
    {
        var member = new Member();
        member.Name = "小章";
    }
}

class Member
{
    public string Name { get; set; }="yao";
}
```

對！就是我們常在幹的事

# 開發環境

- Rider 2020.2
- ASP.NET Core 3.1
- ASP.NET MVC 5 / .NET Framework 4.8
- ASP.NET Web API / .NET Framework 4.8
- Microsoft.Extensions.DependencyInjection 3.1.9

# 服務在 DI Container 內的生命週期

當我們和 DI Container 請求服務實例時，它會根據當時註冊的依據來決定，[`Microsoft.Extensions.DependencyInjection`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection) DI Container 管理物件有三個生命週期，：

- **Singleton**：整個應用程式只建立一個實例 (Instance) 並且共用它。
- **Scoped**：在 ASP.NET Core，每個 Request 都會建立一個新的實例，在這當中經過的 Pipeline，都是相同的實例。Scope 指的就是這種特定範圍內使用相同的實例
- **Transient**：每次都建立一個新的，永不共用。

# 註冊服務到 DI Container，通過 IServiceCollection

容器框架將我們應用程式所需要功能視為「服務」，當我們需要把「服務」交給它處理；在這裡需要呼叫 `IServiceCollection.Add/TryAdd` 開頭的方法把服務放入容器內，簡單來講就是交給它來幫你做 new class 的動作以及取用，還有生命週期的管理。

- Add 會覆蓋原有的註冊定義
- Try 會先檢查是否存在註冊，存在則不處理

> 這裡的「服務」指的不是 WinService 或是一個網頁服務，而是一個 class

## 實例化 IServiceCollection

開始之前要先安裝以下套件

```text
Install-Package Microsoft.Extensions.DependencyInjection
```

首先要建立 `IServiceCollection` 實例，在 ASP.NET Core 在 `Host.CreateDefaultBuilder(args).ConfigureWebHostDefaults` 處理了，請參考 [如何使用 .NET Generic Host for Microsoft.Extensions.Hosting | 余小章 @ 大內殿堂 - 點部落 (dotblogs.com.tw)](https://dotblogs.com.tw/yc421206/2021/04/05/how_to_user_net_generic_for_host_microsoft_extensions_hosting)

.NET Framework / .NET Core 則是要自行處理，我整理出不同框架所需要的步驟。

### ASP.NET Core 3.1

ASP.NET Core 3.1 註冊的動作寫在 `Startup.cs` 的 `ConfigureServices(IServiceCollection services)` 方法，範例如下：

```csharp
public class Startup
{
    ...

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddTransient<ITransientMessager, MultiMessager>()
                .AddSingleton<ISingleMessager, MultiMessager>()
                .AddScoped<IScopeMessager, MultiMessager>()
            ;
    }
}
```

### WinForm .NET Framework 4.8

.NET Framework 則需要自己實例化 IServiceCollection，再註冊服務，最後調用 [`IServiceCollection.BuildServiceProvider`](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider?view=dotnet-plat-ext-3.1) 方法

```csharp
internal class DependencyInjectionConfig
{
    public static IServiceProvider ServiceProvider { get; set; }

    public static IServiceProvider Register()
    {
        var services = new ServiceCollection();
        ConfigureServices(services);
        ServiceProvider = services.BuildServiceProvider();
        return ServiceProvider;
    }

    /// <summary>
    ///     使用 MS DI 註冊
    /// </summary>
    private static IServiceCollection ConfigureServices(IServiceCollection services)
    {
        return services.AddSingleton<Form1>()
                       .AddTransient<Worker>()
                       .AddTransient<ITransientMessager, MultiMessager>()
                       .AddSingleton<ISingleMessager, MultiMessager>()
                       .AddScoped<IScopeMessager, MultiMessager>()      
    }
}
```

在進入點的 `Program.cs` 位置調用 `DependencyInjectionConfig.Register` 方法並取得 `ServiceProvider`

```csharp
internal static class Program
{
    /// <summary>
    ///     The main entry point for the application.
    /// </summary>
    [STAThread]
    private static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        var serviceProvider = DependencyInjectionConfig.Register() as ServiceProvider;

        using (serviceProvider)
        {
            var form = serviceProvider.GetService(typeof(Form1)) as Form;
            Application.Run(form);
        }
    }
}
```

### ASP.NET Web API 2

ASP.NET Web API 要使用 DI Container 注入 Controller 得實作解析器，這樣才能在 Controller 的建構函數注入參數。

*2021/05/12 根據小紀回復修正無法執行物件 `Dispose` 方法*

```csharp
public class DefaultDependencyResolver : IDependencyResolver
{
    private readonly IServiceProvider _serviceProvider;
    private          IServiceScope    _serviceScope;

    public DefaultDependencyResolver(IServiceProvider serviceProvider, IServiceScope serviceScope = null)
    {
        this._serviceProvider = serviceProvider;
        this._serviceScope    = serviceScope;
    }

    public object GetService(Type serviceType)
    {
        return this._serviceProvider.GetService(serviceType);
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        return this._serviceProvider.GetServices(serviceType);
    }

    public IDependencyScope BeginScope()
    {
        this._serviceScope = this._serviceProvider.CreateScope();
        return new DefaultDependencyResolver(this._serviceScope.ServiceProvider,this._serviceScope);
    }

    public void Dispose()
    {
        this._serviceScope?.Dispose();
    }
}
```

Controller 也算是應用程式的進入點，我們可以讓它依賴 Service Locator，通過 `HttpRequestMessage.GetDependencyScope` 擴充方法就能取得 `IDependencyScope`，取出物件的生命週期為 Scope

```csharp
var requestScope = this.Request.GetDependencyScope();
var transient    = requestScope.GetService(typeof(ITransientMessager)) as ITransientMessager;
```

你可以選擇是否要在 Controller 開一個注入點或是讓它依賴 Service Locator。

搜尋 Web API 的 Controller (`IHttpController`)

```csharp
public static class ServiceProviderExtensions
{
    public static IServiceCollection AddControllersAsServices(this IServiceCollection services,
                                                              IEnumerable<Type>       controllerTypes)
    {
        var filter = controllerTypes.Where(t => !t.IsAbstract
                                                && !t.IsGenericTypeDefinition)
                                    .Where(t => typeof(IHttpController).IsAssignableFrom(t)
                                                || t.Name.EndsWith("Controller", StringComparison.OrdinalIgnoreCase));

        foreach (var type in filter)
        {
            services.AddTransient(type);
        }

        return services;
    }
}
```

實例化 `IServiceCollection`，再註冊服務，最後調用 [`IServiceCollection.BuildServiceProvider`](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider?view=dotnet-plat-ext-3.1) 方法

```csharp
public class DependencyInjectionConfig
{
    public static void Register(HttpConfiguration config)
    {
        var services = ConfigureServices();

        var provider = services.BuildServiceProvider();

        var resolver = new DefaultDependencyResolver(provider);
        config.DependencyResolver = resolver;
    }

    /// <summary>
    ///     使用 MS DI 註冊
    /// </summary>
    /// <returns></returns>
    private static ServiceCollection ConfigureServices()
    {
        var services = new ServiceCollection();

        //使用 Microsoft.Extensions.DependencyInjection 註冊
        services.AddControllersAsServices(typeof(DependencyInjectionConfig).Assembly.GetExportedTypes());

        services.AddScoped<IMessager, LogMessager>();

        services.AddTransient<ITransientMessager, MultiMessager>()
                .AddSingleton<ISingleMessager, MultiMessager>()
                .AddScoped<IScopeMessager, MultiMessager>();
        return services;
    }
}
```

在應用程式 Global.asax 進入點，調用 `DependencyInjectionConfig.Register`

```csharp
public class WebApiApplication : HttpApplication
{
    protected void Application_Start()
    {
        GlobalConfiguration.Configure(WebApiConfig.Register);
        GlobalConfiguration.Configure(DependencyInjectionConfig.Register);
    }
}
```

### ASP.NET MVC 5

用 `ServiceScopeHttpModule` 將每個 Request 存放 `IServiceScope`，選用 `HttpContext.Current.Items` 來保留狀態，之前，我寫了一些有關 ASP.NET Request 可以使用的物件，有興趣的可以過去看一下 [[ASP.NET] 使用 Request 傳遞參數 | 余小章 @ 大內殿堂 - 點部落 (dotblogs.com.tw)](https://dotblogs.com.tw/yc421206/2020/07/06/asp_net_via_request_pass_parameter)

```csharp
internal class ServiceScopeHttpModule : IHttpModule
{
    private static ServiceProvider s_serviceProvider;

    public static void SetServiceProvider(IServiceProvider serviceProvider)
    {
        s_serviceProvider = serviceProvider as ServiceProvider;
    }

    public void Dispose()
    {
        s_serviceProvider?.Dispose();
    }

    public void Init(HttpApplication context)
    {
        context.BeginRequest += this.Context_BeginRequest;
        context.EndRequest   += this.Context_EndRequest;
    }

    private void Context_BeginRequest(object sender, EventArgs e)
    {
        var context = ((HttpApplication) sender).Context;
        context.Items[typeof(IServiceScope)] = s_serviceProvider.CreateScope();
    }

    private void Context_EndRequest(object sender, EventArgs e)
    {
        var context = ((HttpApplication) sender).Context;
        if (context.Items[typeof(IServiceScope)] is IServiceScope scope)
        {
            scope.Dispose();
        }
    }
}
```

`DefaultDependencyResolver` 則是取出 `IServiceScope`，這是跟 Web API 不一樣的地方

```csharp
internal class DefaultDependencyResolver : IDependencyResolver
{
    private readonly ServiceProvider _serviceProvider;

    public DefaultDependencyResolver(IServiceProvider serviceProvider)
    {
        this._serviceProvider = serviceProvider as ServiceProvider;
    }

    public void Dispose()
    {
        this._serviceProvider?.Dispose();
    }

    public object GetService(Type serviceType)
    {
        if (HttpContext.Current?.Items[typeof(IServiceScope)] is IServiceScope scope)
        {
            return scope.ServiceProvider.GetService(serviceType);
        }

        return this._serviceProvider.GetService(serviceType);
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        if (HttpContext.Current?.Items[typeof(IServiceScope)] is IServiceScope scope)
        {
            return scope.ServiceProvider.GetServices(serviceType);
        }

        return this._serviceProvider.GetServices(serviceType);
    }
}
```

最後在，Global.asax 調用 `ServiceScopeHttpModule`

```csharp
[assembly: PreApplicationStartMethod(typeof(MvcApplication), "InitModule")]
namespace Mvc5Net48
{
    public class MvcApplication : HttpApplication
    {
        public static void InitModule()
        {
            RegisterModule(typeof(ServiceScopeHttpModule));
        }

        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            DependencyInjectionConfig.Register();
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        }
    }
}
```

有一些代碼跟 Web API 很像，為了節省一些篇幅我就不貼代碼了

[`DependencyInjectionConfig.cs`](https://github.com/yaochangyu/sample.dotblog/blob/master/DI/Lab.MsDI/Mvc5Net48/App_Start/DependencyInjectionConfig.cs)
[`ServiceProviderExtensions.cs`](https://github.com/yaochangyu/sample.dotblog/blob/master/DI/Lab.MsDI/Mvc5Net48/App_Start/ServiceProviderExtensions.cs)

以上就是根據不同的框架準備的環境設定。

## 註冊

- 微軟的容器裡面有一個集合，用 Type 當 Key，一個 Key 對應到一個實體 (Instance)，預設可以不用自己寫 new
- 可以把他想成是一個 `Dictionary<Type,Object>`，註冊的時候把 Type，放進去，取出的時候用 Type，但其實沒簡單，DI Container 還要管理實體的生命週期。
- 「服務」註冊到容器後並不會立即實例化

以下我整理出幾個常用的註冊寫法

## 手動實例化 ServiceDescriptor 註冊

[`Add`](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.extensions.servicecollectiondescriptorextensions.add) 方法

```csharp
services.Add(new ServiceDescriptor(typeof(ISingleMessager),
             typeof(MultiMessager),
             ServiceLifetime.Singleton));
```

從 DI Container 取出時，key 為 `typeof(MultiMessager)`，例如：[`IServiceProvider.GetService(typeof(MultiMessager))`](https://learn.microsoft.com/zh-tw/dotnet/api/system.iserviceprovider.getservice)

## Add 兩個 Key，相同實例

`ServiceDescriptor` 有一個參數是 `Func<IServiceProvider, T>` 用來決定如何取得實例，Model 如下：

```csharp
public interface IModel
{
    Guid Id { get; set; }

    string Name { get; set; }
}

public class Model : IModel
{
    public Guid Id { get; set; } = Guid.NewGuid();

    public string Name { get; set; }
}
```

`descriptor1` 會產生 key=typeof(Model) 和 Model 實例

`descriptor2` 去 DI Container 取得 Model 實例，`implementFactory` 是委派，可以宣告從 DI Container 取出實例或是手動 `new`，有延遲執行的效果。

```csharp
Func<IServiceProvider, Model> implementFactory = p => p.GetService<Model>();

var descriptor1 = new ServiceDescriptor(typeof(Model), typeof(Model), ServiceLifetime.Scoped);
var descriptor2 = new ServiceDescriptor(typeof(IModel), implementFactory, ServiceLifetime.Scoped);
services.Add(new[] {descriptor1, descriptor2});

var provider = services.BuildServiceProvider();
var model1   = provider.GetService<Model>();
var model2   = provider.GetService<IModel>();
```

或者這樣用

```csharp
services.AddSingleton<Model>();
services.AddSingleton<IModel>(p => p.GetService<Model>());
```

執行結果如下
![[1609315735.png]]

## 通過 Fluent API / 方法鏈註冊

用來新增移除服務的 `IServiceCollection` 擴充方法 [`Microsoft.Extensions.DependencyInjection.Extensions`](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.extensions.servicecollectiondescriptorextensions) ，骨子裡面幫你實例化 `ServiceDescriptor`

指定實作型別

```csharp
services.AddSingleton<Form1>()
        .AddTransient<Worker>()
        ;
```

 取出時，key 為 `typeof(Form1)`，例如：[`IServiceProvider.GetService(typeof(Form1))`](https://learn.microsoft.com/zh-tw/dotnet/api/system.iserviceprovider.getservice)

抽象 (Interface) 對應實作

```csharp
services.AddTransient<ITransientMessager, MultiMessager>()
        .AddSingleton<ISingleMessager, MultiMessager>()
        .AddScoped<IScopeMessager, MultiMessager>()
        ;
```

 取出時，key 為抽象 `typeof(ITransientMessager)`，例如：[`IServiceProvider.GetService(typeof(ITransientMessager))`](https://learn.microsoft.com/zh-tw/dotnet/api/system.iserviceprovider.getservice)

自行建立實例

```csharp
services.AddSingleton(provider =>
                      {
                          var form1 = new Form1();
                          form1.Name = "今晚我想來點...";
                          return form1;
                      });
```

註冊時依賴別的 Service，從 `ServiceProvider` 取出 Service

```csharp
services.AddTransient(provider =>
                      {
                          var transient = provider.GetRequiredService<ITransientMessager>();
                          var single    = provider.GetRequiredService<ISingleMessager>();
                          var scope     = provider.GetRequiredService<IScopeMessager>();
                          return new Worker(transient,
                                            scope,
                                            single);
                      });
```

依賴相同的抽象，註冊不同的實作，依賴關係如下：

```csharp
public class Worker
{
    public IMessager Messager { get; set; }

    public Worker(IMessager messager)
    {
        this.Messager = messager;
    }
}

public class Worker2
{
    public IMessager Messager { get; set; }

    public Worker2(IMessager messager)
    {
        this.Messager = messager;
    }
}
```

註冊不同的的實作

```csharp
services.AddTransient(p => new Worker(new LogMessager()));
services.AddTransient(p => new Worker2(new MachineMessager()));
```

## 註冊注意事項

- 每一種生命週期只能註冊一次，重複註冊會以最後一次註冊的為主
- 生命週期長的不能依賴生命週期短的
- 不同的 Key 需要相同實例，不應該直接寫兩次 Add，可透過 [`ServiceDescriptor(Type, Func<IServiceProvider,Object>, ServiceLifetime`)](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor.-ctor) 處理，不要這樣寫

```csharp
//這樣寫會有兩個執行個體
services.AddScoped<Model>();
services.AddScoped<IModel, Model>();
```

- 手動實例化要寫在委派方法才會有延遲 Lazy 執行的效果，不要這樣寫

```csharp
var logger = new LogMessager();
services.AddTransient(p => new Worker(logger));
```

更多註冊服務的設定

[ASP.NET Core 控制器的相依性插入](https://learn.microsoft.com/zh-tw/aspnet/core/mvc/controllers/dependency-injection)
[ASP.NET Core 檢視中的相依性插入](https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/dependency-injection)

# 從 DI Container 取出服務實例，通過 ServiceProvider

.NET Framework 調用 `IServiceCollection.BuildServiceProvider()` 才能取得 `IServiceProvider` 實例

```csharp
var services = new ServiceCollection();
var provider = services.BuildServiceProvider();
```

有了 `IServiceProvider` 實例後才能透過他取出其他的服務，有以下幾種手動取出服務的方式

- [`IServiceProvider.GetService`](https://learn.microsoft.com/zh-tw/dotnet/api/system.iserviceprovider.getservice)：找不到服務會回傳 null
- [`IServiceProvider.GetRequiredService`](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice)：找不到服務會噴例外


有時會需要已經註冊過的服務，在 `IServiceCollection.AddXXX` 擴充方法裡面，有提供 `IServiceProvider` 參數，讓我們取出其他已經註冊過的服務

```csharp
services.AddTransient(provider =>
                      {
                          var transient = provider.GetRequiredService<ITransientMessager>();
                          var single    = provider.GetRequiredService<ISingleMessager>();
                          var scope     = provider.GetRequiredService<IScopeMessager>();
                          return new Worker(transient,
                                            scope,
                                            single);
                      });
```

使用 `IServiceProvider.CreateScope` 建立 Scope，`using` 區段內的服務只有一個實例

```csharp
using (var scope = serviceProvider.CreateScope())
{
    var scopedServiceProvider = scope.ServiceProvider;
    var myScopedService = scopedServiceProvider.GetService<IMyScopedService>();
}
```

不該讓你的物件直接依賴 `IServiceProvider (Service Locator pattern)`，==你的物件應該是透過注入的方式取得其他依賴的實體==；
但==可以集中一個地方或是應用程式的進入點來管理其他服務的注入==。


除了手動操作 `IServiceProvider` 之外，框架也實作了取出服務的擴充方法

## ASP.NET Web API 2

通過 `HttpRequestMessage.GetDependencyScope` 擴充方法取得服務，這是 ASP.NET 框架內實現的功能，讓我們在同一個 Request 使用相同實例的服務

在 Filter 的範例如下：

```csharp
public class LogFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(HttpActionContext actionContext)
    {
        IDependencyScope requestScope = actionContext.Request.GetDependencyScope();
        var transient                 = requestScope.GetService(typeof(ITransientMessager)) as MultiMessager;
        ...
    }
}
```

在 `ApiController` 的範例如下：

```csharp
public class DefaultController : ApiController
{
    [HttpGet]
    public IHttpActionResult Get()
    {
        IDependencyScope requestScope = this.Request.GetDependencyScope();
        var              transient    = requestScope.GetService(typeof(ITransientMessager)) as MultiMessager;
        ....
        return this.Ok();
    }
}
```

## ASP.NET MVC 5

通過 `IDependencyResolver.GetService` 方法取得實例，ASP.NET 框架內實現的功能，讓我們在同一個 `Request` 使用相同實例的服務

```csharp
public class Default1Controller : Controller
{
    // GET: Default
    public ActionResult Index()
    {
        var single    = this.Resolver.GetService(typeof(ISingleMessager)) as ISingleMessager;
        ...
        return this.View();
    }
}
```

通過反編譯得知 `Controller.Resolver` 屬性骨子裡面是 `System.Web.Mvc.DependencyResolver.Current`，所以也可以直接使用這個物件取得容器內的實例
![[1609006475.png]]

## ASP.NET Core

通過 `HttpContext.RequestServices` 取得 `IServiceProvider`，再透過 `IServiceProvider.GetService` 取得實例，ASP.NET Core 框架內實現的功能，讓我們在同一個 Request 使用相同實例的服務

```csharp
[ApiController]
[Route("[controller]")]
public class DefaultController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        IServiceProvider serviceProvider = this.HttpContext.RequestServices;
        var transient = serviceProvider.GetService(typeof(ITransientMessager)) as ITransientMessager;
        var scope = serviceProvider.GetService(typeof(IScopeMessager)) as IScopeMessager;
        var single = serviceProvider.GetService(typeof(ISingleMessager)) as ISingleMessager;
        ...
        return this.Ok();
    }
}
```

# 觀察服務的生命週期

了解 DI Container 相關物件的使用方式，接下來我想要觀察服務的生命週期是不是如我所設想的，比如，

- 宣告成 Transient 的物件，是不是每次都會產生一份新的實例?
- 宣告成 Scope 的物件，是不是在同一個 Request 的生命週期內，是不是同一個實例?
	- 不同的 Request 的實例均為不同?
- 宣告成 Singleton 的物件，是不是在應用程式的生命週期內，只有一份實例?

這裡我需要幾個物件

```csharp
public interface IMessager
{
    string OperationId { get; }
}
public interface IScopeMessager : IMessager
{
}
public interface ISingleMessager : IMessager
{
}
public interface ITransientMessager : IMessager
{
}
public class MultiMessager : IScopeMessager, ISingleMessager, ITransientMessager
{
    public string OperationId { get; } = $"多個接口-{Guid.NewGuid()}";
}
```

## ASP.NET Core 3.1

讓 Controller 依賴 `ITransientMessager`、`IScopeMessager`、`ISingleMessager`

```csharp
[ApiController]
[Route("[controller]")]
public class DefaultController : ControllerBase
{
    private IMessager Transient { get; }

    private IMessager Scope { get; }

    private IMessager Single { get; }

    private readonly ILogger<DefaultController> _logger;

    public DefaultController(ILogger<DefaultController> logger,
                             ITransientMessager         transient,
                             IScopeMessager             scope,
                             ISingleMessager            single)
    {
        this._logger = logger;

        this.Transient = transient;
        this.Scope     = scope;
        this.Single    = single;
    }

    [HttpGet]
    public IActionResult Get()
    {
        var content = "我在 DefaultController.Get \r\n"               +
                      $"transient:{this.Transient.OperationId}\r\n" +
                      $"scope:{this.Scope.OperationId}\r\n"         +
                      $"single:{this.Single.OperationId}";
        this._logger.LogInformation("我在 DefaultController.Get ,transient = {transient},scope = {scope},single = {single}",
                                    this.Transient.OperationId,
                                    this.Scope.OperationId,
                                    this.Single.OperationId);
        return this.Ok(content);
    }
}
```

分別註冊三種不同的生命週期的 `ITransientMessager`、`IScopeMessager`、`ISingleMessager`，對應到相同的 MultiMessager，用來觀察 MultiMessager 的生命週期是否有甚麼變化

```csharp
public class Startup
{
    ...other code
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();

        services.AddTransient(p => new Worker(new LogMessager()));
        services.AddTransient(p => new Worker2(new MachineMessager()));
 
        services.AddTransient<ITransientMessager, MultiMessager>()
                .AddSingleton<ISingleMessager, MultiMessager>()
                .AddScoped<IScopeMessager, MultiMessager>()
            ;
    }
}
```

 Controller 屬於 ASP.NET Core 框架生命週期的一部分，支援建構函數注入


按 Ctrl + F5 執行，跳出瀏覽器

第一次執行

```text
info: WebApiNetCore31.Controllers.DefaultController[0]       我在 DefaultController.Get ,transient = 多個接口-84f4ebe8-2fcf-436f-9dcf-cca33e23a76e,scope = 多個接口-240e5402-4ca5-40b9-89d6-2b58d98ac24d,single = 多個接口-0168818c-b016-4b80-91f7-30ce91c09159
```

對瀏覽器按 F5，第二次執行

```text
info: WebApiNetCore31.Controllers.DefaultController[0]       我在 DefaultController.Get ,transient = 多個接口-1e9e05f5-5ab6-41e4-a827-8c422ea8d937,scope = 多個接口-1d9e1066-c7a5-4c5a-8f8f-f75fab447082,single = 多個接口-0168818c-b016-4b80-91f7-30ce91c09159
```

對瀏覽器按 F5，第三次執行

```text
info: WebApiNetCore31.Controllers.DefaultController[0]       我在 DefaultController.Get ,transient = 多個接口-56e654e5-dca0-4935-b9a5-53a6e9fec201,scope = 多個接口-1bad6360-f31f-4d2d-9596-25b70fcea842,single = 多個接口-0168818c-b016-4b80-91f7-30ce91c09159
```

不管按了幾次 F5，註冊為 Singleton 的 MultiMessager 狀態都不會改變，但是看不太出來 Transient 及 Scoped 的差異，因為他們一直都在變。


為了要觀察生命週期，接下來我需要一個 Filter，讓這個 Request 的生命週期流過 Filter，Filter 是屬於 ASP.NET Core 生命週期的一部分，`ActionExecutingContext` 的物件也可以存取 `IServiceProvider`，當然 Controller 也可以

```csharp
public class LogFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        var transient = context.HttpContext.RequestServices.GetService<ITransientMessager>();
        var scope     = context.HttpContext.RequestServices.GetService<IScopeMessager>();
        var single    = context.HttpContext.RequestServices.GetService<ISingleMessager>();
        var logger    = context.HttpContext.RequestServices.GetService<ILogger<LogFilterAttribute>>();
        logger.LogInformation("我在 LogFilterAttribute ,transient = {transient},scope = {scope},single = {single}",
                              transient.OperationId,
                              scope.OperationId,
                              single.OperationId);

        //Console.WriteLine(content);
    }
}
```

直接針對 Filter 進行單元測試會需要比較大的力氣，如果可以，應該把主要的邏輯放在別的物件，再讓 Filter 依賴使用；或者是用 TestServer 進行集成測試

Filter 可以放在 Global、Controller、Action，以下是 Global 的用法

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers(options => options.Filters.Add(new LogFilterAttribute()));
    ...
}
```

第一次執行

```text
info: WebApiNetCore31.LogFilterAttribute[0]
      我在 LogFilterAttribute ,transient = 多個接口-8e075a8e-eb66-46e6-b14b-c6233e902f2d,scope = 多個接口-94797ff9-9632-471b-8a5e-2f1a0885ffff,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0 info: WebApiNetCore31.Controllers.DefaultController[0]
      我在 DefaultController.Get ,transient = 多個接口-95010c45-61d4-4512-8d6e-724f1befbc75,scope = 多個接口-94797ff9-9632-471b-8a5e-2f1a0885ffff,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0
```

可以觀察到 scope 的內容是一樣的，同一個 Request 所使用的物件生命週期是一樣的

第二次執行

```text
info: WebApiNetCore31.LogFilterAttribute[0]
      我在 LogFilterAttribute ,transient = 多個接口-c74b68f4-f8fb-41b3-9848-aa466a4178a6,scope = 多個接口-d8263a01-1344-41a1-ba1f-dc5da5d922e9,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0 info: WebApiNetCore31.Controllers.DefaultController[0]
      我在 DefaultController.Get ,transient = 多個接口-d86ec1da-e655-4680-9e8c-4861dec2c5c0,scope = 多個接口-d8263a01-1344-41a1-ba1f-dc5da5d922e9,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0
```

第三次執行

```text
info: WebApiNetCore31.LogFilterAttribute[0]
      我在 LogFilterAttribute ,transient = 多個接口-7b8f0c77-c213-40e9-b38e-79ddedc61b58,scope = 多個接口-e6fcd493-cdac-4edf-8797-71be7a39a18c,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0 info: WebApiNetCore31.Controllers.DefaultController[0]
      我在 DefaultController.Get ,transient = 多個接口-a4c378c3-bfb0-42a3-94be-dd636911b652,scope = 多個接口-e6fcd493-cdac-4edf-8797-71be7a39a18c,single = 多個接口-5016dd4b-918e-4d33-b0e8-935dbc18ecf0
```

## ASP.NET Web API 2

跟 ASP.NET Core 的步驟差不多，只列出 Filter，代碼如下，更多的內容請參考 https://github.com/yaochangyu/sample.dotblog/tree/master/DI/Lab.MsDI/WebApiNet48

```csharp
public class LogFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(HttpActionContext actionContext)
    {
        var requestScope = actionContext.Request.GetDependencyScope();

        var transient = requestScope.GetService(typeof(ITransientMessager)) as MultiMessager;
        var scope     = requestScope.GetService(typeof(IScopeMessager)) as MultiMessager;
        var single    = requestScope.GetService(typeof(ISingleMessager)) as MultiMessager;
 
        var logger = LogManager.GetCurrentClassLogger();
        var content = "我在 LogFilterAttribute.OnActionExecuting\r\n" +
                      $"transient:{transient.OperationId}\r\n"      +
                      $"scope:{scope.OperationId}\r\n"              +
                      $"single:{single.OperationId}";
        Console.WriteLine(content);
        logger.Info(content);
    }
}
```

## ASP.NET MVC 5

跟 ASP.NET Core 的步驟差不多，只列出 Filter，代碼如下，更多的內容請參考 https://github.com/yaochangyu/sample.dotblog/tree/master/DI/Lab.MsDI/Mvc5Net48

這裡我用兩種方式取出服務的實例，

第一種：

```csharp
var serviceScope = filterContext.HttpContext?.Items[typeof(IServiceScope)] as IServiceScope;
serviceScope.ServiceProvider.GetService<ITransientMessager>();
```

第二種：

```csharp
DependencyResolver.Current.GetService<IScopeMessager>();
```

完整代碼如下：

```csharp
public class LogFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        var serviceScope = filterContext.HttpContext?.Items[typeof(IServiceScope)] as IServiceScope;
        if (serviceScope == null)
        {
            return;
        }

        var serviceProvider = serviceScope.ServiceProvider;

        var transient = serviceProvider.GetService<ITransientMessager>();
        var single    = serviceProvider.GetService<ISingleMessager>();

        var scope     = serviceProvider.GetService<IScopeMessager>();
        var scope2    = DependencyResolver.Current.GetService<IScopeMessager>();
        var noeq = scope.OperationId == scope2.OperationId;
        Debug.Assert(noeq);

        var logger = LogManager.GetCurrentClassLogger();
        var content = "我在 LogFilterAttribute.OnActionExecuting\r\n" +
                      $"transient:{transient.OperationId}\r\n"      +
                      $"scope:{scope.OperationId}\r\n"              +
                      $"single:{single.OperationId}";
        Console.WriteLine(content);

        logger.Trace(content);
    }
}
```

## WinForm/Console/Desktop App

我在 WinForm 裡的作法是是接依賴 Service Locator，因為我把它視為應用程式的進入點

```csharp
public partial class Form1 : Form
{
    int _counter = 1;
    public Form1()
    {
        this.InitializeComponent();
    }

    private void button1_Click(object sender, EventArgs e)
    {
        var serviceProvider = DependencyInjectionConfig.ServiceProvider;
        var work            = serviceProvider.GetRequiredService<Worker>();
        Console.WriteLine($"{this._counter}=>\r\n"+work.Get());
        
        this._counter++;
    }
}
```

第一次執行

```text
1=>
transient:多個接口-a8fb8fa3-bcee-4651-b4da-7ddb680b5304
scope:多個接口-cc916b98-74d0-4a89-8f16-eb80b8254a6e
single:多個接口-8e05751f-2e0e-43bc-a567-b1c66736df04
```

第二次執行

```text
2=>
transient:多個接口-7ac3e7bf-9d5a-4946-b80a-573fe42e9671
scope:多個接口-cc916b98-74d0-4a89-8f16-eb80b8254a6e
single:多個接口-8e05751f-2e0e-43bc-a567-b1c66736df04
```

第三次執行

```text
3=>
transient:多個接口-238e2adb-468b-4313-8e47-96079b40d5d7
scope:多個接口-cc916b98-74d0-4a89-8f16-eb80b8254a6e
single:多個接口-8e05751f-2e0e-43bc-a567-b1c66736df04
```

執行結果如下
![[1697983434.png]]

可以看到上述執行結果，不論按鈕案幾次 Single/Scope 的值都沒有變動，Scope 配置的物件生命週期，只會在 ASP.NET / ASP.NET Core 生效，原因是 ASP.NET 會根據每一個 `Request` 調用 `IServiceProvider.CreateScope` 產生全新的物件，但其他的框架不會做這件事。

根據需求自行呼叫 `IServiceProvider.CreateScope`，代碼如下：

```csharp
private void button1_Click(object sender, EventArgs e)
{
    var serviceProvider = DependencyInjectionConfig.ServiceProvider;
    using (var serviceScope = serviceProvider.CreateScope())
    {
        var work = serviceScope.ServiceProvider.GetRequiredService<Worker>();
        Console.WriteLine($"{this._counter}=>\r\n" + work.Get());
    }

    this._counter++;
}
```

這時，每一次按下按鈕所得到的 Scope 物件狀態就不一樣了
![[1697984750.png]]

# 官方建議

我列出了幾項，詳細的內容可以參考 [官方文件](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/dependency-injection)，for ASP.NET ([官方文件](https://learn.microsoft.com/zh-tw/dotnet/core/extensions/dependency-injection)，for .NET)

- 避免將 [`IApplicationBuilder.ApplicationServices`](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder.applicationservices) 宣告為靜態欄位或屬性給別處使用
- 避免使用 Service Locator pattern 獲得服務實例 ([`IServiceProvider.GetService`](https://learn.microsoft.com/zh-tw/dotnet/api/system.iserviceprovider.getservice))，應該透過 DI 取得實例

錯誤的做法
![[IncorrectCode.png|Incorrect code]]

正確的做法：

```csharp
public class MyClass
{ 
	private readonly IOptionsMonitor<MyOptions> _optionsMonitor; 
	
	public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
	{ 
		_optionsMonitor = optionsMonitor; 
	} 

	public void MyMethod()
	{
		var option = _optionsMonitor.CurrentValue.Option;

		...
	} 
}
```

詳細內容請參考
[Service Locator is an Anti-Pattern](https://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/)

- 避免在 `ConfigureServices` 呼叫 `BuildServiceProvider`

錯誤的做法：
![[badcodex.png|呼叫 BuildServiceProvider 的錯誤程式碼]]

在上圖中，選取下綠色波浪線會 `services.BuildServiceProvider` 顯示下列 ASP0000 警告：

> ASP0000 Calling 'BuildServiceProvider' from application code results in an additional copy of singleton services being created. Consider alternatives such as dependency injecting services as parameters to 'Configure'.

呼叫 `BuildServiceProvider` 會建立第二個容器，它會建立損毀的單例物件並讓多個容器參考

正確的作法：

```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme) .AddCookie();

	services.AddOptions<CookieAuthenticationOptions>( CookieAuthenticationDefaults.AuthenticationScheme)
		.Configure<IMyService>((options, myService) =>
		{
			options.LoginPath = myService.GetLoginPath();
		}

	services.AddRazorPages();
}
```

- 啟用 Scope 驗證，如需詳細資訊，請參閱 [範圍驗證](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/dependency-injection)
- DI 是用來替代全域靜態屬性取實例的解決方案，如果混用，可能無法展現 DI 的優點

# 不支援的功能

微軟的 DI Container 相較於其他第三方套件，功能並不是十分完善，以下是我試出目前不支援的功能

- 屬性注入
- 方法注入
- 用字串註冊，取出用字串，比如 GetService("xxxx")
- 檔案注入
- 子容器
- 掃描 Assembly 自動註冊，可使用 Scrutor 解決

# 自動註冊

[Scrutor](https://github.com/khellang/Scrutor)

雖然微軟並沒有內建自動註冊，但 Scrutor 已經實現出來了

```text
Install-Package Scrutor
```

根據 Assembly 掃描所有 Controller

```csharp
services.Scan(p => p.FromAssemblies(Assembly.Load("WebApiNet48"))
                    .AddClasses(c => c.AssignableTo<IHttpController>())
                    .AsSelf()
                    .WithScopedLifetime()
             );
```

更多的設定請參考 https://github.com/khellang/Scrutor#scanning

## 找出相對應的 Type

透過 `Assembly.GetTypes()` 加上一些條件、規則，也應該可以省去不少手動設定的苦工
一些比較複雜的對應使用自動註冊相當的麻煩，採取手動註冊的策略或許會比較好

```csharp
sources.AddScoped(typeof(Startup).Assembly.GetExportedTypes()
                                                .Where(t => !t.IsAbstract && !t.IsGenericTypeDefinition)
                                                .Where(t => typeof(IController).IsAssignableFrom(t)
                                                            || t.Name.EndsWith("Controller", StringComparison.OrdinalIgnoreCase)));
```

# 參考

- [.NET Core 中的相依性插入](https://learn.microsoft.com/zh-tw/dotnet/core/extensions/dependency-injection)
- [ASP.NET Core 控制器的相依性插入](https://learn.microsoft.com/zh-tw/aspnet/core/mvc/controllers/dependency-injection)
- [ASP.NET Core 檢視中的相依性插入](https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/dependency-injection)
- [筆記 - 不可不知的 ASP.NET Core 依賴注入](https://blog.darkthread.net/blog/aspnet-core-di-notes/)
- [ASP.NET Core Dependency Injection Deep Dive](https://joonasw.net/view/aspnet-core-di-deep-dive)

# 專案位置

https://github.com/yaochangyu/sample.dotblog/tree/master/DI/Lab.MsDI
