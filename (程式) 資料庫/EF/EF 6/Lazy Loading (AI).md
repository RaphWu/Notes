---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - 資料庫
  - EF6
  - EF_Lazy_Loading
---

# ===== ChatGPT =====

## Lazy Loading 意義

Lazy Loading 是延遲載入導覽屬性資料的技術，當你第一次存取導航屬性時，EF 才自動去資料庫查詢，避免一次載入過多資料。

## Lazy Loading 優點

- 減少初始查詢資料量，提高載入速度
- 程式碼簡潔，不用手動 Include
- 需要時才查詢，節省資源

## Lazy Loading 缺點

- 容易造成 N+1 查詢效能問題
- 隱藏查詢時機，除錯較難
- 序列化 (JSON/XML) 可能觸發額外查詢或例外
- 需要虛擬屬性與 Proxy，增加限制

## Lazy Loading 用法

```csharp
public class Device
{
    public int Id { get; set; }
    public int BrandId { get; set; }
    public virtual Brand Brand { get; set; } // virtual 啟用 Lazy Loading
}
```

```csharp
using (var ctx = new MyContext())
{
    var device = ctx.Devices.First();
    var brandName = device.Brand.Name; // 此時才查詢 Brand
}
```

## Lazy Loading 使用情境（適合）

- 導覽屬性不一定會用到
- 小型系統或單筆讀取
- 程式碼可讀性高優先於極致效能
- 想讓程式碼更直覺

## Lazy Loading 不建議使用情境

- 批次處理、多筆資料
- 需要一次載入所有關聯資料
- Web API 序列化輸出
- 高效能、高可控查詢系統
- 離線資料處理 (Detached Entity)

## Lazy Loading 注意事項

- 導覽屬性必須為 `virtual`
- Context 必須維持開啟 (未 Detach)
- 可能產生大量隱形 SQL 查詢
- 可搭配 `Include()` 手動載入避免 N+1
- 可透過 `context.Configuration.LazyLoadingEnabled = false;` 全域關閉
- 使用 JSON 或 ViewModel 時通常避免 Lazy

## Lazy Loading 與 Eager Loading 比較

|項目|Lazy Loading|Eager Loading (Include)|
|---|---|---|
|查詢時間|延後|立即|
|SQL 次數|多次|一次或少次|
|程式碼簡單|高|中|
|效能可控性|低|高|
|適用情境|單筆、偶爾使用|批次、多筆、輸出|

## 建議實務做法

1. Domain Model 可開啟 Lazy Loading 提升易用性
2. Service / Repository 層用 Include 控制查詢
3. API / ViewModel 禁用 Lazy，轉成 DTO
4. 批次操作務必用 Eager Loading 或 Explicit Loading
5. 驗證是否有 N+1 查詢 (SQL Profiler / Debug Output)

## 總結

Lazy Loading 是「方便但有風險」的功能
適合「小量、單筆、快速開發」
不適合「大量、批次、高效能、API 輸出」
最佳實務：**混合使用、由開發者掌控載入策略**

---

# Lazy Loading 混合使用的實務做法

## 1. Domain/Entity 層保留 Lazy Loading

- 導覽屬性加 `virtual`，Context 保持開啟
- 方便單筆查詢或偶爾存取關聯屬性

```csharp
public class Device
{
    public int Id { get; set; }
    public int BrandId { get; set; }
    public virtual Brand Brand { get; set; } // 可延遲載入
}
```

## 2. Service / Repository 層統一控制查詢

- 批次或列表查詢用 Include 預先載入
- 避免迴圈中觸發多次 SQL（N+1 問題）

```csharp
var devices = context.Devices
    .Include(d => d.Brand)
    .Include(d => d.Condition)
    .ToList();
```

## 3. 單筆明細查詢可用 Lazy

- 顯示單筆資料時，可直接用 Lazy Loading 取得導航屬性

```csharp
var device = context.Devices.Find(id);
var brandName = device.Brand.Name; // 自動載入
```

## 4. API / DTO / JSON 輸出避免 Lazy

- 導覽屬性不直接序列化，轉 DTO 或手動 Include

```csharp
var dto = context.Devices
    .Include(d => d.Brand)
    .Select(d => new DeviceDto
    {
        Id = d.Id,
        BrandName = d.Brand.Name
    })
    .ToList();
```

## 5. 注意 Context 生命周期

- Lazy Loading 必須在 Context 存活期間使用
- 若 Context 已 Dispose，導航屬性存取會出錯

## 6. 實務原則

- 小量資料或單筆查詢 → Lazy Loading，程式簡潔
- 批量查詢、列表、報表 → Include 明確載入
- DTO/API → 全部用 Include 或投影，避免 Lazy

> 混合策略核心概念：**Domain 保留延遲載入，Service/Repository 層決定何時一次載入，API/外部輸出則完全掌控查詢**

---

# ===== Gemini =====

## Entity Framework 6 (EF6) Lazy Loading 資訊總整理

## 1. 意義 (Definition)

在 Entity Framework 6 中，Lazy Loading 是一種機制，用於**延遲載入**導覽屬性 (Navigation Properties) 中的相關資料（一對多、多對多關係中的相關實體），直到您**實際存取**這些屬性時，EF 才會自動發出必要的資料庫查詢。

- **舉例：** 當您從資料庫載入一個 `Department` (部門) 實體時，它所關聯的 `Employees` (員工) 集合並不會立即載入。只有當您的程式碼嘗試存取 `Department.Employees` 這個集合時，EF 才會自動向資料庫發出額外的查詢來取得員工資料。

## 2. 優點 (Advantages)

|**優點**|**說明**|
|---|---|
|**減少初始查詢負載**|程式碼初次載入主實體時，資料庫查詢更快，回傳的資料量更小，減少不必要的傳輸。|
|**資源按需分配**|只有當應用程式真正需要使用相關資料時，才進行資料庫查詢，節省記憶體和處理資源。|
|**簡化程式碼**|開發者不必手動編寫 `Include()` 或手動查詢相關資料的程式碼，提高了開發效率。|

## 3. 缺點與注意事項 (Disadvantages & Cautions)

|**缺點/注意事項**|**說明**|
|---|---|
|**N+1 查詢問題 (最主要缺點)**|如果在一個迴圈中存取多個實體的導覽屬性，Lazy Loading 可能會為每個實體都發出一個額外的查詢，導致執行 $N$ 次查詢，加上初始的 $1$ 次查詢，即為「N+1 查詢問題」。這會極大地影響應用程式效能。|
|**資料庫連線長時間開啟**|Lazy Loading 需要資料庫連線仍然有效才能執行後續的查詢。在 Web 應用程式中，這可能導致不必要的連線時間延長，特別是在處理週期較長的物件時。|
|**序列化問題 (常見於 Web API)**|在將實體序列化（例如轉為 JSON）傳輸到客戶端時，如果啟用了 Lazy Loading，序列化器可能會嘗試存取導覽屬性，這會觸發 Lazy Loading，並可能導致以下問題： 1. **無限迴圈：** 如果存在雙向導覽屬性（父指向子，子指向父），可能導致序列化無限遞迴。 2. **不必要的查詢：** 載入大量不打算傳輸給客戶端的數據。|
|**須設定為 `virtual`**|為了讓 Lazy Loading 生效，EF 必須能夠建立代理類別 (Proxy Classes) 來覆寫（Override）導覽屬性的 getter/setter，因此導覽屬性必須聲明為 `public` 和 `virtual`。|
|**實體必須被追蹤**|Lazy Loading 只對被 `DbContext` 追蹤的實體起作用。一旦實體被 Detach (分離) 或 Context 被 Dispose (銷毀)，Lazy Loading 將失效。|

## 4. 用法 (Implementation)

在 EF6 中，實現 Lazy Loading 需要以下兩個條件：

### 步驟 1：啟用 Proxy Creation（預設為開啟）

在您的 `DbContext` 中，確保 `Configuration.ProxyCreationEnabled` 設定為 `true`。

```csharp
public class MyDbContext : DbContext
{
    public MyDbContext() : base("name=MyConnectionString")
    {
        // 預設為 true，通常無需手動設定
        this.Configuration.LazyLoadingEnabled = true; 
        this.Configuration.ProxyCreationEnabled = true;
    }
    // ... DbSet 屬性
}
```

### 步驟 2：設定導覽屬性為 `virtual`

您想要延遲載入的導覽屬性（不管是單個實體還是一個集合），都必須聲明為 `public` 和 **`virtual`**。

```csharp
public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }

    // 啟用了 Lazy Loading 的集合屬性
    public virtual ICollection<Employee> Employees { get; set; } 
}

public class Employee
{
    public int EmployeeId { get; set; }
    public string Name { get; set; }
    public int DepartmentId { get; set; }

    // 啟用了 Lazy Loading 的單個實體屬性
    public virtual Department Department { get; set; } 
}
```

## 5. 使用情境 (Use Cases)

雖然 N+1 問題使 Lazy Loading 飽受詬病，但在某些情境下它仍是合適的：

|**情境**|**說明**|
|---|---|
|**單一實體的詳細檢視**|當您載入一個實體，且可能只需要存取其一兩個相關實體時，Lazy Loading 是便利的。例如，載入一個 `Order`，然後在頁面下方可能需要顯示 `Customer` 資訊。|
|**很少使用到的導覽屬性**|如果某個導覽屬性很少被存取，啟用 Lazy Loading 可以避免在每次查詢主實體時都載入它。|
|**原型設計與小型應用**|在開發初期或資料量極小、效能要求不高的應用程式中，可以利用 Lazy Loading 快速開發。|

## 6. 替代方案：急切載入 (Eager Loading) 與顯式載入 (Explicit Loading)

為了避免 Lazy Loading 帶來的效能問題，EF 鼓勵開發者多使用以下兩種方式來控制資料載入：

| **載入方式**                    | **用法/情境**                                                                                                   | **說明**                            |
| --------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **急切載入 (Eager Loading)**    | 使用 `Include()` 方法。適用於：**明確知道需要哪些相關資料**，且要避免 N+1 查詢。                                                         | 通過一個查詢（通常是 JOIN）一次性載入主實體及其所有相關實體。 |
| **顯式載入 (Explicit Loading)** | 使用 `Entry().Reference().Load()` 或 `Entry().Collection().Load()`。適用於：**只有在特定條件下才需要**相關資料，或在實體載入後過了一段時間才決定載入。 | 手動告訴 Context 載入特定導覽屬性，這會發出獨立的查詢。  |

**最佳實踐：** 在對效能要求高的場景，應該明確使用 **急切載入 (`Include()`)** 來精確控制查詢的結果和效率，並考慮在全域禁用 Lazy Loading 來強制開發者做出明確選擇。

---

# 全域禁用 Lazy Loading

**「全域禁用 Lazy Loading」** 主要包含您提到的兩點，並且通常是為了防止應用程式中潛在的 N+1 查詢問題，提升效能的可預測性。

以下詳細說明：

## 1. 使用指令關閉 (程式碼配置)

要全域禁用 Lazy Loading，您需要在您的 **`DbContext`** 類別的建構子中，明確地將 `LazyLoadingEnabled` 屬性設定為 `false`。

**指令範例 (EF6):**

```csharp
public class MyDbContext : DbContext
{
    public MyDbContext() : base("name=MyConnectionString")
    {
        // 核心指令：全域禁用 Lazy Loading
        this.Configuration.LazyLoadingEnabled = false; 

        // 通常會同時禁用 Proxy Creation，因為 Lazy Loading 依賴它
        this.Configuration.ProxyCreationEnabled = false; 
    }
    // ... DbSet 屬性
}
```

- 當您執行這個設定後，即使您的導覽屬性被標記為 `virtual`，EF 也不會再自動發出延遲載入的查詢。

## 2. ENTITY 不要加 `virtual`

這是**輔助禁用** Lazy Loading 的一個重要手段，並且在禁用後，這樣做也是一種**最佳實踐**：

|**情況**|**說明**|
|---|---|
|**啟用時**|如果您**沒有**在 `DbContext` 中禁用 Lazy Loading，那麼導覽屬性**必須**加上 `virtual` 才能讓 Lazy Loading 生效。|
|**禁用後**|如果您已經用程式碼指令（步驟 1）全域禁用了 Lazy Loading，那麼： 1. **不需要**再加 `virtual`。 2. **建議移除** `virtual` 關鍵字，因為它不再有 Lazy Loading 的用途，這樣可以避免 EF 執行不必要的 Proxy Creation (代理物件建立)，使您的實體類別更輕量和標準。|

**總結來說：**

- **全域禁用 Lazy Loading (強制關閉)：** 透過設定 `this.Configuration.LazyLoadingEnabled = false;`。
- **最佳實踐 (移除依賴)：** 將所有導覽屬性的 `virtual` 關鍵字移除。

這樣一來，所有資料載入都必須使用 **急切載入 (`Include()`)** 或 **顯式載入**，讓開發者明確掌握資料庫查詢的時機和內容，從而避免難以發現的 N+1 查詢問題。
