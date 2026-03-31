---
aliases:
date: 2024-10-31
update:
author: 孤沉
language: C#
sourceurl: https://www.cnblogs.com/guchen33/p/18518933
tags:
  - CSharp
  - 資料庫
  - EF6
  - SQLite
---

# 使用 EF6 连接 Sqlite

## 1、前置条件

安装以下包

```plantext
EntityFramework
System.Data.Sqlite
```

以上包会自动生成或填充 `Config` 文件
`App.Config` 配置如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<configSections>
		<!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
		<section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
	</configSections>
	<connectionStrings>
		<add name="Sqlite" connectionString="data source=E:\CSharp Project\ApplyClient\IgniteApp\IgniteShared\Db\demo.db;;version=3;" providerName="System.Data.SQLite.EF6"/>
	</connectionStrings>
	<startup>
		<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.8" />
	</startup>
	<entityFramework>
		<providers>
			<provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
			<provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
			<provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
		</providers>
	</entityFramework>
	<system.data>
		<DbProviderFactories>
			<remove invariant="System.Data.SQLite.EF6" />
			<add name="SQLite Data Provider (Entity Framework 6)" invariant="System.Data.SQLite.EF6" description=".NET Framework Data Provider for SQLite (Entity Framework 6)" type="System.Data.SQLite.EF6.SQLiteProviderFactory, System.Data.SQLite.EF6" />
			<remove invariant="System.Data.SQLite" />
			<add name="SQLite Data Provider" invariant="System.Data.SQLite" description=".NET Framework Data Provider for SQLite" type="System.Data.SQLite.SQLiteFactory, System.Data.SQLite" />
		</DbProviderFactories>
	</system.data>
</configuration>
```

## 2、包安装完成后实现代码

```csharp
 public class AccessDbContext : DbContext
 {
     public DbSet<MaterialInfo> MaterialInfos { get; set; }
     public AccessDbContext() : base("Sqlite")
     {

     }
 }
```

### **注意 1**

```cpp
 public AccessDbContext() : base("Sqlite")
 {

 }
```

构造器中 Sqlite 指的是

```csharp
ConfigurationManager.ConnectionStrings["Sqlite"].ConnectionString;
```

它是默认读取出来的，你需要填写 Config 文件中 connectionStrings 的 name

### **注意 2**

此时 Sqlite 的控制权会在 ADO 和 EF6 ORM 框架之间争夺，所以程序会报错，
因为我们安装了 EF6，所以不需要你手动写 `SqliteCommand` 之类的东西，
但是你必须将控制权全权交给 EF6 去控制

```plantext
System.InvalidOperationException:“No Entity Framework provider found for the ADO.NET provider with invariant name 'System.Data.SQLite'. Make sure the provider is registered in the 'entityFramework' section of the application config file. See http://go.microsoft.com/fwlink/?LinkId=260882 for more information.”
```

#### System.Data.SQLite.EF6

这是专门为 EF6 设计的 SQLite 数据提供**程序**。
它告诉 EF6 如何处理 SQLite 数据库连接。

#### System.Data.SQLite

这是通用的 SQLite 数据提供程序**标识符**。
即使你使用的是 EF6，EF6 仍然需要知道如何处理通用的 System.Data.SQLite 标识符。
这是因为 ==EF6 依赖于 ADO.NET 的数据提供程序系统==，而 System.Data.SQLite 是 ADO.NET 系统中定义的标识符。
所以你的 Config 文件必须两个都写

```xml
<provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
<provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
```

## 3、打开 Navicat

导入 Config 目录下的 db 文件
新建表
实现代码

```csharp
AccessDbContext _dbContext = new AccessDbContext();
var info = new MaterialInfo();
info.Id = 1;
info.Name="test";
_dbContext.MaterialInfos.Add(info);
_dbContext.SaveChanges(); // 保存缓存操作
var ListRes = _dbContext.MaterialInfos.Where(t => t.Name == "test").ToList();
```

## 4、你觉得自己新建表新建数据库太慢了

此时你需要一个新的包: SQLite.CodeFirst
它的功能是自动创建数据库，自动创建表
这个时候，你需要继续修改 Config 文件

```xml
<connectionStrings>
	<add name="Sqlite" connectionString="data source=demo.db;version=3;" providerName="System.Data.SQLite.EF6"/>
</connectionStrings>
```

然后使用 SQLite.CodeFirst

```csharp
 public class AccessDbContext : DbContext
 {
     public DbSet<MaterialInfo> MaterialInfos { get; set; }
     public AccessDbContext() : base("Sqlite")
     {

     }
     protected override void OnModelCreating(DbModelBuilder modelBuilder)
     {
         var sqliteConnectionInitializer = new SqliteCreateDatabaseIfNotExists<AccessDbContext>(modelBuilder);
         Database.SetInitializer(sqliteConnectionInitializer);
     }
 }
```

修改 Config 的作用是，SQLite.CodeFirst 会在你程序的 bin 目录下自动生成 demo.db（你起的名称）
如果写成路径，程序不会报错，但是不会自动生成数据库和表

## 5、当然，你也可以使用 SQLite.CodeFirst 自动填充数据

```csharp
// 自定义数据库初始化器
public class MyDbContextInitializer : SqliteDropCreateDatabaseAlways<AccessDbContext>
{
    public MyDbContextInitializer(DbModelBuilder modelBuilder)
        : base(modelBuilder) { }

    protected override void Seed(AccessDbContext context)
    {
        // 在这里填充核心数据或测试数据
        context.Set<MaterialInfo>().Add(new MaterialInfo { Name = "John Doe", Id = 30 });
        context.SaveChanges();
    }
}

internal class Program
{
    static void Main(string[] args)
    {
        Database.SetInitializer(new MyDbContextInitializer(new DbModelBuilder()));
        AccessDbContext _dbContext = new AccessDbContext();
        var info = new MaterialInfo();
        info.Id = 1;
        info.Name = "test";
        for (var i = 0; i < 10; i++)
        {
            new MaterialInfo
            {
                Name = "小李",
                Id = i,
            };
        }
        _dbContext.MaterialInfos.Add(info);
        _dbContext.SaveChanges(); // 保存缓存操作

        var ListRes = _dbContext.MaterialInfos.Where(t => t.Name == "test").ToList();

        Console.WriteLine("-----完成---");
    }
}
```

本文作者：孤沉
本文链接：https://www.cnblogs.com/guchen33/p/18518933
版权声明：本作品采用知识共享署名 - 非商业性使用 - 禁止演绎 2.5 中国大陆 [许可协议](https://www.cnblogs.com/guchen33/p/18518933) 进行许可。
