# 副理 / 課長

- 測高位置定位
- 花瓣辨識方法測試
- CSV檔分資料夾、標題、序號
- 品種顯示
- 高度校正
- 偏心補正
- Offset 取得
  - 靜態歸零
  - 開機校正
  - 濾波
- 有標準扭力計
  - 兩點校正
- 歸零塊

# R

- 狀態機
-

# 再優化

- 溫度補償（NTC / lookup table）
- 非線性校正（LUT / 分段線性）
- 峰值保持（Peak Hold）
- 旋入判定（扭力上升率）

# 完成

- 扭力 – 電壓轉換，小數位數
- 扭力監測畫面座標軸、高低峰值、平均值

---

# 建構式注入順序

```csharp
private readonly ILifetimeScope _scope;
private readonly INavigation _nav;
private readonly IProductManager _pm;
private readonly IProcessRegistry _registry;
private readonly IStatusBarService _sbService;

private readonly ILifetimeScope _scope;
private readonly INavigation _nav;
private readonly IStatusBarService _sbService;
private readonly IAcm _acm;
private readonly IAcmService _acmService;
private readonly IDispService _disp;
private readonly IDaqService _daq;
private readonly IPollingService _pollingService;
private readonly MotionData _motionData;
private readonly TorqueMeterData _daqData;
private readonly DisplacementData _dispData;
private readonly BindingData _bindingData;
```
