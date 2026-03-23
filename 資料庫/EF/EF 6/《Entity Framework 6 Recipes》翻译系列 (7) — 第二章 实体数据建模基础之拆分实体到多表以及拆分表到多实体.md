---
aliases:
date: 2015-05-11
author: 付灿
language: C#
sourceurl: https://www.cnblogs.com/VolcanoCloud/p/4488247.html
tags:
  - CSharp
  - 資料庫
  - EF6
---

## 2-6 拆分实体到多表

*问题*

你有两张或是更多的表，他们共享一样的主键，你想将他们映射到一个单独的实体。

*解决方案*

让我们用图 2-15 所示的两张表来演示这种情况。
![[《Entity Framework 6 Recipes》翻译系列07_图2-15 两张表.png|图2-15，两张表，Prodeuct 和 ProductWebInfo，拥有共同的主键]]

按下面的步骤为这两张表建模一个单独实体：

1. 在你的项目中，创建一个继承至 `DbContext` 的上下文对象 `EF6RecipesContext`；
2. 使用代码清单 2-8 创建一个 POCO 实体 `Product`；　

```csharp title="代码清单2-8 创建一个 POCO 实体 Product"
public class Product
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int SKU { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public string ImageURL { get; set; }
}
```

3. 在 `EF6RecipesContext` 中添加类型为 `DbSet<Product>` 的属性 `Products`；
4. 使用代码清单 2-9 在 `EF6RecipesContext` 中重写 `OnModelCreating()` 方法；

```csharp title="代码清单2-9 重写 OnModelCreating() 方法"
public class EF6RecipesContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public EF6RecipesContext()
        : base("name=EF6CodeFirstRecipesContext")
    {
    }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.Entity<Product>()
        .Map(m => {
            m.Properties(p => new { p.SKU, p.Description, p.Price });
            m.ToTable("Product", "Chapter2");
        })
        .Map(m => {
            m.Properties(p => new { p.SKU, p.ImageURL });
            m.ToTable("ProductWebInfo", "Chapter2");
        });
    }
}
```

### 原理

这种情况常见于遗留系统中，一个表中的每一行都包含额外的，本该属于另一张表的信息。随着数据库变化，这样情况经常发生。没有人愿意去打破现有的代码，而是通过在一个关键的表中添加一些列来解决问题。处理这种情况的答案是，建一张新表来“移植”这对额外的列。

通合并两张或多张并到一个单独的实体，通常也被叫作分拆一个实体到两张或多张数据库表，我可以把每个组成部分当成一个逻辑实体。这过程叫做垂直分拆。

垂直分拆的缺点在，我们获取实体类型实例时，分拆的表需要一个额外的 `join`（连接）来构建实体类型。这个额外的 join 如清单 2-10 所示：

```sql title="清单 2-10 垂直分拆需要额外的 Join 连接"
SELECT
	[Extent1].[SKU] AS [SKU],
	[Extent2].[Description] AS [Description],
	[Extent2].[Price] AS [Price],
	[Extent1].[ImageURL] AS [ImageURL]
FROM [dbo].[ProductWebInfo] AS [Extent1]
INNER JOIN [dbo].[Product] AS [Extent2] ON [Extent1].[SKU] = [Extent2].[SKU]
```

插入和获取 `Product` 实体没有特别的要求。代码清单 2-11 演示了操作被垂直分拆的 `Product` 实体类型

```csharp title="代码清单2-11"
using (var context = new EF6RecipesContext())
{
    var product = new Product
    {
        SKU = 147,
        Description = "Expandable Hydration Pack",
        Price = 19.97M,
        ImageURL = "/pack147.jpg"
    };
    context.Products.Add(product);
    product = new Product
    {
        SKU = 178,
        Description = "Rugged Ranger Duffel Bag",
        Price = 39.97M,
        ImageURL = "/pack178.jpg"
    };
    context.Products.Add(product);
    product = new Product
    {
        SKU = 186,
        Description = "Range Field Pack",
        Price = 98.97M,
        ImageURL = "/noimage.jp"
    };
    context.Products.Add(product);
    product = new Product
    {
        SKU = 202,
        Description = "Small Deployment Back Pack",
        Price = 29.97M,
        ImageURL = "/pack202.jpg"
    };
    context.Products.Add(product);
    context.SaveChanges();
}
using (var context = new EF6RecipesContext())
{
    foreach (var p in context.Products)
    {
        Console.WriteLine("{0} {1} {2} {3}", p.SKU, p.Description,
        p.Price.ToString("C"), p.ImageURL);
    }
}
```

代码清单 2-11 的输出如下：
```asciidoc
147 Expandable Hydration Pack $19.97 /pack147.jpg
178 Rugged Ranger Duffel Bag $39.97 /pack178.jpg
186 Range Field Pack $98.97 /noimage.jpg
202 Small Deployment Back Pack $29.97 /pack202.jpg
```

## 2-7 分拆一张表到多个实体

*问题*

你有这样的一张数据库表，里面包含经常使用的字符，一些不常用的大字段。为了性能，需要避免每个查询都去加载这些字段。你需要将这张表分拆成两个或是更多的实体。

*解决方案*

我们假设你有一张如图 2-16 的表，它存储照片的信息，以及照片的缩略图和全分辨率图。
![[《Entity Framework 6 Recipes》翻译系列07_图2-16  Photograph表.png|图2-16 Photograph 表，有一个二进制的大对象字段，保存图像数据]]

按下面的步骤创建一个包含成本合理且经常使用列的实体，同时创建一个包含成本高且极少使用的高分辨位列的实体：

1. 在你的项目中创建一个继承自 `DbContext` 的上下文对象 `EF6RecipesContext`；
2. 使用代码清单 2-12 创建一个 POCO 实体 `Photograph`；

```csharp title="代理清单 2-12 创建一个 POCO 实体 Photograph"
public class Photograph
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int PhotoId { get; set; }
    public string Title { get; set; }
    public byte[] ThumbnailBits { get; set; }
    [ForeignKey("PhotoId")]
    public virtual PhotographFullImage PhotographFullImage { get; set; }
}
```

3. 使用代码清单 2-13 创建一个 POCO 实体 `PhotographFullImage`；

```csharp title="代理清单2-13 创建一个 POCO 实体 PhotographFullImage"
public class PhotographFullImage
{
    [Key]
    public int PhotoId { get; set; }
    public byte[] HighResolutionBits { get; set; }
    [ForeignKey("PhotoId")]
    public virtual Photograph Photograph { get; set; }
}
```

4. 在上下文对象 `EF6RecipesContext` 中添加 `DbSet<Photograph>` 属性；
5. 在上下文对象 `EF6RecipesContext` 中添加另一个 `DbSet<PhotographFullImage>` 属性;
6. 使用代码清单 2-14 重写上下文对象中的 `OnModelCreating()` 方法；

```csharp title="代码清单2-14 重写上下文对象中的 OnModelCreating() 方法"
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    modelBuilder.Entity<Photograph>()
                .HasRequired(p => p.PhotographFullImage)
                .WithRequiredPrincipal(p => p.Photograph);
    modelBuilder.Entity<Photograph>().ToTable("Photograph", "Chapter2");
    modelBuilder.Entity<PhotographFullImage>().ToTable("Photograph", "Chapter2");
}
```

### 原理

实体框架不直接支持延迟加载某个单一的实体属性。为了得到延迟加载成本昂贵属性的好处，利用实体框架延迟加载关联实体的特性，我们创建一个新的，包含成本昂贵的保存完整图像列的实体 PhotographFullImage,一个 `Photograph` 实体和 `PhotographFullImange` 实体之单的关联。并且我们在概念层添加一个跟数据库引用约束相似的约束，告诉实体框架一个 `PhotographFullImage` 不能离开 `Photograph` 而独立存在。

由于引用约束的存在，在模型中，我们有两件需要注意的事：一个是，当我们新建一个 `PhotographFullImage` 实体的实例或者调用 `SaveChages()` 方法之前，`Photogrpah` 的实例必须存在上下文中。第二个是，如果我删除一个 `photograph`，与之关联的 `photographFullImage` 也会被删除，这有点像是数据库中引用约束的级联删除。

代码清单 2-15 演示从模型中插入和获取数据。

```csharp title="代码清单2-15 插入和延迟加载成本昂贵的字段"
byte[] thumbBits = new byte[100];
byte[] fullBits = new byte[2000];
using (var context = new EF6RecipesContext())
{
    var photo = new Photograph
    {
        Title = "My Dog",
        ThumbnailBits = thumbBits
    };
    var fullImage = new PhotographFullImage { HighResolutionBits = fullBits };
    photo.PhotographFullImage = fullImage;
    context.Photographs.Add(photo);
    context.SaveChanges();
}
using (var context = new EF6RecipesContext())
{
    foreach (var photo in context.Photographs)
    {
        Console.WriteLine("Photo: {0}, ThumbnailSize {1} bytes",
            photo.Title, photo.ThumbnailBits.Length);

        //显式加载存储完整图像的字段
        context.Entry(photo).Reference(p => p.PhotographFullImage).Load();
        Console.WriteLine("Full Image Size: {0} bytes",
            photo.PhotographFullImage.HighResolutionBits.Length);
    }
}
```


代码清单 2-15 的输出如下：
```asciidoc
Photo: My Dog, Thumbnail Size: 100 bytes
Full Image Size: 2000 bytes
```

代码清单 2-15 创建并初始化实体 `Photograph` 和 `PhotographFullmage` 的实例对象，并将他们添加到上下文对象中，然后调用方法 `SaveChanges()` 保存。

在查询中，我们获取数据库中每一个 `photograph`，打印它们的信息，并显示加载与之关系的实体 `PhotographFullImage`。注意，我们没有关闭上下文中默认的延迟加载选项，这正是我们需要的。我们可以选择不去加载 `PhotographFullImage` 的实例，如果获取成百上千张的照片，这将为我们节约大量的时间和带宽。
