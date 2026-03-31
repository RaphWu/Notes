---
aliases:
date: 2025-12-06
update:
author: 焦涛
language: CSharp
sourceurl: https://www.cnblogs.com/Joetao/articles/19314533
tags:
  - CSharp
  - AOP
  - Castle_DynamicProxy
---

# 一、AOP 核心概念

AOP（Aspect-Oriented Programming，面向切面编程）是一种**补充 OOP（面向对象编程）** 的编程思想。核心是将程序中**分散在各个模块的通用功能（如日志、事务、权限校验）** 抽取为独立的“切面（Aspect）”，在不修改原有业务代码的前提下，通过“织入（Weaving）”方式将切面动态融入到业务流程的指定位置（如方法执行前、执行后、异常时）。

## 关键术语

|术语|说明|
|---|---|
|切面（Aspect）|封装通用功能的模块（如日志切面、事务切面），是 AOP 的核心载体。|
|连接点（JoinPoint）|程序执行过程中的“特定位置”（如方法调用前、方法返回后、异常抛出时）。|
|切入点（PointCut）|筛选出需要织入切面的连接点（即“哪些方法需要应用切面”）。|
|通知（Advice）|切面在连接点执行的具体逻辑（如 Before 通知、After 通知、Around 通知）。|
|织入（Weaving）|将切面的通知逻辑融入到目标业务代码的过程（编译时、运行时、加载时）。|

## AOP 与 OOP 的区别

- OOP：按“业务实体”划分模块（如用户类、订单类），解决“是什么”的问题；
- AOP：按“通用功能”划分模块（如日志、事务），解决“怎么做”的问题，避免代码冗余。

# 二、C#中实现 AOP 的常见方式

C#中实现 AOP 的核心思路是**拦截目标方法调用**，常用方式有 3 种：

1. **动态代理（最常用）**：通过 Emit、Castle DynamicProxy 等生成目标类的代理类，拦截方法调用；
2. **特性 + 反射**：自定义特性标记目标方法，通过反射扫描并执行切面逻辑；
3. **框架集成**：如 ASP.NET Core 的 Filter、Autofac 的 Interceptor、PostSharp（编译时织入）。

下面以**“日志切面”** 为例，用「Castle DynamicProxy（动态代理）」实现经典易懂的 AOP 代码（入门友好、无框架依赖）。

# 三、经典代码示例（日志切面）

需求：给所有业务方法自动添加“方法执行前打印参数、执行后打印返回值、异常时打印错误信息”的日志功能，不修改业务代码。

## 步骤 1：安装依赖（Castle DynamicProxy）

通过 NuGet 安装：`Install-Package Castle.Core`（DynamicProxy 是 Castle.Core 的核心组件）

## 步骤 2：定义切面（日志通知 + 切入点）

```csharp
using Castle.DynamicProxy;
using System;
using System.Reflection;

// 1. 日志切面（实现IInterceptor接口，核心逻辑）
public class LogAspect : IInterceptor
{
    // 拦截目标方法调用的核心方法
    public void Intercept(IInvocation invocation)
    {
        // ====== 前置通知（Before）：方法执行前 ======
        var method = invocation.Method;
        var className = method.DeclaringType.Name;
        var methodName = method.Name;
        var parameters = invocation.Arguments; // 方法参数

        Console.WriteLine($"【日志】[{DateTime.Now:HH:mm:ss}] 开始执行：{className}.{methodName}");
        if (parameters.Length > 0)
        {
            var paramStr = string.Join(", ", parameters);
            Console.WriteLine($"【日志】参数：{paramStr}");
        }

        try
        {
            // ====== 执行目标业务方法（Proceed()是关键：触发原方法执行） ======
            invocation.Proceed();

            // ====== 后置通知（AfterReturning）：方法成功执行后 ======
            var returnValue = invocation.ReturnValue; // 方法返回值
            Console.WriteLine($"【日志】[{DateTime.Now:HH:mm:ss}] 执行成功！返回值：{returnValue ?? "无"}");
        }
        catch (Exception ex)
        {
            // ====== 异常通知（AfterThrowing）：方法执行异常时 ======
            Console.WriteLine($"【日志】[{DateTime.Now:HH:mm:ss}] 执行失败！异常：{ex.Message}");
            throw; // 抛出异常，不影响原有业务的异常处理
        }
    }
}

// 2. 代理工厂（简化代理类创建，对外提供统一接口）
public static class ProxyFactory
{
    // 创建目标类的代理对象（自动应用日志切面）
    public static T CreateProxy<T>() where T : class, new()
    {
        var generator = new ProxyGenerator(); // 动态代理生成器
        var interceptor = new LogAspect(); // 加载切面
        // 生成代理类并返回（T必须是接口或可继承的类）
        return generator.CreateClassProxy<T>(interceptor);
    }
}
```

## 步骤 3：定义业务逻辑（无任何 AOP 相关代码）

```csharp
// 业务接口（可选，DynamicProxy也支持直接代理类）
public interface IUserService
{
    string GetUserName(int userId);
    bool UpdateUserAge(int userId, int newAge);
}

// 业务实现类（纯业务逻辑，无日志代码）
public class UserService : IUserService
{
    // 业务方法1：根据用户ID获取用户名
    public string GetUserName(int userId)
    {
        if (userId <= 0)
            throw new ArgumentException("用户ID必须大于0");
        
        // 模拟数据库查询
        return userId == 1 ? "张三" : "未知用户";
    }

    // 业务方法2：更新用户年龄
    public bool UpdateUserAge(int userId, int newAge)
    {
        if (newAge < 0 || newAge > 120)
            return false;
        
        // 模拟数据库更新
        Console.WriteLine($"（业务逻辑）用户{userId}的年龄更新为{newAge}");
        return true;
    }
}
```

## 步骤 4：测试代码（使用代理对象调用业务方法）

```csharp
class Program
{
    static void Main(string[] args)
    {
        // 关键：通过代理工厂创建业务类的代理对象（而非直接new UserService()）
        IUserService userService = ProxyFactory.CreateProxy<UserService>();

        // 调用业务方法（自动触发日志切面）
        Console.WriteLine("=== 测试正常获取用户名 ===");
        var name = userService.GetUserName(1);
        Console.WriteLine($"最终结果：{name}\n");

        Console.WriteLine("=== 测试异常场景 ===");
        try
        {
            userService.GetUserName(-5);
        }
        catch { }
        Console.WriteLine();

        Console.WriteLine("=== 测试更新用户年龄 ===");
        var result = userService.UpdateUserAge(1, 25);
        Console.WriteLine($"最终结果：{result}");
    }
}
```

## 输出结果（日志自动织入）

```makefile
=== 测试正常获取用户名 ===
【日志】[15:30:45] 开始执行：UserService.GetUserName
【日志】参数：1
【日志】[15:30:45] 执行成功！返回值：张三
最终结果：张三

=== 测试异常场景 ===
【日志】[15:30:45] 开始执行：UserService.GetUserName
【日志】参数：-5
【日志】[15:30:45] 执行失败！异常：用户ID必须大于0

=== 测试更新用户年龄 ===
【日志】[15:30:45] 开始执行：UserService.UpdateUserAge
【日志】参数：1, 25
（业务逻辑）用户1的年龄更新为25
【日志】[15:30:45] 执行成功！返回值：True
最终结果：True
```

# 四、AOP 的优点

1. **解耦通用功能与业务逻辑**：日志、事务、权限等通用功能集中在切面，业务代码只关注核心逻辑，避免“重复代码（DRY 原则）”；
2. **提高代码复用性**：一个切面可应用于多个业务模块（如日志切面可同时作用于用户、订单、商品服务）；
3. **便于维护扩展**：修改通用功能时，只需修改切面代码，无需改动所有业务模块（如日志格式变更，仅改 LogAspect）；
4. **不侵入原有代码**：无需修改业务类的方法、属性，通过代理/织入方式动态增强，符合“开闭原则”；
5. **统一规范**：确保通用功能的执行时机、格式一致（如所有方法的日志格式统一）。

# 五、AOP 的缺点

1. **增加系统复杂度**：引入切面、代理、织入等概念，调试时需要跟踪切面逻辑，降低代码可读性（尤其是复杂切面）；
2. **性能损耗**：动态代理、反射等方式会带来轻微性能开销（编译时织入如 PostSharp 性能较好，但有学习成本）；
3. **调试难度提升**：业务方法的执行流程被切面拦截，异常堆栈可能包含代理类信息，定位问题需要熟悉 AOP 框架；
4. **学习成本**：需要理解 AOP 术语（切面、切入点、通知）和具体实现框架（如 Castle、Autofac）的使用；
5. **过度使用风险**：简单功能（如单个方法的日志）使用 AOP 会“杀鸡用牛刀”，增加不必要的复杂性。

# 六、AOP 的适用场景

- 日志记录（请求日志、操作日志）；
- 事务管理（数据库操作的事务开启/提交/回滚）；
- 权限校验（接口访问权限、方法调用权限）；
- 缓存处理（方法结果缓存、缓存失效）；
- 异常处理（统一异常捕获、格式化返回）；
- 性能监控（方法执行时间统计）。

# 总结

AOP 的核心价值是**分离关注点**，让业务代码和通用功能各司其职。在 C#开发中，DynamicProxy 适合简单场景，ASP.NET Core Filter 适合 Web 接口场景，Autofac Interceptor 适合依赖注入场景。使用时需权衡“解耦收益”和“复杂度成本”，避免过度设计。
