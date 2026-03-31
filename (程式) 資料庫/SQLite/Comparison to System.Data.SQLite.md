---
aliases:
date: 2021-10-27
update:
author:
language: C#
sourceurl: https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/compare
tags:
  - CSharp
  - 資料庫
  - SQLite
---

# Comparison to System.Data.SQLite

與 System.Data.SQLite 的比較

- 10/27/2021

In 2005, Robert Simpson created System.Data.SQLite, a SQLite provider for ADO.NET 2.0. In 2010, the SQLite team took over maintenance and development of the project. It's also worth noting that the Mono team forked the code in 2007 as Mono.Data.Sqlite. System.Data.SQLite has a long history and has evolved into a stable and full-featured ADO.NET provider complete with Visual Studio tooling. New releases continue to ship assemblies compatible with every version of .NET Framework back to version 2.0 and even .NET Compact Framework 3.5.
在 2005 年，Robert Simpson 創建了 System.Data.SQLite，這是一個用於 ADO.NET 2.0 的 SQLite 傳遞器。在 2010 年，SQLite 團隊接管了此專案的上傳與開發。還值得注意的是，Mono 團隊在 2007 年將程式碼 fork 為 Mono.Data.Sqlite。System.Data.SQLite 拥有悠久的歷史，並演變成一個穩定且功能完整的 ADO.NET 傳遞器，包含 Visual Studio 工具。新的發行版本持續發布與每一個 .NET Framework 版本相容的組合，甚至包括 .NET Compact Framework 3.5。

The first version of .NET Core (released in 2016) was a single, lightweight, modern, and cross-platform implementation of .NET. Obsolete APIs and APIs with more modern alternatives were intentionally removed. ADO.NET didn't include any of the DataSet APIs (including DataTable and DataAdapter).
.NET Core 的第一版（於 2016 年發布）是單一、輕量級、現代且跨平台的 .NET 實現。過時的 API 和具有更現代替代方案的 API 都被有意移除。ADO.NET 沒有包含任何 DataSet API（包括 DataTable 和 DataAdapter）。

The Entity Framework team was somewhat familiar with the System.Data.SQLite codebase. Brice Lambson, a former member of the EF team, had previously helped the SQLite team add support for Entity Framework versions 5 and 6. Brice was also experimenting with his own implementation of a SQLite ADO.NET provider around the same time that .NET Core was being planned. After a long discussion, the Entity Framework team decided to create Microsoft.Data.Sqlite based on Brice's prototype. This would allow them to create a new lightweight and modern implementation that would align with the goals of .NET Core.
Entity Framework 團隊對 System.Data.SQLite 程式碼庫頗為熟悉。 EF 團隊前成員 Brice Lambson 曾經協助 SQLite 團隊新增對 Entity Framework 5 和 6 版本的支援。在 .NET Core 規劃期間，Brice 也嘗試自行實作一個 SQLite ADO.NET 提供者。經過長時間的討論，Entity Framework 團隊決定基於 Brice 的原型創建 Microsoft.Data.Sqlite。這將使他們能夠創建一個新的輕量級、現代化的實現，並與 .NET Core 的目標保持一致。

As an example of what we mean by more modern, here is code to create a [user-defined function](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/user-defined-functions) in both System.Data.SQLite and Microsoft.Data.Sqlite.
作為我們所謂的更現代的一個例子，這裡是 System.Data.SQLite 和 Microsoft.Data.Sqlite 中創建自定義函數的代碼。

```csharp title=""
// System.Data.SQLite
connection.BindFunction(
    new SQLiteFunctionAttribute("ceiling", 1, FunctionType.Scalar),
    (Func<object[], object>)((object[] args) => Math.Ceiling((double)((object[])args[1])[0])), null);

// Microsoft.Data.Sqlite
connection.CreateFunction(
    "ceiling",
    (double arg) => Math.Ceiling(arg));
```

In 2017, .NET Core 2.0 experienced a change in strategy. It was decided that compatibility with .NET Framework was vital to the success of .NET Core. Many of the removed APIs, including the DataSet APIs, were added back. Like it did for many others, this unblocked System.Data.SQLite allowing it to also be ported to .NET Core. The original goal of Microsoft.Data.Sqlite to be lightweight and modern, however, still remains. See [ADO.NET limitations](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/adonet-limitations) for details about ADO.NET APIs not implemented by Microsoft.Data.Sqlite.
在 2017 年，.NET Core 2.0 的策略發生了變更。決定 .NET Framework 的相容性對於 .NET Core 的成功至關重要。許多被移除的 API，包括 DataSet API，都重新被加入。就像它對許多人一樣，這解除了 System.Data.SQLite 的限制，使其也能被移植到 .NET Core。然而，Microsoft.Data.Sqlite 原本輕量級和現代的目標仍然保持。詳情請參考 ADO.NET 的限制，了解 Microsoft.Data.Sqlite 未實現的 ADO.NET API。

When new features are added to Microsoft.Data.Sqlite, the design of System.Data.SQLite is taken into account. We try, when possible, to minimize changes between the two to ease transitioning between them.
當向 Microsoft.Data.Sqlite 添加新功能時，會考慮 System.Data.SQLite 的設計。在可能的情況下，我們盡力減少兩者之間的變化，以方便在它們之間過渡。

## Data types  數據類型

==The biggest difference between Microsoft.Data.Sqlite and System.Data.SQLite is how data types are handled.== As described in [Data types](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/types), Microsoft.Data.Sqlite doesn't try to hide the underlying quirkiness of SQLite, which allows any arbitrary string to be specified as the column type, and only has four primitive types: INTEGER, REAL, TEXT, and BLOB.
==Microsoft.Data.Sqlite 和 System.Data.SQLite 之間最大的差異在於如何處理數據類型。==如 Data types 中所述，Microsoft.Data.Sqlite 不嘗試隱藏 SQLite 的底層奇特之處，它允許將任意字符串指定為列類型，僅有四種基本類型：INTEGER、REAL、TEXT 和 BLOB。

System.Data.SQLite applies additional semantics to column types mapping them directly to .NET types. This gives the provider a more strongly typed feel, but it has some rough edges. For example, they had to introduce a new SQL statement (TYPES) to specify the column types of expressions in SELECT statements.
System.Data.SQLite 對欄位類型應用額外的語義，將其直接映射到 .NET 類型。這讓提供者有更強類型的感覺，但它有一些粗糙的邊緣。例如，他們必須引入一個新的 SQL 語句（TYPES）來指定 SELECT 語句中表達式的欄位類型。

## Connection strings  連接字串

Microsoft.Data.Sqlite has a lot fewer [connection string](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/connection-strings) keywords. The following table shows alternatives that can be used instead.
Microsoft.Data.Sqlite 的連接字串關鍵字相對較少。下表顯示了可以使用的替代方案。

| Keyword          | Alternative                            |
| ---------------- | -------------------------------------- |
| Cache Size       | Send `PRAGMA cache_size = <pages>`     |
| FailIfMissing    | Use `Mode=ReadWrite`                   |
| FullUri          | Use the Data Source keyword            |
| Journal Mode     | Send `PRAGMA journal_mode = <mode>`    |
| Legacy Format    | Send `PRAGMA legacy_file_format = 1`   |
| Max Page Count   | Send `PRAGMA max_page_count = <pages>` |
| Page Size        | Send `PRAGMA page_size = <bytes>`      |
| Read Only        | Use `Mode=ReadOnly`                    |
| Synchronous      | Send `PRAGMA synchronous = <mode>`     |
| Uri              | Use the Data Source keyword            |
| UseUTF16Encoding | Send `PRAGMA encoding = 'UTF-16'`      |

## Authorization  授權

Microsoft.Data.Sqlite doesn't have any API exposing SQLite's authorization callback. Use issue [#13835](https://github.com/dotnet/efcore/issues/13835) to provide feedback about this feature.
Microsoft.Data.Sqlite 沒有任何 API 會公開 SQLite 的授權回調。請使用 issue #13835 來提供此功能的回饋。

## Data change notifications 資料變更通知

Microsoft.Data.Sqlite doesn't have any API exposing SQLite's data change notifications. Use issue [#13827](https://github.com/dotnet/efcore/issues/13827) to provide feedback about this feature.
Microsoft.Data.Sqlite 沒有任何 API 會公開 SQLite 的資料變更通知。請使用 issue #13827 來提供此功能的回饋。

## Virtual table modules  虛擬表格模組

Microsoft.Data.Sqlite doesn't have any API for creating virtual table modules. Use issue [#13823](https://github.com/dotnet/efcore/issues/13823) to provide feedback about this feature.
Microsoft.Data.Sqlite 沒有任何用於建立虛擬表格模組的 API。請使用 issue #13823 來提供此功能的回饋。

## See also  另請參閱

- [Data types   資料類型](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/types)
- [Connection strings   連接字串](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/connection-strings)
- [Encryption  加密](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/encryption)
- [ADO.NET limitations  ADO.NET 限制](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/adonet-limitations)
- [Dapper limitations  Dapper 限制](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/dapper-limitations)
