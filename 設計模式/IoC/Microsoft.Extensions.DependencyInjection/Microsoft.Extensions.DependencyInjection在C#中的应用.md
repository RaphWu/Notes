---
aliases:
date: 2024-07-18
update:
author: dotNET跨平台
language: C#
sourceurl: https://blog.csdn.net/sD7O95O/article/details/140536216
tags:
  - CSharp
  - IoC
---

# 引言

[`Microsoft.Extensions.DependencyInjection`](https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.dependencyinjection) 是 .NET Core 框架内置的一个依赖注入（Dependency Injection，简称 DI）库，它为开发者提供了一种灵活的方式来管理应用程序中的依赖关系。依赖注入是一种设计模式，旨在降低组件之间的耦合度，提高代码的可测试性和可维护性。本文将详细介绍 `Microsoft.Extensions.DependencyInjection` 的基本用法，并通过代码示例来展示如何在 C# 项目中应用它。

# 安装与基本使用

## 安装

要使用 `Microsoft.Extensions.DependencyInjection`，首先需要在你的项目中安装它。可以通过 NuGet 包管理器来安装。在 Visual Studio 中，你可以通过“管理 NuGet 包”搜索并安装 `Microsoft.Extensions.DependencyInjection` 包。

## 基本使用步骤

1. **创建服务集合**：首先，你需要创建一个 `IServiceCollection` 实例，用于注册服务。
2. **注册服务**：使用服务集合的 `AddTransient`、`AddScoped` 或 `AddSingleton` 方法来注册服务。这些方法决定了服务的生命周期。
3. **构建服务提供者**：通过调用服务集合的 `BuildServiceProvider` 方法来构建 `IServiceProvider` 实例，用于解析服务。
4. **解析服务**：通过服务提供者来解析并使用服务。

## 示例代码

以下是一个简单的示例，展示了如何使用 `Microsoft.Extensions.DependencyInjection` 来注册和使用服务。

### 定义接口与实现类

```csharp
public interface IVehicle
{
    void Run();
}

public class Car : IVehicle
{
    public void Run()
    {
        Console.WriteLine("Car is running");
    }
}

public interface IWeapon
{
    void Fire();
}

public class Gun : IWeapon
{
    public void Fire()
    {
        Console.WriteLine("Bang!");
    }
}
```

### 注册并使用服务

```csharp
using Microsoft.Extensions.DependencyInjection;
using System;

class Program
{
    static void Main(string[] args)
    {
        // 創建服務集合
        var services = new ServiceCollection();

        // 註冊服務
        services.AddTransient<IVehicle, Car>();
        services.AddSingleton<IWeapon, Gun>();

        // 構建服務提供者
        var provider = services.BuildServiceProvider();

        // 解析服務
        var car = provider.GetService<IVehicle>();
        var gun = provider.GetService<IWeapon>();

        // 使用服務
        car.Run();
        gun.Fire();

        // 如果需要，可以解析並使用更多服務...
    }
}
```

# 生命周期

`Microsoft.Extensions.DependencyInjection` 提供了三种服务生命周期：
1. **`Transient`**（瞬时）：每次请求都会创建一个新的实例。
2. **`Scoped`**（作用域）：在同一个作用域内只创建一个实例，对于 Web 应用程序，作用域通常与请求相关联。
3. **`Singleton`**（单例）：整个应用程序生命周期内只创建一个实例。

# 高级用法

## Keyed 服务

当你有多个服务实现同一个接口时，可以使用 `Keyed` 服务来注册和解析特定的实现。

```csharp
services.AddKeyedTransient<IVehicle, Car>("Car");
services.AddKeyedTransient<IVehicle, Truck>("Truck");

// 解析特定的實現
var car = provider.GetKeyedService<IVehicle>("Car");
```

## 使用 ActivatorUtilities

`ActivatorUtilities` 提供了一種靈活的方式來創建和解析依賴項，即使它們沒有被顯式註冊到服務中。

```csharp
public class Driver
{
    private readonly IVehicle _vehicle;

    public Driver(IVehicle vehicle)
    {
        _vehicle = vehicle;
    }

    public void Drive()
    {
        _vehicle.Run();
    }
}

// 使用 ActivatorUtilities 創建 Driver 實例
var driver = ActivatorUtilities.CreateInstance<Driver>(provider, provider.GetService<IVehicle>());
driver.Drive();
```

# 结论

`Microsoft.Extensions.DependencyInjection` 为 C# 开发者提供了一种强大而灵活的方式来管理应用程序中的依赖关系。通过依赖注入，我们可以降低组件之间的耦合度，提高代码的可测试性和可维护性。希望本文的介绍和示例代码能够帮助你更好地理解和应用 `Microsoft.Extensions.DependencyInjection`。
