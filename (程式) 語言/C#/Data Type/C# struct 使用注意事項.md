---
aliases:
date: 2021-05-22
update:
author: Opass
language:
sourceurl: https://www.opasschang.com/docs/csharp-struct-usage-notes
tags:
  - CSharp
  - CSharp_Struct
---

# C# struct 使用注意事項

以前讀過 struct 相關的使用事項，但日常生活還是太常實做 class 而非 struct 了，最近要用的時候寫一寫突然發現自己有一些觀念沒辦法好好解釋給 team member，所以決定整理一下。

## struct 和 class 的差異

### class 是 **reference-type**, struct 是 **value-type**

![[CSharpDataType.webp]]

Reference Type 被分配在 heap，
Value-Type 物件被分配的記憶體位置在 stack 上。

設計在 stack 上就是為了「不用倒垃圾啊！」，heap 上的東西要 gc ，但是 stack 上的物件 stack frame 退下就消失了，因此有比較低的成本。

### struct 不能繼承 （畫重點

對，不行。

如果讓 struct 能夠繼承會帶來一大堆麻煩事。
首先遇到的是空間分配，如果指派子類 struct 到父類 struct 的變數，那我到底要在 stack 上預先分配多少空間？而且實做多型還要準備一張 vTable 放函式指標，根本失去了 struct 作為 value-type light-weight，想要降低成本的初衷。

### struct 可以實做介面

對，可以。介面只是一群喜歡一起逛街玩耍滾來滾去的方法簽章而已。讓 struct 實做介面只是強迫這個 struct 身上一定要有指定的方法簽章。

## 何時該選擇用 struct?

看 [官方建議](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct)

✔️ CONSIDER defining a struct instead of a class if instances of the type are small and commonly short-lived or are commonly embedded in other objects. （如果你又小又短，就可以用 struct）

❌ AVOID defining a struct unless the type has all of the following characteristics:

It logically represents a single value, similar to primitive types (int, double, etc.). （通常邏輯上來說代表一個值）
It has an instance size under 16 bytes. （太大的話 copy 會有成本）
It is immutable. （盡量設計成不可變，如果可以變動又遇上 boxing，你會瘋掉）
It will not have to be boxed frequently.（裝箱有成本，而且會失去減少 gc 的好處）
In all other cases, you should define your types as classes. 畫重點，這行會考（X

絕大多數的情況，應該都是寫 class。

## Boxing 和 Unboxing

### boxing

Boxing 發生在你想把一個 value-type 的物件，當成 reference type 來操作的時候。等於是要在 heap 上找一塊空間，然後把 struct 身上所有的值都 copy 過去。一旦做了這件事，等於幾乎失去了 struct 所有的好處。

何時會發生 boxing ?

```csharp
public static void Main()
{
    int v = 5;
    Object o = v;
}
```

這樣就會了，因為 Object 是 reference type，想把 int 當成 object 一樣操作，就會發生 boxing。

而 Boxing 遠比你想像中的更容易發生，在把 Value-Type 傳入 Object 為參數的方法時也會發生。

```csharp
public static void Main()
{
    int v = 5;
    Object o = v; // 指派 v 給 o ，裝箱一次
    v = 123;
    Console.WriteLine(v + "," + (int)o);
    // 強制把 o 變回 int，會 unboxing 一次，接著
    // v + "," + (int)o 背後等同呼叫 String.Concat(Object, Object, Object)，
    // 所以第一個參數 v 會被 boxing，而且第三個參數 int(o) 也會再被 boxing 回 Object。
}
```

從原本 struct 轉成 Interface 的類型時，也會發生 boxing。

```csharp
struct Point : IComparable
{
    private int _x;
    public Point(int x)
    {
        _x = x;
    }
    public int CompareTo(object obj)
    {
        throw new NotImplementedException();
    }
}

class Program
{
    static void Main(string[] args)
    {
        Point p = new Point(3);
        IComparable c = p; // boxing, interface will need a heap reference
    }
}
``` 

### unboxing

把原本是 boxing 的物件，copy 回到 stack 上的行為就是 unboxing。

```csharp
public static void Main()
{
    int v = 5;
    Object o = v;
    v = (int)o; // 這就是 unboxing
}
```

### 對 struct 進行方法呼叫的效能

#### 直接呼叫覆寫後的方法不會發生 boxing

型別系統設計上，struct type 也是一種 Object，因此可以呼叫 Equals, GetHashCode, ToString 虛擬方法。
此時，如果你有在 struct 上乖乖 override 掉這些方法，並提供自己的實做，那麼你就是個乖寶寶 (O)，因為 struct 沒有辦法再進行繼承，所以編譯器能夠在 compiler time 幫你生成出直接呼叫的程式碼，不會發生 boxing。

#### 直接呼叫沒覆寫的 Object 身上的方法，會發生能避免的 boxing

但是如果你沒有 override 掉，或是你 override 掉了，卻還是在 override 掉的方法實做中呼叫了 base 方法，那麼就會發生 boxing，呼叫原本方法時，才有辦法把傳入 this。（this 會綁定到 boxing 後的記憶體位置上）

#### 呼叫 Object 身上的 non-virtual 方法時，一定會發生 boxing

呼叫 GetType 或 MemberwiseClone 等 Object 身上的 non-virtual instance method，就一定會發生 boxing ，this 總是要有人處理啊

### 當 struct 需要相等性比較時，總是覆寫 Equals 方法與 GetHashCode

是的，極度建議你覆寫，因為你會遇到兩個效能問題

- 呼叫預設的 Equals 實做會導致 Boxing，降低效能
- 預設實做使用反射，降低效能

為什麼呢？這要從 CLR 整個 Object 與 ValueType 的設計講起。

C# 中所有物件都是 Object，而 Object 有一個虛擬方法叫做 Equals，這是因為 CLR 的設計者認為，如果讓整個 C# 的所有物件都可以比較，會很方便。所以他就這麼訂惹。

但是這裡的 Equals，其實意義上不夠明確，當我們講相等時，請和溝通者釐清，你講的是 相等性 (Equal)，還是同一性 (Identical)

**在萬物之母 Object 身上，Equals 的預設實做是 Identical，只有兩個記憶體位置完全相同的物件，才叫做 Equal。**

但是因為設計者把 Equals 設計成 virtual，所以任何子 class 子 struct，都可以覆寫掉 Equals 的行為，因此 Equals 沒辦法總是表達 Identical，所以設計者在萬物之母 Object 身上提供了靜態方法比較同一性 (identical)

```csharp
public static bool ReferenceEquals (object? objA, object? objB); // always 比較 identical
```

對於那些 ValueType 的物件想表達的，Identical 並不重要，Value 是否一樣才是重點。這就是 ValueSemantic，在語意上表達內容的相等。因此在 ValueType 身上，覆寫了 Equals，其實作為透過反射存取自身所有的屬性，並比較記憶體內容是否一樣。

![[value-reference-types-common-type-system.png]]

圖片引用自 msdn https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/

### 你最好自己覆寫 Equals

但因為反射有效能上的問題，所以當你需要比較 ValueType 的相等性時，最好還是自己覆寫 Equals，實做比較你自定義的所有屬性。

你最好順便實做 `IEquatable<T>`

而這一切就像是命運的鎖鏈一樣，當你發現你要覆寫 Equals 時，你通常會想要順便實做泛型版本的 `IEquatable<T>` ，拿到型別安全的好處，並且少掉裝箱的成本。

### 你最好順便覆寫 GetHashCode

而當你需要實做相等性時，你極度有可能是需要在 Dictionary, Set 等需要 Key 的地方使用 GetHashCode，而預設 ValueType 實做的 GetHashCode 會透過反射產生 HashCode。要知道 HashCode 的本質就是 Hash 速度和分散度的取捨，你通常會比預設實作懂更多，你極有可能需要自行覆寫 GetHashCode，而這通常很簡單。

```csharp
(PropA, PropB, PropC, PropD).GetHashCode() // C# 7 可以直接這麼做，讓 ValueTuple 幫你生成
```

至於 == 和 != 就隨意看心情，預設 struct 並沒有提供 == 與 != 的實做，想要這功能要自己做 operator Overload

```csharp
public static bool operator ==(TwoDPoint lhs, TwoDPoint rhs) => lhs.Equals(rhs); // 看心情
public static bool operator !=(TwoDPoint lhs, TwoDPoint rhs) => !(lhs == rhs);
```

除了效能外，另一個你該覆寫的理由是，**記憶體內容表示法不相等，並不代表兩個 ValueType 物件不相等**，這是一個很 tricky 的 case，在某些 .Net Runtime 下會出現不同的結果。([Code Reference]([http://Performance](http://performance/) implications of default struct equality in C#))

```csharp
public struct MyDouble
{
    public double Value { get; }
    public MyDouble(double value) => Value = value;
}
struct Money
{
    public double Amount;
}
class Program
{
    static void Main(string[] args)
    {
        double d1 = -0.0; // 記憶體表示法不同
        double d2 = +0.0;
        bool b2 = new MyDouble(d1).Equals(new MyDouble(d2));
        Console.WriteLine(b2);
        // After testing, false in .Net Framework 4.7.2,
        // But True in .Net Core 3
        Console.ReadLine();
    }
}
```

## 總結

絕大多數的時間都是用 class，只有少數需要效能，而且你很確定不會產生 boxing, unboxing 的成本時才用 struct。
另外如果 struct 需要相等性比較，則一定要覆寫 `Equals` 和實做 `IEquatable<T>`。

## 參考

- 書籍 CLR via C#
- msdn [How to define value equality for a class or struct (C# Programming Guide)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)
- msdn [The C# type system](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- msdn [Structure types (C# reference)](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
- stackoverflow [When should I use a struct rather than a class in C#?](https://stackoverflow.com/questions/521298/when-should-i-use-a-struct-rather-than-a-class-in-c)
