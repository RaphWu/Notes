---
aliases:
date:
update:
author: Raphael Wu
language:
sourceurl:
tags:
---

# UI /UX

## Krypton-Suite

[GitHub](https://github.com/orgs/Krypton-Suite/repositories)
- [Standard Toolkit](https://github.com/Krypton-Suite/Standard-Toolkit)
	- [Standard-Toolkit-Online-Help](https://krypton-suite.github.io/Standard-Toolkit-Online-Help/Source/Help/Output/index.html)
	- [Standard-Toolkit-Demos](https://github.com/Krypton-Suite/Standard-Toolkit-Demos)
- [Extended Toolkit](https://github.com/Krypton-Suite/Extended-Toolkit)
[API Reference](https://github.com/Krypton-Suite/Documentation/tree/main/Documents/API%20Reference)
- [Krypton.Toolkit Namespace](https://github.com/Krypton-Suite/Documentation/blob/main/Documents/API%20Reference/Standard%20Toolkit/79d2eac2-21f4-54ff-7552-b20c33c30600.md)
- 

## SunnyUI.NET

- [GitHub](https://github.com/yhuse/SunnyUI)
- [Gitee](https://gitee.com/yhuse/SunnyUI)
- [帮助文档](https://gitee.com/yhuse/SunnyUI/wikis/pages)

## HZHControls

- [官方網站](https://www.hzhcontrols.cn/)
- [GitHub](https://github.com/kwwwvagaa/NetWinformControl)
- [Gitee](https://gitee.com/kwwwvagaa/net_winform_custom_control)
- [文档](https://www.hzhcontrols.cn/doc.html)
- [Blog](https://www.cnblogs.com/bfyx/p/11364884.html)

## ReaLTaiizor

- [GitHub](https://github.com/ComponentFactory/Krypton)

## MaterialSkin 2 for .NET WinForms

[GitHub](https://github.com/leocb/MaterialSkin)

⚠️ 建議不要使用此程式庫來建立新專案 ⚠️
ℹ️ 目前此專案狀態為：非活躍

## MaterialSkin for .NET WinForms

[GitHub](https://github.com/IgnaceMaes/MaterialSkin)

特色：基於 Google Material Design，提供現代化的 UI 控制項與動畫效果。
授權：MIT 授權，免費且可商用。
使用方式：安裝 NuGet 套件，應用 MaterialSkin 風格於 WinForms 控制項。

ℹ️ This project is no longer under active development.

## Siticone UI

 [官方網站](https://www.siticoneframework.com/start)

- 特色：提供 260 多個現代化 UI 控制項，支援 Visual Studio 設計器。
- 授權：免費版本可用於個人與小型商業用途，需註冊取得授權。

## DockPanel Suite

用於 .Net Windows Forms 開發的 docking library，模擬 Visual Studio .Net。

- [sourceforge.net](https://sourceforge.net/projects/dockpanelsuite/)
- [GitHub](https://github.com/dockpanelsuite/dockpanelsuite)
- [官方網站](https://dockpanelsuite.com/)
- [DockPanel Suite Documentation](https://docs.dockpanelsuite.com/)

### 網路文章

- [DockPanelSuite的基础使用](https://blog.csdn.net/qq_41375318/article/details/135226090)
- [浮动多窗体程序组件DockPanel Suite的使用介绍](https://jytek.com/news?article_id=371)

---

# DataGridView

## SourceGrid

 [GitHub - siemens](https://github.com/siemens/sourcegrid)
 [GitHub - huanlin](https://github.com/braillekit/SourceGrid)
- 特色
	- 功能非常強大的 Grid 控制項，不依賴 DataGridView。
	- 支援 分組、樹狀節點、篩選、格式化、虛擬模式。
	- 可高度自訂儲存格（甚至可放控件）。
- 缺點
	- 學習曲線比較高，API 和 DataGridView 完全不同。

### 網路文章

- [SourceGrid .NET 控件源码深度解析与学习](https://blog.csdn.net/weixin_36474966/article/details/144202808)
- [SourceGrid 应用/中文文档](https://www.cnblogs.com/vainnetwork/archive/2008/05/31/1211123.html)
- [SourceGrid - Open Source C# Grid Control](https://www.codeproject.com/articles/SourceGrid-Open-Source-C-Grid-Control)

## DynamicGrid

[GitHub](https://github.com/TomaszRewak/DynamicGrid)

DynamicGrid 的限制：

- 所有列的高度必须相同。
- 单元格內容和版面配置選項僅限於以下屬性：`Text`、`TextAlignment`、`FontStyle`、`BackgroundColor`、`ForegroundColor`。
- DynamicGrid 只是一個網格渲染引擎，不提供任何列管理功能。它也不允許顯示列標題。這個問題可以通過在網格上方放置一個 `DataGridView` 控制元件來解決（僅用於顯示標題）。
- Dynamic Grid，在出廠時，並不提供任何文字輸入或單元選擇的功能。如果需要，這些功能必須根據個別需求在衍生的類別中實現（另一方面，這也允許更高的靈活性）。

有關此程式庫的更多細節，也可以在這個 [YouTube 影片](https://www.youtube.com/watch?v=M_pu9_LUfXo) 中找到。

## AdvancedDataGridView

[GitHub](https://github.com/davidegironi/advanceddatagridview)

- 特色
	- 基於標準 DataGridView，完全免費。
	- 支援 Filter 和 Sort 下拉選單（像 Excel 那樣）。
	- 支援 樹狀折疊 (TreeGrid)。
	- API 與原生 DataGridView 類似，容易上手。
- 適合場景
	- 如果你想要「保留 DataGridView 的用法」但增加 Excel 式的篩選、群組、折疊功能。

### Zuby’s TreeGridView for WinForms

- 如果你只是要「折疊/展開」，這個專案裡有 TreeGridView，可直接在 DataGridView 上用樹狀結構。

## ObjectListView

 [官方網站](https://objectlistview.sourceforge.net/cs/index.html)
- 特色
	- 雖然是基於 ListView，但在 WinForms 常用來取代 DataGridView。
	- 支援 排序、過濾、群組、樹狀結構 (TreeList)。
	- 支援 列格式化、圖片、CheckBox。
	- 比 DataGridView 容易操作、外觀也更漂亮。
- 缺點
	- 若你一定要基於 DataGridView，這可能不適合，因為它是 ListView 的封裝。

---

# 2D 繪圖

## ScottPlot.NET

- [官方網站](https://scottplot.net/)
- [GitHub](https://github.com/ScottPlot/ScottPlot)

## ZedGraph

ZedGraph 是一組用 C# 編寫的類，用於建立任意資料集的二維折線圖和長條圖。這些類別提供了高度的靈活性——幾乎圖表的每個方面都可以由使用者修改。同時，透過為所有圖表屬性提供預設值，這些類別的使用也變得簡單易用。這些類別還包含用於根據繪製的資料值範圍選擇合適的比例範圍和步長的程式碼。
ZedGraph 還包含一個 UserControl 介面，可在 Visual Studio 窗體編輯器中進行拖放編輯，並可從 C++ 和 VB 等其他語言存取。 ZedGraph 採用 [LGPL](wiki/ZedGraph License) 授權。

- [sourceforge.net](https://sourceforge.net/projects/zedgraph/)
- [GitHub](https://github.com/ZedGraph/ZedGraph)
- [ZedGraph Wiki](https://github.com/ZedGraph/ZedGraph/wiki)
