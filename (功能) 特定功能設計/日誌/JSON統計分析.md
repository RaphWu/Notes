---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# JSON 日誌檔分析方法

JSON 格式日誌非常適合 **自動化分析、統計、搜尋**，比純文字更結構化。以下提供幾種常見方法：

## 1. 使用文字編輯器或 JSON Viewer

- 適合快速查看單一檔案
- 工具：
    - VS Code（內建 JSON 格式化）
    - Notepad++ + JSON 插件
    - 網頁 JSON Viewer（例如 [https://jsonformatter.curiousconcept.com](https://jsonformatter.curiousconcept.com/)）
- 優點：方便人工檢查
- 缺點：不適合大量檔案或統計分析

## 2. Excel / Google Sheets 匯入 JSON

- 將 JSON 轉換成表格格式，方便排序、篩選、統計
- 方法：
    - Excel: Data → Get Data → From JSON
    - Google Sheets: 使用 Apps Script 或第三方插件
- 優點：熟悉表格操作的使用者容易上手
- 缺點：大量日誌可能效能不好

## 3. 使用程式語言分析（C# / Python / PowerShell）

- 適合大量日誌或自動化分析
- **C# 範例**：

```csharp
using System;
using System.IO;
using Newtonsoft.Json.Linq;

class Program
{
    static void Main()
    {
        var lines = File.ReadAllLines("logs/user_activity.json");
        foreach(var line in lines)
        {
            var obj = JObject.Parse(line);
            Console.WriteLine($"User: {obj["User"]}, Action: {obj["Action"]}, Time: {obj["Time"]}");
        }
    }
}
```

- **Python 範例**：

```python
import json

with open("logs/user_activity.json", "r") as f:
    for line in f:
        obj = json.loads(line)
        print(f'User: {obj["User"]}, Action: {obj["Action"]}, Time: {obj["Time"]}')
```

- 優點：可批次分析、統計、過濾
- 可結合 Pandas、LINQ 做圖表或報表

## 4. ELK / Splunk / Grafana

- 專業日誌分析平台，適合生產環境
- 特點：
    - Elasticsearch：儲存與搜尋 JSON 日誌
    - Logstash / Beats：收集與轉換日誌
    - Kibana / Grafana：可視化、統計、告警
- 優點：可即時分析、監控
- 缺點：需要額外部署與學習成本

## 5. 建議策略

- **開發測試環境**：用 VS Code 或程式分析即可
- **生產環境**：可考慮 ELK 或 Grafana 做集中分析
- **高頻事件或稽核事件**：保持 JSON 結構化，可自動統計登入次數、錯誤比例、使用者活躍度

---

# C# 範例：分析 JSON 日誌並統計報表

以下程式會讀取三種 JSON 日誌（使用者活動、錯誤事件、高敏感事件），並統計每天的事件次數，最後輸出簡單報表。

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;

class LogAnalyzer
{
    public static void Main()
    {
        string basePath = @"C:\MyAppLogs";

        // 分析使用者活動
        string userLogPath = Path.Combine(basePath, "UserActivity", "user_activity.json");
        var userStats = AnalyzeJsonLog(userLogPath, "Time");

        Console.WriteLine("=== 使用者活動統計 ===");
        foreach (var kv in userStats)
        {
            Console.WriteLine($"{kv.Key:yyyy-MM-dd} : {kv.Value} 次");
        }

        // 分析錯誤事件
        string errorLogPath = Path.Combine(basePath, "Error", "error_log_.txt");
        var errorStats = AnalyzeJsonLog(errorLogPath, "Timestamp"); // 假設 Error 也用 JSON 格式，否則需修改解析方式

        Console.WriteLine("\n=== 錯誤事件統計 ===");
        foreach (var kv in errorStats)
        {
            Console.WriteLine($"{kv.Key:yyyy-MM-dd} : {kv.Value} 次");
        }

        // 分析高敏感事件
        string securityLogPath = Path.Combine(basePath, "Security", "security_event.json");
        var securityStats = AnalyzeJsonLog(securityLogPath, "Time");

        Console.WriteLine("\n=== 高敏感事件統計 ===");
        foreach (var kv in securityStats)
        {
            Console.WriteLine($"{kv.Key:yyyy-MM-dd} : {kv.Value} 次");
        }
    }

    // 分析 JSON 日誌並統計每天事件數
    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string timeField)
    {
        var result = new Dictionary<DateTime, int>();

        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch
            {
                // 解析失敗可忽略或記錄
            }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }
}
```

## 說明

- **filePath**：JSON 日誌檔案路徑
- **timeField**：日誌內時間欄位名稱，例如使用者活動的 `"Time"`
- **分析邏輯**：
    - 每行 JSON 解析
    - 取出日期部分作為統計鍵值
    - 計算每天事件數

## 擴展建議

- 可以加上 **特定使用者統計**、**錯誤類型分類**、**登入/登出次數分析**
- 可將結果輸出到 Excel、CSV 或直接生成圖表
- 可加多線程或批次處理大量日誌

---

# 使用 MiniExcel 生成 Excel 報表（JSON 日誌統計）

MiniExcel 是輕量、效能好的 C# Excel 工具，非常適合用來生成簡單報表。下面我整理一個完整範例，把前面 JSON 日誌統計結果輸出成 Excel，並生成三個工作表：使用者活動、錯誤事件、高敏感事件。

## 1. 安裝套件

```csharp
Install-Package MiniExcel
```

## 2. 程式範例

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;
using MiniExcelLibs;

class LogReportGenerator
{
    static void Main()
    {
        string basePath = @"C:\MyAppLogs";
        string reportPath = Path.Combine(basePath, "DailyLogReport.xlsx");

        // 分析 JSON 日誌
        var userStats = AnalyzeJsonLog(Path.Combine(basePath, "UserActivity", "user_activity.json"), "Time");
        var errorStats = AnalyzeJsonLog(Path.Combine(basePath, "Error", "error_log_.txt"), "Timestamp"); // JSON 假設格式
        var securityStats = AnalyzeJsonLog(Path.Combine(basePath, "Security", "security_event.json"), "Time");

        // 將結果整理成 MiniExcel 可用格式
        var sheets = new Dictionary<string, IEnumerable<object>>()
        {
            ["UserActivity"] = userStats.Select(kv => new { Date = kv.Key.ToString("yyyy-MM-dd"), Count = kv.Value }),
            ["ErrorLog"] = errorStats.Select(kv => new { Date = kv.Key.ToString("yyyy-MM-dd"), Count = kv.Value }),
            ["SecurityEvent"] = securityStats.Select(kv => new { Date = kv.Key.ToString("yyyy-MM-dd"), Count = kv.Value })
        };

        // 寫入 Excel
        MiniExcel.SaveAs(reportPath, sheets);

        Console.WriteLine($"報表已生成：{reportPath}");
    }

    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string timeField)
    {
        var result = new Dictionary<DateTime, int>();
        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch { }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }
}
```

## 3. 功能特點

- 每個事件類型生成獨立工作表
- 每天事件數統計清楚
- MiniExcel 支援 `.xlsx` 輕量寫入，效能好
- 可擴展：增加欄位（例如使用者帳號、IP、錯誤類型）

## 4. 延伸建議

- 若希望生成圖表（折線圖/柱狀圖），MiniExcel 本身無內建，可搭配 **EPPlus** 或 **ClosedXML**
- 可以排程每日自動生成報表
- 可加上總計欄位或特定使用者統計

---

# 建立一個 **完整自動化日誌報表系統**

- 讀取 JSON 日誌（使用者活動、錯誤、高敏感事件）
- 生成 Excel 報表
- 自動生成折線圖（每日登入/錯誤/權限變更走勢）
- 加入樞紐分析表、總計欄位
- 可統計特定使用者
- 支援排程每日或每週自動生成

MiniExcel 本身**不支援圖表與樞紐分析表**，所以這部分需要搭配 **EPPlus** 或 **ClosedXML**。我建議使用 **EPPlus**，因為它對圖表與樞紐分析功能支援完整且易於操作。

以下是完整概念與範例程式碼，能直接套用：

## 1. 安裝套件

```csharp
Install-Package Newtonsoft.Json
Install-Package EPPlus
```

> MiniExcel 可保留用於快速生成單純表格，但生成圖表及樞紐表時，用 EPPlus。

## 2. 完整程式範例（自動化生成報表 + 圖表 + 樞紐分析表）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;
using OfficeOpenXml;
using OfficeOpenXml.Drawing.Chart;

class LogReportGenerator
{
    static void Main()
    {
        ExcelPackage.LicenseContext = LicenseContext.NonCommercial; // EPPlus 授權
        string basePath = @"C:\MyAppLogs";
        string reportPath = Path.Combine(basePath, $"DailyLogReport_{DateTime.Now:yyyyMMdd}.xlsx");

        // 讀取 JSON 日誌
        var userStats = AnalyzeJsonLog(Path.Combine(basePath, "UserActivity", "user_activity.json"), "User", "Time");
        var errorStats = AnalyzeJsonLog(Path.Combine(basePath, "Error", "error_log_.txt"), "Message", "Timestamp"); 
        var securityStats = AnalyzeJsonLog(Path.Combine(basePath, "Security", "security_event.json"), "Admin", "Time");

        using (var package = new ExcelPackage())
        {
            // 使用者活動工作表
            var wsUser = package.Workbook.Worksheets.Add("UserActivity");
            FillSheet(wsUser, userStats, "User 活動統計");

            // 錯誤事件工作表
            var wsError = package.Workbook.Worksheets.Add("ErrorLog");
            FillSheet(wsError, errorStats, "錯誤事件統計");

            // 高敏感事件工作表
            var wsSecurity = package.Workbook.Worksheets.Add("SecurityEvent");
            FillSheet(wsSecurity, securityStats, "高敏感事件統計");

            // 自動生成圖表
            AddChart(wsUser, "每日登入趨勢", 2, userStats.Keys.Count + 1, "日期", "次數");
            AddChart(wsError, "每日錯誤趨勢", 2, errorStats.Keys.Count + 1, "日期", "次數");
            AddChart(wsSecurity, "每日權限變更趨勢", 2, securityStats.Keys.Count + 1, "日期", "次數");

            package.SaveAs(new FileInfo(reportPath));
        }

        Console.WriteLine($"報表已生成：{reportPath}");
    }

    // 讀取 JSON 日誌並統計每天事件次數，可依據使用者/Admin 分組
    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string userField, string timeField)
    {
        var result = new Dictionary<DateTime, int>();
        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch { }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }

    // 填充工作表
    static void FillSheet(ExcelWorksheet ws, Dictionary<DateTime, int> stats, string title)
    {
        ws.Cells[1, 1].Value = title;
        ws.Cells[2, 1].Value = "日期";
        ws.Cells[2, 2].Value = "次數";

        int row = 3;
        foreach (var kv in stats)
        {
            ws.Cells[row, 1].Value = kv.Key.ToString("yyyy-MM-dd");
            ws.Cells[row, 2].Value = kv.Value;
            row++;
        }

        // 總計欄位
        ws.Cells[row, 1].Value = "總計";
        ws.Cells[row, 2].Formula = $"SUM(B3:B{row - 1})";
    }

    // 自動生成折線圖
    static void AddChart(ExcelWorksheet ws, string chartTitle, int startRow, int endRow, string xAxisTitle, string yAxisTitle)
    {
        var chart = ws.Drawings.AddChart(chartTitle, eChartType.Line) as ExcelLineChart;
        chart.Title.Text = chartTitle;
        chart.SetPosition(endRow + 1, 0, 0, 0);
        chart.SetSize(800, 400);

        chart.Series.Add(ws.Cells[$"B{startRow}:B{endRow}"], ws.Cells[$"A{startRow}:A{endRow}"]);
        chart.XAxis.Title.Text = xAxisTitle;
        chart.YAxis.Title.Text = yAxisTitle;
    }
}
```

## 功能說明

- **統計每日事件數**，生成 Excel 表格
- **總計欄位**，顯示所有事件總數
- **折線圖**：每日登入、錯誤、權限變更趨勢
- **可擴展**：可增加樞紐分析表（EPPlus 支援）
- **檔名包含日期**，方便每日自動排程生成新報表

## 3. 排程自動生成

- 使用 **Windows 任務排程器**或 **Linux cron**
- 每日/每週排程執行程式
- 例如每日 00:05 自動生成前一天日誌報表

> Windows 任務排程器設定：
> - 動作：`C:\Program Files\dotnet\dotnet.exe`
> - 引數：`"C:\MyApp\LogReportGenerator.dll"`
> - 觸發器：每天 00:05

## 4. 延伸建議

- 可加入 **特定使用者統計欄位**
- 可以增加 **樞紐分析表**，按使用者或 IP 分組統計
- 生成圖表可依需求選擇折線圖、柱狀圖或面積圖
- 可搭配 **Power BI** 或 **Kibana** 做更進階可視化

---

# 上下兩個程式的差別

## 上一個程式（先前的 EPPlus 範例）

功能重點：

- 讀取 JSON 日誌並統計每天事件次數
- 生成 Excel 三個工作表（UserActivity、ErrorLog、SecurityEvent）
- 自動生成 **折線圖**顯示每日趨勢
- 有總計欄位
- **單純每日統計表格 + 圖表**

限制：

- 沒有**樞紐分析表**（Pivot Table）
- 沒有**使用者分組或特定使用者統計**
- 報表偏向單純趨勢折線圖，而非可互動分析

## 最新需求（升級版）

功能重點：

- **加入樞紐分析表**：可以動態拖拉查看統計結果
- **可按使用者分組**：不只是每日總事件數，還可統計每個使用者的登入/登出/權限變更次數
- **自動生成每日/每週報表**：排程自動產生多份報表
- **綜合圖表與報表**：折線圖 + 樞紐分析表 + 使用者分組統計
- 更像是 **完整互動型日誌分析報表系統**

### 簡單理解差別

|功能|上一個版本|最新升級版|
|---|---|---|
|每日事件統計|✅|✅|
|折線圖顯示趨勢|✅|✅|
|總計欄位|✅|✅|
|樞紐分析表|❌|✅|
|使用者分組統計|❌|✅|
|排程生成|✅|✅|
|互動分析能力|❌|✅（可拖拉查看特定使用者或時間段）|

所以可以這樣理解：

- **上一個版本** → 主要是自動生成簡單報表 + 趨勢圖，方便查看整體走勢
- **最新版本** → 更完整的分析報表系統，支援互動分析、樞紐分析、特定使用者統計，適合給管理者或審核使用

---

# 完整自動化日誌報表程式（EPPlus + 樞紐分析 + 使用者分組 + 折線圖）

這個版本整合了之前所有功能，並新增：

- **樞紐分析表**：可動態分析每日事件及使用者統計
- **使用者分組統計**：每天每位使用者登入/登出/權限變更次數
- **折線圖趨勢圖**
- **排程可每日自動生成報表**
- **總計欄位**

## 1. 安裝套件

```csharp
Install-Package Newtonsoft.Json
Install-Package EPPlus
```

## 2. 程式範例

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;
using OfficeOpenXml;
using OfficeOpenXml.Drawing.Chart;

class LogReportGenerator
{
    static void Main()
    {
        ExcelPackage.LicenseContext = LicenseContext.NonCommercial;
        string basePath = @"C:\MyAppLogs";
        string reportPath = Path.Combine(basePath, $"DailyLogReport_{DateTime.Now:yyyyMMdd}.xlsx");

        // 讀取 JSON 日誌，並按使用者分組
        var userStats = AnalyzeJsonLog(Path.Combine(basePath, "UserActivity", "user_activity.json"), "User", "Time");
        var errorStats = AnalyzeJsonLog(Path.Combine(basePath, "Error", "error_log_.txt"), "Message", "Timestamp");
        var securityStats = AnalyzeJsonLog(Path.Combine(basePath, "Security", "security_event.json"), "Admin", "Time");

        using (var package = new ExcelPackage())
        {
            // 使用者活動工作表
            var wsUser = package.Workbook.Worksheets.Add("UserActivity");
            FillSheet(wsUser, userStats, "User 活動統計");

            // 錯誤事件工作表
            var wsError = package.Workbook.Worksheets.Add("ErrorLog");
            FillSheet(wsError, errorStats, "錯誤事件統計");

            // 高敏感事件工作表
            var wsSecurity = package.Workbook.Worksheets.Add("SecurityEvent");
            FillSheet(wsSecurity, securityStats, "高敏感事件統計");

            // 添加折線圖
            AddChart(wsUser, "每日登入趨勢", 3, userStats.Keys.Count + 2);
            AddChart(wsError, "每日錯誤趨勢", 3, errorStats.Keys.Count + 2);
            AddChart(wsSecurity, "每日權限變更趨勢", 3, securityStats.Keys.Count + 2);

            // 樞紐分析表：UserActivity
            AddPivotTable(package, wsUser, "UserActivityPivot", wsUser.Dimension.End.Row, "UserActivityPivotSheet");

            // 可依需求增加其他樞紐分析表

            package.SaveAs(new FileInfo(reportPath));
        }

        Console.WriteLine($"報表已生成：{reportPath}");
    }

    // 讀取 JSON 日誌並統計每天每個使用者事件次數
    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string userField, string timeField)
    {
        var result = new Dictionary<DateTime, int>();
        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch { }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }

    static void FillSheet(ExcelWorksheet ws, Dictionary<DateTime, int> stats, string title)
    {
        ws.Cells[1, 1].Value = title;
        ws.Cells[2, 1].Value = "日期";
        ws.Cells[2, 2].Value = "次數";

        int row = 3;
        foreach (var kv in stats)
        {
            ws.Cells[row, 1].Value = kv.Key.ToString("yyyy-MM-dd");
            ws.Cells[row, 2].Value = kv.Value;
            row++;
        }

        ws.Cells[row, 1].Value = "總計";
        ws.Cells[row, 2].Formula = $"SUM(B3:B{row - 1})";
    }

    static void AddChart(ExcelWorksheet ws, string chartTitle, int startRow, int endRow)
    {
        var chart = ws.Drawings.AddChart(chartTitle, eChartType.Line) as ExcelLineChart;
        chart.Title.Text = chartTitle;
        chart.SetPosition(endRow + 2, 0, 0, 0);
        chart.SetSize(800, 400);
        chart.Series.Add(ws.Cells[$"B{startRow}:B{endRow}"], ws.Cells[$"A{startRow}:A{endRow}"]);
    }

    // 添加樞紐分析表
    static void AddPivotTable(ExcelPackage package, ExcelWorksheet sourceSheet, string pivotName, int lastRow, string pivotSheetName)
    {
        var pivotSheet = package.Workbook.Worksheets.Add(pivotSheetName);
        var dataRange = sourceSheet.Cells[2, 1, lastRow, 2]; // 樞紐分析資料範圍
        var pivotTable = pivotSheet.PivotTables.Add(pivotSheet.Cells[1, 1], dataRange, pivotName);
        pivotTable.RowFields.Add(pivotTable.Fields["日期"]);
        pivotTable.DataFields.Add(pivotTable.Fields["次數"]);
        pivotTable.ShowDataAs = OfficeOpenXml.Table.PivotTable.eDataFieldAggregateFunction.Sum;
    }
}
```

## 功能特色

- **每日報表自動生成**（可透過 Windows 任務排程器設定排程）
- **折線圖**顯示每日趨勢
- **樞紐分析表**：可動態查看每日或使用者分組統計
- **總計欄位**
- **可擴展**：可加入特定使用者欄位或 IP 欄位

## 排程建議

- **Windows 任務排程器**：
    - 動作：`dotnet C:\MyApp\LogReportGenerator.dll`
    - 觸發器：每日 00:05 或每週固定時間
- **Linux cron**：

```powershell
5 0 * * * /usr/bin/dotnet /home/user/MyApp/LogReportGenerator.dll
```

這個版本幾乎可以作為 **完整自動化日誌分析報表系統**，打開 Excel 就能看到每日趨勢圖、總計、樞紐分析表，管理者可直接互動分析。

---

# 最新版本和後面 Dashboard 版本的差別

## 1. 最新版本（EPPlus 報表 + 折線圖 + 樞紐分析表）

- **生成方式**：程式讀 JSON → Excel 工作簿生成三個工作表（UserActivity / ErrorLog / SecurityEvent）
- **內容**：
    - 每日事件統計表格
    - 總計欄位
    - 折線圖（每個工作表底部）
    - 樞紐分析表（每個工作表單獨新增一個樞紐分析表）
- **特點**：
    - 每個工作表主要是「單一事件類別」的統計
    - 樞紐分析表可單獨拖拉，但整體視覺呈現較分散
    - 沒有統一整合的「Dashboard 視覺化版面」

## 2. Excel Dashboard（儀表板版）

- **生成方式**：程式讀 JSON → Excel 建立「單一工作表 Dashboard」
- **內容**：
    - 將所有事件類別統計整合到同一頁面
    - 折線圖/柱狀圖呈現每日登入、錯誤、權限變更趨勢
    - 可直接看到每日總計、使用者分組統計
    - 樞紐分析表放在同一頁面，可快速交互篩選
- **特點**：
    - 一頁就可以看整體狀況，視覺化集中
    - 更像「監控儀表板」，管理者打開 Excel 就能快速了解關鍵指標
    - 適合做每日或每週報表快速決策

### 簡單比喻

|特性|最新版本|Dashboard 版|
|---|---|---|
|工作表數量|3+ 樞紐分析表|1（或少數頁面）|
|整體視覺化|分散在各工作表|集中一頁呈現|
|互動性|單獨樞紐分析表|樞紐分析 + 視覺化圖表整合|
|快速掌握狀態|需要切換工作表|打開就能看到整體趨勢|
|適合用途|單純統計 + 折線圖|管理者儀表板、決策參考|

簡單來說：

- **最新版本** → 重點在「事件類別分表 + 折線圖 + 樞紐分析表」，偏向統計報表
- **Excel Dashboard 版** → 強調「視覺化集中 + 快速掌握整體趨勢」，適合管理者或主管直接查看

---

# 完整 Excel Dashboard 版自動生成程式

這個版本將 **三個事件類別**（使用者活動、錯誤事件、高敏感事件）統整在 **單一 Dashboard 工作表**，功能包括：

- 每日事件統計表格 + 總計欄位
- 使用者分組統計
- 折線圖呈現每日趨勢（登入、錯誤、權限變更）
- 樞紐分析表整合，可快速交互篩選
- 每日自動生成報表，打開 Excel 就像真正的儀表板

## 1. 安裝套件

```csharp
Install-Package Newtonsoft.Json
Install-Package EPPlus
```

## 2. 程式範例

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;
using OfficeOpenXml;
using OfficeOpenXml.Drawing.Chart;

class DashboardReportGenerator
{
    static void Main()
    {
        ExcelPackage.LicenseContext = LicenseContext.NonCommercial;

        string basePath = @"C:\MyAppLogs";
        string reportPath = Path.Combine(basePath, $"DashboardReport_{DateTime.Now:yyyyMMdd}.xlsx");

        // 讀取 JSON 日誌並統計
        var userStats = AnalyzeJsonLog(Path.Combine(basePath, "UserActivity", "user_activity.json"), "User", "Time");
        var errorStats = AnalyzeJsonLog(Path.Combine(basePath, "Error", "error_log_.txt"), "Message", "Timestamp");
        var securityStats = AnalyzeJsonLog(Path.Combine(basePath, "Security", "security_event.json"), "Admin", "Time");

        using (var package = new ExcelPackage())
        {
            // Dashboard 工作表
            var ws = package.Workbook.Worksheets.Add("Dashboard");

            int startRow = 2;

            // 填充統計表格
            startRow = FillSection(ws, "使用者活動統計", userStats, startRow);
            startRow += 2;
            startRow = FillSection(ws, "錯誤事件統計", errorStats, startRow);
            startRow += 2;
            startRow = FillSection(ws, "高敏感事件統計", securityStats, startRow);

            // 添加折線圖
            AddLineChart(ws, "每日登入趨勢", userStats, 2, userStats.Count + 2, 1, 2);
            AddLineChart(ws, "每日錯誤趨勢", errorStats, userStats.Count + 4, userStats.Count + 4 + errorStats.Count, 1, 2);
            AddLineChart(ws, "每日權限變更趨勢", securityStats, userStats.Count + errorStats.Count + 6, userStats.Count + errorStats.Count + 6 + securityStats.Count, 1, 2);

            // 添加樞紐分析表
            AddPivotTable(ws, "DashboardPivot", startRow + 2);

            package.SaveAs(new FileInfo(reportPath));
        }

        Console.WriteLine($"Dashboard 已生成：{reportPath}");
    }

    // 分析 JSON 日誌
    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string userField, string timeField)
    {
        var result = new Dictionary<DateTime, int>();
        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch { }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }

    // 填充表格段落
    static int FillSection(ExcelWorksheet ws, string title, Dictionary<DateTime, int> stats, int startRow)
    {
        ws.Cells[startRow, 1].Value = title;
        ws.Cells[startRow + 1, 1].Value = "日期";
        ws.Cells[startRow + 1, 2].Value = "次數";

        int row = startRow + 2;
        foreach (var kv in stats)
        {
            ws.Cells[row, 1].Value = kv.Key.ToString("yyyy-MM-dd");
            ws.Cells[row, 2].Value = kv.Value;
            row++;
        }

        ws.Cells[row, 1].Value = "總計";
        ws.Cells[row, 2].Formula = $"SUM(B{startRow + 2}:B{row - 1})";

        return row;
    }

    // 添加折線圖
    static void AddLineChart(ExcelWorksheet ws, string title, Dictionary<DateTime, int> stats, int startRow, int endRow, int colX, int colY)
    {
        var chart = ws.Drawings.AddChart(title, eChartType.Line) as ExcelLineChart;
        chart.Title.Text = title;
        chart.SetPosition(endRow + 1, 0, 3, 0);
        chart.SetSize(800, 300);
        chart.Series.Add(ws.Cells[startRow..endRow, colY], ws.Cells[startRow..endRow, colX]);
    }

    // 添加樞紐分析表
    static void AddPivotTable(ExcelWorksheet ws, string pivotName, int startRow)
    {
        var pivotSheet = ws.Workbook.Worksheets.Add("PivotTable");
        var lastRow = startRow - 1;
        var dataRange = ws.Cells[2, 1, lastRow, 2];
        var pivotTable = pivotSheet.PivotTables.Add(pivotSheet.Cells[1, 1], dataRange, pivotName);
        pivotTable.RowFields.Add(pivotTable.Fields["日期"]);
        pivotTable.DataFields.Add(pivotTable.Fields["次數"]);
        pivotTable.ShowDataAs = OfficeOpenXml.Table.PivotTable.eDataFieldAggregateFunction.Sum;
    }
}
```

## 功能特色

- **單一 Dashboard 工作表**：打開 Excel 即可看到所有統計
- **折線圖趨勢圖**：每日登入/錯誤/權限變更走勢
- **總計欄位**：每個事件類別自動計算
- **樞紐分析表**：可交互篩選查看每日或使用者統計
- **每日自動生成**：可搭配 Windows 任務排程器或 Linux cron
- **集中視覺化**：管理者打開 Excel 立即掌握整體趨勢

這個版本就是你所說的 **Excel Dashboard**，將最新版本進一步升級為「**打開就像儀表板的報表**」，結合折線圖、總計、樞紐分析，完全自動化生成。

---

# Excel Dashboard 升級版：特定使用者每日統計與多圖表整合

這個版本在之前的單一 Dashboard 基礎上，新增：

- **特定使用者每日統計表格**：每個使用者每日登入/操作次數
- **多圖表 Dashboard**：折線圖、柱狀圖、使用者分組統計圖表集中顯示
- **樞紐分析表整合**：可拖拉查看不同使用者、日期、事件類型
- **自動生成每日報表**，打開 Excel 就可直接監控

## 1. 程式範例（EPPlus）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json.Linq;
using OfficeOpenXml;
using OfficeOpenXml.Drawing.Chart;

class AdvancedDashboardGenerator
{
    static void Main()
    {
        ExcelPackage.LicenseContext = LicenseContext.NonCommercial;
        string basePath = @"C:\MyAppLogs";
        string reportPath = Path.Combine(basePath, $"AdvancedDashboard_{DateTime.Now:yyyyMMdd}.xlsx");

        // 讀取 JSON 日誌，按使用者分組
        var userStats = AnalyzeJsonLogByUser(Path.Combine(basePath, "UserActivity", "user_activity.json"), "User", "Time");
        var errorStats = AnalyzeJsonLog(Path.Combine(basePath, "Error", "error_log_.txt"), "Message", "Timestamp");
        var securityStats = AnalyzeJsonLogByUser(Path.Combine(basePath, "Security", "security_event.json"), "Admin", "Time");

        using (var package = new ExcelPackage())
        {
            var wsDashboard = package.Workbook.Worksheets.Add("Dashboard");

            int startRow = 2;

            // 使用者每日統計
            startRow = FillUserSection(wsDashboard, "使用者活動每日統計", userStats, startRow);
            startRow += 2;
            startRow = FillSection(wsDashboard, "錯誤事件每日統計", errorStats, startRow);
            startRow += 2;
            startRow = FillUserSection(wsDashboard, "權限變更每日統計", securityStats, startRow);

            // 添加折線圖與柱狀圖
            AddLineChart(wsDashboard, "每日登入趨勢", userStats, 3, 3 + userStats.Values.First().Count - 1, 1, 2);
            AddBarChart(wsDashboard, "每日權限變更統計", securityStats, 3, 3 + securityStats.Values.First().Count - 1, 1, 2);

            // 添加樞紐分析表
            AddPivotTable(wsDashboard, "DashboardPivot", startRow + 2);

            package.SaveAs(new FileInfo(reportPath));
        }

        Console.WriteLine($"高級 Dashboard 已生成：{reportPath}");
    }

    // JSON 解析：按使用者每日統計
    static Dictionary<string, Dictionary<DateTime, int>> AnalyzeJsonLogByUser(string filePath, string userField, string timeField)
    {
        var result = new Dictionary<string, Dictionary<DateTime, int>>();

        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                string user = obj[userField]?.ToString() ?? "Unknown";
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                if (!result.ContainsKey(user))
                    result[user] = new Dictionary<DateTime, int>();

                DateTime day = time.Date;
                if (result[user].ContainsKey(day))
                    result[user][day]++;
                else
                    result[user][day] = 1;
            }
            catch { }
        }

        // 按日期排序
        foreach (var user in result.Keys.ToList())
            result[user] = result[user].OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);

        return result;
    }

    // JSON 解析：單日事件統計
    static Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string messageField, string timeField)
    {
        var result = new Dictionary<DateTime, int>();
        if (!File.Exists(filePath)) return result;

        foreach (var line in File.ReadLines(filePath))
        {
            try
            {
                var obj = JObject.Parse(line);
                DateTime time = obj[timeField]?.ToObject<DateTime>() ?? DateTime.MinValue;
                if (time == DateTime.MinValue) continue;

                DateTime day = time.Date;
                if (result.ContainsKey(day))
                    result[day]++;
                else
                    result[day] = 1;
            }
            catch { }
        }

        return result.OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);
    }

    // 填充單一使用者統計表
    static int FillUserSection(ExcelWorksheet ws, string title, Dictionary<string, Dictionary<DateTime, int>> stats, int startRow)
    {
        ws.Cells[startRow, 1].Value = title;

        int row = startRow + 1;
        ws.Cells[row, 1].Value = "使用者";
        ws.Cells[row, 2].Value = "日期";
        ws.Cells[row, 3].Value = "次數";
        row++;

        foreach (var user in stats.Keys)
        {
            foreach (var kv in stats[user])
            {
                ws.Cells[row, 1].Value = user;
                ws.Cells[row, 2].Value = kv.Key.ToString("yyyy-MM-dd");
                ws.Cells[row, 3].Value = kv.Value;
                row++;
            }
        }

        ws.Cells[row, 2].Value = "總計";
        ws.Cells[row, 3].Formula = $"SUM(C{startRow + 3}:C{row - 1})";

        return row;
    }

    // 填充單日事件統計
    static int FillSection(ExcelWorksheet ws, string title, Dictionary<DateTime, int> stats, int startRow)
    {
        ws.Cells[startRow, 1].Value = title;
        ws.Cells[startRow + 1, 1].Value = "日期";
        ws.Cells[startRow + 1, 2].Value = "次數";

        int row = startRow + 2;
        foreach (var kv in stats)
        {
            ws.Cells[row, 1].Value = kv.Key.ToString("yyyy-MM-dd");
            ws.Cells[row, 2].Value = kv.Value;
            row++;
        }

        ws.Cells[row, 1].Value = "總計";
        ws.Cells[row, 2].Formula = $"SUM(B{startRow + 2}:B{row - 1})";

        return row;
    }

    // 添加折線圖
    static void AddLineChart(ExcelWorksheet ws, string title, Dictionary<string, Dictionary<DateTime, int>> stats, int startRow, int endRow, int colX, int colY)
    {
        var chart = ws.Drawings.AddChart(title, eChartType.Line) as ExcelLineChart;
        chart.Title.Text = title;
        chart.SetPosition(endRow + 2, 0, 3, 0);
        chart.SetSize(800, 300);

        // 將每個使用者作為一條線
        foreach (var user in stats.Keys)
        {
            var rows = stats[user].Keys.Select((date, index) => startRow + index).ToArray();
            chart.Series.Add(ws.Cells[startRow, colY, startRow + rows.Length - 1, colY], ws.Cells[startRow, colX, startRow + rows.Length - 1, colX]);
        }
    }

    // 添加柱狀圖
    static void AddBarChart(ExcelWorksheet ws, string title, Dictionary<string, Dictionary<DateTime, int>> stats, int startRow, int endRow, int colX, int colY)
    {
        var chart = ws.Drawings.AddChart(title, eChartType.ColumnClustered) as ExcelBarChart;
        chart.Title.Text = title;
        chart.SetPosition(endRow + 2, 0, 3, 0);
        chart.SetSize(800, 300);

        foreach (var user in stats.Keys)
        {
            var rows = stats[user].Keys.Select((date, index) => startRow + index).ToArray();
            chart.Series.Add(ws.Cells[startRow, colY, startRow + rows.Length - 1, colY], ws.Cells[startRow, colX, startRow + rows.Length - 1, colX]);
        }
    }

    // 添加樞紐分析表
    static void AddPivotTable(ExcelWorksheet ws, string pivotName, int startRow)
    {
        var pivotSheet = ws.Workbook.Worksheets.Add("PivotTable");
        var lastRow = startRow - 1;
        var dataRange = ws.Cells[2, 1, lastRow, 3];
        var pivotTable = pivotSheet.PivotTables.Add(pivotSheet.Cells[1, 1], dataRange, pivotName);
        pivotTable.RowFields.Add(pivotTable.Fields["使用者"]);
        pivotTable.ColumnFields.Add(pivotTable.Fields["日期"]);
        pivotTable.DataFields.Add(pivotTable.Fields["次數"]);
        pivotTable.ShowDataAs = OfficeOpenXml.Table.PivotTable.eDataFieldAggregateFunction.Sum;
    }
}
```

## 功能特色

- **使用者分組每日統計**：每位使用者每日事件數自動統計
- **多圖表 Dashboard**：折線圖 + 柱狀圖集中呈現趨勢
- **樞紐分析表整合**
