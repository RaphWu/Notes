---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# WinForm DataGridView 資料格式編輯教學

## 常用欄位格式設定

* 設定欄位顯示格式（Format 屬性）
  * 用於數字、日期、時間等資料的格式化顯示
  * 常見格式字串："N2"、"C2"、"yyyy/MM/dd"

```csharp
dataGridView1.Columns["Price"].DefaultCellStyle.Format = "N2";
dataGridView1.Columns["Date"].DefaultCellStyle.Format = "yyyy/MM/dd";
```

## 數字格式編輯

* 顯示小數位數
  * 例如顯示兩位小數

```csharp
dataGridView1.Columns["Amount"].DefaultCellStyle.Format = "N2";
```

* 顯示貨幣格式

```csharp
dataGridView1.Columns["Cost"].DefaultCellStyle.Format = "C2";
```

* 自訂數字格式
  * 例如補零或格式化顯示

```csharp
dataGridView1.Columns["Code"].DefaultCellStyle.Format = "0000";
```

## 日期與時間格式編輯

* 自訂日期格式

```csharp
dataGridView1.Columns["CreateDate"].DefaultCellStyle.Format = "yyyy-MM-dd";
```

* 顯示日期與時間

```csharp
dataGridView1.Columns["UpdateTime"].DefaultCellStyle.Format = "yyyy-MM-dd HH:mm:ss";
```

## Bool 資料顯示格式

* 使用 CheckBox 顯示

```csharp
DataGridViewCheckBoxColumn col = new DataGridViewCheckBoxColumn();
col.DataPropertyName = "IsActive";
col.HeaderText = "啟用";
dataGridView1.Columns.Add(col);
```

## 下拉選單 (ComboBox) 欄位

* 套用固定選項

```csharp
DataGridViewComboBoxColumn col = new DataGridViewComboBoxColumn();
col.HeaderText = "狀態";
col.DataPropertyName = "Status";
col.Items.AddRange("啟用", "停用", "暫停");
dataGridView1.Columns.Add(col);
```

* 綁定資料來源

```csharp
DataGridViewComboBoxColumn col = new DataGridViewComboBoxColumn();
col.HeaderText = "分類";
col.DataPropertyName = "CategoryId";
col.DataSource = categoryList;
col.DisplayMember = "Name";
col.ValueMember = "Id";
dataGridView1.Columns.Add(col);
```

## 自訂欄位輸入限制（Input Validation）

* 限制僅能輸入數字

```csharp
private void dataGridView1_EditingControlShowing(object sender, DataGridViewEditingControlShowingEventArgs e)
{
    TextBox tb = e.Control as TextBox;
    if (tb != null && dataGridView1.CurrentCell.ColumnIndex == 2)
    {
        tb.KeyPress -= OnlyNumber_KeyPress;
        tb.KeyPress += OnlyNumber_KeyPress;
    }
}

private void OnlyNumber_KeyPress(object sender, KeyPressEventArgs e)
{
    if (!char.IsDigit(e.KeyChar) && e.KeyChar != (char)Keys.Back)
    {
        e.Handled = true;
    }
}
```

## 使用 CellFormatting 進行動態格式化

* 根據條件改變顯示格式或顏色

```csharp
private void dataGridView1_CellFormatting(object sender, DataGridViewCellFormattingEventArgs e)
{
    if (dataGridView1.Columns[e.ColumnIndex].Name == "Status")
    {
        if ((int)e.Value == 1)
        {
            e.CellStyle.ForeColor = Color.Green;
            e.Value = "啟用";
        }
        else
        {
            e.CellStyle.ForeColor = Color.Red;
            e.Value = "停用";
        }
    }
}
```

## 自訂欄位型態（例如 Button 欄位）

* 加入按鈕欄位

```csharp
DataGridViewButtonColumn btn = new DataGridViewButtonColumn();
btn.HeaderText = "操作";
btn.Text = "編輯";
btn.UseColumnTextForButtonValue = true;
dataGridView1.Columns.Add(btn);
```

## 單元格輸入模式（MaskedTextBox）

* 日期或電話格式限定

```csharp
private void dataGridView1_EditingControlShowing(object sender, DataGridViewEditingControlShowingEventArgs e)
{
    if (dataGridView1.CurrentCell.OwningColumn.Name == "Phone")
    {
        MaskedTextBox mtb = new MaskedTextBox("0000-000-000");
        mtb.Text = e.Control.Text;
        e.Control = mtb;
    }
}
```

---

# 完整 DataGridView 範例（含各種欄位型態）

## 建立 DataTable 與資料列

```csharp
DataTable dt = new DataTable("ExampleTable");

// 字串欄位
dt.Columns.Add("Name", typeof(string));

// 整數欄位
dt.Columns.Add("Age", typeof(int));

// 浮點數欄位
dt.Columns.Add("Salary", typeof(decimal));

// 日期時間欄位
dt.Columns.Add("JoinDate", typeof(DateTime));

// 布林值欄位
dt.Columns.Add("IsActive", typeof(bool));

// 選單欄位用 ID
dt.Columns.Add("DepartmentId", typeof(int));

// GUID 欄位
dt.Columns.Add("Id", typeof(Guid));

// 新增資料列
dt.Rows.Add("王小明", 30, 45000.50m, DateTime.Now, true, 1, Guid.NewGuid());
dt.Rows.Add("李小華", 25, 38000.75m, DateTime.Parse("2025-01-01"), false, 2, Guid.NewGuid());
```

## 將 DataTable 綁定到 DataGridView

```csharp
dataGridView1.DataSource = dt;
```

## 自訂欄位型態

* ComboBox 欄位

```csharp
DataGridViewComboBoxColumn cmbDept = new DataGridViewComboBoxColumn();
cmbDept.HeaderText = "部門";
cmbDept.DataPropertyName = "DepartmentId";
cmbDept.DataSource = new[]
{
    new { Id = 1, Name = "人事部" },
    new { Id = 2, Name = "財務部" },
    new { Id = 3, Name = "研發部" }
};
cmbDept.ValueMember = "Id";
cmbDept.DisplayMember = "Name";
dataGridView1.Columns.Add(cmbDept);
```

* CheckBox 欄位（布林值）

```csharp
DataGridViewCheckBoxColumn chkActive = new DataGridViewCheckBoxColumn();
chkActive.HeaderText = "啟用";
chkActive.DataPropertyName = "IsActive";
dataGridView1.Columns.Add(chkActive);
```

* Button 欄位

```csharp
DataGridViewButtonColumn btnEdit = new DataGridViewButtonColumn();
btnEdit.HeaderText = "操作";
btnEdit.Text = "編輯";
btnEdit.UseColumnTextForButtonValue = true;
dataGridView1.Columns.Add(btnEdit);
```

## 設定格式

* 數字顯示兩位小數

```csharp
dataGridView1.Columns["Salary"].DefaultCellStyle.Format = "N2";
```

* 日期格式

```csharp
dataGridView1.Columns["JoinDate"].DefaultCellStyle.Format = "yyyy/MM/dd";
```

## 事件範例：按鈕點擊

```csharp
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    if (e.RowIndex >= 0 && dataGridView1.Columns[e.ColumnIndex] is DataGridViewButtonColumn)
    {
        string name = dataGridView1.Rows[e.RowIndex].Cells["Name"].Value.ToString();
        MessageBox.Show($"編輯 {name} 資料");
    }
}
```

這個範例結合了：

* DataTable 綁定
* 各種資料型別欄位
* ComboBox、CheckBox、Button 欄位
* 格式化顯示
  可以直接作為 WinForm DataGridView 教學或範例專案使用。

---

# DataGridView 輸入限制範例

## 限制欄位只能輸入數字

```csharp
private void dataGridView1_EditingControlShowing(object sender, DataGridViewEditingControlShowingEventArgs e)
{
    if (dataGridView1.CurrentCell.ColumnIndex == dataGridView1.Columns["Age"].Index)
    {
        TextBox tb = e.Control as TextBox;
        if (tb != null)
        {
            tb.KeyPress -= OnlyNumber_KeyPress;
            tb.KeyPress += OnlyNumber_KeyPress;
        }
    }
}

private void OnlyNumber_KeyPress(object sender, KeyPressEventArgs e)
{
    if (!char.IsDigit(e.KeyChar) && e.KeyChar != (char)Keys.Back)
    {
        e.Handled = true;
    }
}
```

## 限制欄位只能輸入小數

```csharp
private void dataGridView1_EditingControlShowing_Decimal(object sender, DataGridViewEditingControlShowingEventArgs e)
{
    if (dataGridView1.CurrentCell.ColumnIndex == dataGridView1.Columns["Salary"].Index)
    {
        TextBox tb = e.Control as TextBox;
        if (tb != null)
        {
            tb.KeyPress -= OnlyDecimal_KeyPress;
            tb.KeyPress += OnlyDecimal_KeyPress;
        }
    }
}

private void OnlyDecimal_KeyPress(object sender, KeyPressEventArgs e)
{
    TextBox tb = sender as TextBox;
    if (!char.IsDigit(e.KeyChar) && e.KeyChar != (char)Keys.Back && e.KeyChar != '.')
    {
        e.Handled = true;
    }
    if (e.KeyChar == '.' && tb.Text.Contains("."))
    {
        e.Handled = true;
    }
}
```

## 限制日期欄位輸入格式

* 使用 MaskedTextBox 或 DateTimePicker

```csharp
private void dataGridView1_EditingControlShowing_Date(object sender, DataGridViewEditingControlShowingEventArgs e)
{
    if (dataGridView1.CurrentCell.ColumnIndex == dataGridView1.Columns["JoinDate"].Index)
    {
        DateTimePicker dtp = new DateTimePicker();
        dtp.Format = DateTimePickerFormat.Custom;
        dtp.CustomFormat = "yyyy/MM/dd";
        dtp.ValueChanged += (s, ev) =>
        {
            dataGridView1.CurrentCell.Value = dtp.Value;
        };
        e.Control = dtp;
    }
}
```

## 其他提示

* 可針對不同欄位使用不同事件處理
* 建議結合 `CellValidating` 做資料驗證，避免錯誤資料輸入

```csharp
private void dataGridView1_CellValidating(object sender, DataGridViewCellValidatingEventArgs e)
{
    if (dataGridView1.Columns[e.ColumnIndex].Name == "Age")
    {
        if (!int.TryParse(e.FormattedValue.ToString(), out int val))
        {
            MessageBox.Show("請輸入有效數字");
            e.Cancel = true;
        }
    }
}
```

這樣就可以讓 DataGridView 的各種欄位都有**格式化顯示**和**輸入限制**，整個編輯體驗更完整。
