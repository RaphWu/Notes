---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

aliases:
date:
update:
author:
language:
sourceurl:
tags:

# # DbContext 要如何註冊

這個問題非常重要，因為 **DbContext 的生命週期管理** 直接影響到 **效能、資源釋放、交易一致性與執行錯誤**。
我用最清楚的方式幫你分析：

## ❌ DbContext 註冊成 SingleInstance() 是錯誤做法

**DbContext 不是 thread-safe，也不適合長時間存在。**
如果用 `SingleInstance()`：

- 整個程式共享同一個 DbContext
- 多執行緒會衝突
- Cache 會越來越大
- 追蹤的 Entity 會越來越多
- 一段時間後 SaveChanges() 非常慢
- Memory leak
- Transaction 無法正確控制

✅ 官方與 DDD 社群都建議：**DbContext 應該是短生命週期（Per Request / Per Scope / Per Operation）**

## ✅ 正確註冊方式 1（推薦）

```csharp
builder.RegisterType<MyDbContext>().InstancePerLifetimeScope();
```

這代表：

- 每個 LifetimeScope 有自己的 DbContext 實例
- 當 Scope Dispose → DbContext Dispose
- 每個業務操作 / 每個頁面可以各自使用

## ✅ 正確註冊方式 2（只當你要更細控制）

如果你希望「每次 Resolve 都是全新 DbContext」：

```csharp
builder.RegisterType<MyDbContext>().InstancePerDependency();
```

但這會導致同一個 Scope 內多次查詢會用不同 DbContext，
**可能破壞單一交易一致性**，所以比較少用。

## ✅ 那麼在其他類別要怎麼用 DbContext？

你問到的兩種方式：

### ✅ 方式 A：直接在建構式注入 DbContext

```csharp
public class MyService
{
    private readonly MyDbContext _db;

    public MyService(MyDbContext db)
    {
        _db = db;
    }
}
```

📌 前提：**MyService 本身要以 InstancePerLifetimeScope 註冊**

這樣 MyService 使用的 DbContext 會跟 Scope 綁在一起，**最符合 DDD / EF 的推薦做法**

### ✅ 方式 B：先取得 ILifetimeScope 再 Resolve

```csharp
public class MyService
{
    private readonly ILifetimeScope _scope;

    public MyService(ILifetimeScope scope)
    {
        _scope = scope;
    }

    public void DoWork()
    {
        using(var db = _scope.Resolve<MyDbContext>())
        {
            // 作業...
        }
    }
}
```

📌 用在你只想要「在某個操作產生新的 DbContext」的情境
📌 這種方式較靈活，但會造成 Service 與 IoC 耦合 → **不推薦長期使用**

## ❓ 那如果 DbContext 是 SingleInstance() 呢？

那麼不管你是建構式注入還是 Resolve 都一樣，因為全系統只會有一個 DbContext
👉 所以 SingleInstance() 是根本問題，**應該先改掉**

## ✅ 實務最佳組合（WinForms / WPF / Prism / Web 都適用）

|類別|註冊方式|
|||
|DbContext|InstancePerLifetimeScope() ✅|
|Repository / Service|InstancePerLifetimeScope() ✅|
|ViewModel|InstancePerLifetimeScope() ✅|
|Form / UserControl|AsSelf() 或 InstancePerDependency() ✅|
|全局共用物件 (Cache、Config、Logger)|SingleInstance() ✅|

## ✅ 最佳使用方法

在「每次操作/每個頁面」建立一個 LifetimeScope：

```csharp
using(var scope = container.BeginLifetimeScope())
{
    var viewModel = scope.Resolve<MyPageViewModel>();
    // viewModel 裡面注入的 DbContext 就屬於這個 scope
}
```

當 scope.Dispose() 時：

- viewModel Dispose
- service Dispose
- repository Dispose
- DbContext Dispose ✅ 自動完成

## 🎯 最終結論

|問題|建議|
|||
|DbContext 用 SingleInstance()?|❌ 千萬不要|
|DbContext 正確註冊|✅ InstancePerLifetimeScope()|
|在類別建構式直接注入 DbContext?|✅ 最推薦方式|
|用 ILifetimeScope.Resolve() 取得 DbContext?|✅ 可用於特殊情境，但不建議常用|
|DbContext 必須由 IoC 建立?|✅ 不要自己 new|

---

# Autofac + EF6 + WinForms（使用 LifetimeScope 管理 DbContext / ViewModel / UserControl）的完整範例

好，我把你要的 **Autofac + EF6 + WinForms（使用 LifetimeScope 管理 DbContext / ViewModel / UserControl）** 的完整範例整理出來 — 含程式碼片段與重點說明。你可以直接把它貼進專案測試或作為參考。

> 設計目標
> - DbContext 不會是 SingleInstance，而是 `InstancePerLifetimeScope()`。
> - 每個頁面（UserControl）有自己的 lifetime scope；切換頁面時 `Dispose()` scope 並釋放該頁面與 DbContext。
> - 保持 WinForms Designer 可用（UserControl 保留無參建構子 + 使用屬性注入或 Resolve 時用 constructor injection）。
> - MainForm 從容器注入 `ILifetimeScope`，負責為頁面建立 scope。

## 主要檔案（簡化版）

### 1) `AppDbContext`（EF6）

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext() : base("name=DefaultConnection")
    {
    }

    public DbSet<Person> People { get; set; }
}
```

### 2) Repository 與 Service（範例）

```csharp
public interface IPersonRepository
{
    IEnumerable<Person> GetAll();
}

public class PersonRepository : IPersonRepository
{
    private readonly AppDbContext _db;
    public PersonRepository(AppDbContext db) => _db = db;
    public IEnumerable<Person> GetAll() => _db.People.ToList();
}

public class PersonService
{
    private readonly IPersonRepository _repo;
    public PersonService(IPersonRepository repo) => _repo = repo;
    public IEnumerable<Person> GetAll() => _repo.GetAll();
}
```

### 3) ViewModel（短生命週期）

```csharp
public class PageViewModel
{
    private readonly PersonService _service;
    public BindingList<Person> People { get; } = new BindingList<Person>();

    public PageViewModel(PersonService service)
    {
        _service = service;
        // 這通常在 Load 時呼叫或由外部觸發
    }

    public void LoadData()
    {
        People.Clear();
        foreach (var p in _service.GetAll())
            People.Add(p);
    }
}
```

### 4) UserControl（View） — Designer 友善

```csharp
public partial class PageControl : UserControl
{
    // 不在建構子注入，保留無參建構子給 Designer
    public PageViewModel ViewModel { get; set; }

    public PageControl()
    {
        InitializeComponent();
        // Designer-friendly
    }

    protected override void OnLoad(EventArgs e)
    {
        base.OnLoad(e);
        if (ViewModel != null)
        {
            // 綁定 UI -> ViewModel
            dataGridView1.DataSource = ViewModel.People;
            ViewModel.LoadData();
        }
    }

    protected override void Dispose(bool disposing)
    {
        // 可以觀察到 Dispose 時點
        base.Dispose(disposing);
    }
}
```

> 備註：`PageControl` 在 runtime 由 Autofac Resolve 時，可以選擇 constructor injection（若有），但為了 Designer 我們用屬性注入：`PropertiesAutowired()`。

### 5) MainForm（負責切換頁面與 scope 管理）

```csharp
public partial class MainForm : Form
{
    private readonly ILifetimeScope _rootScope;
    private ILifetimeScope _currentPageScope;
    private Control _currentPageControl;

    public MainForm(ILifetimeScope rootScope)
    {
        InitializeComponent();
        _rootScope = rootScope;

        btnPage1.Click += (s, e) => ShowPage<PageControl>("Page1");
        // 其他按鈕...
    }

    private void ShowPage<T>(string pageName) where T : Control
    {
        // Dispose 舊的頁面與 scope
        if (_currentPageControl != null)
        {
            panelContainer.Controls.Remove(_currentPageControl);
            _currentPageControl.Dispose();
            _currentPageControl = null;
        }
        _currentPageScope?.Dispose();
        _currentPageScope = null;

        // 建新的 scope（每個頁面一個 scope）
        _currentPageScope = _rootScope.BeginLifetimeScope();

        // Resolve UserControl（若有 ctor 注入會自動注入；PropertiesAutowired 也會發生）
        var page = _currentPageScope.Resolve<T>();

        // 如果你使用屬性注入 (PropertiesAutowired) 而 ViewModel 為 InstancePerLifetimeScope，
        // 它會在這個 scope 內被建立並注入到 page。
        page.Dock = DockStyle.Fill;
        panelContainer.Controls.Add(page);
        _currentPageControl = page;
    }

    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        base.OnFormClosing(e);
        // 關閉時清理
        _currentPageScope?.Dispose();
        _rootScope.Dispose(); // 視需求：若 rootScope 是 BeginLifetimeScope() 建立的，才 dispose
    }
}
```

## Program.cs（啟動與註冊）

```csharp
static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);

        var builder = new ContainerBuilder();

        // EF DbContext: 每個 LifetimeScope 一個實例
        builder.RegisterType<AppDbContext>()
               .AsSelf()
               .InstancePerLifetimeScope();

        // Repository / Service: 與 DbContext 同一 scope
        builder.RegisterType<PersonRepository>().As<IPersonRepository>().InstancePerLifetimeScope();
        builder.RegisterType<PersonService>().InstancePerLifetimeScope();

        // ViewModel: 每頁 scope 的 VM
        builder.RegisterType<PageViewModel>().InstancePerLifetimeScope();

        // UserControl (View): 註冊成 Self，允許屬性注入（Designer-friendly）
        builder.RegisterType<PageControl>().AsSelf().PropertiesAutowired();

        // MainForm: root scope 注入 ILifetimeScope (會注入為當前 scope)
        builder.RegisterType<MainForm>().AsSelf().InstancePerDependency();

        var container = builder.Build();

        // 你可以將 container 存到靜態 AppContainer（若需要），但不建議濫用
        using (var rootScope = container.BeginLifetimeScope())
        {
            // Resolve MainForm 後啟動
            var mainForm = rootScope.Resolve<MainForm>();
            Application.Run(mainForm);
        }
    }
}
```

## 重點說明（整理）

1. **DbContext 使用 `InstancePerLifetimeScope()`**：
    每個範圍（Scope）會有自己的 DbContext，且在 Scope.Dispose 時自動釋放。
    
2. **UserControl/ ViewModel 也用 `InstancePerLifetimeScope()`**：
    這樣 ViewModel 與 DbContext 同 Scope，共享相同 lifetime，且切換頁面時一起被釋放。
    
3. **MainForm 用 injected `ILifetimeScope`（root scope）建立每頁的 child scope**：
    `BeginLifetimeScope()` → `Resolve<PageControl>()` → 加入 Panel；切換時 `Dispose()` scope 與 control。
    
4. **Designer 友善**：
    _UserControl_ 保留無參建構子並使用屬性注入（`PropertiesAutowired()`）或在 Resolve 時由 Autofac 以 constructor injection 建立（但是 Designer 不能處理有參建構子）。
    
5. **避免把 DbContext 註冊為 SingleInstance()**（全域單例）：會造成 thread-safety、記憶體累積與追蹤問題。

---

# 多人同時存取資料庫與 DbContext 生命週期管理

EF6 的 `DbContext` 不是線程安全的物件，**不能在多個執行緒或多個使用者共用同一個 DbContext 實例**。
在多人同時存取資料庫的情況下，以下原則和建議很重要：

## 1. 每個使用者（Request）使用獨立的 DbContext 實例

* **正確做法是「每個使用者操作範圍（request、action）」建立自己的 DbContext**
* 這樣避免執行緒安全問題與資料狀態競爭。

## 2. 在桌面應用（WinForms）中，多人同時存取通常指

* 多個使用者在不同電腦開啟同一系統，透過資料庫共享資料
* 每台用戶端應自己管理 DbContext，**不應共用 DbContext 實例**

## 3. 針對換頁與資料庫連線

* 每個頁面、或每個功能單元應該建立並使用自己的 DbContext
* 用完就 Dispose，確保資源釋放與狀態乾淨
* 不同用戶端不會互相干擾，資料庫本身會負責處理多用戶連線與鎖定

## 4. 長生命週期 DbContext 的風險

* 在桌面應用維持單一長生命週期 DbContext，**可能導致狀態同步問題與記憶體膨脹**
* 例如：資料變更無法即時反映、快取資料過期
* 不建議多人應用這樣做，除非有完善的同步機制

## 5. 建議的架構

* **每次使用 DbContext 都盡量短暫建立與釋放**，確保獨立安全
* 使用非同步查詢提升 UI 流暢度
* 換頁時建立新的 DbContext 實例
* 讓資料庫和 EF6 處理多用戶鎖定、交易隔離

## 總結表格

| 情境 | 可行性 | 注意事項 |
| -- | | |
| 多用戶共用單一 DbContext 實例 | ❌ 不建議，EF6 非線程安全 | 易產生資料競爭與異常 |
| 每用戶各自管理獨立 DbContext | ✅ 合理且推薦 | 注意建立與 Dispose 範圍管理 |
| 長生命週期 DbContext | ⚠ 風險較高 | 記憶體膨脹、狀態不同步 |

如果你要，我可以幫你設計一套 WinForms + Autofac 管理 DbContext 的生命週期範例，確保多用戶環境安全又高效。需要嗎？

---

# 多人同時存取 EF6 資料庫的 WinForms + Autofac DbContext 管理範例

這個範例示範如何在 WinForms 專案中使用 Autofac 管理 EF6 `DbContext`，確保：

* 每次使用建立新 `DbContext`（短生命週期）
* 換頁時頁面自己建立和 Dispose DbContext
* 多用戶安全，避免共用 DbContext 實例導致問題
* 搭配非同步查詢提升 UI 體驗（需 EF6 支援）

## 1. DbContext 註冊（Autofac）

```csharp
builder.RegisterType<MyDbContext>()
       .InstancePerDependency();  // 每次解析都新建立
```

## 2. 範例頁面 ViewModel

```csharp
public class SomePageViewModel : ObservableRecipient
{
    private readonly MyDbContext _dbContext;

    public ObservableCollection<Employee> Employees { get; } = new ObservableCollection<Employee>();

    public SomePageViewModel(MyDbContext dbContext)
    {
        _dbContext = dbContext;
        LoadDataAsync();
    }

    private async void LoadDataAsync()
    {
        // EF6 非同步需安裝 System.Data.Entity.EF6.Async NuGet
        var data = await _dbContext.Employees.ToListAsync();
        Employees.Clear();
        foreach(var emp in data)
            Employees.Add(emp);
    }

    protected override void OnActivated()
    {
        base.OnActivated();
        // 可註冊 Messenger 訊息
    }

    protected override void OnDeactivated()
    {
        base.OnDeactivated();
        _dbContext.Dispose();  // 換頁時釋放資源
    }
}
```

## 3. 頁面註冊與導航

```csharp
builder.RegisterType<SomePageViewModel>().AsSelf();
builder.RegisterType<SomePage>().AsSelf();

public class SomePage : UserControl
{
    public SomePage(SomePageViewModel vm)
    {
        InitializeComponent();
        DataContext = vm;
    }
}
```

換頁服務中每次 Resolve 都建立新 Scope，自動建立新的 DbContext：

```csharp
public void NavigateTo<TPage>(object? param = null) where TPage : UserControl
{
    _currentScope?.Dispose();
    _currentScope = _container.BeginLifetimeScope(b =>
    {
        if(param != null)
            b.RegisterInstance(param).AsSelf();
    });
    _currentPage = _currentScope.Resolve<TPage>();
    ...
}
```

## 4. 重點說明

* **每換頁，DbContext 是新實例**，不會共用，避免多用戶資料衝突
* **短生命週期 DbContext**，每個 ViewModel 只使用自己獨立的實例
* **呼叫 Dispose()** 釋放資源，避免記憶體洩漏
* **非同步載入資料**，避免 UI 卡頓

---

# 換頁快取服務設計範例（WinForms + Autofac + MVVM）

這裡示範一個簡單的「資料快取服務」，搭配 ViewModel 使用，實現換頁時共享資料、避免每次都從資料庫重讀。

## 1. 資料快取服務介面與實作

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

    public void ClearCache()
    {
        _cache = null;
    }
}
```

## 2. Autofac 註冊

```csharp
builder.RegisterType<MyDbContext>().InstancePerDependency();

builder.RegisterType<EmployeeCacheService>()
       .As<IEmployeeCacheService>()
       .SingleInstance();  // 單例快取服務，跨換頁共享
```

## 3. ViewModel 使用快取服務

```csharp
public class SomePageViewModel : ObservableRecipient
{
    private readonly IEmployeeCacheService _cacheService;

    public ObservableCollection<Employee> Employees { get; } = new ObservableCollection<Employee>();

    public SomePageViewModel(IEmployeeCacheService cacheService)
    {
        _cacheService = cacheService;
        LoadEmployeesAsync();
    }

    private async void LoadEmployeesAsync()
    {
        var employees = await _cacheService.GetEmployeesAsync();
        Employees.Clear();
        foreach(var emp in employees)
            Employees.Add(emp);
    }
}
```

## 4. 使用說明

* **EmployeeCacheService 是單例，共享快取資料**
* 每次換頁，ViewModel 呼叫 `GetEmployeesAsync()`，第一次會從資料庫載入，之後就使用快取資料
* 需要更新資料時，可以呼叫 `ClearCache()` 讓快取失效，下一次才會重新從資料庫讀取

## 5. 快取更新示意（例如新增、修改員工資料後）

```csharp
// 更新資料後清除快取
_cacheService.ClearCache();

// 再次呼叫 GetEmployeesAsync() 就會重新從 DB 載入最新資料
```

## 總結

* 這樣做可避免換頁時每次都從資料庫重讀
* 需要時可手動刷新快取保持資料同步
* 對於頻繁換頁但資料變動不頻繁的情境非常合適
