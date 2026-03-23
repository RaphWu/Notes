---
aliases:
date:
update:
author:
language:
sourceurl: https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/observableobject
tags:
---

在 **CommunityToolkit.Mvvm** 中，這三個核心基底類別各有用途，也可以搭配使用，但不是每個都必須同時用。整理如下：

# 1. ObservableObject

- **用途**：最基本的資料綁定支援
- **功能**：
    - 實作 `INotifyPropertyChanged`
    - 提供 `[ObservableProperty]` 屬性自動生成通知
- **使用場合**：
    - 單純想讓 **屬性變化能自動更新 UI**
    - 不需要 Messenger 或驗證功能
- **範例**：

```csharp
public partial class PersonViewModel : ObservableObject
{
    [ObservableProperty]
    private string name;
}
```

# 2. ObservableRecipient

- **用途**：帶有 **Messenger 功能的 ObservableObject**
- **功能**：
    - 繼承自 `ObservableObject`
    - 可以訂閱、接收訊息（Messenger）
    - 可控制 `IsActive` 屬性：當 `IsActive = false` 時，不接收訊息，避免多餘更新
- **使用場合**：
    - ViewModel 需要與 **其他 ViewModel 或服務透過 Messenger 通訊**
    - 例如跨 ViewModel 更新資料、通知 UI 更新
- **範例**：

```csharp
public partial class PersonViewModel : ObservableRecipient
{
    [ObservableProperty]
    private string name;

    public PersonViewModel()
    {
        IsActive = true; // 啟用訊息接收
        WeakReferenceMessenger.Default.Register<UpdateNameMessage>(this, (r, m) =>
        {
            Name = m.NewName;
        });
    }
}
```

- 這時 `ObservableRecipient` 提供了 `IsActive` 控制訊息接收的機制，比單純 `ObservableObject` 更適合有 Messenger 的場景。

# 3. ObservableValidator

- **用途**：在 `ObservableObject` 基礎上增加 **資料驗證功能**
- **功能**：
    - 實作 `INotifyDataErrorInfo`，方便 UI 自動顯示驗證錯誤
    - 可以在屬性設定時檢查規則
    - 可搭配 `[ObservableProperty]` 使用
- **使用場合**：
    - ViewModel 需要 **資料驗證**，例如表單輸入、用戶設定
    - UI 透過 WinForms 的 `ErrorProvider` 或 WPF 的驗證機制自動顯示錯誤
- **範例**：

```csharp
public partial class PersonViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "姓名不可空白")]
    private string name;
    
    public void ValidateAllProperties() => ValidateAllProperties();
}
```

# 搭配方式

|組合|適用情境|
|---|---|
|`ObservableObject`|單純 Data Binding，屬性變化更新 UI|
|`ObservableRecipient`|需要 Messenger 功能的 ViewModel，跨 ViewModel 或服務通信|
|`ObservableValidator`|需要屬性驗證，UI 顯示錯誤訊息|
|`ObservableRecipient + ObservableValidator`|同時需要 Messenger + 屬性驗證，例如大型表單，且表單資料會跨 ViewModel 傳遞更新|

> 注意：`ObservableRecipient` 與 `ObservableValidator` 都繼承自 `ObservableObject`，所以你不用同時繼承三個。若要同時有 Messenger + 驗證功能，可以自己建立一個中間類別繼承自 `ObservableValidator` 並實作 Messenger 訂閱（或自己封裝）。

![[WinForms ViewModel 使用場景圖表.png]]

---

# 建立一個中間類別繼承自 ObservableValidator 並實作 Messenger 訂閱的封裝範例

想 **同時有 Messenger + 驗證功能**，而 CommunityToolkit.Mvvm 沒有提供一個現成的類別繼承 `ObservableValidator` 並直接整合 Messenger。
這是一個 **中間基底類別** 封裝範例，讓 ViewModel 可以直接繼承使用。

## 中間類別範例

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Messaging;

public abstract class ObservableValidatorRecipient : ObservableValidator
{
    /// <summary>
    /// 控制是否啟用 Messenger 訂閱
    /// </summary>
    public bool IsActive { get; set; } = true;

    protected ObservableValidatorRecipient()
    {
        // 預設啟用 Messenger
        InitializeMessenger();
    }

    /// <summary>
    /// 初始化 Messenger 訂閱
    /// </summary>
    protected virtual void InitializeMessenger()
    {
        // 子類別可在此覆寫，訂閱需要的訊息
    }

    /// <summary>
    /// 訂閱訊息的 Helper 方法
    /// </summary>
    protected void RegisterMessage<TMessage>(Action<TMessage> action)
    {
        WeakReferenceMessenger.Default.Register<TMessage>(this, (r, m) =>
        {
            if (IsActive)
                action(m);
        });
    }
}
```

## 使用範例

```csharp
public partial class PersonViewModel : ObservableValidatorRecipient
{
    [ObservableProperty]
    private string name;

    [ObservableProperty]
    private int age;

    public PersonViewModel()
    {
        // 啟用 Messenger 訂閱
        IsActive = true;

        // 訂閱自定義訊息
        RegisterMessage<UpdateNameMessage>(msg =>
        {
            Name = msg.NewName;
            ValidateProperty(Name, nameof(Name));
        });
    }
}

// 訊息定義
public class UpdateNameMessage
{
    public string NewName { get; }
    public UpdateNameMessage(string newName) => NewName = newName;
}
```

### 功能說明

1. **Messenger 訂閱封裝**
    - `RegisterMessage<T>` 可以避免每個 ViewModel 重複寫 `WeakReferenceMessenger.Default.Register`
    - `IsActive` 可控制訊息接收啟用/停用，方便在 View 消失時暫停接收訊息，避免 memory leak 或不必要的更新

2. **資料驗證**
    - 繼承自 `ObservableValidator`，可以直接使用 `ValidateProperty` 或 `ValidateAllProperties`
    - 配合 `[ObservableProperty]` 或屬性驗證特性，例如 `[Required]`、`[Range]` 等

3. **可擴展**
    
    - 若有更多跨 ViewModel 通訊需求，可以在 `InitializeMessenger()` 裡統一註冊訊息
    - ViewModel 不需知道 Messenger 實作細節，只要呼叫 `RegisterMessage`
