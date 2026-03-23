---
aliases:
date: 2015-05-09
author: 付灿
language: C#
sourceurl: https://www.cnblogs.com/VolcanoCloud/p/4482828.html
tags:
  - CSharp
  - 資料庫
  - EF6
---

## 2-3 无载荷（with NO Payload）的多对多关系建模

*问题*

在数据库中，存在通过一张链接表来关联两张表的情况。链接表仅包含连接两张表形成多对多关系的外键，你需要把这两张多对多关系的表导入到实体框架模型中。

*解决方案*

我们设想，你数据库中的表与图 2-10 一样。
![[《Entity Framework 6 Recipes》翻译系列05_图2-10 艺术家和专辑多对多关系.png|图2-10 艺术家和专辑多对多关系]]

按下面的步骤将这些表和关系导入到模型中：

1. 右键你的项目，选择 Add（增加） ➤New Item（新建项），然后选择 Visual C#条目下的 Data 模板下的 ADO.NET Entity Data Model（ADO.NET 实体数据模型）。
2. 选择 Generate from database 从一个已存在的数据库创建模型，点击 Next（下一步）。
3. 可以选择一个已存在的数据库连接，也可以选择新建一个数据库连接。
4. 在选择数据库窗口，选择表 `Album`,`LinkTable`,以及 `Artist`。然后勾选上确定所生成对象名称的单复数形式、在模型中包含外键列复选框。点击 Finish（完成）。（译注：步骤有省略，因为前面的小节已经有详细步骤）
   向导将创建如图 2-11 所示的模型。
   ![[《Entity Framework 6 Recipes》翻译系列05_图2-11 多对多关系模型.png|图2-11 多对多关系模型]]
`Album` 和 `Artist` 之间的多对多关系被表示成两端带字符 * 的直线。因为一份专辑会包含多位艺术家，而一位艺术家可能负责多份专辑。`Album` 和 `Artist` 之间的导航属性类型为 `EntityCollection`.

### 原理

在图 2-11 中，一个 `artit` 能关联多个 `albums`，反之，一个 `album` 也可能关联多个 `artists`。请注意，图 2-10 中的链接表没有出现在模型中。因为他没有标量属性（也就是说，它没有载荷），实体框架认为它存在的唯一目的是关联 `Album` 和 `Artist`。如果这张链接表有标量属性，实体框架将创建一个不同的模型，如下节所示。

代码清单 2-3 演示，如何在我们的模型中插入 albums 和 artists，以及如何从模型中查询出 artists 和他们的 albums,albums 和他们的 artists。

```csharp title="代码清单 2-3. 通过模型中的多对多关系插入和查询我们的 Artists 和 Albums"
using (var context = new EF6RecipesContext())
{

    // 添加一个拥有两张专辑的艺术家
    var artist = new Artist { FirstName = "Alan", LastName = "Jackson" };
    var album1 = new Album { AlbumName = "Drive" };
    var album2 = new Album { AlbumName = "Live at Texas Stadium" };
    artist.Albums.Add(album1);
    artist.Albums.Add(album2);
    context.Artists.Add(artist);

    //添加两个艺术家的专辑
    var artist1 = new Artist { FirstName = "Tobby", LastName = "Keith" };
    var artist2 = new Artist { FirstName = "Merle", LastName = "Haggard" };
    var album = new Album { AlbumName = "Honkytonk University" };
    artist1.Albums.Add(album);
    artist2.Albums.Add(album);
    context.Albums.Add(album);
    context.Artists.Add(artist1);   //译注：需要加上下面两句（原书示例中，没有这两句），否则artist1和artist2不会保存
    context.Artists.Add(artist2);
    context.SaveChanges();
}

using (var context = new EF6RecipesContext())
{
    Console.WriteLine("Artists and their albums...");
    var artists = context.Artists;
    foreach (var artist in artists)
    {
        Console.WriteLine("{0} {1}", artist.FirstName, artist.LastName);
        foreach (var album in artist.Albums)
        {
            Console.WriteLine("\t{0}", album.AlbumName);
        }
    }
    Console.WriteLine("\nAlbums and their artists...");
    var albums = context.Albums;
    foreach (var album in albums)
    {
        Console.WriteLine("{0}", album.AlbumName);
        foreach (var artist in album.Artists)
        {
            Console.WriteLine("\t{0} {1}", artist.FirstName, artist.LastName);
        }
    }
}
```

代码清单 2-3 输出：
```asciidoc
Artists and their albums...
Alan Jackson
Drive
Live at Texas Stadium
Tobby Keith
Honkytonk University
Merle Haggard
Honkytonk University
Albums and their artists...
Drive
Alan Jackson
Live at Texas Stadium
Alan Jackson
Honkytonk University
Tobby Keith
Merle Haggard
```

创建数据库上下文后，我们创建并初始化一个 `Artist` 的实例和两个 `Album` 的实例。然后将 `albums` 增加到 `artist` 的导航属性，并将 `artist` 实例添加到数据库上下文中。

接下来，我们创建并初始化两个实体类型 `Artist` 的实例和一个实体类型 `Album` 的实例。因为两个艺术家合创一张专辑，所以，我们将 album 分别添加到两个 `atists` 的导航属性（它的类型为 `EntityCllection`）中。将 `album` 添加到上下文对象，同时也会将两个 `artists` 也添加到上下文。

现在对象图已经是上下文中的一部份，只需要调用 `SaveChanges()` 方法保存到数据库即可。

在新的上下文对象中查询，获取艺术家并显示出他们的专辑。然后获取专辑并打印出创作他们的艺术家。

注意，我们未提到图 2-10 中的链接表，其实，它没有作为一个实体出现在我们的模型中。链接表代表的多对多关联，我们通过访问 Artists 和 Albums 导航属性即可完成。

## 2-4 有载荷的多对多关系建模

*问题*

数据库中有多对多关系的表，它们通过拥有载荷数据（除外键外的任何列）的表进行关联，你想创建一个代表这种多对多关系的模型，它将创建成两个表示一对多的关联。

*解决方案*

实体框架不支持带有属性的关联，所以创建上一节那样的模型，将不能实现我们的需求。正如上一节中所说的，如果一个链接表只拥有表示关系的外键，实体框架会把链接表呈现为一个关联而不是一个实体类型。如果链接表包含额外的属性，实体框架将呈现两个单独实体类型来表示链接表。结果就是，模型中包含了两个一对多关联的实体类型来表示底层数据库中的链接表。

假设我们拥有如图 2-12 所示的表及其关系
![[《Entity Framework 6 Recipes》翻译系列05_图2-12 有载荷的多对多关系.png|图2-12 有载荷的多对多关系]]

一个订单可以拥有多个订单项，一个订单项可以属于多个订单，在连接 `Order`、`Item` 实例的关系上有一个 `Count` 属性，这个属性被称为一个有效载荷。

按下面的步骤将这些表和关系导入到模型中：

1. 右键你的项目，选择 Add（增加） ➤New Item（新建项），然后选择 Visual C#条目下的 Data 模板下的 ADO.NET Entity Data Model（ADO.NET 实体数据模型）。
2. 选择 Generate from database 从一个已存在的数据库创建模型，点击 Next（下一步）。
3. 可以选择一个已存在的数据库连接，也可以选择新建一个数据库连接。
4. 在选择数据库窗口，选择表 Order,OrderItem,以及 Item。后勾选上确定所生成对象名称的单复数形式、在模型中包含外键列复选框。点击 Finish（完成）。（译注：步骤有省略，因为前面的小节已经有详细步骤）
   向导将创建如图 2-13 所示的模型。
   ![[《Entity Framework 6 Recipes》翻译系列05_图2-13 从一个含有载荷的多对多关系建模成的两个一对多关联.png|图2-13 从一个含有载荷的多对多关系建模成的两个一对多关联]]

### 原理

正如在上一小节中所说的那样，有载荷的多对多关系，在模型中的导航简单明了。因为实体框架不支持在关联上附加载荷，链接表在模型中呈现为一个与其关联实体有两个一对多关联的实体。因此，`OrderItem` 不是被呈现为一个关联，而是一个实体类型，该实体类型与实体类型 `Order` 有一对多的关联，与实体类型 `Iterm` 有一对多的关联。在前一小节，提到的无载荷的链接表在模型中没有被转化为实体类型，而是被转化成了多对多关联的一部份。

因为这个附加的载荷，订单需要通过额外的一级（代表链接表的实体类型）来获取与其相关联的项。如代码清单 2-4 所示。

```csharp title="代码清单2-4 从模型插入和获取 （下面的代码有些问题，你能找出来了吗？）"
using (var context = new EF6RecipesContext())
{
    var order = new Order
    {
        OrderId = 1,
        OrderDate = new DateTime(2010, 1, 18)
    };
    var item = new Item
    {
        SKU = 1729,
        Description = "Backpack",
        Price = 29.97M
    };
    var oi = new OrderItem { Order = order, Item = item, Count = 1 };
    item = new Item
    {
        SKU = 2929,
        Description = "Water Filter",
        Price = 13.97M
    };
    oi = new OrderItem { Order = order, Item = item, Count = 3 };
    item = new Item
    {
        SKU = 1847,
        Description = "Camp Stove",
        Price = 43.99M
    };
    oi = new OrderItem { Order = order, Item = item, Count = 1 };
    context.Orders.Add(order);
    context.SaveChanges();
}

using (var context = new EF6RecipesContext())
{
    foreach (var order in context.Orders)
    {
        Console.WriteLine("Order # {0}, ordered on {1}",
        order.OrderId.ToString(),
        order.OrderDate.ToShortDateString());

        Console.WriteLine("SKU\tDescription\tQty\tPrice");
        Console.WriteLine("---\t-----------\t---\t-----");
        foreach (var oi in order.OrderItems)
        {
            Console.WriteLine("{0}\t{1}\t{2}\t{3}", oi.Item.SKU,
            oi.Item.Description, oi.Count.ToString(),
            oi.Item.Price.ToString("C"));
        }
    }
}
```

**经网友反映，原书中的示例有问题（具体可见后面的评论），从这里可以看出实践的重要，有句话是这么说：“纸上得来终觉浅，绝知此事要躬行”。调整后的代码如下：**

```csharp title="代码清单2-4 从模型插入和获取"
using (var context = new Recipe4Context())
{
    var order = new Order
    {
        OrderId = 1,
        OrderDate = new DateTime(2010, 1, 18)
    };
    var item = new Item
    {
        SKU = 1729,
        Description = "Backpack",
        Price = 29.97M
    };
    var oi1 = new OrderItem { Order = order, Item = item, Count = 1 };
    item = new Item
    {
        SKU = 2929,
        Description = "Water Filter",
        Price = 13.97M
    };
    var oi2 = new OrderItem { Order = order, Item = item, Count = 3 };
    item = new Item
    {
        SKU = 1847,
        Description = "Camp Stove",
        Price = 43.99M
    };
    var oi3 = new OrderItem { Order = order, Item = item, Count = 1 };
    context.OrderItems.Add(oi1);
    context.OrderItems.Add(oi2);
    context.OrderItems.Add(oi3);
    context.SaveChanges();
}

using (var context = new Recipe4Context())
{
    foreach (var order in context.Orders)
    {
        Console.WriteLine("Order # {0}, ordered on {1}",
                            order.OrderId.ToString(),
                            order.OrderDate.ToShortDateString());
        Console.WriteLine("SKU\tDescription\tQty\tPrice");
        Console.WriteLine("---\t-----------\t---\t-----");
        foreach (var oi in order.OrderItems)
        {
            Console.WriteLine("{0}\t{1}\t{2}\t{3}", oi.Item.SKU,
                                oi.Item.Description, oi.Count.ToString(),
                                oi.Item.Price.ToString("C"));
        }
    }
}
```

代码清单 2-4 的输入如下：
```asciidoc
Order # 1, ordered on 1/18/2010
SKU Description Qty Price
---- ----------- --- ------
1729 Backpack 1 $29.97
1847 Camp Stove 1 $43.99
2929 Water Filter 3 $13.97
```

创建数据库上下文实例之后，我们创建了一个订单 (`Order`) 实例、订单项 (`Iterm`) 实例、(`OrderIterm`) 订单关联项实例，我们使用 `OrderItem` 实体的实例连接 `order` 和 `iterms`。最后，我们调用 `Add()` 方法，将 `orderItem` 添加到上下文中。

随着 `orderItem` 被添加到上下文中，对象图创建完成，我们调用 `SaveChanges()` 方法将其更新到数据库。

为了从数据库中获取实体，我创建了一个新的数据库上下文对象实例，然后迭代 contntx.`Orders` 集合，对于每个订单（`order`）（当然，我们在示例中只有一个），通过迭代 `OrderItems` 导航属性，打印出订单的详细信息。 `OrderItem` 实体的实例可以让我们直接访问 Count 标量属性（载荷），订单上的项可以通过导航属性 `Items` 访问。通过 `OrderItems` 实体获取订单项 `items` 增加了一级访问层次，这是有载荷链接表 (示例中的 `OrderIterms`）在多对多关系的代价。

### 最佳实践

不幸的是，项目往往是以无载荷的多对多关系开始，却以多载荷的多对多关系结束。重构一个模型，特别是开发周期的晚期，解决这种问题特别郁闷，不光是增加一个实体，查询和关联中导航模式也要改变。因此有一些开发人建议，每一个多对多关系一开始就包含一些载荷，作为一个合成键。以此减少对项目的影响。

所以，这里有这样一个最佳实践：==如果你有一个无载荷的多对多关系时，你可以考虑通过增加一标识列将其改变为有载荷的多对多关系。==当你导入表到你的模型时，你将得到两个包含一对多关系的实体，这意味着，你的代码为将来有可能出现的多载荷做好了准备。增加一整型标识列的代价通常很小，但给模型带来了更大的灵活性。
