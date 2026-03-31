---
aliases:
date:
update:
author: ZZZ Projects
language: C#
sourceurl: https://www.entityframeworktutorial.net/code-first/what-is-code-first.aspx
tags:
  - CSharp
  - 資料庫
  - EF6
  - EF_Code_First
---

# Links

[Entity Framework Fluent API](https://entityframework.net/fluent-api)
[Fluent API Configurations in EF 6](https://www.entityframeworktutorial.net/code-first/fluent-api-in-code-first.aspx)
[Entity Mappings using Fluent API in EF 6](https://www.entityframeworktutorial.net/code-first/configure-entity-mappings-using-fluent-api.aspx)
[Property Mappings using Fluent API](https://www.entityframeworktutorial.net/code-first/configure-property-mappings-using-fluent-api.aspx)

# Entity Framework Fluent API

## Discover How to Map Without Annotation 探索如何無注釋地進行映射

[code-first](https://entityframework.net/articles/tag/code-first)
[fluent-api](https://entityframework.net/articles/tag/fluent-api)

When working with Entity Framework Code First the default behavior is to map your POCO classes to tables using a set of conventions baked into EF.
當使用 Entity Framework Code First 時，預設行為是使用 EF 內置的一組慣例將您的 POCO 類別映射到表格。

- Fluent API specifies the model configuration that you can define with data annotations as well as some additional functionality that is not possible with data annotations.
    Fluent API 指定您可以使用資料標注定義的模型配置，以及一些資料標注無法實現的額外功能。
- We can configure many different things by using it because it provides more configuration options than data annotation attributes.
    我們可以使用它來配置許多不同的事物，因為它提供的配置選項比資料標注屬性更多。
- Data annotations and the fluent API can be used together, but precedence of **Fluent API > data annotations > default conventions**.
    資料標注和 Fluent API 可以一起使用，但 **Fluent API 的優先權高於資料標注，再來是預設慣例。**

## DbModelBuilder

DbModelBuilder is typically used to configure a model by overriding `DbContext.OnModelCreating(DbModelBuilder)` .
DbModelBuilder 通常用於配置模型，透過覆蓋 `DbContext.OnModelCreating(DbModelBuilder)` 來完成。

You can override the `DbContext.OnModelCreating` method and use a parameter modelBuilder of type `DbModelBuilder` to configure domain classes.
您可以覆蓋 `DbContext.OnModelCreating` 方法，並使用型別為 `DbModelBuilder` 的 modelBuilder 參數來配置領域類別。

## Key  鍵

You can use the `HasKey()` method to configure the name of the primary key constraint in the database.
您可以使用 `HasKey()` 方法來配置資料庫中主鍵約束的名稱。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .HasKey(a => a.AuthId);
}
```

[Try it  試試看](https://dotnetfiddle.net/x4nMB1)

## Concurrency Token (ConcurrencyCheck)  並行令牌

You can use the `IsConcurrencyToken()` method to configure a property as a concurrency token.
您可以使用 `IsConcurrencyToken()` 方法將屬性配置為並行令牌。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Property(a => a.LastName)
        .IsConcurrencyToken();
}
```

[Try it  試試看](https://dotnetfiddle.net/OwMcdJ)

## Exclude Types or Properties  排除類型或屬性

You can use the `Ignore()` method to exclude a type or a property from the model.
您可以使用 `Ignore()` 方法從模型中排除一個類型或屬性。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Ignore(a => a.IgnoreProperty);
}
```

[Try it  試試看](https://dotnetfiddle.net/wzep6l)

## Required  必需

You can use the `IsRequired()` method to indicate that a property is required.
您可以使用 `IsRequired()` 方法來指示屬性是必需的。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Property(a => a.LastName)
        .IsRequired();

    modelBuilder.Entity<Author>()
        .Property(a => a.FirstName)
        .IsRequired();
}
```

[Try it  試試看](https://dotnetfiddle.net/XHhJWj)

## Table Mapping  表格映射

You can use the `ToTable()` method to configure the table that a type maps to.
您可以使用 `ToTable()` 方法來配置一個類型映射到的表格。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
     modelBuilder.Entity<Author>()
        .ToTable("AuthorsData");
}
```

[Try it  試試看](https://dotnetfiddle.net/KeP2EJ)

## Column Mapping  欄位映射

You can use the `HasColumnName()` method to configure the column to which a property is mapped.
您可以使用 `HasColumnName()` 方法來設定屬性對應的欄位。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Property(a => a.AuthorId)
        .HasColumnName("Author_Id");
}
```

[Try it  試試看](https://dotnetfiddle.net/d683kR)

## Foreign Key  外鍵

You can use the `HasForeignKey()` method to configure the foreign key constraint name for a relationship.
您可以使用 `HasForeignKey()` 方法來設定關聯的外鍵約束名稱。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>()
        .HasRequired<Author>(b => b.Author)
        .WithMany(a => a.Books)
        .HasForeignKey<int>(b => b.AuthorId);
}
```

[Try it  試試看](https://dotnetfiddle.net/GrB0j8)

## Default Schema  預設模式

You can use the `HasDefaultSchema()` method to specify a default schema.
您可以使用 `HasDefaultSchema()` 方法來指定預設模式。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.HasDefaultSchema("Admin");
}
```

[Try it  試試看](https://dotnetfiddle.net/q83NlG)

## Index  索引

You can use the `HasColumnAnnotation()` method to configure the name of an index.
您可以使用 `HasColumnAnnotation()` 方法來配置索引的名稱。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Property(a => a.Email)
        .HasColumnType("VARCHAR")
        .HasMaxLength(450)
        .HasColumnAnnotation("Index", new IndexAnnotation(new IndexAttribute("Index_Email") { IsUnique = true }));
}
```

[Try it  試試看](https://dotnetfiddle.net/2vLIfu)

Last updated: 2025-06-18  最後更新：2025-06-18
Author: ZZZ Projects  作者：ZZZ Projects

---

# Fluent API Configurations in EF 6 《EF 6 中 Fluent API 的配置》

Entity Framework Fluent API is used to configure domain classes to override conventions. EF Fluent API is based on a Fluent API design pattern (a.k.a [Fluent Interface](https://en.wikipedia.org/wiki/Fluent_interface)) where the result is formulated by [method chaining](https://en.wikipedia.org/wiki/Method_chaining).
Entity Framework Fluent API 用於配置領域類別以覆蓋傳統規範。EF Fluent API 基於 Fluent API 設計模式（亦稱為 Fluent Interface），結果通過方法鏈接來構建。

In Entity Framework 6, the [DbModelBuilder](https://msdn.microsoft.com/en-us/library/system.data.entity.dbmodelbuilder\(v=vs.113\).aspx) class acts as a Fluent API using which we can configure many different things. It provides more options of configurations than Data Annotation attributes.
在 Entity Framework 6 中，DbModelBuilder 類別作為 Fluent API 使用，我們可以通過它配置許多不同的事物。它提供的配置選項比數據注釋屬性更多。

To write Fluent API configurations, override the `OnModelCreating()` method of `DbContext` in a context class, as shown below.
要寫 Fluent API 配置，在上下文類別中覆蓋 `DbContext` 的 `OnModelCreating()` 方法，如下所示。

```csharp
public class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Write Fluent API configurations here
	}
}
```

You can use Data Annotation attributes and Fluent API at the same time. Entity Framework gives precedence to Fluent API over Data Annotation attributes.
您可以使用資料標注屬性和 Fluent API 同時使用。Entity Framework 會將 Fluent API 的優先級設定在資料標注屬性之上。

Fluent API configures the following aspects of a model in Entity Framework 6:
Fluent API 在 Entity Framework 6 中配置以下模型的各個方面：

1. Model-wide Configuration: Configures the default Schema, entities to be excluded in mapping, etc.
    模型全範圍配置：配置預設的 Schema、在映射中排除的實體等。
2. Entity Configuration: Configures entity to table and relationship mappings e.g. PrimaryKey, Index, table name, one-to-one, one-to-many, many-to-many etc.
    實體配置：配置實體到表的映射和關係映射，例如主鍵、索引、表名、一對一、一對多、多對多等。
3. Property Configuration: Configures property to column mappings e.g. column name, nullability, foreign key, data type, concurrency column, etc.
    屬性配置：配置屬性到列的映射，例如列名、可空性、外鍵、數據類型、并发列等。

The following table lists important Fluent API methods.
以下表格列出了重要的 Fluent API 方法。

## Fluent API methods

### 類別樹

[namespace System.Data.Entity](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity)
 ├─ [DbModelBuilder](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dbmodelbuilder)
 │ ├─ Entity\<TEntityType\>
 │ ├─ HasDefaultSchema
 │ ├─ ComplexType
 │ └─ RegisterEntityType
 ├─ [Database](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.database)
 │ └─[SetInitializer](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.database.setinitializer)([IDatabaseInitializer\<T\>()](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.idatabaseinitializer-1))
 │ 　　　├─ [CreateDatabaseIfNotExists](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.createdatabaseifnotexists-1)
 │ 　　　├─ [DropCreateDatabaseAlways](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dropcreatedatabasealways-1)
 │ 　　　├─ [DropCreateDatabaseIfModelChanges](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dropcreatedatabaseifmodelchanges-1)
 │ 　　　├─ [MigrateDatabaseToLatestVersion](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.migratedatabasetolatestversion-2)
 │ 　　　└─ [NullDatabaseInitializer](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.nulldatabaseinitializer-1)
 ├─ [DbSet](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dbset-1)
 │ └─SetDatabaseInitializer
 ├─ [DbConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dbconfiguration)
 │ └─SetDatabaseInitializer
 └─ [DbContext](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dbcontext)
    ├─ OnModelCreating (virtual)
    ├─ SaveChanges
    └─ SaveChangesAsync

[namespace System.Data.Entity.ModelConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration)
[namespace System.Data.Entity.ModelConfiguration.Configuration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration)
 ├─ [AssociationMappingConfiguration (abstract)](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.associationmappingconfiguration) 負責關聯 (一對多、多對多、一對一) 的設定。
 │ ├─ [ForeignKeyAssociationMappingConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.foreignkeyassociationmappingconfiguration)
 │ └─ [ManyToManyAssociationMappingConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.manytomanyassociationmappingconfiguration)
 │
 ├─ [RequiredNavigationPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.requirednavigationpropertyconfiguration-2)
 │
 ├─ [StructuralTypeConfiguration (abstract)](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.structuraltypeconfiguration-1)
 │ 　　├─ [ComplexTypeConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.complextypeconfiguration-1)
 │　　 │　 設定複合型別 (Complex Type)。
 │ 　　└─ [EntityTypeConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.entitytypeconfiguration-1)
 │　　　　  最常用，對應一個實體類型、StoredProcedures 的 Fluent API 設定。
 │　　　　  例如 `.HasKey(...)`, `.ToTable(...)`, `.Property(...)`。
 │
 ├─ [PrimitivePropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.primitivepropertyconfiguration)
 │ │　Fluent API 設定單一屬性 (字串長度、精度、是否必要等)。
 │ │　例如 `.Property(s => s.Name).IsRequired().HasMaxLength(50)`。
 │ ├─ [DateTimePropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.datetimepropertyconfiguration)
 │ ├─ [DecimalPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.decimalpropertyconfiguration)
 │ └─ [LengthPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.lengthpropertyconfiguration)
 │ 　　├─ [BinaryPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.binarypropertyconfiguration)
 │ 　　└─ [StringPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.stringpropertyconfiguration)
 │
 ├─ [ManyNavigationPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.manynavigationpropertyconfiguration-2)
 │
 ├─ [CascadableNavigationPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.cascadablenavigationpropertyconfiguration)
 │
 ├─ [PropertyConfiguration (ASP.NET) (abstract)](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnet.odata.builder.propertyconfiguration)
 │ └─ [NavigationPropertyConfiguration](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnet.odata.builder.navigationpropertyconfiguration)
 │
 ├─ [EntityMappingConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.entitymappingconfiguration-1)
 │
 ├─ [ModificationStoredProceduresConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.modificationstoredproceduresconfiguration-1)
 ├─ [ModificationStoredProcedureConfigurationBase (abstract)](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.modificationstoredprocedureconfigurationbase)
 │ ├─ [DeleteModificationStoredProcedureConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.deletemodificationstoredprocedureconfiguration-1)
 │ ├─ [InsertModificationStoredProcedureConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.insertmodificationstoredprocedureconfiguration-1)
 │ ├─ [ManyToManyModificationStoredProcedureConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.manytomanymodificationstoredprocedureconfiguration-2)
 │ └─ [UpdateModificationStoredProcedureConfiguration](https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.updatemodificationstoredprocedureconfiguration-1)
 │
 └─ ConventionTypeConfiguration 與其他 Conventions 類別 → **屬於 Conventions API (慣例 API)**

### DbSet 類別的重要方法

| Method Name                       | Return Type                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Add<br><br>添加                     | Added entity type<br><br>新增實體類型                       | Adds the given entity to the context with the Added state. When the changes are saved, the entities in the Added states are inserted into the database. After the changes are saved, the object state changes to Unchanged.  <br>將給定實體以「已新增」狀態新增至上下文。儲存變更後，處於「已新增」狀態的實體將插入資料庫。儲存變更後，物件狀態將變更為“未變更”。  <br><br>Example:<br>`dbcontext.Students.Add(studentEntity)`                                                                                                                                                                                                                                           |
| AsNoTracking\<Entity\><br><br>無追蹤 | DBQuery\<Entity\><br><br>資料庫查詢                        | Returns a new query where the entities returned will not be cached in the DbContext. (Inherited from DbQuery.)  <br>傳回一個新查詢，其中傳回的實體不會快取在 `DbContext` 中。 （繼承自 `DbQuery`。）  <br>  <br>**Entities returned as AsNoTracking will not be tracked by DBContext. This will be a significant performance boost for read-only entities.  <br>傳回為 AsNoTracking 的實體不會被 DBContext 追蹤。這將顯著提升只讀實體的效能。**  <br>  <br>Example:<br>`var studentList = dbcontext.Students.AsNoTracking<Student>().ToList<Student>();`                                                                                                        |
| Attach(Entity)<br><br>附加實體        | Entity which was passed as parameter<br><br>作為參數傳遞的實體 | Attaches the given entity to the context in the Unchanged state  <br>將給定實體附加到未改變狀態的上下文  <br>  <br>Example:<br>`dbcontext.Students.Attach(studentEntity);`                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Create<br><br>創造                  | Entity<br><br>實體                                      | Creates a new instance of an entity for the type of this set. This instance is not added or attached to the set. The instance returned will be a proxy if the underlying context is configured to create proxies and the entity type meets the requirements for creating a proxy.  <br>為該集合的類型建立一個新的實體實例。此實例未新增或附加到集合。如果底層上下文配置為建立代理，且實體類型符合建立代理的要求，則傳回的實例將是代理。  <br>  <br>Example:<br>`var newStudentEntity = dbcontext.Students.Create();`                                                                                                                                                              |
| Find(int)<br><br>尋找               | Entity type<br><br>實體類型                               | Uses the primary key value to find an entity tracked by the context. If the entity is not in the context, then a query will be executed and evaluated against the data in the data source, and null is returned if the entity is not found in the context or in the data source. Note that the Find also returns entities that have been added to the context but have not yet been saved to the database.  <br>使用主鍵值尋找上下文追蹤的實體。如果該實體不在上下文中，則會執行查詢並根據資料來源中的資料進行評估；如果在上下文或資料來源中均未找到該實體，則傳回 null。請注意，「查找」也會傳回已新增至上下文但尚未儲存到資料庫的實體。  <br>  <br>Example:<br>`Student studEntity = dbcontext.Students.Find(1);` |
| Include<br><br>包括                 | DBQuery<br><br>資料庫查詢                                  | Returns the included non-generic LINQ to Entities query against a DbContext. (Inherited from DbQuery)  <br>傳回包含的針對 DbContext 的非泛型 LINQ to Entities 查詢。 （繼承自 DbQuery）  <br>  <br>Example:<br>`var studentList = dbcontext.Students.Include("StudentAddress").ToList<Student>();`<br>`var studentList = dbcontext.Students.Include(s => s.StudentAddress).ToList<Student>();`                                                                                                                                                                                                                               |
| Remove<br><br>消除                  | Removed entity<br><br>已移除實體                           | Marks the given entity as Deleted. When the changes are saved, the entity is deleted from the database. The entity must exist in the context in some other state before this method is called.  <br>將給定實體標記為「已刪除」。更改儲存後，該實體將從資料庫中刪除。在呼叫此方法之前，該實體必須以其他狀態存在於上下文中。  <br>  <br>Example:<br>`dbcontext.Students.Remove(studentEntity);`                                                                                                                                                                                                                                                                        |
| SqlQuery<br><br>SQL 查詢            | DBSqlQuery<br><br>資料庫查詢                               | Creates a raw SQL query that will return entities in this set. By default, the entities returned are tracked by the context; this can be changed by calling AsNoTracking on the DbSqlQuery\<TEntity\> returned from this method.  <br>建立一個原始 SQL 查詢，用於傳回此集合中的實體。預設情況下，傳回的實體會被上下文追蹤；可以透過在 DbSqlQuery 上呼叫 AsNoTracking 來更改此設定。\<TEntity\> 從此方法返回。  <br>  <br>Example:<br>`var studentEntity = dbcontext.Students.SqlQuery("select * from student where studentid = 1").FirstOrDefault<Student>();`                                                                                                          |

### DbContext 類別的重要方法

<table>
<tbody><tr>
	<th>分類</th>
	<th>Name</th>
	<th>Usage</th>
</tr>

<tr>
	<td rowspan="3">Properties</td>
	<td>ChangeTracker</td>
	<td>Provides access to information and operations for entity instances that this context is tracking.<br>提供對此上下文正在追蹤的實體實例的資訊和操作的存取。</td>
</tr>
<tr>
    <td>Configuration</td>
    <td>Provides access to configuration options.<br>提供對配置選項的存取。</td>
</tr>
<tr>
    <td>Database</td>
    <td>Provides access to database-related information and operations.<br>提供對資料庫相關資訊和操作的存取。</td>
</tr>

<tr>
	<td rowspan="5">Methods</td>
	<td>Entry</td>
	<td>Gets an `DbEntityEntry` for the given entity. The entry provides access to change tracking information and operations for the entity.
取得給定實體的 `DbEntityEntry` 。該條目提供對實體的更改追蹤資訊和操作的存取權。</td>
</tr>
<tr>
	<td>SaveChanges</td>
	<td>Configures the primary key property for the entity type.<br>設定實體型別的主鍵屬性。</td>
</tr>
<tr>
	<td>SaveChangesAsync</td>
	<td>Configures that the class or property should not be mapped to a table or column.<br>設定該類別或屬性不應該映射到表格或欄位。</td>
</tr>
<tr>
	<td>Set</td>
	<td>Allows advanced configuration related to how the entity is mapped to the database schema.<br>允許進行與實體如何映射到資料庫模式相關的進階配置。</td>
</tr>
<tr>
	<td>OnModelCreating</td>
	<td>Configures the name of the column(s) for the left foreign key. The left foreign key points to the parent entity of the navigation property specified in the HasMany call.<br>配置左外鍵的列名稱。左外鍵指向 HasMany 呼叫中指定的導航屬性的父實體。</td>
</tr>
</tbody></table>

### 重要的 Fluent API 方法

<table>
<tbody><tr>
	<th>Configurations<br>配置</th>
	<th>Fluent API Methods<br>Fluent API 方法</th>
	<th>Usage<br>使用</th>
</tr>

<tr>
	<td rowspan="2"><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.dbmodelbuilder">Model-wide Configurations</a><br>模型範圍配置</td>
	<td>HasDefaultSchema()</td>
	<td>Specifies the default database schema.<br>指定預設的資料庫模式。</td>
</tr>
<tr>
	<td>ComplexType()</td>
	<td>Configures the class as complex type.<br>將類別設定為複合型態。</td>
</tr>

<tr>
	<td rowspan="7"><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.entitytypeconfiguration-1">Entity Configurations</a><br>實體配置</td>
	<td>HasIndex()</td>
	<td>Configures the index property for the entity type.<br>設定實體型別的索引屬性。</td>
</tr>
<tr>
	<td>HasKey()</td>
	<td>Configures the primary key property for the entity type.<br>設定實體型別的主鍵屬性。</td>
</tr>
<tr>
	<td>Ignore()</td>
	<td>Configures that the class or property should not be mapped to a table or column.<br>設定該類別或屬性不應該映射到表格或欄位。</td>
</tr>
<tr>
	<td>Map()</td>
	<td>Allows advanced configuration related to how the entity is mapped to the database schema.<br>允許進行與實體如何映射到資料庫模式相關的進階配置。</td>
</tr>
<tr>
	<td>MapLeftKey()</td>
	<td>Configures the name of the column(s) for the left foreign key. The left foreign key points to the parent entity of the navigation property specified in the HasMany call.<br>配置左外鍵的列名稱。左外鍵指向 HasMany 呼叫中指定的導航屬性的父實體。</td>
</tr>
<tr>
	<td>MapRightKey()</td>
	<td>Configures the name of the column(s) for the right foreign key. The right foreign key points to the parent entity of the the navigation property specified in the WithMany call.<br>配置右外鍵的列名稱。右外鍵指向 WithMany 呼叫中指定的導航屬性的父實體。</td>
</tr>
<tr>
	<td>ToTable()</td>
	<td>Configures the table name for the entity.<br>設定實體的表格名稱。</td>
</tr>

<tr>
	<td rowspan="11"><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.primitivepropertyconfiguration">Property Configurations</a><br>屬性配置</td>
	<td>HasColumnAnnotation()</td>
	<td>Sets an annotation in the model for the database column used to store the property.<br>為用於存儲屬性的資料庫列設定一個注釋。</td>
</tr>
<tr>
	<td>IsRequired()</td>
	<td>Configures the property to be required on SaveChanges().<br>設定屬性在 SaveChanges() 時為必填。</td>
</tr>
<tr>
	<td>IsConcurrencyToken()</td>
	<td>Configures the property to be used as an optimistic concurrency token.<br>設定屬性為使用作為樂觀并发令牌。</td>
</tr>
<tr>
	<td>IsOptional()</td>
	<td>Configures the property to be optional which will create a nullable column in the database.<br>設定屬性為可選，將在資料庫中建立一個可為 null 的欄位。</td>
</tr>

<tr>
	<td>HasParameterName()</td>
	<td>Configures the name of the parameter used in the stored procedure for the property.<br>設定屬性在存儲過程中使用的參數名稱。</td>
</tr>
<tr>
	<td>HasDatabaseGeneratedOption()</td>
	<td>Configures how the value will be generated for the corresponding column in the database e.g. computed, identity or none.<br>設定資料庫中對應欄位的值將如何生成，例如計算值、自增或無。</td>
</tr>
<tr>
	<td>HasColumnOrder()</td>
	<td>Configures the order of the database column used to store the property.<br>設定用於存儲屬性的資料庫欄位的順序。</td>
</tr>
<tr>
	<td>HasColumnType()</td>
	<td>Configures the data type of the corresponding column of a property in the database.<br>設定資料庫中屬性對應的欄位資料型態。</td>
</tr>
<tr>
	<td>HasColumnName()</td>
	<td>Configures the corresponding column name of a property in the database.<br>設定資料庫中屬性對應的欄位名稱。</td>
</tr>
<tr>
	<td>HasMaxLength()</td>
	<td>Configures the property to have the specified maximum length.<br>配置屬性以具有指定的最大長度。</td>
</tr>
<tr>
	<td>HasPrecision()</td>
	<td>Configures the precision and scale of the property.<br>配置屬性的精度和比例。</td>
</tr>

<tr>
	<td rowspan="11"><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.manynavigationpropertyconfiguration-2">Navigation Configuration </a><br><br><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.entitytypeconfiguration-1.hasmany">Entity Configuration</a><br><br><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.dependentnavigationpropertyconfiguration-1">Dependent Configuration</a><br><br><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.configuration.cascadablenavigationpropertyconfiguration">Cascadable Configuration</a><br><br>關聯配置</td>
	<td>HasRequired()</td>
	<td>Configures the required relationship which will create a non-nullable foreign key column in the database.<br>設定必要的關係，將在資料庫中建立一個不可為 null 的外鍵欄位。</td>
</tr>
<tr>
	<td>HasOptional()</td>
	<td>Configures an optional relationship which will create a nullable foreign key in the database.<br>設定一個可選關係，將在資料庫中建立一個可為 null 的外鍵。</td>
</tr>
<tr>
	<td>HasMany()</td>
	<td>Configures the Many relationship for one-to-many or many-to-many relationships.<br>設定一對多或多對多關係的 Many 關係。</td>
</tr>
<tr>
	<td>WithMany()</td>
	<td>Configures the relationship to be many:many without a navigation property on the other side of the relationship.<br>將關係配置為多對多，且關係的另一端沒有導航屬性。</td>
</tr>
<tr>
	<td>WithRequired()</td>
	<td>Configures the relationship to be many:required without a navigation property on the other side of the relationship.<br>將關係配置為多：必要，而關係的另一側沒有導航屬性。</td>
</tr>
<tr>
	<td>WithRequiredPrincipal()</td>
	<td>Configures the relationship to be required:required without a navigation property on the other side of the relationship. The entity type being configured will be the principal in the relationship. The entity type that the relationship targets will be the dependent and contain a foreign key to the principal.<br>將關係配置為必要：必要，而關係的另一端沒有導航屬性。正在配置的實體類型將成為關係中的主體。關係所針對的實體類型將成為從屬實體，並包含主體的外鍵。</td>
</tr>
<tr>
	<td>WithRequiredDependent()</td>
	<td>Configures the relationship to be required:required without a navigation property on the other side of the relationship. The entity type being configured will be the dependent and contain a foreign key to the principal. The entity type that the relationship targets will be the principal in the relationship.<br>將關係配置為必要：必要，而關係的另一端沒有導航屬性。配置的實體類型將作為從屬實體，並包含指向主體實體的外鍵。關係所針對的實體類型將成為關係中的主體實體。</td>
</tr>
<tr>
	<td>WithOptional()</td>
	<td>Configures the relationship to be many:optional without a navigation property on the other side of the relationship.<br>將關係配置為多：可選，而關係的另一端沒有導航屬性。</td>
</tr>
<tr>
	<td>Map()</td>
	<td>Allows advanced configuration related to how the entity is mapped to the database schema.<br>允許進行與實體如何映射到資料庫模式相關的進階配置。</td>
</tr>
<tr>
	<td>HasForeignKey()</td>
	<td>Configures the relationship to use foreign key property(s) that are exposed in the object model.<br>配置關係以使用物件模型中公開的外鍵屬性。</td>
</tr>
<tr>
	<td>WillCascadeOnDelete()</td>
	<td>Configures cascade delete to be on for the relationship.<br>配置關係的串聯刪除。</td>
</tr>

<tr>
	<td rowspan="1">StoredProcedures Configuration<br>儲存程序配置</td>
	<td><a href="https://learn.microsoft.com/en-us/dotnet/api/system.data.entity.modelconfiguration.entitytypeconfiguration-1.maptostoredprocedures">MapToStoredProcedures()</a></td>
	<td>Configures the entity type to use INSERT, UPDATE and DELETE stored procedures.<br>將實體類型配置為使用 INSERT、UPDATE 和 DELETE 儲存程序。</td>
</tr>
</tbody></table>

---

# Entity Mappings using Fluent API in EF 6 在 EF 6 中使用 Fluent API 進行實體映射

The Fluent API can be used to configure an entity to map it with database table(s), default schema, etc.
Fluent API 可用於配置實體以將其與資料庫表、預設模式等進行對應。

## Configure Default Schema 配置預設架構

First, let's configure a default schema for the tables in the database. However, you can change the schema while creating the individual tables. The following example sets the Admin schema as a default schema. All the database objects will be created under the Admin schema unless you specify a different schema explicitly.
首先，讓我們為資料庫中的表配置一個預設架構。不過，您可以在建立單一表格時變更該架構。以下範例將 Admin 架構設定為預設架構。除非您明確指定其他架構，否則所有資料庫物件都會在 Admin 架構下建立。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Configure default schema
        modelBuilder.HasDefaultSchema("Admin");
    }
}
```

## Map Entity to Table 將實體映射到表

Code-First will create the database tables with the name of `DbSet` properties in the context class, `Students` and `Standards` in this case. You can override this convention and give a different table name than the `DbSet` properties, as shown below.
Code-First 將在上下文類別中建立以 `DbSet` 屬性名稱命名的資料庫表，在本例中分別 `Students` 和 `Standards`。您可以覆寫此慣例，並使用與 `DbSet` 屬性不同的表名，如下所示。

```csharp
namespace CodeFirst_FluentAPI_Tutorials
{
    public class SchoolDBContext: DbContext
    {
        public SchoolDBContext(): base()
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Standard> Standards { get; set; }
        
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            //Configure default schema
            modelBuilder.HasDefaultSchema("Admin");
                    
            //Map entity to table
            modelBuilder.Entity<Student>().ToTable("StudentInfo");
            modelBuilder.Entity<Standard>().ToTable("StandardInfo","dbo");
        }
    }
}
```

As you can see in the above example, we start with the `Entity<TEntity>()` method. Most of the time, you have to ==start with the `Entity<TEntity>()` method== to configure it using Fluent API. We have used the `ToTable()` method to map the `Student` entity to the `StudentInfo` table and the `Standard` entity to the `StandardInfo` table. Notice that `StudentInfo` is in the Admin schema and the `StandardInfo` table is in the dbo schema because we have specified the dbo schema for the `StandardInfo` table.
如上例所示，我們從 `Entity<TEntity>()` 方法開始。大多數情況下，您必須 ==從 `Entity<TEntity>()` 方法開始==，才能使用 Fluent API 進行設定。我們使用 `ToTable()` 方法將 `Student` 實體對應到 `StudentInfo` 表，並將 `Standard` 實體對應到 `StandardInfo` 表。請注意， `StudentInfo` 位於 Admin 架構中，而 `StandardInfo` 表位於 dbo 架構中，因為我們已為 `StandardInfo` 表指定了 dbo 架構。

![[table-mapping-fluentapi.png|table mapping with fluent api Entity Framework code-first]]

## Map Entity to Multiple Tables 將實體對應到多個表

The following example shows how to map the `Student` entity to multiple tables in the database.
下面的範例顯示如何將 `Student` 實體對應到資料庫中的多個表。

```csharp
namespace CodeFirst_FluentAPI_Tutorials
{
    public class SchoolContext: DbContext
    {
        public SchoolDBContext(): base()
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Standard> Standards { get; set; }
        
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Student>().Map(m =>
            {
                m.Properties(p => new { p.StudentId, p.StudentName});
                m.ToTable("StudentInfo");
            }).Map(m => {
                m.Properties(p => new { p.StudentId, p.Height, p.Weight, p.Photo, p.DateOfBirth});
                m.ToTable("StudentInfoDetail");
            });

            modelBuilder.Entity<Standard>().ToTable("StandardInfo");
        }
    }
}
```

As you can see in the above example, we mapped some properties of the `Student` entity to the `StudentInfo` table and other properties to the `StudentInfoDetail` table using the `Map()` method. Thus, the `Student` entity will split into two tables, as shown below.
如上例所示，我們使用 `Map()` 方法將 `Student` 實體的某些屬性對應到 `StudentInfo` 表，將其他屬性對應到 `StudentInfoDetail` 表。因此， `Student` 實體將拆分為兩個表，如下所示。

![[multiple-table-mapping-fluentapi.png|multiple table mapping with fluent api Entity Framework code-first]]

The `Map()` method requires a delegate method parameter. You can pass an [Action delegate](http://www.tutorialsteacher.com/csharp/csharp-action-delegate) or a [lambda expression](http://www.tutorialsteacher.com/linq/linq-lambda-expression) in the `Map()` method, as shown below.
`Map()` 方法需要一個委託方法參數。您可以在 `Map()` 方法中傳遞 [Action 委託](http://www.tutorialsteacher.com/csharp/csharp-action-delegate) 或 [Lambda 表達式](http://www.tutorialsteacher.com/linq/linq-lambda-expression) ，如下所示。

```csharp
using System.Data.Entity.ModelConfiguration.Configuration;

namespace CodeFirst_FluentAPI_Tutorials
{
    public class SchoolDBContext: DbContext
    {
        public SchoolDBContext(): base()
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Standard> Standards { get; set; }
        
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Student>().Map(delegate(EntityMappingConfiguration<Student> studentConfig)
            {
                studentConfig.Properties(p => new { p.StudentId, p.StudentName });
                studentConfig.ToTable("StudentInfo");
            });

            Action<EntityMappingConfiguration<Student>> studentMapping = m =>
            {
                m.Properties(p => new { p.StudentId, p.Height, p.Weight, p.Photo, p.DateOfBirth });
                m.ToTable("StudentInfoDetail");
            };
            
            modelBuilder.Entity<Student>().Map(studentMapping);
            modelBuilder.Entity<Standard>().ToTable("StandardInfo");
        }
    }
}
```

---

# Property Mappings using Fluent API 使用 Fluent API 的屬性映射

The Fluent API can be used to configure properties of an entity to map it with a db column. Using Fluent API, you can change the corresponding column name, type, size, Null or NotNull, PrimaryKey, ForeignKey, concurrency column, etc.
Fluent API 可用於配置實體的屬性，以便將其對應到資料庫列。使用 Fluent API，您可以變更對應的列名、類型、大小、是否為 Null 或 NotNull、PrimaryKey、ForeignKey、並行列等。

We will configure the following entity classes.
我們將配置以下實體類別。

```csharp
public class Student
{
    public int StudentKey { get; set; }
    public string StudentName { get; set; }
    public DateTime DateOfBirth { get; set; }
    public byte[]  Photo { get; set; }
    public decimal Height { get; set; }
    public float Weight { get; set; }
        
    public Standard Standard { get; set; }
}
    
public class Standard
{
    public int StandardKey { get; set; }
    public string StandardName { get; set; }
    
    public ICollection<Student> Students { get; set; }
}
```

## Configure Primary Key and Composite Primary Key 配置主鍵和複合主鍵

Our domain classes above, do not follow the Code-First convention for primary key because they don't have Id or {Class Name} + "Id" property. So, you can configure a key property using the `HasKey()` method as shown below. Remember that `modelBuilder.Entity<TEntity>()` returns the `EntityTypeConfiguration` object.
上面的領域類別不遵循 Code-First 的主鍵慣例，因為它們沒有 Id 或 {Class Name} + "Id" 屬性。因此，您可以使用 `HasKey()` 方法來配置主鍵屬性，如下所示。請記住， `modelBuilder.Entity<TEntity>()` 回傳 `EntityTypeConfiguration` 物件。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Configure primary key
        modelBuilder.Entity<Student>().HasKey<int>(s => s.StudentKey);
        modelBuilder.Entity<Standard>().HasKey<int>(s => s.StandardKey);

        //Configure composite primary key
        modelBuilder.Entity<Student>().HasKey<int>(s => new { s.StudentKey, s.StudentName }); 
    }
}
```

## Configure Column Name, Type and Order 配置列名稱、類型和順序

The default Code-First convention creates a column for a property with the same name, order, and datatype. You can override this convention, as shown below.
預設的 Code-First 慣例會為具有相同名稱、順序和資料類型的屬性建立一個欄位。您可以覆蓋此慣例，如下所示。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Configure Column
        modelBuilder.Entity<Student>()
                    .Property(p => p.DateOfBirth)
                    .HasColumnName("DoB")
                    .HasColumnOrder(3)
                    .HasColumnType("datetime2");
    }
}
```

As you can see in the above example, the `Property()` method is used to configure a property of an entity. The `HasColumnName()` method is used to change the column name of the `DateOfBirth` property. Also, the `HasColumnOrder()` and `HasColumnType()` methods change the order and datatype of the corresponding column.
如上例所示， `Property()` 方法用於配置實體的屬性。 HasColumnName `HasColumnName()` 方法用來變更 `DateOfBirth` 屬性的列名。此外， `HasColumnOrder()` 和 `HasColumnType()` 方法用於變更對應列的順序和資料類型。

`modelBuilder.Entity<TEntity>().Property(expression)` allows you to use different methods to configure a particular property, as shown below.
`modelBuilder.Entity<TEntity>().Property(expression)` 允許您使用不同的方法來配置特定屬性，如下所示。

![[configure-property-1.png|configure property with fluent api Entity Framework code-first]]

## Configure Null or NotNull Column 配置 Null 或 NotNull 列

EF 6 API will create a NotNull column for a primitive data type property because primitive data type can not be null unless it is marked as nullable using the `?` sign or `Nullable<T>` type.
EF 6 API 將為原始資料型別屬性建立 NotNull 資料列，因為原始資料型別不能為空，除非使用 `?` 符號或 `Nullable<T>` 型別將其標記為可空。

Use `IsOptional()` method to create a nullable column for a property. In the same way, use `IsRequired()` method to create a NotNull column.
使用 `IsOptional()` 方法為屬性建立可空白列。同樣，使用 `IsRequired()` 方法建立 NotNull 欄位。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
            //Configure Null Column
        modelBuilder.Entity<Student>()
                .Property(p => p.Height)
                .IsOptional();
                        
            //Configure NotNull Column
            modelBuilder.Entity<Student>()
                .Property(p => p.Weight)
                .IsRequired();
    }
}
```

## Configure Column Size  配置列大小

Code-First will set the maximum size for a data type for a column. You can override this convention, as shown below.
Code-First 將設定列中每種資料類型的最大大小。您可以覆蓋此慣例，如下所示。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Set StudentName column size to 50
        modelBuilder.Entity<Student>()
                .Property(p => p.StudentName)
                .HasMaxLength(50);
                        
        //Set StudentName column size to 50 and change datatype to nchar 
        //IsFixedLength() changes datatype from nvarchar to nchar
        modelBuilder.Entity<Student>()
                .Property(p => p.StudentName)
                .HasMaxLength(50).IsFixedLength();
                        
        //Set size decimal(2,2)
            modelBuilder.Entity<Student>()
                .Property(p => p.Height)
                .HasPrecision(2, 2);
    }
}
```

As you can see in the above example, we used the `HasMaxLength()` method to set the size of a column. The `IsFixedLength()` method converts nvarchar to nchar type. In the same way, the `HasPrecision()` method changed the precision of the decimal column.
如上例所示，我們使用 `HasMaxLength()` 方法設定列的大小。 IsFixedLength `IsFixedLength()` 方法將 nvarchar 轉換為 nchar 類型。同樣， `HasPrecision()` 方法更改了小數列的精度。

## Configure Concurrency Column 配置並行列

You can configure a property as concurrency column using the `ConcurrencyToken()` method, as shown below.
您可以使用 `ConcurrencyToken()` 方法將屬性配置為並行列，如下所示。

```csharp
public class SchoolDBContext: DbContext
{
    public SchoolDBContext(): base()
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //Set StudentName as concurrency column
        modelBuilder.Entity<Student>()
                .Property(p => p.StudentName)
                .IsConcurrencyToken();
    }
}
```

As you can see in the above example, we set the `StudentName` column as concurrency column so that it will be included in the where clause in update and delete commands.
正如您在上面的範例中看到的，我們將 `StudentName` 列設定為並行列，以便它將包含在更新和刪除命令中的 where 子句中。

You can also use the `IsRowVersion()` method for `byte[]` type property to make it as a concurrency column.
您也可以對 `byte[]` 型別屬性使用 `IsRowVersion()` 方法，使其成為並行列。
