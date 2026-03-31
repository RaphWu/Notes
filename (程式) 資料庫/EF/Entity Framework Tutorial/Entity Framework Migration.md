---
aliases:
date: 2025-06-18
update:
author: ZZZ Projects
language: C#
sourceurl: https://entityframework.net/migration
tags:
  - 資料庫
  - EF6
  - EF_Code_First
---

# Entity Framework Migration 實體框架遷移

Discover How to Update your Database Schema
了解如何更新資料庫架構

The Migrations feature enables you to change the data model and deploy your changes to production by updating the database schema without having to drop and re-create the database. It is the recommended way to evolve your application's database schema if you are using the Code First workflow.
遷移功能使你能夠透過更新資料庫架構來更改資料模型並將更改部署到生產環境，而無需刪除並重新建立資料庫。如果你使用 Code First 工作流程，建議使用此方法來改進應用程式的資料庫架構。

Migrations provide a set of tools that allow:
遷移提供了一組工具，允許：

- Create an initial database that works with your EF model
  建立適用於 EF 模型的初始資料庫
- Generating migrations to keep track of changes you make to your EF model
  生成遷移以追蹤您對 EF 模型所做的更改
- Keep your database up to date with those changes
  讓您的資料庫保持最新狀態以適應這些變化

## Initial Model & Database Using Code First Approach  使用程式碼優先方法的初始模型和資料庫

Create a new application and install the EntityFramework NuGet package.
建立一個新的應用程式並安裝 EntityFramework NuGet 套件。

![Migration-1](https://raw.githubusercontent.com/zzzprojects/EntityFramework-FAQ/master/docs2/images/migration.png)

Once the package is installed, add the following classes.
安裝套件後，新增以下類別。

```csharp
public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
}

public class BookContext : DbContext
{
    public BookContext() : base("BookContext")
    {
    }

    public DbSet<Book> Books { get; set; }
}
```

It defines a single `Book` class that makes up our **domain model** and a `BookContext` class that is our EF Code First context.
它定義了一個組成我們的**領域模型**的 `Book` 類別和一個作為我們的 EF Code First 上下文的 `BookContext` 類別。

You can also specify the connection string in App.config file.
您也可以在 App.config 檔案中指定連接字串。

```xml
<connectionStrings>
  <add name="BookContext" connectionString="Data Source=(localdb)\ProjectsV13;Initial Catalog=BookContext;" providerName="System.Data.SqlClient"/>
</connectionStrings>
```

Now that we have a model let's use it to perform data access.
現在我們有了模型，讓我們用它來執行資料存取。

```csharp
using System;
using System.Data.Entity;

namespace EFDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var db = new BookContext())
            {
                db.Books.Add(new Book { Title = "Introduction to Programming" });
                db.SaveChanges();

                foreach (var book in db.Books)
                {
                    Console.WriteLine(book.Title);
                }
            }
        }
    }

    public class Book
    {
        public int BookId { get; set; }
        public string Title { get; set; }
    }
    public class BookContext : DbContext
    {
        public DbSet<Book> Books { get; set; }
    }
}
```

Let's run your application, and you will see that the database is created automatically.
讓我們運行您的應用程序，您將看到資料庫已自動建立。

![Migration-2](https://raw.githubusercontent.com/zzzprojects/EntityFramework-FAQ/master/docs2/images/migration1.png)

## Update Database  更新資料庫

Now let's change your model by adding another property to the `Book` class.
現在讓我們透過在 `Book` 類別中新增另一個屬性來更改您的模型。

```csharp
public DateTime Date { get; set;}
```

Now if you rerun your application, you would get an `InvalidOperationException`.
現在，如果您重新運行應用程序，您將收到 `InvalidOperationException` 。

> The model backing the 'BlogContext' context has changed since the database was created. Consider using Code First Migrations to update the database ([https://go.microsoft.com/fwlink/?LinkId=238269](https://go.microsoft.com/fwlink/?LinkId=238269)).
> 自資料庫建立以來，支援「BlogContext」上下文的模型已發生變更。請考慮使用 Code First 遷移來更新資料庫 ( [https://go.microsoft.com/fwlink/?LinkId=238269](https://go.microsoft.com/fwlink/?LinkId=238269) )。

It's time to start using Code First Migrations. There are two kinds of Migrations;
現在是時候開始使用 Code First 遷移了。遷移有兩種類型：

- Automated Migration  自動遷移
- Code-based Migration  基於程式碼的遷移

### Automated Migration  自動遷移

To use Automated Migration, run the following command in the Package Manager Console.
若要使用自動遷移，請在套件管理器控制台中執行下列命令。

```powershell
enable-migrations EnableAutomaticMigration:$true
```

Once the command runs successfully, it creates an internal sealed Configuration class derived from `DbMigrationConfiguration` in the Migration folder in your project.
一旦命令成功運行，它將建立一個從專案的 Migration 資料夾中的 `DbMigrationConfiguration` 派生的內部密封配置類別。

```csharp
namespace EFDemo.Migrations
{
    using System;
    using System.Data.Entity;
    using System.Data.Entity.Migrations;
    using System.Linq;

    internal sealed class Configuration : DbMigrationsConfiguration<EFDemo.BookContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = true;
            ContextKey = "EFDemo.BookContext";
        }

        protected override void Seed(EFDemo.BookContext context)
        {
            //  This method will be called after migrating to the latest version.

            //  You can use the DbSet<T>.AddOrUpdate() helper extension method 
            //  to avoid creating duplicate seed data.
        }
    }
}
```

The next step is to set the database initializer in the context class.
下一步是在上下文類別中設定資料庫初始化程序。

```csharp
public class BookContext : DbContext
{
    public BookContext() : base("BookContext")
    {
        Database.SetInitializer(new MigrateDatabaseToLatestVersion<BookContext, EFDemo.Migrations.Configuration>());
    }

    public DbSet<Book> Books { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
    }
}
```

Now EF will automatically take care of the migration when you change the domain classes.
現在，當您變更網域類別時，EF 將自動處理遷移。

![Migration-3](https://raw.githubusercontent.com/zzzprojects/EntityFramework-FAQ/master/docs2/images/migration2.png)

### Code-based Migration  基於程式碼的遷移

In Code First Migrations, you need to execute the following commands in the Package Manager Console.
在 Code First Migrations 中，您需要在套件管理器控制台中執行下列命令。

- **Enable-Migrations**: Enables the migration in your project.
  **Enable-Migrations ：啟用專案中的遷移** 。
- **Add-Migration**: It creates a new migration based on changes you have made to your model since the last migration was created.
  **新增遷移** ：它會根據自上次遷移建立以來對模型所做的變更建立一個新的遷移。
- **Update-Database** It applies any pending migrations to the database.
  **更新資料庫**它將任何待處理的遷移應用到資料庫。

Let's add another property to your domain class.
讓我們為您的網域類別新增另一個屬性。

```csharp
public string Publisher { get; set; }
```

Run the following command in Package Manager Console.
在程式包管理器控制台中執行以下命令。

```powershell
Add-Migration AddPublisher
```

In the Migrations folder, we now have a new `AddPublisher` migration. The migration filename is pre-fixed with a timestamp to help with ordering.
在 Migrations 資料夾中，我們現在有了一個新的 `AddPublisher` 遷移檔案。遷移檔案名稱以時間戳為前綴，方便排序。

```csharp
namespace EFDemo.Migrations
{
    using System;
    using System.Data.Entity.Migrations;

    public partial class AddPublisher : DbMigration
    {
        public override void Up()
        {
            AddColumn("dbo.Books", "Publisher", c => c.String());
        }
        
        public override void Down()
        {
            DropColumn("dbo.Books", "Publisher");
        }
    }
}
```

We could now edit or add to this migration, but everything looks pretty good. Let's use Update-Database to apply this migration to the database.
現在我們可以編輯或添加此遷移，但一切看起來都很好。讓我們使用 Update-Database 將此遷移應用到資料庫。

Run the `Update-Database` command in Package Manager Console.
在程式包管理器控制台中執行 `Update-Database` 指令。

![Migration-4](https://raw.githubusercontent.com/zzzprojects/EntityFramework-FAQ/master/docs2/images/migration3.png)

You can see that the database is now up to date by adding the `Publisher` column in the `Books` table.
透過在 `Books` 表中新增 `Publisher` 列，您可以看到資料庫現在是最新的。
