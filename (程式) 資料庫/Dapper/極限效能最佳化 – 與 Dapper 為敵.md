---
aliases:
date: 2019-10-30
update: 2019-11-05
author: 黃忠成
language: C#
sourceurl: https://dotblogs.com.tw/code6421/2019/11/05/dapperenemy
tags:
  - CSharp
  - 演算法
  - Dapper
---

# 極限效能最佳化 – 與 Dapper 為敵

在擔任技術顧問時最常遇到兩件事，一件是系統架構調整，另一件則是最大化產品價值，前者是用來維護、奠定日後的發展，後者則是在面對競爭者。一般來說，多數的產品在開發初期就會有假想敵，如果你遇到的是沒有假想敵的產品，那麼恭喜你，你拿到了商業模式中最有利的籌碼，就是**稀缺性**，但不幸的是，這樣的機率很低，==在這個年代，多數的商業模式都已經被找出來，差別在於執行細節及價值的定義。==本篇文章以 Dapper 做為假想敵，將追趕它的最有價值部分做一個紀錄，結果其實不是那麼重要，過程才是有趣的部分。

## Dapper 的價值

Dapper 是 .NET/.NET Core 相當有歷史的套件，用於把 ADO.NET 取回的資料填充到物件裡，用一個專業術語來說，就是 Row Mapping，或是稍微放大成 Micro-ORM，Dapper 之所以可以長久不衰，在於其兩個價值，一是==過人的效能==，二是==維持輕量==，因此如果要以他為假想敵，那麼就要在這兩點上下手，例如擁有這兩個價值之外再加幾個附加價值來做出區隔，或是擷取其中一個最有價值的部分，把第二個換成另一種特色，另一種是把其中次要價值的部分再擴大，以完整性來取勝，不管哪種選擇，這兩個價值一定要有其一，而且不能做得比他差太多，否則就是找錯假想敵了。

## 效能之戰

先提一下，這種挑最強的硬碰硬，是很笨的行為，但有時沒得選擇，因為只要這點做的差太多，連擺上台的機會都沒有，==合理的商業模式都是避開最強的部份，用別的來填充，但是重點是不能差太多==，Dapper 是 Open Source，也就是白箱，但多數情況面對的是黑箱，因此這裡把 Dapper 當作黑箱，將整個追趕效能的部分記錄下來，如開頭所提，結果其實不是那麼重要，過程的思路是寫這篇文章最大的目的。

拉回主題，我認為 Dapper 最有價值的部分在於效能，這也是無法忽視的特質，==Row Mapping 這種技術，說穿了就是利用 DataReader 把資料讀出來，然後塞到指定的物件裡==，這裡的第一版選擇最快的解決方案，先做出雛形，再來調教。

```csharp
public static IEnumerable<T> FromSQLRaw<T>(this IDbConnection conn, string sql) where T : class, new()
{
    List<T> result = new List<T>();
    conn.Open();
    try
    {
        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;
        using var dr = cmd.ExecuteReader(CommandBehavior.SequentialAccess | CommandBehavior.CloseConnection);
        while (dr.Read())
        {
            var instance = new T();
            var values = new object[dr.FieldCount];

            dr.GetValues(values);
            foreach (var prop in typeof(T).GetProperties())
            {
                try
                {
                    var fldIndex = dr.GetOrdinal(prop.Name);
                    prop.SetValue(instance, dr.IsDBNull(fldIndex) ? null : values[fldIndex]);
                }
                catch (Exception)
                {
                    continue;
                }
            }
            result.Add(instance);
        }
    }
    finally
    {
        conn.Close();
    }
    return result;
}
```

這裡將處理函式設計為 Extension Method，這也是 Dapper 主要的處理方式，這不需要看到原始碼就可以猜出，不意外，效能真的是天差地遠。

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341038_19856.png)

不過雛型已經出來，我們可以標記可能的熱區，通常被執行最多次，或是迴圈中的東西都可以標記為熱區。

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341053_86179.png)

`GetProperties()` 需要時間 (1)，而且是在迴圈裏面，所以這裡很明顯是可以最佳化的點之一，第二個是 `try ... catch` 部分 (2)，由於處理 `Exception` 需要許多時間，尤其是真的有吐出來時，最佳化這部分其實花不了多少時間，所以放在優先處理區段，第三個可以想見是硬傷 (3)，因為==反射的效能非常差==，但是要處理這個需要花費許多時間，所以排在第二個，第二版鎖定處理 `Exception` 及 `GetProperties` 部分。

```csharp
private static int GetFieldIndex(string name, IDataReader reader)
{
    try
    {
        return reader.GetOrdinal(name);
    }
    catch (Exception)
    {
        return -1;
    }
}

public static IEnumerable<T> FromSQLRaw1<T>(this IDbConnection conn, string sql) where T : class, new()
{

    List<T> result = new List<T>();
    conn.Open();
    try
    {
        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;
        using var dr = cmd.ExecuteReader(CommandBehavior.SequentialAccess | CommandBehavior.CloseConnection);
        Dictionary<PropertyInfo, int> fieldMap = null;
        while (dr.Read())
        {
            var instance = new T();
            var values = new object[dr.FieldCount];
            if (fieldMap == null)
                fieldMap = typeof(T).GetProperties().Select(a => new
                { PropInfo = a, Index = GetFieldIndex(a.Name, dr) }).Where(a => a.Index != -1).OrderBy(a => a.Index).ToDictionary(a => a.PropInfo, v => v.Index);

            dr.GetValues(values);
            foreach (var prop in fieldMap)
            {
                prop.Key.SetValue(instance, values[prop.Value] == DBNull.Value ? null : values[prop.Value]);
            }
            result.Add(instance);
        }
    }
    finally
    {
        conn.Close();
    }
    return result;
}
```

這裡我們用一個 `fieldMap` 搭配 `GetFieldIndex` 函式，將取得屬性、欄位的對應動作縮減至一次，也就是常見的快取手段，==把可以快取的東西移入快取是效能最佳化的主要守則之一，但因為快取的出現，要特別小心過期或是額外付出的成本。==

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341188_78672.png)

如圖，有很明顯的改進，但還是有近一倍的差距，看來不得不面對 PropertyInfo 的 SetValue 效能問題了，要處理這個，得退到 IL 層級，也就是不透過反射直接列舉 IL 來填充資料。

```csharp
private static Action<TType, object> CreateTo<TType>(string property)
{
    var type = typeof(TType);
    var pi = type.GetProperty(property);
    if (pi == null)
        throw new ArgumentException($"property {property} not exists");

    var method = new DynamicMethod($"{type.Name}_{property}_Setter", typeof(void), new[] { type, typeof(object) }, typeof(RowMappingHandler));
    var ilgen = method.GetILGenerator();
    ilgen.Emit(OpCodes.Ldarg_0);
    ilgen.Emit(OpCodes.Ldarg_1);
    if (pi.PropertyType.IsValueType)
    {
        ilgen.Emit(OpCodes.Unbox_Any, pi.PropertyType);
        ilgen.Emit(OpCodes.Call, pi.GetSetMethod());
    }
    else
    {
        ilgen.Emit(OpCodes.Castclass, pi.PropertyType);
        ilgen.Emit(OpCodes.Callvirt, pi.GetSetMethod());
    }
    ilgen.Emit(OpCodes.Ret);
    return method.CreateDelegate(typeof(Action<TType, object>)) as Action<TType, object>;
}

public static IEnumerable<T> FromSQLRaw2<T>(this IDbConnection conn, string sql) where T : class, new()
{
    List<T> result = new List<T>();
    conn.Open();
    try
    {
        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;
        using var dr = cmd.ExecuteReader(CommandBehavior.SequentialAccess | CommandBehavior.CloseConnection);
        Dictionary<string, Delegate> mappedType = new Dictionary<string, Delegate>();
        Dictionary<string, int> fieldMap = null;
        while (dr.Read())
        {
            var instance = new T();
            if (fieldMap == null)
                fieldMap = typeof(T).GetProperties().Select(a => new
                { a.Name, Index = GetFieldIndex(a.Name, dr) }).Where(a => a.Index != -1).OrderBy(a => a.Index).ToDictionary(a => a.Name, v => v.Index);
            var values = new object[dr.FieldCount];
            dr.GetValues(values);

            foreach (var pi in fieldMap)
            {
                if (!mappedType.ContainsKey(pi.Key))
                    mappedType[pi.Key] = CreateTo<T>(pi.Key);
                ((Action<T, object>)mappedType[pi.Key])(instance, values[fieldMap[pi.Key]] == DBNull.Value ? null : values[fieldMap[pi.Key]]);

            }
            result.Add(instance);
        }
    }
    finally
    {
        conn.Close();
    }
    return result;
}
```

結果。

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341367_17509.png)

看來突破倍數的限界了，一般來說到這個階段已經可以打住了，因為再下去是 Micro-Optimize，到達 10 位數以內的 ms 世界，其實 CP 值已經沒那麼大了，而且還會增加維護的困難，不過這裡還是硬幹，以最後一個例子來說，`Delegate` 部分其實可以進行快取，這可以加快第二次處理的效能。

```csharp
private static Dictionary<Type, Dictionary<string, Delegate>> _mappingCache = new Dictionary<Type, Dictionary<string, Delegate>>();

public static IEnumerable<T> FromSQLRaw3<T>(this IDbConnection conn, string sql) where T : class, new()
{
    List<T> result = new List<T>();
    conn.Open();

    try
    {
        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;
        using var dr = cmd.ExecuteReader(CommandBehavior.SequentialAccess | CommandBehavior.CloseConnection);
        Dictionary<string, Delegate> mappedType;
        if (!_mappingCache.TryGetValue(typeof(T), out mappedType))
        {
            _mappingCache.Add(typeof(T), new Dictionary<string, Delegate>());
            mappedType = _mappingCache[typeof(T)];
        }
        Dictionary<string, int> fieldMap = null;
        while (dr.Read())
        {
            var instance = new T();
            if (fieldMap == null)
                fieldMap = typeof(T).GetProperties().Select(a => new
                { a.Name, Index = GetFieldIndex(a.Name, dr) }).Where(a => a.Index != -1).OrderBy(a => a.Index).ToDictionary(a => a.Name, v => v.Index);

            var values = new object[dr.FieldCount];
            dr.GetValues(values);
            foreach (var pi in fieldMap)
            {
                if (!mappedType.ContainsKey(pi.Key))
                    mappedType[pi.Key] = CreateTo<T>(pi.Key);
                ((Action<T, object>)mappedType[pi.Key])(instance, values[fieldMap[pi.Key]] == DBNull.Value ? null : values[fieldMap[pi.Key]]);
            }
            result.Add(instance);
        }
    }
    finally
    {
        conn.Close();
    }
    return result;
}
```

結果。

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341518_12059.png)

恩，好一些了，但這只作用於第二次，重點在於要加快首次處理，我們知道，在使用 `DataReader` 時，最快的方式是呼叫對應型別的取值函式，而不是使用 `GetValues` 或是 `GetValue` 這種函式，這兩個函式會引發後續的 box 及 unbox 的效應，要做到這點，得為每個 `Delegate` 產生不同的形態，這大幅的增加列舉 IL 動作的難度。

```csharp
private static Delegate CreateTo2<TType>(string property)
{
    var type = typeof(TType);
    var pi = type.GetProperty(property);
    if (pi == null)
        throw new ArgumentException($"property {property} not exists");

    var method = new DynamicMethod($"{type.FullName}_{property}_Setter", typeof(void), new[] { type, pi.PropertyType }, typeof(RowMappingHandler));
    var ilgen = method.GetILGenerator();
    ilgen.Emit(OpCodes.Ldarg_0);
    ilgen.Emit(OpCodes.Ldarg_1);
    if (pi.PropertyType.IsValueType)
        ilgen.Emit(OpCodes.Call, pi.GetSetMethod());
    else
        ilgen.Emit(OpCodes.Callvirt, pi.GetSetMethod());
    ilgen.Emit(OpCodes.Ret);
    return method.CreateDelegate(typeof(Action<,>).MakeGenericType(new[] { type, pi.PropertyType }));
}

public static IEnumerable<T> FromSQLRaw4<T>(this IDbConnection conn, string sql) where T : class, new()
{
    List<T> result = new List<T>();
    conn.Open();
    try
    {
        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;
        using var dr = cmd.ExecuteReader(CommandBehavior.CloseConnection | CommandBehavior.SequentialAccess);
        Dictionary<string, Delegate> mappedType;

        if (!_mappingCache.TryGetValue(typeof(T), out mappedType))
        {
            _mappingCache.Add(typeof(T), new Dictionary<string, Delegate>());
            mappedType = _mappingCache[typeof(T)];
        }

        Dictionary<PropertyInfo, int> fieldMap = null;
        bool mapRoundDown = false;
        while (dr.Read())
        {
            var instance = new T();
            if (fieldMap == null)
                fieldMap = typeof(T).GetProperties().Select(a => new
                { PropInfo = a, Index = GetFieldIndex(a.Name, dr) }).Where(a => a.Index != -1).OrderBy(a => a.Index).ToDictionary(a => a.PropInfo, v => v.Index);

            foreach (var pi in fieldMap)
            {
                Delegate func;
                if (!mapRoundDown && !mappedType.ContainsKey(pi.Key.Name))
                {
                    func = CreateTo2<T>(pi.Key.Name);
                    mappedType[pi.Key.Name] = func;
                }
                else
                    func = mappedType[pi.Key.Name];

                if (pi.Key.PropertyType == typeof(int))
                    ((Action<T, int>)func)(instance, dr.IsDBNull(fieldMap[pi.Key]) ? -1 : dr.GetInt32(fieldMap[pi.Key]));
                else if (pi.Key.PropertyType == typeof(string))
                    ((Action<T, string>)func)(instance, dr.IsDBNull(fieldMap[pi.Key]) ? (string)null : dr.GetString(fieldMap[pi.Key]));
            }
            mapRoundDown = true;
            result.Add(instance);
        }
    }
    finally
    {
        conn.Close();
    }
    return result;
}
```

結果。

![](https://dotblogsfile.blob.core.windows.net/user/code6421/09e67a67-25e3-4e83-b350-b3eb56ef1f1b/1572341728_53104.png)

注意，這只是 POC，我並未處理所有的型別，不過這已經足夠證明這個手法可行，但是代價是程式碼看起來越來越難維護了，一般來說到達這個階段就會暫時打住，移往另一個價值發展，畢竟在這硬碰硬太浪費時間了，以這個例子來說，LINQ 那個區段的結果可以快取，還有平展來取回一些效能，但代價是更難維護，所以先就此打住，事實上，Dapper 幾乎列舉了處理 `DataReader` 的整個函式的 IL，連 `Goto` 都用上了，再加上無所不在的快取。

註 1: 這不是完整的程式，我略過了很多錯誤處理

註 2: 還是沒有追上效能，但是處於一個可以接受的區段

註 3: (承上) 因為我想到要列舉更多的 IL 頭就很痛

註 4: 程式 github 連結在 [FB](https://www.facebook.com/cooldotnet/)
