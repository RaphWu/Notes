---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# CsvHelper 27.2.1 完整實戰教學（含進階用法）

下面給你一份 **CsvHelper 27.2.1 完整實戰教學（含進階用法）**，我會從「安裝 → 基本讀寫 → Mapping → 設定 → 進階技巧」一步一步帶你掌握。

## 📦 一、安裝 CsvHelper 27.2.1

👉 套件：CsvHelper

```bash
dotnet add package CsvHelper --version 27.2.1
```

或

```powershell
Install-Package CsvHelper -Version 27.2.1
```

（支援 .NET Framework / .NET Core / .NET 5+） ([NuGet][1])

## 🧱 二、CSV 範例資料

```csv
Id,Name,Age,Birthday
1,Alice,25,2000-01-01
2,Bob,30,1995-05-10
```

## 🧑‍💻 三、基本讀取（最常用）

### 1️⃣ 建立 Model

```csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public DateTime Birthday { get; set; }
}
```

### 2️⃣ 讀取 CSV

```csharp
using System.Globalization;
using CsvHelper;
using System.IO;

using (var reader = new StreamReader("data.csv"))
using (var csv = new CsvReader(reader, CultureInfo.InvariantCulture))
{
    var records = csv.GetRecords<Person>();

    foreach (var r in records)
    {
        Console.WriteLine($"{r.Name} - {r.Age}");
    }
}
```

👉 `GetRecords<T>()` 會自動對應欄位名稱

## ✍️ 四、寫入 CSV

```csharp
using System.Globalization;
using CsvHelper;
using System.IO;
using System.Collections.Generic;

var list = new List<Person>
{
    new Person { Id = 1, Name = "Alice", Age = 25, Birthday = DateTime.Parse("2000-01-01") },
    new Person { Id = 2, Name = "Bob", Age = 30, Birthday = DateTime.Parse("1995-05-10") }
};

using (var writer = new StreamWriter("output.csv"))
using (var csv = new CsvWriter(writer, CultureInfo.InvariantCulture))
{
    csv.WriteRecords(list);
}
```

## 🧩 五、欄位對應（ClassMap ⭐重要）

當 CSV 欄位名稱不一致時

```csv
user_id,user_name,user_age
```

### 建立 Mapping

```csharp
using CsvHelper.Configuration;

public class PersonMap : ClassMap<Person>
{
    public PersonMap()
    {
        Map(m => m.Id).Name("user_id");
        Map(m => m.Name).Name("user_name");
        Map(m => m.Age).Name("user_age");
    }
}
```

### 使用 Mapping

```csharp
using (var reader = new StreamReader("data.csv"))
using (var csv = new CsvReader(reader, CultureInfo.InvariantCulture))
{
    csv.Context.RegisterClassMap<PersonMap>();

    var records = csv.GetRecords<Person>().ToList();
}
```

## ⚙️ 六、CsvConfiguration（超重要）

```csharp
using CsvHelper.Configuration;

var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    HasHeaderRecord = true,     // 是否有標頭
    Delimiter = ",",            // 分隔符
    IgnoreBlankLines = true,
    TrimOptions = TrimOptions.Trim,
};
```

使用：

```csharp
var csv = new CsvReader(reader, config);
```

## 🔄 七、沒有 Header 的 CSV

```csv
1,Alice,25
2,Bob,30
```

```csharp
var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    HasHeaderRecord = false
};

using var csv = new CsvReader(reader, config);

var records = csv.GetRecords<Person>();
```

👉 會依照「屬性順序」對應

## 🎯 八、用 Index 對應欄位

```csharp
using CsvHelper.Configuration.Attributes;

public class Person
{
    [Index(0)]
    public int Id { get; set; }

    [Index(1)]
    public string Name { get; set; }

    [Index(2)]
    public int Age { get; set; }
}
```

## 🧪 九、型別轉換（重要）

### 日期格式

```csharp
Map(m => m.Birthday)
    .TypeConverterOption.Format("yyyy-MM-dd");
```

### 自訂轉換

```csharp
Map(m => m.Name).Convert(row =>
{
    return row.Row.GetField("Name").ToUpper();
});
```

## 🚫 十、忽略欄位

```csharp
Map(m => m.Age).Ignore();
```

或

```csharp
[Ignore]
public int Age { get; set; }
```

## 📊 十一、讀成 DataTable

```csharp
using System.Data;
using CsvHelper;

using var reader = new StreamReader("data.csv");
using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

using var dr = new CsvDataReader(csv);

var dt = new DataTable();
dt.Load(dr);
```

👉 CsvHelper 內建 IDataReader 支援 ([Josh Close][2])

## 🔥 十二、手動逐列讀取（進階）

```csharp
while (csv.Read())
{
    var id = csv.GetField<int>("Id");
    var name = csv.GetField<string>("Name");

    Console.WriteLine(name);
}
```

## 🧠 十三、跳過前幾行（實務很常見）

```csharp
using var reader = new StreamReader("data.csv");

// 跳過前 2 行
reader.ReadLine();
reader.ReadLine();

using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

var records = csv.GetRecords<Person>();
```

## ⚠️ 十四、常見錯誤整理

### 1️⃣ 中文亂碼

```csharp
new StreamReader("data.csv", Encoding.UTF8);
```

### 2️⃣ 欄位對不起來

✔ 用 `ClassMap`
✔ 或 `[Index]`

### 3️⃣ CSV 有逗號

```csv
"Hello, world",123
```

👉 CsvHelper 會自動處理 ✔

### 4️⃣ 欄位數不一致

👉 需設定：

```csharp
BadDataFound = null
```

## ✅ 總結（重點）

| 功能    | 方法                        |
| ----- | ------------------------- |
| 基本讀取  | `GetRecords<T>()`         |
| 寫入    | `WriteRecords()`          |
| 欄位不一致 | `ClassMap` ⭐              |
| 無標頭   | `HasHeaderRecord = false` |
| 進階控制  | `CsvConfiguration`        |

# 「百萬筆效能」＋「亂格式 CSV 容錯」

我直接給你一套 **實戰級寫法（可上線用）**。

## 🚀 一、大檔案（百萬筆）最佳化策略

### 🔥 核心原則（很重要）

1. ❌ 不要 `.ToList()`（會爆記憶體）
2. ✅ 使用 **串流逐筆讀取**
3. ✅ 邊讀邊處理（寫 DB / 處理）
4. ✅ 關閉不必要驗證
5. ✅ 批次處理（Batch）

### ✅ 範例：百萬筆安全讀取（低記憶體）

```csharp
using System.Globalization;
using CsvHelper;
using CsvHelper.Configuration;

var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    BufferSize = 1024 * 64,         // 提高 Buffer
    BadDataFound = null,            // 忽略壞資料
    MissingFieldFound = null,       // 忽略欄位缺失
    HeaderValidated = null,         // 關閉 Header 驗證
};

using var reader = new StreamReader("big.csv");
using var csv = new CsvReader(reader, config);

int count = 0;

foreach (var record in csv.GetRecords<Person>())
{
    Process(record);   // 👈 立即處理，不累積

    count++;
    if (count % 10000 == 0)
    {
        Console.WriteLine($"已處理 {count} 筆");
    }
}
```

### ⚡ 更快：手動讀（效能再提升）

👉 比 `GetRecords<T>()` 更快（少反射）

```csharp
while (csv.Read())
{
    var person = new Person
    {
        Id = csv.GetField<int>(0),
        Name = csv.GetField<string>(1),
        Age = csv.GetField<int>(2)
    };

    Process(person);
}
```

### 📦 批次寫入 DB（強烈建議）

```csharp
var batch = new List<Person>(1000);

foreach (var record in csv.GetRecords<Person>())
{
    batch.Add(record);

    if (batch.Count >= 1000)
    {
        BulkInsert(batch);  // 👈 一次寫入
        batch.Clear();
    }
}

// 收尾
if (batch.Count > 0)
    BulkInsert(batch);
```

### ⚙️ 效能優化重點

| 技巧              | 效果           |
| --------------- | ------------ |
| `BufferSize` 調大 | 減少 IO        |
| 手動 `Read()`     | 少反射，快 20~40% |
| Batch 寫入        | DB 效能大提升     |
| 關閉驗證            | 減少 overhead  |

## 🧨 二、亂格式 CSV（重點中的重點）

現實 CSV 常長這樣👇

```csv
Id,Name,Age
1,"Alice, A.",25
2,Bob,
3,"Charlie ""The Boss""",35
4,David,abc
5
```

問題：

* 欄位缺失 ❌
* 多逗號 ❌
* 引號錯誤 ❌
* 型別錯誤 ❌

### ✅ 容錯設定（必備）

```csharp
var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    BadDataFound = context =>
    {
        Console.WriteLine($"壞資料: {context.RawRecord}");
    },

    MissingFieldFound = null,  // 忽略缺欄位

    HeaderValidated = null,    // 忽略 header 錯誤

    ReadingExceptionOccurred = ex =>
    {
        Console.WriteLine($"錯誤: {ex.Exception.Message}");
        return false; // 繼續讀
    }
};
```

### ✅ 安全讀取（防爆版本）

```csharp
while (csv.Read())
{
    try
    {
        var id = csv.GetField<int?>("Id");
        var name = csv.GetField<string>("Name");
        var age = csv.GetField<int?>("Age");

        var person = new Person
        {
            Id = id ?? 0,
            Name = name ?? "Unknown",
            Age = age ?? -1
        };

        Process(person);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"跳過錯誤資料: {ex.Message}");
    }
}
```

### 🧠 技巧：TryGetField（更安全）

```csharp
if (!csv.TryGetField<int>("Age", out var age))
{
    age = -1;
}
```

### 🔧 自訂清洗（超實用）

```csharp
Map(m => m.Age).Convert(row =>
{
    var raw = row.Row.GetField("Age");

    if (int.TryParse(raw, out int age))
        return age;

    return -1; // fallback
});
```

## 🧪 三、極端亂格式（分隔符亂掉）

例如：

```csv
1|Alice|25
2|Bob|30
```

👉 解法：

```csharp
Delimiter = "|"
```

## 💣 四、處理破碎 CSV（最難）

例如：

* 換行在欄位內
* quote 沒關

👉 必開：

```csharp
Mode = CsvMode.RFC4180
```

或：

```csharp
DetectDelimiter = true
```

## 🧱 五、終極穩定模板（推薦直接用）

```csharp
var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    BufferSize = 1024 * 64,
    BadDataFound = null,
    MissingFieldFound = null,
    HeaderValidated = null,
    IgnoreBlankLines = true,
    TrimOptions = TrimOptions.Trim,
};

using var reader = new StreamReader("data.csv", Encoding.UTF8);
using var csv = new CsvReader(reader, config);

while (csv.Read())
{
    try
    {
        var id = csv.GetField<int?>(0) ?? 0;
        var name = csv.GetField<string>(1) ?? "";
        var age = csv.GetField<int?>(2) ?? -1;

        Process(new Person { Id = id, Name = name, Age = age });
    }
    catch
    {
        // 忽略壞資料
    }
}
```

## ✅ 總結（實務重點）

### 🚀 大檔案

* 用 `Read()` 或 `GetRecords()`（不要 ToList）
* 批次處理
* 關閉驗證

### 🧨 亂格式

* `BadDataFound = null`
* `TryGetField`
* `Convert()` 清洗
* try-catch 保底

## 🎯 如果你要更進一步（我可以幫你）

我可以幫你直接做：

* 🔥 100 萬筆 → SQL Server 最快匯入（BulkCopy）
* 🔥 CSV → API 上傳解析（ASP.NET）
* 🔥 自動偵測欄位格式（AI 清洗）
* 🔥 Excel 匯出亂格式修復

👉 直接貼你的 CSV（哪怕很亂），我可以幫你寫「專用解析器」 👍
