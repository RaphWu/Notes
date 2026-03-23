---
aliases:
date: 2022-10-10
update:
author: "\r于大大大洋"
language: C#
sourceurl: https://blog.csdn.net/arrowzz/article/details/61953193
tags:
  - CSharp
  - 資料庫
  - SQLite
---

# 摘要

本文详细介绍使用 SQLite 与 Entity Framework 进行 CodeFirst 开发的过程，包括配置、常见问题及解决方案。涉及如何正确配置 `App.config` 文件，以及如何处理数据迁移等问题。

# 负二、配置说明

>最近想做个东西，用到了 SQLite，按照之前的方法步骤搭建的结果失败了，因为 SQLite 的版本升级了，导致 Migration 自动迁移各种报错，而且我查了一下自动迁移的包貌是不再更新了。
>—2018 年 1 月 24 日

能正常使用的配置清单如下（不要升级包，升级包会导致一系列的坑）：

- EF 6.1.3
- SQLite.CodeFirst 1.3.1.18
- System.Data.SQLite 1.0.105.2
- System.Data.SQLite.Core 1.0.105.2
- System.Data.SQLite.EF6 1.0.105.2
- System.Data.SQLite.EF6.Migrations 1.0.104
- System.Data.SQLite.Linq 1.0.105.2

文章最底下可以下载源码，能正常使用，但是 SQLite 升级到了 1.0.106 之后，就不能再使用原来的 `System.Data.SQLite.EF6.Migrations` 了。

# 负一、吐个槽

关于 SQLite 的 CodeFirst，我找了很久，有很多博客都写过，但是真正好用的非常少，还有很多根本就是 DBFirst 的代码，真是坑的我够呛，研究了几天也算有点成果，整理一下，希望对路过的朋友有所帮助。

# 零、冷知识

## 1、SQLite.CodeFirst

使用 NuGet 安装的 EF 和 SQLite 可能是没有 CodeFirst 功能的，安装了“`SQLite.CodeFirst`”之后可以为 SQLite 提供 CodeFirst 的功能。

## 2、System.Data.SQLite.EF6.Migrations 数据库迁移

SQLite 是没有数据库迁移 `MigrationSqlGenerator` 功能的 ，我所查询的资料是这样的（有错请指正，非常感谢），甚至有人说得自己写一个数据迁移的功能，那是不是有点辛苦。

还好在 NuGet 上有一个“`System.Data.SQLite.EF6.Migrations`”（详细内容可以访问他的网站，网站上有说明文档），安装之后可以做简单的数据库修改，可以添加表中的列，但是并不能删除和修改列，因为 SQLite 并不认识 `drop column` 和 `rename column`，至于为什么不认识看下条。

## 3、SQLite 不支持的语法

关于这个，可以自己百度查一下 ^ _ ^ ~

## 4、关于 SQLite 中的数据关系

我在试验的过程中，发现 SQLite 的数据关系并不可靠，尤其是 EF 的子查询我就没有尝试成功，还有使用外键的时候也并不正常。

## 5、SQLite 中的 Guid

在 SQLite 中使用 Guid 的时候，因为 ==SQLite 中 Guid 会使用 BLOB 类型来存储（就是二进制）==，所以用 Linq 直接 ` Id==id` 是查不到东西的，使用 tostring 也得不到正确的格式，==建议在使用 EF SQLite 的时候，使用 `string` 来存储 Guid 数据。==

使用 SQLite 读取工具打开数据库文件，就会发现 Guid 是 BLOB 存储的，并且顺序和 C# 生成的 Guid 并不相同。（如果非要使用 Guid 可以研究一下 Guid 和二进制的转换。）

# 一、安装过程

OK 进入正题：

使用 NuGet 安装时，相关联的都会自动下载，所以这点还是很方便的，这里列一下安装完之后都有啥：

- EntityFramework
- EntityFramework.zh-Hans（中文说明）
- SQLite.CodeFirst（实现 CodeFirst）
- System.Data.SQLite
- System.Data.SQLite.Core
- System.Data.SQLite.EF6
- System.Data.SQLite.EF6.Migrations（提供数据库迁移）
- System.Data.SQLite.Linq

安装过程就这样……

# 二、简单配置一下

虽然安装的时候会自动写入一些配置信息，但是还是可能有些遗漏，查漏补缺一下：

`App.config` 文件：

`entityFramework`->`providers` 下检查添加：

```xml
<provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
```

`connectionStrings` 下检查添加：

```xml
<add name="DefaultConnection" connectionString="data source=sqliter.db" providerName="System.Data.SQLite" />
```

# 三、数据库工具类

一个简单例子只需要 4 个类：

1. Entity 类（实体类）
2. Mapping 类（实体配置）
3. DbContext 类（数据库操作）
4. Configuration 类（设置信息）

首先来一个简单“栗子”：

## Entity 类

```csharp
public class Bus
{
    public string Id { get; set; }
    public string Name { get; set; }
}

public class Person
{
    public string Id { get; set; }
    public string Name { get; set; }
}
```

## Mapping 类

```csharp
public class Mapping
{
    public class BusMap : EntityTypeConfiguration<Bus>
    {
        public BusMap()
        {
        }
    }
    public class PersonMap : EntityTypeConfiguration<Person>
    {
        public PersonMap()
        {
        }
    }
}
```

## DBContext 类

```csharp
public class SQLiteDb : DbContext
{
    public SQLiteDb() : base("DefaultConnection")
    {
        Database.SetInitializer(new MigrateDatabaseToLatestVersion<SQLiteDb, Configuration>());
    }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
        modelBuilder.Configurations.AddFromAssembly(typeof(SQLiteDb).Assembly);
    }
}
```

## Configuration 类

```csharp
public class Configuration : DbMigrationsConfiguration<SQLiteDb>
{
    public Configuration()
    {
        AutomaticMigrationsEnabled = true;
        AutomaticMigrationDataLossAllowed = true;
        SetSqlGenerator("System.Data.SQLite", new SQLiteMigrationSqlGenerator());
    }

    protected override void Seed(SQLiteDb context)
    {
        //  This method will be called after migrating to the latest version.

        //  You can use the DbSet<T>.AddOrUpdate() helper extension method 
        //  to avoid creating duplicate seed data. E.g.
        //
        //    context.People.AddOrUpdate(
        //      p => p.FullName,
        //      new Person { FullName = "Andrew Peters" },
        //      new Person { FullName = "Brice Lambson" },
        //      new Person { FullName = "Rowan Miller" }
        //    );
        //
    }
}
```

# 四、操作代码

```csharp
public partial class Form1 : Form
{
    //创建 Bus
    Bus Bus = new Bus() { Id = Guid.NewGuid().ToString(), Name = "11 路 " };
    public Form1()
    {
        InitializeComponent();
    }
    private void Form1_Load(object sender, EventArgs e)
    {
        //默认往数据库添加一个 Bus
        using (SQLiteDb db = new SQLiteDb())
        {
            db.Set<Bus>().Add(Bus);
            db.SaveChanges();
        }
    }
    //添加人员
    void InsertPerson(string name)
    {
        using (SQLiteDb db = new SQLiteDb())
        {
            Bus dbBus = db.Set<Bus>().Where(x => x.Id == Bus.Id).FirstOrDefault();
            db.Set<Person>().Add(new Person() { Id = Guid.NewGuid().ToString(), Name = name, Bus = dbBus });
            db.SaveChanges();
        }
    }
    //删除人员
    void DeletePerson(string id)
    {
        using (SQLiteDb db = new SQLiteDb())
        {
            Person p = new Person() { Id = id };
            Person person = db.Set<Person>().Attach(p);
            db.Set<Person>().Remove(person);
            db.SaveChanges();
        }
    }
    //更新人员
    void UpdatePerson(string id, string name)
    {
        using (SQLiteDb db = new SQLiteDb())
        {
            Person p = db.Set<Person>().Where(x => x.Id == id).FirstOrDefault();
            p.Name = name;
            db.SaveChanges();
        }
    }

    #region 点击按钮操作
    private void BtInsert_Click(object sender, EventArgs e)
    {
        //清空文本框
        TbTxt.Text = "";
        //插入人员
        InsertPerson(TbInsert.Text);
        //显示人员信息
        using (SQLiteDb db = new SQLiteDb())
        {
            List<Person> persons = db.Set<Person>().ToList();
            if (persons != null)
            {
                persons.ForEach(x =>
                {
                    TbTxt.Text += x.Id + "  " + x.Name + Environment.NewLine;
                });
            }
        }
    }
    private void BtDelete_Click(object sender, EventArgs e)
    {
        TbTxt.Text = "";
        DeletePerson(TbDelete.Text);
        using (SQLiteDb db = new SQLiteDb())
        {
            List<Person> persons = db.Set<Person>().ToList();
            if (persons != null)
            {
                persons.ForEach(x =>
                {
                    TbTxt.Text += x.Id + "  " + x.Name + Environment.NewLine;
                });
            }
        }
    }
    private void BtUpdate_Click(object sender, EventArgs e)
    {
        TbTxt.Text = "";
        UpdatePerson(TbUpdate.Text, DateTime.Now.ToString("mm:ss"));
        using (SQLiteDb db = new SQLiteDb())
        {
            List<Person> persons = db.Set<Person>().ToList();
            if (persons != null)
            {
                persons.ForEach(x =>
                {
                    TbTxt.Text += x.Id + "  " + x.Name + Environment.NewLine;
                });
            }
        }
    }
    private void BtSelect_Click(object sender, EventArgs e)
    {
        TbTxt.Text = "";
        if (TbSelect.Text == "")
        {
            //查询所有人员信息
            using (SQLiteDb db = new SQLiteDb())
            {
                List<Person> persons = db.Set<Person>().ToList();
                if (persons != null)
                {
                    persons.ForEach(x =>
                    {
                        TbTxt.Text += x.Id + "  " + x.Name + Environment.NewLine;
                    });
                }
            }
        }
        else
        {
            //根据Id查询人员
            using (SQLiteDb db = new SQLiteDb())
            {
                Person person = db.Set<Person>().Where(x => x.Id == TbSelect.Text).FirstOrDefault();
                TbTxt.Text = person.Id + "  " + person.Name + Environment.NewLine;
            }
        }
    }
    #endregion
}
```

长了点，但一看就懂的。

# 五、异常处理

## 一、“System.InvalidOperationException”类型的未经处理的异常在 EntityFramework.dll 中发生

其他信息: No Entity Framework provider found for the ADO.NET provider with invariant name ‘System.Data.SQLite’. Make sure the provider is registered in the ‘entityFramework’ section of the application config file. See http://go.microsoft.com/fwlink/?LinkId=260882 for more information.

解决：`App.config` 中，使用 NuGet 安装了 SQLite，SQLite.Core,EF……之后，默认的配置是这样的：

```xml
<providers>
  <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
  <provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
</providers>
```

而异常提示 provider with invariant name ‘System.Data.SQLite’，所以只需要加上这行 `provider` 配置就可以。

```xml
<providers>
  <provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
</providers>
```

## 二、“System.InvalidOperationException”类型的未经处理的异常在 EntityFramework.dll 中发生

其他信息: Unable to determine the principal end of an association between the types ‘SQLiter.TestEF.DbTool.IdCard’ and ‘SQLiter.TestEF.DbTool.Student’. The principal end of this association must be explicitly configured using either the relationship fluent API or data annotations.

解决：在 MVC 中如果数据库表关系是一对一的，需要加 `[Required]` ，如：

```csharp
public class Employee
{
    //员工必须有工牌
    [Required]
    public virtual ECard ECard { get; set; }
}
```

## 三、“System.Data.Entity.Validation.DbEntityValidationException”类型的未经处理的异常在 EntityFramework.dll 中发生

其他信息: Validation failed for one or more entities. See ‘EntityValidationErrors’ property for more details.

解决：检查是否有字段是必须的。

注：可以通过 catch 来获取更详细的提示：

```csharp
catch (DbEntityValidationException dbEx)
{
    foreach (var validationErrors in dbEx.EntityValidationErrors)
    {
        foreach (var validationError in validationErrors.ValidationErrors)
        {
            Console.WriteLine(
                    "Class: {0}, Property: {1}, Error: {2}",
                    validationErrors.Entry.Entity.GetType().FullName,
                    validationError.PropertyName,
                    validationError.ErrorMessage);
        }
    }
}
```
