---
aliases:
date: 2020-02-14
update: 2020-10-07
author: 余小章
language: C#
sourceurl: https://dotblogs.com.tw/yc421206/2020/02/14/sqlite_code_first_migration_2
tags:
  - CSharp
  - 演算法
  - SQLite
---

# EF6 SQLite Code First 和 Migration (2)

續上篇，[[EF6][SQLite] SQLite Code First 和 Migration](https://dotblogs.com.tw/yc421206/2020/02/10/sqlite_code_first_migration)，那篇使用的 System.Data.SQLite.EF6.Migrations 套件，不過在 .NET Framework 4 的環境下一直有問題，不得已的情況下要改用 SQLite.CodeFirst

# 開發環境

- VS 2019
- .NET Framework 4.0
- EF 6.3
- System.Data.SQLite 1.0.112.0

# 實作

## 安裝套件

```plantext
`Install-Package System.Data.SQLite`
`Install-Package SQLite.CodeFirst`
```

packages.config 相依套件如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
	<package id="EntityFramework" version="6.3.0" targetFramework="net40" />
	<package id="SQLite.CodeFirst" version="1.5.3.29" targetFramework="net40" />
	<package id="System.Data.SQLite" version="1.0.112.0" targetFramework="net40" />
	<package id="System.Data.SQLite.Core" version="1.0.112.0" targetFramework="net40" />
	<package id="System.Data.SQLite.EF6" version="1.0.112.0" targetFramework="net40" />
	<package id="System.Data.SQLite.Linq" version="1.0.112.0" targetFramework="net40" />
</packages>
```

## 組態

### 增加 Provider 

```xml
<provider invariantName="System.Data.SQLite"
          type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
<entityFramework>
    <providers>
        <provider invariantName="System.Data.SqlClient"
                  type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
        <provider invariantName="System.Data.SQLite.EF6"
                  type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
        <provider invariantName="System.Data.SQLite"
                  type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
    </providers>
</entityFramework>
```

### 增加連線字串

```xml
<connectionStrings>
    <add name="DefaultConnection" connectionString="data source=lab.db" providerName="System.Data.SQLite" />
</connectionStrings>
```

## POCO

```cs
[Table("Employee")]
public class Employee
{
    [Required]
    [Key]
    public Guid Id { get; set; }
 
    [StringLength(100)]
    public string Name { get; set; }
 
    //public int Age { get; set; }
}
```

## LabDbContext

這裡在 `OnModelCreating` 調用 `new SqliteDropCreateDatabaseWhenModelChanges<LabDbContext>(modelBuilder) `

```csharp
public class LabDbContext : DbContext
{
    private static readonly bool[] s_migrated = {false};
 
    public DbSet<Employee> Employees { get; set; }
 
    public LabDbContext() : base("DefaultConnection")
    {
    }
 
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        if (!s_migrated[0])
        {
            lock (s_migrated)
            {
                if (!s_migrated[0])
                {
                    var initializer = new SqliteDropCreateDatabaseWhenModelChanges<LabDbContext>(modelBuilder);
                    Database.SetInitializer(initializer);
 
                    s_migrated[0] = true;
                }
            }
        }
    }
}
```

### 還原測試資料庫

為了觀察 Auto Magration 運作是否正常，只砍測試資料，就不砍 DB 檔案了

```cs
[TestCleanup]
public void After()
{
    using (var db = new LabDbContext())
    {
        //db.Database.ExecuteSqlCommand("delete from Identity;");
 
        db.Database.ExecuteSqlCommand("delete from Employee;");
    }
}
 
[TestInitialize]
public void Before()
{
    using (var db = new LabDbContext())
    {
        //db.Database.ExecuteSqlCommand("delete from Identity;");
 
        db.Database.ExecuteSqlCommand("delete from Employee;");
    }
}
```

新增案例

```cs
[TestMethod]
public void Insert()
{
    var toDb = new Employee
    {
        Id   = Guid.NewGuid(),
        Name = "yao",
        //Age = 18
    };
    using (var db = new LabDbContext())
    {
        db.Employees.Add(toDb);
        var count = db.SaveChanges();
        Assert.AreEqual(1, count);
    }
}
```

資料庫結構如下，使用 `SQLite.CodeFirst` 的 `Migration` 資料表叫 History_XXX

[![](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617255_55466.png)](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617255_55466.png)

[![](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617292_13711.png)](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617292_13711.png)

### Auto Migration

接下來，把 `Employee` 的 `Age` 註解打開，再跑一次測試，打開 `lab.db` 可以看到 `Age` 欄位已經被建立了

[![](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617551_04411.png)](https://dotblogsfile.blob.core.windows.net/user/%E4%BD%99%E5%B0%8F%E7%AB%A0/ce511bc0-4616-48b7-81c3-b29ebaed9a24/1581617551_04411.png)

省去了呼叫 `Add-Migration`、`Update-Database` 命令，使用起來還算方便的。

因為用的是 `SqliteDropCreateDatabaseWhenModelChanges`，所以當 POCO 異動後，資料就會不見了，如果資料結構不常異動的話這個方案似乎可以考慮，想想，現實生活上好像沒有這回事 XDD

# 範例位置

[https://github.com/yaochangyu/sample.dotblog/tree/master/ORM/EF6/Lab.EF6.SqliteCodeFirstNet4](https://github.com/yaochangyu/sample.dotblog/tree/master/ORM/EF6/Lab.EF6.SqliteCodeFirstNet4) 
