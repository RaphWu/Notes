---
aliases:
date: 2003-01-20
update:
author: Davide Icardi
language: C#
sourceurl: https://www.codeproject.com/articles/SourceGrid-Open-Source-C-Grid-Control
tags:
  - CSharp
  - WinForm
  - SourceGrid
---

# Introduction<br />介紹

SourceGrid is a Windows Forms control written entirely in C#, my goal is to create a simple but flexible grid to use in all of the cases in which it is necessary to visualize or to change a series of data in a table format. There are a lot of controls of this type available, but often are expensive, difficult to be customize or not compatible with. NET. The Microsoft DataGrid for me is too DataSet orientated and therefore results often complicated to use in the cases in which the source data isn't a DataSet and often is not enough customizable.
SourceGrid 是一個完全以 C# 寫成的 Windows Forms 控制元件，我的目標是創建一個簡單但靈活的網格，用於所有需要視覺化或以表格格式更改一系列數據的情況。有很多這類控制元件可供選擇，但通常很昂貴、難以自定義或與 .NET 不兼容。對我來說，Microsoft DataGrid 太過於 DataSet 指向，因此在來源數據不是 DataSet 的情況下，使用起來往往很複雜，而且通常也不夠可自定義。

I want to thank Chirs Beckett, Anthony Berglas, Wayne Tanner, Ernesto Perales, Vadim Katsman, Jeffery Bell, Gmonkey, cmwalolo, Kenedy, zeromus, Darko Damjanovic, John Pierre and a lot of other persons who helped me with code, bugs report and with new ideas and suggestions.
我想感謝 Chirs Beckett、Anthony Berglas、Wayne Tanner、Ernesto Perales、Vadim Katsman、Jeffery Bell、Gmonkey、cmwalolo、Kenedy、zeromus、Darko Damjanovic、John Pierre 和許多其他幫助我處理程式碼、錯誤報告以及提供新點子和建議的人。

This control is compiled with the Microsoft Framework. NET 1.1 and reference the assembly SourceLibrary.dll 1.2.0.0, this is a small library with common functionality. I introduced this dll in the ZIP file, but is possible to download the entire code and the last binary from the site http://sourcegrid.codeplex.com/. For use SourceGrid is necessary have Visual Studio.NET 2003 or a compatible development environment.
這個控制項是使用 Microsoft 框架 .NET 1.1 編譯的，並參考了組合 SourceLibrary.dll 1.2.0.0，這是一個包含常見功能的輕量級庫。我在 ZIP 檔案中引入了這個 DLL，但您可以從網站 http://sourcegrid.codeplex.com/ 下載整個程式碼和最後的二元檔。要使用 SourceGrid，需要擁有 Visual Studio.NET 2003 或相容的開發環境。

The last version of this control is downloadable from site http://sourcegrid.codeplex.com/.
這個控制項的最後版本可以從網站 http://sourcegrid.codeplex.com/ 下載。

In this article I will want to supply a panoramic on the utilization and on the functionality of the control SourceGrid, for the details on the classes, properties or methods you can consult the documentation in CHM format or the example project in the ZIP file.
在這篇文章中，我將提供 SourceGrid 控制項的應用和功能概觀，有關類別、屬性或方法的細節，您可以參考 CHM 格式的文件或 ZIP 檔案中的範例專案。

# Use SourceGrid<br />使用 SourceGrid

In the assembly `SourceGrid2.dll` are present 2 controls that can be inserted in the Toolbox of Visual Studio and used in any Form:

- `GridVirtual` - A grid of virtual cells (`ICellVirtual`).
- `Grid` - A grid of real cells (`ICell`).

在 `SourceGrid2.dll` 這個組件中有兩個控制項可以插入 Visual Studio 的工具箱中，並且可以用在任何 Form ：

- `GridVirtual` - 一個虛擬單元格的網格 (`ICellVirtual`) 。
- `Grid` - 一個實際單元格的網格 (`ICell`) 。

There are therefore two fundamental distinctions to do: virtual cells and real cells. **Virtual cells** are the cells that determine the appearance and the behavior of the cell but don't contain the value, the **real cells** have the same properties of the virtual cells but contain also the value of the cell, they are therefore associated to a specific position of the grid.
因此有兩個基本區別需要進行：虛擬單元格和實際單元格。**虛擬單元格**是決定單元格外觀和行為的單元格，但不包含值；**實際單元格**具有與虛擬單元格相同的屬性，但還包含單元格的值，因此它們與網格的特定位置相關聯。

![[SourceGrid_OpenSourceCSharpGridControl02.jpg]]

Every cells are composed by three fundamental parts:

- `DataModel` : The `DataModel` is the class that manages the value of the cells. Converts the value of the cell in a string for visual representation, create the editor of the cell and validate the inserted values.
- `VisualModel` : The `VisualModel` is the class that draws the cell and contains the visual properties.
- `BehaviorModel` : The `BehaviorModel` is the class that supplies the behavior of the cell.

每個 cells 由三個基本部分組成：

- `DataModel` ：`DataModel` 是管理單元格值的類別。它將單元格的值轉換為字串以進行視覺化呈現，建立單元格的編輯器並驗證插入的值。
- `VisualModel` ：`VisualModel` 是繪製單元格並包含視覺化屬性的類別。
- `BehaviorModel` ：`BehaviorModel` 是提供儲存格行為的類別。

每一個單元都是由三個基本部分組成：

- `DataModel` : `DataModel` 是管理單元值之類別。將單元值轉換為字串以進行視覺呈現，建立單元的編輯器，並驗證插入的值。
- `VisualModel` : `VisualModel` 是繪製單元並包含視覺屬性的類別。
- `BehaviorModel` : `BehaviorModel` 是提供單元行為的類別。

This subdivision grants a great flexibility and reusability of code, save time and supplies a solid base for every type of customizations. For the more common cases there are some classes already arranged and configured, but with little lines of code is possible to create personalized cells (see the next paragraphs for the details).
此分類提供了極大的彈性和程式碼的可重用性，節省時間，並為所有類型的自訂提供堅實的基礎。對於更常見的情況，有一些類別已經排列和設定好，但只需少量程式碼即可創建自訂單元（請參閱下一節的細節）。

# `Grid`

The `Grid` control is the ideal if you want the greatest flexibility and simplicity but with not many cells. In fact in this control every cells are represented by a .NET class and therefore occupies a specific quantity of resources. Moreover this is the only grid that supports features of `RowSpan` and `ColumnSpan`.
`Grid` 控制項是如果您想要最大的靈活性和簡單性，但單元格不多時的理想選擇。事實上，在這個控制中，每個單元格都由一個 .NET 類別表示，因此佔用特定的資源量。此外，這是唯一支援 `RowSpan` 和 `ColumnSpan` 功能的網格。

After to have inserted the control in the form we can begin to write our code to use the grid in the Load event of the form like this:
在將控制插入表單後，我們可以開始編寫代碼，在表單的 `Load` 事件中使用網格，如下所示：

CS

grid1.Redim(2, 2);
grid1[0,0] = new SourceGrid2.Cells.Real.Cell("Hello from Cell 0,0");
grid1[1,0] = new SourceGrid2.Cells.Real.Cell("Hello from Cell 1,0");
grid1[0,1] = new SourceGrid2.Cells.Real.Cell("Hello from Cell 0,1");
grid1[1,1] = new SourceGrid2.Cells.Real.Cell("Hello from Cell 1,1");
The previous code creates a table with 2 lines and 2 columns (Redim method) and populates every positions with a cell. I have used the SourceGrid2.Cells.Real namespace where are present all the real cells.
之前的程式碼建立了一個有 2 行和 2 欄的表格 ( Redim 方法)，並且將每一個位置都填入單元格。我使用了 SourceGrid2.Cells.Real 命名空間，這裡有所有真正的單元格。

Every cells contains all the necessary display properties, for example to change the background color of the cell we can write:
每一個單元格都包含所有必要的顯示屬性，例如要更改單元格的背景顏色，我們可以寫：

CS

SourceGrid2.Cells.Real.Cell l_Cell = new SourceGrid2.Cells.Real.Cell(
  "Custom back color");
l_Cell.BackColor = Color.LightGreen;
grid1[0,0] = l_Cell;
These are the main visual properties of a cell (for an entire list to consult the documentation of the grid): BackColor, ForeColor, Border, Font, TextAlignment, WordWrap ...,
這些是單元格的主要視覺屬性（要查閱網格的文件以獲取完整列表）： BackColor, ForeColor, Border, Font, TextAlignment, WordWrap ... ，

Now we try to create an entire grid with headers, automatic sort, resize of the columns with the mouse, string and DateTime editor and a checkbox.
現在我們嘗試建立一個整個的網格，包含標題、自動排序、用滑鼠縮放欄位、字串和日期時間編輯器以及核取方塊。

CS

grid1.BorderStyle = BorderStyle.FixedSingle;
grid1.ColumnsCount = 3;
grid1.FixedRows = 1;
grid1.Rows.Insert(0);
grid1[0,0] = new SourceGrid2.Cells.Real.ColumnHeader("String");
grid1[0,1] = new SourceGrid2.Cells.Real.ColumnHeader("DateTime");
grid1[0,2] = new SourceGrid2.Cells.Real.ColumnHeader("CheckBox");
for (int r = 1; r < 10; r++)
{
  grid1.Rows.Insert(r);
  grid1[r,0] = new SourceGrid2.Cells.Real.Cell("Hello "
    + r.ToString(), typeof(string));
  grid1[r,1] = new SourceGrid2.Cells.Real.Cell(
    DateTime.Today, typeof(DateTime));
  grid1[r,2] = new SourceGrid2.Cells.Real.CheckBox(true);
}
grid1.AutoSizeAll();
In the previous code I have set the grid border, the number of columns, the number of fixed rows and created the first header row. For the header I have used a ColumnHeader cell. With a simple cycle for I have then created the other cells using for each column a specific type. The Cell class creates automatically an appropriate editor for the type specified (in this case a TextBox and a DateTimePicker). For the last column I have used a CheckBox cell that allows the display of a checkbox directly on the cell. The form should look equal to the one in the following figure, this example is present also in the project SampleProject with the ZIP file.
在前面的程式中，我已設定網格邊框、列數、固定行數，並建立了第一個標題列。對於標題列，我使用了 ColumnHeader 單元格。接著，我使用一個簡單的迴圈，為每一列創建了特定類型的其他單元格。 Cell 類別會自動為指定的類型（在本例中為 TextBox 和 DateTimePicker）建立適當的編輯器。對於最後一列，我使用了 CheckBox 單元格，它允許在單元格上直接顯示核取方塊。表單應該看起來與下列圖片相同，此範例也包含在 SampleProject 專案中，隨 ZIP 檔案提供。

Article image

GridVirtual
The GridVirtual control is the ideal when is necessary to visualize a lot of cells and you have already available a structure data like a DataSet, an Array, a document XML or other data structure. This type of grid have the same features of the Grid control except for the automatic sort (this because the grid cannot order automatically any external data structure without copying its content) and the feature of RowSpan and ColumnSpan that allow to span a cell across other adjacent cells. Another disadvantage is that to create a virtual grid is a little difficult.
當需要顯示大量單元格且已有一個結構化數據結構，例如 DataSet ， Array ， XML 文件或其他數據結構時， GridVirtual 控制元件是理想選擇。這類網格具有與 Grid 控制元件相同的特性，除了自動排序（因為網格無法在不複製其內容的情況下自動排序任何外部數據結構）以及允許單元格跨越其他相鄰單元格的 RowSpan 和 ColumnSpan 功能。另一個缺點是，建立虛擬網格相對困難。

The main concept in a virtual grid is that the cells do not contain the values, but read and write the value in an external data structure. This idea was implemented with an abstract class CellVirtual in which is necessary to redefine the methods GetValue and SetValue. To use the GridVirtual is therefore necessary to create a class that derives from CellVirtual and to personalize the reading using the data source chosen. Usual is better to create also a control that derive from GridVirtual, to have a greater flexibility and a more solid code, overriding the method GetCell. If you prefer you can directly use the GridVirtual control and the event GettingCell. The purpose of the method GetCell and of the event GettingCell is to return, for a given position (row and column), the chosen cell. This allows a large flexibility because you can return for a specific type any ICellVirtual, for example you could return a cell of type header when the row is 0.
在虛擬網格中的主要概念是，單元格不包含值，而是在外部數據結構中讀取和寫入值。這個想法是透過一個抽象類別 CellVirtual 實現的，其中必須重新定義方法 GetValue 和 SetValue. 。因此要使用 GridVirtual 就必須建立一個繼承自 CellVirtual 的類別，並使用選擇的數據來源來自定義讀取。通常最好也建立一個繼承自 GridVirtual 的控制項，以獲得更大的靈活性與更穩固的程式碼，並覆寫方法 GetCell 。如果您偏好，可以直接使用 GridVirtual 控制項和事件 GettingCell 。方法 GetCell 和事件 GettingCell 的目的是為了回傳一個給定位置（行和列）的選擇單元格。這提供了高度的靈活性，因為您可以回傳任何特定類型的 ICellVirtual ，例如，當行為 0 時，您可以回傳一個標題類型的單元格。

In the following example I create a virtual grid that reads and writes the values in an array. First I have inserted the control GridVirtual in a form, then I write this code that defines our virtual class:
在以下範例中，我建立了一個在陣列中讀取和寫入值的虛擬網格。首先我在表單中插入了控制項 GridVirtual ，然後我寫了這段程式碼來定義我們的虛擬類別：

CS

public class CellStringArray : SourceGrid2.Cells.Virtual.CellVirtual
{
  private string[,] m_Array;
  public CellStringArray(string[,] p_Array):base(typeof(string))
  {
    m_Array = p_Array;
  }
  public override object GetValue(SourceGrid2.Position p_Position)
  {
    return m_Array[p_Position.Row, p_Position.Column];
  }
  public override void SetValue(SourceGrid2.Position p_Position,
    object p_Value)
  {
    m_Array[p_Position.Row, p_Position.Column] = (string)p_Value;
    OnValueChanged(new SourceGrid2.PositionEventArgs(p_Position, this));
  }
}
With the previous code I have created a virtual cell with an editor of type string that reads and writes the values in an array specified in the constructor. After the call to the SetValue method we should call the OnValueChanged method to notify the grid to update this cell.
先前我已創建了一個虛擬單元格，其編輯器類型為 string ，可在構造函式中指定的陣列中讀寫值。在調用 SetValue 方法後，應呼叫 OnValueChanged 方法以通知網格更新此單元格。

In the event Load of the Form I have insert this code:
在 Form 的 Load 事件中，我插入這段程式碼：

CS

private void frmSample15_Load(object sender, System.EventArgs e)
{
  gridVirtual1.GettingCell += new SourceGrid2.PositionEventHandler(
    gridVirtual1_GettingCell);
  gridVirtual1.Redim(1000,1000);
  string[,] l_Array = new string[gridVirtual1.RowsCount,
    gridVirtual1.ColumnsCount];
  m_CellStringArray = new CellStringArray(l_Array);
  m_CellStringArray.BindToGrid(gridVirtual1);
}
I have added an event handler to GettingCell event, created the grid and the array with 1000 rows and 1000 columns, then I have created a new instance of the previous cell and with the method BindToGrid I have linked the cell to the grid. I have created a single cell that will be used for every position of the matrix. Is always necessary to call the method BindToGrid on the cells that we want to use in a virtual grid.
我為 GettingCell 事件添加了一個事件處理程序，創建了具有 1000 行和 1000 列的網格和陣列，然後我創建了先前單元格的新實例，並使用 BindToGrid 方法將單元格連接到網格。我創建了一個單個單元格，它將用於矩陣的每個位置。在虛擬網格中使用我們想要使用的單元格時，總是必須呼叫 BindToGrid 方法。

In order to finish we should write the method GettingCell and declare the variable for the cell:
為了完成，我們應該寫方法 GettingCell 與宣告該單元的變數：

CS

private CellStringArray m_CellStringArray;
private void gridVirtual1_GettingCell(object sender,
  SourceGrid2.PositionEventArgs e)
{
  e.Cell = m_CellStringArray;
}
The result should look equal to the one in the following picture, this example is present also in the project SampleProject included in the ZIP file.
結果應該看起來與下列圖片相同，這個範例也包含在 ZIP 檔案中的 SampleProject 專案內。

Article image

VisualModel
Namespace: SourceGrid2.VisualModels
命名空間：SourceGrid2.VisualModels

Every cell have a property VisualModel that returns an interface of type IVisualModel. The cell uses this interface to draw and to customize the visual properties of the cell.
每一個單元格都有一個屬性 VisualModel ，它回傳一個類型為 IVisualModel 的介面。單元格使用這個介面來繪製和自訂單元格的視覺屬性。

The purpose of the VisualModel is to separate the drawing code from the rest of the code and allows to sharing the same visual model between more cells. In fact the same instance of VisualModel can be used on more cells simultaneously optimizing the use of the resources of the system. The default VisualModel classes are read-only, however each VisualModel is provided with a Clone method that allows you to create identical instances of the same model.
VisualModel 的目的是為了將繪製程式碼與其他程式碼分開，並允許在更多單元格之間共用相同的視覺模型。事實上，同一個 VisualModel 的實例可以同時用於更多單元格上，優化系統資源的利用。預設的 VisualModel 類別是唯讀的，不過每個 VisualModel 都提供了一個 Clone 方法，允許您創建相同模型的相同實例。

These are the default VisualModel classes in the namespace SourceGrid2.VisualModels:
這些是命名空間 SourceGrid2.VisualModels: 中的預設 VisualModel 類別。

SourceGrid2.VisualModels.Common - Used for classic cells. In this model you can customize the colors, the font, the borders and a lot other properties.
SourceGrid2.VisualModels.Common - 用於傳統細胞。在這個模型中，您可以自訂顏色、字型、邊框和其他許多屬性。
SourceGrid2.VisualModels.CheckBox* - Used for checkbox style cells. The checkbox can be selected, disabled and can contains a caption.
SourceGrid2.VisualModels.CheckBox* - 用於核取方塊風格細胞。核取方塊可以選取、停用，並可以包含標題。
SourceGrid2.VisualModels.Header* - Used for header style cells with 3D borders. You can configure the borders to gradually vanish from the color of the border to the color of the cell for a better three-dimensional effect.
SourceGrid2.VisualModels.Header* - 用於具有 3D 邊框的標題風格細胞。您可以設定邊框從邊框顏色漸變消失到細胞顏色，以獲得更好的立體效果。
SourceGrid2.VisualModels.MultiImages - Allows to drawing more then one image in the cell.
SourceGrid2.VisualModels.MultiImages - 允許在電池中繪製多於一個圖像。
*The VisualModel marked with an asterisk require a special interface to work correctly, for example the CheckBox model needs a cell that supports the ICellCheckBox interface.
* 標註為 VisualModel 的項目需要特殊介面才能正確運作，例如 CheckBox 型號需要支援 ICellCheckBox 介面的電池。

Each of these classes contains one or more static properties with some default read-only instances easily useable:
這些類別中各包含一個或多個靜態屬性，具有一些可輕鬆使用的預設唯讀實例：

SourceGrid2.VisualModels.Common.Default
SourceGrid2.VisualModels.Common.LinkStyle
SourceGrid2.VisualModels.CheckBox.Default
SourceGrid2.VisualModels.CheckBox.MiddleLeftAlign
SourceGrid2.VisualModels.Header.Default
SourceGrid2.VisualModels.Header.ColumnHeader
SourceGrid2.VisualModels.Header.RowHeader
This code shows how to assign the same VisualModel to more cells previously created and then change some properties:
這段程式碼展示了如何將相同的 VisualModel 指派給先前建立的多個電池，然後更改一些屬性：

CS

SourceGrid2.VisualModels.Common l_SharedVisualModel =
  new SourceGrid2.VisualModels.Common();
grid1[0,0].VisualModel = l_SharedVisualModel;
grid1[1,0].VisualModel = l_SharedVisualModel;
grid1[2,0].VisualModel = l_SharedVisualModel;
l_SharedVisualModel.BackColor = Color.LightGray;
Consider also that when you write Cell.BackColor the property calls automatically the BackColor property of the VisualModel associated. To facilitate the utilization of the more common properties if you write Cell.BackColor = Color.Black; the cell automatically clone the current VisualModel , changes the backcolor to the cloned instance and assigns the cloned instance again to the cell.
考量當您撰寫 Cell.BackColor 時，屬性會自動呼叫相關的 BackColor 屬性。為了便於使用更常見的屬性，若您撰寫 Cell.BackColor = Color.Black; ，單元格會自動複製目前的 VisualModel ，將背景色變更為複製的實例，並再次將複製的實例分配給單元格。

DataModel
Namespace: SourceGrid2.DataModels

To represent the value of a cell in a string format and to supply a cell data editor, is necessary to populate the property DataModel of the cell. If this property is null is not possible to change the value of the cell and the string conversion will be a simple ToString of the value.
為了將單元格的值以字串格式表示，並提供單元格資料編輯器，必須填入單元格的屬性 DataModel 。如果此屬性為 null，則無法更改單元格的值，且字串轉換將只會是值的簡單 ToString 。

Usual a DataModel use a TypeConverter of the type asked to manage the necessary conversion, particularly the string conversion (used to represent the cell value).
通常 DataModel 使用所請求類型的 TypeConverter 來管理必要的轉換，特別是字串轉換（用於表示單元格值）。

These are the default DataModel classes in the namespace SourceGrid2.DataModels:
這些是命名空間 SourceGrid2.DataModels: 的預設 DataModel 類別。

DataModelBase - Supplies the methods of conversion and allows the alteration of the cell value only by code, does not supply graphic interface. This class is the base of all the other editors and is used also to manage read-only cells but with customized formattings or special editors (for example the CheckBox cell use a read-only editor because the value is changed clicking directly on the checkbox).
DataModelBase - 提供轉換方法，並僅允許透過程式碼修改單元格值，不提供圖形介面。此類別是所有其他編輯器的基礎，也用於管理唯讀單元格，但可使用自訂格式或特殊編輯器（例如，CheckBox 單元格使用唯讀編輯器，因為值是透過直接點擊核取方塊來更改）。
EditorControlBase - Abstract Class that help to use a control as editor for the cell.
EditorControlBase - 一個抽象類別，幫助將控制項用作單元格的編輯器。
EditorTextBox - A TextBox editor. This is one of the more used editor by all types that support string conversion (string, int, double, enum,....)
EditorTextBox - 一個 TextBox 編輯器。這是所有支援字串轉換類型中較常用的一種編輯器 (string, int, double, enum,....)
EditorComboBox - A ComboBox editor.
EditorComboBox - 一個 ComboBox 編輯器。
EditorDateTime - A DateTimePicker editor.
EditorDateTime - 一個 DateTimePicker 編輯器。
EditorNumericUpDown - A NumericUpDown editor.
EditorNumericUpDown - 一個 NumericUpDown 編輯器。
EditorTextBoxButton - A TextBox editor with a button to open a details mask.
EditorTextBoxButton - 一個具有開啟細節遮罩按鈕的 TextBox 編輯器。
EditorUITypeEditor - Supplies the editing of the cell of all types that have an UITypeEditor. A lot of types support this interface: DateTime, Font, a lot of enum, ...
EditorUITypeEditor - 提供對所有具有 UITypeEditor 的單元格進行編輯。許多類型支援這個介面：DateTime、Font、許多 enum、...
A DataModel can be shared between more cells, for example you can use the same DataModel for every cell of a column.
一個 DataModel 可以在多個單元格之間共用，例如你可以為某列的每一個單元格使用相同的 DataModel 。

To create an editable cell there are 2 possibilities:
要建立一個可編輯的單元格，有兩種可能性：

Create the cell specifying the value type. In this manner the cell calls automatically an utility function, Utility.CreateDataModel, that returns a DataModel in base to the type specified.
建立單元格時指定值類型。這樣，單元格會自動呼叫一個工具函數 Utility.CreateDataModel ，根據指定的類型回傳一個 DataModel 。
CS

grid1[0,0] = new SourceGrid2.Cells.Real.Cell("Hello",
  typeof(string));
Create separately the DataModel and then assign it to the cells:
建立獨立地 DataModel ，然後將其指派給單元格：
CS

SourceGrid2.DataModels.IDataModel l_SharedDataModel =
   SourceGrid2.Utility.CreateDataModel(typeof(string));
grid1[0,0].DataModel = l_SharedDataModel;
grid1[1,0].DataModel = l_SharedDataModel;
This method is recommended when you want to use the same editor for more of cells.
當您想要使用相同的編輯器來編輯更多單元格時，建議使用此方法。
If you need a greater control on the type of editor or there are special requirements is possible to create manually the editor class. In this case for example I create manually the class EditorTextBox and then I call the property MaxLength and CharacterCasing.
如果您需要對編輯器類型有更大的控制權，或者有特殊需求，可以手動建立編輯器類別。在這種情況下，例如我手動建立類別 EditorTextBox ，然後我調用屬性 MaxLength 和 CharacterCasing。

CS

SourceGrid2.DataModels.EditorTextBox l_TextBox =
   new SourceGrid2.DataModels.EditorTextBox(typeof(string));
l_TextBox.MaxLength = 20;
l_TextBox.AttachEditorControl(grid1);
l_TextBox.GetEditorTextBox(grid1).CharacterCasing =
  CharacterCasing.Upper;
grid1[2,0].DataModel = l_TextBox;
Some properties are defined to a DataModel level, while other to an editor control level, in this case the property CharacterCasing is defined to a TextBox control level. To use these properties is necessary therefore to force a linking of the editor to the grid with the method AttachEditorControl and then call the method GetEditorTextBox to returns the instance of the TextBox. This mechanism is also useful for create special editor like the ComboBox editor. To insert a ComboBox you must write this code:
一些屬性定義在 DataModel 層級，而其他屬性定義在編輯器控制層級，在這種情況下，屬性 CharacterCasing 定義在 TextBox 控制層級。因此要使用這些屬性，必須使用方法 AttachEditorControl 強制編輯器鏈接到網格，然後調用方法 GetEditorTextBox 返回 TextBox 的實例。這種機制也很有用於創建特殊的編輯器，例如 ComboBox 編輯器。要插入 ComboBox，您必須編寫這段代碼：

CS

SourceGrid2.DataModels.EditorComboBox l_ComboBox =
    new SourceGrid2.DataModels.EditorComboBox(
                typeof(string),
                new string[]{"Hello", "Ciao"},
                false);
grid1[3,0].DataModel = l_ComboBox;
Of course is possible to create custom DataModel editor with custom control or special behaviors. In the following picture it is possible to observe most of the editors available and some options like image properties:
當然可以建立自訂 DataModel 編輯器，搭配自訂控制項或特殊行為。在下圖中可以觀察到大部分可用的編輯器以及一些選項，例如圖片屬性：

Article image

BehaviorModel
Namespace: SourceGrid2.BehaviorModels
命名空間：SourceGrid2.BehaviorModels

Every cell have a collection of BehaviorModel that you can read with the Behaviors property. A BehaviorModel is a class that characterizes the behavior of the cell. A model can be shared between more cells and allows a great flexibility and simplicity of any new feature.
每一個單元格都有一個 BehaviorModel 的集合，您可以使用 Behaviors 屬性來讀取它。一個 BehaviorModel 是一個類別，它描述了單元格的行為。一個模型可以在多個單元格之間共享，並允許任何新功能的極大靈活性和簡便性。

These are the default classes of type BehaviorModel:
這些是類型 BehaviorModel 的預設類別：

SourceGrid2.BehaviorModels.Common - Common behavior of a cell.
SourceGrid2.BehaviorModels.Common - 單元格的通用行為。
SourceGrid2.BehaviorModels.Header - Behavior of a header.
SourceGrid2.BehaviorModels.Header - 標題的行為。
SourceGrid2.BehaviorModels.RowHeader - Behavior of a row header, with resize feature.
SourceGrid2.BehaviorModels.RowHeader - 行標題的行為，具有調整大小功能。
SourceGrid2.BehaviorModels.ColumnHeader* - Behavior of a column header, with sort and resize feature. (need ICellSortableHeader)
SourceGrid2.BehaviorModels.ColumnHeader * - 列標題的行為，具有排序和調整大小功能。(需要 ICellSortableHeader )
SourceGrid2.BehaviorModels.CheckBox* - Behavior of a CheckBox. (need ICellCheckBox)
SourceGrid2.BehaviorModels.CheckBox * - Check Box 的行為。(需要 ICellCheckBox )
SourceGrid2.BehaviorModels.Cursor* - Allows to link a cursor to a specific cell. (need ICellCursor)
SourceGrid2.BehaviorModels.Cursor * - 允許將游標連接到特定單元格。(需要 ICellCursor )
SourceGrid2.BehaviorModels.Button - Behavior of a Button.
SourceGrid2.BehaviorModels.Button - 按鈕的行為。
SourceGrid2.BehaviorModels.Resize - Allows a cell to be resized with the mouse (this model is automatically used by header models).
SourceGrid2.BehaviorModels.Resize - 允許使用滑鼠調整單元格大小（此模式會自動由標題模式使用）。
SourceGrid2.BehaviorModels.ToolTipText* - Allows to show a ToolTipText linked to a cell. (need ICellToolTipText)
SourceGrid2.BehaviorModels.ToolTipText * - 允許顯示與單元格連結的工具提示文字。（需要 ICellToolTipText ）
SourceGrid2.BehaviorModels.Unselectable - Blocks a cell to receive the focus.
SourceGrid2.BehaviorModels.Unselectable - 阻止單元格接收焦點。
SourceGrid2.BehaviorModels.ContextMenu* - Allows to show a contextmenu linked to a cell. (need a ICellContextMenu)
SourceGrid2.BehaviorModels.ContextMenu * - 允許顯示與單元格相關的右鍵菜單。(需要一個 ICellContextMenu )
SourceGrid2.BehaviorModels.CustomEvents - Expose a list of events that you can use without deriving from a BehaviorModel.
SourceGrid2.BehaviorModels.CustomEvents - 提供一組您可以無需繼承自 BehaviorModel 即可使用的事件清單。
SourceGrid2.BehaviorModels.BindProperty - Allows to link the value of a cell to an external property.
SourceGrid2.BehaviorModels.BindProperty - 允許將單元格的值鏈接到外部屬性。
SourceGrid2.BehaviorModels.BehaviorModelGroup - Allows to create a BehaviorModel that automatically calls a list of BehaviorModel, useful when a behavior needs other behaviors to work correctly.
SourceGrid2.BehaviorModels.BehaviorModelGroup - 允許創建一個 BehaviorModel ，它會自動調用一組 BehaviorModel ，當一種行為需要其他行為正確運作時非常有用。
*The BehaviorModel marked with an asterisk need special cells to complete their tasks, for example the class CheckBox requires of a cell that supports the interface ICellCheckBox.
* 標註為 BehaviorModel 的星號需要特殊格體才能完成其任務，例如類別 CheckBox 需要支援介面 ICellCheckBox 的格體。

Every class have some static properties that return a default instance of the class:
每個類別都有一些靜態屬性，它們會回傳類別的預設實例：

SourceGrid2.BehaviorModels.Common.Default
SourceGrid2.BehaviorModels.Button.Default
SourceGrid2.BehaviorModels.CheckBox.Default
SourceGrid2.BehaviorModels.ColumnHeader.SortableHeader
SourceGrid2.BehaviorModels.ColumnHeader.NotSortableHeader
SourceGrid2.BehaviorModels.Cursor.Default
SourceGrid2.BehaviorModels.Header.Default
SourceGrid2.BehaviorModels.Resize.ResizeHeight
SourceGrid2.BehaviorModels.Resize.ResizeWidth
SourceGrid2.BehaviorModels.Resize.ResizeBoth
SourceGrid2.BehaviorModels.RowHeader.Default
SourceGrid2.BehaviorModels.ToolTipText.Default
SourceGrid2.BehaviorModels.Unselectable.Default
In the following code example I create a BehaviorModel that change the backcolor of the cell when the user moves the mouse over the cell.
在以下的程式碼範例中，我建立了一個 BehaviorModel ，當使用者將滑鼠移動到格體上時，會改變格體的背景顏色。

CS

public class CustomBehavior : SourceGrid2.BehaviorModels.BehaviorModelGroup
{
  public override void OnMouseEnter(SourceGrid2.PositionEventArgs e)
  {
    base.OnMouseEnter (e);
    ((SourceGrid2.Cells.Real.Cell)e.Cell).BackColor = Color.LightGreen;
  }
  public override void OnMouseLeave(SourceGrid2.PositionEventArgs e)
  {
    base.OnMouseLeave (e);
    ((SourceGrid2.Cells.Real.Cell)e.Cell).BackColor = Color.White;
  }
}
To use this BehaviorModel insert in Load event of a form this code:
若要使用此 BehaviorModel ，請將此程式碼插入表單的 Load 事件中：

CS

grid1.Redim(2,2);

CustomBehavior l_Behavior = new CustomBehavior();
for (int r = 0; r < grid1.RowsCount; r++)
  for (int c = 0; c < grid1.ColumnsCount; c++)
  {
    grid1[r,c] = new SourceGrid2.Cells.Real.Cell("Hello");
    grid1[r,c].Behaviors.Add(l_Behavior);
  }
Cells 格子
Namespace: SourceGrid2.Cells
命名空間：SourceGrid2.Cells

These are the default cells available:
這些是可用的預設格子：

SourceGrid2.Cells.Virtual - This namespace contains all the virtual cells that can be used with a GridVirtual control, these are all abstract cells and you must derive from these cells to use your custom data source.
SourceGrid2.Cells.Virtual - 這個命名空間包含所有可與 GridVirtual 控制元件一起使用的虛擬格子，這些都是抽象格子，您必須從這些格子繼承才能使用自訂資料來源。
CellVirtual - Base cell for each other implementation, use for the most common type of virtual cells.
CellVirtual - 每個其他實現的基礎單元，用於最常見的虛擬單元類型。
Header - A header cell.
Header - 一個標題單元。
ColumnHeader - A column header cell.
ColumnHeader - 一個欄標題單元。
RowHeader - A row header cell.
RowHeader - 一個列標題單元。
Button - A button cell.
Button - 一個按鈕細胞。
CheckBox - A checkbox cell.
CheckBox - 一個核取方塊細胞。
ComboBox - A combobox cell.
ComboBox - 一個下拉式選單細胞。
Link - A link style cell.
Link - 一個連結樣式細胞。
SourceGrid2.Cells.Real - This namespace contains all the real cells that can be used with a Grid control.
SourceGrid2.Cells.Real - 此命名空間包含所有可與 Grid 控制元件搭配使用的實際單元格。
Cell - Base cell for each other implementation, use for the most common type of real cells.
Cell - 其他實現的基礎單元格，用於最常見的實際單元格類型。
Header - A header cell.
Header - 一個標題單元格。
ColumnHeader - A column header cell.
ColumnHeader - 一個列標題單元格。
RowHeader - A row header cell.
RowHeader - 一行標頭單元格。
Button - A button cell.
Button - 一個按鈕單元格。
CheckBox - A checkbox cell.
CheckBox - 一個核取方塊單元格。
ComboBox - A combobox cell.
ComboBox - 一個下拉式選單單元格。
Link - A link style cell.
Link - 一個鏈接樣式單元格。
The goal of these classes is to simplify the use of VisualModel, DataModel and BehaviorModel. If we look at the code of any of these classes we can see that these classes use the previous models according to the role of the cell. There are however models that require special interfaces and in this case the cells implement all the required interfaces. This is for example the code of the cell SourceGrid2.Cells.Real.CheckBox:
這些類的目標是簡化 VisualModel, DataModel and BehaviorModel 的使用。如果我們查看其中任何一個類的代碼，我們可以看到這些類根據單元格的角色的使用先前模型。然而，有一些模型需要特殊的接口，在這種情況下，單元格實現了所有需要的接口。這是單元格 SourceGrid2.Cells.Real.CheckBox 的代碼，例如：

CS

public class CheckBox : Cell, ICellCheckBox
{
  public CheckBox(string p_Caption, bool p_InitialValue)
  {
    m_Caption = p_Caption;

    DataModel = new SourceGrid2.DataModels.DataModelBase(typeof(bool));
    VisualModel = SourceGrid2.VisualModels.CheckBox.MiddleLeftAlign;
    Behaviors.Add(BehaviorModels.CheckBox.Default);
    
    Value = p_InitialValue;
  }
  public bool Checked
  {
    get{return GetCheckedValue(Range.Start);}
    set{SetCheckedValue(Range.Start, value);}
  }
  private string m_Caption;
  public string Caption
  {
    get{return m_Caption;}
    set{m_Caption = value;}
  }
  public virtual bool GetCheckedValue(Position p_Position)
  {
    return (bool)GetValue(p_Position);
  }
  public virtual void SetCheckedValue(
    Position p_Position, bool p_bChecked)
  {
    if (DataModel!=null && DataModel.EnableEdit)
      DataModel.SetCellValue(this, p_Position, p_bChecked);
  }
  public virtual CheckBoxStatus GetCheckBoxStatus(Position p_Position)
  {
    return new CheckBoxStatus(DataModel.EnableEdit,
    GetCheckedValue(p_Position), m_Caption);
  }
}
As you can see the CheckBox class simply use the models SourceGrid2.DataModels.DataModelBase(typeof(bool)), SourceGrid2.VisualModels.CheckBox.MiddleLeftAlign e BehaviorModels.CheckBox.Default implements the ICellCheckBox interface with its method GetCheckBoxStatus. The methods Checked, Caption, GetCheckedValue and SetCheckedValue are methods to simplify the editing of the value of the cell.
如您所見， CheckBox 類別僅使用模型 SourceGrid2.DataModels.DataModelBase(typeof(bool)), SourceGrid2.VisualModels.CheckBox.MiddleLeftAlign e BehaviorModels.CheckBox.Default ，實作 ICellCheckBox 接口，並透過其方法 GetCheckBoxStatus 。方法 Checked, Caption, GetCheckedValue 和 SetCheckedValue 則是用於簡化單元格值編輯的方法。

Structure of the Grid
網格結構
Rows and Columns 行與列
The main components of a grid are the rows and the columns. To manipulate these informations SourceGrid supplies 2 properties:
網格的主要元件是行和列。為了操作這些資訊，SourceGrid 提供了 2 個屬性：

Rows - Returns a collection of type RowInfoCollection that is a strip of classes RowInfo.
Rows - 傳回一個類型為 RowInfoCollection 的集合，它是一個 RowInfo 的條帶。
Columns - Returns a collection of type ColumnInfoCollection that is a list of classes ColumnInfo.
Columns - 回傳一個型別為 ColumnInfoCollection 的集合，該集合為類別 ColumnInfo 的列表。
These are some of the RowInfo class properties: Height, Top, Bottom, Index, Tag. These are instead the properties of the ColumnInfo class:Width, Left, Right, Index, Tag.
這些是 RowInfo 類別的屬性： Height, Top, Bottom, Index, Tag 。這些反而是 ColumnInfo 類別的屬性： Width, Left, Right, Index, Tag 。

There are many ways to manipulate rows and columns:
操作行和列的方法有許多種：

CS

grid1.Redim(2,2);
CS

grid1.RowsCount = 2;
grid1.ColumnsCount = 2;
CS

grid1.Rows.Insert(0);
grid1.Rows.Insert(1);
grid1.Columns.Insert(0);
grid1.Columns.Insert(1);
These three examples perform all the same task of creating a table with 2 rows and 2 columns.
這三個範例都執行相同的任務，即建立一個具有 2 行和 2 列的表格。

To change the width or the height of a row or a column you can use this code:
要更改行或列的寬度或高度，您可以使用這段程式碼：

CS

grid1.Rows[0].Height = 100;
grid1.Columns[0].Width = 100;
The properties Top, Bottom, Left and Right are automatically calculated using the width and the height of the rows and columns.
屬性 Top, Bottom, Left and Right 是根據行和列的寬度和高度自動計算的。

Panels 面板
To manage correctly scrollbars, columns and rows fixed and a lot other details, the grid inside has a panels structure like this:
要正確管理滾動條、固定列和行以及許多其他細節，內部的網格具有如下的面板結構：

Article image

NaN. TopLeftPanel - Keeps fixed row and fixed column cells.
NaN. TopLeftPanel - 保留固定行和固定列的单元格。
NaN. TopPanel - Keeps fixed rows.
NaN. TopPanel - 保留固定行。
NaN. LeftPanel - Keeps fixed columns.
NaN. LeftPanel - 保持固定欄位。
NaN. ScrollablePanel - Keeps all not fixed cells.
NaN. ScrollablePanel - 保持所有非固定單元格。
NaN. HScrollBar - Horizontal ScrollBar
NaN. HScrollBar - 水平捲軸
NaN. VScrollBar - Vertical ScrollBar.
NaN. VScrollBar - 垂直捲軸。
NaN. BottomRightPanel - Panel to manage the small space between the two scrollbars.
NaN. BottomRightPanel - 用於管理兩個捲軸之間小空間的面板。
Events 事件
The mouse and keyboard events can be used with a BehaviorModel or can be connected directly to the grid. All the events are first fired to the panels and then automatically moved to GridVirtual and Grid control. To use these events you can write this code:
滑鼠和鍵盤事件可以與 BehaviorModel 使用，或可直接連接到網格。所有的事件首先觸發到面板，然後自動移動到 GridVirtual 和 Grid 控制元件。要使用這些事件，您可以編寫這段程式碼：

CS

grid.MouseDown += new System.Windows.Forms.MouseEventHandler(
  grid_MouseDown);
This can be done also with the Visual Studio designer. Look at the example 8 in the project SampleProject for details.
這也可以在 Visual Studio 設計器中完成。查看 SampleProject 專案中的範例 8 以了解詳情。

ContextMenu 右鍵菜單
The grid has a default ContextMenu that can be customized with the ContextMenuStyle property. It is possible to connect a ContextMenu to the Selection object with the Grid.Selection.ContextMenuItems, that will be used for all selected cells or otherwise you can connect a ContextMenu directly to a specific cell. Look at the example 10 in the project SampleProject for further details.
網格有一個預設的右鍵菜單，可以使用 ContextMenuStyle 屬性進行自訂。可以透過 Grid.Selection.ContextMenuItems 將右鍵菜單連接到 Selection 物件，這將用於所有選擇的單元格；或者您可以將右鍵菜單直接連接到特定單元格。查看 SampleProject 專案中的範例 10 以獲得更詳細的資訊。

Other Informations 其他資訊
Focus and Selection 焦點與選擇
A cell can be selected of can have the focus. Only one cell can have the focus, identified by the FocusCellPosition property of the grid, instead many cells can be selected. A cell is selected when is present in the Selection object of the grid. The cell with the focus receives all of the mouse and keyboard events, while the selected cells can receive actions like the copy/paste.
一個單元格可以被選擇或擁有焦點。只有一個單元格可以擁有焦點，由網格的 FocusCellPosition 屬性識別，而不是許多單元格可以被選擇。當單元格存在於網格的 Selection 物件中時，該單元格被選擇。擁有焦點的單元格接收所有鼠標和鍵盤事件，而選擇的單元格可以接收複製/貼上等操作。

Position and Range 位置與範圍
Two of the most used objects in the project SourceGrid are the struct Position and Range. The struct Position identifies a position with a Row and a Column, while the struct Range identifies a group of cells delimited from a start Position and an end Position.
在 SourceGrid 專案中最常用的兩個物件是結構 Position 和 Range 。結構 Position 識別一個位置，具有行和列，而結構 Range 識別一組由起始 Position 和結束 Position 界定的單元格。

Performance 效能
To optimize performance of this control use the GridVirtual control when is necessary to visualize a lot of cells and try always to share the models (DataModel, VisualModel, BehaviorModel) between more possible cells. The performance of the grid is quite good even if the drawing code can be still optimized, especially when scrolling. It is possible to consult the project SampleProject for further information on the performance of the grid.
為了優化此控制的效能，當需要顯示大量單元格時請使用 GridVirtual 控制元件，並盡量在更多可能的單元格之間共享 (DataModel, VisualModel, BehaviorModel) 模型。即使繪圖程式碼仍可優化，網格的效能仍然相當不錯，尤其是在捲動時。您可以參考 SampleProject 專案以獲取更多有關網格效能的資訊。

Extensions 擴充功能
In the project SampleProject are present a lot of examples and parts of code that can give ideas or suggestions of how implements custom grid, particularly in the folder Extensions are present some grids that supply functionality like the binding to a DataSet (DataTable), to an Array and to an ArrayList.
在 SampleProject 專案中有許多範例和程式碼片段，可以提供實現自訂網格的靈感或建議，特別是在 Extensions 文件夾中，有一些網格提供類似於與 DataSet (DataTable) 綁定、與 Array 綁定以及與 ArrayList. 綁定的功能。

Screenshots 截圖
Article image

Article image

Article image

How To 如何
How to select an entire row:
如何選擇整行：

CS

grid1.Rows[1].Select = true;
How to select all the cells:
如何選擇所有單元格：

CS

grid1.Selection.AddRange(grid1.CompleteRange);
How to create an editor with advanced validation rule:
如何建立具有進階驗證規則的編輯器：

CS

grid1[0,0] = new SourceGrid2.Cells.Real.Cell(2, typeof(int));
grid1[0,0].DataModel.MinimumValue = 2;
grid1[0,0].DataModel.MaximumValue = 8;
grid1[0,0].DataModel.DefaultValue = null;
grid1[0,0].DataModel.AllowNull = true;
How to create a ComboBox editor to display a value different from the real used value, in this case is displayed a string while the real value is a double.
如何建立一個 ComboBox 編輯器來顯示與實際使用值不同的值，在此情況下顯示的是字串，而實際值是 double。

CS

double[] l_RealValues = new double[]{0.0,0.5,1.0};
SourceGrid2.DataModels.EditorComboBox l_EditorCombo =
   new SourceGrid2.DataModels.EditorComboBox(typeof(double));
l_EditorCombo.StandardValues = l_RealValues;
l_EditorCombo.StandardValuesExclusive = true;
l_EditorCombo.AllowStringConversion = false;
SourceLibrary.ComponentModel.Validator.ValueMapping l_Mapping =
   new SourceLibrary.ComponentModel.Validator.ValueMapping();
l_Mapping.ValueList = l_RealValues;
l_Mapping.DisplayStringList = new string[]{"Zero", "One Half", "One"};
l_Mapping.BindValidator(l_EditorCombo);
grid1[0,0] = new SourceGrid2.Cells.Real.Cell(0.5, l_EditorCombo);
Features 功能
What SourceGrid can do:
SourceGrid 可以做到什麼：

It is possible to customize the graphic appearance, the type of editor and the behavior (cursor, tooltiptext, contextmenu ...,) of every cell.
可以自訂圖形外觀、編輯器類型以及每個單元格的行為（游標、工具提示文字、右鍵菜單等等）。
Supports natively all of the types of data that have a TypeConverter or an UITypeEditor associated.
原生支援所有有 TypeConverter 或 UITypeEditor 相關的數據類型。
Any .NET control can be used like editor with few lines of code.
任何 .NET 控制項都可以用幾行程式碼作為編輯器使用。
You can insert, delete and move rows and columns.
您可以插入、刪除和移動行和列。
The height and the width can be customized independently for every columns and rows or can be calculated automatic based to the content of the cells.
每個行和列的高度和寬度可以獨立自訂，或可根據單元格內容自動計算。
Supports features of RowSpan and ColumnSpan, to unite more cells.
支援 RowSpan 和 ColumnSpan 的功能，以合併更多單元格。
Supports automatic operations of Copy and Paste.
支援複製和貼上自動操作。
Supports natively column sort.
支援原生欄位排序。
You can change the width and the height of the columns and rows.
您可以更改欄位和列的高度和寬度。
In every cell is possible to customize the image and the alignment of the text and the image.
在每一個單元格中都可以自訂圖片和文字與圖片的對齊方式。
Supports MultiLine and WordWrap text.
支援多行與文字換行。
Supports an HTML export.
支援 HTML 導出。
With some extension supports data binding features.
透過某些擴充功能支援資料綁定功能。
Support virtual cells used to binding any type of data source.
支援虛擬格子，用於綁定任何類型的資料來源。
... and what cannot do
...以及什麼無法做到

SourceGrid doesn't have a designer, all should be done with code.
SourceGrid 沒有設計器，所有內容都必須透過程式碼完成。
No printing support. 沒有打印支援。
Change SourceGrid code 修改 SourceGrid 程式碼
It is allowed to change, recompile and to distribute the control SourceGrid for private and commercial use, I ask only to maintain the Copyright notes at the end of the page. I recommend to change the file SourceGrid2.snk with a personalized version to not have problems of compatibility with different versions of the control. Consult MSDN for further information: http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cptutorials/html/_4__A_Shared_Component.asp
可以修改、重新編譯並分發 SourceGrid 控制項用於私人及商業用途，我只要求在頁面結尾維持版權聲明。我建議將 SourceGrid2.snk 檔案更換為個人化版本，以避免與不同版本的控制項產生兼容性問題。請參閱 MSDN 以獲取更多資訊：http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cptutorials/html/_4__A_Shared_Component.asp

Future developments 未來發展
Enhancement of the drawing code.
繪圖程式的增強。
Support for Masked Edit textbox.
支援 Masked Edit 文本方塊。
Known problems 已知問題
There is not Cut support.
沒有剪切支援。
The editor NumericUpDown is not aligned correctly to the cell.
編輯器 NumericUpDown 沒有正確對齊到單元格。
The Shift key does not work with the arrows keys and there are still some problems with header cells.
Shift 鍵與方向鍵無法正常使用，且表頭單元格仍然存在一些問題。
Previous versions 舊版本
Version 2 of SourceGrid introduce many changes, and is not possible to list the all. The manner of utilization is very similar, but is not simple to convert a code written with previous versions. These are some suggestions:
版本 2 的 SourceGrid 引入了許多變更，不可能一一列舉。使用方式非常相似，但將先前版本編寫的代碼轉換過來並不簡單。這是一些建議：

The major features of the grid work with the ICellVirtual interface, no more with Cell class. This interface contains only necessary methods and therefore is poorer. For this the code that before was:
網格的主要功能與 ICellVirtual 接口一起使用，不再與 Cell 類別一起使用。這個接口僅包含必要的 方法，因此相對較弱。因此，先前編寫的代碼為：
CS

grid[0,0] = new SourceGrid.Cell("Ciao");
grid[0,0].BackColor = Color.White;
now should be transformed in
現在應該轉換為：
CS

SourceGrid2.Cells.Real.Cell l_Cell =
  new SourceGrid2.Cells.Real.Cell("Ciao");
l_Cell.BackColor = Color.White;
grid[0,0] = l_Cell;
In the prior version the base cell was identified from the Cell class while now is the interface ICellVirtual and the major change of this class is that does not contain informations about the position (row and column).
在先前版本中，基礎單元從 Cell 類別識別，而現在則是接口 ICellVirtual ，這個類別的主要變更是它不包含關於位置（行和列）的信息。
A lot of the methods of the grid that before used the Cell type now use the Position type, is however possible to extract the interface ICellVirtual (and then cast to more specific interfaces) given a struct Position with the method Grid.GetCell
許多先前使用 Cell 類型的網格方法現在使用 Position 類型，不過可以根據具有方法 Grid.GetCell 的結構 Position 提取介面 ICellVirtual （然後轉換為更特定的介面）。
Now the grid support natively the property BorderStyle that is able therefore to eliminate the eventual Panel that before was necessary to introduce a border.
現在網格原生支援屬性 BorderStyle ，因此可以移除先前為了引入邊框而必要的 Panel。
All of the code that first was bound to the events of a cell now must be moved in a BehaviorModel, you can use for example the SourceGrid2.BehaviorModels.CustomEvents.
所有最初綁定到單元事件碼的程式碼現在必須移至 BehaviorModel 中，例如可以使用 SourceGrid2.BehaviorModels.CustomEvents 。
The Selection object is no more a collection of cells but a collection of Range.
Selection 物件不再是一組單元，而是一組 Range 。
With the insertion of the Rows and Columns objects the code that first should work on lines and columns now results simpler, besides a lot of methods that before were in the grid now are in the RowInfoCollection, RowInfo, ColumnInfoCollection or ColumnInfo classes.
在插入 Rows 和 Columns 物件後，原本應該先處理行和列的程式碼現在變得簡單許多，而且原本在網格中的許多方法現在則在 RowInfoCollection 、 RowInfo 、 ColumnInfoCollection 或 ColumnInfo 類別中。
The object CellsContainer is no more present, and even if logically was replaced from the panels, the more commons are linked directly to the grid and therefore the code that before used CellsContainer now could use directly the Grid control.
物件 CellsContainer 已經不再存在，雖然在邏輯上被面板取代了，但更常見的是直接連接到網格，因此原本使用 CellsContainer 的程式碼現在可以直接使用 Grid 控制元件。
The old object ICellModel now is the object IDataModel, while the object VisualProperties now became IVisualModel.
舊物件 ICellModel 現在變成了物件 IDataModel ，而物件 VisualProperties 現在變成了 IVisualModel 。
The class CellControl for now is no more supported, I will think in future if introduce it again.
CellControl 類別現在已不再支援，我會在未來考慮是否要再次引入它。
History 歷史
2.0.3.0 (25 March 2004)
Moved and reviewed to the SourceLibrary project controls ComboBoxEx and TextBoxButton.
移動並檢查至 SourceLibrary 專案控制 ComboBoxEx 和 TextBoxButton。
2.0.2.1 (24 March 2004)
Fixed a bug on Range class. When adding or removing rows ColumnSpan and RowsSpan informations were not preserved.
修正了 Range 類別的錯誤。當添加或刪除行時，ColumnSpan 和 RowsSpan 的資訊沒有被保留。
2.0.2.0 (23 March 2004)
Fixed a bug on ColumnHeader sort when used without FixedRows
修正了在沒有使用 FixedRows 時，.ColumnHeader 排序的錯誤。
2.0.1.0 (23 March 2004)
Divided interface IDataModel and partially moved to SourceLibrary.ComponentModel.Validator.
分離介面 IDataModel，部分移至 SourceLibrary.ComponentModel.Validator。
Renamed methods StringToObject to StringToValue and ObjectToString to ValueToString.
將方法名稱從 StringToObject 改為 StringToValue，從 ObjectToString 改為 ValueToString。
Now to prevent editing the textbox of a ComboBox editor now you must use property AllowStringConversion = false. StandardValuesExclusive property allows to insert only the values present in the StandardValueList.
現在為了防止編輯 ComboBox 編輯器的文本方塊，您必須使用屬性 AllowStringConversion = false。StandardValuesExclusive 屬性允許僅插入 StandardValueList 中存在的值。
Renamed method SupportStringConversion to IsStringConversionSupported().
將方法 SupportStringConversion 改為 IsStringConversionSupported()。
Removed methods ExportValue and ImportValue.
已移除方法 ExportValue 和 ImportValue。
EditorTextBox, EditorTextBoxButton e EditorComboBox now use TextBoxTyped control as textbox.
EditorTextBox、EditorTextBoxButton 和 EditorComboBox 現在使用 TextBoxTyped 控制元件作為文本方塊。
Added editor EditorTextBoxNumeric for input char validation for numeric data type.
新增了 EditorTextBoxNumeric 編輯器，用於輸入數值類型數據的字符驗證。
AutoSize method for Header cell now add some extra space for sort icon.
Header cell 的 AutoSize 方法現在為排序圖示增加了額外的空間。
Added properties Grid.Columns[0].AutoSizeMode and Grid.Rows[0].AutoSizeMode for prevent autosize and stretch for specific columns/rows.
新增了 Grid.Columns[0].AutoSizeMode 和 Grid.Rows[0].AutoSizeMode 屬性，用於防止特定列/行的自動大小調整和拉伸。
Added properties Grid.Selection.SelectedRows and Grid.Selection.SelectedColumns that returns an array of selected rows or columns.
新增了 Grid.Selection.SelectedRows 和 Grid.Selection.SelectedColumns 屬性，它們回傳一個陣列，包含選中的行或列。
Added methods OnEditStarting and OnEditEnded to the cell and to BehaviorModel, called when editing start and end.
為單元格和 BehaviorModel 新增了 OnEditStarting 和 OnEditEnded 方法，當編輯開始和結束時會被呼叫。
Renamed methods IDataModel.StartEdit to InternalStartEdit and EndEdit to InternalEndEdit because these are internal methods only, to start or end editing call Cell.StartEdit / Cell.EndEdit.
將方法 IDataModel.StartEdit 改名為 InternalStartEdit，EndEdit 改名為 InternalEndEdit，因為這些是內部方法，僅供開始或結束編輯使用，要開始或結束編輯請呼叫 Cell.StartEdit / Cell.EndEdit。
Renamed method DataModel.GetDisplayString to DataModel.ValueToDisplayString.
將方法 DataModel.GetDisplayString 改為 DataModel.ValueToDisplayString。
Added many examples for cell type and editors (See Sample 3).
為單元類型和編輯器添加了許多範例（請參考範例 3）。
Fixed a bug in AutoSize when called with no rows or columns.
修正了在沒有行或列的情況下調用 AutoSize 時的錯誤。
Fixed a bug in SetFocusCell that don't put the focus in the grid.
修正了 SetFocusCell 的錯誤，該錯誤不會將焦點放在網格中。
Fixed a bug for MouseDown when in editing mode.
修正了編輯模式下的 MouseDown 錯誤。
2.0.0.0 (15 March 2004)
2.0.0.0 (2004 年 3 月 15 日)
First release of 2.0 version
2.0 版本首次釋出
