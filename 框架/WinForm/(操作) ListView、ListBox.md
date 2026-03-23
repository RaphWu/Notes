---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# ListView / ListBox

## 更新 DataSource 的順序

### 核心原則

- 不直接操作 UI 項目集合（Items），而是操作資料來源
- 更新資料完成後，再一次性通知 UI 重新繫結
- 盡量避免在迴圈中反覆設定 DataSource

### ListBox 使用 DataSource 的正確順序

- 準備資料集合（建議使用 BindingList 或 ObservableCollection）
- 若資料筆數可能變動，先清空或同步集合內容
- 暫時解除 DataSource
- 更新或替換資料集合
- 重新指定 DataSource
- 視需求設定 DisplayMember、ValueMember

#### 標準流程

- listBox.DataSource = null
- 更新資料集合內容
- listBox.DisplayMember = "Name"
- listBox.ValueMember = "Id"
- listBox.DataSource = dataList

#### 為什麼要先設為 null

- 避免舊的 CurrencyManager 快取舊資料
- 防止筆數變動時 UI 殘留舊項目
- 確保重新建立完整的繫結流程

### ListView（非虛擬模式）更新順序

- BeginUpdate
- 清空 Items
- 重新建立 ListViewItem
- EndUpdate

#### 標準流程

- listView.BeginUpdate()
- listView.Items.Clear()
- 依資料來源重新加入 Items
- listView.EndUpdate()

### ListView + 資料來源的設計建議

- ListView 本身不支援 DataSource
- 建議將資料集中管理於 List 或 BindingList
- UI 只負責「整體重建」顯示，不做差異更新

### 常見錯誤

- 在 foreach 中重複設定 DataSource
- 同時操作 Items 與 DataSource
- 使用 List 卻期待 UI 自動更新
- 筆數改變時只更新 FirstOrDefault 或單一項目

### 實務建議

- 需要自動反映新增、刪除、修改
    - 使用 BindingList
- 需要完整掌控更新時機
    - 每次資料變動後重新指定 DataSource
- 大量資料更新
    - 暫停繫結後一次性重建 UI

### 總結

- ListBox：資料 → 解除繫結 → 更新 → 重新繫結
- ListView：資料 → BeginUpdate → 清空 → 重建 → EndUpdate
- 筆數可能變動時，一律視為「整體更新」而非單筆修改

---

## 原則總覽

在 WinForms 中更新 ListBox / ListView 的 DataSource，正確順序的核心目標是：

- 避免畫面閃爍
- 避免 SelectedIndex 失效
- 避免不必要的重新繫結
- 保持效能與 UI 穩定性

ListBox 與 ListView 的「正確順序」不完全相同，必須分開看。

## ListBox（使用 DataSource）

### 正確更新順序

1. 暫停 UI 更新
2. 更新資料集合內容
3. 通知 Binding 變更
4. 恢復 UI 更新
5. 視需要還原選取狀態

### 標準寫法

```csharp
listBox.BeginUpdate();

// 不要重設 DataSource
// 只更新原本綁定的集合內容

listBox.EndUpdate();
```

### 資料來源建議

- `BindingList<T>` 為首選
- 次選 `ObservableCollection<T>`（需包一層 `BindingSource`）

### 為什麼不要每次設 DataSource

- 會強制整個控制項重繪
- SelectedIndex 會被重置
- DisplayMember / ValueMember 可能失效
- 效能差

## ListView（不支援 DataSource）

### 正確更新順序

ListView 是「命令式 UI」，必須直接操作 Items。

### 正確更新順序

1. BeginUpdate()
2. Items.Clear() 或差異更新
3. 加入 / 更新 ListViewItem
4. EndUpdate()

### 標準寫法

```csharp
listView.BeginUpdate();

listView.Items.Clear();
// or 差異更新 Items

listView.EndUpdate();
```

### ListView 的重點差異

- 沒有 DataSource
- 每次更新都要自己控制
- Clear + Add 在 ListView 是可接受的

## BindingSource 介入時的順序

### 推薦結構

- 資料集合 → BindingSource → ListBox

### 正確更新順序

1. BeginUpdate()
2. 更新集合內容
3. bindingSource.ResetBindings(false)
4. EndUpdate()

### 範例

```csharp
listBox.BeginUpdate();

bindingList.Clear();
foreach (var item in newItems)
{
    bindingList.Add(item);
}
bindingSource.ResetBindings(false);

listBox.EndUpdate();
```

## SelectedItem / SelectedValue 的保留策略

### 正確時機

- 更新資料前先記錄
- 更新完成後再還原

### 範例

```csharp
var selectedId = listBox.SelectedValue;

// 更新資料

listBox.SelectedValue = selectedId;
```

## 常見錯誤清單

- 先設 DataSource = null 再重設
- 每次 new List 再指定 DataSource
- 在更新過程中操作 SelectedIndex
- 忘記 BeginUpdate / EndUpdate
- 對 ListBox 使用 ListView 的 Items 操作方式

## 實務結論

- ListBox：資料驅動，更新「集合內容」，不要換 DataSource
- ListView：UI 驅動，直接操作 Items
- DataSource 只設一次
- 差異更新永遠比整批重設好
- 選取狀態一定要最後處理

## 「ListBox + BindingList + 差異同步 + 選取保留」的完整可直接用範例

### 標準範例目標

- 適用 WinForms
- ListBox + BindingSource + BindingList
- 差異同步資料
- 保留 SelectedValue
- 不重設 DataSource
- 可直接複製使用

### 資料模型範例

```csharp
public class ClassifyInfo
{
    public int Id { get; set; }
    public string DisplayText { get; set; }

    public void UpdateFrom(Employee emp)
    {
        Id = emp.Id;
        DisplayText = $"{emp.EmployeeId} {emp.EmployeeName}";
    }
}
```

### 初始化（只做一次）

```csharp
BindingList<ClassifyInfo> _items = new BindingList<ClassifyInfo>();
BindingSource _bs = new BindingSource();
```

```csharp
_bs.DataSource = _items;
listBox1.DataSource = _bs;
listBox1.DisplayMember = nameof(ClassifyInfo.DisplayText);
listBox1.ValueMember = nameof(ClassifyInfo.Id);
```

### 差異同步方法（重點）

```csharp
void SyncEmployees(IEnumerable<Employee> employees)
{
    listBox1.BeginUpdate();
    var selectedId = listBox1.SelectedValue as int?;
    var src = employees.ToDictionary(e => e.Id);

    for (int i = _items.Count - 1; i >= 0; i--)
    {
        if (!src.ContainsKey(_items[i].Id))
        {
            _items.RemoveAt(i);
        }
    }

    foreach (var emp in employees)
    {
        var item = _items.FirstOrDefault(x => x.Id == emp.Id);
        if (item == null)
        {
            var newItem = new ClassifyInfo();
            newItem.UpdateFrom(emp);
            _items.Add(newItem);
        }
        else
        {
            item.UpdateFrom(emp);
        }
    }

    _bs.ResetBindings(false);

    if (selectedId.HasValue)
    {
        listBox1.SelectedValue = selectedId.Value;
    }

    listBox1.EndUpdate();
}
```

### 呼叫方式

```csharp
SyncEmployees(employeeList);
```

### 為什麼這樣是正確順序

- BeginUpdate / EndUpdate：避免閃爍
- 先記錄選取，再更新資料
- 移除不存在項目
- 重用既有 Item
- BindingSource 統一通知
- 最後才還原選取

### 注意事項

- BindingList 自動通知 Add / Remove
- Item 屬性變更若需即時反映，請實作 INotifyPropertyChanged
- DisplayText 若是計算屬性，需在 UpdateFrom 後 Raise PropertyChanged

### 什麼情況要改用其他做法

- 項目數 > 10,000：需虛擬化或 ListView
- 頻繁更新（<100ms）：需批次更新策略
- 多執行緒來源：務必回 UI 執行緒

---

## 表面矛盾的原因

- 「先 DataSource = null」與「不要每次重設 DataSource」討論的是**不同層級的更新**
- 一個是**整體資料來源切換**
- 一個是**既有資料來源內部變動**

## 何時需要 DataSource = null

- 資料來源物件「本身被換掉」
- 筆數可能增減，且 UI 出現殘留或不同步
- DisplayMember / ValueMember 有變更
- 曾發生繫結異常或狀態不乾淨

### 典型情境

- 原本綁定 List
- 改成另一個新的 List
- 或由查詢重新產生全新集合實例

### 正確做法

- listBox.DataSource = null
- listBox.DataSource = newDataList

## 何時「不要」每次重設 DataSource

- DataSource 物件「不變」
- 只是在同一個集合內新增、刪除、修改項目
- 使用支援變更通知的集合型別

### 典型情境

- BindingList
- ObservableCollection
- 自行實作 IBindingList / INotifyCollectionChanged

### 正確做法

- 維持 DataSource 不變
- 直接操作集合內容（Add / Remove / Clear / Reset）

## 關鍵分界線

- 是否「更換 DataSource 實例」
- 是否需要「重建 CurrencyManager」

### 判斷表

- 更換集合實例
    - 必須 DataSource = null
- 同一集合內部變動
    - 不要重設 DataSource

## 常見誤用

- 使用 List 卻期待 UI 自動更新
- 每新增一筆就重新指定 DataSource
- BindingList 卻仍反覆 DataSource = null

## 簡化心智模型

- DataSource 是「管道」
- 集合內容是「水」
- 換水管
    - 先拔掉再接（DataSource = null）
- 水在管內流動
    - 不要重接管線

## 結論

- DataSource = null 是**切換資料來源時的初始化動作**
- 不要每次重設 DataSource 是**集合內部更新的效能與正確性原則**
- 兩者不衝突，前提不同

List_Brands.SelectedIndex = _vmBrands.FindIndex(m => m.Name == newBrandName);
