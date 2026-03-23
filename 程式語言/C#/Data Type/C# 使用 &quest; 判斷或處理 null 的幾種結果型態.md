---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# C# 使用 `?` 判斷或處理 `null` 的幾種結果型態

以下依實際語言特性分類說明，避免混淆不同語法用途。

# 一、條件運算子 `?:`（三元運算子）

用於依條件回傳不同結果，與 `null` 無直接關聯，但常搭配使用。

- 條件成立回傳左值，否則回傳右值
- 結果型別為左右兩邊可推斷的共同型別

```csharp
var result = obj != null ? obj.Value : defaultValue;
```

# 二、Null 條件運算子 `?.`

在物件為 `null` 時，整個運算結果為 `null`，不會丟出例外。

- 用於屬性、方法、索引子
- 回傳型別一定是「可為 null」的型別

```csharp
int? length = text?.Length;
string name = person?.Name;
var value = dict?[key];
```

結果說明：

- 若左側為 `null` → 結果為 `null`
- 若左側非 `null` → 正常存取

# 三、Null 合併運算子 `??`

在左值為 `null` 時，回傳右值。

- 常用於提供預設值
- 不會判斷「空字串」或「0」

```csharp
string name = input ?? "Unknown";
int count = nullableCount ?? 0;
```

結果說明：

- 左值為 `null` → 回傳右值
- 左值非 `null` → 回傳左值

# 四、Null 合併指定運算子 `??=`

僅在左值為 `null` 時，才進行指定。

```csharp
cache ??= new Dictionary<string, string>();
```

結果說明：

- 左值為 `null` → 指派右值
- 左值非 `null` → 不動作

# 五、Nullable Value Type `T?`

用於值型別，代表「可為 null」。

- 編譯後實際型別為 `Nullable<T>`
- 與 Null 條件運算子不同概念

```csharp
int? number = null;
bool? flag = true;
```

相關屬性：

- `HasValue`
- `Value`

```csharp
if (number.HasValue)
{
    int v = number.Value;
}
```

# 六、Null Forgiving Operator `!`

用於告訴編譯器「我確定不為 null」，不會產生任何執行期檢查。

- 只影響編譯期 Nullable Reference Types 分析
- 若實際為 `null`，仍會發生例外

```csharp
string text = GetText()!;
int length = text.Length;
```

# 七、組合用法的實際結果差異

```csharp
int length = person?.Name?.Length ?? 0;
```

判斷流程：

- `person` 為 `null` → 整體為 `null` → `?? 0` → 結果為 `0`
- `Name` 為 `null` → 同上
- 兩者皆非 `null` → 回傳實際 `Length`

# 八、常見誤解整理

- `?.` 不等於「回傳預設值」，只會回傳 `null`
- `??` 不會判斷空字串、`0`、`false`
- `!` 不會做任何保護，只是關閉警告
- `T?` 與 `?.` 完全不同層級的語法

如果你是針對「EF / DTO / ViewModel」或「WinForm DataBinding」實際案例想確認某一種用法的結果型別與生命週期，可以直接給範例，我可以逐行解析。
