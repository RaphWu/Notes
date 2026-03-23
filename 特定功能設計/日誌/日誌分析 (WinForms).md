---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 日誌分析結果直接顯示在螢幕上，即時 Dashboard

## 1. 設計概念

- **WinForms 表單**作為 Dashboard
- **DataGridView** 顯示每日事件統計與使用者分組統計
- **Chart 控制項**顯示折線圖或柱狀圖（登入趨勢、錯誤趨勢、權限變更趨勢）
- **按鈕 / 計時器**：可手動刷新或定時自動刷新日誌分析
- **JSON 日誌解析**：與之前 EPPlus 分析邏輯一致

## 2. WinForms 控制項

- **DataGridView** × 3（UserActivity、ErrorLog、SecurityEvent）
- **Chart** × 3（折線圖 / 柱狀圖）
- **Button** × 1（刷新分析結果）
- **Timer**（可選，用於定時刷新）

## 3. 範例程式碼（C# WinForms）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Windows.Forms;
using Newtonsoft.Json.Linq;
using System.Windows.Forms.DataVisualization.Charting;

public class LogDashboardForm : Form
{
    private DataGridView dgvUser, dgvError, dgvSecurity;
    private Chart chartUser, chartError, chartSecurity;
    private Button btnRefresh;

    private string basePath = @"C:\MyAppLogs";

    public LogDashboardForm()
    {
        this.Text = "日誌即時 Dashboard";
        this.Width = 1200;
        this.Height = 800;

        // 初始化 DataGridView
        dgvUser = new DataGridView { Width = 350, Height = 200, Top = 10, Left = 10 };
        dgvError = new DataGridView { Width = 350, Height = 200, Top = 220, Left = 10 };
        dgvSecurity = new DataGridView { Width = 350, Height = 200, Top = 430, Left = 10 };

        // 初始化 Chart
        chartUser = new Chart { Width = 750, Height = 200, Top = 10, Left = 370 };
        chartError = new Chart { Width = 750, Height = 200, Top = 220, Left = 370 };
        chartSecurity = new Chart { Width = 750, Height = 200, Top = 430, Left = 370 };

        chartUser.ChartAreas.Add(new ChartArea("UserArea"));
        chartError.ChartAreas.Add(new ChartArea("ErrorArea"));
        chartSecurity.ChartAreas.Add(new ChartArea("SecurityArea"));

        // 初始化按鈕
        btnRefresh = new Button { Text = "刷新", Width = 100, Height = 30, Top = 640, Left = 10 };
        btnRefresh.Click += BtnRefresh_Click;

        this.Controls.Add(dgvUser);
        this.Controls.Add(dgvError);
        this.Controls.Add(dgvSecurity);
        this.Controls.Add(chartUser);
        this.Controls.Add(chartError);
        this.Controls.Add(chartSecurity);
        this.Controls.Add(btnRefresh);

        LoadDashboard();
    }

    private void BtnRefresh_Click(object sender, EventArgs e)
    {
        LoadDashboard();
    }

    private void LoadDashboard()
    {
        // 清除舊資料
        dgvUser.Rows.Clear(); dgvUser.Columns.Clear();
        dgvError.Rows.Clear(); dgvError.Columns.Clear();
        dgvSecurity.Rows.Clear(); dgvSecurity.Columns.Clear();

        chartUser.Series.Clear();
        chartError.Series.Clear();
        chartSecurity.Series.Clear();

        // 讀取 JSON
        var userStats = AnalyzeJsonLog(basePath + @"\UserActivity\user_activity.json", "User", "Time");
        var errorStats = AnalyzeJsonLog(basePath + @"\Error\error_log_.txt", "Message", "Timestamp");
        var securityStats = AnalyzeJsonLog(basePath + @"\Security\security_event.json", "Admin", "Time");

        // DataGridView 顯示
        FillDataGridView(dgvUser, userStats);
        FillDataGridView(dgvError, errorStats);
        FillDataGridView(dgvSecurity, securityStats);

        // Chart 顯示
        FillChart(chartUser, "每日登入趨勢", userStats);
        FillChart(chartError, "每日錯誤趨勢", errorStats);
        FillChart(chartSecurity, "每日權限變更趨勢", securityStats);
    }

    private Dictionary<DateTime, int> AnalyzeJsonLog(string filePath, string userField, string timeField)
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

    private void FillDataGridView(DataGridView dgv, Dictionary<DateTime, int> stats)
    {
        dgv.Columns.Add("Date", "日期");
        dgv.Columns.Add("Count", "次數");

        foreach (var kv in stats)
        {
            dgv.Rows.Add(kv.Key.ToString("yyyy-MM-dd"), kv.Value);
        }
    }

    private void FillChart(Chart chart, string title, Dictionary<DateTime, int> stats)
    {
        var series = chart.Series.Add(title);
        series.ChartType = SeriesChartType.Line;
        foreach (var kv in stats)
        {
            series.Points.AddXY(kv.Key.ToString("yyyy-MM-dd"), kv.Value);
        }
        chart.ChartAreas[0].AxisX.Interval = 1;
        chart.ChartAreas[0].AxisX.Title = "日期";
        chart.ChartAreas[0].AxisY.Title = "次數";
    }

    [STAThread]
    public static void Main()
    {
        Application.EnableVisualStyles();
        Application.Run(new LogDashboardForm());
    }
}
```

### 4. 功能特色

- **即時可視化**：開啟程式就能看到最新日誌統計
- **DataGridView + Chart**：表格與圖表同步展示
- **刷新按鈕**：可手動更新日誌分析
- **可擴展**：可加入 Timer 自動刷新，或分組統計特定使用者

---

# WinForms 即時 Dashboard 升級版：多使用者分組 + 動態篩選

這個版本的目標是 **完整即時監控日誌事件**，功能包括：

- **多使用者分組折線圖**：每位使用者每日登入/操作趨勢
- **權限變更柱狀圖**：每日權限變更事件統計
- **動態篩選器**：可選擇使用者或日期範圍
- **DataGridView 顯示每日統計**：同步表格與圖表
- **手動刷新 + 自動刷新**（Timer 控制）

## 1. WinForms 控制項設計

- **ComboBox**：選擇特定使用者
- **DateTimePicker** ×2：選擇開始與結束日期
- **DataGridView**：顯示統計表格
- **Chart** ×2：折線圖（登入趨勢）、柱狀圖（權限變更）
- **Button**：刷新
- **Timer**：自動刷新

## 2. 範例程式碼（C# WinForms）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Windows.Forms;
using Newtonsoft.Json.Linq;
using System.Windows.Forms.DataVisualization.Charting;

public class AdvancedLogDashboard : Form
{
    private ComboBox cmbUsers;
    private DateTimePicker dtStart, dtEnd;
    private Button btnRefresh;
    private DataGridView dgvStats;
    private Chart chartLine, chartBar;
    private Timer autoRefreshTimer;
    private string basePath = @"C:\MyAppLogs";

    private Dictionary<string, Dictionary<DateTime, int>> userStats;
    private Dictionary<string, Dictionary<DateTime, int>> securityStats;

    public AdvancedLogDashboard()
    {
        this.Text = "高級日誌 Dashboard";
        this.Width = 1200; this.Height = 800;

        // 篩選器
        cmbUsers = new ComboBox { Top = 10, Left = 10, Width = 200 };
        dtStart = new DateTimePicker { Top = 10, Left = 220, Width = 120 };
        dtEnd = new DateTimePicker { Top = 10, Left = 350, Width = 120 };
        btnRefresh = new Button { Text = "刷新", Top = 10, Left = 480, Width = 100 };
        btnRefresh.Click += BtnRefresh_Click;

        // DataGridView
        dgvStats = new DataGridView { Top = 50, Left = 10, Width = 350, Height = 700 };

        // Charts
        chartLine = new Chart { Top = 50, Left = 370, Width = 750, Height = 300 };
        chartBar = new Chart { Top = 380, Left = 370, Width = 750, Height = 370 };
        chartLine.ChartAreas.Add(new ChartArea("LineArea"));
        chartBar.ChartAreas.Add(new ChartArea("BarArea"));

        this.Controls.AddRange(new Control[] { cmbUsers, dtStart, dtEnd, btnRefresh, dgvStats, chartLine, chartBar });

        // Timer 自動刷新
        autoRefreshTimer = new Timer();
        autoRefreshTimer.Interval = 5 * 60 * 1000; // 每5分鐘刷新
        autoRefreshTimer.Tick += (s, e) => LoadDashboard();
        autoRefreshTimer.Start();

        LoadDashboard();
    }

    private void BtnRefresh_Click(object sender, EventArgs e) => LoadDashboard();

    private void LoadDashboard()
    {
        // 讀取 JSON 日誌
        userStats = AnalyzeJsonLogByUser(basePath + @"\UserActivity\user_activity.json", "User", "Time");
        securityStats = AnalyzeJsonLogByUser(basePath + @"\Security\security_event.json", "Admin", "Time");

        // 更新使用者選單
        cmbUsers.Items.Clear();
        cmbUsers.Items.AddRange(userStats.Keys.ToArray());
        if (cmbUsers.Items.Count > 0) cmbUsers.SelectedIndex = 0;

        UpdateDashboard();
    }

    private void UpdateDashboard()
    {
        dgvStats.Rows.Clear(); dgvStats.Columns.Clear();
        chartLine.Series.Clear(); chartBar.Series.Clear();

        string selectedUser = cmbUsers.SelectedItem?.ToString();
        DateTime start = dtStart.Value.Date;
        DateTime end = dtEnd.Value.Date;

        // DataGridView
        dgvStats.Columns.Add("Date", "日期");
        dgvStats.Columns.Add("Count", "次數");

        if (!string.IsNullOrEmpty(selectedUser) && userStats.ContainsKey(selectedUser))
        {
            foreach (var kv in userStats[selectedUser].Where(d => d.Key >= start && d.Key <= end))
            {
                dgvStats.Rows.Add(kv.Key.ToString("yyyy-MM-dd"), kv.Value);
            }

            // 折線圖
            var seriesLine = chartLine.Series.Add(selectedUser + "登入趨勢");
            seriesLine.ChartType = SeriesChartType.Line;
            foreach (var kv in userStats[selectedUser].Where(d => d.Key >= start && d.Key <= end))
                seriesLine.Points.AddXY(kv.Key.ToString("MM-dd"), kv.Value);
        }

        // 權限變更柱狀圖
        foreach (var user in securityStats.Keys)
        {
            var seriesBar = chartBar.Series.Add(user);
            seriesBar.ChartType = SeriesChartType.Column;
            foreach (var kv in securityStats[user].Where(d => d.Key >= start && d.Key <= end))
                seriesBar.Points.AddXY(kv.Key.ToString("MM-dd"), kv.Value);
        }

        chartLine.ChartAreas[0].AxisX.Title = "日期";
        chartLine.ChartAreas[0].AxisY.Title = "次數";
        chartBar.ChartAreas[0].AxisX.Title = "日期";
        chartBar.ChartAreas[0].AxisY.Title = "次數";
    }

    private Dictionary<string, Dictionary<DateTime, int>> AnalyzeJsonLogByUser(string filePath, string userField, string timeField)
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

                if (!result.ContainsKey(user)) result[user] = new Dictionary<DateTime, int>();
                DateTime day = time.Date;
                if (result[user].ContainsKey(day)) result[user][day]++;
                else result[user][day] = 1;
            }
            catch { }
        }

        foreach (var user in result.Keys.ToList())
            result[user] = result[user].OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);

        return result;
    }

    [STAThread]
    public static void Main()
    {
        Application.EnableVisualStyles();
        Application.Run(new AdvancedLogDashboard());
    }
}
```

## 3. 功能特色

- **多使用者分組折線圖**：每位使用者每日登入趨勢
- **權限變更柱狀圖**：顯示每日高敏感事件統計
- **動態篩選**：選擇使用者、開始/結束日期
- **DataGridView 表格**：與圖表同步
- **自動刷新**：Timer 每 5 分鐘更新，可手動刷新
- **即時監控**：不需生成 Excel，即可在螢幕上看到 Dashboard

這個 WinForms 版本就是真正的 **螢幕即時 Dashboard**，完全可以用來監控每日登入、錯誤、權限變更趨勢，並支援特定使用者分析。

---

# WinForms 高階 Dashboard：多使用者對比折線圖 + 柱狀圖

這個版本在上一個即時 Dashboard 的基礎上，增加了 **多使用者趨勢對比**：

- **折線圖**：多使用者每日登入趨勢對比
- **柱狀圖**：多使用者每日權限變更事件對比
- **DataGridView**：同步顯示選擇日期範圍的使用者統計
- **動態篩選器**：可選擇日期範圍，也可切換部分使用者
- **自動刷新 + 手動刷新**

## 1. WinForms 控制項設計

- **CheckedListBox**：選擇多個使用者做圖表對比
- **DateTimePicker** ×2：選擇開始與結束日期
- **DataGridView**：顯示統計表格
- **Chart** ×2：折線圖（登入趨勢）、柱狀圖（權限變更）
- **Button**：刷新
- **Timer**：自動刷新

## 2. 範例程式碼（C# WinForms）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Windows.Forms;
using Newtonsoft.Json.Linq;
using System.Windows.Forms.DataVisualization.Charting;

public class MultiUserDashboard : Form
{
    private CheckedListBox clbUsers;
    private DateTimePicker dtStart, dtEnd;
    private Button btnRefresh;
    private DataGridView dgvStats;
    private Chart chartLine, chartBar;
    private Timer autoRefreshTimer;
    private string basePath = @"C:\MyAppLogs";

    private Dictionary<string, Dictionary<DateTime, int>> userStats;
    private Dictionary<string, Dictionary<DateTime, int>> securityStats;

    public MultiUserDashboard()
    {
        this.Text = "多使用者日誌 Dashboard";
        this.Width = 1200; this.Height = 850;

        // 篩選器
        clbUsers = new CheckedListBox { Top = 10, Left = 10, Width = 200, Height = 150 };
        dtStart = new DateTimePicker { Top = 10, Left = 220, Width = 120 };
        dtEnd = new DateTimePicker { Top = 10, Left = 350, Width = 120 };
        btnRefresh = new Button { Text = "刷新", Top = 10, Left = 480, Width = 100 };
        btnRefresh.Click += BtnRefresh_Click;

        // DataGridView
        dgvStats = new DataGridView { Top = 170, Left = 10, Width = 350, Height = 650 };

        // Charts
        chartLine = new Chart { Top = 170, Left = 370, Width = 750, Height = 300 };
        chartBar = new Chart { Top = 480, Left = 370, Width = 750, Height = 340 };
        chartLine.ChartAreas.Add(new ChartArea("LineArea"));
        chartBar.ChartAreas.Add(new ChartArea("BarArea"));

        this.Controls.AddRange(new Control[] { clbUsers, dtStart, dtEnd, btnRefresh, dgvStats, chartLine, chartBar });

        // Timer 自動刷新
        autoRefreshTimer = new Timer();
        autoRefreshTimer.Interval = 5 * 60 * 1000; // 每5分鐘刷新
        autoRefreshTimer.Tick += (s, e) => LoadDashboard();
        autoRefreshTimer.Start();

        LoadDashboard();
    }

    private void BtnRefresh_Click(object sender, EventArgs e) => UpdateDashboard();

    private void LoadDashboard()
    {
        userStats = AnalyzeJsonLogByUser(basePath + @"\UserActivity\user_activity.json", "User", "Time");
        securityStats = AnalyzeJsonLogByUser(basePath + @"\Security\security_event.json", "Admin", "Time");

        clbUsers.Items.Clear();
        clbUsers.Items.AddRange(userStats.Keys.ToArray());
        for (int i = 0; i < clbUsers.Items.Count; i++) clbUsers.SetItemChecked(i, true);

        UpdateDashboard();
    }

    private void UpdateDashboard()
    {
        dgvStats.Rows.Clear(); dgvStats.Columns.Clear();
        chartLine.Series.Clear(); chartBar.Series.Clear();

        var selectedUsers = clbUsers.CheckedItems.Cast<string>().ToList();
        DateTime start = dtStart.Value.Date;
        DateTime end = dtEnd.Value.Date;

        // DataGridView 顯示多使用者統計
        dgvStats.Columns.Add("User", "使用者");
        dgvStats.Columns.Add("Date", "日期");
        dgvStats.Columns.Add("Count", "次數");

        foreach (var user in selectedUsers)
        {
            if (!userStats.ContainsKey(user)) continue;
            foreach (var kv in userStats[user].Where(d => d.Key >= start && d.Key <= end))
            {
                dgvStats.Rows.Add(user, kv.Key.ToString("yyyy-MM-dd"), kv.Value);
            }
        }

        // 折線圖：多使用者登入對比
        foreach (var user in selectedUsers)
        {
            if (!userStats.ContainsKey(user)) continue;
            var series = chartLine.Series.Add(user);
            series.ChartType = SeriesChartType.Line;
            foreach (var kv in userStats[user].Where(d => d.Key >= start && d.Key <= end))
                series.Points.AddXY(kv.Key.ToString("MM-dd"), kv.Value);
        }
        chartLine.ChartAreas[0].AxisX.Title = "日期";
        chartLine.ChartAreas[0].AxisY.Title = "登入次數";

        // 柱狀圖：多使用者權限變更對比
        foreach (var user in selectedUsers)
        {
            if (!securityStats.ContainsKey(user)) continue;
            var series = chartBar.Series.Add(user);
            series.ChartType = SeriesChartType.Column;
            foreach (var kv in securityStats[user].Where(d => d.Key >= start && d.Key <= end))
                series.Points.AddXY(kv.Key.ToString("MM-dd"), kv.Value);
        }
        chartBar.ChartAreas[0].AxisX.Title = "日期";
        chartBar.ChartAreas[0].AxisY.Title = "權限變更次數";
    }

    private Dictionary<string, Dictionary<DateTime, int>> AnalyzeJsonLogByUser(string filePath, string userField, string timeField)
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

                if (!result.ContainsKey(user)) result[user] = new Dictionary<DateTime, int>();
                DateTime day = time.Date;
                if (result[user].ContainsKey(day)) result[user][day]++;
                else result[user][day] = 1;
            }
            catch { }
        }

        foreach (var user in result.Keys.ToList())
            result[user] = result[user].OrderBy(kv => kv.Key).ToDictionary(kv => kv.Key, kv => kv.Value);

        return result;
    }

    [STAThread]
    public static void Main()
    {
        Application.EnableVisualStyles();
        Application.Run(new MultiUserDashboard());
    }
}
```

## 3. 功能特色

- **多使用者折線圖**：可同時對比多個使用者每日登入趨勢
- **多使用者柱狀圖**：每日權限變更對比
- **DataGridView**：表格與圖表同步，方便檢視細節
- **動態篩選器**：選擇多位使用者及日期範圍
- **自動刷新 + 手動刷新**：保持 Dashboard 最新
- **即時監控**：無需生成 Excel，螢幕上直接呈現完整趨勢

這個版本就是真正的 **WinForms BI 類 Dashboard**，可即時監控每日登入、錯誤、權限變更，支援多使用者對比與篩選，非常適合管理者或運維使用。
