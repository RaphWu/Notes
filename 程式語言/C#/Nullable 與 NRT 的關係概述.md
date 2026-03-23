---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Nullable 與 NRT 的關係概述

在 C# 中，「Nullable」有兩種不同層次的意義，必須先區分清楚：

- Nullable Value Type
    - `int?`、`DateTime?`
    - 早在 C# 2.0 就存在，與 NRT 無關
- Nullable Reference Type（NRT）
    - `string?`、`object?`
    - C# 8.0 引入，必須啟用 NRT 才有語意與編譯器檢查

結論先講：
**程式中用到 `?` 不代表一定要開 NRT，但「是否開啟 NRT」會徹底改變 `string?` 的語意與編譯行為。**

# 不開 NRT 的行為（預設舊模式）

## 特性

- `string`、`string?` 沒有語意差別
- 編譯器不做 null 流程分析
- 不會產生 CS86xx 系列警告
- `?` 幾乎只是「標記」，沒有約束力

## 設定方式

```xml
<Nullable>disable</Nullable>
```

## 適用情境

- 舊專案（.NET Framework、WinForm、WPF）大量歷史程式碼
- 不希望被大量 Nullable 警告淹沒
- 團隊尚未建立 Nullable 規範

## 風險

- NullReferenceException 只能在執行期發現
- `string?` 對閱讀者沒有實質保障效果

# 開啟 NRT 的行為（現代 C#）

## 特性

- `string` 表示「不可為 null」
- `string?` 表示「允許為 null」
- 編譯器進行資料流分析
- 產生 CS8600–CS8699 系列警告

## 設定方式（專案層級）

```xml
<Nullable>enable</Nullable>
```

或僅開警告分析：

```xml
<Nullable>warnings</Nullable>
```

## 適用情境

- 新專案（.NET 6+、.NET 8）
- 核心邏輯層（Core / Domain / Application）
- 公用 Library、NuGet 套件、Framework

## 成本

- 初期需要處理大量警告
- 需要團隊共識與 coding guideline
- DI、ORM、序列化需特別處理初始化問題

# Nullable 設定選項完整比較

## 專案層級設定

```xml
<Nullable>enable</Nullable>
<Nullable>disable</Nullable>
<Nullable>warnings</Nullable>
```

## 原始碼層級控制

```csharp
#nullable enable
#nullable disable
#nullable restore
```

## 常見組合策略

- Core / Domain
    - `<Nullable>enable</Nullable>`
- Infrastructure / UI
    - `<Nullable>warnings</Nullable>` 或 `disable`
- 舊專案漸進式導入
    - 專案 disable
    - 新檔案 `#nullable enable`

# 什麼情況「一定要開」NRT

- 公開 API（給別人用）
- NuGet 套件
- Domain Model / Business Rule
- 非 UI 的核心邏輯

理由：

- Null 是語意的一部分，應該進入型別系統
- API 使用者可以在編譯期就被保護

# 什麼情況「可以先不要開」

- WinForm Designer 自動產生的程式碼
- 大量反射、Property Injection
- 舊系統維護階段，穩定性優先於重構

# 決策建議（實務結論）

- Nullable Value Type
    - 一直都該用，與 NRT 無關
- Nullable Reference Type
    - 新程式碼：**一定開**
    - 舊程式碼：**分層、分階段開**
- 不要為了「少警告」而全面 disable
- 也不要為了「理想潔癖」一次全開導致專案失控
