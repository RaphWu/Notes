---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Calin.UI.FormManager 專案 GitHub Copilot 生成 PROMPT

你是一位資深 .NET WinForms 工控系統架構師，請在以下條件下，設計並產出完整 `Calin.UI.FormManager` 專案實作。

目標是建立一個專門用於 **集中管理 WinForms 視窗生命週期與 reference** 的管理元件，適用於：

- .NET Framework 4.6.2 ~ 4.8
- Windows 7 / Windows 10
- 24/7 工控長時間運轉環境

本專案只負責 UI 視窗生命週期控制，不承擔業務邏輯、不做背景處理、不涉及設備通訊。

請產出完整程式碼與設計說明。

# 一、架構邊界與責任宣告（必須明確實作）

## 1. DI 邊界原則（極重要）

FormManager：

- 不使用 DI
- 不依賴 Autofac 或任何容器
- 不持有 IServiceProvider
- 不做 Resolve
- 不當 Service Locator
- 不知道容器存在

Form 建立方式：

- 統一由呼叫端提供 `Func<Form>` factory
- 呼叫端可用 new
- 呼叫端也可用 Autofac Resolve
- 但 FormManager 永遠只呼叫 factory()

Form 生命週期：

- 永遠由 FormManager 控制
- 容器不可控制 Form Dispose
- 不可使用 SingleInstance 註冊 Form

請在 README 中明確寫出：

> 容器只負責建立 Form，生命週期由 FormManager 控制。

# 二、全域設計優先原則（強制遵守）

優先順序：

1. 穩定性
2. 相容性
3. 效能
4. 可預測性
5. 可維護性
6. 擴充性
7. 優雅設計

強制要求：

1. 必須支援 24/7 長時間運作
2. 不可有記憶體洩漏
3. 不可產生 GDI / Handle 累積
4. Dispose 必須完整釋放資源且可安全重入
5. 所有 public 方法必須 thread-safe
6. 避免 race condition / deadlock / re-entrancy
7. 不可使用 Thread.Abort
8. 僅使用 Windows 7/10 相容 API
9. 減少物件分配與 GC 壓力
10. 僅建立必要抽象層
11. 所有例外必須捕捉並可回報
12. 每個類別與介面必須有 XML Summary（正體中文）
13. 若設計與穩定性或效能衝突，優先穩定性與效能

說明：

- 本專案不涉及背景 Task
- 不涉及 I/O 執行緒
- 不涉及設備並行控制
- 不需實作 CancellationToken
- 不做事件排程

# 三、專案定位

專案名稱：
Calin.UI.FormManager

性質：
基礎 UI 管理類別庫（Class Library）

設計核心：

1. 不讓 Form 自行 new 自己
2. 統一由呼叫端提供 factory
3. FormManager 僅集中管理 reference
4. 不使用 DI
5. 不使用泛型工廠
6. 不使用反射
7. 不使用事件匯流排
8. 不使用背景執行緒
9. 不使用 Timer
10. 不使用弱參考
11. 不隱藏邏輯
12. 呼叫流程明確可單步除錯
13. UI Thread 統一入口

# 四、三種視窗型別設計

## 1️⃣ ModalForm（阻斷流程視窗）

API：

ShowModal(Func factory, Form owner)

特性：

1. 使用 ShowDialog(owner)
2. 同時間只能存在一個
3. 開啟時顯示於螢幕正中央
4. FormBorderStyle.FixedDialog
5. 可移動
6. 不可改尺寸
7. 關閉後立即 Dispose
8. 不保留 reference

行為要求：

1. using 包裹
2. 嚴格指定 Owner
3. 不可 TopMost
4. Debug 可檢查重入
5. 結束後不得殘留 Handle

## 2️⃣ ToolForm Type 1（監控視窗 - 每次重建）

API：

ShowTool(string id, Func factory, Form owner)

特性：

1. 可與主畫面同時操作
2. Owner = 主畫面
3. TopMost = true
4. 可同時多個
5. 使用 string ID 識別
6. 關閉後 Dispose
7. 關閉時記錄位置
8. 下次重建時還原位置

管理方式：

1. 開啟期間存於 Dictionary
2. FormClosed 時移除 reference
3. 寫入 JSON 位置資訊
4. 不可長期殘留

## 3️⃣ ToolForm Type 2（Persistent Tool 視窗）

API：

ShowPersistent(string id, Func factory, Form owner)
ClosePersistent(string id)
DisposeAllPersistent()

特性：

1. 關閉僅 Hide
2. 由 FormManager 長期持有 reference
3. 可同時多個
4. 位置持久化
5. 僅 Application 結束時 Dispose

行為要求：

1. 已存在 → Show + Activate
2. FormClosing → Cancel + Hide
3. 關閉時寫入 JSON
4. Owner 必須指定
5. TopMost = true

# 五、位置持久化設計

需實作：

LocationStore

要求：

1. 使用 JSON
2. 不使用外部套件
3. 記錄：
    - ID
    - X
    - Y
    - Width
    - Height
4. 啟動時載入
5. 關閉時寫入
6. Thread-safe
7. 僅必要時寫入
8. 檔案損毀需自動復原

# 六、UI Thread 與 Thread-Safe 設計

1. FormManager 建立時記錄 UI SynchronizationContext
2. 所有 public API：
    - 若非 UI Thread → 自動 Invoke
3. 不使用背景執行緒
4. Dictionary 操作需安全
5. 不鎖 UI 執行緒
6. 不產生 re-entrancy

# 七、穩定性強化機制

## 1️⃣ ValidateLocation

1. 若超出 Screen.WorkingArea
2. 多螢幕變動導致黑屏
3. 自動重新置中

## 2️⃣ Z-Order 控制

1. 強制指定 Owner
2. Tool 強制 TopMost
3. 不使用全域 TopMost hack

## 3️⃣ GDI / Handle 安全

1. 嚴格 Dispose
2. Modal 不可殘留
3. 非 Persistent 不可隱性殘留
4. 不可存在隱藏未釋放視窗

## 4️⃣ GC 壓力控制

1. 僅使用必要 Dictionary
2. 不建立大量集合
3. 不建立長期匿名委派鏈
4. JSON 僅於關閉時寫入

# 八、Debug 安全機制（#if DEBUG）

1. 重複 ID 開啟檢查
2. Modal 重入檢查
3. Owner 未指定檢查
4. Persistent 重複 Dispose 檢查
5. 未在 UI Thread 呼叫警告

Release 不增加額外負擔。

# 九、README 必須包含

1. 架構責任邊界說明
2. DI 使用方式說明
3. 禁止容器控制 Form 生命週期
4. 禁止 SingleInstance 註冊 Form
5. 三種視窗使用範例（含 DI / 非 DI）

# 十、輸出內容

請生成：

1. FormManager 完整實作
2. LocationStore 實作
3. ValidateLocation 方法
4. 三種視窗範例
5. MainForm 呼叫範例
6. README
7. 工控 24/7 穩定性分析
8. Thread-Safe 設計說明
9. XML Summary（正體中文）

# 十一、程式碼必須符合

1. 可直接貼入 .NET 4.6.2~4.8 專案
2. 不依賴外部套件
3. 不使用 DI
4. 不使用反射
5. 不使用泛型工廠
6. 不使用事件匯流排
7. 不使用背景監控
8. 不產生 GDI/Handle 累積
9. 可單步除錯
10. 可安全 24/7 長期運行

請開始生成完整專案實作。
