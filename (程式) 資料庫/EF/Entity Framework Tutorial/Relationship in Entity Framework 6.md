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

# Link

[Relationships between Entities in Entity Framework 6](https://www.entityframeworktutorial.net/entityframework6/entity-relationships.aspx)
[Configure One-to-Zero-or-One Relationship in Entity Framework 6](https://www.entityframeworktutorial.net/code-first/configure-one-to-one-relationship-in-code-first.aspx)
[Configure One-to-Many Relationships in EF 6](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx)
[Configure Many-to-Many Relationships in Code-First](https://www.entityframeworktutorial.net/code-first/configure-many-to-many-relationship-in-code-first.aspx)

---

# Relationships between Entities in Entity Framework 6 Entity Framework 6 中實體之間的關係

Here, you will learn how Entity Framework manages the relationships between entities.
在這裡，您將了解實體框架如何管理實體之間的關係。

Entity Framework supports three types of relationships, same as database: 1) One-to-One 2) One-to-Many, and 3) Many-to-Many.
實體框架支援三種類型的關係，與資料庫相同：1）一對一，2）一對多，3）多對多。

We have created an Entity Data Model for the SchoolDB database in the [Create Entity Data Model](https://www.entityframeworktutorial.net/entityframework6/create-entity-data-model.aspx) chapter. The following figure shows the visual designer for that EDM with all the entities and relationships among them.
我們在 [「建立實體資料模型」一章中為 SchoolDB 資料庫建立了一個實體資料](https://www.entityframeworktutorial.net/entityframework6/create-entity-data-model.aspx) 模型。下圖展示了該實體資料模型的可視化設計器，其中包含所有實體及其之間的關係。

![[entityrelation2.png|Entity relationships in Entity Framework]]

Let's see how each relationship (association) is being managed by Entity Framework.
讓我們看看實體框架如何管理每個關係（關聯）。

## One-to-One Relationship  一對一關係

As you can see in the above figure, `Student` and `StudentAddress` have a One-to-One relationship (zero or one). A student can have only one or zero addresses. Entity Framework adds the `Student` [reference navigation property](https://www.entityframeworktutorial.net/basics/entity-in-entityframework.aspx#reference-navigation-property) into the `StudentAddress` entity and the `StudentAddress` navigation entity into the `Student` entity. Also, the `StudentAddress` entity has both `StudentId` properties as the primary key and the foreign key, which makes it a one-to-one relationship.
如上圖所示， `Student` 和 `StudentAddress` 是一對一關係（零個或一個）。每個學生只能有一個或零個地址。 Entity Framework 將 `Student` [參考導覽屬性](https://www.entityframeworktutorial.net/basics/entity-in-entityframework.aspx#reference-navigation-property) 新增至 `StudentAddress` 實體中，並將 `StudentAddress` 導覽實體新增至 `Student` 實體。此外， `StudentAddress` 實體同時具有 `StudentId` 屬性作為主鍵和外鍵，這使得它們成為一對一關係。

```csharp title="One-to-One Relationship"
public partial class Student
{
    public Student()
    {
        this.Courses = new HashSet<Course>();
    }

    public int StudentID { get; set; } // <-- PK
    public string StudentName { get; set; }
    public Nullable<int> StandardId { get; set; }
    public byte[] RowVersion { get; set; }
    
    public virtual StudentAddress StudentAddress { get; set; } // <-- Navigation Entity
}
    
public partial class StudentAddress
{
    public string Address1 { get; set; }
    public string Address2 { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    
    public int StudentID { get; set; }           // <-- PK & FK
    public virtual Student Student { get; set; } // <-- Reference Navigation Property
}
```

In the above example, the `StudentId` property needs to be primary key as well as foreign key. This can be configured using Fluent API in the `OnModelCreating` method of the context class.
在上面的範例中， `StudentId` 屬性既需要作為主鍵，也需要作為外鍵。這可以在 context 類別的 `OnModelCreating` 方法中使用 Fluent API 進行配置。

## One-to-Many Relationship 一對多關係

The `Standard` and `Teacher` entities have a One-to-Many relationship marked by multiplicity where 1 is for One and * is for Many. This means that `Standard` can have many Teachers whereas `Teacher` can associate with only one `Standard`.
`Standard` 和 `Teacher` 實體之間存在一對多關係，以多重性為標誌，其中 1 表示“一”，* 表示“多”。這意味著 `Standard` 可以有多個 Teachers ，而 `Teacher` 只能與一個 `Standard` 關聯。

To represent this, the `Standard` entity has the [collection navigation property](https://www.entityframeworktutorial.net/entityframework6/entity-in-entityframework.aspx#collection-navigation-property) `Teachers` (please notice that it's plural), which indicates that one `Standard` can have a collection of `Teachers` (many teachers). And the `Teacher` entity has a `Standard` navigation property (reference property), which indicates that `Teacher` is associated with one `Standard`. Also, it contains the `StandardId` foreign key (PK in `Standard` entity). This makes it a One-to-Many relationship.
為了表示這一點， `Standard` 實體具有 [集合導航屬性](https://www.entityframeworktutorial.net/entityframework6/entity-in-entityframework.aspx#collection-navigation-property) `Teachers` （請注意，它是複數），這表示一個 `Standard` 可以包含一組 `Teachers`（多位教師）。且 `Teacher` 實體具有一個 `Standard` 導航屬性（參考屬性），該屬性指示 `Teacher` 與一個 `Standard` 關聯。此外，它還包含 `StandardId` 外鍵（ `Standard` 實體中的主鍵）。這使得它們成為一對多關係。

```csharp title="One-to-Many Relationship"
public partial class Standard
{
    public Standard()
    {
        this.Teachers = new HashSet<Teacher>();
    }

    public int StandardId { get; set; } // <-- PK
    public string StandardName { get; set; }
    public string Description { get; set; }
    
    public virtual ICollection<Teacher> Teachers { get; set; } // <-- Collection Navigation Property
}

public partial class Teacher
{
    public Teacher()
    {
        this.Courses = new HashSet<Course>();
    }

    public int TeacherId { get; set; }
    public string TeacherName { get; set; }
    public Nullable<int> TeacherType { get; set; }
    
    public Nullable<int> StandardId { get; set; }  // <-- FK (PK in `Standard` entity)
    public virtual Standard Standard { get; set; } // <-- Navigation Property (Reference Property)
}
```

## Many-to-Many Relationship 多對多關係

The `Student` and `Course` have a Many-to-Many relationship marked by * multiplicity. It means one `Student` can enroll in many Courses and also, one `Course` can be taught to many Students.
`Student` 和 `Course` 之間存在多對多關係，以 * 表示。這意味著一個 `Student` 可以註冊多門課程，一門 `Course` 也可以教導多個學生。

The database includes the `StudentCourse` joining table which includes the primary key of both the tables (`Student` and `Course` tables). Entity Framework represents many-to-many relationships by not having the entity set (`DbSet property`) for the joining table in the CSDL and visual designer. Instead it manages this through mapping.
資料庫包含 `StudentCourse` 連接表，該表包含兩個表（ `Student` 表和 `Course` 表）的主鍵。 Entity Framework 在 CSDL 和視覺化設計器中不為連接表設定實體集（ `DbSet property` ）來表示多對多關係。相反，它透過映射來管理這一點。

As you can see in the above figure, the `Student` entity includes the [collection navigation property](https://www.entityframeworktutorial.net/entityframework6/what-is-entity-in-entityframework.aspx#collection-navigation-property) `Courses` and `Course` entity includes the collection navigation property `Students` to represent a many-to-many relationship between them.
如下圖所示， `Student` 實體包含 [集合導航屬性](https://www.entityframeworktutorial.net/entityframework6/what-is-entity-in-entityframework.aspx#collection-navigation-property) `Courses` ， `Course` 實體包含集合導航屬性 `Students` ，以表示它們之間的多對多關係。

The following code snippet shows the `Student` and `Course` entity classes.
以下程式碼片段顯示了 `Student` 和 `Course` 實體類別。

```csharp title="Many-to-Many Relationship"
public partial class Student
{
    public Student()
    {
        this.Courses = new HashSet<Course>();
    }

    public int StudentID { get; set; }
    public string StudentName { get; set; }
    public Nullable<int> StandardId { get; set; }
    public byte[] RowVersion { get; set; }
    
    public virtual ICollection<Course> Courses { get; set; } // <-- Collection Navigation Property
}

public partial class Course
{
    public Course()
    {
        this.Students = new HashSet<Student>();
    }

    public int CourseId { get; set; }
    public string CourseName { get; set; }
    public System.Data.Entity.Spatial.DbGeography Location { get; set; }
    
    public virtual ICollection<Student> Students { get; set; } // <-- Collection Navigation Property
}
```

**Note:** Entity Framework supports many-to-many relationships only when the joining table (`StudentCourse` in this case) does **NOT** include any columns other than PKs of both tables. If the joining table contains additional columns, such as `DateCreated`, then the EDM creates an entity for the middle table as well and you will have to manage CRUD operations for many-to-many entities manually.
**注意：** 僅當連接表（本例中為 `StudentCourse` ） **不**包含除兩個表的主鍵之外的任何欄位時，Entity Framework 才會支援多對多關係。如果連線表包含其他欄位（例如 `DateCreated` ，則 EDM 也會為中間表建立實體，您將需要手動管理多對多實體的 CRUD 操作。

Open EDM in XML view. You can see that SSDL (storage schema) has the `StudentCourse` EntitySet, but CSDL doesn't have it. Instead, it's being mapped in the navigation property of the `Student` and `Course` entities. MSL (C-S Mapping) has mapping between `Student` and `Course` put into the `StudentCourse` table in the <AssociationSetMapping/> section.
在 XML 視圖中開啟 EDM。您可以看到 SSDL（儲存架構）包含 `StudentCourse` EntitySet，但 CSDL 沒有。相反，它被映射到 `Student` 和 `Course` 實體的導航屬性。 MSL（CS 映射）將 `Student` 和 `Course` 之間的映射放入了部分中的 `StudentCourse` 表中。

![[entityrelation3.png|entity relationships in entity framework]]

Thus, a many-to-many relationship is being managed by C-S mapping in EDM. So, when you add a `Student` in a `Course` or a `Course` in a `Student` entity and when you save it, it will then insert the PK of the added student and course in the `StudentCourse` table. So, this mapping not only enables a convenient association directly between the two entities, but also manages querying, inserts, and updates across these joins.
因此，EDM 中的 C-S 映射管理多對多關係。因此，當您在 `Course` 中新增 `Student` ，或在 `Student` 實體中新增 `Course` 並儲存時，系統會將新增的學生和課程的主鍵插入到 `StudentCourse` 表中。 因此，這種映射不僅能夠直接在兩個實體之間建立方便的關聯，而且還能夠管理這些連接的查詢、插入和更新。

---

# Configure One-to-Zero-or-One Relationship in Entity Framework 6 在 Entity Framework 6 中配置一對零或一關係

Here, you will learn to configure One-to-Zero-or-One relationships between two entities.
在這裡，您將學習如何配置兩個實體之間的一對零或一關係。

We will implement a one-to-zero-or-one relationship between the following `Student` and `StudentAddress` entities.
我們將在以下 `Student` 和 `StudentAddress` 實體之間實現一對零或一的關係。

```csharp
public class Student
{
    public int StudentId { get; set; }
    public string StudentName { get; set; }

    public virtual StudentAddress Address { get; set; }
}

public class StudentAddress
{
    public int StudentAddressId { get; set; }
    public string Address1 { get; set; }
    public string Address2 { get; set; }
    public string City { get; set; }
    public int Zipcode { get; set; }
    public string State { get; set; }
    public string Country { get; set; }

    public virtual Student Student { get; set; }
}
```

==A one-to-zero-or-one relationship happens when a primary key of one table becomes PK & FK in another table== in a relational database such as SQL Server. So, we need to configure the above entities in such a way that EF creates the `Students` and `StudentAddresses` tables in the DB and makes the `StudentId` column in `Student` table as PrimaryKey (PK) and `StudentAddressId` column in the `StudentAddresses` table as PK and ForeignKey (FK) both.
在關聯式資料庫（例如 SQL Server）中，==當一個表的主鍵成為另一個表的主鍵和外鍵時，就會發生一對零或一的關係。==因此，我們需要以這樣的方式配置上述實體，即 EF 在資料庫中建立 `Students` 和 `StudentAddresses` 表，並將 `Student` 表中的 `StudentId` 列設定為主鍵（主鍵），將 `StudentAddresses` 表中的 `StudentAddressId` 列設定為主鍵和外鍵（外鍵）。

## Configure One-to-Zero-or-One Relationship using Data Annotation Attributes 使用資料註解屬性配置一對零或一關係

Here, we will apply data annotation attributes on the `Student` and `StudentAddress` entities to establish a one-to-zero-or-one relationship.
在這裡，我們將在 `Student` 和 `StudentAddress` 實體上應用資料註解屬性來建立一對零或一的關係。

The `Student` entity follows the default [code-first convention](https://www.entityframeworktutorial.net/code-first/code-first-conventions.aspx) as it includes the `StudentId` property which will be the key property. So, we don't need to apply any attributes on it because EF will make the `StudentId` column as a PrimaryKey in the `Students` table in the database.
`Student` 實體遵循預設的 [程式碼優先慣例](https://www.entityframeworktutorial.net/code-first/code-first-conventions.aspx) ，因為它包含 `StudentId` 屬性，該屬性將作為鍵屬性。因此，我們不需要在其上套用任何特性，因為 EF 會將 `StudentId` 欄位設定為資料庫中 `Students` 表中的 PrimaryKey。

For the `StudentAddress` entity, we need to configure the `StudentAddressId` as PK & FK both. The `StudentAddressId` property follows the default convention for primary key. So, we don't need to apply any attribute for PK. However, we also need to make it a foreign key which points to `StudentId` of the `Student` entity. So, apply `[ForeignKey("Student")]` on the `StudentAddressId` property which will make it a foreign key for the `Student` entity, as shown below.
針對 `StudentAddress` 這個實體，我們需要將 `StudentAddressId` 設定為 PK 與 FK 兩者。`StudentAddressId` 屬性遵循主鍵的預設慣例，所以不需要應用任何 PK 屬性。然而，我們也需要將它設定為外鍵，指向 `Student` 實體的 `StudentId`。因此，在 `StudentAddressId` 屬性上應用 `[ForeignKey("Student")]`，使其成為 `Student` 實體的外鍵，如下所示。

```csharp
public class Student
{
    public int StudentId { get; set; } // <-- PK
    public string StudentName { get; set; }

    public virtual StudentAddress Address { get; set; } // <-- Navigation Property
}

public class StudentAddress
{
    [ForeignKey("Student")] // <-- FK
    public int StudentAddressId { get; set; } // <-- PK

    public string Address1 { get; set; }
    public string Address2 { get; set; }
    public string City { get; set; }
    public int Zipcode { get; set; }
    public string State { get; set; }
    public string Country { get; set; }

    public virtual Student Student { get; set; } // <-- Navigation Property
}
```

Thus, you can use data annotation attributes to configure a one-to-zero-or-one relationship between two entities.
因此，您可以使用資料註解屬性來配置兩個實體之間的一對零或一的關係。

**Note:** `Student` includes the `StudentAddress` navigation property and `StudentAddress` includes the `Student` navigation property. With the one-to-zero-or-one relationship, a `Student` can be saved without `StudentAddress` but the `StudentAddress` entity cannot be saved without the `Student` entity. EF will throw an exception if you try to save the `StudentAddress` entity without the `Student` entity.
**注意：** `Student` 包含 `StudentAddress` 導航屬性， `StudentAddress` 也包含 `Student` 導航屬性。由於存在一對零或一的關係，因此可以在沒有 `StudentAddress` 情況下保存 `Student` ，但只有在 `Student` 實體存在的情況下才能保存 `StudentAddress` 實體。如果嘗試在不包含 `Student` 實體的情況下保存 `StudentAddress` 實體，EF 會拋出異常。

## Configure a One-to-Zero-or-One relationship using Fluent API 使用 Fluent API 配置一對零或一關係

Here, we will use Fluent API to configure a one-to-zero-or-one relationship between the `Student` and `StudentAddress` entities.
在這裡，我們將使用 Fluent API 在 `Student` 和 `StudentAddress` 實體之間配置一對零或一的關係。

The following example sets a one-to-zero or one relationship between `Student` and `StudentAddress` using Fluent API.
以下範例使用 Fluent API 設定 `Student` 和 `StudentAddress` 之間的一對零或一的關係。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    // Configure Student & StudentAddress entity 配置 Student 和 StudentAddress 實體
    modelBuilder.Entity<Student>()
                .HasOptional(s => s.Address) // Mark Address property optional in Student entity
							                 // 將學生實體中的地址屬性標記為可選
                .WithRequired(ad => ad.Student); // mark Student property as required in StudentAddress entity. Cannot save StudentAddress without Student
									            // 將 StudentAddress 實體中的 Student 屬性標記為必要。無法在沒有 Student 的情況下保存 StudentAddress
}
```

In the above example, we start with the `Student` entity. The `HasOptional()` method configures the `Address` navigation property in `Student` entity as optional (not required when saving the `Student` entity). Then, the `WithRequired()` method makes the `Student` navigation property of `StudentAddress` as required (required when saving the `StudentAddress` entity; it will throw an exception when the `StudentAddress` entity is saved without the `Student` navigation property). This will also make the `StudentAddressId` as ForeignKey.
在上面的範例中，我們從 `Student` 實體開始。 `HasOptional()` 方法將 `Student` 實體中的 `Address` 導航屬性配置為可選（儲存 `Student` 實體時不是必要的）。然後，`WithRequired()` 方法將 `StudentAddress` 的 `Student` 導航屬性設定為必要（儲存 `StudentAddress` 實體時是必要的；如果儲存 `StudentAddress` 實體時沒有 `Student` 導覽屬性，則會拋出例外）。這也會將 `StudentAddressId` 設定為 ForeignKey。

Thus, you can configure a one-to-zero-or-one relationship between two entities where the `Student` entity can be saved without attaching the `StudentAddress` object to it but the `StudentAddress` entity cannot be saved without attaching an object of the `Student` entity. This makes one end required.
因此，您可以在兩個實體之間配置一對零或一的關係，其中，無需附加 `StudentAddress` 對象即可保存 `Student` 實體，但若不附加 `Student` 實體的對象，則無法保存 `StudentAddress` 實體。這使得一端成為必需。

EF API will create the following tables in the database.
EF API 將在資料庫中建立以下表。

![[onetoone-1.png|one-to-one relationship in code first]]

## Configure a One-to-One relationship using Fluent API 使用 Fluent API 配置一對一關係

We can configure a one-to-one relationship between entities using Fluent API where both ends are required, meaning that the `Student` entity object must include the `StudentAddress` entity object and the `StudentAddress` entity must include the `Student` entity object in order to save it.
我們可以使用 Fluent API 來配置實體之間的一對一關係，其中兩端都是必需的，這表示 `Student` 實體物件必須包含 `StudentAddress` 實體對象，並且 `StudentAddress` 實體必須包含 `Student` 實體物件才能保存它。

**Note:** One-to-one relationships are technically not possible in MS SQL Server. These will always be one-to-zero-or-one relationships. EF forms One-to-One relationships on entities not in the DB.
**注意:** 從技術上講，MS SQL Server 無法實現一對一關係。這些關係總是一對零或一對一的關係。 EF 會與資料庫中不存在的實體建立一對一關係。

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    // Configure StudentId as FK for StudentAddress
    // 將 StudentId 配置為 StudentAddress 的 FK
    modelBuilder.Entity<Student>()
                .HasRequired(s => s.Address)
                .WithRequiredPrincipal(ad => ad.Student);
}
```

In the above example, `modelBuilder.Entity<Student>().HasRequired(s => s.Address)` makes the `Address` property of `StudentAddress` as required and `.WithRequiredPrincipal(ad => ad.Student)` makes the `Student` property of the `StudentAddress` entity as required. Thus it configures both ends as required. So now, when you try to save the `Student` entity without the `StudentAddress` entity, or the `StudentAddress` entity without the `Student`, it will throw an exception.
在上面的範例中， `modelBuilder.Entity<Student>().HasRequired(s => s.Address)` 將 `StudentAddress` 的 `Address` 屬性設為必需，而 `.WithRequiredPrincipal(ad => ad.Student)` 將 `StudentAddress` 實體的 `Student` 屬性設為必要。這樣，它將兩端都配置為必需。因此，現在，當您嘗試儲存不包含 `StudentAddress` 實體的 `Student` 實體，或儲存不包含 `Student` 的 `StudentAddress` 實體時，就會拋出例外。

Create a read-only Entity Data Model for the above example using [EF Power Tools](https://www.entityframeworktutorial.net/code-first/entity-framework-power-tools.aspx). The entities will appear like the diagram shown below:
使用 [EF Power Tools](https://www.entityframeworktutorial.net/code-first/entity-framework-power-tools.aspx) 為上述範例建立唯讀實體資料模型。實體將如下圖所示：

![[onetoone-2.png|one-to-one relationship in code first]]

---

# Configure One-to-Many Relationships in EF 6 在 EF 6 中配置一對多關係

Here, we will learn how to configure One-to-Many relationships between two entities (domain classes) in Entity Framework 6.x using the code-first approach.
在這裡，我們將學習如何使用程式碼優先方法在 Entity Framework 6.x 中配置兩個實體（領域類別）之間的一對多關係。

Let's configure a one-to-many relationship between the following `Student` and `Grade` entities where there can be many students in one grade.
讓我們在以下 `Student` 和 `Grade` 實體之間配置一對多關係，其中一個年級可以有多個學生。

```csharp

public class Student
{
    public int StudentId { get; set; }
    public string StudentName { get; set; }
}
       
public class Grade
{
    public int GradeId { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }
}
```

After implementing the one-to-many relationship in the above entities, the database tables for `Student` and `Grade` will look like below.
在上述實體中實現一對多關係後， `Student` 和 `Grade` 的資料庫表將如下所示。

![[onetomany-db.png|one-to-many relationship in code first]]

The one-to-many relationship can be configured in the following ways.
可以透過以下方式配置一對多關係。

1. [By following Conventions](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#conventions-for-one-to-many-ef6)
   遵守慣例
2. [By using Fluent API Configurations](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#configure-one-to-many-using-fluent-api)
   透過使用 Fluent API 配置

## Conventions for One-to-Many Relationships 一對多關係的慣例

There are certain conventions in Entity Framework which if followed in entity classes (domain classes) will automatically result in a one-to-many relationship between two tables in the database. You don't need to configure anything else.
Entity Framework 中有一些慣例，如果在實體類別（領域類別）中遵循這些慣例，資料庫中兩個資料表之間會自動建立一對多關係。您無需進行任何其他配置。

Let's look at an example of all the conventions which create a one-to-many relationship.
讓我們來看一個創建一對多關係的所有慣例的範例。

### Convention 1  慣例 1

We want to establish a one-to-many relationship between the `Student` and `Grade` entities where many students are associated with one `Grade`. It means that each `Student` entity points to a `Grade`. This can be achieved by including a reference navigation property of type `Grade` in the `Student` entity class, as shown below.
我們希望在 `Student` 和 `Grade` 實體之間建立一對多關係，即多個學生對應一個 `Grade` 。這意味著每個 `Student` 實體都指向一個 `Grade` 。這可以透過在 `Student` 實體類別中新增一個 `Grade` 類型的引用導航屬性來實現，如下所示。

```csharp

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Grade Grade { get; set; } // <--
}

public class Grade
{
    public int GradeId { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }
}
```

In the above example, the `Student` class includes a reference navigation property of `Grade` class. So, there can be many students in a single grade. This will result in a one-to-many relationship between the `Students` and `Grades` table in the database, where the `Students` table includes foreign key `Grade_GradeId` as shown below.
在上面的範例中， `Student` 類別包含一個 `Grade` 類別的引用導覽屬性。因此，一個年級可以有多個學生。這將導致資料庫中 `Students` 表和 `Grades` 表之間存在一對多關係，其中 `Students` 表包含外鍵 `Grade_GradeId` 如下所示。

![[onetomany-1.png]]

Notice that the reference property is nullable, so it creates a nullable foreign key column `Grade_GradeId` in the `Students` table. You can [configure `NotNull` foreign key using fluent API](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#notnull-foreignkey-using-fluent-api).
請注意，引用屬性是可空的，因此它會在 `Students` 表中建立一個可空的外鍵列 `Grade_GradeId` 。您可以 [使用 Fluent API 來設定「NotNull」外鍵](https://www.entityframeworktutorial.net/code-first/configure-one-to-many-relationship-in-code-first.aspx#notnull-foreignkey-using-fluent-api) 。

### Convention 2  慣例 2

Another convention is to include a collection navigation property in the principal entity as shown below.
另一個慣例是在主體實體中包含一個集合導航屬性，如下所示。

```csharp

public class Student
{
    public int StudentId { get; set; }
    public string StudentName { get; set; }
}

public class Grade
{
    public int GradeId { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }

    public ICollection<Student> Students { get; set; } // <--
}
```

In the above example, the `Grade` entity includes a collection navigation property of type `ICollection<Student>`. This also results in a one-to-many relationship between the `Student` and `Grade` entities. This example produces the same result in the database as convention 1.
在上面的範例中， `Grade` 實體包含一個類型為 `ICollection<Student>` 的集合導航屬性。這也導致了 `Student` 和 `Grade` 實體之間的一對多關係。此範例在資料庫中產生的結果與慣例 1 相同。

### Convention 3  慣例 3

Including navigation properties at both ends will also result in a one-to-many relationship, as shown below.
包含兩端的導航屬性也會導致一對多的關係，如下所示。

```csharp

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Grade Grade { get; set; } // <--
}

public class Grade
{
    public int GradeID { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }
    
    public ICollection<Student> Students { get; set; } // <--
}
```

In the above example, the `Student` entity includes a reference navigation property of the `Grade` type and the `Grade` entity class includes a collection navigation property of the `ICollection<Student>` type which results in a one-to-many relationship. This example produces the same result in the database as convention 1.
在上面的範例中， `Student` 實體包含一個 `Grade` 類型的引用導覽屬性，而 `Grade` 實體類別包含一個 `ICollection<Student>` 類型的集合導覽屬性，這導致了一對多關係。此範例在資料庫中產生的結果與慣例 1 相同。

### Convention 4  慣例 4

A fully defined relationship at both ends will create a one-to-many relationship, as shown below.
兩端完全定義的關係將建立一對多關係，如下所示。

```csharp

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    public int GradeId { get; set; } // <--
    public Grade Grade { get; set; } // <--
}

public class Grade
{

    public int GradeId { get; set; }
    public string GradeName { get; set; }
    
    public ICollection<Student> Students { get; set; } // <--
}
```

In the above example, the `Student` entity includes foreign key property `GradeId` with its reference property `Grade`. This will create a one-to-many relationship with the NotNull foreign key column in the `Students` table, as shown below.
在上面的範例中， `Student` 實體包含外鍵屬性 `GradeId` 及其引用屬性 `Grade` 。這將與 `Students` 表中的 NotNull 外鍵列建立一對多關係，如下所示。

![[onetomany-2.png]]

If the data type of `GradeId` is nullable integer, then it will create a null foreign key.
如果 `GradeId` 的資料型別是可空整數，那麼它將建立一個空外鍵。

```csharp

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }

    public int? GradeId { get; set; }
    public Grade Grade { get; set; }
}
```

The above code snippet will create a nullable `GradeId` column in the database because we have used `Nullable<int>` type (`?` is a shortcut for `Nullable<int>`)
上面的程式碼片段會在資料庫中建立一個可空的 `GradeId` 列，因為我們使用了 `Nullable<int>` 類型（ `?` 是 `Nullable<int>` 的捷徑）

## Configure a One-to-Many Relationship using Fluent API 使用 Fluent API 配置一對多關係

Generally, you don't need to configure the one-to-many relationship in entity framework because one-to-many relationship conventions cover all combinations. However, you may configure relationships using Fluent API at one place to make it more maintainable.
通常情況下，您不需要在實體框架中配置一對多關係，因為一對多關係的慣例涵蓋了所有組合。但是，您也可以使用 Fluent API 在一個地方配置關係，以使其更易於維護。

Consider the following `Student` and `Grade` entity classes.
考慮以下 `Student` 和 `Grade` 實體類。

```csharp

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }

    public int CurrentGradeId { get; set; }
    public Grade CurrentGrade { get; set; }
}

public class Grade
{
    public int GradeId { get; set; }
    public string GradeName { get; set; }
    public string Section { get; set; }

    public ICollection<Student> Students { get; set; }
}
```

You can configure a one-to-many relationship for the above entities using Fluent API by overriding the `OnModelCreating` method in the context class, as shown below.
您可以使用 Fluent API 透過重寫上下文類別中的 `OnModelCreating` 方法來為上述實體配置一對多關係，如下所示。

```csharp

public class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    public DbSet<Grade> Grades { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // configures one-to-many relationship
        modelBuilder.Entity<Student>()
            .HasRequired<Grade>(s => s.CurrentGrade)
            .WithMany(g => g.Students)
            .HasForeignKey<int>(s => s.CurrentGradeId);          
    }
}
```

Let's understand the above code step by step.
讓我們一步一步理解上面的程式碼。

- First, we need to start configuring with any one entity class. So, `modelBuilder.Entity<student>()` starts with the `Student` entity.
  首先，我們需要從任一個實體類別開始配置。因此， `modelBuilder.Entity<student>()` 從 `Student` 實體開始。
- Then, `.HasRequired<Grade>(s => s.CurrentGrade)` specifies that the `Student` entity requires the `CurrentGrade` property. This will create a NotNull foreign key column in the DB.
  然後， `.HasRequired<Grade>(s => s.CurrentGrade)` 指定 `Student` 實體需要 `CurrentGrade` 屬性。 這將在資料庫中建立一個 NotNull 外鍵列。
- Now, it's time to configure the other end of the relationship - the `Grade` entity.
  現在，是時候配置關係的另一端 - `Grade` 實體了。
- `.WithMany(g => g.Students)` specifies that the `Grade` entity class includes many `Student` entities. Here, many infers the `ICollection` type property.
  `.WithMany(g => g.Students)` 指定 `Grade` 實體類別包含多個 `Student` 實體。這裡的 many 推斷出 `ICollection` 類型屬性。
- Now, if the `Student` entity does not follow the Id property convention for foreign key, then we can specify the name of the foreign key using the `HasForeignKey` method.
  現在，如果 `Student` 實體不遵循外鍵的 Id 屬性慣例，那麼我們可以使用 `HasForeignKey` 方法指定外鍵的名稱。
- `.HasForeignKey<int>(s => s.CurrentGradeId);` specifies the foreign key property in the `Student` entity.
  `.HasForeignKey<int>(s => s.CurrentGradeId);` 指定 `Student` 實體中的外鍵屬性。

Alternatively, you can start configuring the relationship with the `Grade` entity instead of the `Student` entity. The following code produces the same result as above.
或者，您可以開始配置與 `Grade` 實體的關係，而不是與 `Student` 實體的關係。以下程式碼產生與上述相同的結果。

```csharp

modelBuilder.Entity<Grade>()
    .HasMany<Student>(g => g.Students)
    .WithRequired(s => s.CurrentGrade)
    .HasForeignKey<int>(s => s.CurrentGradeId);
```

The above example will create the following tables in the database.
上述範例將在資料庫中建立下表。

![[onetomany-4.png]]

### Configure the NotNull ForeignKey using Fluent API 使用 Fluent API 配置 NotNull ForeignKey

In convention 1, we have seen that it creates an optional one-to-many relationship which in turn creates a nullable foreign key column in the database. To make it a NotNull column, use the `HasRequired()` method as shown below.
在慣例 1 中，我們已經看到它建立了一個可選的一對多關係，該關係又在資料庫中建立了一個可空的外鍵列。若要使其成為 NotNull 列，請使用 `HasRequired()` 方法，如下所示。

```csharp

modelBuilder.Entity<Student>()
    .HasRequired<Grade>(s => s.CurrentGrade)
    .WithMany(g => g.Students);
```

### Configure Cascade Delete using Fluent API 使用 Fluent API 配置串聯刪除

Cascade delete means automatically deleting child rows when the related parent row is deleted. For example, if `Grade` is deleted then all the students in that `Grade` should also be deleted automatically. The following code configures the cascade delete using the `WillCascadeOnDelete` method.
串聯刪除是指當父行被刪除時，子行也會自動被刪除。例如，如果刪除了 `Grade` ，則該 `Grade` 的所有學生也應自動被刪除。以下程式碼使用 `WillCascadeOnDelete` 方法配置串聯刪除。

```csharp

modelBuilder.Entity<Grade>()
    .HasMany<Student>(g => g.Students)
    .WithRequired(s => s.CurrentGrade)
    .WillCascadeOnDelete();
```

---

# Configure Many-to-Many Relationships in Code-First 在 Code-First 中配置多對多關係

Here, we will learn how to configure a Many-to-Many relationship between the `Student` and `Course` entity classes. `Student` can join multiple courses and multiple students can join one `Course`.
在這裡，我們將學習如何在 `Student` 和 `Course` 實體類別之間配置多對多關係。 `Student` 可以加入多門課程，多個學生也可以加入一 `Course` 。

## Many-to-Many Relationship by Following Conventions 遵循慣例的多對多關係

EF 6 includes default conventions for many-to-many relationships. ==You need to include a collection navigation property at both ends.== For example, the `Student` class should have a collection navigation property of `Course` type, and the `Course` class should have a collection navigation property of `Student` type to create a many-to-many relationship between them without any configuration, as shown below:
EF 6 包含多對多關係的預設慣例。==您需要在兩端都包含一個集合導航屬性。==例如， `Student` 類別應該具有 `Course` 類型的集合導航屬性， `Course` 類別也應該具有 `Student` 類型的集合導航屬性，這樣無需任何配置即可在它們之間建立多對多關係，如下所示：

```csharp
public class Student
{
    public Student()
    {
        this.Courses = new HashSet<Course>();
    }

    public int StudentId { get; set; }
    [Required]
    public string StudentName { get; set; }

    public virtual ICollection<Course> Courses { get; set; }
}

public class Course
{
    public Course()
    {
        this.Students = new HashSet<Student>();
    }

    public int CourseId { get; set; }
    public string CourseName { get; set; }

    public virtual ICollection<Student> Students { get; set; }
}
```

The following is the context class that includes the `Student` and `Course` entities.
以下是包含 `Student` 和 `Course` 實體的上下文類別。

```csharp
public class SchoolDBContext : DbContext
{
    public SchoolDBContext() : base("SchoolDB-DataAnnotations")
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
        
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
    }
}
```

EF API will create `Students`, `Courses` and also the joining table `StudentCourses` in the database for the above example. The `StudentCourses` table will include the PK (Primary Key) of both tables - `Student_StudentId` & `Course_CourseId`, as shown below.
EF API 將在資料庫中建立 `Students` 、 `Courses` 以及連接表 `StudentCourses` ，用於上述範例。 `StudentCourses` 表將包含兩個表的主鍵 (PK) - `Student_StudentId` 和 `Course_CourseId` ，如下所示。

![[manytomany-fg.png]]

**Note:** EF automatically creates a joining table with the name of both entities and the suffix 's'.
**注意:** EF 會自動建立一個連接表，其中包含兩個實體的名稱和後綴「s」。

## Configure a Many-to-Many Relationship using Fluent API 使用 Fluent API 配置多對多關係

As you have seen above, the default conventions for a many-to-many relationship creates a joining table with the default naming conventions. Use Fluent API to customize a joining table name and column names, as shown below:
如上所示，多對多關係的預設慣例會建立一個具有預設命名慣例的連接表。使用 Fluent API 可以自訂連線表名和列名，如下所示：

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
                .HasMany<Course>(s => s.Courses)
                .WithMany(c => c.Students)
                .Map(cs =>
					{
						cs.MapLeftKey("StudentRefId");
						cs.MapRightKey("CourseRefId");
						cs.ToTable("StudentCourse");
					});
}
```

In the above example, the `HasMany()` and `WithMany()` methods are used to configure a many-to-many relationship between the `Student` and `Course` entities. The `Map()` method takes an Action type delegate, hence, we can pass the lambda expression to customize column names in a joining table. We can specify the PK property name of `Student` in `MapLeftKey()` (we started with the `Student` entity, so it will be the left table) and the PK of the `Course` table in `MapRightKey()` method. The `ToTable()` method specifies the name of a joining table (`StudentCourse` in this case).
在上面的範例中， `HasMany()` 和 `WithMany()` 方法用來配置 `Student` 和 `Course` 實體之間的多對多關係。 Map `Map()` 方法接受一個 Action 類型的委託，因此我們可以傳遞 lambda 表達式來自訂連接表中的列名。我們可以在 `MapLeftKey()` 中指定 `Student` 的主鍵屬性名稱（我們從 `Student` 實體開始，因此它是左表），並在 `MapRightKey()` 方法中指定 `Course` 表的主鍵。 ToTable `ToTable()` 方法指定連接表的名稱（在本例中為 `StudentCourse` ）。

The above code will create a joining table `StudentCourse` with two Primary Keys `StudentRefId` and `CourseRefId` which will also be Foreign Keys, as shown below:
上述程式碼將建立一個連接表 `StudentCourse` ，它具有兩個主鍵 `StudentRefId` 和 `CourseRefId` ，它們也是外鍵，如下所示：

![[manytomany-fg2.png]]

In this way, you can override the default conventions for a many-to-many relationship and customize a joining table name and its columns.
透過這種方式，您可以覆寫多對多關係的預設慣例，並自訂連線表名及其列。
