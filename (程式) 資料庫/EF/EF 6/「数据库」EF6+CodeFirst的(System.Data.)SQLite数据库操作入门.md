---
aliases:
date: 2019-11-05
update:
author: Raink_LH
language: C#
sourceurl: https://blog.csdn.net/Raink_LH/article/details/102912670
tags:
  - CSharp
  - 資料庫
  - SQLite
---

在 Web 开发中，应用程序、网页等对数据库的的访问都是对固定数据库的访问，程序的每一次打开，网页的每一次登录等，都是在访问同一个固定数据库。

但是在桌面程序中，大多数时候都需要在本地创建数据库，并且随着软件使用情况，可能需要创建多个数据库，比如根据月份的数据库等，这该如何操作呢？

EF+CodeFirst 的方式，常把数据库的连接字符串写在【`App.config`】文件中，并且需要使用 `Add-Migration` 的指令来生成数据库，这样想想其实有点鸡肋，如果我写一个程序，用户拿去在自己电脑上用，还得先装数据库，还得执行 `Add-Migration` 指令，用户估计会疯掉。

那么有没有更加灵活的方式，在开发中能使用 EF6+CodeFirst 模式，程序启动时会自动创建本地数据库。

折腾了两天，终于勉强实现了。记录一下方法吧

# 1、安装依赖包

用控制台项目为例，创建一个.Net Framework 的控制台项目，设置项目名称【SqLiteCoFirst】

接着打开 NuGet

需要安装的库包括如下三个，2 在安装时会安装其他几个依赖的项目。

```text
1、EntityFramework
2、System.Data.SQLite
	System.Data.SQLite.Linq
	System.Data.SQLite.Core
	System.Data.SQLite.EF6
3、SQLite.CodeFirst
```

# 2、配置 App.config

安装好依赖库之后，打开项目下的【`App.config`】文件，

修改【`entityFramework`】下的【`providers`】节点内的设置

默认情况下应该是

```xml
<provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
```

我们需要把这一条设置改为 sqlite 的设置，如下：

```xml
<provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
```

修改完后，这个【`App.config`】如下供参考：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
  </startup>
  <entityFramework>
    <providers>
      <!--<provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />-->
      <provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
      <provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
    </providers>
  </entityFramework>
  <system.data>
    <DbProviderFactories>
      <remove invariant="System.Data.SQLite.EF6" />
      <add name="SQLite Data Provider (Entity Framework 6)" invariant="System.Data.SQLite.EF6" description=".NET Framework Data Provider for SQLite (Entity Framework 6)" type="System.Data.SQLite.EF6.SQLiteProviderFactory, System.Data.SQLite.EF6" />
   </DbProviderFactories>
  </system.data>
</configuration>
```

# 3、代码部分

## 3.1、模型类

创建一个【`Person`】类：

```csharp
//NameSpace: SqLiteCoFirst
//FileName: Person
//Create By: raink
//Create Time: 2019/11/5 11:13:41

namespace SqLiteCoFirst
{
    class Person
    {
        public int PersonId { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }
}
```

## 3.2、 上下文类

基于 `DbContext` 的上下文类：

```csharp
//NameSpace: SqLiteCoFirst
//FileName: PersonContext
//Create By: raink
//Create Time: 2019/11/5 11:14:30

using System.Data.Entity;
using System.Data.Common;
using SQLite.CodeFirst;

namespace SqLiteCoFirst
{
    class PersonContext : DbContext
    {
        //用 System.Data.SQLite::SQLiteConnection 创建 DbConnection 进行构造初始化
        public PersonContext(DbConnection dbConnection, bool contextOwnsConnection = true)
               : base(dbConnection, true)
        {
        }

        //使用SQLite.CodeFirst::SqliteCreateDatabaseIfNotExists进行创建数据库
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            var sqliteConnectionInitializer = new SqliteCreateDatabaseIfNotExists<PersonContext>(modelBuilder);
            Database.SetInitializer(sqliteConnectionInitializer);
        }
 
        public DbSet<Person> people { get; set; }
    }
}
```

这里做简单说明：

因为想在让数据库在程序运行时再确定存储位置和名称，因此不采用如下的构造函数。

```csharp
//---------- 非新增代码
public PersonContext() : base("name = connectstring")
{
}
```

再者，考虑借助【`System.Data.SQLite`】命名空间下的【`SQLiteConnection`】类（基类是 `DbConnection`）可以创建 `DbConnection` 的实例化对象，因此【`PersonContext` 】的构造采用带参数的构造，参数采用基类的：

```csharp
//---------- 非新增代码
// 摘要:
// Constructs a new context instance using the existing connection to connect to
// a database. The connection will not be disposed when the context is disposed
// if contextOwnsConnection is false.
//
// 参数:
// existingConnection:
// An existing connection to use for the new context.
//
// contextOwnsConnection:
// If set to true the connection is disposed when the context is disposed, otherwise
// the caller must dispose the connection.
public DbContext(DbConnection existingConnection, bool contextOwnsConnection);
```

同时重载一下基类 (`DbContext`) 的:

```csharp
//---------- 非新增代码
// 摘要:
// This method is called when the model for a derived context has been initialized,
// but before the model has been locked down and used to initialize the context.
// The default implementation of this method does nothing, but it can be overridden
// in a derived class such that the model can be further configured before it is
// locked down.
protected virtual void OnModelCreating(DbModelBuilder modelBuilder);
```

使用【`SQLite.CodeFirst`】命名空间下的【`SqliteCreateDatabaseIfNotExists`】进行数据库创建即可。

## 3.3、程序中数据库的指定、连接、写入

CodeFirst 模式下对于数据库要写的代码就写完了，接下来就是如何操作了。

在控制台的主程序中，添加下面代码（详情见注释）

```csharp
using System.Collections.Generic;
using System.Data.SQLite;
using System.Data.Common;

namespace SqLiteCoFirst
{
    class Program
    {
        static void Main(string[] args)
        {
            //创建一些假冒数据
            List<Person> peoples = new List<Person>()
            {
                new Person() { Name = " 小 A", Age = 10},
                new Person() { Name = " 小 B", Age = 11, },
                new Person() { Name = " 小 C", Age = 12, },
                new Person() { Name = " 小 D", Age = 13, },
                new Person() { Name = " 小 E", Age = 14, },
            };
            //这里指定两个数据库
            DbConnection dbConnection_1 = new SQLiteConnection("data source=D:\\SqLiteCoFirst1.db");
            DbConnection dbConnection_2 = new SQLiteConnection("data source=D:\\SqLiteCoFirst2.db");

            //对两个数据库分别操作
            var ctx1 = new PersonContext(dbConnection_1);
            var ctx2 = new PersonContext(dbConnection_2);
            ctx1.people.AddRange(peoples);
            ctx2.people.AddRange(peoples);
            ctx1.SaveChanges();
            ctx2.SaveChanges();
            
        }
    }
}
```

然后直接运行得到：
![[「数据库」EF6+CodeFirst的(System.Data.)SQLite数据库操作入门.jpeg]]

完成。
