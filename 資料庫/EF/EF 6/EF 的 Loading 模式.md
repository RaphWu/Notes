---
aliases:
date: 2016-02-24
update:
author: VITO
language: C#
sourceurl: https://vito-note.blogspot.com/2016/02/ef-loading.html
tags:
---

# EF 的 Loading 模式

當使用 EF 的 ORM 架構來存取資料時，EF 預設會自動依據你的物件名稱自動去資料提取資料，並將資料放在記憶體中進行追蹤。 而這個自動提取資料的模式，可區分成：延遲載入、預先載入、明確載入。 不同的載入方法各有其優缺點，必須依使用的情境才能決定好壞。

![[ef-loading-02.png]]

![[ef-loading-01.png]]

# 導覽屬性

當資料庫中若有設定資料表關連性，在 ORM 過程產生 POCO 時，資料表的欄位會轉化成類別的屬性，如果是設定關連至其他資料表的欄位，則會轉化成**導覽屬性**。 導覽屬性分成：集合導覽屬性、參考導覽屬性。 EF 就是透過導覽屬性才可以自動幫我們載入相關資料。 反之若要由 Code First 建立 POCO ，你必須知道導覽屬性與其它屬性的差異。

- POCO 類別必須是 `public` ，而且不可以 `sealed` 。
- 導覽屬性必須是 `virtual` （VB 中的 `overridable` ）。

例如：在底下例子中， OrderDetails 是一個**集合導覽屬性**，它的資料參考自 OrderDetails 資料表，是一個 OrderDetail 集合型態。 而 Employee 是一個**參考導覽屬性**，它的資料參考自 Employee 資料表，是一個 Employee 型態。

```csharp
public partial class Order
{
    public Order()
    {
        this.OrderDetails = new HashSet<OrderDetail>();
    }

    public int OrderID { get; set; }
    public string CustomerID { get; set; }
    public Nullable<int> EmployeeID { get; set; }

    public virtual Employee Employee { get; set; }
    public virtual ICollection<OrderDetail> OrderDetails { get; set; }
}
```

# 延遲載入（Lazy Loading, 消極式載入）

在預設情況下，EF 會使用延遲載入方式來提取資料，也就是 EF 不會一開始就將相關連的資料一次由資料庫端提取，而是等需要用到的時候去才會提取。 例如下面這個例子，當使用 Order 這個 Entity 時，若有使用到 OrderDetails 和 Product 物件的屬性時才會自動去讀取資料庫中的相關資料。

底下這段程式碼會對資料庫進行 16 （1+5+10）次存取，因為它每次只提取必需使用到的資料。（資料庫內容如文章上頭的圖片內容）

```csharp
var query = context.Orders.Where(o => o.CustomerID == "VINET"); //+5
foreach (var order in query)
{
    foreach (var detail in order.OrderDetails)
    {
        Console.WriteLine("OrderID:{0} ProductID:{1} ProductName:{2}",
                          detail.OrderID,
                          detail.ProductID,
                          detail.Product.ProductName);
    }
}
```

產生的 SQL 語法

```sql
SELECT ... FROM [dbo].[Orders] AS [Extent1] WHERE N'VINET' = [Extent1].[CustomerID]
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE [Extent1].[OrderID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=10248
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=11
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=42
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=72
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE [Extent1].[OrderID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=10274
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=71
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=72
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE [Extent1].[OrderID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=10295
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=56
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE [Extent1].[OrderID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=10737
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=13
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=41
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE [Extent1].[OrderID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=10739
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=36
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=52
```

## 關閉延遲載入

預設 DBContext 的載入方式是使用延遲載入，你可以修改 Entity Model 屬性，將延遲載入關閉。

![[ef-loading-03.png]]

或者在程式碼中關閉延遲載入。（Configuration 屬性是 EF4.1 以上版本中的 DBContext 物件屬性。使用時必須加入 [System.Data.Entity](http://msdn.microsoft.com/zh-tw/library/system.data.entity.aspx) 參考）

```csharp
using System.Data.Entity;
    using (TestDB1Entities context = new TestDB1Entities())
    {
        context.Configuration.LazyLoadingEnabled = false;
        ......
    }
```

# 預先載入（Eager Loading, 積極式載入）

當你關閉延遲載入再執行上述程式碼時，你會發現 order.OrderDetails 將會是一個空集合，因為 EF 將不會再自動的幫你提取需要使用到的關連資料。 這時候你可以採用預先載入，利用 [Include](https://msdn.microsoft.com/zh-tw/library/gg696785\(v=vs.113\).aspx) 方法，在一次查詢中就載入相關連的資料。

```csharp
context.Configuration.LazyLoadingEnabled = false;

    var query = context.Orders
        .Include(o => o.OrderDetails)
        .Include(o => o.Customer)
        .Where(o => o.CustomerID == "VINET");

    foreach (var order in query)
    {
        foreach (var detail in order.OrderDetails)
        {
            Console.WriteLine("OrderID:{0} ProductID:{1} ProductName:{2}", detail.OrderID, detail.ProductID, detail.Product.ProductName);
        }
    }
```

上面程式碼只會對資料庫進行 1 次存取，也就是在一開始時就將 Customer 和 OrderDetails 這二個相關的導覽資料都一併 Join 出來。 不過這個例子在最後的 foreache 中，我們要提取 Product.ProductName 資訊，可是因為我們關閉了延遲載入，所以系統不會自動載入資料，在這裡會產生失敗。

## 預先載入多層次的關連

若要預先載入多層次的關連，只要在 [Include](https://msdn.microsoft.com/zh-tw/library/gg696785\(v=vs.113\).aspx) 中使用「.」表示它們的關連即可。 如果多層次中的屬性是一個集合屬性，就必須使用 Select() 明確指出要延申山去的關連屬性。

```csharp
context.Configuration.LazyLoadingEnabled = false;

    var query = context.Orders
        .Include(o => o.Customer)
        .Include(o => o.OrderDetails.Select(od => od.Product))
        .Where(o => o.CustomerID == "VINET");

    Console.WriteLine(context.OrderDetails.Local.Count);  //0

    foreach (var order in query)
    {
        foreach (var detail in order.OrderDetails)
        {
            Console.WriteLine("OrderID:{0} ProductID:{1} ProductName:{2}", detail.OrderID, detail.ProductID, detail.Product.ProductName);
        }
    }
```

產生的 SQL 語法

```sql
SELECT
...
FROM (
	SELECT
	...
	FROM [dbo].[Orders] AS [Extent1]
	LEFT OUTER JOIN [dbo].[Customers] AS [Extent2] ON [Extent1].[CustomerID] = [Extent2].[CustomerID]
	LEFT OUTER JOIN (
		SELECT [Extent3].[OrderID] AS [OrderID], [Extent3].[ProductID] AS [ProductID1], [Extent3].[UnitPrice] AS [UnitPrice1], [Extent3].[Quantity] AS [Quantity], [Extent3].[Discount] AS [Discount], [Extent4].[ProductID] AS [ProductID2], [Extent4].[ProductName] AS [ProductName], [Extent4].[SupplierID] AS [SupplierID], [Extent4].[CategoryID] AS [CategoryID], [Extent4].[QuantityPerUnit] AS [QuantityPerUnit], [Extent4].[UnitPrice] AS [UnitPrice2], [Extent4].[UnitsInStock] AS [UnitsInStock], [Extent4].[UnitsOnOrder] AS [UnitsOnOrder], [Extent4].[ReorderLevel] AS [ReorderLevel], [Extent4].[Discontinued] AS [Discontinued]
		FROM [dbo].[OrderDetails] AS [Extent3]
		INNER JOIN [dbo].[Products] AS [Extent4] ON [Extent3].[ProductID] = [Extent4].[ProductID] ) AS [Join2] ON [Extent1].[OrderID] = [Join2].[OrderID]
		WHERE N'VINET' = [Extent1].[CustomerID]
) AS [Project1]
ORDER BY [Project1].[OrderID] ASC, [Project1].[CustomerID1] ASC, [Project1].[C1] ASC
```

# 明確載入（Explicit Loading）

使用**延遲載入**時，每次對資料庫只會提取少量資料，但必須來回多次。 而使用**預先載入**則是一次提取全部資料，雖然大量減少了來回提取資料的次數，但提取全部資料有時候可能是效能的殺手。 所以還有一種載入方式就是**明確載入**，它在程式碼中依實際的需求載入需要的資料。

**明確載入**原則上會同延遲載入一樣，分次取得資料，但是常使用在以下情況：

- 載入非導覽屬性的關連資料，也就是說，你要取得沒有設定資料庫關連的關連資料。
- 你不想取得所有集合導覽屬性的所有關連資料，你可以利用明確載入來篩選過濾。

要使用**明確載入**，必須先透過 [DbContext.Entry](http://msdn.microsoft.com/zh-tw/library/system.data.entity.dbcontext.entry.aspx) 方法取得物件實體，有了物件實體就可以取得該實體的相關資訊，例如 [State](http://msdn.microsoft.com/zh-tw/library/system.data.entity.infrastructure.dbentityentry.state.aspx) 、 [CurrentValues](http://msdn.microsoft.com/zh-tw/library/system.data.entity.infrastructure.dbentityentry.currentvalues.aspx) 、 [OriginalValues](http://msdn.microsoft.com/zh-tw/library/system.data.entity.infrastructure.dbentityentry.originalvalues.aspx) 等等。 同時也可以透過 [Reference](https://msdn.microsoft.com/zh-tw/library/gg696201.aspx) 和 [Collection](https://msdn.microsoft.com/zh-tw/library/gg679502.aspx) 來提取相關的導覽屬性。 例如底下例子分別取得參考導覽屬性（Product）和集合導覽屬性（OrderDetails）。

```csharp
using (var context = new NorthwindEntities())
{
    context.Configuration.LazyLoadingEnabled = false;

    var query = context.Orders
        .Where(o => o.CustomerID == "VINET");

    foreach (var order in query)
    {
        context.Entry(order)
            .Collection(x => x.OrderDetails)
            .Query()
            .Where(od => od.UnitPrice > 15)
            .Load();
        foreach (var detail in order.OrderDetails)
        {
            context.Entry(detail)
            .Reference(od => od.Product)
            .Load();
            Console.WriteLine("OrderID:{0} ProductID:{1} ProductName:{2}", detail.OrderID, detail.ProductID, detail.Product.ProductName);
        }
    }
}
```

上面例子中，使用 [Collection](https://msdn.microsoft.com/zh-tw/library/gg679502.aspx) 方法讀取導覽屬性時，再透過 [Query](https://msdn.microsoft.com/zh-tw/library/gg696278\(v=vs.113\).aspx) 方法取得此集合所使用的查詢，並使用 Where 方法來篩選資料。 這裡有一點要注意，這裡的 [Collection](https://msdn.microsoft.com/zh-tw/library/gg679502.aspx) 方法必須使用 lambda expression ，不能使用字串的型別參數才可以回傳泛型的列舉資料，後續才可以使用 Lambda 相關的語法進行操作。

```csharp
context.Entry(order)
       .Collection(o => o.OrderDetails)
       .Query()
       .Where(od => od.UnitPrice < 20)
       .Load();

    foreach (var detail in order.OrderDetails)
    {
        Console.WriteLine("{0} {1}", detail.ProductID, detail.UnitPrice);
    }
```

產生的 SQL 語法

```sql
SELECT ... FROM [dbo].[Orders] AS [Extent1] WHERE N'VINET' = [Extent1].[CustomerID]
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE ([Extent1].[OrderID] = @EntityKeyValue1) AND ([Extent1].[UnitPrice] > cast(15 as decimal(18)))',N'@EntityKeyValue1 int',@EntityKeyValue1=10248
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=72
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE ([Extent1].[OrderID] = @EntityKeyValue1) AND ([Extent1].[UnitPrice] > cast(15 as decimal(18)))',N'@EntityKeyValue1 int',@EntityKeyValue1=10274
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=71
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=72
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE ([Extent1].[OrderID] = @EntityKeyValue1) AND ([Extent1].[UnitPrice] > cast(15 as decimal(18)))',N'@EntityKeyValue1 int',@EntityKeyValue1=10295
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=56
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE ([Extent1].[OrderID] = @EntityKeyValue1) AND ([Extent1].[UnitPrice] > cast(15 as decimal(18)))',N'@EntityKeyValue1 int',@EntityKeyValue1=10737
exec sp_executesql N'SELECT ... FROM [dbo].[OrderDetails] AS [Extent1] WHERE ([Extent1].[OrderID] = @EntityKeyValue1) AND ([Extent1].[UnitPrice] > cast(15 as decimal(18)))',N'@EntityKeyValue1 int',@EntityKeyValue1=10739
exec sp_executesql N'SELECT ... FROM [dbo].[Products] AS [Extent1] WHERE [Extent1].[ProductID] = @EntityKeyValue1',N'@EntityKeyValue1 int',@EntityKeyValue1=36
```

參考連結

- [Loading Related Entities](https://msdn.microsoft.com/en-us/data/jj574232.aspx)
- [Entity Framework: Filtering Related Entity Collections](http://www.leftycoder.com/entity-framework-filtering-related-entity-collections/)
