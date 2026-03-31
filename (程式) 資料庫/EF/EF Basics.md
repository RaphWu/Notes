---
aliases:
date:
update:
author: ZZZ Projects
language: C#
sourceurl: https://www.entityframeworktutorial.net/entityframework6/what-is-entityframework.aspx
tags:
  - CSharp
  - 資料庫
  - EntityFramework
---

- [What is Entity Framework?](https://www.entityframeworktutorial.net/entityframework6/what-is-entityframework.aspx)
- [EF Architecture](https://www.entityframeworktutorial.net/entityframework6/entityframework-architecture.aspx)
- [How EF Works?](https://www.entityframeworktutorial.net/entityframework6/how-entity-framework-works.aspx)
- [Entities](https://www.entityframeworktutorial.net/entityframework6/entity-in-entityframework.aspx)
- [Context Class](https://www.entityframeworktutorial.net/entityframework6/context-class-in-entity-framework.aspx)
- [Development Approaches](https://www.entityframeworktutorial.net/entityframework6/choosing-development-approach-with-entity-framework.aspx)
- [Persistence Scenarios](https://www.entityframeworktutorial.net/entityframework6/persistence-in-entity-framework.aspx)

---

# What is Entity Framework? 什麼是實體框架？

Prior to .NET 3.5, we (developers) often used to write ADO.NET code or Enterprise Data Access Block to save or retrieve application data from the underlying database. We used to open a connection to the database, create a DataSet to fetch or submit the data to the database, convert data from the DataSet to .NET objects or vice-versa to apply business rules. This was a cumbersome and error-prone process. Microsoft has provided a framework called "Entity Framework" to automate all these database-related activities for your application.
在 .NET 3.5 之前，我們（開發人員）經常編寫 ADO.NET 程式碼或企業資料存取區塊 (EDI) 來從底層資料庫保存或檢索應用程式資料。我們過去常常需要開啟資料庫連接，建立 DataSet 來取得或提交資料到資料庫，然後將 DataSet 中的資料轉換為 .NET 對象，反之亦然，以應用業務規則。這是一個繁瑣且容易出錯的過程。微軟提供了一個名為「Entity Framework」的框架，可以自動化應用程式所有與資料庫相關的活動。

Entity Framework is an open-source [ORM framework](https://en.wikipedia.org/wiki/Object-relational_mapping "Object-relational Mapping") for .NET applications supported by Microsoft. It enables developers to work with data using objects of domain-specific classes without focusing on the underlying database tables and columns where this data is stored. With the Entity Framework, developers can work at a higher level of abstraction when they deal with data, and can create and maintain data-oriented applications with less code compared with traditional applications.
Entity Framework 是 Microsoft 支援的 .NET 應用程式的開源 [ORM 框架](https://en.wikipedia.org/wiki/Object-relational_mapping "Object-relational Mapping") 。 它使開發人員能夠使用特定於網域的類別的物件來處理數據，而無需關注儲存這些資料的底層資料庫表和列。借助實體框架，開發人員可以在更高的抽象層級處理數據，並且與傳統應用程式相比，可以使用更少的程式碼來創建和維護面向數據的應用程式。

Official Definition: â€œEntity Framework is an object-relational mapper (O/RM) that enables .NET developers to work with a database using .NET objects. It eliminates the need for most of the data-access code that developers usually need to write.â€
官方定義：“Entity Framework 是一個物件關係映射器 (O/RM)，它使 .NET 開發人員能夠使用 .NET 物件操作資料庫。它消除了開發人員通常需要編寫的大部分資料存取程式碼。”

The following figure illustrates where the Entity Framework fits into your application.
下圖說明了實體框架在您的應用程式中的位置。

![[ef-in-app-architecture.png]]

As per the above figure, Entity Framework fits between the business entities (domain classes) and the database. It saves data stored in the properties of business entities and also retrieves data from the database and converts it to business entity objects automatically.
如上圖所示，Entity Framework 位於業務實體（領域類別）和資料庫之間。它保存儲存在業務實體屬性中的數據，並從資料庫中檢索資料並自動將其轉換為業務實體物件。

## Entity Framework Features 實體框架功能

- **Cross-platform:** EF Core is a cross-platform framework which can run on Windows, Linux and macOS.
  **跨平台：** EF Core 是一個跨平台框架，可以在 Windows、Linux 和 macOS 上運作。
- **Modeling:** EF (Entity Framework) creates an EDM (Entity Data Model) based on POCO (Plain Old CLR Object) entities with get/set properties of different data types. It uses this model when querying or saving entity data to the underlying database.
  **建模：** EF（實體框架）基於 POCO（普通舊式 CLR 物件）實體建立 EDM（實體資料模型），並具有不同資料類型的 get/set 屬性。在查詢實體資料或將其儲存到底層資料庫時，EF 會使用此模型。
- **Querying:** EF allows us to use LINQ queries (C#/VB.NET) to retrieve data from the underlying database. The database provider will translate this LINQ query to the database-specific query language (e.g. SQL for a relational database). EF also allows us to execute raw SQL queries directly to the database.
  **查詢：** EF 允許我們使用 LINQ 查詢（C#/VB.NET）從底層資料庫檢索資料。資料庫提供者會將此 LINQ 查詢轉換為特定於資料庫的查詢語言（例如，關聯式資料庫的 SQL）。 EF 也允許我們直接對資料庫執行原始 SQL 查詢。
- **Change Tracking:** EF keeps track of changes that occur to instances of your entities (Property values) which need to be submitted to the database.
  **變更追蹤：** EF 追蹤需要提交到資料庫的實體實例（屬性值）發生的變更。
- **Saving:** EF executes INSERT, UPDATE, and DELETE commands to the database based on the changes that occurred to your entities when you call the `SaveChanges()` method. EF also provides the asynchronous `SaveChangesAsync()` method.
  **儲存：** 當您呼叫 `SaveChanges()` 方法時，EF 會根據實體發生的變更向資料庫執行 INSERT、UPDATE 和 DELETE 指令。 EF 也提供非同步 `SaveChangesAsync()` 方法。
- **Concurrency:** EF uses Optimistic Concurrency by default to protect overwriting changes made by another user since data was fetched from the database.
  **並發：** EF 預設使用樂觀並發來保護從資料庫取得資料後其他使用者所做的變更不會被覆寫。
- **Transactions:** EF performs automatic transaction management while querying or saving data. It also provides options to customize transaction management.
  **事務：** EF 在查詢或儲存資料時執行自動事務管理。它還提供自訂事務管理的選項。
- **Caching:** EF includes a first level of caching out of the box. So, repeated querying will return data from the cache instead of hitting the database.
  **快取：** EF 內建了一級快取功能。因此，重複查詢將從快取中返回數據，而無需存取資料庫。
- **Built-in Conventions:** EF follows conventions over the configuration programming pattern, and includes a set of default rules which automatically configure the EF model.
  **內建約定：** EF 遵循配置程式模式的約定，並包含一組自動配置 EF 模型的預設規則。
- **Configurations:** EF allows us to configure the EF model by using data annotation attributes or Fluent API to override default conventions.
  **配置：** EF 允許我們使用資料註解屬性或 Fluent API 來配置 EF 模型，以覆寫預設約定。
- **Migrations:** EF provides a set of migration commands that can be executed on the NuGet Package Manager Console or the Command Line Interface to create or manage underlying database Schema.
  **遷移：** EF 提供了一組遷移命令，可以在 NuGet 套件管理器控制台或命令列介面上執行，以建立或管理底層資料庫模式。

## Entity Framework Latest Versions 實體框架最新版本

Microsoft introduced Entity Framework in 2008 with .NET Framework 3.5. Since then, it released many versions of Entity Framework. Currently, there are two latest versions of Entity Framework: EF 6 and EF Core. The following table lists important differences between EF 6 and EF Core.
微軟於 2008 年隨 .NET Framework 3.5 推出了實體架構 (Entity Framework)。此後，微軟發布了多個版本的實體框架。目前，實體框架有兩個最新版本：EF 6 和 EF Core。下表列出了 EF 6 和 EF Core 之間的重要差異。

[![ef6 vs ef core](https://www.entityframeworktutorial.net/images/basics/ef6-vs-efcore.png)](https://www.entityframeworktutorial.net/images/basics/ef6-vs-efcore.png)

### EF 6 Version History

[What's new in EF6](https://learn.microsoft.com/en-us/ef/ef6/what-is-new)
[Past Releases of Entity Framework](https://learn.microsoft.com/en-us/ef/ef6/what-is-new/past-releases)

| EF Version   EF 版本                 | Release Year   發行年份 | .NET Framework   .NET 框架                                        |
| ---------------------------------- | ------------------- | --------------------------------------------------------------- |
| EF 6                               | 2013                | .NET 4.0 & .NET 4.5, VS 2012  <br>.NET 4.0 和 .NET 4.5，與 2012 相比 |
| EF 5                               | 2012                | .NET 4.0, VS 2012  <br>.NET 4.0，VS 2012                         |
| EF 4.3                             | 2011                | .NET 4.0, VS 2012  <br>.NET 4.0，VS 2012                         |
| EF 4.0                             | 2010                | .NET 4.0, VS 2010  <br>.NET 4.0，VS 2010                         |
| EF 1.0 (or 3.5)  <br>EF 1.0（或 3.5） | 2008                | .NET 3.5 SP1, VS 2008  <br>.NET 3.5 SP1，VS 2008                 |

### EF Core Version History

[EF Core releases and planning](https://learn.microsoft.com/en-us/ef/core/what-is-new/)

|EF Core Version   EF Core 版本|Release Date   發布日期|Target Framework   目標框架|
|---|---|---|
|EF Core 9.0|Nov 2024   2024 年 11 月|.NET 8|
|EF Core 8.0|Nov 2023   2023 年 11 月|.NET 8|
|EF Core 7.0|Nov 2022   2022 年 11 月|.NET 6|
|EF Core 6.0|Nov 2021   2021 年 11 月|.NET 6|
|EF Core 5.0|Nov 2020   2020 年 11 月|.NET Standard 2.1  .NET 標準 2.1|
|EF Core 3.0|Sept 2019   2019 年 9 月|.NET Standard 2.1  .NET 標準 2.1|
|EF Core 2.0|August 2017   2017 年 8 月|.NET Standard 2.0  .NET 標準 2.0|
|EF Core 1.0|June 2016   2016 年 6 月|.NET Standard 1.6  .NET 標準 1.6|

---

# Entity Framework Architecture 實體框架架構

The following figure shows the overall architecture of the Entity Framework.
下圖展示了 Entity Framework 的整體架構。

![[ef-architecture.png]]

Let's look at the components of the architecture individually.
讓我們分別看一下該架構的各個元件。

**EDM (Entity Data Model):** EDM consists of three main parts - ==Conceptual model==, ==Mapping== and ==Storage model==.
**EDM（實體資料模型）：** EDM 由三個主要部分組成 - ==概念模型==，==映射==和==儲存模型==。

**Conceptual Model:** The conceptual model contains the model classes and their relationships. This will be independent from your database table design.
**概念模型：** 概念模型包含模型類別及其關係。這與資料庫表設計無關。

**Storage Model:** The storage model is the database design model which includes tables, views, stored procedures, and their relationships and keys.
**儲存模型：** 儲存模型是資料庫設計模型，包括表格、視圖、預存程序及其關係和鍵。

**Mapping:** Mapping consists of information about how the conceptual model is mapped to the storage model.
**映射：** 映射包含有關概念模型如何映射到儲存模型的資訊。

**LINQ to Entities:** LINQ-to-Entities (L2E) is a query language used to write queries against the object model. It returns entities, which are defined in the conceptual model. You can use your LINQ skills here.
**LINQ to Entities：** LINQ to Entities (L2E) 是一種用來編寫針對物件模型的查詢語言。它會傳回在概念模型中定義的實體。您可以在這裡運用您的 LINQ 技能。

**Entity SQL:** Entity SQL is another query language (for EF 6 only) just like LINQ to Entities. However, it is a little more difficult than L2E and the developer will have to learn it separately.
**Entity SQL：** Entity SQL 是另一種查詢語言（僅適用於 EF 6），類似於 LINQ to Entities。但是，它比 L2E 稍微難一些，開發人員必須單獨學習它。

**Object Service:** Object service is the main entry point for accessing data from the database and returning it. Object service is responsible for materialization, which is the process of converting data returned from an entity client data provider (next layer) to an entity object structure.
**對象服務：** 對象服務是從資料庫存取資料並傳回資料的主要入口點。物件服務負責實體化，即將實體客戶端資料提供者（下一層）傳回的資料轉換為實體物件結構的過程。

**Entity Client Data Provider:** The main responsibility of this layer is to convert LINQ-to-Entities or Entity SQL queries into a SQL query which is understood by the underlying database. It communicates with the ADO.Net data provider which in turn sends or retrieves data from the database.
**實體客戶端資料提供者：** 此層的主要職責是將 LINQ-to-Entities 或 Entity SQL 查詢轉換為底層資料庫可以理解的 SQL 查詢。它與 ADO.Net 資料提供者通信，後者又從資料庫發送或檢索資料。

**ADO.Net Data Provider:** This layer communicates with the database using standard ADO.Net.
**ADO.Net 資料提供者：** 此層使用標準 ADO.Net 與資料庫通訊。

---

# How Entity Framework Works? 實體框架如何運作？

Entity Framework API (EF6 & EF Core) includes the ability to map domain (entity) classes to the database schema, translate & execute LINQ queries to SQL, track changes that occurred on entities during their lifetime, and save changes to the database.
實體框架 API（EF6 和 EF Core）包括將網域（實體）類別對應到資料庫模式、將 LINQ 查詢轉換並執行為 SQL、追蹤實體在其生命週期內發生的變更以及將變更儲存到資料庫的能力。

![[ef-api.png]]

## Entity Data Model 實體資料模型

The very first task of EF API is to build an Entity Data Model (EDM). EDM is an in-memory representation of the entire metadata: conceptual model, storage model, and mapping between them.
EF API 的首要任務是建立實體資料模型 (EDM)。 EDM 是整個元資料的記憶體表示：概念模型、儲存模型以及它們之間的對應。

![[ef-edm.png]]

**Conceptual Model:** EF builds the conceptual model from your domain classes, context class, default conventions followed in your domain classes, and configurations.
**概念模型：** EF 從您的領域類別、上下文類別、域類別中遵循的預設約定和配置建立概念模型。

**Storage Model:** EF builds the storage model for the underlying database schema. In the code-first approach, this will be inferred from the conceptual model. In the database-first approach, this will be inferred from the targeted database.
**儲存模型：** EF 為底層資料庫模式建立儲存模型。在程式碼優先方法中，這將從概念模型推斷出來。在資料庫優先方法中，這將從目標資料庫推斷出來。

**Mappings:** EF includes mapping information on how the conceptual model maps to the database schema (storage model).
**映射：** EF 包含概念模型如何映射到資料庫模式（儲存模型）的映射資訊。

EF performs CRUD operations using this EDM. It uses EDM in building SQL queries from LINQ queries, building INSERT, UPDATE, and DELETE commands, and transforms database results into entity objects.
EF 使用此 EDM 執行 CRUD 操作。它使用 EDM 從 LINQ 查詢建立 SQL 查詢，建立 INSERT、UPDATE 和 DELETE 命令，並將資料庫結果轉換為實體物件。

## Querying  查詢

EF API translates LINQ-to-Entities queries to SQL queries for relational databases using EDM and also converts results back to entity objects.
EF API 使用 EDM 將 LINQ-to-Entities 查詢轉換為關聯式資料庫的 SQL 查詢，並將結果轉換回實體物件。

![[ef-querying.png]]

## Saving  儲存

EF API infers INSERT, UPDATE, and DELETE commands based on the state of entities when the `SaveChanges()` method is called. The ChangeTracker keeps track of the states of each entity as and when an action is performed.
當呼叫 `SaveChanges()` 方法時，EF API 會根據實體的狀態推論 INSERT、UPDATE 和 DELETE 指令。 ChangeTracker 會在執行操作時追蹤每個實體的狀態。

![[ef-saving.png]]

## Basic Workflow in Entity Framework 實體架構中的基本工作流程

Here you will learn about the basic CRUD workflow using Entity Framework.
在這裡您將了解使用實體框架的基本 CRUD 工作流程。

The following figure illustrates the basic workflow.
下圖說明了基本工作流程。

![[basic-workflow.png]]

Let's understand the above EF workflow:
讓我們來理解一下上面的 EF 工作流程：

1. First of all, you need to define your model. Defining the model includes defining your domain classes, context class derived from DbContext, and configurations (if any). EF will perform CRUD operations based on your model.
    首先，您需要定義您的模型。定義模型包括定義您的網域類別、從 DbContext 派生的上下文類別以及配置（如果有）。 EF 將根據您的模型執行 CRUD 操作。
2. To insert data, add a domain object to a context and call the `SaveChanges()` method. EF API will build an appropriate INSERT command and execute it to the database.
    若要插入數據，請將網域物件新增至上下文並呼叫 `SaveChanges()` 方法。 EF API 將建立適當的 INSERT 命令並將其執行到資料庫中。
3. To read data, execute the LINQ-to-Entities query in your preferred language (C#/VB.NET). EF API will convert this query into SQL query for the underlying relational database and execute it. The result will be transformed into domain (entity) objects and displayed on the UI.
    若要讀取數據，請使用您首選的語言（C#/VB.NET）執行 LINQ-to-Entities 查詢。 EF API 會將此查詢轉換為底層關聯式資料庫的 SQL 查詢並執行。結果將轉換為網域（實體）物件並顯示在 UI 上。
4. To edit or delete data, update or remove entity objects from a context and call the `SaveChanges()` method. EF API will build the appropriate UPDATE or DELETE command and execute it to the database.
    若要編輯或刪除數據，請從上下文中更新或刪除實體對象，然後呼叫 `SaveChanges()` 方法。 EF API 將建立相應的 UPDATE 或 DELETE 命令並將其執行到資料庫中。

---

# What is an Entity in Entity Framework? 實體框架中的實體是什麼？

An entity in Entity Framework is a class that maps to a database table. This class must be included as a `DbSet<TEntity>` type property in the `DbContext` class. EF API maps each entity to a table and each property of an entity to a column in the database.
Entity Framework 中的實體是對應到資料庫表格的類別。此類別必須作為 `DbSet<TEntity>` 類型屬性包含在 `DbContext` 類別中。 EF API 將每個實體對應到一個表，將實體的每個屬性對應到資料庫中的一列。

For example, the following `Student`, and `Grade` are domain classes in the school application.
例如，以下 `Student` 和 `Grade` 是學校應用程式中的網域類別。

```csharp
public class Student
{
    public int StudentID { get; set; }
    public string StudentName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public byte[] Photo { get; set; }
    public decimal Height { get; set; }
    public float Weight { get; set; }

    public Grade Grade { get; set; }
}

public class Grade
{
    public int GradeId { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }

    public ICollection<Student> Students { get; set; }
}
```

The above classes become entities when they are included as `DbSet<TEntity>` properties in a context class (the class which derives from `DbContext`), as shown below.
當上述類別作為 `DbSet<TEntity>` 屬性包含在上下文類別（從 `DbContext` 派生的類別）中時，它們就成為實體，如下所示。

```csharp
public class SchoolContext : DbContext
{
    public SchoolContext()
    {

    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Grade> Grades { get; set; }
}
```

In the above context class, `Students`, and `Grades` properties of type `DbSet<TEntity>` are called entity sets. The `Student`, and `Grade` are entities. EF API will create the `Students` and `Grades` tables in the database, as shown below.
在上面的上下文類別中，類型為 `DbSet<TEntity>` 的 `Students` 和 `Grades` 屬性稱為實體集合。 `Student` 和 `Grade` 是實體。 EF API 將在資料庫中建立 `Students` 和 `Grades` 表，如下所示。

![[dbtables-for-entities.png]]

An Entity can include two types of properties: **Scalar Properties** and **Navigation Properties**.
實體可以包含兩種類型的屬性：**標量屬性**和**導覽屬性**。

## Scalar Property  標量屬性

The primitive type properties are called scalar properties. Each scalar property maps to a column in the database table which stores an actual data. For example, `StudentID, StudentName, DateOfBirth, Photo, Height, Weight` are the scalar properties in the `Student` entity class.
原始型別屬性稱為標量屬性。每個標量屬性都會對應到資料庫表中儲存實際資料的列。例如， `StudentID, StudentName, DateOfBirth, Photo, Height, Weight` 是 `Student` 實體類別中的標量屬性。

```csharp
public class Student
{
    // scalar properties
    public int StudentID { get; set; }
    public string StudentName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public byte[] Photo { get; set; }
    public decimal Height { get; set; }
    public float Weight { get; set; }

    //reference navigation properties
    public Grade Grade { get; set; }
}
```

EF API will create a column in the database table for each scalar property, as shown below.
EF API 將在資料庫表中為每個標量屬性建立一列，如下所示。

![[dbcolumns-for-scalar-properties.png]]

## Navigation Property  導覽屬性

The navigation property represents a relationship to another entity.
導覽屬性表示與另一個實體的關係。

There are two types of navigation properties: **Reference Navigation** and **Collection Navigation**
導覽屬性有兩種類型：參考導覽和集合導覽

### Reference Navigation Property 參考導覽屬性

If an entity includes a property of another entity type, it is called a Reference Navigation Property. It points to a single entity and represents multiplicity of one (1) in the entity relationships.
如果一個實體包含另一個實體類型的屬性，則該屬性稱為參考導覽屬性。它指向單一實體，並在實體關係中表示「1」的多重性。

EF API will create a ForeignKey column in the table for the navigation properties that points to a PrimaryKey of another table in the database. For example, `Grade` are reference navigation properties in the following `Student` entity class.
EF API 會在表格中為導覽屬性建立一個 ForeignKey 資料列，該列指向資料庫中另一個資料表的 PrimaryKey。例如， `Grade` 是以下 `Student` 實體類別中的參考導覽屬性。

```csharp
public class Student
{
    // scalar properties
    public int StudentID { get; set; }
    public string StudentName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public byte[] Photo { get; set; }
    public decimal Height { get; set; }
    public float Weight { get; set; }

    //reference navigation property
    public Grade Grade { get; set; }
}
```

In the database, EF API will create a ForeignKey `Grade_GradeId` in the `Students` table, as shown below.
在資料庫中，EF API 將在 `Students` 表中建立一個 ForeignKey `Grade_GradeId` ，如下所示。

[![Entity Properties in Entity Framework](https://www.entityframeworktutorial.net/images/basics/ref-property-in-dbtable.png "Entity Properties in Entity Framework")](https://www.entityframeworktutorial.net/images/basics/ref-property-in-dbtable.png)

### Collection Navigation Property 集合導覽屬性

If an entity includes a property of generic collection of an entity type, it is called a collection navigation property. It represents multiplicity of many (\*).
如果實體包含某個實體類型的通用集合屬性，則該屬性稱為集合導覽屬性。它表示多個（\*）的多重性。

EF API does not create any column for the collection navigation property in the related table of an entity, but it creates a column in the table of an entity of generic collection. For example, the following `Grade` entity contains a generic collection navigation property `ICollection<Student>`. Here, the `Student` entity is specified as generic type, so EF API will create a column `Grade_GradeId` in the `Students` table in the database.
EF API 不會在實體的相關表中為集合導覽屬性建立任何列，但它會在泛型集合實體的表中建立一列。例如，下列 `Grade` 實體包含一個泛型集合導覽屬性 `ICollection<Student>` 。這裡， `Student` 實體被指定為泛型類型，因此 EF API 將在資料庫的 `Students` 表中建立一個列 `Grade_GradeId` 。

![[col-navigation-dbtable.png]]

## States of Entities   實體狀態

EF API maintains the state of each entity during its lifetime. Each entity has a state based on the operation performed on it via the context class. The entity state represented by an enum `System.Data.Entity.EntityState` in EF 6 and `Microsoft.EntityFrameworkCore.EntityState` in EF Core with the following values:
EF API 會在每個實體的生命週期內維持其狀態。每個實體的狀態取決於透過 context 類別對其執行的操作。實體狀態在 EF 6 中以枚舉 `System.Data.Entity.EntityState` 表示，在 EF Core 中以枚舉 `Microsoft.EntityFrameworkCore.EntityState` 表示，其值如下：

1. Added  額外
2. Modified   修改的
3. Deleted  已刪除
4. Unchanged   未改變
5. Detached   獨立式

The Context not only holds the reference to all the entity objects as soon as retrieved from the database, but also keeps track of entity states and maintains modifications made to the properties of the entity. This feature is known as _Change Tracking_.
上下文不僅在從資料庫檢索到所有實體物件後立即保存其參考，而且還追蹤實體狀態並維護對實體屬性所做的修改。此功能稱為 _ 變更追蹤 _ 。

The change in entity state from the Unchanged to the Modified state is the only state that's automatically handled by the context. All other changes must be made explicitly using proper methods of `DbContext` or `DbSet`. (You will learn about these methods in EF 6 and EF Core sections.)
實體狀態從「未更改」到「已修改」的變更是上下文自動處理的唯一狀態。所有其他變更必須使用 `DbContext` 或 `DbSet` 的正確方法明確進行。 （您將在 EF 6 和 EF Core 章節中了解這些方法。）

EF API builds and executes the INSERT, UPDATE, and DELETE commands based on the state of an entity when the `context.SaveChanges()` method is called. It executes the INSERT command for the entities with Added state, the UPDATE command for the entities with Modified state and the DELETE command for the entities in Deleted state. The context does not track entities in the Detached state. The following figure illustrates the significance of entity states:
當呼叫 `context.SaveChanges()` 方法時，EF API 根據實體的狀態建構並執行 INSERT、UPDATE 和 DELETE 指令。 它對具有 Added 狀態的實體執行 INSERT 命令，並對具有 Modified 狀態的實體執行 UPDATE 命令，並對具有 Deleted 狀態的實體執行 DELETE 命令。 上下文不會追蹤處於 Detached 狀態的實體。下圖說明了實體狀態的重要性：

![[entity-states.png]]

Thus, entity states play an important role in Entity Framework.
因此，實體狀態在實體框架中扮演重要角色。

---

# Context Class in Entity Framework 實體框架中的上下文類

The context class is a very important class while working with EF 6 or EF Core. It represents a session with the underlying database using which you can perform CRUD (Create, Read, Update, Delete) operations.
在使用 EF 6 或 EF Core 時，Context 類別是一個非常重要的類別。它代表與底層資料庫的會話，您可以使用該會話執行 CRUD（建立、讀取、更新、刪除）操作。

The context class in Entity Framework is a class which derives from [System.Data.Entity.DbContext](https://docs.microsoft.com/en-us/dotnet/api/system.data.entity.dbcontext) in EF 6 and EF Core both. An instance of the context class represents Unit Of Work and Repository patterns wherein it can combine multiple changes under a single database transaction.
Entity Framework 中的 context 類別是從 EF 6 和 EF Core 中的 [System.Data.Entity.DbContext](https://docs.microsoft.com/en-us/dotnet/api/system.data.entity.dbcontext) 派生的類別。 上下文類別的實例代表工作單元和儲存庫模式，其中它可以在單一資料庫事務下組合多個變更。

The context class is used to query or save data to the database. It is also used to configure domain classes, database related mappings, change tracking settings, caching, transaction etc.
上下文類別用於查詢或將資料保存到資料庫。它也用於配置網域類別、資料庫相關映射、更改追蹤設定、快取、事務等。

The following `SchoolContext` class is an example of a context class.
以下 `SchoolContext` 類別是上下文類別的一個範例。

```csharp
using System.Data.Entity;

public class SchoolContext : DbContext
{
    public SchoolContext()
    {

    }
    // Entities        
    public DbSet<Student> Students { get; set; }
    public DbSet<StudentAddress> StudentAddresses { get; set; }
    public DbSet<Grade> Grades { get; set; }
}
```

In the above example, the `SchoolContext` class is derived from `DbContext`, which makes it a context class. It also includes an entity set for `Student`, `StudentAddress`, and `Grade` entities (learn about it next).
在上面的範例中， `SchoolContext` 類別派生自 `DbContext` ，這使其成為上下文類別。它還包含 `Student` 、 `StudentAddress` 和 `Grade` 實體集（稍後會介紹）。

Learn more about the context class in the [EF 6 DbContext](https://www.entityframeworktutorial.net/entityframework6/dbcontext.aspx) and [EF Core DbContext](https://www.entityframeworktutorial.net/efcore/entity-framework-core-dbcontext.aspx) chapters.
在 [EF 6 DbContext](https://www.entityframeworktutorial.net/entityframework6/dbcontext.aspx) 和 [EF Core DbContext](https://www.entityframeworktutorial.net/efcore/entity-framework-core-dbcontext.aspx) 章節中了解更多有關上下文類別的資訊。

---

# Development Approaches with Entity Framework 使用實體框架的開發方法

There are three different approaches you can use while developing your application using Entity Framework:
使用實體框架開發應用程式時，可以使用三種不同的方法：

1. Database-First  資料庫優先
2. Code-First  程式碼優先
3. Model-First  模型優先

## Database-First Approach  資料庫優先方法

In the database-first development approach, you generate the context and entities for the existing database using EDM wizard integrated in Visual Studio or executing EF commands.
在資料庫優先開發方法中，您可以使用 Visual Studio 中整合的 EDM 精靈或執行 EF 命令為現有資料庫產生上下文和實體。

![[databasefirst.png]]

EF 6 supports the database-first approach extensively. Visit EF 6 DB-First section to learn about the database-first approach using EF 6.
EF 6 廣泛支援資料庫優先方法。造訪 EF 6 DB-First 部分，了解如何使用 EF 6 實作資料庫優先方法。

EF Core includes limited support for this approach.
EF Core 對此方法的支援有限。

## Code-First Approach  程式碼優先方法

Use this approach when you do not have an existing database for your application. In the code-first approach, you start writing your entities (domain classes) and context class first and then create the database from these classes using migration commands.
如果您的應用程式沒有現成的資料庫，請使用此方法。在程式碼優先方法中，您首先開始編寫實體（域類）和上下文類，然後使用遷移命令從這些類建立資料庫。

Developers who follow the Domain-Driven Design (DDD) principles, prefer to begin with coding their domain classes first and then generate the database required to persist their data.
遵循領域驅動設計 (DDD) 原則的開發人員傾向於先編寫領域類，然後產生保存資料所需的資料庫。

![[code-first.png]]

Visit the [EF 6 Code-First Tutorials](https://www.entityframeworktutorial.net/code-first/what-is-code-first.aspx) section to learn EF 6 code-first development from scratch.
造訪 [EF 6 Code-First 教學](https://www.entityframeworktutorial.net/code-first/what-is-code-first.aspx) 部分，從頭開始學習 EF 6 程式碼優先開發。

Visit [EF Core](https://www.entityframeworktutorial.net/efcore/entity-framework-core.aspx) section to learn about the code-first approach in EF Core.
造訪 [EF Core](https://www.entityframeworktutorial.net/efcore/entity-framework-core.aspx) 部分以了解 EF Core 中的程式碼優先方法。

## Model-First Approach  模型優先方法

In the model-first approach, you create entities, relationships, and inheritance hierarchies directly on the visual designer integrated in Visual Studio and then generate entities, the context class, and the database script from your visual model.
在模型優先方法中，您可以直接在 Visual Studio 中整合的視覺化設計器上建立實體、關係和繼承層次結構，然後從視覺化模型產生實體、上下文類別和資料庫腳本。

![[model-first.png]]

EF 6 includes limited support for this approach.
EF 6 對此方法提供了有限的支援。

EF Core does not support this approach.
EF Core 不支援此方法。

## Choosing the Development Approach for Your Application 為您的應用程式選擇開發方法

Use the following flow chart to decide which is the right approach to develop your application using Entity Framework:
使用以下流程圖來確定使用實體框架開發應用程式的正確方法：

![[choose-modeling.png]]

As per the above figure, if you already have an existing application with domain classes, then you can use the code-first approach because you can create a database from your existing classes. If you have an existing database, then you can create an EDM from an existing database in the database-first approach. If you do not have an existing database or domain classes, and you prefer to design your DB model on the visual designer, then go for the Model-first approach.
如上圖所示，如果您已經有一個包含領域類別的現有應用程序，那麼您可以使用程式碼優先方法，因為您可以從現有類別建立資料庫。如果您已經有資料庫，那麼您可以使用資料庫優先方法從現有資料庫建立 EDM。如果您沒有現有資料庫或領域類，並且您更喜歡在視覺化設計器上設計資料庫模型，那麼請選擇模型優先方法。

---

# Persistence Scenarios in Entity Framework 實體框架中的持久化場景

There are two scenarios when persisting (saving) an entity to the database using Entity Framework: the Connected Scenario and the Disconnected Scenario.
使用實體框架將實體持久化（儲存）到資料庫時有兩種情況：連線情況和斷開情況。

## Connected Scenario  連結場景

In the connected scenario, the same instance of the context class (derived from `DbContext`) is used in retrieving and saving entities. It keeps track of all entities during its lifetime. This is useful in windows applications with the local database or the database on the same network.
在連接場景中，上下文類別（派生自 `DbContext`）的同一個實例用於檢索和保存實體。它會在其生命週期內追蹤所有實體。這對於使用本機資料庫或同一網路上的資料庫的 Windows 應用程式非常有用。

![[persistance-fg1.png]]

**Pros:  優點：**

- Performs fast.  執行速度很快。
- The context keeps track of all entities and automatically sets an appropriate state as and when changes occur to entities.
    上下文追蹤所有實體，並在實體發生變化時自動設定適當的狀態。

**Cons:  缺點：**

- The context stays alive, so the connection with the database stays open.
    上下文保持活動狀態，因此與資料庫的連線保持開啟。
- Utilizes more resources.
    利用更多資源。

## Disconnected Scenario  斷網場景

In the disconnected scenario, different instances of the context are used to retrieve and save entities to the database. An instance of the context is disposed after retrieving data and a new instance is created to save entities to the database.
在離線場景中，上下文的不同實例用於檢索實體並將其保存到資料庫。檢索資料後，上下文的一個實例將被釋放，並建立一個新的實例以將實體儲存到資料庫。

![[persistance-fg2.png]]

The disconnected scenario is complex because an instance of the context doesn't track entities, so you must set an appropriate state to each entity before saving entities using `SaveChanges()`. In the figure above, the application retrieves an entity graph using Context 1 and then the application performs some CUD (Create, Update, Delete) operations using Context 2. Context 2 doesn't know what operation has been performed on the entity graph in this scenario.
斷開連線的場景比較複雜，因為上下文實例不會追蹤實體，因此在使用 `SaveChanges()` 儲存實體之前，必須為每個實體設定適當的狀態。在上圖中，應用程式使用上下文 1 檢索實體圖，然後使用上下文 2 執行一些 CUD（建立、更新、刪除）操作。在這種情況下，上下文 2 並不知道對實體圖執行了哪些操作。

This is useful in web applications or applications with a remote database.
這在 Web 應用程式或具有遠端資料庫的應用程式中很有用。

**Pros:  優點：**

- Utilizes less resources compared to the connected scenario.
    與連接場景相比，使用的資源更少。
- No open connection with the database.
    沒有開啟與資料庫的連線。

**Cons:  缺點：**

- Need to set an appropriate state to each entity before saving.
    在儲存之前需要為每個實體設定適當的狀態。
- Performs slower than the connected scenario.
    執行速度比連線場景慢。
