---
aliases:
date:
update:
author:
language: CSharp
sourceurl:
tags:
  - SourceGrid
  - WinForm
---

# SourceGrid

Lib Version: 4.11
Doc Version: 2.0

[Download binary and source](http://sourceforge.net/projects/sourcegrid)
[Official Web Site](http://www.devage.com/)

![[SourceGrid_Overview.png|SourceGrid_Overview]]

# Introduction 介紹

SourceGrid is a .NET Windows Forms grid control written entirely in C# with managed code. SourceGrid can be used to visualize or to change data in a table format.
SourceGrid 是一個完全使用 C# 寫成的 .NET Windows Forms 網格控制項，採用受管代碼。SourceGrid 可用於視覺化或更改表格格式中的數據。

SourceGrid con be used bound to a data source (typically a DataView) or creating each cell directly.
SourceGrid 可以繫結到數據來源（通常是一個 DataView）或直接建立每一個單元格。

There are a lot of controls of this type available, but often are expensive, difficult to customize or too DataSet oriented.
有許多這類控制項可供選擇，但通常很昂貴、難以自訂或過於以 DataSet 為中心。

SourceGrid use only managed code (without API or Interop) and can be used with any .NET 2 compatible environments.
SourceGrid 僅使用受管代碼（無需 API 或 Interop），並可在任何與.NET 2 兼容的環境中使用。

In this article I want to supply an overview of the utilization and functionalities of the SourceGrid control. For details on the classes, properties or methods you can consult the documentation in CHM format or the example project in the ZIP file.
在本文中，我想提供 SourceGrid 控制項的應用和功能概述。有關類別、屬性或方法的詳細資訊，您可以參考 CHM 格式的文件或 ZIP 檔中的範例專案。

For more information, a discussion forums, bug tracker system or to download the latest release go to the SourceForge page: [http://sourceforge.net/projects/sourcegrid](http://sourceforge.net/projects/sourcegrid) or to my home page [http://www.devage.com/](http://www.devage.com/)
如需更多資訊，請前往討論區、錯誤追蹤系統下載最新釋出，請到 SourceForge 頁面：http://sourceforge.net/projects/sourcegrid 或我的個人網站 http://www.devage.com/

# Quick Start 快速啟動

## Installation 安裝

To use SourceGrid you must have a .NET 2 compatible development environment (like Visual Studio 2005).
要使用 SourceGrid，您必須擁有一個 .NET 2 兼容的開發環境（例如 Visual Studio 2005）。

Download the latest release from [http://sourceforge.net/projects/sourcegrid](http://sourceforge.net/projects/sourcegrid). Unzip the file and reference from your project these assemblies:
從 http://sourceforge.net/projects/sourcegrid 下載最新釋出。解壓縮檔案，並從您的專案參考這些組合：

- **SourceGrid.dll** - SourceGrid core library - SourceGrid 核心程式庫
- **DevAge.Core.dll** - Helper library for common features - 常見功能輔助程式庫
- **DevAge.Windows.Forms.dll** - Helper library for Windows Forms - Windows Forms 輔助程式庫
- *SourceGrid.Extensions.dll* - Optional library with some SourceGrid extensions like DataGrid, PlanningGrid - 可選程式庫，包含一些 SourceGrid 扩展，如 DataGrid、PlanningGrid

Typically I suggest to always copy in the same location the \*.xml files that you can find in the same directory of the assemblies to use the IDE IntelliSense features.
通常我建議您將 \*.xml 檔案複製到與組合（assemblies）相同目錄的位置，以便使用 IDE 的 IntelliSense 功能。

Open the form where you want to add the grid control, open the IDE ToolBox, right-click and select "Choose Items...". Browse and add SourceGrid.dll and SourceGrid.Extensions.dll assemblies on the IDE ToolBox.
打開您想添加網格控制的表單，打開 IDE 工具箱，右鍵點擊並選擇「選擇項目...」。瀏覽並將 SourceGrid.dll 和 SourceGrid.Extensions.dll 組合添加到 IDE 工具箱。

These assemblies are the same assemblies required by the runtime that you must redistribute with your application to the end-user.
這些組合與運行時所需的組合相同，您必須將其與您的應用程序一起重新分發給最終用戶。

## SourceGrid controls 控制項

There are 2 main controls inside the SourceGrid.dll assembly:
SourceGrid.dll 組件內有兩個主要控制項：

- `GridVirtual` - A grid of virtual cells `(ICellVirtual)` - 一個虛擬單元格的網格。
- `Grid` - A grid of real cells `(ICell)` - 一個實際單元格的網格。

There are therefore two fundamentally distinct objects: ==virtual cells== and ==real cells==.
Virtual cells are cells that determine **the appearance** and **the behaviour** of the cell but **don't contain the value**.
The real cells have the **same properties** as virtual cells but also **contain the value of the cell**, and are therefore associated to a **specific position** in the grid.
因此有兩種根本不同的物件：==虛擬單元格==和==實際單元格==。
虛擬單元格是決定單元格**外觀**和**行為**，但**不包含值**的單元格。
實際單元格與虛擬單元格具有**相同的屬性**，但還**包含單元格的值**，因此與網格中的**特定位置**相關聯。

You can use the `Grid` control for any kinds of grid where you ==don't need to display large amounts of cells== (typically less than 50.000 cells). If you have to display large amount of cells you must usually use a `GridVirtual` derived controls, see the next chapters for more information.
您可以使用 `Grid` 控制項來處理任何==不需要顯示大量單元格的網格==（通常少於 50,000 個單元格）。如果您必須顯示大量單元格，則通常必須使用 `GridVirtual` 源控制的衍生物，請參閱後續章節以獲取更多資訊。
Typically in this article I will use the `Grid` control because is more simply to use especially for simply examples. But consider that basically the same code can be used also for `GridVirtual`.
在本文中，我通常會使用 `Grid` 控制項，因為它更簡單易用，特別是對於簡單的範例。但請考慮到，基本上相同的程式碼也可以用於 `GridVirtual` 。
The `Grid` is also used if you have unusual grids that require maximum flexibility.
如果您有需要極致靈活性的特殊網格， `Grid` 控制項也會被使用。

Drag the Grid control inside your forms as any other .NET control and start using it.
將 Grid 控制項像其他任何 .NET 控制項一樣拖放到您的表單中，然後開始使用它。

## Basic examples 基本範例

For now SourceGrid has a poor design time support, so usually you must write manually the code to manipulate the grid.
目前 SourceGrid 的設計時支援不佳，因此通常您必須手動編寫程式碼來操作網格。
Suppose that you have a `Grid` control named `grid1` you can write this code in the `Form.Load` event:
假設您有一個名為 `Grid` 的 `grid1` 控制項，您可以在此 `Form.Load` 事件中編寫此程式碼：

```csharp
grid1.BorderStyle = BorderStyle.FixedSingle;
grid1.ColumnsCount = 3;
grid1.FixedRows = 1;
grid1.Rows.Insert(0);
grid1[0,0] = new SourceGrid.Cells.ColumnHeader("String");
grid1[0,1] = new SourceGrid.Cells.ColumnHeader("DateTime");
grid1[0,2] = new SourceGrid.Cells.ColumnHeader("CheckBox");
for (int r = 1; r < 10; r++)
{
    grid1.Rows.Insert(r);
    grid1[r,0] = new SourceGrid.Cells.Cell("Hello " + r.ToString(), typeof(string));
    grid1[r,1] = new SourceGrid.Cells.Cell(DateTime.Today, typeof(DateTime));
    grid1[r,2] = new SourceGrid.Cells.CheckBox(null, true);
}

grid1.AutoSizeCells();
```

As you can see you can use the grid like a 2 dimension array. In the previous code I have set the grid border, the number of columns, the number of fixed rows and created the first header row. For the header I have used a `ColumnHeader` cell. With a simple loop I have created the other cells using a specific type for each column. The `Cell` class automatically creates an appropriate editor for the type specified (in this case a TextBox and a DateTimePicker). For the last column I have used a `CheckBox` cell that allows the display of a checkbox directly in the cell. Each kind of cell defines its own visual aspect and behaviour.
如您所見，您可以像使用二維陣列一樣使用網格。在前面的程式中，我已設定網格邊框、列數、固定行數，並建立了第一個標題行。對於標題，我使用了 `ColumnHeader` 結構。透過一個簡單的迴圈，我使用特定類型建立了其他結構，每個列使用不同的類型。 `Cell` 類別會自動為指定的類型（在本例中為 TextBox 和 DateTimePicker）建立適當的編輯器。對於最後一列，我使用了 `CheckBox` 結構，它允許在結構中直接顯示核取方塊。每種結構都定義了自己的視覺外觀和行為。

The form should look like the one in the following picture.
表單應該看起來像以下圖片中所示的樣子。

![[BasicGrid.png|FirstGrid]]

The grid created supports sorting, column sizing and editing of the cells.
格子所建立的支援排序、欄位大小調整以及單元格編輯。

Here some other important features:
這裡有一些其他重要的功能：

- If you want to read or change the value of a cell you can use the `grid1[r,c].Value` property where r and c are the row and column of the cell.
  如果您想要讀取或更改單元格的值，您可以使用 `grid1[r,c].Value` 屬性，其中 r 和 c 分別是單元格的行和列。
- To remove a row you can write `grid1.Rows.Remove(r)`
  要刪除一行，您可以寫入 `grid1.Rows.Remove(r)`
- To change the width of a column you can write `grid1.Columns[c].Width = 100`
  要變更欄寬度，您可以寫 `grid1.Columns[c].Width = 100`

If you want to change some visual properties of the cell you must use the `View` class.
如果您想變更單元格的一些視覺屬性，您必須使用 `View` 類別。

Let's see another example:
讓我們看另一個例子：

```csharp
grid1.BorderStyle = BorderStyle.FixedSingle;

grid1.ColumnsCount = 3;
grid1.FixedRows = 1;
grid1.Rows.Insert(0);

SourceGrid.Cells.Views.ColumnHeader boldHeader = new SourceGrid.Cells.Views.ColumnHeader();
boldHeader.Font = new Font(grid1.Font, FontStyle.Bold | FontStyle.Underline);

SourceGrid.Cells.Views.Cell yellowView = new SourceGrid.Cells.Views.Cell();
yellowView.BackColor = Color.Yellow;
SourceGrid.Cells.Views.CheckBox yellowViewCheck = new SourceGrid.Cells.Views.CheckBox();
yellowViewCheck.BackColor = Color.Yellow;

grid1[0, 0] = new SourceGrid.Cells.ColumnHeader("String");
grid1[0, 0].View = boldHeader;

grid1[0, 1] = new SourceGrid.Cells.ColumnHeader("DateTime");
grid1[0, 1].View = boldHeader;

grid1[0, 2] = new SourceGrid.Cells.ColumnHeader("CheckBox");
grid1[0, 2].View = boldHeader;
for (int r = 1; r < 10; r++)
{
    grid1.Rows.Insert(r);

    grid1[r, 0] = new SourceGrid.Cells.Cell("Hello " + r.ToString(), typeof(string));
    grid1[r, 0].View = yellowView;

    grid1[r, 1] = new SourceGrid.Cells.Cell(DateTime.Today, typeof(DateTime));
    grid1[r, 1].View = yellowView;

    grid1[r, 2] = new SourceGrid.Cells.CheckBox(null, true);
    grid1[r, 2].View = yellowViewCheck;
}
```

I have created a column header view with a style `FontStyle.Bold` | `FontStyle.Underline`, one view for a standard cell with a yellow backcolor and a checkbox view always with a yellow backcolor. Then I have assigned these instances to the `View` property of each cell.
我已創建了一個帶有 `FontStyle.Bold` | `FontStyle.Underline` 风格的列標題視圖，一個標準單元格的視圖帶有黃色背景色，一個總是帶有黃色背景色的復選框視圖。然後我將這些實例分配給每個單元格的 `View` 屬性。

The form should look like the one in the following picture.
表單應該看起來像以下圖片中所示的樣子。

![[BasicGrid2.png]]

You can note that I have assigned the same instance of the `View` class for many cells. This is useful to optimize the resources used.
您可以注意到我為多個單元格分配了相同的 `View` 類實例。這有助於**優化**所使用的資源。

Each cell can have an editor (`Editor` property) associated. The editor is used to edit the cell value. You can manually create an editor class (see `SourceGrid.Cells.Editors` namespace) or use the `SourceGrid.Cells.Editors.Factory` class to create an editor based on a `Type`. You can also use the `Cell` constructor that automatically calls `SourceGrid.Cells.Editors.Factory` if you specify the `Type` parameter.
每個單元格可以有一個編輯器（ `Editor` 屬性）相關聯。編輯器用於編輯單元格的值。您可以手動建立一個編輯器類別（參考 `SourceGrid.Cells.Editors` 命名空間）或使用 `SourceGrid.Cells.Editors.Factory` 類別基於一個 `Type` 建立編輯器。您也可以使用 `Cell` 建構子，如果您指定了 `Type` 參數，它會自動呼叫 `SourceGrid.Cells.Editors.Factory` 。

Here an example that creates some cells and the editors associated using one of the methods described.
這裡有一個範例，使用其中一種方法建立一些單元格及其相關的編輯器。

```csharp
//A DateTime editor
grid1[r, c] = new SourceGrid.Cells.Cell(DateTime.Today, typeof(DateTime));

//A string editor
grid1[r, c] = new SourceGrid.Cells.Cell("Ciao", typeof(string));

//A double editor
grid1[r, c] = new SourceGrid.Cells.Cell(58.4);
grid1[r, c].Editor = SourceGrid.Cells.Editors.Factory.Create(typeof(double));
```

Like the `View` classes also the editors can be shared between one or more cells.
就像 `View` 類別一樣，編輯器也可以在單一或多個格子之間共用。

Now you can start to work with SourceGrid, for more advanced information see the chapters below.
現在您可以開始使用 SourceGrid，有更多進階資訊請參閱下方的章節。

# Basic Concepts 基本概念

## Grid Control 網格控制項

The `Grid` control is the ideal if you want the greatest flexibility and simplicity but without many cells. In fact, in this control every cell is represented by a .NET class and therefore occupies a specific quantity of resources. Moreover this is the only grid that supports features of RowSpan and ColumnSpan (cells merge).
若您想要最大的靈活性和簡易性，但不需要太多單元格， `Grid` 控制項是理想之選。實際上，在這個控制項中，**每個單元格都由一個 .NET 類別表示，因此佔用特定的資源量**。此外，這是目前唯一**支援 RowSpan 和 ColumnSpan 功能**的 Grid（單元格可合併）。

Using the control in a Windows form is trivial. It's just like adding any other control like a Button or a DataGrid. First, create or open a Windows Application project and open a Windows Form into the designer. Now you are ready to customize the Toolbox: Right-Click the Toolbox, .NET Framework Components, Browse, select the DevAge.SourceGrid.dll. The Grid Control is now added to the Toolbox and can be inserted in Windows Form as any other control.
在 Windows 表單中使用此控制非常簡單。就像添加按鈕或 DataGrid 等其他控制一樣。首先，建立或開啟 Windows 應用程式專案，並在設計器中開啟 Windows 表單。現在您可以開始自訂工具箱：右鍵點擊工具箱，選擇 .NET Framework Components，瀏覽，選擇 DevAge.SourceGrid.dll.。網格控制現在已添加到工具箱，可以像插入其他控制一樣插入 Windows 表單。

After inserting the control in the form we can begin to write our code using the grid. For example in the `Load` event of the form we can write this code:
在表單中插入控制後，我們可以開始使用此網格撰寫程式碼。例如，在 `Load` 表單的事件中，我們可以撰寫這段程式碼：

```csharp
grid1.Redim(2, 2);
grid1[0,0] = new SourceGrid.Cells.Cell("Hello from Cell 0,0");
grid1[1,0] = new SourceGrid.Cells.Cell("Hello from Cell 1,0");
grid1[0,1] = new SourceGrid.Cells.Cell("Hello from Cell 0,1");
grid1[1,1] = new SourceGrid.Cells.Cell("Hello from Cell 1,1");
```

The previous code creates a table with 2 lines and 2 columns (`Redim` method) and populates every position with a cell. I have used the `SourceGrid.Cells` namespace which contains the definition for real cells.
前述程式碼會建立一個具有 2 行與 2 列的表格 ( `Redim` 方法)，並將每個位置填入一個單元格。我使用了 `SourceGrid.Cells` 命名空間，其中包含實際單元格的定義。

To read a specific value of the grid you can use the value property of the cell like this: `object val = grid1[1,0].Value;`.
若要讀取特定單元格的值，您可以像這樣使用單元格的 value 屬性： `object val = grid1[1,0].Value;` 。

See the other chapters for more information.
請參閱其他章節以獲取更多資訊。

## GridVirtual Control 虛擬網格控制項

The `GridVirtual` control is ideal when it is necessary to visualize a lot of cells and you already have available structured data like a `DataSet`, an `Array`, a document XML or other data structure.
當需要視覺化大量單元格且已擁有可用之結構化數據，例如 `DataSet` ， `Array` ，文檔 XML 或其他數據結構時， `GridVirtual` 控制項是理想選擇。

This type of grid have the same features of the `Grid` control except for the automatic sort (this because the grid cannot automatically order any external data structure without copying its content) and the feature of RowSpan and ColumnSpan that allows spanning of a cell across other adjacent cells.
這種網格具有與 `Grid` 控制項相同的功能，除了自動排序（因為網格無法在不複製其內容的情況下自動排序任何外部數據結構）以及允許單元格跨越其他相鄰單元格的 RowSpan 和 ColumnSpan 功能。

Another disadvantage is that creating a virtual grid is a little more difficult.
另一個缺點是建立虛擬網格稍微困難一些。

The main concept in a virtual grid is that each cell reads and writes its value from an external data structure and the grid doesn't maintain all the rows and columns but are normally read directly from the data source. This idea was implemented with an abstract `GridVirtual` class and with the abstract methods `CreateRowsObject`, `CreateColumnsObject` and `GetCell`. Then you can use a special `IValueModel` to read the values directly from the data source.
虛擬網格的主要概念在於每一個格子會從外部數據結構讀取和寫入其值，而網格並不維護所有行和列，但通常會直接從數據來源讀取。這個想法是透過一個抽象的 `GridVirtual` 類別以及抽象方法 `CreateRowsObject` 、 `CreateColumnsObject` 和 `GetCell` 來實現的。接著你可以使用一個特殊的 `IValueModel` 來直接從數據來源讀取值。

To use `GridVirtual` it is therefore necessary to create a class that derives from `GridVirtual` and personalize the reading of the data source overriding `CreateRowsObject`, `CreateColumnsObject` and `GetCell` methods.
要使用 `GridVirtual` ，因此必須建立一個繼承自 `GridVirtual` 的類別，並透過覆寫 `CreateRowsObject` 、 `CreateColumnsObject` 和 `GetCell` 方法來自訂數據來源的讀取方式。

The purpose of the method `GetCell` is to return, for a given position (row and column), the chosen cell, the methods `CreateRowsObject` and `CreateColumnsObject` are used to create the columns and the rows objects for the used data source. This allows great flexibility because you can return any `ICellVirtual` for a specific type; for example you could return a cell of type header when the row is 0.
方法 `GetCell` 的目的是為了返回一個給定位置（行和列）所選擇的格子，方法 `CreateRowsObject` 和 `CreateColumnsObject` 則用於為所使用的數據來源建立列和行物件。這提供了高度的靈活性，因為你可以為特定類型返回任何 `ICellVirtual` ；例如，當行是 0 時，你可以返回一個類型為標題的格子。

Usually you don't need to use directly the `GridVirtual` but one of the derived controls. For now I have implemented two controls that directly use the virtual grid features:
通常你不需要直接使用 `GridVirtual` ，而是使用其中一個衍生的控制元件。目前我已實作了兩個直接使用虛擬網格功能的控制元件：

- `DataGrid` - A grid bound to a DataView object - 一個繫結到 DataView 物件的網格。
- `ArrayGrid` - A grid bound to an Array object - 一個繫結到 Array 物件的網格。

If you want to create your custom control to read from a specific data source you can see `ArrayGrid` class for an example.
如果您想要建立自訂控制項來讀取特定資料來源，您可以參考 `ArrayGrid` 類別作為範例。

## Cell overview 單元總覽

Every cell is composed of four fundamental parts based on a modified Model-View-Controller pattern:
每一個單元都是由四個基礎部分組成，基於一個修改過的 Model-View-Controller 模式：

- `Model` : The Model is the class that manages the value of the cells. It contains the values or properties associated and is accessible by the other components.
  Model 是**管理單元值**的類別。它包含相關的值或屬性，並可被其他組件存取。
- `View` : The View is the class that draws the cell and contains the visual properties.
  View 是**繪製單元**的類別，並包含**視覺屬性**。
- `Controller` : The Controller is the class that supplies the behaviour of the cell.
  Controller 是**提供單元行為**的類別。
- `Editor` : The Editor is the class used to customize the editor of the cell.
  Editor 是用來**自訂單元編輯器**的類別。

![[SourceGridMVC.GIF|SourceGridMVC]]

This subdivision grants great flexibility and reusability of code, saves time and supplies a solid base for every type of customization.
這個分類提供了極大的彈性和程式碼的可重用性，節省時間並為每種自訂提供堅實的基礎。

For the more common cases there are some classes already arranged and configured, but with a few lines of code is possible to create personalized cells (see the next paragraphs for the details).
針對較常見的情況，有一些已經排列和設定好的類別，但僅需幾行程式碼即可建立個人化的單元（請參閱下一段落了解詳情）。

## Rows and Columns 列和欄

The main components of a grid are the rows and the columns. To manipulate this information, SourceGrid supplies 2 properties:
格子主要組成部分是行和列。為了操作這些資訊，SourceGrid 提供了 2 個屬性：

- `Rows` - Manages the rows information, the base class is the `RowsBase` class.
  管理行資訊，基礎類別是 `RowsBase` 類別。
- `Columns` - Manages the columns information, the base class is the `ColumnsBase` class.
  管理列資訊，基礎類別是 `ColumnsBase` 類別。

When using a real grid the base classes are extended with the `RowInfoCollection` and the `ColumnInfoCollection`. These are collection of `RowInfo` and `ColumnInfo` classes. When using a virtual grid you must extend the base classes with your custom code to provide information of your data source.
在實際使用格子時，基礎類別會透過 `RowInfoCollection` 和 `ColumnInfoCollection` 進行擴充。這些是 `RowInfo` 和 `ColumnInfo` 類別的集合。在實際使用虛擬格子時，您必須透過自訂程式碼擴充基礎類別，以提供您資料來源的資訊。

## Manipulating Rows and Columns on real grid 操作實際網格的行和列

NOTE: Only valid for real grid.
注意：僅適用於實際網格。

These are some of the `RowInfo` class properties: `Height`, `Top`, `Bottom`, `Index`, `Tag`. In contrast, these are the properties of the `ColumnInfo` class:`Width`, `Left`, `Right`, `Index`, `Tag`.
這些是 `RowInfo` 類別的一些屬性： `Height` 、 `Top` 、 `Bottom` 、 `Index` 、 `Tag` 。相比之下，這些是 `ColumnInfo` 類別的屬性： `Width` 、 `Left` 、 `Right` 、 `Index` 、 `Tag` 。

There are many ways to create rows and columns:
創建行和列的方法有很多：

```csharp
grid1.Redim(2,2);
```

```csharp
grid1.RowsCount = 2;
grid1.ColumnsCount = 2;
```

```csharp
grid1.Rows.Insert(0);
grid1.Rows.Insert(1);
grid1.Columns.Insert(0);
grid1.Columns.Insert(1);
```

These three examples perform all the same task of creating a table with 2 rows and 2 columns.
這三個範例都執行相同的任務，即建立一個具有 2 行和 2 列的表格。

To change the width or the height of a row or a column you can use this code:
要變更列或行的寬度或高度，您可以使用這段程式碼：

```csharp
grid1.Rows[0].Height = 100;
grid1.Columns[0].Width = 100;
```

The properties `Top`, `Bottom`, `Left` and `Right` are automatically calculated using the width and the height of the rows and columns.
屬性 `Top` 、 `Bottom` 、 `Left` 和 `Right` 會自動使用列和行的寬度和高度來計算。

After allocating the rows and columns, you must create for each position of the grid the required cells, like this code:
分配完行和列之後，您必須為網格的每個位置創建所需的單元格，就像這段程式碼：

```csharp
grid1.Redim(2,2);
grid1[0, 0] = new SourceGrid.Cells.Cell("Cell 0, 0");
grid1[1, 0] = new SourceGrid.Cells.Cell("Cell 1, 0");
grid1[0, 1] = new SourceGrid.Cells.Cell("Cell 0, 1");
grid1[1, 1] = new SourceGrid.Cells.Cell("Cell 1, 1");
```

## Model 模型

Namespace: **SourceGrid.Cells.Models**

The purpose of the `Model` classes is to separate the data in the cells from the cell object. This separation is used for two main reasons:
`Model` 類別的目的是將單元格中的**數據**與**單元格物件**分開。這種分離主要有兩個原因：

- To implement virtual grid, where the values are stored only in the original data source. In this case the `Model`, if implemented in the right way, reads the data directly from the data source.
  為了實現虛擬網格，其中僅僅在原始數據來源中存儲值。在這種情況下，如果 `Model` 正確實現，則會直接從數據來源讀取數據。
- To extend the cells classes but maintaining an easy reusability of the code. This is because you don't have to change the base `Cell` class to add new features, but you can simply add a new `Model`.
  為了擴展單元格類別，但同時保持代碼的易於重用性。這是因為您不必更改基礎 `Cell` 類別以添加新功能，但可以簡單地添加一個新的 `Model` 。

Every cell has a property `Model` that gets or sets a `ModelContainer` object. This class is a collection of `IModel` interface. Each `IModel` interface contains all the properties used for a specific feature.
每一個單元格都有一個屬性 `Model` ，用於取得或設定一個 `ModelContainer` 物件。這個類別是一組 `IModel` 介面的集合。每個 `IModel` 介面都包含所有用於特定功能的屬性。

The main `Model` is the `ValueModel` which contains the value of the cell, but there also other models and you can crate your custom models. Here are the default models:
主要 `Model` 是包含單元格值的 `ValueModel` ，但還有其他模型，你可以建立自訂模型。這裡是預設模型：

- `IValueModel`
- `IToolTipText`
- `ICheckBox`
- `ISortableHeader`

Each of these models contains the properties specified for each features, for example the `IToolTipText` contains the ToolTipText string.
每個這些模型都包含為每個功能指定的屬性，例如 `IToolTipText` 包含 ToolTipText 字串。

Each cell has a collection of models to allow using many features on a single cell.
每個單元格有一組模型，允許在單個單元格上使用多個功能。

Usually using real cell you can use a model that simply implement the right interface storing directly the required data; using virtual grid you can implement the interface to bind the values directly to an external data source.
通常使用實際單元格時，你可以使用一個簡單實現正確介面並直接存儲所需數據的模型；使用虛擬網格時，你可以實現介面將值直接綁定到外部數據源。

## View 檢視

Namespace: **SourceGrid.Cells.Views**

Every cell has a property `View` to gets or sets an interface of type `IView`. The cell uses this interface to draw and to customize the visual properties of the cell.
每個單元格都有一個屬性 `View` 用於取得或設定類型為 `IView` 的介面。單元格使用這個介面來繪製和自訂單元格的視覺屬性。

The purpose of the `View` is to separate the drawing code from the rest of the code and allows sharing of the same visual properties between cells. In fact the same instance of `View` can be used on many cells simultaneously thus optimizing the use of the resources of the system.
`View` 的目的是將繪製程式碼與其他程式碼分開，並允許單元格之間共享相同的視覺屬性。事實上，同一個 `View` 的實例可以同時用於多個單元格，從而優化系統資源的利用。

These are the default `View` classes in the namespace `SourceGrid.Cells.Views:`
這些是命名空間 `SourceGrid.Cells.Views:` 中的預設 `View` 類別。

- `SourceGrid.Cells.Views.Cell` - Used for classic cells. In this view you can customize the colours, the font, the borders and a lot other properties.
  用於經典單元。在這個視圖中，您可以自訂顏色、字體、邊框和其他許多屬性。
- `SourceGrid.Cells.Views.Button` - Used for a cell with a button style with theme support.
  用於具有按鈕風格且支援主題的單元。
- `SourceGrid.Cells.Views.Link` - Used for a cell with a link style.
  用於具有連結風格的單元。
- `SourceGrid.Cells.Views.CheckBox`* - Used for checkbox style cells. The checkbox can be selected, disabled and can contain a caption.
  用於選項方塊樣式單元格。選項方塊可以被選中、禁用，並可包含標題。
- `SourceGrid.Cells.Views.Header`* - Used for a generic header cell.
  用於通用標題單元格。
- `SourceGrid.Cells.Views.ColumnHeader`* - Used for a column header cell with theme support.
  用於支援主題的列標題單元格。
- `SourceGrid.Cells.Views.RowHeader` - Used for row header cell with theme support.
  用於支援主題的行標題單元格。
- `SourceGrid.Cells.Views.MultiImages` - Allows the drawing of more than one image in the cell.
  允許在單元格中繪製多於一個圖像。

\**The `View` marked with an asterisk requires special components to work correctly, for example the `CheckBox` view needs an `ICheckBox` `Model`.*
\**標註有星號的 `View` 需要特殊組件才能正確運作，例如 `CheckBox` 視圖需要一個 `ICheckBox` `Model` 。*

Some of the previous classes contain one or more static properties with some convenient default instances.
一些先前類別包含一個或多個具有一些方便預設實例的靜態屬性。

This code shows how to create a `View` class, change some properties and assign it to more cells previously created:
這段程式碼展示了如何創建一個 `View` 類別，更改一些屬性並將其分配給先前創建的多個單元格：

```csharp
SourceGrid.Cells.Views.Cell view = new SourceGrid.Cells.Views.Cell();
view.BackColor = Color.Khaki;
grid1[0,0].View = view;
grid1[2,0].View = view;
```

With some line of code you can create your custom `View`, in this case I suggest deriving your class from `Cell`, which already has implemented some default methods. In the following code example I create a `View` that draws a red ellipse over the cell:
使用一些程式碼，您可以創建您自己的 `View` ，在這種情況下，我建議您從 `Cell` 繼承您的類別，它已經實現了一些預設方法。在以下程式碼範例中，我創建了一個 `View` ，它在單元格上繪製一個紅色橢圓形：

```csharp
public class MyView : SourceGrid.Cells.Views.Cell
{
	protected override void DrawCell_Background(SourceGrid.Cells.ICellVirtual p_Cell,
	    SourceGrid.Position p_CellPosition, PaintEventArgs e, Rectangle p_ClientRectangle)
	{
        base.DrawCell_Background (p_Cell, p_CellPosition, e, p_ClientRectangle);

        e.Graphics.DrawEllipse(Pens.Red, p_ClientRectangle);
	}
}
```

To use this new `View` in a cell use this code:
要在 cell 中使用這個新的 `View` ，請使用這段程式碼：

```csharp
MyView myView = new MyView();
//...... 代码用來填充網格
grid1[r, c].View = myView;
```

## Controller 控制器

Namespace: **SourceGrid.Cells.Controllers**

Every cell has a property `Controller` that gets or sets a `ControllerContainer` object. This class is a collection of `IController` interface. Each `IController` interface can be used to extend the behaviour of the cell using many events like `Click, MouseDown, KeyPress,` ...
每一個單元格都有一個屬性 `Controller` ，用於取得或設定一個 `ControllerContainer` 物件。這個類別是一組 `IController` 介面的集合。每個 `IController` 介面都可以用來擴充單元格的行為，使用許多事件，例如 `Click, MouseDown, KeyPress,` ...

These are the default classes:
這些是預設的類別：

- `SourceGrid.Cells.Controllers.Cell` - Common behaviour of a cell.
  單元格的通用行為。
- `SourceGrid.Cells.Controllers.Button` - A cell that acts as a button.
  一個扮演按鈕角色的單元格。
- `SourceGrid.Cells.Controllers.CheckBox`* - A cell with a behaviour of a CheckBox. (need `ICheckBox` Model)
  一個具有 CheckBox 行為的細胞。(需要 `ICheckBox` 模型)
- `SourceGrid.Cells.Controllers.ColumnFocus` - Used to set the focus on the column when clicking the header cell.
  用於在點擊標題細胞時將焦點設置在該列上。
- `SourceGrid.Cells.Controllers.ColumnSelector` - Used to set select the column when clicking the header cell.
  用於在點擊標題細胞時選擇該列。
- `SourceGrid.Cells.Controllers.CustomEvents` - Exposes a list of events that you can use without creating a new `Controller`.
  暴露一組您可以無需創建新的 `Controller` 即可使用的事件列表。
- `SourceGrid.Cells.Controllers.FullColumnSelection` - Used to enable the columns selection mode.
  用來啟用欄選擇模式。
- `SourceGrid.Cells.Controllers.FullRowSelection` - Used to enable the rows selection mode.
  用來啟用列選擇模式。
- `SourceGrid.Cells.Controllers.MouseCursor` - Used to display a mouse cursor over the cell.
  用來在單元格上顯示滑鼠光標。
- `SourceGrid.Cells.Controllers.MouseInvalidate` - Used to invalidate the cell area when receiving a mouse event.
  用來在接收滑鼠事件時使單元格區域失效。
- `SourceGrid.Cells.Controllers.Resizable` - Used to create a resizable cell, for both width and height.
  用來建立可調整大小的格子，適用於寬度和高度。
- `SourceGrid.Cells.Controllers.RowFocus` - Used to set the focus on the row when clicking the header cell.
  用來在點擊標題格子時，將焦點設定在該列。
- `SourceGrid.Cells.Controllers.RowSelector` - Used to set select the row when clicking the header cell.
  用來在點擊標題格子時，選取該列。
- `SourceGrid.Cells.Controllers.SortableHeader`* - Used to create a sortable column header cell. (need a `ISortableHeader` Model)
  用來建立可排序的列標題格子。(需要一個 `ISortableHeader` 模型)
- `SourceGrid.Cells.Controllers.ToolTipText`* - Used to create a cell with a ToolTip. (need `IToolTipText` Model)
  用於建立具有工具提示的單元格。(需要 `IToolTipText` 模型)
- `SourceGrid.Cells.Controllers.Unselectable` - Used to create an unselectable cell.
  用於建立不可選擇的單元格。

\**The `Controller` marked with an asterisk need special models to complete their tasks.*
\**標有星號的 `Controller` 需要特殊的模型才能完成其任務。*

Some of the previous classes contain one or more static properties with some convenient default instances.
部分先前類別包含一個或多個具有方便預設實例的靜態屬性。
Here the list of events that can be used inside a `Controller`:
這裡是可以用於 `Controller` 內的事件清單：

- Mouse: `OnMouseDown`, `OnMouseUp`, `OnMouseMove`, `OnMouseEnter`, `OnMouseLeave`
- Keys: `OnKeyUp`, `OnKeyDown`, `OnKeyPress`
- Click: `OnDoubleClick`, `OnClick`
- Focus: `OnFocusLeaving`, `OnFocusLeft`, `OnFocusEntering`, `OnFocusEntered`, `CanReceiveFocus`
- Cell Value: `OnValueChanging`, `OnValueChanged`
- Edit: `OnEditStarting`, `OnEditStarted`, `OnEditEnded`

With some line of code you can create your custom `Controller`, in this case I suggest deriving your class from `ControllerBase`, which already have implemented some default methods. In the following code example I create a `Controller` that changes the backcolor of the cell when the user moves the mouse over it:
透過一些程式碼，您可以創建您自訂的 `Controller` ，在此情況下，我建議您從 `ControllerBase` 繼承您的類別，因為它已實現了一些預設方法。在以下程式碼範例中，我創建了一個 `Controller` ，當使用者將滑鼠移到上面時，會改變單元格的背景顏色：

```csharp
public class MyController : SourceGrid.Cells.Controllers.ControllerBase
{
	private SourceGrid.Cells.Views.Cell MouseEnterView = new SourceGrid.Cells.Views.Cell();
	private SourceGrid.Cells.Views.Cell MouseLeaveView = new SourceGrid.Cells.Views.Cell();
	public MyController()
	{
		MouseEnterView.BackColor = Color.Green;
	}

	public override void OnMouseEnter(SourceGrid.CellContext sender, EventArgs e)
	{
		base.OnMouseEnter (sender, e);

		sender.Cell.View = MouseEnterView;

		sender.Grid.InvalidateCell(sender.Position);
	}
	public override void OnMouseLeave(SourceGrid.CellContext sender, EventArgs e)
	{
		base.OnMouseLeave (sender, e);

		sender.Cell.View = MouseLeaveView;
		
		sender.Grid.InvalidateCell(sender.Position);
	}
}
```

To use this new `Controller` in a cell use this code:
要在細胞中使用這個新的 `Controller` ，請使用這段程式碼：

```csharp
myController myController = new myController();
//...... 代码用來填充網格
grid1[r, c].AddController(myController);
```

You can also add a controller to the whole grid to apply the same controller to all the cells:
您可以將控制器新增至整個網格，以便將相同的控制器套用至所有格子：

```csharp
grid1.Controller.AddController(new MyController());
```

Consider for example the Controller below that open a messagebox each time a user click on a cell:
例如，考慮以下控制器，每次使用者點擊格子時會打開一個訊息方塊：

```csharp
class ClickController : SourceGrid.Cells.Controllers.ControllerBase
{
    public override void OnClick(SourceGrid.CellContext sender, EventArgs e)
    {
        base.OnClick(sender, e);

        object val = sender.Value;
        if (val != null)
            MessageBox.Show(sender.Grid, val.ToString());
    }
}
```

You can add this controller to all the cells with this code:
您可以透過這段程式碼將此控制器添加到所有單元格中：

```csharp
grid1.Controller.AddController(new ClickController());
```

Here another example of how to use a controller to check when the value of a cell change, using the `OnValueChanged` event:
這裡是另一個範例，展示如何使用控制器來檢查當單元格的值變更時，使用 `OnValueChanged` 事件：

```csharp
public class ValueChangedEvent : SourceGrid.Cells.Controllers.ControllerBase
{
    public override void OnValueChanged(SourceGrid.CellContext sender, EventArgs e)
    {
        base.OnValueChanged(sender, e);

        string val = "Value of cell {0} is '{1}'";

        MessageBox.Show(sender.Grid, string.Format(val, sender.Position, sender.Value));
    }
}
```

You can add this controller to all the cells with this code:
您可以透過這段程式碼將此控制器添加到所有單元格中：

```csharp
grid1.Controller.AddController(new ValueChangedEvent());
```

Each time the value of a cell change the previous controller show a messagebox with the position of the cell and the new value.
每次單元格的值變更時，先前的主控會顯示一個訊息方塊，其中包含單元格的位置和新的值。

## Editor 編輯器

Namespace: **SourceGrid.Cells.Editors**

Every cell has a property `Editor` that gets or sets an `EditorBase` object. This class is used to supply a cell editor. If this property is null it is not possible to edit the cell.
每個單元格有一個屬性 `Editor` ，用來取得或設定一個 `EditorBase` 物件。這個類別是用來提供單元格編輯器。如果這個屬性是空的，就不可能編輯單元格。

Usually the `Editor` uses the `Model` class to manage the necessary conversion, particularly the string conversion (used to represent the cell value).
通常 `Editor` 使用 `Model` 類別來管理必要的轉換，特別是字串轉換（用來表示單元格的值）。

These are the default classes:
這些是預設類別：

- `ComboBox` - A `ComboBox` editor.
- `DateTimePicker` - A `DateTimePicker` editor.
- `NumericUpDown` - A `NumericUpDown` editor.
- `TextBox` - A `TextBox` editor. This is one of the more used editors by all types that support string conversion `(string, int, double, enum,....)`.
  這是所有支援字串轉換的類型中較常使用的編輯器 `(string, int, double, enum,....)` 。
- `TextBoxCurrency` - A `TextBox` editor specialized for numeric currency values.
  一個專門用於數值貨幣值的 `TextBox` 編輯器。
- `TextBoxNumeric` - A `TextBox` editor specialized for numeric values.
  一個專門用於數值 `TextBox` 編輯器。
- `TimePicker` - A `DateTimePicker` editor specialized for time values.
  一個專為時間值設計的 `DateTimePicker` 編輯器。
- `TextBoxUITypeEditor` - Supplies the editing of the cell of all types that have a `UITypeEditor`. This is a very useful class because a lot of types support this class: DateTime, Font, enums, and you can also create your custom `UITypeEditor`.
  提供對所有具有 `UITypeEditor` 的單元格的編輯。這是一個非常實用的類別，因為許多類型支援這個類別：DateTime、Font、列舉，你也可以創建自己的自訂 `UITypeEditor` 。
- `ImagePicker` - An editor that can be used to select an image file and edit a byte[] values.
  一個可以用於選擇圖像檔案並編輯 byte[] 值的編輯器。

An `Editor` can be shared between many cells of the same grid; for example you can use the same `Editor` for every cell of a column, but always with the same grid.
一個 `Editor` 可以在相同網格的許多單元格之間共用；例如，你可以為某個欄的每個單元格使用相同的 `Editor` ，但始終使用相同的網格。

Each `Editor` class has a property `Control` that returns an instance of the `Windows Forms` control used to edit the cell. You can use this instance to customize the editor control or for advanced features.
每一個 `Editor` 類別都有一個 `Control` 屬性，它會回傳一個用來編輯單元格的 `Windows Forms` 控制器的實例。您可以利用這個實例來自訂編輯器控制項或實現進階功能。

Here some ways to create an editable cell:
這裡有一些創建可編輯單元格的方法：

- Create the cell specifying the value type. In this way the cell automatically calls a utility function, `SourceGrid.Cells.Editor.Factory.Create`, which returns a valid `Editor` for the type specified or null.
  指定值類型來創建單元格。這樣，單元格會自動呼叫一個工具函數 `SourceGrid.Cells.Editor.Factory.Create` ，它會回傳一個符合所指定類型的有效 `Editor` ，或者回傳 null。

```csharp
//String cell
grid1[0, 0] = new SourceGrid.Cells.Cell("Hello", typeof(string));
//Double cell
grid1[0, 1] = new SourceGrid.Cells.Cell(0.7, typeof(double));
```

- Create the `Editor` separately and then assign it to the cells, This method is recommended when you want to use the same editor for several cells:
  分別建立 `Editor` 再指定給格子的值，此方法當您想使用相同編輯器於多個格子時建議使用：

```csharp
//String editor
SourceGrid.Cells.Editors.IEditor editorString = SourceGrid.Cells.Editor.Factory.Create(typeof(string));
//Double editor
SourceGrid.Cells.Editors.IEditor editorDouble = SourceGrid.Cells.Editor.Factory.Create(typeof(double));
//String cell
grid1[0, 0] = new SourceGrid.Cells.Cell("Hello");
grid1[0, 0].Editor = editorString;
//Double cell
grid1[0, 1] = new SourceGrid.Cells.Cell(0.7);
grid1[0, 1].Editor = editorDouble;
```

- Create manually the correct editor and then assign it to the cells:
  手動建立正確的編輯器，然後將其指派給格子：

```csharp
SourceGrid.Cells.Editors.TextBox txtBox = new SourceGrid.Cells.Editors.TextBox(typeof(string));
grid1[2,0].Editor = txtBox;
```

If you need greater control on the type of editor or there are special requirements I suggest to manually creating the editor.
如果您需要對編輯器類型有更大的控制權，或者有特殊需求，我建議手動建立編輯器。

In this case, for example, I manually create the class `EditorTextBox` and then set the property `MaxLength`.
在此情況下，例如，我手動建立類別 `EditorTextBox` ，然後設定屬性 `MaxLength` 。

```csharp
//String editor
SourceGrid.Cells.Editors.TextBox editorString = new SourceGrid.Cells.Editors.TextBox(typeof(string));
editorString.Control.MaxLength = 10;
//String cell
grid1[0, 0] = new SourceGrid.Cells.Cell("Hello");
grid1[0, 0].Editor = editorString;
```

The editor can be used also to customize the format of the cell, in the code below for example I create a cell with a custom numeric format:
這個編輯器也可以用來自訂格子的格式，在下面的程式碼範例中，我創建了一個具有自訂數字格式的格子：

```csharp
//Double editor
SourceGrid.Cells.Editors.TextBoxNumeric editorDouble = 
    new SourceGrid.Cells.Editors.TextBoxNumeric(typeof(double));
editorDouble.TypeConverter = 
    new DevAge.ComponentModel.Converter.NumberTypeConverter(typeof(double), "#,###.00");
//String cell
grid1[0, 0] = new SourceGrid.Cells.Cell(9419.3894);
grid1[0, 0].Editor = editorDouble;
```

I have used the `TypeConverter` property to customize the conversion to and from string values. There are many other `TypeConverter` available:
我已使用 `TypeConverter` 屬性來自定義從字串值轉換過來和轉換回去的過程。還有許多其他 `TypeConverter` 可用：

- `DevAge.ComponentModel.Converter.NumberTypeConverter` - For numeric types like double, decimal, int, float
  用於數值類型，如 double、decimal、int、float
- `DevAge.ComponentModel.Converter.PercentTypeConverter` - For numeric types like double, decimal and float but with percent format.
  用於數值類型（如 double、decimal 和 float），但以百分比格式顯示。
- `DevAge.ComponentModel.Converter.CurrencyTypeConverter` - For Decimal and double types with currency format.
  用於 decimal 和 double 類型，以貨幣格式顯示。
- `DevAge.ComponentModel.Converter.DateTimeTypeConverter` - For DateTime type.
  用於 DateTime 類型。

Below another example that create a DateTime editor with a custom format:
以下另一個範例，創建一個具有自訂格式的 DateTime 輸入器：

```csharp
//DateTime editor with custom format
string[] dtParseFormats = new string[] { dtFormat2 };
System.Globalization.DateTimeStyles dtStyles =
    System.Globalization.DateTimeStyles.AllowInnerWhite |
    System.Globalization.DateTimeStyles.AllowLeadingWhite |
    System.Globalization.DateTimeStyles.AllowTrailingWhite |
    System.Globalization.DateTimeStyles.AllowWhiteSpaces;
TypeConverter dtConverter = 
    new DevAge.ComponentModel.Converter.DateTimeTypeConverter(dtFormat2, dtParseFormats, dtStyles);
SourceGrid.Cells.Editors.TextBoxUITypeEditor editorDt2 = 
    new SourceGrid.Cells.Editors.TextBoxUITypeEditor(typeof(DateTime));
editorDt2.TypeConverter = dtConverter;

grid[currentRow, 1] = new SourceGrid.Cells.Cell(DateTime.Today);
grid[currentRow, 1].Editor = editorDt2;
```

The following picture shows most of the editors available and some special cells (picture from Sample 3):
以下圖片顯示了大部分可用的編輯器以及一些特殊單元格（圖片來自範例 3）：

![[SourceGrid_EditorsCells.jpg|EditorsCells]]

It is possible to create a custom `Editor` with custom control or special behaviours with few lines of code.
可以透過幾行程式碼來建立自訂的 `Editor` ，並且可以設定自訂控制項或特殊行為。

You can derive your custom class from `EditorControlBase` and create any Windows Forms control. This is an example of an editor that uses `DateTimePicker` control:
您可以從 `EditorControlBase` 繼承自訂類別，並建立任何 Windows Forms 控制項。這是一個使用 `DateTimePicker` 控制項的編輯器範例：

```csharp
public class DateTimePicker : EditorControlBase
{
	public DateTimePicker():base(typeof(System.DateTime))
	{
	}
	
	protected override Control CreateControl()
	{
        System.Windows.Forms.DateTimePicker dtPicker = new System.Windows.Forms.DateTimePicker();
        dtPicker.Format = DateTimePickerFormat.Short;
        dtPicker.ShowCheckBox = AllowNull;
        return dtPicker;
	}

    protected override void OnChanged(EventArgs e)
    {
        base.OnChanged(e);
        if (Control != null)
            Control.ShowCheckBox = AllowNull;
    }
    
	public new System.Windows.Forms.DateTimePicker Control
	{
        get
        {
            return (System.Windows.Forms.DateTimePicker)base.Control;
        }
	}
	
	protected override void OnStartingEdit(CellContext cellContext, Control editorControl)
	{
        base.OnStartingEdit(cellContext, editorControl);
        System.Windows.Forms.DateTimePicker dtPicker =
            (System.Windows.Forms.DateTimePicker)editorControl;
        dtPicker.Font = cellContext.Cell.View.Font;
	}
	
	public override void SetEditValue(object editValue)
	{
        if (editValue is DateTime)
            Control.Value = (DateTime)editValue;
        else if (editValue == null)
            Control.Checked = false;
        else
            throw new SourceGridException("Invalid edit value, expected DateTime");
	}
	
	public override object GetEditedValue()
	{
        if (Control.Checked)
            return Control.Value;
        else
    		return null;
	}

    protected override void OnSendCharToEditor(char key)
    {
    }
}
```

Consider that you can share the same instance of the editor with many cells but only with a single grid control. Basically each editor is associated with only one grid.
考量到您可以與多個單元格共用相同的一個編輯器實例，但僅能與單一個網格控制元件共用。基本上，**每個編輯器僅與一個網格相關聯。**

For large grid it is always a good idea to share the editors, because each editor has a Windows Forms Control associated and so it is quite heavy.
**對於大型網格來說，共用編輯器總是一個好主意，因為每個編輯器都有一個 Windows Forms 控制元件相關聯，所以相當重。**

## Advanced cells 進階單元格

Namespace: **SourceGrid.Cells**

These are the default cells available:
這些是可用的預設單元格：

- **SourceGrid.Cells.Virtual** - This namespace contains all the virtual cells that can be used with a `GridVirtual` control.
    SourceGrid.Cells.Virtual - 這個命名空間包含所有可與 `GridVirtual` 控制元件一起使用的虛擬單元格。
    - `CellVirtual` - Base cell for each other implementations; use for the most common type of virtual cells.
      其他實現的基礎單元格；用於最常見的虛擬單元格類型。
    - `Header` - A header cell.
    - `ColumnHeader` - A column header cell.
    - `RowHeader` - A row header cell.
    - `Button` - A button cell.
    - `CheckBox` - A checkbox cell.
    - `ComboBox` - A combobox cell.
    - `Link` - A link style cell.
    - `Image` - An image cell.
- **SourceGrid.Cells** - This namespace contains all the real cells that can be used with a `Grid` control.
  此命名空間包含所有可與 `Grid` 控制項一起使用的實際單元格。
    - `Cell` - Base cell for all other implementations; use for the most common type of real cells.
      所有其他實現的基礎單元格；用於最常見的實際單元格類型。
    - `Header` - A header cell.
    - `ColumnHeader` - A column header cell.
    - `RowHeader` - A row header cell.
    - `Button` - A button cell.
    - `CheckBox` - A checkbox cell.
    - `ComboBox` - A combobox cell.
    - `Link` - A link style cell.
    - `Image` - An image cell.

The goal of these classes is to simplify the use of `View, Model, Controller and Editor` classes. If we look at the code of any of the cell classes we can see that these classes use these components according to the role of the cell.
這些類別的目標是簡化 `View, Model, Controller and Editor` 類別的使用。如果我們查看任何單元格類別的程式碼，就會發現這些類別是根據單元格的角色來使用這些組件。

This is, for example, the code of the cell `SourceGrid.Cells.CheckBox`:
例如，這是單元格 `SourceGrid.Cells.CheckBox` 的程式碼：

```csharp
public class CheckBox : Cell
{
	public CheckBox(string caption, bool checkValue):base(checkValue)
	{
		if (caption != null && caption.Length > 0)
			View = Views.CheckBox.MiddleLeftAlign;
		else
			View = Views.CheckBox.Default;

		Model.AddModel(new Models.CheckBox());
		AddController(Controllers.CheckBox.Default);
		AddController(Controllers.MouseInvalidate.Default);
		Editor = new Editors.EditorBase(typeof(bool));
		Caption = caption;
	}
	private Models.CheckBox CheckBoxModel
	{
		get{return (Models.CheckBox)Model.FindModel(typeof(Models.CheckBox));}
	}
	public bool Checked
	{
		get{return CheckBoxModel.GetCheckBoxStatus(this, Range.Start).Checked;}
		set{CheckBoxModel.SetCheckedValue(this, Range.Start, value);}
	}
	public string Caption
	{
		get{return CheckBoxModel.Caption;}
		set{CheckBoxModel.Caption = value;}
	}
}
```

## Focus and Selection 焦點與選擇

A cell can be selected or can have the focus. Only one cell can have the focus, identified by the `Grid.Selection.ActivePosition` property of the grid, while many cells can be selected. A cell is selected when is present in the `Selection` object of the grid.
一個格子可以被選擇或擁有焦點。僅有一個格子可以擁有焦點，由格子的 `Grid.Selection.ActivePosition` 屬性識別，而多個格子可以被選擇。當一個格子存在於格子的 `Selection` 物件中時，該格子即被選擇。

The cell with the ==focus== receives all the mouse and keyboard events, while the ==selected== cells can receive actions like copy, paste and clear.
擁有==焦點==的格子會接收所有**滑鼠**和**鍵盤**事件，而==選擇==的格子則可以接收**複製**、**貼上**和**清除**等操作。

To set the focus on a specific cell you can use `Grid.Selection.Focus(Position pos)` method, using for input the position of the cell that will receive the focus, use `Position.Empty` as parameter to remove the focus.
要將焦點設定在特定單元格，您可以使用 `Grid.Selection.Focus(Position pos)` 方法，輸入將獲得焦點的單元格位置，使用 `Position.Empty` 作為參數以移除焦點。

Use the `Grid.Selection.SelectCell` method or `SelectRange` method to add or remove specific cells from the selection.
要從選擇中添加或移除特定單元格，可以使用 `Grid.Selection.SelectCell` 方法或 `SelectRange` 方法。

To list all the selected cells you can use `Grid.Selection.GetRanges()` method that returns a list of selected Range, or check if a particular cell is selected using the `IsSelectedCell` method. You can customize many aspects of the selection with these properties: `Grid.Selection.BackColor`, `Grid.Selection.Border`, `Grid.Selection.FocusBackColor`, ...
要列出所有選擇的單元格，您可以使用 `Grid.Selection.GetRanges()` 方法，该方法返回選擇的 Range 列表，或使用 `IsSelectedCell` 方法檢查特定單元格是否被選擇。您可以使用這些屬性： `Grid.Selection.BackColor` 、 `Grid.Selection.Border` 、 `Grid.Selection.FocusBackColor` 、...

You can also use these events to respond to specific action of the user: `Grid.Selection.FocusRowEntered`, `Grid.Selection.FocusRowLeaving`, `Grid.Selection.FocusColumnEntered`, `Grid.Selection.FocusColumnLeaving`, `Grid.Selection.CellLostFocus`, `Grid.Selection.CellGotFocus`, `Grid.Selection.SelectionChanged`, ...
您也可以使用這些事件來回應用戶的特定操作： `Grid.Selection.FocusRowEntered` 、 `Grid.Selection.FocusRowLeaving` 、 `Grid.Selection.FocusColumnEntered` 、 `Grid.Selection.FocusColumnLeaving` 、 `Grid.Selection.CellLostFocus` 、 `Grid.Selection.CellGotFocus` 、 `Grid.Selection.SelectionChanged` 、...

You can set the selection mode using the `Grid.SelectionMode` property. The available options are: GridSelectionMode.Cell, GridSelectionMode.Row and GridSelectionMode.Column. In this way you can set the grid to select only entire row, entire column or only single cell.
您可以透過 `Grid.SelectionMode` 屬性來設定選擇模式。可用的選項有：GridSelectionMode.Cell、GridSelectionMode.Row 和 GridSelectionMode.Column.。這樣您就可以將網格設定為僅選擇整行、整列或僅單一單元格。

To enable or disable multi selection you must use the `Grid.Selection.EnableMultiSelection` property. You can select multiple cells using the mouse or with the Ctrl or Shift key.
要啟用或停用多重選擇，您必須使用 `Grid.Selection.EnableMultiSelection` 屬性。您可以使用滑鼠或使用 Ctrl 或 Shift 鍵來選擇多個單元格。

## Position and Range 位置與範圍

Two of the most used objects in the project are the structures `Position` and `Range`. The struct `Position` identifies a position with a Row and a Column, while the struct `Range` identifies a group of cells delimited from a start `Position` and an end `Position`.
專案中最常用的兩個物件是結構 `Position` 和 `Range` 。結構 `Position` 識別一個位置，包含行和列，而結構 `Range` 識別一組由起點 `Position` 和終點 `Position` 界定的單元格。

You can read the actual location of a specified cell using `grid.PositionToRectangle` method. The resulting rectangle is relative to the grid client area. You can convert the resulting Rectangle to an absolute screen position using `grid.PointToScreen`.
您可以透過 `grid.PositionToRectangle` 方法讀取指定單元格的實際位置。所產生的矩形是相對於網格客戶區域。您可以將產生的 Rectangle 轉換為絕對螢幕位置使用 `grid.PointToScreen` 。

You can obtain the `Position` for a specific client area `Point` using `grid.PositionAtPoint` method.
您可以透過 `grid.PositionAtPoint` 方法取得特定客戶區域 `Point` 的 `Position` 。

## CellContext 單元格上下文

`CellContext` is a structure that encapsulates a `Cell` and a `Position` and contains all the method to manipulate the cells.
`CellContext` 是一個結構，封裝了一個 `Cell` 和一個 `Position` ，並包含所有用來操作單元格的方法。

The most important methods are:
最重要的方法有：

- StartEdit/EndEdit - To start/stop editing on a specified cell
  在指定格子上開始/停止編輯
- DisplayText - Returns the text of the cell (string)
  回傳格子的文字（字串）
- Value - Returns the value of the cell (object)
  回傳格子的值（物件）

Here a common example that show how to use a `CellContext` class:
這裡有一個常見的例子，展示如何使用一個 `CellContext` 類別：

```csharp
SourceGrid.CellContext context = new SourceGrid.CellContext(grid, new SourceGrid.Position(r, c));
context.Value = "hello";
context.StartEdit();
```

Usually a `CellContext` instance is automatically created as a parameter of the controller events, in this way you can always access the main cell properties.
通常一個 `CellContext` 實例會自動作為控制器事件的參數建立，這樣你就可以一直存取主細胞屬性。

# Advanced Features 進階功能

## Border 邊框

Each `View` class has a property `Border` of type `DevAge.Drawing.IBorder`. The `DevAge.Drawing.IBorder` is a generic interface that can be used to draw a border around the cell.
每個 `View` 類別都有一個型別為 `DevAge.Drawing.IBorder` 的 `Border` 屬性。 `DevAge.Drawing.IBorder` 是一個通用介面，可以用來在細胞周圍繪製邊框。

Usually the `IBorder` interface it is implemented by the `DevAge.Drawing.RectangleBorder` structure.
通常 `IBorder` 接口是由 `DevAge.Drawing.RectangleBorder` 结构實現的。

Below an example to change a border of a cell:
以下是一個更改單元格邊框的例子：

```csharp
DevAge.Drawing.Border border = new DevAge.Drawing.Border(Color.Red, 1);
DevAge.Drawing.RectangleBorder cellBorder = new DevAge.Drawing.RectangleBorder(border, border);

SourceGrid.Cells.Views.Cell view = new SourceGrid.Cells.Views.Cell();
view.Border = cellBorder;

grid[r, c].View = view;
```

The default border set only the Right and Bottom side, in this way when you have a group of cells you don't see a double border.
預設的邊框只設定右側和底側，這樣當你有一組單元格時，你就看不到重複的邊框。

You can also set the border for the grid using the `Grid.BorderStyle` property.
您也可以使用 `Grid.BorderStyle` 屬性來設定網格的邊框。

See form sample 26 for more information.
請參考表單範例 26 以獲取更多資訊。

## ToolTip 工具提示

You can bind a ToolTip on each cell. You must create a `SourceGrid.Cells.Controllers.ToolTipText` controller and associate it to a cell. Here an example:
您可以在每個單元格上綁定一個工具提示。您必須建立一個 `SourceGrid.Cells.Controllers.ToolTipText` 控制器並將其關聯到單元格。這裡有一個範例：

```csharp
SourceGrid.Cells.Controllers.ToolTipText toolTipController =
    new SourceGrid.Cells.Controllers.ToolTipText();
toolTipController.ToolTipTitle = "ToolTip example";
toolTipController.ToolTipIcon = ToolTipIcon.Info;
toolTipController.IsBalloon = true;

grid1[r, c] = new SourceGrid.Cells.Cell("Hello");
grid1[r, c].ToolTipText = "Example of tooltip, bla bla bla ....";
grid1[r, c].AddController(toolTipController);
```

The `ToolTipText` property of the cell automatically populates a `SourceGrid.Cells.Models.IToolTipText` interface bound to the standard cell.
該 `ToolTipText` 屬性會自動填入一個 `SourceGrid.Cells.Models.IToolTipText` 接口，該接口與標準單元相綁定。

![[CellToolTip.png]]

See form sample 26 for more information.
參考表單範例 26 以獲取更多資訊。

Note: To use the ToolTip on a GridVirtual you must define your custom `SourceGrid.Cells.Models.IToolTipText` implementation that read the value from your data source and then add the controller and the model to the virtual cell. See form sample 41.
注意：要在 GridVirtual 上使用 ToolTip，您必須定義您自己的 `SourceGrid.Cells.Models.IToolTipText` 實現，從您的數據來源讀取值，然後將控制器和模型添加到虛擬單元。請參考表單範例 41。

## ContextMenu 菜單列舉

You can create a ContextMenu (PopUp Menu) for a cell using the code below. First define a controller with the ContextMenu:
您可以使用以下程式碼為單元創建一個菜單列舉（彈出菜單）。首先定義一個具有菜單列舉的控制器：

```csharp
//Define a controller with a ContextMenu
public class PopupMenu : SourceGrid.Cells.Controllers.ControllerBase
{
	ContextMenu menu = new ContextMenu();
	public PopupMenu()
	{
        menu.MenuItems.Add("Menu 1", new EventHandler(Menu1_Click));
        menu.MenuItems.Add("Menu 2", new EventHandler(Menu2_Click));
	}

    public override void OnMouseUp(SourceGrid.CellContext sender, MouseEventArgs e)
    {
        base.OnMouseUp (sender, e);
    
        if (e.Button == MouseButtons.Right)
            menu.Show(sender.Grid, new Point(e.X, e.Y));
    }
    
    private void Menu1_Click(object sender, EventArgs e)
    {
        //TODO Your code here
    }
    private void Menu2_Click(object sender, EventArgs e)
    {
        //TODO Your code here
    }
}
```

Then add the controller to a cell:
然後將控制器加入單元格：

```csharp
PopupMenu menuController = new PopupMenu();
...
grid1[r, c] = new SourceGrid.Cells.Cell("Hello");
grid1[r, c].AddController(menuController);
```

See form sample 26 for more information.
請參考表單範例 26 以獲取更多資訊。

## Clipboard 剪貼簿

SourceGrid supports clipboard copy, paste and cut operations. To enable these features you can use this code:
SourceGrid 支援剪貼簿複製、貼上和剪下操作。要啟用這些功能，您可以使用這段程式碼：

```csharp
grid1.ClipboardMode = SourceGrid.ClipboardMode.All;
```

The user can then copy, paste and cut the selected cells using Ctrl+C, Ctrl+V or Ctrl+X or delete the content using Delete key.
使用者可以複製、貼上或剪下選定的單元格，使用 Ctrl+C、Ctrl+V 或 Ctrl+X，或使用刪除鍵刪除內容。

## Drag and Drop 拖放

**NOT SUPPORTED:** From version 4.5 drag and drop is no more supported, I will try to insert this feature again soon.
不支援：從版本 4.5 開始，拖放功能不再支援，我會盡快嘗試重新加入此功能。

# Suggestions 建議

Usually with SourceGrid you must do all the work with code, for this reason one import suggestion is to organize it in a good and reusable manner. Here some tips:
通常在使用 SourceGrid 時，您必須透過程式碼來完成所有工作，因此一個建議是將其組織得良好且可重用。這裡提供一些技巧：

If you need to customize some aspects of the cell I usually suggest to create a new class and write your customization code here. In this manner you can reuse your class in all your application and forms. Suppose for example that you want to create all the cells with a gray back color, you can write these new classes:
如果您需要自訂細胞的一些方面，我**通常建議您建立一個新的類別**，並將您的自訂程式碼寫在這裡。這樣您可以將您的類別重用在整個應用程式和表單中。例如，假設您想要建立所有具有灰色背景色的細胞，您可以寫這些新的類別：

```csharp
public class GrayView : SourceGrid.Cells.Views.Cell
{
	public new static readonly GrayView Default = new GrayView();
	public GrayView()
	{
		BackColor = Color.LightGray;
	}
}

public class GrayCell : SourceGrid.Cells.Cell
{
	public GrayCell(object val):base(val)
	{
		View = GrayView.Default;
	}
}
```

This rule can be applied to any aspects of the grid, for example you can create a new `Controller` class and use it in your cell.
這個規則可以套用於網格的任何方面，例如您可以建立一個新的 `Controller` 類別，並在您的單元格中使用它。

Another important feature that we can see in the previous code is that I have assigned the `View` property from a `static` variable. In this way you can share the same instance with many cells, and optimize the resources of the system.
在之前的程式碼中，我們可以看到另一個重要的功能是，我從一個 `static` 變數中指派了 `View` 屬性。這樣您可以與許多單元格共享相同的實例，並優化系統的資源。

Another little suggestion is about the use of namespaces. Many SourceGrid classes have the same name of some system classes. For example there is a `CheckBox` cell with the same name of the `CheckBox` control of `System.Windows.Forms`, for this reason is sometime difficult to use a normal using statement. If you don't like to use a long namespace and want a more readable code I suggest to rename the namespace in the using statement. For example you can replace this code:
另一個小建議是關於命名空間的使用。許多 SourceGrid 類別與某些系統類別有相同的名稱。例如，有一個 `CheckBox` 結構與 `CheckBox` 控制項的 `System.Windows.Forms` 有相同的名稱，因此有時使用正常的 using 語句會有些困難。如果你不喜歡使用長的命名空間並且希望程式碼更易讀，我**建議在 using 語句中重新命名命名空間**。例如，你可以用以下程式碼替換：

```csharp
SourceGrid.Cells.Button bt = new SourceGrid.Cells.Button();
```

為：

```csharp
using Cells = SourceGrid.Cells;
.................
Cells.Button bt = new Cells.Button();
```

# Extensions 擴充功能

Other then the default `Grid` control there are some extensions that can be used in some specific cases. You can directly use one of these extensions or you can copy the code and create your own extension.
除了預設的 `Grid` 控制元件外，有一些擴充功能可以在特定情況下使用。您可以直接使用這些擴充功能，或者您可以複製程式碼並建立您自己的擴充功能。

## DataGrid 資料網格

`SourceGrid.DataGrid` is control derived from `GridVirtual` used for binding the data to a `DevAge.ComponentModel.BoundListBase` class.
`SourceGrid.DataGrid` 是一個衍生的控制項，用於將資料綁定到 `DevAge.ComponentModel.BoundListBase` 類別。

The `BoundListBase` is an abstract class that can be used to as a generic layer to binding any kind of list control to a list data source.
`BoundListBase` 是一個抽象類別，可用作通用層，將任何種類的清單控制項綁定到清單資料來源。

Typically a data source is a `System.Data.DataView` or a custom list class. Currently there are 2 concrete classes that use `BoundListBase`:
通常，資料來源是一個 `System.Data.DataView` 或自訂清單類別。目前有兩個使用 `BoundListBase` 的具體類別：

- `DevAge.ComponentModel.BoundDataView` - Used for `System.Data.DataView`.
- `DevAge.ComponentModel.BoundList<T>` - Used for any `List<T>` class. You can use any kind of object with a default constructor and public property.
  用於任何 `List<T>` 類別。您可以使用任何具有預設建構函式和公共屬性的物件。

Basically the `DataGrid` control uses a special `Model` class of type `IValueModel` that reads the data directly from the data source. `SourceGrid.DataGrid` has a property `DataSource` used to store the `BoundListBase` object. Here is a simple example on how to use this control:
基本上， `DataGrid` 控制使用一種特殊類型為 `IValueModel` 的 `Model` 類別，直接從資料來源讀取資料。 `SourceGrid.DataGrid` 有一個屬性 `DataSource` 用於儲存 `BoundListBase` 物件。這裡有一個簡單的範例說明如何使用這個控制項：

```csharp
//Create a sample DataTable
DataTable table = new DataTable();
table.Columns.Add("A", typeof(string));
table.Columns.Add("B", typeof(bool));
table.Rows.Add(new object[]{"Row 1", false});
table.Rows.Add(new object[]{"Row 2", true});
table.Rows.Add(new object[]{"Row 3", false});

dataGrid1.DataSource = new DevAge.ComponentModel.BoundDataView(table.DefaultView);
```

In the previous code I have created a `DataTable` with 2 columns and some rows, then you can use the `DefaultView` property to retrieve a `DataView` class and assign it to the `DataGrid` control by creating an instance of a `BoundDataView` class. If you want you can customize the columns using the `Columns` property. This property returns a collection of `DataGridColumn` objects, you can customize each column to create customized cells. Here is an example:
在前面的程式中，我已創建了一個 `DataTable` ，包含 2 個欄位和一些列，接著您可以使用 `DefaultView` 屬性來取得 `DataView` 類別，並將其指派給 `DataGrid` 控制元件，方法是建立一個 `BoundDataView` 類別的實例。如果您需要，可以使用 `Columns` 屬性來自訂欄位。此屬性會傳回一組 `DataGridColumn` 物件的集合，您可以自訂每一欄來建立自訂的細胞。這裡有一個範例：

```csharp
//Create a custom View class
SourceGrid.Cells.Views.Cell view = new SourceGrid.Cells.Views.Cell();
view.BackColor = Color.LightBlue;
//Manually add the column
SourceGrid.DataGridColumn gridColumn;
gridColumn = dataGrid.Columns.Add("Country", "Country", typeof(string));
gridColumn.DataCell.View = view;
```

In this example I have manually created a column and assigned a LightBlue back color at the cell.
在這個範例中，我手動建立了一個欄位，並將單元格的背景色指定為淺藍色。

You can also use the `DataGridColumn.Conditions` to dynamically change the data cell. For example in the following code I create a condition to use a different cell backcolor for even rows (alternate backcolor):
您也可以使用 `DataGridColumn.Conditions` 來動態變更資料格。例如在以下程式碼中我建立一個條件來使用不同的格背景色來區分偶數列（交錯背景色）：

```csharp
SourceGrid.Conditions.ICondition condition =
    SourceGrid.Conditions.ConditionBuilder.AlternateView(gridColumn.DataCell.View, Color.LightGray, Color.Black);
gridColumn.Conditions.Add(condition);
```

In the following other example I create condition to use a View with a bold font and a green forecolor when a specific column is true.
在以下其他範例中，我建立條件以使用具有粗體字體和綠色前景色的 View，當特定欄位為 true 時。

```csharp
SourceGrid.Conditions.ConditionView selectedConditionBold =
    new SourceGrid.Conditions.ConditionView(viewSelected);
selectedConditionBold.EvaluateFunction =
    delegate(SourceGrid.DataGridColumn column, int gridRow, object itemRow)
    {
        DataRowView row = (DataRowView)itemRow;
        return row["Selected"] is bool && (bool)row["Selected"] == true;
    };
gridColumn.Conditions.Add(selectedConditionBold);
```

Currently there are 2 types of conditions: `SourceGrid.Conditions.ConditionView` that can be used to use a special view and `SourceGrid.Conditions.ConditionCell` to use a completely different cell. Both conditions can be used with a delegate to evaluate the row using your specific code.
目前有 2 種條件： `SourceGrid.Conditions.ConditionView` 可用於使用特殊視圖，以及 `SourceGrid.Conditions.ConditionCell` 可用於使用完全不同的單元格。這兩種條件都可以與委派搭配使用，以使用您的特定程式碼評估列。

In the sample project you can find more examples and code.
在範例專案中可以找到更多範例和程式碼。

## ArrayGrid 陣列網格

`SourceGrid.ArrayGrid` is a control derived from `GridVirtual` used to bind the grid to an array. You can use this control setting the `DataSource` property with any `System.Array`. You can then customize the cells using the `ValueCell, Header, RowHeader and ColumnHeader` properties.
`SourceGrid.ArrayGrid` 是一個衍生的控制元件 `GridVirtual` ，用於將網格綁定到陣列。您可以透過此控制元件設定 `DataSource` 屬性，並使用任何 `System.Array` 。接著，您可以使用 `ValueCell, Header, RowHeader and ColumnHeader` 屬性來自訂單元格。

## PlanningGrid 規劃網格

`SourceGrid.Planning.PlanningGrid` control is a `UserControl` that internally use a real `Grid` to create a grid that can be used to show appointments. You can call the `LoadPlanning` method to set the range of the visible days and hours, and then add the appointments using the `Appointments` properties. This is a collection of `IAppointment` interface (implemented by the `AppointmentBase` class).
`SourceGrid.Planning.PlanningGrid` 控制元件是一個 `UserControl` ，內部使用一個實際的 `Grid` 來建立一個可以顯示預約的網格。您可以呼叫 `LoadPlanning` 方法來設定可見天日期和時間的範圍，然後使用 `Appointments` 屬性來添加預約。這是一個 `IAppointment` 接口（由 `AppointmentBase` 類別實作）的集合。

Each appointment has a `View` property that can be used to customize the appointment cell.
每個預約都有一個 `View` 屬性，可用於自訂預約單元。

# References 參考資料

DevAgeSourcePack references these assemblies:
DevAgeSourcePack 參考這些組件：

- `DevAge.Core.dll, DevAge.Windows.Forms.dll` - An open source library with many commons features for .NET. Contains the core of the drawing library and some of the controls used as editors. You can find this library at [DevAgeSourcePack article](http://www.devage.com/Wiki/ViewArticle.aspx?name=devagesourcepack&version=0)
  一個開源程式庫，包含許多 .NET 的通用功能。包含繪圖程式庫的核心以及一些用作編輯器的控制項。您可以在 DevAgeSourcePack 文章中找到這個程式庫。

Remember to redistribute these assemblies with SourceGrid.dll.
請記得與 SourceGrid.dll 一起重新分發這些組件。

# Future developments 未來發展

- Improve performance.
  提升效能。
- Support for other platform: PocketPC, Mono, ...
  支援其他平台：PocketPC、Mono、...
- Better support for Windows Forms designer and other WYSIWYG features.
  更好的支援 Windows Forms 設計器及其他 WYSIWYG 功能。

# VB.NET

As any .NET control you can use SourceGrid with any .NET language, also VB.NET. Unfortunately I don't have any example for VB.NET, consider anyway that the code is very similar to C#. Here some of the more common differences:
如同任何 .NET 控制項，您可以使用 SourceGrid 搭配任何 .NET 語言，包括 VB.NET。不過，我沒有 VB.NET 的範例，但請考慮到代碼與 C# 非常相似。這裡列出一些較常見的差異：

| C#                                                                                                                                                                                                                                                                                                                                                         | VB.NET                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| double x;                                                                                                                                                                                                                                                                                                                                                  | Dim x as Double                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| typeof(string)                                                                                                                                                                                                                                                                                                                                             | GetType(String)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Class.Event += new EventHandler(Class_Event);  <br>類別.Event += new EventHandler(類別 _ 事件);                                                                                                                                                                                                                                                                  | AddHandler Class.Event, Addressof Class_Event  <br>AddHandler 類別.Event, Addressof 類別 _ 事件                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| grid[r,c]                                                                                                                                                                                                                                                                                                                                                  | grid.Item(r,c)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| public class Dog<br>{<br>    public Dog()<br>    {<br>    }<br>    public string Name<br>    {<br>        get{return mName;}<br>        set{mName = value;}<br>    }<br>}<br>                    public class Dog<br>{<br>    public Dog()<br>    {<br>    }<br>public string Name<br>    {<br>get{回傳 mName;}<br>        set{mName = value;}<br>    }<br>} | Public class Dog<br>    Public sub New()<br>    <br>    End sub<br>    <br>    Public Property Name() as String<br>        Get<br>            Return mName<br>        End Get<br>        Set<br>            mName = Value<br>        End Set<br>    End Property<br>End class<br>                    Public class Dog<br>Public Sub New()<br><br>    End Sub<br><br>    Public Property Name() As String<br>        Get<br>回傳 mName<br>        End Get<br>        Set<br>            mName = Value<br>結束設定<br>    結束屬性<br>結束類別 |
| List\<string\> x;                                                                                                                                                                                                                                                                                                                                          | Dim x as List(Of String)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

# FAQ 常見問題集

Frequently Asked Questions:

Q: **Can I use this project with Visual Studio 2005?**
A: Yes, SourceGrid is compiled with .NET 2 and Visual Studio 2005.

Q: **Can I use this project with Visual Studio 2003?**
Q: 我可以使用這個專案與 Visual Studio 2003 嗎？
A: No, SourceGrid is compiled with .NET 2 and Visual Studio 2005. If you want you can use an older version (like SourceGrid 3).
A: 不可以，SourceGrid 是使用 .NET 2 和 Visual Studio 2005 編譯的。如果你想要，你可以使用較舊的版本（例如 SourceGrid 3）。

Q: **How can I create a tree view with SourceGrid? (hierarchical view)**
Q: 我如何可以使用 SourceGrid 創建樹狀視圖？（分層視圖）
A: For now SourceGrid doesn't support this kind of view. I hope to work on this for a future release.
A: 目前 SourceGrid 不支援這種視圖。我希望能在未來的釋出中處理這個問題。

Q: **Can I use these controls with ASP.NET?**
Q: 我可以在 ASP.NET 中使用這些控制項嗎？
A: No, these are Windows Forms Controls. With Internet Explorer you can host a Windows Forms Control inside an html page, but it is usually not a good solution for web pages.
A: 不，這些是 Windows Forms 控制項。使用 Internet Explorer 你可以在 html 頁面中主機一個 Windows Forms 控制項，但這通常不是為網頁的良好解決方案。

Q: **Can I use these controls with .NET Compact Framework? And with Mono?**
Q: 我可以在 .NET Compact Framework 中使用這些控制項嗎？還有在 Mono 中呢？
A: Currently this project is compatible only with Microsoft .NET Framework and Windows platforms. I'm working for porting some of the code to Mono and Compact Framework. Consider anyway that all the code is written with managed C# so probably it is not too difficult to use this library with other frameworks.
A: 目前此專案僅與 Microsoft .NET Framework 和 Windows 平台相容。我正在將部分程式碼移植到 Mono 和 Compact Framework。不過，請考慮的是，所有程式碼都是使用受控 C# 寫成的，所以可能使用這個程式庫與其他框架並不困難。

# Special Thanks 特別感謝

A special thanks to all who have helped to make this project with suggestions, bug reports, ideas and code.
感謝所有對此專案提供建議、錯誤報告、點子和程式碼的人。
I've also benefited greatly from:
我也從以下人員獲益匪淺：

- Microsoft Sandcastle project and Sandcastle builder ([http://www.codeplex.com/SHFB](http://www.codeplex.com/SHFB)) that I have used to build the chm documentation.
  Microsoft Sandcastle 專案和 Sandcastle 建置工具（ http://www.codeplex.com/SHFB），我使用它來建置 chm 文件。

# History 歷史

## 4.11 (27 Nov 2007)

- IMPORTANT: Optimized drawing code to draw only the invalidated region (thanks to Xentor for the suggestions).
    重要：優化繪圖程式碼，僅繪製無效區域（感謝 Xentor 的建議）。
- IMPORTANT: Optimized selection code to invalidated only the changed region (thanks to Xentor for the suggestions).
    重點：優化選擇程式碼，僅無效變更區域（感謝 Xentor 的建議）。
- Changed the RangeRegion class to be directly a collection (ICollection) of Range objects. Removed the GetRanges method that now is no more required because you can directly access and iterate the Range using the RangeRegion class.
    將 RangeRegion 類別改為直接為 Range 物件的集合 (ICollection)。移除了 GetRanges 方法，因為現在不再需要，可以直接使用 RangeRegion 類別存取和迭代 Range。
- Changed the SelectionChanged event to use the new RangeRegionChangedEventArgs.
    將 SelectionChanged 事件改為使用新的 RangeRegionChangedEventArgs.。
- Fixed the DataGrid to check if the edit of the datasource is enabled (AllowEdit) when editing rows.
    修復了 DataGrid，在編輯行時會檢查來源資料是否啟用編輯 (AllowEdit)。

## 4.10 (16 Oct 2007)

- Fixed a bug in the RecalcCustomScrollBars when moving the focus on a partial visible row/column and both scrollbars are visible. Now I correctly calculate the rows to scroll.
    修復了在移動焦點至部分可見的列/行時，且兩個捲軸都可见時，RecalcCustomScrollBars 中的錯誤。現在我能正確計算要捲動的列數。
- Added the DataGridRows.DataSourceIndexToGridRowIndex method to convert a DataSource index to a grid row index.
    新增了 DataGridRows.DataSourceIndexToGridRowIndex 方法，用於將 DataSource 索引轉換為網格行索引。
- Added the property ElementsDrawMode to the ViewBase class to support 'covering' drawing mode for the elements.
    為 ViewBase 類別新增了屬性 ElementsDrawMode，以支援元素「覆蓋」繪圖模式。
- Changed the order of the event dispatched to fix some problems when you remove, add or change a row inside an event (for example MouseDown, MouseUp, Click). Now I first process system events and then custom events.
    修改了事件派發的順序，以解決在移除、新增或更改事件內的行時（例如 MouseDown、MouseUp、Click）所出現的一些問題。現在我先處理系統事件，然後再處理自訂事件。
- Fixed a bug on the ComboBox control when using a value not in the list and enter and leave the editing from another ComboBox.
    修正了 ComboBox 控制項的錯誤，當使用列表中不存在的值並從另一個 ComboBox 離開編輯狀態時。
- Fixed a display bug in the editors to show the control only after changing the value to reduce the flickering.
    修正了編輯器中的顯示錯誤，僅在更改值後才顯示控制項，以減少閃爍現象。
- Fixed a bug when drawing merged cells, to not draw many times the same cells
    修正了繪製合併單元格時的錯誤，避免多次繪製相同的單元格。
- Updated the DevAgeSourcePack references to version 4.10 to fix some bugs (see DevAgeSourcePack History section for details)
    更新 DevAgeSourcePack 的參考版本為 4.10 以修正一些錯誤（詳情請參閱 DevAgeSourcePack 歷史部分）

## 4.9 (17 June 2007)

4.9 (2007 年 6 月 17 日)

- **IMPORTANT:**Added the ComboBox View to draw a ComboBox on the cell not only when activating the editor. See form 14 for an example.
    重要：新增 ComboBox View，讓在單元格上繪製 ComboBox 不僅僅在啟動編輯器時才進行。請參考表單 14 為例。
- Fixed a bug in the Rows/Columns to block AutoSize and Stretch for invisible columns/rows.
    修正了 Rows/Columns 的錯誤，以阻止對不可見的列/行進行 AutoSize 和 Stretch。
- Fixed a bug in the ColumnsInsideRegion and RowsInsideRegion methods for invisible columns/rows
    修正了在 ColumnsInsideRegion 和 RowsInsideRegion 方法中對於不可見欄位/列的 bug
- Extended sample 41 to show how to use select a DataGrid row and show the current selected row.
    擴展範例 41 以展示如何選擇一個 DataGrid 列，並顯示當前選擇的列。
- Fixed a strange bug when using Selection.Focus(Position.Emtpy) that cause the grid to receive the focus.
    修正了在使用 Selection.Focus(Position.Emtpy) 時出現的奇怪 bug，該 bug 會導致網格獲得焦點。
- Fixed a bug in the RangeData.LoadData method to check if a value can be converted to string, used for example with the clipboard copy operation.
    修正了在 RangeData.LoadData 方法中的 bug，該方法用於檢查是否可以將值轉換為字串，例如用於剪貼板複製操作。
- Updated the DevAgeSourcePack references to version 4.9 to fix some bugs (see DevAgeSourcePack History section for details)
    更新 DevAgeSourcePack 的參考至版本 4.9 以修正一些錯誤（詳情請參閱 DevAgeSourcePack 歷史部分）
- Fixed a bug in the scroll code with big fixed rows or columns, thanks to Sacha.
    修正了在具有大固定行或列的捲動程式碼中的錯誤，感謝 Sacha。
- Fixed a bug when exporting the grid to an image with merged cells (RowSpan/ColumnSpan > 0).
    修正了在將資料表匯出為圖像時，若合併的細胞（RowSpan/ColumnSpan > 0）出現錯誤的問題。
- Added a Grid property to the PlanningGrid to get the internal grid instance.
    在 PlanningGrid 中新增了一個 Grid 屬性，用於取得內部資料表實例。
- Restored the old code for the Button or Link cell using the KeyDown event to fire the action following the system Button behavior.
    使用 KeyDown 事件恢復舊的按鈕或連結單元格的代碼，以符合系統按鈕的行為。
- Marked as obsolete the CellControl class replaced by the Grid.LinkedControls collection.
    將 CellControl 類別標記為過時，已被 Grid.LinkedControls 集合取代。
- Fixed a bug when using RemoveFocusCellOnLeave (to don't reset the selection)
    修復了使用 RemoveFocusCellOnLeave 時的錯誤（不重置選擇）。
- Fixed a bug when drawing the selection border and there is a single selected range
    修復了在繪製選擇邊框且僅有一個選擇範圍時的錯誤。
- Fixed a bug when using FocusFirstCellOnEnter to move the focus on the selection if there is a valid selected range (FocusFirstCell method)
    修正了使用 FocusFirstCellOnEnter 移動焦點至選擇範圍的錯誤（FocusFirstCell 方法）
- Added the ignorePartial parameter in the ShowCell method to don't move the scrollbars when the user move the focus on a partial visible cell.
    在 ShowCell 方法中新增了 ignorePartial 參數，用於當使用者將焦點移至部分可見的單元格時，不會移動滾動條。
- Fixed a bug in the Selection.ResetSelect and in the Selection.Focus when removing cells
    修正了 Selection.ResetSelect 和 Selection.Focus 在移除單元格時的錯誤

## 4.8 (6 May 2007)

- Updated the DevAgeSourcePack references to version 4.8 to fix some bugs (see DevAgeSourcePack History section for details)
    更新 DevAgeSourcePack 的參考版本為 4.8 以解決一些錯誤（詳情請參閱 DevAgeSourcePack 歷史部分）
- Renamed IBindedList to IBoundList
    將 IBindedList 改名為 IBoundList
- Renamed BindedListBase to BoundListBase
    將 BindedListBase 改名為 BoundListBase
- Renamed BindedList to BoundList
    將 BindedList 改名為 BoundList
- Renamed BindedDataView to BoundDataView
    改名為 BoundDataView
- Fixed a bug in the TextBoxUITypeEditor when the text is not valid, especially for the DateTime editor (1615918).
    修復了 TextBoxUITypeEditor 中的錯誤，當文本無效時，特別是對於 DateTime 編輯器 (1615918)。
- Fixed a bug in the AutoSize and Measure of merged cells that caused the merged cells to not autosize correctly (1601792).
    修復了合併單元格的 AutoSize 和 Measure 中的錯誤，導致合併單元格無法正確自動調整大小 (1601792)。
- Signed SourceGrid assemblies with the .snk file
    使用 .snk 檔案簽署了 SourceGrid 組件
- Replaced the BindedListBase with the IBoundList interface in the DataGrid implementation
    將 DataGrid 實現中的 BindedListBase 替換為 IBoundList 接口
- Added a `Visible` property to the `ColumnInfo` and `RowInfo` class to show or hide columns or rows. For example you can hide a column with this code: `grid1.Columns[0].Visible = false;`. See form 50 for an example. (1569378)
    在 `ColumnInfo` 和 `RowInfo` 類中新增了 `Visible` 屬性，用於顯示或隱藏欄位或列。例如，您可以使用這段程式碼 `grid1.Columns[0].Visible = false;` 來隱藏一個欄位。請參考表單 50 為例。(1569378)
- Fixed a bug in the CellControl.UnBindToGrid method, used when removing or replacing CellControl cells.
    修復了 CellControl.UnBindToGrid 方法中的錯誤，該方法用於移除或替換 CellControl 結構。
- Added support for nullable checkbox value. Now when the value of a checkbox is null the Undefined checkbox status is used. The Checked property now returns a Nullable bool value (bool?).
    新增了可為空的核取方塊值支援。現在當核取方塊的值為 null 時，會使用 Undefined 的核取方塊狀態。Checked 屬性現在回傳一個可為 null 的布林值 (bool?)。
- Fixed documentation English errors
    修正文件說明英文錯誤
- Added `HeaderHeight` property in the `DataGridRows` class, which can be used to set the header height of the `DataGrid` control. (1604395)
    在 `DataGridRows` 類別中新增了 `HeaderHeight` 屬性，可用於設定 `DataGrid` 控件的標題高度。(1604395)

## 4.7 (16 April 2007)

4.7 (2007 年 4 月 16 日)

- Fixed a bug in the scroll code (ShowCell method)
    修正了捲動程式碼 (ShowCell 方法) 的錯誤
- Extended the Controller documentation
    擴展了 Controller 文件說明

## 4.6 (15 April 2007)

4.6 (2007 年 4 月 15 日)

- Fixed the SourceGrid.Cells.Views.Button to show when the button has the focus.
    修正了 SourceGrid.Cells.Views.Button 在按鈕獲得焦點時才顯示的問題。
- Fixed a bug when using a Button or Link cell that cause the execute event to be fired many times when you hold the key Space. Now I have handled the event on the KeyUp and added the Enter key (before I used the KeyPress).
    修正了使用按鈕或鏈接單元格時的錯誤，當您按住空格鍵時，執行事件會被觸發多次。現在我在 KeyUp 上處理了事件，並添加了 Enter 鍵（之前我使用的是 KeyPress）。
- Removed the border default padding to fix some bugs on the cell borders
    移除了邊框的預設填充，以修正單元格邊框的一些錯誤
- Updated the DevAgeSourcePack references to version 4.6
    更新了 DevAgeSourcePack 的參考版本為 4.6
- Fixed the scrollbars to scroll only the required columns/rows and correctly stop at the end
    修復了滾動條，使其僅滾動所需的列/行，並正確地在末尾停止
- Fixed a bug on the Measure method used to AutoSize the columns and rows size
    修復了 Measure 方法中用於自動調整列和行大小的錯誤
- Added the CHM documentation api reference using Sandcastle
    使用 Sandcastle 添加了 CHM 文件說明文件 API 參考

## 4.5 BETA (18 March 2007)

4.5 BETA (2007 年 3 月 18 日)

This is a major release with many inportant changes and improvements. Consider that this is a BETA release and probably there are still some bugs and problems. This version can can be unstable. Use the forum for submitting bugs or if you have problems.
這是一個主要發行版本，包含許多重要的變更和改進。請考慮這是一個 BETA 版本，可能仍然存在一些錯誤和問題。此版本可能不穩定。請使用論壇提交錯誤或如果您有問題。

- **IMPORTANT:** New structure of the selection class. See 'Focus and Selection' paragraph in this article for more information.
  重要：選擇類別的新結構。請參閱本文中的「焦點和選擇」段落以獲取更多信息。
- **IMPORTANT:** New SourceGrid.DataGrid class that now support also custom object binding and reviewed some code. See DataGrid section for more information.
  重要：新增 SourceGrid.DataGrid 類別，現支持自訂物件綁定並檢閱部分程式碼。請參閱 DataGrid 節以獲取更多資訊。
- **IMPORTANT:** New internal structure of the SourceGrid.Grid class, to store rows and columns information in a list object for better performance.
    重要：新增 SourceGrid.Grid 類別的內部結構，用於在清單物件中儲存行和列資訊，以提升效能。
- **IMPORTANT:** Removed all the Panels structure (GridSubPanel class).
    重要：已移除所有 Panels 結構 (GridSubPanel 類別)。
- **IMPORTANT:** Reviewed positioning methods
    重要：檢閱了定位方法
- **IMPORTANT:** Reviewed view methods to customize sub elements (IText, IImage, ...)
    重要：檢查了可自訂子元素（IText、IImage、...）的檢視方法
- **IMPORTANT:** Reviewed grid controllers and events structure.
    重要：檢查了網格控制器和事件結構。
- **IMPORTANT:** Events and methods more integrated with the standard .NET conventions
    重要：事件和方法更整合標準的 .NET 常規
- **IMPORTANT:** Updated DevAgeSourcePack references to version 4.3
    重要：更新了 DevAgeSourcePack 的參考版本為 4.3
- **IMPORTANT:** Removed many obsolete code
    重要：已移除許多過時的程式碼
- **IMPORTANT:** Removed drag and drop support (I will try to implement it again soon)
    重要：已移除拖曳放置支援（我會盡快再實作它）
- Fixed a bug on handling TabStop property and Tab navigation. Now the grid move the focus on the other controls of the form when required.
    修正了處理 TabStop 屬性和 Tab 鎖定導航的錯誤。現在，當需要時，網格會將焦點移至表單的其他控制項。
- Added the property `Editor.UseCellViewProperties`. The default is true. Set this property to false to manually change the editor control ForeColor, BackColor and Font, otherwise the editor read these properties from the View class.
    新增了屬性 `Editor.UseCellViewProperties` 。預設為 true。將此屬性設為 false 以手動更改編輯器控制項的前景色、背景色和字型，否則編輯器會從視圖類別讀取這些屬性。
- Added the event `Editor.KeyPress` that can be used to handle any key press of the editor (also when the user start the edit operation with a key)
    新增了事件 `Editor.KeyPress` ，可用來處理編輯器中的任何鍵盤按鍵壓下（也包含當使用者以鍵盤開始編輯操作時）
- Fixed a bug in the ComboBox when using DropDownList and custom display text
    修正了 ComboBox 在使用 DropDownList 和自訂顯示文字時的 bug
- Removed resizable headers for ArrayGrid
    移除了 ArrayGrid 的可調整標題
- Fixed a bug in the NumericUpDown editor to correctly use Maximum, Minimum and Increment variables
    修正了 NumericUpDown 編輯器在正確使用 Maximum、Minimum 和 Increment 變數時的 bug
- Fixed a bug when changing the active cell and in the code executed when the grid lost the focus. Now only if requred the focus is moved inside the grid. The bug caused the grid to receive the focus when calling grid.Selection.Focus(Empty). See Selection.Focus method for more information.
    修正了在切換活動單元格以及當網格失去焦點時執行的程式碼中的錯誤。現在只有在需要時才會將焦點移至網格內。這個錯誤導致了在呼叫 grid.Selection.Focus(Empty) 時網格接收焦點。請參考 Selection.Focus 方法以獲取更多資訊。
- Added an example on form 3 that use GDI+ drawing features.
    在表單 3 中新增了一個使用 GDI+ 繪圖功能的範例。
- Changed the clipboard implementation. See Clipboard section for more information.
    更改了剪貼板實現。請參考剪貼板節以獲取更多資訊。
- Fixed a bug on some Views copy constructor (clone).
    修正了某些 Views 的拷貝建構函式（克隆）中的錯誤。
- Added to the form 3 a sample to to implement an editor to select a file using OpenFileDialog.
    新增至表單 3 一個範例，以實作一個編輯器來選擇檔案使用 OpenFileDialog。
- Changed the PlanningGrid control to don't stretch or autosize the first 2 column headers.
    已將 PlanningGrid 控制項變更為不拉伸或自動調整前兩個列標題的大小。
- Improved the PlanningGrid control. Added the PlanningGrid.AppointmentClick and PlanningGrid.AppointmentDoubleClick events that can be used when the used click on the grid and reviewed some of the code.
    已改善 PlanningGrid 控制項。新增了 PlanningGrid.AppointmentClick 和 PlanningGrid.AppointmentDoubleClick 事件，這些事件可以在使用者點擊表格時使用，並檢查了一些程式碼。
- Moved the SelectionMode property from grid.Selection.SelectionMode to grid.SelectionMode
    已將 SelectionMode 屬性從 grid.Selection.SelectionMode 移至 grid.SelectionMode。
- Removed the GridVirtual.InvalidateCells method (use directly the GridVirtual.Invalidate)
    刪除了 GridVirtual.InvalidateCells 方法（直接使用 GridVirtual.Invalidate）
- New schema version with automatic revision and build numbers
    新的 schema 版本，具有自動修訂和建置編號
- Removed the selection events on the cell Controller. If you need to customized the selection you must directly use the Selection object.
    刪除了單元 Controller 上的選擇事件。如果您需要自訂選擇，必須直接使用 Selection 物件。
- New structure for the LinkedControlList (Editors and CellControl)
    LinkedControlList 的新結構（編輯器和單元 Control）
- Added the sample form 49 to show the use of the DataGrid bound with custom entities
    新增範例表單 49 以展示與自訂實體綁定的 DataGrid 的使用方式
- Reviewed sample code and documentation
    檢閱範例程式碼與文件
- Removed the StyleGrid property, style no more supported
    移除 StyleGrid 屬性，風格不再支援

## 4.0.0.4 BETA (10 Nov 2006)

- Update references to DevAgeSourcePack 4.0.0.8. (Removed ODL.dll reference)
    更新 DevAgeSourcePack 4.0.0.8 的參考。(已移除 ODL.dll 參考)
- Improved View classes for a better customization. Now support flat headers, not themed headers, alternate back color, linear gradient, .... See [View](https://app.immersivetranslate.com/html/#View) and form 28 or 48 for an example. (Request ID: 1571395)
    改良 View 類別以進行更好的自訂。現在支援扁平化的標題、非主題化的標題、交替的背景色、線性漸層等。請參考 View 和表單 28 或 48 為例。(請求 ID: 1571395)
- Fixed a bug when drawing merged cells (ColSpan or RowSpan > 1) (Request ID: 1588870).
    修正了在繪製合併的單元格時的錯誤 (ColSpan 或 RowSpan > 1) (請求 ID: 1588870)。
- Fixed a bug to show the scrollbars only when necessary (Request ID: 1584957)
    修正了僅在必要時才顯示滾動條的錯誤 (請求 ID: 1584957)
- Fixed a bug in the MouseSelection class to don't execute again the Editor.EndEdit, because it is already executed inside the Controllers.Grid
    修正了 MouseSelection 類別中的錯誤，以避免再次執行 Editor.EndEdit，因為它已在 Controllers.Grid 中執行
- Changed the View.Border property to return an IBorder instance. The previous RectangleBorder class implements the IBorder interface, but this interface can also be implemented by different border style. See also See [Border](https://app.immersivetranslate.com/html/#Border)
    將 View.Border 屬性變更為回傳 IBorder 實例。之前的 RectangleBorder 類別實作了 IBorder 接口，但這個接口也可以由不同的邊框風格實作。另請參閱 See Border
- Fixed a bug to correctly call GiveFeedback event, and some other minors improvements. (Request ID: 1585697)
    修正了正確呼叫 GiveFeedback 事件的錯誤，以及一些其他的微小改進。(請求 ID: 1585697)
- Fixed the Button and CheckBox controllers to execute the action only when the left mouse button is pressed.
    修正了 Button 和 CheckBox 控制器，使其只在按下滑鼠左鍵時才執行動作
- Added new features for the ToolTip. Now you can set the Icon, Title, BackColor and ForeColor properties of the ToolTipText Controller. See [ToolTip](https://app.immersivetranslate.com/html/#ToolTip) or form sample 26 for more information.
    新增了工具提示的功能。現在您可以設定工具提示文字控制器的 Icon、Title、BackColor 和 ForeColor 屬性。請參考工具提示或表單範例 26 以獲取更多資訊。
- Removed the GridToolTipActive property (no more used, probably you must remove the property from the designer generated code).
    移除了 GridToolTipActive 屬性（不再使用，您可能需要從設計器產生的程式碼中移除此屬性）。
- Improved AutoScroll and IsVisibleCell test. Now if a cell is partially visible the autoscroll is not executed.
    改進了 AutoScroll 和 IsVisibleCell 測試。現在如果一個格子部分可見，自動捲動不會執行。

## 4.0.0.3 BETA (13 October 2006)

4.0.0.3 BETA (2006 年 10 月 13 日)

- Fixed a bug in the invalidate method that caused the cells to don't refresh correctly.
    修正了 invalidate 方法中的錯誤，該錯誤導致單元格無法正確刷新。
- Fixed a bug in the editors when start editing with a keybord key. Now only the TextBox use the key. I have removed the SendKeys features because some problems on international keybords.
    修正了編輯器中的錯誤，當使用鍵盤按鍵開始編輯時會出現問題。現在只有 TextBox 使用鍵盤按鍵。我已移除 SendKeys 功能，因為國際鍵盤存在一些問題。
- Fixed a bug when using the & character inside a cell text.
    修正了在單元格文本中使用 & 字符時的錯誤。
- Update references to DevAgeSourcePack 4.0.0.5
    更新了對 DevAgeSourcePack 4.0.0.5 的引用。
- Improved the mouse multi selection code and automatic scrolling.
    改進了滑鼠多選擇程式碼和自動捲動。

## 4.0.0.2 BETA (08 October 2006)

- Changed some internal structure of the View class for better flexibility.
    修改了 View 類別的一些內部結構，以提升彈性。
- Extended "Sample 3" to show how to implement rotated text (see RotateTextView).
    擴展了「範例 3」，展示如何實現旋轉文字（請參考 RotateTextView）。
- Fixed some important bugs for the scrollbars.
    修正了滾動條的一些重要錯誤。
- Reviewed many internal parts to handle scrolling and to calculate positions of the cells. Now all the cell positions are relative to the current view, so there is no more the concept of an aboslute cell position. This is usefull to correctly handle grid with many rows.
    檢查了許多內部部分以處理滾動並計算單元格的位置。現在所有單元格的位置都是相對於當前視圖的，因此不再有絕對單元格位置的概念。這對於正確處理具有多行的大型網格非常有用。
- Implemented the AutoSizeMode features also for GridVirtual derived grids.
    也為 GridVirtual 派生的網格實現了 AutoSizeMode 功能。
- Updated references to DevAgeSourcePack 4.0.0.4
    更新了 DevAgeSourcePack 4.0.0.4 的參考。
- Fixed a bug in the AutoSize methods.
    修正了 AutoSize 方法中的錯誤。

## 4.0.0.1 BETA (01 October 2006)

SourceGrid 4 is the evolution of the SourceGrid 3 control. The major differences are:
SourceGrid 4 是 SourceGrid 3 控件的演變。主要差異在於：

- Some minor corrections for VS 2005
    對 VS 2005 進行了些許修正。
- Replaced Grid.AutoSize method with Grid.AutoSizeCells
    替換了 Grid.AutoSize 方法為 Grid.AutoSizeCells
- Changed the default selection border width to 2 pixels
    將預設選擇邊框寬度改為 2 像素
- Changed the ComboBoxType control with the DevAgeComboBox. DevAgeComboBox is a new class that internally use the system ComboBox for a better compatibility and a more robust code support.
    將 ComboBoxType 控制項替換為 DevAgeComboBox。DevAgeComboBox 是一個新的類別，內部使用系統的 ComboBox 以獲得更好的相容性，並提供更穩固的程式碼支援。
- Replaced the TypedComboBox, TypedTextBox and other windows form controls with the DevAgeCombo, DevAgeTextBox, ... for better compatibility using system controls.
    替換了 TypedComboBox、TypedTextBox 和其他 Windows Forms 控制項為 DevAgeCombo、DevAgeTextBox、... 以使用系統控制項來提高相容性。
- Review some of the code for the Validator classes and added a Changed event.
    檢視部分 Validator 類別的程式碼，並新增了一個 Changed 事件。
- Fixed some error when display UITypeEditor controls.
    修正了在顯示 UITypeEditor 控制項時的一些錯誤。
- Added a RectangleBorderRelative property to the Selection and the HighlightedRange class, used to fix a problem when drawing range on fixed cols or rows
    為 Selection 和 HighlightedRange 類別新增了一個 RectangleBorderRelative 屬性，用於解決在固定欄或列上繪製範圍時的問題。
- Added MultiColumnsComparer, to sort multiple columns automatically
    新增了 MultiColumnsComparer，用於自動排序多個欄位。
- Added Selection.MoveActiveCell method to move the focus to a specific direction. Used with the arrow and tab keys.
    新增了 Selection.MoveActiveCell 方法，用於將焦點移至特定方向。搭配箭頭鍵和 Tab 鍵使用。
- Added the new DevAge.Drawing.VisualElements classes to draw elements parts (image, text, xp theme element, ...). This is basically a little drawing framework that support aggregation of parts of elements.
    新增了新的 DevAge.Drawing.VisualElements 類別，用於繪製元素部分（圖片、文字、xp 主题元素等）。基本上是一個小型的繪圖框架，支援元素部分的聚合。
- Used the new DevAge.Drawing.VisualElements for all SourceGrid cells drawing.
    所有 SourceGrid 結點的繪製都使用了新的 DevAge.Drawing.VisualElements。
- Removed the DevAge.WindowsPlatform project. Now all the XP theme support is 100% managed (no Interop or windows API).
    移除了 DevAge.WindowsPlatform 專案。現在所有 xp 主题支援都完全由 100% 管理（無需 Interop 或 windows API）。
- Merged the DevAge.Data and DevAge.Utilities with the DevAge.Core project.
    合併了 DevAge.Data 和 DevAge.Utilities 到 DevAge.Core 專案。
- Merged the DevAge.Drawing project with the DevAge.Windows.Forms project.
    合併了 DevAge.Drawing 專案到 DevAge.Windows.Forms 專案。
- Used the new TextRenderer class.
    使用了新的 TextRenderer 類別。
- Add the ability to change the Font of the cell headers
    增加了更改單元格標題字型的功能。
- Added the Mirrored property for RTL (RightToLeft support)
    新增了 Mirrored 屬性以支援 RTL (從右到左)
- Moved the SourceGrid3.Cells.Real classes to the namespace SourceGrid3.Cells for a more compact naming.
    將 SourceGrid3.Cells.Real 類別移至 SourceGrid3.Cells 命名空間，以更緊湊的命名方式
- Renamed SourceGrid3 namespace to SourceGrid
    將 SourceGrid3 命名空間改名為 SourceGrid
- Renamed DevAge.SourceGrid3 assembly to DevAge.SourceGrid
    將 DevAge.SourceGrid3 程式集改名為 DevAge.SourceGrid
- Renamed DevAge.SourceGrid3.Extensions assembly to DevAge.SourceGrid.Extension
    DevAge.SourceGrid3.Extensions 已重新命名為 DevAge.SourceGrid.Extension
- SourceGrid.Utilities.CreateEditor replaced with SourceGrid.Cells.Editors.Factory.Create
    SourceGrid.Utilities.CreateEditor 已被 SourceGrid.Cells.Editors.Factory.Create 取代

## Previous versions

舊版本

You can find more information about previous SourceGrid versions at this link: [http://www.devage.com/](http://www.devage.com/)
您可以透過此連結找到更多有關舊版 SourceGrid 的資訊：http://www.devage.com/

# License 授權

SourceGrid LICENSE (MIT style)

Copyright (c) 2006 www.devage.com, Davide Icardi

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
特此授權，不收取任何費用，任何獲得此軟體及相關文件（稱為「軟體」）副本的人，可無限制地處理此軟體，包括但不限於使用、複製、修改、合併、發布、分發、再授權，以及/或銷售軟體副本，並允許獲得軟體的人進行此類操作，但必須遵守以下條件：

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
上述版權聲明與此授權聲明應包含於軟體的所有副本或其重要部分。

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
軟體依「原樣」提供，不提供任何種類的保證，明示或暗示，包括但不限於商業性、適合特定目的及不侵權之保證。在任何情況下，作者或版權持有人均不對任何索賠、損害或其他責任負責，無論該責任係由契約、侵權行為或其他方式所引起，並且與軟體或軟體之使用或其他交易有關。
