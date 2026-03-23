---
aliases:
date:
update:
author: ZZZ Projects
language: C#
sourceurl: https://www.entityframeworktutorial.net/code-first/code-first-conventions.aspx
tags:
  - CSharp
  - 資料庫
  - EF6
  - EF_Code_First
---

# 主頁

https://www.entityframeworktutorial.net/code-first/what-is-code-first.aspx

# EF 6 Code-First Conventions EF 6 程式碼優先慣例

Conventions are sets of default rules which automatically configure a conceptual model based on your domain classes while using the code-first approach. As you have seen in the code-first example in the previous chapter, EF API configured primary keys, foreign keys, relationships, column data types, etc. from the domain classes without any additional configurations. This is because of the EF code-first conventions. If they are followed in domain classes, then the database schema will be configured based on the conventions. These EF 6.x Code-First conventions are defined in the [System.Data.Entity.ModelConfiguration.Conventions](https://msdn.microsoft.com/en-us/library/system.data.entity.modelconfiguration.conventions\(v=vs.113\).aspx) namespace.
慣例是一組預設規則，在使用程式碼優先方法時，它們會根據域類別自動配置概念模型。如您在上一章的程式碼優先範例中所見，EF API 從網域類別中配置了主鍵、外鍵、關係、列資料類型等，而無需任何其他配置。這是由於 EF 程式碼優先慣例。如果在域類別中遵循這些慣例，則資料庫模式將基於慣例進行配置。這些 EF 6.x 程式碼優先慣例在 [System.Data.Entity.ModelConfiguration.Conventions](https://msdn.microsoft.com/en-us/library/system.data.entity.modelconfiguration.conventions\(v=vs.113\).aspx) 命名空間中定義。

The following table lists default Code-First conventions:
下表列出了預設的 Code-First 慣例：

| Default Convention For  預設慣例            | Description<br>描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Schema  方案                              | By default, EF creates all the DB objects into the **dbo** schema.  <br>預設情況下，EF 將所有 DB 物件建立到 **dbo** 模式中。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Table Name  表名                          | \<Entity Class Name\> + 's'<br>EF will create a DB table with the entity class name suffixed by 's' e.g. `Student` domain class (entity) would map to the `Students` table.  <br>EF 將建立一個 DB 表，其實體類別名稱以「s」為後綴，例如 `Student` 領域類別（實體）將對應到 `Students` 表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Primary Key Name   主鍵名稱                 | 1) Id<br>2) \<Entity Class Name\> + "Id" (case insensitive 不區分大小寫)  <br>  <br>EF will create a primary key column for the property named `Id` or \<Entity Class Name\> + "Id" (case insensitive).  EF 將為名為 `Id` 的屬性建立主鍵列或 \<Entity Class Name\>+“Id”（不區分大小寫）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Foreign Key Property Name  <br>外鍵屬性名稱   | By default EF will look for the foreign key property with the same name as the principal entity primary key name.  <br>預設情況下，EF 將尋找與主體實體主鍵名稱同名的外鍵屬性。  <br>If the foreign key property does not exist, then EF will create an FK column in the DB table with \<Dependent Navigation Property Name\> + "\_" + \<Principal Entity Primary Key Property Name\>  <br>如果外鍵屬性不存在，那麼 EF 將在 DB 表中建立一個 FK column \<Dependent Navigation Property Name\> + “\_” + \<Principal Entity Primary Key Property Name\>  <br>e.g. EF will create `Grade_GradeId` foreign key column in the `Students` table if the `Student` entity does not contain foreign key property for `Grade`.  <br>例如，如果 `Student` 實體不包含 `Grade` 的外鍵屬性，EF 將在 `Students` 表中建立 `Grade_GradeId` 外鍵列。 |
| Null Column   空列                        | EF creates a **null** column for all **reference type properties** and **nullable primitive properties**. e.g. `string`, `Nullable<int>`, `Student`, `Grade` (all class type properties)  <br>EF 為所有引用類型屬性和可空原始屬性建立一個空白列。例如 `string`、`Nullable<int>`、`Student`、`Grade`（所有類別類型屬性）                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Not Null Column   非空列                   | EF creates **NotNull** columns for **Primary Key properties** and **non-nullable value** type properties e.g. `int`, `float`, `decimal`, `datetime` etc.  <br>EF 為**主鍵屬性**和**非空值類型屬性**建立 **NotNull** columns，例如 int、float、decimal、datetime 等。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| DB Columns order   資料庫列順序               | EF will create **DB** columns in the same order as the properties in an entity class. However, primary key columns would be moved first.  <br>EF 將依照與實體類別中屬性相同的順序建立 **DB** columns。但是，主鍵列將首先移動。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Properties mapping to DB  <br>映射到資料庫的屬性 | By default, all properties will map to the database. Use the `[NotMapped]` attribute to exclude a property or class from DB mapping.  <br>預設情況下，所有屬性都會對應到資料庫。使用 `[NotMapped]` 特性可以將屬性或類別從資料庫映射中排除。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Cascade delete  瀑布刪除                    | Enabled by default for all types of relationships.  <br>對所有類型的關係預設為啟用。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

The following table lists C# data types mapped with SQL Server data types.
下表列出了與 SQL Server 資料類型對應的 C# 資料類型。

| C# Data Type | Mapping to SQL Server Data Type    |
| ------------ | ---------------------------------- |
| int          | int                                |
| string       | nvarchar(Max)                      |
| decimal      | decimal(18,2)                      |
| float        | real                               |
| byte[]       | varbinary(Max)                     |
| datetime     | datetime                           |
| bool         | bit                                |
| byte         | tinyint                            |
| short        | smallint                           |
| long         | bigint                             |
| double       | float                              |
| char         | No mapping                         |
| sbyte        | No mapping  <br>(throws exception) |
| object       | No mapping                         |

The following figure illustrates the conventions mapping with the database.
下圖說明了與資料庫的慣例映射。

![[ef6-codefirst-conventions.png|Entity Framework code-first conventions]]

## Relationship Convention  關係慣例

EF 6 infers the One-to-Many relationship using the navigation property by default convention. Visit [Convention for One-to-Many relationship](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#conventions-for-one-to-many-ef6) chapter for more information.
EF 6 預設使用導航屬性來推斷一對多關係。更多信息，請參閱 [一對多關係慣例](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#conventions-for-one-to-many-ef6) 章節。

**Note:** EF 6 does not include default conventions for One-to-One and Many-to-Many relationships. You need to configure them either using Fluent API or Data Annotations.
**注意：** EF 6 不包含一對一和多對多關係的預設慣例。您需要使用 Fluent API 或資料註解進行設定。

## Complex Type Convention  複雜類型慣例

Code First creates the complex type for the class which does not include a key property and also the primary key is not registered using a data annotation attribute or Fluent API.
Code First 為不包含鍵屬性的類別建立複雜類型，且主鍵未使用資料註解屬性或 Fluent API 註冊。

This was an overview of code first conventions. These conventions can be overridden using Data Annotations or Fluent API.
這是 Code First 慣例的概述。可以使用 Data Annotations 或 Fluent API 覆寫這些慣例。
