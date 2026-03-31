---
aliases:
date: 2008-01-18
author: Cheryl Simmons
language: C#
sourceurl: https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734(v=msdn.10)
tags:
  - CSharp
---

# Behind the Scenes: Improvements to Windows Forms Data Binding in the .NET Framework 2.0, Part 1

幕後：改善 .NET Framework 2.0 中 Windows Forms 資料綁定，第一部分

- 01/18/2008

Cheryl Simmons  雪莉·史密斯
Microsoft Corporation

August 2006  2006 年 8 月

Applies to:  適用於：
   Microsoft Visual Studio 2005
   Microsoft .NET Framework 2.0
   Microsoft Windows Forms

`Summary:` Learn about additions to the `Binding` and `ControlBindingsCollection` classes that enable easier formatting of bound data. (9 printed pages)
摘要：了解 Binding 和 ControlBindingsCollection 類別的增強功能，可讓您更容易地格式化綁定資料。(9 頁印刷內容)

Download code samples in C# and Visual Basic (23 KB) at the [Microsoft Download Center](https://go.microsoft.com/fwlink/?linkid=70370&clcid=0x409).
從 Microsoft 下載中心下載 C# 和 Visual Basic 的程式碼範例 (23 KB)。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#contents)

#### Contents  內容

Introduction  引言
Formatting and Advanced Binding Dialog Box
格式化與進階綁定對話方塊
Automatic Formatting Using Code
使用程式碼自動格式化
Using DataSourceUpdateMode
使用 DataSourceUpdateMode
Automatic Handling of Null and DBNull Values
自動處理 Null 和 DBNull 值
Improved Handling of Errors and Exceptions
改善錯誤和例外處理
Conclusion  結論

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#introduction)

## Introduction  介紹

The most important addition to Windows Forms data binding is probably the `BindingSource` component. The `BindingSource` component simplifies the developer's job by providing currency management, change notification and the ability to easily access the members in a bound list. There are, however, some other lesser-known improvements to the data binding story worth discussing, and in fact, are important additions that complement the functionality offered by the `BindingSource` component.
Windows Forms 資料綁定最重要的新增功能，大概就是 BindingSource 元件了。BindingSource 元件透過提供貨幣管理、變更通知以及輕鬆存取綁定清單內成員的方式，簡化了開發者的工作。然而，資料綁定方面還有一些較少人知的改進功能值得討論，實際上，這些也是重要的新增功能，能補充 BindingSource 元件所提供的功能。

The `Binding` object has several new members in the .NET Framework 2.0 that enable greater control over the binding operation. For example, you can control how data is formatted in a bound control, when the data source is updated, and how `null` and `DBNull` values in the data source are handled. These new members are also supported with corresponding `Add` methods in the `ControlBindingsCollection`. You can take advantage of these additions by using the `Formatting and Advanced Binding` dialog box in Visual Studio or through code. In addition, the `Binding` object has better support for handling exceptions and errors that can occur in the binding process with the addition of the `BindingComplete` event. This article describes these changes to the `Binding` and `ControlBindingsCollection` objects that you can take advantage of in your application whether or not you choose to use a `BindingSource` component.
繫結物件在 .NET Framework 2.0 中有數個新的成員，可讓您對繫結操作有更進一步的控制。例如，您可以控制繫結控制項中的資料格式、資料來源更新時間，以及如何處理資料來源中的 null 和 DBNull 值。這些新的成員也透過 ControlBindingsCollection 中的對應 Add 方法得到支援。您可透過使用 Visual Studio 中的格式化與進階繫結對話方塊，或透過程式碼來利用這些新增功能。此外，繫結物件透過新增 BindingComplete 事件，對於處理繫結過程中可能發生的例外狀況和錯誤有更好的支援。本文將說明這些對於繫結和 ControlBindingsCollection 物件的變更，無論您是否選擇使用 BindingSource 元件，這些變更都可在您的應用程式中加以利用。

This article starts by introducing the `Formatting and Advanced Binding` dialog box. The rest of this article focuses on how to complete the same types of tasks you would with the dialog box, but in code. Three of the examples use a `DataTable`, and the fourth uses a `BindingSource` object. All the examples can be used with or without a `BindingSource` as the data source for the application, although using a `BindingSource`, especially if you are binding multiple controls to the same data source, is recommended.
%% 本文首先介紹了格式化與進階綁定對話方塊。本文其餘部分則著重於如何透過程式碼完成您原本可以使用對話方塊完成的相同類型的任務。其中三個範例使用 DataTable，而第四個範例則使用 BindingSource 物件。所有範例都可以使用或不需要 BindingSource 作為應用程式的資料來源，不過建議使用 BindingSource，特別是如果您要將多個控制項綁定到相同的資料來源。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#formatting-and-advanced-binding-dialog-box)

## Formatting and Advanced Binding Dialog Box

格式化與進階綁定對話方塊

The `Formatting and Advanced Binding` dialog box in Visual Studio lets you bind control properties to a data source and format the result. This dialog box is designed to take advantage of the changes to the `Binding` object. To access the `Formatting and Advanced Binding` dialog box in Visual Studio, right-click the control you want to bind to data, and select `Properties`. In the `Properties` window shown in Figure 1, expand the `(DataBindings)` item and select the `(Advanced)` item.
Visual Studio 中的格式化與進階綁定對話方塊可讓您將控制項屬性綁定到資料來源，並格式化結果。此對話方塊設計用於利用 Binding 物件所進行的變更。要在 Visual Studio 中存取格式化與進階綁定對話方塊，請右鍵按一下您要綁定資料的控制項，然後選擇屬性。在圖 1 所示的屬性窗口中，展開 (DataBindings) 項目，然後選擇 (Advanced) 項目。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480734.wfbindp101\(en-us,msdn.10\).gif)

`Figure 1. DataBindings settings in the Properties window
圖 1. 屬性窗口中的 DataBindings 設定`

Click the ellipsis button next to `(Advanced)` to open the `Formatting and Advanced Binding` dialog box, shown in Figure 2.
按一下 (進階) 旁邊的省略號按鈕，以打開格式化與進階繫結對話方塊，如圖 2 所示。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480734.wfbindp102\(en-us,msdn.10\).gif)

`Figure 2. The Formatting and Advanced Binding dialog box
圖 2. 格式化與進階繫結對話方塊`

When using the `Formatting and Advanced Binding` dialog box, you select a data source for your application. Your data source can be a `BindingSource` object, a database table, a Web Service type, or an object. Once you have selected a data source, if the data source is not a `BindingSource`, a `BindingSource` for the data source is automatically created. You can then set how the data source is updated when a change is made to the control value, as well as the format type and null value for the binding.
在使用格式化與進階繫結對話方塊時，您會為您的應用程式選擇一個資料來源。您的資料來源可以是一個 BindingSource 物件、一個資料庫表格、一個 Web Service 類型，或是一個物件。一旦您選擇了資料來源，如果資料來源不是 BindingSource，就會自動為該資料來源建立一個 BindingSource。接著，您可以設定當控制項值變更時，資料來源如何更新，以及繫結的格式類型與 null 值。

> `Note`   It is important to note that most of the tasks described in this article can be accomplished using the `Formatting and Advanced Binding` dialog box.
> 注意 值得注意的是，本文中描述的大多數任務都可以使用格式化與進階繫結對話方塊來完成。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#automatic-formatting-using-code)

## Automatic Formatting Using Code

使用程式碼進行自動格式化

In earlier versions of the .NET Framework you had to manually perform the type conversions and formatting using the `Format` and `Parse` events of the `Binding` object. You can now do this by enabling formatting on the `Binding` object, either by setting the `FormattingEnabled` property directly or passing `true` to the `Add` method of the `ControlBindingsCollection`. In addition, you will need to specify a format string, either using the `Add` method, or by setting the `FormatString` property of the `Binding` object.
在 .NET Framework 的早期版本中，您必須使用 Binding 物件的 Format 和 Parse 事件來手動執行類型轉換和格式化。現在，您可以通过啟用 Binding 物件的格式化來完成這些操作，可以直接設定 FormattingEnabled 屬性，或將 true 傳遞給 ControlBindingsCollection 的 Add 方法。此外，您還需要指定一個格式字串，可以使用 Add 方法，或設定 Binding 物件的 FormatString 屬性。

The following code includes two formatting examples. The first formatting example shows how to create a binding between a `DateTime` value in the data source and the `Text` value of a `TextBox` control. The `FormattingEnabled` property is set on the `Binding` object, the `FormatString` value specifies that the `DateTime` should be formatted as a long date/time string, and the binding is added to the collection.
以下程式碼包含兩個格式化範例。第一個格式化範例說明了如何建立一個來自資料來源中的 DateTime 值與 TextBox 控制項的 Text 值之間的繫結。在 Binding 物件上設定 FormattingEnabled 屬性，FormatString 值指定 DateTime 應該格式化為長日期/時間字串，然後繫結被加入到集合中。

The second formatting example demonstrates how to create a `Binding` object that binds data in a `DataTable` to the `Text` property of `TextBox` controls and format the data. The hire date is formatted as a long date and the starting salary is formatted as currency. This example uses the `Add` method of the `ControlBindingsCollection` to create the `Binding` object. This example also sets the `FormatString` of the binding directly. You will see examples of how to set the `FormatString` using the `Add` method of the `ControlBindingsCollection` later in this article.
第二個格式化範例示範如何建立一個繫結物件，將 DataTable 中的資料繫結到 TextBox 控制項的 Text 屬性，並格式化資料。聘用日期格式化為長日期，起薪則格式化為貨幣。此範例使用 ControlBindingsCollection 的 Add 方法來建立繫結物件。此範例也直接設定繫結的 FormatString。你將在本文後續部分看到如何使用 ControlBindingsCollection 的 Add 方法來設定 FormatString 的範例。

Copy  複製

```csharp
Dim table1 As New DataTable()
table1.Columns.Add("Name", GetType(String))
table1.Columns.Add("Hire Date", GetType(DateTime))
table1.Columns.Add("Starting salary", GetType(Integer))

table1.Rows.Add(New Object() {"Claus Hansen", New DateTime(2003, 5, 5), _
  63000})
Dim dateBinding As New Binding("Text", table1, "Hire Date")

' Set FormattingEnabled and FormatString directly on the Binding object.
' The "D" format specifier indicates the long date pattern.
dateBinding.FormattingEnabled = True
dateBinding.FormatString = "D"

' Add the binding to the dateTextBox binding collection.
dateTextBox.DataBindings.Add(dateBinding)

' Set FormattingEnabled as part of the 
' ControlBindingsCollection.Add method.
Dim salaryBinding As Binding = salaryTextBox.DataBindings.Add("text", _
  table1, "Starting Salary", True)

' The "c" format specifier indicates currency format.
salaryBinding.FormatString = "c"

Dim nameBinding As Binding = nameTextBox.DataBindings.Add("Text", table1, _
  "Name", True)
```

The results are shown in Figure 3.
結果如圖 3 所示。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480734.wfbindp103\(en-us,msdn.10\).gif)

`Figure 3. Formatted Currency and Date
圖 3. 格式化貨幣和日期`

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#using-datasourceupdatemode)

## Using DataSourceUpdateMode

使用 DataSourceUpdateMode

The `DataSourceUpdateMode` enumeration lets you specify when control property changes are propagated to the data source. The default value `DataSourceUpdateMode.OnValidation` means the data source is updated when the user tabs away from a control with changes to a data bound value. `DataSourceUpdateNever` indicates changes to a bound control value are never propagated to the data source, and `OnPropertyChange` indicates the changed value will be propagated when any change to the control value is made. With `OnPropertyChange` or `OnValidation`, if the control value does not pass validation, the user will not be able to tab away from the control or close the form until a correct value is entered.
DataSourceUpdateMode 枚舉讓您可以指定當控制項屬性變更時，是否將變更傳播到數據來源。預設值 DataSourceUpdateMode.OnValidation 表示當使用者從具有變更的數據綁定值控制項按 Tab 離開時，會更新數據來源。DataSourceUpdateNever 表示綁定控制項的值變更不會傳播到數據來源，而 OnPropertyChange 表示當控制項值有任何變更時，會傳播變更的值。使用 OnPropertyChange 或 OnValidation 時，如果控制項值未通過驗證，使用者將無法按 Tab 離開控制項或關閉表單，直到輸入正確的值。

The following example demonstrates how to set the `DataSourceUpdateMode` using the `Add` method of the `ControlBindingsCollection`. The hire date in the `DataTable` is updated when any change to the hire date text box is made. The starting salary in the `DataTable` is updated when the starting salary text box loses the focus and the value passes validation. If an invalid value (such as one or more letters) is entered in to the starting salary text box, the user can't tab away from the text box.
以下範例說明了如何使用 ControlBindingsCollection 的 Add 方法來設定 DataSourceUpdateMode。當任 何改變僱傭日期文字方塊時，DataTable 中的僱傭日期會更新。當起始薪資文字方塊失去焦點且值通過驗證時，DataTable 中的起始薪資會更新。若在起始薪資文字方塊中輸入無效值（例如一個或多個字母），使用者無法將焦點從文字方塊移開。

Copy  複製

```csharp
Dim table1 As New DataTable()
table1.Columns.Add("Name", GetType(String))
table1.Columns.Add("Hire Date", GetType(DateTime))
table1.Columns.Add("Starting salary", GetType(Integer))
table1.Rows.Add(New Object() {"Claus Hansen", New DateTime(2003, 5, 5), _
  63000})

' Hire Date in table1 is updated when any change to the hire date
' text box is made.
Dim dateBinding2 As Binding = dateTextBox.DataBindings.Add("Text", table1, _
  "Hire Date", True, DataSourceUpdateMode.OnPropertyChanged)
dateBinding2.FormatString = "D"

' Starting Salary in table1 is updated when the starting salary
' text box loses focus and the value passes validation.
Dim salaryBinding As Binding = salaryTextBox.DataBindings.Add("text", _
  table1, "Starting salary", True, DataSourceUpdateMode.OnValidation)
salaryBinding.FormatString = "c"

Dim nameBinding As Binding = nameTextBox.DataBindings.Add("Text", table1, _
  "Name", True)
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#automatic-handling-of-null-and-dbnull-values)

## Automatic Handling of Null and DBNull Values

自動處理 Null 和 DBNull 值

In earlier versions of Windows Forms data binding, when handling the `Format` and `Parse` events, you had to consider how you would want a `DBNull` or `null` value in the data source to appear in the control. This would require that you parse a particular control value to `null` or `DBNull` in the data source. This is now much easier with the `NullValue` property of `Binding` object.
在早期版本的 Windows Forms 資料綁定中，當處理 Format 和 Parse 事件時，您必須考慮如何讓來源資料中的 DBNull 或 null 值在控制項中顯示。這需要您解析控制項值為來源資料中的 null 或 DBNull。現在，使用 Binding 物件的 NullValue 屬性，這個過程變得容易多了。

The following example demonstrates how a `DBNull` value in the data source is displayed with the `NullValue` property in the bound control. The hire date is a `DBNull` value in the `DataTable`, but is displayed as "New Hire" in the text box.
以下範例說明了如何在繫結的控件中使用 NullValue 屬性顯示來源中的 DBNull 值。錄用日期在 DataTable 中是 DBNull 值，但在文字方塊中顯示為「新僱員」。

Copy  複製

```csharp
Dim table1 As New DataTable()
table1.Columns.Add("Name", GetType(String))
table1.Columns.Add("Hire Date", GetType(DateTime))
table1.Columns.Add("Starting salary", GetType(Integer))

' Initialize Hire Date to DBNull.
table1.Rows.Add(New Object() {"Claus Hansen", DBNull.Value, 63000})

' If Hire Date is null or DBNull, change to "New Hire".
Dim dateBinding2 As Binding = dateTextBox.DataBindings.Add("Text", table1, _
  "Hire Date", True, DataSourceUpdateMode.OnValidation, "New Hire", "D")

Dim salaryBinding As Binding = salaryTextBox.DataBindings.Add("text", _
  table1, "Starting salary", True)
salaryBinding.FormatString = "c"

Dim nameBinding As Binding = nameTextBox.DataBindings.Add("Text", table1, _
  "Name", True)
```

The results are shown in Figure 4.
結果如圖 4 所示。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480734.wfbindp104\(en-us,msdn.10\).gif)

`Figure 4. DBNull formatted as the string "New Hire"
圖 4. DBNull 以字串 "New Hire" 的格式呈現`

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#improved-handling-of-errors-and-exceptions)

## Improved Handling of Errors and Exceptions

錯誤和例外處理的改進

Whenever you enable formatting on a data binding you now have greater control over error and exception handling using the `BindingComplete` event of the `Binding` object.
每當您在數據綁定上啟用格式化時，您現在可以使用 Binding 物件的 BindingComplete 事件對錯誤和例外處理進行更精細的控制。

The following example shows how you might handle the `BindingComplete` event to provide feedback to the user when a bound control value is changed to an invalid value. This example shows a business object, `Employee`, which does not allow for a value of less than 20000 for its `Salary` property. For binding purposes, the employee objects are stored in a `BindingSource`.
以下範例說明了如何處理 BindingComplete 事件，以便在綁定控制項的值變更為無效值時向用戶提供反饋。此範例展示了一個業務物件， `Employee` ，其 `Salary` 屬性不允许值小於 20000。為了綁定目的，員工物件存儲在 BindingSource 中。

First, I create the `BindingSource`, add two Employee objects, and establish the bindings. I also associate the `BindingComplete` event with its event-handling method.
首先，我創建了 BindingSource，添加了兩個 Employee 物件，並建立了綁定。我還將 BindingComplete 事件與其事件處理方法關聯起來。

Copy  複製

```csharp
' Declare the Binding object.
Dim WithEvents salaryBinding As Binding

Dim employeeBindingSource As New BindingSource()
employeeBindingSource.Add(New Employee("Hansen", "Claus", 63000, _
  New DateTime(2002, 5, 5)))
employeeBindingSource.Add(New Employee("Han", "Mu", 54000, _
  New DateTime(2001, 3, 4)))
nameTextBox.DataBindings.Add("Text", employeeBindingSource, _
  "LastName", True)
dateTextBox.DataBindings.Add("Text", employeeBindingSource, _
  "StartDate", True, DataSourceUpdateMode.OnValidation)

salaryBinding = salaryTextBox.DataBindings.Add("text", _
  employeeBindingSource, "Salary", True, DataSourceUpdateMode.OnValidation, _
  "Unknown", "c")
```

Next, I handle the `BindingComplete` event. This event-handler passes the error message to an `ErrorProvider` control.
接下來，我處理 BindingComplete 事件。這個事件處理程式會將錯誤訊息傳送給 ErrorProvider 控制項。

Copy  複製

```csharp
Sub salaryBinding_BindingComplete(ByVal sender As Object, _
  ByVal e As BindingCompleteEventArgs) Handles salaryBinding.BindingComplete

    ' If the BindingComplete state is anything other than success, 
    ' set the ErrorProvider to the error message.
    If e.BindingCompleteState <> BindingCompleteState.Success Then
        errorProvider1.SetError( _
          CType(e.Binding.BindableComponent, Control), e.ErrorText)
    Else
        errorProvider1.SetError( _
          CType(e.Binding.BindableComponent, Control), "")
    End If
End Sub
```

The following code shows the `Employee` business object.
以下程式碼顯示了 `Employee` 商業物件。

Copy  複製

```csharp
Public Class Employee
    Private lastNameValue As String
    Private firstNameValue As String
    Private salaryValue As Integer
    Private startDateValue As DateTime
    
    Public Sub New(ByVal lastName As String, ByVal firstName As String, _
      ByVal salary As Integer, ByVal startDate As DateTime)
        lastNameValue = lastName
        firstNameValue = firstName
        salaryValue = salary
        startDateValue = startDate
    End Sub

    Public Property LastName() As String
        Get
            Return lastNameValue
        End Get
        Set(ByVal value As String)
            lastNameValue = value
        End Set
    End Property
    
    Public Property FirstName() As String 
        Get
            Return firstNameValue
        End Get
        Set
            firstNameValue = value
        End Set
    End Property 
    
    Public Property Salary() As Integer 
        Get
            Return salaryValue
        End Get
        Set
            If value < 20000 Then
                Throw New Exception("Salary cannot be less than $20,000")
            Else
                salaryValue = value
            End If
        End Set
    End Property 
    
    Public Property StartDate() As DateTime 
        Get
            Return startDateValue
        End Get
        Set
            startDateValue = value
        End Set
    End Property
    
    Public Overrides Function ToString() As String 
        Return LastName & ", " & FirstName & vbLf & "Salary:" _
          & salaryValue & vbLf & "Start date:" & startDateValue
    End Function
End Class
```

In order to see the error-handling occur, run the example and change the salary to a value less than $20,000. Figure 5 is a screen shot of the feedback to the user by handling the `BindingComplete` event.
若要觀察錯誤處理的過程，請執行範例並將薪資改為小於 20,000 美元的值。圖 5 是透過處理 BindingComplete 事件來回饋使用者畫面的螢幕擷取畫面。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480734.wfbindp105\(en-us,msdn.10\).gif)

`Figure 5. Example that shows how the BindingComplete event can be used to verify the data in a bound control
圖 5. 顯示如何使用 BindingComplete 事件來驗證綁定控制項中的資料`

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480734\(v=msdn.10\)#conclusion)

## Conclusion  結論

The `BindingSource` might be one of the most talked about additions for Windows Forms data binding, but there are many other useful features that were added to the `Binding` and `ControlBindingsCollection` objects that complement the functionality of the `BindingSource`. With these changes you can easily format bound data, handle null values in the data source and gracefully handle exceptions that occur in the binding process, whether or not you use a `BindingSource` component on your form. For more information on Windows Forms, see:
繫結來源（BindingSource）可能是 Windows Forms 繫結中最常被討論的新增功能，但還有許多其他實用的功能被新增到繫結（Binding）和 ControlBindingsCollection 物件中，以補充繫結來源的功能。透過這些變更，您可以輕鬆地格式化繫結的資料、處理來源中的 null 值，並在繫結過程中順利處理發生的例外狀況，無論您是否在表單上使用繫結來源元件。如需更多有關 Windows Forms 的資訊，請參考：

- [Windows Forms on Microsoft .NET Framework Developer Center  
    Microsoft .NET Framework 開發者中心中的 Windows Forms](https://go.microsoft.com/fwlink/?linkid=70365&clcid=0x409)
- [WindowsForms.NET](https://go.microsoft.com/fwlink/?linkid=70366&clcid=0x409)
- [Windows Forms Documentation Updates Blog  
    Windows Forms 文件更新部落格](https://go.microsoft.com/fwlink/?linkid=69202&clcid=0x409)

Continue to [Behind the Scenes: Improvements to Windows Forms Data Binding in the .NET Framework 2.0, Part 2](https://go.microsoft.com/fwlink/?linkid=69697&clcid=0x409).
繼續探索：.NET Framework 2.0 中 Windows Forms 資料綁定之改進，第二部分。
