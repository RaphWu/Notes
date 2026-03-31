---
aliases:
date: 2013-11-09
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-5-prop-injection/
tags:
  - CSharp
  - IoC
  - Autofac
---

# Autofac 筆記 5- 屬性注入

前面談過 [傳入建構參數](http://blog.darkthread.net/post-2013-11-04-autofac-notes-4-constructor.aspx)，但並非所有物件參數都可由建構式傳入，有些要透過屬性指定 (例如: `new MyObject() { SomeProperty = SomeValue };)`，而這也是 IoC/DI 的工作職掌之一，專業術語叫 **Property Injection** (屬性注入)。

解說前先介紹幾個測試用類別: `Worker` 類別有個屬性 `Logger`，接受實作 `ILogger` 介面的記錄元件；我們簡單寫個 `Logger` 類別實作 `ILogger`，將訊息輸出到 Console 敷衍兩下湊數。

```csharp
using System;
public class Worker
{
    public ILogger Logger { get; set; }
    public void DoSomething(string command)
    {
        Console.WriteLine("JOB:" + command);
        Logger.Log(command);
    }
}
public interface ILogger
{
    void Log(string msg);
}
public class Logger : ILogger
{
    public void Log(string msg)
    {
        Console.WriteLine("LOG:" + msg);
    }
}
```

Autofac [指定屬性的方法](https://code.google.com/p/autofac/wiki/PropertyInjection) 有三種。

第一種的做法是透過 `Register()` 自訂物件建立細節，`Register()` Lambda 所傳入的 `c` 即為 Autofac 容器 (`IContainer` 或 `ILifetimeScope`)，可透過 `c.ResolveType<ILogger>` 取得已註冊的 ILogger 實作。

```csharp
private static void test1()
{
    ContainerBuilder builder = new ContainerBuilder();
    builder.RegisterType<Logger>().As<ILogger>();
    //方法自訂建構程序，傳回物件。建立物件時一併指定Property
    builder.Register(c => 
        new Worker() { 
            Logger = c.Resolve<ILogger>() 
        });
    IContainer container = builder.Build();
 
    var worker = container.Resolve<Worker>();
    worker.DoSomething("Wash the dog");
}
```

第二種做法是利用 Autofac 物件建立事件 `OnActivated`，於物件建立完成後指定。`OnActivated` 事件傳入參數的 `Instance` 屬性為剛建好的物件，而 `Context` 屬性則為 Autofac 容器。(PS: 除了 `OnActivated`，還有 `OnActivating` 事件可以置換 `Instance`、注入屬性或進行其他初始化；`OnRelease` 事件則可取代物件原有的 `Dispose()` 邏輯，提供良好的自訂彈性，細節可參考 [文件](https://code.google.com/p/autofac/wiki/LifetimeEvents))

```csharp
private static void test2()
{
    ContainerBuilder builder = new ContainerBuilder();
    builder.RegisterType<Logger>().As<ILogger>();
    //利用OnActivated事件，物件建立後指定Property
    //OnActivated事件會傳入IActivatedEventArgs，
    //其中的Instance為剛建好的物件、Context為IContainer或ILifetimeScope容器
    builder.RegisterType<Worker>().OnActivated(
        e => e.Instance.Logger = e.Context.Resolve<ILogger>());
    IContainer container = builder.Build();
 
    var worker = container.Resolve<Worker>();
    worker.DoSomething("Wash the dog");
}
```

第三種方法我覺得最酷!

`RegisterType()` 時直接加上 `PropertyAutowired()`，則 Autofac 建立物件時將一併掃瞄物件所有屬性，只要該屬性型別已被註冊，就自動產生 (或取得) `Instance` 傳入，即便事後增加 Property 也無需更動註冊程序，算是貫徹了 IoC/DI 的精神，深得我心。

```csharp
private static void test3()
{
    ContainerBuilder builder = new ContainerBuilder();
    builder.RegisterType<Logger>().As<ILogger>();
    //透過PropertyAutowired()交由Autofac自動解析
    builder.RegisterType<Worker>().PropertiesAutowired();
    IContainer container = builder.Build();
 
    var worker = container.Resolve<Worker>();
    worker.DoSomething("Wash the dog");
}
```
