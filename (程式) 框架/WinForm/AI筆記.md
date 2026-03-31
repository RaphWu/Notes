---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# MessengerViewModelBase

很好，我來幫你把剛才的 `MessengerViewModelBase` 架構**擴充成一個可應用在 MVVM 專案中完整支援**：

* ✅ 訊息註冊與自動清除
* ✅ Dialog 資料傳遞 * ✅ 導覽（Navigation）事件傳遞
* ✅ 可選的通用訊息定義（不需一直寫 message 類別）

# 🧱 基礎：MessengerViewModelBase.cs

（與前面相同）

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Messaging;

public abstract class MessengerViewModelBase : ObservableRecipient
{
    protected MessengerViewModelBase()
    {
        IsActive = true; // 啟用後會自動觸發 OnActivated
    }

    protected override void OnActivated()
    {
        RegisterMessages();
    }

    protected override void OnDeactivated()
    {
        UnregisterMessages();
    }

    protected virtual void RegisterMessages() { }

    protected virtual void UnregisterMessages()
    {
        WeakReferenceMessenger.Default.UnregisterAll(this);
    }
}
```

## 🧩 新增功能 1：Dialog 傳遞訊息支援

## 📁 `DialogRequestMessage.cs`

```csharp
public class DialogRequestMessage<TResponse>
{
    public Action<TResponse> Respond { get; }

    public DialogRequestMessage(Action<TResponse> respond)
    {
        Respond = respond;
    }
}
```

## ✅ 用法範例

### 發送方（開啟 Dialog）

```csharp
WeakReferenceMessenger.Default.Send(new DialogRequestMessage<YourDialogResult>(result =>
{
    // 這裡是 Dialog 關閉後回傳資料的位置
    HandleDialogResult(result);
}));
```

### Dialog ViewModel

```csharp
public class DialogViewModel : MessengerViewModelBase
{
    protected override void RegisterMessages()
    {
        WeakReferenceMessenger.Default.Register<DialogRequestMessage<YourDialogResult>>(this, (r, m) =>
        {
            // 顯示對話框、處理輸入
            var result = new YourDialogResult { ... };
            m.Respond(result); // 回傳結果
        });
    }
}
```

## 🧭 新增功能 2：Navigation 訊息（ViewModel 導頁）

### 📁 `NavigationMessage.cs`

```csharp
public class NavigationMessage
{
    public string TargetPageKey { get; }
    public object? Parameter { get; }

    public NavigationMessage(string targetPageKey, object? parameter = null)
    {
        TargetPageKey = targetPageKey;
        Parameter = parameter;
    }
}
```

### ✅ 發送頁面導覽

```csharp
WeakReferenceMessenger.Default.Send(new NavigationMessage("DetailPage", id));
```

### ✅ 處理頁面切換的地方（通常是 App 或 NavigationService）

```csharp
WeakReferenceMessenger.Default.Register<NavigationMessage>(this, (r, m) =>
{
    // 根據 m.TargetPageKey 做導航，例如：
    navigationService.NavigateTo(m.TargetPageKey, m.Parameter);
});
```

### 🧰 通用訊息傳遞：PayloadMessage\<T\>

如果你懶得一直寫 message 類別，可自訂一個泛型訊息類型。

### 📁 `PayloadMessage.cs`

```csharp
public class PayloadMessage<T>
{
    public T Payload { get; }

    public PayloadMessage(T payload)
    {
        Payload = payload;
    }
}
```

### ✅ 發送簡單資料

```csharp
WeakReferenceMessenger.Default.Send(new PayloadMessage<string>("更新標題"));
```

### ✅ 接收

```csharp
WeakReferenceMessenger.Default.Register<PayloadMessage<string>>(this, (r, m) =>
{
    Console.WriteLine(m.Payload); // 拿到字串
});
```

## ✅ 總結你的 MVVM 架構工具包

| 功能                        | 說明                              |
| - | - |
| `MessengerViewModelBase`  | 自動註冊/解除所有訊息                     |
| `DialogRequestMessage<T>` | 支援開啟 Dialog 並取得回傳結果             |
| `NavigationMessage`       | 實現 ViewModel to ViewModel 的頁面跳轉 |
| `PayloadMessage<T>`       | 快速傳遞任何資料的訊息封裝                   |

## ✅ 下一步建議

若你有多頁切換、Dialog 重複使用、或有進一步需要封裝 NavigationService / DialogService，我可以幫你做：

* `INavigationService` 介面設計
* `IDialogService` 封裝 async 回傳
* 支援 WPF, MAUI, 或 WinUI

要不要我幫你接著擴充這套服務層？還是你目前的架構平台我可以先針對性優化？只要告訴我你用的框架（WPF/MAUI/...），我可以幫你套進去。

---

# WinForms 換頁 + Autofac + Messenger 架構模板

太棒了！我們來打造一套適用於 **WinForms** 的 **頁面導航架構**，結合以下功能：

## ✅ 功能目標

* ✔ 使用 Autofac 自動建立頁面（`UserControl`）
* ✔ 每個頁面一個 `LifetimeScope`（切換時自動釋放）
* ✔ 支援傳參數（ViewModel 注入初始資料）
* ✔ 整合 `MessengerViewModelBase` → 可自動註冊/解除訊息
* ✔ 一行切換頁面：`NavigateTo<SomePage>(param)`

## 📦 1. 共用接口與管理器

### 📁 `INavigationService.cs`

```csharp
public interface INavigationService
{
    void NavigateTo<TPage>(object? parameter = null) where TPage : UserControl;
}
```

### 📁 `NavigationService.cs`

```csharp
using Autofac;
using System.Windows.Forms;

public class NavigationService : INavigationService
{
    private readonly IContainer _container;
    private readonly Panel _hostPanel;

    private ILifetimeScope? _currentScope;
    private UserControl? _currentPage;

    public NavigationService(IContainer container, Panel hostPanel)
    {
        _container = container;
        _hostPanel = hostPanel;
    }

    public void NavigateTo<TPage>(object? parameter = null) where TPage : UserControl
    {
        // 清除舊頁面與 DI 範圍
        if (_currentPage is MessengerViewModelBase oldVm)
            oldVm.IsActive = false;

        _hostPanel.Controls.Clear();
        _currentPage?.Dispose();
        _currentScope?.Dispose();

        // 建立新範圍
        _currentScope = _container.BeginLifetimeScope(builder =>
        {
            // 傳參數到 Page 或 ViewModel（若需要）
            if (parameter != null)
            {
                builder.RegisterInstance(parameter).AsSelf();
            }
        });

        _currentPage = _currentScope.Resolve<TPage>();

        // 若 Page 是 MessengerViewModelBase，自動啟用
        if (_currentPage is MessengerViewModelBase newVm)
            newVm.IsActive = true;

        _hostPanel.Controls.Add(_currentPage);
        _currentPage.Dock = DockStyle.Fill;
    }
}
```

## 🧱 2. 建立頁面基底類別

### 📁 `MessengerViewModelBase.cs`

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Messaging;

public abstract class MessengerViewModelBase : ObservableRecipient
{
    protected MessengerViewModelBase()
    {
        IsActive = true;
    }

    protected override void OnActivated()
    {
        RegisterMessages();
    }

    protected override void OnDeactivated()
    {
        WeakReferenceMessenger.Default.UnregisterAll(this);
    }

    protected virtual void RegisterMessages() { }
}
```

## 🧪 3. 範例：建立一個頁面

### 📁 `MaintiFlowPage.cs`

```csharp
public partial class MaintiFlowPage : UserControl
{
    public MaintiFlowPage(SomePageViewModel vm)
    {
        InitializeComponent();
        DataContext = vm;
    }
}
```

### 📁 `SomePageViewModel.cs`

```csharp
public class SomePageViewModel : MessengerViewModelBase
{
    public string Title { get; set; }

    public SomePageViewModel(SomeParameter param)
    {
        Title = $"參數值為：{param.Value}";
    }

    protected override void RegisterMessages()
    {
        WeakReferenceMessenger.Default.Register<SomeMessage>(this, (r, m) =>
        {
            Title = m.Content;
        });
    }
}
```

## 🧪 4. Autofac 註冊（例如在 `Program.cs` 或 Startup）

```csharp
var builder = new ContainerBuilder();

// 註冊頁面和 ViewModel
builder.RegisterType<MaintiFlowPage>().AsSelf();
builder.RegisterType<SomePageViewModel>().AsSelf();

// 建立容器
var container = builder.Build();

// 建立導航服務（假設你的 MainForm 有一個 panelMain）
var navigationService = new NavigationService(container, mainForm.panelMain);

// 傳到主視窗
mainForm.SetNavigationService(navigationService);
```

## 🚀 5. 呼叫導航切換頁面（在 MainForm）

```csharp
public partial class MainForm : Form
{
    private INavigationService _nav;

    public void SetNavigationService(INavigationService nav)
    {
        _nav = nav;
    }

    private void btnSwitch_Click(object sender, EventArgs e)
    {
        _nav.NavigateTo<MaintiFlowPage>(new SomeParameter { Value = "From Main" });
    }
}
```

## ✅ 完整支援情境

| 功能                                 | 支援？                                 |
| - | -- |
| 換頁自動釋放資源                           | ✅ `_currentScope.Dispose()`         |
| ViewModel 支援 Messenger 與自動取消註冊     | ✅ `MessengerViewModelBase`          |
| 支援傳參數                              | ✅ 可傳任意物件進 ViewModel                 |
| Page 或 ViewModel 可根據參數決定邏輯         | ✅ 可使用 DI 注入                         |
| 換頁自動觸發 OnActivated / OnDeactivated | ✅ 是                                 |
| 控制項記憶體洩漏？                          | ❌ 不會，因為舊頁面會 Dispose、Messenger 註冊會解除 |

## 🧰 可加強功能（如有需要我可以幫你加）

* ✅ 自動解析 ViewModel + 自動繫結到 Page（不用手動寫 ViewModel）
* ✅ 頁面導航回退（Back）
* ✅ Dialog 彈跳 + 結果回傳
* ✅ 支援多個 Panel 同時導航（像 TabControl）

---

# NavigationService 與 DialogService 封裝 (WinForms + Autofac + CommunityToolkit.Mvvm)

以下是符合你需求的完整範例，包含導航服務與對話框服務，並整合 CommunityToolkit.Mvvm 的 Messenger 與 ObservableRecipient。

## 1. 介面定義

```csharp
public interface INavigationService
{
    void NavigateTo<TPage>(object? parameter = null) where TPage : UserControl;
}

public interface IDialogService
{
    /// <summary>
    /// 顯示 Dialog 並回傳結果
    /// </summary>
    Task<TResult?> ShowDialogAsync<TDialog, TResult>(object? parameter = null)
        where TDialog : Form
        where TResult : class;
}
```

## 2. NavigationService 實作（WinForms）

```csharp
using Autofac;
using System.Windows.Forms;

public class NavigationService : INavigationService
{
    private readonly IContainer _container;
    private readonly Panel _hostPanel;

    private ILifetimeScope? _currentScope;
    private UserControl? _currentPage;

    public NavigationService(IContainer container, Panel hostPanel)
    {
        _container = container;
        _hostPanel = hostPanel;
    }

    public void NavigateTo<TPage>(object? parameter = null) where TPage : UserControl
    {
        if (_currentPage is ObservableRecipient oldVm)
            oldVm.IsActive = false;

        _hostPanel.Controls.Clear();
        _currentPage?.Dispose();
        _currentScope?.Dispose();

        _currentScope = _container.BeginLifetimeScope(builder =>
        {
            if (parameter != null)
                builder.RegisterInstance(parameter).AsSelf();
        });

        _currentPage = _currentScope.Resolve<TPage>();

        if (_currentPage is ObservableRecipient newVm)
            newVm.IsActive = true;

        _hostPanel.Controls.Add(_currentPage);
        _currentPage.Dock = DockStyle.Fill;
    }
}
```

## 3. DialogService 實作（WinForms）

```csharp
using Autofac;
using System.Threading.Tasks;
using System.Windows.Forms;

public class DialogService : IDialogService
{
    private readonly IContainer _container;

    public DialogService(IContainer container)
    {
        _container = container;
    }

    public Task<TResult?> ShowDialogAsync<TDialog, TResult>(object? parameter = null)
        where TDialog : Form
        where TResult : class
    {
        var tcs = new TaskCompletionSource<TResult?>();

        using var scope = _container.BeginLifetimeScope(builder =>
        {
            if (parameter != null)
                builder.RegisterInstance(parameter).AsSelf();
        });

        var dialog = scope.Resolve<TDialog>();

        void Handler(object? sender, FormClosedEventArgs e)
        {
            if (dialog is IDialogResult<TResult> dialogResultProvider)
                tcs.TrySetResult(dialogResultProvider.GetResult());
            else
                tcs.TrySetResult(null);

            dialog.FormClosed -= Handler;
            dialog.Dispose();
        }

        dialog.FormClosed += Handler;
        dialog.Show();

        return tcs.Task;
    }
}
```

## 4. Dialog Result 介面（給 Dialog Form 實作）

```csharp
public interface IDialogResult<TResult>
{
    TResult? GetResult();
}
```

## 5. 範例 Dialog Form 實作

```csharp
public partial class ConfirmDialog : Form, IDialogResult<bool>
{
    private bool _result = false;

    public ConfirmDialog()
    {
        InitializeComponent();
    }

    public bool GetResult() => _result;

    private void btnOk_Click(object sender, EventArgs e)
    {
        _result = true;
        Close();
    }

    private void btnCancel_Click(object sender, EventArgs e)
    {
        _result = false;
        Close();
    }
}
```

## 6. 使用範例

```csharp
// 導航切換頁面
_navigationService.NavigateTo<MaintiFlowPage>(new SomeParameter { Value = "Hello" });

// 顯示 Dialog 並等待結果
bool? confirmed = await _dialogService.ShowDialogAsync<ConfirmDialog, bool>();
if (confirmed == true)
{
    // 使用者按了 OK
}
```

## 7. Autofac 註冊示範

```csharp
var builder = new ContainerBuilder();

builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();

builder.RegisterType<MaintiFlowPage>().AsSelf();
builder.RegisterType<SomePageViewModel>().AsSelf();

builder.RegisterType<ConfirmDialog>().AsSelf();

var container = builder.Build();
```

---

# 完整範例專案架構示範（WinForms + Autofac + MVVM + 快取 + 導航）

下面是整合前面提到的快取服務 (註：在 EF6 那邊)、導航服務、DialogService 的一個簡單示範架構，幫你快速上手：

## 1. Autofac 註冊

```csharp
var builder = new ContainerBuilder();

// DbContext 每次解析新建
builder.RegisterType<MyDbContext>().InstancePerDependency();

// 快取服務單例（跨頁共享資料）
builder.RegisterType<EmployeeCacheService>().As<IEmployeeCacheService>().SingleInstance();

// 導航服務需要傳入主Panel (主視窗上放頁面的容器)
builder.Register(c => new NavigationService(c.Resolve<IContainer>(), mainPanel))
       .As<INavigationService>()
       .SingleInstance();

builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();

builder.RegisterType<SomePage>().AsSelf();
builder.RegisterType<SomePageViewModel>().AsSelf();
```

## 2. EmployeeCacheService

```csharp
public interface IEmployeeCacheService
{
    Task<IReadOnlyList<Employee>> GetEmployeesAsync();
    void ClearCache();
}

public class EmployeeCacheService : IEmployeeCacheService
{
    private readonly MyDbContext _dbContext;
    private IReadOnlyList<Employee>? _cache;

    public EmployeeCacheService(MyDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<IReadOnlyList<Employee>> GetEmployeesAsync()
    {
        if (_cache == null)
        {
            _cache = await _dbContext.Employees.ToListAsync();
        }
        return _cache;
    }

    public void ClearCache() => _cache = null;
}
```

## 3. SomePageViewModel 範例

```csharp
public class SomePageViewModel : ObservableRecipient
{
    private readonly IEmployeeCacheService _cacheService;
    public ObservableCollection<Employee> Employees { get; } = new();

    public SomePageViewModel(IEmployeeCacheService cacheService)
    {
        _cacheService = cacheService;
        LoadEmployeesAsync();
    }

    private async void LoadEmployeesAsync()
    {
        var employees = await _cacheService.GetEmployeesAsync();
        Employees.Clear();
        foreach (var emp in employees)
            Employees.Add(emp);
    }
}
```

## 4. SomePage 範例

```csharp
public partial class SomePage : UserControl
{
    public SomePage(SomePageViewModel vm)
    {
        InitializeComponent();
        this.DataContext = vm;
    }
}
```

## 5. NavigationService 範例（簡化）

```csharp
public class NavigationService : INavigationService
{
    private readonly IContainer _container;
    private readonly Panel _hostPanel;
    private ILifetimeScope? _scope;
    private UserControl? _currentPage;

    public NavigationService(IContainer container, Panel hostPanel)
    {
        _container = container;
        _hostPanel = hostPanel;
    }

    public void NavigateTo<TPage>(object? parameter = null) where TPage : UserControl
    {
        _scope?.Dispose();
        _hostPanel.Controls.Clear();

        _scope = _container.BeginLifetimeScope(builder =>
        {
            if (parameter != null)
                builder.RegisterInstance(parameter).AsSelf();
        });

        _currentPage = _scope.Resolve<TPage>();
        _hostPanel.Controls.Add(_currentPage);
        _currentPage.Dock = DockStyle.Fill;
    }
}
```

## 6. 使用示範（Form 或主控件中）

```csharp
// 假設 mainPanel 是放 UserControl 的 Panel
var navigationService = container.Resolve<INavigationService>();

// 換頁呼叫
navigationService.NavigateTo<SomePage>();
```

## 7. 更新資料後刷新快取

```csharp
var cacheService = container.Resolve<IEmployeeCacheService>();

// 新增/修改/刪除資料後
cacheService.ClearCache();
```

# 總結

這樣架構：

* 換頁時 ViewModel 用快取服務讀資料，不用每次都重讀 DB
* 快取服務由單例管理，方便資料共享和更新
* Autofac 負責注入 DbContext 和快取服務，生命週期清楚
* NavigationService 負責換頁，確保頁面與 ViewModel 分離管理
