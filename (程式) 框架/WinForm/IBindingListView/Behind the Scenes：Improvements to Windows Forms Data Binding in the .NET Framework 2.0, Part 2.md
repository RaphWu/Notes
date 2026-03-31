---
aliases:
date: 2008-01-18
author: Cheryl Simmons
language: C#
sourceurl: https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736(v=msdn.10)
tags:
---

# Behind the Scenes: Improvements to Windows Forms Data Binding in the .NET Framework 2.0, Part 2

幕後：.NET Framework 2.0 中 Windows Forms 資料綁定之改進，第二部分

- 01/18/2008

Cheryl Simmons  雪莉·史密斯
Microsoft Corporation

August 2006  2006 年 8 月

Applies to:  適用於：
   Microsoft Visual Studio 2005
   Microsoft .NET Framework 2.0
   Microsoft Windows Forms

`Summary:` Learn about the generic `BindingList` and how to extend this generic collection type to add sorting and searching functionality. (11 printed pages)
摘要：了解通用 BindingList，以及如何擴展這個通用集合類別以增加排序和搜尋功能。(11 頁印刷頁面)

Download code samples in C# and Visual Basic (29 KB) at the [Microsoft Download Center](https://go.microsoft.com/fwlink/?linkid=70372&clcid=0x409).
從 Microsoft 下載中心下載 C# 和 Visual Basic 的程式碼範例 (29 KB)。

Review [Part 1](https://go.microsoft.com/fwlink/?linkid=69696&clcid=0x409) of this two-part article.
檢閱本兩部分文章的第一部分。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#contents)

#### Contents  內容

Introduction  介紹
About the Generic BindingList
關於 Generic BindingList
Overview of the Code Sample
程式碼範例的概觀
Searching the Generic BindingList
搜尋泛型 BindingList
Sorting the Generic BindingList
排序泛型 BindingList
Next Steps  下一步
Conclusion  結論

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#introduction)

## Introduction  介紹

One of the most talked about new features for Windows Forms data binding is the `BindingSource` component. This component greatly simplifies the developer's job by providing currency management, change notification, and the ability to easily access the members in a bound list. As long as the data source implements the `IBindingList` interface and has searching and sorting implemented, you can use the `BindingSource` to search and sort the data source. The `DataView` class is an example of a type that implements `IBindingList`. However, sometimes you won't be working with a class that implements `IBindingList` and will have to create and bind to a list of custom business objects by providing your own implementation of the `IBindingList` interface. To help with these situations, the Microsoft .NET Framework introduces the generic `BindingList` type, aka `BindingList` of `T`. This article shows how to use the generic `BindingList` in your data-bound applications to provide searchable, sortable lists of custom business objects.
Windows Forms 資料綁定最受討論的新功能之一是 BindingSource 元件。這個元件透過提供金錢管理、變更通知以及輕鬆存取綁定清單中的成員，大大簡化了開發者的工作。只要資料來源實作了 IBindingList 接口，並且已實作了搜尋和排序，您就可以使用 BindingSource 來搜尋和排序資料來源。DataView 類別就是實作了 IBindingList 接口的一個例子。然而，有時候您可能不會在處理實作了 IBindingList 的類別，而必須建立並綁定自訂商業對象的清單，同時提供您自己的 IBindingList 接口實作。為了幫助處理這些情況，Microsoft .NET Framework 引入了泛型 BindingList 類型，也就是 BindingList of T。本文說明如何在您的資料綁定應用程式中使用泛型 BindingList，以提供可搜尋、可排序的自訂商業對象清單。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#about-the-generic-bindinglist)

## About the Generic BindingList

關於 Generic BindingList

The generic `BindingList` is a generic implementation of the `IBindingList` interface. The `IBindingList` interface is an extension of the `IList` interface, a key interface for Windows Forms data binding. In addition to features inherited from the `IList` interface, classes that implement `IBindingList` can optionally support sorting, searching, and change notification. The `IBindingList` interface is a fairly complex interface and by providing a generic implementation with the generic `BindingList` type, building bindable lists of custom business objects for consumption by the Windows Forms data binding engine has become much easier. When you use the generic `BindingList` to store your business objects in a data-binding scenario, the list allows for easy insertion and removal from the list and automatically provides change notification when changes are made to the list. Optionally you can implement searching and sorting on the generic `BindingList` by overriding a few properties and methods.
Generic BindingList 是 IBindingList 介面的泛型實現。IBindingList 介面是 IList 介面的擴充，而 IList 介面是 Windows Forms 資料綁定中的關鍵介面。除了繼承自 IList 介面的功能外，實現 IBindingList 的類別可以選擇支援排序、搜尋和變更通知。IBindingList 介面相當複雜，透過提供 Generic BindingList 類別的泛型實現，讓建構可綁定的自訂商業物件清單，以供 Windows Forms 資料綁定引擎使用變得更加容易。當您在資料綁定場景中使用 Generic BindingList 存放您的商業物件時，該清單允許輕鬆地插入和移除，並在清單變更時自動提供變更通知。您可以選擇透過覆寫一些屬性和方法，在 Generic BindingList 上實現搜尋和排序。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#overview-of-the-code-sample)

## Overview of the Code Sample

程式碼範例概觀

To demonstrate how to add searching and sorting capability to a data source, I created a simple code sample. Imagine you have a list that contains several `Employee` objects. The ability to search the list for a particular employee, by name, salary or start date is functionality that you want to offer in your application. In addition to searching the list, users of your application will want to sort the list of Employees by name, salary, hire date, and so on. As you will see in this example, the implementation of sorting on the generic `BindingList` bound to a `DataGridView` control allows the `DataGridView` to offer sorting without additional code.
為了展示如何為數據來源添加搜尋和排序功能，我創建了一個簡單的程式碼範例。想像一下，您有一個包含多個 `Employee` 物件的清單。搜尋清單中特定員工的能力，根據姓名、薪資或開始日期，是您希望在應用程式中提供的功能。除了搜尋清單之外，您的應用程式使用者還希望根據姓名、薪資、聘用日期等對員工清單進行排序。從這個範例中，您將看到，針對繫結到 DataGridView 控件的泛型 BindingList 進行排序的實現，允許 DataGridView 無需額外程式碼即可提供排序功能。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#introducing-the-employee-business-object)

### Introducing the Employee Business Object

介紹員工業務物件

The code sample uses a simple `Employee` business object. The following is the implementation of the `Employee` type:
程式碼範例使用了一個簡單的 `Employee` 業務物件。以下是 `Employee` 類型的實現：

Copy  複製

```csharp
public class Employee
{
    private string lastNameValue;
    private string firstNameValue;
    private int salaryValue;
    private DateTime startDateValue;

    public Employee() 
    {
        lastNameValue = "Last Name";
        firstNameValue = "First Name";
        salaryValue = 0;
        startDateValue = DateTime.Today;
    }

    public Employee(string lastName, string firstName, 
        int salary, DateTime startDate)
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

    public override string ToString()
    {
        return LastName + ", " + FirstName + "\n" +
        "Salary:" + salaryValue + "\n" +
        "Start date:" + startDateValue;
    }
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#extending-the-generic-bindinglist)

### Extending the Generic BindingList

擴充泛型 BindingList

In the code sample, to implement the searching and sorting capability for the list of `Employee` objects, I created a type named `SortableSearchableList` based on `BindingList`. Because the methods and properties you have to override to implement searching and sorting are protected, you must extend the generic `BindingList`. Also, I chose to make the `SortableSearchableList` type a generic list, so that you use the `<T>` syntax in the class declaration. Making the extended list generic lets you reuse the list in multiple situations.
在程式碼範例中，為了實現對 `Employee` 物件的清單的搜尋和排序功能，我建立了一個名為 `SortableSearchableList` 的類別，基於 BindingList。因為實現搜尋和排序必須覆寫的方法和屬性是受保護的，所以您必須擴展泛型 BindingList。此外，我選擇將 `SortableSearchableList` 類別設為泛型清單，以便在類別宣告中使用 語法。讓擴展的清單成為泛型，可以讓您在多種情況下重用這個清單。

Copy  複製

```csharp
public class SortableSearchableList<T> : BindingList<T>
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#searching-the-generic-bindinglist)

## Searching the Generic BindingList

搜尋泛型 BindingList

Implementing search on a generic `BindingList` requires various steps. First, you have to indicate that searching is supported by overriding the `SupportsSearchingCore` property. Next, you have to override the `FindCore` method, which performs the search. Finally, you expose the search functionality.
在泛型 BindingList 上實現搜尋需要多個步驟。首先，您必須透過覆寫 SupportsSearchingCore 屬性來表示支援搜尋。接下來，您必須覆寫執行搜尋的 FindCore 方法。最後，您必須公開搜尋功能。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#indicating-search-is-supported)

### Indicating Search is Supported

表示支援搜尋

The `SupportsSearchingCore` property is a protected read-only property that you have to override. The `SupportsSearchingCore` property indicates whether the list supports searching. By default this property is set to `false`. You set this property to `true` to indicate searching is implemented on your list.
支援搜尋核心屬性是一個受保護的唯讀屬性，您必須覆寫它。支援搜尋核心屬性表示清單是否支援搜尋。預設此屬性設定為 false。您將此屬性設為 true 以表示您的清單已實現搜尋功能。

Copy  複製

```csharp
protected override bool SupportsSearchingCore
{
    get
    {
        return true;
    }
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#implementing-the-search-funtionality)

### Implementing the Search Funtionality

實現搜尋功能

Next, you have to override the `FindCore` method and provide an implementation to search the list. The `FindCore` method searches a list and returns the index of a found item. The `FindCore` method takes a `PropertyDescriptor` object that represents the property or column of the data source type to search for and a key object, which represents the value to search for. The following code shows a simple implementation of case-sensitive search functionality.
接下來，您必須覆寫 FindCore 方法，並提供搜尋清單的實現。FindCore 方法搜尋清單並傳回找到項目的索引。FindCore 方法接受一個 PropertyDescriptor 物件，該物件代表要搜尋的資料來源類型屬性或欄位，以及一個代表要搜尋值的鍵物件。以下程式碼顯示了一個區分大小寫的搜尋功能簡單實現。

Copy  複製

```csharp
protected override int FindCore(PropertyDescriptor prop, object key)
{
    // Get the property info for the specified property.
    PropertyInfo propInfo = typeof(T).GetProperty(prop.Name);
    T item;

    if (key != null)
    {
        // Loop through the items to see if the key
        // value matches the property value.
        for (int i = 0; i < Count; ++i)
        {
            item = (T)Items[i];
            if (propInfo.GetValue(item, null).Equals(key))
                return i;
        }
    }
    return -1;
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#exposing-the-search-functionality)

### Exposing the Search Functionality

公開搜尋功能

Finally, you can expose the search functionality with a public method, or call the search functionality on the underlying `IBindingList` using its `Find` method. In order to simplify calls to the search functionality, the publicly exposed `Find` method shown in the following code takes a takes a string and a key to search for. Next, the code attempts to convert the property string to a `PropertyDescriptor` object. If the conversion is successful, the `PropertyDescriptor` and the key are passed to the `FindCore` method.
最後，您可以透過一個公開方法來暴露搜尋功能，或是在底層的 IBindingList 上呼叫其 Find 方法來使用搜尋功能。為了簡化對搜尋功能的呼叫，以下程式碼中公開的 `Find` 方法接受一個字串和一個搜尋鍵。接著，程式碼嘗試將屬性字串轉換為 PropertyDescriptor 物件。如果轉換成功，則將 PropertyDescriptor 和鍵傳遞給 FindCore 方法。

Copy  複製

```csharp
public int Find(string property, object key)
{
    // Check the properties for a property with the specified name.
    PropertyDescriptorCollection properties = 
        TypeDescriptor.GetProperties(typeof(T));
    PropertyDescriptor prop = properties.Find(property, true);

    // If there is not a match, return -1 otherwise pass search to
    // FindCore method.
    if (prop == null)
        return -1;
    else
       return FindCore(prop, key);
}
```

Figure 1 shows the search functionality in action. When you enter a last name and then click the `Search` button, the search functionality identifies the first row where the last name appears in the grid, if it is found.
圖 1 顯示了搜尋功能的運作方式。當您輸入姓氏然後點擊搜尋按鈕時，搜尋功能會識別在資料表中出現姓氏的第一個資料列，如果找到的話。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480736.wfbindp201\(en-us,msdn.10\).gif)

`Figure 1. Search implemented by using the generic BindingList
圖 1. 使用泛型 BindingList 實現的搜尋`

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#sorting-the-generic-bindinglist)

## Sorting the Generic BindingList

對泛型 BindingList 進行排序

In addition to searching the list, users of your application will want to sort the list of `Employees` by name, salary, hire date, and so on. You can implement sorting on the generic `BindingList` by overriding the `SupportsSortingCore` and `IsSortedCore` properties and the `ApplySortCore` and `RemoveSortCore` methods. Optionally, you can override the `SortDirectionCore` and `SortPropertyCore` properties to return the sort direction and property, respectively. Finally, if you want to ensure new items added to the list are sorted, you will have to override the `EndNew` method.
除了搜尋清單之外，您的應用程式使用者也會希望根據姓名、薪資、聘用日期等來排序 `Employees` 的清單。您可以透過覆寫 SupportsSortingCore 和 IsSortedCore 屬性，以及 ApplySortCore 和 RemoveSortCore 方法來在泛型 BindingList 上實現排序。選擇性地，您可以覆寫 SortDirectionCore 和 SortPropertyCore 屬性來分別傳回排序方向和屬性。最後，如果您想要確保新增到清單中的項目是已排序的，您將必須覆寫 EndNew 方法。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#indicating-sorting-is-supported)

### Indicating Sorting Is Supported

支援排序

The `SupportsSortingCore` property is a protected read-only property that you have to override. The `SupportsSortingCore` property indicates whether the list supports sorting. You set this property to `true` to indicate sorting is implemented on your list.
SupportsSortingCore 屬性是一個受保護的唯讀屬性，您必須覆寫它。SupportsSortingCore 屬性表示清單是否支援排序。您將此屬性設為 true，以表示您的清單已實現排序功能。

Copy  複製

```csharp
protected override bool SupportsSortingCore
{
    get { return true; }
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#indicating-sorting-has-occurred)

### Indicating Sorting Has Occurred

已發生排序

The `IsSortedCore` property is a protected read-only property that you have to override. The `IsSortedCore` property indicates whether the list has been sorted.
IsSortedCore 屬性是一個受保護的唯讀屬性，您必須覆寫它。IsSortedCore 屬性表示清單是否已排序。

Copy  複製

```csharp
bool isSortedValue;
protected override bool IsSortedCore
{
    get { return isSortedValue; }
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#applying-the-sort)

### Applying the Sort  應用排序

Next, you have to override the `ApplySortCore` method and provide the actual sorting details. The `ApplySortCore` method takes a `PropertyDescriptor` object that identifies the list item property to sort by, and a `ListSortDescription` enumeration that indicates whether the list should be sorted in ascending or descending order.
接下來，您必須覆寫 ApplySortCore 方法，並提供實際的排序細節。ApplySortCore 方法接受一個 PropertyDescriptor 物件，用於識別要根據其排序的清單項目屬性，以及一個 ListSortDescription 列舉，用於指示清單是否應按升序或降序排序。

In the following code, the list is sorted only if the selected property implements the `IComparable` interface, which provides a `CompareTo` method. Most simple types, such as strings, integers, and `DateTime` types, implement this interface. This is a reasonable decision because all the properties of the `Employee` object are simple types. In addition, the implementation provides some simple error handling to indicate when a user attempts to sort the data source by a property that is not a simple type.
在以下程式碼中，清單僅在選定的屬性實現了 IComparable 接口時才會排序，該接口提供 CompareTo 方法。大多數簡單類型，例如字串、整數和 DateTime 類型，都實現了這個接口。這是一個合理的決定，因為 `Employee` 物件的所有屬性都是簡單類型。此外，實現還提供了一些簡單的錯誤處理，以指示當使用者嘗試根據一個非簡單類型的屬性來排序資料來源時的情況。

In the following code, a copy of the unsorted list is saved for use by the `RemoveSortCore` method. Then the values of the specified property are copied into an `ArrayList`. A call is made to the `ArrayList` `Sort` method, which in turn calls the `CompareTo` method of the array members. Finally, by using the values in the sorted array, the generic `BindingList` values are reorganized based on whether the sort is a descending or ascending sort. Finally, a `ListChanged` event is raised indicating the list has been reset, so bound controls will refresh their values. The following code shows an implementation of the sort functionality.
在以下程式碼中，會將未排序的清單複製一份供 RemoveSortCore 方法使用。接著，會將指定屬性的值複製到一個 ArrayList 中。接著呼叫 ArrayList 的 Sort 方法，這個方法會呼叫陣列成員的 CompareTo 方法。最後，利用排序後陣列中的值，根據是降序排序還是升序排序，重新組織泛型 BindingList 的值。最後會引發一個 ListChanged 事件，表示清單已重設，以便綁定的控制項能夠更新其值。以下程式碼展示了一個排序功能的實作。

Copy  複製

```csharp
ListSortDirection sortDirectionValue;
PropertyDescriptor sortPropertyValue;

protected override void ApplySortCore(PropertyDescriptor prop, 
    ListSortDirection direction)
{
    sortedList = new ArrayList();

    // Check to see if the property type we are sorting by implements
    // the IComparable interface.
    Type interfaceType = prop.PropertyType.GetInterface("IComparable");

    if (interfaceType != null)
    {
        // If so, set the SortPropertyValue and SortDirectionValue.
        sortPropertyValue = prop;
        sortDirectionValue = direction;

        unsortedItems = new ArrayList(this.Count);

        // Loop through each item, adding it the the sortedItems ArrayList.
        foreach (Object item in this.Items) {
            sortedList.Add(prop.GetValue(item));
            unsortedItems.Add(item);
        }
        // Call Sort on the ArrayList.
        sortedList.Sort();
        T temp;

        // Check the sort direction and then copy the sorted items
        // back into the list.
        if (direction == ListSortDirection.Descending)
            sortedList.Reverse();

        for (int i = 0; i < this.Count; i++)
        {
            int position = Find(prop.Name, sortedList[i]);
            if (position != i) {
                temp = this[i];
                this[i] = this[position];
                this[position] = temp;
            }
        }

        isSortedValue = true;

        // Raise the ListChanged event so bound controls refresh their
        // values.
        OnListChanged(new ListChangedEventArgs(ListChangedType.Reset, -1));
    }
    else
        // If the property type does not implement IComparable, let the user
        // know.
        throw new NotSupportedException("Cannot sort by " + prop.Name +
            ". This" + prop.PropertyType.ToString() + 
            " does not implement IComparable");
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#removing-the-sort)

### Removing the Sort  拿掉排序

The next step is to override the `RemoveSortCore` method. The `RemoveSortCore` method removes the last sort applied to the list and raises a `ListChanged` event to indicate the list has been reset. The `RemoveSort` method exposes the remove sort functionality.
下一步是覆寫 RemoveSortCore 方法。RemoveSortCore 方法會移除列表中最后一次應用的排序，並引發 ListChanged 事件以表示列表已重置。 `RemoveSort` 方法揭露了移除排序的功能。

Copy  複製

```csharp
protected override void RemoveSortCore()
{
    int position;
    object temp;
    // Ensure the list has been sorted.
    if (unsortedItems != null)
    {
        // Loop through the unsorted items and reorder the
        // list per the unsorted list.
        for (int i = 0; i < unsortedItems.Count; )
        {
            position = this.Find("LastName", 
                unsortedItems[i].GetType().
                GetProperty("LastName").GetValue(unsortedItems[i], null));
            if (position > 0 && position != i)
            {
                temp = this[i];
                this[i] = this[position];
                this[position] = (T)temp;
                i++;
            }
            else if (position == i)
                i++;
            else
                // If an item in the unsorted list no longer exists,
                // delete it.
                unsortedItems.RemoveAt(i);
        }
        isSortedValue = false;
        OnListChanged(new ListChangedEventArgs(ListChangedType.Reset, -1));
    }
}

public void RemoveSort()
{
    RemoveSortCore();
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#exposing-sort-direction-and-property)

### Exposing Sort Direction and Property

揭露排序方向和屬性

Finally, override the protected `SortDirectionCore` and `SortPropertyCore` properties. The `SortDirectionCore` property indicates the direction of the sort, either ascending or descending. The `SortPropertyCore` property indicates the property descriptor used for sorting the list.
最後，覆寫受保護的 SortDirectionCore 和 SortPropertyCore 屬性。SortDirectionCore 屬性表示排序方向，為升序或降序。SortPropertyCore 屬性表示用於排序清單的屬性描述符。

Copy  複製

```csharp
ListSortDirection sortDirectionValue;
PropertyDescriptor sortPropertyValue;

protected override PropertyDescriptor SortPropertyCore
{
    get { return sortPropertyValue; }
}

protected override ListSortDirection SortDirectionCore
{
    get { return sortDirectionValue; }
}
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#exposing-the-sort-functionality)

### Exposing the Sort Functionality

公開排序功能

In the sample, I bound the custom generic `BindingList` of `Employees` (`SortableSearchableList`) to a `DataGridView` control. The `DataGridView` control detects whether sorting is implemented, and because it does, when a column of the `DataGridView` is clicked, the contents of the `DataGridView` are sorted by the contents of the column in ascending order. The user can click the column again to sort the items in descending order. The `DataGridView` makes calls to the list through the `IBindingList` interface.
在範例中，我將 `Employees` ( `SortableSearchableList` ) 的自訂泛型 BindingList 綁定到 DataGridView 控制項。DataGridView 控制項檢測是否實現排序，因為它實現了，所以當點擊 DataGridView 的某一列時，DataGridView 的內容會根據該列的內容進行升序排序。使用者可以再次點擊該列來進行降序排序。DataGridView 透過 IBindingList 接口呼叫清單。

Similar to the searching code, you could optionally expose a public method that calls `ApplySortCore`, or alternatively (like the `DataGridView` control), make calls to the underlying `IBindingList`.
與搜尋程式碼類似，您可以選擇公開一個呼叫 ApplySortCore 的公用方法，或者像 DataGridView 控制項一樣，呼叫底層的 IBindingList。

Copy  複製

```csharp
((IBindingList)employees).ApplySort(somePropertyDescriptor,
    ListSortDirection.Ascending);
```

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#ensuring-items-added-to-the-list-are-sorted)

### Ensuring Items Added to the List Are Sorted

確保新增至清單的項目是排序的

Since the `Employee` object exposes a default constructor, the `AllowNew` property of the `BindingList` returns `true`, allowing the user to add additional employees to the list. Since the user can add new items, you also want to ensure that items added to the list are added in sorted order, if a sort has been applied. To do this, I've overridden the `EndNew` method. In this method override, I check that the list has been sorted by ensuring the `SortPropertyValue` is not `null`, I check to see if the new item is being added to the end of the list (indicating the list needs to be resorted), and I then call `ApplySortCore` on the list.
由於 `Employee` 物件擁有預設建構子，BindingList 的 AllowNew 屬性回傳 true，允許使用者將額外的員工新增至清單。由於使用者可以新增新項目，您也想要確保若已套用排序，新增至清單的項目是依排序順序新增。為了達到這個目的，我覆寫了 EndNew 方法。在這個方法覆寫中，我檢查清單是否已排序，方法是確保 SortPropertyValue 不為 null，我檢查新項目是否被新增至清單的末尾（表示清單需要重新排序），然後我對清單呼叫 ApplySortCore 方法。

Copy  複製

```csharp
public override void EndNew(int itemIndex)
{
    // Check to see if the item is added to the end of the list,
    // and if so, re-sort the list.
    if (sortPropertyValue != null && itemIndex == this.Count - 1)
        ApplySortCore(this.sortPropertyValue, this.sortDirectionValue);

    base.EndNew(itemIndex);
}
```

Figure 2 shows the sort functionality in action. When you click a column in the `DataGridView`, the contents are sorted. If you click the `Remove sort` button, the last sort applied is removed.
圖 2 顯示了排序功能的使用情況。當您點擊 DataGridView 中的某一列時，內容會被排序。如果您點擊移除排序按鈕，則會移除最後應用的排序。

![](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/images/aa480736.wfbindp202\(en-us,msdn.10\).gif)

`Figure 2. Sort implemented by using the generic BindingList
圖 2. 使用通用 BindingList 實現的排序`

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#next-steps)

### Next Steps  下一步

As stated earlier, this is a simple example of how you could implement searching and sorting on the generic BindingList. The sorting algorithm that is used in this sample depends on a type implementing the `IComparable` interface. If you try to sort by a property that does not implement the `IComparable` interface, an exception will be thrown. This logic works well if you are sorting custom business objects whose properties are mainly simple types. To take the functionality in this sample a step further, you could consider implementing the `IComparable` interface for any properties of your custom business object that are complex types. For example, if your `Employee` object has an `Address` property that is of complex type `Address`, decide the business rules for sorting the `Address` type (by street name, number, and so on) and implement the `IComparable` interface and the `CompareTo` method on the `Address` type accordingly.
如前所述，這是一個簡單的範例，說明了如何在通用 BindingList 上實現搜尋和排序。此範例中使用的排序演算法取決於實現了 IComparable 介面的類型。如果您嘗試根據未實現 IComparable 介面的屬性進行排序，將會拋出例外。如果您正在排序屬性主要是簡單類型的自定義業務物件，這種邏輯運作良好。為了進一步擴展此範例的功能，您可以考慮為自定義業務物件的任何複雜類型屬性實現 IComparable 介面。例如，如果您的 `Employee` 物件有一個 `Address` 屬性，其類型為複雜類型 Address，請決定排序 Address 類型的業務規則（按街道名稱、編號等），並在 Address 類型上相應地實現 IComparable 介面和 CompareTo 方法。

Additionally, when a new item is added to the end of the list, the list is automatically resorted. Depending on the business logic behind your application, you could consider resorting the list regardless of where an item is added.
此外，當新的項目被加入到清單的末端時，清單會自動重新排序。根據您的應用程式的商業邏輯，您可以考慮無論項目被加入到哪裡都重新排序清單。

Finally, you might want to add filtering and multi-column sorting to your data source. In this case you could consider implementing the `IBindingListView` interface on your data source.
最後，您可能想要為您的數據來源添加過濾和多欄位排序。在這種情況下，您可以考慮在您的數據來源上實現 IBindingListView 介面。

[](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa480736\(v=msdn.10\)#conclusion)

### Conclusion  結論

The `BindingSource` component is a very useful addition to the Windows Forms data binding story. However, if you are working with lists of custom business objects, you might want to add searching and sorting capabilities. The new generic `BindingList` makes creating your own searchable and sortable binding lists of custom business objects more straightforward. For more information on these and other features of Windows Forms, see:
BindingSource 元件是 Windows Forms 數據綁定故事中非常實用的新增內容。然而，如果您正在處理自定義商業對象的清單，您可能想要添加搜尋和排序功能。新的泛型 BindingList 使創建自定義的、可搜尋和可排序的綁定清單更加簡單。有關這些以及其他 Windows Forms 功能的更多信息，請參考：

- [Windows Forms on Microsoft .NET Framework Developer Center  
    Microsoft .NET Framework 開發者中心中的 Windows Forms](https://go.microsoft.com/fwlink/?linkid=70365&clcid=0x409)
- [WindowsForms.NET](https://go.microsoft.com/fwlink/?linkid=70366&clcid=0x409)
- [Windows Forms Documentation Updates Blog  
    Windows Forms 文件更新部落格](https://go.microsoft.com/fwlink/?linkid=69202&clcid=0x409)
