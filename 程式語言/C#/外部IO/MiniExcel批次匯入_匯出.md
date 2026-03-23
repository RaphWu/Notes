---
date: 2024-03-10
author: 余小章
sourceurl: https://dotblogs.azurewebsites.net/yc421206/2024/03/09/via_mini_excel_batch_chunk_import_export_file
tags:
  - CSharp
  - MiniExcel
---

# 通過 MiniExcel 批次匯入/匯出

最近有機會又要操作 Excel，很久以前就知道 MiniExcel，趁假日有機會來把玩一下，這次的重點是研究批次讀寫的使用方式，看看有沒有機會降低一次載入大量 Excel 所造成的記憶體損耗。除了 Excel 之外，它也支援 Csv 呢。還沒開始之前看一下效能比較圖。

![[MiniExcel批次匯入_匯出_比較.png|效能比較圖]]

## 開發環境

- Windows 11
- Rider 2023.3.3
- .NET 8
- MiniExcel 1.31.3
  [mini-software/MiniExcel: Fast, Low-Memory, Easy Excel .NET helper to import/export/template spreadsheet (support Linux, Mac) (github.com)](https://github.com/mini-software/MiniExcel/tree/master)

## 定義型別

跟其他的 Excel 套件一樣，這裡也是用 Attribute 定義欄位名稱、寬度...等等

```csharp
class Data
{
    [ExcelColumnIndex("A")]
    [ExcelColumnName("商品名稱")]
    public string SaleName { get; set; }

    [ExcelColumnIndex("B")]
    [ExcelColumnName("規格")]
    public string Option { get; set; }

    [ExcelColumnIndex("C")]
    [ExcelColumnName("編號")]
    public string Id { get; set; }

    [ExcelColumnIndex("D")]
    [ExcelColumnName("Code")]
    public string Code { get; set; }

    [ExcelColumnIndex("E")]
    [ExcelColumnName("庫存")]
    public string StockQty { get; set; }
}
```

更多的細節參考
[MiniExcel/README.zh-Hant.md at master · mini-software/MiniExcel (github.com)](https://github.com/mini-software/MiniExcel/blob/master/README.zh-Hant.md#excel-%E5%88%97%E5%B1%AC%E6%80%A7-excel-column-attribute-)

## 讀檔

- 支援強型別集合(`IEnumerable<T>`)、`dynamic` 集合
- 支援延遲載入
- 讀檔時，下列方法擇一使用
  - `var inputRows = await MiniExcel.QueryAsync<Data>(inputPath);`
  - `var inputRows = await inputStream.QueryAsync<Data>();`

```csharp
[Fact]
public async Task 批次匯入大檔案()
{
    //import large excel file
    var inputPath = "10萬.xlsx";
    var chunkSize = 1024;

    await using var inputStream = File.OpenRead(inputPath);
    var results = new List<Data>();

    // chunk read the data
    var inputRows = await inputStream.QueryAsync<Data>();
    foreach (var chunks in inputRows.Chunk(chunkSize))
    {
        results.AddRange(chunks);
    }
}
```

官方文件強調

> 除非必要請不要使用 `ToList` 等方法讀取全部資料到記憶體

## 存檔

- 支援強型別、匿名型別、字典集合、`IEnumerable<IDictionary<string, object>>)`
- 存檔時，下列方法擇一
  - `await MiniExcel.SaveAsAsync(outputPath, value, overwriteFile: true);`
  - `await outputStream.SaveAsAsync(value);`

```csharp
public async Task 強型別另存會員表()
{
    var outputPath = "MemberResult.xlsx";

    await using var outputStream = File.Open(outputPath, FileMode.OpenOrCreate, FileAccess.ReadWrite);
    var value = new List<Member>()
    {
        new()
        {
            Id = "1",
            Name = "Alice",
            Birthday = DateTime.Now,
            Age = 25,
            Phone = "1234567890",
            Reason = null,
        },
        new()
        {
            Id = "2",
            Name = "Bob",
            Birthday = DateTime.Now,
            Age = 35,
            Phone = "1234567890",
            Reason = "年齡超過30",
        }
    };

    // await MiniExcel.SaveAsAsync(outputPath, value, overwriteFile: true);
    await outputStream.SaveAsAsync(value);
}
```

## 批次匯入並另存範本檔

在 Excel 定義模板 `{{變量名稱}}`, 或是集合渲染 `{{集合名稱.欄位名稱}}`
![[MiniExcel批次匯入_匯出_模板.png]]

定義型別

```csharp
class Member
{
    [ExcelColumnName(excelColumnName: "編號", aliases: ["MemberId"])]
    public string Id { get; set; }

    [ExcelColumnName(excelColumnName: "姓名", aliases: ["FullName"])]
    public string Name { get; set; }

    [ExcelColumnName(excelColumnName: "生日")]
    public DateTime Birthday { get; set; }

    // [ExcelColumnName(excelColumnName: "年齡", aliases: ["Age"])]
    [DisplayName("年齡")]
    public int Age { get; set; }

    [DisplayName("電話")]

    public string Phone { get; set; }

    [DisplayName("失敗原因")]
    public string Reason { get; set; }
}
```

先用 `QueryAsync` 讀檔，每批讀兩筆，再用 `SaveAsByTemplateAsync` 另存新檔

```csharp
[Fact]
public async Task 批次匯入後批次填充範本()
{
    //import excel file
    var inputPath = "Import.xlsx";
    var outputPath = "MemberResult.xlsx";
    var templatePath = "Template/Member.xlsx";
    var chunkSize = 2;

    await using var inputStream = File.OpenRead(inputPath);
    var inputRows = await inputStream.QueryAsync<Member>();
    var results = new List<Member>();
    foreach (var chunks in inputRows.Chunk(chunkSize))
    {
        foreach (var row in chunks)
        {
            var age = (DateTime.Now - row.Birthday).TotalDays / 365.25;
            if (age > 30)
            {
                row.Reason = "年齡超過30";
            }

            row.Age = (int)age;
        }

        results.AddRange(chunks);
        var value = new
        {
            Members = results
        };

        //Append to the same file
        await MiniExcel.SaveAsByTemplateAsync(outputPath, templatePath, value);
    }
}
```

每次兩筆兩筆的寫入同一個檔案，最終有 10 筆，這好棒
![[MiniExcel批次匯入_匯出_SaveAsByTemplateAsync.png]]

經實驗，當筆數大時，這樣的寫法檔案會產不出來，需要把存檔的動作放在迴圈外面

```csharp
[Fact]
public async Task 產生大資料填充範本()
{
    //import excel file
    var outputPath = "MemberResult.xlsx";
    var templatePath = "Template/Member.xlsx";
    var chunkSize = 128;

    //generate 10000000 member row
    var inputRows = Enumerable.Range(1, 600000).Select(x => new Member
    {
        Id = x.ToString(),
        Name = "Name" + x,
        Birthday = DateTime.Now,
        Phone = "1234567890"
    });
    var results = new List<Member>();
    foreach (var chunks in inputRows.Chunk(chunkSize))
    {
        foreach (var row in chunks)
        {
            var age = (DateTime.Now - row.Birthday).TotalDays / 365.25;
            if (age > 30)
            {
                row.Reason = "年齡超過30";
            }

            row.Age = (int)age;
        }

        results.AddRange(chunks);
    }

    var value = new
    {
        Members = results
    };

    //Append to the same file
    MiniExcel.SaveAsByTemplate(outputPath, templatePath, value);
}
```

## SaveAsByTemplate vs SaveAs 比較

比較了一下，填充範本跟直接匯出所需要的時間，兩者的花費的時間差蠻多的，匯出 800000 筆的場景

`MiniExcel.SaveAsByTemplate()` 約花費 40sec

`MiniExcel.SaveAs()` 約花費 7sec

## 結論

最讓我驚豔的就是批次匯入，這是過往我沒有過的使用體驗，往往碰到大一點的檔案，記憶體就被吃掉一大票，現在就看看在真實現場的表現如何

範例位置
[sample.dotblog/Excel/Lab.MiniExcelQuery at master · yaochangyu/sample.dotblog (github.com)](https://github.com/yaochangyu/sample.dotblog/tree/master/Excel/Lab.MiniExcelQuery)
