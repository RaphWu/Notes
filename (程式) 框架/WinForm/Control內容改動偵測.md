---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

示範如何追蹤多個控制項（TextBox、ComboBox、CheckBox）的變更狀態，並用一個 `isModified` flag 來判斷整個表單是否有被改過。

```csharp
public partial class Form1 : Form
{
    private bool isModified = false;

    // 用來記錄初始化時的原始值
    private Dictionary<Control, object> originalValues = new Dictionary<Control, object>();

    public Form1()
    {
        InitializeComponent();

        // 載入完成後初始化追蹤
        this.Load += Form1_Load;
    }

    private void Form1_Load(object sender, EventArgs e)
    {
        // 假設有 textBox1, comboBox1, checkBox1

        // 記錄初始值
        originalValues[textBox1] = textBox1.Text;
        originalValues[comboBox1] = comboBox1.Text; // 如果是 SelectedValue，用 comboBox1.SelectedValue
        originalValues[checkBox1] = checkBox1.Checked;

        // 訂閱事件，當控制項有變動時設 isModified = true
        textBox1.TextChanged += Control_ValueChanged;
        comboBox1.SelectedIndexChanged += Control_ValueChanged;
        checkBox1.CheckedChanged += Control_ValueChanged;

        // 重設修改旗標
        isModified = false;
    }

    private void Control_ValueChanged(object sender, EventArgs e)
    {
        if (sender is Control ctrl && originalValues.ContainsKey(ctrl))
        {
            object originalValue = originalValues[ctrl];
            object currentValue = null;

            if (ctrl is TextBox tb)
                currentValue = tb.Text;
            else if (ctrl is ComboBox cb)
                currentValue = cb.Text; // 或 cb.SelectedValue
            else if (ctrl is CheckBox chk)
                currentValue = chk.Checked;

            if (!object.Equals(originalValue, currentValue))
            {
                isModified = true;
            }
            else
            {
                // 可能有多個控制項，這裡可檢查所有控制項是否都未改動
                isModified = originalValues.Any(kvp =>
                {
                    var c = kvp.Key;
                    var ov = kvp.Value;
                    object cv = null;
                    if (c is TextBox t) cv = t.Text;
                    else if (c is ComboBox cbox) cv = cbox.Text;
                    else if (c is CheckBox ch) cv = ch.Checked;
                    return !object.Equals(ov, cv);
                });
            }

            // 可在這裡更新 UI，比如啟用/禁用 儲存按鈕
            // btnSave.Enabled = isModified;
        }
    }

    private void btnSave_Click(object sender, EventArgs e)
    {
        if (isModified)
        {
            // 儲存邏輯
            MessageBox.Show("資料已儲存。");

            // 儲存成功後重置修改旗標和原始值
            foreach (var ctrl in originalValues.Keys.ToList())
            {
                if (ctrl is TextBox tb)
                    originalValues[ctrl] = tb.Text;
                else if (ctrl is ComboBox cb)
                    originalValues[ctrl] = cb.Text;
                else if (ctrl is CheckBox chk)
                    originalValues[ctrl] = chk.Checked;
            }

            isModified = false;
            // btnSave.Enabled = false;
        }
        else
        {
            MessageBox.Show("資料未修改，無需儲存。");
        }
    }
}
```

# 這個範例說明

* `originalValues` 字典紀錄每個控制項在表單載入時的原始值。
* 每當控制項內容改變（TextChanged、SelectedIndexChanged、CheckedChanged）時，都會觸發 `Control_ValueChanged`。
* `Control_ValueChanged` 會判斷目前值是否與原始值不同，如果有任一控制項不同，`isModified` 設為 `true`，否則檢查所有控制項是否都沒改動，設回 `false`。
* 儲存按鈕 `btnSave` 在 `Click` 事件中會檢查 `isModified`，有改動才執行儲存，並重置原始值。

# WinForms 表單資料變更偵測的完整範例

包含常見的幾種控制項（TextBox、ComboBox、CheckBox、DateTimePicker、NumericUpDown），並且設計成易擴充的架構，讓你只要新增控制項時，簡單加入追蹤就好。

# WinForms 控制項變更偵測完整範例

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;

public partial class Form1 : Form
{
    // 紀錄原始值
    private Dictionary<Control, object> originalValues = new Dictionary<Control, object>();

    // 表示表單是否有修改
    private bool isModified = false;

    public Form1()
    {
        InitializeComponent();

        this.Load += Form1_Load;
    }

    private void Form1_Load(object sender, EventArgs e)
    {
        // 加入所有你想監控的控制項
        TrackControl(textBox1);
        TrackControl(comboBox1);
        TrackControl(checkBox1);
        TrackControl(dateTimePicker1);
        TrackControl(numericUpDown1);

        // 初始時無修改
        isModified = false;
        UpdateSaveButton();
    }

    /// <summary>
    /// 登記並記錄控制項的原始值，並訂閱相應事件
    /// </summary>
    private void TrackControl(Control ctrl)
    {
        object value = GetControlValue(ctrl);
        originalValues[ctrl] = value;

        // 根據控制項類型訂閱事件
        if (ctrl is TextBox tb)
        {
            tb.TextChanged += Control_ValueChanged;
        }
        else if (ctrl is ComboBox cb)
        {
            cb.SelectedIndexChanged += Control_ValueChanged;
        }
        else if (ctrl is CheckBox chk)
        {
            chk.CheckedChanged += Control_ValueChanged;
        }
        else if (ctrl is DateTimePicker dtp)
        {
            dtp.ValueChanged += Control_ValueChanged;
        }
        else if (ctrl is NumericUpDown nud)
        {
            nud.ValueChanged += Control_ValueChanged;
        }
        else
        {
            // 其他控制項，如有需要可在此擴充
            throw new NotSupportedException($"控制項類型 {ctrl.GetType().Name} 尚未支援");
        }
    }

    /// <summary>
    /// 根據控制項類型取得其值
    /// </summary>
    private object GetControlValue(Control ctrl)
    {
        if (ctrl is TextBox tb)
            return tb.Text;
        if (ctrl is ComboBox cb)
            return cb.SelectedIndex >= 0 ? cb.SelectedValue ?? cb.Text : null;
        if (ctrl is CheckBox chk)
            return chk.Checked;
        if (ctrl is DateTimePicker dtp)
            return dtp.Value;
        if (ctrl is NumericUpDown nud)
            return nud.Value;

        return null;
    }

    /// <summary>
    /// 控制項值改變時觸發
    /// </summary>
    private void Control_ValueChanged(object sender, EventArgs e)
    {
        if (sender is Control ctrl && originalValues.ContainsKey(ctrl))
        {
            var originalValue = originalValues[ctrl];
            var currentValue = GetControlValue(ctrl);

            if (!object.Equals(originalValue, currentValue))
            {
                isModified = true;
            }
            else
            {
                // 檢查所有控制項是否都未改動，若是，設 isModified = false
                isModified = originalValues.Any(kvp =>
                {
                    var c = kvp.Key;
                    var ov = kvp.Value;
                    var cv = GetControlValue(c);
                    return !object.Equals(ov, cv);
                });
            }

            UpdateSaveButton();
        }
    }

    /// <summary>
    /// 更新儲存按鈕狀態（例如啟用或禁用）
    /// </summary>
    private void UpdateSaveButton()
    {
        btnSave.Enabled = isModified;
    }

    private void btnSave_Click(object sender, EventArgs e)
    {
        if (isModified)
        {
            // 執行儲存邏輯
            MessageBox.Show("資料已儲存。");

            // 儲存後重置原始值
            foreach (var ctrl in originalValues.Keys.ToList())
            {
                originalValues[ctrl] = GetControlValue(ctrl);
            }

            isModified = false;
            UpdateSaveButton();
        }
        else
        {
            MessageBox.Show("資料未修改，無需儲存。");
        }
    }

    private void btnCancel_Click(object sender, EventArgs e)
    {
        if (isModified)
        {
            var result = MessageBox.Show("資料已修改，確定要放棄嗎？", "確認", MessageBoxButtons.YesNo);
            if (result == DialogResult.Yes)
            {
                // 還原所有控制項值到原始值
                foreach (var kvp in originalValues)
                {
                    SetControlValue(kvp.Key, kvp.Value);
                }

                isModified = false;
                UpdateSaveButton();
            }
        }
        else
        {
            this.Close();
        }
    }

    /// <summary>
    /// 設定控制項的值
    /// </summary>
    private void SetControlValue(Control ctrl, object value)
    {
        if (ctrl is TextBox tb)
            tb.Text = value?.ToString() ?? "";
        else if (ctrl is ComboBox cb)
        {
            if (value != null && cb.Items.Contains(value))
                cb.SelectedItem = value;
            else
                cb.Text = value?.ToString() ?? "";
        }
        else if (ctrl is CheckBox chk)
        {
            if (value is bool b) chk.Checked = b;
        }
        else if (ctrl is DateTimePicker dtp)
        {
            if (value is DateTime dt) dtp.Value = dt;
        }
        else if (ctrl is NumericUpDown nud)
        {
            if (value is decimal dec) nud.Value = dec;
        }
    }
}
```

# 重點說明

1. **原始值紀錄**：一開始就紀錄所有受監控控制項的初始值，方便做比較和還原。
2. **事件訂閱**：根據控制項類型，訂閱對應的變更事件。
3. **變更偵測**：每當某個控制項值改變，跟原始值比對，若有差異，就設定 `isModified = true`，沒有差異則檢查其他控制項。
4. **還原功能**：新增取消按鈕的行為，可以還原表單控制項至原始值。
5. **擴充方便**：只要控制項類型有支援，呼叫 `TrackControl()` 加入即可，若遇到不支援的類型可擴充 `TrackControl` 和 `GetControlValue`。

## 這樣你以後做表單，只要

* 新增控制項後
* 在 `Form_Load` 依序呼叫 `TrackControl(你的控制項)`

就能自動監控資料變動，還能控制「儲存按鈕」狀態，以及做還原。

如果要我幫你改成自訂 UserControl 或是更大型專案的變更追蹤架構，也可以告訴我！
這樣的功能是 WinForms 裡常見且實用的技巧，祝你開發順利～ 😎👍
