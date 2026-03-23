---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# RBAC（Role-Based Access Control）角色導向權限模型概念與實作邏輯

## 一、核心概念

RBAC 的思想是：
「使用者（User）不直接擁有權限（Permission），而是透過角色（Role）間接取得。」

這種設計讓權限管理更有彈性與可維護性，避免逐一修改使用者權限的混亂。

## 二、主要元素

1. **User（使用者）**
    系統中的實際操作人員。
    範例：Alice、Bob

2. **Role（角色）**
    對應某類身份或職責。
    範例：Admin、Editor、Viewer

3. **Permission（權限）**
    對應系統操作的細項。
    範例：CreatePost、DeleteUser、ReadReport

4. **Session（會話）**
    使用者登入後的活動期間。
    - 一個使用者在一個會話中可以啟用多個角色。

## 三、關聯邏輯

1. **User → Role**
   使用者可被指派一個或多個角色。

```csharp
Alice → [Admin, Editor]
```

2. **Role → Permission**
   角色擁有多項操作權限。

```csharp
Editor → [CreatePost, EditPost]
```

3. **權限檢查（Authorization）流程**
   系統檢查使用者是否透過角色擁有該權限。

```json
if user.Roles contains any role where role.Permissions includes "DeleteUser"
    → Authorized
else
    → Forbidden
```

## 四、範例架構（邏輯示意）

```csharp
User
    └── Role
            └── Permission
```

範例資料：

```json
User: Alice
Roles: [Admin, Editor]
Permissions(Admin): [ManageUser, DeleteUser]
Permissions(Editor): [CreatePost, EditPost]
```

授權檢查邏輯：

```csharp
Alice 想執行 DeleteUser
→ 系統檢查 Alice 的角色
→ Admin 角色有 DeleteUser 權限
→ 授權通過
```

## 五、RBAC 模型的層級版本

1. **RBAC0（基本模型）**
    User–Role–Permission 的基本結構。

2. **RBAC1（角色階層）**
    支援角色繼承，例如 Manager 繼承 Employee 的權限。

3. **RBAC2（約束模型）**
    增加安全限制，例如：
    - 同一使用者不能同時是 Auditor 與 Operator。
    - 強制職責分離。

4. **RBAC3（完整模型）**
    結合 RBAC1 + RBAC2。

## 六、實作思路（ASP.NET Core 為例）

1. 定義角色（RoleManager）
2. 定義權限（可自訂或以 Claims 表示）
3. 將角色與權限綁定（通常存於資料庫）
4. 登入後建立 ClaimsPrincipal
5. 在控制器或 API 層進行授權檢查：

```csharp
[Authorize(Roles = "Admin")]
[Authorize(Policy = "CanDeleteUser")]
```

## 七、延伸思考

RBAC 的侷限是「角色爆炸」——系統成長後，角色太多導致維護困難。
現代替代方案包含：

- **ABAC（Attribute-Based Access Control）**：依屬性決定權限，例如使用者部門、時間、地點。
- **PBAC（Policy-Based Access Control）**：以政策規則判斷授權條件。

---

# RBAC 範例（.NET 4.6.1 + EF6 + Fluent API 完整設定）

## 1. 資料庫模型

```csharp
public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string PasswordHash { get; set; }
    public virtual ICollection<Role> Roles { get; set; }
}

public class Role
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<User> Users { get; set; }
    public virtual ICollection<Permission> Permissions { get; set; }
}

public class Permission
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Role> Roles { get; set; }
}
```

## 2. DbContext 與 Fluent API 設定

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext() : base("name=AppDbContext") { }

    public DbSet<User> Users { get; set; }
    public DbSet<Role> Roles { get; set; }
    public DbSet<Permission> Permissions { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // User Table
        modelBuilder.Entity<User>()
            .ToTable("Users")
            .HasKey(u => u.Id);

        modelBuilder.Entity<User>()
            .Property(u => u.Username)
            .IsRequired()
            .HasMaxLength(50);

        modelBuilder.Entity<User>()
            .Property(u => u.PasswordHash)
            .IsRequired()
            .HasMaxLength(128);

        // Role Table
        modelBuilder.Entity<Role>()
            .ToTable("Roles")
            .HasKey(r => r.Id);

        modelBuilder.Entity<Role>()
            .Property(r => r.Name)
            .IsRequired()
            .HasMaxLength(50);

        // Permission Table
        modelBuilder.Entity<Permission>()
            .ToTable("Permissions")
            .HasKey(p => p.Id);

        modelBuilder.Entity<Permission>()
            .Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(50);

        // User-Role Many-to-Many
        modelBuilder.Entity<User>()
            .HasMany(u => u.Roles)
            .WithMany(r => r.Users)
            .Map(m =>
            {
                m.ToTable("UserRoles");
                m.MapLeftKey("UserId");
                m.MapRightKey("RoleId");
            });

        // Role-Permission Many-to-Many
        modelBuilder.Entity<Role>()
            .HasMany(r => r.Permissions)
            .WithMany(p => p.Roles)
            .Map(m =>
            {
                m.ToTable("RolePermissions");
                m.MapLeftKey("RoleId");
                m.MapRightKey("PermissionId");
            });
    }
}
```

## 3. 初始化測試資料

```csharp
using (var db = new AppDbContext())
{
    var adminRole = new Role { Name = "Admin" };
    var userRole = new Role { Name = "User" };

    var permCreate = new Permission { Name = "CreateItem" };
    var permDelete = new Permission { Name = "DeleteItem" };

    adminRole.Permissions = new List<Permission> { permCreate, permDelete };
    userRole.Permissions = new List<Permission> { permCreate };

    var adminUser = new User { Username = "admin", PasswordHash = "hashed_password" };
    adminUser.Roles = new List<Role> { adminRole };

    var normalUser = new User { Username = "user1", PasswordHash = "hashed_password" };
    normalUser.Roles = new List<Role> { userRole };

    db.Roles.Add(adminRole);
    db.Roles.Add(userRole);
    db.Users.Add(adminUser);
    db.Users.Add(normalUser);

    db.SaveChanges();
}
```

## 4. 權限檢查範例

```csharp
public bool HasPermission(string username, string permissionName)
{
    using (var db = new AppDbContext())
    {
        var user = db.Users
            .Include("Roles.Permissions")
            .FirstOrDefault(u => u.Username == username);

        if (user == null) return false;

        return user.Roles.Any(r => r.Permissions.Any(p => p.Name == permissionName));
    }
}
```

這個範例完整展示了 RBAC 的資料結構、Fluent API 設定、初始化資料，以及簡單的權限檢查邏輯，適用於 .NET 4.6.1 + EF6。
