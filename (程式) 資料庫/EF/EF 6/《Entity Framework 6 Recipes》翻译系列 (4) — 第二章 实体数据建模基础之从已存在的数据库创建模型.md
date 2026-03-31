---
aliases:
date: 2015-05-08
update:
author: 付灿
language: C#
sourceurl: https://www.cnblogs.com/VolcanoCloud/p/4481582.html
tags:
  - CSharp
  - 資料庫
  - EF6
---

## 2-2 从已存在的数据库创建模型

*问题*

有一个存在的数据库，它拥有表、也许还有视图、外键。你想通过它来创建一个模型。

*解决方案*

让我们设想，你拥有一个描述诗人（Poet) 以及他们的诗 (Poem)，还有他们之间关系的数据库。如图 2-7 所示。
![[《Entity Framework 6 Recipes》翻译系列04_图2-7 一个关于诗人及他们的诗的简单数据库.png|图2-7 一个关于诗人及他们的诗的简单数据库]]

从上图可以看出，一个诗人可能是一首或多首诗的作者，每首诗可以按其韵律来分类，韵律是诗句的基本模式。上图未显示数据库中将表连接在一起的视图，它让我们更容易的枚举诗人，诗和韵律。

按下列步骤，将表、视图以及关系导入到模型：

1. 右键你的项目，选择 Add（增加） ➤New Item（新建项）。
2. 选择 Visual C#条目下的 Data 模板下的 ADO.NET Entity Data Model（ADO.NET 实体数据模型）。
3. 选择 Generate from database 从一个已存在的数据库创建模型，点击 Next（下一步）。
4. 可以选择一个已存在的数据库连接，也可以选择新建一个数据库连接，如果你选择新建，你将要选择数据库服务器、认证方式（Windwos or SQL Server）以及一个数据库。你一旦选择好，便可以点击 Test Connection（测试连接) 测试连接是否可用。测试好后，点击 Next（下一步）。

弹出的对话框中显示了数据库所有的表、视图以及存储过程。选择你希望包含在模型中的项。我们选择所有的表（Meter,Poem,Poet），视图（vwLibrary），然后勾选上确定所生成对象名称的单复数形式、在模型中包含外键列复选框。我们将会进一步对此时行讨论。图 2-8 展示了我们所做的选择。
![[《Entity Framework 6 Recipes》翻译系列04_图2-8 选择表、视图包含进模型.png|图2-8 选择表、视图包含进模型，勾选上确定所生成对象名称的单复数形式、在模型中包含外键列复选框]]

当点击 Finish（完成），向导会生成一个包含三张表和一个视图的模型。向导从数据库读取外键约束，并推导出 Poet 和 Poem(s) 之间的一对多关系，还有 Meter 和 Poem(s) 之间的一对多关系。
![[《Entity Framework 6 Recipes》翻译系列04_图2-9 概念模型.png|图2-9 概念模型]]

图 2-9 展示了一个包含表 `Poet`,`Poem` 以及 `Meter`、视图 `vwLibrary` 的模型。

现在，你拥有了一个可以在代码中使用的模型。请注意， vwLibrary 实体是基于数据中的视图 vwLibrary 的。在绝大多数据库中，视图是只读的，插入、删除、更新不被支持。在实体框架中也同样如此，实体框架把视图视为只读。你可以通过映射存储过程来解决基于视图的实体的创建、更新和删除动作。我们将在第六章对此进行演示。

### 原理

让我们一起来看导入向导为我们创建的模型。请注意，实体中已经包含了标量属性和导航属性。标量属性被映射到数据库表中列，导航属性却来至数据库中的表间关系。

在数据库关系图中，一个 `poem` 拥有一个 `meter` 和一个 `poet`（作者）。这符合 `Meter` 和 `Poet` 中的导航属性。如果我有一个 `Poem` 实体的实例，导航属性 `Poet` 将引用一个 `Poet` 实体的实例，导航属性 `Meter` 将引用一个 `Meter` 实体的实例。

一个 `Poet` 可能是多首诗的作者，它其中的导航属性 `Poems` 包含一个 `Poem` 实体的实例集合。这个集合可能是空的，这代表该诗人还未创建任何一首诗。对于 `Meter` 实体，其中的导航属性 `Poems` 同样也是一个集合。该导航属性包含属于指定 `Meter` 的 `Poem` 实体实例的集合。SQL Server 不支持在视图上创建关系，这在模型中反映为一个没有导向属性的 `vwLibrary` 实体。

导入向导在包含集合的导航属性的名称单复数形式上表示得相当的聪明。当你右键实体去查看他的 **Properties（属性）**，你会看到每个实体的实体集的名称同样为复数形式。例如 `Poem` 实体的实体集名称为 `Poems`. 这种自动添加复数利益于，勾选上确定所生成对象名称的单复数形式复选框。

勾选在模型中包含外键列复选框选项，使外键也包含在了模型中。虽然看上去，在拥有导航属性的同时好像没有必要再拥有外键属性。对此，我们会在接下来的技巧中演示拥有一个可直接访问外键属性的好处。

代码清单 2-2，演示了如何在模型中创建 Poet,Poem 以及 Meter 实体的实例并将他们保存到数据为。同时也演示了，如何通过查询模型从数据库中获取到 `poets` 和 `poems`。

```csharp title="代码清单2-2 在模型中插入和查询"
using (var context = new EF6RecipesContext())
{
    var poet = new Poet { FirstName = "John", LastName = "Milton" };
    var poem = new Poem { Title = "Paradise Lost" };
    var meter = new Meter { MeterName = "Iambic Pentameter" };
    poem.Meter = meter;
    poem.Poet = poet;
    context.Poems.Add(poem);
    poem = new Poem { Title = "Paradise Regained" };
    poem.Meter = meter;
    poem.Poet = poet;
    context.Poems.Add(poem);
    poet = new Poet { FirstName = "Lewis", LastName = "Carroll" };
    poem = new Poem { Title = "The Hunting of the Shark" };
    meter = new Meter { MeterName = "Anapestic Tetrameter" };
    poem.Meter = meter;
    poem.Poet = poet;
    context.Poems.Add(poem);
    poet = new Poet { FirstName = "Lord", LastName = "Byron" };
    poem = new Poem { Title = "Don Juan" };
    poem.Meter = meter;
    poem.Poet = poet;
    context.Poems.Add(poem);
    context.SaveChanges();
}

using (var context = new EF6RecipesContext())
{
    var poets = context.Poets;
    foreach (var poet in poets)
    {
        Console.WriteLine("{0} {1}", poet.FirstName, poet.LastName);
        foreach (var poem in poet.Poems)
        {
            Console.WriteLine("\t{0} ({1})", poem.Title, poem.Meter.MeterName);
        }
    }
}

// 使用视图 vwLibrary
using (var context = new EF6RecipesContext())
{
    var items = context.vwLibraries;
    foreach (var item in items)
    {
        Console.WriteLine("{0} {1}", item.FirstName, item.LastName);
        Console.WriteLine("\t{0} ({1})", item.Title, item.MeterName);
    }
}
```


代码清单 2-2 中第一个代码块，我们创建 了 `Poet`,`Poem` 以及 `Meter` 实体类型的实例，诗人 John Milton，他的诗“Paradise Lost" 以及该诗的韵律，该诗属于五步抑扬格 -Iambic Pentameter（诗的一种韵律）。我们一旦创建 `Poem` 实体类型的实例 `peom`，并设置 `poem` 的 `Meter` 属性指向 `meter` 实例，`Poet` 属性指向 `poet` 实例。使用相同的方法，我们创建其它的实体及关系。一旦完成创建，我们就可以调用 `SaveChanges()` 方法生成并执行适当的 SQL 语句，在底层数据库中插入数据。

代码清单 2-2 的输出发下：
```asciidoc
Lord Byron
Don Juan (Anapestic Tetrameter)
Lewis Carroll
The Hunting of the Shark (Anapestic Tetrameter)
John Milton
Paradise Regained (Iambic Pentameter)
Paradise Lost (Iambic Pentameter)
Lewis Carroll
The Hunting of the Shark (Anapestic Tetrameter)
Lord Byron
Don Juan (Anapestic Tetrameter)
John Milton
Paradise Regained (Iambic Pentameter)
John Milton
Paradise Lost (Iambic Pentameter)
```

在代码中，我们最开始创建实例 `poet`,`poem` 以及 John Milton 第一首诗的韵律 `meter`。一旦我们创建好，就可以设置 `poem` 的导航属性 `Meter` 为 `meter` 实例,`Poet` 导航属性为 `poet` 实例。实例 `poem` 的所有设置已完成，调用 `Add()` 方法将其添加到上下文中。接下来的工作将由实体框架来完成，包括添加 `poem` 到 `poet` 实例的导航属性 `Poems` 集合中，添加 `poem` 到 `meter` 实例的导航属性 `Poems` 集合中。使用相同的步骤完成剩下的任务。为了缩减代码，我们复用了变量和实例。

一旦创建完所有实例，并完成导航属性的初始化，我们就完成了一个对象图的创建。实体框架保持创建对象图的所有修改，这此跟踪是在数据库上下文中实现的。变量 `context` 包含一个数据库上下文的实例（它的类型是 `DbContext`），它是我们用来跟踪创建对象图所做修改的对象，调用其 `SaveChanges()` 方法，可以将所有修改发送到数据库中。

接来可以查询模型，以验证我们保存在数据库的数据。我们使用了一个新的上下文对象以及使用 LINQ to Entities 来查询。我们本来可以重用之前的上下文对象，但是我们知道它已经在内存中有了对象图，接下来的查询都不会通过数据库就直接从内存中就获取了（译注：这样就达不到验证的目的了）。

使用 LINQ to Entities,我们查询出所有的诗人（`poets`），然后打印出每一位诗人的姓名，以及该诗人的每一首诗的详细信息。代码非常简单，但使用了两个嵌套的循环来实现。

最后一段代码块使用 `VwLibrary` 实体，该实体基于 `vwLibrary` 视图。`vwLibrary` 视图把表连接在一起使其扁平化以提供一个更清晰的视角。我们通过 `vwLibraties` 实体集查询每位诗人的相关信息时，仅需要一次循环。输出有略有不同，因为我们在每首诗中重复了诗人的的姓名。

此示例中最后需要注意的是，我们没有通过视图 `vwLibrary` 插入 `poests`,`poems` 以及 `meters`。因为在绝大多数的数据库中，视图是只读的。我在实体框架中，我们同样不能能过视图实体插入（或者更新、删除）实体。当然，我们将会在本书后面部分给你演示如何克服这个困难！
