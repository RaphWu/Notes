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
---

# 聯級刪除 Cascade Delete

# EF6 聯級刪除（Cascade Delete）行為說明與設定

## 1. EF6 預設聯級刪除

* 在一對多或一對一關係中，當主端（Principal，通常是 A）被刪除時，依照設定會自動刪除依賴端（Dependent，通常是 B）。
* 預設情況下，EF6 會在資料庫建立 `ON DELETE CASCADE` 外鍵約束（如果你有設定聯級刪除）。
* 聯級刪除是單向的，通常是「刪除主端 => 刪除依賴端」。
* 刪除依賴端不會反向刪除主端（EF6 沒有雙向聯級刪除）。

---

## 2. 你需求的狀況

* **刪除 A 時，要刪除 B** （主端刪除時刪除依賴端）
* **刪除 B 時，不刪除 A** （依賴端刪除時不影響主端）

這是典型且合理的關係設定，EF6 預設也是如此。

---

## 3. 如何設定

### (1) Fluent API 設定範例

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<A>()
        .HasMany(a => a.Bs)
        .WithRequired(b => b.A)
        .WillCascadeOnDelete(true);  // 啟用刪除 A 時聯級刪除 Bs
}
```

說明：

* `WillCascadeOnDelete(true)` 表示刪除主端 `A` 時，會自動刪除相關的 `B`。
* `WithRequired` 表示 `B` 必須有一個 `A`，關係為必填。
* 刪除 `B` 不會影響 `A`。

---

### (2) 資料庫行為

* EF 建立的外鍵會帶有 `ON DELETE CASCADE`。
* 資料庫會確保「刪除 A 自動刪除 B」。
* 刪除 B 時，只刪除 B，不會刪除 A。

---

## 4. 如果刪除 B 也刪除 A，會怎麼發生？

* EF6 不會自動產生這種「雙向」聯級刪除。
* 若你在資料庫層面手動建立雙向 `ON DELETE CASCADE` 外鍵，會導致死循環錯誤（不允許）。
* 因此，不可能同時雙向啟用聯級刪除。

---

## 5. 總結

* **刪除 A 連動刪除 B：在 Fluent API 用 `WillCascadeOnDelete(true)` 設定即可**。
* **刪除 B 不影響 A：EF6 預設行為，不需額外設定**。
* 不要嘗試建立雙向聯級刪除，會有資料庫限制及錯誤。

---

需要我幫你示範完整的 Entity 與 Fluent API 設定範例嗎？

---

# EF6 中無聯級刪除時，刪除主端（A）對依賴端（B）外鍵的影響

## 1. 目前情況

* `B` 有一個外鍵（FK）指向 `A`（主端）。
* **沒有啟用聯級刪除（Cascade Delete）**。
* 你嘗試刪除 `A` 的某筆資料。

---

## 2. 刪除 `A` 時 `B` 的 FK 狀況

### (1) 資料庫層面

* 如果資料庫 FK 約束沒有設定聯級刪除，也沒有設定「ON DELETE SET NULL」，
* 直接刪除 `A` 會導致 **外鍵約束違反錯誤**，資料庫拒絕刪除。

### (2) EF6 行為

* EF6 不會自動更新或清空 `B` 的外鍵欄位。
* 若你直接呼叫 `_context.Set<A>().Remove(a)` 並 `SaveChanges()`，資料庫會拒絕刪除，因為 `B` 有參照。

---

## 3. 若要刪除 `A`，你必須先處理 `B`

常見作法：

* **先刪除所有依賴 `A` 的 `B` 記錄**，確保沒有 FK 參照。
* 或是將 `B` 的 FK 欄位設為 `NULL`（前提是外鍵允許 `NULL`，且設定 FK 為 `ON DELETE SET NULL`）。
* 再刪除 `A`。

---

## 4. 外鍵欄位設定對刪除行為影響

| FK 約束設定            | 刪除主端 A 時 B 的外鍵行為       |
| ------------------ | ---------------------- |
| ON DELETE CASCADE  | 自動刪除相關 B 記錄            |
| ON DELETE SET NULL | 自動將 B 的 FK 欄位設為 NULL   |
| 無聯級刪除或無 SET NULL   | 刪除 A 失敗，資料庫報 FK 約束違反錯誤 |

---

## 5. 小結

* **沒有聯級刪除且 FK 不允許 NULL 時，刪除 A 會失敗。**
* 必須先處理 B（刪除或修改 FK）後才能刪除 A。
* EF6 不會自動幫你處理這些動作，需要在程式碼中自行控制。

---

需要我示範 EF6 如何先刪除 B 再刪除 A 的程式碼範例嗎？

---

# EF6 先刪除依賴端 B，再刪除主端 A 範例

```csharp
// 假設 Entity 定義
public class A
{
    public int Id { get; set; }
    public virtual ICollection<B> Bs { get; set; }
}

public class B
{
    public int Id { get; set; }
    public int AId { get; set; }
    public virtual A A { get; set; }
}

// 刪除流程示範
public void DeleteAWithBs(int aId, MyDbContext context)
{
    // 先取出 A
    var a = context.As.Include(x => x.Bs).SingleOrDefault(x => x.Id == aId);
    if (a == null) return;

    // 刪除所有相關 B
    context.Bs.RemoveRange(a.Bs);

    // 再刪除 A
    context.As.Remove(a);

    // 儲存變更（一次送出）
    context.SaveChanges();
}
```

---

## 說明

* 使用 `Include` 預先載入 A 的相關 Bs（避免多次查詢）。
* `RemoveRange` 批次刪除所有 B。
* 最後刪除 A。
* `SaveChanges()` 一次送出，確保資料一致。

---

需要我示範用非同步版本（`async/await`）嗎？

---

# EF6 非同步刪除依賴端 B 再刪除主端 A 範例

```csharp
using System.Data.Entity;
using System.Threading.Tasks;

public async Task DeleteAWithBsAsync(int aId, MyDbContext context)
{
    // 預先載入 A 和相關的 Bs
    var a = await context.As.Include(x => x.Bs)
                           .SingleOrDefaultAsync(x => x.Id == aId);
    if (a == null) return;

    // 批次刪除所有 B
    context.Bs.RemoveRange(a.Bs);

    // 刪除 A
    context.As.Remove(a);

    // 非同步儲存變更
    await context.SaveChangesAsync();
}
```

---

# 說明

* 使用 `Include` 預先載入相關資料，避免多次查詢。
* `RemoveRange` 批次刪除依賴資料。
* `await` 搭配 `SingleOrDefaultAsync` 和 `SaveChangesAsync`，非同步執行，效能更佳。

---

需要我示範如何用此方法在 ASP.NET MVC 或 Web API Controller 裡呼叫嗎？

# EF6 使用 Fluent API 設定外鍵為 ON DELETE SET NULL

---

## 1. 前提條件

* 外鍵欄位必須允許 `NULL`（nullable）。
* 這樣才能設定刪除主表時，將外鍵欄位設為 `NULL`。

---

## 2. Fluent API 設定範例

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<B>()
        .HasOptional(b => b.A)                // 外鍵是可選 (nullable)
        .WithMany(a => a.Bs)
        .HasForeignKey(b => b.AId)            // 外鍵欄位
        .WillCascadeOnDelete(false);          // 關閉 Cascade Delete
}
```

---

## 3. 解釋

* `HasOptional()`：表示 `B` 可以沒有關聯的 `A`（FK 欄位可為 null）。
* `.WillCascadeOnDelete(false)`：關閉聯級刪除，不會自動刪除 `B`。
* EF6 不會直接產生 `ON DELETE SET NULL`，但此設定搭配資料庫設定可實現類似效果。

---

## 4. 資料庫層面注意事項

* EF6 不會自動在資料庫建立 `ON DELETE SET NULL` 外鍵約束。
* 需要你手動在資料庫中修改外鍵約束，加入 `ON DELETE SET NULL`。
  * 例如使用 SQL Server Management Studio 執行 ALTER FOREIGN KEY 指令。

---

## 5. 小結

| EF6 Fluent API 設定                                 | 實際產生的資料庫行為                                  |
| ------------------------------------------------- | ------------------------------------------- |
| `HasOptional(...)` + `WillCascadeOnDelete(false)` | 不會有 Cascade Delete，需手動設定 ON DELETE SET NULL |

---

## 6. 若要在資料庫設定 `ON DELETE SET NULL` 範例（SQL Server）

```sql
ALTER TABLE [dbo].[B]
DROP CONSTRAINT FK_B_A; -- 刪除舊外鍵

ALTER TABLE [dbo].[B]
ADD CONSTRAINT FK_B_A FOREIGN KEY (AId)
REFERENCES [dbo].[A](Id)
ON DELETE SET NULL;
```

---

需要我示範如何手動修改資料庫外鍵，或者如何用 Migration 自訂 SQL 嗎？
