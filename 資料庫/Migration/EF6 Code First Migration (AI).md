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

# ⭐ EF6 Code First Migration 教學

回答你要的三件事：

1. 📁 **檔案要放哪裡？**
2. ▶ **如何執行？**
3. 🔄 **事前、事後的工作是什麼？**

完全依照「你已知道用途，只想知道怎麼做」的角度來寫。

## 1️⃣ 檔案放在哪裡？

在 EF6 專案中，一般會有一個資料夾：

```csharp
YourProject
 ├─ Models/
 ├─ Migrations/   <-- Migration 會放這裡
 ├─ Context/
```

👉 **Migration 都放在 `Migrations` 資料夾（由 EF 自動建立）**

如果你第一次執行 Migration，它會自動生成：

```csharp
Migrations
 ├─ 202311150123456_InitialCreate.cs
 ├─ Configuration.cs (若使用 Enable-Migrations 產生)
```

所以你不需要手動建立檔案，**只要跑指令即可產生**。

## 2️⃣ 如何執行（全部指令逐步示範）

以下指令請在 **Package Manager Console** 中執行
（路徑：`Tools > NuGet Package Manager > Package Manager Console`）

### ❗ 第一次使用 Migration：請執行 Enable-Migrations

```powershell
Enable-Migrations
```

🔧 作用：

* 建立 `Migrations` 資料夾
* 建立 `Configuration.cs`（內含 Seed() 方法）

> **只需做一次**，專案之後就不用再做了。

### ➤ 產生 Migration（每次模型變更後）

例如你新增 CarbonCopy 導覽屬性：

```powershell
Add-Migration AddEmployeeCarbonCopy
```

會生成：

```csharp
Migrations
 ├─ 202511201234567_AddEmployeeCarbonCopy.cs
```

它會包含：

* Up()
* Down()

你可以修改內容（允許）

### ➤ 套用 Migration 到資料庫

```powershell
Update-Database
```

資料庫就會同步更新。

## 3️⃣ 事前工作、事後工作（重點說明）

## 📌 事前工作（Before）

### ✔ (1) 確保 DbContext 已設定連線字串（Web.config 或 App.config）

例如：

```xml
<connectionStrings>
  <add name="MyDbContext"
       connectionString="Data Source=.;Initial Catalog=MyDB;Integrated Security=True"
       providerName="System.Data.SqlClient"/>
</connectionStrings>
```

### ✔ (2) Model（Entity）寫好

例如你剛剛調整過 Employee class。

### ✔ (3) OnModelCreating 的 Fluent API 寫好

例如：

```csharp
modelBuilder.Entity<Employee>()
    .HasMany(e => e.CarbonCopies)
    .WithMany(e => e.CarbonCopiedBy)
    .Map(m =>
    {
        m.ToTable("EmployeeCarbonCopies");
        m.MapLeftKey("EmployeeId");
        m.MapRightKey("CarbonCopyEmployeeId");
    });
```

### ✔ (4) 建議：把所有 ICollection 在 constructor 裡初始化

（避免 NRE）

## 📌 事後工作（After）

### ✔ (1) 檢查資料庫確實更新

檢查新的 Table / Column 是否存在。

### ✔ (2) 若需要，修改 Seed()

位於：

```csharp
Migrations/Configuration.cs
```

例如加入初始員工資料：

```csharp
protected override void Seed(MyDbContext context)
{
    if (!context.Employees.Any())
    {
        context.Employees.Add(new Employee { EmployeeId = "001", EmployeeName = "Admin" });
    }
}
```

### ✔ (3) 部屬後，需要在正式機執行 Update-Database

若你不想手動執行，可啟用自動 Migration（不建議正式機）

## 4️⃣ 常見問題（你將會遇到的）

### ❓ 1. Migration 找不到

需要在 Package Manager Console 先選擇專案：

```csharp
Default project:  選你的 EF 專案
```

### ❓ 2. 產生的 Migration 空白？

表示你沒有任何模型改動。

### ❓ 3. 資料庫與模型衝突

你可以加上：

```powershell
Update-Database -Verbose
```

來觀察 SQL 執行過程。

## 5️⃣ 最完整的工作流程（實務）

你今天修改 Employee entity → 加上 CarbonCopy
👉 你的正確流程如下：

### (1) 修改 Model（你已完成）

### (2) 修改 Fluent API（你已完成）

### (3) 移除多餘的 CarbonCopys（你已確認）

### (4) 產生 Migration

```powershell
Add-Migration AddEmployeeCarbonCopy
```

### (5) 套用 Migration

```powershell
Update-Database
```

### (6) 檢查資料庫中是否多了一張

```csharp
EmployeeCarbonCopies
```

### (7) 若需要，更新 Seed()

---

# ⭐ 你在開發機做 Migration → 會不會影響部署的資料庫？正式環境會發生什麼？

## ✔ 結論先講（最重要）

**在開發機執行 Migration (`Add-Migration`、`Update-Database`)
只會改你的開發機資料庫，不會自動影響正式機。**

正式機資料庫**絕對不會自己變更**。
要更新正式機，你必須手動或自動部署 Migration。

## 1️⃣ 你在開發機做 Migration 時，實際發生什麼？

你在開發機做這些：

```powershell
Add-Migration AddEmployeeCarbonCopy
Update-Database
```

發生的事：

### ✔ A. 產生 Migration 檔（.cs 檔）

放在：

```csharp
Migrations/
   2025xxxx_AddEmployeeCarbonCopy.cs
```

**這個檔案是「變更記錄」，要部署到正式環境的核心。**

### ✔ B. 執行 `Update-Database` 只更新你本機的資料庫

也就是你的 **LocalDB / 開發用 SQL Server**。

⚠️ **完全不會改到正式機**
⚠️ **完全不會自動同步到正式機**

## 2️⃣ 那正式機的資料庫會怎麼改變？（最重要的一段）

正式環境的資料庫，預設是不會自動變更的。

正式機要同步資料庫 Schema，你有三種方式：

### ✔ 方式 A：在正式機執行 Migration（最推薦 & 最標準）

到正式機伺服器上執行：

```powershell
Update-Database
```

條件：

* 正式機有你的 Web 專案
* 連線字串指向正式 DB
* 有 PM Console 可以跑（IIS Server 沒有）

通常不會在正式機跑 PM，所以更常用方式是 B。

### ✔ 方式 B：部署時「自動執行 Migration」

如果你啟用：

```csharp
Database.SetInitializer(
    new MigrateDatabaseToLatestVersion<MyDbContext, Configuration>());
```

**當程式在正式機啟動時，會自動更新資料庫 Schema。**

這是 EF 官方功能，但需要注意：

⚠️ 正式機若更新時出錯 → 整站無法啟動
⚠️ 正式機若有大量資料、表鎖定 → 有風險

**一般企業正式環境不常使用自動 Migration**

### ✔ 方式 C：把 Migration 產生的 SQL Script 部署到正式機（最安全）

使用：

```powershell
Update-Database -Script
```

它會產生一份 SQL：

```csharp
ALTER TABLE ...
CREATE TABLE EmployeeCarbonCopies ...
```

你把這份 SQL：

* 寄給 DBA
* 交給 SA
* 或用 SQL Server Management Studio 套用

這是企業最常用方式。

## 3️⃣ 那「執行環境」（正式機）到底會怎麼變？

完全取決於 **你怎麼部署 Migration**。

| 部署方式                    | 正式機會發生什麼               |
| -- | - |
| ✔ 你沒有做任何 Migration 部署   | 正式 DB 什麼都不會變           |
| ✔ 你手動執行 Update-Database | 正式 DB 結構會更新            |
| ✔ 啟用自動 Migration        | 正式站啟動時會自動更新            |
| ✔ 使用 SQL Script         | 正式 DB 被 DBA 套用 SQL 後更新 |

## 4️⃣ 實務建議流程（工程團隊通用）

### 🔧 開發階段

1. 修改 Model
2. 執行 `Add-Migration`
3. 執行 `Update-Database`（本機）

### 🚀 部署到正式機

1. `Add-Migration` 建立 Migration 檔
2. `Update-Database -Script` 產生 SQL
3. 把 SQL 交給 DBA 或可控時段套用
4. 將程式部署到正式機
5. 程式啟動 → 使用新的 DB Schema

## 5️⃣ 回到你的專案情境（你剛加了 CarbonCopy）

你的正式機會：

* ⚠️ **完全不會自動新增 EmployeeCarbonCopies 這張表**
* ⚠️ **也不會自動加欄位**
* ⚠️ **也不會自動改資料內容**

你的開發機改變的是：

* 本機 DB 多了一張 EmployeeCarbonCopies
* Migration 檔產生好了（這才是要部署到正式機的）

## 🎯 最後總結（超重要）

### ✔ 在開發機做 Migration 不會自動影響正式機

### ✔ 正式機資料庫要自己部署 Migration

### ✔ 最安全方法：產生 SQL Script 讓 DBA 套用

`MigrateDatabaseToLatestVersion<TContext, TMigrationsConfiguration>`
是 **EF6 內建的自動套用 Migration 的 Database Initializer**。

簡單說：

> **當程式啟動、第一次建立 DbContext 時，EF 會自動套用尚未執行的 Migrations，到資料庫。**

這可讓程式在「正式環境」或「測試環境」自動更新資料庫 schema，而不需要手動跑 Update-Database。

## ⭐ 一、最完整寫法（推薦）

放在 **Global.asax → Application_Start()**：

```csharp
protected void Application_Start()
{
    Database.SetInitializer(
        new MigrateDatabaseToLatestVersion<MyDbContext, MyDbMigrationsConfiguration>());

    // 其他設定...
}
```

其中：

* `MyDbContext` → 你的 DbContext
* `MyDbMigrationsConfiguration` → `Migrations/Configuration.cs` 裡的類別

你的 Configuration.cs 通常長這樣：

```csharp
internal sealed class MyDbMigrationsConfiguration 
    : DbMigrationsConfiguration<MyDbContext>
{
    public MyDbMigrationsConfiguration()
    {
        AutomaticMigrationsEnabled = false;
    }

    protected override void Seed(MyDbContext context)
    {
        // Optional: 你想放初始資料可寫在這裡
    }
}
```

✔ 這樣程式啟動時 → EF 會檢查 MigrationsHistory 表，執行未套用的 Migration。

## ⭐ 二、在 Console App / Windows Service 的寫法（等同）

放在程式啟動前：

```csharp
Database.SetInitializer(
    new MigrateDatabaseToLatestVersion<MyDbContext, MyDbMigrationsConfiguration>());
```

然後建立你的 DbContext 就會自動套用 Migration。

## ⭐ 三、注意事項（實務上非常重要）

### ⚠ 1. 若正式機資料量很大，Migration 會造成啟動延遲

因為程式啟動時會套用 Migration。

如果 Migration 很重（ALTER TABLE、加 column、index…）
網站可能：

* 啟動很慢
* timeout
* 卡住
* 甚至佔鎖導致服務暫停

### ⚠ 2. Migration 若寫錯 → 正式機整站不能啟動

因為 DB 套用失敗，DbContext 建立不了。

### ⚠ 3. 多台 Web Server（Load Balancer）容易發生 race condition

如果有兩台 WebServer 同時啟動：

* Server A：正在套 Migration
* Server B：還沒套 → 建立 DbContext → 失敗

企業正式機 **通常不會開啟自動 Migration**。

## ⭐ 四、最佳實務（企業級流程）

### ✔ 開發機

* `Add-Migration`
* `Update-Database`

### ✔ 部署前

* `Update-Database -Script` → 產生 SQL
* 給 DBA 套用

### ✔ 正式機

* 程式不使用 MigrateDatabaseToLatestVersion
* 不允許自動更新 Schema

## ⭐ 五、如果你堅持要開啟自動 Migration（例如內部小系統）

你可以加上：

```csharp
Database.SetInitializer(
    new MigrateDatabaseToLatestVersion<MyDbContext, MyDbMigrationsConfiguration>());
```

然後在 web.config 加上：

```xml
<appSettings>
    <add key="DatabaseAutoMigrate" value="true" />
</appSettings>
```

程式中做條件判斷，只在 Debug 或 QA 用：

```csharp
bool migrate = bool.Parse(ConfigurationManager.AppSettings["DatabaseAutoMigrate"]);

if (migrate)
{
    Database.SetInitializer(
        new MigrateDatabaseToLatestVersion<MyDbContext, MyDbMigrationsConfiguration>());
}
else
{
    Database.SetInitializer<NullDatabaseInitializer<MyDbContext>>();
}
```

---

# Migration 產生 SQL Script → 正式機手動套用

## ⭐ 整體流程（非常簡單，只有三步）

1. **在開發機產生 Migration Script（.sql）**
2. **把 Script 放到正式機（通常是跳板機或連線到正式 DB）**
3. **用 SQL Server Management Studio（SSMS）執行這份 script**

下面我會示範每個步驟。

### 1️⃣ 在開發機產生 SQL Script

在 Visual Studio → Package Manager Console 執行：

```powershell
Update-Database -Script -SourceMigration:$InitialDatabase
```

這會產生完整 SQL：

* CREATE TABLE
* ALTER TABLE
* INDEX
* etc.

然後會跳出 SQL editor → 你存檔，例如：

```csharp
2025-11-20_AddCarbonCopy.sql
```

這就是「你要丟到正式機執行」的檔案。

### 2️⃣ 把 SQL Script 放到正式機

正式機通常：

* 不能跑 Visual Studio
* 不能跑 EF Migration 指令
* 但可以用 SSMS 連線資料庫

你可以：

✔ 用 RDP 連線正式機 → 開 SSMS
或
✔ 你在自己的電腦上用 SSMS 直接連線正式 DB（如果你有權限）

把 SQL Script 檔案複製到正式機或直接在你電腦上執行（只要能連線）。

### 3️⃣ 使用 SSMS 手動套用 SQL Script（最重要步驟）

#### 打開 SQL Server Management Studio（SSMS）

1. **連線到正式資料庫**
2. 展開你的資料庫
3. 右鍵資料庫 → **New Query**
4. 將你產生的 `.sql` 內容貼進 query editor
5. 按下 **Execute**

## ⭐ 執行後你應該看到什麼？

成功時會看到：

```csharp
Command(s) completed successfully.
```

或者每個 migration step 會出現：

```csharp
(1 row(s) affected)
```

如果是建立表格：

```csharp
CREATE TABLE [dbo].[EmployeeCarbonCopies] ...
```

代表 Migration 已成功部署。

## ⭐ 推薦的流程（實務 Swift & Safe SOP）

1. **開發機：**

   * 修改模型
   * `Add-Migration`
   * `Update-Database`（本機 DB）
   * `Update-Database -Script` 產生 SQL

2. **檢查 Script（快速看）**

   * 是否有 DROP TABLE（危險）
   * 是否有 DROP COLUMN（需確認）
   * ALTER TABLE 是否合理
   * CREATE TABLE 是否正確

3. **正式機：**

   * SSMS 開 query
   * 貼 Script
   * 執行 Execute
   * 確認成功訊息

## ⭐ 為什麼 SQL Script 是最安全的方法？

因為：

* 你可以「看得見即將發生的變更」
* 正式機不會自動改結構（避免程式啟動時爆炸）
* 你是唯一管理者，能掌控時機
* 任何錯誤都會第一時間看到

大部分企業都要求：

✔ 開發 → Migration
✔ DBA → 執行 SQL Script

你現在是一人作業，就是開發者兼 DBA，非常 OK！

---

# 使用 SQLite + EF6

Migration 可以做，但和 SQL Server 完全不同，因為：

> **SQLite 的 ALTER TABLE 功能很弱，只能做最基本的變動。**
> 例如：
> ✔ 新增欄位
> ❌ 不能刪欄
> ❌ 不能修改欄位型別
> ❌ 不能刪除/修改外鍵
> ❌ 很多 Migration 指令會失敗

因此 **SQLite 的 Migration 行為（尤其是 EF6）會受到限制**，你需要遵循特定流程。

下面我會給你：

⭐ 一、EF6 + SQLite 是否支援 Migration？
⭐ 二、如何執行 Migration？（步驟）
⭐ 三、SQLite Migration 的限制
⭐ 四、實務建議工作流程
⭐ 五、替代方案（最實用）

## ⭐ 一、EF6 + SQLite 是否支援 Migration？

支援，但前提是：

* 你使用的 SQLite Provider 要支援 Migration。
* EF6 原生不支援 SQLite，你必須使用第三方 provider，例如：

### ✔ System.Data.SQLite

（官方 SQLite provider）

### ✔ Devart SQLite

（商業版，支援度最好）

其中 **System.Data.SQLite** 的 Migration 支援度最弱，但仍能跑出 Migration SQL。

## ⭐ 二、如何在 SQLite 執行 Migration？

### ✔ Step 1：產生 Migration（與 SQL Server 完全相同）

```powershell
Add-Migration AddEmployeeCarbonCopy
```

SQLite 的 Migration 檔會產生，但內容可能會被「簡化」。

### ✔ Step 2：產生 SQL Script

```powershell
Update-Database -Script
```

你會得到 `.sql`，例如：

```csharp
ALTER TABLE Employees ADD COLUMN SomeColumn TEXT;
CREATE TABLE EmployeeCarbonCopies (...);
```

### ✔ Step 3：將 SQL Script 套到 SQLite DB

SQLite 不用 SSMS，而是使用專用工具：

### 可用 GUI 工具

✔ DB Browser for SQLite（最推薦）
✔ SQLiteStudio
✔ DBeaver

### 實際操作

1. 開啟 DB Browser for SQLite
2. 選擇打開正式機的 DB 檔（通常是 `.sqlite` 或 `.db`）
3. 點選 **Execute SQL**
4. 貼上 Migration SQL Script
5. 按下「Execute」

完成！

## ⭐ 三、SQLite Migration 的限制（超重要）

SQLite 的 ALTER TABLE 功能限制如下：

| 操作 | 支援？ |
| -- | |
| ADD COLUMN | ✔ 支援 |
| RENAME TABLE | ✔ 支援 |
| RENAME COLUMN | ✔ 支援（SQLite 3.25+） |
| CREATE TABLE | ✔ 支援 |
| DROP COLUMN | ❌ 不支援 |
| ALTER COLUMN TYPE | ❌ 不支援 |
| DROP FOREIGN KEY | ❌ 不支援 |
| MODIFY CONSTRAINT | ❌ 不支援 |

這導致 EF6 產出的 Migration，**常常不能直接用在 SQLite**。

## ⭐ 四、實務 SQLite Migration 工作流程（建議）

如果你使用 SQLite 並且要用 Migration，我建議：

### ✔（1）把 Migration 產生出來

```powershell
Add-Migration MyChange
```

### ✔（2）產生 SQL Script

```powershell
Update-Database -Script
```

### ✔（3）手動修改 Script

**把不支援的 ALTER 移除 / 改為重新建立表格**

例如 EF 產生：

```csharp
ALTER TABLE Employees DROP COLUMN OldColumn;
```

→ 替換為：

```csharp
-- SQLite 不支援 DROP COLUMN
-- 手動流程

CREATE TABLE Employees_temp AS SELECT Id, Name FROM Employees;
DROP TABLE Employees;
ALTER TABLE Employees_temp RENAME TO Employees;
```

SQLite 官方建議：

> **當你需要刪欄、改欄位型別 → 重建整個表格**

### ✔（4）手動執行 Script（DB Browser for SQLite）

## ⭐ 五、如果你不想手改 Migration（推薦替代方案）

因為 SQLite 對 Migration 支援太差，實務上許多開發者這樣做：

### ✔ 方案 A：使用自動刪除 DB + 重建（開發環境才適用）

```csharp
Database.SetInitializer(
    new DropCreateDatabaseIfModelChanges<MyDbContext>());
```

SQLite 小專案常用，但正式機不適合。

### ✔ 方案 B：使用 FluentMigrator 等 SQLite 友善工具（最佳）

EF 的 Migration 不適合 SQLite。
轉用：

* FluentMigrator
* Flyway
* DbUp

它們能「針對 SQLite 能力」產出正確 SQL。

### ✔ 方案 C：換成 EF Core（SQLite Migration 完整支援）

EF Core 對 SQLite Migration 的支援比 EF6 好太多。

如果你使用 EF Core：

* Migration 完整支援 SQLite
* 不需要手改
* 可自動建立新表格

## 🎯 最後總結

| 問題 | 答案 |
| - | |
| SQLite 能否跑 EF6 Migration？ | ✔ 可以，但功能有限 |
| Migration SQL 要怎麼套用？ | ✔ 用 DB Browser for SQLite 執行 |
| Migration SQL 會不會失敗？ | ⚠ 很多 ALTER 指令會失敗，需要手動修改 |
| 最佳實務是什麼？ | ✔ 產生 Script → 手動調整 → 在 SQLite 上執行 |

---

# 如果 Employee 的 CarbonCopy 關聯要變更時，EF6 Code First Migration 的流程與變化，並說明可能會發生的 SQL 與程式層影響

## 1️⃣ 變更情境範例

假設你想對 CarbonCopy 關聯做其中一種變更：

1. **新增欄位**

   * 比如：在 EmployeeCarbonCopies 表加一個 `IsActive` 欄位
2. **改變關聯性**

   * 比如：原本 Cascade Delete 改成不刪除
3. **刪除欄位或關聯**

   * 例如：移除某些 CarbonCopy 記錄

## 2️⃣ 流程概述（EF6 Code First）

假設你要做 **新增 IsActive 欄位**：

### Step 1：修改 Model

```csharp
public class Employee
{
    // 原有
    public virtual ICollection<Employee> CarbonCopies { get; set; }

    // 新增中間表屬性
    // EF6 多對多無法直接新增中間表欄位，這種情況要改成實體化的中間表：
}

public class EmployeeCarbonCopy
{
    public int EmployeeId { get; set; }
    public int CarbonCopyId { get; set; }
    public bool IsActive { get; set; } = true;

    public virtual Employee Employee { get; set; }
    public virtual Employee CarbonCopy { get; set; }
}
```

> EF6 的多對多 **不支援在中間表新增欄位**，所以要改成「明確實體化中間表」。

### Step 2：新增 Migration

```powershell
Add-Migration UpdateCarbonCopy
```

EF6 會自動比對 Model 與資料庫：

* 如果只是多對多關聯改變，EF6 會產生：
  * Drop & Recreate Join Table（有些情況會先 Drop 舊表，再建新表）
* 如果是新增欄位 → 建立新欄位並更新 FK

Migration 會生成新的檔案：

```csharp
Migrations/
    20251120_AddEmployeeCarbonCopy.cs
    20251121_UpdateCarbonCopy.cs
```

### Step 3：套用 Migration

```powershell
Update-Database
```

EF6 會依照 Migration 檔依序套用：

* 先執行 20251120_AddEmployeeCarbonCopy
* 再執行 20251121_UpdateCarbonCopy

> 注意：多對多關聯改動可能會刪除舊資料 → 如果資料重要，需要手動備份 / 寫 Seed Script 保留資料。

## 3️⃣ SQL 變化範例

如果是 **新增中間表欄位 IsActive**：

```sql
ALTER TABLE EmployeeCarbonCopies ADD IsActive BIT NOT NULL DEFAULT(1);
```

如果是 **改 Cascade Delete**：

```sql
ALTER TABLE EmployeeCarbonCopies
DROP CONSTRAINT FK_EmployeeCarbonCopies_Employees_EmployeeId;

ALTER TABLE EmployeeCarbonCopies
ADD CONSTRAINT FK_EmployeeCarbonCopies_Employees_EmployeeId
FOREIGN KEY (EmployeeId) REFERENCES Employees(Id) ON DELETE NO ACTION;
```

如果 EF6 自動重建 Join Table：

```sql
CREATE TABLE EmployeeCarbonCopies_New (
    EmployeeId INT NOT NULL,
    CarbonCopyId INT NOT NULL,
    IsActive BIT NOT NULL DEFAULT(1),
    CONSTRAINT PK_EmployeeCarbonCopies_New PRIMARY KEY(EmployeeId, CarbonCopyId)
);

INSERT INTO EmployeeCarbonCopies_New(EmployeeId, CarbonCopyId)
SELECT EmployeeId, CarbonCopyId FROM EmployeeCarbonCopies;

DROP TABLE EmployeeCarbonCopies;

EXEC sp_rename 'EmployeeCarbonCopies_New', 'EmployeeCarbonCopies';
```

> 這種情況下舊資料可以保留，但欄位有新增。

## 4️⃣ 注意事項

1. **多對多改成有欄位的中間表**

   * EF6 原生多對多 Join Table 無法加欄位
   * 改成「實體化中間表」是唯一正確做法

2. **資料保護**

   * EF6 Migration Drop & Recreate Join Table → 舊資料可能丟失
   * 需要手動在 Migration 裡加 `Sql("INSERT INTO ...")` 保留資料

3. **版本控制**

   * 每次修改 CarbonCopy 關聯，都要產生新的 Migration
   * 檔名清楚，例如 `AddIsActiveToCarbonCopy` / `UpdateCarbonCopyCascadeDelete`

4. **正式機部署**

   * 先產生 SQL Script → 審核 → 執行
   * 盡量避免自動 Migration 在正式機上跑，因為多對多重建可能造成資料短暫丟失

## 5️⃣ 建議流程總結

1. **修改 Model**（必要時實體化中間表）
2. **Add-Migration 新檔案**
3. **檢查 Migration 檔**（Up / Down）
4. **Update-Database 或產生 SQL Script**
5. **正式機執行 SQL Script**（備份資料庫）

> 每次變更 CarbonCopy 都建議走 Migration + SQL Script 流程，確保資料安全。

---

# EF6 Migration 跨專案 DbContext

EF6 找的 **是 `DbContext`，不是單純的 Entity class**。你現在情況是：

* `TaskPulse.Entity` → 只有實體 (Employee, TaskPulse…)
* `TaskPulse.Core` → 放 `DbContext`（例如 `TaskPulseDbContext`）
* `TaskPulse` → 應用程式入口（exe）

所以如果你在 **`TaskPulse.Entity` 專案** 執行 `Enable-Migrations`，EF6 **找不到 DbContext** 就會報錯：

```text
No context type was found in the assembly 'TaskPulse'.
```

## 解法：跨專案指定 DbContext

你需要告訴 EF6 **Migration 應該用哪個專案的 DbContext**，命令如下：

```powershell
Enable-Migrations -ProjectName TaskPulse.Core -StartUpProjectName TaskPulse
```

* `-ProjectName TaskPulse.Core` → 放置 Migration 的專案（也就是包含 DbContext 的專案）
* `-StartUpProjectName TaskPulse` → EF 會用啟動專案的 App.config 找 connection string

新增 Migration：

```powershell
Add-Migration AddEmployeeCarbonCopy -ProjectName TaskPulse.Core -StartUpProjectName TaskPulse
```

更新資料庫：

```powershell
Update-Database -ProjectName TaskPulse.Core -StartUpProjectName TaskPulse
```

## 額外注意事項

1. **Migration 檔會生成在 `TaskPulse.Core`**

   * 如果你想把 Migration 放在另一個專案（例如 Entity 專案），DbContext 必須在那個專案，否則 EF6 找不到。

2. **Connection String**

   * EF6 Migration 使用 **啟動專案的 App.config / Web.config**
   * 確保 DbContext 的建構子名稱與 connection string 名稱一致：

```csharp
public class TaskPulseDbContext : DbContext
{
    public TaskPulseDbContext() : base("name=TaskPulseDb") { }
}
```

3. **跨專案 Migration 流程**

   * DbContext 在 Core
   * Migration 在 Core
   * 實體在 Entity
   * 執行 Migration 時 EF6 會自動引用 Entity 專案的類別

✅ **總結**

* EF6 Migration 找的是 **DbContext**，不是 Entity
* DbContext 在 Core → Migration 必須放在 Core
* 執行 Migration 時要指定 `-ProjectName TaskPulse.Core -StartUpProjectName TaskPulse`
* Entity 專案不需要直接生成 Migration
