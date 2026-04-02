# 目錄

- [目錄](#目錄)
- [UserControl 實現 Dispose 的方式](#usercontrol-實現-dispose-的方式)

---

# UserControl 實現 Dispose 的方式

問題點：

- 因為 Windows Forms Designer 確實會覆蓋 Designer.cs 文件的內容，這意味著在 Designer.cs 修改的內容會在下次使用設計器時被覆蓋。
- Designer.cs 內已有 `protected override void Dispose(bool disposing) {}`，再繼承 IDispose 會有重覆命名的問題。

```csharp
public partial class Monitor : UserControl
{
    public Monitor()
    {
        InitializeComponent();
        this.Disposed += OnDisposed;
    }

    #region Dispose

    /// <summary>
    /// 當 UserControl 被 Dispose 時執行的清理邏輯
    /// </summary>
    private void OnDisposed(object sender, EventArgs e)
    {
        try
        {
            // 釋放資源
        }
        catch (Exception ex)
        {
            // 記錄清理錯誤但不拋出，避免影響正常的 Dispose 流程
            log.Information("...");
        }
    }

    #endregion Dispose
}
```

[🔝](#目錄)
