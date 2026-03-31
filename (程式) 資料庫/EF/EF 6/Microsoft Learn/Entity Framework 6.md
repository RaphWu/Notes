---
aliases:
date: 2020-10-15
update:
author: Microsoft
language: C#
sourceurl: https://learn.microsoft.com/en-us/ef/ef6/
tags:
  - CSharp
  - 資料庫
  - EF6
---

# Entity Framework 6

Entity Framework 6 (EF6) is a tried and tested object-relational mapper (O/RM) for .NET with many years of feature development and stabilization.
Entity Framework 6 (EF6) 是一個經過驗證的 .NET 物件關係映射器 (O/RM)，擁有多年功能開發和穩定化。

As an O/RM, EF6 reduces the impedance mismatch between the relational and object-oriented worlds, enabling developers to write applications that interact with data stored in relational databases using strongly-typed .NET objects that represent the application's domain, and eliminating the need for a large portion of the data access "plumbing" code that they usually need to write.
作為一個 O/RM，EF6 減少了關係模型和物件導向世界之間的阻抗不匹配，使開發者能夠使用代表應用程式領域的強型別 .NET 物件來編寫應用程式，與關係型資料庫中的資料互動，並消除了他們通常需要撰寫的大量資料存取「管線」程式碼。

EF6 implements many popular O/RM features:

- Mapping of [POCO](https://learn.microsoft.com/en-us/ef/ef6/resources/glossary#poco) entity classes which do not depend on any EF types
- Automatic change tracking
- Identity resolution and Unit of Work
- Eager, lazy and explicit loading
- Translation of strongly-typed queries using [LINQ (Language INtegrated Query)](https://aka.ms/AA6hsvu)
- Rich mapping capabilities, including support for:
    - One-to-one, one-to-many and many-to-many relationships
    - Inheritance (table per hierarchy, table per type and table per concrete class)
    - Complex types
    - Stored procedures
- A visual designer to create entity models.
- A "Code First" experience to create entity models by writing code.
- Models can either be generated from existing databases and then hand-edited, or they can be created from scratch and then used to generate new databases.
- Integration with .NET Framework application models, including ASP.NET, and through databinding, with WPF and WinForms.
- Database connectivity based on ADO.NET and numerous [providers](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/providers/) available to connect to SQL Server, Oracle, MySQL, SQLite, PostgreSQL, DB2, etc.

EF6 實現了許多流行的 O/RM 功能：

- 對映不依賴任何 EF 類型的 [POCO](https://learn.microsoft.com/en-us/ef/ef6/resources/glossary#poco) 實體類
- 自動變更追蹤
- 身份解析和工作單元
- 即時載入、延遲載入和明確載入
- 使用 [LINQ（語言整合查詢）](https://aka.ms/AA6hsvu) 轉換強型別查詢
- 豐富的映射功能，包括支援：
	- 一對一、一對多和多對多關係
	- 繼承（每個層次結構一個表、每個類型一個表和每個具體類別一個表）
	- 複雜類型
	- 儲存程序
- 用於建立實體模型的視覺化設計器。
- 「程式碼優先」體驗，透過編寫程式碼建立實體模型。
- 模型可以從現有資料庫產生並手動編輯，也可以從頭建立並用於產生新的資料庫。
- 與 .NET Framework 應用程式模型（包括 ASP.NET）集成，並透過資料綁定與 WPF 和 WinForms 集成。
- 基於 ADO.NET 和眾多 [提供者](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/providers/) 的資料庫連接，可連接到 SQL Server、Oracle、MySQL、SQLite、PostgreSQL、DB2 等。

# Should I use EF6 or EF Core? 我應該使用 EF6 還是 EF Core？

EF Core is a more modern, lightweight and extensible version of Entity Framework that has very similar capabilities and benefits to EF6. EF Core is a complete rewrite and contains many new features not available in EF6, although it also still lacks some of the most advanced mapping capabilities of EF6. Consider using EF Core in new applications if the feature set matches your requirements. [Compare EF Core & EF6](https://learn.microsoft.com/en-us/ef/efcore-and-ef6/) examines this choice in greater detail.
EF Core 是一個更現代化、輕量級且可擴充的 Entity Framework 版本，其功能與優勢與 EF6 非常相似。EF Core 是完全重寫的，包含許多在 EF6 中不存在的全新功能，儘管它仍然缺少一些 EF6 最先進的映射能力。如果功能集符合您的需求，請考慮在新的應用程式中使用 EF Core。Compare EF Core & EF6 更詳細地探討了這個選擇。

# [Get Started  開始使用](https://learn.microsoft.com/en-us/ef/ef6/get-started)

Add the EntityFramework NuGet package to your project or install the [Entity Framework Tools for Visual Studio](https://aka.ms/AA6i8c5). Then watch videos, read tutorials, and advanced documentation to help you make the most of EF6.
將 EntityFramework NuGet 套件添加到您的專案中，或安裝 Visual Studio 的 Entity Framework 工具。然後觀看影片、閱讀教學和進階文件，以幫助您充分利用 EF6。

# Past Entity Framework Versions

過往的 Entity Framework 版本

This is the documentation for the latest version of Entity Framework 6, although much of it also applies to past releases. Check out [What's New](https://learn.microsoft.com/en-us/ef/ef6/what-is-new/) and [Past Releases](https://learn.microsoft.com/en-us/ef/ef6/what-is-new/past-releases) for a complete list of EF releases and the features they introduced.
這是 Entity Framework 6 最新版本的文件，雖然其中大部分內容也適用於過往版本。請參考「新功能」和「過往版本」以獲取 EF 版本發布的完整清單及其引進的功能。
