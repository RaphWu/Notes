---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 結論

- **釋放資源、儲存狀態、正常關閉流程控制 → FormClosing**
- **UI 已銷毀後的補充清理、紀錄最終狀態 → FormClosed**
- **大多數「結束動作」應該寫在 FormClosing**

# FormClosing 與 FormClosed 的本質差異

- **FormClosing**
    - 視窗尚未關閉，控制項仍存在
    - 可取消關閉（`e.Cancel = true`）
    - 可顯示對話框、詢問使用者
    - 適合進行「是否允許關閉」與「安全收尾」
- **FormClosed**
    - 視窗已關閉，Handle 已釋放
    - 無法取消
    - 不適合再操作 UI
    - 僅適合做與 UI 無關的最終處理

# 建議放在 FormClosing 的典型工作

- 詢問使用者是否要離開
- 停止背景執行緒、Timer、Task
- 關閉硬體通訊（PLC、COM、Socket、DAQ）
- 儲存設定檔、使用者狀態
- 停止資料擷取或即時控制流程
- 控制應用程式結束順序

# 適合放在 FormClosed 的情境

- 記錄「程式已結束」的日誌
- 通知其他非 UI 元件視窗已消失
- 回收與 UI 無關的全域資源（極少見）
- 多視窗架構中，追蹤視窗生命週期狀態

# 實務建議寫法

- **90% 結束邏輯寫在 FormClosing**
- **FormClosed 只保留極輕量、不可逆的處理**
- 若涉及硬體或背景流程，務必在 FormClosing 完成關閉

# 常見錯誤

- 在 FormClosed 中才停止 Thread 或 Timer
- 在 FormClosed 操作控制項或顯示對話框
- 重要資源釋放寫在 FormClosed，導致例外或未正常關閉

# 一句話原則

- **能被取消、需要 UI、需要判斷的 → FormClosing**
- **單純事後通知或紀錄 → FormClosed**

如果你是工控或即時控制程式，請把「停機邏輯」一律放在 **FormClosing**，不要賭 FormClosed 一定會被完整執行。
