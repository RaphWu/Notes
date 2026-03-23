---
aliases:
date: 2013-11-04
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-4-constructor/
tags:
  - CSharp
  - IoC
  - Autofac
---

# Autofac 筆記 4- 建構參數與建構式選擇

在先前的範例 ([1](http://blog.darkthread.net/post-2013-11-02-autofac-notes-2.aspx) [2](http://blog.darkthread.net/post-2013-11-03-autofac-notes-3-lifetime-scope.aspx))，透過 `Resolve<T>()` 建立的物件都只有單一建構式且不需建構參數，如果有多個建構式或建構時需要建構參數時，Autofac 會如何處理?

當類別有多個建構式時，Autofac 會依 "**能符合最多個容器提供參數的建構式優先**" 做為選擇依據。其 [英文原文](https://autofac.readthedocs.io/en/latest/advanced/constructor-selection.html#iconstructorselector) 為 "Autofac automatically chooses the constructor with the **most parameters that are able to be obtained from the container**"，坦白說並不容易理解，經過一些實驗我也才理出頭緒。
> 新版：
> For this, we can implement the `IConstructorSelector` interface. Autofac’s default implementation of this interface (`MostParametersConstructorSelector`) chooses the constructor with the **most parameters that are able to be obtained from the container** at the time of resolve.
> 為此，我們可以實作 `IConstructorSelector` 介面。 Autofac 對此介面的預設實作（ `MostParametersConstructorSelector` ）會在解析時選擇從容器中取得最多參數的建構子。

首先，容器提供建構式參數的來源有幾種:

1. 註冊時直接寫死，例如: `builder.Register(o => new Boo(1, 2, 3));`，這種情境下，建構參數固定，要使用哪一個建構式也固定。
2. `Resolve<T>` 時一併傳入參數物件，Autofac 提供的參數物件有 `NamedParameter(指定參數名稱)`、`TypedParameter(指定參數型別)`、`PositionalParameter(指定參數順序)` 等幾種選擇。
3. 透過 `builder.RegisterType<Boo>().WithParameters(new Parameters[] { new NamedParameter("i", 1) })`，提供部分或全部建構參數。
4. 若參數物件屬自定型別，而該型別經 `RegiterType<T>` 註冊，則建構時 Autofac 會自動以 `Resolve<T>()` 取得物件當成參數之一。

換言之，除了第一種建構參數及建構式固定不需選擇，Autofac 在建構物件時會由後面三種來源取得參數構成參數清單，以該清單比對所有建構式，過濾後保留建構參數全部在清單者，再由其中選擇一使用。當有多個建構式吻合，則以建構參數最多者為準先 (這就是所謂 " 符合最多個容器所能提供參數 ")，若比對結果最多符合參數的建構式有兩個以上，則 Autofac 無法做決定就會拋出錯誤。

還是佷抽象對吧? 用個複雜實例來釐清說明。假設有一個多建構式的類別如下:

```csharp
using System;
 
public class ArgW { }
public class ArgX { }
public class ArgY { }
public class ArgZ { }
 
public class MultiConstructor
{
    public MultiConstructor(ArgW w)
    {
        Console.WriteLine("Constructor ArgW");
    }
    public MultiConstructor(int i, int j, ArgY y)
    {
        Console.WriteLine("Constructor int, int, ArgY");
    }
    public MultiConstructor(ArgY y, ArgZ z)
    {
        Console.WriteLine("Constructor ArgY, ArgZ");
    }
    public MultiConstructor(ArgX x) {
        Console.WriteLine("Constructor ArgX");
    }
    public MultiConstructor(ArgW w, ArgX x)
    {
        Console.WriteLine("Constructor ArgW, ArgX");
    }
    public MultiConstructor(ArgX x, ArgZ z)
    {
        Console.WriteLine("Constructor ArgX, ArgZ");
    }
}
```

這個類別共有六個建構式，為了 `TypedParameter` 型別比對，宣告 `ArgW`, `ArgX`, `ArgY`, `ArgZ` 四個自訂型別作為建構式參數型別。`ContainerBuilder` 除了註冊 `MultiConstructor`，也註冊 `ArgW`，使其可透過 Autofac 自動取得。測試共有六組，傳入參數分別為:

1. 依順序傳入 `1`, `2`, `ArgY`
2. 提供 `ArgY`, `ArgZ` 兩種型別的參數值
3. 不提供任何參數
4. 指定 `x = ArgX`
5. 指定 `x = ArgX, z = ArgZ`
6. 指定 `i = 1, y = ArgY`

```csharp
static void Main(string[] args)
{
    ContainerBuilder builder = new ContainerBuilder();
    builder.RegisterType<MultiConstructor>();
    builder.RegisterType<ArgW>();
    IContainer container = builder.Build();

    var obj1 = container.Resolve<MultiConstructor>(
        new PositionalParameter(0, 1),
        new PositionalParameter(1, 2),
        new PositionalParameter(2, new ArgY())
        );
    var obj2 = container.Resolve<MultiConstructor>(
        new TypedParameter(typeof(ArgY), new ArgY()),
        new TypedParameter(typeof(ArgZ), new ArgZ())
        );
    var obj3 = container.Resolve<MultiConstructor>();
    var obj4 = container.Resolve<MultiConstructor>(
        new NamedParameter("x", new ArgX())
        );

    try
    {
        var obj5 = container.Resolve<MultiConstructor>(
            new NamedParameter("x", new ArgX()),
            new NamedParameter("z", new ArgZ()));
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error:" + ex.Message);
    }
    var obj6 = container.Resolve<MultiConstructor>(
        new NamedParameter("i", 1),
        new NamedParameter("y", new ArgY()));
    Console.ReadLine();
}
```

猜看看結果為何? 有能力直接回答的同學請到台前領糖果後由教室前門離開去操場玩，不需要繼續往下看浪費時間。其他同學可參考我的題解:

由於 `RegisterType<ArgW>`，不管 `Resolve<MultiConstructor>` 傳入參數為何，`ArgW` 都可列入參數清單，各測試的比對結果如下:

1. 依順序傳入 `1`, `2`, `ArgY`
    符合建構式: `(ArgW w)`、`(int i, int j, ArgY y)`
    前者只有 1 個參數，後者有 3 個參數，3 > 1，後者勝出 => `Constructor int, int, ArgY`
2. 提供 `ArgY`, `ArgZ` 兩種型別的參數值
    符合的建構式: `(ArgW w)`、`(ArgY y, ArgZ z)`
    2 > 1，後者勝出 => `Constructor ArgY, ArgZ`
3. 不提供任何參數
    唯一符合者: `(ArgW w)`
    同額參選，直接當選 => `Constructor ArgW`
4. 指定 `x = ArgX`
    符合建構式: `(ArgW w)`、`(ArgX x)`、`(ArgW w, ArgX x)`
    兩個參數勝出 => `Constructor ArgW, ArgX`
5. 指定 `x = ArgX, z = ArgZ`
    符合者: `(ArgW w)`、`(ArgX x)`、`(ArgW w, ArgX x)`、`(ArgX x, ArgZ z)`
    2 > 1，但參數為 2 的有兩個，無法決定，丟出例外
    Error:Cannot choose between multiple constructors with equal length 2 on type 'M
    ultiConstructor'. Select the constructor explicitly, with the UsingConstructor()
    configuration method, when the component is registered.
6. 指定 `i = 1, y = ArgY`
    少了 `j`，`(int i, int j , ArgY y)` 不算符合，故只剩 `(ArgW w)` 同額參選 => `Constructor ArgW`

由以上測試，再歸納一次 Autofac 挑選建構式的原理: 建構式所有參數都必須在參數清單中才能入選 (參數可能來自 `Resolve<T>` 時傳入、`RegisterType<T>().WithPrameters()` 指定，或是參數型別已 `RegisterType<T>` 自動取得)，若有多個建構式入選，以參數最多者優先，若參數最多的建構式有多個就丟出例外。

如果不喜歡 " 參數最多者優先 " 的邏輯，Autofac 也允許自訂邏輯，方法是寫個類別實作 `IConstructorSelector` 決定由多個符合建構式挑選的邏輯，並在註冊時透過 `UsingConstructor()` 指定之。如果連 " 判定建構式是否符合目前的參數清單 " 也想自訂，則可透用 `FindConstructorsWith` 實現。不過基於 KISS(Keep It Simple, Stupid) 法則，實務上應該很少人會把架構搞到這麼複雜吧~****
