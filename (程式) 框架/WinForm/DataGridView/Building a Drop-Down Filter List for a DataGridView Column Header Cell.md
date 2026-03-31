---
aliases:
date: 2008-01-18
update:
author: Karl Erickson
language: C#
sourceurl: https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727(v=msdn.10)
tags:
---

# Building a Drop-Down Filter List for a DataGridView Column Header Cell

建立 DataGridView 列標題單元格的下拉篩選清單

- 01/18/2008

Karl Erickson  卡爾·埃里克森
Microsoft Corporation

July 2006  2006 年 7 月
Updated December 2007  更新於 2007 年 12 月

Applies to:  適用於：
   Microsoft® Visual Studio® 2005
   Microsoft® .NET Framework 2.0
   Microsoftâ Windowsâ Forms 2.0

`Summary:` The Microsoft® Windows Forms `DataGridView` control in Microsoft® Visual Studio® 2005 provides a grid experience similar to Microsoft® Excel, but does not provide the column filtering drop-down lists that Excel provides in its AutoFilter feature. However, the `DataGridView` can bind to data sources that provide filtering, such as the ADO.NET `DataView`. This article describes how to build a custom `DataGridView` column header cell that displays drop-down filter lists, by taking advantage of the filtering capabilities of the data source and the new `BindingSource` component.
摘要：Microsoft® Visual Studio® 2005 中的 Microsoft® Windows Forms DataGridView 控制項提供類似 Microsoft® Excel 的網格體驗，但並不提供 Excel 自動篩選功能中的列篩選下拉清單。然而，DataGridView 可以綁定到提供篩選功能的数据源，例如 ADO.NET DataView。本文說明了如何利用数据源的篩選功能和新 BindingSource 元件，來建置顯示下拉篩選清單的自定義 DataGridView 列標題單元。

Download code samples in C# and Visual Basic (128 KB) at the[Microsoft Download Center](https://go.microsoft.com/fwlink/?linkid=70432).
從 Microsoft 下載中心下載 C# 和 Visual Basic 的程式碼範例 (128 KB)。

# Contents  內容

     Overview of Column Filtering
列篩選概觀
     Column Filtering and the DataGridView Control
列篩選與 DataGridView 控制項
     The DataGridViewAutoFilter Class Library
DataGridViewAutoFilter 類別庫
     Using the DataGridViewAutoFilter Library
使用 DataGridViewAutoFilter 庫
     DataGridViewAutoFilterColumnHeaderCell Class Implementation Details
DataGridViewAutoFilterColumnHeaderCell 類別實現細節
     DataGridViewAutoFilterTextBoxColumn Class Implementation Details
DataGridViewAutoFilterTextBoxColumn 類別實現細節
     Possible Enhancements  可能的增強功能
     Additional Resources  額外資源

# Overview of Column Filtering

列篩選的概觀

The `DataGridView` control is an excellent tool for displaying several rows of tabular data. However, displaying a large amount of data in the `DataGridView` control at one time can make it difficult for users to find the information they need. One way to address this issue is to enable automatic sorting, so that users can sort by a particular column. Another way is to implement column filtering, so that users can display only the rows with a particular value in a particular column.
DataGridView 控制項是一個優秀的工具，用於顯示多行表格數據。然而，一次在 DataGridView 控制項中顯示大量數據可能會使用戶難以找到他們需要的信息。解決此問題的一種方法是啟用自動排序，以便使用戶可以按特定列排序。另一種方法是實現列過濾，以便使用戶可以僅顯示在特定列中有特定值的行。

The `DataGridView` control already provides automatic sorting by enabling users to click the column header, assuming that the data source supports sorting. There is no built-in support for filtering, however. You can use a data source that supports filtering, but you have to provide your own user interface (UI) to enable users to affect the filtering.
DataGridView 控制項已經提供了自動排序功能，允許使用戶點擊列標頭（假設數據來源支援排序）。然而，它沒有內建的過濾支援。您可以使用支援過濾的數據來源，但您必須提供自己的使用者介面（UI），以啟使用戶影響過濾。

Only some data sources support filtering. One typical approach is to bind a `DataGridView` control to a `BindingSource` component that is bound to a `DataSet`. This is the configuration created when you use the Visual Studio 2005 Windows Forms Designer to set up data binding. You can then set the `BindingSource.Filter` property in response to user input.
僅有部分資料來源支援篩選。一種典型的做法是將 DataGridView 控制項繫結到繫結來源 (BindingSource) 元件，而此繫結來源元件又繫結到 DataSet。這是您使用 Visual Studio 2005 Windows Forms 設計器設定資料繫結時所建立的設定。接著，您可以根據使用者輸入設定繫結來源 (BindingSource) 的 Filter 屬性。

When you have space on your form, you can provide a user interface for setting column filter options using text boxes and combo boxes. But when space is limited or when column filtering is a common task in your application, you might want to provide column filtering drop-down lists directly on the column headers, like in the Excel AutoFilter feature. This is particularly useful when you display a dock-filled `DataGridView` in a child form.
當您的表單上有空間時，您可以使用文字方塊和組合方塊提供使用者介面來設定欄位篩選選項。但當空間有限，或當欄位篩選是您應用程式中的常見任務時，您可能想要直接在欄位標題上提供欄位篩選下拉清單，就像 Excel 的 AutoFilter 功能一樣。這在您在子表單中顯示一個停靠滿的 DataGridView 時特別有用。

 Note  注意

Note: For the remainder of this article, I will use the Excel term "AutoFilter" to refer to Excel-style column-filtering drop-down lists.
注意：在本篇文章的剩餘部分，我將使用 Excel 的「AutoFilter」術語來指稱 Excel �風格的欄位篩選下拉清單。

# Column Filtering and the DataGridView Control

列篩選與 DataGridView 控制項

You may wonder why the `DataGridView` control doesn’t include an AutoFilter feature. The design goal of the `DataGridView` was to create a flexible grid control with all the most common features, while recognizing that almost everyone will have special needs that can't be satisfied by the common features. The control had to cover the basics, but also had to be easily customizable, both in terms of appearance and behavior—providing a big improvement over the `DataGrid` control it replaces. An AutoFilter feature is not a necessity, since you can provide UI elsewhere on a form to manage filtering. You can also implement an AutoFilter feature yourself, although this involves advanced customization.
你可能會好奇為何 DataGridView 控制項不包含自動篩選功能。DataGridView 的設計目標是建立一個具備所有最常見功能的靈活網格控制項，同時認識到幾乎每個人都有特殊需求，這些需求無法由常見功能滿足。這個控制項必須涵蓋基本功能，但也必須容易進行自訂，無論是外觀還是行為——這比它所取代的 DataGrid 控制項有顯著的進步。自動篩選功能並非必要，因為你可以在表單的其他位置提供 UI 來管理篩選。你也可以自行實現自動篩選功能，儘管這涉及進階的自訂。

You can customize the `DataGridView` control on a few different levels. For example, you can set style properties and handle paint events in your client application, or you can provide custom column and cell types, often by extending existing column and cell types in order to leverage existing functionality. Creating a new editing cell type that hosts an arbitrary Windows Forms control, for example, is relatively trivial when you extend the `DataGridViewTextBoxCell` class.
你可以從幾個不同的層級來自訂 DataGridView 控制項。例如，你可以在客戶端應用程式中設定樣式屬性並處理繪製事件，或者你可以提供自訂的欄位和單元類型，通常透過擴充現有的欄位和單元類型來利用現有功能。例如，當你擴充 DataGridViewTextBoxCell 類別時，建立一個包含任意 Windows Forms 控制項的新編輯單元類型相對簡單。

To implement an AutoFilter feature, you have to derive from the `DataGridViewColumnHeaderCell` class, which provides no editing control functionality. Extending it to display a drop-down list is more difficult than just tweaking the `DataGridViewComboBoxCell` class. Instead, you must do all the work of painting the drop-down button, hit-testing the mouse clicks, positioning and displaying the drop-down list, and responding appropriately to user selections.
為了實現自動篩選功能，您必須繼承 DataGridViewColumnHeaderCell 類別，該類別不提供編輯控制功能。將其擴展以顯示下拉清單比僅僅調整 DataGridViewComboBoxCell 類別更困難。相反地，您必須完成所有繪製下拉按鈕、處理滑鼠點擊的命中測試、定位和顯示下拉清單，以及適當地回應用戶選擇的工作。

# The DataGridViewAutoFilter Class Library

DataGridViewAutoFilter 類別庫

The sample accompanying this article provides a demonstration implementation of the AutoFilter feature in the form of a `DataGridViewAutoFilter` library. This library contains a `DataGridViewAutoFilterColumnHeaderCell` class and a supporting column class. The sample is not a full-featured, general-purpose, bug-free, nor Microsoft-supported solution, but it does provide a groundwork, which you can modify or use to guide your own implementation.
本文附帶的範例提供了一個以 `DataGridViewAutoFilter` 庫形式展示自動篩選功能實現的示範。該庫包含一個 `DataGridViewAutoFilterColumnHeaderCell` 類別和一個支援的列類別。該範例不是一個功能完整、通用、無錯誤或受 Microsoft 支援的解決方案，但它確實提供了一個基礎，您可以修改或使用它來指導您自己的實現。

Figure 1 shows the `DataGridViewAutoFilter` library in action.
圖 1 展示了 `DataGridViewAutoFilter` 庫的運作情況。

![Figure 1. DataGridViewAutoFilter](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480727.datagridviewautofilter01\(en-us,msdn.10\).gif)

Figure 1. The AutoFilter feature implemented by using the DataGridViewAutoFilter library.
圖 1. 使用 DataGridViewAutoFilter 庫實現的自動篩選功能。

The features and dependencies of the sample are elaborated in the following sections to help you understand what you can use immediately and what you have to provide yourself. These are not exhaustive lists of the features and limitations, but they may help guide your decision to investigate further or seek another solution. Before using any of the sample code in production applications, be sure to thoroughly test all scenarios that you want to support.
樣本的特性與相依性在以下各節中詳細說明，以協助您了解可以立即使用哪些功能，以及需要自行提供哪些內容。這些並非特性與限制的完整清單，但可以協助您決定是否需要進一步研究或尋找其他解決方案。在於生產應用程式中使用任何樣本程式碼之前，請務必仔細測試所有您想要支援的情境。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#features)

## Features  特性

The sample `DataGridViewAutoFilter` library provides the following features:
樣本 `DataGridViewAutoFilter` 庫提供以下功能：

- Support for multi-column filtering. Each drop-down list displays all unique values in a column, taking the filters of other columns into consideration. If a column is filtered, its drop-down list displays the values that would appear in the column if it were not filtered, and the current filter value is selected.
    支援多欄位過濾。每個下拉清單會顯示欄位中所有獨特值，並考慮其他欄位的過濾條件。如果某欄位已過濾，其下拉清單會顯示若未過濾該欄位時會出現的值，並選取目前的過濾值。
- Support for automatic sorting. A user can sort a column by clicking the header cell and the sorting glyph will appear next to the drop-down button.
    支援自動排序。使用者可以透過點擊標題單元格來排序欄位，排序符號會出現在下拉按鈕旁邊。
- Support for the following special filter options: (All), (Blanks), and (NonBlanks).
    支援以下特殊過濾選項：(全部), (空白), 和 (非空白)。
- Support for adding the AutoFilter feature to applications at design time in Visual Studio 2005.
    支援在 Visual Studio 2005 的設計時期，將 AutoFilter 功能加入應用程式。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#features-not-implemented)

## Features Not Implemented  未實現的功能

The sample `DataGridViewAutoFilter` library does not provide some features that you may expect from using Excel, or that you may require for your particular application. The features not implemented include:
範例 `DataGridViewAutoFilter` 庫未提供您可能從使用 Excel 得到的某些功能，或您可能為特定應用程式需要的某些功能。未實現的功能包括：

- Sorting options in the drop-down list. Automatic sorting is supported, which makes this Excel feature unnecessary.
    下拉清單中的排序選項。支援自動排序，這使得 Excel 的此功能不再必要。
- Additional special filter options, such as (Custom...) and (Top 10...). These options are outside the scope of this sample. All filtering is for a particular value or for one of the supported special filter options. If you need to provide complex filtering options, you can implement your own Custom AutoFilter dialog box, possibly similar to the one in Excel.
    額外的特殊過濾選項，例如 (自訂...) 和 (前 10 個...)。這些選項不在本範例的範圍內。所有過濾都是針對特定值或支援的特殊過濾選項之一。如果您需要提供複雜的過濾選項，您可以實現自己的 Custom AutoFilter 對話方塊，可能與 Excel 中的類似。
- No support for filtering in unbound or virtual mode, or in bound mode with data sources that do not provide filtering. This functionality is outside the scope of this sample. If you cannot use an external data source that provides filtering, you must implement filtering yourself.
    不支援未綁定或虛擬模式中的過濾，或綁定模式中數據來源未提供過濾的情況。此功能不在本範例的範圍內。如果您無法使用提供過濾的外部數據來源，您必須自行實現過濾。
- No support for arbitrary data formatting. The strings displayed in the drop-down list are retrieved through the `DataGridViewCell.GetFormattedValue` property using the column's cell style. You can use the `DataGridViewCellStyle.Format` property to indicate date or currency formatting. However, if you have special formatting needs, you must implement them yourself.
    不支援隨意數據格式化。下拉清單中顯示的字串是透過 DataGridViewCell.GetFormattedValue 屬性，使用列的單元格樣式取得的。您可以使用 DataGridViewCellStyle.Format 屬性來指定日期或貨幣格式。但是，如果您有特殊的格式需求，您必須自行實現。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#dependencies)

## Dependencies  相依性

The sample `DataGridViewAutoFilter` library depends on four things:
範例 `DataGridViewAutoFilter` 庫依賴四件事：

- The `DataGridView.DataSource` must be set to a `BindingSource` component.
    DataGridView.DataSource 必須設定為 BindingSource 元件。
- The `BindingSource` component must be bound to an `IBindingListView` implementation.
    BindingSource 元件必須繫結至 IBindingListView 實作。
- The `IBindingListView.SupportsFiltering` property value must be `true`.
    IBindingListView.SupportsFiltering 屬性值必須為 true。
- The `IBindingListView.Filter` property must support multi-column filtering.
    IBindingListView.Filter 屬性必須支援多欄位篩選。

These four requirements are fulfilled automatically when you use the Visual Studio 2005 Windows Forms Designer to set up data binding through ADO.NET. (For more information, see [How to: Bind Data to the Windows Forms DataGridView Control Using the Designer](https://go.microsoft.com/fwlink/?linkid=70426) in the [MSDN2 library](https://go.microsoft.com/fwlink/?linkid=70419).) Alternatively, you can bind to any `IBindingListView` implementation that meets the requirements.
當您使用 Visual Studio 2005 Windows Forms Designer 透過 ADO.NET 設定資料繫結時，這四個需求會自動滿足。(如需更多資訊，請參閱 MSDN2 library 中的「如何：使用 Designer 將資料繫結至 Windows Forms DataGridView 控制項」。) 另者，您可以繫結至符合需求的任何 IBindingListView 實作。

If you need to bind to an existing data source, you must bind the data source to a `BindingSource` component and bind the `BindingSource` to your `DataGridView` control. The dependency on the `BindingSource` component is necessary because of the `BindingSource.RaiseListChangedEvents` property. This property can be set to `false` to enable temporary modification of the filter without updating the data displayed in the `DataGridView` control. This is necessary to calculate the values that should appear in a drop-down list, even when the column itself is filtered. If you can provide similar functionality in an alternative data source, you can work around this dependency.
如果您需要繫結到現有的數據來源，您必須將數據來源繫結到 BindingSource 元件，並將 BindingSource 繫結到您的 DataGridView 控制項。對於 BindingSource 元件的相依性是必要的，因為它的 BindingSource.RaiseListChangedEvents 屬性。此屬性可以設為 false 以啟用臨時修改濾鏡，而無需更新 DataGridView 控制項中顯示的數據。這是必要的，以便計算應出現在下拉清單中的值，即使該欄本身也經過濾。如果您可以在替代數據來源中提供類似的功能，您可以繞過這個相依性。

To support filtering, the `BindingSource` must be bound to an `IBindingListView` with a `SupportsFiltering` property value of `true`. This is the case when you bind to an ADO.NET object such as a `DataSet` (for example, when you set up data binding through the designer). Although the `DataSet` class does not implement `IBindingListView` itself, the `BindingSource` can retrieve a `DataView` from the appropriate `DataTable` within the `DataSet`. In this case, the `BindingSource.Filter` property maps to the `DataView.RowFilter` property, which makes use of the `DataColumn.Expression` property.
為了支援篩選，BindingSource 必須綁定到一個 SupportsFiltering 屬性值為 true 的 IBindingListView。這在你綁定到 ADO.NET 物件（例如 DataSet，例如，當你透過設計師設定資料綁定時）的情況下是這樣的。儘管 DataSet 類別本身沒有實作 IBindingListView，但 BindingSource 可以從 DataSet 內適當的 DataTable 中取得 DataView。在這種情況下，BindingSource.Filter 屬性對應到 DataView.RowFilter 屬性，後者使用 DataColumn.Expression 屬性。

The `IBindingListView` interface does not dictate how its `Filter` property must be implemented, but any application that uses the `Filter` property must make certain assumptions about the format of the filter string. The sample `DataGridViewAutoFilter` library expects the `IBindingListView.Filter` property to accept filter strings for Boolean expressions using the syntax documented in the `DataColumn.Expression` topic in the .NET Framework managed reference documentation.
IBindingListView 介面並未規定其 Filter 屬性必須如何實作，但任何使用 Filter 屬性的應用程式必須對過濾字串的格式做出某些假設。範例 `DataGridViewAutoFilter` 庫預期 IBindingListView.Filter 屬性會接受用於布林表達式的過濾字串，其語法與 .NET Framework 受管理參考文件中的 DataColumn.Expression 主題中所述的語法相同。

# Using the DataGridViewAutoFilter Library

使用 DataGridViewAutoFilter 庫

The `DataGridViewAutoFilter` library is a single assembly that you can reference in your projects to gain access to the `DataGridViewAutoFilterColumnHeaderCell` and `DataGridViewAutoFilterTextBoxColumn` classes. You can use these classes either programmatically or through the Windows Forms Designer in Visual Studio 2005.
`DataGridViewAutoFilter` 库是一個單一的組件，您可以參考它在專案中來存取 `DataGridViewAutoFilterColumnHeaderCell` 和 `DataGridViewAutoFilterTextBoxColumn` 類別。您可以透過程式設計或使用 Visual Studio 2005 中的 Windows Forms 設計器來使用這些類別。

To display the AutoFilter drop-down button in a header cell, a column's `HeaderCell` property must be set to an instance of the `DataGridViewAutoFilterColumnHeaderCell` class. You can set the `HeaderCell` property of specific columns programmatically. The Windows Forms Designer, however, does not let you set the `HeaderCell` property in a `Properties` window.
要顯示在標頭單元格中的自動篩選下拉按鈕，必須將某個欄位的 HeaderCell 屬性設定為 `DataGridViewAutoFilterColumnHeaderCell` 類別的實例。您可以透過程式設計設定特定欄位的 HeaderCell 屬性。然而，Windows Forms Designer 不允許您在屬性視窗中設定 HeaderCell 屬性。

To support the designer experience, the `DataGridViewAutoFilterTextBoxColumn` is provided. In the designer, you can select this column type in the `Edit Columns and Add Columns` dialog boxes. The `DataGridViewAutoFilterTextBoxColumn` extends the `DataGridViewTextBoxColumn` in order to set the `HeaderCell` property to a new instance of the `DataGridViewAutoFilterColumnHeaderCell` class.
為了支援設計器體驗， `DataGridViewAutoFilterTextBoxColumn` 已被提供。在設計器中，您可以在編輯欄位與新增欄位對話方塊中選擇此欄位類型。 `DataGridViewAutoFilterTextBoxColumn` 繼承了 DataGridViewTextBoxColumn，以便將 HeaderCell 屬性設定為 `DataGridViewAutoFilterColumnHeaderCell` 類別的新實例。

After you have added one or more AutoFilter header cells to your application, you may want to provide user feedback about the current filter state, and provide a way for users to show all rows. You can use the `BindingSource` component to retrieve the filtered row count and to remove the filter, but the `DataGridViewAutoFilter` cell and column classes provide this functionality in convenience methods.
在您已為應用程式添加一個或多個 AutoFilter 標題格之後，您可能想要提供使用者關於當前過濾狀態的回饋，並提供一種讓使用者能夠顯示所有列的方式。您可以使用 BindingSource 元件來取得過濾後的列數並移除過濾，但 `DataGridViewAutoFilter` 格和列類別則在便利方法中提供此功能。

You may also want to provide keyboard access to the drop-down list from your form code. In Excel, users can navigate to a cell containing a drop-down button, and then press ALT+UP ARROW or ALT+DOWN ARROW. In the `DataGridView` control, however, users cannot navigate to a header cell. Instead, you may want to enable ALT+UP/DOWN ARROW to display the drop-down list for whichever column contains the current cell.
您也可以從表單程式中提供鍵盤存取下拉清單的方式。在 Excel 中，使用者可以鍵盤導航至包含下拉按鈕的單元格，接著按下 ALT+ 上箭頭或 ALT+ 下箭頭。但在 DataGridView 控制項中，使用者無法鍵盤導航至標題單元格。相反地，您可以啟用 ALT+ 上/下箭頭來顯示包含當前單元格的欄位的下拉清單。

The following procedures describe four usage scenarios for the `DataGridViewAutoFilter` classes. The first procedure describes how to add the AutoFilter feature to your Windows Forms application using the designer. The second procedure describes how to add the AutoFilter feature programmatically. The third procedure describes how to enhance your client application by displaying a filter status string and a `Show All` option. Finally, the fourth procedure describes how to enable users to display the drop-down list using the keyboard.
以下步驟說明了 `DataGridViewAutoFilter` 類別的四種使用情境。第一個步驟說明如何使用設計師將 AutoFilter 功能加入您的 Windows Forms 應用程式。第二個步驟說明如何以程式設計的方式加入 AutoFilter 功能。第三個步驟說明如何透過顯示過濾狀態字串和 Show All 選項來增強您的客戶端應用程式。最後，第四個步驟說明如何讓使用者能夠使用鍵盤顯示下拉清單。

To add the AutoFilter feature to your application using the designer:
要在設計器中將 AutoFilter 功能加入您的應用程式：

1. In your Windows Application project, add a reference to the DataGridViewAutoFilter.dll assembly.
    在您的 Windows 應用程式專案中，加入對 DataGridViewAutoFilter.dll 介面的參考。
2. Add a `DataGridView` control to a form and bind data to it using the `Choose Data Source` task on the control's smart tag. For more information, see [How to: Bind Data to the Windows Forms DataGridView Control Using the Designer](https://go.microsoft.com/fwlink/?linkid=70426) in the [MSDN2 library](https://go.microsoft.com/fwlink/?linkid=70419).
    將 DataGridView 控制項加入表單，並使用控制項的智慧標籤上的「選擇資料來源」任務將資料繫結至它。如需更多資訊，請參閱 MSDN2 儲存庫中的「如何：使用設計器將資料繫結至 Windows Forms DataGridView 控制項」。
3. After the columns have been generated, on the control's smart tag, click `Edit Columns`.
    在列生成之後，在控制項的智慧標籤上，點擊「編輯列」。
4. In the `Edit Columns` dialog box, select a column.
    在編輯欄位對話方塊中，選擇一個欄位。
5. In the `Properties` window in the dialog box, select the `ColumnType` property and choose `DataGridViewAutoFilterTextBoxColumn` from the drop-down list.
    在對話方塊的屬性視窗中，選擇 ColumnType 屬性，並從下拉清單中選擇 `DataGridViewAutoFilterTextBoxColumn` 。
6. Repeat steps 4 and 5 for all columns that you want to use the AutoFilter feature with.
    對所有您想要使用 AutoFilter 功能的欄位，重複步驟 4 和 5。
7. After you have completed column configuration, run your application and confirm that drop-down buttons appear in the headers of the columns you selected.
    完成欄位設定後，執行您的應用程式，並確認您選擇的欄位標頭出現下拉按鈕。

Figure 2 shows a sample of the `Edit Columns` dialog box being used to set the column type to `DataGridViewAutoFilterTextBoxColumn`.
圖 2 展示了使用編輯欄位對話方塊將欄位類型設為 `DataGridViewAutoFilterTextBoxColumn` 的範例。

![Fig 2. The Edit Columns dialog box with the ColumnType property set to DataGridViewAutoFilterTextBoxColumn.](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480727.dtgrdvwaf02\(en-us,msdn.10\).gif)

Figure 2. The Edit Columns dialog box with the ColumnType property set to DataGridViewAutoFilterTextBoxColumn.
圖 2. 欄位類型屬性設為 DataGridViewAutoFilterTextBoxColumn 的編輯欄位對話方塊。

To add the AutoFilter feature to your application programmatically:
若要程式設計方式將 AutoFilter 功能加入您的應用程式：

1. In your Windows Application project, add a reference to the DataGridViewAutoFilter.dll assembly.
    在你的 Windows Application 專案中，加入對 DataGridViewAutoFilter.dll 介件的參考。
    
2. Add the following `using` statement to the top of your code file so that you don't have to qualify the `DataGridViewAutoFilter` class names.
    在您的程式碼檔案的最上方加入下列 using 語句，以便您不必限定 `DataGridViewAutoFilter` 類別名稱。
    
3. Add a `DataGridView` control to a form.
    將一個 DataGridView 控制項加入表單。
    
4. Handle the `DataGridView.BindingContextChanged` event using either the designer or the event-hookup code shown. The event handler must be associated with the event before the `DataSource` property is set so that the event will occur as a result of data binding.
    使用設計工具或顯示的事件連接程式碼來處理 DataGridView.BindingContextChanged 事件。事件處理常式必須在設定 DataSource 屬性之前與事件關聯，以便事件會因資料綁定而發生。
    
5. Set the `DataGridView.DataSource` property. The following code assumes you have already created and initialized a `BindingSource` component named `customersBindingSource`, either through the designer, or in code. For more information on setting up data binding programmatically, see [How to: Bind Data to the Windows Forms DataGridView Control](https://go.microsoft.com/fwlink/?linkid=70427) in the [MSDN2 library](https://go.microsoft.com/fwlink/?linkid=70419).
    設定 DataGridView.DataSource 屬性。下列程式碼假設您已經建立並初始化了一個名為 `customersBindingSource` 的 BindingSource 元件，無論是透過設計工具或程式碼。如需更多有關如何以程式設計方式設定資料綁定的資訊，請參閱 MSDN2 儲存庫中的「如何：將資料綁定到 Windows Forms DataGridView 控制項」一文。
    
6. In the `BindingContextChanged` event handler, set the `DataGridViewColumn.HeaderCell` property for the columns you want to affect. The following example code iterates through the columns in a `DataGridView` control and sets each header to a new `DataGridViewAutoFilterColumnHeaderCell` object.
    在 BindingContextChanged 事件處理程序中，為您想要影響的列設定 DataGridViewColumn.HeaderCell 屬性。以下範例程式碼會遍歷 DataGridView 控制項中的所有列，並將每個標題設定為新的 `DataGridViewAutoFilterColumnHeaderCell` 物件。
    
7. Run your application and confirm that drop-down buttons appear in the headers of the columns you selected.
    執行您的應用程式，並確認您選擇的列標題中出現下拉按鈕。

To add a StatusStrip control that displays the filter status and a Show All option:
若要添加顯示篩選狀態和「顯示全部」選項的 StatusStrip 控制項：

1. Follow one of the previous procedures to add the AutoFilter feature to a `DataGridView` control.
    按照先前步驟中的一種方法，為 DataGridView 控制項添加 AutoFilter 功能。
    
2. If you haven't already done so, add the following `using` statement to the top of your code file so that you don't have to qualify the `DataGrideViewAutoFilter`  class names.
    %% 如果您尚未完成，請在程式碼檔案的最上方加入下列 using 語句，以便您不必限定 `DataGrideViewAutoFilter` 類別名稱。

    Copy  複製

    ```csharp
        using DataGridViewAutoFilter;
    ```

3. Add a `StatusStrip` control to your form named `statusStrip1` and containing two `ToolStripStatusLabel` components named `filterStatusLabel` and `showAllLabel`.
    在您的表單中添加一個名為 `statusStrip1` 的 StatusStrip 控制項，並包含兩個名為 `filterStatusLabel` 和 `showAllLabel` 的 ToolStripStatusLabel 組件。
    
4. Configure the labels by using the following code or equivalent settings in the `Properties` window of the designer.
    透過下列程式碼或設計器屬性窗格中的相當設定來配置標籤。

    Copy  複製

    ```csharp
      filterStatusLabel.Text = "";
      filterStatusLabel.Visible = false;
      showAllLabel.Text = "Show &All";
      showAllLabel.Visible = false;
      showAllLabel.IsLink = true;
      showAllLabel.LinkBehavior = LinkBehavior.HoverUnderline;
    ```

5. Handle the `Click` event for the `showAllLabel` component.
    處理 `showAllLabel` 組件的 Click 事件。

    Copy  複製

    ```csharp
      // Add this code to the constructor or associate 
      // the handler with the event using the designer. 
      showAllLabel.Click += new EventHandler(showAllLabel_Click);
    
      // ...
    
      private void showAllLabel_Click(object sender, EventArgs e)
      {
          DataGridViewAutoFilterTextBoxColumn.RemoveFilter(dataGridView1);
      }
    ```

6. Handle the `DataGridView.DataBindingComplete` event.
    處理 DataGridView.DataBindingComplete 事件。

    Copy  複製

    ```csharp
          // Add this code to the constructor or associate 
        // the handler with the event using the designer. 
        dataGridView1.DataBindingComplete += 
            new DataGridViewBindingCompleteEventHandler(
            dataGridView1_DataBindingComplete);
    
        // ...
    
        void dataGridView1_DataBindingComplete(object sender,
            DataGridViewBindingCompleteEventArgs e)
        {
            String filterStatus = DataGridViewAutoFilterColumnHeaderCell
                .GetFilterStatus(dataGridView1);
            if (String.IsNullOrEmpty(filterStatus))
            {
                showAllLabel.Visible = false;
                filterStatusLabel.Visible = false;
            }
            else
            {
                showAllLabel.Visible = true;
                filterStatusLabel.Visible = true;
                filterStatusLabel.Text = filterStatus;
    
        }
    ```

7. Run your application and apply a filter.
    執行您的應用程式並應用過濾器。

Figure 3 shows a sample of the resulting `StatusStrip` after a filter has been applied.
圖 3 顯示在應用過濾器後結果的 StatusStrip 範例。

![Fig 3. A StatusStrip control displaying the filter status and a Show All option.](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480727.datagridviewautofilter01\(en-us,msdn.10\).gif)

Figure 3. A StatusStrip control displaying the filter status and a Show All option.
圖 3。顯示過濾狀態和 Show All 選項的 StatusStrip 控制項。

To enable users to display the drop-down list using the keyboard:
為了讓使用者能夠使用鍵盤顯示下拉清單：

1. If you haven't already done so, add the following `using` statement to the top of your code file so that you don't have to qualify the `DataGridViewAutoFilter` class names.
    %% 如果您尚未完成，請在程式碼檔案的最上方加入下列 using 語句，以便您不必限定 `DataGridViewAutoFilter` 類別名稱。

    Copy  複製

    ```csharp
      using DataGridViewAutoFilter;
    ```

2. Handle the `DataGridView.KeyDown` event.
    處理 DataGridView.KeyDown 事件。

    Copy  複製

    ```csharp
            // Add this code to the constructor.
        this.dataGridView1.KeyDown += new 
            KeyEventHandler(dataGridView1_KeyDown);
    
    
        // ...
    
        void dataGridView1_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Alt && (e.KeyCode == Keys.Down || e.KeyCode == Keys.Up))
            {
                DataGridViewAutoFilterColumnHeaderCell filterCell =
                    this.dataGridView1.CurrentCell.OwningColumn.HeaderCell as 
                    DataGridViewAutoFilterColumnHeaderCell;
                if (filterCell != null)
                {
                    filterCell.ShowDropDownList();
                    e.Handled = true;
                }
            }
        }
    ```

3. Run your application and press ALT+UP ARROW or ALT+DOWN ARROW to open the drop-down list for the current column. Press ESC to close the drop-down list.
    執行您的應用程式，並按下 ALT+ 上方向鍵 或 ALT+ 下方向鍵來打開目前欄位的下拉清單。按下 ESC 鍵來關閉下拉清單。

# DataGridViewAutoFilterColumnHeaderCell Class Implementation Details

DataGridViewAutoFilterColumnHeaderCell 類別實現細節

Now that you know how to use the `DataGridViewAutoFilter` library and know some of its capabilities, the rest of this article is devoted to describing the details of how the `DataGridViewAutoFilter` library works.
現在您已經知道如何使用 `DataGridViewAutoFilter` 庫，並了解其部分功能，這篇文章的剩餘部分將專注於描述 `DataGridViewAutoFilter` 庫的詳細運作方式。

The `DataGridViewAutoFilter` library might meet your requirements as it is, making it unnecessary to know any of the implementation details. However, you might want to customize or extend its capabilities to better serve your needs, or you might discover a bug that you want to fix. Additionally, understanding the implementation will help if you want to create other custom header cell types for the `DataGridView` control.
`DataGridViewAutoFilter` 庫可能已符合您的需求，因此無需了解任何實作細節。然而，您可能想要自訂或擴展其功能，以更好地滿足您的需求，或者您可能發現了一個想要修復的錯誤。此外，了解實作方式將有助於您，如果您想要為 DataGridView 控制項創建其他自訂標題單元類型。

The primary class in the `DataGridViewAutoFilter` library is `DataGridViewAutoFilterColumnHeaderCell` class. This class derives from the `DataGridViewColumnHeaderCell` class.
`DataGridViewAutoFilter` 庫中的主要類別是 `DataGridViewAutoFilterColumnHeaderCell` 類別。這個類別繼承自 DataGridViewColumnHeaderCell 類別。

The `DataGridViewAutoFilterColumnHeaderCell` implementation details can be divided into four categories:
`DataGridViewAutoFilterColumnHeaderCell` 的實現細節可以分為四個類別：

- Initialization.  初始化。
- Displaying the drop-down button.
    顯示下拉按鈕。
- Displaying, hiding, and handling user interaction with the drop-down list.
    顯示、隱藏以及處理用戶與下拉清單的互動。
- Filtering the bound data when the user selects a filter from the list.
    過濾綁定資料當使用者從清單選擇過濾器時。

The following sections describe these categories in more detail and provide example code to illustrate various points. The example code is excerpted from the accompanying `DataGridViewAutoFilter` library, but some details are left out for brevity. For the full details, see the complete source code in the sample.
以下各節將更詳細地說明這些類別，並提供範例程式碼來說明各種要點。範例程式碼摘自隨附的 `DataGridViewAutoFilter` 庫，但為了簡潔起見，省略了一些細節。如需完整細節，請參考範例中的完整原始碼。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#initialization)

## Initialization  初始化

The primary initialization task is to create a `ListBox` control for use as a drop-down list. The `DataGridViewAutoFilter` library uses a single `ListBox`-derived control for all AutoFilter header cells in an application, storing it in a static variable called `dropDownListBox`. Using a single instance is possible because only one drop-down list is displayed at a time. It is also possible because the contents, size, and location of the drop-down list are easiest to determine at the time the list is displayed, since they are affected by frequent changes to a `DataGridView` control such as filtering, resizing, and scrolling.
主要初始化任務是建立一個 ListBox 控制項，用作下拉清單。 `DataGridViewAutoFilter` 庫使用單一個衍生自 ListBox 的控制項，用於應用程式中所有 AutoFilter 標題細胞，並將其儲存於名為 `dropDownListBox` 的靜態變數中。之所以可以單一實例，是因為每次僅顯示一個下拉清單。這也是可行的，因為下拉清單的內容、大小和位置，在清單顯示時最容易確定，因為它們會受到 DataGridView 控制項經常變化的影響，例如篩選、調整大小和捲動。

The Windows Forms `ListBox` control is used because it provides the appearance and behavior of an Excel AutoFilter list, including a vertical scroll bar to navigate long lists. In contrast, the `ToolStripDropDown` control is not used because it lacks a vertical scroll bar, instead using up and down buttons in line with its typical usage as a menu. The `ComboBox` control is not used because its drop-down list cannot easily be displayed without displaying the text box part of the control.
使用 Windows Forms ListBox 控制項，是因為它提供了 Excel AutoFilter 清單的外觀和行為，包括垂直捲動條以導航長清單。相較之下，ToolStripDropDown 控制項不使用，是因為它沒有垂直捲動條，而是使用與其作為菜單的典型用法相符的上和下按鈕。ComboBox 控制項不使用，是因為它的下拉清單無法輕易地顯示，除非同時顯示控制項的文本框部分。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-filterlistbox-class)

### The FilterListBox Class  FilterListBox 類別

The `FilterListBox` control extends the `ListBox` control to ensure that it receives keyboard messages that would normally be intercepted by the parent `DataGridView` control. Additionally, the `FilterListBox` class changes a few `ListBox` property settings to configure the control for use as a drop-down list.
`FilterListBox` 控制項擴展了 ListBox 控制項，確保它能接收通常會被父級 DataGridView 控制項攔截的鍵盤訊息。此外， `FilterListBox` 類別會更改一些 ListBox 屬性設定，以將控制項配置為下拉清單使用。

The `FilterListBox` class overrides the protected `IsInputKey` and `ProcessKeyMessage` methods in order to intercept all keystrokes except those handled by the operating system, such as ALT+F4. The `Control.ProcessKeyMessage` method normally dispatches messages to the `ProcessKeyPreview` method of the control's parent. If the control's parent is not interested in the message, it goes to the control's own `ProcessKeyEventArgs` method in order to generate keyboard events.
`FilterListBox` 類別會覆寫受保護的 IsInputKey 和 ProcessKeyMessage 方法，以便攔截所有鍵盤輸入，除了作業系統處理的鍵盤輸入，例如 ALT+F4。Control.ProcessKeyMessage 方法通常會將訊息分派到控制項父級的 ProcessKeyPreview 方法。如果控制項的父級對訊息不感興趣，它會傳遞到控制項自己的 ProcessKeyEventArgs 方法，以便產生鍵盤事件。

Unfortunately, the `DataGridView.ProcessKeyPreview` method intercepts all keyboard messages unless they are for an editing control hosted by an ordinary (non-header) cell in edit mode. (For more information about hosting editing controls, see [How to: Host Controls in Windows Forms DataGridView Cells](https://go.microsoft.com/fwlink/?linkid=70428) in the [MSDN2 library](https://go.microsoft.com/fwlink/?linkid=70419).) For keyboard handling, the `DataGridView` control ignores all other child controls. As a result, keystrokes that you expect the hosted control to handle are handled by the `DataGridView` control instead.
不幸的是，DataGridView.ProcessKeyPreview 方法會攔截所有鍵盤訊息，除非它們是針對由一般（非標題）單元格所主機的編輯控制項（在編輯模式中）。（有關主機編輯控制項的更多資訊，請參閱 MSDN2 儲存庫中的「如何：在 Windows Forms DataGridView 單元格中主機控制項」。）在鍵盤處理方面，DataGridView 控制項會忽略所有其他子控制項。因此，您預期由主機控制項處理的按鍵敲擊，反而會由 DataGridView 控制項來處理。

To prevent this behavior, the `ProcessKeyMessage` override in the `FilterListBox` class skips the parent-control processing and instead sends the keyboard message directly to the `ProcessKeyEventArgs` method. The `ProcessKeyEventArgs` method then raises `FilterListBox` keyboard events that the `DataGridViewAutoFilterColumnHeaderCell` class can handle.
為了防止這種行為， `FilterListBox` 類別中的 ProcessKeyMessage 覆寫會略過父控制項處理，並直接將鍵盤訊息傳送至 ProcessKeyEventArgs 方法。接著，ProcessKeyEventArgs 方法會引發 `FilterListBox` 鍵盤事件，而 `DataGridViewAutoFilterColumnHeaderCell` 類別可以處理這些事件。

The source code for the `FilterListBox` class follows:
`FilterListBox` 類別的來源代碼如下：

  private class FilterListBox : ListBox
  {
      public FilterListBox()
      {
          Visible = false;
          IntegralHeight = true;
          BorderStyle = BorderStyle.FixedSingle;
          TabStop = false;
      }

      protected override bool IsInputKey(Keys keyData)
      {
          return true;
      }

      protected override bool ProcessKeyMessage(ref Message m)
      {
          return ProcessKeyEventArgs(ref m);
      }
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-constructors-and-the-clone-method)

### The Constructors and the Clone Method

建構函式與 Clone 方法

The `DataGridViewAutoFilterColumnHeaderCell` class provides an empty default constructor and a constructor overload that takes an existing `DataGridViewColumnHeaderCell` instance. The constructor overload copies the property values of the specified cell to the new instance. This is useful when you want to replace an existing header cell with an AutoFilter cell, but keep all other header details the same.
`DataGridViewAutoFilterColumnHeaderCell` 類別提供一個空的預設建構子，以及一個接受現有 DataGridViewColumnHeaderCell 實例的建構子覆載。這個建構子覆載會將指定 cell 的屬性值複製到新的實例。當您想要用 AutoFilter cell 取代現有的標題 cell，但同時保留所有其他標題細節時，這會很有用。

The `Clone` method also uses the constructor overload to create a new instance with the same property values as an existing instance. For this reason, the constructor overload determines whether the specified column header cell is also an AutoFilter cell so that the AutoFilter-specific property values can be copied.
Clone 方法也使用建構子覆載來建立一個具有與現有實例相同屬性值的新的實例。因此，建構子覆載會判斷指定的列標題 cell 是否也是一個 AutoFilter cell，以便複製 AutoFilter 特定的屬性值。

The `DataGridView` control clones columns and their header cells when the `DataSource` changes and a new set of columns must be automatically generated. The clones are used for any columns in the new schema that match columns in the old schema. That way you do not have to reconfigure columns that are the same in multiple data sources.
當 DataSource 變更時，DataGridView 控制項會複製欄位及其標題單元格，並且需要自動產生新的欄位集。這些複製的欄位用於新架構中與舊架構相符的任何欄位。這樣，您就不必重新設定在多個資料來源中相同的欄位。

For this reason, it is important to make sure that you override the `Clone` method when you extend a `DataGridView` cell or column type and copy any public property values that you add.
因此，當您擴展 DataGridView 單元格或欄位類型時，覆寫 Clone 方法並複製您所添加的任何公用屬性值非常重要。

The source code for the non-default class constructor and the `Clone` method follows:
非預設類別建構函式和 Clone 方法的原始碼如下：

  public DataGridViewAutoFilterColumnHeaderCell(DataGridViewColumnHeaderCell oldHeaderCell)
  {
      this.ContextMenuStrip = oldHeaderCell.ContextMenuStrip;
      this.ErrorText = oldHeaderCell.ErrorText;
      this.Tag = oldHeaderCell.Tag;
      this.ToolTipText = oldHeaderCell.ToolTipText;
      this.Value = oldHeaderCell.Value;
      this.ValueType = oldHeaderCell.ValueType;
      if (oldHeaderCell.HasStyle)
      {
          this.Style = oldHeaderCell.Style;
      }

      DataGridViewAutoFilterColumnHeaderCell filterCell =
          oldHeaderCell as DataGridViewAutoFilterColumnHeaderCell;
      if (filterCell != null)
      {
          this.FilteringEnabled = filterCell.FilteringEnabled;
          this.AutomaticSortingEnabled = filterCell.AutomaticSortingEnabled;
          this.DropDownListBoxMaxLines = filterCell.DropDownListBoxMaxLines;
          this.currentDropDownButtonPaddingOffset =
              filterCell.currentDropDownButtonPaddingOffset;
      }
  }

  public override object Clone()
  {
      return new DataGridViewAutoFilterColumnHeaderCell(this);
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-ondatagridviewchanged-method)

### The OnDataGridViewChanged Method

OnDataGridViewChanged 方法

An AutoFilter header cell is not part of a `DataGridView` control at the time it is created, but only after it has been added to a column contained in a `DataGridView` control. When this occurs, the `OnDataGridViewChanged` method is called. The `DataGridViewAutoFilterColumnHeaderCell` class overrides this method to provide initialization that cannot occur before the `DataGridView` control is available.
一個 AutoFilter 標題單元格在創建時並非 DataGridView 控制件的其中一部分，只有在它被加入到包含於 DataGridView 控制件的欄中後才成為其一部分。當這種情況發生時，會呼叫 OnDataGridViewChanged 方法。 `DataGridViewAutoFilterColumnHeaderCell` 類別會覆寫這個方法，以提供在 DataGridView 控制件可用之前無法進行的初始化。

When an AutoFilter header cell gains access to the `DataGridView` control that contains it, it can access the control's data source and confirm that the column type is appropriate for filtering and sorting. For example, if the column is an `Image` column, the cell's `FilteringEnabled` property is set to `false` and the drop-down button is not displayed.
當 AutoFilter 標題單元格獲得對其包含的 DataGridView 控制件的存取權時，它可以存取控制件的数据來源，並確認欄的類型適合過濾和排序。例如，如果欄是 Image 欄，單元格的 `FilteringEnabled` 屬性會被設為 false，且下拉按鈕不會顯示。

The `OnDataGridViewChanged` method also ensures that the `DataGridView` control is properly configured. First, it makes sure that the control's `SortMode` property is not set to `Automatic`. Normally, automatic sorting occurs when the user clicks anywhere on a column header cell. For AutoFilter cells, however, automatic sorting must not occur when the user clicks the portion of the cell occupied by the drop-down button. For this reason, sorting must be implemented manually and the `SortMode` property must never be set to `Automatic`.
OnDataGridViewChanged 方法也確保 DataGridView 控制項正確配置。首先，它確保控制項的 SortMode 屬性沒有設為自動。通常，自動排序發生在用戶單擊任何列標頭單元格時。對於 AutoFilter 單元格，然而，當用戶單擊佔據下拉按鈕的單元格部分時，自動排序必須不發生。由於這個原因，排序必須手動實現，SortMode 屬性絕不能設為自動。

The `OnDataGridViewChanged` method also calls the `VerifyDataSource` method to check whether the `DataGridView` control is bound to a valid data source. The control is not required to have a valid `DataSource` property value at this time, but if it does, the `VerifyDataSource` method confirms that it is a `BindingSource`, and throws an exception if it isn't.
OnDataGridViewChanged 方法也呼叫 `VerifyDataSource` 方法來檢查 DataGridView 控制項是否綁定到有效的數據來源。此時，控制項不需要有有效的 DataSource 屬性值，但如果有的話， `VerifyDataSource` 方法確認它是一個 BindingSource，如果不是，則拋出例外。

In addition to verifying the data source, the `OnDataGridViewChanged` method associates event handlers with various `DataGridView` events that affect the display of the drop-down button and list or the state of the data source. This method also initializes the drop-down button bounds so that any initial column autosizing will accommodate the button width.
除了驗證數據來源外，「OnDataGridViewChanged」方法會將事件處理程序關聯到影響下拉按鈕和清單的顯示或數據來源狀態的各種 DataGridView 事件。此方法還初始化下拉按鈕的邊界，以便任何初始列自動調整都能容納按鈕寬度。

The source code for the `OnDataGridViewChanged` method follows:
「OnDataGridViewChanged」方法的源代碼如下：

  protected override void OnDataGridViewChanged()
  {
      if (this.DataGridView == null) return;

      if (OwningColumn != null)
      {
          if (OwningColumn is DataGridViewImageColumn ||
          (OwningColumn is DataGridViewButtonColumn &&
          ((DataGridViewButtonColumn)OwningColumn)
              .UseColumnTextForButtonValue) ||
          (OwningColumn is DataGridViewLinkColumn &&
          ((DataGridViewLinkColumn)OwningColumn).UseColumnTextForLinkValue))
          {
              AutomaticSortingEnabled = false;
              FilteringEnabled = false;
          }

          if (OwningColumn.SortMode == DataGridViewColumnSortMode.Automatic)
          {
              OwningColumn.SortMode =
                  DataGridViewColumnSortMode.Programmatic;
          }
      }

      VerifyDataSource();
      HandleDataGridViewEvents();
      SetDropDownButtonBounds();
      base.OnDataGridViewChanged();
 }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#displaying-the-drop-down-button)

## Displaying the Drop-Down Button

顯示下拉按鈕

The drop-down button is not an actual `Button` control. A `Button` control is not necessary since the `DataGridViewCell` type provides an `OnMouseDown` method that can be overridden to handle mouse clicks. The AutoFilter cell just has to paint a button and check for clicks in the appropriate part of the header.
下拉按鈕並非實際的 Button 控制元件。因為 DataGridViewCell 類別提供了 OnMouseDown 方法，可以覆寫來處理滑鼠點擊，所以不需要 Button 控制元件。AutoFilter 细胞只需要繪製一個按鈕，並在標題適當的部分檢查點擊。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-paint-method)

### The Paint Method  繪製方法

The `Paint` method is responsible for painting the drop-down button in the header cell. Several factors influence the appearance of the drop-down button:
繪製方法負責在標題細胞中繪製下拉按鈕。有幾個因素影響下拉按鈕的外觀：

- The size of the drop-down button is based on the height of the header text.
    下拉按鈕的大小基於標題文字的高度。
- The location of the drop-down button is based on the location of the header cell and the value of the `DataGridView.RightToLeft` property.
    下拉按鈕的位置取決於標頭單元的位置以及 DataGridView.RightToLeft 屬性的值。
- If the application is currently themed, the drop-down button is also themed.
    如果應用程式目前有主題設定，下拉按鈕也會有主題設定。
- If `dropDownListBox` is currently visible, the drop-down button is shown in its pressed state.
    如果 `dropDownListBox` 目前可見，下拉按鈕會以按下狀態顯示。
- If the column is currently filtered, the drop-down button is highlighted.
    如果該欄位目前有過濾，下拉按鈕會被強調顯示。
- If the cell's `FilteringEnabled` property is `false`, the drop-down button is not displayed.
    若該單元的 `FilteringEnabled` 屬性為 false，則下拉按鈕不會顯示。

These factors can change while an application is running, so most of them must be determined at the time the `Paint` method is called. The size and location may remain relatively constant, however, so these values are stored in a `DropDownButtonBounds` property and reused as long as they do not change. The `DropDownButtonBounds` property also indicates the region in which a user can click the header cell to display the drop-down list.
這些因素可能在使用應用程式時改變，因此大多數因素必須在繪製方法被呼叫時確定。然而，大小和位置可能相對不變，因此這些值儲存在 `DropDownButtonBounds` 屬性中，只要它們不變就會重用。 `DropDownButtonBounds` 屬性也指示使用者可以點擊表頭單元以顯示下拉清單的區域。

The button's size and location must change whenever its header cell changes size or location. This can result from user actions such as horizontally scrolling the `DataGridView` control, resizing its columns or column headers, and resizing the control itself (for example, resizing a form that contains a dock-filled `DataGridView` control). As described earlier, the `OnDataGridViewChanged` method attaches handlers to various `DataGridView` events. The handlers for events related to scrolling and resizing call the `InvalidateDropDownButtonBounds` method, which just sets the button bounds to `Rectangle.Empty`.
當表頭單元的大小或位置變更時，按鈕的大小和位置必須隨之改變。這可能是由於使用者動作所導致，例如水平捲動 DataGridView 控制項、調整其欄位或欄位標題的大小，以及調整控制項本身的大小（例如，調整包含停靠填充 DataGridView 控制項的表單的大小）。如前所述，OnDataGridViewChanged 方法會將處理程序附加到各種 DataGridView 事件。與捲動和調整大小相關的事件的處理程序會呼叫 `InvalidateDropDownButtonBounds` 方法，該方法僅將按鈕界限設為 Rectangle.Empty。

When the `Paint` method is called, if the `DropDownButtonBounds` value is `Empty`, the `SetDropDownButtonBounds` method is called to initialize the `DropDownButtonBounds` value.
當繪製方法被呼叫時，如果 `DropDownButtonBounds` 值為 Empty，則會呼叫 `SetDropDownButtonBounds` 方法來初始化 `DropDownButtonBounds` 值。

The source code for the `Paint` method follows:
Paint 方法 的原始碼如下：

  protected override void Paint(
      Graphics graphics, Rectangle clipBounds, Rectangle cellBounds,
      int rowIndex, DataGridViewElementStates cellState,
      object value, object formattedValue, string errorText,
      DataGridViewCellStyle cellStyle,
      DataGridViewAdvancedBorderStyle advancedBorderStyle,
      DataGridViewPaintParts paintParts)
  {
      // Use the base method to paint the default appearance.
      base.Paint(graphics, clipBounds, cellBounds, rowIndex,
          cellState, value, formattedValue,
          errorText, cellStyle, advancedBorderStyle, paintParts);

      // Continue only if filtering is enabled and ContentBackground is
      // part of the paint request.
      if (!FilteringEnabled ||
          (paintParts & DataGridViewPaintParts.ContentBackground) == 0)
      {
          return;
      }

      // Retrieve the current button bounds.
      Rectangle buttonBounds = DropDownButtonBounds;

      // Continue only if the buttonBounds is big enough to draw.
      if (buttonBounds.Width < 1 || buttonBounds.Height < 1) return;

      // Paint the button manually or using visual styles if visual styles
      // are enabled, using the correct state depending on whether the
      // filter list is showing and whether there is a filter in effect
      // for the current column.
      if (Application.RenderWithVisualStyles)
      {
          ComboBoxState state = ComboBoxState.Normal;

          if (dropDownListBoxShowing)
          {
              state = ComboBoxState.Pressed;
          }
          else if (filtered)
          {
              state = ComboBoxState.Hot;
          }
          ComboBoxRenderer.DrawDropDownButton(
              graphics, buttonBounds, state);
      }
      else
      {
          // Determine the pressed state in order to paint the button
          // correctly and to offset the down arrow.
          Int32 pressedOffset = 0;
          PushButtonState state = PushButtonState.Normal;
          if (dropDownListBoxShowing)
          {
              state = PushButtonState.Pressed;
              pressedOffset = 1;
          }
          ButtonRenderer.DrawButton(graphics, buttonBounds, state);

          // If there is a filter in effect for the column, paint the
          // down arrow as an unfilled triangle. If there is no filter
          // in effect, paint the down arrow as a filled triangle.
          if (filtered)
          {
              graphics.DrawPolygon(SystemPens.ControlText, new Point[] {
                  new Point(
                      buttonBounds.Width / 2 +
                          buttonBounds.Left - 1 + pressedOffset,
                      buttonBounds.Height * 3 / 4 +
                          buttonBounds.Top - 1 + pressedOffset),
                  new Point(
                      buttonBounds.Width / 4 +
                          buttonBounds.Left + pressedOffset,
                      buttonBounds.Height / 2 +
                          buttonBounds.Top - 1 + pressedOffset),
                  new Point(
                      buttonBounds.Width * 3 / 4 +
                          buttonBounds.Left - 1 + pressedOffset,
                      buttonBounds.Height / 2 +
                          buttonBounds.Top - 1 + pressedOffset)
              });
          }
          else
          {
              graphics.FillPolygon(SystemBrushes.ControlText, new Point[] {
                  new Point(
                      buttonBounds.Width / 2 +
                          buttonBounds.Left - 1 + pressedOffset,
                      buttonBounds.Height * 3 / 4 +
                          buttonBounds.Top - 1 + pressedOffset),
                  new Point(
                      buttonBounds.Width / 4 +
                          buttonBounds.Left + pressedOffset,
                      buttonBounds.Height / 2 +
                          buttonBounds.Top - 1 + pressedOffset),
                  new Point(
                      buttonBounds.Width * 3 / 4 +
                          buttonBounds.Left - 1 + pressedOffset,
                      buttonBounds.Height / 2 +
                          buttonBounds.Top - 1 + pressedOffset)
              });
          }
      }

  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-setdropdownbuttonbounds-and-adjustpadding-methods)

### The SetDropDownButtonBounds and AdjustPadding Methods

SetDropDownButtonBounds 和 AdjustPadding 方法

The `SetDropDownButtonBounds` method initializes the `DropDownButtonBounds` value. The drop-down button size is based on the preferred header cell height for a single line of header text. The button location is aligned with the bottom right corner of the cell, or the bottom left in right-to-left environments. Both the size and location are adjusted to provide a visual offset that varies depending on whether visual styles are enabled.
`SetDropDownButtonBounds` 方法會初始化 `DropDownButtonBounds` 值。下拉按鈕的大小基於單行標題文字的偏好標題單元高度。按鈕的位置與單元的右下角對齊，或在從右到左的環境中與左下角對齊。大小和位置都會調整，以提供視覺偏移，該偏移取決於是否啟用視覺樣式。

The `SetDropDownButtonBounds` method also calls the `AdjustPadding` method to modify the `DataGridViewCellStyle.Padding` property in effect for the cell, based on the drop-down button width. The padding adjustment enables the `DataGridView` control to account for the drop-down button width when resizing columns automatically and when displaying the sorting glyph. Because of the `Padding` adjustment, the cell's `GetPreferredSize` method does not have to be overridden to customize the automatic sizing.
`SetDropDownButtonBounds` 方法也呼叫 `AdjustPadding` 方法來修改單元格的 DataGridViewCellStyle.Padding 屬性，根據下拉按鈕寬度進行實際調整。這個填充調整讓 DataGridView 控制能在自動縮放欄位時，以及顯示排序符號時，考慮下拉按鈕的寬度。由於有這個填充調整，單元格的 GetPreferredSize 方法就不必被覆寫來自定義自動大小。

The source code for the `SetDropDownButtonBounds` and `AdjustPadding` methods follows:
`SetDropDownButtonBounds` 和 `AdjustPadding` 方法的原始碼如下：

  private void SetDropDownButtonBounds()
  {
      // Retrieve the cell display rectangle, which is used to
      // set the position of the drop-down button.
      Rectangle cellBounds =
          this.DataGridView.GetCellDisplayRectangle(
          this.ColumnIndex, -1, false);

      // Initialize a variable to store the button edge length,
      // setting its initial value based on the font height.
      Int32 buttonEdgeLength = this.InheritedStyle.Font.Height + 5;

      // Calculate the height of the cell borders and padding.
      Rectangle borderRect = BorderWidths(
          this.DataGridView.AdjustColumnHeaderBorderStyle(
          this.DataGridView.AdvancedColumnHeadersBorderStyle,
          new DataGridViewAdvancedBorderStyle(), false, false));
      Int32 borderAndPaddingHeight = 2 +
          borderRect.Top + borderRect.Height +
          this.InheritedStyle.Padding.Vertical;
      Boolean visualStylesEnabled =
          Application.RenderWithVisualStyles &&
          this.DataGridView.EnableHeadersVisualStyles;
      if (visualStylesEnabled)
      {
          borderAndPaddingHeight += 3;
      }

      // Constrain the button edge length to the height of the
      // column headers minus the border and padding height.
      if (buttonEdgeLength >
          this.DataGridView.ColumnHeadersHeight -
          borderAndPaddingHeight)
      {
          buttonEdgeLength =
              this.DataGridView.ColumnHeadersHeight -
              borderAndPaddingHeight;
      }

      // Constrain the button edge length to the
      // width of the cell minus three.
      if (buttonEdgeLength > cellBounds.Width - 3)
      {
          buttonEdgeLength = cellBounds.Width - 3;
      }

      // Calculate the location of the drop-down button, with adjustments
      // based on whether visual styles are enabled.
      Int32 topOffset = visualStylesEnabled ? 4 : 1;
      Int32 top = cellBounds.Bottom - buttonEdgeLength - topOffset;
      Int32 leftOffset = visualStylesEnabled ? 3 : 1;
      Int32 left = 0;
      if (this.DataGridView.RightToLeft == RightToLeft.No)
      {
          left = cellBounds.Right - buttonEdgeLength - leftOffset;
      }
      else
      {
          left = cellBounds.Left + leftOffset;
      }

      // Set the dropDownButtonBoundsValue value using the calculated
      // values, and adjust the cell padding accordingly. 
      dropDownButtonBoundsValue = new Rectangle(left, top,
          buttonEdgeLength, buttonEdgeLength);
      AdjustPadding(buttonEdgeLength + leftOffset);
  }

  private void AdjustPadding(Int32 newDropDownButtonPaddingOffset)
  {
      // Determine the difference between the new and current
      // padding adjustment.
      Int32 widthChange = newDropDownButtonPaddingOffset -
          currentDropDownButtonPaddingOffset;

      // If the padding needs to change, store the new value and
      // make the change.
      if (widthChange != 0)
      {
          // Store the offset for the drop-down button separately from
          // the padding in case the client needs additional padding.
          currentDropDownButtonPaddingOffset =
              newDropDownButtonPaddingOffset;
       
          // Create a new Padding using the adjustment amount, then add it
          // to the cell's existing Style.Padding property value.

          Padding dropDownPadding = new Padding(0, 0, widthChange, 0);
          this.Style.Padding = Padding.Add(
              this.InheritedStyle.Padding, dropDownPadding);
      }
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#displaying-hiding-and-handling-user-interaction-with-the-drop-down-filter-list)

## Displaying, Hiding, and Handling User Interaction With the Drop-Down Filter List

顯示、隱藏和處理用戶互動與下拉篩選清單

The drop-down list is normally displayed when the user clicks the drop-down button. The `OnMouseDown` method is responsible for handling user mouse clicks on the column header. If the user clicks the column header but the mouse click is not within the bounds specified by the `DropDownButtonBounds` property, and automatic sorting is enabled, the column is sorted and the sorting glyph is displayed next to the drop-down button. If the mouse click is within the `DropDownButtonBounds` and the drop-down list is not already showing, it is displayed using the `ShowDropDownList` method.
下拉清單通常在用戶點擊下拉按鈕時顯示。OnMouseDown 方法負責處理用戶在列標題上的鼠標點擊。如果用戶點擊列標題，但鼠標點擊不在 `DropDownButtonBounds` 屬性指定的範圍內，且自動排序已啟用，則會對列進行排序，並在下拉按鈕旁顯示排序符號。如果鼠標點擊在 `DropDownButtonBounds` 範圍內，且下拉清單尚未顯示，則會使用 `ShowDropDownList` 方法顯示。

As described earlier, you can also handle the `DataGridView.KeyDown` event to display the drop-down list when the user presses a particular key combination, such as ALT+DOWN ARROW. When the correct keystrokes are detected, the event handler calls the `ShowDropDownList` method to display the drop-down list.
如前所述，您也可以處理 DataGridView.KeyDown 事件，以在用戶按下特定組合鍵（例如 ALT+ 向下箭頭）時顯示下拉清單。當正確的鍵盤輸入被檢測到時，事件處理程序會呼叫 `ShowDropDownList` 方法來顯示下拉清單。

After the drop-down list appears, the user can navigate the list using the keyboard, scroll the list if a scroll bar is showing, select a value using the keyboard or mouse, or click elsewhere.
下拉清單出現後，使用者可以使用鍵盤瀏覽清單，如果出現捲軸則可以捲動清單，使用鍵盤或滑鼠選擇值，或點擊其他地方。

Clicking a filter option, or selecting an option and pressing ENTER, calls the `UpdateFilter` method (described in a following section) followed by the `HideDropDownList` method. Clicking somewhere other than the drop-down list so that the drop-down list loses input focus calls `HideDropDownList`, which removes the event handlers from their `dropDownListBox` events in addition to hiding the list control and removing it from the `DataGridView` control. The `HideDropDownList` method is also called when the user presses ESC or when a `DataGridView` event occurs that would change the location or contents of the drop-down list.
按一下篩選選項，或選擇一個選項後按下 ENTER，會呼叫 `UpdateFilter` 方法（後續章節所述）接著呼叫 `HideDropDownList` 方法。若在 drop-down 列表以外的位置點擊，導致 drop-down 列表失去輸入焦點，則會呼叫 `HideDropDownList` ，它除了隱藏列表控制項並從 DataGridView 控制項中移除它之外，還會從其 `dropDownListBox` 事件中移除事件處理程序。當使用者按下 ESC 鍵，或發生會改變 drop-down 列表位置或內容的 DataGridView 事件時，也會呼叫 `HideDropDownList` 方法。

Hiding the drop-down list in certain `DataGridView` event handlers is necessary because some UI elements, such as `ToolStrip`-related controls, do not capture input focus. If user interactions with such controls result in changes to the `DataGridView` control or its contents, it might be inappropriate to continue displaying the drop-down list. For example, if the user clicks a "Show All" link on a `StatusStrip` control, like the one described earlier, the data source will become unfiltered. In this case, if the drop-down list is still showing, it may contain incorrect filter options based on the previous filter setting, and it may also be in the wrong location if the filter change caused several columns to resize.
隱藏下拉清單在某些 DataGridView 事件處理器中是必要的，因為某些 UI 元素，例如與 ToolStrip 相關的控件，不會攔截輸入焦點。如果使用者與這些控件的互動導致 DataGridView 控制項或其內容變更，繼續顯示下拉清單可能不適當。例如，如果使用者點擊先前描述的 StatusStrip 控件上的「顯示全部」連結，資料來源將會取消過濾。在此情況下，如果下拉清單仍然顯示，它可能包含基於先前過濾設定的錯誤過濾選項，而且如果過濾變更導致多個欄位變更大小，它也可能位於錯誤的位置。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-showdropdownlist-method)

### The ShowDropDownList Method

ShowDropDownList 方法

The details of displaying the drop-down list are in the `ShowDropDownList` method. This method performs the following actions:
顯示下拉清單的細節在 `ShowDropDownList` 方法中。此方法執行以下動作：

1. Populates the `dropDownListBox.Items` collection with filter options. The primary task is to retrieve values from the data source. This task is handled by the `PopulateFilters` method, described in the following section. After the filter values have been retrieved, the `ShowDropDownList` method adds them to the `Items` collection and highlights the current filter value if there is one in effect for the column.
    填補 `dropDownListBox.Items` 集合的過濾選項。主要任務是從數據來源中擷取值。此任務由以下節中描述的 `PopulateFilters` 方法處理。在擷取過濾值之後， `ShowDropDownList` 方法將它們添加到 Items 集合中，並高亮顯示當前有效的過濾值（如果有的話）。
2. Sets the `dropDownListBox.Bounds` property based on the`dropDownListBox`  contents and several other factors. This action is handled by the `SetDropDownListBoxBounds` method, which is described in a following section.
    根據 `dropDownListBox` 的內容以及其他幾個因素設定 `dropDownListBox.Bounds` 屬性。此操作由以下節中描述的 `SetDropDownListBoxBounds` 方法處理。
3. Associates event handlers with `dropDownListBox` events to manage user interactions with the drop-down list.
    將事件處理程序與 `dropDownListBox` 事件關聯，以管理用戶與下拉列表的互動。
4. Displays the newly configured `dropDownListBox` on the `DataGridView` control.
    在 DataGridView 控制項上顯示新配置的 `dropDownListBox` 。

As soon as the drop-down list appears, the user can interact with it as described earlier.
一旦下拉清單出現，使用者就可以像先前所描述的那樣與它互動。

The source code for the `ShowDropDownList` method follows:
`ShowDropDownList` 方法的原始碼如下：

  public void ShowDropDownList()
  {
      PopulateFilters();

      String[] filterArray = new String[filters.Count];
      filters.Keys.CopyTo(filterArray, 0);
      dropDownListBox.Items.Clear();
      dropDownListBox.Items.AddRange(filterArray);
      dropDownListBox.SelectedItem = selectedFilterValue;

      HandleDropDownListBoxEvents();

      SetDropDownListBoxBounds();
      dropDownListBox.Visible = true;
      dropDownListBoxShowing = true;
      this.DataGridView.Controls.Add(dropDownListBox);
      dropDownListBox.Focus();

      // Invalidate the cell so that the drop-down button will repaint
      // in the pressed state.
      this.DataGridView.InvalidateCell(this);
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-populatefilters-method)

### The PopulateFilters Method

PopulateFilters 方法

The drop-down filter list for a particular column must contain one copy of each value that appears in the column. This assumes that the column is not currently applying a filter. If the column is filtered, the filter must be ignored so that the list values are the same as if it weren't filtered. The filter list contains only values that can appear in the column regardless of the column's own filter, so the filter values for all other columns remain in effect even though the column's own filter value is not in effect. 
特定欄位的下拉篩選清單必須包含該欄位中出現的每個值的副本。這假設該欄位目前沒有套用篩選。如果該欄位已篩選，則必須忽略篩選，以便清單值與未篩選時相同。篩選清單僅包含無論欄位本身的篩選如何都會出現在欄位中的值，因此即使該欄位自己的篩選值未生效，其他所有欄位的篩選值仍然有效。

The values displayed in `dropDownFilterList` must be formatted just as they are for display in the `DataGridView` control. When a filter value is selected, however, its formatted value is not always compatible with the `BindingSource.Filter` property. For this reason, string representations of both the formatted and unformatted values are stored in an `OrderedDictionary` instance called `filters`. An `OrderedDictionary` is used so that the `ShowDropDownList` method can populate `dropDownListBox` by accessing the formatted values in their correct order through the dictionary's `Keys` collection. When a filter value is selected, the `UpdateFilter` method can use the formatted display value as a dictionary key to retrieve the unformatted value, which is compatible with the `BindingSource.Filters` property.
`dropDownFilterList` 顯示的值必須與 DataGridView 控制項中的顯示格式相同。然而，當選擇過濾值時，其格式化值有時與 BindingSource.Filter 屬性不兼容。因此，格式化值和未格式化值的字串表示都存儲在名為 `filters` 的 OrderedDictionary 實例中。使用 OrderedDictionary 是為了讓 `ShowDropDownList` 方法能夠通過字典的 Keys 集合，以正確的順序訪問格式化值來填充 `dropDownListBox` 。當選擇過濾值時， `UpdateFilter` 方法可以使用格式化顯示值作為字典鍵來檢索與 BindingSource.Filters 屬性兼容的未格式化值。

To retrieve the necessary filter values, the `PopulateFilters` method performs the following actions:
為了檢索必要的過濾值， `PopulateFilters` 方法執行以下操作：

1. Sets the `BindingSource.RaiseListChangedEvents` property to `false`. This lets the `PopulateFilters` method modify the current `BindingSource.Filter` property value without causing the `DataGridView` to refresh its display.
    將 BindingSource.RaiseListChangedEvents 屬性設置為 false。這允許 `PopulateFilters` 方法在不引起 DataGridView 刷新其顯示的情況下修改當前 BindingSource.Filter 屬性值。
2. Caches the current `Filter` value.
    緩存當前 Filter 值。
3. Calls the `FilterWithoutCurrentColumn` method, which parses the current filter string by removing the part related to the current column.
    呼叫 `FilterWithoutCurrentColumn` 方法，將目前的過濾字串解析，移除與目前欄位相關的部分。
4. Sets the `Filter` property to the parsed value.
    將 Filter 屬性設為解析後的值。
5. Clears the filters dictionary.
    清除過濾字典。
6. Retrieves the current column's value for each row in the `BindingSource` and adds it to an `ArrayList`. `Null` and `DBNull.Value` values are excluded, but their presence is noted for later.
    從 BindingSource 中取得目前欄位每行的值，並將其加入至 ArrayList 中。排除 Null 和 DBNull.Value 值，但記錄其存在，以供後續使用。
7. Sorts the `ArrayList`. The `ArrayList.Sort` method uses the `IComparable` implementation of the value type, so strings will be sorted alphabetically, numeric values numerically, and `DateTime` values according to calendar order.
    對 ArrayList 進行排序。ArrayList.Sort 方法使用值類型的 IComparable 執行個體，因此字串將會依字母順序排序，數值將會依數值大小排序，DateTime 值則會依曆書順序排序。
8. For each value in the sorted `ArrayList`, determines its formatted string representation by calling the `DataGridViewCell.GetFormattedValue` method and passing in the column's `InheritedStyle` property value. This ensures that the values appear in the drop-down list with the same formatting they have in the `DataGridView` cells. 
    對於排序後的 ArrayList 中的每個值，透過呼叫 DataGridViewCell.GetFormattedValue 方法並傳入該欄位的 InheritedStyle 屬性值，來判斷其格式化字串表示。這可確保值在下拉清單中顯示的格式與 DataGridView 單元格中的格式相同。
9. Adds each formatted value along with the unformatted string representation of each value to the filters dictionary if it has not already been added, excluding empty strings, but noting their presence.
    若過濾器字典尚未包含該值，則將每個格式化值以及每個值的未格式化字串表示加入至過濾器字典中，但排除空白字串，並註記其存在。
10. Adds special filter options to the filters dictionary as follows:
    將特殊過濾選項加入至過濾器字典中，如下所示：
    - Always adds (All) as the first item in the list.
        永遠將（全部）作為清單中的第一項加入。
    - Adds (Blanks) and (NonBlanks) to the end of the list if the column contained both empty strings (or nulls) and non-empty strings.
        若該欄位同時包含空白字串（或 null）和非空白字串，則會在清單末尾添加 (Blanks) 和 (NonBlanks)。
    - Restores the `BindingSource.Filter` property to the cached value.
        將 BindingSource.Filter 屬性恢復至快取值。
    - Sets the `BindingSource.RaiseListChangedEvents` property to `true` to resume normal operations.
        將 BindingSource.RaiseListChangedEvents 屬性設為 true 以恢復正常運作。

The source code for the `PopulateFilters` method follows:
`PopulateFilters` 方法的原始碼如下：

  private void PopulateFilters()
  {
      if (this.DataGridView == null) return;

      // Cast the data source to a BindingSource.
      BindingSource data = this.DataGridView.DataSource as BindingSource;

      // Prevent the data source from notifying the DataGridView of changes.
      data.RaiseListChangedEvents = false;

      // Cache the current BindingSource.Filter value and then change
      // the Filter property to temporarily remove any filter for the
      // current column.
      String oldFilter = data.Filter;
      data.Filter = FilterWithoutCurrentColumn(oldFilter);

      // Reset the filters dictionary and initialize some flags
      // to track whether special filter options are needed.
      filters.Clear();
      Boolean containsBlanks = false;
      Boolean containsNonBlanks = false;

      // Initialize an ArrayList to store the values in their original
      // types. This enables the values to be sorted appropriately. 
      ArrayList list = new ArrayList(data.Count);

      // Retrieve each value and add it to the ArrayList if it isn't
      // already present.
      foreach (Object item in data)
      {
          Object value = null;

          // Use the ICustomTypeDescriptor interface to retrieve properties
          // if it is available; otherwise, use reflection. The
          // ICustomTypeDescriptor interface is useful to customize

          // which values are exposed as properties. For example, the
          // DataRowView class implements ICustomTypeDescriptor to expose
          // cell values as property values.       
          //
          // Iterate through the property names to find a case-insensitive
          // match with the DataGridViewColumn.DataPropertyName value.
          // This is necessary because DataPropertyName is case-
          // insensitive, but the GetProperties and GetProperty methods
          // used below are case-sensitive.
          ICustomTypeDescriptor ictd = item as ICustomTypeDescriptor;
          if (ictd != null)
          {
              PropertyDescriptorCollection properties =
                  ictd.GetProperties();
              foreach (PropertyDescriptor property in properties)
              {
                  if (String.Compare(this.OwningColumn.DataPropertyName,
                      property.Name, true /*case insensitive*/,
                      System.Globalization.CultureInfo.InvariantCulture)

                      == 0)
                  {
                      value = property.GetValue(item);
                      break;
                  }
              }
          }
          else
          {
              PropertyInfo[] properties = item.GetType().GetProperties(
                  BindingFlags.Public | BindingFlags.Instance);
              foreach (PropertyInfo property in properties)
              {
                  if (String.Compare(this.OwningColumn.DataPropertyName,
                      property.Name, true /*case insensitive*/,
                     System.Globalization.CultureInfo.InvariantCulture)
                      == 0)
                  {
                      value = property.GetValue(item,
                          null /*property index*/);
                      break;
                  }
              }
          }

          // Skip empty values, but note that they are present.
          if (value == null || value == DBNull.Value)
          {
              containsBlanks = true;
              continue;
          }

          // Add values to the ArrayList if they are not already there.
          if (!list.Contains(value))
          {
              list.Add(value);
          }
      }

      // Sort the ArrayList. The default Sort method uses the IComparable
      // implementation of the stored values so that string, numeric, and
      // date values will all be sorted correctly.
      list.Sort();

      // Convert each value in the ArrayList to its formatted representation
      // and store both the formatted and unformatted string representations
      // in the filters dictionary.
      foreach (Object value in list)
      {
          // Use the cell's GetFormattedValue method with the column's
          // InheritedStyle property so that the dropDownListBox format
          // will match the display format used for the column's cells.
          String formattedValue = null;
          DataGridViewCellStyle style = OwningColumn.InheritedStyle;
          formattedValue = (String)GetFormattedValue(value, -1, ref style,
              null, null, DataGridViewDataErrorContexts.Formatting);

          if (String.IsNullOrEmpty(formattedValue))
          {
              // Skip empty values, but note that they are present.
              containsBlanks = true;
          }
          else if (!filters.Contains(formattedValue))
          {
              // Note whether non-empty values are present.
              containsNonBlanks = true;

              // For all non-empty values, add the formatted and
              // unformatted string representations to the filters
              // dictionary.
              filters.Add(formattedValue, value.ToString());
          }
      }

      // Restore the filter to the cached filter string and
      // re-enable data source change notifications.
      if (oldFilter != null) data.Filter = oldFilter;
      data.RaiseListChangedEvents = true;

      // Add special filter options to the filters dictionary
      // along with null values, since unformatted representations
      // are not needed.
      filters.Insert(0, "(All)", null);
      if (containsBlanks && containsNonBlanks)
      {
          filters.Add("(Blanks)", null);
          filters.Add("(NonBlanks)", null);
      }
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-setdropdownlistboxbounds-method)

### The SetDropDownListBoxBounds Method

SetDropDownListBoxBounds 方法

The `SetDropDownListBoxBounds` method initializes the size and location of the drop-down list. The preferred size depends primarily on the `dropDownListBox` contents, which are the formatted values stored in the `Keys` collection of the `filters` dictionary. The `SetDropDownListBoxBounds` method first calls the `Graphics.MeasureString` method for each filter value. For each value, the width is stored if it is wider than all previous values, and the height is added to an accumulating total height for all values. The results are then used to determine the preferred size.
`SetDropDownListBoxBounds` 方法初始化下拉清單的大小和位置。建議的大小主要取決於 `dropDownListBox` 內容，這些是儲存在 `filters` 字典的 Keys 集合中的格式化值。 `SetDropDownListBoxBounds` 方法首先對每個篩選值呼叫 Graphics.MeasureString 方法。對於每個值，如果它比所有先前值更寬，則儲存其寬度，並將其高度加到所有值的累積總高度中。然後使用這些結果來確定建議的大小。

The preferred height is the smallest of the following values:
建議的高度是以下值中最小的一個：

- The accumulated height of all filter values.
    所有篩選值的累積高度。
- The user-specified maximum height as calculated from the `DropDownListBoxMaxLines` property value.
    用戶指定的最大高度，這是根據 `DropDownListBoxMaxLines` 屬性值計算得出的。
- The available height of the `DataGridView` control client area.
    DataGridView 控制項客戶區域的可用高度。

The preferred width is the width of the widest filter value, plus the width of the scrollbar if the preferred height does not display all `dropDownListBox` items, plus a small amount for padding.
理想寬度是最大篩選值的寬度，如果理想高度無法顯示所有 `dropDownListBox` 項目，則加上滾動條的寬度，並加上一點填充寬度。

The preferred `dropDownListBox` location is based on the location of the drop-down button and the edge of the `DataGridView` control. The right edge of the list should ideally be aligned with the right edge of the drop-down button. If `RightToLeft` is enabled, the left edge of the list is aligned to the left edge of the button instead. If aligning the list to the button would make it overlap the edge of the `DataGridView` control, however, then the overlapping edge of the list should be at the edge of the control instead.
理想 `dropDownListBox` 位置基於下拉按鈕的位置和 DataGridView 控制項的邊緣。清單的右邊緣應該與下拉按鈕的右邊緣對齊。如果啟用了 RightToLeft，則清單的左邊緣與按鈕的左邊緣對齊。然而，如果將清單與按鈕對齊會使其與 DataGridView 控制項的邊緣重疊，那麼清單的重疊邊緣應該在控制項的邊緣。

After the `dropDownListBox` size and location have been specified, the drop-down list is ready for display.
在指定了 `dropDownListBox` 大小和位置後，下拉清單就準備好顯示了。

The source code for the `SetDropDownListBoxBounds` method follows:
`SetDropDownListBoxBounds` 方法的原始碼如下：

  private void SetDropDownListBoxBounds()
  {
      // Declare variables that will be used in the calculation,
      // initializing dropDownListBoxHeight to account for the
      // ListBox borders.
      Int32 dropDownListBoxHeight = 2;
      Int32 currentWidth = 0;
      Int32 dropDownListBoxWidth = 0;
      Int32 dropDownListBoxLeft = 0;

      // For each formatted value in the filters dictionary Keys collection,
      // add its height to dropDownListBoxHeight and, if it is wider than
      // all previous values, set dropDownListBoxWidth to its width.
      using (Graphics graphics = dropDownListBox.CreateGraphics())
      {
          foreach (String filter in filters.Keys)
          {
              SizeF stringSizeF = graphics.MeasureString(
                  filter, dropDownListBox.Font);
              dropDownListBoxHeight += (Int32)stringSizeF.Height;
              currentWidth = (Int32)stringSizeF.Width;
              if (dropDownListBoxWidth < currentWidth)
              {
                  dropDownListBoxWidth = currentWidth;
              }
          }
      }

      // Increase the width to allow for horizontal margins and borders.
      dropDownListBoxWidth += 6;

      // Constrain the dropDownListBox height to the
      // DropDownListBoxMaxHeightInternal value, which is based on

      // the DropDownListBoxMaxLines property value but constrained by
      // the maximum height available in the DataGridView control.
      if (dropDownListBoxHeight > DropDownListBoxMaxHeightInternal)
      {
          dropDownListBoxHeight = DropDownListBoxMaxHeightInternal;

          // If the preferred height is greater than the available height,
          // adjust the width to accommodate the vertical scroll bar.
          dropDownListBoxWidth += SystemInformation.VerticalScrollBarWidth;
      }

      // Calculate the ideal location of the left edge of dropDownListBox
      // based on the location of the drop-down button and taking the
      // RightToLeft property value into consideration.
      if (this.DataGridView.RightToLeft == RightToLeft.No)
      {
          dropDownListBoxLeft = DropDownButtonBounds.Right -
              dropDownListBoxWidth + 1;
      }
      else
      {
          dropDownListBoxLeft = DropDownButtonBounds.Left - 1;
      }

      // Determine the left and right edges of the available horizontal
      // width of the DataGridView control.
      Int32 clientLeft = 1;
      Int32 clientRight = this.DataGridView.ClientRectangle.Right;
      if (this.DataGridView.DisplayedRowCount(false) <
          this.DataGridView.RowCount)
      {
          if (this.DataGridView.RightToLeft == RightToLeft.Yes)
          {
              clientLeft += SystemInformation.VerticalScrollBarWidth;
          }
          else
          {
              clientRight -= SystemInformation.VerticalScrollBarWidth;
          }
      }

      // Adjust the dropDownListBox location and/or width if it would
      // otherwise overlap the left or right edge of the DataGridView.
      if (dropDownListBoxLeft < clientLeft)
      {
          dropDownListBoxLeft = clientLeft;
      }
      Int32 dropDownListBoxRight =
          dropDownListBoxLeft + dropDownListBoxWidth + 1;
      if (dropDownListBoxRight > clientRight)
      {
          if (dropDownListBoxLeft == clientLeft)
          {
              dropDownListBoxWidth -=
                  dropDownListBoxRight - clientRight;
          }
          else
          {
              dropDownListBoxLeft -=
                  dropDownListBoxRight - clientRight;
              if (dropDownListBoxLeft < clientLeft)
              {
                  dropDownListBoxWidth -= clientLeft - dropDownListBoxLeft;
                  dropDownListBoxLeft = clientLeft;
              }
          }
      }

      // Set the ListBox.Bounds property using the calculated values.
      dropDownListBox.Bounds = new Rectangle(dropDownListBoxLeft,
          DropDownButtonBounds.Bottom, // top of drop-down list box
          dropDownListBoxWidth, dropDownListBoxHeight);
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#handling-listbox-events)

### Handling ListBox Events  處理 ListBox 事件

The `dropDownListBox` events that need handling are the `LostFocus`, `MouseClick`, and `KeyDown` events. If the user clicks somewhere other than the drop-down list so that the drop-down list loses input focus, the `LostFocus` event handler calls the `HideDropDownList` method. If the user clicks a filter option or presses ENTER, the `MouseClick` or `KeyDown` event handler calls the `UpdateFilter` method then the `HideDropDownList` method. If the user presses ESC, the `KeyDown` event handler calls just the `HideDropDownList` method.
需要處理的事件有 LostFocus、MouseClick 和 KeyDown 事件。如果使用者點擊下拉清單以外的位置，導致下拉清單失去輸入焦點，LostFocus 事件處理程序會呼叫 `HideDropDownList` 方法。如果使用者點擊篩選選項或按下 ENTER，MouseClick 或 KeyDown 事件處理程序會先呼叫 `UpdateFilter` 方法，然後呼叫 `HideDropDownList` 方法。如果使用者按下 ESC，KeyDown 事件處理程序僅呼叫 `HideDropDownList` 方法。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#filtering-the-bound-data)

## Filtering the Bound Data  篩選綁定資料

When the user clicks a filter option from a drop-down list for a particular column, the data source is filtered so that only rows with the selected value in that column are displayed. Because there is only one `BindingSource.Filter` property for the entire data source, the selected filter option must be combined with the filter options for all columns into a single filter string.
當使用者從特定欄位的下拉清單點擊篩選選項時，資料來源會被篩選，僅顯示該欄位中選定值的行。由於整個資料來源僅有一個 BindingSource.Filter 屬性，因此選定的篩選選項必須與所有欄位的篩選選項組合為單一篩選字串。

There are four methods that work with filter strings: `UpdateFilter`, `FilterWithoutCurrentColumn`, `RemoveFilter`, and `GetFilterStatus`.
有四個與篩選字串互動的方法： `UpdateFilter` 、 `FilterWithoutCurrentColumn` 、 `RemoveFilter` 和 `GetFilterStatus` 。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-updatefilter-method)

### The UpdateFilter Method  更新篩選方法

The `UpdateFilter` method is responsible for modifying the `BindingSource.Filter` value in response to a user selection in the drop-down filter list. To do this, it performs the following actions:
此 `UpdateFilter` 方法負責在用戶從下拉篩選清單中選擇時，修改 BindingSource.Filter 的值。為此，它執行以下操作：

1. Calls the `FilterWithoutCurrentColumn` method to retrieve a parsed filter string that does not include the current column's filter value.
    呼叫 `FilterWithoutCurrentColumn` 方法以取得未包含當前欄位篩選值的解析篩選字串。
2. If the user selected the (All) option, sets the Filter property to the parsed value.
    如果用戶選擇了 (全部) 選項，則將 Filter 屬性設為解析值。
3. If the user selected an option other than (All), creates a filter string for the column and adds it to the current Filter value.
    若使用者選擇的選項不是 (全部)，則為該欄位建立過濾字串，並將其加入目前的過濾值。

The filter string for ordinary filter values is in the following form:
一般過濾值的過濾字串格式如下：

  [_columnName_]='_filterValue_'

For the (Blanks) and (NonBlanks) options, the filter string converts the value to a string, using the empty string for null values, and then tests whether the converted value is of zero length. These filter strings are in the following forms:
對於 (空白) 和 (非空白) 選項，過濾字串會將值轉換為字串，對於 null 值使用空字串，然後測試轉換後的值是否為零長度。這些過濾字串的格式如下：

  LEN(ISNULL(CONVERT([_columnName_],'System.String'),''))=0
  LEN(ISNULL(CONVERT([_columnName_],'System.String'),''))>0

If the `BindingSource.Filter` property value is null or empty, the `UpdateFilter` method sets it to the column filter string. If the `Filter` property is not null or empty, the column filter is appended to the end of the Filter value and delimited from existing values with the string `" AND "`.
如果 BindingSource.Filter 屬性值為 null 或空白， `UpdateFilter` 方法會將其設為欄位過濾字串。如果 Filter 屬性不是 null 或空白，則將欄位過濾值附加到 Filter 值的末尾，並使用字串 `" AND "` 與現有值分隔。

The source code for the `UpdateFilter` method follows:
`UpdateFilter` 方法的原始碼如下：

  private void UpdateFilter()
  {
      // Continue only if the selection has changed.
      if (dropDownListBox.SelectedItem.ToString()
          .Equals(selectedFilterValue))
      {
          return;
      }

      // Store the new selection value.
      selectedFilterValue = dropDownListBox.SelectedItem.ToString();

      // Cast the data source to an IBindingListView.
      IBindingListView data =
          this.DataGridView.DataSource as IBindingListView;

      // If the user selection is (All), remove any filter currently
      // in effect for the column.
      if (selectedFilterValue.Equals("(All)"))
      {
          data.Filter = FilterWithoutCurrentColumn(data.Filter);
          filtered = false;
          currentColumnFilter = String.Empty;
          return;
      }

      // Declare a variable to store the filter string for this column.
      String newColumnFilter = null;

      // Store the column name in a form acceptable to the Filter property,
      // using a backslash to escape any closing square brackets.

      String columnProperty =
          OwningColumn.DataPropertyName.Replace("]", @"\]");

      // Determine the column filter string based on the user selection.
      // For (Blanks) and (NonBlanks), the filter string determines whether
      // the column value is null or an empty string. Otherwise, the filter
      // string determines whether the column value is the selected value.
      switch (selectedFilterValue)
      {
          case "(Blanks)":
              newColumnFilter = String.Format(
                  "LEN(ISNULL(CONVERT([{0}],'System.String'),''))=0",
                  columnProperty);
              break;
          case "(NonBlanks)":
              newColumnFilter = String.Format(
              "LEN(ISNULL(CONVERT([{0}],'System.String'),''))>0",
                  columnProperty);
              break;
          default:
              newColumnFilter = String.Format("[{0}]='{1}'",
                  columnProperty,
                  ((String)filters[selectedFilterValue])
                  .Replace("'", "''")); 
              break;
      }

      // Determine the new filter string by removing the previous column
      // filter string from the BindingSource.Filter value, then appending
      // the new column filter string, using " AND " as appropriate.
      String newFilter = FilterWithoutCurrentColumn(data.Filter);
      if (String.IsNullOrEmpty(newFilter))
      {
          newFilter += newColumnFilter;
      }
      else
      {
          newFilter += " AND " + newColumnFilter;
      }

      // Set the filter to the new value.
      try
      {
          data.Filter = newFilter;
      }
      catch (InvalidExpressionException ex)
      {
          throw new NotSupportedException(
              "Invalid expression: " + newFilter, ex);
      }

      // Indicate that the column is currently filtered
      // and store the new column filter for use by subsequent
      // calls to the FilterWithoutCurrentColumn method.
      filtered = true;
      currentColumnFilter = newColumnFilter;
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-filterwithoutcurrentcolumn-method)

### The FilterWithoutCurrentColumn Method

FilterWithoutCurrentColumn 方法

The `FilterWithoutCurrentColumn` method parses a given filter string in order to remove the filter for the current column. Although it does not modify the `BindingSource.Filter` property directly, this method is used by the `UpdateFilter` and `PopulateFilters` methods to modify the `Filter` property.
`FilterWithoutCurrentColumn` 方法會解析給定的篩選字串，以便移除當前欄位的篩選。雖然它不直接修改 BindingSource.Filter 屬性，但此方法會被 `UpdateFilter` 和 `PopulateFilters` 方法使用來修改 Filter 屬性。

To parse the filter string, the `FilterWithoutCurrentColumn` method uses the `currentColumnFilter` field, which stores only the current column's portion of the `BindingSource.Filter` property value. The `FilterWithoutCurrentColumn` method searches the specified filter string for the `currentColumnFilter` value. If the column value is found, it returns a copy of the specified string without the column value, and without any extraneous `" AND "` delimiters that prevent the return value from being a valid filter string.
為了解析篩選字串， `FilterWithoutCurrentColumn` 方法會使用 `currentColumnFilter` 字段，該字段僅儲存 BindingSource.Filter 屬性值中當前欄位的部分。 `FilterWithoutCurrentColumn` 方法會在指定的篩選字串中搜尋 `currentColumnFilter` 值。如果找到欄位值，則會返回指定字串的副本，但不包含欄位值，也無任何額外的 `" AND "` 隔離符，這些隔離符會阻止返回值成為有效的篩選字串。

The source code for the `FilterWithoutCurrentColumn` method follows:
`FilterWithoutCurrentColumn` 方法的原始碼如下：

  private String FilterWithoutCurrentColumn(String filter)
  {
      // If there is no filter in effect, return String.Empty.
      if (String.IsNullOrEmpty(filter))
      {
          return String.Empty;
      }

        // If the column is not filtered, return the filter string unchanged. 
      if (!filtered)
      {
          return filter;
      }

      if (filter.IndexOf(currentColumnFilter) > 0)
      {
          // If the current column filter is not the first filter, return
          // the specified filter value without the current column filter
          // and without the preceding " AND ".
          return filter.Replace(
              " AND " + currentColumnFilter, String.Empty);
      }
      else
      {
          if (filter.Length > currentColumnFilter.Length)
          {
              // If the current column filter is the first of multiple
              // filters, return the specified filter value without the
              // current column filter and without the subsequent " AND ".
              return filter.Replace(
                  currentColumnFilter + " AND ", String.Empty);
          }
          else
          {
              // If the current column filter is the only filter,
              // return the empty string.
              return String.Empty;
          }
      }
  }

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480727\(v=msdn.10\)#the-removefilter-and-getfilterstatus-methods)

### The RemoveFilter and GetFilterStatus Methods

移除篩選與取得篩選狀態方法

The static `RemoveFilter` and `GetFilterStatus` methods are provided to make it easier for a client application to display a filter status string and expose a `Show All` option. They are `static` because they relate to the data source rather than to an individual cell. The data source can be different for different `DataGridView` controls, however, so these methods require the client application to pass in a reference to a `DataGridView` control. Although these methods are implemented on the cell class, they are also exposed through the `DataGridViewAutoFilterTextBoxColumn` class for convenience.
提供的靜態 `RemoveFilter` 和 `GetFilterStatus` 方法，讓客戶應用程式更容易顯示篩選狀態字串並揭露「顯示全部」選項。它們是靜態的，因為它們與資料來源相關，而不是與個別單元格相關。不過，資料來源可能會因不同的 DataGridView 控制項而不同，因此這些方法需要客戶應用程式傳入一個 DataGridView 控制項的參考。雖然這些方法是在單元格類別上實現的，但它們也透過 `DataGridViewAutoFilterTextBoxColumn` 類別公開，以提供便利。

The `RemoveFilter` method sets the `BindingSource.Filter` property value to `null` to remove all filters. This method enables client applications to implement a `Show All` option without having to access the `Filter` property directly. A client application can set the `Filter` property to `null` or `String.Empty` without causing any problems, but setting it to any other value may interfere with the filtering provided by the `DataGridViewAutoFilter` library. For this reason, it is a good idea to keep modifications to the `Filter` property hidden from client code.
`RemoveFilter` 方法會將 BindingSource.Filter 屬性設定為 null 以移除所有篩選。這個方法讓客戶應用程式能夠實現 Show All 選項，而不必直接存取 Filter 屬性。客戶應用程式可以將 Filter 屬性設定為 null 或 String.Empty 而不會造成問題，但設定為其他值可能會干擾 `DataGridViewAutoFilter` 庫提供的篩選功能。因此，將 Filter 屬性的修改對客戶程式碼隱藏是一個好主意。

The `GetFilterStatus` method returns a status string containing the filtered and unfiltered `BindingSource.Count` property values. This method does more work than the `RemoveFilter` method, since it must temporarily modify the `Filter` property value to retrieve the unfiltered count. It does this by using the same process as described earlier for the `PopulateFilters` method. If the data source is not currently filtered, the `GetFilterStatus` method returns an empty string. Otherwise, it returns a string in the following format:
`GetFilterStatus` 方法會回傳一個包含已篩選與未篩選的 BindingSource.Count 屬性值的狀態字串。此方法比 `RemoveFilter` 方法需要做更多的工作，因為它必須暫時修改 Filter 屬性值才能取得未篩選的計數。它透過與先前描述的 `PopulateFilters` 方法相同的方式來做到這一點。如果資料來源目前沒有被篩選， `GetFilterStatus` 方法會回傳一個空白字串。否則，它會回傳下列格式的字串：

_unfilteredCount_ of _filteredCount_ records found
未過濾數量為過濾記錄數量的記錄數

The source code for the `GetFilterStatus` method follows:
`GetFilterStatus` 方法的原始碼如下：

  public static String GetFilterStatus(DataGridView dataGridView)
  {
      // Continue only if the specified value is valid.
      if (dataGridView == null)
      {
          throw new ArgumentNullException("dataGridView");
      }

      // Cast the data source to a BindingSource.
      BindingSource data = dataGridView.DataSource as BindingSource;

      // Return String.Empty if there is no appropriate data source or
      // there is no filter in effect.
      if (String.IsNullOrEmpty(data.Filter) ||
          data == null ||
          data.DataSource == null ||
          !data.SupportsFiltering)
      {
          return String.Empty;
      }

      // Retrieve the filtered row count.
      Int32 currentRowCount = data.Count;

      // Retrieve the unfiltered row count by
      // temporarily unfiltering the data.
      data.RaiseListChangedEvents = false;
      String oldFilter = data.Filter;
      data.Filter = null;
      Int32 unfilteredRowCount = data.Count;
      data.Filter = oldFilter;
      data.RaiseListChangedEvents = true;

      Debug.Assert(currentRowCount <= unfilteredRowCount,
          "current count is greater than unfiltered count");

      // Return String.Empty if the filtered and unfiltered counts
      // are the same, otherwise, return the status string.
      if (currentRowCount == unfilteredRowCount)
      {
          return String.Empty;
      }
      return String.Format("{0} of {1} records found",
          currentRowCount, unfilteredRowCount);
  }

For more information about using the `RemoveFilter` and `GetFilterStatus` methods, see "Using the `DataGridViewAutoFilter` library" earlier in this article.
更多有關使用 `RemoveFilter` 和 `GetFilterStatus` 方法的資訊，請參閱本文稍早提及的「使用 `DataGridViewAutoFilter` 庫」。

# Additional Properties  額外屬性

The`DataGridViewAutoFilterColumnHeaderCell` class adds three properties to those it inherits from the `DataGridViewColumnHeaderCell` base class: `AutomaticSortingEnabled`, `FilteringEnabled`, and `DropDownListBoxMaxLines`.
`DataGridViewAutoFilterColumnHeaderCell` 類別向繼承自 DataGridViewColumnHeaderCell 基類的屬性中新增了三個屬性： `AutomaticSortingEnabled` 、 `FilteringEnabled` 和 `DropDownListBoxMaxLines` 。

The `AutomaticSortingEnabled` property lets you disable automatic sorting while keeping a column `SortMode` property value of `Programmatic`. Automatic sorting with the AutoFilter feature requires the `SortMode` property value to be `Programmatic` to prevent clicks on the drop-down button from sorting the column. If you want to handle sorting yourself, however, you must leave the `SortMode` property set to `Programmatic` in order to reserve space in the column header for the sorting glyph.
`AutomaticSortingEnabled` 屬性可讓您停用自動排序，同時保持欄位 SortMode 屬性的值為 Programmatic。使用 AutoFilter 功能的自動排序需要 SortMode 屬性的值為 Programmatic，以防止點擊下拉按鈕時排序欄位。如果您想自行處理排序，但必須將 SortMode 屬性保持為 Programmatic，以保留在欄位標頭中放置排序符號的空间。

The `FilteringEnabled` property lets you disable filtering. When `FilteringEnabled` is `false`, the drop-down button is not displayed and the header appears and behaves like an ordinary column header cell.
`FilteringEnabled` 屬性可讓您停用過濾。當 `FilteringEnabled` 為 false 時，下拉按鈕不會顯示，標題會像一般欄位標題細胞一樣出現並行為。

The `DropDownListBoxMaxLines` property lets you customize the preferred maximum number of lines that appear in the drop-down list. The actual height is constrained by the available height in the `DataGridView` control, but this property lets you limit the height even further.
`DropDownListBoxMaxLines` 屬性可讓您自訂下拉清單中出現的偏好最大行數。實際高度會受到 DataGridView 控制項中可用高度的限制，但這個屬性可以讓您進一步限制高度。

These cell properties are also exposed through the `DataGridViewAutoFilterTextBoxColumn` class for convenience.
這些細胞屬性也透過 `DataGridViewAutoFilterTextBoxColumn` 類別提供，以方便使用。

# DataGridViewAutoFilterTextBoxColumn Class Implementation Details

DataGridViewAutoFilterTextBoxColumn 類別實作細節

The `DataGridViewAutoFilterTextBoxColumn` class is provided as an example of how to add AutoFilter support to an existing column type. Having an AutoFilter column type is particularly useful to enable users to add the AutoFilter feature to their applications by using the Windows Forms Designer, as described earlier in this article.
`DataGridViewAutoFilterTextBoxColumn` 類別是作為如何為現有欄位類型添加 AutoFilter 支援的範例。擁有 AutoFilter 欄位類型對於讓使用者能夠透過 Windows Forms Designer 將 AutoFilter 功能添加到他們的應用程式中特別有用，正如本文前面所述。

The important code in the `DataGridViewAutoFilterTextBoxColumn` class is the class constructor, which does nothing except set the `DefaultHeaderCellType` property to the `DataGridViewAutoFilterColumnHeaderCell` type and set the `SortMode` property to `Programmatic` so that clicks on the drop-down button will not trigger automatic sorting.
`DataGridViewAutoFilterTextBoxColumn` 類別中的重要程式碼是類別建構函式，它除了將 DefaultHeaderCellType 屬性設定為 `DataGridViewAutoFilterColumnHeaderCell` 類型，以及將 SortMode 屬性設定為 Programmatic，以便點擊下拉按鈕不會觸發自動排序外，不會做任何事。

  public DataGridViewAutoFilterTextBoxColumn() : base()
  {
      base.DefaultHeaderCellType =
          typeof(DataGridViewAutoFilterColumnHeaderCell);
      base.SortMode = DataGridViewColumnSortMode.Programmatic;
  }

The remaining code in the `DataGridViewAutoFilterTextBoxColumn` class is provided for convenience. The `DefaultHeaderCellType` and `SortMode` properties are reimplemented to hide them from the designer and to throw an exception if the `SortMode` property is set to `Automatic`. Additionally, the column class exposes the new public properties and the static methods of the cell class. These members just wrap the cell members, so setting a property value on a column instance will update the same property in the column's header cell instance.
`DataGridViewAutoFilterTextBoxColumn` 類別中剩下的程式碼是為了方便。DefaultHeaderCellType 和 SortMode 屬性被重新實作，以隱藏它們不讓設計器看到，並且如果 SortMode 屬性被設定為 Automatic，則會拋出例外。此外，欄位類別會公開新的公用屬性以及細胞類別的靜態方法。這些成員只是封裝了細胞成員，因此設定欄位實例的屬性值會更新欄位標頭細胞實例中的相同屬性。

# Possible Enhancements  可能的增強功能

The `DataGridViewAutoFilter` library provides a basic user interface for column filtering and shows you how to customize a DataGridView column header cell. It also demonstrates how to host a Windows Forms control in a cell when you are unable to derive from an existing cell type.
`DataGridViewAutoFilter` 套件提供基本的列過濾使用者介面，並示範如何自訂 DataGridView 列標題單元格。它也說明當無法從現有單元格類型繼承時，如何在單元格中裝載 Windows Forms 控制項。

As an AutoFilter feature, the `DataGridViewAutoFilter` library leaves several areas for potential improvement. The following list describes enhancements that you may want to consider making to the `DataGridViewAutoFilterColumnHeaderCell`  class:
作為自動過濾功能， `DataGridViewAutoFilter` 套件仍有幾個潛在的改進領域。下列清單描述了您可能想要考慮對 `DataGridViewAutoFilterColumnHeaderCell` 類別進行的增強：

- Custom filtering: Displaying a custom-filter dialog box like the one in Excel that allows you to specify filter values such as "contains", "does not contain", "begins with", and so on.
    客製化篩選：顯示類似 Excel 的客製化篩選對話方塊，可讓您指定篩選值，例如 " 包含 "、" 不包含 "、" 開頭為 " 等。
- Filtering with any data source, in unbound mode, or in virtual mode.
    可使用任何資料來源進行篩選，無論是未綁定模式或虛擬模式。
- Support for special cell values, such as images.
    支援特殊單元格值，例如圖片。
- Integration with additional header-cell features, such as a multi-column sort feature with header labels that indicate the sorted columns and their sort precedence.
    與其他標題單元格功能整合，例如具有標示排序欄位及其排序優先順序的標題多欄位排序功能。

# Additional Resources  額外資源

To provide feedback on this article and on the `DataGridViewAutoFilter` library, and to check for updates, see the [Windows Forms Documentation Updates Blog](https://go.microsoft.com/fwlink/?linkid=69202).
提供對此文章和 `DataGridViewAutoFilter` 庫的回饋，並檢查更新，請參閱 Windows Forms 文件更新部落格。

The following resources provide additional information about customizing DataGridView cells:
以下資源提供關於自訂 DataGridView 單元格的額外資訊：

- [Customizing the Windows Forms DataGridView Control](https://go.microsoft.com/fwlink/?linkid=70418) in the [MSDN2 library](https://go.microsoft.com/fwlink/?linkid=70419).
    MSDN2 庫中的「自訂 Windows Forms DataGridView 控制項」。
- "Build a custom RadioButton cell and column for the DataGridView control" ([whitepaper](https://go.microsoft.com/fwlink/?linkid=70420) and [sample](https://go.microsoft.com/fwlink/?linkid=70421)).
    「為 DataGridView 控制項建立自訂 RadioButton 單元格和欄位」（白皮書和範例）。
- "Build a custom NumericUpDown cell and column for the DataGridView control" ([whitepaper](https://go.microsoft.com/fwlink/?linkid=70422) and [sample](https://go.microsoft.com/fwlink/?linkid=70424)).
    " 建置自訂的 NumericUpDown 資料格與欄位，供 DataGridView 控制元件使用 " (白皮書與範例)。

For more information on Windows Forms in general, see:
若想進一步了解 Windows Forms 的相關資訊，請參考：

- [Windows Forms on Microsoft .NET Framework Developer Center](https://go.microsoft.com/fwlink/?linkid=70365).
    Microsoft .NET Framework 開發者中心上的 Windows Forms。
- [WindowsClient.NET](https://go.microsoft.com/fwlink/?linkid=70366).   WindowsClient.NET。
- [MSDN Windows Forms Forum](https://go.microsoft.com/fwlink/?linkid=70425).
    MSDN Windows Forms 论坛
