---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# ChatGPT 提示詞

我使用 EF6 做資料庫，撰寫 WINFORM 桌面程式，我現在想對 EF6 的資料做統一的管理，在所有模組中只保存一組資料，並且統一管理一組 VIEWMODEL 給所有 VIEW 共用。DI 使用 AUTOFAC，MVVM 使用 CommunityToolkit.Mvvm，不使用 AutoMapper。
需要對 DbContext 做快取、變更追蹤、自動儲存。頁面切換有多層，有 ALIVE 功能，目前已有相關的 NAV 及 REGION 功能。暫不需要線程安全機制。
資料快取由 自製 DataStore 持有，ViewModel 不直接持有 EF 的實體，而是持有 DTO/ObservableModel，快取存放的是 EF 實體的完整樹狀 DTO（含必要的導航屬性展開）
三個主要集合 Employees/Machines/TaskOrders，有可能多個 View 會同時編輯同一筆資料

# 架構設計方向

資料管理核心分三層：DbContext 管理、快取 (DataStore) 管理、ViewModel 管理

## DbContext 管理

- 單例或範圍內共用 DbContext，但保留 ALIVE 機制以支援頁面切換。
- 提供 CRUD API，帶變更追蹤，並在必要時自動呼叫 SaveChanges。
- 對查詢結果回傳實體，不直接給 ViewModel。

## DataStore 快取

- 持有完整 EF 實體樹狀 DTO，含必要導航屬性。
- 支援快取更新、清除、合併功能。
- 當 DbContext 變更時同步更新快取。
- 快取資料做 Observable，方便 ViewModel 更新。

## ViewModel 管理

- 不持有 EF 實體，持有 DTO/ObservableModel。
- 可訂閱 DataStore 變化，更新界面。
- 多個 View 共用同一 ViewModel 實例。
- 支援多層頁面切換，Region/Navigator 控制 View 顯示。

## 多人編輯/同步

- 快取更新後通知所有訂閱 ViewModel。
- 支援「版本檢查/衝突檢查」機制（可用 Timestamp 或 RowVersion）。
- 修改時只更新 DTO，最終由 DataStore 將變更寫回 DbContext。

## Autofac DI 配置

- DataStore 註冊為單例。
- ViewModel 註冊為單例或範圍內共用，視頁面生命週期決定。
- DbContext 註冊為範圍內共用，配合 ALIVE 機制。

## 快取/變更追蹤流程

1. ViewModel 從 DataStore 取得 ObservableModel。
2. 使用者操作修改 DTO。
3. DataStore 將 DTO 映射回 EF 實體，更新 DbContext。
4. DbContext SaveChanges 自動觸發。
5. DataStore 快取同步更新，通知其他 ViewModel。

## 技術注意點

- DTO 與 EF 實體對應可使用 AutoMapper 或自製映射。
- ObservableModel 可使用 CommunityToolkit.Mvvm 的 ObservableObject。
- 對集合（Employees/Machines/TaskOrders）可建立 CollectionViewModel，統一管理新增/刪除/更新操作。
- 頁面切換不釋放 ViewModel，保持資料 ALIVE。
- 多 View 同時編輯同一筆資料時，變更合併策略需明確（LastWriteWins 或手動衝突解決）。

## 目標結果

- 單一資料來源 (DbContext + DataStore)。
- 多 View 共用同一 ViewModel。
- 自動追蹤變更與儲存。
- 快取支援完整資料樹及導航屬性。
- ALIVE 與 Region/Navigator 功能整合。

---

# EF6 + WinForms MVVM 統一資料管理範例

## 1. DbContext 定義

```csharp
using System.Data.Entity;
using Calin.TaskPulse.Entity.Core;

public class AppDbContext : DbContext
{
    public AppDbContext() : base("name=AppDbConnection") { }

    public DbSet<Employee> Employees { get; set; }
    public DbSet<Machine> Machines { get; set; }
    public DbSet<TaskOrder> TaskOrders { get; set; }

    public void SaveChangesWithTracking()
    {
        base.SaveChanges();
        // 可加入額外的變更事件或快取通知
    }
}
```

## 2. DTO / ObservableModel

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using System.Collections.ObjectModel;

public class EmployeeDto : ObservableObject
{
    private int _id;
    public int Id { get => _id; set => SetProperty(ref _id, value); }

    private string _name;
    public string Name { get => _name; set => SetProperty(ref _name, value); }

    // 可加入導航屬性
    public ObservableCollection<TaskOrderDto> TaskOrders { get; set; } = new ObservableCollection<TaskOrderDto>();
}

public class TaskOrderDto : ObservableObject
{
    private int _id;
    public int Id { get => _id; set => SetProperty(ref _id, value); }

    private string _description;
    public string Description { get => _description; set => SetProperty(ref _description, value); }
}
```

## 3. DataStore

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Collections.ObjectModel;
using AutoMapper;

public class DataStore
{
    private readonly AppDbContext _dbContext;
    private readonly IMapper _mapper;

    public ObservableCollection<EmployeeDto> Employees { get; private set; } = new ObservableCollection<EmployeeDto>();
    public ObservableCollection<TaskOrderDto> TaskOrders { get; private set; } = new ObservableCollection<TaskOrderDto>();

    public DataStore(AppDbContext dbContext, IMapper mapper)
    {
        _dbContext = dbContext;
        _mapper = mapper;
        LoadData();
    }

    private void LoadData()
    {
        var emps = _dbContext.Employees.Include("TaskOrders").ToList();
        Employees = new ObservableCollection<EmployeeDto>(_mapper.Map<List<EmployeeDto>>(emps));
        var tasks = _dbContext.TaskOrders.ToList();
        TaskOrders = new ObservableCollection<TaskOrderDto>(_mapper.Map<List<TaskOrderDto>>(tasks));
    }

    public void UpdateEmployee(EmployeeDto dto)
    {
        var entity = _dbContext.Employees.Find(dto.Id);
        if(entity != null)
        {
            _mapper.Map(dto, entity);
            _dbContext.SaveChangesWithTracking();
        }
    }

    public void AddEmployee(EmployeeDto dto)
    {
        var entity = _mapper.Map<Employee>(dto);
        _dbContext.Employees.Add(entity);
        _dbContext.SaveChangesWithTracking();
        dto.Id = entity.Id;
        Employees.Add(dto);
    }
}
```

## 4. ViewModel 範例

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
using System.Collections.ObjectModel;

public class EmployeeViewModel : ObservableObject
{
    private readonly DataStore _dataStore;

    public ObservableCollection<EmployeeDto> Employees => _dataStore.Employees;

    private EmployeeDto _selectedEmployee;
    public EmployeeDto SelectedEmployee { get => _selectedEmployee; set => SetProperty(ref _selectedEmployee, value); }

    public RelayCommand SaveCommand { get; }

    public EmployeeViewModel(DataStore dataStore)
    {
        _dataStore = dataStore;
        SaveCommand = new RelayCommand(Save);
    }

    private void Save()
    {
        if(SelectedEmployee != null)
            _dataStore.UpdateEmployee(SelectedEmployee);
    }
}
```

## 5. DI (Autofac) 註冊

```csharp
using Autofac;

var builder = new ContainerBuilder();

builder.RegisterType<AppDbContext>().InstancePerLifetimeScope();
builder.RegisterType<DataStore>().SingleInstance();
builder.RegisterType<EmployeeViewModel>().SingleInstance();

// AutoMapper 設定
var config = new AutoMapper.MapperConfiguration(cfg =>
{
    cfg.CreateMap<Employee, EmployeeDto>().ReverseMap();
    cfg.CreateMap<TaskOrder, TaskOrderDto>().ReverseMap();
});
builder.RegisterInstance(config.CreateMapper()).SingleInstance();

var container = builder.Build();
```

## 6. View 綁定示例 (WinForms)

```csharp
public partial class EmployeeForm : Form
{
    private readonly EmployeeViewModel _vm;

    public EmployeeForm(EmployeeViewModel vm)
    {
        InitializeComponent();
        _vm = vm;
        employeeBindingSource.DataSource = _vm.Employees;
        txtName.DataBindings.Add("Text", _vm, "SelectedEmployee.Name", false, DataSourceUpdateMode.OnPropertyChanged);
    }
}
```

## 7. 關鍵特性

- ViewModel 不持有 EF 實體。
- DataStore 保持快取完整樹狀 DTO。
- CRUD 操作經由 DataStore 反映回 DbContext。
- ObservableCollection 支援多 View 同步更新。
- AutoMapper 負責 EF 實體與 DTO 映射。

---

# DTO 設計與使用範例

## 1. 定義 Entity 與 DTO

```csharp
// Entity: EF6
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
    public ICollection<Order> Orders { get; set; }
}

// DTO: 傳輸或中介層
public class CustomerDTO
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public int OrderCount { get; set; } // 聚合資訊
}
```

## 2. DTO ↔ Entity 轉換

```csharp
// Entity -> DTO
public static CustomerDTO ToDTO(this Customer entity)
{
    return new CustomerDTO
    {
        Id = entity.Id,
        Name = entity.Name,
        Email = entity.Email,
        OrderCount = entity.Orders?.Count ?? 0
    };
}

// DTO -> Entity (更新模式)
public static void UpdateEntity(this CustomerDTO dto, Customer entity)
{
    entity.Name = dto.Name;
    entity.Email = dto.Email;
    // Orders 與 CreatedAt 不在 DTO 中更新
}
```

## 3. DTO ↔ ViewModel 轉換

```csharp
// ViewModel: MVVM 使用 CommunityToolkit.Mvvm
public partial class CustomerViewModel : ObservableObject
{
    [ObservableProperty]
    private int id;

    [ObservableProperty]
    private string name;

    [ObservableProperty]
    private string email;

    [ObservableProperty]
    private int orderCount;
}

// DTO -> ViewModel
public static CustomerViewModel ToViewModel(this CustomerDTO dto)
{
    return new CustomerViewModel
    {
        Id = dto.Id,
        Name = dto.Name,
        Email = dto.Email,
        OrderCount = dto.OrderCount
    };
}

// ViewModel -> DTO
public static CustomerDTO ToDTO(this CustomerViewModel vm)
{
    return new CustomerDTO
    {
        Id = vm.Id,
        Name = vm.Name,
        Email = vm.Email,
        OrderCount = vm.OrderCount
    };
}
```

## 4. 儲存方式與變動偵測

1. **集中管理單一 DbContext**
    - 建立 `DbContext` 單例或範圍注入
    - 使用快取儲存 DTO 列表，例如 `ObservableCollection<CustomerDTO>`

2. **變動追蹤**
    - DTO 不直接追蹤 EF 變化，可透過 ViewModel 的 `PropertyChanged` 觸發更新
    - 保存時將 DTO 更新回 Entity，呼叫 `DbContext.SaveChanges()`

```csharp
// 更新流程
var dto = customerViewModel.ToDTO();
var entity = dbContext.Customers.Find(dto.Id);
dto.UpdateEntity(entity);
dbContext.SaveChanges();
```

3. **快取管理**
    - 建立 `Repository` 或 `DataService` 管理單一資料集合
    - 切換頁面時直接綁定 ViewModel，減少重複查詢

```csharp
public class CustomerService
{
    private readonly DbContext _db;
    private readonly ObservableCollection<CustomerDTO> _cache;

    public CustomerService(DbContext db)
    {
        _db = db;
        _cache = new ObservableCollection<CustomerDTO>(_db.Customers.Include(c => c.Orders).ToList().Select(c => c.ToDTO()));
    }

    public ObservableCollection<CustomerDTO> Customers => _cache;

    public void Update(CustomerDTO dto)
    {
        var entity = _db.Customers.Find(dto.Id);
        dto.UpdateEntity(entity);
        _db.SaveChanges();
    }
}
```

## 5. 擴展概念

- DTO 可包含多個聚合欄位，方便 ViewModel 綁定
- DTO 也可以是 `readonly`，減少不必要變動
- 可設計批次更新與變動記錄，支援 Undo/Redo

這種方式將資料流分離：Entity 負責資料庫，DTO 負責資料傳輸與快取，ViewModel 負責 UI 綁定與變動追蹤。

---

# Entity、DTO、ViewModel 各自 Service 設計

## 1. EntityService：管理 EF6 Entity

```csharp
public class CustomerEntityService
{
    private readonly DbContext _db;

    public CustomerEntityService(DbContext db)
    {
        _db = db;
    }

    public Customer GetById(int id)
    {
        return _db.Set<Customer>().Include(c => c.Orders).FirstOrDefault(c => c.Id == id);
    }

    public void Update(Customer entity)
    {
        _db.Entry(entity).State = EntityState.Modified;
        _db.SaveChanges();
    }

    public void Add(Customer entity)
    {
        _db.Set<Customer>().Add(entity);
        _db.SaveChanges();
    }

    public void Delete(Customer entity)
    {
        _db.Set<Customer>().Remove(entity);
        _db.SaveChanges();
    }

    public List<Customer> GetAll()
    {
        return _db.Set<Customer>().Include(c => c.Orders).ToList();
    }
}
```

## 2. DTOService：管理快取 DTO 與轉換

```csharp
public class CustomerDTOService
{
    private readonly CustomerEntityService _entityService;
    private readonly List<CustomerDTO> _cache = new();

    public CustomerDTOService(CustomerEntityService entityService)
    {
        _entityService = entityService;
        LoadCache();
    }

    private void LoadCache()
    {
        _cache.Clear();
        var entities = _entityService.GetAll();
        _cache.AddRange(entities.Select(e => e.ToDTO()));
    }

    public List<CustomerDTO> GetAll() => _cache;

    public CustomerDTO GetById(int id) => _cache.FirstOrDefault(c => c.Id == id);

    public void Update(CustomerDTO dto)
    {
        var entity = _entityService.GetById(dto.Id);
        dto.UpdateEntity(entity);
        _entityService.Update(entity);
        LoadCache(); // 保持 DTO 快取一致
    }

    public void Add(CustomerDTO dto)
    {
        var entity = new Customer();
        dto.UpdateEntity(entity);
        _entityService.Add(entity);
        LoadCache();
    }

    public void Delete(CustomerDTO dto)
    {
        var entity = _entityService.GetById(dto.Id);
        _entityService.Delete(entity);
        LoadCache();
    }
}
```

## 3. ViewModelService：管理 UI ViewModel 與通知

```csharp
public class CustomerViewModelService
{
    private readonly CustomerDTOService _dtoService;
    public ObservableCollection<CustomerViewModel> Customers { get; } = new();

    public CustomerViewModelService(CustomerDTOService dtoService)
    {
        _dtoService = dtoService;
        LoadViewModels();
    }

    private void LoadViewModels()
    {
        Customers.Clear();
        foreach (var dto in _dtoService.GetAll())
        {
            Customers.Add(dto.ToViewModel());
        }
    }

    public void Update(CustomerViewModel vm)
    {
        var dto = vm.ToDTO();
        _dtoService.Update(dto);
        LoadViewModels(); // 保持 ViewModel 與 DTO 一致
    }

    public void Add(CustomerViewModel vm)
    {
        var dto = vm.ToDTO();
        _dtoService.Add(dto);
        LoadViewModels();
    }

    public void Delete(CustomerViewModel vm)
    {
        var dto = vm.ToDTO();
        _dtoService.Delete(dto);
        LoadViewModels();
    }
}
```

## 4. 使用流程

1. `CustomerEntityService` 直接操作資料庫 Entity
2. `CustomerDTOService` 管理快取 DTO 與 Entity 轉換
3. `CustomerViewModelService` 管理 UI 綁定，監控變動

這樣可以將各層責任完全分離：

- Entity 層專注資料庫
- DTO 層專注快取與資料傳輸
- ViewModel 層專注 UI 綁定與事件通知

這個架構可配合 DI 容器（如 Autofac）統一注入與單例管理。
