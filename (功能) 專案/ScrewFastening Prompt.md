---
aliases:
created: 
update:
author:
language:
sourceurl:
tags:
date:
---

# GitHub Copilot 統一整合 Prompt（骨架生成用，最終版）

以下內容請**整段直接貼給 GitHub Copilot Chat**，或放在方案的 `README.md / Architecture.md`，再要求 Copilot「依此架構產生骨架程式碼」。
此 Prompt 的目標是：**先生成正確結構的骨架，細節之後人工補齊。**

## 系統定位與目標

你是一位資深 C# WinForm 工業機台控制系統工程師。
請協助建立一套「機台工序流程編輯與執行系統」。

此系統提供 ENGINEER 使用，用來：

- 從既有工序模板中選擇工序
- 組合成一條可排序的流程
- 調整各工序參數
- 儲存後由機台依序執行

這是一套**工程工具型 UI**，不是展示型 UI。

## 執行環境與限制（必須嚴格遵守）

- 作業系統：Windows 7
- Framework：.NET Framework 4.5 / 4.6.2
- UI 技術：WinForm
- 不使用 WPF
- 不使用第三方 UI Framework
- 使用 Autofac 作為 DI Container
- 已有自製 Region / Navigation 機制（可導航 UserControl，可傳遞參數）
- UI 與 Engine 必須完全解耦
- 不使用 switch / if 判斷工序型別

## 核心資料模型（不可偏離）

每一個工序步驟使用以下結構表示：

- OrderNo：流程中的順序
- ProcessId：工序識別字串（例如 AXIS_MOVE）
- ParamJson：JSON 字串，存放該工序的參數
- Enabled：是否啟用

請定義一個 `ProcessStepEntity` 類別承載上述資料。

## 工序系統的基本概念

- 工序（Process）是**預先定義好規格的能力**
- ENGINEER 只能選用、設定參數、排序
- 不能在 UI 任意發明新邏輯
- 每個工序都有預設參數（Default Param）

## 工序範例（僅作示範）

- 單軸依指定速度移動到指定座標
- 兩軸同動依指定速度移動到指定座標
- 讀取感測器值，判斷是否落在指定範圍，成立時執行指定行為（FUNC）

## ParamJson 使用原則

- ParamJson 是**儲存格式**
- UI 不直接讓 ENGINEER 編輯 JSON
- UI 內部使用強型別 Param 類別
- JSON 與 Param 類別之間需可互相轉換

## Engine 層設計（不可依賴 UI）

請設計以下介面：

IProcessHandler

- 負責實際執行工序邏輯
- 僅接收 ParamJson
- 不知道 UI、Editor、Region 的存在
- 支援 CancellationToken
- 可被 ProcessRunner 呼叫

## UI Editor 層設計（WinForm UserControl）

請設計以下介面：

IProcessEditor

- 實作於 WinForm UserControl
- 負責編輯單一工序的參數
- 提供 Load(string paramJson)
- 提供 Save() 回傳 JSON
- 提供 ParamChanged 事件
- Editor 不知道 OrderNo
- Editor 不知道整個流程

## Process Descriptor（工序中介定義，核心）

請建立 `ProcessDescriptor` 類別，用於描述一種工序：

- ProcessId
- DisplayName
- HandlerType
- EditorType
- DefaultParamJson

此 Descriptor 用於：

- 左側工序來源（Process Source）
- 新增工序時套用預設參數
- 對應 Handler 與 Editor

## 工序來源與流程編輯 UI 架構

UI 採用三區設計：

- 左側：Process Source
    - 顯示所有可用工序模板
    - 來自 ProcessDescriptor
- 中間：Process List
    - 顯示實際流程（List）
    - 可新增、刪除、上移、下移
- 右側：Process Editor
    - 顯示對應工序的參數編輯畫面

## Process List 行為規則

- 使用 DataGridView 顯示
- 調整順序時：
    - 只調整集合順序
    - 再重新計算 OrderNo
- 第一版可使用 上移 / 下移 按鈕
- 拖放排序非必要

## Region Navigation 使用方式

- Process Editor 透過 Region Navigation 顯示
- Navigation 可傳遞參數
- Navigation 時傳入：
    - ProcessId
    - ParamJson
    - 或整個 ProcessStepEntity

Editor：

- 不自行查資料
- 只使用 Navigation 傳入的參數

## Autofac 註冊規則（非常重要）

- 使用 Autofac Keyed Registration
- Key = ProcessId（字串）
- IProcessHandler 使用 Keyed 註冊
- IProcessEditor 使用 Keyed 註冊
- Editor 使用 InstancePerDependency
- 嚴禁在任何地方使用 switch / if 判斷 ProcessId

## 工序選取與 Editor 切換流程

當 Process List 選取項目改變時：

- 取得 SelectedStep.ProcessId
- Autofac Resolve 對應的 IProcessEditor
- 呼叫 editor.Load(SelectedStep.ParamJson)
- 導航至 Editor Region

當 Editor 參數變更時：

- Editor 觸發 ParamChanged
- 呼叫 editor.Save()
- 更新 SelectedStep.ParamJson
- 更新 Process List 的摘要顯示

## 流程執行器（ProcessRunner）

請設計 ProcessRunner 類別：

- 依 OrderNo 依序執行
- Enabled = false 的工序需跳過
- 透過 IProcessHandler 執行
- 支援 CancellationToken
- 不依賴任何 UI 類別
- 提供基本錯誤處理結構（不中斷架構）

## 非目標（請勿產生）

- 不使用 WPF
- 不使用 MVVM Framework
- 不產生流程圖 UI
- 不直接讓 UI 編輯 JSON
- 不一次生成完整成品系統

## 程式碼生成要求

- 先產生骨架與介面
- 類別與檔案結構清楚
- 命名偏工業控制語意
- 符合 SOLID
- 適合長期維護與擴充

---

## 使用方式建議

- 先貼整段 Prompt 給 Copilot Chat
- 要求「依此架構產生骨架程式碼」
- 再逐一請 Copilot 補齊各模組
- 若 Copilot 偏離（WPF、switch、UI 綁 Engine），請立即中斷重來

這份 Prompt 已完整整合：

- 你的工序模型思維
- Process Source / List / Editor 三區 UI
- Autofac + Region 導航
- 工業機台實務可維護性

你接下來只需要做一件事：
**用你的實際工序，把骨架一個一個填滿即可。**

---

# GitHub Copilot 統一整合 Prompt（完整可直接使用）

以下是一段**「一次就讓 Copilot 對齊整個系統設計」**的整合型 Prompt。
建議放在「方案說明文件」或「核心專案的 README / Architecture.md」，也可分段貼給 Copilot Chat。

## 系統背景與目標

```text
你是一位資深 C# WinForm 工業機台控制系統工程師，
請協助建立一套「機台工序流程編輯與執行系統」。

此系統用於工程師設定機台作業流程，
流程由多個工序（Process Step）組成，
機台執行時依設定順序逐一執行。
```

## 執行環境與技術限制

```text
環境限制如下，請嚴格遵守：
- 作業系統：Windows 7
- Framework：.NET Framework 4.5 / 4.6.2
- UI 技術：WinForm（不可使用 WPF）
- 使用 Autofac 作為 DI Container
- 已有自製 Region / Navigation 機制（類似 Prism）
- UI 與 Engine 必須完全解耦
- 不使用第三方 UI Framework
```

## 核心資料模型（不可偏離）

```text
每一個工序（Process Step）以以下結構表示：

- OrderNo        ：執行順序
- ProcessId      ：工序類型識別字串（例如 AXIS_MOVE）
- ParamJson      ：JSON 字串，存放工序參數
- Enabled        ：是否啟用

請定義一個 ProcessStepEntity 類別承載上述資料。
```

## 工序設計原則

```text
- ProcessId 是唯一識別
- 不使用 switch / if 判斷 ProcessId
- 新增工序不應修改既有程式碼
- ParamJson 是儲存格式，不是 UI 操作格式
- UI 不直接操作 JSON 物件（JObject）
```

## Engine 層（與 UI 完全無關）

```text
請設計以下介面：

IProcessHandler
- 負責實際執行工序邏輯
- 只接收 ParamJson
- 不知道 UI、Editor、Region 的存在
- 支援 CancellationToken
```

## UI Editor 層（WinForm UserControl）

```text
請設計以下介面：

IProcessEditor
- 實作於 WinForm UserControl
- 負責顯示與編輯單一工序的參數
- 提供 Load(string paramJson)
- 提供 Save() 回傳 JSON
- 提供 ParamChanged 事件
- Editor 不知道 OrderNo、不知道整個流程
```

## Process Descriptor（關鍵中介）

```text
請建立 ProcessDescriptor 類別，用於描述工序型別：

- ProcessId
- DisplayName
- HandlerType
- EditorType
- DefaultParamJson

此 Descriptor 用於：
- UI 下拉選單
- 建立預設參數
- 對應 Handler 與 Editor
```

## Autofac 註冊規則（非常重要）

```text
請使用 Autofac Keyed Registration：

- Key = ProcessId（字串）
- IProcessHandler 使用 Keyed 註冊
- IProcessEditor 使用 Keyed 註冊
- Editor 使用 InstancePerDependency

嚴禁在程式中出現：
- switch(ProcessId)
- if/else 判斷工序型別
```

## UI 架構（Region + Navigation）

```text
UI 採用 Region 分區設計：

- ProcessListRegion
    - 顯示工序列表（DataGridView）
- ProcessEditorRegion
    - 顯示工序參數編輯畫面（UserControl）
- ToolBarRegion
    - 新增 / 刪除 / 上移 / 下移

左側列表只顯示摘要，
右側 Editor 顯示完整可編輯參數。
```

## 工序列表（Process List）行為

```text
- 使用 DataGridView 顯示 List<ProcessStepEntity>
- 支援新增、刪除
- 順序調整先用「上移 / 下移」按鈕
- 調整順序時：
    - 只改集合順序
    - 再重新計算 OrderNo
- 第一版不強制實作 Drag & Drop
```

## 列表與 Editor 的互動流程

```text
當工序列表選取項目改變時：

1. 取得 SelectedStep.ProcessId
2. 透過 Autofac ResolveKeyed<IProcessEditor>
3. 呼叫 editor.Load(SelectedStep.ParamJson)
4. 透過 NavigationService 導航至 ProcessEditorRegion

當 Editor 參數變更時：
1. Editor 觸發 ParamChanged
2. 呼叫 editor.Save()
3. 更新 SelectedStep.ParamJson
4. 更新列表摘要顯示
```

## 參數編輯設計原則

```text
- 工程師不直接編輯 JSON
- 每個工序可有專屬 Editor
- Editor 內部使用強型別 Param 類別
- JSON 與 Param 類別之間需可互轉
```

## 範例工序需求（請用來產生示範程式碼）

```text
請至少示範一個工序：

AxisMove（軸移動）
- AxisMoveParam：
    - AxisNo
    - Position
    - Speed
- AxisMoveHandler : IProcessHandler
- AxisMoveEditor : UserControl, IProcessEditor
```

## 流程執行器（ProcessRunner）

```text
請設計一個 ProcessRunner：

- 依 OrderNo 順序執行
- Enabled = false 的工序需跳過
- 使用 IProcessHandler 執行
- 支援 CancellationToken
- 不依賴任何 UI 類別
```

## 非目標（請勿生成）

```text
- 不使用 WPF
- 不使用 MVVM Framework
- 不生成流程圖 UI
- 不生成第三方套件依賴
- 不產生一次性巨大檔案
```

## 程式碼品質要求

```text
- 符合 SOLID 原則
- 命名清楚，偏工業控制語意
- 可長期維護
- 適合大型機台專案
- 可逐步擴充新工序
```

# 使用建議（實務）

- 建議先貼整段 Prompt 給 Copilot Chat
- 再針對單一檔案要求「依此架構產生」
- 若 Copilot 開始混用 WPF、switch、或 UI 綁 Engine，請立即中斷重來

這一份 Prompt 已完整涵蓋：

- 你的資料模型想法
- WinForm UI 限制
- Autofac + Region 架構
- 工業機台實務可維護性

如果你之後想要：

- 拆成「每個專案一段 Prompt」
- 依你現有 Navigation API 微調語句
- 或轉成英文版給國外團隊使用
