---
aliases:
date: 2023-03-22
update:
author: 馬爾斯的Blog
language: C#
sourceurl: https://mars23003.github.io/blog/posts/csharp/efcore/eager-lazy-explicitloading/
tags:
  - 資料庫
  - EntityFramework
  - EF_Lazy_Loading
---

# [C#] 掌握 EF Core 的載入策略：Eager Loading、Lazy Loading 和 Explicit Loading

本文深入探討了 EF Core 中的 Eager Loading、Lazy Loading 和 Explicit Loading 等不同的載入策略。我們探討了這些載入策略的優缺點，以及如何使用它們來優化您的資料庫查詢。此外，我們還提供了實用的範例來演示每種載入策略如何在 EF Core 中實現。

## Eager Loading

### 概念說明

在 Entity Framework Core 中，如果要將關聯資料載入查詢結果，可以使用 Eager Loading，它是將所有相關資料都載入記憶體中的方式。Eager Loading 可以使用 Include 方法來載入關聯資料。

以下是 Eager Loading 的基本語法：

```csharp
var orders = _context.Orders.Include(o => o.OrderDetails).ToList();
```

這個查詢將會載入 Orders 和相關的 OrderDetails 資料。

### 範例說明

以下是 Microsoft 官方文件中有關 Eager Loading 的範例說明：

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Author)
        .ToList();
}
```

這個範例會載入 Blogs、Blogs 的所有 Posts，以及每個 Post 的 Author 資料。

## Lazy Loading

### 概念說明

在 Entity Framework Core 中，如果要使用 Lazy Loading，必須先啟用它。它是在需要時才載入相關資料的方式。可以使用 virtual 關鍵字來啟用 Lazy Loading。此外，必須在 DbContext 的 OnConfiguring 方法中啟用 Lazy Loading。

以下是啟用 Lazy Loading 的基本語法：

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseLazyLoadingProxies()
        .UseSqlServer(myConnectionString);
```

使用 Lazy Loading 的時候，可以在需要時載入相關資料：

```csharp
var orders = _context.Orders.ToList();

foreach (var order in orders)
{
    var orderDetails = order.OrderDetails.ToList();
}
```

### 範例說明

以下是 Microsoft 官方文件中有關 Lazy Loading 的範例說明：

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs.ToList();

    foreach (var blog in blogs)
    {
        // The Posts property will be lazily loaded here.
        if (blog.Posts.Count == 0)
        {
            blog.Posts.Add(
                new Post { Title = "Test Post", Content = "Test Content" });
        }
    }

    context.SaveChanges();
}
```

這個範例會載入 Blogs 資料，然後在需要時載入 Posts 資料。

## Explicit Loading

### 概念說明

Explicit Loading 是一種導入關聯實體的方式，透過 Load() 方法從 DbContext 實例中明確的導入特定的關聯實體，此時可以選擇一次導入一個或多個關聯實體。Explicit Loading 通常用於需要導入單一實體的特定資訊，並且應用程式的需求比 Eager Loading 更為嚴格。

### 範例

以下是 Explicit Loading 的範例程式碼，採用 Entity Framework Core 6 的版本：

```csharp
using (var context = new MyContext())
{
    var order = context.Orders.Find(1);

    // Load related OrderDetails for an order manually
    context.Entry(order)
        .Collection(b => b.OrderDetails)
        .Load();

    // Load related Product and Category for each OrderDetail
    foreach (var orderDetail in order.OrderDetails)
    {
        context.Entry(orderDetail)
            .Reference(b => b.Product)
            .Load();

        context.Entry(orderDetail.Product)
            .Reference(b => b.Category)
            .Load();
    }
}
```

在這個範例中，透過 Find() 方法取得 OrderId 為 1 的訂單，然後透過 Collection() 方法將關聯的 OrderDetails 導入，最後使用迴圈分別從每個 OrderDetail 實體中導入 Product 和 Category 實體。

該篇參考官方 [Loading Related Data](https://learn.microsoft.com/zh-tw/ef/core/querying/related-data/)
