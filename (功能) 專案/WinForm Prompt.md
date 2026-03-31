---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# MessageBox / Toast Notification / MessageTip

# 使用建議（實務）

- 三輪 PROMPT 可以一次準備好
- 實際使用時請「一輪一貼」
- 每一輪完成後先檢視：
    - 是否有寫死尺寸
    - 是否混入不必要抽象
    - 是否破壞前一輪行為

如果你下一步想要，我可以再幫你做：

- 每一輪的「完成檢查清單」
- Round 1 專用的「Copilot 修正追問 PROMPT」
- 或直接幫你審視 Copilot 產出的程式碼結構是否踩雷

---

# Round 1：Toast MVP（最小可用實作）PROMPT

你是一位資深 .NET WinForms 工程師，請先實作 **Toast Notification 的 MVP**，目的在於驗證行為與架構方向是否正確，而不是做完整功能。

## 目標

- 只做 Toast Notification
- 不做 Dialog、不做 MessageTip
- 不抽共用核心，但要避免未來重構困難

## 功能需求

- 使用 WinForms
- 無邊框 Form（FormBorderStyle = None）
- StartPosition = Manual
- 顯示於螢幕右下角（Screen.WorkingArea）
- 可同時顯示多個 Toast，並往上堆疊
- 每個 Toast 指定秒數後自動關閉
- 關閉後其餘 Toast 需重新排列
- 不搶焦點（不可影響主視窗操作）

## 視覺與尺寸

- 視窗大小由文字內容自動決定
- 文字量測使用 TextRenderer.MeasureText
- 不可寫死 Width / Height
- 不使用 AutoSize + Dock 混搭
- Toast 的 **標題列（Title）與內容（Content）** 字型與大小須可以自訂
- 可在 MessageStyle 中預留屬性，雖然 MVP 可能只用 Content 顯示

## 架構要求

- 命名空間：`Calin.Infrastructure.MessageSystem.Presenter.Toast`
- 拆出：
    - ToastForm（只負責顯示）
    - ToastManager（負責堆疊與位置計算）
- Timer 控制生命週期
- 不做動畫、主題、Icon、Button

## 技術限制

- 不使用第三方 UI 套件
- 不使用 async/await 操作 UI
- 可在 **.NET Framework 4.6.2 / 4.8 / .NET 8 Windows** 下運作

## DI 與 NuGet 預留

- 將使用 AutoFac 註冊 ToastManager
- 暫時只寫類別，不做註冊

## 產出要求

- 類別責任說明
- ToastForm 程式碼
- ToastManager 程式碼
- 示範如何呼叫 ShowToast()
- 撰寫簡單 README.md（說明 Toast MVP 使用方式）

---

# Round 1（Toast MVP）專用：Copilot 修正追問 PROMPT

以下這組 PROMPT 是「**在 Copilot 已經幫你產出 Round 1 程式碼之後**」，你用來**校正、修剪、避免走歪**的追問清單。
用途不是加功能，而是**把 MVP 拉回正確軌道**。

你不需要全部用，通常挑 2～4 條就夠。

## **使用順序建議**（很重要）

實務上我建議你用這個順序：

1. 追問 1（責任是否乾淨）
2. 追問 2（尺寸是否正確）
3. 追問 4（堆疊是否穩定）
4. 視需要再用 3 或 5
5. 最後用 6 清掉過度設計

## 一句實務總結

**Round 1 的修正追問不是在「變強」，而是在「防止你未來後悔」。**

如果你要，我可以下一步再幫你準備：

- Round 2 專用的「抽象品質檢查 PROMPT」
- 或一份「Toast MVP 通過與否」的快速審核清單

## 追問 1：修正架構責任（最常用）

請檢視目前的 Toast MVP 程式碼，確認是否有以下問題：

- 堆疊位置計算寫在 ToastForm 裡
- ToastForm 同時負責顯示與生命週期管理
- Form 直接管理其他 Toast 的位置

如果有，請重構為：

- ToastForm 只負責顯示與關閉事件
- ToastManager 專責：
    - 記錄目前顯示中的 Toast
    - 計算右下角堆疊位置
    - 在 Toast 關閉後重新排列

請給出重構後的類別責任與必要程式碼，不新增任何新功能。
更新 README.md。

## 追問 2：尺寸計算是否真的由內容驅動（關鍵）

請檢查目前 ToastForm 是否存在以下情況：

- Width / Height 寫死或半寫死
- 使用 AutoSize
- 使用 Label.AutoSize
- 使用 Dock + AutoSize 混用

如果有，請修正為：

- 使用 TextRenderer.MeasureText 量測文字
- 由量測結果計算視窗 Size
- 明確寫出尺寸計算流程

請只修改尺寸計算相關程式碼，其餘行為保持不變。
更新 README.md。

## 追問 3：避免未來重構困難的寫法

請檢視 Toast MVP 中是否有以下「未來難以抽共用核心」的寫法：

- 在 OnPaint 中硬切座標
- 在 Form 建構子中混合視覺、定位、計時邏輯
- 使用匿名 Timer 或 Lambda 導致生命週期不清楚

如果有，請重構成：

- 明確的欄位（Timer 為成員）
- 清楚的 Initialize / Start / Close 流程
- 不改變外部呼叫方式

更新 README.md。

## 追問 4：多個 Toast 關閉後的重排正確性

請模擬以下情境並檢查程式碼是否正確：

- 同時顯示 3 個 Toast
- 中間那一個先關閉
- 上下 Toast 是否會正確重新排列

若目前邏輯無法保證正確，請調整 ToastManager 的重排流程，但不要加入動畫或新功能。
更新 README.md。

## 追問 5：確保不影響主視窗焦點

請檢查 ToastForm 是否可能：

- 搶走目前 ActiveForm 焦點
- 造成使用者輸入中斷

若有風險，請修正為：

- 不影響主視窗操作
- 使用 WinForms 合理方式避免焦點切換
- 不使用 hack 或非官方 API

更新 README.md。

## 追問 6：確認 MVP 邊界（防止過度設計）

請檢視目前 Toast MVP 是否已經包含以下「不屬於 MVP」的內容：

- Icon
- Button
- 動畫
- Theme / Style 抽象
- MessageBox 行為

如果有，請移除或延後，並說明哪些部分應留到下一輪重構再做。
更新 README.md。

---

# Round 2：抽出共用核心（MessageStyle / LayoutEngine）PROMPT

你是一位資深 .NET WinForms 架構師，請在 **已完成 Toast MVP 程式碼** 基礎上進行重構，抽出共用核心。

## 核心重點

- **MessageStyle** 需支援：
    - TitleFont / TitleFontSize / TitleForeColor / TitleBackColor
    - ContentFont / ContentFontSize / ContentForeColor / ContentBackColor
    - ButtonFont / ButtonFontSize / ButtonForeColor / ButtonBackColor
    - Padding / Spacing
    - Icon（預留）
- **MessageLayoutEngine**：
    - 輸入：MessageStyle、Title 文字、Content 文字、Button 集合、最大寬度
    - 輸出：TitleBounds、ContentBounds、ButtonBounds、IconBounds、TotalSize

## 命名與架構

- namespace：`Calin.Infrastructure.MessageSystem.Core`
- LayoutEngine 不依賴 Toast / Dialog / Tip
- ToastForm 改為只吃 LayoutEngine 計算結果，並能套用 TitleFont / ContentFont / 顏色
- 不在 OnPaint 中硬切座標
- 不使用 Label AutoSize

## 產出要求

- 類別關係說明
- MessageStyle 程式碼
- MessageLayoutEngine 程式碼
- ToastForm 改用 LayoutEngine + 可自訂字型與顏色的範例程式碼
- README.md 更新，補充 Title/Content/按鈕自訂字型與顏色說明

---

# Round 3：完整系統補齊（Dialog / Button / Tip / NuGet / AutoFac / README.md）PROMPT

你是一位資深 .NET WinForms 架構師，請在「已完成共用核心與 Toast」的前提下，補齊完整訊息系統，並可發佈為 NuGet。

## 核心要求

- Dialog / Toast / MessageTip 都必須套用 **MessageStyle**
- 使用者可自訂：
    - 標題列字型、字體尺寸、文字顏色、背景顏色
    - 內容字型、字體尺寸、文字顏色、背景顏色
    - 按鈕字型、字體尺寸、文字顏色、背景顏色
- LayoutEngine 自動計算 Title / Content / Button 尺寸與位置

## DialogButtonDefinition / DialogButtonSet

- 支援 ButtonFont / ButtonForeColor / ButtonBackColor
- Icon 支援 Image / Icon / Embedded Resource，可左右排列
- Default / Cancel 標記

## MessageTip / Toast

- 同樣套用 MessageStyle，支援自訂字型、顏色
- 尺寸與位置由 LayoutEngine 計算

## NuGet 與 Target Framework

- NuGet 封裝：`Calin.Infrastructure.MessageSystem`
- 支援：
    - net462
    - net48
    - net8.0-windows

## AutoFac DI

- Module：`Calin.Infrastructure.MessageSystem.MessageSystemModule.cs`
- 註冊 ToastManager、MessageDialogPresenter、MessageTipPresenter

## 產出要求

1. 類別與責任說明
2. MessageDialogPresenter 實作（DialogButtonSet + 字型顏色自訂）
3. ButtonPanel + Icon 排版 + 字型顏色自訂
4. MessageTip Presenter 實作
5. ToastPresenter 已套用字型與顏色自訂
6. AutoFac 註冊範例
7. NuGet 專案結構與 multi-target 設定說明
8. 撰寫完整 README.md
    - 說明三種訊息形式使用方式
    - DialogButtonSet 建立與使用方法
    - Toast / MessageTip 使用示範
    - DI AutoFac 註冊方式
    - **標題列 / 內容 / 按鈕字型、字體大小與顏色自訂示範**
    - 支援 .NET462 / .NET48 / .NET8
    - NuGet 使用方式
    - 圖表使用 Mermaid 格式
