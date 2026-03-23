---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 名詞

| 英文                                 | 中文     | 大陸     | 其他翻譯      |
| ---------------------------------- | ------ | ------ | --------- |
| Primary Key                        | 主鍵     |        |           |
| Foreign Key                        | 外鍵     |        |           |
| Principal Key                      |        |        |           |
| Alternative Key                    | 替代索引鍵  |        |           |
| Simple key                         |        | 单键     |           |
| Composite key                      |        | 组合键    |           |
| Concurrency<br>Concurrent          | 並行     | 並發     |           |
| Parallel                           | 平行     | 並行     |           |
| Transaction                        | 事務     |        |           |
| Convention                         | 慣例     | 约定、约束  |           |
| Migration                          | 移轉     |        | 遷移        |
| Scaffolding<br>Reverse Engineering | 逆向工程   |        | 反向工程、還原工程 |
| domain                             | 領域     |        | 網域、域      |
| Scalar Properties                  | 標量屬性   |        | 純量屬性、數值屬性 |
| Navigation Properties              | 導覽屬性   | 导航属性   |           |
| Reference Navigation Property      | 參考導覽屬性 | 引用导航属性 |           |
| Collection Navigation Property     | 集合導覽屬性 | 集合导航属性 |           |
| Inverse Navigation Property        | 反向導覽屬性 | 逆导航属性  |           |
| Shadow Properties                  | 隱藏屬性   | 影子属性   |           |
| Indexer Properties                 | 索引器屬性  |        |           |
| Backing Field                      | 支援欄位   |        |           |
| Value Conversion                   | 值轉換    |        |           |
| Data Seeding                       | 資料初始化  |        |           |
| Data Annotations                   | 資料註解   | 数据标注   |           |
| ComplexType                        | 複雜類型   |        |           |
| Association                        | 關聯     |        | 關係        |
| Cascade Delete                     | 串聯刪除   | 級聯刪除   |           |
| Relationship                       | 關聯     | 关系     |           |
| Constraint                         |        | 约束     |           |
| Join Table                         | 中介表    | 连接表    |           |
|                                    |        |        |           |

---

# Entity Framework 綜合

## Microsoft Ignite

- [# Databinding with WPF](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/databinding/wpf)
- [Databinding with WinForms](https://learn.microsoft.com/en-us/ef/ef6/fundamentals/databinding/winforms)

## 網路文章

- **重要** [Entity Framework Tutorial](https://www.entityframeworktutorial.net/)
- **重要** [ZZZ Projects](https://zzzprojects.com/)
	- [Entity Framework Core Tutorial](https://entityframeworkcore.com/)
	- [Entity Framework Tutorial](https://entityframework.net/)

## 網路文章

- [Category: EF - 黑暗執行緒](https://blog.darkthread.net/blog/category/EF)

---

# Entity Framework Core

## Microsoft Learn

- [Entity Framework 文件中樞](https://learn.microsoft.com/zh-tw/ef/)
- [Entity Framework Core](https://learn.microsoft.com/zh-tw/ef/core/)

## 網路文章

- [認識 Entity Framework Core 載入關聯資料的三種方法](https://blog.miniasp.com/post/2022/04/21/Loading-Related-Data-in-EF-Core)

---

# Entity Framework 6

## 網路文章

### Code-First

- [Entity Framework Code First And Migrations: Part Two](https://www.c-sharpcorner.com/article/entity-framework-code-first-and-migrations-part-two/)

---

# SQLite

- [官方網站](https://sqlite.org/)
- [GitHub](https://github.com/sqlite/sqlite)

## System.Data.SQLite

- [官方網站](https://system.data.sqlite.org/)

### 官方討論區

#### [System.Data.SQLite 2.0.1](https://system.data.sqlite.org/home/forumpost/2061f3498095e05b)

https://system.data.sqlite.org/home/forumpost/4653a372d4cefcb8
To use System.Data.SQLite 2.0 in your project, add these two nuget packages:
要在您的專案中使用 System.Data.SQLite 2.0，請加入這兩個 nuget 套件：

```csharp
<ItemGroup>
	<PackageReference id="System.Data.SQLite" version="2.0.1"/>
	<PackageReference id="SourceGear.sqlite3" version="3.50.3"/>
</ItemGroup>
```

The first one is System.Data.SQLite itself.
第一個是 System.Data.SQLite 本身。

The second one is package that contains the native SQLite builds. This is one of the major changes in SDS 2.x. The C# code and the C code are more separate than before. Instead of using C++/CLI to communicate from C# to C, DllImport (P/Invoke) is used, and the native (aka unmanaged, the stuff in C) code simply needs to be a regular SQLite shared library.
第二個是包含原生 SQLite 构建的套件。這是 SDS 2.x 中的主要變更之一。C# 程式碼和 C 程式碼比以前更分離。不再使用 C++/CLI 從 C# 到 C 的通訊，而是使用 DllImport (P/Invoke)，而原生 (又名未管理，C 中的內容) 程式碼只需要是一個普通的 SQLite 共享庫。

Updating to a new SQLite version no longer requires an update to SDS.
更新到新的 SQLite 版本不再需要更新 SDS。

https://system.data.sqlite.org/home/forumpost/2802f0935ff64976
System.Data.SQLite 2.x requires a regular SQLite shared library with "e_sqlite" as its base name. That means, for example, on Windows, the name would be "e_sqlite3.dll", and on Linux it would be "libe_sqlite3.so".
System.Data.SQLite 2.x 需要「e_sqlite」作為基本名稱的常態 SQLite 共享程式庫。這表示，例如，在 Windows 上，名稱會是「e_sqlite3.dll」，而在 Linux 上則是「libe_sqlite3.so」。

Unless you need a build which includes additional extensions compiled in, on Windows, you can simply download the standard sqlite3.dll (from https://sqlite.org/download.html) and rename it to e_sqlite3.dll.
除非您需要包含額外擴充功能編譯進去的建置版本，在 Windows 上，您可以簡單下載標準的 sqlite3.dll（來自 https://sqlite.org/download.html）並將其改名為 e_sqlite3.dll。

If you want to build your own SQLite library for use with SDS, simply follow whatever procedures you would normally use to build a shared library in C, and use e_sqlite3 as the base name of the linker output.
如果您想為使用於 SDS 而建置自己的 SQLite 程式庫，只需按照通常在 C 中建置共享程式庫的步驟來進行，並將鏈接器輸出的基本名稱使用為 e_sqlite3。

For convenience, various nuget packages are available. The following packages contain SQLite builds with the necessary "e_sqlite3" base name:
為了方便起見，有多種 nuget 套件可供使用。下列套件包含具有必要「e_sqlite3」基本名稱的 SQLite 建置版本：

- SourceGear.sqlite3 -- a basic build with commonly-used compile options
  SourceGear.sqlite3 -- 一個基本的建置，包含常見的編譯選項
- SourceGear.sqlite3.ext -- this build contains the extensions which were integrated into System.Data.SQLite 1.x (vtshim, zipfile, sha1, regexp, etc)
  SourceGear.sqlite3.ext -- 這個建置包含已整合到 System.Data.SQLite 1.x 中的擴充功能 (vtshim, zipfile, sha1, regexp 等)

## SQLite CodeFirst

- [GitHub](https://github.com/msallin/SQLiteCodeFirst)

### 網路文章

- [使用EF6连接Sqlite](https://www.cnblogs.com/guchen33/p/18518933)

---

# Browser

## DbGate

- [官方網站](https://dbgate.org/)
- [GitHub](https://github.com/dbgate/dbgate)

## SQLite Expert Personal 5 - 64bit

- [官方網站](https://www.sqliteexpert.com/)

---

# T4 檔輔助

## T4Editor

- [GitHub](https://github.com/Tim-Maes/T4Editor)

## # T4 Language

- [GitHub](https://github.com/bricelam/T4Language)

---

# POCO 產生器

## EntityFramework Reverse POCO Generator

- [官方網站](https://www.reversepoco.co.uk/)
- [GitHub](https://github.com/sjh37/EntityFramework-Reverse-POCO-Code-First-Generator)
- [Documentation](https://github.com/sjh37/EntityFramework-Reverse-POCO-Code-First-Generator/wiki)
- [v2.37.5 Final revision of version 2](https://github.com/sjh37/EntityFramework-Reverse-POCO-Code-First-Generator/releases/tag/v2.37.5)
- [v3.1.3](https://github.com/sjh37/EntityFramework-Reverse-POCO-Code-First-Generator/releases/tag/v3.1.3)

gmail
Squ...@0422

對於擁有 .edu 或 .ac 電子郵件地址的學術界人士是免費的。如果沒有授權，V3.2.0 會自動切換到免費（有限）的試用版本。

## Entity Framework Visual Designer

Entity Framework visual design surface and code-first code generation for EF6, Core and beyond.
Entity Framework visual design 呈現與 code-first 產生，適用於 EF6、Core 及其後續版本。

- [官方網站](https://msawczyn.github.io/EFDesigner/)
- [GitHub](https://github.com/msawczyn/EFDesigner2022)
- [Documentation](https://msawczyn.github.io/EFDesigner/)
