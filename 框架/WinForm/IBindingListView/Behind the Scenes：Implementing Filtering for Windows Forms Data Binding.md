---
aliases:
date:
author: Cheryl Simmons
language: C#
sourceurl: https://www.microsoft.com/en-ie/download/details.aspx?id=17914
tags:
  - CSharp
  - 資料庫
---

# Behind the Scenes: Implementing Filtering for Windows Forms Data Binding

Cheryl Simmons
Microsoft Corporation
April 2008

Applies to:

- Microsoft Visual Studio 2005
- Microsoft .NET Framework 2.0
- Microsoft Windows Forms

Summary: Learn how to extend the generic `BindingList` and implement the filtering functionality of the `IBindingListView`.
摘要：學習如何擴展通用的 `BindingList` 並實作 `IBindingListView` 的過濾功能。

Contents

- Introduction<br />簡介
- Filtering Overview<br />過濾概述
- Determining the Filtering Expression Format<br />確定過濾表達式格式
- Creating a Generic Filtered Class<br />建立通用過濾類
- Disabling Advanced Sorting<br />停用進階排序
- Implementing the SupportsFiltering Property<br />實作 SupportsFiltering 屬性
- Implementing the Filter Property<br />實作 Filter 屬性
- Implementing the RemoveFilter Method<br />實作 RemoveFilter 方法
- Using the Filtered List<br />使用過濾列表
- Next Steps<br />後續步驟
- Conclusion<br />總結

# Introduction<br />簡介

When you bind Windows Forms controls to an ADO.NET data source, specifically with a `BindingSource` as an intermediary, the searching, sorting, and filtering capabilities built in to a `DataView` are automatically exposed by the `BindingSource`. However, when you want to bind to a list of business objects, you have to implement these features yourself. A good solution is a generic implementation that can be reused with various business objects. This article describes how to extend the generic `BindingList` and implement the filtering functionality of the `IBindingListView`. This implementation enables you to store custom business objects in a list that you can search, sort, and filter.
當您將 Windows Forms 控件綁定到 ADO.NET 資料源時，特別是使用 `BindingSource` 作為中間介體，`DataView` 中內建的搜索、排序和過濾功能將自動由 `BindingSource` 暴露。然而，當您想要綁定到商業對象的列表時，您必須自行實現這些功能。一個好的解決方案是通用的實現，可以與各種商業對象重用。本文描述了如何擴展通用的 `BindingList` 和實現 `IBindingListView` 的過濾功能。這種實現使您能夠在列表中存儲自定義商業對象，並可以進行搜索、排序和過濾。

# Filtering Overview<br />過濾概述

In the summer of 2006, I published an article that demonstrated how to extend the generic `BindingList` and implement basic searching and sorting functionality. To read this article, see [Behind the Scenes: Improvements to Windows Forms Data Binding in the .NET Framework 2.0, Part 2](http://go.microsoft.com/fwlink/?LinkID=69697). I use the example presented in that article as the starting point for this article, and I add filtering capabilities. The final result is an extension of the generic binding list that you can search, sort, and filter. The example provided in this article correctly interacts with the `BindingSource` and bound controls to offer filtering functionality. In addition, I`ve designed this example to interoperate with the drop-down filtering implemented in [Building a Drop-Down Filter List for a DataGridView Column Header Cell](http://go.microsoft.com/fwlink/?LinkId=105803). The filtering implementation, like the sorting implementation, relies on the `IComparable` interface to do the heavy lifting.
在 2006 年夏天，我發表了一篇文章，展示了如何擴展泛型 `BindingList` 並實現基本的搜索和排序功能。要閱讀這篇文章，請參考 [在幕後：.NET Framework 2.0中Windows Forms數據綁定的改進，第二部分](http://go.microsoft.com/fwlink/?LinkID=69697)。我將那篇文章中的範例作為本篇文章的起點，並增加了過濾功能。最終結果是一個可以搜索、排序和過濾的泛型綁定列表的擴展。本篇文章提供的範例正確地與 `BindingSource` 和綁定的控件互動，以提供過濾功能。此外，我設計了這個範例以與 [為DataGridView列標題單元實現下拉過濾列表](http://go.microsoft.com/fwlink/?LinkId=105803) 中的下拉過濾功能互操作。過濾實現，就像排序實現一樣，依賴於 `IComparable` 接口來完成繁重的任務。

Filtering can be implemented in various ways. In this article I walk you through the following steps:
過濾可以以多種方式實施。在本文中，我將為您介紹以下步驟：

- Determining the filtering expression format to use.
- Creating a generic filtered class.
- Disabling the advanced sorting capabilities of IBindingListView.
- Implementing the filtering capabilities of IBindingListView.
- Using and testing the filtered list populated with business objects and bound to a DataGridView control.
- 決定使用何種過濾表達式格式。
- 建立一個通用的過濾類別。
- 禁用 IBindingListView 的先進排序功能。
- 實現 IBindingListView 的過濾功能。
- 使用並測試用業務對象填充的過濾清單，並將其綁定到 DataGridView 控件。

# Determining the Filtering Expression Format<br />確定過濾表達式格式

Before I can implement filtering, I have to determine the format of filtering expressions that my application will accept. A filtering expression can follow any format that you want. However, to maintain consistency with ADO.NET, the filter expressions used for this article follows a subset of the guidelines established by the `DataColumn.Expression` property. These are the same guidelines used by the `DataView.RowFilter` property for single-column filtering. The `RowFilter` property expects a column name followed by an operator and a value to filter on. Since I am working with business objects, the filter I demonstrate expects a property name instead of a column name. To interoperate with the `DataGridView` drop-down auto-filtering, I accept optional brackets around the property name, which allow symbols to be used in the filtering expression. In addition, the filtering is limited to the equal to, less than, and greater than operators (=, <, >). The filter value should be in single quotation marks if it is a string or date value. The following are some examples of filtering expressions that my code handles.
在實施過濾功能之前，我必須確定應用程式將接受的過濾表達式格式。過濾表達式可以遵循您想要的任何格式。然而，為了與 ADO.NET 保持一致性，本文中使用的過濾表達式遵循了由 `DataColumn.Expression` 屬性建立的指南的子集。這些是與 `DataView.RowFilter` 屬性用於單列過濾相同的指南。`RowFilter` 屬性期望一個列名後跟一個操作符和一個過濾值。由於我正在處理商業對象，所以我展示的過濾器期望屬性名而不是列名。為了與 `DataGridView` 下拉自動過濾功能相互操作，我接受屬性名可選的括號，這允許在過濾表達式中使用符號。此外，過濾僅限於等於、小於和大於操作符（=、<、>）。如果過濾值是字串或日期值，則應使用單引號。以下是我代碼處理的一些過濾表達式範例。

- LastName = 'Smith'
- Salary > 50000
- Birthday < `1/1/1968`

# Creating a Generic Filtered Class<br />建立通用過濾類

To add filtering, I start by creating a generic class named `FilteredBindingList`. This class extends the generic binding list and implements the `IBindingListView` interface. The code I use for searching and sorting is very similar to the code I presented in [Behind the Scenes: Improvements to Windows Forms Data Binding in the .NET Framework 2.0, Part 2](http://go.microsoft.com/fwlink/?LinkID=69697). In the next few sections I discuss code I added to implement the filtering portion of the `IBindingListView`. The following is the class declaration. I extend the generic `BindingList` class and implement the `IBindingListView` interface.
為了新增過濾功能，我先建立一個名為 `FilteredBindingList` 的泛型類別。該類別擴展了泛型綁定列表並實作了 `IBindingListView` 介面。我用於搜尋和排序的程式碼與我在 [幕後：.NET Framework 2.0 中 Windows 窗體資料綁定的改進，第 2 部分](http://go.microsoft.com/fwlink/?LinkID=69697) 中展示的程式碼非常相似。在接下來的幾節中，我將討論我為實作 `IBindingListView` 的過濾部分而新增的程式碼。以下是類別聲明。我擴展了泛型 `BindingList` 類別並實作了 `IBindingListView` 介面。
```csharp
public class FilteredBindingList<T> : BindingList<T>, IBindingListView
{
    ...
}
```

# Disabling Advanced Sorting<br />停用進階排序

The `IBindingListView` interface extends the `IBindingList` interface by offering filtering and advanced (multi-column) sorting capability. Since this example of `IBindingListView` implements the filtering portion only, I disable the advanced sorting capability. I set the `SupportsAdvancedSorting` property to always return `false` to notify bound controls that the data source does not perform advanced sorting. I set the `SortDescriptions` property to return `null`. I set the `ApplySort` overload that takes a collection of sort descriptions to throw a `NotSupportedException` to indicate that multiple-column sorting cannot be performed. The following code shows how I disable the advanced sorting capability of `IBindingListView`.
`IBindingListView` 介面擴展了 `IBindingList` 接口，提供了過濾和進階（多列）排序功能。由於此 `IBindingListView` 範例僅實作了過濾部分，因此我停用了進階排序功能。我將 `SupportsAdvancedSorting` 屬性設定為始終傳回 `false`，以通知綁定控制項資料來源不執行進階排序。我將 `SortDescriptions` 屬性設為傳回 `null`。我將接受排序描述集合的 `ApplySort` 重載設定為拋出 `NotSupportedException`，以指示無法執行多列排序。以下程式碼顯示如何停用 `IBindingListView` 的進階排序功能。
```csharp
#region AdvancedSorting

public bool SupportsAdvancedSorting
{
    get { return false; }
}

public ListSortDescriptionCollection SortDescriptions
{
    get { return null; }
}

public void ApplySort(ListSortDescriptionCollection sorts)
{
    throw new NotSupportedException();
}

#endregion AdvancedSorting
```

# Implementing the SupportsFiltering Property<br />實作 SupportsFiltering 屬性

The first step to provide filtering is to implement the `SupportsFiltering` property. The `SupportsFiltering` property notifies bound controls and components that filtering is supported. The following code implements `SupportsFiltering` and always returns `true`.
提供過濾功能的第一步是實作 `SupportsFiltering` 屬性。 `SupportsFiltering` 屬性會通知綁定的控制項和元件已支援過濾功能。以下程式碼實作了 `SupportsFiltering` 屬性，並且始終傳回 `true`。
```csharp
public bool SupportsFiltering
{
    get { return true; }
}
```

# Implementing the Filter Property<br />實作 Filter 屬性

The next requirement is to implement the `Filter` property. The `Filter` property is the guts of filtering for the data source. When the filter property is set, it is applied to the data source and the data source returns the items that fit the filter criteria.
下一個要求是實作 `Filter` 屬性。 `Filter` 屬性是資料來源過濾的核心。設定 filter 屬性後，它會套用到資料來源，資料來源會傳回符合過濾條件的項目。

First, I declare a private string to hold the filter, and then expose it publicly, as shown in the following code.
首先，我聲明一個私有字串來保存過濾器，然後將其公開，如下面的程式碼所示。
```csharp
private string filterValue = null;
public string Filter
{
    ...
}
```

The get implementation for the `Filter` property is simple; just return `filterValue`.
`Filter` 屬性的取得實作很簡單；只需傳回 `filterValue`。
```csharp
get
{
    return filterValue;
}
```

Implementing the set operation for the `Filter` property is more difficult and requires some preliminary steps. As mentioned previously, when the filter value is set, the filter is applied to the list and the list returns the items that meet the filter criteria. In addition, the original list must be cached because the filter can be removed, in which case, the list should revert to its original contents. Therefore, the unfiltered list is stored in a read-only property named `OriginalList`. I chose a public read-only property to allow for the code to be unit-tested. The following code shows the declaration for the unfiltered list.
實作 `Filter` 屬性的設定操作更加困難，需要一些準備步驟。如前所述，在設定過濾器值後，過濾器將應用於列表，列表將傳回符合過濾條件的項目。此外，由於過濾器可能會被移除，因此必須快取原始列表，在這種情況下，列表應該恢復到其原始內容。因此，未過濾的清單儲存在名為 `OriginalList` 的唯讀屬性中。我選擇了一個公共只讀屬性，以便對程式碼進行單元測試。以下程式碼展示了未過濾清單的聲明。
```csharp
private List<T> originalListValue = new List<T>();
public List<T> OriginalList
{
    get
    { return originalListValue; }
}
```

Since I want to enable the user to add and remove items from the list, I override the `OnListChanged` method and check for items added and removed from the list. To keep my unfiltered list up-to-date, I add or remove these items from the unfiltered list as necessary.
由於我希望允許使用者在清單中新增和移除項目，我重寫了 `OnListChanged` 方法，並檢查清單中新增和移除的項目。為了讓我的未過濾清單保持最新，我會根據需要在未過濾清單中新增或移除這些項目。

To eliminate problems with the bound controls, I check to see whether the list is filtered, and turn off the ability to add items to the list if a filter is applied. Since the ability to filter can be overridden on the client-side, I check to see whether a filter is applied when an item is added. Once the item is added, I reapply the filter to the list, which includes the newly added item.
為了消除綁定控制項的問題，我會檢查清單是否已過濾，並在應用了篩選器的情況下關閉向清單新增項目的功能。由於過濾功能可以在客戶端被覆蓋，因此我會在新增項目時檢查是否已套用了過濾器。新增項目後，我會將篩選器重新套用至包含新新增項目的清單。
```csharp
protected override void OnListChanged(ListChangedEventArgs e)
{
    // If the list is reset, check for a filter. If a filter
    // is applied, don't allow items to be added to the list.
    if (e.ListChangedType == ListChangedType.Reset)
    {
        if (Filter == null || Filter == "")
            AllowNew = true;
        else
            AllowNew = false;
    }
    
    // Add the new item to the original list.
    if (e.ListChangedType == ListChangedType.ItemAdded)
    {
        OriginalList.Add(this[e.NewIndex]);
        if (!String.IsNullOrEmpty(Filter))
        {
            string cachedFilter = this.Filter;
            this.Filter = "";
            this.Filter = cachedFilter;
        }
    }
    
    // Remove the new item from the original list.
    if (e.ListChangedType == ListChangedType.ItemDeleted)
        OriginalList.RemoveAt(e.NewIndex);
        
    base.OnListChanged(e);
}
```

## Determining if the Filter is Valid<br />確定過濾器是否有效

The first step in setting the `Filter` property is to check whether the filter value has changed. If the filter value is unchanged, the set operation returns. Next, I check to see whether the filter matches the expected format, and throw an exception if it does not. For readability purposes, I`ve created a method named `BuildRegExForFilterFormat` to build up the regular expression that checks the filter format.
設定 `Filter` 屬性的第一步是檢查篩選器值是否已變更。如果過濾器值未更改，則設定操作返回。接下來，我檢查過濾器是否符合預期格式，如果不匹配，則拋出異常。為了方便閱讀，我建立了一個名為 `BuildRegExForFilterFormat` 的方法來建立檢查篩選器格式的正規表示式。
```csharp
// Build a regular expression to determine if
// filter is in correct format.
public static string BuildRegExForFilterFormat()
{
    StringBuilder regex = new StringBuilder();
    // Look for optional literal brackets,
    // followed by word characters or space.
    regex.Append(@"\[?[\w\s]+\]?\s?");
    // Add the operators: > < or =.
    regex.Append(@"[><=]");
    //Add optional space followed by optional quote and
    // any character followed by the optional quote.
    regex.Append(@"\s?'?.+'?");
	return regex.ToString();
}
```

At this point in the set operation, I temporarily disable change notifications for the list. It is important that I do this step because next I manipulate the contents of the list. In the context of an application, bound controls should not be notified of these internal changes. To disable change notifications, I set the `RaiseListChangedEvents` property to `false`. This property is set back to `true` when the set call is finished.
在設定操作的這個階段，我暫時停用了清單的變更通知。執行此步驟非常重要，因為接下來我將操作清單的內容。在應用程式上下文中，綁定控制項不應收到這些內部變更的通知。為了停用更改通知，我將 `RaiseListChangedEvents` 屬性設為 `false`。設定呼叫完成後，此屬性將重新設定為 `true`。

Now, I split the filter string at `AND` which indicates a multi-column filter. This allows me to loop through each filter part, parse the filter string, and apply it to the data source. Using this approach, filters are applied incrementally and in the same order that they are found in the filter string. Next, I set the filter value and set the `RaiseListChangedEvents` property back to `true`. Finally, I call `OnListChanged` with parameters to signal bound controls to refresh. In the following code, the filter parsing and application is accomplished by two separate methods; `ParseFilter` and `ApplyFilter`. These methods, and their supporting methods, are examined in detail, in the next section.
現在，我將過濾器字串以 `AND` 分​​隔開來，這表示這是一個多列過濾器。這樣我就可以循環遍歷每個過濾器部分，解析過濾器字串，並將其套用到資料來源。使用這種方法，過濾器會以增量方式套用，並且順序與它們在過濾器字串中找到的順序相同。接下來，我設定過濾器值，並將 `RaiseListChangedEvents` 屬性重新設定為 `true`。最後，我呼叫帶有參數的 `OnListChanged` 來通知綁定的控制項進行刷新。在下面的程式碼中，過濾器的解析和應用由兩個獨立的方法完成：`ParseFilter` 和 `ApplyFilter`。下一節將詳細介紹這些方法及其支援方法。
```csharp
set
{
    if (filterValue == value) return;

    // If the value is not null or empty, but doesn't
    // match expected format, throw an exception.
    if (!string.IsNullOrEmpty(value) && !Regex.IsMatch(value, BuildRegExForFilterFormat(), RegexOptions.Singleline))
        throw new ArgumentException("Filter is not in " + "the format: propName[<>=]'value'.");

    //Turn off list-changed events.
    RaiseListChangedEvents = false;

    // If the value is null or empty, reset list.
    if (string.IsNullOrEmpty(value))
        ResetList();
    else
    {
        int count = 0;
        string[] matches = value.Split(new string[] { " AND " }, StringSplitOptions.RemoveEmptyEntries);
        while (count < matches.Length)
        {
            string filterPart = matches[count].ToString();
            // Check to see if the filter was set previously.
            // Also, check if current filter is a subset of
            // the previous filter.
            if (!String.IsNullOrEmpty(filterValue) && !value.Contains(filterValue))
                ResetList();
            // Parse and apply the filter.
            SingleFilterInfo filterInfo = ParseFilter(filterPart);
            ApplyFilter(filterInfo);
            count++;
        }
    }

    // Set the filter value and turn on list changed events.
    filterValue = value;
    RaiseListChangedEvents = true;
    OnListChanged(new ListChangedEventArgs(ListChangedType.Reset, -1));
}
```

The following is the `ResetList` method. I copy the items from the original list back to my list and reapply the sort, if necessary. For sort implementation details, see the complete code example.
以下是 `ResetList` 方法。我將原始列表中的項目複製回我的列表，並在必要時重新套用排序。有關排序實現的詳細信息，請參閱完整的程式碼範例。
```csharp
private void ResetList()
{
    this.ClearItems();
    foreach (T t in originalListValue)
        this.Items.Add(t);
    if (IsSortedCore)
        ApplySortCore(SortPropertyCore, SortDirectionCore);
}
```

## Parsing the Filter<br />解析過濾器

Parsing the filter can be very labor-intensive, depending on the sophistication of the filtering that is implemented. You must determine the filter operator and the filter property, and then store these values. I chose to store them in a structure named `SingleFilterInfo`. The following code shows you the structure.
解析過濾器可能非常耗費人力，這取決於所實現的過濾的複雜程度。您必須確定篩選器運算子和篩選器屬性，然後儲存這些值。我選擇將它們儲存在名為 `SingleFilterInfo` 的結構體中。以下程式碼展示了該結構體。
```csharp
public struct SingleFilterInfo
{
    internal string PropName;
    internal PropertyDescriptor PropDesc;
    internal Object CompareValue;
    internal FilterOperator OperatorValue;
}
```

The comparison value is declared as an Object because I do not know this type until I determine the property to filter by. The filter operator is declared as an `FilterOperator` enumeration value. The following enumeration shows the filter operators I accept.
比較值聲明為 Object，因為在確定要過濾的屬性之前，我不知道它的類型。過濾運算子宣告為 `FilterOperator` 枚舉值。以下枚舉顯示了我接受的過濾運算子。
```csharp
// Enum to hold filter operators. The chars
// are converted to their integer values.
public enum FilterOperator
{
    EqualTo = '=',
    LessThan = '<',
    GreaterThan = '>',
    None = ' '
}
```

With this structure and enumeration in place, I can perform the actual filter parsing. First I declare a `SingleFilterInfo` object and use the `DetermineFilterOperator` method to determine the operator and store this value. Next, I split the filter string and examine the first value. The code retrieves and uses a `TypeConverter` to convert the filter value to the property type. The code throws exceptions if the specified property does not exist or the type conversion cannot be performed.
有了這個結構和枚舉，我就可以執行實際的過濾器解析了。首先，我宣告一個 `SingleFilterInfo` 對象，並使用 `DetermineFilterOperator` 方法確定運算子並儲存該值。接下來，我拆分過濾器字串並檢查第一個值。程式碼檢索並使用 `TypeConverter` 將過濾器值轉換為屬性類型。如果指定的屬性不存在或無法執行類型轉換，程式碼將拋出異常。
```csharp
internal SingleFilterInfo ParseFilter(string filterPart)
{
    SingleFilterInfo filterInfo = new SingleFilterInfo();
    filterInfo.OperatorValue = DetermineFilterOperator(filterPart);
    string[] filterStringParts = filterPart.Split(new char[]
    {
        (char)filterInfo.OperatorValue
    });
    filterInfo.PropName = filterStringParts[0].Replace("[", ""). Replace("]", "").Replace(" AND ", "").Trim();

    // Get the property descriptor for the filter property name.
    PropertyDescriptor filterPropDesc = TypeDescriptor.GetProperties(typeof(T))[filterInfo.PropName];

    // Convert the filter compare value to the property type.
    if (filterPropDesc == null)
        throw new InvalidOperationException("Specified property to " +
            "filter " + filterInfo.PropName +
            " on does not exist on type: " + typeof(T).Name);
    filterInfo.PropDesc = filterPropDesc;

    string comparePartNoQuotes = StripOffQuotes(filterStringParts[1]);
    try
    {
        TypeConverter converter = TypeDescriptor.GetConverter(filterPropDesc.PropertyType);
        filterInfo.CompareValue = converter.ConvertFromString(comparePartNoQuotes);
    }
    catch (NotSupportedException)
    {
        throw new InvalidOperationException("Specified filter" +
            "value " + comparePartNoQuotes + " can not be converted" +
            "from string. Implement a type converter for " +
            filterPropDesc.PropertyType.ToString());
    }
    return filterInfo;
}
```

The following are the `DetermineFilterOperator` and `StripOffQuotes` methods that are used by `ParseFilter`.
以下是 `ParseFilter` 使用的 `DetermineFilterOperator` 和 `StripOffQuotes` 方法。
```csharp
internal FilterOperator DetermineFilterOperator(string filterPart)
{
    // Determine the filter's operator.
    if (Regex.IsMatch(filterPart, "[^>^<]="))
        return FilterOperator.EqualTo;
    else if (Regex.IsMatch(filterPart, "<[^>^=]"))
        return FilterOperator.LessThan;
    else if (Regex.IsMatch(filterPart, "[^<]>[^=]"))
        return FilterOperator.GreaterThan;
    else
        return FilterOperator.None;
}

internal static string StripOffQuotes(string filterPart)
{
    // Strip off quotes in compare value if they are present.
    if (Regex.IsMatch(filterPart, "'.+'"))
    {
        int quote = filterPart.IndexOf('\'');
        filterPart = filterPart.Remove(quote, 1);
        quote = filterPart.LastIndexOf('\'');
        filterPart = filterPart.Remove(quote, 1);
        filterPart = filterPart.Trim();
    }
    return filterPart;
}
```

## Applying the Filter<br />應用過濾器

Next, I apply the filter by passing the `SingleFilterInfo` object I built in `ParseFilter` to the `ApplyFilter` method. First, I check to see whether the specified property type implements the `IComparable` interface. The filter mechanism relies on the `Compare` method of this interface and the code throws an exception if the property does not implement this interface. Depending on the operator, I use the `Compare` method to populate the results list. Finally, I clear the original list and copy the results list to this list.
接下來，我將在 `ParseFilter` 中建置的 `SingleFilterInfo` 物件傳遞給 `ApplyFilter` 方法來套用過濾器。首先，我檢查指定的屬性類型是否實作了 `IComparable` 介面。過濾機制依賴於此接口的 `Compare` 方法，如果屬性未實作此接口，程式碼將拋出異常。根據運算符，我使用 `Compare` 方法填入結果清單。最後，我清除原始清單並將結果清單複製到此清單中。
```csharp
internal void ApplyFilter(SingleFilterInfo filterParts)
{
    List<T> results;
    // Check to see if the property type we are filtering by implements
    // the IComparable interface.
    Type interfaceType =
        TypeDescriptor.GetProperties(typeof(T))[filterParts.PropName]
        .PropertyType.GetInterface("IComparable");
    if (interfaceType == null)
        throw new InvalidOperationException("Filtered property" +
        " must implement IComparable.");
    results = new List<T>();
    // Check each value and add to the results list.
    foreach (T item in this)
    {
        if (filterParts.PropDesc.GetValue(item) != null)
        {
            IComparable compareValue = filterParts.PropDesc.GetValue(item) as IComparable;
            int result = compareValue.CompareTo(filterParts.CompareValue);
            if (filterParts.OperatorValue == FilterOperator.EqualTo && result == 0)
                results.Add(item);
            if (filterParts.OperatorValue == FilterOperator.GreaterThan && result > 0)
                results.Add(item);
            if (filterParts.OperatorValue == FilterOperator.LessThan && result < 0)
                results.Add(item);
        }
    }
    this.ClearItems();
    foreach (T itemFound in results)
        this.Add(itemFound);
}
```

# Implementing the RemoveFilter Method<br />實作 RemoveFilter 方法

The final step is to implement the `RemoveFilter` method. This implementation just checks whether the `Filter` property is equal to `null`. If `Filter` is not `null`, `Filter` is set to `null`. Setting `Filter` to `null` calls the set implementation of `Filter`, which clears the filtered items and copies the original list contents back into the list.
```csharp
public void RemoveFilter()
{
    if (Filter != null) Filter = null;
}
```

# Using the Filtered List<br />使用過濾列表

To use the filtered list, I created a new Windows Forms application. In my example, the filtered list and application are located in the same namespace and assembly. Depending on how you have configured your application, you may have to add an assembly reference and a using statement for the namespace that contains the `FilteredListView` class. Using the Windows Forms Designer, I added a `BindingSource`, two `Button` controls, two `ComboBox` controls, a `DataGridView` and a `TextBox` to the form. I left the names set to the default; _dataGridView1_, _bindingSource1_, and so on. I set the `Dock` property of the `DataGridView` to `Top` and I set the `Text` property of _button1_ and _button2_ to `Apply Filter` and `Remove Filter`, respectively. When finished, the form resembles the following illustration.

![BindingFiltering01.jpg](data:image/jpeg;base64,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6f8A9DNbz/hr5HPH+L9/5o+q/C4/4onw+f8AqE2n/ohK5Xwv/wAilo3/AF4wf+i1rrfC4/4obQD/ANQm0/8ARCVyXhf/AJFLRv8Arxg/9FrUT+FGkN2atFFFZmgUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFAFrwSM+HX/6/73/0plr5f8T/API2az/1/T/+hmvqPwOM+HH/AOv+9/8ASmWvlzxP/wAjZrP/AF/T/wDoZref8NfI54/xfv8AzR9Y+Fh/xQegH/qEWv8A6ISuP8L/APIpaN/14wf+i1rsvCo/4oDQP+wRa/8AohK43wv/AMilo3/XjB/6LWonsjWO7NWiiisywooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKANDwIM+G3/AOwhe/8ApTJXyz4n/wCRs1n/AK/p/wD0M19U+Ahnw0//AGEL3/0pkr5W8T/8jZrP/X9P/wChmtpfB9xgv4vyf5o+sfC0yDwHoCFvm/si1GP+2CVx/hf/AJFLRv8Arxg/9FrXSeGf+RO8P/8AYKtP/RCVzfhf/kUtG/68YP8A0WtKatFFwd2zVooorI0CiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA0PAkyR+G3DNg/2hen/AMmZK+WfE/8AyNms/wDX9P8A+hmvpvwd/wAgBv8Ar+vf/SqWvmTxP/yNms/9f0//AKGa3mrU18jni/3v3/mj6l8M/wDIneH/APsFWn/ohK5vwv8A8ilo3/XjB/6LWuk8M/8AIneH/wDsFWn/AKISub8L/wDIpaN/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)

## Adding a Business Object

The following code is a simple `Employee` object I use to demonstrate the filtered list. If you want to use this business object, copy it to the form, after the form class. Otherwise, you will have to reference the assembly that contains your business object.
```csharp
public class Employee
{
    private string lastNameValue;
    private string firstNameValue;
    private int salaryValue;
    private DateTime startDateValue;

    public Employee() {
        LastName = "Last Name";
        FirstName = "First Name";
        Salary = 40000;
        StartDate = DateTime.Today;
    }

    public Employee(string lastName, string firstName, int salary, DateTime startDate)
    {
        LastName = lastName;
        FirstName = firstName;
        Salary = salary;
        StartDate = startDate;
    }

    public string LastName
    {
        get { return lastNameValue; }
        set { lastNameValue = value; }
    }

    public string FirstName
    {
        get { return firstNameValue; }
        set { firstNameValue = value; }
    }

    public int Salary
    {
        get { return salaryValue; }
        set { salaryValue = value; }
    }

    public DateTime StartDate
    {
        get { return startDateValue; }
        set { startDateValue = value; }
    }
```

## Establishing the Data Source

Now that I have a business object, the next step is to set up the data source. The following code is a list declaration and the form`s constructor. This constructor populates the list, and sets the data source for the binding source and other controls. If I want to use a different business object, I simply replace the code that adds `Employee` objects to the list with code that populates the list with a different business object.
```csharp
FilteredBindingList<Employee> businessObjects;

public Form1()
{
    InitializeComponent();
    
    businessObjects = new FilteredBindingList<Employee>();
    businessObjects.Add(new Employee("Haas", "Jonathan", 35000, new DateTime(2005, 12, 12)));
    businessObjects.Add(new Employee("Adams", "Ellen", 55000, new DateTime(2004, 1, 11)));
    businessObjects.Add(new Employee("Hanif", "Kerim", 45000, new DateTime(2003, 12, 4)));
    businessObjects.Add(new Employee("Akers", "Kim", 35600, new DateTime(2002, 12, 12)));
    businessObjects.Add(new Employee("Harris", "Phyllis", 60000, new DateTime(2004, 10, 10)));
    businessObjects.Add(new Employee("Andersen", "Elizabeth", 65000, new DateTime(2006, 4, 4)));
    businessObjects.Add(new Employee("Alberts", "Amy", 35000, new DateTime(2007, 2, 1)));
    businessObjects.Add(new Employee("Hamlin", "Jay", 38000, new DateTime(2004, 8, 8)));
    businessObjects.Add(new Employee("Hee", "Gordon", 50000, new DateTime(2004, 10, 12)));
    businessObjects.Add(new Employee("Penor", "Lori", 40000, new DateTime(2006, 12, 20)));
    businessObjects.Add(new Employee("Pfeiffer", "Michael", 49000, new DateTime(2002, 1, 29)));
    businessObjects.Add(new Employee("Perry", "Brian", 54000, new DateTime(2005, 9, 25)));
    businessObjects.Add(new Employee("Philips", "Carol", 65000, new DateTime(2005, 8, 14)));
    businessObjects.Add(new Employee("Pica", "Guido", 52000, new DateTime(2006, 6, 23)));
    businessObjects.Add(new Employee("Jean", "Virginie", 38000, new DateTime(2002, 5, 2)));
    businessObjects.Add(new Employee("Reiter", "Tsvi", 39000, new DateTime(2003, 4, 12)));

    bindingSource1.DataSource = businessObjects;
    dataGridView1.DataSource = bindingSource1;
    PropertyDescriptorCollection objectProps = TypeDescriptor.GetProperties(businessObjects[0]);
    BindingSource bindingSource2 = new BindingSource();

    foreach (PropertyDescriptor prop in objectProps)
    {
         if (prop.PropertyType.GetInterface("IComparable", true) != null)
             bindingSource2.Add(prop.Name);
    }

    comboBox1.DataSource = bindingSource2;
    comboBox2.Items.AddRange(new string[] { "<", "=", ">" });
    comboBox2.SelectedIndex = 0;
}
```

## Setting the Filter and Removing It

The final step is to handle the `Click` events for the Apply Filter and Remove Filter button controls. To set the filter, I built a filter string out of the values selected for the combo boxes and the text value entered for the text box. Setting the `Filter` property on the binding source sets the filter on the underlying list. To remove the filter, I simply call the `RemoveFilter` method exposed by the `BindingSource`. This method in turn, calls the `RemoveFilter` method on the underlying list.
```csharp
private void button1_Click(object sender, EventArgs e)
{
    try
    {
        bindingSource1.Filter = GetFilterString();
    }
    catch (ArgumentException ex)
    {
        MessageBox.Show(ex.Message + " Try another filter expression.");
    }
}

public string GetFilterString()
{
    StringBuilder query = new StringBuilder();
    if (comboBox1.SelectedItem != null)
        query.Append((string)comboBox1.SelectedItem);
    else
        query.Append(comboBox1.Text);
    if (comboBox2.SelectedItem != null)
        query.Append((string)comboBox2.SelectedItem);
    else
        query.Append(comboBox2.Text);

    query.Append("'" + textBox1.Text + "'");
    return query.ToString();
}

private void button2_Click(object sender, EventArgs e)
{
    bindingSource1.RemoveFilter();
}
```

I added the following two lines to the end of the form`s constructor to associate the events with their event-handling methods.
```csharp
this.button1.Click += new System.EventHandler(this.button1_Click);
this.button2.Click += new System.EventHandler(this.button2_Click);
```

## Running the Application

Now I compile and run the application. When the application is running, I can select values in the combo boxes, enter a filter value in the text box, and click the Apply Filter button to see the `DataGridView` refresh with the filtered contents. In addition, I can sort the grid contents before or after I apply a filter. The following illustration shows a list sorted by last name and a filter applied. The filter is _LastName > Ham_.

![BindingFiltering02.jpg](data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/2wBDAAgGBgcGBQgHBwcJCQgKDBQNDAsLDBkSEw8UHRofHh0aHBwgJC4nICIsIxwcKDcpLDAxNDQ0Hyc5PTgyPC4zNDL/2wBDAQkJCQwLDBgNDRgyIRwhMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjL/wAARCAEsAioDASIAAhEBAxEB/8QAHwAAAQUBAQEBAQEAAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEII0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxMXGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6erx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYkNOEl8RcYGRomJygpKjU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADAMBAAIRAxEAPwD0XQPD+n3Ok2+oajawX11eRLOzXCCRUVhuVVVuBgEc4znNXzo3htSQdK0cEcEG1i4/SpfDv/IraN/14W//AKLWuY1XxPMuo3WirbRMhmkiDCQmQkR+bnbjGO3Wqq1I0oXYqVKdaTUemr9Do10bw4zBV0rRyTwALWL/AAqb/hHNE/6Ammf+Acf/AMTWCt7bXWs6ctrZNFi5BdvsrR4Xa3UlR3xWv4zklg8JXskDTrJmMD7PKY5DmRQQrAjBIOM5HXrXPhcX7em5uNrO39aIqrS5JKN7lj/hHNE/6Aumf+Acf/xNJ/wjui/9ATTP/AOP/wCJrMtb+8sPs2l6Xol4LydZLhotZ1QsUjUqpPmAzk5LDCjjqTjukPjG4u3jFppAZVs2urlpbkJ5JR2RkGFO5tyHBHBwTkcZ6pTjFXf9f1ZmSTZqf8I7ov8A0BdM/wDAOP8A+JpD4d0X/oC6Z/4Bx/8AxNYH/Cc6qtnLcSeGlQJp66mP9PBAgwSQ3yZEnHCgFT3YVoyG4Hj/AE511G7ezutOuJBaMVESFWhwQAASfmPLFsZOMZqlq7f11/yBK6v/AF0/zLZ8PaL/ANAXTP8AwCi/+Jph0DRP+gJpf/gFF/8AE1tOBVdqpWZLuZn9g6L/ANATS/8AwCi/+JpP7B0X/oCaX/4BRf8AxNaBpKdkK7Ki+H9DPXQ9L/8AAKL/AOJp3/CPaH/0A9L/APAKP/4mriHmpc0WQXZnf8I9of8A0A9L/wDAKP8A+Jo/4R7Q/wDoB6X/AOAUf/xNaNFOyFdmd/wj2h/9APS//AKP/wCJo/4R7Q/+gJpf/gFH/wDE1o0maOVBdmd/wj+h/wDQD0v/AMAo/wD4mk/4R/Q/+gJpf/gFF/8AE1o0UcqFdmd/wj+h/wDQE0v/AMAov/iaT+wND/6Aml/+AUX/AMTWiTSU+VBzMz/7A0T/AKAml/8AgFF/8TSf2Bon/QE0v/wCi/8Aia0aSnyrsLmZn/2Bon/QE0v/AMAov/iaT+wdE/6Aml/+AUX/AMTWhRRyrsHMzOOg6L/0BNL/APAKL/4mk/sLRcf8gXS//AKL/wCJrRNJT5V2FzPuZv8AYejf9AXS/wDwCi/+JpP7D0b/AKAul/8AgFF/8TWiabRyrsHM+5Q/sTRv+gLpf/gFF/8AE0f2Jo3/AEBdL/8AAKL/AOJq9RT5V2Dmfco/2Ho3/QF0v/wCi/8AiaT+xNG/6Aul/wDgFF/8TV+g0cq7BzPuUP7E0f8A6Aul/wDgFF/8TTTouj/9AbS//AKL/wCJrQppo5V2Dmfczm0bSB/zBtL/APAGL/4mmnSNJ/6A2l/+AMX/AMTWgwphFLlXYOZ9xkeh6K6AnRNL/wDAKL/4mn/2Don/AEBNL/8AAKL/AOJqa3cfdq1RyrsPmZn/ANg6J/0BNL/8Aov/AIml/sDRP+gJpf8A4BRf/E1fpRS5V2DmZQ/sDRP+gJpf/gFF/wDE0f2Bon/QE0v/AMAov/ia0KKXKuwXZmnQdEH/ADBNL/8AAKL/AOJo/sHRP+gJpf8A4BRf/E1oNSU+Vdg5mUP7B0T/AKAml/8AgFF/8TS/2Bon/QE0v/wCi/8AiavUtHKuwczKH9gaJ/0BNL/8Aov/AIml/sDRP+gJpf8A4BRf/E1foo5V2DmZR/4R/Q/+gJpf/gFF/wDE0h0DRB/zBNL/APAKL/4mtDNKeRS5UO7Mz+wdE/6Aml/+AUX/AMTR/YOif9ATS/8AwCi/+JrQopcqC7M7+wdF/wCgJpf/AIBRf/E0f2Fov/QE0v8A8Aov/ia0KSiyHdmJqeh6MtmhXR9NU/arYErZxg4M8YI4XoQSPxrR/sHw/wD9AXS//AOP/wCJqHWjjS2bOAs9uxPoBPGSfyFY92t2HtyyXM8KykzRQzGNmXawGDuX+Iqeo6fhUOOpaZvf2F4f/wCgLpf/AIBx/wDxNVtR0PQU0q8ZNG0xXWCQqVtIwQQp5BxWT5lv/wBATWf/AAYv/wDJFRst6mgSLcyO0ws2EhLZy3lnP61PKO5u6jpOg2VsZI/D+lSyNLHEiNaxqCzuEGTtOBlhng1m+RpH2v7J/Y/gv7Tv8vyftab9+cbdvkZznjFO1RbuQHy1eUx3McvlhhlgkqsQMkDOB3qh5U/nbvsut+V9p+0+Rvtdm7zPMx1zjd70OI0zoNO0nQby28yTw/pMUiyyROi2sbAMjshwdoyMqccCuZs4dJWwtAdL0xibeIktZREklASSSuTWtpiXaJmRXiMlxLL5ZYZUPIzAHBIzg9q88WC+aC0ZZ5QDawdGP/PJa58TJwSsd+ApRqOSkdsIdIP/ADCdK/8AAGH/AOJqOys11C3gmtfDOizrM5i/dwR5gcckTAxjZgZORu6cZ3Lnk0tr/wD57y/99GtXTLxtOspIG06e6a8G3UHlkC7o8EBI8N23Hk4z7ZG3npVeZvnf4nZisMoRXsld+hq239lTiXdpGinZIUDw2kbxvjHKsYxkZyM47cZGDUhi0gf8wnSv/AGH/wCJrkza3YZ1hlulhBxEJSA+3tnaSM9uPTt0qF7bUCf9fN/30ah1pczS/M2jhKTppu1/Sx2+m+HdJ1e11ZTZQQyC6Cxy28axtH/o8J42jpkk46ZJ45rzC48Tz2dzLavPuaFzGSAeSDj+leseAo5otO1JZixb7avLd8W0AP6g18+a25/t7Uf+vqX/ANCNejFXpps+fqvlruK21/Q+m/Dn/Iq6N/14W/8A6LWqV54atrm4mu47OzS+cPi7K5kBPAOcZ4XjrUulXJs/AdjdKoZoNKjkCnoSsIOP0rmdI8Q/EHW9ItNUstB0E211EssZa7lBwRnkY61baVrkLm1sdBpGhX2n3c009zBKJdmFjQoE2tn3zmt28trfULVra6j8yFipK7iMkEEcj3Arj/tnxL/6AHh//wADZP8ACj7Z8S/+gB4f/wDA2T/CnOopu8gjBxVkdPqmj2GsNA92kwlgz5U1vcSQSKDwQHjZWweMjODgegpING0y2jEcFosaC1FmFViB5Qz8vX3PPX3rmftnxL/6AHh//wADZP8ACj7X8S/+gB4f/wDA2T/CofI9x2kdK+i6ZJBJA1tmOSz+wsPMbmDBGzr7nnr71BN4d0ybVoNUb7aLuABYil/OqKo2/LsD7cHauRjBxzmueivviTMhaPQNAZQxXP2yXqCQf4fUGn/aPiX/ANC9oH/gZL/hTUo3uFpJW/r+tEdk0maiY5rkftHxL/6F7QP/AAMl/wAKPP8AiV/0L2gf+Bkv+FNTiLkZ1dIa5TzviV/0L2gf+Bsv+FHnfEr/AKF7QP8AwNl/wp+0iLkZ1YODUynIrjvN+JX/AEL2gf8AgbL/AIUon+JY6eHtA/8AAyX/AAo9pEORnZUma4/7T8TP+he0D/wMl/wo+0/Ez/oXtA/8DJf8KftIhyM6+iuQ+0/Ev/oXtA/8DJf8KPtHxL/6F7QP/AyX/Cj2kRezkdfSZrkPtHxL/wChf0D/AMDJf8KPP+Jf/QvaB/4GS/4Ue0iHs5HXUlcl5/xL/wChe0D/AMDJf8KPP+Jf/QvaB/4GS/4U/axD2cjraSuT8/4l/wDQvaB/4GS/4Uef8S/+he0D/wADJf8ACj2sRezkdZSVyfnfEr/oXtA/8DZf8KPO+JX/AEL2gf8AgZL/AIU/axD2cjrKSuU874lf9C9oH/gZL/hR53xK/wChe0D/AMDZf8KPaxD2UjqjTa5fzfiV/wBC9oH/AIGy/wCFJ5nxJ/6F7QP/AANl/wAKPaxD2UjqKSuY8z4k/wDQvaB/4Gy/4UnmfEk/8y9oH/gbL/hR7aIeykdRRXMb/iT/ANC9oH/gbL/hRv8AiT/0L2gf+Bsv+FHtoh7KR01Ia5nf8Sf+he0D/wADZf8ACjd8Sf8AoXtB/wDA2X/Cj20Q9lI6Qio2Fc/u+JH/AEL2g/8AgbL/AIUh/wCFjn/mXtB/8DZf8KPbRD2UjoFO1gavI25Qa5Db8R/+he0H/wADZf8ACnq/xJQYHh7Qcf8AX7L/AIUe1iHs5HXUVyfn/Ev/AKF7QP8AwMl/wo8/4l/9C9oH/gZL/hS9rEPZyOtorkvP+Jf/AEL2gf8AgZL/AIUef8S/+hf0D/wNl/wo9rEPZyOuNMNcp5/xL/6F7QP/AAMl/wAKPO+JR/5l7QP/AAMl/wAKFViHs5HV0VyfnfEr/oXtA/8AA2X/AAo874lf9C9oH/gbL/hT9rEPZyOtorkvO+JX/QvaB/4GS/4Uvn/Ev/oXtA/8DJf8KXtYh7OR1opa5Hz/AIl/9C9oH/gZL/hR5/xL/wChf0D/AMDJf8KPaxH7OR1pptcp5/xLP/MvaB/4GS/4UnnfEr/oXtA/8DJf8KXtIh7OR1hpK5TzviV/0L2gf+Bsv+FHnfEr/oXtA/8AA2X/AAo9pEOSR1LoksbRyIrowKsrDIYHqCO4qkNIswAF+1Io4CpfTqo+gD4FcrPrvjm1naGfSPDySL95TdzHHfstR/8ACS+NP+gZ4d/8Cp//AIik5xZSjJHX/wBk2v8Afvf/AAYXH/xymvo1lIjI5vGRhhla/uCCPQ/PXJf8JL40/wCgZ4e/8Cp//iKP+El8af8AQM8Pf+BU/wD8RS5ohyyOxXSbQnl77/wY3H/xypho1mf477/wY3H/AMXXEf8ACS+NB/zDPDv/AIFT/wDxFPHinxsP+YZ4d/8AAmf/AOIo5ohyyO1/sWz/AL99/wCDG4/+LqmnhHR40VES+VFAVVGpXIAA4AA8zpXLf8JV43/6Bnh3/wACZ/8A4il/4Svxv/0DPDv/AIEz/wDxFS3B7lLnWzOrHhXSh2v/APwZ3P8A8cpf+EX0v0v/APwZ3P8A8crk/wDhK/G//QM8O/8AgTP/APEUf8JX43/6Bnh3/wACZ/8A4ilan2HzVe7+86s+FtKPa/8A/Bnc/wDxyk/4RTSfS/8A/Bnc/wDxys4TfEMkj7D4W4OP+Pu4/wDjdL5nxE/58fCv/gXcf/G6dqfYOar3OktbW3srZLe1iWKFM4Rfc5J9yTkknkk5r5V1v/kPaj/19S/+hGvpLwlrd5rul3UuoW9vBdWt9PZyLbszITG20kFucE5r5t1z/kP6l/19S/8AoZrR/Bp/W5hr7VX7P9D6Otv+SbQ/9gVf/RFP+Gf/ACTXw/8A9eUf8qZbf8k2h/7Aq/8Aoin/AAz/AOSa+H/+vKP+VZz6G0ep1dFFFQUFFFFAEGmMBZsCf+W83/o1qr6fqzXFxrCzKBHY3PlJ5aMWZfKR+QMknLHoPTiksZ40gdWkUETzcFv+mjVhXGj6k1zqy2uvWdvZak5eRRasZ4yY1jJSUSgA/KCDsOD60v8AL/IaNT/hNNEEDytJeoyOqeS+nXCzEsCRtiMe9hhWOQpHyt6Gsm58eNd6RqF1oNqLiW2vbe3h89JEjuVlMZyrFQMkOQMZxwTwRWRF4BSKG6VLzQIjM0LCKDRlSD92HGWTzS287871dWBUc9RWrZ+GntraW2l1tbiJ7q1ug8sZMu+IRhtzlzu3eWOcAjJyWppK/wB35q/6iuaUHi+3kuriQpM9gtrbTxG3tJZpcyGQEFUDHjYO3Bzmnv478OxmMNeTZdVbAs5iV3MyKGwnyMWVl2tg7hjGeKwtR8GQXd7qFxFqVt5V3NFN9jvLfz7fKiTcrJvXepaQvjIwwB56UaV4Ni02zEH9p2x/eQSYgthFGvl3DzbVXedoO/aBnjHfpRuN6bHQt4z0JIoJTczlJlLZFnMfKAYqTLhP3QBBGX2/dPoa3fMX1rzm88AWF1c/aDcaXLLJ5iyve6clyVVpXkHlbmwjDzGGSGBwCV4xXcrPAihRJGABgYIo6A99C7vX1o3r61T+0xf89U/76FH2mL/nqn/fQoEXN6+tG9fWqf2mL/nqn/fQo+0xf89U/wC+hQBbZ1yvPf8AoaxfFetSaJoTXkFzaWzefDEZ7xS0USvIqlmAZeACT94VeNzF8v71Pvf3h6GqWq28Op20UP2pI/LuIZ88HPlyK+OvfbjPvStqvkO5nWPjSwhjVb/WLDUWk2+TPpNtK4lLeZ8qovmZwIm5DHochcDL18f6Ncalp9lZG5uGu5hHu+yTLtUo7LIMp8yMUIDD5cZOcKaS50aK48Tw6z/aCL5ez91tBztSZeuf+m2en8Pvxkab4Ql028trmLWrMeRcrKIo7Py4mXa6sTGsm0SsJOXUKCVGUNPqJ7aFyz8aXk1vcrcW9vHdR6mlvGAGKyW7XJhD9eGGGB5xkA9DitaTxpocBlFxNeW7RkDbPp9xGXy4QbAyDf8AMyj5c/eHqKyrjwvbz21hGNTEctpqDXgkVR86NMZTERnpnbz6qD7Vk2PgGO01GK6bU9OzFszJDYeXNcFZo5d00nmHzHPl43YHLE47UR6J9/8AIb6tf1qdXB420C4SV47uYCKMyNvtJkJwwUqoZAWYMQpQZYEgYya0tN1ey1a3ae0eTarlHSaF4nRhzhkcBlOCDyOhB6GuPv8AwXZ6jafZ7i/hdQJyqyQhkLSXCzruUthlBQAr/ECeRWv4b0q28P2MsCvpqtLKZGWws0tYhwBgICT0HVmY5J5AwALbUH5HS719aN6+tU/tMX/PVP8AvoUfaYv+eqf99CgRc3r60b19ap/aYv8Anqn/AH0KPtMX/PVP++hQBc3r60iOuDz3P86qfaYv+eqf99CkFzFz+9T7x/iHqaQGFP4hvj4tbTEvdItVSWNY7K8V1nvIyoZ5In3AcZYbQjcoclc8TXvjG0VkgsxOt19phjaO8sp4NyNMsbMm9V3Y3dRkDIz1qG/0281G9VZ9atzpi3MdyIDbDzlZCGVVlDgBdyg8oTgkZ5yMCx8Ax2moRXTanp2Y9mZILDy55ys0cm6aTzD5jny8bsDlicdqcelxvrY6uTxpocBlFxNeW7RkDbPp9xGXy4QbAyDf8zKPlz94eoq5Y+I9L1EWv2W4ZzdCUxK0Lo37ttr7lYArtYgHdjk4riLHwDHaajFdNqenZi2ZkhsPLmuCs0cu6aTzD5jny8bsDlicdq2tBsfI17WdVnUwLcTbbaGWRGKJgb3G0nAdxuxnPAJwTgCWmoPfQ6/evrRvX1qn9pi/56p/30KPtMX/AD1T/voUCLm9fWjevrVP7TF/z1T/AL6FH2mL/nqn/fQoAub19aN6+tU/tMX/AD1T/voUfaYv+eqf99CgC5vX1pGdcrz3/oaqfaYv+eqf99CkNzF8v71Pvf3h6GkBB4h1SbStEnvLVEaRCo3SKzJEpYBpGC8lVBLEDGQOo6jG0jxgZdQt7C6ubPUxcQzXEWoaTDK0LpGVGCBvAbJYY3n7q92AGvqDzT2hWx1GK0uAwZZHQSrwejLkEg9OCD6EVzS+F7g3c94+vQpdXkdxHdyQW5j/ANYkaq0X7wmMr5SHJL5OemeBbjNqTxzoUcJZpL1JdwRYJNOuEmZirMMRmPeQQjcgY+U+lUYfEmqa7/Yp0eS0sFv7CS8k+3WUsxUqYxtA3xHHzn5u+ARwazNG8Ew6Xqi3pvNKiAZW8nTtPFqh2xypkjzGyx83JP8AsgVbg0PU7KHSDZ67pq3On2j2ZeawZ0kQlMHaJgVICDPJB54HSnZX/rs/1sK7NSDxlZRWEcupR3MEoeSKYw2k00MbRuUYmRUKquVJyxHByaSz8c6ZPpgvJ47yItcTQLCllPLI3lsVLBFj3bcAEnGATgnNc5rHgKDUoREmo2D5hZHe/shcskjOzvLF86iN2LnJwfur6VLeeCIbyOIzXekXMlvPcvAL7ThcRLHM+8qyGQZcMOHBHGRt5pD0v5HfQXUFzbxzwSrJDKodHU5DKRkEVJvX1rK06O207TbayjliKQRLGCqogOBj7qgKPoABVn7TF/z1T/voU2lfQSvbUub19aN6+tU/tMX/AD1T/voUfaYv+eqf99CgC5vX1o3r61T+0xf89U/76FH2mL/nqn/fQoA4zxAc67dfUf8AoIrMrQ1xg2tXLAggsMEfQVn0hhRRRQAUUUUAFFFFABRRRQB3d3eW+n2tzeXcoigiyzuQTgZ7AcknoAOSTgVS8Pa+mvWjTGzuLKQMf3NwBuKZ+VuOORjI7Hjngm69/ZwTSJLdwRuGOVaQA/zpV1Oxdgq31uxJ4AlU5/WgDmPAH/Hhrv8A2H7/AP8ARpr571z/AJGDUv8Ar6l/9DNfQngD/jw13/sP3/8A6NNfPeuf8jBqX/X1L/6Ga3/5d/d+pzP+MvR/ofR1t/yTWH/sCr/6Ipvw1sIZ/hxoEjtOGNnH9y4kUdB2BAp9sP8Ai2kP/YFX/wBEVP8ADD/kmnh//rzj/lWc+htDqbz6ZbrjD3X/AIFS/wDxVM/s6D+/c/8AgVL/APFV5t8EvG+u+KtCns9YtZ510/EceqsRib/pm5Jy0gGDuGcgjdg4L+qVJRUOnQf37n/wKl/+KqGayiSF2WS5BCkg/apf/iq0DUFz/qJP90/yoA8z1bVtRsdYvba2vrmOGOdwqiU8cmqf/CQax/0E7r/v6a0ZQh8ZakJGZU/0ncVGSBsboMjP50kVpYz6ZbBZZpLWOSZ5JJMQleIx28zIyR0Gea+nowoKlDmpptpdF29PI+dqyrOpPlm1q+vn/wAEz/8AhINY/wCgndf9/TR/wkGsf9BO6/7+mtKTRdNiuLaItcyfaphHGyyABQURgTlMn7/oM+1N1dY7TQra3i80GRw0hDAKx8qM8qFGfvcc+p5zWijhpNKNNa+SIbxCTcqj082Z/wDwkGsf9BO6/wC/po/4SDWP+gndf9/TWbRXT9VofyL7kc/1mt/O/vZpf8JBrH/QTuv+/po/4SDWP+gndf8Af01m0UfVaH8i+5B9Zrfzv72aX/CQax/0E7r/AL+mj/hINY/6Cd1/39NZtFH1Wh/IvuQfWa387+9ml/wkGsf9BO6/7+mj/hINY/6Cd1/39NZtFH1Wh/IvuQfWa387+9nV+FtSv9S1+C1u766khZWJXzmGcKSOQc16F/Zdv/fuv/AuX/4qvNPBH/I1W3+6/wD6CatXbTv4t1GHVLeNvDLaqqzYfJluDBD5SyLjHlZGOvLsmeOvzWZwhDEuMVZWPoMunKeH5pO7ueg/2Xb/AN+6/wDAuX/4qj+y7f8Av3X/AIFy/wDxVcBZ+OfE91BbzNptpbx6h5TWbTrGNivNGhyq3DPJgScnbHggAj5uHWnijxIA8MT6YfIuI4ZWkhmYyPLdyw5XMpKqNgbblv7owMEcFn+h3PQ73+y7f+/df+Bcv/xVH9l2/wDfuv8AwLl/+Kqv4ev7nUtHWe88o3CzTQyNChVGMcjJkKSSM7c4ycZ61qUgKf8AZdv/AH7r/wAC5f8A4qj+y7f+/df+Bcv/AMVVyigCn/Zdv/fuv/AuX/4qj+y7f+/df+Bcv/xVXKKAKf8AZdv/AH7r/wAC5f8A4qj+y7f+/df+Bcv/AMVVyigCn/Zdv/fuv/AuX/4qq1jYRTW7NJLdEiaVQftUvQSMB/F6AVq1zWq6jcab4ame1iuWnlu5YVe3tnnaHdKwMmxFYnaMnGOSAOM5oYGz/Zdv/fuv/AuX/wCKo/su3/v3X/gXL/8AFV5bbabbap4c0i7UaTNbaTZXjtba7ZvI7gSKdwVypXhMGQ5wT0PNaDpFK765DZRxa0dVWKJyg81IzbgiLdjO3aScdM80r6f13sH9fhc9C/su3/v3X/gXL/8AFUf2Xb/37r/wLl/+KrzRNN02BNJtLW1gWx1G0099QVYwFuC0ww0gx8xckgk5z3rt/CUMVrY31rbRpFZwahPHbxooVUQNyqgcABiwwPSqtv8AP8Gl+or/AKfir/oan9l2/wDfuv8AwLl/+Ko/su3/AL91/wCBcv8A8VVyikMp/wBl2/8Afuv/AALl/wDiqP7Lt/791/4Fy/8AxVXKKAKf9l2/9+6/8C5f/iqP7Lt/791/4Fy//FVcooAp/wBl2/8Afuv/AALl/wDiqrXNhFHcWarLdASTFXH2qXkeW5/veoFatU7z/j60/wD6+D/6KkoAP7Lt/wC/df8AgXL/APFUf2Xb/wB+6/8AAuX/AOKrDvbOCP4hWFyZJwZ9MuxJuuX2KA9vyqltqdeSoBPeszRo7LTPtut6Fp0Ntps8MdvZxoNp1CbcQsreoJYAOfmYbmPGCTt/XdfiDOv/ALLt/wC/df8AgXL/APFUf2Xb/wB+6/8AAuX/AOKrgNbuDp2t6Jpsy6j5FleQStLHYzMl3cSMS7l1Qrxknbnq54G0Vu6n/ZWoR3uqa3GJtMiYWltAQWExDjcNg+/vkCqFOQdg9aOl/P8Ay/zDrb+up0X9l2/9+6/8C5f/AIqj+y7f+/df+Bcv/wAVXncnh+2jie01iwtYrePTb29tbMqrpZEupwnUBkBXleAWbbxUllDC88GsXkUS6z/a0UElywAkWI267k39QuwlsZxnmha/152/MTdv68r/AJHoH9l2/wDfuv8AwLl/+Ko/su3/AL91/wCBcv8A8VXN+GbKy0rxRf2un29rbWU9lBNbraPvSZQzjzpG4zI2QM85Cg7m6L2FHmPrYp/2Xb/37r/wLl/+Ko/su3/v3X/gXL/8VVyigCn/AGXb/wB+6/8AAuX/AOKo/su3/v3X/gXL/wDFVcooA801HP8Aad2pZ22zOoLsWOASByeTwKrVZ1H/AJCt7/18Sf8AoRqtQAUUUUAFFFFABRRRQAUUUUAd5ZMRaSEdftE//o1qw9VnlNtMu848t+/+ya27P/j0k/6+J/8A0a1YGp/6ib/rm/8A6CaaAg+H/wDx4a7/ANh+/wD/AEaa+e9c/wCRg1L/AK+pf/QzX0L8Pv8Ajw13/sP3/wD6NNfPWuf8jBqX/X1L/wChmtv+Xf3fqc7/AIy9H+h9IW3/ACTOH/sCr/6IqT4Yso+Gnh/JA/0OPqfam2w/4tjD/wBgVf8A0RUnww/5Jp4f/wCvOP8AlWcuhtE3NO03TtE0u20zTIIrazt02RRIeFH48kk5JJ5JJJ5NWd6f31/76FSydqjyfWpGNLp/fX/voVBcMhgkw6fdP8Q9Kskn1qvck+RJyfun+VAHleuCdfEWoPEJBmeQblzyCSCKpwzX1sVMElzEVJKmNmXGeuMeuBXrdteW9jp/mXMmxHvHiU4Jyzzsqjj1JAq6l5byX01ksmbiFEkkTB4ViwU56c7W/KvVp5rKFNQ5E7K33HmzyyMpufM1d3PGJLjUJZVlkmunkVt4dmYkNxzn14H5Co5Gu5lVZTO6r0DZIHAH8gB+Ar3Ko4Z4riPfDKkqbiu5GDDIJBHHcEEH3FWs4ktoIh5VF7zZ4b5Mv/PN/wDvk0eTL/zzf/vk17tRT/tqp/Khf2RD+ZnhPky/883/AO+TR5Mv/PN/++TXu1FH9tVP5UH9kQ/mZ4T5Mv8Azzf/AL5NHky/883/AO+TXu1FH9tVP5UH9kQ/mZ4T5Mv/ADzf/vk0eTL/AM83/wC+TXu1FH9tVP5UH9kQ/mZ5P4PYWviW3luCIowr5eT5QPlPc16HLLoFxBcQTSaZJDcnM8btGVlOAPmB+9wAOfQVZvP+PrT/APr4P/oqSqMvivSINTk0+SW5E0cqwu/2KYwq7AEKZdnlgkMvG7qQOprzcViXiKnO1Y9DDUFQp8idxsNv4Vtp554IdGimuJFlmkRYlaR1O4MxHUg8gnvzUqt4dQsVOlgswdiDHywYuCfcMSwPqSetaNpdwX9lBd2z74J41kjfBG5SMg4PI4qnda/pdjffYrq8SGfaGIcEKAVdgS2MAYjc8nt7isNVobj4b/SLePZDd2MSbi21JEUZJJJ4PUkkn3NP/tbTf+ghaf8Af5f8auUUgKf9rab/ANBC0/7/AC/40f2tpv8A0ELT/v8AL/jVyigCn/a2m/8AQQtP+/y/40f2tpv/AEELT/v8v+NXKKAKf9rab/0ELT/v8v8AjR/a2m/9BC0/7/L/AI1cooAp/wBrab/0ELT/AL/L/jVXT9T09LZw99bKfPmODKo4MjEHr6VrVkrqNppOjXF7ey+VbxTzbm2ljkzMAAACSSSAAASSQBQBDcweFryKCO6i0aeO3cyQrKsTCJic7lB6HPcU8r4aOqjVSNJ/tELsF3iPztuMY39cY96il8ZaHDaxXD3Fxsk35VbKZpI9hAcyIE3RhcjJcDGRVn/hJNK/tYaZ58n2gtt3CCTyt2zft83bs3bedu7OO1AFWKy8Iw211bRWuhxwXZzcxJHEFmP+2OjfjVyzutD0+0jtLKfTra2jGEhhdERR14UcCqsPjHRJ7We5S4uPLhCMd1nMrSBztQxqUzIGPAKA5rT07UbbVbNbu0d2iYsv7yNo2VlJBDKwDKQQeCAaAG/2tpv/AEELT/v8v+NH9rab/wBBC0/7/L/jVyigCn/a2m/9BC0/7/L/AI0f2tpv/QQtP+/y/wCNXKKAKf8Aa2m/9BC0/wC/y/40f2tpv/QQtP8Av8v+NXKKAKf9rab/ANBC0/7/AC/41Vu9T09rmxK31sQs5LESrwPLcZPPqR+da1U7z/j60/8A6+D/AOipKAIZrvRLh981xp8jeW0W53QnY2Ny89jgZHfArPsdN8G6XOJ9PstBtJgciS3ihjYHBGcgDsSPxNaf9t6WdZOji+gbUViM72yvl0QbfmYD7v3lxnGe2cGotJ8Q6brbSLYyzMY1Vz5tvJFuRs7XXeo3qcHDLke9C8gfmSS3+kThBNd2MgRg675EO1h0IyevvVW5j8M3unJp10mkT2MYUJbSiNol29MKeBjtUv8AwkmjG+tbH+0Yftd27pBBn55Cm7cQvXaNjfN0468ils/EWmX+pyafbzStcR7/AL1vIiPsba+x2UK+CcHaTigNigdL8Fm0htDYaB9mgcyRQ+TDsjc9WVcYB9xVor4aOqjVSNJ/tELsF3iPztuMY39cY96S/wDFejaZNJFc3EpkiJ8xYbaWYxgBSWbYpwoDDLHgZxnNSDxLpJ1UaaLhzOTtDiCQxbtm/b5u3Zu287d2cdqLgNsB4b0rzv7O/sqz8598v2fy4/Mb1bbjJ9zVz+1tN/6CFp/3+X/Gq+leItM1oyiymlPlqsh823kh3I2cOu9RuU4PzLke9WtO1G01bT4b+wmE9rMN0ciggMM4yM/SgBv9rab/ANBC0/7/AC/40f2tpv8A0ELT/v8AL/jVyigCn/a2m/8AQQtP+/y/40f2tpv/AEELT/v8v+NXKKAPM791k1K7dGDI08hVlOQRuPIqvVnUf+Qre/8AXxJ/6EarUAFFFFABRRRQAUUUUAFFFFAHd2f/AB6Sf9fE/wD6NasDU/8AUTf9c3/9BNb9n/x6Sf8AXxP/AOjWrA1P/UTf9c3/APQTTQEXw9H+ga9/2H7/AP8ARpr551z/AJGDUv8Ar6l/9DNfRHw7GbDXv+w/f/8Ao018765/yMGpf9fUv/oZrVfB936nO/4q9H+h9KWw/wCLXw/9gRf/AERTvhh/yTTw/wD9ecf8qhtph/wrOFMf8wVef+2FSfDEkfDTw/8AKT/ocfT6VnJG0TrJO1R06Rjx8jfp/jTMn+436f40hgaguf8AUSf7p/lUxJ/uN+n+NQXB/cSfK33T6en1oAxdbs7i+8NLBatOkp1VD5lugZ4wLvJYAgjgAnkEcc1ga1p2raffaipu9Xvo7hbRDfmGUvCoM5OEshE8gB2ggHjzAScDFdpYXM0cEirY3EoFxN86NHg/vG9WB/SrP2yf/oGXf/fUX/xdLpb+un+RVzzTTrDX9Rs43u7nxApje1jiKyXNtujN3KrsULbv9VszvLEDBJzzU8cOt22raUm/xA6RXDIkTNcFWQXUnzNNuZT+725Ey/MuNjg5NeifbJ/+gZd/99Rf/F0fbJ/+gZd/99Rf/F076387k9LFyiqf2yf/AKBl3/31F/8AF0fbJ/8AoGXf/fUX/wAXSGXKKp/bJ/8AoGXf/fUX/wAXR9sn/wCgZd/99Rf/ABdAFyiqf2yf/oGXf/fUX/xdH2yf/oGXf/fUX/xdAFyiqf2yf/oGXf8A31F/8XR9sn/6Bl3/AN9Rf/F0AF5/x9af/wBfB/8ARUlczF4dnvtX1ye6vL9LQ6gk0dkEjEMxSKIq+Sm84dR0cDK4x1rcu7uY3NiTp9yMTkgFo/m/dvwPn/H8KtfbJ/8AoGXf/fUX/wAXSaHc88TTNS0PQNOuoLjVojbaOL25Wa6l8vzYTEwiKsdkYK+YpUBQRyemRS1LTtfNzJqKQak13cW/2kOiuWhZorwrGpHQpvjTA747mvSbvZfweReaHJcw7g3lzCF13A5BwX6ggEVP9sn/AOgZd/8AfUX/AMXVX/r+v60F/X9f11OT8ODU18baj9obVpIGExY3KyxxR/ONiqGLRP8ALna0RU4zvXPI7iqf2yf/AKBl3/31F/8AF0fbJ/8AoGXf/fUX/wAXS6JB1LlFU/tk/wD0DLv/AL6i/wDi6Ptk/wD0DLv/AL6i/wDi6ALlFU/tk/8A0DLv/vqL/wCLo+2T/wDQMu/++ov/AIugC5RVP7ZP/wBAy7/76i/+Lo+2T/8AQMu/++ov/i6ALlcl4gs57vwyDA1wpt9T+0ObaLzZQq3BJZEwdzAfMBg5x0boeh+2T/8AQMu/++ov/i6q6fdzLbOBp9y37+Y5DR/89G45ft0oA403epJG95qFpql8k9jd2dtI2nP50nzgx+bGifuyw7lVHygkDNCaderEPD5trv7ab9Lj7T9nk8ryxAAW83GzOQUxuz7YrvPtk/8A0DLv/vqL/wCLo+2T/wDQMu/++ov/AIulZWt/W9/zDz/ra35Hn6i4uRpt6mmahHFpVtZR3UTWMqsWWQFgilQZNgGcoGB7V2HhdXayvbpoZoUu72aeKOaJo3CE4BKMAVzjOCAea0Ptk/8A0DLv/vqL/wCLo+2T/wDQMu/++ov/AIuqvv8AP8Wm/wAhW/T8FZfmXKKp/bJ/+gZd/wDfUX/xdH2yf/oGXf8A31F/8XSGXKKp/bJ/+gZd/wDfUX/xdH2yf/oGXf8A31F/8XQBcoqn9sn/AOgZd/8AfUX/AMXR9sn/AOgZd/8AfUX/AMXQBcqnef8AH1p//Xwf/RUlH2yf/oGXf/fUX/xdVbu7mNzYk6fcjE5IBaP5v3b8D5/x/CgCnrFncXXiayMUcmz+zL2IyhTtR2aDaCexODj6H0rA02a+uGtVsrC/tZk0yHTWeWzkiEUpJLEFlAYIqsQwypJUA5Ndr9sn/wCgZd/99Rf/ABdH2yf/AKBl3/31F/8AF0LT+vX/ADYPVf15f5Iy9T0/yLnwxBZ27/Z7S9wQikiNBbTKCT2GSoye5HrXO2k15aS2MEWmXz3emNeGTdaSrHI0km2PEhXawYsGJUnaAScYrtvtk/8A0DLv/vqL/wCLo+2T/wDQMu/++ov/AIuh67h28jltcuJNJsLTw8kepGO7idr3UrXT5rgrk/PtEaNiRyzEZ4UZPOADQks5XeTRLSyvY3k1IXUMxtZVjWDyRgmQrtBGNm0nd7V3H2yf/oGXf/fUX/xdH2yf/oGXf/fUX/xdJq+/9a3/AK+fcP6/Q4KO01bWrWCHTdPeDyNNt7K8j1BZrNXG7MkaOYyTwpXcoIw5wQa6nwVDe2/hW2h1CzS0nR5B5KOzALvYjqqkfl0xWp9sn/6Bl3/31F/8XR9sn/6Bl3/31F/8XVX/AK9dRJW/r5Fyiqf2yf8A6Bl3/wB9Rf8AxdH2yf8A6Bl3/wB9Rf8AxdIZcoqn9sn/AOgZd/8AfUX/AMXR9sn/AOgZd/8AfUX/AMXQBwGo/wDIVvf+viT/ANCNVqsX7FtSu2KlCZ5CVbGR8x4OOKr0AFFFFABRRRQAUUUUAFFFFAHd2f8Ax6Sf9fE//o1qwNT/ANRN/wBc3/8AQTW/Z/8AHpJ/18T/APo1qwNT/wBRN/1zf/0E00Anw5H/ABL9e/7D9/8A+jTXzrrn/Iwal/19S/8AoZr6H+HsojsNeBGc6/f/APo0188a5/yMGpf9fUv/AKGa1Xwfd+pg/wCKvR/ofRVt/wAk5h/7Aq/+iKufDD/kmnh//rzj/lVO2/5J1D/2BV/9EVc+GH/JNPD/AP15x/yqanQ0h1Oqk7VHUknao6goQ1Bc/wCok/3T/KpzUFz/AKiT/dP8qAKbal/ZelJN5Xm+ZqHkY3bceZcFM9D03Zx7U668SWGn6leW+pXFtY29tFDJ9puZ1jRjIXAX5sAH936859qq6how17w/9heOCWI6j5ksdwu5HRLncykYOcgEYPFZV74Kls7q6fw5bWNjaXH2fzbS1mew87Z5u4GSFCycvGcrydhU4BpdPn/l/wAEo6NvE2gIIS+uaaonTzIc3cY8xcE7l55GAeR6GqqeLdOk1JoFnt3s/JjmS9W6j8tlYSknJIyAIjyuevQAE1zuk+BdQtrVlujZGR5raQjz5JsCO7knYF3Xc3yuACeSQc461BffD7VLr7WBPZbZ/OwGduj/AGrGfl/6eEz9G9svuI7yw1bTdVWVtO1C1vFify5DbzLIEb+6dpOD7VcrLs9MkttevL392IJrWCCNF6qYzITxjAHzjH41qUgQUUUUAFFFFABRRRQBTvP+PrT/APr4P/oqSq1/rttpmpx2148Nvbtay3Ml1NKESMIyLg54539c9verN5/x9af/ANfB/wDRUlcxqGg+Jr27upvt1qpjikhtJY5XikljkmRyjlU/dEInlh13HndwRijr/Xb/ADGdHaa9o1/EZbPVrC4jEbS74blHGxThmyD0B4J7VejkSWNZI3V0cBlZTkEHoQa8z/4RLUI57e0miKT399Ibgxzz3qLYtEiyxvcSKp3MY1wDzyMDAJHpwAAwOBT6E9f6/r+kFFFFIYUUUUAFFFFABRRRQAVkie8t9Ilews1vLn7TKqRNMIl5mYEs2DgAcnAJ44BrWrHK3r6NMNPjtJZ/tM2YrssI5F81tykgErkZ5w30NJgjGufG11FYRzRaVDJMkd3JdK17sjRbZ9kgjfYd5J5UEKMZyV6VZbxbKuoEtp6LpK3AtWumuCJVk8vf/qtmNuSFzvzntWPP4O1c2EEUdtpEyqJvLspZ5Fg092IMbwERkkoAQPlTGflK9KvN4X1WS5axmktZdLe7F5JcmVhOzeUFK+Xs28uN27f3xijW39d/8v8Ah3sGn9fh+P8ASHReMr/yE8/R4I7q6jglsYReMwlWV9o8xvLBQrwWAD4B6mt/RdSl1Oyke4t0t7mGZ4JokkMiq6nHDFVJBGCCQOvSuXTwzr8kVvPcDTheadDbRWipcuUm8t9zM58sbNwAGAHx6muk0CwurGzna+EK3dzcyXEqQuXRCx4UMVUnCgDOB9KrTX5/mrfhe/6C1/L8tfxsatFFFIYUUUUAFFFFABVO8/4+tP8A+vg/+ipKuVTvP+PrT/8Ar4P/AKKkoAyG8TTr4k/s46cn2T7ULLz/ALR+980w+aCItvKbeN27Oc/LgZp2p+I7m3vRZ6fp8d1K1wtqjzXJhj83y2kYMwRiMKo5AOS2OME1n3Xh7Wf+Ell1K1i01384yxXsszrceX5eBbEBCBGX5J3Y5ztLc1Y1Xw3cy6PYWkENlqSwymS7tdQYpDeMwYszEK/O9t4BUjPpgEL7P9f13/4bUb30/r+tP60L0HiIXWi6XfwWbvNqOwxW28A8jc3zdMBQxz0OBzzWbL4s1O0mezvNFt4tQkWFrWBL4ureZJsAkby/kIPJwHGAcE4q3o2j6nZQiW+nguby3tRb23zsVBI3MSSM8ttXudsYPUkVkWfh/wARmweS+tdKOr/aIbtrpb+RxcSI2dhBhHlR7dwUDdjPQkkl/a8v0/rX8BPbT+v62LMnjDUfKaODRreS+t0uJLyE3rKkaxMAdjeUS5bIKgqv1FaZ8T27+INN0q3iaU3kBuGlBwIl25QH1LYb6bfpWI3hvxBF595bpppvr6O5juo3uXEcPmMCjI3lkvtC4wVXOeoqxaeCJtO16wvrXWrxreGVpJreUQ4OYhGAp8rdjCqOW4A4NEdtf6f9WE99P6Wn/BOwooooGFFFFABRRRQB5pqP/IVvf+viT/0I1WqzqP8AyFb3/r4k/wDQjVagAooooAKKKKACiiigAooooA7uz/49JP8Ar4n/APRrVgan/qJv+ub/APoJrUfUrTTNPEl3IyLLeTRJtjZyW8xzjCgnoprHu7iK6sp5YWZkCOPmRl/hP94CmgIvAP8Ax467/wBh+/8A/Rpr591z/kYNS/6+pf8A0M19BeAv+PHXf+w/f/8Ao018+65/yMGpf9fUv/oZrf8A5d/15nM/4y9H+h9E23/JOof+wKv/AKIq38Md3/CtPD+CB/ocfUe1Ubdv+Lewj/qCr/6T1ofDD/kmnh//AK84/wCVZ1OhrDqdRIG4+Zf++f8A69M+b1X/AL5/+vUsnao6gsad3qv/AHyf8aguN3kScr90/wAJ9PrVg1Bc/wCok/3T/KgCGwS9MEhhuLdI/tE2FeBmI/eN3Dj+VWfL1L/n7tP/AAGb/wCOViarqkuj+HBeRNIuNTVH8qEysUa62sAoBJJBI4GfSs25+Ilhp+o6hNdNdJZJHbRwQXNv9jdpXMpbBuPLGNqA5YgfLgc8FdL/ANf1qM63y9S/5+7T/wABm/8AjlHl6l/z92n/AIDN/wDHK5Kf4seGIILSYz7lnjMj4uLceSoYoc5lG/lW4i35xxnIz3CsHUMpBUjII70WAqeXqX/P3af+Azf/AByjy9S/5+7T/wABm/8AjlXKKAKfl6l/z92n/gM3/wAco8vUv+fu0/8AAZv/AI5VyigCn5epf8/dp/4DN/8AHKPL1L/n7tP/AAGb/wCOVcooAp+XqX/P3af+Azf/AByjy9S/5+7T/wABm/8AjlXKKAMm7j1D7TY7rm2J887cW7DB8t+vz88Zq15epf8AP3af+Azf/HKLz/j60/8A6+D/AOipK5S71a7+yz65ca9fWFol1LDDb22mi5hVYmZS022NnwSrEkOgAIGQeSdQ6HV+XqX/AD92n/gM3/xyjy9S/wCfu0/8Bm/+OVh3XjmxszcvNY6h9khEwS7VEMc7xAl0Qb92Rtf7ygHacE8Zqaj4yuW8mK10jUrYrfW0FzPKICkG+Rco2JCclGB+UHG8ZIOcHbzA6fy9S/5+7T/wGb/45R5epf8AP3af+Azf/HKwP+E8shGJH0zU0SUI1oTHGftaPIsYeMByQMuhw+04YHHXG1pGrLq0M5NpcWc1vKYZre42F0bAbqjMpyGU8E9fWhagS+XqX/P3af8AgM3/AMco8vUv+fu0/wDAZv8A45VyigCn5epf8/dp/wCAzf8Axyjy9S/5+7T/AMBm/wDjlXKKAKfl6l/z92n/AIDN/wDHKPL1L/n7tP8AwGb/AOOVcooAp+XqX/P3af8AgM3/AMcqrp8eoG2fZc2wHnzdbdjz5jZ/j9a1q5jWp9St/C0zaSt2141/sH2OON5QpucOVEg2Z27uW4HU0Abfl6l/z92n/gM3/wAco8vUv+fu0/8AAZv/AI5XGHU9UvYTaWWtanbT2dncTztdQ23nGZGAWOQLGU2jnlMZBGGoXxFqboNe+3Siz+2Lbf2eI4/LKGEHdnbv3bzn7+MdqV1v/W9vzDy/ra/5HZ+XqX/P3af+Azf/AByjy9S/5+7T/wABm/8AjlcQuqa7brY2cutTyzarBayrOYYQbVnkAkEYEYBG0/LvDHI5zXV+HLm6ms7qC8uXuZrS7lt/PdVVpFBypIUBc4IBwAOOlVb9fw0f5iuv689S55epf8/dp/4DN/8AHKPL1L/n7tP/AAGb/wCOVcopDKfl6l/z92n/AIDN/wDHKPL1L/n7tP8AwGb/AOOVcooAp+XqX/P3af8AgM3/AMco8vUv+fu0/wDAZv8A45VyigCn5epf8/dp/wCAzf8Axyqt3HqH2mx3XNsT5524t2GD5b9fn54zWtVO8/4+tP8A+vg/+ipKADy9S/5+7T/wGb/45R5epf8AP3af+Azf/HKx9Q1HU7DxZHG1zC+mvp1zcLbrBhw8Zi5Zyxz99sABevOeMUtF1a+sDv1jU5byKXS01Bi8Ua+S2fnVNirleVwDk8daFr/Xr/kwen9en+Z0vl6l/wA/dp/4DN/8co8vUv8An7tP/AZv/jlYU11r1rf+HPtF5AsV7dOl3b/ZwXBMU0gUSZxtXao+7k7c7uSCtvcaja+LltJNYkv42hlmu4TBGkVmuR5WCo3AnkYdmyAx4xRtuBueXqX/AD92n/gM3/xyjy9S/wCfu0/8Bm/+OVzGqXmqPoUutLrk+nwvueztoLWJ5JScCFPnUlt2M7QA2XxuGKrSa3rMEj6tcXjrDFfCzk05UjMWPKGTu2793mHP38YHSle24HYeXqX/AD92n/gM3/xyjy9S/wCfu0/8Bm/+OVyVhrmp6VGhv72XU5r3T4bm3hdI0xcO23y0KIvyEsvLZIAJJxXQeFLu9vvDNncajMs14wcSyKgUMQ7DgDoOKqwk0y75epf8/dp/4DN/8co8vUv+fu0/8Bm/+OVcopDKfl6l/wA/dp/4DN/8co8vUv8An7tP/AZv/jlXKKAPM78MNSuw5Bfz5NxUYBO49B2qvVnUf+Qre/8AXxJ/6EarUAFFFFABRRRQAUUUUAFFFFAHT6hbXdzpcS2izsy38zOIbw2x275BywB3DJHy/Q9qzZYpodLnWfeH2scPdGc/dPcgYrprP/j0k/6+J/8A0a1YGp/6ib/rm/8A6CaYFbwF/wAeOu/9h+//APRpr591z/kYNS/6+pf/AEM17/4FbFjrn/Yfv/8A0aa+f9cP/FQal/19S/8AoZrf/l3/AF5nM/4y9H+h9B25/wCLfw/9gVf/AEnrU+GH/JNPD/8A15x/yrKt/wDkQIf+wKv/AKT1p/DFQfhp4f6/8ecfQ+1Z1OhrT6nWSdqjp0ijjlv++jTNo9W/76NQWBqC5/1En+6f5VMVHq3/AH0f8aguB+4k5b7p/iPp9aAIE0+LUtMEMzOqx3zTgoQDuScuOo6ZUZqPUvDFrqN+9/8Aarq2vf3XlTwFN0JTeAVDKRkiV1O4EYPTvU1hYwzQSSM9wCbibhLiRR/rG7BgKs/2ZB/z0u//AALl/wDiqQzJfwjGXSSPWdXhnMXlXU6TJ5l0m4th2KErgs+DHsKhsDAAx0QGBiqf9mQf89Lv/wAC5f8A4qj+zIP+el3/AOBcv/xVAFyiqf8AZkH/AD0u/wDwLl/+Ko/syD/npd/+Bcv/AMVQBcoqn/ZkH/PS7/8AAuX/AOKo/syD/npd/wDgXL/8VQBcoqn/AGZB/wA9Lv8A8C5f/iqP7Mg/56Xf/gXL/wDFUAXKKp/2ZB/z0u//AALl/wDiqP7Mg/56Xf8A4Fy//FUAF5/x9af/ANfB/wDRUlY+peDYNRF7CNV1S1sb0lriztpEWN3PVgShdcnBIVgpOcg7mzeu9PhW5sQHufmnIObqQ/8ALNzx83HSrX9mQf8APS7/APAuX/4qjzAxbjwRY3RuElvb82svnNHa70EcDygh3T5N2TubhiwG44Aqa98JW17qRuv7QvoYXniuZbSJo/KlljxtZsoW6KoIDAHA4zzWp/ZkH/PS7/8AAuX/AOKo/syD/npd/wDgXL/8VQBiweCbKFot99fzpbmMWqSumLZEkWQRrhASuUQZbc2FAzW3Z6fFZT3s0bOWvJ/PkDEYDbFTjjphB+tJ/ZkH/PS7/wDAuX/4qj+zIP8Anpd/+Bcv/wAVQBcoqn/ZkH/PS7/8C5f/AIqj+zIP+el3/wCBcv8A8VQBcoqn/ZkH/PS7/wDAuX/4qj+zIP8Anpd/+Bcv/wAVQBcoqn/ZkH/PS7/8C5f/AIqj+zIP+el3/wCBcv8A8VQBcrJWyN/pMkC3dzaP9qlZJ7ZwroRKxHUFSPZgQe4q1/ZkH/PS7/8AAuX/AOKqrp+nwvbOS9z/AK+YcXUg6SMOzUAZ03gm2kt1RNV1KKZlmS4uUaIyXKykFw+6MqM7RjaFwBgYFWB4SsxqguhdXYtvNE/2DKeQZBH5Yf7u/O3HG7GRnFaf9mQf89Lv/wAC5f8A4qj+zIP+el3/AOBcv/xVAGJD4ItorVoW1TUZHVIY7aZzFvtVibdGExGAcH++GJxzmtnSdMTSbL7Ok81wzSPLJPPt3yOxJLHaAvfsAKd/ZkH/AD0u/wDwLl/+Ko/syD/npd/+Bcv/AMVQBcoqn/ZkH/PS7/8AAuX/AOKo/syD/npd/wDgXL/8VQBcoqn/AGZB/wA9Lv8A8C5f/iqP7Mg/56Xf/gXL/wDFUAXKKp/2ZB/z0u//AALl/wDiqP7Mg/56Xf8A4Fy//FUAXKp3n/H1p/8A18H/ANFSUf2ZB/z0u/8AwLl/+Kqrd6fCtzYgPc/NOQc3Uh/5ZuePm46UAWZ9LgudTgv5C5khgltwmRtKyFC2eM5/djHPc1kWPg21s3iMmo392sPlJGk5iwscRJSP5UUlQxDZOWJVckjIOx/ZkH/PS7/8C5f/AIqj+zIP+el3/wCBcv8A8VQtA30FvNOhvrmxnlZw1lOZ4wpGCxjdMHjphz6c4rCtfBYtUu4Tr2rTWt5K8txBKLciQs2WBYRByCPl+9wvAxgY3P7Mg/56Xf8A4Fy//FUf2ZB/z0u//AuX/wCKoAz9X8Nf2rqlnqEer6hYTWiOkQthCyDdjLbZY3G7AxkYOCR3OWf8Ipbtqn2yS/vZIzKJ3tG8vynm8vy/MOEDZx2DBc84rT/syD/npd/+Bcv/AMVR/ZkH/PS7/wDAuX/4qgDGtPA+kxxGHUd2sQiKOGKLUooZEhjTO1VAQA43Hlst71qaHodh4d0tNO02BIbdGZgqoq5LEk/dAHf9BUv9mQf89Lv/AMC5f/iqP7Mg/wCel3/4Fy//ABVAFyiqf9mQf89Lv/wLl/8AiqP7Mg/56Xf/AIFy/wDxVAFyiqf9mQf89Lv/AMC5f/iqP7Mg/wCel3/4Fy//ABVAHAaj/wAhW9/6+JP/AEI1WqxfqE1K7QZIWeQDcST949SeTVegAooooAKKKKACiiigAooooA7uz/49JP8Ar4n/APRrVgan/qJv+ub/APoJrfs/+PST/r4n/wDRrVgan/qJv+ub/wDoJpoDP8EHFnrn/Yfv/wD0aa8C1z/kYNS/6+pf/QzXvPgtsWutj/qPX/8A6NNeDa5/yMGpf9fUv/oZrf8A5d/15nN/y++T/Q+gbdx/wgMI7/2Kv/pOK1/hh/yTTw//ANecf8q5OKd18IwJ2/sVf/Sauq+GLKPhp4fyQP8AQ4+p9qir0NafU6yTtUdOkdOPnX86ZvT++v8A30KzLA1Bc/6iT/dP8qmLp/fX/voVBcMhgkw6fdP8Q9KAOc8UwfafB5g+y2935mqxr9nuTiKTN4PlY7W4Pf5T9DWW8uu+FPJ0vR9IsoZ9Qkmu1tbNElt7WNFjUom+S3B3E7yeMZb5T1rrra60w2zwXk9puS6lfy5nXKsJWZTg9CDgg07UT4c1i3W31M6VewKwcR3PlyKGHGcNkZ5PPvSWidv62/yKOasvFuu3cP8AaTR6dHYpeW1s9simaR/NSIkrKsmzgy8EBgwHbrUX/CX695Nk4OlO2qRxTWoWGT/RleeOMrJ8/wC8OJRyNnKkY9Ot8/QdrL5um4aRZWG5OXXG1j7jauD2wPSoIIfC9tJPJBFo8T3EommaNYlMkgO4OxHVgeQTzmnpf+v68xdDXt1mW3jW4kjknCgSPGhRWbuQpJIHtk/U1JVP+1tN/wCghaf9/l/xo/tbTf8AoIWn/f5f8aQFyiqf9rab/wBBC0/7/L/jR/a2m/8AQQtP+/y/40AXKKp/2tpv/QQtP+/y/wCNH9rab/0ELT/v8v8AjQBcoqn/AGtpv/QQtP8Av8v+NH9rab/0ELT/AL/L/jQAXn/H1p//AF8H/wBFSVg29+dKg8YagIjKba6aYRg43bbaI4z+Fal3qentc2JW+tiFnJYiVeB5bjJ59SPzqtLY+EJ9T/tOW10OTUNwb7U8cRlyBgHf1yAPWl39P8v8hmFL4i8Rx38ejm40eW7lZG+1x20nlLG8Mr42ebksDF13jIYcDv1Phy5e88M6VdSDDzWkUjfMzclATyxLH6kk+pNV7OHwvp0KQ2UWj20SSGVUgWJFVyNpYAdCQcZ9KuQ6hpNvCkMN5ZRxRqFRElQKoHAAAPAqtNfkT2L9FU/7W03/AKCFp/3+X/Gj+1tN/wCghaf9/l/xpDLlFU/7W03/AKCFp/3+X/Gj+1tN/wCghaf9/l/xoAuUVT/tbTf+ghaf9/l/xo/tbTf+ghaf9/l/xoAuUVT/ALW03/oIWn/f5f8AGj+1tN/6CFp/3+X/ABoAuVzWq6jcab4ame1iuWnlu5YVe3tnnaHdKwMmxFYnaMnGOSAOM5rZ/tbTf+ghaf8Af5f8aq6fqenpbOHvrZT58xwZVHBkYg9fShgedabbaXqvh+1gvbdprSz0i9kg+227BkdZQDIBIud4G3DdeT71aSItGNZa3jPiEamkInKDzdv2YZj3ddu0lsZxnmu1urfwrfQww3kOjXEUDF4UmWJ1jY8kqD0PuKeV8NHVRqpGk/2iF2C7xH523GMb+uMe9K39fO//AAPT7g/r8Lf8H1OBTTdNgTSbS1tYFsdRtNPfUFWMBbgtMMNIMfMXJIJOc967fwlDFa2N9a20aRWcGoTx28aKFVEDcqoHAAYsMD0pYrLwjDbXVtFa6HHBdnNzEkcQWY/7Y6N+NXLO60PT7SO0sp9OtraMYSGF0RFHXhRwKq+/z/Fr8tl69BW/T8F+pp0VT/tbTf8AoIWn/f5f8aP7W03/AKCFp/3+X/GkMuUVT/tbTf8AoIWn/f5f8aP7W03/AKCFp/3+X/GgC5RVP+1tN/6CFp/3+X/Gj+1tN/6CFp/3+X/GgC5VO8/4+tP/AOvg/wDoqSj+1tN/6CFp/wB/l/xqrd6np7XNiVvrYhZyWIlXgeW4yefUj86AM29s4I/iFYXJknBn0y7Em65fYoD2/KqW2p15KgE96zNGjstM+263oWnQ22mzwx29nGg2nUJtxCyt6glgA5+ZhuY8YJ6ma70S4ffNcafI3ltFud0J2NjcvPY4GR3wKz7HTfBulzifT7LQbSYHIkt4oY2BwRnIA7Ej8TQun9dX/mN7GLqPh/TpLy00y1to7rxAnlTy6hIMtZr5hdpQf4Gdt+FXG45z8qkirpX2HSrzT9ZMMcd3OdQbUJkjHmSqkhyGI5bawVQDnHQV015YeD9QvhfXtpodzeDGLiaOF5Bjp8xGeO1PW28KJeTXiw6Mt1OytNMEiDyEMGBZupIYAjPcA0Ctqc/rOj2N1Yx29/psGoeJNSWWSGGY7lt2YKDIeyiMbF3gbuABy2DRnsltribUVRJteh1YWq3bIPOZBbj5d3XaVy2M4yc11upWfhLWpUl1W30S+kjG1GukilKj0BbOBT0h8Lx6imoxx6Ot8iCNLlViEioBgKG6gY4xSt/Xz/pfMP6/r8/kee3On29tp1ha6TZ2/kajZWDX0eRClyWmABlYD+PLKxwSQSMHoe/8JG3GjyQW2nxaeLe5lhktYJTJDG6tyIzgfL3wFXGSMA5ogtPCdtb3dvBb6LFBeZ+0xxpEqz5z98Dhup6+tW7O60PT7SO0sp9OtraMYSGF0RFHXhRwKq+/9f1b+rCt/X9d/wCrmnRVP+1tN/6CFp/3+X/Gj+1tN/6CFp/3+X/GkMuUVT/tbTf+ghaf9/l/xo/tbTf+ghaf9/l/xoA4DUf+Qre/9fEn/oRqtVi/dZNSu3RgyNPIVZTkEbjyKr0AFFFFABRRRQAUUUUAFFFFAHd2f/HpJ/18T/8Ao1qwNT/1E3/XN/8A0E1v2f8Ax6Sf9fE//o1qwNT/ANRN/wBc3/8AQTTQGP4Qbbba1/2Hr/8A9G14Vrn/ACMGpf8AX1L/AOhmvcvCf/HrrX/Ye1D/ANG14brn/Iwal/19S/8AoZrf/l3/AF5nN/y++T/Q95htlbwTDJjn+xV/9J63/hh/yTTw/wD9ecf8qyLf/kQ4f+wMv/pPWv8ADD/kmnh//rzj/lUVehrT6nVSdqjyfWpJO1R1mWBJ9ar3JPkScn7p/lU5qC5/1En+6f5UAVv7UtNH0aS8vGkWEXUifuoXlYs0zKoCoCxJJA4FFr4o0m9mt4IZbjz53dFhezmSRSgUtvVkBQAMvLAD5l55FZut2dxfeGlgtWnSU6qh8y3QM8YF3ksAQRwATyCOOayb/wAK3thrsmoaXcahd6s+mXbLdzzFUe4IiWMOqBYs7RgDaM7ATkrkSn38/wAFcqx299qNnptu097cxwRqrNl2xkKpZsDqcKpPHYGq+na5Y6rNLDaG4LwqjSCW1li2bgGAJdRhsEEr1AIyBmvKrrTdevNBubRbnXby1lDbkNre27b/ACJiRmaZ5GBby8jiMnAGSWFa97oWpaXa6oNMl1WC3l1aP7RK8l5dSGAQLhkVZRK37wgEowOBg5C4qmrP+vL/ADF/X9f1uen0Vw/g+w1N9R8/VLvVpY7e1j+zGfzrdGy8wJaMyNuOzZ/rCW+6WAau4oasAUUUUgCiiigAooooAp3n/H1p/wD18H/0VJVZvEmko+rI94FOkoHvdyMBEpXcDnHzcA9M/nVm8/4+tP8A+vg/+ipK4q80HU7rXtTlt7dha3d95V0XG0PAIoHDDP3hmN4+M/6w+hpNvX0/yGjt9P1Ky1W0S6sbhJ4XVWDKeQGUMMjqDtYHBweRT7e8t7qW5jhk3PbSeVKMEbW2hsc9eGB49a8z0qHUbfTbKHWo9fj0qNVUpZLciVZBbQBBiH94Ez52cfLu684q9YaVqERuNTU66t3/AGpa7EnlZS0RSBZGkjjPlucb8sQQCDgjFW0ua3T/AIKJ6HotQNeW6X8dk0mLmSNpUTB5VSoJz06sv515MjeKZGv/ALC2qwtPal7iGW2v2FsfOj3KjyykSuIzLjyNmccfw46Dwfaammu2811Lf3VqtvcpDPdWc8BUFoDt/fyPLjIfG8g8HA2gGlFXtfz/AF/rzG9LnoFFFFIAooooAKKKKACqemkC0kJOALifJP8A11arlc3qlnqOoeG5rTTUtnaW7kWdLidoQ8PnNvUOqOQWHy9OhPQ4oYE8njLQY7O0vPtrPb3SNLFJFbySDy1OGkbap2ICRlmwo9asHxHpY1f+yzPJ9p3bM/Z5PK3bd+3zduzdt527s47VwUlnf2/hO1E9hq9lqht7m3ij0qPzklDOSsUhaPMYJwQ2FAGfnGcVaXTr1I/+EeNtdm8/tBbj7ULeTyfLEABfzcbM5BTG7Ptilff+uv6LX+rh2/rp/np/Vjq4fGOiT2s9ylxceXCEY7rOZWkDnahjUpmQMeAUBzWnp2o22q2a3do7tExZf3kbRsrKSCGVgGUgg8EA156ouLkabeppmoRxaVbWUd1E1jKrFlkBYIpUGTYBnKBge1dh4XV2sr26aGaFLu9mnijmiaNwhOASjAFc4zggHmqtv8/zVvv3+Qrv8vy/TY3KKKKQwooooAKKKKACqd5/x9af/wBfB/8ARUlXKp3n/H1p/wD18H/0VJQAwa1pja0dGW+hbUhEZmtlbLogKjLAfd+8uM4z2zg1FqPiLTNLn8i5mlM/y4hgt5JpGzuIwsaknhWJwOAMnAxVTUxJB4r0+/8As88lvb6deeY0UTPyWgIUADliFbA6nBxWdqF/f6Bo0LrZ3L6lqk5e4mhs5bpbQlerLEpLBVCoo43EDJAyaOif9b2HY1R4t0Rp7WJLxnN0kbxukEjIoc4Te4XbGWIwA5BJ461b1LXNP0gqL2dkd13IiRPI78hcKqgkklgAAMnsODXBajpUYtrWy0UeI45LqO2j+a0Kw3CrISZJmaPdEy5ZiCYy3AAbpXRXOpXWl6TeeIBpt3c3l5IkcFulvI7xRZwm9VUuAMtI3y5G4jGQKBIuy+MtDhtIrlri4McvmfKtlMzxhDhzIgTdGFJGS4AGasN4m0lNU/s5rlxMDtLmCTyQ2zftMu3y923nG7OO1cbhbCJ7xLbVr/7bY3UUkjaXOrvcuytgxlN0aN0BI2gIASetPtYLrT3i0qayvpLmHUo7xpktZGjaJIFyRIBtLZUpt3bj6ULz/rW34LX0E/L+tP8APT1Ox0rxDputSSR2UsrPGiybZbeSEsjZw671G9Tg/MuR71qVyXhS+Otahc6teW+oW19JCqLa3NhPAltFkkLukQB3JOWKkjgAcDJ62mAUUUUhhRRRQB5pqP8AyFb3/r4k/wDQjVarOo/8hW9/6+JP/QjVagAooooAKKKKACiiigAooooA7uz/AOPST/r4n/8ARrVgan/qJv8Arm//AKCa37P/AI9JP+vif/0a1YGp/wCom/65v/6CaaAxPCn/AB661/2HtQ/9G14brn/Iwal/19S/+hmvcfCn/HrrX/Ye1D/0bXh2uf8AIwal/wBfUv8A6Ga6P+XS/ruc3/L75P8AQ+gbf/kQ4f8AsDL/AOk4rV+GJI+Gnh/5Sf8AQ4+n0rKt/wDkQ4f+wMv/AKTitf4Yf8k08P8A/XnH/Ks6vQ0p9TqJGPHyN+n+NMyf7jfp/jUsnao6zNBpJ/uN+n+NQXB/cSfK33T6en1qwaguf9RJ/un+VAENhczRwSKtjcSgXE3zo0eD+8b1YH9Ks/bJ/wDoGXf/AH1F/wDF1mXepz6ToLXFtbR3M73xgjilmMSlpLgoCWCsQBuz0NNtvFVrBbXT+IJtP0iW2uvsshe9BhZyiuNsjqmflYcYHIPpmktf69P80M1ftk//AEDLv/vqL/4uj7ZP/wBAy7/76i/+LqrdeKPD9jO0F5rumW8yjc0c13GjAYByQT6EH8avxXtrO6JDcwyO8QmRUkBLRno4x1U+vSgCL7ZP/wBAy7/76i/+Lo+2T/8AQMu/++ov/i6uUUAU/tk//QMu/wDvqL/4uj7ZP/0DLv8A76i/+Lq5RQBT+2T/APQMu/8AvqL/AOLo+2T/APQMu/8AvqL/AOLq5RQBT+2T/wDQMu/++ov/AIuj7ZP/ANAy7/76i/8Ai6uUUAZN3dzG5sSdPuRickAtH837t+B8/wCP4Va+2T/9Ay7/AO+ov/i6Lz/j60//AK+D/wCipKwZfFd5Fq97G2mW/wDZlnfQ2Mtz9sbzi8ixkEReXggGVQfn6An2o3dgN77ZP/0DLv8A76i/+Lo+2T/9Ay7/AO+ov/i6oDxf4eIuZDrWmrbW+wSXJvYfLDPnCk7sg8dwM54zziZ/E2gRi3L65pqi5CmDN3GPN3Z27efmzg4x1xQBZ+2T/wDQMu/++ov/AIuj7ZP/ANAy7/76i/8Ai6uUUAU/tk//AEDLv/vqL/4uj7ZP/wBAy7/76i/+Lq5RQBT+2T/9Ay7/AO+ov/i6Ptk//QMu/wDvqL/4urlFAFP7ZP8A9Ay7/wC+ov8A4uj7ZP8A9Ay7/wC+ov8A4urlFAFP7ZP/ANAy7/76i/8Ai6q6fdzLbOBp9y37+Y5DR/8APRuOX7dK1q5zVNcHh3wxcaiUt3K3jRgXFx5EY33BTLPhtqjdknB6UAlc1/tk/wD0DLv/AL6i/wDi6Ptk/wD0DLv/AL6i/wDi6w7zxeNP8MW2rTw2lxNcyBIYtPvPPikBPLLKUXICgsTt7Y54p8/iqa31WRHsIhpUV19je7+0HzBL5e//AFezG3Py535z2oem4Gz9sn/6Bl3/AN9Rf/F0fbJ/+gZd/wDfUX/xdc3F4yv/ACE8/R4I7q6jglsYReMwlWV9o8xvLBQrwWAD4B6mt/RdSl1Oyke4t0t7mGZ4JokkMiq6nHDFVJBGCCQOvSiwrol+2T/9Ay7/AO+ov/i6Ptk//QMu/wDvqL/4urlFAyn9sn/6Bl3/AN9Rf/F0fbJ/+gZd/wDfUX/xdXKKAKf2yf8A6Bl3/wB9Rf8AxdH2yf8A6Bl3/wB9Rf8AxdXKKAKf2yf/AKBl3/31F/8AF1Vu7uY3NiTp9yMTkgFo/m/dvwPn/H8K1qp3n/H1p/8A18H/ANFSUAH2yf8A6Bl3/wB9Rf8AxdH2yf8A6Bl3/wB9Rf8AxdZA8US/261q1lEtgLz7B9pNz+98/wAveP3W37pzjO7OeduOas3Wr6k91Pb6RpcV4YXWN5Z7ryI1baWYEhGbgFOinlsHGCaOl/6/rUOti99sn/6Bl3/31F/8XR9sn/6Bl3/31F/8XXLWvj032rafYW8GlxPdW8U5W81PypCWZ1ZYkEbebjYTnIByOlb+oareRXj2emact7cJGrvvnESRlmwu5sHjAcnAJG0cHIoAtfbJ/wDoGXf/AH1F/wDF0fbJ/wDoGXf/AH1F/wDF1zTeMtQaJ47fRreW9tluHvIjfEIiwsFPluIyXJzwCq9CDjFWY/Fsk2oIY7CI6Q1wtqLv7QfM8wxhwfL2Y28hc7857Urq1w2Nz7ZP/wBAy7/76i/+Lo+2T/8AQMu/++ov/i6xvB/iiTxVYG82aUkWxGCWWpfanjLDO2UeWoRhxxk9/SulqmrAU/tk/wD0DLv/AL6i/wDi6Ptk/wD0DLv/AL6i/wDi6uUUgKf2yf8A6Bl3/wB9Rf8AxdH2yf8A6Bl3/wB9Rf8AxdXKKAPM79i2pXbFShM8hKtjI+Y8HHFV6s6j/wAhW9/6+JP/AEI1WoAKKKKACiiigAooooAKKKKAO1SSe3WSEWc0n76Rt6MmCGct3YHvWZdWV7cxugs5VLqygs0eBkY7NXQD7z/7xp460AedeFP+PXWv+w9qH/o2vDtc/wCRg1L/AK+pf/QzXuXhT/j11r/sPah/6Nrw3XP+Rg1L/r6l/wDQzXS/4S/rucv/AC++T/Q+gbf/AJEOH/sDL/6Titf4Yf8AJNPD/wD15x/yrFt5B/wgsI4/5Ay/+k4rY+GO7/hWnh/BA/0OPqPas6vQ1pdTrJO1R06QNx8y/wDfP/16Z83qv/fP/wBeszQDUFz/AKiT/dP8qmO71X/vk/41BcbvIk5X7p/hPp9aAMzUNGGveH/sLxwSxHUfMljuF3I6Jc7mUjBzkAjB4qhr3gxZG059DtLeCGzSaMWMN7Np0WJCpLB7cZyCv3cYO49CBW7YJemCQw3Fukf2ibCvAzEfvG7hx/KrPl6l/wA/dp/4DN/8cqbdCrnLaV4Ml0uSzZxaNFbXouSEDsSgtBAAA25idw4BY8dyateB9Lks7K5upUkRZZDDZpLC0Tx2kbN5KMjcjAZjyAcEZAPFb/l6l/z92n/gM3/xyjy9S/5+7T/wGb/45VXYi5RVPy9S/wCfu0/8Bm/+OUeXqX/P3af+Azf/ABykBcoqn5epf8/dp/4DN/8AHKPL1L/n7tP/AAGb/wCOUAXKKp+XqX/P3af+Azf/AByjy9S/5+7T/wABm/8AjlAFyiqfl6l/z92n/gM3/wAco8vUv+fu0/8AAZv/AI5QAXn/AB9af/18H/0VJXG3ngu/n8S6hfR2GjA3V7FcR6o0jfa4I1SNWjUeX0bYw/1gGHOQeQeou49Q+02O65tifPO3FuwwfLfr8/PGateXqX/P3af+Azf/ABylYdzjh4P1m0sLaGzmtwI7Szt5oYryW184RLKGAljQug3OjAqMnaQcA0zS/A+o2uiaxbztZi6vrFreNhPJNsJlnfDSOu5h+9Xk8kgnFdp5epf8/dp/4DN/8co8vUv+fu0/8Bm/+OVTbbbEW1GFA9BS1T8vUv8An7tP/AZv/jlHl6l/z92n/gM3/wAcpAXKKp+XqX/P3af+Azf/AByjy9S/5+7T/wABm/8AjlAFyiqfl6l/z92n/gM3/wAco8vUv+fu0/8AAZv/AI5QBcoqn5epf8/dp/4DN/8AHKPL1L/n7tP/AAGb/wCOUAXKxJILmfR3+x29lcTx3kkiRXoPlsVmY/eAJVuMhsHB7Gr/AJepf8/dp/4DN/8AHKq6fHqBtn2XNsB583W3Y8+Y2f4/WgDl7zwFqGoWNxcNqz2OpXBlYwWpja3QSSK5UGSIt/CuWG3JGcDpV6bwzqc99JZyPbPpMt59se4aYict5e3Z5YQLy43bgw64210nl6l/z92n/gM3/wAco8vUv+fu0/8AAZv/AI5SsrWDzOSTwzr8kVvPcDTheadDbRWipcuUm8t9zM58sbNwAGAHx6muk0CwurGzna+EK3dzcyXEqQuXRCx4UMVUnCgDOB9Ks+XqX/P3af8AgM3/AMco8vUv+fu0/wDAZv8A45VX/r13FZf15bFyiqfl6l/z92n/AIDN/wDHKPL1L/n7tP8AwGb/AOOUhlyiqfl6l/z92n/gM3/xyjy9S/5+7T/wGb/45QBcoqn5epf8/dp/4DN/8co8vUv+fu0/8Bm/+OUAXKp3n/H1p/8A18H/ANFSUeXqX/P3af8AgM3/AMcqrdx6h9psd1zbE+eduLdhg+W/X5+eM0AYuq+G9QvvEsepR6foYNvJ50d5hkuZtqHZC52HC78EsGPAA2d6m1bRdaXQ7LTdJa2lQyFtRae5e3e4ByW2uiOVLOcnA6ZAIzkb3l6l/wA/dp/4DN/8co8vUv8An7tP/AZv/jlFtLB1uc7e6Hq15bJZR6bodpbTxwJO8Mj+ZbrG2QqfuwJAB93OzaSeDVm80zXo/D06aZJaLrN5MJbmV5mRVBIDBH2MchAEUlewJHatny9S/wCfu0/8Bm/+OUeXqX/P3af+Azf/ABygFocq/h/W4bSH+ztP0m1lNnNZSQfbpHRFdgwlDmHLtncSCBktktUkPhbUre5Sxi+xHSEu1uxK8jGXKxBQhj24I3gNnf04xXTeXqX/AD92n/gM3/xyjy9S/wCfu0/8Bm/+OUf1+N/z1/4Af1+n5aGToelajBqhvdQt9NtRFZpZww6e7MjKrE7jlV2joAgzt5+Y5roqp+XqX/P3af8AgM3/AMco8vUv+fu0/wDAZv8A45QBcoqn5epf8/dp/wCAzf8Axyjy9S/5+7T/AMBm/wDjlAFyiqfl6l/z92n/AIDN/wDHKPL1L/n7tP8AwGb/AOOUAcBqP/IVvf8Ar4k/9CNVqsX4YaldhyC/nybiowCdx6DtVegAooooAKKKKACiiigAooooA9CH3n/3jTx1pg+8/wDvGnjrQB534U/49da/7D2of+ja8N1z/kYNS/6+pf8A0M17f4WcLa60P+o9qH/o2vD9dP8AxUOpf9fUv/oZrpf8JHL/AMv/AJP9D3aBv+KLhH/UGX/0mFdH8MP+SaeH/wDrzj/lXMwA/wDCGw/9gZf/AEmFdN8MP+SaeH/+vOP+VKv9n0NKPX1Oqk7VHUknao6wNRDUFz/qJP8AdP8AKpzUFz/qJP8AdP8AKgDD16/uNO8KPPbXVxbOdREbS20AmlVWutrbUKtuOCcDafpVKPxdJ4f0aS81f7fd2jXZitbm9+z2MzpsU/OkxgAO7eAAuSFBx3PQpp8WpaYIZmdVjvmnBQgHck5cdR0yozRrOgLq89vcx6heWFzAkkazWoiLFH27lIkRxzsXkAHjr1qdUn/XYrS6uZ1r44sr2WM2+n6g9m0sMLXuyMRI0qoyAgvuOfMUfKpwTzjrXUVz9j4QsLDS10+Ka6aJZ4Ljc7KW3QiML/D0PlLn6np26Cqdun9bfrclX6hRRRSGFFFFABRRRQAUUUUAU7z/AI+tP/6+D/6KkrjL3xLqtrqviWzF4NzFYdKDRLiGXbEpzx83zTo3Oeh7V2d5/wAfWn/9fB/9FSVUPhrTZL2S7ljaWZro3aMzf6uQxCL5cdto6HPPPYYTW/p/kNHOaH45mm0+0gmsr7VNRa3WRhawxR5xFCzkl5FXrMD/AA+gHHOn/wAJvZSPA1vp9/cWcjQI94gjEULTbSisGcMTh0J2qcbqm0jwdp+jTrNbTXTMIDBiRlI2lIk7KOcQr+Z9sYb+C9Si1S0trGcQaHFJbPMrXau05hC4LRG34Y7FXKygcA46g03d/d/wfwJtZFn/AITWWTVVnXTr6LSl06e7Vpjbxi5CvGBIjNINoCsT85Tg9PTc8N+JLLxRp0l5YghI5TC6mSOTDAA/ejd0PBHRj6dciss+AbNhtfVdUeOKIQ2sbNEVtVEiSLs/d87WjTG/dwMHNbOi6MNGjugb67vprqczyzXXl7i21V6IqqBhR2oVrf13/wAhv+vu/wAzTooopAFFFFABRRRQAVzGtT6lb+Fpm0lbtrxr/YPsccbyhTc4cqJBszt3ctwOprp6yVsjf6TJAt3c2j/apWSe2cK6ESsR1BUj2YEHuKTGjjbrxPcI9nYnXNYshFDPLeXEunxT3CyIyqEkWKNkWNcksygDG35xnJvT63qlvcS6sdTaW0hvxZmzjijELp5Q+cHaX3bzkfPjHGK1ZPBlu9osUeq6lDOfNE93G8fm3AlILh8oVGSBjaqlcYUgVKnhCyj1FbhLq7W1WQTCwBTyPMEflhvu787QON2MjOKNbef/AAf8tF/TF1/r+t9X+HY5xdU123Wxs5danlm1WC1lWcwwg2rPIBIIwIwCNp+XeGORzmur8OXN1NZ3UF5cvczWl3Lb+e6qrSKDlSQoC5wQDgAcdKoQ+CLaK1aFtU1GR1SGO2mcxb7VYm3RhMRgHB/vhicc5rZ0nTE0my+zpPNcM0jyyTz7d8jsSSx2gL37ACq01+f5q33K/wDwRa/l+Wv3v+kXqKKKQwooooAKKKKACqd5/wAfWn/9fB/9FSVcqnef8fWn/wDXwf8A0VJQBj6hqOp2HiyONrmF9NfTrm4W3WDDh4zFyzljn77YAC9ec8YpaLq19YHfrGpy3kUulpqDF4o18ls/OqbFXK8rgHJ4610k+lwXOpwX8hcyQwS24TI2lZChbPGc/uxjnuayLHwba2bxGTUb+7WHykjScxYWOIkpH8qKSoYhsnLEquSRkEXn/W/+a+4Htp/W3/B+8imutetb/wAOfaLyBYr26dLu3+zguCYppAokzjau1R93J253ckFyRahb+LIoR4i1C4t0ie5uoJ47YRohyqLlYlcZO4g7ukZznNbl5p0N9c2M8rOGspzPGFIwWMbpg8dMOfTnFVptCtp01FXmn/4mDqZyCMlFAHljjhSAQe/zNyCcgD+vxf6GBql5qj6FLrS65Pp8L7ns7aC1ieSUnAhT51JbdjO0ANl8bhiq0mt6zBI+rXF46wxXws5NOVIzFjyhk7tu/d5hz9/GB0roNX8Nf2rqlnqEer6hYTWiOkQthCyDdjLbZY3G7AxkYOCR3OWf8Ipbtqn2yS/vZIzKJ3tG8vynm8vy/MOEDZx2DBc84pWfT+tf8r/h6h/X9fMxbDXNT0qNDf3supzXunw3NvC6Rpi4dtvloURfkJZeWyQASTiug8KXd7feGbO41GZZrxg4lkVAoYh2HAHQcVTtPA+kxxGHUd2sQiKOGKLUooZEhjTO1VAQA43Hlst71qaHodh4d0tNO02BIbdGZgqoq5LEk/dAHf8AQVWn9fgJXNGiiikMKKKKAPNNR/5Ct7/18Sf+hGq1WdR/5Ct7/wBfEn/oRqtQAUUUUAFFFFABRRRQAUUUUAehD7z/AO8aeOtMH3n/AN408daAPMvDZxb6z/2HtQ/9G14nrv8AyMOp/wDX3L/6Ga9o8PNiHWR/1HtQ/wDRteLa9/yMOp/9fcv/AKGa63/BRx/8v/k/0PfYFH/CEQn/AKgq/wDpOK3fhioPw08P9f8Ajzj6H2rJt4/+KDhb/qCr/wCk4rY+GH/JNPD/AP15x/yrOs78vobUludRIo45b/vo0zaPVv8Avo1LJ2qOsTUaVHq3/fR/xqC4H7iTlvun+I+n1qwaguf9RJ/un+VAENhYwzQSSM9wCbibhLiRR/rG7BgKs/2ZB/z0u/8AwLl/+KrmPFMH2nweYPstvd+Zqsa/Z7k4ikzeD5WO1uD3+U/Q1lWmi6vo3iKBNPTR9LkmtrudbaON57e2TNuNqKPKJ3FSxPy4LHg95T7+f4K5VjvP7Mg/56Xf/gXL/wDFUf2ZB/z0u/8AwLl/+KrzJPE1ze6zaakYlR3CKEE0uAJGsWP8XH3zwMKe4OTnXTxf4hTS7aaQaVcXGoafHeWqxL5QiZnjQo3mTASE+b8vzJkrtzzkVZ2Edt/ZkH/PS7/8C5f/AIqj+zIP+el3/wCBcv8A8VVXw1qU+q6JFc3ZiNyHkjl8qJ4gGVypBR+VPHIyRnoWGCdakBT/ALMg/wCel3/4Fy//ABVH9mQf89Lv/wAC5f8A4qrlFAFP+zIP+el3/wCBcv8A8VR/ZkH/AD0u/wDwLl/+Kq5RQBT/ALMg/wCel3/4Fy//ABVH9mQf89Lv/wAC5f8A4qrlFAGTd6fCtzYgPc/NOQc3Uh/5ZuePm46Va/syD/npd/8AgXL/APFUXn/H1p//AF8H/wBFSVy2m6ZoN4t5rGsxWv8AaUOpSRveSvslgKy4iRZMhkBXZhQQDv6HccgdDqf7Mg/56Xf/AIFy/wDxVH9mQf8APS7/APAuX/4quCTxv4ha5jjSG1aC+Eb2FzNZ+UrxtPHHu2rcOzDbKD8wjPA45OGa54g1mXw9fJey6dJbXSXlkIoIHjlR4UfMhJkYYJjb5cZUOvzHHIldXDZ2PQP7Mg/56Xf/AIFy/wDxVH9mQf8APS7/APAuX/4qrafcX6UtAlqU/wCzIP8Anpd/+Bcv/wAVR/ZkH/PS7/8AAuX/AOKq5RQMp/2ZB/z0u/8AwLl/+Ko/syD/AJ6Xf/gXL/8AFVcooAp/2ZB/z0u//AuX/wCKo/syD/npd/8AgXL/APFVcooAp/2ZB/z0u/8AwLl/+Kqrp+nwvbOS9z/r5hxdSDpIw7NWtXG+KEmm8J/Zo4kmSfVFimikk8tJEa5IKM2DhWOFPB4JGDnFHYDpv7Mg/wCel3/4Fy//ABVH9mQf89Lv/wAC5f8A4qvNrrSrS/gimj0/w5bQ6NFe+fpl8n2uFWDoWMIJQInyY37cKWI2dandIpXfXIbKOLWjqqxROUHmpGbcERbsZ27STjpnmlfS/wDW9kH9fdqehf2ZB/z0u/8AwLl/+Ko/syD/AJ6Xf/gXL/8AFV5omm6bAmk2lrawLY6jaae+oKsYC3BaYYaQY+YuSQSc5712/hKGK1sb61to0is4NQnjt40UKqIG5VQOAAxYYHpVW3+f4NL9RX/T8Vf9DU/syD/npd/+Bcv/AMVR/ZkH/PS7/wDAuX/4qrlFIZT/ALMg/wCel3/4Fy//ABVH9mQf89Lv/wAC5f8A4qrlFAFP+zIP+el3/wCBcv8A8VR/ZkH/AD0u/wDwLl/+Kq5RQBT/ALMg/wCel3/4Fy//ABVVbvT4VubEB7n5pyDm6kP/ACzc8fNx0rWqnef8fWn/APXwf/RUlAB/ZkH/AD0u/wDwLl/+Ko/syD/npd/+Bcv/AMVXDyw26+Om1Qxwqi6osDXpb/SVb7PjyAP+ePO/Oeufk/jFrxDZJq3gTxFq1yZ0a5s5ZYBHM8RWFI28sHaQSDkuVPHz4IOKFqrjSvLlOu/syD/npd/+Bcv/AMVR/ZkH/PS7/wDAuX/4quJ8U6q3/CS6NbTRajHa2d1bOpisZ3jnkckH51QrhV7Z5Lnj5a1NT/srUI73VNbjE2mRMLS2gILCYhxuGwff3yBVCnIOwetHS/n/AJf5kp/kn950X9mQf89Lv/wLl/8AiqP7Mg/56Xf/AIFy/wDxVedyeH7aOJ7TWLC1it49Nvb21syqulkS6nCdQGQFeV4BZtvFPtYY2ki1m4ihXXBqscLXMigOsZt1yhfqE2ksRnGcmlf+vnb8xv8Ar7r/AJHoP9mQf89Lv/wLl/8AiqP7Mg/56Xf/AIFy/wDxVcj4C0xPD8qaT5GiPM+nQ3DXml23lGQZKjzGyTJnkh+M/N8oruqpqwXKf9mQf89Lv/wLl/8AiqP7Mg/56Xf/AIFy/wDxVXKKQFP+zIP+el3/AOBcv/xVH9mQf89Lv/wLl/8AiquUUAeZ36hNSu0GSFnkA3Ek/ePUnk1XqzqP/IVvf+viT/0I1WoAKKKKACiiigAooooAKKKKAPQh95/9408daqXt01laXNylrcXRiy3kW4UyPzztDEAnvjPbjJ4qroWurrtv9oj06+tYsZVrlYxu+m12oA4Pw5HuTWT/ANR7UP8A0bXjGuj/AIqHU/8Ar7l/9DNe6eE491rrR/6j2of+ja8N10f8VBqX/X1L/wChmupv90jkt+/+T/Q+hbb/AJJ/D/2Bl/8AScVX+HXjDwxY/D3Q7a78R6Rb3EdoivFLfRIynHQgtkGrFtn/AIQCH/sDL/6TiuxrKr0NqfUy38deEDjHirQ//BjD/wDFU3/hOfCP/Q1aH/4MYf8A4qtOSVY8ZDnPQIhY/kBTPtS/88rn/wABpP8A4msjUzj448I/9DVof/gxh/8AiqhuPG3hMwSAeKdEJ2ngajD6f71a/wBqX/nlc/8AgNJ/8TR9qX/nlc/+A0n/AMTTAxrPxj4MkszHc+JNBYC5kkCyX0JwRKzK2C30IP0NWm8Z+CXmEzeJfD7SqhQOb+EsFOCRnd0OBx7Cr/2pf+eVz/4DSf8AxNH2pf8Anlc/+A0n/wATSAxf7f8Ah0FCrrHhdMABSt1bgrjbjHPGNiY/3F9BVTTb74ZaVpT6bb6t4XMEsax3G65ts3IAxmXGA569R3NdL9qX/nlc/wDgNJ/8TR9qX/nlc/8AgNJ/8TQBnWnjDwNYWsdrZ+I/DtvbxDbHFDfQIiD0ADYFTf8ACd+D/wDoa9D/APBjD/8AFVb+1L/zyuf/AAGk/wDiaPtS/wDPK5/8BpP/AImgCp/wnfg//oa9D/8ABjD/APFUf8J34P8A+hr0P/wYw/8AxVW/tS/88rn/AMBpP/iaPtS/88rn/wABpP8A4mgCp/wnfg//AKGvQ/8AwYw//FUf8J34P/6GvQ//AAYw/wDxVW/tS/8APK5/8BpP/iaPtS/88rn/AMBpP/iaAKn/AAnfg/8A6GvQ/wDwYw//ABVH/Cd+D/8Aoa9D/wDBjD/8VVv7Uv8Azyuf/AaT/wCJp0cyyEgLIp64eNk/mBQBkXfjjwk1zYlfFOiELOSxGoRcDy3GT83qR+dRS6/8OZ9Uj1SXV/CsmoxgBLt7m3Mqj2fOR1PfvXQVC1wisV2TtjjKQuw/MCgDAt9V+GNpdSXNtf8AhCG4kYM8sc1srsQwYEkHJO4A/UZqU6/8OWu7m7bV/CpubpPLuJjc2++ZOBtds5YcDg+lbX2pf+eVz/4DSf8AxNH2pf8Anlc/+A0n/wATQBU/4Tvwf/0Neh/+DGH/AOKo/wCE78H/APQ16H/4MYf/AIqrf2pf+eVz/wCA0n/xNSIwdQy5wfUYoAof8J34P/6GvQ//AAYw/wDxVH/Cd+D/APoa9D/8GMP/AMVVv7UnaO4Putu5H5gUfal/55XP/gNJ/wDE0AVP+E78H/8AQ16H/wCDGH/4qj/hO/B//Q16H/4MYf8A4qrf2pf+eVz/AOA0n/xNH2pf+eVz/wCA0n/xNAFT/hO/B/8A0Neh/wDgxh/+Ko/4Tvwf/wBDXof/AIMYf/iqt/al/wCeVz/4DSf/ABNH2pf+eVz/AOA0n/xNAFT/AITvwf8A9DXof/gxh/8AiqoQeL/BVzps9re+ItAlhllnEkM17Cyupkbggtggg1tfal/55XP/AIDSf/E0fal/55XP/gNJ/wDE0AYL6x8M5bW1tZNR8JPb2jbraJp7YpCeuUGcKfpUx8R/Dw6qNVOteF/7RC7Bd/arfztuMY35zjHvWx9qX/nlc/8AgNJ/8TR9qX/nlc/+A0n/AMTQBgxax8M4ba6totR8JRwXZzcxJPbBZj/tjOG/Grln4v8AAun2kdpZeIvDttbRjCQw3sCIo68KGwK0vtS/88rn/wABpP8A4mj7Uv8Azyuf/AaT/wCJoAqf8J34P/6GvQ//AAYw/wDxVH/Cd+D/APoa9D/8GMP/AMVV6OVZM4DgjqHQqfyIp9AGd/wnfg//AKGvQ/8AwYw//FUf8J34P/6GvQ//AAYw/wDxVaNFAGd/wnfg/wD6GvQ//BjD/wDFUf8ACd+D/wDoa9D/APBjD/8AFVo0UAZ3/Cd+D/8Aoa9D/wDBjD/8VVW78ceEmubEr4p0QhZyWI1CLgeW4yfm9SPzrbooA56fXfhvdXz31xqvhSW7kjMTzyXFu0jIRgqWJyQQSMdMVcm8aeCri3e3m8TeH5IZFKPG9/CVZSMEEFsEY7Vq0UAZUnjTwTMEEviXw+4Rg6Br+E7WHQj5uCKq3PiH4d3unJp11rHhaexjChLaW5t2iXb0wpOBjtW/RQBy51H4Wm0htDeeDvs0DmSKHzbbZG56sq5wD7irR8R/Dw6qNVOteF/7RC7Bd/arfztuMY35zjHvW9RQBhWHiT4e6V539na14Ys/OffL9nurePzG9W2kZPuauf8ACd+D/wDoa9D/APBjD/8AFVo0UAZ3/Cd+D/8Aoa9D/wDBjD/8VR/wnfg//oa9D/8ABjD/APFVo0UAZ3/Cd+D/APoa9D/8GMP/AMVR/wAJ34P/AOhr0P8A8GMP/wAVWjRQB5bf+KvDr6lduuvaWytPIQReRkEbjz1qv/wlHh//AKDumf8AgXH/AI161RQB5L/wlHh//oO6Z/4Fx/40f8JR4f8A+g7pn/gXH/jXrVFAHkv/AAlHh/8A6Dumf+Bcf+NH/CUeH/8AoO6Z/wCBcf8AjXrVFAHkv/CUeH/+g7pn/gXH/jR/wlHh/wD6Dumf+Bcf+NetUUAeS/8ACUeH/wDoO6Z/4Fx/40f8JR4f/wCg7pn/AIFx/wCNetUUAYI8ceEQW/4qjRclj/y/xev+9QPG/g9RhfE+iAZJwL6LqeT/ABVvUUAebeDruOfTtWnt5Ulhl1y+eOSNgyuplyCCOCCD1rxHXT/xUGpf9fUv/oZr6UaPfqOpn/p7H/oiGvmzXB/xUGpf9fUv/oZrpf8ACX9dzl/5ffJ/ofRFuP8Ai30P/YGX/wBJ666uSt/+SfQ/9gZf/SeutrOr0NafUVGKM7jqsbEZ/Cqdnry/8Itp+r30Uxa5t4pHSztZZyGZQeEQM2P85q7GnmOyZxujYZ/KuWfwbrFxoVnpF5q+k3NpZCMRRSaS5jlCKVAmQzkSDBzj5fmVT2xWXf5frc1NWfxvoMEULm5nbzV3KFtJiVyxQK+E/dsXUqA2CWBAGeKzY/G8s0/h25WzEem6natNcmVXWS3JaNUIyBld0mCcDIIYcda1p8O5bUacE1O1jFqTua208QuoMrSFInV8oh3bSjb1IA4BrY07wq1vbafBf3cV2tpp0mnuqQGNZUYpgkF2wdqAHnkknjpTX9fj/wAABLbxfbRWIm1RbiMmeePzILKeSJEjmZAXdVZU4UEliO54FOk8d+HopZI2vJ8xlwxWynK/I21yGCYIU8MQcLkZIyK527+F8lzpsdm+q2l0Viki87UdO+0PGWleQyRfvFEch34LYOdinjGK2pfBXmWc1v8A2hjzLa9g3eT0+0SiTP3v4cYx39qF/n/wAZrw+ItMn1U6bHNKbgMyBjbyCJmUZZVlK7GYc5UMSMHjg41K5DTfAdrpniQ6nEum+X58lwrf2an2ovJncGuCSSuWYgBQcYG4gEHr6XRB1CiiigAooooAKKKKAMOfxp4VtbiW3uPEujQzxOUkjkv4lZGBwQQWyCDxitK6/wBZH9G/pWLeeA/Dl/4xtfFNxp0b6pbphX/hZhjY7L0LqBhT2z/srt2rr/WR/Rv6UARU241KHS7KOe5SbyC5Dyxxl1iGT8z45C+rYwOpwOadTbiznvrKOGK9ltIy581oABI65Pyhj93PcgZx0IPIAL0Usc8KTQyLJFIoZHQ5DA8ggjqKivr+z0yzkvL+7gtLWPG+aeQRouSAMseBkkD8afbW0VpbR28C7Yo12qCST+JPJPueTSzwQ3VvLb3EUc0EqFJI5FDK6kYIIPBBHGKGC8zP03xLoOs3DW+l63pt9OqF2jtbpJWC5AyQpJxkgZ9xUv8AE/8Avt/M1neFPBeheCrO4ttEtPJW4lMsruxd25O1Sx52qDgD8eSSTo/xP/vt/M0AZmo+J7LQ9X0601O/sbGzuLSSTzbqZYsupjAUFiB0ZjjrxUGn+NrCTSor68eQxTSzBLi0tJpoPLSVkVmkRWVRhQSSQO/ArXXTfM1ay1PzceTaPB5e3rvKNnOe2zpjvXJ6v8Nm1ON4/t9g4dZVBvdO+0GAvK8m6H94ojf58FsHOxemMULpf+tWDNuTx34eilkja8nzGXDFbKcr8jbXIYJghTwxBwuRkjIro65SXwV5lnNb/wBoY8y2vYN3k9PtEokz97+HGMd/auqAwAPSjoHUWiiigAooooAKKKKACiiigCpcf8fB/wBwfzNR1Jcf8fB/3B/M1HQAUUUUAFFFFAHOrp9pq+v6oNQiFwtu0aRI54UFAxwPqTTr/RfDGl2E17eWEEdvCu522MxA9gMkn2AzTbaZovEGtYOMyxf+ilqp4hk1C9NjY2TpGzzCZ5pbdpYkWPDAMAy8ltuBuHQ9cGt6lScWlF20X5I5aVGlNOU4pu76ebNOLw34enhSWLTrdo3UMrAHBB5BqO80Lw1YWc13c6fAkEKGSR9pOFAyTgc15zHPqtvc3mkPPq51HT7eGLTzZrOluz7pCjsFJTbt2A+YSvykZODT559Qv7G+W0k1i4uma/ju1l854GizIEWMN+7LbhGBsy2Mg96n21TpJ/eafVqPWC+5HpI8L6CRn+zIPyNZEq+DYdQmspLBhJA22aX7FOYYjtDfNLt8teCDy3GapaFPqJ126tJrq5e10tnVHeUt5xl2yKGOckxqduD/AHgaq3dtq6r4ovLW/wBShZ3cw2sMaBZT9nQBlOzzM5H8LDlfrQq1Rv4nt3BYaj/IvuR0lnpPhXUN/wBltLWUxsVdRkMuGZTkHnG5GGehwcVa/wCEW0H/AKBkH5GvNYrbVrRtbks01GC8uY8h9k7J5X2uUybQCBv2EEBSGOcr1JrU8PQ6pNqdqt3f6tLYRCaSMutxbAsGi2hhJI0jDPmYEh5+bgrihVqjduZ/eJ4aj/IvuR23/CLaD/0DIPyNJ4dHlR6hbKzGK3vGiiBOdq7VOPzJqb7U/wDe/Sq/hxix1Zj3v2/9ASq55Sg+Z32J9nCFWPIkt9vkTSWltfa7cLeQR3CQ20JjSVQyqWaTcdp4ydi8+1Z+oXfhXTb1rOfSFedY1lZbbRpLgKpJAJMcbAZ2nr6VfkkMet3hBxm2t/8A0KauavrLVLzxJqc1nq17p26yhRHihiZJHBl670bOMjhSOv0rnex1I6i30zQrq2iuINM06SGVA6OtsmGUjIPSpf7F0f8A6BOn/wDgKn+FeW2za8NQsjHJqFhHGlutrZxWlw6JEEUOrN5qxAg+YD5il8YK5O2rdjba1AlpP9p1p5ljsJCJrmZlMjORPuUnBG0DKkYXrgHmqfkI9H/sXR/+gTp//gKn+FH9i6P/ANAnT/8AwFT/AArzywXWLmGdILnWYrv7IXumupJVT7YrhlEe7jYcOCI/kKkA9qr6lL4gvLKzvpbjUrK3vZJZ5oUhuZZIDhRDHsgkR1G0MTzt3E5HINL+v6/rt3Dqegalb+HdJsmu7zS7NYgyp+7sBKxZiFUBUUsSSQOBVOO88IOiM9lZW5ckbLrT/IdcKzZZZEBVcIx3EAHacGszVF1K88JafbyT3P23zrQyTJCgkUiRCzlfnUEYJP3gPcVzni3T9XunmtHm1TUYUt8rKVwS3lXOceUqrnlF4Az8oOc8u1rgtT0z+xdG/wCgTp//AIDJ/hS/2Lo//QJ0/wD8BU/wrze4k1x/EUckF7qcUAMH2KIWt04aDau7zGMqxq2d+fNUv0IydorrPDC3Fn4es/tE15JdSwpJObuV5H8wqN33ydvI6DA68daLaXA17G3hs9ZvILWNYYDbwy+UgwgYtICQOgyFXp6Vp1l2TmTWrlicn7JB/wCjJq1KQBRRRQAUUUUAYcYzf6p/19j/ANERV8z65/yMGpf9fUv/AKGa+moh/p2qf9fY/wDREVfMuuf8jBqX/X1L/wChmul/wl/Xc5f+X/yf6H0Pb/8AJPof+wMv/pPXW1yVv/yT6H/sDL/6T11tZ1ehrT6gCVYMpIIp/my/89D+QplFZGo/zZf+eh/IUebL/wA9D+QplFAD/Nl/56H8hR5sv/PQ/kKZRQA/zZf+eh/IUebL/wA9D+QplFAD/Nl/56H8hR5sv/PQ/kKZRQA/zZf+eh/IUebL/wA9D+QplFAD/Nl/56H8hR5sv/PQ/kKZRQA/zZf+eh/IUxizsGZiSBgUUUAFKrugwrkDOccUlFAD/Nl/56H8hR5sv/PQ/kKZRQA/zZf+eh/IVGBj+dLRQAAlRgMwHoCaXc399/8Avo0lFAC7m/vv/wB9Gjc399/++jSUUALub++//fRo3N/ff/vo0lFAC7m/vv8A99Gjc399/wDvo0lFAC7m/vv/AN9Gjc399/8Avo0lFAC7m/vv/wB9Gjc399/++jSUUAHcnkk9zRRRQAUUUUAFFFFAHP3ukaiuqXF3YPbMtxtLpOzLhgoXjAOeAKi+weIP7mmf9/pP/iK6WitPavqkY+wjfRtfM5caXrizNMINKErgKziR9zAZwCdnbJ/M0RaXrkCbIoNKjXJbakjgZJyT9zuSTXUUUe08kHsF/M/vOUt9G1izRktrTR4FdzIyxO6gsTkscJ1J6mpvsHiD+5pn/f6T/wCIrpaKPaeSD2C/mf3nNfYPEH9zTP8Av9J/8RR9g8Qf3NM/7/Sf/EV0tFHtPJB7BfzP7zmvsHiD+5pn/f6T/wCIrT0XTpdOtZRPIrzzymaQp90EgDA/ACtKih1G1aw40YxlzXbfmzKvbK7a/a5tlglWSJY3SSQxkFSxBBCt/fYEY9Kh+yal/wA+dr/4HN/8ZrborM1MT7JqX/Pna/8Agc3/AMZo+yal/wA+dr/4HN/8ZrbooAxPsmpf8+dr/wCBzf8Axmj7JqX/AD52v/gc3/xmtuigDE+yal/z52v/AIHN/wDGaPsmpf8APna/+Bzf/Ga26KAMT7JqX/Pna/8Agc3/AMZo+yal/wA+dr/4HN/8ZrbooAztOs7mG5nubkRI0iJEkcTlwqqWOSxAySXPYdq0aKKACiiigAooooAxoP8Aj+1X/r7H/oiKvmTXP+Rg1L/r6l/9DNfTlv8A8f2qf9fY/wDREVfMeuf8jBqX/X1L/wChmul/wl/Xc5f+X3yf6H0XpMYvvBNlbxyKPN0xIN3UK3lBT+RrU+26p/z5WH/gY/8A8arx+/8AE+qeE3aLTZlMDsW8qVdyqc8kemc1nf8AC2vEvpZf9+T/AI1caaqRTMamJ9jNxZ7j9t1T/nysP/Ax/wD41R9t1T/nysP/AAMf/wCNV4d/wtrxL6WX/fk/40f8La8S+ll/35P+NV9WRH9oLse4/bdU/wCfKw/8DH/+NUfbdU/58rD/AMDH/wDjVeHf8La8S+ll/wB+T/jR/wALa8S+ll/35P8AjR9WQf2gux7j9t1T/nysP/Ax/wD41R9u1T/nzsP/AAMf/wCNV4d/wtrxL6WX/fk/40f8La8S+ll/35P+NH1ZB/aC7HuP27VP+fKx/wDAx/8A41R9u1T/AJ8rH/wMf/41Xh3/AAtrxL6WX/fk/wCNH/C2vEvpZf8Afk/40fVkH9oLse4/btU/587H/wADH/8AjVH2/U/+fOx/8DH/APjVeHf8LZ8S+ll/35P+NH/C2fEnpZf9+T/jR9WQf2hHse4/b9T/AOfOx/8AAx//AI1Sfb9T/wCfOx/8DH/+NV4f/wALZ8Sell/35P8AjR/wtnxJ6WX/AH5P+NH1ZB/aEex7h9v1P/nzsf8AwMf/AONUv27VP+fOw/8AAx//AI1Xh3/C2fEnpZf9+T/jR/wtrxL6WX/fk/40fVkH9oLse5fbdU/587D/AMDH/wDjVH23VP8AnzsP/Ax//jVeHf8AC2/EvpZf9+T/AI0f8Lb8S+ll/wB+T/jR9WQf2gux7j9t1T/nysP/AAMf/wCNUn23VP8AnysP/Ax//jVeH/8AC2/EvpZf9+T/AI0n/C2vEvpZf9+T/jR9WQf2gux7j9t1T/nysP8AwMf/AONUfbtU/wCfKx/8DH/+NV4f/wALa8S+ll/35P8AjSf8La8S+ll/35P+NH1ZB/aEex7j9u1P/nzsf/Ax/wD41Sfb9T/587H/AMDH/wDjVeH/APC2fEvpZf8Afk/40f8AC2fEnpZf9+T/AI0fVkH9oR7HuH2/U/8Anzsf/Ax//jVH9oan/wA+dj/4GP8A/Gq8O/4Wx4k9LL/vyf8AGj/hbHiT+7Zf9+T/AI0fVkH9oR7HuH9o6l/z52P/AIGP/wDGqP7R1L/nzsf/AAMf/wCNV4f/AMLX8R/3bL/vyf8AGk/4Wv4j/u2X/fk/40fVkH9oR7HuP9pal/z52P8A4GP/APGqT+0tS/587H/wMf8A+NV4f/wtbxH/AHbL/vyf/iqP+FreIv7tl/35P/xVH1ZB/aEex7gNS1I/8udj/wCBj/8Axqni91Q9LOw/8DH/APjVeGf8LW8Rf3bL/vyf/iqcPi14lHQWX/fk/wCNL6sg/tCPY9y+2ap/z5WH/gY//wAao+2ar/z5WH/gY/8A8arw7/hbfib0sv8Avyf8aP8Ahbfib0sv+/J/xo+rIP7Qj2Pcftmq/wDPlYf+Bj//ABqj7Zqv/PlYf+Bj/wDxqvDv+Ft+Jv8Apy/78n/Gj/hbfib/AKcv+/J/xo+rIP7Qj2Pcftmq/wDPlYf+Bj//ABqj7Zqv/PlYf+Bj/wDxqvDv+Ft+Jv8Apy/78n/Gj/hbfib/AKcv+/J/xo+rIf8AaEex7j9s1X/nysP/AAMf/wCNUfbNV/58rD/wMf8A+NV4d/wtvxN/05f9+T/jR/wtvxN/05f9+T/jR9WQf2hHse4/bNV/58rD/wADH/8AjVH2zVf+fKw/8DH/APjVeHf8Lb8Tf9OX/fk/40f8Lb8Tf9OX/fk/40fVkH9oR7HuP2zVf+fKw/8AAx//AI1R9s1X/nysP/Ax/wD41Xh3/C2/E3/Tl/35P+NH/C2/E3/Tl/35P+NH1ZB/aEex7j9s1X/nysP/AAMf/wCNUfbNU/58rD/wMf8A+NV4d/wtvxN/05f9+T/jR/wtvxN6WX/fk/40fVkL+0I9j3H7bqn/AD5WH/gY/wD8apPt2qf8+dh/4GP/APGq8P8A+Ft+JfSy/wC/J/xpP+FteJfSy/78n/Gj6sg/tCPY9x+3an/z52P/AIGP/wDGqT7fqf8Az52P/gY//wAarw//AIWz4l9LL/vyf8aP+Fs+JPSy/wC/J/xp/VkH9oR7HuH2/U/+fOx/8DH/APjVH2/U/wDnzsf/AAMf/wCNV4f/AMLZ8Sell/35P+NH/C2fEnpZf9+T/jR9WQf2hHse4fb9T/587H/wMf8A+NUf2hqf/PnY/wDgY/8A8arw/wD4Wz4k9LL/AL8n/Gk/4Wx4k9LL/vyf8aPqyD+0I9j3D+0dS/587H/wMf8A+NUn9p6kP+XOx/8AAx//AI1XiH/C1/Ef92y/78n/ABpP+FreIj/DZf8Afk//ABVH1ZB/aEex7gNT1EnH2Ox/8C3/APjVSC81Q/8ALnYf+Bj/APxqvCv+Fq+Iv7ll/wB+j/8AFVIPi34lAwBZf9+T/jS+rIf9oR7HuX2vVf8AnysP/Ax//jVH2vVf+fKw/wDAx/8A41Xhv/C3PE3/AE5f9+T/AI0f8Lc8Tf8ATl/35P8AjR9WQf2hHse5fa9V/wCfKw/8DH/+NUfa9V/58rD/AMDH/wDjVeHf8Lc8Tf8ATl/35P8AjR/wtzxN/wBOX/fk/wCNH1ZB/aEezPcfteq/8+Vh/wCBj/8Axqk+26p/z5WH/gY//wAarw//AIW54m/6cv8Avyf8aT/hbfiU9rL/AL8n/Gj6sg/tCPY9w+26p/z5WH/gY/8A8ao+26p/z5WH/gY//wAarw7/AIW14l9LL/vyf8aP+FteJfSy/wC/J/xp/VkL+0F2Pcftuqf8+Vh/4GP/APGqPtuqf8+Vh/4GP/8AGq8O/wCFteJfSy/78n/Gj/hbXiX0sv8Avyf8aPqyD+0F2PbbdJYzdz3PlI80vmlY3LKgCIvUgZ+5np3r5p1m3kk13UHUAq1zIQQRyNxrqh8RPEGtA2c00UMb8M0CbWx9cmrUen2yxoAnAAFZVnyLkRvQftX7Xpsf/9k=)

# Next Steps<br />後續步驟

I`ve shown you a simple application that uses my filtering implementation, but there are various ways that the application could be enhanced. For example, I may want to add controls that interact with the search functionality.

The filtering algorithm that is used in this sample depends on a type implementing the `IComparable` interface. If I sort or filter by a property that does not implement the `IComparable` interface, an exception will be thrown. This logic works well if I am filtering custom business objects whose properties are mainly simple types. To take the functionality in this sample a step further, I could consider implementing the `IComparable` interface and a `TypeConverter` for any properties of my custom business object that are complex types.

Also, as stated earlier, this code was designed to work with the `DataGridView` auto filter feature shown in [Building a Drop-Down Filter List for a DataGridView Column Header Cell](http://go.microsoft.com/fwlink/?LinkId=105803). Using this code, I have a data source that is capable of interacting with this popular Excel-like feature. You can see this interaction by downloading the sample solution, which includes the filtered binding list and accompanying unit tests, the employee object, the sample form shown in this article as well as a Windows Form that uses the auto-filtering `DataGridView`.

Alternatively, if my application does not require a generic filtering solution and I am using the .NET Framework 3.5, I can use LINQ on the client-side to perform complex filtering on a list of custom objects.

# Conclusion<br />總結

In this article, I showed you one way to implement filtering. I created a filtered class that extended the generic binding list and implemented the `IBindingListView` interface. Next, I implemented the `SupportsFiltering`, `Filter`, and `RemoveFilter` members. To use the filter, I set the `Filter` property on the binding source. To test the filter, I created a simple Windows Forms application. I also suggested ways that the filtering implementation and the corresponding application could be improved. Now, you can use this filtering implementation and unit tests to start building your own custom filtering solution.
