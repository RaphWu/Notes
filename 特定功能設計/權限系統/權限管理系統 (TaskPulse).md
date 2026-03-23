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

# 提示詞

我使用 C#+EF6+.NET461+WINFORM，AUTOFAC，我要設計一套權限管理系統。
DB 裡有部門 (名稱為 Department)+ 所有員工 (名稱為 Employee)，員工的登入有另一個獨立的系統，這裡不需要記錄密碼，但仍要記錄登入動作。
公司裡還有許多部門，都各自有部門的權限，所有員工都擁有所屬部門的權限。
我還要另一套權限表，比如各部門的高階主管或是有特殊需求的群組可以另給一些權限，幫我命名為 UserGroup。
使用 PBKDF2 密碼、AES 加密敏感欄位，所有權限資料均存在 EF6 裡，我會設計 UI 給有增修權限的員工去修改，所以也要一起封裝 CRUD 功能。
請幫我用 AUTOFAC 封裝呼叫介面，因為有很多模組要跨 NAMESPACE 呼叫，還需要能取得目前使用者是誰，以及該 USER 的所有權限，讓我可以在各模組裡，決定那些功能要開給 USER，那些功能要關閉。
我還要使用 Serilog 記錄所有的活動，包含那個 USER 登入登出，以及 CRUD 動作，請用 JSON 格式，我好將來可以抓紀錄出來分析。
使用 MemoryCache 提升效能，以及 EF6 Fluent API 設定。

## 架構設計

### 核心實體

* Employee：員工資訊，含密碼（PBKDF2）與敏感欄位（AES 加密）
* Department：部門資訊
* Role：部門角色/權限（RBAC）
* UserGroup：特定群組權限（ABAC）
* Permission：權限項目
* ActivityLog：操作日誌，JSON 格式儲存

### 權限邏輯

* RBAC：部門角色對應權限
* ABAC：UserGroup 對應額外權限
* 最終使用者權限 = 部門權限 + UserGroup 額外權限

### 功能需求

* CRUD：員工、部門、角色、群組、權限
* 使用 MemoryCache 提升讀取權限效能
* Autofac 注入服務
* Serilog 記錄 JSON 格式日誌
* EF6 Fluent API 設定關聯與索引

## 1. 實體範例（Entity）

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PasswordHash { get; set; }
    public string DepartmentId { get; set; }
    public string SensitiveData { get; set; } // AES 加密
    public virtual Department Department { get; set; }
    public virtual ICollection<UserGroup> UserGroups { get; set; }
}

public class Department
{
    public string Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Role> Roles { get; set; }
}

public class Role
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Permission> Permissions { get; set; }
    public string DepartmentId { get; set; }
    public virtual Department Department { get; set; }
}

public class UserGroup
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Permission> Permissions { get; set; }
    public virtual ICollection<Employee> Members { get; set; }
}

public class Permission
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class ActivityLog
{
    public int Id { get; set; }
    public string UserId { get; set; }
    public string Action { get; set; }
    public string Data { get; set; }
    public DateTime Timestamp { get; set; }
}
```

## 2. EF6 Fluent API 設定範例

```csharp
public class MyDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Role> Roles { get; set; }
    public DbSet<UserGroup> UserGroups { get; set; }
    public DbSet<Permission> Permissions { get; set; }
    public DbSet<ActivityLog> ActivityLogs { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>()
            .HasRequired(e => e.Department)
            .WithMany(d => d.Roles)
            .HasForeignKey(e => e.DepartmentId);

        modelBuilder.Entity<Employee>()
            .HasMany(e => e.UserGroups)
            .WithMany(ug => ug.Members)
            .Map(m => 
            {
                m.ToTable("EmployeeUserGroups");
                m.MapLeftKey("EmployeeId");
                m.MapRightKey("UserGroupId");
            });

        modelBuilder.Entity<Role>()
            .HasMany(r => r.Permissions)
            .WithMany()
            .Map(m =>
            {
                m.ToTable("RolePermissions");
                m.MapLeftKey("RoleId");
                m.MapRightKey("PermissionId");
            });

        modelBuilder.Entity<UserGroup>()
            .HasMany(ug => ug.Permissions)
            .WithMany()
            .Map(m =>
            {
                m.ToTable("UserGroupPermissions");
                m.MapLeftKey("UserGroupId");
                m.MapRightKey("PermissionId");
            });
    }
}
```

## 3. 密碼與敏感資料加密工具

```csharp
public static class SecurityHelper
{
    public static string HashPassword(string password)
    {
        using (var rfc = new Rfc2898DeriveBytes(password, 16, 10000))
        {
            var salt = rfc.Salt;
            var hash = rfc.GetBytes(32);
            return Convert.ToBase64String(salt) + ":" + Convert.ToBase64String(hash);
        }
    }

    public static bool VerifyPassword(string password, string stored)
    {
        var parts = stored.Split(':');
        var salt = Convert.FromBase64String(parts[0]);
        var hash = Convert.FromBase64String(parts[1]);

        using (var rfc = new Rfc2898DeriveBytes(password, salt, 10000))
        {
            var computed = rfc.GetBytes(32);
            return computed.SequenceEqual(hash);
        }
    }

    public static string EncryptAES(string plainText, byte[] key, byte[] iv)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = key;
            aes.IV = iv;
            var encryptor = aes.CreateEncryptor();
            var bytes = Encoding.UTF8.GetBytes(plainText);
            var encrypted = encryptor.TransformFinalBlock(bytes, 0, bytes.Length);
            return Convert.ToBase64String(encrypted);
        }
    }

    public static string DecryptAES(string cipherText, byte[] key, byte[] iv)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = key;
            aes.IV = iv;
            var decryptor = aes.CreateDecryptor();
            var bytes = Convert.FromBase64String(cipherText);
            var decrypted = decryptor.TransformFinalBlock(bytes, 0, bytes.Length);
            return Encoding.UTF8.GetString(decrypted);
        }
    }
}
```

## 4. 權限服務

```csharp
public interface IPermissionService
{
    Task<List<Permission>> GetUserPermissionsAsync(int employeeId);
    bool HasPermission(int employeeId, string permissionName);
}

public class PermissionService : IPermissionService
{
    private readonly MyDbContext _db;
    private readonly IMemoryCache _cache;

    public PermissionService(MyDbContext db, IMemoryCache cache)
    {
        _db = db;
        _cache = cache;
    }

    public async Task<List<Permission>> GetUserPermissionsAsync(int employeeId)
    {
        return await _cache.GetOrCreateAsync($"permissions_{employeeId}", async entry =>
        {
            var employee = await _db.Employees
                .Include(e => e.Department.Roles.Select(r => r.Permissions))
                .Include(e => e.UserGroups.Select(ug => ug.Permissions))
                .FirstOrDefaultAsync(e => e.Id == employeeId);

            var perms = employee.Department.Roles.SelectMany(r => r.Permissions)
                .Union(employee.UserGroups.SelectMany(ug => ug.Permissions))
                .Distinct()
                .ToList();
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
            return perms;
        });
    }

    public bool HasPermission(int employeeId, string permissionName)
    {
        var perms = GetUserPermissionsAsync(employeeId).Result;
        return perms.Any(p => p.Name == permissionName);
    }
}
```

## 5. 使用 Autofac 封裝跨模組呼叫

```csharp
var builder = new ContainerBuilder();

builder.RegisterType<MyDbContext>().AsSelf().InstancePerLifetimeScope();
builder.RegisterType<PermissionService>().As<IPermissionService>().SingleInstance();
builder.RegisterType<EmployeeService>().As<IEmployeeService>().SingleInstance();
builder.RegisterType<ActivityLogService>().As<IActivityLogService>().SingleInstance();
builder.RegisterType<MemoryCache>().As<IMemoryCache>().SingleInstance();

var container = builder.Build();
```

## 6. Serilog JSON 日誌範例

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.File(new JsonFormatter(), "logs/log.json")
    .CreateLogger();

public class ActivityLogService : IActivityLogService
{
    private readonly MyDbContext _db;
    public ActivityLogService(MyDbContext db)
    {
        _db = db;
    }

    public async Task LogAsync(string userId, string action, object data)
    {
        var log = new ActivityLog
        {
            UserId = userId,
            Action = action,
            Data = JsonConvert.SerializeObject(data),
            Timestamp = DateTime.UtcNow
        };
        _db.ActivityLogs.Add(log);
        await _db.SaveChangesAsync();
        Log.Information("{@Log}", log);
    }
}
```

這套架構：

* 支援 RBAC + ABAC 權限控制
* 使用 MemoryCache 提升查詢效能
* 使用 Autofac 封裝服務，支援跨模組呼叫
* 用 Serilog 記錄 JSON 格式日誌
* EF6 Fluent API 設定關聯
* PBKDF2 密碼與 AES 加密敏感欄位

如果你需要，我可以幫你做 **完整 CRUD 範例 + UI 呼叫範例 + 權限檢查示範**，讓每個模組可以直接判斷功能是否開啟。

# CRUD 範例與 UI 權限檢查設計

以下提供完整 CRUD 服務範例，包含權限檢查與日誌記錄，讓各模組可直接使用。

## 1. 員工 CRUD 服務介面

```csharp
public interface IEmployeeService
{
    Task<Employee> CreateAsync(Employee employee, string currentUserId);
    Task<Employee> UpdateAsync(Employee employee, string currentUserId);
    Task DeleteAsync(int employeeId, string currentUserId);
    Task<Employee> GetByIdAsync(int employeeId);
    Task<List<Employee>> GetAllAsync();
}
```

## 2. 員工 CRUD 服務實作

```csharp
public class EmployeeService : IEmployeeService
{
    private readonly MyDbContext _db;
    private readonly IActivityLogService _logService;
    private readonly IMemoryCache _cache;

    public EmployeeService(MyDbContext db, IActivityLogService logService, IMemoryCache cache)
    {
        _db = db;
        _logService = logService;
        _cache = cache;
    }

    public async Task<Employee> CreateAsync(Employee employee, string currentUserId)
    {
        employee.PasswordHash = SecurityHelper.HashPassword(employee.PasswordHash);
        _db.Employees.Add(employee);
        await _db.SaveChangesAsync();

        _cache.Remove($"permissions_{employee.Id}");

        await _logService.LogAsync(currentUserId, "Create Employee", new { employee.Id, employee.Name });
        return employee;
    }

    public async Task<Employee> UpdateAsync(Employee employee, string currentUserId)
    {
        var existing = await _db.Employees.FindAsync(employee.Id);
        if (existing == null) throw new Exception("Employee not found");

        existing.Name = employee.Name;
        existing.DepartmentId = employee.DepartmentId;
        if (!string.IsNullOrEmpty(employee.PasswordHash))
            existing.PasswordHash = SecurityHelper.HashPassword(employee.PasswordHash);
        if (!string.IsNullOrEmpty(employee.SensitiveData))
            existing.SensitiveData = employee.SensitiveData;

        await _db.SaveChangesAsync();
        _cache.Remove($"permissions_{employee.Id}");

        await _logService.LogAsync(currentUserId, "Update Employee", new { employee.Id, employee.Name });
        return existing;
    }

    public async Task DeleteAsync(int employeeId, string currentUserId)
    {
        var existing = await _db.Employees.FindAsync(employeeId);
        if (existing == null) throw new Exception("Employee not found");

        _db.Employees.Remove(existing);
        await _db.SaveChangesAsync();
        _cache.Remove($"permissions_{employeeId}");

        await _logService.LogAsync(currentUserId, "Delete Employee", new { employeeId });
    }

    public Task<Employee> GetByIdAsync(int employeeId)
    {
        return _db.Employees
            .Include(e => e.Department)
            .Include(e => e.UserGroups)
            .FirstOrDefaultAsync(e => e.Id == employeeId);
    }

    public Task<List<Employee>> GetAllAsync()
    {
        return _db.Employees
            .Include(e => e.Department)
            .Include(e => e.UserGroups)
            .ToListAsync();
    }
}
```

## 3. 權限檢查輔助

```csharp
public static class PermissionChecker
{
    public static bool CanAccess(IPermissionService permissionService, int userId, string permission)
    {
        return permissionService.HasPermission(userId, permission);
    }
}
```

## 4. UI 呼叫示範（WinForm 範例）

```csharp
private readonly IEmployeeService _employeeService;
private readonly IPermissionService _permissionService;
private readonly string _currentUserId;

public void LoadUI(IEmployeeService employeeService, IPermissionService permissionService, string currentUserId)
{
    _employeeService = employeeService;
    _permissionService = permissionService;
    _currentUserId = currentUserId;

    // 權限控制按鈕
    btnAddEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeCreate");
    btnEditEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeUpdate");
    btnDeleteEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeDelete");
}

private async void btnAddEmployee_Click(object sender, EventArgs e)
{
    var emp = new Employee { Name = txtName.Text, DepartmentId = txtDeptId.Text, PasswordHash = txtPassword.Text };
    await _employeeService.CreateAsync(emp, _currentUserId);
    MessageBox.Show("新增成功");
}

private async void btnEditEmployee_Click(object sender, EventArgs e)
{
    var emp = await _employeeService.GetByIdAsync(int.Parse(txtEmpId.Text));
    emp.Name = txtName.Text;
    await _employeeService.UpdateAsync(emp, _currentUserId);
    MessageBox.Show("更新成功");
}

private async void btnDeleteEmployee_Click(object sender, EventArgs e)
{
    await _employeeService.DeleteAsync(int.Parse(txtEmpId.Text), _currentUserId);
    MessageBox.Show("刪除成功");
}
```

## 5. 日誌與分析

* 所有 CRUD 操作都會寫入 `ActivityLog` 與 Serilog JSON 日誌
* 日誌範例：

```json
{
    "UserId": "user001",
    "Action": "Create Employee",
    "Data": { "Id": 1001, "Name": "王小明" },
    "Timestamp": "2025-11-06T10:15:30Z"
}
```

* 可用於後續分析：誰新增/修改/刪除、權限操作記錄、登入登出等

這套設計可以直接套用到 **各模組**，只要注入 `IPermissionService` 與 `IEmployeeService`，UI 就能自動根據使用者權限開關功能按鈕，且所有操作都有日誌記錄。

# UserGroup + RBAC/ABAC 權限管理設計

以下設計可讓你同時管理 **部門角色（RBAC）** 與 **特殊群組（ABAC）** 權限，並即時反映在使用者功能權限上。

## 1. UserGroup CRUD 服務介面

```csharp
public interface IUserGroupService
{
    Task<UserGroup> CreateAsync(UserGroup group, string currentUserId);
    Task<UserGroup> UpdateAsync(UserGroup group, string currentUserId);
    Task DeleteAsync(int groupId, string currentUserId);
    Task<UserGroup> GetByIdAsync(int groupId);
    Task<List<UserGroup>> GetAllAsync();
    Task AssignUsersAsync(int groupId, List<int> userIds, string currentUserId);
    Task AssignPermissionsAsync(int groupId, List<int> permissionIds, string currentUserId);
}
```

## 2. UserGroup CRUD 服務實作

```csharp
public class UserGroupService : IUserGroupService
{
    private readonly MyDbContext _db;
    private readonly IActivityLogService _logService;
    private readonly IMemoryCache _cache;

    public UserGroupService(MyDbContext db, IActivityLogService logService, IMemoryCache cache)
    {
        _db = db;
        _logService = logService;
        _cache = cache;
    }

    public async Task<UserGroup> CreateAsync(UserGroup group, string currentUserId)
    {
        _db.UserGroups.Add(group);
        await _db.SaveChangesAsync();

        await _logService.LogAsync(currentUserId, "Create UserGroup", new { group.Id, group.Name });
        return group;
    }

    public async Task<UserGroup> UpdateAsync(UserGroup group, string currentUserId)
    {
        var existing = await _db.UserGroups.FindAsync(group.Id);
        if (existing == null) throw new Exception("UserGroup not found");

        existing.Name = group.Name;
        await _db.SaveChangesAsync();

        _cache.RemoveByPrefix("permissions_"); // 更新群組權限後清空所有使用者快取

        await _logService.LogAsync(currentUserId, "Update UserGroup", new { group.Id, group.Name });
        return existing;
    }

    public async Task DeleteAsync(int groupId, string currentUserId)
    {
        var existing = await _db.UserGroups.FindAsync(groupId);
        if (existing == null) throw new Exception("UserGroup not found");

        _db.UserGroups.Remove(existing);
        await _db.SaveChangesAsync();

        _cache.RemoveByPrefix("permissions_");

        await _logService.LogAsync(currentUserId, "Delete UserGroup", new { groupId });
    }

    public Task<UserGroup> GetByIdAsync(int groupId)
    {
        return _db.UserGroups
            .Include(ug => ug.Permissions)
            .Include(ug => ug.Members)
            .FirstOrDefaultAsync(ug => ug.Id == groupId);
    }

    public Task<List<UserGroup>> GetAllAsync()
    {
        return _db.UserGroups
            .Include(ug => ug.Permissions)
            .Include(ug => ug.Members)
            .ToListAsync();
    }

    public async Task AssignUsersAsync(int groupId, List<int> userIds, string currentUserId)
    {
        var group = await _db.UserGroups.Include(ug => ug.Members).FirstOrDefaultAsync(ug => ug.Id == groupId);
        if (group == null) throw new Exception("UserGroup not found");

        group.Members.Clear();
        var users = await _db.Employees.Where(e => userIds.Contains(e.Id)).ToListAsync();
        foreach (var u in users) group.Members.Add(u);

        await _db.SaveChangesAsync();
        _cache.RemoveByPrefix("permissions_");

        await _logService.LogAsync(currentUserId, "Assign Users to Group", new { groupId, userIds });
    }

    public async Task AssignPermissionsAsync(int groupId, List<int> permissionIds, string currentUserId)
    {
        var group = await _db.UserGroups.Include(ug => ug.Permissions).FirstOrDefaultAsync(ug => ug.Id == groupId);
        if (group == null) throw new Exception("UserGroup not found");

        group.Permissions.Clear();
        var perms = await _db.Permissions.Where(p => permissionIds.Contains(p.Id)).ToListAsync();
        foreach (var p in perms) group.Permissions.Add(p);

        await _db.SaveChangesAsync();
        _cache.RemoveByPrefix("permissions_");

        await _logService.LogAsync(currentUserId, "Assign Permissions to Group", new { groupId, permissionIds });
    }
}
```

> 注意：`_cache.RemoveByPrefix("permissions_")` 是假設你有擴充 MemoryCache，可以根據前綴清掉所有使用者快取，使權限立即生效。

## 3. UI 權限管理示範（WinForm）

```csharp
private readonly IUserGroupService _groupService;
private readonly IPermissionService _permissionService;
private readonly string _currentUserId;

private async void LoadGroups()
{
    var groups = await _groupService.GetAllAsync();
    dgvGroups.DataSource = groups.Select(g => new { g.Id, g.Name }).ToList();
}

private async void btnAssignUsers_Click(object sender, EventArgs e)
{
    int groupId = int.Parse(txtGroupId.Text);
    var selectedUserIds = dgvUsers.SelectedRows.Cast<DataGridViewRow>()
                             .Select(r => (int)r.Cells["Id"].Value).ToList();

    await _groupService.AssignUsersAsync(groupId, selectedUserIds, _currentUserId);
    MessageBox.Show("指派使用者完成");
}

private async void btnAssignPermissions_Click(object sender, EventArgs e)
{
    int groupId = int.Parse(txtGroupId.Text);
    var selectedPermissionIds = dgvPermissions.SelectedRows.Cast<DataGridViewRow>()
                                   .Select(r => (int)r.Cells["Id"].Value).ToList();

    await _groupService.AssignPermissionsAsync(groupId, selectedPermissionIds, _currentUserId);
    MessageBox.Show("指派權限完成");
}
```

## 4. 權限檢查與功能開關範例

```csharp
private void ApplyPermissionsToUI()
{
    btnAddEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeCreate");
    btnEditEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeUpdate");
    btnDeleteEmployee.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "EmployeeDelete");
    btnManageGroups.Enabled = PermissionChecker.CanAccess(_permissionService, _currentUserId, "UserGroupManage");
}
```

## 5. 設計重點

* **RBAC**：部門角色對應權限
* **ABAC**：UserGroup 對應額外權限
* **權限合併**：最終使用者權限 = 部門角色權限 + UserGroup 額外權限
* **MemoryCache**：查詢權限快取，提升 UI 渲染效能
* **日誌紀錄**：CRUD 與群組操作全部寫入 JSON 日誌，方便後續分析
* **UI 即時權限控制**：依使用者權限動態開關功能按鈕

這套設計讓你可以：

1. 管理部門角色 RBAC
2. 管理特殊群組 ABAC
3. 即時反映使用者權限
4. 使用 MemoryCache 提升效能
5. Serilog JSON 日誌完整記錄操作

# 跨模組權限服務與 Autofac 注入設計

以下設計可讓各模組（WinForm、報表、工具模組）統一取得當前使用者與權限，並能跨 Namespace 呼叫。

## 1. 使用者上下文介面

```csharp
public interface ICurrentUserService
{
    int UserId { get; }
    string UserName { get; }
    Task<List<string>> GetPermissionsAsync();
}
```

## 2. 使用者上下文實作（整合 MemoryCache + PermissionService）

```csharp
public class CurrentUserService : ICurrentUserService
{
    private readonly IPermissionService _permissionService;
    private readonly IMemoryCache _cache;
    private readonly int _userId;
    private readonly string _userName;

    public CurrentUserService(IPermissionService permissionService, IMemoryCache cache, int userId, string userName)
    {
        _permissionService = permissionService;
        _cache = cache;
        _userId = userId;
        _userName = userName;
    }

    public int UserId => _userId;
    public string UserName => _userName;

    public Task<List<string>> GetPermissionsAsync()
    {
        return _cache.GetOrCreateAsync($"permissions_{_userId}", async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
            var perms = await _permissionService.GetUserPermissionsAsync(_userId);
            return perms;
        });
    }
}
```

> 註：`IPermissionService.GetUserPermissionsAsync` 返回該使用者 **部門角色 + UserGroup 合併權限**

## 3. Autofac 模組註冊範例

```csharp
var builder = new ContainerBuilder();

// EF6 DbContext
builder.RegisterType<MyDbContext>().AsSelf().InstancePerLifetimeScope();

// CRUD 服務
builder.RegisterType<EmployeeService>().As<IEmployeeService>().InstancePerDependency();
builder.RegisterType<UserGroupService>().As<IUserGroupService>().InstancePerDependency();
builder.RegisterType<PermissionService>().As<IPermissionService>().SingleInstance();
builder.RegisterType<ActivityLogService>().As<IActivityLogService>().SingleInstance();

// MemoryCache
builder.RegisterType<MemoryCache>().As<IMemoryCache>().SingleInstance();

// 當前使用者 (需在登入時指定 UserId / UserName)
builder.Register(c => new CurrentUserService(
        c.Resolve<IPermissionService>(), 
        c.Resolve<IMemoryCache>(), 
        currentUserId, 
        currentUserName
    ))
    .As<ICurrentUserService>()
    .SingleInstance();
```

> `currentUserId` 與 `currentUserName` 可以在使用者登入成功後設定

## 4. 各模組呼叫示範

```csharp
public class EmployeeForm : Form
{
    private readonly IEmployeeService _employeeService;
    private readonly ICurrentUserService _currentUser;

    public EmployeeForm(IEmployeeService employeeService, ICurrentUserService currentUser)
    {
        _employeeService = employeeService;
        _currentUser = currentUser;
    }

    private async void LoadUI()
    {
        var perms = await _currentUser.GetPermissionsAsync();
        btnAddEmployee.Enabled = perms.Contains("EmployeeCreate");
        btnEditEmployee.Enabled = perms.Contains("EmployeeUpdate");
        btnDeleteEmployee.Enabled = perms.Contains("EmployeeDelete");
    }

    private async void btnAddEmployee_Click(object sender, EventArgs e)
    {
        var emp = new Employee { Name = txtName.Text, DepartmentId = int.Parse(txtDeptId.Text), PasswordHash = txtPassword.Text };
        await _employeeService.CreateAsync(emp, _currentUser.UserId.ToString());
    }
}
```

> 使用者上下文可直接跨 Namespace 注入，模組不需知道其它模組實作細節

## 5. 設計重點

* **統一使用者上下文**：所有模組只需注入 `ICurrentUserService`，即可取得使用者資訊及權限
* **跨模組共享**：Autofac 保證不同 Namespace 的 WinForm、報表、工具模組都能呼叫相同服務
* **權限快取**：MemoryCache 快取使用者權限，提升 UI/報表性能
* **即時權限控制**：更新 UserGroup 或部門權限後，清除快取即可立即生效
* **日誌追蹤**：所有 CRUD 與 UserGroup 操作均可透過 `IActivityLogService` 記錄 JSON 日誌

這套架構可以讓你實現：

1. RBAC + ABAC 權限管理
2. 多模組跨 Namespace 使用同一套權限與使用者資訊
3. UI、報表、工具模組統一權限檢查
4. MemoryCache 提升效能，Serilog JSON 日誌完整記錄

如果你願意，我可以幫你設計 **完整 EF6 Fluent API 實體與關聯設定範例**，包括 Employee、Department、Permission、UserGroup 的關聯與加密欄位設定，讓你的資料模型可直接用於 CRUD 與權限管理。

# EF6 Fluent API 實體與關聯設定範例

以下設計包含 **Employee、Department、Permission、UserGroup**，同時支援 **RBAC + ABAC**，並示範敏感欄位 AES 加密及 PBKDF2 密碼哈希存儲。

## 1. Entity 定義

```csharp
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Permission> Permissions { get; set; } = new HashSet<Permission>();
    public virtual ICollection<Employee> Employees { get; set; } = new HashSet<Employee>();
}

public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PasswordHash { get; set; } // PBKDF2
    public string SensitiveData { get; set; } // AES 加密
    public int DepartmentId { get; set; }
    public virtual Department Department { get; set; }
    public virtual ICollection<UserGroup> UserGroups { get; set; } = new HashSet<UserGroup>();
}

public class Permission
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Department> Departments { get; set; } = new HashSet<Department>();
    public virtual ICollection<UserGroup> UserGroups { get; set; } = new HashSet<UserGroup>();
}

public class UserGroup
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Employee> Members { get; set; } = new HashSet<Employee>();
    public virtual ICollection<Permission> Permissions { get; set; } = new HashSet<Permission>();
}

public class ActivityLog
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public string Action { get; set; }
    public string DataJson { get; set; }
    public DateTime Timestamp { get; set; }
}
```

## 2. DbContext 與 Fluent API 設定

```csharp
public class MyDbContext : DbContext
{
    public MyDbContext() : base("name=MyConnectionString") { }

    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Permission> Permissions { get; set; }
    public DbSet<UserGroup> UserGroups { get; set; }
    public DbSet<ActivityLog> ActivityLogs { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Employee
        modelBuilder.Entity<Employee>()
            .Property(e => e.Name)
            .IsRequired()
            .HasMaxLength(100);

        modelBuilder.Entity<Employee>()
            .Property(e => e.PasswordHash)
            .IsRequired()
            .HasMaxLength(256);

        modelBuilder.Entity<Employee>()
            .Property(e => e.SensitiveData)
            .HasMaxLength(512);

        modelBuilder.Entity<Employee>()
            .HasRequired(e => e.Department)
            .WithMany(d => d.Employees)
            .HasForeignKey(e => e.DepartmentId);

        modelBuilder.Entity<Employee>()
            .HasMany(e => e.UserGroups)
            .WithMany(ug => ug.Members)
            .Map(m =>
            {
                m.ToTable("EmployeeUserGroups");
                m.MapLeftKey("EmployeeId");
                m.MapRightKey("UserGroupId");
            });

        // Department
        modelBuilder.Entity<Department>()
            .Property(d => d.Name)
            .IsRequired()
            .HasMaxLength(100);

        modelBuilder.Entity<Department>()
            .HasMany(d => d.Permissions)
            .WithMany(p => p.Departments)
            .Map(m =>
            {
                m.ToTable("DepartmentPermissions");
                m.MapLeftKey("DepartmentId");
                m.MapRightKey("PermissionId");
            });

        // Permission
        modelBuilder.Entity<Permission>()
            .Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(100);

        modelBuilder.Entity<Permission>()
            .HasMany(p => p.UserGroups)
            .WithMany(ug => ug.Permissions)
            .Map(m =>
            {
                m.ToTable("UserGroupPermissions");
                m.MapLeftKey("PermissionId");
                m.MapRightKey("UserGroupId");
            });

        // UserGroup
        modelBuilder.Entity<UserGroup>()
            .Property(ug => ug.Name)
            .IsRequired()
            .HasMaxLength(100);

        // ActivityLog
        modelBuilder.Entity<ActivityLog>()
            .Property(a => a.Action)
            .IsRequired()
            .HasMaxLength(100);

        modelBuilder.Entity<ActivityLog>()
            .Property(a => a.DataJson)
            .IsRequired();
    }
}
```

## 3. 敏感欄位 AES 加密示例

```csharp
public static class SecurityHelper
{
    private static readonly byte[] Key = Encoding.UTF8.GetBytes("Your32ByteLengthKeyHere!");
    private static readonly byte[] IV = Encoding.UTF8.GetBytes("16ByteIVVector!");

    public static string EncryptAES(string plainText)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            var encryptor = aes.CreateEncryptor(aes.Key, aes.IV);

            using (var ms = new MemoryStream())
            using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
            using (var sw = new StreamWriter(cs))
            {
                sw.Write(plainText);
                sw.Flush();
                cs.FlushFinalBlock();
                return Convert.ToBase64String(ms.ToArray());
            }
        }
    }

    public static string DecryptAES(string cipherText)
    {
        var buffer = Convert.FromBase64String(cipherText);
        using (var aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            var decryptor = aes.CreateDecryptor(aes.Key, aes.IV);

            using (var ms = new MemoryStream(buffer))
            using (var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read))
            using (var sr = new StreamReader(cs))
            {
                return sr.ReadToEnd();
            }
        }
    }

    public static string HashPassword(string password)
    {
        using (var rfc = new Rfc2898DeriveBytes(password, 16, 10000))
        {
            var salt = rfc.Salt;
            var hash = rfc.GetBytes(32);
            var result = new byte[48];
            Buffer.BlockCopy(salt, 0, result, 0, 16);
            Buffer.BlockCopy(hash, 0, result, 16, 32);
            return Convert.ToBase64String(result);
        }
    }

    public static bool VerifyPassword(string password, string hashed)
    {
        var bytes = Convert.FromBase64String(hashed);
        var salt = new byte[16];
        Buffer.BlockCopy(bytes, 0, salt, 0, 16);
        var hash = new byte[32];
        Buffer.BlockCopy(bytes, 16, hash, 0, 32);

        using (var rfc = new Rfc2898DeriveBytes(password, salt, 10000))
        {
            var newHash = rfc.GetBytes(32);
            return newHash.SequenceEqual(hash);
        }
    }
}
```

## 4. 設計重點

* Employee **密碼使用 PBKDF2**，敏感欄位 AES 加密
* **多對多關聯**：Department ↔ Permission, UserGroup ↔ Permission, Employee ↔ UserGroup
* ActivityLog JSON 記錄 CRUD / 權限操作
* Fluent API 設定表名稱、關聯與欄位長度，支援 EF6

這樣你的系統就可以：

1. RBAC 部門角色 + ABAC UserGroup 權限合併
2. CRUD、UserGroup 管理、權限即時生效
3. 密碼安全存儲 + 敏感資料加密
4. 日誌完整 JSON 記錄，可分析使用者操作

# 整合版 C# WinForm + EF6 + Autofac + RBAC/ABAC 權限管理

我幫你把 **整個系統架構**、**實體模型**、**服務層**、**權限控制**、**日誌紀錄**、**密碼/敏感資料加密**、**Autofac DI** 等全部整合成一個完整可運作的設計範例。這樣你就不用再在不同片段裡找東西。

## 1. Entity 模型與 Fluent API

```csharp
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Permission> Permissions { get; set; } = new HashSet<Permission>();
    public virtual ICollection<Employee> Employees { get; set; } = new HashSet<Employee>();
}

public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PasswordHash { get; set; } // PBKDF2
    public string SensitiveData { get; set; } // AES
    public int DepartmentId { get; set; }
    public virtual Department Department { get; set; }
    public virtual ICollection<UserGroup> UserGroups { get; set; } = new HashSet<UserGroup>();
}

public class Permission
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Department> Departments { get; set; } = new HashSet<Department>();
    public virtual ICollection<UserGroup> UserGroups { get; set; } = new HashSet<UserGroup>();
}

public class UserGroup
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Employee> Members { get; set; } = new HashSet<Employee>();
    public virtual ICollection<Permission> Permissions { get; set; } = new HashSet<Permission>();
}

public class ActivityLog
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public string Action { get; set; }
    public string DataJson { get; set; }
    public DateTime Timestamp { get; set; }
}

public class MyDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Permission> Permissions { get; set; }
    public DbSet<UserGroup> UserGroups { get; set; }
    public DbSet<ActivityLog> ActivityLogs { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Employee>()
            .Property(e => e.Name).IsRequired().HasMaxLength(100);
        modelBuilder.Entity<Employee>()
            .Property(e => e.PasswordHash).IsRequired().HasMaxLength(256);
        modelBuilder.Entity<Employee>()
            .Property(e => e.SensitiveData).HasMaxLength(512);
        modelBuilder.Entity<Employee>()
            .HasRequired(e => e.Department)
            .WithMany(d => d.Employees)
            .HasForeignKey(e => e.DepartmentId);
        modelBuilder.Entity<Employee>()
            .HasMany(e => e.UserGroups)
            .WithMany(ug => ug.Members)
            .Map(m => { m.ToTable("EmployeeUserGroups"); m.MapLeftKey("EmployeeId"); m.MapRightKey("UserGroupId"); });

        modelBuilder.Entity<Department>()
            .Property(d => d.Name).IsRequired().HasMaxLength(100);
        modelBuilder.Entity<Department>()
            .HasMany(d => d.Permissions)
            .WithMany(p => p.Departments)
            .Map(m => { m.ToTable("DepartmentPermissions"); m.MapLeftKey("DepartmentId"); m.MapRightKey("PermissionId"); });

        modelBuilder.Entity<Permission>()
            .Property(p => p.Name).IsRequired().HasMaxLength(100);
        modelBuilder.Entity<Permission>()
            .HasMany(p => p.UserGroups)
            .WithMany(ug => ug.Permissions)
            .Map(m => { m.ToTable("UserGroupPermissions"); m.MapLeftKey("PermissionId"); m.MapRightKey("UserGroupId"); });

        modelBuilder.Entity<UserGroup>()
            .Property(ug => ug.Name).IsRequired().HasMaxLength(100);

        modelBuilder.Entity<ActivityLog>()
            .Property(a => a.Action).IsRequired().HasMaxLength(100);
        modelBuilder.Entity<ActivityLog>()
            .Property(a => a.DataJson).IsRequired();
    }
}
```

## 2. 安全工具（PBKDF2 + AES）

```csharp
public static class SecurityHelper
{
    private static readonly byte[] Key = Encoding.UTF8.GetBytes("Your32ByteLengthKeyHere!");
    private static readonly byte[] IV = Encoding.UTF8.GetBytes("16ByteIVVector!");

    public static string EncryptAES(string plainText)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = Key; aes.IV = IV;
            using (var ms = new MemoryStream())
            using (var cs = new CryptoStream(ms, aes.CreateEncryptor(), CryptoStreamMode.Write))
            using (var sw = new StreamWriter(cs))
            {
                sw.Write(plainText); sw.Flush(); cs.FlushFinalBlock();
                return Convert.ToBase64String(ms.ToArray());
            }
        }
    }

    public static string DecryptAES(string cipherText)
    {
        var buffer = Convert.FromBase64String(cipherText);
        using (var aes = Aes.Create())
        {
            aes.Key = Key; aes.IV = IV;
            using (var ms = new MemoryStream(buffer))
            using (var cs = new CryptoStream(ms, aes.CreateDecryptor(), CryptoStreamMode.Read))
            using (var sr = new StreamReader(cs))
            {
                return sr.ReadToEnd();
            }
        }
    }

    public static string HashPassword(string password)
    {
        using (var rfc = new Rfc2898DeriveBytes(password, 16, 10000))
        {
            var salt = rfc.Salt;
            var hash = rfc.GetBytes(32);
            var result = new byte[48];
            Buffer.BlockCopy(salt, 0, result, 0, 16);
            Buffer.BlockCopy(hash, 0, result, 16, 32);
            return Convert.ToBase64String(result);
        }
    }

    public static bool VerifyPassword(string password, string hashed)
    {
        var bytes = Convert.FromBase64String(hashed);
        var salt = new byte[16]; Buffer.BlockCopy(bytes, 0, salt, 0, 16);
        var hash = new byte[32]; Buffer.BlockCopy(bytes, 16, hash, 0, 32);
        using (var rfc = new Rfc2898DeriveBytes(password, salt, 10000))
        {
            return rfc.GetBytes(32).SequenceEqual(hash);
        }
    }
}
```

## 3. 服務介面與實作

```csharp
public interface ICurrentUserService
{
    int UserId { get; }
    string UserName { get; }
    Task<List<string>> GetPermissionsAsync();
}

public interface IPermissionService
{
    Task<List<string>> GetUserPermissionsAsync(int userId);
}

public interface IEmployeeService
{
    Task<List<Employee>> GetAllAsync();
    Task<Employee> GetByIdAsync(int id);
    Task CreateAsync(Employee emp, string performedByUserId);
    Task UpdateAsync(Employee emp, string performedByUserId);
    Task DeleteAsync(int id, string performedByUserId);
}

public interface IActivityLogService
{
    Task LogAsync(string userId, string action, string dataJson);
}
```

### 3.1 CurrentUserService + MemoryCache

```csharp
public class CurrentUserService : ICurrentUserService
{
    private readonly IPermissionService _permissionService;
    private readonly IMemoryCache _cache;
    private readonly int _userId;
    private readonly string _userName;

    public CurrentUserService(IPermissionService permissionService, IMemoryCache cache, int userId, string userName)
    {
        _permissionService = permissionService; _cache = cache; _userId = userId; _userName = userName;
    }

    public int UserId => _userId;
    public string UserName => _userName;

    public Task<List<string>> GetPermissionsAsync()
    {
        return _cache.GetOrCreateAsync($"permissions_{_userId}", async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
            return await _permissionService.GetUserPermissionsAsync(_userId);
        });
    }
}
```

### 3.2 EmployeeService + ActivityLog + Serilog JSON

```csharp
public class EmployeeService : IEmployeeService
{
    private readonly MyDbContext _db;
    private readonly IActivityLogService _logService;

    public EmployeeService(MyDbContext db, IActivityLogService logService)
    {
        _db = db; _logService = logService;
    }

    public async Task<List<Employee>> GetAllAsync() => await _db.Employees.Include("Department").ToListAsync();
    public async Task<Employee> GetByIdAsync(int id) => await _db.Employees.Include("Department").FirstOrDefaultAsync(e => e.Id == id);

    public async Task CreateAsync(Employee emp, string performedByUserId)
    {
        _db.Employees.Add(emp); await _db.SaveChangesAsync();
        await _logService.LogAsync(performedByUserId, "CreateEmployee", JsonConvert.SerializeObject(emp));
    }

    public async Task UpdateAsync(Employee emp, string performedByUserId)
    {
        _db.Entry(emp).State = EntityState.Modified; await _db.SaveChangesAsync();
        await _logService.LogAsync(performedByUserId, "UpdateEmployee", JsonConvert.SerializeObject(emp));
    }

    public async Task DeleteAsync(int id, string performedByUserId)
    {
        var emp = await _db.Employees.FindAsync(id);
        if (emp != null)
        {
            _db.Employees.Remove(emp); await _db.SaveChangesAsync();
            await _logService.LogAsync(performedByUserId, "DeleteEmployee", JsonConvert.SerializeObject(emp));
        }
    }
}
````

### 3.3 ActivityLogService + Serilog

```csharp
public class ActivityLogService : IActivityLogService
{
    private readonly ILogger _logger;
    public ActivityLogService()
    {
        _logger = new LoggerConfiguration()
            .WriteTo.File("logs/activity.json", rollingInterval: RollingInterval.Day)
            .Enrich.WithProperty("Application", "WinFormApp")
            .CreateLogger();
    }

    public async Task LogAsync(string userId, string action, string dataJson)
    {
        var logEntry = new { UserId = userId, Action = action, Data = dataJson, Timestamp = DateTime.UtcNow };
        _logger.Information("{@LogEntry}", logEntry);
        await Task.CompletedTask;
    }
}
````

## 4. Autofac DI 容器註冊範例

```csharp
var builder = new ContainerBuilder();

builder.RegisterType<MyDbContext>().InstancePerLifetimeScope();
builder.RegisterType<ActivityLogService>().As<IActivityLogService>().SingleInstance();
builder.RegisterType<EmployeeService>().As<IEmployeeService>().InstancePerLifetimeScope();
builder.RegisterType<PermissionService>().As<IPermissionService>().InstancePerLifetimeScope();
builder.Register(c => new MemoryCache(new MemoryCacheOptions())).As<IMemoryCache>().SingleInstance();

// 假設登入時取得 userId / userName
int currentUserId = 1; string currentUserName = "Alice";
builder.RegisterType<CurrentUserService>()
       .As<ICurrentUserService>()
       .WithParameter("userId", currentUserId)
       .WithParameter("userName", currentUserName)
       .SingleInstance();

var container = builder.Build();
```

## 5. WinForm 權限控制

```csharp
private async void Form_Load(object sender, EventArgs e)
{
    var currentUser = container.Resolve<ICurrentUserService>();
    var perms = await currentUser.GetPermissionsAsync();

    btnAdd.Enabled = perms.Contains("EmployeeCreate");
    btnEdit.Enabled = perms.Contains("EmployeeUpdate");
    btnDelete.Enabled = perms.Contains("EmployeeDelete");
}
```

## ✅ 設計重點

* **整合 RBAC + ABAC**：部門權限 + UserGroup 特殊權限
* **CRUD + 日誌**：Serilog JSON 格式，包含使用者、動作、資料
* **MemoryCache 加速權限查詢**
* **安全資料**：PBKDF2 密碼、AES 敏感欄位
* **Autofac DI**：跨模組呼叫統一 CurrentUserService
* **Fluent API**：EF6 資料表關聯設計完整

# IMemoryCache?

你問得很好，`IMemoryCache` 是 **.NET Core / .NET 5+** 系統內建的記憶體快取介面，但你目前是 **.NET Framework 4.6.1**（WinForm + EF6），所以原生的 `IMemoryCache` 是沒有的。

在 .NET Framework 4.6.1 上，你有幾種選擇：

## 1. 使用 `System.Runtime.Caching.MemoryCache`（推薦）

.NET Framework 自帶 `MemoryCache` 類別，功能跟 `IMemoryCache` 類似：

```csharp
using System.Runtime.Caching;

public class MemoryCacheService
{
    private readonly ObjectCache _cache = MemoryCache.Default;

    public T GetOrCreate<T>(string key, Func<T> factory, TimeSpan? expiration = null)
    {
        if (_cache.Contains(key))
            return (T)_cache[key];

        var value = factory();
        var policy = new CacheItemPolicy { AbsoluteExpiration = DateTimeOffset.Now.Add(expiration ?? TimeSpan.FromMinutes(30)) };
        _cache.Set(key, value, policy);
        return value;
    }
}
```

用法：

```csharp
var cacheService = new MemoryCacheService();
var permissions = cacheService.GetOrCreate("permissions_1", () => LoadUserPermissionsFromDb(1));
```

## 2. 使用 NuGet 套件 `Microsoft.Extensions.Caching.Memory`

這個套件是 .NET Standard / .NET Core 風格的 `IMemoryCache`，也能用在 .NET Framework 4.6.1：

```powershell
Install-Package Microsoft.Extensions.Caching.Memory
```

使用方式和我之前示範的 `IMemoryCache` 一樣：

```csharp
using Microsoft.Extensions.Caching.Memory;

var cache = new MemoryCache(new MemoryCacheOptions());
var permissions = cache.GetOrCreate("permissions_1", entry => {
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
    return LoadUserPermissionsFromDb(1);
});
```

### 建議

* 如果只是在 WinForm + EF6 專案內使用快取，**使用 System.Runtime.Caching.MemoryCache 就足夠**
* 如果想要和現代 ASP.NET Core 或 DI 整合，則用 **Microsoft.Extensions.Caching.Memory**

好的，我幫你把整合範例裡的 `IMemoryCache` 改成 **.NET Framework 4.6.1 的 `MemoryCache`**，這樣 WinForm 專案就可以直接用。

### 1. MemoryCacheService 封裝

```csharp
using System;
using System.Runtime.Caching;

public class MemoryCacheService
{
    private readonly ObjectCache _cache = MemoryCache.Default;

    public T GetOrCreate<T>(string key, Func<T> factory, TimeSpan? expiration = null)
    {
        if (_cache.Contains(key))
            return (T)_cache[key];

        var value = factory();
        var policy = new CacheItemPolicy
        {
            AbsoluteExpiration = DateTimeOffset.Now.Add(expiration ?? TimeSpan.FromMinutes(30))
        };
        _cache.Set(key, value, policy);
        return value;
    }
}
```

### 2. CurrentUserService 改用 MemoryCacheService

```csharp
public class CurrentUserService
{
    private readonly IPermissionService _permissionService;
    private readonly MemoryCacheService _cache;
    private readonly int _userId;
    private readonly string _userName;

    public CurrentUserService(IPermissionService permissionService, MemoryCacheService cache, int userId, string userName)
    {
        _permissionService = permissionService;
        _cache = cache;
        _userId = userId;
        _userName = userName;
    }

    public int UserId => _userId;
    public string UserName => _userName;

    public async Task<List<string>> GetPermissionsAsync()
    {
        return _cache.GetOrCreate($"permissions_{_userId}", () =>
        {
            // 由同步方式取得權限，因為 MemoryCache 不支援 async lambda
            return _permissionService.GetUserPermissionsAsync(_userId).GetAwaiter().GetResult();
        }, TimeSpan.FromMinutes(30));
    }
}
```

### 3. Autofac 註冊範例

```csharp
var builder = new ContainerBuilder();

builder.RegisterType<MyDbContext>().InstancePerLifetimeScope();
builder.RegisterType<ActivityLogService>().As<IActivityLogService>().SingleInstance();
builder.RegisterType<EmployeeService>().As<IEmployeeService>().InstancePerLifetimeScope();
builder.RegisterType<PermissionService>().As<IPermissionService>().InstancePerLifetimeScope();
builder.RegisterType<MemoryCacheService>().SingleInstance();

// 假設登入時取得 userId / userName
int currentUserId = 1; string currentUserName = "Alice";
builder.RegisterType<CurrentUserService>()
       .WithParameter("userId", currentUserId)
       .WithParameter("userName", currentUserName)
       .AsSelf()
       .SingleInstance();

var container = builder.Build();
```

### 4. 使用範例（WinForm 權限控制）

```csharp
private async void Form_Load(object sender, EventArgs e)
{
    var currentUser = container.Resolve<CurrentUserService>();
    var perms = await currentUser.GetPermissionsAsync();

    btnAdd.Enabled = perms.Contains("EmployeeCreate");
    btnEdit.Enabled = perms.Contains("EmployeeUpdate");
    btnDelete.Enabled = perms.Contains("EmployeeDelete");
}
```

這樣整個系統就可以在 **.NET 4.6.1 WinForm** 下使用快取，不需要依賴 .NET Core 的 `IMemoryCache`，權限查詢也會被快取加速。

如果你要，我可以幫你再把 **PermissionService** 的權限查詢邏輯也改成快取版本，讓權限查詢更快。

你希望我做嗎？

好的，我幫你把 **PermissionService** 改成使用 MemoryCacheService 快取權限，這樣每次查詢使用者權限就不用每次都打 DB，提高效能。

### 1. PermissionService 改寫

```csharp
public class PermissionService : IPermissionService
{
    private readonly MyDbContext _db;
    private readonly MemoryCacheService _cache;

    public PermissionService(MyDbContext db, MemoryCacheService cache)
    {
        _db = db;
        _cache = cache;
    }

    public async Task<List<string>> GetUserPermissionsAsync(int userId)
    {
        // 使用快取
        return _cache.GetOrCreate($"permissions_{userId}", () =>
        {
            // 取得員工
            var employee = _db.Employees
                .Include("Department.Permissions")
                .Include("UserGroups.Permissions")
                .FirstOrDefault(e => e.Id == userId);

            if (employee == null) return new List<string>();

            // 部門權限
            var deptPerms = employee.Department?.Permissions.Select(p => p.Name).ToList() ?? new List<string>();

            // UserGroup 特殊權限
            var groupPerms = employee.UserGroups
                .SelectMany(ug => ug.Permissions)
                .Select(p => p.Name)
                .ToList();

            // 合併去重
            return deptPerms.Union(groupPerms).Distinct().ToList();
        }, TimeSpan.FromMinutes(30));
    }
}
```

### 2. Autofac 註冊更新

```csharp
var builder = new ContainerBuilder();

builder.RegisterType<MyDbContext>().InstancePerLifetimeScope();
builder.RegisterType<ActivityLogService>().As<IActivityLogService>().SingleInstance();
builder.RegisterType<EmployeeService>().As<IEmployeeService>().InstancePerLifetimeScope();
builder.RegisterType<PermissionService>().As<IPermissionService>().InstancePerLifetimeScope();
builder.RegisterType<MemoryCacheService>().SingleInstance();

int currentUserId = 1;
string currentUserName = "Alice";
builder.RegisterType<CurrentUserService>()
       .WithParameter("userId", currentUserId)
       .WithParameter("userName", currentUserName)
       .AsSelf()
       .SingleInstance();

var container = builder.Build();
```

### 3. 使用範例（WinForm）

```csharp
private async void Form_Load(object sender, EventArgs e)
{
    var currentUser = container.Resolve<CurrentUserService>();
    var perms = await currentUser.GetPermissionsAsync();

    btnAdd.Enabled = perms.Contains("EmployeeCreate");
    btnEdit.Enabled = perms.Contains("EmployeeUpdate");
    btnDelete.Enabled = perms.Contains("EmployeeDelete");
}
```

這樣每個使用者的權限 **第一次查詢會讀 DB**，之後 30 分鐘內都由快取提供，提高系統效能。

# 快取自動更新/清除機制

當員工、部門或 UserGroup 權限變動時，自動刷新 MemoryCache，確保權限即時生效。

## 1. MemoryCacheService 加入快取移除方法

```csharp
using System;
using System.Runtime.Caching;

public class MemoryCacheService
{
    private readonly ObjectCache _cache = MemoryCache.Default;

    public T GetOrCreate<T>(string key, Func<T> factory, TimeSpan? expiration = null)
    {
        if (_cache.Contains(key))
            return (T)_cache[key];

        var value = factory();
        var policy = new CacheItemPolicy
        {
            AbsoluteExpiration = DateTimeOffset.Now.Add(expiration ?? TimeSpan.FromMinutes(30))
        };
        _cache.Set(key, value, policy);
        return value;
    }

    public void Remove(string key)
    {
        if (_cache.Contains(key))
            _cache.Remove(key);
    }

    public void RemoveByPrefix(string prefix)
    {
        foreach (var item in _cache)
        {
            if (item.Key.StartsWith(prefix))
                _cache.Remove(item.Key);
        }
    }
}
```

## 2. PermissionService 加入快取刷新方法

```csharp
public class PermissionService : IPermissionService
{
    private readonly MyDbContext _db;
    private readonly MemoryCacheService _cache;

    public PermissionService(MyDbContext db, MemoryCacheService cache)
    {
        _db = db;
        _cache = cache;
    }

    public async Task<List<string>> GetUserPermissionsAsync(int userId)
    {
        return _cache.GetOrCreate($"permissions_{userId}", () =>
        {
            var employee = _db.Employees
                .Include("Department.Permissions")
                .Include("UserGroups.Permissions")
                .FirstOrDefault(e => e.Id == userId);

            if (employee == null) return new List<string>();

            var deptPerms = employee.Department?.Permissions.Select(p => p.Name).ToList() ?? new List<string>();
            var groupPerms = employee.UserGroups
                .SelectMany(ug => ug.Permissions)
                .Select(p => p.Name)
                .ToList();

            return deptPerms.Union(groupPerms).Distinct().ToList();
        }, TimeSpan.FromMinutes(30));
    }

    // 當員工權限變動時呼叫
    public void RefreshUserPermissions(int userId)
    {
        _cache.Remove($"permissions_{userId}");
    }

    // 當部門權限變動時呼叫，刷新該部門所有員工
    public void RefreshDepartmentPermissions(int departmentId)
    {
        var userIds = _db.Employees
            .Where(e => e.DepartmentId == departmentId)
            .Select(e => e.Id)
            .ToList();

        foreach (var id in userIds)
            _cache.Remove($"permissions_{id}");
    }

    // 當 UserGroup 權限變動時呼叫，刷新所有屬於該群組的員工
    public void RefreshUserGroupPermissions(int userGroupId)
    {
        var userIds = _db.Employees
            .Where(e => e.UserGroups.Any(ug => ug.Id == userGroupId))
            .Select(e => e.Id)
            .ToList();

        foreach (var id in userIds)
            _cache.Remove($"permissions_{id}");
    }
}
```

## 3. CRUD 操作時觸發快取刷新

例如在 EmployeeService 增加/修改員工：

```csharp
public class EmployeeService : IEmployeeService
{
    private readonly MyDbContext _db;
    private readonly PermissionService _permissionService;
    private readonly IActivityLogService _logService;

    public EmployeeService(MyDbContext db, PermissionService permissionService, IActivityLogService logService)
    {
        _db = db;
        _permissionService = permissionService;
        _logService = logService;
    }

    public async Task UpdateEmployeeAsync(Employee emp, int performedByUserId)
    {
        _db.Entry(emp).State = EntityState.Modified;
        await _db.SaveChangesAsync();

        // 刷新該員工快取
        _permissionService.RefreshUserPermissions(emp.Id);

        await _logService.LogAsync(performedByUserId, "UpdateEmployee", JsonConvert.SerializeObject(emp));
    }

    public async Task UpdateDepartmentPermissionsAsync(int departmentId, List<int> permissionIds, int performedByUserId)
    {
        var dept = _db.Departments.Include("Permissions").First(d => d.Id == departmentId);
        dept.Permissions.Clear();
        var perms = _db.Permissions.Where(p => permissionIds.Contains(p.Id)).ToList();
        foreach (var p in perms) dept.Permissions.Add(p);
        await _db.SaveChangesAsync();

        // 刷新該部門所有員工快取
        _permissionService.RefreshDepartmentPermissions(departmentId);

        await _logService.LogAsync(performedByUserId, "UpdateDepartmentPermissions", JsonConvert.SerializeObject(new { DepartmentId = departmentId, Permissions = permissionIds }));
    }

    public async Task UpdateUserGroupPermissionsAsync(int userGroupId, List<int> permissionIds, int performedByUserId)
    {
        var group = _db.UserGroups.Include("Permissions").First(g => g.Id == userGroupId);
        group.Permissions.Clear();
        var perms = _db.Permissions.Where(p => permissionIds.Contains(p.Id)).ToList();
        foreach (var p in perms) group.Permissions.Add(p);
        await _db.SaveChangesAsync();

        // 刷新該群組所有員工快取
        _permissionService.RefreshUserGroupPermissions(userGroupId);

        await _logService.LogAsync(performedByUserId, "UpdateUserGroupPermissions", JsonConvert.SerializeObject(new { UserGroupId = userGroupId, Permissions = permissionIds }));
    }
}
```

這樣設計後：

* **權限查詢快取**：每個使用者權限最多 30 分鐘快取一次
* **即時刷新**：當員工、部門或 UserGroup 權限變動時，會自動移除對應快取，下次查詢就會重新載入 DB
* **效能 + 正確性兼顧**：提高系統效能，同時確保權限變更即時生效

如果你願意，我可以幫你把 **整個 WinForm UI** 改成直接使用這套快取自動更新機制，按 CRUD 操作就會自動刷新使用者權限，並立即影響按鈕顯示/隱藏。

# 操作員工、部門或 UserGroup 權限時，會自動刷新使用者快取，並立即影響按鈕權限

以下是一個範例架構：

## 1. Form 欄位與 DI

```csharp
public partial class MainForm : Form
{
    private readonly EmployeeService _employeeService;
    private readonly CurrentUserService _currentUser;
    private readonly PermissionService _permissionService;

    public MainForm(EmployeeService employeeService, CurrentUserService currentUser, PermissionService permissionService)
    {
        InitializeComponent();
        _employeeService = employeeService;
        _currentUser = currentUser;
        _permissionService = permissionService;
    }
}
```

## 2. Form Load 權限控制

```csharp
private async void MainForm_Load(object sender, EventArgs e)
{
    await RefreshUIPermissionsAsync();
}

private async Task RefreshUIPermissionsAsync()
{
    var perms = await _currentUser.GetPermissionsAsync();

    btnAddEmployee.Enabled = perms.Contains("EmployeeCreate");
    btnEditEmployee.Enabled = perms.Contains("EmployeeUpdate");
    btnDeleteEmployee.Enabled = perms.Contains("EmployeeDelete");

    btnEditDeptPermissions.Enabled = perms.Contains("DepartmentUpdate");
    btnEditGroupPermissions.Enabled = perms.Contains("UserGroupUpdate");
}
```

## 3. CRUD 操作呼叫快取刷新

### 3.1 更新員工

```csharp
private async void btnEditEmployee_Click(object sender, EventArgs e)
{
    var emp = GetSelectedEmployeeFromUI();
    emp.Name = txtName.Text;

    await _employeeService.UpdateEmployeeAsync(emp, _currentUser.UserId);

    await RefreshUIPermissionsAsync(); // 立即刷新當前使用者權限
}
```

### 3.2 更新部門權限

```csharp
private async void btnEditDeptPermissions_Click(object sender, EventArgs e)
{
    int deptId = GetSelectedDepartmentId();
    var permissionIds = GetSelectedPermissionIdsFromUI();

    await _employeeService.UpdateDepartmentPermissionsAsync(deptId, permissionIds, _currentUser.UserId);

    await RefreshUIPermissionsAsync();
}
```

### 3.3 更新 UserGroup 權限

```csharp
private async void btnEditGroupPermissions_Click(object sender, EventArgs e)
{
    int groupId = GetSelectedUserGroupId();
    var permissionIds = GetSelectedPermissionIdsFromUI();

    await _employeeService.UpdateUserGroupPermissionsAsync(groupId, permissionIds, _currentUser.UserId);

    await RefreshUIPermissionsAsync();
}
```

## 4. 特點

* **權限快取**：每個使用者快取一次，效能佳
* **自動刷新**：CRUD 操作後，立即刷新對應快取
* **即時 UI 控制**：按鈕顯示/隱藏會依權限即時更新
* **跨模組安全**：CurrentUserService + PermissionService 統一管理權限
* **日誌完整**：所有 CRUD 動作會透過 Serilog JSON 紀錄

# PBKDF2 密碼儲存與驗證以及 AES 加密敏感欄位，並整合到剛才的 WinForm + EF6 專案中

## 1. PBKDF2 密碼儲存與驗證

### 1.1 密碼工具類別

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public static class PasswordHelper
{
    private const int SaltSize = 16; // 128 bit
    private const int KeySize = 32;  // 256 bit
    private const int Iterations = 10000;

    public static string HashPassword(string password)
    {
        using (var rng = new RNGCryptoServiceProvider())
        {
            byte[] salt = new byte[SaltSize];
            rng.GetBytes(salt);

            using (var pbkdf2 = new Rfc2898DeriveBytes(password, salt, Iterations, HashAlgorithmName.SHA256))
            {
                var key = pbkdf2.GetBytes(KeySize);
                var hashBytes = new byte[SaltSize + KeySize];
                Buffer.BlockCopy(salt, 0, hashBytes, 0, SaltSize);
                Buffer.BlockCopy(key, 0, hashBytes, SaltSize, KeySize);
                return Convert.ToBase64String(hashBytes);
            }
        }
    }

    public static bool VerifyPassword(string password, string hashedPassword)
    {
        var hashBytes = Convert.FromBase64String(hashedPassword);
        byte[] salt = new byte[SaltSize];
        Buffer.BlockCopy(hashBytes, 0, salt, 0, SaltSize);

        using (var pbkdf2 = new Rfc2898DeriveBytes(password, salt, Iterations, HashAlgorithmName.SHA256))
        {
            var key = pbkdf2.GetBytes(KeySize);
            for (int i = 0; i < KeySize; i++)
            {
                if (hashBytes[i + SaltSize] != key[i])
                    return false;
            }
            return true;
        }
    }
}
```

### 1.2 EF6 Employee Entity 密碼欄位

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PasswordHash { get; set; }  // PBKDF2 hash
    public int DepartmentId { get; set; }
    public virtual Department Department { get; set; }
    public virtual ICollection<UserGroup> UserGroups { get; set; }
}
```

## 2. AES 加密敏感欄位

### 2.1 AES 工具類別

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public static class AESHelper
{
    private static readonly byte[] Key = Encoding.UTF8.GetBytes("12345678901234567890123456789012"); // 32 bytes
    private static readonly byte[] IV = Encoding.UTF8.GetBytes("1234567890123456"); // 16 bytes

    public static string Encrypt(string plainText)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            using (var encryptor = aes.CreateEncryptor())
            using (var ms = new MemoryStream())
            {
                using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                using (var sw = new StreamWriter(cs))
                {
                    sw.Write(plainText);
                }
                return Convert.ToBase64String(ms.ToArray());
            }
        }
    }

    public static string Decrypt(string cipherText)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            using (var decryptor = aes.CreateDecryptor())
            using (var ms = new MemoryStream(Convert.FromBase64String(cipherText)))
            using (var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read))
            using (var sr = new StreamReader(cs))
            {
                return sr.ReadToEnd();
            }
        }
    }
}
```

### 2.2 EF6 Entity 範例（加密欄位）

```csharp
public class EmployeeProfile
{
    public int Id { get; set; }
    public int EmployeeId { get; set; }
    public virtual Employee Employee { get; set; }

    public string EncryptedPhone { get; set; }  // AES 加密
    public string EncryptedAddress { get; set; }

    [NotMapped]
    public string Phone
    {
        get => AESHelper.Decrypt(EncryptedPhone);
        set => EncryptedPhone = AESHelper.Encrypt(value);
    }

    [NotMapped]
    public string Address
    {
        get => AESHelper.Decrypt(EncryptedAddress);
        set => EncryptedAddress = AESHelper.Encrypt(value);
    }
}
```

## 3. CRUD 範例（員工密碼 + 個人資料）

```csharp
public async Task AddEmployeeAsync(string name, string password, string phone, string address, int deptId, int performedByUserId)
{
    var emp = new Employee
    {
        Name = name,
        PasswordHash = PasswordHelper.HashPassword(password),
        DepartmentId = deptId
    };
    _db.Employees.Add(emp);
    await _db.SaveChangesAsync();

    var profile = new EmployeeProfile
    {
        EmployeeId = emp.Id,
        Phone = phone,
        Address = address
    };
    _db.EmployeeProfiles.Add(profile);
    await _db.SaveChangesAsync();

    await _logService.LogAsync(performedByUserId, "AddEmployee", JsonConvert.SerializeObject(new { emp.Id, emp.Name }));
}
```

## 4. 登入驗證範例

```csharp
public Employee Login(string username, string password)
{
    var emp = _db.Employees.FirstOrDefault(e => e.Name == username);
    if (emp != null && PasswordHelper.VerifyPassword(password, emp.PasswordHash))
        return emp;

    return null;
}
```

這樣整個系統就：

* **安全密碼存儲**：PBKDF2
* **敏感資料加密**：AES
* **CRUD + 快取刷新 + UI 即時更新 + 日誌**
* 完整整合到 **WinForm + EF6 + Autofac** 架構

# 可直接運行的完整 WinForm 範例專案

整合以下功能：

* 員工登入與驗證（PBKDF2 密碼）
* 員工個人資料（AES 加密電話與地址）
* 部門權限管理（RBAC）
* UserGroup 權限管理（ABAC）
* CRUD 操作自動刷新快取與 UI 按鈕權限
* Serilog JSON 日誌紀錄所有操作
* 使用 EF6 + Fluent API
* Autofac DI

## 1. 專案架構

```csharp
MyApp
│
├─ Data
│   ├─ MyDbContext.cs
│   ├─ Entities
│   │   ├─ Employee.cs
│   │   ├─ EmployeeProfile.cs
│   │   ├─ Department.cs
│   │   ├─ Permission.cs
│   │   ├─ UserGroup.cs
│
├─ Services
│   ├─ MemoryCacheService.cs
│   ├─ PermissionService.cs
│   ├─ EmployeeService.cs
│   ├─ CurrentUserService.cs
│   ├─ ActivityLogService.cs
│   ├─ AuthService.cs
│
├─ UI
│   ├─ LoginForm.cs
│   ├─ MainForm.cs
│
├─ Program.cs
├─ AutofacConfig.cs
```

## 2. 登入畫面 LoginForm

```csharp
public partial class LoginForm : Form
{
    private readonly AuthService _authService;

    public LoginForm(AuthService authService)
    {
        InitializeComponent();
        _authService = authService;
    }

    private void btnLogin_Click(object sender, EventArgs e)
    {
        var username = txtUsername.Text;
        var password = txtPassword.Text;

        var user = _authService.Login(username, password);
        if (user != null)
        {
            // 登入成功，開啟 MainForm
            var mainForm = Program.Container.Resolve<MainForm>(
                new NamedParameter("currentUserId", user.Id),
                new NamedParameter("currentUserName", user.Name)
            );
            mainForm.Show();
            this.Hide();
        }
        else
        {
            MessageBox.Show("帳號或密碼錯誤");
        }
    }
}
```

## 3. MainForm

* 顯示員工列表、部門與 UserGroup 權限
* CRUD 操作後立即刷新快取與 UI
* 按鈕依使用者權限動態啟用

```csharp
public partial class MainForm : Form
{
    private readonly EmployeeService _employeeService;
    private readonly CurrentUserService _currentUser;
    private readonly PermissionService _permissionService;

    public MainForm(EmployeeService employeeService, CurrentUserService currentUser, PermissionService permissionService)
    {
        InitializeComponent();
        _employeeService = employeeService;
        _currentUser = currentUser;
        _permissionService = permissionService;
    }

    private void MainForm_Load(object sender, EventArgs e)
    {
        RefreshUIPermissions();
        LoadEmployeeList();
    }

    private void RefreshUIPermissions()
    {
        var perms = _currentUser.GetPermissions();
        btnAddEmployee.Enabled = perms.Contains("EmployeeCreate");
        btnEditEmployee.Enabled = perms.Contains("EmployeeUpdate");
        btnDeleteEmployee.Enabled = perms.Contains("EmployeeDelete");
        btnEditDeptPermissions.Enabled = perms.Contains("DepartmentUpdate");
        btnEditGroupPermissions.Enabled = perms.Contains("UserGroupUpdate");
    }

    private void LoadEmployeeList()
    {
        dgvEmployees.DataSource = _employeeService.GetAllEmployees();
    }

    private async void btnAddEmployee_Click(object sender, EventArgs e)
    {
        var empName = txtName.Text;
        var password = txtPassword.Text;
        var phone = txtPhone.Text;
        var address = txtAddress.Text;
        var deptId = (int)cmbDepartments.SelectedValue;

        await _employeeService.AddEmployeeAsync(empName, password, phone, address, deptId, _currentUser.UserId);
        LoadEmployeeList();
        RefreshUIPermissions();
    }

    // 編輯/刪除員工與部門/UserGroup CRUD 同理
}
```

## 4. AuthService（登入驗證）

```csharp
public class AuthService
{
    private readonly MyDbContext _db;

    public AuthService(MyDbContext db) => _db = db;

    public Employee Login(string username, string password)
    {
        var emp = _db.Employees.FirstOrDefault(e => e.Name == username);
        if (emp != null && PasswordHelper.VerifyPassword(password, emp.PasswordHash))
            return emp;
        return null;
    }
}
```

## 5. Autofac DI 註冊範例

```csharp
var builder = new ContainerBuilder();
builder.RegisterType<MyDbContext>().InstancePerLifetimeScope();
builder.RegisterType<MemoryCacheService>().SingleInstance();
builder.RegisterType<PermissionService>().SingleInstance();
builder.RegisterType<ActivityLogService>().SingleInstance();
builder.RegisterType<EmployeeService>().SingleInstance();
builder.RegisterType<AuthService>().SingleInstance();
builder.RegisterType<CurrentUserService>()
       .WithParameter("userId", 0)
       .WithParameter("userName", "")
       .InstancePerDependency();
builder.RegisterType<LoginForm>();
builder.RegisterType<MainForm>();

Program.Container = builder.Build();
Application.Run(Program.Container.Resolve<LoginForm>());
```

## 6. 特點

* **完整登入與權限控制**（RBAC + ABAC）
* **密碼與敏感資料加密**（PBKDF2 + AES）
* **CRUD + 快取刷新 + UI 即時更新**
* **Serilog JSON 日誌**
* **EF6 + Fluent API**
* **Autofac DI**，跨模組可使用 CurrentUserService 取得使用者資訊與權限

# Serilog 日誌直接以 JSON 格式輸出

## 1. 安裝 Serilog JSON 套件

NuGet 安裝：

```csharp
Serilog
Serilog.Sinks.File
Serilog.Sinks.Console
Serilog.Formatting.Json
```

## 2. Serilog 設定 JSON 輸出

```csharp
using Serilog;
using Serilog.Formatting.Json;

public static class LogConfig
{
    public static void Init()
    {
        Log.Logger = new LoggerConfiguration()
            .Enrich.FromLogContext()
            .WriteTo.Console(new JsonFormatter())  // JSON 格式輸出到 Console
            .WriteTo.File(new JsonFormatter(), "logs/log-.json", rollingInterval: RollingInterval.Day)
            .CreateLogger();
    }
}
```

> 這樣所有輸出都會是 **標準 JSON 格式**，每一筆日誌都是獨立 JSON 物件。

## 3. ActivityLogService

```csharp
using Newtonsoft.Json;

public class ActivityLogService
{
    public async Task LogAsync(int userId, string action, object data = null)
    {
        var logEntry = new
        {
            Timestamp = DateTime.Now,
            UserId = userId,
            Action = action,
            Data = data
        };

        Log.Information("{@LogEntry}", logEntry);  // Serilog 會自動轉成 JSON
        await Task.CompletedTask;
    }
}
```

## 4. 使用範例（登入、CRUD）

```csharp
// 登入
_logService.LogAsync(user.Id, "Login", new { Username = user.Name });

// 新增員工
await _logService.LogAsync(currentUserId, "AddEmployee", new { emp.Id, emp.Name, Phone = phone, Address = address });

// 編輯員工
await _logService.LogAsync(currentUserId, "EditEmployee", emp);

// 刪除員工
await _logService.LogAsync(currentUserId, "DeleteEmployee", emp);
```

## 5. 輸出範例（logs/log-YYYYMMDD.json）

```json
{
  "Timestamp": "2025-11-06T14:32:21.123+08:00",
  "UserId": 1,
  "Action": "AddEmployee",
  "Data": {
    "Id": 101,
    "Name": "王小明",
    "Phone": "0988123456",
    "Address": "台北市中正區"
  }
}
{
  "Timestamp": "2025-11-06T14:35:10.456+08:00",
  "UserId": 2,
  "Action": "Login",
  "Data": {
    "Username": "林小華"
  }
}
```

* 每筆日誌都是 JSON 物件，可直接用 Python、Power BI、ElasticSearch 等分析
* Rolling 日誌，每天生成新檔案
