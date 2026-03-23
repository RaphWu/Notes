---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 官方簡介

- 為了統一所有研華運動設備的用戶接口，研華運動設備採用了新的軟體架構，名為「通用運動架構 」。
- 該架構定義了所有用戶介面和具有的所有運動功能，包括單一軸和多軸。這種統一的程式設計平台使用戶能夠以相同的方式操作設備。
- 架構包括三層：**裝置驅動層**、**整合層** 和 **應用層**。用戶無需了解如何操作特定設備的特定驅動，只需了解通用運動驅動即可。即使支援該架構的設備發生變化，應用也無需修改。
- 研華通用運動 (ACM) 架構定義了三種操作對象：設備、軸心和群組。每種物件都擁有自己的方法、屬性和狀態。
- 所有操作都可透過呼叫對應 ACM API 來完成。設備、軸和群組的常用呼叫流程由通用運動架構進行定義。詳細資訊請參考 “ 呼叫流程”。

---

# 命名規則

## 功能函數命名規則

|操作對象|命名規則|說明|舉例|
|---|---|---|---|
|設備|Acm_DevXXX|執行設備功能|Acm_DevOpen|
|AIO/DIO 操作|Acm_DaqXXX|執行 DI/DO、AI/AO 功能|Acm_DaqDoGetBit|
|軸操作|Acm_AxXXXX|執行軸功能|Acm_AxOpen|
|群組操作|Acm_GpXXXX|執行群組功能|Acm_GpAddAxis|

## 屬性命名規則

### 參數

參數屬性值會經常變化。

|命名規則|分類規則|
|---|---|
|設備|PAR_DevXXX|
|軸|PAR_AxXXXX|
|群組|PAR_GpXXXX|
|DIO/AIO|PAR_DaqXXX|

### 配置

配置屬性值部分可重新設置，其他不可設置，主要設定屬性可變但不會經常變化。

|命名規則|分類規則|
|---|---|
|設備|CFG_DevXXX|
|軸|CFG_AxXXXX|
|群組|CFG_GpXXXX|
|DIO/AIO|CFG_DaqXXX|

### 特性

此屬性與硬體特性有關，只能取得設備的相關屬性，不能設定。

|命名規則|分類規則|
|---|---|
|設備|FT_DevXXX|
|軸|FT_AxXXX|
|DIO/AIO|FT_DaqXXX|

---

# 資料類型定義和 Windows 資料類型對照表

|新類型|C++|備註|C#|
|---|---|---|--|
|U8|uchar|8 bit 無符號整數|System.Byte|
|U16|ushort|16 bit 無符號整數|System.UInt16|
|int|ulong|32 bit 無符號整數|System.UInt32|
|U64|ulonglong|64 bit 無符號整數|System.UInt64|
|I8|char|8 bit 帶符號整數|System.SByte|
|I16|short|16 bit 帶符號整數|System.Int16|
|I32|int|32 bit 帶符號整數|System.Int32|
|I64|longlong|64 bit 帶符號整數|System.Int64|
|F32|flaot|32 bit 浮點變數|System.Single|
|F64|double|64 bit 浮點變數|System.Double|
|PU8|uchar*|指標指向 8 bit 無符號整數|
|PU16|ushort*|指標指向 16 bit 無符號整數|
|PU32|ulong*|指標指向 32 bit 無符號整數|
|PU64|ulonglong*|指標指向 64 bit 無符號整數|
|PI8|char*|指標指向 8 bit 帶符號整數|
|PI16|short*|指標指向 16 bit 帶符號整數|
|PI32|int*|指標指向 32 bit 帶符號整數|
|PI64|longlong*|指標指向 64 bit 帶符號整數|
|PF32|flaot*|指標指向 32 bit 浮點變數|
|PF64|double*|指標指向 64 bit 浮點變數|

---

# AcmStatus 運動狀態

在使用運動控制卡之前，必須配置一些參數（例如速度、ALM 訊號、左限位和右限位）。配置軸卡屬性的過程稱為系統配置過程 (system configuration process)。

AcmStatus 檔案定義了研華通用運動架構 (ACM) 中所有的狀態值。這些狀態值用於表示設備、軸和群組的當前狀態，並可用於監控和控制運動過程。

|函數|說明|
|---|---|
|`Acm_AxGetState` |取得軸的目前狀態，將傳回 16 位元的狀態值。|
|`Acm_AxGetMotionStatus` |取得軸的目前運動狀態，將傳回 32 位元的軸當前運動狀態值。|

當打開板卡，打開軸後，正常情況下，軸的狀態將為 `Ready` 。只有當軸的狀態為 `Ready` 時，才能執行新的運動操作，如連續運動。
當開啟板卡未開啟軸時，軸的狀態為停用 (`STA_AX_DISABLE`), 此時執行運動操作將會報錯。

## 軸速度

|函數|說明|
|---|---|
|`Acm_AxChangeVel`|當軸在運動過程中，指令軸改變運轉速度。|
|`Acm_AxGetCmdVelocity`|取得指定軸的目前運行速度。|
|`Acm_AxChangeVelEx`|改變運轉軸的運轉速度、加速度、減速度。|
|`Acm_AxChangeVelExByRate`|依比率改變運轉軸的運轉速度、加速度、減速度。|
|`Acm_AxChangeVelByRate`|依比例改變運轉軸的運轉速度。|

- 開啟板卡和軸後，可設定 / 取得軸的當前速度值，如初速、加速度、運轉速度。
- 設定的速度參數不能大於運動軸的最大速度參數，否則會報錯，例如配置軸的最大速度 (`CFG_AxMaxVel`) 為 10000，設定軸的運轉速度 (`PAR_AxVelHigh`) 為 11000，運行時將會提示報錯。
- 在軸的運轉過程中，可呼叫函數改變運行軸的 **運轉速度**、**加速度**、**減速度**。

## 編碼器

|函數|說明|
|---|---|
|`Acm_AxSetCmdPosition`|設定指定軸的理論 (指令) 位置。|
|`Acm_AxSetActualPosition`|設定指定軸的實際 (回饋) 位置。|
|`Acm_AxGetCmdPosition`|取得指定軸的目前理論 (指令) 位置。|
|`Acm_AxGetActualPosition`|取得指定軸的目前實際 (回饋) 位置。|
|`Acm_AxGetState`|取得軸的目前狀態。|

## 軸的運動 I/O 狀態

|函數|說明|
|---|---|
|`Acm_AxGetMotionIO`|取得軸的運動 I/O 狀態，傳回 32 位元的狀態值。|

驅動器警報標誌、限位觸發標誌觸發以後，不會自動清除 0。只有當產生異常的原因消
除後，呼叫 `Acm_AxResetError` 函數，軸的狀態變成 `Ready`。

## 群組狀態

|函數|說明|
|---|---|
|`Acm_GpGetState`|取得群組的目前狀態，將傳回 16 位元的群組狀態值。|

當打開板卡，打開軸後，需要呼叫函數添加軸到群組中，當群組中軸的數量大於等於
1 時，正常情況下，群組的狀態將為 `Ready`，此時可執行群組操作。當群組的狀態為
`Disable` 時，此時執行群組運動操作將會報錯。

## 群組速度

|函數|說明|
|---|---|
|`Acm_GpGetCmdVel`|取得群組的目前的速度值|

闆卡初始化後，可設定 / 取得群組的目前速度值，如初速度，加速度，運行速度。運行過程中，可呼叫 `Acm_GpGetCmdVel` 函數取得群組的目前速度值。

---

# 軸運動

## Single-axis motion 單軸運動

單軸運動是指單軸的移動，包括以下幾種模式：

- P2P motion 點對點運動。
- Continuous motion 連續運動。
- Change position motion 位置改變運動。
- Change velocity motion 速度改變運動。
- Simultaneous start/stop motion 同步啟動/停止運動。
- Imposed motion 疊加運動。
- Jog motion JOG 運​​動、手輪運動。
- Homing 歸位運動。

下面函數，除連續運動沒有外，其餘 7 種運動都有：

| 函數                 | 說明                       |
| ------------------ | ------------------------ |
| Acm_SetU32Property | 設定屬性值 (屬性值為無符號 32 位元整數)。 |
| Acm_SetI32Property | 設定屬性值 (屬性值為有符號 32 位元整數)。 |
| Acm_SetF64Property | 設定屬性值 (屬性值為 Double 型)。   |
| Acm_GetU32Property | 取得屬性值 (屬性值為無符號 32 位元整數)。 |
| Acm_GetI32Property | 取得屬性值 (屬性值為有符號 32 位元整數)。 |
| Acm_GetF64Property | 取得屬性值 (屬性值為 Double 型)。   |
| Acm_AxGetState     | 取得軸的目前狀態。                |
| Acm_AxResetError   | 重設軸的狀態。                  |

下面函數，為點對點運動、位置改變運動、速度改變運動、疊加運動、歸位運動都有的：

| 函數              | 說明             |
| --------------- | -------------- |
| Acm_AxStopDec   | 指令軸以設定的減速停止運行。 |
| Acm_AxStopEmg   | 指令軸立刻停止（無減速）。  |
| Acm_AxStopDecEx | 下達停止指令時可指定減速度。 |

### 點對點運動

P.75

| 函數                      | 說明                  |
| ----------------------- | ------------------- |
| Acm_AxMoveAbs           | 單軸的絕對點到點運動。         |
| Acm_AxMoveRel           | 單軸的相對點到點運動。         |
| Acm_AxSetCmdPosition    | 設定指定軸的理論 (指令) 位置。   |
| Acm_AxSetActualPosition | 設定指定軸的實際 (回饋) 位置。   |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論 (指令) 位置。 |
| Acm_AxGetActualPosition | 取得指定軸的目前實際 (回饋) 位置。 |
| Acm_AxGetCmdVelocity    | 取得目前軸的理論 (指令) 速度。   |

| 參數            | 說明                    |
| ------------- | --------------------- |
| PAR_AxVelLow  | 設定/取得此軸的初始速度 (起始速度)。  |
| PAR_AxVelHigh | 設定/取得該軸的運轉速度。         |
| PAR_AxAcc     | 設定/取得該軸的加速度。          |
| PAR_AxDec     | 設定/取得該軸的減速度。          |
| PAR_AxJerk    | 設定/取得速度曲線的類型：T/S 型曲線。 |

| 配置           | 說明           |
| ------------ | ------------ |
| CFG_AxMaxVel | 配置運動軸的最大速度。  |
| CFG_AxMaxAcc | 配置運動軸的最大加速度。 |
| CFG_AxMaxDec | 配置運動軸的最大減速。  |

- 當軸的狀態為 `Ready` 時，將能執行點到點運動。
- 點到點運動分為兩種：**相對運動** 和 **絕對運動**。
	- 相對運動：以目前的理論指令位置為參考位置作位置的移動。
	- 絕對運動：以絕對零點指令位置為參考位置作位置的移動。
- 點到點運動模式下，可單獨設定各軸的目標位置、初速、加速度等運動參數，各軸獨立運作或停止。在運動過程中，可改變目標位置和運行速度。
- 流程圖在 5.2.2.3 節 (P.77)。

### 連續運動

P.78

| 參數            | 說明                  |
| ------------- | ------------------- |
| PAR_AxVelLow  | 設定/取得該軸的低速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得此軸的高速度 (驅動速度)。 |
| PAR_AxAcc     | 設定/取得該軸的加速度。        |
| PAR_AxDec     | 設定/取得該軸的減速度。        |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。  |

| 配置           | 說明           |
| ------------ | ------------ |
| CFG_AxMaxVel | 配置運動軸的最大速度。  |
| CFG_AxMaxAcc | 配置運動軸的最大加速度。 |
| CFG_AxMaxDec | 配置運動軸的最大減速。  |

- 連續運動指命令軸依規定速度和方向執行沒有終點的運動。
- 執行連續運動過程中，可單獨設定各軸的速度參數。如初速度、加速度等。各軸將獨立運行或停止。
- 連續運動過程中，可改變運轉軸的運轉速度。
- 執行連續運動時，軸的狀態必須為 `Ready`。軸的狀態可透過 `Acm_AxGetState` 函數取得。
- 流桯圖在 5.2.3.3 (P.79)。

### 位置改變運動

P.81

| 函數                      | 說明                  |
| ----------------------- | ------------------- |
| Acm_AxChangePos         | 改變執行點到點運動軸的終點位置。    |
| Acm_AxMoveAbs           | 單軸的絕對點到點運動。         |
| Acm_AxMoveRel           | 單軸的相對點到點運動。         |
| Acm_AxSetCmdPosition    | 設定指定軸的理論 (指令) 位置。   |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論 (指令) 位置。 |
| Acm_AxSetActualPosition | 設定指定軸的實際 (回饋) 位置。   |
| Acm_AxGetActualPosition | 取得指定軸的目前實際 (回饋) 位置。 |

| 參數            | 說明                  |
| ------------- | ------------------- |
| PAR_AxVelLow  | 設定/取得該軸的低速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得此軸的高速度 (驅動速度)。 |
| PAR_AxAcc     | 設定/取得該軸的加速度。        |
| PAR_AxDec     | 設定/取得該軸的減速度。        |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。  |

| 配置           | 說明           |
| ------------ | ------------ |
| CFG_AxMaxVel | 配置運動軸的最大速度。  |
| CFG_AxMaxAcc | 配置運動軸的最大加速度。 |
| CFG_AxMaxDec | 配置運動軸的最大減速。  |

- 變位運動指軸在執行點到點運動過程中, 可改變點到點運動的目標位置。
- 當呼叫 `ChangePos` 函數改變運行軸的目標位置時，執行結果與軸的當前運動位置有關：
	- 當 Change 的位置大於目前運動位置時，將運行到 Change 的位置。
	- 當 Change 的位置小於初始設定的目標位置，運行結果與軸的目前位置有關。見 P.82 圖：
- 流程圖位於 5.2.4.3 節 (P.83)。

### 速度改變運動

P.84

| 函數                      | 說明                  |
| ----------------------- | ------------------- |
| Acm_AxMoveVel           | 軸以規定速度進行沒有終點的運動。    |
| Acm_AxMoveAbs           | 開始單軸的絕對點到點運動。       |
| Acm_AxMoveRel           | 開始單軸的相對點到點運動。       |
| Acm_AxChangeVel         | 當軸在運動過程中，指令軸改變速度。   |
| Acm_AxChangeVelEx       | 改變運轉軸的運轉速度、加速度、減速度。 |
| Acm_AxChangeVelByRate   | 依照設定的比例改變目前軸的運轉速度。  |
| Acm_AxChangeVelExByRate | 依比率改變軸運轉速度、加速度、減速度。 |
| Acm_AxGetCmdVelocity    | 取得指定軸的目前理論速度。       |
| Acm_AxSetCmdPosition    | 設定指定軸的理論位置。         |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論位置。       |
| Acm_AxSetActualPosition | 設定指定軸的實際位置。         |
| Acm_AxGetActualPosition | 取得指定軸的目前實際位置。       |

| 參數            | 說明                  |
| ------------- | ------------------- |
| PAR_AxVelLow  | 設定/取得該軸的低速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得此軸的高速度 (驅動速度)。 |
| PAR_AxAcc     | 設定/取得該軸的加速度。        |
| PAR_AxDec     | 設定/取得該軸的減速度。        |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。  |

| 配置           | 說明           |
| ------------ | ------------ |
| CFG_AxMaxVel | 配置運動軸的最大速度。  |
| CFG_AxMaxAcc | 配置運動軸的最大加速度。 |
| CFG_AxMaxDec | 配置運動軸的最大減速。  |

- 變速運動指當軸執行點到點運動或連續運動時，改變目前軸的運轉速度、加速度、減速度。**改變的速度需大於初速**。
- 呼叫 `Acm_AxChangeVel` 函數改變指定軸的運行速度。若命令成功下達，下次運動之前沒有重新設定運行速度，新的運行速度會作用到下次運動中。
- 呼叫 `Acm_AxChangeVelEx` 函數改變運行速度、加速度、減速度。若命令成功下達，下次運動前沒有重新設定運動速度，則新的運行速度會作用到下次運動。
- 若新的加速度、減速度設定為 `0`，則使用上一次設定的加速度或減速度值。
- 呼叫 `Acm_AxChangeVelExByRate` 函數根據比率改變運行速度，同時改變加速度和減速度。
- 新的速度，加速度和減速度僅對目前運動有效。若新的加速度減速度為 `0`，則使用上一次設定的加速度或減速度值。
- 呼叫 `Acm_AxChangeVelByRate` 函數根據比率改變運行速度。新的速度僅對當前運動有效。
- 流程圖在 5.2.5.3 節 (P.86)。

### 同步啟動/停止運動

P.87

| 函數                       | 說明                |
| ------------------------ | ----------------- |
| Acm_AxSimStartSuspendAbs | 設定軸為等待做相對點到點運動狀態。 |
| Acm_AxSimStartSuspendRel | 設定軸為等待做絕對點到點運動狀態。 |
| Acm_AxSimStartSuspendVel | 設定軸為等待做連續運動狀態。。   |
| Acm_AxSimStart           | 啟動所有等待啟動觸發的軸。。    |
| Acm_AxSimStop            | 停止所有觸發的軸。         |
| Acm_AxSetCmdPosition     | 設定指定軸的理論位置。       |
| Acm_AxGetCmdPosition     | 取得指定軸的目前理論位置。     |
| Acm_AxSetActualPosition  | 設定指定軸的實際位置。       |
| Acm_AxGetActualPosition  | 取得指定軸的目前實際位置。     |
| Acm_AxGetCmdVelocity     | 取得目前軸的理論 (指令) 速度。 |

| 參數            | 說明                  |
| ------------- | ------------------- |
| PAR_AxVelLow  | 設定/取得該軸的低速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得該軸的高速度 (驅動速度)。 |
| PAR_AxAcc     | 設定/取得該軸的加速度。        |
| PAR_AxDec     | 設定/取得該軸的減速度。        |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。  |

| 配置                   | 說明               |
| -------------------- | ---------------- |
| CFG_AxMaxVel         | 配置運動軸的最大速度。      |
| CFG_AxMaxAcc         | 配置運動軸的最大加速度。     |
| CFG_AxMaxDec         | 配置運動軸的最大減速。      |
| CFG_AxSimStartSource | 設定/取得目前軸的同步起停模式。 |

| 特性                     | 說明          |
| ---------------------- | ----------- |
| FT_AxSimStartSourceMap | 軸支援的同步起停模式。 |

- 同步啟動/停止運動指令等待軸 ( 等待同時開始運轉的多個軸 ( 一個以上 ))，根據同步啟動模式，同時做點到點運動或連續運動，並可同時停止所有軸的運轉。同步啟運動的所有軸，同時啟動後將以各自設定的速度運行到目標位置。
- 運轉過程中執行 `Acm_AxSimStop` 指令，將同時給予所有軸停止的訊號，各軸會依據各自的設定的減速度停止運轉。
- 同步啟動、同步停止模式透過屬性 `CFG_AxSimStartSource` 設定，不同的屬性值，代表不同的同步啟動模式。
- 設定同步啟動模式後，根據運動模式設定軸為等待狀態：
    1. 當運動模式為絕對點到點運動時，呼叫 `Acm_AxSimStartSuspendAb` 函數將軸設為等待狀態。
    2. 當運動模式為相對點位運動時，呼叫 `Acm_AxSimStartSuspendRel` 函數將軸設為等待狀態。
    3. 當運動模式為連續運動時，呼叫 `Acm_AxSimStartSuspendRel` 函數將軸設為等待狀態。
- 最後呼叫 Acm_AxSimStart 函數發出同步啟動停止訊號，啟動所有等待啟動觸發的軸，軸會根據同步啟動停止模式啟動觸發。
- 流程圖在 5.2.6.3 節 (P.90)。

### 疊加運動

P.92

| 函數                      | 說明            |
| ----------------------- | ------------- |
| Acm_AxMoveImpose        | 在目前運動上疊加新的運動。 |
| Acm_AxMoveAbs           | 開始單軸的絕對點到點運動。 |
| Acm_AxMoveRel           | 開始單軸的相對點到點運動。 |
| Acm_AxSetCmdPosition    | 設定指定軸的理論位置。   |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論位置。 |
| Acm_AxSetActualPosition | 設定指定軸的實際位置。   |
| Acm_AxGetActualPosition | 取得指定軸的目前實際位置。 |

| 參數            | 說明                  |
| ------------- | ------------------- |
| PAR_AxVelLow  | 設定/取得該軸的低速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得此軸的高速度 (驅動速度)。 |
| PAR_AxAcc     | 設定/取得該軸的加速度。        |
| PAR_AxDec     | 設定/取得該軸的減速度。        |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。  |

| 配置           | 說明           |
| ------------ | ------------ |
| CFG_AxMaxVel | 配置運動軸的最大速度。  |
| CFG_AxMaxAcc | 配置運動軸的最大加速度。 |
| CFG_AxMaxDec | 配置運動軸的最大減速。  |

- 疊加運動指在目前運動上疊加新的運動，運動停止的終止位置將為原始位置加、減新設定的位置。
  例如軸執行點到點運動，目標位置為 10000，呼叫疊加運動函數 `Acm_AxMoveImpose` 設定疊加相對位置為 3000，則軸運行到 13000 後停止運轉；當設定的目標位置為 -3000 時，則軸運行到 7000 後停止運轉。
- 支持新疊加的運動的速度曲線為 T 型曲線。
- 疊加運動功能函數 `Acm_AxMoveImpose` 可設定新疊加段的位置和速度。
- 整個速度曲線由該運動的 `NewVel`、原始運動的 `PAR_AxVelLow`、`PAR_AxVelHigh`、`PAR_AxAcc`、`PAR_AxDec`、`PAR_AxJerk` 決定。
- 不能在疊加運動上疊加新的運動。
- 流程圖在 5.2.7.3 節 (P.94)。

### JOG 運​​動、手輪運動

P.95

| 函數                      | 說明            |
| ----------------------- | ------------- |
| Acm_AxSetExtDrive       | 啟用或停用外部驅動模式。  |
| Acm_AxJog               | 指定軸執行 Jog 運動。 |
| Acm_AxSetCmdPosition    | 設定指定軸的理論位置。   |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論位置。 |
| Acm_AxSetActualPosition | 設定指定軸的實際位置。   |
| Acm_AxGetActualPosition | 取得指定軸的目前實際位置。 |

| 參數            | 說明                   |
| ------------- | -------------------- |
| PAR_AxVelLow  | 設定/取得此軸的初始速度 (起始速度)。 |
| PAR_AxVelHigh | 設定/取得該軸的運轉速度。        |
| PAR_AxAcc     | 設定/取得該軸的加速度。         |
| PAR_AxDec     | 設定/取得該軸的減速度。         |
| PAR_AxJerk    | 設定速度曲線的類型：T/S 型曲線。   |

| 配置                   | 說明                        |
| -------------------- | ------------------------- |
| CFG_AxExtSelEnable   | 啟用/停用外部驅動。                |
| CFG_AxExtPulseNum    | 理論脈衝個數。                   |
| CFG_AxExtPulseInMode | 設定/取得外部驅動脈衝輸入模式。          |
| CFG_AxJogVelLow      | 設定/取得執行 Jog 運動時的初始速度。     |
| CFG_AxJogVLTime      | 設定/取得執行 Jog 運動時低速運轉保持的時間。 |
| CFG_AxJogVelHigh     | 設定/取得執行 Jog 運動的運行速度。      |
| CFG_AxJogAcc         | 設定/取得執行 Jog 運動時的加速度。      |
| CFG_AxJogDec         | 設定/取得執行 Jog 運動時的減速度。      |
| CFG_AxJogJerk        | 設定/取得執行 Jog 運動時的速度曲線。     |
| CFG_AxMaxVel         | 配置運動軸的最大速度。               |
| CFG_AxMaxAcc         | 配置運動軸的最大加速度。              |
| CFG_AxMaxDec         | 配置運動軸的最大減速。               |
| CFG_AxJogPAssign     | 設定/取得 DI 通道為 JOG+ 訊號源。    |
| CFG_AxJogNAssign     | 設定/取得 DI 頻道為 JOG- 訊號源。    |

| 特性                   | 說明             |
| -------------------- | -------------- |
| FT_AxExtDriveMap     | 取得軸支援的外部驅動特性。  |
| FT_AxJogMap          | 取得軸支援的 Jog 特性。 |
| FT_AxExtMasterSrcMap | 取得軸支援的外部驅動源。   |

- Jog 運動、手輪運動模式下，可單獨設定各軸的初速、運轉速度、加速度等速度參數，支援同一時間有兩個以上的軸在 JOG 狀態。
- Jog 運動模式下，透過設定屬性 `CFG_AxJogVelLow`、`CFG_AxJogVelHigh`、`CFG_AxJogAcc`、`CFG_AxJogDec` 的值，設定 Jog 運行時的初速、運行速度、加速度、減速度。
- 手輪運動模式下，透過設定參數 `PAR_AxVelLow`、`PAR_AxVelHigh`、`PAR_AxVelAcc`、`PAR_AxVelDec` 的值，設定手輪運動時的初始速度、運轉速度、加速度、減速度。
- 執行 Jog 運動時，可透過設定屬性 `CFG_AxJogVLTime` 的值，設定低速運轉保持的時間。即在該段時間內，軸將以初始速度運轉。
- 在停止 Jog 運作時，可繼續觸發 Jog 運動，當同向硬體觸發時，軸將直接加速到運行速度，當反向硬體觸發時，軸將減速到 0 後，再加速到運行速度。
- 流程圖在 5.2.8.3 節 (P.98)。

### 歸位運動

P.99

| 函數                      | 說明            |
| ----------------------- | ------------- |
| Acm_AxHome              | 指令軸開始回原點運動。   |
| Acm_AxMoveHome          | 讓指定單軸執行回原點運動。 |
| Acm_AxMoveGantryHome    | 龍門回原點運動。      |
| Acm_AxSetCmdPosition    | 設定指定軸的理論位置。   |
| Acm_AxSetActualPosition | 設定指定軸的實際位置。   |
| Acm_AxGetCmdPosition    | 取得指定軸的目前理論位置。 |
| Acm_AxGetActualPosition | 取得指定軸的目前實際位置。 |

| 參數                      | 說明                     |
| ----------------------- | ---------------------- |
| PAR_AxHomeCrossDistance | 設定原點跨越距離（單位：PPU）。      |
| PAR_AxHomeExSwitchMode  | 設定 Acm_AxHomeEx 的停止條件。 |
| PAR_AxVelLow            | 設定/取得該軸的低速度 (起始速度)。    |
| PAR_AxVelHigh           | 設定/取得該軸的運轉速度。          |
| PAR_AxAcc               | 設定/取得該軸的加速度。           |
| PAR_AxDec               | 設定/取得該軸的減速度。           |
| PAR_AxJerk              | 設定速度曲線的類型：T/S 型曲線。     |
| PAR_AxHomeVelLow        | 設定/取得回 Home 時的初速)。     |
| PAR_AxHomeVelHigh       | 設定/取得回 Home 時的運行速度。    |
| PAR_AxHomeAcc           | 設定/取得回 Home 時的加速度。     |
| PAR_AxHomeDec           | 設定/取得回 Home 時的減速度。     |
| PAR_AxHomeJerk          | 設定速度曲線的類型：T/S 型曲線。     |

| 配置                    | 說明                  |
| --------------------- | ------------------- |
| CFG_AxOrgLogic        | 設定/取得 ORG 訊號的邏輯準位。  |
| CFG_AxElLogic         | 設定/取得硬體限位訊號的邏輯準位。   |
| CFG_AxEzLogic         | 設定/取得 EZ 訊號的有效邏輯電平。 |
| CFG_AxHomeResetEnable | 啟用/停用邏輯計數器重設功能。     |
| CFG_AxOrgReact        | 設定回原點結束時的行為模式。      |
| CFG_AxMaxVel          | 配置運動軸的最大速度。         |
| CFG_AxMaxAcc          | 配置運動軸的最大加速度。        |
| CFG_AxMaxDec          | 配置運動軸的最大減速。         |

| 特性               | 說明             |
| ---------------- | -------------- |
| FT_AxHomeMap     | 取得軸所支援的原點相關特性。 |
| FT_AxHomeModeMap | 所支援的回原點方式。     |

- 原點復歸指軸回 `ORG`、`LMT+`、`LMT-`、`EZ` 的行為模式。
- 原點復歸模式下，`ORG`、`LMT+-`、`EZ` 訊號也稱為**原點訊號 (Home 訊號 )**，即電機回原點時控制它停止的訊號。
- 馬達停止模式分為 **減速停止** 和 **立即停止**，透過屬性 `CFG_AxOrgReact` 設定。
- 研華通用運動支援以兩種函數來執行原點復歸運動：`Acm_AxHome` 和 `Acm_AxMoveHome`。若**主從軸**有建立龍門關係，可透過 `Acm_AxMoveGantryHome` 使回原點運動中，同時監看主從軸的原點復歸訊號。
- 在呼叫 `Acm_AxHome` 執行回 Home 之前，透過 `PAR_AxVelLow`、`PAR_AxVelHigh`、`PAR_AxAcc`、`PAR_AxDec`、`PAR_AxJerk` 設定初速度，運轉速度、加速度、減速度、速度曲線類型。
- 在呼叫 `Acm_AxMoveHome` 和 `Acm_AxMoveGantryHome` 執行回 Home 之前，透過 `PAR_AxHomeVelLow`、`PAR_AxHomeVelHigh`、 `AR_AxHomeAcc`、`PAR_AxHomeDec`、`PAR_AxHomeJerk` 設定初速度，運轉速度、加速度、減速度、速度曲線類型。
- 原點復歸模式分為兩種類型，一種由 `研華定義`，另一種是 `CiA402` 定義的典型原點復歸運動，每種模式的運作方式將在後文中得以說明。

#### 研華定義原點復歸

要求軸開始典型原點復歸運動。研華運動控制卡共提供 16 種原點復歸模式。
註：16 種 Home 模式中的 `MODE3_Ref`~`MODE16_LmtSearchReFind_Ref`，某些階段會使用初速，所以透過 `PAR_AxVelLow` 設定的初速度須大於 0。
注！ 脈衝型闆卡若將 `CFG_AxHomeResetEnable` 設為 “`True`”，則指令位置和實際位置將在原點復歸運動後設定為 0。總線型闆卡除連接研華 AMAX 脈衝型模組外，不支援自動清零功能，使用者需呼叫 `Acm_AxSetCmdPosition` 設定為 0。

*說明*：
下圖的 a、b、c、d 的意思如下：
- a：當梯形 PTP 運動遇到 `ORG`/`EL` 訊號時，速度會下降。
- b：梯形 PTP 運動在結束前，由 `HomeCrossDistance` 決定運行距離。 `ORG`/`EL` 信號有效。
- c：梯形 PTP 以 `VelLow` 執行勻速運動，並將在遇到 `ORG`/`EL` 訊號時迅速停止。
- d：梯形 PTP 運動以 `VelLow` 運行，在運動結束前，由 `HomeCrossDistance` 決定運行距離。 `ORG`/`EL` 訊號有效。
- • ：此實心原點表示運動終點。

注！ 梯形 PTP 運動的特性：運動開始時，速度將以 `Acc` 從 `VelLow` 加速至 `VelHigh`（若距離夠長）；運動結束時，速度將以 `Dec` 從 `VelHigh` 減速至 `VelLow`。

16 種 Home 模式如下圖所示。
