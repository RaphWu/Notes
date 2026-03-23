---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - WPF
  - CSharp
  - Autofac
  - MVVM
---

# 簡介

下面是一個完整可參考的範例架構（程式碼片段），在 WPF 下使用 Autofac 與 CommunityToolkit.Mvvm 實作：
- 多層頁面導航（History Stack）與回退
- 每頁生命週期（OnLoaded / OnUnloaded / OnNavigatedTo / OnNavigatedFrom / Dispose）
- 每頁可設定 `KeepAlive`，KeepAlive 頁面以「平行 scope」建立，不會被上層 Dispose 影響
- Dialog 服務管理生命週期、可傳入複雜參數並回傳結果
- 支援 Refresh/Update 的 Service（含簡易 Cache 範例）
   
下面把檔案按功能組織，程式碼可直接放入專案調整後執行。

## 專案檔案總覽

- App.xaml / App.xaml.cs
- Program.cs（Autofac 註冊啟動）
- Views
    - MainWindow.xaml
    - Page1View.xaml
    - Page2View.xaml
    - Page3View.xaml
    - MyDialogView.xaml
- ViewModels
    - ViewModelBase.cs
    - MainViewModel.cs
    - Page1ViewModel.cs
    - Page2ViewModel.cs
    - Page3ViewModel.cs
    - MyDialogViewModel.cs
- Services
    - INavigationService.cs / NavigationService.cs
    - IDialogService.cs / DialogService.cs
    - IPageDataService.cs / PageDataService.cs
- App.xaml 中 DataTemplate 映射 ViewModel -> View

## ViewModelBase（生命週期、KeepAlive）

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using System;

public abstract class ViewModelBase : ObservableObject, IDisposable
{
    // 是否保留實例（KeepAlive）。覆寫為 true 表示要保留。
    public virtual bool KeepAlive => false;

    // 當頁面被顯示（或切回）時呼叫
    public virtual void OnLoaded() { }

    // 當頁面被隱藏（或將要離開）時呼叫
    public virtual void OnUnloaded() { }

    // 導航到此頁時傳入參數
    public virtual void OnNavigatedTo(object parameter) { }

    // 導航離開時回傳資料（Dialog 或 Navigate 回傳）
    public virtual object OnNavigatedFrom() => null;

    // 釋放資源
    public virtual void Dispose() { }
}
```

## INavigationService / NavigationService（Mix 模式：巢狀 + 平行 Scope）

```csharp
using Autofac;
using System;
using System.Collections.Generic;

public interface INavigationService
{
    ViewModelBase Current { get; }
    void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase;
    void GoBack();
    void ReplaceCurrentWith<TViewModel>(object parameter = null) where TViewModel : ViewModelBase;
}

public class NavigationService : INavigationService, IDisposable
{
    private readonly ILifetimeScope _rootScope;
    // History: 存放 (vm, scope) 的 stack（保留的 KeepAlive 頁面）
    private readonly Stack<(ViewModelBase vm, ILifetimeScope scope)> _history = new Stack<(ViewModelBase, ILifetimeScope)>();
    public ViewModelBase Current { get; private set; }
    private ILifetimeScope _currentScope;

    public NavigationService(ILifetimeScope rootScope)
    {
        _rootScope = rootScope;
    }

    // Navigate：若新頁面 KeepAlive=true，從 RootScope 建立「平行 scope」；否則從 currentScope 建立巢狀 scope
    public void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase
    {
        // Unload current
        if (Current != null)
        {
            Current.OnUnloaded();
            if (!Current.KeepAlive)
            {
                // 非 KeepAlive：Dispose current 和它的 scope
                Current.Dispose();
                _currentScope?.Dispose();
            }
            else
            {
                // KeepAlive：推入 history（保留 scope）
                _history.Push((Current, _currentScope));
            }
        }

        // Create new scope depending on KeepAlive of the type (we need an instance to check KeepAlive)
        // Strategy: 先從 appropriate scope 建立、然後檢查 vm.KeepAlive；但為了避免多解，先從 parentScope 建立，若發現 vm.KeepAlive 且 parent 是非 root 會把 scope 轉為 root
        ILifetimeScope createFromScope = _currentScope ?? _rootScope;
        var tempScope = createFromScope.BeginLifetimeScope();
        var vm = tempScope.Resolve<TViewModel>();
        if (vm.KeepAlive)
        {
            // 若需要 KeepAlive，釋放剛建立的 tempScope，改為從 root 建立新平行 scope
            tempScope.Dispose();
            var keepScope = _rootScope.BeginLifetimeScope();
            vm = keepScope.Resolve<TViewModel>();
            _currentScope = keepScope;
        }
        else
        {
            _currentScope = tempScope;
        }

        vm.OnNavigatedTo(parameter);
        vm.OnLoaded();

        Current = vm;
    }

    // ReplaceCurrentWith：強制用新的頁面替換目前頁面（會 Dispose 目前即使 KeepAlive）
    public void ReplaceCurrentWith<TViewModel>(object parameter = null) where TViewModel : ViewModelBase
    {
        if (Current != null)
        {
            Current.OnUnloaded();
            Current.Dispose();
            _currentScope?.Dispose();
            Current = null;
            _currentScope = null;
        }
        Navigate<TViewModel>(parameter);
    }

    // GoBack：取回 history
    public void GoBack()
    {
        if (_history.Count == 0) return;

        // 當前頁面卸載
        if (Current != null)
        {
            Current.OnUnloaded();
            if (!Current.KeepAlive)
            {
                Current.Dispose();
                _currentScope?.Dispose();
            }
            // 若當前是 KeepAlive，通常會被推入 history already - 視需求可選擇 push 或 not
        }

        var (vm, scope) = _history.Pop();
        Current = vm;
        _currentScope = scope;
        Current.OnLoaded();
    }

    public void Dispose()
    {
        // Dispose current and history scopes
        try
        {
            Current?.Dispose();
            _currentScope?.Dispose();
        }
        catch { }
        while (_history.Count > 0)
        {
            var (_, s) = _history.Pop();
            try { s.Dispose(); } catch { }
        }
    }
}
```

## IDialogService / DialogService（Dialog 也使用 scope）

```csharp
using Autofac;
using System.Threading.Tasks;
using System.Windows;

public interface IDialogService
{
    Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase;
}

public class DialogService : IDialogService
{
    private readonly ILifetimeScope _rootScope;

    public DialogService(ILifetimeScope rootScope)
    {
        _rootScope = rootScope;
    }

    public async Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase
    {
        // Dialog 一律用新的 scope（平行於 root），確保生命週期獨立
        var scope = _rootScope.BeginLifetimeScope();
        var vm = scope.Resolve<TViewModel>();
        vm.OnNavigatedTo(parameter);
        vm.OnLoaded();

        var window = new Window
        {
            Content = vm,
            SizeToContent = SizeToContent.WidthAndHeight,
            WindowStartupLocation = WindowStartupLocation.CenterOwner,
            Owner = Application.Current.MainWindow
        };

        // Modal show
        window.ShowDialog();

        // 取得結果並 Dispose
        var result = (TResult)vm.OnNavigatedFrom();
        vm.Dispose();
        scope.Dispose();
        return await Task.FromResult(result);
    }
}
```

## IPageDataService / PageDataService（Refresh / Cache 範例）

```csharp
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

public interface IPageDataService
{
    Task<T> GetOrFetchAsync<T>(string key, Func<Task<T>> factory);
    Task InvalidateAsync(string key);
}

public class PageDataService : IPageDataService
{
    private readonly ConcurrentDictionary<string, object> _cache = new ConcurrentDictionary<string, object>();
    public Task<T> GetOrFetchAsync<T>(string key, Func<Task<T>> factory)
    {
        if (_cache.TryGetValue(key, out var o) && o is T t)
            return Task.FromResult(t);
        return FetchAndCacheAsync(key, factory);
    }

    private async Task<T> FetchAndCacheAsync<T>(string key, Func<Task<T>> factory)
    {
        var v = await factory();
        _cache[key] = v;
        return v;
    }

    public Task InvalidateAsync(string key)
    {
        _cache.TryRemove(key, out _);
        return Task.CompletedTask;
    }
}
```

## 範例 ViewModels（Main, Page1, Page2, Page3, Dialog）

```csharp
using CommunityToolkit.Mvvm.Input;
using System;
using System.Threading.Tasks;

public class MainViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    public IRelayCommand NavigateToPage1Command { get; }
    public IRelayCommand NavigateToPage4Command { get; }

    public MainViewModel(INavigationService nav)
    {
        _nav = nav;
        NavigateToPage1Command = new RelayCommand(() => _nav.Navigate<Page1ViewModel>());
        NavigateToPage4Command = new RelayCommand(() => _nav.Navigate<Page4ViewModel>());
    }
}

public class Page1ViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    public override bool KeepAlive => true; // 示範：主頁保留
    public IRelayCommand NextCommand { get; }

    public Page1ViewModel(INavigationService nav)
    {
        _nav = nav;
        NextCommand = new RelayCommand(() => _nav.Navigate<Page2ViewModel>(new { Id = 42 }));
    }

    public override void OnLoaded() => Console.WriteLine("Page1 Loaded");
    public override void OnUnloaded() => Console.WriteLine("Page1 Unloaded");
    public override void OnNavigatedTo(object parameter) => Console.WriteLine($"Page1 OnNavigatedTo: {parameter}");
    public override void Dispose() => Console.WriteLine("Page1 Dispose");
}

public class Page2ViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    private readonly IPageDataService _dataService;
    public override bool KeepAlive => true; // 中間頁面示範 KeepAlive

    public IRelayCommand GoToPage3Command { get; }
    public IRelayCommand OpenDialogCommand { get; }

    public string Info { get; private set; }

    public Page2ViewModel(INavigationService nav, IPageDataService dataService)
    {
        _nav = nav;
        _dataService = dataService;
        GoToPage3Command = new RelayCommand(() => _nav.Navigate<Page3ViewModel>(new { Info = Info }));
        OpenDialogCommand = new RelayCommand(async () => await OpenDialogAsync());
    }

    public override void OnNavigatedTo(object parameter)
    {
        // 可能第一次呼叫或第二次呼叫；KeepAlive = true 時，導航機制不會重建，要在此或 Update() 中處理刷新
        if (parameter != null)
        {
            dynamic p = parameter;
            Info = $"FromParam:{p?.Id}";
        }
        Console.WriteLine($"Page2 OnNavigatedTo: {Info}");
    }

    public async Task UpdateAsync(object parameter)
    {
        // 使用 service 做 Refresh（可用 cache 避免 EF 反覆查詢）
        var data = await _dataService.GetOrFetchAsync($"page2-{parameter}", async () =>
        {
            // 模擬 EF 查詢
            await Task.Delay(100);
            return $"FetchedData_{DateTime.Now.Ticks}";
        });
        Info = data;
    }

    private async Task OpenDialogAsync()
    {
        var dialogService = ((NavigationService)_nav).ResolveDialogService(); // 下面我們會註冊並提供 Resolve 方法，或直接注入 IDialogService 到 VM
        var result = await dialogService.ShowDialogAsync<MyDialogViewModel, string>(new { Message = "請確認" });
        Console.WriteLine($"Dialog result: {result}");
    }

    public override void OnLoaded() => Console.WriteLine("Page2 Loaded");
    public override void OnUnloaded() => Console.WriteLine("Page2 Unloaded");
    public override void Dispose() => Console.WriteLine("Page2 Dispose");
}

public class Page3ViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    public override bool KeepAlive => false;
    public string Info { get; private set; }
    public IRelayCommand GoBackCommand { get; }

    public Page3ViewModel(INavigationService nav)
    {
        _nav = nav;
        GoBackCommand = new RelayCommand(() => _nav.GoBack());
    }

    public override void OnNavigatedTo(object parameter)
    {
        dynamic p = parameter;
        Info = p?.Info ?? "No info";
        Console.WriteLine($"Page3 received: {Info}");
    }
}

public class MyDialogViewModel : ViewModelBase
{
    public string Message { get; set; }
    private string _result;
    public IRelayCommand ConfirmCommand { get; }

    public MyDialogViewModel()
    {
        ConfirmCommand = new RelayCommand(() => Close("OK"));
    }

    public override void OnNavigatedTo(object parameter)
    {
        dynamic p = parameter;
        Message = p?.Message ?? "No message";
    }

    public void Close(string result)
    {
        _result = result;
        Application.Current.Windows[Application.Current.Windows.Count - 1].Close();
    }

    public override object OnNavigatedFrom() => _result;
}
```

## View XAML 範例（DataTemplate + ContentControl 自動映射）

App.xaml：

```xml
<Application x:Class="WpfApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="clr-namespace:WpfApp.ViewModels"
             xmlns:views="clr-namespace:WpfApp.Views"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <DataTemplate DataType="{x:Type vm:MainViewModel}">
            <views:MainWindow />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:Page1ViewModel}">
            <views:Page1View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:Page2ViewModel}">
            <views:Page2View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:Page3ViewModel}">
            <views:Page3View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:MyDialogViewModel}">
            <views:MyDialogView />
        </DataTemplate>
    </Application.Resources>
</Application>
```

MainWindow.xaml（作為容器，綁定 NavigationService.Current）：

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Demo" Height="450" Width="800">
    <Grid>
        <ContentControl Content="{Binding Current}" />
    </Grid>
</Window>
```

Page1View.xaml：

```xml
<UserControl x:Class="WpfApp.Views.Page1View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="Page1 (KeepAlive)" FontWeight="Bold"/>
        <Button Content="Go Page2" Command="{Binding NextCommand}" Margin="0,8,0,0"/>
    </StackPanel>
</UserControl>
```

Page2View.xaml：

```xml
<UserControl x:Class="WpfApp.Views.Page2View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="Page2 (KeepAlive)" FontWeight="Bold"/>
        <TextBlock Text="{Binding Info}" Margin="0,6,0,0"/>
        <Button Content="Go Page3" Command="{Binding GoToPage3Command}" Margin="0,6,0,0"/>
        <Button Content="Open Dialog" Command="{Binding OpenDialogCommand}" Margin="0,6,0,0"/>
    </StackPanel>
</UserControl>
```

Page3View.xaml：

```xml
<UserControl x:Class="WpfApp.Views.Page3View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="Page3" FontWeight="Bold"/>
        <TextBlock Text="{Binding Info}" Margin="0,6,0,0"/>
        <Button Content="Go Back" Command="{Binding GoBackCommand}" Margin="0,6,0,0"/>
    </StackPanel>
</UserControl>
```

MyDialogView.xaml：

```xml
<UserControl x:Class="WpfApp.Views.MyDialogView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="{Binding Message}" />
        <Button Content="Confirm" Command="{Binding ConfirmCommand}" Margin="0,6,0,0"/>
    </StackPanel>
</UserControl>
```

## Autofac 註冊與啟動（Program.cs 或 App.xaml.cs）

```csharp
using Autofac;
using System.Windows;

public partial class App : Application
{
    private IContainer _container;
    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);
        var builder = new ContainerBuilder();

        // Services
        builder.RegisterType<PageDataService>().As<IPageDataService>().SingleInstance();
        builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
        builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();

        // ViewModels
        builder.RegisterType<MainViewModel>();
        builder.RegisterType<Page1ViewModel>();
        builder.RegisterType<Page2ViewModel>();
        builder.RegisterType<Page3ViewModel>();
        builder.RegisterType<MyDialogViewModel>();

        _container = builder.Build();

        // 建立主視窗並綁定 MainViewModel（NavigationService 會管理 Current）
        var nav = _container.Resolve<INavigationService>() as NavigationService;
        var mainVm = _container.Resolve<MainViewModel>();
        var mainWindow = new MainWindow { DataContext = nav };
        mainWindow.Show();

        // 啟始導航到 Page1（或 MainViewModel 自己導航）
        nav.Navigate<Page1ViewModel>();
    }

    protected override void OnExit(ExitEventArgs e)
    {
        _container?.Dispose();
        base.OnExit(e);
    }
}
```

## 補充：在 VM 中取得 DialogService（建議透過 DI 直接注入）

```csharp
// 更好的做法：把 IDialogService 注入到需要的 ViewModel（例如 Page2ViewModel 的建構子）
builder.RegisterType<Page2ViewModel>().WithParameter(ResolvedParameter.ForNamed<IDialogService>("dialog"));
```

## 使用要點與最佳實務

- 每頁 `KeepAlive` 只控制「是否保留 ViewModel 與它的 Scope」。若 true，NavigationService 會從 RootScope 建立平行 scope，避免被上層 Dispose 覆蓋。
- KeepAlive 頁面在第二次 Navigate 不會被重建，若需要接收新參數請在 `OnNavigatedTo` 同時提供 `Update` / `Refresh` 方法由 NavigationService 或呼叫端手動觸發，以確保資料更新。
- 建議把資料存取、Cache 邏輯放在 `IPageDataService` 等 service，讓 ViewModel 保持輕量，方便測試與重用。
- Dialog 使用獨立 scope，ShowDialog 後立即 Dispose，避免資源遺留。
- 設計時衡量 KeepAlive 的「記憶體成本 vs 初始化成本」，僅對高頻且初始化昂貴的頁面啟用 KeepAlive。

## 結語

這套 Mix 模式給你：

- 完全掌控的生命週期與範圍管理（Autofac scope 安全）
- 彈性的 KeepAlive 設計（平行 scope）
- Dialog 與頁面一致的生命週期行為
- 把資料刷新責任交由 Service（可用 Cache 避免頻繁 EF 查詢）

---

# 把 NavigationService 補上更完整的 API

非常好，你的方向越來越接近一個完整可商用的導航框架了。
我幫你把整體設計重新整理成一個完整、清晰、模組化的版本，涵蓋你提到的所有功能，包括：

- 多層導航與歷史堆疊
- 頁面生命週期監控（Loaded / Unloaded / Dispose）
- 支援 KeepAlive 與 Autofac Scope 安全
- 參數傳入與返回
- Dialog 服務
- Refresh / Update Service
- NavigationService 的完整 API（含 RefreshCurrent、ClearHistory、FindViewModel、Remove 等）

## 架構概念

- 每個頁面（View + ViewModel）都由 Autofac 動態解析。
- NavigationService 維護一個 NavigationStack，並控制頁面生命週期。
- 每個 ViewModel 實作 `INavigationAware` 與 `ILifecycleAware`。
- 每個頁面可設定 KeepAlive 決定是否共用 Scope 或建立新 Scope。
- DialogService 使用相同的生命周期控制機制，可回傳結果。
- Refresh / Update 由獨立的 Service 提供。

## 主要介面定義

```csharp
public interface INavigationAware
{
    void OnNavigatedTo(object? parameter);
    void OnNavigatedFrom();
}

public interface ILifecycleAware : IDisposable
{
    void OnLoaded();
    void OnUnloaded();
}

public interface IKeepAlive
{
    bool KeepAlive { get; }
}

public interface IRefreshable
{
    void Refresh();
    void Update(object? parameter);
}
```

## ViewModel 基底類別

```csharp
public abstract class ViewModelBase : ObservableObject, INavigationAware, ILifecycleAware, IKeepAlive, IRefreshable
{
    public virtual bool KeepAlive => false;

    public virtual void OnNavigatedTo(object? parameter) { }
    public virtual void OnNavigatedFrom() { }
    public virtual void OnLoaded() { }
    public virtual void OnUnloaded() { }
    public virtual void Refresh() { }
    public virtual void Update(object? parameter) { }
    public virtual void Dispose() { }
}
```

## NavigationService 設計

```csharp
public interface INavigationService
{
    Task NavigateAsync<TViewModel>(object? parameter = null);
    Task GoBackAsync();
    Task ClearHistoryAsync();
    Task RefreshCurrentAsync();
    TViewModel? FindViewModel<TViewModel>() where TViewModel : class;
    void Remove<TViewModel>() where TViewModel : class;
    void DisposeCurrent();
}

public class NavigationService : INavigationService
{
    private readonly ILifetimeScope _rootScope;
    private readonly Stack<PageContext> _navigationStack = new();

    public NavigationService(ILifetimeScope rootScope)
    {
        _rootScope = rootScope;
    }

    public async Task NavigateAsync<TViewModel>(object? parameter = null)
    {
        var scope = _rootScope.BeginLifetimeScope();
        var vm = scope.Resolve<TViewModel>() as ViewModelBase 
                 ?? throw new InvalidOperationException($"Cannot resolve {typeof(TViewModel).Name}");
        var page = ResolveView(vm);

        if (_navigationStack.TryPeek(out var current))
        {
            current.ViewModel.OnNavigatedFrom();
            current.ViewModel.OnUnloaded();

            if (!current.ViewModel.KeepAlive)
                DisposeScope(current);
        }

        _navigationStack.Push(new PageContext(scope, page, vm));
        vm.OnNavigatedTo(parameter);
        vm.OnLoaded();

        await ShowPageAsync(page);
    }

    public async Task GoBackAsync()
    {
        if (_navigationStack.Count <= 1)
            return;

        var current = _navigationStack.Pop();
        current.ViewModel.OnNavigatedFrom();
        current.ViewModel.OnUnloaded();

        if (!current.ViewModel.KeepAlive)
            DisposeScope(current);

        var previous = _navigationStack.Peek();
        previous.ViewModel.OnNavigatedTo(null);
        previous.ViewModel.OnLoaded();

        await ShowPageAsync(previous.View);
    }

    public Task ClearHistoryAsync()
    {
        while (_navigationStack.Count > 1)
        {
            var ctx = _navigationStack.Pop();
            ctx.ViewModel.OnUnloaded();
            ctx.ViewModel.OnNavigatedFrom();
            DisposeScope(ctx);
        }
        return Task.CompletedTask;
    }

    public Task RefreshCurrentAsync()
    {
        if (_navigationStack.TryPeek(out var ctx))
            ctx.ViewModel.Refresh();
        return Task.CompletedTask;
    }

    public TViewModel? FindViewModel<TViewModel>() where TViewModel : class
    {
        return _navigationStack.Select(x => x.ViewModel).OfType<TViewModel>().FirstOrDefault();
    }

    public void Remove<TViewModel>() where TViewModel : class
    {
        var tempStack = new Stack<PageContext>(_navigationStack.Reverse());
        _navigationStack.Clear();

        foreach (var ctx in tempStack)
        {
            if (ctx.ViewModel is TViewModel)
            {
                ctx.ViewModel.OnUnloaded();
                ctx.ViewModel.OnNavigatedFrom();
                DisposeScope(ctx);
            }
            else
            {
                _navigationStack.Push(ctx);
            }
        }
    }

    public void DisposeCurrent()
    {
        if (_navigationStack.TryPeek(out var ctx))
        {
            ctx.ViewModel.Dispose();
            DisposeScope(ctx);
        }
    }

    private static void DisposeScope(PageContext context)
    {
        context.ViewModel.Dispose();
        context.Scope.Dispose();
    }

    private static UserControl ResolveView(ViewModelBase vm)
    {
        var viewTypeName = vm.GetType().Name.Replace("ViewModel", "View");
        var viewType = Assembly.GetExecutingAssembly().GetType(viewTypeName)
                       ?? throw new InvalidOperationException($"Cannot find view for {viewTypeName}");
        var view = Activator.CreateInstance(viewType) as UserControl 
                   ?? throw new InvalidOperationException($"Cannot create view {viewTypeName}");
        view.DataContext = vm;
        return view;
    }

    private Task ShowPageAsync(UserControl page)
    {
        // 可以在這裡實作切換邏輯，例如 ContentControl.Content = page
        return Task.CompletedTask;
    }

    private record PageContext(ILifetimeScope Scope, UserControl View, ViewModelBase ViewModel);
}
```

## DialogService 設計

```csharp
public interface IDialogService
{
    Task<TResult?> ShowDialogAsync<TViewModel, TResult>(object? parameter = null);
}

public class DialogService : IDialogService
{
    private readonly ILifetimeScope _rootScope;

    public DialogService(ILifetimeScope rootScope)
    {
        _rootScope = rootScope;
    }

    public async Task<TResult?> ShowDialogAsync<TViewModel, TResult>(object? parameter = null)
    {
        using var scope = _rootScope.BeginLifetimeScope();
        var vm = scope.Resolve<TViewModel>() as ViewModelBase 
                 ?? throw new InvalidOperationException($"Cannot resolve {typeof(TViewModel).Name}");
        var view = NavigationService.ResolveView(vm);

        vm.OnNavigatedTo(parameter);
        vm.OnLoaded();

        var dialog = new Window { Content = view, Owner = Application.Current.MainWindow };
        var result = await ShowDialogInternal<TResult>(dialog, vm);

        vm.OnUnloaded();
        vm.OnNavigatedFrom();
        return result;
    }

    private Task<TResult?> ShowDialogInternal<TResult>(Window dialog, ViewModelBase vm)
    {
        var tcs = new TaskCompletionSource<TResult?>();

        if (vm is IDialogResult<TResult> resultVm)
        {
            resultVm.RequestClose += (r) =>
            {
                dialog.Close();
                tcs.TrySetResult(r);
            };
        }

        dialog.ShowDialog();
        return tcs.Task;
    }
}

public interface IDialogResult<TResult>
{
    event Action<TResult?> RequestClose;
}
```

## Refresh / Update Service

```csharp
public interface IDataRefreshService
{
    Task<T> GetDataAsync<T>(object? parameter);
}

public class DataRefreshService : IDataRefreshService
{
    private readonly ICacheService _cacheService;
    private readonly IYourDbContext _db;

    public DataRefreshService(ICacheService cacheService, IYourDbContext db)
    {
        _cacheService = cacheService;
        _db = db;
    }

    public async Task<T> GetDataAsync<T>(object? parameter)
    {
        var key = parameter?.ToString() ?? typeof(T).Name;
        if (_cacheService.TryGet<T>(key, out var cached))
            return cached;

        var data = await QueryDatabaseAsync<T>(parameter);
        _cacheService.Set(key, data);
        return data;
    }

    private Task<T> QueryDatabaseAsync<T>(object? parameter)
    {
        // 模擬 EF6 資料查詢
        return Task.FromResult(Activator.CreateInstance<T>());
    }
}
```

## Autofac 註冊

```csharp
var builder = new ContainerBuilder();

// 註冊所有 ViewModel
builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly())
       .Where(t => t.Name.EndsWith("ViewModel"))
       .AsSelf()
       .InstancePerDependency();

// 註冊服務
builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();
builder.RegisterType<DataRefreshService>().As<IDataRefreshService>().SingleInstance();

// 建立容器
var container = builder.Build();
```

## 重點說明

1. **KeepAlive 頁面 Scope 平行化**：
    - KeepAlive 頁面使用 RootScope 建立，避免被上層 Dispose。
2. **多層導航支援**：
    - Stack 保存整個頁面歷史，可自由回退或清空。
3. **Refresh / Update 分離**：
    - Service 負責資料刷新，頁面只需呼叫即可。
4. **Dialog 統一生命週期管理**：
    - 透過同一套 ViewModel 生命周期與回傳機制實作。
5. **完整 API 支援**：
    - `RefreshCurrent`、`ClearHistory`、`FindViewModel`、`Remove`、`DisposeCurrent` 一應俱全。

這個版本是你可以直接延伸成「可重用框架」的等級。

---

# ===== 教學部分 =====

---

# WPF + MVVM 使用 Autofac 與 CommunityToolkit.Mvvm 實作多層頁面切換與 Dialog 服務

這個需求可以拆解成幾個重點：頁面切換管理、頁面生命週期、參數傳入與返回、以及 Dialog 服務。以下提供一個可操作的架構設計示例。

## 1. 建立 ViewModel 與 View 的基底

建立一個可監控頁面生命週期的基底 `ViewModelBase`。

```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public abstract class ViewModelBase : ObservableObject, IDisposable
{
    public virtual void OnLoaded() { }
    public virtual void OnUnloaded() { }
    public virtual void OnNavigatedTo(object parameter) { }
    public virtual object OnNavigatedFrom() { return null; }
    public virtual void Dispose() { }
}
```

## 2. 建立頁面導航服務

```csharp
public interface INavigationService
{
    void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase;
    void GoBack();
}

public class NavigationService : INavigationService
{
    private readonly ILifetimeScope _scope;
    private readonly Stack<ViewModelBase> _history = new Stack<ViewModelBase>();

    public ViewModelBase Current { get; private set; }

    public NavigationService(ILifetimeScope scope)
    {
        _scope = scope;
    }

    public void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase
    {
        Current?.OnUnloaded();
        if(Current != null)
            _history.Push(Current);

        Current = _scope.Resolve<TViewModel>();
        Current.OnNavigatedTo(parameter);
        Current.OnLoaded();
    }

    public void GoBack()
    {
        if(_history.Count == 0) return;
        Current?.OnUnloaded();
        Current.Dispose();

        Current = _history.Pop();
        Current.OnLoaded();
    }
}
```

## 3. Autofac 註冊

```csharp
var builder = new ContainerBuilder();
builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
builder.RegisterType<MainViewModel>();
builder.RegisterType<Page1ViewModel>();
builder.RegisterType<Page2ViewModel>();
builder.RegisterType<Page3ViewModel>();
var container = builder.Build();
```

## 4. 在 View 中綁定 ViewModel

使用 `ContentControl` 作為頁面容器，綁定 `Current`：

```xml
<ContentControl Content="{Binding Current}" />
```

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = container.Resolve<MainViewModel>();
    }
}
```

## 5. 支援 Dialog 服務

```csharp
public interface IDialogService
{
    Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase;
}

public class DialogService : IDialogService
{
    private readonly ILifetimeScope _scope;

    public DialogService(ILifetimeScope scope)
    {
        _scope = scope;
    }

    public async Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase
    {
        var vm = _scope.Resolve<TViewModel>();
        vm.OnNavigatedTo(parameter);

        var window = new Window
        {
            Content = vm,
            SizeToContent = SizeToContent.WidthAndHeight,
            WindowStartupLocation = WindowStartupLocation.CenterOwner,
            Owner = Application.Current.MainWindow
        };

        window.ShowDialog();

        var result = (TResult)vm.OnNavigatedFrom();
        vm.Dispose();
        return await Task.FromResult(result);
    }
}
```

## 6. 頁面生命週期與參數管理

- `OnLoaded` / `OnUnloaded`：頁面顯示或隱藏時觸發
- `OnNavigatedTo(object parameter)`：頁面初始化時接收參數
- `OnNavigatedFrom()`：返回時傳回結果
- `Dispose()`：釋放資源或取消訂閱

## 7. 使用範例

```csharp
// 導航到下一頁
_navigationService.Navigate<Page2ViewModel>(new { Id = 123 });

// 打開 Dialog
var result = await _dialogService.ShowDialogAsync<MyDialogViewModel, MyResultType>(new { Name = "Test" });
```

## 這個架構可以支援

- 多層頁面切換（History Stack）
- 頁面生命週期監控（Loaded / Unloaded / Dispose）
- 複雜參數傳入與返回
- Dialog 服務管理生命週期和回傳資料
- MVVM 與 DI（Autofac）整合

---

# WPF + MVVM 完整 3 層頁面切換與 Dialog 範例

下面我整理出一個可運行的簡化專案結構，展示 3 層頁面切換、生命週期管理，以及 Dialog 服務與參數傳入/返回。

## 1. 專案結構

- ViewModels
    - ViewModelBase.cs
    - MainViewModel.cs
    - Page1ViewModel.cs
    - Page2ViewModel.cs
    - Page3ViewModel.cs
    - MyDialogViewModel.cs
- Views
    - MainWindow.xaml
    - Page1View.xaml
    - Page2View.xaml
    - Page3View.xaml
    - MyDialogView.xaml
- Services
    - INavigationService.cs / NavigationService.cs
    - IDialogService.cs / DialogService.cs
- App.xaml / App.xaml.cs
- Program.cs (Autofac 註冊與啟動)

## 2. ViewModel 基底

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using System;

public abstract class ViewModelBase : ObservableObject, IDisposable
{
    public virtual void OnLoaded() { }
    public virtual void OnUnloaded() { }
    public virtual void OnNavigatedTo(object parameter) { }
    public virtual object OnNavigatedFrom() { return null; }
    public virtual void Dispose() { }
}
```

## 3. Navigation 服務

```csharp
using Autofac;
using System.Collections.Generic;

public interface INavigationService
{
    void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase;
    void GoBack();
    ViewModelBase Current { get; }
}

public class NavigationService : INavigationService
{
    private readonly ILifetimeScope _scope;
    private readonly Stack<ViewModelBase> _history = new Stack<ViewModelBase>();

    public ViewModelBase Current { get; private set; }

    public NavigationService(ILifetimeScope scope)
    {
        _scope = scope;
    }

    public void Navigate<TViewModel>(object parameter = null) where TViewModel : ViewModelBase
    {
        Current?.OnUnloaded();
        if (Current != null)
            _history.Push(Current);

        Current = _scope.Resolve<TViewModel>();
        Current.OnNavigatedTo(parameter);
        Current.OnLoaded();
    }

    public void GoBack()
    {
        if (_history.Count == 0) return;
        Current?.OnUnloaded();
        Current.Dispose();

        Current = _history.Pop();
        Current.OnLoaded();
    }
}
```

## 4. Dialog 服務

```csharp
using Autofac;
using System.Threading.Tasks;
using System.Windows;

public interface IDialogService
{
    Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase;
}

public class DialogService : IDialogService
{
    private readonly ILifetimeScope _scope;

    public DialogService(ILifetimeScope scope)
    {
        _scope = scope;
    }

    public async Task<TResult> ShowDialogAsync<TViewModel, TResult>(object parameter = null) where TViewModel : ViewModelBase
    {
        var vm = _scope.Resolve<TViewModel>();
        vm.OnNavigatedTo(parameter);

        var window = new Window
        {
            Content = vm,
            SizeToContent = SizeToContent.WidthAndHeight,
            WindowStartupLocation = WindowStartupLocation.CenterOwner,
            Owner = Application.Current.MainWindow
        };

        window.ShowDialog();

        var result = (TResult)vm.OnNavigatedFrom();
        vm.Dispose();
        return await Task.FromResult(result);
    }
}
```

## 5. 範例 ViewModels

```csharp
public class MainViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    public MainViewModel(INavigationService nav)
    {
        _nav = nav;
        _nav.Navigate<Page1ViewModel>();
    }

    public void GoToPage1() => _nav.Navigate<Page1ViewModel>();
    public void GoToPage2() => _nav.Navigate<Page2ViewModel>();
}

public class Page1ViewModel : ViewModelBase
{
    public override void OnLoaded() => Console.WriteLine("Page1 Loaded");
    public override void OnUnloaded() => Console.WriteLine("Page1 Unloaded");
}

public class Page2ViewModel : ViewModelBase
{
    private readonly IDialogService _dialog;
    public Page2ViewModel(IDialogService dialog)
    {
        _dialog = dialog;
    }

    public async void OpenDialog()
    {
        var result = await _dialog.ShowDialogAsync<MyDialogViewModel, string>(new { Message = "Hello" });
        Console.WriteLine($"Dialog returned: {result}");
    }
}

public class Page3ViewModel : ViewModelBase { }

public class MyDialogViewModel : ViewModelBase
{
    public string Message { get; set; }
    private string _result;
    public override void OnNavigatedTo(object parameter)
    {
        dynamic p = parameter;
        Message = p.Message;
    }
    public void CloseDialog(string result)
    {
        _result = result;
        Application.Current.Windows[Application.Current.Windows.Count - 1].Close();
    }
    public override object OnNavigatedFrom() => _result;
}
```

## 6. XAML 範例

MainWindow.xaml：

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Demo" Height="350" Width="525">
    <ContentControl Content="{Binding Current}" />
</Window>
```

Page1View.xaml：

```xml
<UserControl x:Class="WpfApp.Page1View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel>
        <TextBlock Text="Page 1" />
        <Button Content="Next" Command="{Binding NavigateToPage2Command}" />
    </StackPanel>
</UserControl>
```

DialogView.xaml：

```xml
<UserControl x:Class="WpfApp.MyDialogView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="{Binding Message}" />
        <Button Content="OK" Command="{Binding CloseDialogCommand}" />
    </StackPanel>
</UserControl>
```

## 7. Autofac 註冊與啟動

```csharp
var builder = new ContainerBuilder();
builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();

builder.RegisterType<MainViewModel>();
builder.RegisterType<Page1ViewModel>();
builder.RegisterType<Page2ViewModel>();
builder.RegisterType<Page3ViewModel>();
builder.RegisterType<MyDialogViewModel>();

var container = builder.Build();
Application.Current.MainWindow.DataContext = container.Resolve<MainViewModel>();
```

## 這個範例展示

- 3 層頁面切換（Main → Page1 → Page2 → Page3）
- Page 生命週期管理（Loaded / Unloaded / Dispose）
- Dialog 服務管理參數與返回值
- 使用 Autofac 注入依賴
- 可擴充到任意深度與複雜資料結構

---

# 多層頁面與 Dialog 完整互動範例

以下展示 Page2 → Page3 → Dialog 的完整流程，包括參數傳入、返回值，以及回退處理。

## 1. Page2ViewModel

```csharp
using CommunityToolkit.Mvvm.Input;

public class Page2ViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    private readonly IDialogService _dialog;

    public IRelayCommand GoToPage3Command { get; }
    public IRelayCommand OpenDialogCommand { get; }

    public Page2ViewModel(INavigationService nav, IDialogService dialog)
    {
        _nav = nav;
        _dialog = dialog;

        GoToPage3Command = new RelayCommand(GoToPage3);
        OpenDialogCommand = new RelayCommand(OpenDialog);
    }

    private void GoToPage3()
    {
        _nav.Navigate<Page3ViewModel>(new { Info = "Hello from Page2" });
    }

    private async void OpenDialog()
    {
        var result = await _dialog.ShowDialogAsync<MyDialogViewModel, string>(new { Message = "Please confirm" });
        Console.WriteLine($"Dialog returned: {result}");
    }

    public override void OnLoaded() => Console.WriteLine("Page2 Loaded");
    public override void OnUnloaded() => Console.WriteLine("Page2 Unloaded");
}
```

## 2. Page3ViewModel

```csharp
using CommunityToolkit.Mvvm.Input;

public class Page3ViewModel : ViewModelBase
{
    private readonly INavigationService _nav;
    public string Info { get; private set; }

    public IRelayCommand GoBackCommand { get; }

    public Page3ViewModel(INavigationService nav)
    {
        _nav = nav;
        GoBackCommand = new RelayCommand(() => _nav.GoBack());
    }

    public override void OnNavigatedTo(object parameter)
    {
        dynamic p = parameter;
        Info = p?.Info ?? "No info";
        Console.WriteLine($"Page3 received parameter: {Info}");
    }

    public override void OnLoaded() => Console.WriteLine("Page3 Loaded");
    public override void OnUnloaded() => Console.WriteLine("Page3 Unloaded");
}
```

## 3. MyDialogViewModel

```csharp
using CommunityToolkit.Mvvm.Input;
using System.Windows;

public class MyDialogViewModel : ViewModelBase
{
    public string Message { get; set; }
    public IRelayCommand CloseDialogCommand { get; }

    private string _result;

    public MyDialogViewModel()
    {
        CloseDialogCommand = new RelayCommand(() => CloseDialog("Confirmed"));
    }

    public override void OnNavigatedTo(object parameter)
    {
        dynamic p = parameter;
        Message = p?.Message ?? "No message";
    }

    public void CloseDialog(string result)
    {
        _result = result;
        Application.Current.Windows[Application.Current.Windows.Count - 1].Close();
    }

    public override object OnNavigatedFrom() => _result;
}
```

## 4. Page3View.xaml

```xml
<UserControl x:Class="WpfApp.Page3View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="{Binding Info}" FontWeight="Bold" />
        <Button Content="Go Back" Command="{Binding GoBackCommand}" Margin="0,10,0,0"/>
    </StackPanel>
</UserControl>
```

## 5. Page2View.xaml

```xml
<UserControl x:Class="WpfApp.Page2View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="10">
        <TextBlock Text="Page 2" FontWeight="Bold" />
        <Button Content="Go to Page3" Command="{Binding GoToPage3Command}" Margin="0,5,0,0"/>
        <Button Content="Open Dialog" Command="{Binding OpenDialogCommand}" Margin="0,5,0,0"/>
    </StackPanel>
</UserControl>
```

## 6. 使用流程

1. MainViewModel → Page1ViewModel
2. Page1 → Page2 (使用 `_nav.Navigate<Page2ViewModel>(parameter)`)
3. Page2 → Page3 (帶參數 `Info`)
4. Page2 打開 Dialog，傳入 `Message`，回傳 `string` 結果
5. Page3 可 GoBack → Page2
6. 每個頁面觸發 `OnLoaded`, `OnUnloaded`, `Dispose` 生命週期

## 7. Autofac 註冊補充

```csharp
builder.RegisterType<Page1ViewModel>();
builder.RegisterType<Page2ViewModel>();
builder.RegisterType<Page3ViewModel>();
builder.RegisterType<MyDialogViewModel>();
builder.RegisterType<MainViewModel>();
builder.RegisterType<NavigationService>().As<INavigationService>().SingleInstance();
builder.RegisterType<DialogService>().As<IDialogService>().SingleInstance();
```

## 這個範例可以

- 支援至少 3 層頁面導航
- 頁面可接收複雜參數並返回結果
- Dialog 可傳入訊息並返回結果
- 每個 ViewModel 都能監控 Loaded / Unloaded / Dispose
- 支援 History Stack 回退

整個流程可直接跑 WPF，完全符合 MVVM + Autofac + CommunityToolkit.Mvvm 的架構。

---

# WPF 自動綁定 ViewModel → View 範例（DataTemplate 方式）

透過 DataTemplate，可以讓 `ContentControl` 自動對應 ViewModel 到對應的 View，完全不需要手動切換 UserControl。這對多層頁面切換特別乾淨。

## 1. App.xaml 設定 DataTemplate

```xml
<Application x:Class="WpfApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="clr-namespace:WpfApp.ViewModels"
             xmlns:views="clr-namespace:WpfApp.Views"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <DataTemplate DataType="{x:Type vm:Page1ViewModel}">
            <views:Page1View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:Page2ViewModel}">
            <views:Page2View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:Page3ViewModel}">
            <views:Page3View />
        </DataTemplate>
        <DataTemplate DataType="{x:Type vm:MyDialogViewModel}">
            <views:MyDialogView />
        </DataTemplate>
    </Application.Resources>
</Application>
```

## 2. MainWindow.xaml 綁定 ContentControl

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Demo" Height="350" Width="525">
    <ContentControl Content="{Binding Current}" />
</Window>
```

- `ContentControl` 的 `Content` 綁定到 `NavigationService.Current`
- 當 `Current` 是 `Page2ViewModel` 時，自動套用 DataTemplate → `Page2View`
- 無需在程式碼中手動切換 UserControl

## 3. ViewModel 不需知道 View

- 每個 ViewModel 僅負責資料、命令與生命週期
- View 的外觀與 XAML 完全解耦
- 多層導航與 Dialog 仍保持完全 MVVM

## 4. Dialog 也適用 DataTemplate

```csharp
var window = new Window
{
    Content = vm, // vm 是 ViewModel，對應 DataTemplate 的 View 會自動套用
    SizeToContent = SizeToContent.WidthAndHeight,
    WindowStartupLocation = WindowStartupLocation.CenterOwner,
    Owner = Application.Current.MainWindow
};
window.ShowDialog();
```

- Dialog 只需要傳入 ViewModel
- DataTemplate 自動渲染 View
- 回傳值仍由 `OnNavigatedFrom()` 處理

## 這種方式優點

- View 與 ViewModel 完全解耦
- ContentControl 可自動切換多層頁面
- Dialog 與頁面共用相同機制
- 易於擴充至更多頁面或複雜參數

整個 WPF MVVM 架構乾淨、易維護，也完全符合你最初需求的多層導航 + 生命週期管理 + Dialog 參數回傳。

---

# WPF 多層頁面與 Dialog 流程圖（概念示意）

下面用文字方式描述流程圖的結構，方便理解 Navigation Stack、頁面生命週期與 Dialog 返回。你可以照這個邏輯畫圖或用 PlantUML/Draw.io 繪製。

```text
[MainViewModel]
        │
        ▼ Navigate
[Page1ViewModel] ── OnLoaded
        │
        ▼ Navigate
[Page2ViewModel] ── OnLoaded
        │          │
        │          ▼ Open Dialog
        │       [MyDialogViewModel] ── OnNavigatedTo
        │          │
        │          ▼ CloseDialog
        │       OnNavigatedFrom → 返回結果給 Page2ViewModel
        │
        ▼ Navigate
[Page3ViewModel] ── OnNavigatedTo(parameter)
        │
        ▼ GoBack
[Page2ViewModel] ── OnLoaded
        │
        ▼ GoBack
[Page1ViewModel] ── OnLoaded
        │
        ▼ GoBack
[MainViewModel] ── OnLoaded
```

## 流程說明

- 每次 `Navigate<TViewModel>(parameter)`：
    - 當前頁面觸發 `OnUnloaded`
    - 新頁面觸發 `OnNavigatedTo(parameter)` → `OnLoaded`
    - 原頁面推入 Stack
- `GoBack()`：
    - 當前頁面 `OnUnloaded` → `Dispose`
    - 從 Stack 彈出前一頁 → `OnLoaded`
- Dialog：
    - 傳入 ViewModel → 自動套用 DataTemplate
    - `OnNavigatedTo(parameter)` 處理初始化參數
    - 關閉時 `OnNavigatedFrom()` 返回結果
- 整個流程保持 MVVM + DI 解耦
- 任何頁面都可傳入複雜物件作參數，也可回傳結果

這個文字流程圖清楚展現：

1. Navigation Stack 的多層結構
2. 頁面 Loaded / Unloaded / Dispose 的生命週期
3. Dialog 傳入/返回的交互

---

# 與 Prism KeepAlive 比較

這是一個非常核心的設計問題：**頁面或 ViewModel 的生命週期管理策略**，決定了應用的記憶體使用、狀態保持以及複雜度。咱們可以對比你提到的兩種方式：

## 1. PRISM 的頁面保留（KeepAlive）

**特點**：

- Page 或 ViewModel 會被 **緩存在 Region 或 NavigationService 中**
- 頁面離開畫面時不會釋放，而是保留狀態
- 再次導航回來時可以直接使用原來的實例，狀態保持

**優點**：

- 狀態保持自然：使用者返回頁面時，表單資料、滾動位置、Tab 選擇等仍然存在
- 對於 **頻繁切換的頁面**，可以避免反覆建立與初始化，提高性能
- 較適合 **大型應用的主導航結構**（例如 Dashboard → Detail → SubDetail）

**缺點**：

- 記憶體使用量增加，尤其多層頁面、或資料量大時
- 生命週期管理複雜：OnNavigatedFrom / OnNavigatedTo 事件需要更小心，避免狀態污染
- 若頁面或 ViewModel 綁定大量事件或訂閱，容易出現 **內存洩漏**

**適用場景**：

- 頁面切換頻繁且需要保留使用者狀態
- 大型管理系統、表單流程、Dashboard
- 狀態重建成本高，或初始化過程耗時

## 2. 用完就丟（我之前的示例）

**特點**：

- 每次 Navigate 都 **重新建立 ViewModel / View**
- 離開頁面時觸發 Dispose，釋放所有資源
- History Stack 可選擇只保留 ViewModel 的少量參數，而非整個物件

**優點**：

- 記憶體使用量較低，適合大量頁面或長流程
- 生命週期簡單：Loaded → Unloaded → Dispose，事件和訂閱容易管理
- 容易避免內存洩漏

**缺點**：

- 返回上一頁會重新初始化頁面，需要重新綁定資料
- 若初始化耗時，會出現延遲或重新讀取資料的成本
- 狀態保存需要額外實作，例如透過參數或中央狀態管理（State Service）

**適用場景**：

- 資料量大或頁面多，不希望長時間占用記憶體
- 單次操作流程或 Wizard 式頁面
- 資料初始化成本低，或能透過參數快速重建狀態

## 3. 設計複雜度比較

|特性|PRISM KeepAlive|用完就丟|
|---|---|---|
|記憶體管理|高需求，需要注意洩漏|低需求，Dispose 釋放資源|
|狀態保持|自動保持，簡單對使用者|需額外保存狀態|
|初始化|一次初始化，多次使用|每次導航初始化|
|開發複雜度|中高，需要處理事件/訂閱|低，生命週期清晰|
|適合場景|Dashboard、多層導航、重複訪問|Wizard、單次流程、長頁面列表|

### 總結

- **PRISM KeepAlive**：適合需要狀態保持與高使用者互動的多層頁面，但需要小心內存管理
- **用完就丟**：適合資源敏感、狀態可重建、頁面多且長的流程，開發維護相對簡單

簡單比喻：

- KeepAlive = **冰箱裡的食材**，保留現狀方便下次使用，但佔地方
- 用完就丟 = **吃完即洗碗**，乾淨整潔，但下次要重煮

---

# KeepAlive 與 Autofac Scope 分離

認真分析，這種設計雖然解決了你目前的需求，但從 **架構設計、維護性、性能與複雜度** 的角度，有幾個優缺點值得思考。

## 1. 優點

- **KeepAlive 彈性**：可以針對每層頁面設定是否保留狀態，非常直觀
- **Autofac Scope 安全**：每頁一個 Scope，Dispose 管理清楚，不會意外釋放 KeepAlive 頁面
- **生命週期清晰**：OnLoaded / OnUnloaded / Dispose 流程明確
- **適用多層導航與 Dialog**：歷史 Stack + Scope 保證狀態與依賴完整

## 2. 潛在問題

### 2.1 記憶體累積風險

- 每個 KeepAlive 頁面都會保留 Scope 與所有依賴
- 多層導航、頻繁跳轉 → Stack 可能快速增加
- 複雜頁面或大量資料會導致記憶體消耗高

### 2.2 導航與 Scope 管理複雜

- 需要自己維護 Stack、Scope 與 Dispose 流程
- 當頁面間有事件訂閱、Timer、資料連線時，容易漏掉 Dispose 或重複釋放
- 需要額外規範：哪個頁面 Scope 需要跟隨 KeepAlive，哪個不需要

### 2.3 測試與維護成本高

- 每個頁面可能有不同 KeepAlive 設定
- 測試不同層級組合可能出現難以預期的資源保留/釋放問題
- 對新開發者來說，理解 Stack + Scope + KeepAlive 的互動需要時間

## 3. 是否「好的設計」？

- **適合中大型應用，且頁面有明顯狀態保留需求**
    - 例如 Dashboard、多 Tab、多步驟 Wizard
- **不適合頁面多、狀態可重建、或流程一次性**
    - 這時 KeepAlive + Scope 分層會增加不必要的複雜度

> 核心原則：**狀態保留的收益是否大於架構複雜度與資源消耗成本**

## 4. 替代方案思考

1. **統一 Scope + 狀態管理服務（StateService）**

- 所有頁面都是「用完就丟」
- 需要保留狀態的資料放到 StateService
- 減少 Scope 複雜度和記憶體累積

2. **只對少數頁面 KeepAlive**

- 不必每層都搞 Scope
- 保留核心頁面，其他頁面用完即丟
- 減少歷史 Stack 複雜度

3. **短生命頁面 + 快速初始化**

- 如果頁面初始化很快，完全用「用完就丟」的方式
- 避免 Scope 與 KeepAlive 的管理開銷

### 總結

這個設計「可行、靈活、安全」，但是**有代價**：

- 架構複雜度高
- 記憶體消耗不可忽視
- 測試維護成本上升

適合**必須保留中間頁面狀態**的情境，不適合單純列表或一次性流程。

---

# 典型的 **頻繁操作、多層導航 + 資料存取** 的場景

## 1. 你的特性分析

- **三大功能區塊**，每個區塊有多個小頁
- **深度 3~4 層**，切換頻繁
- **大部分頁面會存取 EF6 資料庫**
- **部分操作可以透過 Cache**
- **使用者可能在同一區塊頻繁切換小頁面**

## 2. KeepAlive 的考量

KeepAlive 的核心是「保留頁面狀態、避免重新初始化」，對你的場景影響如下：

### 2.1 頁面初始化成本

- 如果 **頁面初始化涉及 EF6 查詢、複雜 UI 綁定或計算**，重新建立可能很耗時
- KeepAlive 可以減少重新查詢或重新載入 UI 的延遲

### 2.2 記憶體使用

- 每個 KeepAlive 頁面都會保留 Scope、ViewModel、資料
- 頁面多、層數深 → Stack 快速增長 → 記憶體消耗可能可觀
- 如果有 Cache，可部分替代 KeepAlive，讓頁面用完就丟，返回時直接從 Cache 載入資料

### 2.3 頁面切換頻率

- 頻繁切換同一區塊內的頁面 → KeepAlive 的好處更明顯
- 例如 Tab 切換、Wizard 內多步驟 → 保留狀態可以提升 UX

### 2.4 狀態保存需求

- 表單、輸入資料、選擇狀態 → 如果需要保留，用 KeepAlive
- 如果頁面只是查詢列表或簡單展示 → 用完就丟更簡單

## 3. 建議策略

### 3.1 分層策略

- **第 1 層（區塊主頁）**
    - KeepAlive 建議 **true**
    - 優點：快速返回區塊主頁，不用重載 EF6 查詢或 UI
- **第 2 層（小頁/子頁）**
    - 根據使用頻率與初始化成本決定
    - 高頻操作 + 初始化慢 → KeepAlive = true
    - 低頻、簡單頁面 → KeepAlive = false
- **第 3~4 層（深層詳細頁 / 對話操作頁）**
    - 大部分可用 **用完就丟**
    - 若使用者可能頻繁返回且初始化成本高 → KeepAlive = true

### 3.2 Cache + KeepAlive 混合

- KeepAlive = true → 保留 ViewModel 狀態
- Cache = true → 即使用完就丟，資料也能快速重新載入
- 這樣可以 **減少記憶體累積**，又保持操作效率

### 3.3 Scope 管理建議

- KeepAlive 頁面用 **獨立 Scope 或全局 Scope**
- 非 KeepAlive 頁面用 **局部 Scope** → Dispose 後釋放資源

## 4. 實務建議

- **高頻操作 + 大量 EF6 初始化** → KeepAlive
- **低頻頁面或一次性操作** → 用完就丟 + Cache
- **深層頁面** → 儘量用用完就丟，避免 Stack 累積
- **每個區塊可建立單獨 NavigationService / Scope** → 易於管理 KeepAlive 與 Dispose

### 小結

KeepAlive 並不是「越多越好」，應該根據 **初始化成本、切換頻率、狀態保存需求、記憶體消耗** 綜合決策。
你目前場景中：

- **區塊主頁** → 強烈建議 KeepAlive
- **子頁 / 深層頁** → 根據需求選擇
- **Cache** 可以部分替代 KeepAlive，減少記憶體壓力

---

# 與 Prism 比較

PRISM 有時頁面在第 2 次呼叫之後，OnNavigatedTo 和 OnNavigatedFrom 似乎就失效的情況，其實是 PRISM 在 **KeepAlive 與 Navigation 生命週期** 的常見坑。我來幫你分析原因，以及實務上怎麼處理。

## 1. 產生原因

### 1.1 KeepAlive 與 ViewModel 實例共用

在 PRISM 中：

- 如果你的頁面（Region）設定 `KeepAlive = true`，PRISM **只會保留第一個實例**
- 再次 Navigate 到同一個頁面時：
    - PRISM 不會重新建立 View / ViewModel
    - `OnNavigatedTo` 可能只在第一次呼叫，第二次不再觸發
    - `OnNavigatedFrom` 也可能不會再次呼叫

這就是你看到「第 2 次之後失效」的原因。

### 1.2 導航參數不觸發事件

PRISM 的 `OnNavigatedTo(INavigationParameters parameters)` 只在：

- 新建實例的時候觸發
- 或明確使用 `KeepAlive = false` 時觸發

如果頁面是 KeepAlive，PRISM 預設不會呼叫 `OnNavigatedTo` 第二次，即使你傳了新參數。

## 2. 處理方式

### 2.1 使用 `IConfirmNavigationRequest` 或手動刷新

PRISM 提供 `IConfirmNavigationRequest` 介面：

```csharp
public class Page2ViewModel : BindableBase, INavigationAware, IConfirmNavigationRequest
{
    public void OnNavigatedTo(INavigationParameters parameters)
    {
        // 只會第一次呼叫
    }

    public void OnNavigatedFrom(INavigationParameters parameters)
    {
    }

    public void ConfirmNavigationRequest(NavigationContext context, Action<bool> continuationCallback)
    {
        // 強制每次導航確認
        continuationCallback(true);
    }
}
```

> 這可以確保 PRISM 在 KeepAlive 頁面導航時有回調，但仍需注意 `OnNavigatedTo` 不一定自動帶新參數。

### 2.2 手動處理參數更新

對於 KeepAlive 頁面，**建議在 NavigationService 或 ViewModel 加一個 `Update` 方法**，每次 Navigate 前呼叫：

```csharp
public void Update(INavigationParameters parameters)
{
    // 每次 Navigate 都手動刷新資料或狀態
}
```

PRISM KeepAlive 的頁面不會自動重新觸發 OnNavigatedTo，手動呼叫是最穩定方式。

### 2.3 避免 KeepAlive 或依需求選擇

如果頁面：

- 每次都需要刷新狀態或資料
- 或參數每次不同

那麼建議 **KeepAlive = false**，讓 PRISM 每次都重新建立 ViewModel → OnNavigatedTo 保證觸發

### 2.4 與 Cache 搭配

- KeepAlive = false → 每次 Navigate 都會重新建立 ViewModel，但你可以用 **Cache 服務** 保留資料，避免 EF6 重查
- KeepAlive = true → ViewModel 保留，但 Navigation 參數可能需要手動刷新

## 3. 實務建議

1. **頻繁切換 + 需要保留頁面狀態** → KeepAlive = true，資料刷新用 Update()
2. **每次都要重新讀資料或不同參數** → KeepAlive = false，配合 Cache 避免 EF6 重新查詢
3. **深層頁面或一次性操作** → KeepAlive = false

> 簡單說：KeepAlive = true → 生命週期事件不保證每次觸發，需要手動管理；KeepAlive = false → 事件穩定，但每次重新建立。

---

# Autofac + 自訂 NavigationService 的方案

了解，你現在完全轉向 **Autofac + 自訂 NavigationService** 的方案，那就不必再受 PRISM KeepAlive 的那些坑限制。其實你之前提的「第 2 層 KeepAlive / 第 1 層 Dispose」的疑慮，就是 PRISM 無法完全自由控制生命週期的典型案例。

現在可以完全掌控每個頁面的 Scope、Dispose 與參數傳遞，優勢非常明顯：

## 1. 生命週期完全可控

- **每個頁面可選擇 KeepAlive 或用完就丟**
- **每頁 Scope 可獨立**，上層 Dispose 不影響 KeepAlive 頁面
- **OnLoaded / OnUnloaded / Dispose / OnNavigatedTo / OnNavigatedFrom** 都可以自己定義觸發時機
- 不再被 PRISM 自動緩存邏輯束縛

## 2. 參數與返回值更彈性

- Navigate 時可帶任何複雜參數（物件、DTO、Func、Action）
- Dialog 也可使用相同 Scope / NavigationService 架構，回傳結果簡單
- 可以設計 Update() 或 Refresh() 方法，讓 KeepAlive 頁面在導航時自動刷新狀態

## 3. 高頻切換頁面的策略建議

- **主頁面 / 區塊頁面** → KeepAlive = true，Scope 獨立
- **子頁 / 深層頁** → 大部分用完就丟，Scope 局部
- **高頻操作但初始化耗時的子頁** → KeepAlive = true + 更新資料機制
- **Cache 配合用完就丟** → 保留資料而非整個 ViewModel

這樣可以兼顧：

- 記憶體消耗（不用全 KeepAlive）
- 使用者體驗（高頻操作頁面快速返回）
- EF6 資料庫效率（避免頻繁查詢）

## 4. 設計複雜度與維護性

- **Scope + KeepAlive + Stack** → 複雜度比 PRISM 高，但完全可控
- **好處**：邏輯透明，不會出現 PRISM 那種事件失效或參數不刷新的問題
- **建議**：規範每個頁面 Scope 管理，明確 KeepAlive 屬性，必要時加入 Refresh()

總結：

- 你目前改用 Autofac + 自訂 NavigationService 是個明智選擇
- 可以把 PRISM 的坑全部避開，生命週期、Scope、參數控制都完全掌握
- 關鍵是設計好 **KeepAlive / Scope / Cache / Refresh 機制**，尤其在多層深度與高頻切換場景

---

# 使用 KeepAlive 之後，是不是 SCOPE 就不是巢狀了？會違背 AUTOFAC 設計理念嗎？

你的疑問非常尖銳，這涉及 **Autofac Scope 的原理**與「KeepAlive 與巢狀 Scope」之間的設計取捨。我仔細拆解給你看。

## 1. Autofac Scope 原則

- **Scope 是巢狀的**：每個 `BeginLifetimeScope()` 都是父 Scope 的子 Scope
- **Dispose 時間**：子 Scope Dispose → 只會清掉子 Scope 下的物件，不會影響父 Scope
- **用途**：
    - 控制物件生命週期
    - 支援依賴隔離與自動 Dispose

## 2. KeepAlive 與 Scope 的衝突

假設你的導航邏輯：

- 每個頁面 KeepAlive → 頁面不 Dispose
- 原本的思路是「每頁建立獨立 Scope」

### 2.1 巢狀 Scope 模式

```text
RootScope
  └─ Page1 Scope
       └─ Page2 Scope (KeepAlive)
            └─ Page3 Scope
```

- Page1 Dispose → Page1 Scope Dispose → Page2 Scope 如果是 Page1 的子 Scope，也會被 Dispose
- **問題**：KeepAlive 第 2 層頁面會被意外 Dispose

### 2.2 平行 Scope 模式（非巢狀）

```text
RootScope
  ├─ Page1 Scope
  ├─ Page2 Scope (KeepAlive)
  └─ Page3 Scope
```

- 每頁 Scope 都直接從 RootScope 建立
- KeepAlive 頁面不受上層 Dispose 影響
- 生命週期可完全控制

## 3. 是否違背 Autofac 設計理念？

**技術上不違背**：

- Autofac 設計原則是「Scope 只是一個生命週期管理單位」
- 你完全可以建立 **平行 Scope**，只是失去了巢狀依賴隔離的特性
- Autofac 沒有強制要求 Scope 一定要巢狀，只是巢狀可以讓子 Scope 自動清理依賴物件

**代價**：

- 失去巢狀 Scope 的自動管理優勢
- KeepAlive Scope 必須手動 Dispose
- 如果多層頁面頻繁切換，平行 Scope 數量增加，可能增加記憶體壓力

## 4. 實務設計建議

1. **KeepAlive 頁面用平行 Scope**
    - 不受上層 Dispose 影響
    - 可以自由控制 OnLoaded/OnUnloaded/Dispose
2. **非 KeepAlive 頁面用巢狀 Scope**
    - 自動跟隨上層 Dispose
    - 方便資源管理
3. **Mix 模式**：
    - RootScope → 長壽命物件（單例 / Service）
    - 每個 KeepAlive 頁面 Scope 直接從 RootScope 建立
    - 每個一次性頁面 Scope 作為上一頁子 Scope

> 簡單比喻：
> - 巢狀 Scope = 「父子房間，父房間拆掉，子房間也會倒」
> - 平行 Scope = 「獨立房間，不管父房間拆不拆，子房間都還在」

### 結論

- **KeepAlive + Autofac Scope → 平行 Scope 是合理且常用做法**
- 不會違背 Autofac 設計理念，只是犧牲了巢狀 Scope 的自動 Dispose 便利
- 對高頻、多層導航 + KeepAlive 的場景，是實務上最穩定的選擇

---

# Mix 模式

Mix 模式的實作複雜度確實比「全用巢狀 Scope」或「全用平行 Scope」高一些，但其實是可控的，只要把規則理清楚。仔細分析如下：

## 1. Mix 模式核心概念

1. **KeepAlive 頁面 → 平行 Scope**
    
    - 從 RootScope 直接建立
    - 不隨上層 Dispose
    - 生命週期需手動管理（Dispose/OnUnloaded）
        
2. **一次性頁面 → 巢狀 Scope**
    
    - 從上一層 Scope 建立
    - 跟隨上層 Dispose 自動清理
    - 方便資源管理
        
3. **NavigationService 管理 Stack**
    
    - 每個頁面維護 `(ViewModel, Scope, KeepAlive)`
    - GoBack / Navigate 時判斷 KeepAlive 來決定 Dispose 邏輯

## 2. 複雜度來源

### 2.1 Scope 管理

- 每個 KeepAlive 頁面要單獨保存 Scope
- Dispose 時要注意：非 KeepAlive → 自動 Dispose；KeepAlive → 保留，或手動清理

### 2.2 Stack / History 管理

- 不同層級的 Scope 與 KeepAlive 組合
- Stack 不能只是單純存 ViewModel，要存 Scope + KeepAlive 狀態
- GoBack、Forward 還要處理不同 Scope 的釋放順序

### 2.3 參數刷新

- KeepAlive 頁面在第二次 Navigate 時不會重建
- 需要額外設計 `Update(parameters)` 或 Refresh 機制

### 2.4 Dialog / 子頁整合

- Dialog 也可能是 KeepAlive 或一次性
- Scope 也要和 Navigation Stack 協調，避免 Dispose 衝突

## 3. 實務建議降低複雜度

1. **規範頁面類型**
    
    - 每個頁面明確設定 KeepAlive
    - KeepAlive 頁面 Scope → RootScope
    - 一次性頁面 Scope → 前一頁 Scope
        
2. **NavigationService 統一管理**
    
    - Stack 存 `(ViewModel, Scope, KeepAlive)`
    - GoBack / Navigate 統一判斷 KeepAlive → 決定 Dispose
    - OnLoaded / OnUnloaded / Dispose 流程統一調用
        
3. **Refresh 機制**
    
    - KeepAlive 頁面必須有 Update(parameters)
    - 避免 KeepAlive 導致狀態老舊或資料不一致
        
4. **Cache 配合**
    
    - EF6 查詢結果可放 Cache
    - 用完就丟頁面 + Cache 資料 → 避免 KeepAlive 節省記憶體

## 4. 評估複雜度

|項目|Mix 模式複雜度|備註|
|---|---|---|
|Scope 管理|中高|KeepAlive 平行 Scope + 一次性巢狀 Scope|
|Stack / History|中|每個頁面要存 Scope + KeepAlive 狀態|
|參數更新|低中|KeepAlive 頁面要額外 Refresh / Update|
|Dialog / 子頁|中|Scope 與 Stack 協調|
|記憶體管理|中|KeepAlive 節省初始化但增加記憶體占用|

> 整體：複雜度比純巢狀或純平行高，但**可控且邏輯清晰**
> 對多層深度 + 頻繁切換 + EF6 + Cache 的場景，是實務上最穩定的策略
