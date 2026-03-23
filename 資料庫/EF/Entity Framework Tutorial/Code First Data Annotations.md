---
aliases:
date: 2020-10-15
update:
author: Microsoft
language: C#
sourceurl: https://learn.microsoft.com/en-us/ef/ef6/modeling/code-first/data-annotations
tags:
  - CSharp
  - 資料庫
  - EF6
  - EF_Code_First
---

# Code First Data Annotations Code First 資料註釋

> Note  筆記
>
> **EF4.1 Onwards Only** - The features, APIs, etc. discussed in this page were introduced in Entity Framework 4.1. If you are using an earlier version, some or all of this information does not apply.
> **僅限 EF4.1 及更高版本** - 本頁中討論的功能、API 等是在 Entity Framework 4.1 中引入的。如果你使用的是早期版本，則部分或全部資訊不適用。

The content on this page is adapted from an article originally written by Julie Lerman (<http://thedatafarm.com>).
本頁內容改編自 Julie Lerman 撰寫的文章（<http://thedatafarm.com> ）。

Entity Framework Code First allows you to use your own domain classes to represent the model that EF relies on to perform querying, change tracking, and updating functions. Code First leverages a programming pattern referred to as '**convention over configuration**.' Code First will assume that your classes follow the conventions of Entity Framework, and in that case, will automatically work out how to perform its job. However, if your classes do not follow those conventions, you have the ability to add configurations to your classes to provide EF with the requisite information.
Entity Framework Code First 讓您可以使用自己的網域類別來表示 EF 所依賴的模型，以便執行查詢、變更追蹤和更新功能。 Code First 利用一種稱為「**慣例優於配置**」的程式模式。 Code First 會假定您的類別遵循 Entity Framework 的慣例，在這種情況下，它將自動決定如何執行其工作。但是，如果您的類別不遵循這些慣例，您可以在類別中新增配置，以便為 EF 提供必要的資訊。

Code First gives you two ways to add these configurations to your classes. One is using simple attributes called **DataAnnotations**, and the second is using Code First’s **Fluent API**, which provides you with a way to describe configurations imperatively, in code.
Code First 提供了兩種將這些配置新增至類別的方法。一種是使用名為 **DataAnnotations** 的簡單屬性，另一種是使用 Code First 的 **Fluent API**，它提供了一種在程式碼中以命令式方式描述配置的方法。

This article will focus on using DataAnnotations (in the System.ComponentModel.DataAnnotations namespace) to configure your classes – highlighting the most commonly needed configurations. DataAnnotations are also understood by a number of .NET applications, such as ASP.NET MVC which allows these applications to leverage the same annotations for client-side validations.
本文將重點放在如何使用 DataAnnotations（位於 `System.ComponentModel.DataAnnotations` 命名空間中）來配置類別，並重點介紹最常用的配置。許多 .NET 應用程式（例如 ASP.NET MVC）也能理解 DataAnnotations，這使得這些應用程式能夠利用相同的註解進行用戶端驗證。

## The model  模型

I’ll demonstrate Code First DataAnnotations with a simple pair of classes: `Blog` and `Post`.
我將使用一對簡單的類別來示範 Code First DataAnnotations：`Blog` 和 `Post`。

```csharp
public class Blog
{
	public int Id { get; set; }
	public string Title { get; set; }
	public string BloggerName { get; set;}
	public virtual ICollection<Post> Posts { get; set; }
}

public class Post
{
	public int Id { get; set; }
	public string Title { get; set; }
	public DateTime DateCreated { get; set; }
	public string Content { get; set; }
	public int BlogId { get; set; }
	public ICollection<Comment> Comments { get; set; }
}
```

As they are, the `Blog` and `Post` classes conveniently follow code first convention and require no tweaks to enable EF compatibility. However, you can also use the annotations to provide more information to EF about the classes and the database to which they map.
事實上，`Blog` 和 `Post` 類別遵循了程式碼優先的慣例，無需進行任何調整即可實現 EF 相容性。不過，你也可以使用註解向 EF 提供有關類別及其所映射到的資料庫的更多資訊。

## Key  鍵

Entity Framework relies on every entity having a key value that is used for entity tracking. One convention of Code First is implicit key properties; Code First will look for a property named “`Id`”, or a combination of class name and “Id”, such as “BlogId”. This property will map to a primary key column in the database.
Entity Framework 依賴每個實體都有一個用於實體追蹤的鍵值。 Code First 的一個慣例是隱式鍵屬性；Code First 將尋找名為「`Id`」的屬性，或類別名稱和「Id」的組合，例如「BlogId」。此屬性將會對應到資料庫中的主鍵列。

The `Blog` and `Post` classes both follow this convention. What if they didn’t? What if Blog used the name `PrimaryTrackingKey` instead, or even `foo`? If code first does not find a property that matches this convention it will throw an exception because of Entity Framework’s requirement that you must have a key property. You can use the key annotation to specify which property is to be used as the EntityKey.
`Blog` 和 `Post` 類別都遵循此慣例。如果不遵循呢？如果 Blog 改用 `PrimaryTrackingKey` 或 `foo` 名稱會怎麼樣？如果 Code First 找不到符合此慣例的屬性，則會引發異常，因為 Entity Framework 要求必須具有鍵屬性。您可以使用鍵註解來指定要用作 EntityKey 的屬性。

```csharp
public class Blog
{
	[Key]
	public int PrimaryTrackingKey { get; set; }
	public string Title { get; set; }
	public string BloggerName { get; set;}
	public virtual ICollection<Post> Posts { get; set; }
}
```

If you are using code first’s database generation feature, the Blog table will have a primary key column named PrimaryTrackingKey, which is also defined as Identity by default.
如果您使用程式碼優先的資料庫產生功能，Blog 表將有一個名為 `PrimaryTrackingKey` 的主鍵列，該列預設也定義為 `Identity`。

![[jj591583-figure01.png|Blog table with primary key]]

### Composite keys  複合鍵

Entity Framework supports composite keys - primary keys that consist of more than one property. For example, you could have a `Passport` class whose primary key is a combination of `PassportNumber` and `IssuingCountry`.
Entity Framework 支援複合鍵 - 由多個屬性組成的主鍵。例如，您可以有一個 `Passport` 類，其主鍵是 `PassportNumber` 和 `IssuingCountry` 的組合。

```csharp
public class Passport
{
	[Key]
	public int PassportNumber { get; set; }
	[Key]
	public string IssuingCountry { get; set; }
	public DateTime Issued { get; set; }
	public DateTime Expires { get; set; }
}
```

Attempting to use the above class in your EF model would result in an `InvalidOperationException`:
嘗試在您的 EF 模型中使用上述類別將導致 `InvalidOperationException` ：

Unable to determine composite primary key ordering for type '`Passport`'. Use the `ColumnAttribute` or the `HasKey` method to specify an order for composite primary keys.
無法確定「`Passport`」類型的複合主鍵順序。請使用 `ColumnAttribute` 或 `HasKey` 方法指定複合主鍵的順序。_

==In order to use composite keys, Entity Framework requires you to define an order for the key properties.== You can do this by using the `Column` annotation to specify an order.
==為了使用組合鍵，Entity Framework 要求您定義鍵屬性的順序。==您可以使用 `Column` 註釋來指定順序。

> Note  筆記
>
> The order value is relative (rather than index based) so any values can be used. For example, 100 and 200 would be acceptable in place of 1 and 2.
> 順序值是相對的（而非基於索引），因此可以使用任何值。例如，可以用 100 和 200 代替 1 和 2。

```csharp
public class Passport
{
	[Key]
	[Column(Order=1)]
	public int PassportNumber { get; set; }
	[Key]
	[Column(Order = 2)]
	public string IssuingCountry { get; set; }
	public DateTime Issued { get; set; }
	public DateTime Expires { get; set; }
}
```

If you have entities with composite foreign keys, then you must specify the same column ordering that you used for the corresponding primary key properties.
如果您有具有複合外鍵的實體，那麼您必須指定與對應主鍵屬性相同的列順序。

Only the relative ordering within the foreign key properties needs to be the same, the exact values assigned to **Order** do not need to match. For example, in the following class, 3 and 4 could be used in place of 1 and 2.
外鍵屬性中的相對順序只需相同，賦給 **Order** 的確切值無需匹配。例如，在下面的類別中，可以用 3 和 4 代替 1 和 2。

```csharp
public class PassportStamp
{
	[Key]
	public int StampId { get; set; }
	public DateTime Stamped { get; set; }
	public string StampingCountry { get; set; }

	[ForeignKey("Passport")]
	[Column(Order = 1)]
	public int PassportNumber { get; set; }

	[ForeignKey("Passport")]
	[Column(Order = 2)]
	public string IssuingCountry { get; set; }

	public Passport Passport { get; set; }
}
```

## Required  必需的

The `Required` annotation tells EF that a particular property is required.
`Required` 註解告訴 EF 需要某個特定屬性。

Adding `Required` to the `Title` property will force EF (and MVC) to ensure that the property has data in it.
在 `Title` 屬性中新增 `Required` 將強制 EF（和 MVC）確保該屬性中包含資料。

```csharp
[Required]
public string Title { get; set; }
```

With no additional code or markup changes in the application, an MVC application will perform client side validation, even dynamically building a message using the property and annotation names.
無需在應用程式中進行任何額外的程式碼或標記更改，MVC 應用程式將執行客戶端驗證，甚至使用屬性和註釋名稱動態建置訊息。

![[jj591583-figure02.png]]

The `Required` attribute will also affect the generated database by making the mapped property non-nullable. Notice that the `Title` field has changed to “not null”.
`Required` 特性也會影響產生的資料庫，因為它會使映射的屬性不可為空。請注意，`Title` 欄位已變更為“非空”。

> Note  筆記
>
> In some cases it may not be possible for the column in the database to be non-nullable even though the property is required. For example, when using a TPH inheritance strategy data for multiple types is stored in a single table. If a derived type includes a required property the column cannot be made non-nullable since not all types in the hierarchy will have this property.
> 在某些情況下，即使屬性是必需的，資料庫中的列也可能無法設定為不可為空。例如，當使用 TPH 繼承策略時，多種類型的資料會儲存在單一表中。如果衍生型別包含必要屬性，則無法將該資料列設為不可為空，因為並非層次結構中的所有型別都具有此屬性。

![[jj591583-figure03.png]]

## MaxLength and MinLength  

The `MaxLength` and `MinLength` attributes allow you to specify additional property validations, just as you did with `Required`.
`MaxLength` 和 `MinLength` 屬性可讓您指定額外的屬性驗證，就像您對 `Required` 所做的那樣。

Here is the `BloggerName` with length requirements. The example also demonstrates how to combine attributes.
以下是 `BloggerName` 的長度要求。此範例還示範如何組合屬性。

```csharp
[MaxLength(10),MinLength(5)]
public string BloggerName { get; set; }
```

The `MaxLength` annotation will impact the database by setting the property’s length to 10.
`MaxLength` 註解將透過將屬性的長度設為 10 來影響資料庫。

![[jj591583-figure04.png]]

MVC client-side annotation and EF 4.1 server-side annotation will both honor this validation, again dynamically building an error message: “The field BloggerName must be a string or array type with a maximum length of '10'.” That message is a little long. Many annotations let you specify an error message with the ErrorMessage attribute.
MVC 用戶端註解和 EF 4.1 伺服器端註解都會遵循此驗證，並再次動態產生錯誤訊息：「BloggerName 欄位必須是字串或陣列類型，且最大長度為『10』。」這則訊息有點長。許多註解允許使用 ErrorMessage 屬性指定錯誤訊息。

```csharp
[MaxLength(10, ErrorMessage="BloggerName must be 10 characters or less"),MinLength(5)]
public string BloggerName { get; set; }
```

You can also specify ErrorMessage in the Required annotation.
您也可以在 Required 註解中指定 ErrorMessage。

![[jj591583-figure05.png]]

## NotMapped  未映射

Code first convention dictates that every property that is of a supported data type is represented in the database. But this isn’t always the case in your applications. For example you might have a property in the `Blog` class that creates a code based on the `Title` and `BloggerName` fields. That property can be created dynamically and does not need to be stored. You can mark any properties that do not map to the database with the `NotMapped` annotation such as this `BlogCode` property.
Code First 慣例要求所有受支援資料類型的屬性都應在資料庫中表示。但您的應用程式並非總是如此。例如，您可能在 `Blog` 類別中有一個屬性，它會根據 `Title` 和 `BloggerName` 欄位建立程式碼。該屬性可以動態創建，無需儲存。您可以使用 `NotMapped` 註釋標記任何未對應到資料庫的屬性，例如此 `BlogCode` 屬性。

```csharp
[NotMapped]
public string BlogCode
{
	get
	{
		return Title.Substring(0, 1) + ":" + BloggerName.Substring(0, 1);
	}
}
```

## ComplexType  複雜類型

It’s not uncommon to describe your domain entities across a set of classes and then layer those classes to describe a complete entity. For example, you may add a class called `BlogDetails` to your model.
用一組類別來描述領域實體，然後將這些類別分層來描述一個完整的實體，這種做法並不罕見。例如，你可以在模型中新增一個名為 `BlogDetails` 的類別。

```csharp
public class BlogDetails
{
	public DateTime? DateCreated { get; set; }

	[MaxLength(250)]
	public string Description { get; set; }
}
```

Notice that `BlogDetails` does not have any type of key property. In domain driven design, `BlogDetails` is referred to as a **value object**. Entity Framework refers to value objects as complex types.  **Complex types cannot be tracked on their own.**
請注意， `BlogDetails` 沒有任何類型的鍵屬性。在領域驅動設計中， `BlogDetails` 稱為**值物件**。 Entity Framework 將值物件稱為複雜型別。**複雜類型無法單獨追蹤。**

However as a property in the `Blog` class, `BlogDetails` will be tracked as part of a `Blog` object. In order for code first to recognize this, you must mark the `BlogDetails` class as a `ComplexType`.
但是，作為 `Blog` 類別中的一個屬性， `BlogDetails` 將作為 `Blog` 物件的一部分進行追蹤。為了讓 Code First 能夠識別這一點，必須將 `BlogDetails` 類別標記為 `ComplexType` 。

```csharp
[ComplexType]
public class BlogDetails
{
	public DateTime? DateCreated { get; set; }

	[MaxLength(250)]
	public string Description { get; set; }
}
```

Now you can add a property in the `Blog` class to represent the `BlogDetails` for that blog.
現在您可以在 `Blog` 類別中新增一個屬性來表示該部落格的 `BlogDetails` 。

```csharp
public BlogDetails BlogDetail { get; set; }
```

In the database, the `Blog` table will contain all of the properties of the blog including the properties contained in its `BlogDetail` property. By default, each one is preceded with the name of the complex type, "BlogDetail".
在資料庫中， `Blog` 表將包含部落格的所有屬性，包括其 `BlogDetail` 屬性中包含的屬性。預設情況下，每個屬性前面都帶有複雜類型的名稱“BlogDetail”。

![[jj591583-figure06.png]]

## ConcurrencyCheck  並行檢查

The `ConcurrencyCheck` annotation allows you to flag one or more properties to be used for concurrency checking in the database when a user edits or deletes an entity. If you've been working with the EF Designer, this aligns with setting a property's `ConcurrencyMode` to `Fixed`.
`ConcurrencyCheck` 註釋允許你標記一個或多個屬性，以便在使用者編輯或刪除實體時用於資料庫中的並行檢查。如果你一直在使用 EF 設計器，這與將屬性的 `ConcurrencyMode` 設定為 `Fixed` 一致。

Let’s see how `ConcurrencyCheck` works by adding it to the `BloggerName` property.
讓我們透過將 `ConcurrencyCheck` 新增至 `BloggerName` 屬性來了解它的工作原理。

```csharp
[ConcurrencyCheck, MaxLength(10, ErrorMessage="BloggerName must be 10 characters or less"),MinLength(5)]
public string BloggerName { get; set; }
```

When `SaveChanges` is called, because of the `ConcurrencyCheck` annotation on the `BloggerName` field, the original value of that property will be used in the update. The command will attempt to locate the correct row by filtering not only on the key value but also on the original value of `BloggerName`.  Here are the critical parts of the `UPDATE` command sent to the database, where you can see the command will update the row that has a `PrimaryTrackingKey` is 1 and a `BloggerName` of “Julie” which was the original value when that blog was retrieved from the database.
呼叫 `SaveChanges` 時，由於 `BloggerName` 欄位帶有 `ConcurrencyCheck` 註釋，因此更新操作將使用其屬性的原始值。該命令將嘗試透過篩選鍵值和 `BloggerName` 的原始值來定位正確的行。以下是發送到資料庫的 `UPDATE` 命令的關鍵部分，您可以看到該命令將更新 `PrimaryTrackingKey` 為 1 且 `BloggerName` 為「Julie」的行（該值是從資料庫檢索該部落格時的原始值）。

```SQL
where (([PrimaryTrackingKey] = @4) and ([BloggerName] = @5))
@4=1,@5=N'Julie'
```

If someone has changed the blogger name for that blog in the meantime, this update will fail and you’ll get a **DbUpdateConcurrencyException** that you'll need to handle.
如果在此期間有人更改了該部落格的部落客姓名，則此更新將失敗，並且您將收到需要處理的 **DbUpdateConcurrencyException** 。

## TimeStamp  時間戳

It's more common to use rowversion or timestamp fields for concurrency checking. But rather than using the `ConcurrencyCheck` annotation, you can use the more specific `TimeStamp` annotation as long as the type of the property is byte array. Code first will treat `Timestamp` properties the same as `ConcurrencyCheck` properties, but it will also ensure that the database field that code first generates is non-nullable. You can only have one `timestamp` property in a given class.
更常見的是使用 rowversion 或 timestamp 欄位進行並行檢查。但是，除了使用 `ConcurrencyCheck` 註釋之外，您還可以使用更具體的 `TimeStamp` 註釋，只要屬性類型為位元組數組即可。 Code First 會將 `Timestamp` 屬性與 `ConcurrencyCheck` 屬性相同，但它也會確保 Code First 產生的資料庫欄位不可為空。每個類別中只能有一個 `timestamp` 屬性。

Adding the following property to the `Blog` class:
在 `Blog` 類別中新增以下屬性：

```csharp
[Timestamp]
public Byte[] TimeStamp { get; set; }
```

results in code first creating a non-nullable timestamp column in the database table.
導致程式碼首先在資料庫表中建立一個不可空的時間戳記列。

![[jj591583-figure07.png]]

## Table and Column  表和列

If you are letting Code First create the database, you may want to change the name of the tables and columns it is creating. You can also use Code First with an existing database. But it's not always the case that the names of the classes and properties in your domain match the names of the tables and columns in your database.
如果您使用 Code First 建立資料庫，則可能需要變更其建立的資料表和資料列的名稱。您也可以將 Code First 與現有資料庫一起使用。但是，網域中的類別和屬性的名稱並不總是與資料庫中的表格和列的名稱相符。

My class is named `Blog` and by convention, code first presumes this will map to a table named `Blogs`. If that's not the case you can specify the name of the table with the `Table` attribute. Here for example, the annotation is specifying that the table name is **InternalBlogs**.
我的類別名為 `Blog` ，按照慣例，程式碼首先假定它將映射到名為 `Blogs` 的表。如果不是這種情況，您可以使用 `Table` 屬性指定表的名稱。例如，此處的註解指定表名為 **InternalBlogs** 。

```csharp
[Table("InternalBlogs")]
public class Blog
```

The `Column` annotation is a more adept in specifying the attributes of a mapped column. You can stipulate a name, data type or even the order in which a column appears in the table. Here is an example of the `Column` attribute.
`Column` 註釋更擅長指定對映列的屬性。您可以指定列的名稱、資料類型，甚至列在表格中的顯示順序。以下是 `Column` 屬性的範例。

```csharp
[Column("BlogDescription", TypeName="ntext")]
public String Description {get;set;}
```

Don’t confuse Column’s `TypeName` attribute with the DataType DataAnnotation. DataType is an annotation used for the UI and is ignored by Code First.
不要將 Column 的 `TypeName` 屬性與 DataType DataAnnotation 混淆。 DataType 是用於 UI 的註釋，會被 Code First 忽略。

Here is the table after it’s been regenerated. The table name has changed to **InternalBlogs** and `Description` column from the complex type is now `BlogDescription`. Because the name was specified in the annotation, code first will not use the convention of starting the column name with the name of the complex type.
這是重新生成後的表。表名已更改為 **InternalBlogs** ，並且 `Description` 列從複雜類型變為 `BlogDescription` 。由於名稱是在註釋中指定的，因此 Code First 不會使用以複雜類型名稱開頭的列名慣例。

![[jj591583-figure08.png]]

## DatabaseGenerated  資料庫生成

An important database features is the ability to have computed properties. If you're mapping your Code First classes to tables that contain computed columns, you don't want Entity Framework to try to update those columns. But you do want EF to return those values from the database after you've inserted or updated data. You can use the `DatabaseGenerated` annotation to flag those properties in your class along with the `Computed` enum. Other enums are `None` and `Identity`.
資料庫的一個重要特性是能夠擁有計算屬性。如果您將 Code First 類別對應到包含計算列的表，您不希望 EF 嘗試更新這些列。但您確實希望 EF 在插入或更新資料後從資料庫傳回這些值。您可以使用 `DatabaseGenerated` 註釋以及 `Computed` 枚舉在類別中標記這些屬性。其他枚舉包括 `None` 和 `Identity` 。

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.Computed)]
public DateTime DateCreated { get; set; }
```

You can use database generated on byte or timestamp columns when code first is generating the database, otherwise you should only use this when pointing to existing databases because code first won't be able to determine the formula for the computed column.
當程式碼優先產生資料庫時，您可以使用在位元組或時間戳記列上產生的資料庫，否則您只應在指向現有資料庫時使用它，因為程式碼優先將無法確定計算列的公式。

You read above that by default, a key property that is an integer will become an identity key in the database. That would be the same as setting `DatabaseGenerated` to `DatabaseGeneratedOption.Identity`. If you do not want it to be an identity key, you can set the value to `DatabaseGeneratedOption.None`.
如上所述，預設情況下，整數類型的鍵屬性將成為資料庫中的標識鍵。這與將 `DatabaseGenerated` 設定為 `DatabaseGeneratedOption.Identity` 相同。如果您不想將其作為識別鍵，可以將其值設定為 `DatabaseGeneratedOption.None` 。

## Index  索引

> Note  筆記
>
> **EF 6.1 Onwards Only** - The `Index` attribute was introduced in Entity Framework 6.1. If you are using an earlier version the information in this section does not apply.
> **僅限 EF 6.1 以上版本** - `Index` 特性是在 Entity Framework 6.1 中引入的。如果您使用的是早期版本，則本節中的資訊不適用。

You can create an index on one or more columns using the **IndexAttribute**. Adding the attribute to one or more properties will cause EF to create the corresponding index in the database when it creates the database, or scaffold the corresponding **CreateIndex** calls if you are using Code First Migrations.
您可以使用 **IndexAttribute** 在一個或多個列上建立索引。將屬性新增至一個或多個屬性將導致 EF 在建立資料庫時在資料庫中建立對應的索引，或者如果您使用 Code First 遷移，則建立對應的 **CreateIndex** 呼叫。

For example, the following code will result in an index being created on the `Rating` column of the `Posts` table in the database.
例如，以下程式碼將導致在資料庫中 `Posts` 表的 `Rating` 列上建立索引。

```csharp
public class Post
{
	public int Id { get; set; }
	public string Title { get; set; }
	public string Content { get; set; }
	[Index]
	public int Rating { get; set; }
	public int BlogId { get; set; }
}
```

By default, the index will be named `IX_<property name>` (IX_Rating in the above example). You can also specify a name for the index though. The following example specifies that the index should be named `PostRatingIndex`.
預設情況下，索引將命名為 `IX_<property name>`（上例為 IX_Rating ）。您也可以為索引指定名稱。以下範例指定索引應命名為 `PostRatingIndex` 。

```csharp
[Index("PostRatingIndex")]
public int Rating { get; set; }
```

By default, indexes are non-unique, but you can use the `IsUnique` named parameter to specify that an index should be unique. The following example introduces a unique index on a `User`'s login name.
預設情況下，索引是非唯一的，但您可以使用 `IsUnique` 命名參數來指定索引應該是唯一的。以下範例在 `User` 的登入名稱上引入了唯一索引。

```csharp
public class User
{
	public int UserId { get; set; }

	[Index(IsUnique = true)]
	[StringLength(200)]
	public string Username { get; set; }

	public string DisplayName { get; set; }
}
```

### Multiple-Column Indexes  多列索引

Indexes that span multiple columns are specified by using the same name in multiple Index annotations for a given table. When you create multi-column indexes, you need to specify an order for the columns in the index. For example, the following code creates a multi-column index on `Rating` and `BlogId` called `IX_BlogIdAndRating`. `BlogId` is the first column in the index and `Rating` is the second.
跨多列的索引透過在給定表的多個 Index 註解中使用相同的名稱來指定。建立多列索引時，需要指定索引中各列的順序。例如，下列程式碼在 `Rating` 和 `BlogId` 上建立一個名為 `IX_BlogIdAndRating` 的多列索引。 `BlogId` 是索引中的第一列， `Rating` 是第二列。

```csharp
public class Post
{
	public int Id { get; set; }
	public string Title { get; set; }
	public string Content { get; set; }
	[Index("IX_BlogIdAndRating", 2)]
	public int Rating { get; set; }
	[Index("IX_BlogIdAndRating", 1)]
	public int BlogId { get; set; }
}
```

## Relationship Attributes: InverseProperty and ForeignKey 關係屬性

> Note  筆記
>
> This page provides information about setting up relationships in your Code First model using Data Annotations. For general information about relationships in EF and how to access and manipulate data using relationships, see [Relationships & Navigation Properties](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/relationships).*
> 本頁提供有關使用資料註釋在 Code First 模型中設定關係的資訊。有關 EF 中的關係以及如何使用關係存取和操作資料的常規信息，請參閱 [關係和導航屬性](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/relationships) 。 *

Code first convention will take care of the most common relationships in your model, but there are some cases where it needs help.
程式碼優先慣例將處理模型中最常見的關係，但在某些情況下它需要幫助。

Changing the name of the key property in the `Blog` class created a problem with its relationship to `Post`. 
更改 `Blog` 類別中關鍵屬性的名稱會導致其與 `Post` 的關係出現問題。

When generating the database, code first sees the `BlogId` property in the Post class and recognizes it, by the convention that it matches a class name plus **Id**, as a foreign key to the `Blog` class. But there is no `BlogId` property in the blog class. The solution for this is to create a navigation property in the `Post` and use the `ForeignKey` DataAnnotation to help code first understand how to build the relationship between the two classes (using the `Post.BlogId` property) as well as how to specify constraints in the database.
產生資料庫時，程式碼首先會看到 `BlogId` Post 類別中的屬性，並根據與類別名稱加 **Id** 相符的慣例將其識別為 `Blog` 類別的外鍵。但 blog 類別中沒有 `BlogId` 屬性。此問題的解決方案是在 `Post` 中建立導航屬性，並使用 `ForeignKey` DataAnnotation 幫助程式碼首先了解如何建立兩個類別之間的關係（使用 `Post.BlogId` 屬性），以及如何在資料庫中指定約束。

```csharp
public class Post
{
		public int Id { get; set; }
		public string Title { get; set; }
		public DateTime DateCreated { get; set; }
		public string Content { get; set; }
		public int BlogId { get; set; }
		[ForeignKey("BlogId")]
		public Blog Blog { get; set; }
		public ICollection<Comment> Comments { get; set; }
}
```

The constraint in the database shows a relationship between `InternalBlogs.PrimaryTrackingKey` and `Posts.BlogId`. 
資料庫中的約束顯示了 `InternalBlogs.PrimaryTrackingKey` 和 `Posts.BlogId` 之間的關係。

![[jj591583-figure09.png]]

The `InverseProperty` is used when you have multiple relationships between classes.
當類別之間存在多種關係時，請使用 `InverseProperty` 。

In the `Post` class, you may want to keep track of who wrote a blog post as well as who edited it. Here are two new navigation properties for the Post class.
在 `Post` 類別中，您可能想要追蹤部落格文章的作者以及編輯者。以下是 Post 類別的兩個新導航屬性。

```csharp
public Person CreatedBy { get; set; }
public Person UpdatedBy { get; set; }
```

You’ll also need to add in the `Person` class referenced by these properties. The `Person` class has navigation properties back to the `Post`, one for all of the posts written by the person and one for all of the posts updated by that person.
您還需要新增這些屬性所引用的 `Person` 類別。 `Person` 類別具有指向 `Post` 的導航屬性，一個用於保存該用戶發布的所有帖子，另一個用於保存該用戶更新的所有帖子。

```csharp
public class Person
{
		public int Id { get; set; }
		public string Name { get; set; }
		public List<Post> PostsWritten { get; set; }
		public List<Post> PostsUpdated { get; set; }
}
```

Code first is not able to match up the properties in the two classes on its own. The database table for `Posts` should have one foreign key for the `CreatedBy` person and one for the `UpdatedBy` person but code first will create four foreign key properties: **Person_Id**, **Person_Id1**, **CreatedBy_Id** and **UpdatedBy_Id**.
Code First 無法單獨匹配這兩個類別中的屬性。 Posts 的 `Posts` 表應該有一個用於 `CreatedBy` 人員的外鍵和一個用於 `UpdatedBy` 人員的外鍵，但 Code First 會建立四個外鍵屬性： **Person_Id** 、 **Person_Id1** 、 **CreatedBy_Id** 和 **UpdatedBy_Id** 。

![[jj591583-figure10.png]]

To fix these problems, you can use the `InverseProperty` annotation to specify the alignment of the properties.
為了解決這些問題，您可以使用 `InverseProperty` 註解來指定屬性的對齊方式。

```csharp
[InverseProperty("CreatedBy")]
public List<Post> PostsWritten { get; set; }

[InverseProperty("UpdatedBy")]
public List<Post> PostsUpdated { get; set; }
```

Because the `PostsWritten` property in Person knows that this refers to the `Post` type, it will build the relationship to `Post.CreatedBy`. Similarly, `PostsUpdated` will be connected to `Post.UpdatedBy`. And code first will not create the extra foreign keys.
因為 Person 中的 `PostsWritten` 屬性知道 this 引用的是 `Post` 型，所以它會與 `Post.CreatedBy` 建立關係。同樣， `PostsUpdated` 也會與 `Post.UpdatedBy` 建立關係。並且 Code First 不會創建額外的外鍵。

![[jj591583-figure11.png]]

## Summary  概括

`DataAnnotations` not only let you describe client and server side validation in your code first classes, but they also allow you to enhance and even correct the assumptions that code first will make about your classes based on its conventions. With `DataAnnotations` you can not only drive database schema generation, but you can also map your code first classes to a pre-existing database.
`DataAnnotations` 不僅允許你在 Code First 類別中描述客戶端和伺服器端驗證，還允許你增強甚至修正 Code First 根據其慣例對類別所做的假設。使用 `DataAnnotations`，你不僅可以驅動資料庫架構生成，還可以將 Code First 類別對應到現有資料庫。

While they are very flexible, keep in mind that `DataAnnotations` provide only the most commonly needed configuration changes you can make on your code first classes. To configure your classes for some of the edge cases, you should look to the alternate configuration mechanism, Code First’s `Fluent API` .
雖然 `DataAnnotations` 非常靈活，但請記住，==它們僅提供您可以在 Code First 類別上進行的最常用的配置變更。==要針對某些邊緣情況配置您的類，您應該考慮另一種配置機制，即 Code First 的 `Fluent API`。
