---
aliases:
date: 2015-05-10
author: 付灿
language: C#
sourceurl: https://www.cnblogs.com/VolcanoCloud/p/4487504.html
tags:
  - CSharp
  - 資料庫
  - EF6
---

## 2-5 使用 Code First 建模自引用关系

*问题*

你的数据库中一张自引用的表，你想使用 Code First 将其建模成一个包含自关联的实体。

*解决方案*

我们假设你有如图 2-14 所示的数据库关系图的自引用表。
![[《Entity Framework 6 Recipes》翻译系列06_图2-14 一张自引用表.png|图2-14 一张自引用表]]

按下面的步骤为这张自引用的表及关系建模：

1. 在项目中创建一个继承至 `DbContext` 上下文的类 `EF6RecipesContext`。
2. 使用代码清单 2-5 创建一个 `PictureCategoryPOCO`（简单 CLR 对象）实体。

```csharp title="代码单清2-5 创建一个 POCO 实体 PictureCategory"
[Table("PictureCategory", Schema = "Chapter2")] //书中没有这一句，示例会把数据保存到dbo.PictureCategory表里，
												//而不是Chapter2.PictureCategory里，以致不少朋友认为没有保存成功，加上这一句就不会有问题了　　　　
public class PictureCategory
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int CategoryId { get; private set; }
    public string Name { get; set; }
    public int? ParentCategoryId { get; private set; }
    [ForeignKey("ParentCategoryId")]
    public virtual PictureCategory ParentCategory { get; set; } //书中没有virtual关键字，这会导致导航属性不能加载，后面的输出就只有根目录！！
    public virtual List<PictureCategory> Subcategories { get; set; }
    public PictureCategory()
    {
        Subcategories = new List<PictureCategory>();
    }
}
```

3. 在创建的上下文对象 `EF6RecipesContext` 中添加一个 `DbSet<PictureCategory>` 属性。
4. 在 `EF6RecipesContext` 中重写方法 `OnModelCreating` 配置双向关联 (`ParentCategory` 和 `SubCategories`），如代码清单 2-6 所示。

```csharp title="代理清单2-6 重写方法 OnModelCreating"
public class EF6RecipesContext : DbContext
{
    public DbSet<PictureCategory> PictureCategories { get; set; }
    public EF6RecipesContext()   //原文这里有误，被写成PictureContext()
        : base("name=EF6CodeFirstRecipesContext")
    {
    }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.Entity<PictureCategory>()
                    .HasMany(cat => cat.SubCategories)
                    .WithOptional(cat => cat.ParentCategory);
    }
}
```

### 原理

数据库的关系有以下特征：**维度 (degree)**、**多重性 (multiplicity)** 以及 **方向 (derection)**。

维度是指关系中的实体（表）的数量。一维和二维关系是常见的。三维和 N 维 (n-Place）关系只存在于理论上。

多重性，是指表示关系的线段两端的实体类型（译注：这里应该是指表，因为实体类型用于模型中）数量。你可能已经看到这样的多重性表示，0...1（零或者一），1（一）和 \*（很多）。

最后，方向可以是双向，也可以是单向。

实体数据模型支持当前流行数据库的数据库关系，它通过一个名为关联的类型来表示。一个关联类型可以是一维或者二维的，多重性可以是 0...1，1 和 \*，方向是双向的。

示例中的维度是一维（只涉及 `PictureCategory` 实体），多重性是 0...1 和 \*，方向当然是双向。

示例中的情况，自引用表一般指父子关系，每个父亲有多个孩子，同时，一个孩子只有一个父亲。因为父亲这端的关系多重性是 0...1 而不是 1，这对于孩子来说意味着它可能没有父亲。这正好可以被利用来表示根节点。一个没有父亲的节点，它是整个继承层次的顶端。

代码清单 2-7 演示，通过递归从根节点开始枚举图片目录。当然根节点是一个没有父亲的节点。
```csharp title="代码清单2-7"
static void RunExample()
{
    using (var context = new EF6RecipesContext())
    {
        var louvre = new PictureCategory { Name = "Louvre" };
        var child = new PictureCategory { Name = "Egyptian Antiquites" };
        louvre.Subcategories.Add(child);
        child = new PictureCategory { Name = "Sculptures" };
        louvre.Subcategories.Add(child);
        child = new PictureCategory { Name = "Paintings" };
        louvre.Subcategories.Add(child);
        var paris = new PictureCategory { Name = "Paris" };
        paris.Subcategories.Add(louvre);
        var vacation = new PictureCategory { Name = "Summer Vacation" };
        vacation.Subcategories.Add(paris);
        context.PictureCategories.Add(paris);
        context.SaveChanges();
    }
    using (var context = new EF6RecipesContext())
    {
        var roots = context.PictureCategories.Where(c => c.ParentCategory == null);
        roots.ForEach(root => Print(root, 0));
    }
}
static void Print(PictureCategory cat, int level)
{
    StringBuilder sb = new StringBuilder();
    Console.WriteLine("{0}{1}", sb.Append(' ', level).ToString(), cat.Name);
    cat.Subcategories.ForEach(child => Print(child, level + 1));
}
```

代码清单 2-7 输出显示，根结点为 `Summer Vacation`。它的第一个（只有一个）孩子是 `Paris`。 `Paris` 有孩子 `Louver`。最后，我在 `Louver` 照片目录访问到了目录集合。
```asciidoc
Summer Vacation
  Paris
    Louvre
      Egyptian Antiquities
      Sculptures
      Paintings
```

显然，代码稍微有点点复杂了，我们最开始创建并初始化多个实体类型的实例，通过将这些照片目录一起增加到目录 `louver` 来将它们添加进对象图，然后中我们将 `louver` 目录添加到 `paris` 目录，最后我们将 `paris` 目录添加到 `summer vacation` 目录。我们从下到上构建了整个继承体系。

一旦调用 `SaveChange()` 方法，所有的目录都将插入到数据库，我可以查询表中的数据，看是不是所有的行都被正确地插入了。

对于获取部分代码，我们最开始获取一个根实体，它是一个没有父亲的目录，在例中，我们创建了一个 `summer vacation` 实体，但并没有把它设置成任何实体的孩子。这让它成为整个继承体系的根节点。

现在，从根节点开始，我们调用另一个我们编写的方法:`Print()`，`Print()` 方法接受一对参数，第一个参数是一个 `PicturCategory` 实例对象，第二个参数是一个在继承体系中表示层级或深度的整型。对于根目录，`summer vacation`，它在继承体系的顶端，我们传 `0` 给 `print()` 方法. 方法调用会是这样 `Prin(root,0)`。

在 `Print()` 方法中，我们输出目录的名称，并在名称前根据目录在继承体系所处的深度，加上相应数量的前导空格。`StringBuilder` 类的方法 `Append()` 接受两个参数，一个是字符和一个整型，他创建一个 `StringBuilder` 实例，并附加整型参数指定数量的字符。在我们的调用中，我们使用空格和目录的深度（level）作为参数，他返回一个目录深度数量的空格字符串。我们调用 `StringBuilder` 的 `Sostring()` 方法，将 `StringBuilder` 实例转换成一个 `string` 实例。

现在到了递归部分，我们通过 `children` 迭代孩子目录，为每一个孩子目录调用 `Print()` 方法，并确保 `Levle` 递增。 当遍历完 `children`，我们就返回。最终的结果如前面的输出。

在 6-5 中，我们会展示另一个方法，存储过程中使用表表达式，在存储端能过关系图迭代，然后返回一个扁平化的结果集。
