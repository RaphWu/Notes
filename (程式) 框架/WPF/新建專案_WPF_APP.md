---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 使用的語言及框架

- 業務處理程式語言：C#。
- 應用程式開發框架：最新版本的 .NET。
- 展示層開發框架：WPF。
- 模組化及 MVVM 處理：Prism 框架。
- IoC 容器：Unity (內建於 Prism 框架內)。
- 延伸模組：Prism Template Pack。

---

# 建立解決方案

1. 選擇 `建立新的專案`。
2. 選擇 `Prism Blank App (WPF)`。

   ![](./Images/NewApp_WPF/Prism%20Blank%20App.png)

3. 設定專案：(這裡以 `RaphaelApp` 為例)

   - 設定 `專案名稱`、`方案名稱` 及 `位置`(記得 `位置` 後多建一層分類用資料夾)。

   - `將解決方案與專案置於相同目錄中` 不打勾。

   ![](./Images/NewApp_WPF/專案名稱設定.png)

4. 執行 `建立`。`Container` 選擇 `Unity`。

   ![](./Images/NewApp_WPF/IoC選擇.png)

5. 新建專案的目錄結構如下：

   ![](./Images/NewApp_WPF/新建專案_方案總管.png)

   ![](./Images/NewApp_WPF/新建專案_檔案總管.png)

6. 調整資料夾結構

   7. 結束 Visual Studio。

   8. 將「方案」資料夾 `RaphaelApp` 更改名稱為 `src`，並依需要可另外增加 `docs` 等其他用途資料夾。

   ![](./Images/NewApp_WPF/調整資料夾結構.png)

   9. 進入 `src` 資料夾，重新開啟方案 `RaphaelApp.sln`。

10. 設定 `RaphaelApp` 屬性：

   - `應用程式`→`一般`：

     - `目標 Framework` 變更為目標版本。

   - `應用程式`→`Win32資源`：

     - 若有設計 Icon 時，設定在 `圖示` 項目。

   - `建置`→`輸出`：

     - 如果輸出的 bin 資料夾與 obj 資料夾要轉移至其他目錄時，設定 `基底輸出路徑` 為目標資料夾位置。

   - `套件`→`一般`：<br/>

     此頁面用於設定此專案的相關資訊。詳細設定可參考微軟說明網頁：[套件撰寫最佳做法](https://learn.microsoft.com/zh-tw/nuget/create-packages/package-authoring-best-practices)，以下摘錄幾個重要項目。

     - `套件識別碼` (Package ID)：<br/>

       此為專案 ID，基本上不變更，因為它也是預設的 `組件名稱` (AssemblyName) 及 `命名空間`(namespace)。<br/>

       若要對專案設定特別名稱 (給人看的)，請設定在 `職稱` (Title) 欄位。

     - 確認 `套件版本` (Package Version)。

     - `作者` (Authors) 輸入 `Raphael.Wu`。

     - 確認 `說明` (Description) 的內容。

     - `組件中性語言` 選擇 `zh-Hant`。(相關資訊參考 [指定應用程式的中性語言](https://learn.microsoft.com/zh-tw/dotnet/maui/fundamentals/localization?view=net-maui-8.0#specify-the-apps-neutral-language))

---

# 安裝 Nuget

<table>

<tr>

  <td align=center>分類</td>

  <td align=center>Nuget 名稱</td>

  <td align=center>說明</td>

  <td align=center>必裝</td>

</tr><tr>

  <td rowspan="2">程式架構</td>

  <td>Prism.Unity</td>

  <td>有使用 IoC 的組件。(通常為主程式組件)<br/>註：以 Prism 架構建立新專案時會自動導入。</td>

  <td>√</td>

</tr><tr>

  <td>Prism.Wpf</td>

  <td>新建的其他組件/模組 (Assembly/Module) 只要安裝這個即可。</td>

  <td>√</td>

</tr><tr>

  <td rowspan="4">UI / UX</td>

  <td>MaterialDesignThemes</td>

  <td>Google 開發的 WPF 樣式和控件庫。</td>

  <td>√</td>

</tr><tr>

  <td>MahApps.Metro</td>

  <td>由 Jan Karger 等人維護的 WPF 樣式和控件庫。</td>

  <td>√</td>

</tr><tr>

  <td>MaterialDesignThemes.MahApps</td>

  <td>若要同時安裝上述兩套控件庫時，必需再加裝這組 NuGet，用於解決樣式衝突問題。</td>

  <td>√</td>

</tr><tr>

  <td>MahApps.Metro.IconPacks.xxxxx</td>

  <td>Icons 擴充。</td>

  <td></td>

</tr><tr>

  <td>Logger</td>

  <td>Serilog<br/>其他相關 Sink</td>

  <td></td>

  <td></td>

</tr><tr>

  <td rowspan="5">資料處理</td>

  <td>WPFLocalizeExtension</td>

  <td>多國語言介面開發。</br>由資源取用字串。</td>

  <td>√</td>

</tr><tr>

  <td>Newtonsoft.Json</td>

  <td>JSON 格式資料。</td>

  <td></td>

</tr><tr>

  <td>System.Data.SQLite.Core</td>

  <td>SQLite 資料庫。</td>

  <td></td>

</tr><tr>

  <td>Dapper<br/>Dapper.Contrib</td>

  <td>資料庫 ROM。</td>

  <td></td>

</tr><tr>

  <td>MiniExcel</td>

  <td>EXCEL 檔案處理。</td>

  <td></td>

</tr><tr>

  <td rowspan="3">通訊</td>

  <td>IoTClient</td>

  <td>TCP/IP。<br/>Modbus。</td>

  <td></td>

</tr><tr>

  <td>TouchSocket<br/>HP-Socket</td>

  <td>TCP/IP。</td>

  <td></td>

</tr><tr>

  <td>SerialPortStream</td>

  <td>Serial Port。</td>

  <td></td>

</tr><tr>

  <td rowspan="5">自訂</td>

  <td>RaphaelWu.CSharp</td>

  <td>C#基礎擴充。</td>

  <td></td>

</tr><tr>

  <td>RaphaelWu.SQLite</td>

  <td>SQLite 資料庫操作。</td>

  <td></td>

</tr><tr>

  <td>RaphaelWu.WPF</td>

  <td>WPF 基礎擴充。</td>

  <td></td>

</tr><tr>

  <td>RaphaelWu.FA</td>

  <td>MC 協議。</td>

  <td></td>

</tr><tr>

  <td>ZoomAndPan</td>

  <td>Zoom & Pan 模組。</td>

  <td></td>

</tr>

</table>

---

# 基本設定

## MainWindow.xaml

<table>

<tr>

  <td align=center>項目</th><td align=center>內容</th><td align=center>備註</th>

</tr><tr>

  <td>Title</td>

  <td>應用程式標題</td>

  <td>新專案模版預設是從 MainWindowViewModel.cs 中設定標題，如果要改直接在 MainWindow.xmal 中設定標題，則 MainWindowViewModel.cs 中以下程式碼可刪除。<br/><br/>

      private string _title = "Prism Application";<br/>

      public string Title<br/>

      {<br/>

        get { return _title; }<br/>

        set { SetProperty(ref _title, value); }<br/>

      }<br/><br/>

  有安裝 WPFLocalizeExtension 時，可設定為 Title="{lex:Loc ApplicationName}"，從資源讀取。</td>

</tr><tr>

  <td>Width<br/>Height<br/>SizeToContent</td>

  <td>應用程式長寬</td>

  <td><il>

    <li>視窗模式：直接指定長和寬；或不指定長寬而設定 SizeToContent="WidthAndHeight" 讓 WPF 渲染器自行調整至內容大小。</li>

    <li>全螢幕模式：Width 和 Height 設為機台螢幕的尺寸，方便設計畫面時可以看到大致的控件安排。而全螢幕模式的切換，為方便 DEBUG 採用後台程式碼方式切換，參見四、全螢幕模式。</li>

  </il></td>

</tr><tr>

  <td>WindowStartupLocation</td>

  <td>應用程式起始位置</td>

  <td>建議直接設定為 "CenterScreen"。(適用有強迫症的程式設計師)</td>

</tr>

</table>

---

## 擴充 Application.Resources 為合併資源的結構，以便後續 UI/IX 資源的載入

<font color="red">App.xaml</font>

```XML

<prism:PrismApplication x:Class="Glorytek.ProjectName.App"

  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"

  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"

  xmlns:mah="http://metro.mahapps.com/winfx/xaml/controls"

  xmlns:md="http://materialdesigninxaml.net/winfx/xaml/themes"

  xmlns:prism="http://prismlibrary.com/">

  

  <Application.Resources>

    <ResourceDictionary>

      <ResourceDictionary.MergedDictionaries>

        (後續資源插入處)

      </ResourceDictionary.MergedDictionaries>

    </ResourceDictionary>

  </Application.Resources>

</prism:PrismApplication>

```

---

## 補上 App.xaml.cs 缺漏的部分

補上缺漏的 PrismApplication。

<font color="red">App.xaml.cs</font>

```CSharp

public partial class App : PrismApplication

```

---

## UI/UX 資源載入

這裡的 UI (User Interface) 及 UX (User Experience) 採用的是 Google 開發的 [Material Design In XAML Toolkit (MaterialDesignThemes)](http://materialdesigninxaml.net/) 及由 Jan Karger 等人維護的 [MahApps.Metro](https://github.com/MahApps/MahApps.Metro)，其風格屬於 [扁平化設計](https://en.wikipedia.org/wiki/Flat_design)，兩套都是開放原始碼。不過兩套風格有些許不同 (比如文字輸入框)，但可相輔相成，讓 WPF 設計出適合各種風格的介面。

### 一、載入 MahApps.Metro 資源

要載入 MahApps.Metro 的樣版資源，需新增以下三行內容為資源：

<font color="red">App.xaml</font>

```XML

<ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Controls.xaml" />

<ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Fonts.xaml" />

<ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Themes/Dark.Blue.xaml" />

```

第三行中的 Dark.Blue 可設定 MahApps.Metro 為暗底或亮底佈景主題，以及控件的主色調。 主色調建議和 MaterialDesignThemes 設定一樣。突顯色調則會被 MaterialDesignThemes 覆蓋，故突顯色調只要 MaterialDesignThemes 設定即可。可用的色調可至 [官方 Github](https://github.com/MahApps/MahApps.Metro) 下載 Demo 程式先做搭配，或參見 [官方說明文件](https://mahapps.com/docs/themes/usage) 後再輸入。

![](./Images/NewApp_WPF/MahAppsMetro_Palette.png)

---

### 二、載入 MaterialDesignThemes 資源

要載入 MaterialDesignThemes 的樣版資源，需新增以下二行內容為資源：

<font color="red">App.xaml</font>

```XML

<md:BundledTheme BaseTheme="Light" PrimaryColor="Blue" SecondaryColor="Amber" />

<ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesign3.Defaults.xaml" />

```

- 第一行中的 Dark 可與 Light 切換，用於設定暗底佈景主題或亮底佈景主題。
- PrimaryColor 用於設定主色調；SecondaryColor 則用於設定突顯色調。 主色調建議和 MahApps.Metro 設定一樣。可用的色調可至 [官方 Github](https://github.com/MaterialDesignInXAML/MaterialDesignInXamlToolkit) 下載 Demo 程式先做搭配後再輸入。

![](./Images/NewApp_WPF/MaterialDesignThemesDemo_Palette.png)

---

### 三、同時安裝 MaterialDesignThemes 和 MahApps.Metro 時

兩套 UI/UX 均有獨到之處，且可相輔相乘，但需要做一些額外設定以解決樣式衝突。

除了加裝 MaterialDesignTheme 及 MahApps 的 NuGet 套件外，尚需再增加以下二行內容為資源：(放在上述兩組資源的載入設定後面)

<font color="red">App.xaml</font>

```XML

<ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.MahApps;component/Themes/MaterialDesignTheme.MahApps.Fonts.xaml" />

<ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.MahApps;component/Themes/MaterialDesignTheme.MahApps.Flyout.xaml" />

```

對於兩套不相同之處 (比如要開發行動裝置應用程式，兩套預設字型並不相同)，其詳細資訊請參考 [官方 Wiki: MahAppsMetro integration](https://github.com/MaterialDesignInXAML/MaterialDesignInXamlToolkit/wiki/MahAppsMetro-integration)。

---

## 主視窗樣式設定

### 一、視窗模式，有 TitleBar，變更主視窗樣式 (MahApps.Metro 的做法)

MahApps.Metro 有 MetroWindow 功能，可自訂視窗的樣式。主視窗的部分基本的修改如下：

#### MainWindow.xaml

修改的內容如下：

1. 增加 `xmlns:mah` 名稱定義。
2. Window 要改成 `mah:MetroWindow`。

<font color="red">MainWindow.xaml</font>

```XML

<mah:MetroWindow

  x:C`lass="Glorytek.ProjectName.Views.MainWindow"

  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"

  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"

  xmlns:mah="http://metro.mahapps.com/winfx/xaml/controls"

  xmlns:prism="http://prismlibrary.com/"

  Title="Glorytek.ProjectName"

  Width="525" Height="350"

  prism:ViewModelLocator.AutoWireViewModel="True">

  <Grid>

    <ContentControl prism:RegionManager.RegionName="ContentRegion" />

  </Grid>`

</mah:MetroWindow>

```
