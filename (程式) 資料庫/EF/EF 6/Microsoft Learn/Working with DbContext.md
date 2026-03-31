---
aliases:
date: 2020-10-15
update:
author: Microsoft
language: C#
sourceurl: https://learn.microsoft.com/en-us/ef/ef6/fundamentals/working-with-dbcontext
tags:
  - CSharp
  - 資料庫
  - EF6
---

# Working with DbContext  處理 DbContext

In order to use Entity Framework to query, insert, update, and delete data using .NET objects, you first need to [Create a Model](https://learn.microsoft.com/en-us/ef/ef6/modeling/) which maps the entities and relationships that are defined in your model to tables in a database.
若要使用 Entity Framework 以 .NET 物件查詢、插入、更新和刪除資料，您首先需要建立一個模型，將模型中定義的實體和關係映射到資料庫的表中。

Once you have a model, the primary class your application interacts with is `System.Data.Entity.DbContext` (often referred to as the context class). You can use a `DbContext` associated to a model to:
建立模型後，您的應用程式主要互動的類別是 `System.Data.Entity.DbContext` （通常稱為上下文類別）。您可以使用與模型關聯的 `DbContext` 來：

- Write and execute queries
    撰寫和執行查詢
- Materialize query results as entity objects
    將查詢結果實體化為物件
- Track changes that are made to those objects
    追蹤對這些物件的變更
- Persist object changes back on the database
    將物件的變更持續寫回資料庫
- Bind objects in memory to UI controls
    將記憶體中的物件綁定到 UI 控制項

This page gives some guidance on how to manage the context class.
本頁面提供一些有關如何管理上下文類別的指導。

# Defining a DbContext derived class 定義 DbContext 派生類別

The recommended way to work with context is to define a class that derives from `DbContext` and exposes `DbSet` properties that represent collections of the specified entities in the context. If you are working with the EF Designer, the context will be generated for you. If you are working with Code First, you will typically write the context yourself.
與上下文互動的建議方式是定義一個派生自 `DbContext` 的類別，並公開 `DbSet` 屬性，這些屬性代表上下文中指定的實體集合。如果您正在使用 EF 設計師，上下文將會為您產生。如果您正在使用先進的 Code First 方法，您通常需要自行撰寫上下文。

```csharp
public class ProductContext : DbContext
{
    public DbSet<Category> Categories { get; set; }
    public DbSet<Product> Products { get; set; }
}
```

Once you have a context, you would query for, add (using `Add` or `Attach` methods ) or remove (using `Remove`) entities in the context through these properties. Accessing a `DbSet` property on a context object represent a starting query that returns all entities of the specified type. Note that just accessing a property will not execute the query. A query is executed when:
一旦您擁有上下文，您會透過這些屬性來查詢、新增（使用 `Add` 或 `Attach` 方法）或刪除（使用 `Remove` ）上下文中的實體。在上下文物件上存取 `DbSet` 屬性代表一個起始查詢，該查詢會傳回所有指定類型的實體。請注意，僅僅存取屬性不會執行查詢。當查詢執行時：

- It is enumerated by a `foreach` (C#) or `For Each` (Visual Basic) statement.
  它由 `foreach` (C#) 或 `For Each` (Visual Basic) 陳述式進行列舉。
- It is enumerated by a collection operation such as `ToArray`, `ToDictionary`, or `ToList`.
  它由集合操作，例如 `ToArray` 、 `ToDictionary` 或 `ToList` 進行列舉。
- LINQ operators such as `First` or `Any` are specified in the outermost part of the query.
  LINQ 運算子，例如 `First` 或 `Any` ，會在查詢的最外層指定。
- One of the following methods are called: the `Load` extension method, `DbEntityEntry.Reload`, `Database.ExecuteSqlCommand`, and `DbSet<T>.Find`, if an entity with the specified key is not found already loaded in the context.
  如果指定鍵的實體尚未在上下文中已經載入，則會呼叫下列其中一種方法： `Load` 擴充功能、 `DbEntityEntry.Reload` 、 `Database.ExecuteSqlCommand` 和 `DbSet<T>.Find` 。

# Lifetime  生命週期

The lifetime of the context begins when the instance is created and ends when the instance is either disposed or garbage-collected. **Use `using` if you want all the resources that the context controls to be disposed at the end of the block.** When you use `using`, the compiler automatically creates a try/finally block and calls dispose in the `finally` block.
`DbContext` 的生命週期從實例建立時開始，到實例被處置或被垃圾收集結束。**如果您希望 `DbContext` 控制的資源在塊結束時被處置，請使用 `using`。** 當您使用 `using` 時，編譯器會自動建立一個 `try/finally` 區塊，並在 `finally` 區塊中呼叫 `Dispose`。

```csharp
public void UseProducts()
{
    using (var context = new ProductContext())
    {     
        // Perform data access using the context
    }
}
```

Here are some general guidelines when deciding on the lifetime of the context:
在決定 `DbContext` 的生命週期時，以下是一些一般性的指導原則：

- When working with Web applications, use a context instance per request.
  在處理網站應用程式時，==每個請求使用一個 `DbContext` 實例。==
- When working with Windows Presentation Foundation (WPF) or Windows Forms, use a context instance per form. This lets you use change-tracking functionality that context provides.
  在處理 Windows Presentation Foundation (WPF) 或 Windows Forms 時，==每個表單使用一個上下文實例。==這讓你可以使用上下文提供的變更追蹤功能。
- If the context instance is created by a dependency injection container, it is usually the responsibility of the container to dispose the context.
  如果上下文實例是由相依性注入容器創建的，通常是由容器負責處置上下文。
- If the context is created in application code, ==remember to dispose of the context when it is no longer required.==
  如果上下文是在應用程式碼中創建的，請記得在不再需要時處置上下文。
- When working with long-running context consider the following:
  在處理長時間執行的上下文時，請考慮以下事項：
    - As you load more objects and their references into memory, the memory consumption of the context may increase rapidly. This may cause performance issues.
      當您將更多物件及其參考載入記憶體時，上下文的記憶體消耗可能會快速增加。這可能會導致效能問題。
    - **The context is not thread-safe**, therefore it should not be shared across multiple threads doing work on it concurrently.
      上下文並非線程安全，因此不應在多個線程同時對其進行工作時共享。
    - If an exception causes the context to be in an unrecoverable state, the whole application may terminate.
      如果發生例外導致上下文處於無法恢復的狀態，整個應用程式可能會終止。
    - The chances of running into concurrency-related issues increase as the gap between the time when the data is queried and updated grows.
      當資料查詢與更新之間的時間差距增長時，遇到並發相關問題的機率會增加。

# Connections  連接

By default, the context manages connections to the database. The context opens and closes connections as needed. For example, the context opens a connection to execute a query, and then closes the connection when all the result sets have been processed.
預設情況下，上下文會管理與資料庫的連接。上下文會根據需要開啟和關閉連接。例如，上下文會開啟一個連接來執行查詢，然後在所有結果集都處理完畢後關閉連接。

There are cases when you want to have more control over when the connection opens and closes. For example, when working with SQL Server Compact, it is often recommended to maintain a separate open connection to the database for the lifetime of the application to improve performance. You can manage this process manually by using the `Connection` property.
有時候，您可能需要對連接的開啟和關閉時機有更多的控制權。例如，在處理 SQL Server Compact 時，==通常建議在應用程式的整個生命週期中維持一個獨立的開啟連接到資料庫，以提升效能==。您可以使用 `Connection` 屬性來手動管理這個過程。
