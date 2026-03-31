# 目錄

[🔝](#目錄)

- [目錄](#目錄)
- [問題條件整理](#問題條件整理)
  - [單位統一（關鍵）](#單位統一關鍵)
  - [感測器轉換關係](#感測器轉換關係)
  - [ADC 基本關係](#adc-基本關係)
  - [正向推估（100 g·cm 對應 ADC）](#正向推估100-gcm-對應-adc)
  - [反推公式（ADC → 扭力）](#反推公式adc--扭力)
    - [Nm 表示](#nm-表示)
    - [g·cm 表示](#gcm-表示)
  - [快速公式（建議直接用）](#快速公式建議直接用)
  - [工程上重要結論](#工程上重要結論)
  - [建議](#建議)
- [整數優先、最後再除](#整數優先最後再除)
  - [原始關係](#原始關係)
  - [重排（分子分母顛倒）](#重排分子分母顛倒)
    - [Nm 表示（建議用）](#nm-表示建議用)
    - [g·cm 表示（精確版，避免浮點）](#gcm-表示精確版避免浮點)
  - [若要「全整數運算」（工控常用）](#若要全整數運算工控常用)
  - [建議實作（C# 範例）](#建議實作c-範例)
  - [補充（精度觀點）](#補充精度觀點)
- [扭力量測完整流程（工控等級）](#扭力量測完整流程工控等級)
  - [1. 原始模型](#1-原始模型)
  - [2. 校正模型（核心）](#2-校正模型核心)
    - [Gain 定義（Nm）](#gain-定義nm)
  - [3. 建議「整數化 Gain」](#3-建議整數化-gain)
  - [4. 直接輸出 g·cm（推薦）](#4-直接輸出-gcm推薦)
    - [Gain_gcm 推導](#gain_gcm-推導)
  - [5. Offset 取得方式（重要）](#5-offset-取得方式重要)
    - [方法1：靜態歸零](#方法1靜態歸零)
    - [方法2：開機校正（工控常用）](#方法2開機校正工控常用)
  - [6. 濾波（強烈建議）](#6-濾波強烈建議)
    - [移動平均（最簡單穩定）](#移動平均最簡單穩定)
    - [IIR（低延遲）](#iir低延遲)
  - [7. C# 實作（整數安全版）](#7-c-實作整數安全版)
  - [8. 進階校正（建議）](#8-進階校正建議)
  - [9. 工程結論（重點）](#9-工程結論重點)
  - [10. 若要再優化](#10-若要再優化)

---

# 問題條件整理

- HBK T22 扭力計
  - 量程：5 Nm ↔ 5 V（線性，1 Nm/V）
  - 目標扭力：約 100 g·cm
- USB-4704 ADC
  - 差動模式 14-bit（16384 levels）
- 目標：ADC → 電壓 → 扭力（Nm / g·cm）

## 單位統一（關鍵）

先把 g·cm 轉成 Nm：

- 1 g·cm = 9.80665 × 10⁻⁵ Nm
- 100 g·cm ≈ **0.00980665 Nm**

## 感測器轉換關係

- 5 Nm → 5 V
  → 1 Nm → 1 V
  → **V = Torque(Nm)**

## ADC 基本關係

假設 ADC 參考電壓 = 5 V：

- 1 LSB = 5 / 16384 ≈ **0.00030518 V**
- ADC值 = V / 5 × 16384

## 正向推估（100 g·cm 對應 ADC）

1. 扭力 → 電壓
   - 0.00980665 Nm → 0.00980665 V

2. 電壓 → ADC
   - ADC ≈ 0.00980665 / 5 × 16384 ≈ **32 counts**

👉 結論：
100 g·cm 約只佔 **ADC ≈ 32 / 16384 ≈ 0.2% FS**

## 反推公式（ADC → 扭力）

### Nm 表示

- Torque(Nm) = ADC × (5 / 16384)

### g·cm 表示

- Torque(g·cm) = ADC × (5 / 16384) × (1 / 9.80665×10⁻⁵)
- 簡化：
  - Torque(g·cm) ≈ **ADC × 3.113**

## 快速公式（建議直接用）

```
Torque_gcm = ADC * 3.113
Torque_Nm  = ADC * 0.00030518
```

## 工程上重要結論

- 100 g·cm → ADC ≈ 32 counts
- 有效解析度極低（僅約 5~6 bits 有效）
- 雜訊、offset、溫漂會嚴重影響

## 建議

- 增加前級放大（Instrumentation Amplifier）
  - 例如放大 ×50 → 32 counts → 1600 counts

- 或改用低量程扭力計（例如 0.5 Nm FS）
- 或提高 ADC 解析度（18~24 bit）

如果你有實際 ADC reference（不是5V）或感測器不是完全1:1，我可以幫你推一版「含 offset / gain calibration」的完整公式。

---

# 整數優先、最後再除

降低浮點誤差（避免先除再乘）

## 原始關係

- Torque(Nm) = ADC × (5 / 16384)
- Torque(g·cm) ≈ ADC × 3.113

## 重排（分子分母顛倒）

### Nm 表示（建議用）

```
Torque_Nm = (ADC * 5) / 16384
```

### g·cm 表示（精確版，避免浮點）

由：

- 1 Nm = 10197.2 g·cm

推得：

```
Torque_gcm = (ADC * 5 * 10197.2) / 16384
```

為避免小數，放大：

```
Torque_gcm = (ADC * 5 * 101972) / (16384 * 10)
```

或進一步整理（推薦）

```
Torque_gcm = (ADC * 509860) / 163840
```

## 若要「全整數運算」（工控常用）

可用 scale 技術（避免任何浮點）：

```
Torque_gcm_x100 = (ADC * 509860) / 1638.4   ← 不建議（仍有小數）
```

改為完全整數：

```
Torque_gcm_x10 = (ADC * 50986) / 16384
```

最後再 /10 得到實際值

## 建議實作（C# 範例）

```csharp
// 回傳 g·cm（保留1位小數）
int torque_x10 = (adc * 50986) / 16384;
double torque = torque_x10 / 10.0;
```

## 補充（精度觀點）

- 你目前系統瓶頸不是浮點誤差，而是：
  - ADC解析度不足（100 g·cm ≈ 32 counts）

- 即使用 double，誤差 << 1 count，可忽略

如果你後續要做：

- 校正（offset / gain）
- 溫漂補償
- 濾波（moving average / FIR）

我可以幫你整理一套「工控等級完整量測鏈公式」。

---

# 扭力量測完整流程（工控等級）

以下流程包含：Offset → Gain → 濾波 → 單位轉換
設計目標：**全整數優先 + 可校正 + 可擴充**

## 1. 原始模型

- 感測器：5 Nm ↔ 5 V（線性）
- ADC：14-bit（0 ~ 16383）
- 理想關係：
  - Torque(Nm) = (ADC × 5) / 16384

實務上需加入：

- Offset（零點漂移）
- Gain（比例誤差）

## 2. 校正模型（核心）

```text
Torque = (ADC_raw - ADC_offset) × Gain
```

### Gain 定義（Nm）

```text
Gain = FullScale_Torque / (ADC_FS - ADC_offset)
```

你的條件：

- FullScale_Torque = 5 Nm
- ADC_FS ≈ 16384

→

```text
Gain ≈ 5 / (16384 - ADC_offset)
```

## 3. 建議「整數化 Gain」

避免浮點：

```text
Gain_scaled = Gain × 10^6   （或 10^5 視精度需求）
```

→ 實際運算：

```text
Torque_Nm = (ADC_raw - ADC_offset) × Gain_scaled / 10^6
```

## 4. 直接輸出 g·cm（推薦）

避免中間 Nm：

```text
Torque_gcm = (ADC_raw - ADC_offset) × Gain_gcm
```

### Gain_gcm 推導

- 1 Nm = 10197.2 g·cm

→

```text
Gain_gcm = (5 × 10197.2) / (16384 - ADC_offset)
```

整數化：

```text
Gain_gcm_scaled = Gain_gcm × 10
```

→ 最終：

```text
Torque_gcm = (ADC_raw - ADC_offset) × Gain_gcm_scaled / 10
```

## 5. Offset 取得方式（重要）

### 方法1：靜態歸零

- 無負載時取平均 N 筆（建議 N=100~1000）

```text
ADC_offset = avg(ADC_raw)
```

### 方法2：開機校正（工控常用）

- 開機後延遲 → 自動取樣
- 避免人為忘記歸零

## 6. 濾波（強烈建議）

你的訊號只有 ~32 counts，非常敏感

### 移動平均（最簡單穩定）

```text
ADC_filtered = (Σ ADC[i]) / N
```

- 建議 N：
  - 靜態：32~128
  - 動態：8~16

### IIR（低延遲）

```text
y[n] = y[n-1] + α(x[n] - y[n-1])
```

- α 建議：0.05 ~ 0.2

## 7. C# 實作（整數安全版）

```csharp
// 參數（校正後）
int adcOffset = 120;              // 量測得到
int gain_gcm_x10 = 31;            // 由校正計算（範例值）

// 濾波後ADC
int adc = adcFiltered;

// 計算（避免負值）
int delta = adc - adcOffset;
if (delta < 0) delta = 0;

// g·cm（保留1位小數）
int torque_x10 = delta * gain_gcm_x10;

// 最終值
double torque = torque_x10 / 10.0;
```

## 8. 進階校正（建議）

使用「兩點校正」提高準確度：

1. 無負載 → ADC0
2. 已知標準扭力（例如 1 Nm）→ ADC1

```text
Gain = (Torque1 - Torque0) / (ADC1 - ADC0)
```

→ 比單點 FS 更準確（補償線性誤差）

## 9. 工程結論（重點）

- 目前系統解析度嚴重不足（100 g·cm ≈ 32 counts）
- 必做：
  - Offset 校正
  - 濾波

- 強烈建議：
  - 類比放大（×20~×50）
  - 或改低量程感測器

- 軟體最佳策略：
  - 全整數運算（scaled integer）
  - 最後一步才轉 double

## 10. 若要再優化

可再加入：

- 溫度補償（NTC / lookup table）
- 非線性校正（LUT / 分段線性）
- 峰值保持（Peak Hold）
- 旋入判定（扭力上升率）
