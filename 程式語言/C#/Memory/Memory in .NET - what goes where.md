---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Memory in .NET - what goes where .NET 中的記憶體 - 什麼存放在哪裡

A lot of confusion has been wrought by people explaining the difference between value types and reference types as "value types go on the stack, reference types go on the heap". This is simply untrue (as stated) and this article attempts to clarify matters somewhat.
很多人解釋值類型和參考類型之間的區別時，說「值類型在堆疊上，參考類型在堆上」，這純粹是不正確的（如上所述），本文試圖在一定程度上澄清這些問題。

## What's in a variable? 變數中有什麼？

The key to understanding the way memory works in .NET is to understand **what a variable is**, and **what its value is**. At the most basic level, a variable is just an association between a name (used in the program's source code) and a slot of memory. A variable has a value, which is the contents of the memory slot it's associated with. The size of that slot, and the interpretation of the value, depends on the type of the variable - and this is where the difference between value types and reference types comes in.
理解 .NET 中記憶體運作方式的關鍵在於理解**變數是什麼**，以及**它的值是什麼**。在最基本的層級，變數只是一個名稱（用於程式碼中）和記憶體位置之間的關聯。變數有一個值，即它所關聯的記憶體位置的內容。該位置的寬度和值的解釋取決於變數的類型——這就是值類型和參考類型之間的區別所在。

The value of a reference type variable is always either a reference or `null`. If it's a reference, it must be a reference to an object which is compatible with the type of the variable. For instance, a variable declared as `Stream s` will always have a value which is either `null` or a reference to an instance of the `Stream` class. (Note that an instance of a subclass of `Stream`, eg `FileStream`, is also an instance of `Stream`.) The slot of memory associated with the variable is just the size of a reference, however big the actual object it refers to might be. (On the 32-bit version of .NET, for instance, a reference type variable's slot is always just 4 bytes.)
一個參考類型變數的值總是或者是參考或者 `null` 。如果它是參考，那麼它必須是參考到一個與變數類型兼容的物件。例如，一個宣告為 `Stream s` 的變數總是會有一個值是 `null` 或者是 `Stream` 類別的實例的參考。(注意， `Stream` 的子類別的實例，例如 `FileStream` ，也是 `Stream` 的實例。) 與變數相關聯的記憶體槽的大小只是參考的大小，無論它實際參考的物件有多大。(例如，在 .NET 的 32 位版本中，一個參考類型變數的槽總是只有 4 個位元組。)

The value of a value type is always the data for an instance of the type itself. For instance, suppose we have a struct declared as:
變數的值總是該類型實例的數據本身。例如，假設我們有一個宣告為：

```csharp
struct PairOfInts
{
    public int a;
    public int b;
}
```

The value of a variable declared as `PairOfInts pair` is the pair of integers itself, not a reference to a pair of integers. The slot of memory is large enough to contain both integers (so it must be 8 bytes). Note that a value type variable can never have a value of `null` - it wouldn't make any sense, as `null` is a reference type concept, meaning "the value of this reference type variable isn't a reference to any object at all".
宣告為 `PairOfInts pair` 的變數的值是整數對本身，而不是指向一對整數的參考。記憶體的槽位足夠大，可以包含兩個整數（所以必須是 8 個位元組）。請注意，值類型變數永遠不會有 `null` 的值 - 這沒有任何意義，因為 `null` 是一個參考類型的概念，意思是「這個參考類型變數的值並不是指向任何物件」。

## So where are things stored? 那麼東西存放在哪裡呢？

The memory slot for a variable is stored on either the stack or the heap. It depends on the context in which it is declared:
變數的記憶體槽位存放在堆疊或堆上。這取決於它被宣告的上下文：

- Each local variable (ie one declared in a method) is stored on the stack. That includes reference type variables - the variable itself is on the stack, but remember that the value of a reference type variable is only a reference (or `null`), not the object itself. Method parameters count as local variables too, but if they are declared with the `ref` modifier, they don't get their own slot, but share a slot with the variable used in the calling code. See [my article on parameter passing](https://jonskeet.uk/csharp/parameters.html) for more details.
  每個局部變數（即宣告在方法中的變數）都儲存在堆疊上。這包括參考類型變數 - 變數本身在堆疊上，但請記住參考類型變數的值僅僅是一個參考（或 `null` ），而不是物件本身。方法參數也計算為局部變數，但如果它們使用 `ref` 修飾詞宣告，它們不會獲得自己的槽位，而是與呼叫代碼中使用的變數共享一個槽位。請參閱我關於參數傳遞的文章以獲取更多細節。
- Instance variables for a reference type are always on the heap. That's where the object itself "lives".
  參考類型的實例變數總是在堆上。那就是物件本身「居住」的地方。
- Instance variables for a value type are stored in the same context as the variable that declares the value type. The memory slot for the instance effectively contains the slots for each field within the instance. That means (given the previous two points) that a struct variable declared within a method will always be on the stack, whereas a struct variable which is an instance field of a class will be on the heap.
  值類型的實例變數儲存在宣告值類型的變數的相同上下文中。實例的記憶槽位實際上包含實例中每個欄位的槽位。這意味著（根據前兩點）在方法中宣告的 struct 變數總是在堆疊上，而作為類別實例欄位的 struct 變數則在堆上。
- Every static variable is stored on the heap, regardless of whether it's declared within a reference type or a value type. There is only one slot in total no matter how many instances are created. (There don't need to be any instances created for that one slot to exist though.) The details of exactly _which_ heap the variables live on are complicated, but explained in detail in an [MSDN article on the subject](http://msdn.microsoft.com/msdnmag/issues/05/05/JITCompiler/).
  每一個靜態變數都儲存在堆疊上，無論它是宣告在參考類型或值類型內。總共只有一個位置，無論建立多少個實例都是如此。（儘管不需要建立任何實例，這個位置也可以存在。）關於變數究竟存放在哪個堆疊的細節很複雜，但已在 MSDN 上關於此主題的文章中詳細解釋。

There are a couple of exceptions to the above rules - [captured variables](https://jonskeet.uk/csharp/csharp2/delegates.html#captured.variables) (used in anonymous methods and lambda expressions) are local in terms of the C# code, but end up being compiled into instance variables in a type associated with the delegate created by the anonymous method. The same goes for local variables in an [iterator block](https://jonskeet.uk/csharp/csharp2/iterators.html).
以上規則有幾個例外——被捕捉的變數（用於匿名方法和 Lambda 表達式）在 C# 語言中是局部的，但最終會編譯成與匿名方法創建的委託相關聯的類型的實例變數。迭代塊中的局部變數也是如此。

## A worked example  一個實例說明

The above may all sound a bit complicated, but a full example should make things a bit clearer. Here's a short program which does nothing useful, but should demonstrate the points raised above.
以上可能聽起來有點複雜，但一個完整的例子應該能讓事情變得清晰一些。這裡有一個短小的程序，它沒有任何實用性，但應該能說明上述要點。

```csharp
using System;

struct PairOfInts
{
    static int counter = 0;

    public int a;
    public int b;

    internal PairOfInts(int x, int y)
    {
        a = x;
        b = y;
        counter++;
    }
}

class Test
{
    PairOfInts pair;
    string name;

    Test(PairOfInts p, string s, int x)
    {
        pair = p;
        name = s;
        pair.a += x;
    }

    static void Main()
    {
        PairOfInts z = new PairOfInts(1, 2);
        Test t1 = new Test(z, "first", 1);
        Test t2 = new Test(z, "second", 2);
        Test t3 = null;
        Test t4 = t1;
        // XXX
    }
}
```

Let's look at what's where in memory at the line marked with the comment "XXX". (Assume that nothing is being garbage collected.)
讓我們看看在標註「XXX」的這行程式碼中，記憶體裡有哪些東西以及它們的位置。（假設沒有任何東西正在進行垃圾收集。）

- There's a `PairOfInts` instance on the stack, corresponding with variable `z`. Within that instance, `a=1` and `b=2`. (The 8 byte slot needed for `z` itself might then be represented in memory as `01 00 00 00 02 00 00 00`.)
  堆疊上有一個 `PairOfInts` 的實例，對應於變數 `z` 。在那個實例內部，有 `a=1` 和 `b=2` 。（用於 `z` 本身的 8 字節槽位在記憶體中可能會被表示為 `01 00 00 00 02 00 00 00` 。）
- There's a `Test` reference on the stack, corresponding with variable `t1`. This reference refers to an instance on the heap, which occupies "something like" 20 bytes: 8 bytes of header information (which all heap objects have), 8 bytes for the `PairOfInts` instance, and 4 bytes for the string reference. (The "something like" is because the specification doesn't say how it has to be organised, or what size the header is, etc.) The value of the `pair` variable within that instance will have `a=2` and `b=2` (possibly represented in memory as `02 00 00 00 02 00 00 00`). The value of the `name` variable within that instance will be a reference to a string object (which is also on the heap) and which (probably through other objects, such as a char array) represents the sequence of characters in the word "first".
  堆疊上有一個 `Test` 的參考，對應於變數 `t1` 。這個參考指向堆上的一個實例，它佔據「大約」20 字節：8 字節的標頭資訊（所有堆物件都有），8 字節用於 `PairOfInts` 的實例，以及 4 字節用於字串參考。（「大約」是因為規格沒有說明它必須如何組織，或標頭的大小等。）那個實例內部 `pair` 變數的值將會有 `a=2` 和 `b=2` （可能在記憶體中表示為 `02 00 00 00 02 00 00 00` ）。那個實例內部 `name` 變數的值將會是一個指向字串物件的參考（這個物件也在堆上），並且（可能透過其他物件，例如字元陣列）代表著單詞「first」中的字元序列。
- There's a second `Test` reference on the stack, corresponding with variable `t2`. This reference refers to a second instance on the heap, which is very similar to the one described above, but with a reference to a string representing "second" instead of "first", and with a value of `pair` where `a=3` (as 2 has been added to the initial value 1). If `PairOfInts` were a reference type instead of a value type, there would only be one instance of it throughout the whole program, and just several references to the single instance, but as it is, there are several instances, each with different values inside.
  有一個堆疊上的第二個 `Test` 參考，對應變數 `t2` 。這個參考指向堆疊上的第二個實例，與上面描述的非常相似，但參考的是代表「第二」的字串，而不是「第一」，且值為 `pair` ，其中 `a=3` （因為已經將初始值 1 加上 2）。如果 `PairOfInts` 是參考類型而不是值類型，整個程式中只會有一個實例，且只有幾個參考指向這個單一實例，但目前的情況是，有多個實例，每個實例內部都有不同的值。
- There's a third `Test` reference on the stack, corresponding with variable `t3`. This reference is `null` - it doesn't refer to any instance of `Test`. (There's some ambiguity about whether this counts as a `Test` reference or not - it doesn't make any difference though, really - I generally think of `null` as being a reference which doesn't refer to any object, rather than being an absence of a reference in the first place. The Java Language Specification gives quite nice terminology, saying that a reference is either `null` or a pointer to an object of the appropriate type.)
  有一個堆疊上的第三個 `Test` 參考，對應變數 `t3` 。這個參考是 `null` - 它不指向任何 `Test` 的實例。（關於這是否算作 `Test` 參考存在一些模糊之處 - 但實際上沒有差別 - 我通常認為 `null` 是一個不指向任何物件的參考，而不是說在最初就沒有參考。Java 語言規範給出了很棒的術語，說明參考要么是 `null` ，要么是指向適當類型的物件指標。）
- There's a fourth `Test` reference on the stack, corresponding with variable `t4`. This reference refers to the same instance as `t1` - ie the values of `t1` and `t4` are the same. Changing the value of one of these variables would not change the value of the other, but changing a value within the object they both refer to using one reference would make that change visible via the other reference. (For instance, if you set `t1.name="third";` then examined `t4.name`, you'd find it referred to "third" as well.)
  堆疊上有一個第四個 `Test` 參考，對應變數 `t4` 。這個參考指向與 `t1` 相同的實例 - 也就是 `t1` 和 `t4` 的值相同。改變其中一個變數的值不會改變另一個變數的值，但使用其中一個參考在它們都指向的物件內改變值，會讓這個變化透過另一個參考顯現出來。(例如，如果你設定 `t1.name="third";` 然後檢查 `t4.name` ，你會發現它也指向 "third" 。)
- Finally, there's the `PairOfInts.counter` variable, which is on the heap (as it's static). There's only a single "slot" for the variable, however many (or few) `PairOfInts` values there are.
  最後，有個 `PairOfInts.counter` 變數，它在堆疊上（因為它是靜態的）。無論有多少（或多少少） `PairOfInts` 值，這個變數只有一個 " 槽 "。
