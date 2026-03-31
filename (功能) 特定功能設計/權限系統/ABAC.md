---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# ABAC 權限模型概念與實作邏輯

## 概念

ABAC（Attribute-Based Access Control，屬性基礎存取控制）是一種靈活的權限控制模型，它的核心思想是：**存取決策不是單純依賴使用者角色，而是依據「使用者、資源、環境」等各種屬性進行判斷**。

主要概念：

1. **使用者屬性（User Attributes）**
    描述使用者的特徵，例如：職位、部門、年資、身份等。

2. **資源屬性（Resource Attributes）**
    描述被存取對象的特徵，例如：文件類型、敏感等級、創建時間、所屬專案等。

3. **環境屬性（Environment Attributes）**
    描述存取行為發生的情境，例如：時間、地點、裝置安全性、網路位置等。

4. **動作（Action）**
    存取操作的類型，例如：讀取、寫入、刪除、修改、下載。

5. **政策（Policy）**
    ABAC 的核心是政策規則，它描述了「在特定屬性條件下，哪些動作是允許或拒絕的」。

政策通常表達為條件語句，例如：

- 如果使用者屬於「人事部」且文件屬於「薪資資料」，則允許「讀取」。
- 如果存取時間在下班後，則拒絕所有「修改」操作。

ABAC 的特點是靈活、可擴展、可細緻控制，但實作與管理相對複雜。

## 實作邏輯

ABAC 的實作可以拆解成幾個步驟：

1. **定義屬性集合**
    - 列出系統中可能的使用者、資源、環境屬性。
    - 屬性可以存於資料庫、LDAP、JWT 或其他系統。

2. **建立政策規則**
    - 將存取控制需求轉化為政策語句。
    - 可以使用 JSON、XML、程式碼或專門的策略引擎表示。

3. **在存取點做授權判斷**
    - 當使用者請求存取資源時，系統收集相關屬性。
    - 將屬性與政策規則匹配，決定是否允許操作。
    - 可以有允許優先或拒絕優先的策略。

4. **動態評估**
    - ABAC 支持根據當前環境屬性動態決策。
    - 例如：同一個使用者在公司網路內可以存取文件，但在外網則拒絕。

5. **記錄與審計**
    - 每次授權決策應記錄，方便追蹤與安全審計。
    - 可與日誌系統（如 Serilog）結合。

## 範例邏輯（C# 輔助理解）

```csharp
class User
{
    public string Department { get; set; }
    public int Seniority { get; set; }
}

class Document
{
    public string Type { get; set; }
    public string Owner { get; set; }
}

class Environment
{
    public bool IsOfficeNetwork { get; set; }
}

bool CanAccess(User user, Document doc, Environment env, string action)
{
    // 範例政策規則
    if (action == "Read" && user.Department == "HR" && doc.Type == "Salary")
        return env.IsOfficeNetwork; // 只有辦公網路可以讀取

    if (action == "Edit" && user.Seniority > 5)
        return true; // 高資歷人員可以修改

    return false; // 其他皆拒絕
}
```

這個範例展示了：

- 使用者屬性、資源屬性、環境屬性如何影響決策
- 政策規則如何以程式條件表示
- 決策是動態的，可依屬性不同而改變結果

## 思路與建議

1. **先整理屬性，再寫政策**
    ABAC 的核心是屬性，所以先列出完整屬性集合很重要。
    
2. **政策設計盡量可組合**
    避免硬編碼每一種情況，使用可組合規則提高擴展性。
    
3. **結合角色或組**
    在大型系統中，ABAC 可與 RBAC 結合，角色決定初步權限，ABAC 精細控制條件。
    
4. **測試多種情境**
    模擬不同使用者、資源、環境組合，驗證政策是否正確。
    
5. **動態性與可維護性**
    政策應該可以不改程式碼就能調整，建議使用策略引擎或配置文件管理。
