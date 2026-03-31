# 目錄

- [目錄](#目錄)
- [官方](#官方)
- [CsvHelper 27.2.1 完整實戰教學（含進階用法）](#csvhelper-2721-完整實戰教學含進階用法)
    - [📦 一、安裝 CsvHelper 27.2.1](#-一安裝-csvhelper-2721)
    - [🧱 二、CSV 範例資料](#-二csv-範例資料)
    - [🧑‍💻 三、基本讀取（最常用）](#-三基本讀取最常用)
      - [1️⃣ 建立 Model](#1️⃣-建立-model)
      - [2️⃣ 讀取 CSV](#2️⃣-讀取-csv)
    - [✍️ 四、寫入 CSV](#️-四寫入-csv)
    - [🧩 五、欄位對應（ClassMap ⭐重要）](#-五欄位對應classmap-重要)
      - [建立 Mapping](#建立-mapping)
      - [使用 Mapping](#使用-mapping)
    - [⚙️ 六、CsvConfiguration（超重要）](#️-六csvconfiguration超重要)
    - [🔄 七、沒有 Header 的 CSV](#-七沒有-header-的-csv)
    - [🎯 八、用 Index 對應欄位](#-八用-index-對應欄位)
    - [🧪 九、型別轉換（重要）](#-九型別轉換重要)
      - [日期格式](#日期格式)
      - [自訂轉換](#自訂轉換)
    - [🚫 十、忽略欄位](#-十忽略欄位)
    - [📊 十一、讀成 DataTable](#-十一讀成-datatable)
    - [🔥 十二、手動逐列讀取（進階）](#-十二手動逐列讀取進階)
    - [🧠 十三、跳過前幾行（實務很常見）](#-十三跳過前幾行實務很常見)
    - [⚠️ 十四、常見錯誤整理](#️-十四常見錯誤整理)
      - [1️⃣ 中文亂碼](#1️⃣-中文亂碼)
      - [2️⃣ 欄位對不起來](#2️⃣-欄位對不起來)
      - [3️⃣ CSV 有逗號](#3️⃣-csv-有逗號)
      - [4️⃣ 欄位數不一致](#4️⃣-欄位數不一致)
    - [總結（重點）](#總結重點)
- [「百萬筆效能」＋「亂格式 CSV 容錯」](#百萬筆效能亂格式-csv-容錯)
    - [🚀 一、大檔案（百萬筆）最佳化策略](#-一大檔案百萬筆最佳化策略)
      - [🔥 核心原則（很重要）](#-核心原則很重要)
      - [範例：百萬筆安全讀取（低記憶體）](#範例百萬筆安全讀取低記憶體)
      - [⚡ 更快：手動讀（效能再提升）](#-更快手動讀效能再提升)
      - [📦 批次寫入 DB（強烈建議）](#-批次寫入-db強烈建議)
      - [⚙️ 效能優化重點](#️-效能優化重點)
    - [🧨 二、亂格式 CSV（重點中的重點）](#-二亂格式-csv重點中的重點)
      - [容錯設定（必備）](#容錯設定必備)
      - [安全讀取（防爆版本）](#安全讀取防爆版本)
      - [🧠 技巧：TryGetField（更安全）](#-技巧trygetfield更安全)
      - [🔧 自訂清洗（超實用）](#-自訂清洗超實用)
    - [🧪 三、極端亂格式（分隔符亂掉）](#-三極端亂格式分隔符亂掉)
    - [💣 四、處理破碎 CSV（最難）](#-四處理破碎-csv最難)
    - [🧱 五、終極穩定模板（推薦直接用）](#-五終極穩定模板推薦直接用)
    - [總結（實務重點）](#總結實務重點)
      - [🚀 大檔案](#-大檔案)
      - [🧨 亂格式](#-亂格式)
    - [🎯 如果你要更進一步（我可以幫你）](#-如果你要更進一步我可以幫你)
- [ClassMap用法](#classmap用法)
  - [方法一（最推薦）：用 CsvHelper + 忽略 Records](#方法一最推薦用-csvhelper--忽略-records)
    - [🧩 建立 ClassMap（重點🔥）](#-建立-classmap重點)
    - [🧩 寫入方式](#-寫入方式)
    - [✨ 優點](#-優點)
  - [方法二（超簡單但硬寫）](#方法二超簡單但硬寫)
  - [🚀 方法三（偷懶技巧🔥）](#-方法三偷懶技巧)
  - [🧠 建議選哪個？](#-建議選哪個)
  - [🧾 一句話結論](#-一句話結論)
  - [✨ 額外建議（很重要）](#-額外建議很重要)
- [格式化資料](#格式化資料)
  - [1️⃣ 日期 / 時間格式化（最常用）](#1️⃣-日期--時間格式化最常用)
  - [2️⃣ TimeSpan 格式化](#2️⃣-timespan-格式化)
  - [3️⃣ 數值小數位控制](#3️⃣-數值小數位控制)
  - [4️⃣ 自訂轉換（最強🔥）](#4️⃣-自訂轉換最強)
  - [5️⃣ 改欄位名稱（Header）](#5️⃣-改欄位名稱header)
  - [6️⃣ 排序欄位](#6️⃣-排序欄位)
  - [🚀 完整範例（含格式化 + Ignore）](#-完整範例含格式化--ignore)
  - [🧠 小提醒（很重要）](#-小提醒很重要)
      - [❗ Excel 問題](#-excel-問題)
  - [🧾 一句話結論](#-一句話結論-1)
- [工業設備記錄（MES / 產線資料）標準的做法](#工業設備記錄mes--產線資料標準的做法)
  - [🧠 核心概念（一定要有）](#-核心概念一定要有)
  - [建議資料結構](#建議資料結構)
    - [🔷 主檔（Fastening 主資料）](#-主檔fastening-主資料)
    - [🔷 明細（Records）](#-明細records)
  - [🚀 寫入流程（實務標準流程）](#-寫入流程實務標準流程)
    - [1️⃣ 產生唯一ID](#1️⃣-產生唯一id)
    - [2️⃣ 寫主檔（1筆）](#2️⃣-寫主檔1筆)
    - [3️⃣ 寫明細（多筆）](#3️⃣-寫明細多筆)
  - [📂 最終 CSV 長這樣](#-最終-csv-長這樣)
    - [🧾 header.csv](#-headercsv)
    - [📊 detail.csv](#-detailcsv)
  - [✨ 為什麼這樣設計（重點🔥）](#-為什麼這樣設計重點)
    - [1. 可追溯（Traceability）](#1-可追溯traceability)
    - [2. 不會資料爆炸](#2-不會資料爆炸)
    - [3. 未來可上資料庫（超重要）](#3-未來可上資料庫超重要)
    - [4. 支援多機台 / 多產品](#4-支援多機台--多產品)
  - [⚡ 進階優化（實務會做）](#-進階優化實務會做)
    - [🔸 ID 可讀化（不要純 GUID）](#-id-可讀化不要純-guid)
    - [🔸 加 Batch / 工單](#-加-batch--工單)
  - [🧾 一句話結論](#-一句話結論-2)
- [TimeSpan → 相對秒數](#timespan--相對秒數)
  - [✅ 最乾淨做法（用 CsvHelper Map）](#-最乾淨做法用-csvhelper-map)
- [CSV 每天自動分檔](#csv-每天自動分檔)
  - [檔名設計（標準）](#檔名設計標準)
  - [自動切檔（跨天）](#自動切檔跨天)
- [只存 FAIL（異常資料）](#只存-fail異常資料)
  - [定義判斷條件](#定義判斷條件)
  - [寫入前過濾](#寫入前過濾)
  - [進階（建議）](#進階建議)
- [即時寫入不卡 UI（多執行緒）](#即時寫入不卡-ui多執行緒)
  - [❌ 錯誤做法](#-錯誤做法)
  - [正確做法：Producer-Consumer（佇列）](#正確做法producer-consumer佇列)
  - [建立佇列](#建立佇列)
  - [🧩 UI / 主程式（只負責丟資料）](#-ui--主程式只負責丟資料)
  - [🧩 背景寫檔執行緒](#-背景寫檔執行緒)
- [加強版（避免資料遺失）](#加強版避免資料遺失)
  - [程式結束時](#程式結束時)
  - [等待寫完](#等待寫完)
- [🧠 整體架構圖（重點）](#-整體架構圖重點)
- [3 個實務優化](#3-個實務優化)
  - [1. 批次寫入（效能提升）](#1-批次寫入效能提升)
  - [2. Auto Flush（避免斷電資料不見）](#2-auto-flush避免斷電資料不見)
  - [3. 檔案不存在寫 Header](#3-檔案不存在寫-header)

---

# 官方

[CsvHelper](https://joshclose.github.io/CsvHelper/)
[Getting Started](https://joshclose.github.io/CsvHelper/getting-started/)

[⬆ 回到目錄](#目錄)

---

# CsvHelper 27.2.1 完整實戰教學（含進階用法）

下面給你一份 **CsvHelper 27.2.1 完整實戰教學（含進階用法）**，我會從「安裝 → 基本讀寫 → Mapping → 設定 → 進階技巧」一步一步帶你掌握。

### 📦 一、安裝 CsvHelper 27.2.1

👉 套件：CsvHelper

```bash
dotnet add package CsvHelper --version 27.2.1
```

或

```powershell
Install-Package CsvHelper -Version 27.2.1
```

（支援 .NET Framework / .NET Core / .NET 5+） ([NuGet][1])

### 🧱 二、CSV 範例資料

```csv
Id,Name,Age,Birthday
1,Alice,25,2000-01-01
2,Bob,30,1995-05-10
```

### 🧑‍💻 三、基本讀取（最常用）

#### 1️⃣ 建立 Model

```csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public DateTime Birthday { get; set; }
}
```

#### 2️⃣ 讀取 CSV

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

### ✍️ 四、寫入 CSV

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

### 🧩 五、欄位對應（ClassMap ⭐重要）

當 CSV 欄位名稱不一致時

```csv
user_id,user_name,user_age
```

#### 建立 Mapping

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

#### 使用 Mapping

```csharp
using (var reader = new StreamReader("data.csv"))
using (var csv = new CsvReader(reader, CultureInfo.InvariantCulture))
{
    csv.Context.RegisterClassMap<PersonMap>();

    var records = csv.GetRecords<Person>().ToList();
}
```

### ⚙️ 六、CsvConfiguration（超重要）

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

### 🔄 七、沒有 Header 的 CSV

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

### 🎯 八、用 Index 對應欄位

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

### 🧪 九、型別轉換（重要）

#### 日期格式

```csharp
Map(m => m.Birthday)
    .TypeConverterOption.Format("yyyy-MM-dd");
```

#### 自訂轉換

```csharp
Map(m => m.Name).Convert(row =>
{
    return row.Row.GetField("Name").ToUpper();
});
```

### 🚫 十、忽略欄位

```csharp
Map(m => m.Age).Ignore();
```

或

```csharp
[Ignore]
public int Age { get; set; }
```

### 📊 十一、讀成 DataTable

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

### 🔥 十二、手動逐列讀取（進階）

```csharp
while (csv.Read())
{
    var id = csv.GetField<int>("Id");
    var name = csv.GetField<string>("Name");

    Console.WriteLine(name);
}
```

### 🧠 十三、跳過前幾行（實務很常見）

```csharp
using var reader = new StreamReader("data.csv");

// 跳過前 2 行
reader.ReadLine();
reader.ReadLine();

using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

var records = csv.GetRecords<Person>();
```

### ⚠️ 十四、常見錯誤整理

#### 1️⃣ 中文亂碼

```csharp
new StreamReader("data.csv", Encoding.UTF8);
```

#### 2️⃣ 欄位對不起來

✔ 用 `ClassMap`
✔ 或 `[Index]`

#### 3️⃣ CSV 有逗號

```csv
"Hello, world",123
```

👉 CsvHelper 會自動處理 ✔

#### 4️⃣ 欄位數不一致

👉 需設定：

```csharp
BadDataFound = null
```

### 總結（重點）

| 功能       | 方法                      |
| ---------- | ------------------------- |
| 基本讀取   | `GetRecords<T>()`         |
| 寫入       | `WriteRecords()`          |
| 欄位不一致 | `ClassMap` ⭐             |
| 無標頭     | `HasHeaderRecord = false` |
| 進階控制   | `CsvConfiguration`        |

---

# 「百萬筆效能」＋「亂格式 CSV 容錯」

我直接給你一套 **實戰級寫法（可上線用）**。

### 🚀 一、大檔案（百萬筆）最佳化策略

#### 🔥 核心原則（很重要）

1. ❌ 不要 `.ToList()`（會爆記憶體）
2. 使用 **串流逐筆讀取**
3. 邊讀邊處理（寫 DB / 處理）
4. 關閉不必要驗證
5. 批次處理（Batch）

#### 範例：百萬筆安全讀取（低記憶體）

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

#### ⚡ 更快：手動讀（效能再提升）

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

#### 📦 批次寫入 DB（強烈建議）

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

#### ⚙️ 效能優化重點

| 技巧              | 效果              |
| ----------------- | ----------------- |
| `BufferSize` 調大 | 減少 IO           |
| 手動 `Read()`     | 少反射，快 20~40% |
| Batch 寫入        | DB 效能大提升     |
| 關閉驗證          | 減少 overhead     |

### 🧨 二、亂格式 CSV（重點中的重點）

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

- 欄位缺失 ❌
- 多逗號 ❌
- 引號錯誤 ❌
- 型別錯誤 ❌

#### 容錯設定（必備）

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

#### 安全讀取（防爆版本）

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

#### 🧠 技巧：TryGetField（更安全）

```csharp
if (!csv.TryGetField<int>("Age", out var age))
{
    age = -1;
}
```

#### 🔧 自訂清洗（超實用）

```csharp
Map(m => m.Age).Convert(row =>
{
    var raw = row.Row.GetField("Age");

    if (int.TryParse(raw, out int age))
        return age;

    return -1; // fallback
});
```

### 🧪 三、極端亂格式（分隔符亂掉）

例如：

```csv
1|Alice|25
2|Bob|30
```

👉 解法：

```csharp
Delimiter = "|"
```

### 💣 四、處理破碎 CSV（最難）

例如：

- 換行在欄位內
- quote 沒關

👉 必開：

```csharp
Mode = CsvMode.RFC4180
```

或：

```csharp
DetectDelimiter = true
```

### 🧱 五、終極穩定模板（推薦直接用）

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

### 總結（實務重點）

#### 🚀 大檔案

- 用 `Read()` 或 `GetRecords()`（不要 ToList）
- 批次處理
- 關閉驗證

#### 🧨 亂格式

- `BadDataFound = null`
- `TryGetField`
- `Convert()` 清洗
- try-catch 保底

### 🎯 如果你要更進一步（我可以幫你）

我可以幫你直接做：

- 🔥 100 萬筆 → SQL Server 最快匯入（BulkCopy）
- 🔥 CSV → API 上傳解析（ASP.NET）
- 🔥 自動偵測欄位格式（AI 清洗）
- 🔥 Excel 匯出亂格式修復

👉 直接貼你的 CSV（哪怕很亂），我可以幫你寫「專用解析器」 👍

[⬆ 回到目錄](#目錄)

---

# ClassMap用法

- `Records` 另外存（另一個 CSV）
- 這裡只存「主資料（FasteningRecord 除了 Records）」

其實可以用「很乾淨又快速」的寫法，不需要手動一個一個欄位寫 👇

## 方法一（最推薦）：用 CsvHelper + 忽略 Records

如果你用的是 CsvHelper，可以直接「忽略欄位」👇

### 🧩 建立 ClassMap（重點🔥）

```csharp id="classmap"
public class FasteningRecordMap : ClassMap<FasteningRecord>
{
    public FasteningRecordMap()
    {
        Map(m => m.StartTime);
        Map(m => m.EndTime);
        Map(m => m.TotalRecords);
        Map(m => m.MaxTorque);
        Map(m => m.HeightTop);
        Map(m => m.FirstFastening);
        Map(m => m.ReturnFastening);
        Map(m => m.FirstFasteningAdject);
        Map(m => m.ReturnFasteningAdject);

        Map(m => m.Records).Ignore(); // 27.2.1版寫法
        Ignore(m => m.Records);       // 新版寫法
    }
}
```

### 🧩 寫入方式

```csharp id="write-main"
using (var stream = File.Open("main.csv", FileMode.Append))
using (var writer = new StreamWriter(stream))
using (var csv = new CsvWriter(writer, CultureInfo.InvariantCulture))
{
    csv.Context.RegisterClassMap<FasteningRecordMap>();

    csv.WriteRecord(fasteningRecord);
    csv.NextRecord(); // 換行
}
```

### ✨ 優點

- 不用手動寫欄位
- 可維護性高
- 結構改了只改 Map
- 自動處理格式

## 方法二（超簡單但硬寫）

如果你不想建 Map，也可以直接：

```csharp id="manual"
csv.WriteField(fasteningRecord.StartTime);
csv.WriteField(fasteningRecord.EndTime);
csv.WriteField(fasteningRecord.TotalRecords);
csv.WriteField(fasteningRecord.MaxTorque);
csv.WriteField(fasteningRecord.HeightTop);
csv.WriteField(fasteningRecord.FirstFastening);
csv.WriteField(fasteningRecord.ReturnFastening);
csv.WriteField(fasteningRecord.FirstFasteningAdject);
csv.WriteField(fasteningRecord.ReturnFasteningAdject);

csv.NextRecord();
```

👉 缺點：欄位多會很醜 😅

## 🚀 方法三（偷懶技巧🔥）

如果你懶得寫 Map，又想自動化：

👉 用「匿名物件」直接排除 Records：

```csharp id="anon"
var data = new
{
    fasteningRecord.StartTime,
    fasteningRecord.EndTime,
    fasteningRecord.TotalRecords,
    fasteningRecord.MaxTorque,
    fasteningRecord.HeightTop,
    fasteningRecord.FirstFastening,
    fasteningRecord.ReturnFastening,
    fasteningRecord.FirstFasteningAdject,
    fasteningRecord.ReturnFasteningAdject
};

csv.WriteRecord(data);
csv.NextRecord();
```

## 🧠 建議選哪個？

| 方法            | 推薦程度   | 情境     |
| --------------- | ---------- | -------- |
| ClassMap        | ⭐⭐⭐⭐⭐ | 正式專案 |
| 匿名物件        | ⭐⭐⭐⭐   | 快速開發 |
| 手動 WriteField | ⭐⭐       | 很少欄位 |

## 🧾 一句話結論

👉 **最乾淨做法：用 ClassMap + Ignore(Records)**

## ✨ 額外建議（很重要）

因為你用的是 `struct`：

👉 CsvHelper 對 struct 支援 OK
但**建議改成 class（如果可以）**，原因：

- struct 是 value type（會複製）
- 大資料（含 List）效能不好
- CsvHelper mapping 對 class 比較直覺

---

# 格式化資料

這正是 CsvHelper 的強項之一 👍
`ClassMap` 不只是對應欄位，**也可以直接做格式化（Format / Convert）**。

我幫你整理幾種實務常用的👇

## 1️⃣ 日期 / 時間格式化（最常用）

```csharp id="datetime-format"
Map(m => m.StartTime)
    .TypeConverterOption.Format("yyyy-MM-dd HH:mm:ss.fff");

Map(m => m.EndTime)
    .TypeConverterOption.Format("yyyy-MM-dd HH:mm:ss.fff");
```

👉 輸出會變成：

```
2026-03-27 14:25:30.123
```

## 2️⃣ TimeSpan 格式化

```csharp id="timespan-format"
Map(m => m.FirstFastening)
    .TypeConverterOption.Format(@"hh\:mm\:ss\.fff");
```

👉 注意：

- `:` 要寫成 `\:`（逃脫字元）
- CsvHelper 用的是 .NET 格式

## 3️⃣ 數值小數位控制

```csharp id="double-format"
Map(m => m.MaxTorque)
    .TypeConverterOption.Format("0.000");

Map(m => m.HeightTop)
    .TypeConverterOption.Format("0.00");
```

👉 輸出：

```
5.123
10.50
```

## 4️⃣ 自訂轉換（最強🔥）

如果你要做邏輯處理（例如 null、單位轉換、判斷）：

```csharp id="convert"
Map(m => m.MaxTorque).Convert(args =>
{
    var value = args.Value.MaxTorque;
    return value <= 0 ? "N/A" : value.ToString("0.00");
});
```

## 5️⃣ 改欄位名稱（Header）

```csharp id="header"
Map(m => m.StartTime).Name("Start Time");
Map(m => m.MaxTorque).Name("Max Torque(Nm)");
```

## 6️⃣ 排序欄位

```csharp id="index"
Map(m => m.StartTime).Index(0);
Map(m => m.EndTime).Index(1);
Map(m => m.MaxTorque).Index(2);
```

👉 避免欄位順序亂掉（很重要）

## 🚀 完整範例（含格式化 + Ignore）

```csharp id="full-map"
public class FasteningRecordMap : ClassMap<FasteningRecord>
{
    public FasteningRecordMap()
    {
        Map(m => m.StartTime)
            .Name("Start Time")
            .TypeConverterOption.Format("yyyy-MM-dd HH:mm:ss.fff")
            .Index(0);

        Map(m => m.EndTime)
            .TypeConverterOption.Format("yyyy-MM-dd HH:mm:ss.fff")
            .Index(1);

        Map(m => m.TotalRecords).Index(2);

        Map(m => m.MaxTorque)
            .Name("Max Torque")
            .TypeConverterOption.Format("0.000")
            .Index(3);

        Map(m => m.HeightTop)
            .TypeConverterOption.Format("0.00")
            .Index(4);

        Map(m => m.FirstFastening)
            .TypeConverterOption.Format(@"hh\:mm\:ss\.fff")
            .Index(5);

        Map(m => m.ReturnFastening)
            .TypeConverterOption.Format(@"hh\:mm\:ss\.fff")
            .Index(6);

        Map(m => m.FirstFasteningAdject)
            .TypeConverterOption.Format(@"hh\:mm\:ss\.fff")
            .Index(7);

        Map(m => m.ReturnFasteningAdject)
            .TypeConverterOption.Format(@"hh\:mm\:ss\.fff")
            .Index(8);

        Map(m => m.Records).Ignore();
    }
}
```

## 🧠 小提醒（很重要）

#### ❗ Excel 問題

如果你是給 Excel 用：

👉 建議時間格式用：

```
yyyy-MM-dd HH:mm:ss
```

👉 不然 Excel 有時會解析錯 😅

## 🧾 一句話結論

👉 **FasteningRecordMap 不只可以格式化，還能做轉換、欄位命名、排序，甚至邏輯處理**

---

# 工業設備記錄（MES / 產線資料）標準的做法

我直接給你一套「實務會用、可長期擴充」的設計，不只是能寫 CSV，之後要上資料庫也OK。

## 🧠 核心概念（一定要有）

👉 **主檔（Header） + 明細（Detail）用一個「唯一ID」關聯**

不是用時間去對（時間會撞）
👉 **一定要用 ID（GUID 或流水號）**

## 建議資料結構

### 🔷 主檔（Fastening 主資料）

```csharp id="header"
public class FasteningHeader
{
    public string Id { get; set; }  // ⭐ 關聯用（GUID）

    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public int TotalRecords { get; set; }

    public double MaxTorque { get; set; }
    public double HeightTop { get; set; }

    public TimeSpan FirstFastening { get; set; }
    public TimeSpan ReturnFastening { get; set; }

    public TimeSpan FirstFasteningAdject { get; set; }
    public TimeSpan ReturnFasteningAdject { get; set; }
}
```

### 🔷 明細（Records）

```csharp id="detail"
public class FasteningDetail
{
    public string Id { get; set; } // ⭐ 對應 Header.Id

    public long SequenceNumber { get; set; }
    public double Interval { get; set; }
    public double Height { get; set; }
    public double Torque { get; set; }
}
```

## 🚀 寫入流程（實務標準流程）

### 1️⃣ 產生唯一ID

```csharp id="guid"
var id = Guid.NewGuid().ToString();
```

### 2️⃣ 寫主檔（1筆）

```csharp id="write-header"
var header = new FasteningHeader
{
    Id = id,
    StartTime = fasteningRecord.StartTime,
    EndTime = fasteningRecord.EndTime,
    TotalRecords = fasteningRecord.TotalRecords,
    MaxTorque = fasteningRecord.MaxTorque,
    HeightTop = fasteningRecord.HeightTop,
    FirstFastening = fasteningRecord.FirstFastening,
    ReturnFastening = fasteningRecord.ReturnFastening,
    FirstFasteningAdject = fasteningRecord.FirstFasteningAdject,
    ReturnFasteningAdject = fasteningRecord.ReturnFasteningAdject
};

csvHeader.WriteRecord(header);
csvHeader.NextRecord();
```

### 3️⃣ 寫明細（多筆）

```csharp id="write-detail"
var details = fasteningRecord.Records.Select(r => new FasteningDetail
{
    Id = id, // ⭐ 同一個ID
    SequenceNumber = r.SequenceNumber,
    Interval = r.Interval,
    Height = r.Height,
    Torque = r.Torque
});

csvDetail.WriteRecords(details);
```

## 📂 最終 CSV 長這樣

### 🧾 header.csv

```id="header-csv"
Id,StartTime,EndTime,TotalRecords,MaxTorque,...
A123,2026-03-27 10:00,...,100,5.2,...
B456,2026-03-27 11:00,...,120,6.1,...
```

### 📊 detail.csv

```id="detail-csv"
Id,SequenceNumber,Interval,Height,Torque
A123,1,0.01,10.5,2.3
A123,2,0.02,10.6,2.5
B456,1,0.01,11.0,2.8
```

## ✨ 為什麼這樣設計（重點🔥）

### 1. 可追溯（Traceability）

👉 一筆鎖付資料 = 一個 ID
👉 出問題可以整包抓出來

### 2. 不會資料爆炸

👉 Header 不會重複寫 100 次
👉 CSV 檔案大小差很多

### 3. 未來可上資料庫（超重要）

這就是標準：

- Header → 主表
- Detail → 子表（Foreign Key）

### 4. 支援多機台 / 多產品

你之後只要加欄位：

```csharp id="extend"
public string MachineId { get; set; }
public string ProductId { get; set; }
```

👉 系統直接升級成 MES 等級

## ⚡ 進階優化（實務會做）

### 🔸 ID 可讀化（不要純 GUID）

```csharp id="custom-id"
var id = DateTime.Now.ToString("yyyyMMddHHmmssfff");
```

👉 或：

```id="f3sf3v"
20260327143000123_001
```

### 🔸 加 Batch / 工單

```csharp id="batch"
public string WorkOrder { get; set; }
public string BatchNo { get; set; }
```

## 🧾 一句話結論

👉 **主檔 + 明細 = 用同一個 ID 串起來（不要用時間）**

---

# TimeSpan → 相對秒數

## ✅ 最乾淨做法（用 CsvHelper Map）

用 CsvHelper 的 `Convert`

```csharp id="timespan-seconds"
Map(m => m.FirstFastening).Convert(args =>
    args.Value.FirstFastening.TotalSeconds.ToString());

Map(m => m.ReturnFasteningAdject).Convert(args =>
    args.Value.ReturnFasteningAdject.TotalSeconds.ToString("0.###"));
```

👉 輸出會變：

```id="2t8yqk"
1.234
2.567
```

---

# CSV 每天自動分檔

## 檔名設計（標準）

```csharp id="filename"
string GetFilePath(string folder, string prefix)
{
    var date = DateTime.Now.ToString("yyyyMMdd");
    return Path.Combine(folder, $"{prefix}_{date}.csv");
}
```

👉 結果：

```
header_20260327.csv
detail_20260327.csv
```

---

## 自動切檔（跨天）

```csharp id="rotate-file"
DateTime currentDate = DateTime.Now.Date;

void CheckRotate(ref StreamWriter writer, string folder, string prefix)
{
    if (DateTime.Now.Date != currentDate)
    {
        writer?.Dispose();

        currentDate = DateTime.Now.Date;
        var path = GetFilePath(folder, prefix);

        writer = new StreamWriter(path, true);
    }
}
```

---

# 只存 FAIL（異常資料）

## 定義判斷條件

```csharp id="fail-condition"
bool IsFail(FasteningRecord r)
{
    return r.MaxTorque < 3.0   // 扭力不足
        || r.MaxTorque > 10.0  // 過高
        || r.TotalRecords == 0;
}
```

## 寫入前過濾

```csharp id="fail-filter"
if (!IsFail(fasteningRecord))
    return; // PASS 不存
```

## 進階（建議）

加欄位：

```csharp id="result-field"
public string Result { get; set; } // PASS / FAIL
```

---

# 即時寫入不卡 UI（多執行緒）

👉 這是最關鍵的（很多人寫錯）

## ❌ 錯誤做法

- UI thread 直接寫 CSV
  👉 會卡頓

## 正確做法：Producer-Consumer（佇列）

## 建立佇列

```csharp id="queue"
BlockingCollection<FasteningRecord> queue = new();
```

## 🧩 UI / 主程式（只負責丟資料）

```csharp id="producer"
queue.Add(fasteningRecord);
```

👉 超快，不會卡

## 🧩 背景寫檔執行緒

```csharp id="consumer"
Task.Run(() =>
{
    foreach (var record in queue.GetConsumingEnumerable())
    {
        if (!IsFail(record))
            continue;

        var id = Guid.NewGuid().ToString();

        // 寫 header
        csvHeader.WriteRecord(ConvertToHeader(record, id));
        csvHeader.NextRecord();

        // 寫 detail
        var details = record.Records.Select(r => new FasteningDetail
        {
            Id = id,
            SequenceNumber = r.SequenceNumber,
            Interval = r.Interval,
            Height = r.Height,
            Torque = r.Torque
        });

        csvDetail.WriteRecords(details);
    }
});
```

---

# 加強版（避免資料遺失）

## 程式結束時

```csharp id="complete"
queue.CompleteAdding();
```

## 等待寫完

```csharp id="wait"
consumerTask.Wait();
```

---

# 🧠 整體架構圖（重點）

```
[設備/感測]
      ↓
[UI Thread]  → queue.Add()
      ↓
BlockingCollection
      ↓
[背景寫檔 Thread]
      ↓
CSV（每日分檔 + FAIL only）
```

---

# 3 個實務優化

## 1. 批次寫入（效能提升）

```csharp id="batch"
var batch = queue.GetConsumingEnumerable().Take(100);
csv.WriteRecords(batch);
```

## 2. Auto Flush（避免斷電資料不見）

```csharp id="flush"
writer.AutoFlush = true;
```

## 3. 檔案不存在寫 Header

```csharp id="header-write"
if (!File.Exists(path))
{
    csv.WriteHeader<FasteningHeader>();
    csv.NextRecord();
}
```
