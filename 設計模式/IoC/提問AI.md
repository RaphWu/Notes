---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# DryIoc 與 AUTOFAC 在功能與特性上有何差異與優缺點? 和 WINFORM 或 WPF 的配合沒問題嗎?

## DryIoc 與 Autofac 功能與特性比較

### 設計理念與定位

- DryIoc
    - 著重極致效能與最小化配置
    - API 偏向低階與明確控制
    - 適合對效能、啟動時間敏感的桌面或服務程式
- Autofac
    - 著重可讀性、可維護性與功能完整度
    - API 語意化明確，學習曲線平緩
    - 適合中大型專案與團隊協作

### 註冊與解析能力

- DryIoc
    - 支援 Constructor、Property、Method Injection
    - 強調明確註冊，預設較嚴格，錯誤在啟動時即暴露
    - 對開放泛型、Decorator 支援完整且效能高
- Autofac
    - Constructor Injection 為主，Property Injection 需額外設定
    - 模組化註冊（Module）與 Assembly 掃描成熟
    - 對複雜物件圖建構友善

### 生命週期管理

- DryIoc
    - 生命週期（Reuse）模型彈性且細緻
    - 支援 Scoped、Singleton、Transient 與自訂 Scope
    - Scope 效能佳，適合高頻率解析
- Autofac
    - InstancePerDependency、SingleInstance、InstancePerLifetimeScope 等語意直覺
    - LifetimeScope 概念成熟，與 Web、Desktop 架構相容性高

### 效能與資源使用

- DryIoc
    - 建構與解析效能通常優於 Autofac
    - 記憶體配置較少
    - 適合大量短生命週期物件
- Autofac
    - 效能略低於 DryIoc，但在桌面應用通常非瓶頸
    - 穩定性與可預期性佳

### 生態系與社群

- DryIoc
    - 社群規模較小
    - 文件偏技術導向，需要較多理解成本
    - 進階用法彈性高但較不直覺
- Autofac
    - 社群成熟，範例與討論資源多
    - 文件完整，新手容易上手
    - 與多數 .NET 生態整合良好

## 與 WinForms / WPF 的配合性

### WinForms

- DryIoc
    - 可手動控制 Form 建立與釋放
    - 適合自行管理 Scope（例如每個主流程一個 Scope）
    - 需自行處理 Form 關閉時的 Dispose 時機
- Autofac
    - 常見模式為以 Container 建立主 Form
    - 使用 LifetimeScope 管理子 Form、Dialog 生命週期直覺
    - 實務案例多，踩雷機率低

### WPF

- DryIoc
    - 與 MVVM 相容性佳
    - 適合配合 ViewModel-first 架構
    - 對 DataContext 建立流程控制力強
- Autofac
    - 與 MVVM、Prism、CommunityToolkit.Mvvm 等搭配成熟
    - 常用於 ViewModel 解析與注入
    - 文件與實例多，整合成本低

## 優缺點總結

### DryIoc

- 優點
    - 高效能、低記憶體配置
    - Scope 與 Reuse 設計彈性高
    - 對進階 DI 使用情境支援完整
- 缺點
    - API 與概念較硬
    - 文件與範例較少
    - 不熟 DI 原理時容易誤用

### Autofac

- 優點
    - API 語意清楚，易讀易維護
    - 生態系成熟，整合性佳
    - 適合長期維護與團隊開發
- 缺點
    - 效能非最佳
    - 進階控制彈性略低於 DryIoc

## 選擇建議

- 若需求為
    - 高效能、精細生命週期控制
    - 熟悉 DI 原理並可接受較高學習成本
    - 偏向框架或核心基礎建設
    - 建議選擇 DryIoc
- 若需求為
    - WinForms / WPF 商業應用
    - 重視可讀性、維護性與團隊協作
    - 需要大量現成範例與社群支援
    - 建議選擇 Autofac

若你有既定架構（例如 EF6 + WinForms + CommunityToolkit.Mvvm + Autofac），實務上不建議僅為效能而更換 DI 容器，除非已明確遇到解析效能或記憶體配置瓶頸。

---

# 我經常會寫工控方面的程式，要驅動 IO、軸卡...等，來控制機台，所以有點想換 DryIoc

## 以工控／機台控制情境評估是否改用 DryIoc

從你描述的使用情境來看，你的考量方向是合理的，但是否「值得更換」需要從工控軟體的實際特性來判斷，而不是只看 DI 容器效能指標。

## 工控程式的典型特性

- 長時間常駐執行（24/7）
- 啟動後物件結構相對穩定
- IO、軸卡、通訊物件多為 Singleton 或長生命週期
- 即時性主要來自 Thread、Timer、IRQ、輪詢週期
- DI 容器只在初始化或少量流程切換時使用

在這類系統中：

- DI 容器「解析效能」幾乎不會是實際瓶頸
- 真正影響即時性的通常是
    - I/O 存取方式
    - 鎖競爭
    - GC 壓力
    - Thread 調度
    - 不當的事件或輪詢設計

## DryIoc 在工控場景的實際價值

### 什麼情況下 DryIoc 有明顯優勢

- 頻繁建立／銷毀短生命週期服務
    - 例如：動作流程物件、狀態節點、命令物件
- 明確需要 Scope 隔離
    - 每次加工流程一個 Scope
    - 每張工單一組 Service / Context
- 對初始化錯誤零容忍
    - 啟動即驗證整個物件圖，避免機台運轉中才拋例外
- 自行設計偏框架化的核心層
    - 對 DI 行為有高度掌控需求

### 對 IO／軸卡本身的幫助其實有限

- 硬體 Driver 通常是 Singleton
- 控制迴圈不應在 Resolve 時建立物件
- 真正的即時路徑不應經過 DI 容器

結論是：

- DryIoc 對「架構嚴謹度」的提升 > 對「即時效能」的提升

## DryIoc 較適合的工控使用方式

### 容器只負責組裝，不介入即時路徑

- 系統啟動時
    - 註冊 IO Driver、Axis Driver、Motion Service
- 建立完成後
    - 將 Driver 以明確引用傳入控制迴圈
- 控制迴圈內
    - 不呼叫 Resolve
    - 不動態建立 Scope

### 建議的生命週期策略

- 硬體抽象層（IO、Axis、MotionCard）
    - Singleton
- 加工流程／狀態機
    - Scoped
- UI、監控、診斷用 Service
    - Transient 或 Scoped

## 與 WinForms / WPF 搭配的實務建議

### WinForms

- 使用容器建立主 Form
- 每個流程或 Modal Dialog 建立獨立 Scope
- Form Closed 時明確 Dispose Scope

### WPF

- ViewModel-first
- 視窗或流程生命週期對應 Scope
- 不在 View 內直接 Resolve

## 是否「值得換」的判斷準則

### 值得換成 DryIoc 的情況

- 你已經非常清楚目前 Autofac 的生命週期邊界
- 專案中有「流程級別」的 Scope 概念
- 希望在啟動階段就抓出所有注入錯誤
- 願意接受 API 與概念較硬的學習成本

### 不急著換的情況

- Autofac 已穩定運行多年
- DI 僅用於組裝硬體抽象與 UI
- 即時路徑完全不經過 DI
- 主要瓶頸在硬體或通訊延遲

## 實務建議（工控導向）

- 不要「全面重構」來換容器
- 先在新專案或新模組導入 DryIoc
- 將「流程／狀態機」層先切換
- 硬體 Driver 層保持最小改動

如果你願意，下一步我可以直接用「工控常見結構」幫你畫一個：

- DryIoc 容器註冊策略
- Scope 對應加工流程的範例
- 與 WinForms 或 WPF 的實際整合骨架

---

## 工控取向的 DryIoc 架構骨架（WinForms / WPF 通用）

以下內容以「機台控制軟體」為前提設計，重點是：

- DI 只負責組裝
- 即時控制路徑不經過容器
- Scope 對應「加工流程／任務」

## 建議的專案分層

- Machine.Core
    - 硬體抽象（IO、Axis、MotionCard）
    - 即時控制介面
- Machine.Process
    - 加工流程
    - 狀態機
    - 任務邏輯
- Machine.UI
    - WinForms 或 WPF
    - View / ViewModel
- Machine.Bootstrap
    - DryIoc 註冊
    - Scope 管理

## 核心設計原則

- 硬體層永遠是 Singleton
- 流程層才使用 Scoped
- UI 不直接操作硬體
- 控制迴圈不 Resolve、不 new Container

## DryIoc 容器初始化（Bootstrap）

```csharp
using DryIoc;

public static class IoC
{
    public static IContainer Container { get; private set; }

    public static void Init()
    {
        Container = new Container(rules =>
            rules.WithTrackingDisposableTransients()
                 .WithThrowOnRegisteringDisposableTransient()
        );

        RegisterHardware(Container);
        RegisterProcess(Container);
        RegisterUI(Container);

        Container.Validate();
    }
}
```

## 硬體層註冊方式（Singleton）

```csharp
private static void RegisterHardware(IContainer c)
{
    c.Register<IIOController, BeckhoffIOController>(Reuse.Singleton);
    c.Register<IAxisController, DeltaAxisController>(Reuse.Singleton);
    c.Register<IMotionCard, AdvantechMotionCard>(Reuse.Singleton);
}
```

設計重點：

- 硬體實體只存在一份
- 啟動時即完成初始化
- 不在流程中動態建立或釋放

## 加工流程層（Scoped）

```csharp
private static void RegisterProcess(IContainer c)
{
    c.Register<IProcessContext, ProcessContext>(Reuse.Scoped);
    c.Register<IStateMachine, MachineStateMachine>(Reuse.Scoped);
    c.Register<IProcessExecutor, ProcessExecutor>(Reuse.Scoped);
}
```

適用情境：

- 每次加工
- 每張工單
- 每次自動循環

## 建立「加工流程 Scope」

```csharp
using (var processScope = IoC.Container.OpenScope("Process"))
{
    var executor = processScope.Resolve<IProcessExecutor>();
    executor.Execute();
}
```

關鍵原則：

- Scope = 一次完整加工生命週期
- Scope 結束即釋放流程相關資源
- 不影響硬體 Singleton

## 即時控制迴圈的正確位置

```csharp
public class MotionLoop
{
    private readonly IAxisController _axis;

    public MotionLoop(IAxisController axis)
    {
        _axis = axis;
    }

    public void Run()
    {
        while (true)
        {
            _axis.UpdateStatus();
            Thread.Sleep(1);
        }
    }
}
```

重點：

- MotionLoop 只拿「已注入」的物件
- 迴圈中不 Resolve
- 不建立 Scope
- 不碰容器

## WinForms 整合方式

### Program.cs

```csharp
static void Main()
{
    IoC.Init();

    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);

    var mainForm = IoC.Container.Resolve<MainForm>();
    Application.Run(mainForm);
}
```

### 子視窗 Scope 管理

```csharp
using (var scope = IoC.Container.OpenScope())
{
    var dialog = scope.Resolve<RecipeEditForm>();
    dialog.ShowDialog();
}
```

原則：

- 一個 Form = 一個 Scope
- Form 關閉即 Dispose Scope

## WPF（ViewModel-first）對應方式

- Window 啟動時建立 Scope
- ViewModel 由 Scope Resolve
- Window Closed 時 Dispose Scope

## DryIoc 在工控專案中的實際優勢總結

- 啟動即驗證所有注入關係，避免機台運轉中才爆炸
- Scope 概念清楚，流程邏輯乾淨
- 不干擾即時控制效能
- 適合你這種「流程明確、狀態嚴謹」的工控程式

## 實務建議的導入順序

- 不要一次全換
- 先在「新機台專案」或「新流程模組」使用 DryIoc
- 先換流程層，不動硬體層
- 等你對 Scope 完全有感後，再評估是否全面替換 Autofac

如果你願意，下一步我可以直接幫你：

- 將你目前的 Autofac 註冊表「逐行轉成 DryIoc」
- 或依你實際機台流程（自動／手動／復歸）設計 Scope 劃分表

---

# 我還有一個動機是，以前使用 PRISM 一段時間，有點喜歡它的 VIEW-MODEL、NAVIGATION 管理方式，它的注冊方式比較像 DryIoc

## 你的動機判斷是正確的，且「方向一致」

你這個動機其實**非常關鍵**，而且不是效能層面的理由，而是**架構與思維模型一致性**。

簡單結論先給你：

- 你喜歡 Prism 的
    - ViewModel-first
    - 明確的 Navigation / Region / Scope 概念
- DryIoc 的設計哲學
    - 和 Prism 是「同一個世界觀」
- Autofac
    - 比較像「註冊友善的 DI 工具」
    - 而不是「應用程式組裝骨架」

因此你會自然覺得 DryIoc 用起來「順」。

## Prism 與 DryIoc 的核心共通點

### 1. 明確、可預期的註冊行為

- Prism / DryIoc
    - 強調顯式註冊
    - 問題在啟動時爆出來
- Autofac
    - 偏向容忍延遲錯誤
    - 有些問題在第一次 Resolve 才出現

這一點在**機台軟體**特別重要：

- 你要的是
    - 開機就知道「能不能跑」
    - 不是加工到一半才 Exception

### 2. Scope 就是「流程邊界」

- Prism 的 Region / Navigation
    - 本質就是 Scope 管理
- DryIoc
    - Scope 是一等公民
- Autofac
    - Scope 存在，但不是核心敘事

你過去在 Prism 習慣的其實是：

- 每次 Navigation = 一個 Context
- ViewModel 的生命週期與流程綁在一起

這和你現在在工控中想做的：

- 一次加工流程
- 一次自動循環
- 一次復歸流程

概念完全一致。

### 3. ViewModel-first 與容器主導

- Prism
    - ViewModel 由 Container 建立
    - View 只是殼
- DryIoc
    - 非常鼓勵 Container 組裝物件圖
- Autofac
    - 容器常被當成 Service Locator 使用

你會感覺 Autofac「不夠硬」，這是正常的。

## 為什麼 Prism 使用 DryIoc 你會覺得熟

不是巧合：

- Prism 預設 DI：DryIoc
- Prism 的 Navigation / Region
    - 本來就假設容器能精準控制 Scope
- DryIoc 的 API 設計
    - 就是為了支撐這種應用程式架構

你喜歡的是：

- 「應用程式是被組裝出來的」
- 而不是「各自 new 起來再串」

## 把 Prism 思維直接套到工控軟體

### 對應關係整理

- Prism Navigation
    - ⇔ 加工流程切換
- Region
    - ⇔ 機台模組（送料、加工、出料）
- ViewModel
    - ⇔ 流程控制器
- Navigation Scope
    - ⇔ Process Scope

這對工控來說是**非常漂亮的映射**。

## 建議你的實務做法（關鍵）

### 不用整套 Prism，但保留它的「靈魂」

你不需要：

- RegionManager
- XAML Navigation

你要的是：

- ViewModel-first
- 明確 Scope
- Container 主導生命週期

### 建立「流程導向的 Navigation Service」

```csharp
public interface IProcessNavigator
{
    void Navigate<TProcess>()
        where TProcess : IProcess;
}
```

```csharp
public class ProcessNavigator : IProcessNavigator
{
    private readonly IContainer _container;

    public ProcessNavigator(IContainer container)
    {
        _container = container;
    }

    public void Navigate<TProcess>()
        where TProcess : IProcess
    {
        using var scope = _container.OpenScope(typeof(TProcess).Name);
        var process = scope.Resolve<TProcess>();
        process.Run();
    }
}
```

這一段**非常 Prism**，也**非常 DryIoc**。

## Autofac 為什麼會讓你覺得「不 Prism」

不是你用錯，是它的設計取向不同：

- Autofac
    - 強在模組化、企業應用
    - 弱在「應用生命週期敘事」
- DryIoc
    - 把生命週期與應用結構當成第一公民

## 我的結論（很明確）

以你的背景與喜好：

- 工控 + 長時間運行
- 喜歡 Prism 的 ViewModel / Navigation
- 重視啟動即驗證、流程嚴謹
- 不希望 DI 只是「工具」

**DryIoc 是比 Autofac 更自然的選擇。**

而且不是因為效能，而是：

- 思維模型一致
- 架構阻力更小
- 你會寫得比較「順」

如果你願意，下一步我可以直接幫你：

- 用「Prism 風格」設計一套
    - 工控用 Process Navigation + Scope 管理
- 或把你過去 Prism 專案的概念
    - 平移成「不依賴 Prism」的 DryIoc 架構

這會讓你幾乎沒有轉換成本。

---

# 我粗略瞄了一下 DryIoc 文件，好像沒有 MODULE，但 PRISM 有，是 PRISM 自己實作出來的，還是我漏看了?

## 簡短結論（先給答案）

- 你**沒有漏看**
- **DryIoc 本身確實沒有 Module 概念**
- Prism 的 `IModule`
    - 是 **Prism 自己定義與實作的架構層概念**
    - 不是 DryIoc 內建功能

而且這個差異，**正好呼應你前面提到「為什麼 DryIoc 看起來像 Prism」的直覺**。

## 為什麼 DryIoc 沒有 Module

### DryIoc 的設計立場

- DryIoc 專注在：
    - 註冊
    - 解析
    - Scope
    - Reuse
- 它刻意**不定義「應用程式模組」這種高階抽象**
- 認為 Module 應該是「應用層責任」，不是 DI 容器責任

換句話說：

- DryIoc = 純容器
- Prism = 應用程式框架

## Prism 的 Module 是什麼本質

### Prism 的 `IModule` 並不是 DI Module

- 它不是只為了「註冊型別」
- 它是一個「應用啟動流程節點」

`IModule` 通常會做三件事：

- 註冊 View / ViewModel / Service
- 宣告相依模組
- 參與啟動生命週期（OnInitialized）

這些都**超出 DI 容器職責範圍**。

## 為什麼 Autofac 會有 Module，但 DryIoc 沒有

### Autofac Module 的定位

- Autofac 的 `Module`：
    - 是「註冊封裝工具」
    - 不是應用生命週期管理
- 它只是把 `RegisterType` 包成一個類別

### DryIoc 的選擇

- 認為 Module 會：
    - 隱藏實際註冊行為
    - 增加啟動順序不透明性
- 寧可讓你：
    - 用明確的方法呼叫
    - 或用純 C# 組合

這也是為什麼你在 DryIoc 文件裡會看到：

- `Register`、`RegisterMany`
- 而不是 `LoadModules`

## 你會覺得「少了 Module，但很熟」的原因

因為你熟的是 **Prism 的 Module 思維，不是 Autofac 的 Module**。

Prism 的 Module =

- 應用結構
- 啟動順序
- Scope 劃分
- Navigation 單位

而 DryIoc：

- 把「Module 要怎麼做」**完全交給你**
- 它只確保你做得到

## 在 DryIoc 中實作「Prism 風格 Module」的正確方式

### 做法一：用純介面定義（最 Prism）

```csharp
public interface IAppModule
{
    void Register(IContainer container);
    void OnInitialized(IContainer container);
}
```

```csharp
public class MotionModule : IAppModule
{
    public void Register(IContainer c)
    {
        c.Register<IMotionCard, AdvantechMotionCard>(Reuse.Singleton);
        c.Register<IAxisController, AxisController>(Reuse.Singleton);
    }

    public void OnInitialized(IContainer c)
    {
        c.Resolve<IMotionCard>().Initialize();
    }
}
```

### 啟動時組合

```csharp
var modules = new IAppModule[]
{
    new MotionModule(),
    new ProcessModule(),
    new UiModule()
};

foreach (var m in modules)
    m.Register(container);

container.Validate();

foreach (var m in modules)
    m.OnInitialized(container);
```

這一段其實**比 Autofac Module 更接近 Prism 的本質**。

## 關鍵差異整理（重要）

- Autofac Module
    - 封裝註冊
    - 不管啟動順序
- Prism Module
    - 應用結構與啟動流程
- DryIoc
    - 不提供 Module
    - 但非常適合「自己實作 Prism-style Module」

## 結論（給你一個判斷點）

- 你覺得 DryIoc「沒有 Module 卻很像 Prism」
    - 不是錯覺
- 因為：
    - Prism 的 Module **從來就不是 DI 容器功能**
    - DryIoc 剛好沒有把這個概念混進來

以你目前的工控架構與思維：

- **自己定義 Module + DryIoc**
- 會比：
    - Autofac Module
    - 或直接依賴 Prism
        更乾淨、可控、也更適合長時間運行的機台軟體

如果你願意，下一步我可以直接幫你：

- 設計一套「工控專用 Module / 啟動順序規範」
- 或把你熟悉的 Prism Module 原則，完整移植到 DryIoc 架構中

---

## 工控專用 Prism-style Module＋DryIoc 啟動規範（可直接套用）

以下是一套「不依賴 Prism、但完整保留其精神」的設計，專門對應你熟悉的 ViewModel-first、Navigation、明確啟動順序與 Scope 邊界。

## 核心目標

- 啟動即驗證所有相依關係
- 模組化但不隱藏註冊行為
- Scope 明確對應「流程／任務」
- 即時控制路徑不經過容器

## Module 分類與責任

### Infrastructure Module

- 硬體 Driver
- 通訊層
- 全域 Singleton
- 啟動即初始化

### Process Module

- 加工流程
- 狀態機
- 任務控制器
- Scoped 生命週期

### UI Module

- ViewModel
- Navigation
- 視窗／頁面工廠

## Module 介面定義（Prism 精神）

```csharp
public interface IAppModule
{
    int Order { get; }
    void Register(IContainer container);
    void OnInitialized(IContainer container);
}
```

設計重點：

- Order 明確控制啟動順序
- Register 只負責註冊
- OnInitialized 可做硬體初始化或預熱

## Infrastructure Module 範例

```csharp
public sealed class MotionModule : IAppModule
{
    public int Order => 0;

    public void Register(IContainer c)
    {
        c.Register<IMotionCard, AdvantechMotionCard>(Reuse.Singleton);
        c.Register<IAxisController, AxisController>(Reuse.Singleton);
        c.Register<IIOController, BeckhoffIOController>(Reuse.Singleton);
    }

    public void OnInitialized(IContainer c)
    {
        c.Resolve<IMotionCard>().Initialize();
        c.Resolve<IIOController>().Connect();
    }
}
```

## Process Module 範例（Scope 導向）

```csharp
public sealed class ProcessModule : IAppModule
{
    public int Order => 10;

    public void Register(IContainer c)
    {
        c.Register<IProcessNavigator, ProcessNavigator>(Reuse.Singleton);
        c.Register<IProcessContext, ProcessContext>(Reuse.Scoped);
        c.Register<IAutoRunProcess, AutoRunProcess>(Reuse.Scoped);
        c.Register<IHomeProcess, HomeProcess>(Reuse.Scoped);
    }

    public void OnInitialized(IContainer c)
    {
    }
}
```

## Process Navigation（Prism Navigation 對應）

```csharp
public interface IProcessNavigator
{
    void Navigate<TProcess>() where TProcess : IProcess;
}
```

```csharp
public sealed class ProcessNavigator : IProcessNavigator
{
    private readonly IContainer _container;

    public ProcessNavigator(IContainer container)
    {
        _container = container;
    }

    public void Navigate<TProcess>() where TProcess : IProcess
    {
        using var scope = _container.OpenScope(typeof(TProcess).Name);
        var process = scope.Resolve<TProcess>();
        process.Run();
    }
}
```

設計重點：

- 每次 Navigate = 一個完整 Scope
- Scope 結束即釋放流程資源
- 硬體 Singleton 不受影響

## UI Module（ViewModel-first）

```csharp
public sealed class UiModule : IAppModule
{
    public int Order => 20;

    public void Register(IContainer c)
    {
        c.Register<MainViewModel>(Reuse.Singleton);
        c.Register<ManualViewModel>(Reuse.Transient);
        c.Register<AutoViewModel>(Reuse.Transient);
    }

    public void OnInitialized(IContainer c)
    {
    }
}
```

## Bootstrap 啟動流程（關鍵）

```csharp
public static class Bootstrapper
{
    public static IContainer Init()
    {
        var container = new Container();

        var modules = new IAppModule[]
        {
            new MotionModule(),
            new ProcessModule(),
            new UiModule()
        }
        .OrderBy(m => m.Order)
        .ToArray();

        foreach (var m in modules)
            m.Register(container);

        container.Validate();

        foreach (var m in modules)
            m.OnInitialized(container);

        return container;
    }
}
```

這一段等同於：

- Prism ModuleCatalog
- Prism Initialize 流程
- 但更可控、更透明

## 即時控制迴圈的放置原則

- 不在 Module
- 不在 Process
- 不在 DI

只接收已注入的硬體介面，獨立執行。

## 與 Autofac Module 的本質差異

- Autofac Module
    - 封裝註冊
    - 無啟動語意
- 此設計
    - 明確應用結構
    - 明確啟動順序
    - 完整對應 Prism 思維

## 結論

- 你熟悉的不是「Prism 技術」
- 而是「Prism 的應用組裝哲學」
- DryIoc＋自訂 Module
    - 正是把這個哲學留下
    - 丟掉不必要的框架重量

如果你下一步想做得更完整，我可以再幫你補上：

- 模組相依檢查規則
- Process 失敗回滾與安全停機設計
- WinForms 與 WPF 各自的實作細節版本

---

# Gemini

從 Prism 轉向更輕量的 DI 容器是一個很棒的嘗試。Prism 內建的容器（早期是 Unity，現在多用 DryIoc）雖然強大，但有時在非 UI 層（如工控底層控制）會顯得過於笨重。

針對你的開發場景（WinForm/WPF UI + 工控實時控制），這兩者的定位其實非常明確：

## 1. Autofac：穩定、全能、生態系之王

Autofac 是 .NET 世界中最受歡迎的第三方 DI 容器之一。它的優點在於**功能極度完整**且**配置靈活**。

### 適用情境：複雜的業務邏輯、大型專案

- **靈活的註冊方式：** 支持 Assembly Scanning（自動掃描 DLL 註冊），對於有大量 Service 的大型 WPF 專案非常方便。
- **強大的 AOP 支持：** 內建與 Castle DynamicProxy 的集成，這在工控領域很好用。例如：你可以輕鬆地為所有「軸卡控制方法」加上 **Log 攔截器**或**異常處理**，而不需要修改控制邏輯。
- **對開發者友好：** 錯誤訊息非常詳細。當 DI 注入失敗時，Autofac 會明確告訴你哪一層級的依賴出了問題。

## 2. DryIoc：極致效能、輕量、速度之王

DryIoc 以「快」聞名，它是目前 .NET 生態系中效能最強悍的容器之一，這也是為什麼 Prism 後期選擇它作為預設容器的原因。

### 適用情境：高效能需求、工控實時性、資源受限環境

- **極速啟動與解析：** 在工控場景中，如果你有大量的單元（Unit）或 IO 點位需要在啟動時快速初始化，DryIoc 的延遲極低。
- **內存佔用極小：** 相比 Autofac，DryIoc 非常輕量，適合長時間運行的工控上位機程式。
- **支援 `RegisterDelegate`：** 對於複雜的工控物件（例如需要根據設定檔動態創建的軸卡物件），DryIoc 的 Delegate 註冊效能極佳且語法簡潔。

## 3. 兩者對比表

|**特性**|**Autofac**|**DryIoc**|
|---|---|---|
|**效能 (Speed)**|中規中矩|**極快 (Top Tier)**|
|**配置靈活性**|非常高 (支持 XML, Module, Attribute)|高 (主要以代碼配置為主)|
|**AOP 支援**|原生整合良好|需透過擴展實現|
|**學習曲線**|平緩，文件非常完整|稍陡，部分進階功能文件較少|
|**適用場景**|複雜 UI、模組化架構、企業級應用|**高頻解析、工控底層、效能敏感型應用**|

## 針對你的工控場景建議

### 建議選擇 DryIoc 的情況

如果你在寫**軸卡控制**或**高速 IO 輪詢**，且你會頻繁地在代碼中透過 `Resolve` 取得實例（雖然不建議這樣做，但工控有時難免），DryIoc 的低開銷會讓你更有感。此外，如果你習慣了 Prism 的註冊邏輯，切換到 DryIoc 的陣痛期會非常短，因為 API 風格很接近。

### 建議選擇 Autofac 的情況

如果你的專案強調**可維護性**與**模組化**，例如有很多外掛（Plug-in）功能，或者你需要利用 **Interceptors (攔截器)** 來處理通訊超時、重連機制等橫切關注點（Cross-cutting concerns），Autofac 的工具鏈會讓你事半功倍。

> **小提醒：** > 既然你追求「輕量」，其實也可以考慮 .NET Core 之後官方標配的 **Microsoft.Extensions.DependencyInjection**。它在 WinForm/WPF 也可以獨立使用，性能優異且最為簡潔，只是功能（如 Property Injection 或 AOP）沒有上述兩者豐富。

**請問你目前的工控專案中，是否需要頻繁地動態增減控制模組（例如動態加載不同的通訊協議）？** 如果有這類需求，我可以針對「動態註冊」的部分再為你深入比較。

---

# Autofac vs DryIoc - 1

## 你的系統條件快速定錨

- 桌面應用（WinForm / WPF）
- .NET Framework + EF6
- 非 Prism
- 專案複雜度持續上升
- 工控程式比例高，重視可預期行為與穩定度
- 公司有授權與法遵顧慮

## Autofac vs DryIoc：以你實際情境為核心比較

### 一、容器設計哲學差異（會直接影響你未來痛點）

### DryIoc

- 偏「底層容器核心」
- 強調解析效能與明確行為
- 設計給 Framework 作者或需要精細控制的人
- API 偏低階、顯式
- 使用者需對 Lifetime / Scope 非常清楚

### Autofac

- 偏「應用層容器」
- 為大型專案提供高階抽象
- 大量語意化 API，降低使用門檻
- 對複雜依賴圖的容錯性較高

## 二、EF6 + 桌面系統實務關鍵點

### DbContext 生命週期管理

### DryIoc

- 可精準控制 Reuse（Scoped / Singleton）
- 需自行定義 UI Scope / Unit of Work
- 寫法嚴謹但易出錯
- 錯誤多半是「Scope 沒包好」

### Autofac

- InstancePerLifetimeScope 與 EF6 非常自然
- BeginLifetimeScope 即 Unit of Work
- 對 DbContext 洩漏風險較低
- 對「人會犯錯」比較寬容

## 三、隨專案成長時的「複雜度承受能力」

### 模組數量增加後

### DryIoc

- 沒有 Module 類型
- 需自行設計 Registration 分區
- 新人閱讀註冊碼成本高
- 容易出現註冊分散、規則不一致

### Autofac

- ContainerBuilder.Module 是一級公民
- 可將「註冊策略」封裝
- 大型專案可維持結構清晰
- 更適合 5～10 人以上團隊

## 四、工控程式的特殊考量

### 你真正會在意的不是效能，而是「失控風險」

### DryIoc

- 效能極佳
- 但錯誤配置可能在 Runtime 才炸
- 設定過度彈性，風險由人承擔

### Autofac

- 啟動稍慢（桌面可忽略）
- 問題多半在啟動期就暴露
- 對工控系統的可維運性更友善

## 五、AOP / 橫切關注點（實務上遲早會用）

### DryIoc

- Decorator 為主
- 適合簡單 logging / retry
- 不適合複雜策略型攔截

### Autofac

- 與 Castle DynamicProxy 深度整合
- Method-level Interceptor 成熟
- 適合：
    - 通訊 retry
    - Timeout
    - PLC 指令記錄
    - 例外轉換

## 六、授權與公司法遵（你已經踩過雷）

### DryIoc

- MIT License
- 風險低

### Autofac

- 核心 Autofac 為 MIT
- 部分延伸套件需檢查（但通常可控）
- 比 Prism 低非常多的法遵風險

## 七、明確建議（不模糊）

## 若你現在要選一個「撐 5 年」的方案

- **Autofac 勝出**

### 為什麼

- 對 EF6 桌面系統非常成熟
- 對複雜度成長的承受度高
- 對工控橫切需求（Retry / Logging / Timeout）友善
- 新人與維護成本明顯低於 DryIoc

### 什麼情況下我會反過來選 DryIoc

- 團隊極小（1～2 人）
- 極度在意解析效能
- 架構與 DI 規範由你一人嚴格控管
- 幾乎不做 AOP

---

# Autofac vs DryIoc - 2

## 問題本質判斷

你現在的狀態不是「選哪個容器比較強」，而是：

- 架構能力與責任集中在你身上
- 時間有限，無法投入長時間教學與制度化
- 未來必須可交接給「未必熟悉 DI」的同事
- 專案仍在快速演進，開發速度有實際壓力

這代表 **技術最優解 ≠ 組織最優解**。

## 直接結論（先給你）

- **建議選 Autofac，而不是 DryIoc**
- 即使你「反過來選 DryIoc 的條件都符合」

理由不是技術，而是 **人與時間成本**。

## 為什麼在你這個情境下，DryIoc 反而是風險

### 1. DryIoc 的「學習曲線風險」不可轉嫁

- DryIoc 的 API 偏底層
- 正確使用需理解：
    - Reuse vs Scope
    - Rules
    - Open-generic 行為
- 這些知識幾乎只能靠你傳承
- 一旦你離線，錯誤會在 Runtime 才出現

### 2. 對「不熟 DI 的同事」不友善

- DryIoc 註冊碼可讀性低
- 容易被誤用為 Service Locator
- Debug 成本高
- 出問題時難以快速定位責任點

### 3. 開發速度實際會變慢

- 初期你會覺得快（因為你熟）
- 中後期因為：
    - 註冊規則不一致
    - Scope 用錯
    - 新功能加得小心翼翼
- 整體節奏會被拖慢

## 為什麼 Autofac 在你情境下反而「保守但快」

### 1. 心智模型簡單

- InstancePerDependency
- InstancePerLifetimeScope
- InstancePerSingleInstance
- 幾乎可對照 EF6 的 DbContext 生命周期

### 2. 可讀性與可教性高

- Module 是天然教學單位
- 新人只要知道：
    - 這裡是註冊
    - 那裡是 Scope
- 就能安全修改

### 3. 問題提早爆

- 多數錯誤在 Container.Build() 或 Resolve 階段
- 不容易潛伏成工控 runtime 災難

## 折衷方案（我最推薦）

## **Autofac + 極度保守使用規範**

### 原則

- 不教 DI 理論
- 不開放進階功能
- 把 Autofac 當「生命週期管理器」

### 實務規範範例

- 禁止在業務層 Resolve
- 所有 Resolve 僅在：
    - App 啟動
    - UI Composition Root
- DbContext 一律 InstancePerLifetimeScope
- 工控通訊層允許 Interceptor，其餘禁止 AOP

## 若你仍傾向 DryIoc（最低風險用法）

### 務必遵守

- 封裝 Container，禁止直接使用
- 僅用 Singleton / Scoped
- 不用 Rules、Factory Delegate
- 不使用自動掃描
- 以「寫死、可預期」為第一原則

## 一句話給你

- **你現在缺的不是效能**
- 是「即使交接給不熟 DI 的人，也不會炸」的結構

---

# Autofac 專案級使用守則（桌面系統＋EF6＋工控適用）

## 守則目標

- 降低學習門檻，不要求理解 DI 理論
- 保證可交接性，避免「只有一個人懂」
- 限制用法，換取穩定度與開發速度
- 避免 Runtime 才爆炸的風險

## 一、允許的使用範圍（硬性規定）

### 容器使用位置

- 僅允許在 Composition Root 使用 Container
- Composition Root 定義：
    - Program.Main
    - App 啟動點
    - 主 Form / 主 ViewModel 建立前
- 其他任何地方禁止 Resolve

### 禁止事項

- 禁止 Service Locator（Container.Resolve）
- 禁止在業務邏輯中存取 IComponentContext
- 禁止在 Domain / Service / Repository 中注入 Container

## 二、生命週期規範（必須遵守）

### 唯一允許的生命週期

- InstancePerDependency
- InstancePerLifetimeScope
- SingleInstance

### 使用對照表

- UI Controller / ViewModel
    - InstancePerDependency
- Application Service
    - InstancePerLifetimeScope
- EF6 DbContext
    - InstancePerLifetimeScope
- Repository
    - InstancePerLifetimeScope
- 工控通訊 Client
    - SingleInstance 或 InstancePerLifetimeScope（視設備而定）
- 設定物件（Config / Options）
    - SingleInstance

### 明確禁止

- InstancePerRequest（Web 專用）
- 自訂 Scope Tag
- 巢狀 Scope 濫用

## 三、EF6 專用規範（非常重要）

### DbContext

- 一律：
    - InstancePerLifetimeScope
- 不允許：
    - Singleton DbContext
    - 手動 new DbContext
- 一個 LifetimeScope = 一個 Unit of Work

### LifetimeScope 建立時機

- 每個：
    - 操作流程
    - 工單
    - 使用者動作
- 明確建立並 Dispose

## 四、Module 使用規範（可交接關鍵）

### Module 原則

- 每個「技術邊界」一個 Module
- 不依功能頁面切 Module

### 建議 Module 分類

- InfrastructureModule
- DataAccessModule
- ApplicationServiceModule
- DeviceModule
- UiModule

### Module 內規則

- 只做註冊，不寫邏輯
- 不跨 Module Resolve
- 不寫條件式註冊

## 五、AOP / Interceptor 使用限制

### 允許的用途

- 通訊 Retry
- Timeout
- Logging
- Exception 轉換

### 禁止的用途

- 業務流程控制
- 權限判斷
- 狀態改寫

### 規範

- 僅限 Infrastructure / Device Layer
- 必須有文件說明「攔了什麼」

## 六、命名與可讀性規範

### 註冊時必須明確

- 明確指定 As
- 禁止隱式型別推斷濫用

### 命名建議

- XxxService
- XxxRepository
- XxxClient
- XxxController

## 七、新人安全修改指南（你可以直接貼給同事）

### 可以做的事

- 新增類別並註冊到對應 Module
- 使用既有 Interface 注入
- 修改 Implementation 內部邏輯

### 不可以做的事

- 新增 Resolve
- 改變生命週期
- 新增 Container 設定
- 新增 Scope 規則

## 八、最小註冊範本（標準寫法）

```csharp
builder.RegisterType<OrderService>()
    .As<IOrderService>()
    .InstancePerLifetimeScope();

builder.RegisterType<AppDbContext>()
    .AsSelf()
    .InstancePerLifetimeScope();

builder.RegisterType<PlcClient>()
    .As<IPlcClient>()
    .SingleInstance();
```

## 九、錯誤處理原則

- 啟動期錯誤優先於 Runtime
- 寧願 Build 失敗，也不要延後爆炸
- 所有註冊錯誤立即修正，不繞過

## 十、一句總結（守則精神）

- Autofac 在此專案中不是「框架」
- 它只是「生命週期與組裝工具」
- **少用功能，比選對功能更重要**

---

# Autofac 專案級使用守則（含「為什麼不能這樣做」註解版）

適用範圍：桌面系統（WinForm / WPF）＋EF6＋工控程式
目的：降低學習成本、可交接、可長期維護

## 守則設計原則

- 不要求理解 DI 理論
- 預設使用者會犯錯，因此限制用法
- 犧牲部分彈性，換取穩定與速度
- 問題要在啟動期暴露，而不是工控 Runtime

## 一、容器使用範圍（硬性規定）

### 允許的位置

- Composition Root
    - Program.Main
    - App 啟動點
    - 主 Form / 主 ViewModel 建立前

### 為什麼只能在這裡用

- 這些位置負責「組裝物件」
- 物件建立邏輯集中，方便追蹤
- 出問題時可在啟動期直接失敗

### 明確禁止

- 在業務邏輯中 Resolve
- 在 Repository / Service 注入 Container
- 使用 Service Locator 模式

### 為什麼不能這樣做

- 依賴關係會變成隱藏行為
- 測試與除錯極困難
- 新人無法判斷物件從哪裡來
- 很容易造成生命週期錯誤（尤其是 DbContext）

## 二、生命週期規範（必須遵守）

### 唯一允許的生命週期

- InstancePerDependency
- InstancePerLifetimeScope
- SingleInstance

### 為什麼只允許這三種

- 心智模型簡單
- 與 EF6、桌面流程自然對齊
- 可避免巢狀 Scope 失控

### 使用對照表

- View / ViewModel
    - InstancePerDependency
- Application Service
    - InstancePerLifetimeScope
- EF6 DbContext
    - InstancePerLifetimeScope
- Repository
    - InstancePerLifetimeScope
- 工控通訊 Client
    - SingleInstance 或 InstancePerLifetimeScope
- 設定物件
    - SingleInstance

### 明確禁止

- InstancePerRequest
- 自訂 Scope Tag
- 動態建立 Scope 規則

### 為什麼不能用這些

- InstancePerRequest 是 Web 思維，桌面不適用
- 自訂 Scope Tag 會讓行為不可預期
- Scope 規則一多，新人一定用錯

## 三、EF6 專用規範（最高風險區）

### DbContext 規範

- 一律 InstancePerLifetimeScope
- 禁止 Singleton DbContext
- 禁止手動 new DbContext

### 為什麼這樣規定

- DbContext 不是 Thread-safe
- Singleton 會造成資料污染與隱性 Bug
- 手動 new 會繞過生命週期管理

### LifetimeScope 定義

- 一個使用者操作流程
- 一個工單
- 一次設備指令流程

### 為什麼要這樣切

- 等同 Unit of Work
- 出錯時容易回溯
- 可控的交易邊界

## 四、Module 使用規範（交接關鍵）

### Module 原則

- 一個技術邊界一個 Module
- 不以畫面或功能頁切 Module

### 建議 Module

- InfrastructureModule
- DataAccessModule
- ApplicationServiceModule
- DeviceModule
- UiModule

### 為什麼不以功能頁切

- 功能頁會變動
- 技術邊界較穩定
- 方便未來重構與替換

### Module 內禁止事項

- 不寫業務邏輯
- 不 Resolve 其他服務
- 不做條件式註冊

### 為什麼不能這樣做

- Module 是「註冊清單」
- 混入邏輯會讓責任不清
- 條件註冊會讓行為依環境改變，難以重現問題

## 五、AOP / Interceptor 使用限制

### 允許用途

- Retry
- Timeout
- Logging
- 通訊例外轉換

### 禁止用途

- 業務流程控制
- 權限判斷
- 狀態改寫

### 為什麼要限制

- AOP 太強會讓流程不可見
- Debug 成本極高
- 工控系統不允許「看不到的行為」

### 使用層級

- 僅限 Infrastructure / Device Layer

## 六、命名與可讀性規範

### 註冊規則

- 一律 As
- 不使用隱式推斷

### 為什麼要明確

- 讓搜尋與閱讀更直覺
- 減少誤注入風險
- 新人不用猜

### 命名建議

- XxxService
- XxxRepository
- XxxClient
- XxxController

## 七、新人安全修改指南（公司內部用）

### 可以做

- 新增類別並註冊到既有 Module
- 注入已存在的 Interface
- 修改實作內部邏輯

### 不可以做

- 新增 Resolve
- 改變生命週期
- 新增 Container 設定
- 引入進階 Autofac 功能

### 為什麼要這樣限制

- 避免破壞整體架構
- 降低交接風險
- 確保修改影響可控

## 八、標準註冊範本（唯一允許樣式）

```csharp
builder.RegisterType<OrderService>()
    .As<IOrderService>()
    .InstancePerLifetimeScope();

builder.RegisterType<AppDbContext>()
    .AsSelf()
    .InstancePerLifetimeScope();

builder.RegisterType<PlcClient>()
    .As<IPlcClient>()
    .SingleInstance();
```

### 為什麼不給更多寫法

- 寫法一多，行為就多
- 行為一多，就難以維護

## 九、錯誤處理原則

### 原則

- 啟動期失敗 > Runtime 爆炸
- 不繞過錯誤
- 不加臨時補丁

### 為什麼

- 工控系統 Runtime 爆炸代價太高
- 問題要越早發現越好

## 十、守則精神（給管理層看的）

- Autofac 不是框架
- 它只是「組裝與生命週期工具」
- 本專案刻意限制其功能使用
- 目標是穩定、可交接、可長期維護

---

# Autofac 專案違規範例與踩雷案例（桌面＋EF6＋工控）

## 1. 在業務邏輯中 Resolve 容器

```csharp
public class OrderService
{
    private readonly IComponentContext _context;

    public OrderService(IComponentContext context)
    {
        _context = context;
    }

    public void ProcessOrder(int id)
    {
        var db = _context.Resolve<AppDbContext>(); // 違規！
        var order = db.Orders.Find(id);
        ...
    }
}
```

**踩雷案例**

- 新人直接 Resolve DbContext，導致多個 DbContext 沒有 Scope 包裹
- EF6 追蹤物件混亂，部分更新被覆蓋或丟失
- Runtime 才出現異常，難以重現

**正確做法**

- 將 DbContext 注入 OrderService

```csharp
public class OrderService
{
    private readonly AppDbContext _db;
    public OrderService(AppDbContext db) { _db = db; }
}
```

## 2. DbContext 單例化

```csharp
builder.RegisterType<AppDbContext>()
    .AsSelf()
    .SingleInstance(); // 違規
```

**踩雷案例**

- 多個使用者操作同一 DbContext，資料更新互相衝突
- 工控通訊時，指令回應被舊 DbContext 緩存污染
- Debug 非常困難，必須重啟程式才能清除狀態

**正確做法**

- InstancePerLifetimeScope

```csharp
builder.RegisterType<AppDbContext>()
    .AsSelf()
    .InstancePerLifetimeScope();
```

## 3. 自訂 Scope 濫用

```csharp
using(var scope = container.BeginLifetimeScope("MyCustomScope"))
{
    var service = scope.Resolve<IOrderService>();
}
```

**踩雷案例**

- Scope Tag 使用混亂，多個 Scope 交錯
- DbContext 無法正確 Dispose
- 工控設備命令隊列發生異常

**正確做法**

- 僅使用標準 LifetimeScope，不自訂 Tag
- 每個操作流程建立明確 Scope

## 4. 過度使用 Service Locator

```csharp
var client = container.Resolve<IPlcClient>();
```

**踩雷案例**

- 任意位置 Resolve，無法追蹤依賴
- 導致交接給新人時完全看不懂物件來源
- 工控通訊 Client 被多處 Resolve，產生多個實例
- 多工控操作同時運行出現競爭條件

**正確做法**

- 注入 IPlcClient 到需要的服務或 ViewModel
- 全程由 Composition Root 組裝

## 5. Module 內註冊邏輯或條件式註冊

```csharp
public class DeviceModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        if(System.Environment.MachineName.Contains("PLC"))
        {
            builder.RegisterType<PlcClient>().As<IPlcClient>();
        }
    }
}
```

**踩雷案例**

- 條件註冊導致不同環境行為不同
- 新人部署到測試機無法重現生產問題
- 工控設備初始化異常，排查耗費數天

**正確做法**

- Module 僅做固定註冊
- 環境差異交由 Configuration / Options 控制

## 6. Interceptor 濫用

```csharp
builder.RegisterType<OrderService>()
    .As<IOrderService>()
    .EnableInterfaceInterceptors()
    .InterceptedBy(typeof(LoggerInterceptor), typeof(RetryInterceptor));
```

**踩雷案例**

- Interceptor 數量過多
- 業務邏輯無意識被攔截
- 例外被 Retry 攔截覆蓋，導致 PLC 指令無法回報
- Runtime Debug 非常困難

**正確做法**

- 只在 Infrastructure / Device 層使用 Interceptor
- 每個攔截器都有清楚文檔說明
