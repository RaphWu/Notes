---
aliases:
date: 2020-10-15
update:
author: Microsoft
language: C#
sourceurl: https://learn.microsoft.com/en-us/ef/ef6/fundamentals/relationships
tags:
  - CSharp
  - 資料庫
  - EF6
---

# Relationships, navigation properties, and foreign keys 關係、導航屬性和外鍵

This article gives an overview of how Entity Framework manages relationships between entities. It also gives some guidance on how to map and manipulate relationships.
本文概述了 Entity Framework 如何管理實體之間的關係。它還提供了一些關於如何映射和操作關係的指導。

# Relationships in EF  EF 中的關係

In relational databases, **relationships** (also called **associations**) between tables are defined through foreign keys. A **foreign key (FK)** is a column or combination of columns that is used to establish and enforce a link between the data in two tables. There are generally three types of relationships: one-to-one, one-to-many, and many-to-many. In a **one-to-many** relationship, the foreign key is defined on the table that represents the many end of the relationship. The **many-to-many** relationship involves defining a third table (called a **junction** or **join table**), whose primary key is composed of the foreign keys from both related tables. In a **one-to-one** relationship, the primary key acts additionally as a foreign key and there is no separate foreign key column for either table.
在關係資料庫中，表之間的**關係**（也稱為**關聯**）是透過外鍵定義的。 **外鍵 (FK)** 是用於建立和加強兩個表之間資料連結的一列或多列組合。關係通常有三種類型：一對一、一對多和多對多。在**一對多**關係中，外鍵定義在代表關係多端的表上。 **多對多**關係涉及定義第三個表（稱為**連接表**或**聯接表**），其主鍵由兩個相關表的外鍵組成。在**一對一**關係中，主鍵也充當外鍵，並且兩個表都沒有單獨的外鍵列。

The following image shows two tables that participate in one-to-many relationship. The `Course` table is the dependent table because it contains the `DepartmentID` column that links it to the `Department` table.
以下圖片顯示兩個參與一對多關係的表格。`Course` 表格是相依表格，因為它包含 `DepartmentID` 欄位，該欄位將其連接到 `Department` 表格。

![[Relationships, navigation properties, and foreign keys_01.png]]

In Entity Framework, an entity can be related to other entities through an **association** or **relationship**. Each relationship contains two ends that describe the **entity type** and the **multiplicity of the type** (one, zero-or-one, or many) for the two entities in that relationship. The relationship may be governed by a **referential constraint**, which describes which end in the relationship is a principal role and which is a dependent role.
在 Entity Framework 中，實體可以透過關聯或關係與其他實體相關聯。每個關係包含兩個端點，這些端點描述了實體類型以及該關係中兩個實體的類型多樣性（一個、零或一個、或多個）。關係可能受參考約束的管轄，該約束描述了關係中的哪一端是主角色，哪一端是相依角色。

**Navigation properties** provide a way to navigate an association between two entity types. Every object can have a navigation property for every relationship in which it participates. Navigation properties allow you to navigate and manage relationships in both directions, returning either a reference object (if the multiplicity is either one or zero-or-one) or a collection (if the multiplicity is many). You may also choose to have one-way navigation, in which case you define the navigation property on only one of the types that participates in the relationship and not on both.
導航屬性提供了一種方法來導航兩個實體類型之間的關聯。每個物件都可以對其參與的每個關係擁有一個導航屬性。導航屬性允許您雙向導航和管理關係，返回參考物件（如果複數性是「一個」或「零或一個」）或集合（如果複數性是多個）。您也可以選擇單向導航，這種情況下，您僅在參與關係的一種類型上定義導航屬性，而不是在兩種類型上都定義。

It is recommended to include properties in the model that map to foreign keys in the database. With foreign key properties included, you can create or change a relationship by modifying the foreign key value on a dependent object. This kind of association is called a **foreign key association**. Using foreign keys is even more essential when working with disconnected entities. Note, that when working with 1-to-1 or 1-to-0..1 relationships, there is no separate foreign key column, the primary key property acts as the foreign key and is always included in the model.
建議在模型中包含對應資料庫外鍵的屬性。包含外鍵屬性後，您可以通过修改依賴物件上的外鍵值來建立或更改關係。這種關聯稱為**外鍵關聯**。在處理斷開連接的實體時，使用外鍵更加重要。注意，在處理 1 對 1 或 1 對 0..1 關係時，沒有獨立的外鍵列，主鍵屬性作為外鍵並且總是包含在模型中。

When foreign key columns are not included in the model, the association information is managed as an independent object. Relationships are tracked through object references instead of foreign key properties. This type of association is called an **independent association**. The most common way to modify an independent association is to modify the navigation properties that are generated for each entity that participates in the association.
當外鍵欄位未包含在模型中時，關聯資訊會被管理為獨立物件。關係透過物件參考來追蹤，而不是透過外鍵屬性。這種類型的關聯稱為**獨立關聯**。修改獨立關聯最常見的方法是修改參與關聯的每個實體所產生的導航屬性。

You can choose to use one or both types of associations in your model. However, ==if you have a pure many-to-many relationship that is connected by a join table that contains only foreign keys, the EF will use an **independent association** to manage such many-to-many relationship.   ==
您可以選擇在您的模型中使用一種或兩種類型的關聯。但是，==如果您有一個純粹的 Many-to-Many 關聯，它透過僅包含外鍵的連接表連接，EF 將使用**獨立關聯**來管理這種 Many-to-Many 關聯。==

The following image shows a conceptual model that was created with the Entity Framework Designer. The model contains two entities that participate in one-to-many relationship. Both entities have navigation properties. `Course` is the dependent entity and has the `DepartmentID` foreign key property defined.
下列圖片顯示了使用 Entity Framework Designer 建立的概念模型。該模型包含兩個參與一對多關聯的實體。兩個實體都有導航屬性。`Course` 是相依實體，並定義了 `DepartmentID` 外鍵屬性。

![[Relationships, navigation properties, and foreign keys_02.png]]

The following code snippet shows the same model that was created with Code First.
下列程式碼片段顯示了使用 Code First 建立的同樣模型。

```csharp
public class Course
{
  public int CourseID { get; set; }
  public string Title { get; set; }
  public int Credits { get; set; }
  public int DepartmentID { get; set; }
  public virtual Department Department { get; set; }
}

public class Department
{
   public Department()
   {
     this.Courses = new HashSet<Course>();
   }  
   public int DepartmentID { get; set; }
   public string Name { get; set; }
   public decimal Budget { get; set; }
   public DateTime StartDate { get; set; }
   public int? Administrator {get ; set; }
   public virtual ICollection<Course> Courses { get; set; }
}
```

# Configuring or mapping relationships 設定或對應關係

The rest of this page covers how to access and manipulate data using relationships. For information on setting up relationships in your model, see the following pages.
本頁的剩餘部分說明了如何使用關係來存取和操作資料。有關在您的模型中設定關係的資訊，請參閱以下頁面。

- To configure relationships in Code First, see [Data Annotations](https://learn.microsoft.com/en-us/ef/ef6/modeling/code-first/data-annotations) and [Fluent API – Relationships](https://learn.microsoft.com/en-us/ef/ef6/modeling/code-first/fluent/relationships).
    若要在 Code First 中設定關係，請參閱資料註解與流暢 API – 關係。
- To configure relationships using the Entity Framework Designer, see [Relationships with the EF Designer](https://learn.microsoft.com/en-us/ef/ef6/modeling/designer/relationships).
    若要使用 Entity Framework 設計師設定關係，請參閱使用 EF 設計師的關係。

# Creating and modifying relationships 建立與修改關係

In a _foreign key association_, when you change the relationship, the state of a dependent object with an `EntityState.Unchanged` state changes to `EntityState.Modified`. In an independent relationship, changing the relationship does not update the state of the dependent object.
在外鍵關聯中，當您變更關係時，具有 `EntityState.Unchanged` 狀態的相依物件狀態會變更為 `EntityState.Modified` 。在獨立關係中，變更關係不會更新相依物件的狀態。

The following examples show how to use the foreign key properties and navigation properties to associate the related objects. With foreign key associations, you can use either method to change, create, or modify relationships. With independent associations, you cannot use the foreign key property.
以下範例說明了如何使用外鍵屬性和導航屬性來關聯相關物件。在外鍵關聯中，您可以使用任一方法來變更、建立或修改關係。在獨立關聯中，您無法使用外鍵屬性。

- By assigning a new value to a foreign key property, as in the following example.
  透過將新值指派給外鍵屬性，例如以下範例。

```csharp
course.DepartmentID = newCourse.DepartmentID;
```

- The following code removes a relationship by setting the foreign key to `null`. Note, that the foreign key property must be nullable.
  以下程式碼透過將外鍵設為 null 來移除關係。請注意，外鍵屬性必須可為 null。

```csharp
course.DepartmentID = null;
```

> Note  注意
>
> If the reference is in the added state (in this example, the course object), the reference navigation property will not be synchronized with the key values of a new object until `SaveChanges` is called. Synchronization does not occur because the object context does not contain permanent keys for added objects until they are saved. If you must have new objects fully synchronized as soon as you set the relationship, use one of the following methods.*
> 如果參考處於新增狀態（在本例中為課程物件），參考導航屬性將不會與新物件的鍵值同步，直到呼叫 `SaveChanges`。同步不會發生，因為物件上下文在物件保存之前不包含新增物件的永久鍵。如果您必須在設置關係後立即使新物件完全同步，請使用以下其中一種方法。*

- By assigning a new object to a navigation property. The following code creates a relationship between a course and a `department`. If the objects are attached to the context, the `course` is also added to the `department.Courses` collection, and the corresponding foreign key property on the `course` object is set to the key property value of the `department`.
  透過將新物件指派給導航屬性。以下程式碼在 `course` 和 `department` 之間建立關係。如果物件已附加到上下文， `course` 也會被添加到 `department.Courses` 集合，並且在 `course` 物件上的對應外鍵屬性會設為 `department` 的鍵屬性值。

```csharp
course.Department = department;
```

- To delete the relationship, set the navigation property to `null`. If you are working with Entity Framework that is based on .NET 4.0, then the related end needs to be loaded before you set it to null. For example:
  要刪除關係，請將導航屬性設為 `null` 。如果您正在使用基於 .NET 4.0 的 Entity Framework，則需要在設為 null 之前載入相關端。例如：

```csharp
context.Entry(course).Reference(c => c.Department).Load();
course.Department = null;
```

Starting with Entity Framework 5.0, that is based on .NET 4.5, you can set the relationship to null without loading the related end. You can also set the current value to null using the following method.
從 Entity Framework 5.0 開始，基於 .NET 4.5，您可以不載入相關端就將關係設為 null。您也可以使用以下方法將當前值設為 null。

```csharp
context.Entry(course).Reference(c => c.Department).CurrentValue = null;
```

- By deleting or adding an object in an entity collection. For example, you can add an object of type `Course` to the `department.Courses` collection. This operation creates a relationship between a particular `course` and a particular `department`. If the objects are attached to the context, the department reference and the foreign key property on the `course` object will be set to the appropriate `department`.
  透過在實體集合中刪除或添加物件。例如，您可以將類型為 `Course` 的物件添加到 `department.Courses` 集合中。此操作會在特定的課程和特定的 `department` 之間建立關係。如果物件已附加到上下文，部門參考和課程物件上的外鍵屬性將會設為適當的 `department` 。

```csharp
department.Courses.Add(newCourse);
```

- By using the `ChangeRelationshipState` method to change the state of the specified relationship between two entity objects. This method is most commonly used when working with N-Tier applications and an _independent association_ (it cannot be used with a foreign key association). Also, to use this method you must drop down to `ObjectContext`, as shown in the example below.
  透過使用 `ChangeRelationshipState` 方法來改變兩個實體物件之間的指定關係狀態。此方法最常在處理 N-Tier 應用程式和獨立關聯時使用（無法與外鍵關聯一起使用）。此外，要使用此方法，您必須降到 `ObjectContext` ，如下面的範例所示。

  In the following example, there is a many-to-many relationship between Instructors and Courses. Calling the `ChangeRelationshipState` method and passing the `EntityState.Added` parameter, lets the `SchoolContext` know that a relationship has been added between the two objects:
  在以下範例中，Instructors 和 Courses 之間存在多對多的關係。呼叫 `ChangeRelationshipState` 方法並傳遞 `EntityState.Added` 參數，讓 `SchoolContext` 知道這兩個物件之間已經建立了關係：

```csharp
((IObjectContextAdapter)context).ObjectContext.ObjectStateManager.
	ChangeRelationshipState(course, instructor, c => c.Instructor, EntityState.Added);
```

Note that if you are updating (not just adding) a relationship, you must delete the old relationship after adding the new one:
注意，如果您正在更新（而不仅仅是添加）關係，您必須在添加新關係後刪除舊關係：

```csharp
((IObjectContextAdapter)context).ObjectContext.ObjectStateManager.
	ChangeRelationshipState(course, oldInstructor, c => c.Instructor, EntityState.Deleted);
```

# Synchronizing the changes between the foreign keys and navigation properties 同步外鍵和導航屬性之間的變更

When you change the relationship of the objects attached to the context by using one of the methods described above, Entity Framework needs to keep foreign keys, references, and collections in sync. Entity Framework automatically manages this synchronization (also known as **relationship fix-up**) for the POCO entities with proxies. For more information, see [Working with Proxies](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/proxies).
當您使用上述方法之一來更改附加到上下文的物件關係時，Entity Framework 需要保持外鍵、參考和集合的同步。Entity Framework 會自動管理這個同步（也稱為**關係修復**）對於具有代理的 POCO 實體。如需更多資訊，請參閱與代理一起使用。

If you are using POCO entities without proxies, you must make sure that the `DetectChanges` method is called to synchronize the related objects in the context. Note that the following APIs automatically trigger a `DetectChanges` call.
如果您使用沒有代理的 POCO 實體，您必須確保呼叫 `DetectChanges` 方法以同步上下文中相關的物件。請注意，以下 API 會自動觸發 `DetectChanges` 呼叫。

- `DbSet.Add`
- `DbSet.AddRange`
- `DbSet.Remove`
- `DbSet.RemoveRange`
- `DbSet.Find`
- `DbSet.Local`
- `DbContext.SaveChanges`
- `DbSet.Attach`
- `DbContext.GetValidationErrors`
- `DbContext.Entry`
- `DbChangeTracker.Entries`
- Executing a LINQ query against a `DbSet`
    執行針對 `DbSet` 的 LINQ 查詢

# Loading related objects  載入相關物件

In Entity Framework you commonly use navigation properties to load entities that are related to the returned entity by the defined association. For more information, see [Loading Related Objects](https://learn.microsoft.com/en-us/ef/ef6/querying/related-data).
在 Entity Framework 中，您通常使用導航屬性來載入與返回的實體相關的實體，這些相關性是透過定義的關聯性建立的。如需更多資訊，請參閱載入相關物件。

> Note  注意
>
> In a foreign key association, when you load a related end of a dependent object, the related object will be loaded based on the foreign key value of the dependent that is currently in memory:
> 在一個外鍵關聯中，當您載入相依物件的一個相關端時，相關物件將會根據目前記憶體中相依實體的外鍵值來載入：

```csharp
// Get the course where currently DepartmentID = 2.
Course course = context.Courses.First(c => c.DepartmentID == 2);

// Use DepartmentID foreign key property
// to change the association.
course.DepartmentID = 3;

// Load the related Department where DepartmentID = 3
context.Entry(course).Reference(c => c.Department).Load();
```

In an independent association, the related end of a dependent object is queried based on the foreign key value that is currently in the database. However, if the relationship was modified, and the reference property on the dependent object points to a different principal object that is loaded in the object context, Entity Framework will try to create a relationship as it is defined on the client.
在一個獨立關聯中，會根據資料庫中當前的外鍵值來查詢相依物件的相關端。但是，如果關聯被修改了，且相依物件的參考屬性指向了在物件上下文中已載入的不同主體物件，Entity Framework 將嘗試根據客戶端上定義的方式來建立關聯。

# Managing concurrency  管理並發性

In both foreign key and independent associations, concurrency checks are based on the entity keys and other entity properties that are defined in the model. When using the EF Designer to create a model, set the `ConcurrencyMode` attribute to `fixed` to specify that the property should be checked for concurrency. When using Code First to define a model, use the `ConcurrencyCheck` annotation on properties that you want to be checked for concurrency. When working with Code First you can also use the `TimeStamp` annotation to specify that the property should be checked for concurrency. You can have only one `timestamp` property in a given class. Code First maps this property to a non-nullable field in the database.
在關聯鍵和獨立關聯中，並發性檢查都是基於模型中定義的實體鍵和其他實體屬性。當使用 EF 設計器來建立模型時，將 `ConcurrencyMode` 屬性設為 `fixed` 以指定該屬性應該進行並發性檢查。當使用 Code First 來定義模型時，在您想要進行並發性檢查的屬性上使用 `ConcurrencyCheck` 標註。在 Code First 中，您也可以使用 `TimeStamp` 標註來指定該屬性應該進行並發性檢查。在特定類別中只能有一個 `timestamp` 屬性。Code First 會將此屬性對應到資料庫中的非 Null 資料欄位。

We recommend that you always use the foreign key association when working with entities that participate in concurrency checking and resolution.
我們建議您==在處理參與競爭檢查和解析的實體時，總是使用外鍵關聯。==

For more information, see [Handling Concurrency Conflicts](https://learn.microsoft.com/en-us/ef/ef6/saving/concurrency).
更多資訊，請參閱處理競爭狀態衝突。

# Working with overlapping Keys

處理重疊的鍵

Overlapping keys are composite keys where some properties in the key are also part of another key in the entity. You cannot have an overlapping key in an independent association. To change a foreign key association that includes overlapping keys, we recommend that you modify the foreign key values instead of using the object references.
重疊的鍵是指組合鍵，其中鍵中的某些屬性也是實體中另一個鍵的一部分。您不能在獨立關聯中使用重疊鍵。要更改包含重疊鍵的外鍵關聯，我們建議您修改外鍵值，而不是使用物件參考。
