---
aliases:
date:
update:
author: ZZZ Projects
language: C#
sourceurl: https://www.entityframeworktutorial.net/code-first/database-initialization-in-code-first.aspx
tags:
  - CSharp
  - 資料庫
  - EF6
  - EF_Code_First
---

# Link

[Database Initialization in Entity Framework 6](https://www.entityframeworktutorial.net/code-first/database-initialization-in-code-first.aspx)
[Database Initialization Strategies in EF 6 Code-First](https://www.entityframeworktutorial.net/code-first/database-initialization-strategy-in-code-first.aspx)

---

# Database Initialization in Entity Framework 6 資料庫初始化

We have seen that Code-First creates a database automatically in the [Simple Code First Example](https://www.entityframeworktutorial.net/code-first/simple-code-first-example.aspx) section. Here, we will learn how EF decides the database name and server while initializing a database in code-first approach.
我們已經在 [「簡單 Code First 範例」](https://www.entityframeworktutorial.net/code-first/simple-code-first-example.aspx) 部分中看到，Code-First 會自動建立資料庫。在這裡，我們將學習 EF 如何在 Code-First 方法中初始化資料庫時確定資料庫名稱和伺服器。

The following figure shows a database initialization workflow, based on the parameter passed in the base constructor of the `context` class, which is derived from `DbContext`:
下圖展示了資料庫初始化的工作流程，該流程基於在 `context` 類別的基底構造函數中傳遞的參數，該類別派生自 `DbContext`：

![[database-init-fg1.png]]

As per the above figure, the base constructor of the context class can have the following parameters.
根據上圖，上下文類別的基本建構函數可以具有以下參數。

1. No Parameter  無參數
2. Database Name  資料庫名稱
3. Connection String Name  連接字串名稱

## No Parameter  無參數

If you do not specify the parameter in the base constructor of the `context` class then it creates a database in your local SQLEXPRESS server with a name that matches your `{Namespace}.{Context class name}`. For example, EF will create a database named `SchoolDataLayer.Context` for the following context class:
如果您未在 `context` 類別的基底建構函式中指定該參數，則它會在您的本機 SQLEXPRESS 伺服器中建立一個資料庫，其名稱與您的 `{Namespace}.{Context 類別名稱}` 相符。例如，EF 將為以下 context 類別建立一個名為 `SchoolDataLayer.Context` 的資料庫：

```csharp
namespace SchoolDataLayer
{
    public class Context: DbContext
    {
        public Context(): base()
        {

        }
    }
}
```

## Database Name  資料庫名稱

You can also specify the database name as a parameter in a base constructor of the `context` class. If you specify a database name parameter, then Code First creates a database with the name you specified in the base constructor in the local SQLEXPRESS database server. For example, Code First will create a database named `MySchoolDB` for the following `context` class.
您也可以在 `context` 類別的基底建構函式中將資料庫名稱指定為參數。如果指定了資料庫名稱參數，則 Code First 會在本機 SQLEXPRESS 資料庫伺服器中建立具有您在基底建構函式中指定的名稱的資料庫。例如，Code First 將為以下 `context` 類別建立名為 `MySchoolDB` 的資料庫。

```csharp
namespace SchoolDataLayer
{
    public class Context: DbContext
    {
        public Context(): base("MySchoolDB")
        {

        }
    }
}
```

## Connection String Name  連接字串名稱

You can also define the connection string in app.config or web.config and specify the connection string name starting with "`name=`" in the base constructor of the context class. Consider the following example where we pass the `name=SchoolDBConnectionString` parameter in the base constructor.
您也可以在 app.config 或 web.config 中定義連接字串，並在 context 類別的基底類別建構子中指定以「`name=`」開頭的連接字串名稱。請考慮以下範例，我們在基底類別建構子中傳遞了 `name=SchoolDBConnectionString` 參數。

```csharp
namespace SchoolDataLayer
{
    public class SchoolDBContext: DbContext
    {
        public SchoolDBContext() : base("name=SchoolDBConnectionString")
        {
        }
    }
}
```

App.config:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <connectionStrings>
    <add name="SchoolDBConnectionString"
    connectionString="Data Source=.;Initial Catalog=SchoolDB-ByConnectionString;Integrated Security=true"
    providerName="System.Data.SqlClient"/>
    </connectionStrings>
</configuration>
```

In the above context class, we specify a connection string name as a parameter. Please note that the connection string name should start with "name=", otherwise it will consider it as a database name. The database name in the connection string in app.config is _SchoolDB-ByConnectionString_. EF will create a new _SchoolDB-ByConnectionString_ database or use the existing _SchoolDB-ByConnectionString_ database in the local SQL Server. Make sure that you include `providerName = "System.Data.SqlClient"` for the SQL Server database in the connection string.
在上面的上下文類別中，我們指定了一個連接字串名稱作為參數。請注意，連接字串名稱應以“name=”開頭，否則會將其視為資料庫名稱。 app.config 中連接字串的資料庫名稱為 _SchoolDB-ByConnectionString_ 。 EF 會建立一個新的 _SchoolDB-ByConnectionString_ 資料庫，或使用本機 SQL Server 中現有的 _SchoolDB-ByConnectionString_ 資料庫。請確保在連接字串中包含 SQL Server 資料庫的 `providerName = "System.Data.SqlClient"` 。

---

# Database Initialization Strategies in EF 6 Code-First 資料庫初始化策略

You already created a database after running your Code-First application the first time, but what about the second time onwards? Will it create a new database every time you run the application? What about the production environment? How do you alter the database when you change your domain model? To handle these scenarios, you have to use one of the database initialization strategies.
第一次執行 Code-First 應用程式後，你已經建立了一個資料庫，但第二次之後呢？每次運行應用程式時都會建立一個新的資料庫嗎？在生產環境中怎麼辦？當你更改域模型時，該如何修改資料庫？為了處理這些情況，你必須使用其中一種資料庫初始化策略。

There are four different database initialization strategies:
有四種不同的資料庫初始化策略：

1. **CreateDatabaseIfNotExists:** This is the **default** initializer. As the name suggests, it will create the database if none exists as per the configuration. However, if you change the model class and then run the application with this initializer, then it will throw an exception.
    **CreateDatabaseIfNotExists：** 這是**預設的**初始化方法。顧名思義，如果配置中不存在資料庫，它將建立資料庫。但是，如果你更改了模型類，然後使用此初始化方法運行應用程序，則會拋出異常。
2. **DropCreateDatabaseIfModelChanges:** This initializer drops an existing database and creates a new database, if your model classes (entity classes) have been modified. So, you don't have to worry about maintaining your database schema, when your model classes change.
    **DropCreateDatabaseIfModelChanges：** 如果您的模型類別（實體類別）已被修改，此初始化程序將刪除現有資料庫並建立新資料庫。因此，當您的模型類別發生變更時，您不必擔心維護資料庫架構。
3. **DropCreateDatabaseAlways:** As the name suggests, this initializer drops an existing database every time you run the application, irrespective of whether your model classes have changed or not. This will be useful when you want a fresh database every time you run the application, for example when you are developing the application.
    **DropCreateDatabaseAlways：** 顧名思義，這個初始化方法每次執行應用程式時都會刪除一個現有的資料庫，無論模型類別是否改變。當你希望每次執行應用程式時都建立一個新的資料庫時，例如在開發應用程式時，這個方法就非常有用。
4. **Custom DB Initializer:** You can also create your own custom initializer, if the above does not satisfy your requirements or you want to do another process that initializes the database using the above initializer.
    **自訂資料庫初始化程序：** 如果上述內容無法滿足您的要求，或者您想使用上述初始化程序執行另一個初始化資料庫的過程，您也可以建立自己的自訂初始化程序。

To use one of the above DB initialization strategies, you have to set the DB Initializer using the `Database` class in a context class, as shown below:
若要使用上述 DB 初始化策略之一，您必須使用上下文類別中的 `Database` 類別來設定 DB 初始化程序，如下所示：

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base("SchoolDBConnectionString")
    {
        Database.SetInitializer<SchoolDBContext>(new CreateDatabaseIfNotExists<SchoolDBContext>());

        //Database.SetInitializer<SchoolDBContext>(new DropCreateDatabaseIfModelChanges<SchoolDBContext>());
        //Database.SetInitializer<SchoolDBContext>(new DropCreateDatabaseAlways<SchoolDBContext>());
        //Database.SetInitializer<SchoolDBContext>(new SchoolDBInitializer());
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
}
```

You can also create your custom DB initializer, by inheriting one of the initializers, as shown below:
您也可以透過繼承其中一個初始化程序來建立自訂 DB 初始化程序，如下所示：

```csharp
public class SchoolDBInitializer : CreateDatabaseIfNotExists<SchoolDBContext>
{
    protected override void Seed(SchoolDBContext context)
    {
        base.Seed(context);
    }
}
```

In the above example, the `SchoolDBInitializer` is a custom initializer class that is derived from `CreateDatabaseIfNotExists`. This separates the database initialization code from a context class.
在上面的範例中， `SchoolDBInitializer` 是一個從 `CreateDatabaseIfNotExists` 派生的自訂初始化器類別。 這將資料庫初始化程式碼與上下文類別分開。

## Set the DB Initializer in the Configuration File 在設定檔中設定 DB 初始化程序

You can also set the DB initializer in the configuration file. For example, to set the default initializer in `app.config`:
您也可以在設定檔中設定資料庫初始化器。例如，要在 `app.config` 中設定預設初始化器：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <appSettings>
    <add key="DatabaseInitializerForType SchoolDataLayer.SchoolDBContext, SchoolDataLayer"
        value="System.Data.Entity.DropCreateDatabaseAlways`1[[SchoolDataLayer.SchoolDBContext, SchoolDataLayer]], EntityFramework" />
    </appSettings>
</configuration>
```

You can set the custom DB initializer, as follows:
您可以設定自訂 DB 初始化程序，如下所示：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <appSettings>
    <add key="DatabaseInitializerForType SchoolDataLayer.SchoolDBContext, SchoolDataLayer"
            value="SchoolDataLayer.SchoolDBInitializer, SchoolDataLayer" />
    </appSettings>
</configuration>
```

# Turn off the DB Initializer 關閉資料庫初始化程序

You can turn off the database initializer for your application. Suppose that you don't want to lose existing data in the production environment, then you can turn off the initializer, as shown below:
您可以為您的應用程式關閉資料庫初始化程式。假設您不想遺失生產環境中的現有數據，那麼您可以關閉初始化程序，如下所示：

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext() : base("SchoolDBConnectionString")
    {
        //Disable initializer
        Database.SetInitializer<SchoolDBContext>(null);
    }
    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
}
```

You can also turn off the initializer in the configuration file, for example:
您也可以在設定檔中關閉初始化程序，例如：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <appSettings>
    <add key="DatabaseInitializerForType SchoolDataLayer.SchoolDBContext, SchoolDataLayer"
            value="Disabled" />
    </appSettings>
</configuration>
```
