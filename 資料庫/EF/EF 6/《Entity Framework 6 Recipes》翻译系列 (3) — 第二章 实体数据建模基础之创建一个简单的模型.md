---
aliases:
date: 2015-05-06
update:
author: 付灿
language: C#
sourceurl: https://www.cnblogs.com/VolcanoCloud/p/4479105.html
tags:
  - CSharp
  - 資料庫
  - EF6
---

# 第二章 实体数据建模基础

很有可能，你才开始探索实体框架，你可能会问“我们怎么开始？”，如果你真是这样的话，那么本章就是一个很好的开始。如果不是，你已经建模，并在实体分裂和继承方面感觉良好，那么你可以跳过本章。

本章将带你漫游使用实体框架建模的基本实例，建模是实体框架的核心特性，同时也是区别实体框架和微软早期的数据访问平台的特性。一旦建好模，你就可以面向模型编写代码，而不用面向关系数据库中的行和列。

本章以创建一个简单概念模型的实例开始，然后让实体框架创建底层的数据库，剩下的实例，将向你展示，如何通过数据库中已存在的表以及它们之间的关系来建模。

## 2-1 创建一个简单模型

*问题*

你有一个崭新的项目，需要创建一个模型。

*解决方案*

我们设想你需要创建一个管理，人员姓名、电话号码的应用程序。为了保持尽可能的简单，我们假设你只需要一个实体类型：`Person`。

按以下步骤来创建模型：

1. 右键你的项目，然后选择➤New Item(新建项)。
2. 从模板中选择 ADO.NET Entity Data Model（ADO.NET 实体数据模型），然后点击 Add（增加）。该模板在 Visual C#条目下面的 Data 项下面。（如图 2-1）。
![[《Entity Framework 6 Recipes》翻译系列03_图2-1 增加一个新的.emdx文件.png|图2-1 增加一个新的.emdx文件，其中包含使用XML描述的概念模型、存储模型、映射层]]

3. 在向导第一步选择 Empty Model（空模型）并点击 `Finish`（完成）按钮。向导将创建一个新的设计器界面上为空的概念模型。
4. 右键设计器界面，选择增加➤Entity（实体）。
5. 在 Entity Name(实体名称）字段键入 `Person`，选中 Create a Key Propety(创建实体键）复选框，使用 `Id` 作为实体键并确保其类型为 `Int32`。点击 OK 按钮，一个新的 Person 实体便出现在设计器窗口中（如图 2-2）。
   ![[《Entity Framework 6 Recipes》翻译系列03_图2-2 在概念模型中添加一个代表Person的实体类.png|图2-2 在概念模型中添加一个代表 Person 的实体类]]

6. 右键 `Person` 实体顶部，然后选择 Add（添加） ➤Scalar Property（标量属性）。一个新的标量属性便添加到了 `Person` 实体中。
7. 将增加的标量属性重命名为 `Firstname`。然后继续添加标量属性 `LastName`、`MiddleName` 和 `PhoneNumber`。
8. 右键 `Id` 属性并选择 `Properties`（属性），在属性窗口中，如果 `StoreGneneratedPattern` 属性值未设置为 `Identity` 时，将其设置为 `Identity`。该标识是指 `Id` 属性的值将由存储层（数据库）计算产生。最终得到的数据库脚本会标识 `Id` 列为 `identity` 列，存储逻辑模型也会知道，数据库将会自动管理该列的值。
   完成后的概念模型如图 2-3 所示。
   ![[《Entity Framework 6 Recipes》翻译系列03_图2-3 模型包含一个代表Person的实体类型.png|图2-3 模型包含一个代表 Person 的实体类型]] 你已经完成了一个简单的概念模型。不过从该模型生成数据库，还有一些工作要做：
9. 需要更改我们模型的一些属性，以帮助我们生成数据库。右键计设器窗口，选择 properties（属性）。更改数据库架构名称（Database Schema name）为 Chapter2，更改实体容器名称（Entity Container Name）为 EF6RecipesContext。如图 2-4 所示。
   ![[《Entity Framework 6 Recipes》翻译系列03_图2-4 更改模型属性.png|图2-4 更改模型属性]]
10. 右键设计器窗口并选择 Generate Database Script from Model（根据模型生成数据库）。选择一个已存在的数据库连接或者新建一个连接。图 2-5，我们选择创建一个新的本地数据库 EF6Recipes 的连接。
    ![[《Entity Framework 6 Recipes》翻译系列03_图2-5 创建一个实体框架从概念模型创建数据库脚本要使用的数据库连接.png|图2-5 创建一个实体框架从概念模型创建数据库脚本要使用的数据库连接]]
11. 单击 OK 按钮完成连接属性设置，然后单击 Next（下一步）预览数据库脚本（如图 2-6）。一旦点击 Finish（完成），生成的脚本就被添加到你的项目中。
    ![[《Entity Framework 6 Recipes》翻译系列03_图2-6 在.edmx文件中生成存储逻辑模型并创建数据库脚本.png|图2-6 在.edmx 文件中生成存储逻辑模型并创建数据库脚本]]
12. 在 SSMS（SQL Server Management Studio）查询窗口中执行上面生成的数据库脚本创建数据库和 `People` 表。

### 原理

实体框架设计器是一个创建概念模型、存储模型和映射层的强有力工具。它提供双向建模的功能，你可以从一个空白的模型设计窗口建模，也可通过导入一个已存在的数据库来创建概念模型、存储模型和映射层。当前版本提供了有限的双向建模功能，它允许从模型重新创建数据库，根据数据库的改变来更新模型。

概念模型拥有很多影响生成存储逻辑模型和数据库脚本的属性，我们更改了其中的两个属性，第一个是容器的名称，他是继承至 `DbContext` 上下文对象。我们给它命名为 `EF6RecipesContext`，它与本书使用的上下文对象保持一致。

另一个是，更改了表示生成存储逻辑模型和数据库脚本的架构名称为“`Chaper2`”。

代码清单 2-1 演示了，我们创建和插入 Person 实体类型的实例，并将所有 Person 实体保存到数据库。

```csharp title="代码清单2-1 从模型中插入和获取数据"
using (var context = new EF6RecipesContext())
{
    var person = new Person
    {
        FirstName = "Robert",
        MiddleName = "Allen",
        LastName = "Doe",
        PhoneNumber = "867-5309"
    };
    context.People.Add(person);
    person = new Person
    {
        FirstName = "John",
        MiddleName = "K.",
        LastName = "Smith",
        PhoneNumber = "824-3031"
    };
    context.People.Add(person);
    person = new Person
    {
        FirstName = "Billy",
        MiddleName = "Albert",
        LastName = "Minor",
        PhoneNumber = "907-2212"
    };
    context.People.Add(person);
    person = new Person
    {
        FirstName = "Kathy",
        MiddleName = "Anne",
        LastName = "Ryan",
        PhoneNumber = "722-0038"
    };
    context.People.Add(person);
    context.SaveChanges();
}
using (var context = new EF6RecipesContext())
{
    foreach (var person in context.People)
    {
        System.Console.WriteLine("{0} {1} {2}, Phone: {3}",
            person.FirstName, person.MiddleName,
            person.LastName, person.PhoneNumber);
    }
}
```

代码清单 2-1 的输出为：
```bash
John K. Smith, 　　　　    Phone: 824-3031
Robert Allen Doe, 　　　　Phone: 867-5309
Kathy Anne Ryan, Phone: 722-0038
Billy Albert Minor, Phone: 907-2212
```

### 最佳实践

当创建一个上下文实例时，我们使用 `{sharp} using()` 语句：
```csharp
using (var context = new EF6RecipesContext())
{
	...
}
```

如果你不熟悉这种模式，也没关系，因为它很简单。一般情况下，我们通过 `new` 操作符并将结果赋值给变量来得到一个对象的实例，当这个变量超出其生命周期，该对象不再被别的任何对象引用。垃圾回收器会在某一个时间点开始释放该对象所占的内存工作。对于我们.NET 应用程序中的大多数对象来说，这是一个巨大的工作，因为他们会一直占用着资源，直到垃圾回收器开始工作。而垃圾回收器具有不确定性，因为他总是按自己的计划来完成其工作，对此我能施加的影响很有限。

`DbContext` 上下文对象的实例占用着像数据库连接这样的，我们希望不使用时就立即释放的系统资源。我们真的不希望数据库连接的释放工作要等到垃圾回收器来完成。

`using()` 语句有很好的特性。首先，当代码执行完 `using(){}` 代码块时，上下文中的 `Dispose()` 方法会被自动调用。因为 `DbContext` 上下文实现了 `IDisposable` 接口，该方法将关闭所有数据库连接，清理任何需要被释放的资源。

其次，不管怎样，只要代码离开 `using(){}` 代码块，方法 `Dispose()` 就会被调用。最重要的是，代码块里即使遇到 `return` 语句或者是抛出了异常，它都能保证资源得到合理的释放。

这里的最佳实践是，==当创建 `DbContext` 上下文对象时总是使用 `using(){}` 代码块，它将进一步帮助你创建健壮的代码。==

本篇就到这里吧，如果有翻译不当的地方，恳请指正。如果你觉得本系值得与更多人分享，那么请点击右下角的推荐。谢谢。本文由 VolcanoCloud 翻译，转载请注明出处。谢谢！
