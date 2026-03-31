---
aliases:
date: 2025-04-04
update:
author: CornerChat543
language: C#, XAML
sourceurl: https://cornerchat543.github.io/2025/04/wpf-mvvm-from-single-to-multiple-combinations/
tags:
  - CSharp
  - WPF
  - MVVM
  - CommunityToolkit_Mvvm
---

# WPF MVVM 從單一到多元的 View 與 ViewModel 組合變化

MVVM 是 WPF 應用程式開發中的核心架構，透過 資料綁定 (Binding) 和 命令模式 (Command)，讓 View 和 ViewModel 解耦，提高可維護性。

在使用過程中，個人最初從最基本的 一對一 關係開始學習，隨著專案需求變得更複雜，也開始嘗試 一對多、多對多，甚至多對一 的組合方式。這些不同的組合變化，某種程度上也反應了自己在 MVVM 架構上的成長，因此想記錄下這四種組合，作為經驗的整理。

## 開發環境與套件

本次的開發環境，如下

- IDE : Visual Studio 2022
- .NET 版本 : . NET 6.0
- Nuget 套件: CommunityToolkit.Mvvm 8.2.2

其中，`CommunityToolkit.Mvvm` 套件能幫助減少 ViewModel 中許多重複的樣板程式碼。如果對這個套件不太熟悉，建議可以先查閱相關資料，或參考本 Blog 其他文章來進一步了解。

另外，接下來的四個範例，雖然是分開的專案，但它們的 `App.xaml` 都會包含相同的資源與樣式。為了避免重複，在範例程式碼中不會每次都列出這部分。

以下為 `App.xaml` 中共通的資源設定：

```xml
<Application.Resources>
    <Thickness x:Key="Margin.Top">0,10,0,0</Thickness>
    <Style x:Key="BorderStyle" TargetType="Border">
        <Setter Property="Margin" Value="5"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="BorderBrush" Value="Gray"/>
        <Setter Property="CornerRadius" Value="8"/>
        <Setter Property="Background" Value="White"/>

        <Setter Property="Effect">
            <Setter.Value>
                <DropShadowEffect Color="#D2D2D2"
                              Direction="270"
                              ShadowDepth="3"
                              BlurRadius="10"
                              Opacity="0.3"/>
            </Setter.Value>
        </Setter>
    </Style>
</Application.Resources>
```

## 組合 1 - 單一 View 與 ViewModel 的基本結構

首先介紹最基礎的 WPF MVVM 架構，

單一 View 搭配一個 ViewModel，

這是最入門的架構，

ViewModel 直接負責 View 所需的資料與邏輯。

![[View與ViewModel組合變化_單一 View 與 ViewModel 架構圖.png]]

ViewModel 的程式碼 `MainViewModel` ，此範例中，定義了四個屬性：`Class`、`Number`、`English` 和 `Math`，用來表示班級、號碼與兩科成績。
此外，`ClickCommand` 負責在按鈕點擊時隨機產生這些屬性的值，模擬動態資料變化。

```csharp title="MainViewModel"
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace WpfViewAndVmSample01
{
    public partial class MainViewModel : ObservableObject
    {
        [ObservableProperty]
        private string? _class;

        [ObservableProperty]
        private int _number;

        [ObservableProperty]
        private int _english;

        [ObservableProperty]
        private int _math;

        public MainViewModel()
        { }

        [RelayCommand]
        private void Click()
        {
            var random = new Random();

            Class = $"{(char)random.Next('A', 'F')}";
            Number = random.Next(1, 31);
            English = random.Next(0, 101);
            Math = random.Next(0, 101);
        }
    }
}
```

View 的部分，則使用一個 Window，為了簡化示範，這裡直接在 `DataContext` 中建立 ViewModel，實際專案中則建議透過依賴注入或其他方式來管理 ViewModel。

UI 布局上，畫面分為上下兩個區塊：
- 上方區塊顯示 班級 (`Class`) 與 號碼 (`Number`)，並搭配一個按鈕來產生資料。
- 下方區塊則顯示 英文 (`English`) 與 數學 (`Math`) 的成績。

```xml title="MainWindow"
<Window.DataContext>
    <local:MainViewModel/>
</Window.DataContext>

<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <Border Grid.Row="0" Style="{StaticResource BorderStyle}">
        <StackPanel Margin="5">
            <!--Class-->
            <TextBlock Text="Class:"/>
            <TextBox Text="{Binding Path=Class}"/>

            <!--Number-->
            <TextBlock Margin="{StaticResource Margin.Top}"
                       Text="Number:"/>
            <TextBox Text="{Binding Path=Number}"/>

            <!--Click-->
            <Button Margin="{StaticResource Margin.Top}"
                    Height="50"
                    Content="Click"
                    Command="{Binding Path=ClickCommand}"/>
        </StackPanel>
    </Border>
    <Border Grid.Row="1" Style="{StaticResource BorderStyle}">
        <StackPanel Margin="5">
            <!--English-->
            <TextBlock Text="English:"/>
            <TextBox Text="{Binding Path=English}"/>

            <!--Math-->
            <TextBlock Margin="{StaticResource Margin.Top}"
                       Text="Math:"/>
            <TextBox Text="{Binding Path=Math}"/>
        </StackPanel>
    </Border>
</Grid>
```

執行的畫面與結果，如下圖，

![[View與ViewModel組合變化_範例執行畫面01.png]]

看完最基礎的 MVVM 架構後，接下來的範例將基於這個基礎進行變化，透過不同的方式組合 View 與 ViewModel，來應對更複雜的需求。

## 組合 2 - 單一 View 與多個 ViewModel 的分工與互動

在物件導向設計中，通常會將程式模組化，讓不同的物件負責不同的功能。
同樣地，在 MVVM 架構下，當應用程式變得更為複雜時，可能不再適合使用單一 ViewModel 來處理所有邏輯，而是拆分成多個 ViewModel。

![[View與ViewModel組合變化_單一 View 與多個 ViewModel 架構圖.png]]

在範例 2 中，將範例 1 的 ViewModel 拆分為兩個，分別負責不同部分的邏輯。
其中， `SubViewModel01` 負責班級與號碼的處理，`SubViewModel02` 則處理成績的數據。
此外，此範例也簡單示範了 ViewModel 之間的溝通方式，這邊用最簡單的 `Action` 來示範。
而 ViewModel 之間的互動方式有很多種，例如 觀察者模式、`Event`、`Action`/`Func`、`RX` 或 `Messenger` 等。
開發時可以根據需求選擇最適合的方式，或是使用自己較熟悉的工具來實作。

```csharp title="SubViewModel01"
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace WpfViewAndVmSample02
{
    public partial class SubViewModel01 : ObservableObject
    {
        public Action? Action
        { get; set; }

        [ObservableProperty]
        private string? _class;

        [ObservableProperty]
        private int _number;

        public SubViewModel01()
        { }

        [RelayCommand]
        private void Click()
        {
            var random = new Random();

            Class = $"{(char)random.Next('A', 'F')}";
            Number = random.Next(1, 31);
            Action?.Invoke();
        }
    }
}
```

```csharp title="SubViewModel02"
using CommunityToolkit.Mvvm.ComponentModel;

namespace WpfViewAndVmSample02
{
    public partial class SubViewModel02 : ObservableObject
    {
        [ObservableProperty]
        private int _english;

        [ObservableProperty]
        private int _math;

        public SubViewModel02()
        { }

        public void DoWork()
        {
            var random = new Random();

            English = random.Next(0, 101);
            Math = random.Next(0, 101);
        }
    }
}
```

此範例，使用了 `MainViewModel` 來管理多個子 ViewModel。
這樣的架構可以讓 View 只需要設定一個 `DataContext`，並透過 `MainViewModel` 來統一管理 ViewModel 之間的關聯，實務上，則根據狀況而改變。

這邊 `MainViewModel` 主要負責：
- 管理 `SubViewModel` 的實例 (這邊直接 `new`，也可以用依賴注入)
- 建立 `SubViewModel` 之間的關聯

```csharp title="MainViewModel"
using CommunityToolkit.Mvvm.ComponentModel;

namespace WpfViewAndVmSample02
{
    public class MainViewModel : ObservableObject
    {
        public SubViewModel01 SubViewModel01 { get; set; }
        public SubViewModel02 SubViewModel02 { get; set; }

        public MainViewModel()
        {
            SubViewModel01 = new();
            SubViewModel02 = new();
            SubViewModel01.Action = SubViewModel02.DoWork;
        }
    }
}
```

View 的部分，整體結構與範例 1 基本相同，但因為使用了多個 ViewModel，所以需要透過不同的 Binding 方式來綁定資料。
這裡的兩個區塊，剛好可以示範兩種不同的 Binding 方式。

方式 1：直接綁定屬性 (不額外設定 `DataContext`)

- 直接綁定 ViewModel 屬性名

```xml
<TextBox Text="{Binding Path=SubViewModel01.Class}"/>
```

- 設定 `DataContext`，並綁定內部屬性

```xml
<TextBox DataContext="{Binding Path=SubViewModel01}" Text="{Binding Path=Number}"/>
```

方式 2：設定容器 `DataContext` 來簡化內部 Binding

- 設定容器 (如 `StackPanel` ) 的 `DataContext`，內部元素可直接綁定其屬性，無須重複打前綴

```xml
<StackPanel Margin="5" DataContext="{Binding Path=SubViewModel02}">
	<!--English-->
	<TextBlock Text="English:"/>
	<TextBox Text="{Binding Path=English}"/>
	
	<!--Math-->
	<TextBlock Margin="{StaticResource Margin.Top}" Text="Math:"/>
	<TextBox Text="{Binding Path=Math}"/>
</StackPanel>
```

而在某些情況下，因為 UI 設計的因素，可能會遇到「穿插」的 Binding 狀況。
在這種情況下，可以使用 `RelativeSource` 或 `ElementName` 來實現對更高層級的 ViewModel 或屬性的綁定。

```xml title="MainWindow"
<Window.DataContext>
    <local:MainViewModel/>
</Window.DataContext>

<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
	</Grid.RowDefinitions>

    <Border Grid.Row="0" Style="{StaticResource BorderStyle}">
        <StackPanel Margin="5">
            <!--Class-->
            <TextBlock Text="Class:"/>
            <TextBox Text="{Binding Path=SubViewModel01.Class}"/>

            <!--Number-->
            <TextBlock Margin="{StaticResource Margin.Top}" Text="Number:"/>
            <TextBox DataContext="{Binding Path=SubViewModel01}" Text="{Binding Path=Number}"/>

            <!--Click-->
            <Button Margin="{StaticResource Margin.Top}" Height="50" Content="Click"
                    Command="{Binding Path=SubViewModel01.ClickCommand}"/>
        </StackPanel>
    </Border>
    <Border Grid.Row="1" Style="{StaticResource BorderStyle}">
        <StackPanel Margin="5" DataContext="{Binding Path=SubViewModel02}">
            <!--English-->
            <TextBlock Text="English:"/>
            <TextBox Text="{Binding Path=English}"/>

            <!--Math-->
            <TextBlock Margin="{StaticResource Margin.Top}" Text="Math:"/>
            <TextBox Text="{Binding Path=Math}"/>
        </StackPanel>
    </Border>
</Grid>
```

到此範例 2，展示了 1 個 View 搭配多個 ViewModel 的情況。
從 `MainView` 的程式碼來看，雖然程式碼簡潔了不少，但架構變得多層且複雜了，在大型專案中，可能會遇到找不到想要綁定的屬性情況。

## 組合 3 - 多個 View 與多個 ViewModel 的模組化設計

既然範例 2 拆分了多個 ViewModel ，那麼再來就是把 View 也進行拆分。

![[View與ViewModel組合變化_多個 View 與多個 ViewModel 架構圖.png]]

在 ViewModel 部分，包含 `SubViewModel01`、`SubViewModel02` 和 `MainViewModel`，這些類別的內容與範例 2 相同，因此不再重複說明。

```csharp title="SubViewModel01"
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace WpfViewAndVmSample03
{
    public partial class SubViewModel01 : ObservableObject
    {
        public Action? Action
        { get; set; }

        [ObservableProperty]
        private string? _class;

        [ObservableProperty]
        private int _number;

        public SubViewModel01()
        { }

        [RelayCommand]
        private void Click()
        {
            var random = new Random();

            Class = $"{(char)random.Next('A', 'F')}";
            Number = random.Next(1, 31);
            Action?.Invoke();
        }
    }
}
```

```csharp title="SubViewModel02"
using CommunityToolkit.Mvvm.ComponentModel;

namespace WpfViewAndVmSample03
{
    public partial class SubViewModel02 : ObservableObject
    {
        [ObservableProperty]
        private int _english;

        [ObservableProperty]
        private int _math;

        public SubViewModel02()
        { }

        public void DoWork()
        {
            var random = new Random();

            English = random.Next(0, 101);
            Math = random.Next(0, 101);
        }
    }
}
```

```csharp title="MainViewModel"
using CommunityToolkit.Mvvm.ComponentModel;

namespace WpfViewAndVmSample03
{
    public class MainViewModel : ObservableObject
    {
        public SubViewModel01 SubViewModel01
        { get; set; }

        public SubViewModel02 SubViewModel02
        { get; set; }

        public MainViewModel()
        {
            SubViewModel01 = new();
            SubViewModel02 = new();
            SubViewModel01.Action = SubViewModel02.DoWork;
        }
    }
}
```

接下來 View 的部分，對應 ViewModel，同樣拆分成兩個 `SubView`，這裡使用 `UserControl` 來示範，當然也可以用其他方式來實現。
`SubView01` 的部分，將原本的第一個 UI 區塊拆分出來，獨立成一個 `UserControl`，其內容與原本在 `MainWindow` 的 UI 相同，負責顯示 `Class` 和 `Number` 屬性，並提供按鈕觸發命令。

```xml title="SubView01"
<Border Style="{StaticResource BorderStyle}">
    <StackPanel Margin="5">
        <!--Class-->
        <TextBlock Text="Class:"/>
        <TextBox Text="{Binding Path=Class}"/>

        <!--Number-->
        <TextBlock Margin="{StaticResource Margin.Top}" Text="Number:"/>
        <TextBox Text="{Binding Path=Number}"/>

        <!--Click-->
        <Button Margin="{StaticResource Margin.Top}" Height="50" Content="Click"
            Command="{Binding Path=ClickCommand}"/>
    </StackPanel>
</Border>
```

而 `SubView02` 的部分，則負責顯示 `English` 和 `Math` 的分數顯示。

```xml title="SubView02"
<Border Style="{StaticResource BorderStyle}">
    <StackPanel Margin="5">
        <!--English-->
        <TextBlock Text="English:"/>
        <TextBox Text="{Binding Path=English}"/>

        <!--Math-->
        <TextBlock Margin="{StaticResource Margin.Top}" Text="Math:"/>
        <TextBox Text="{Binding Path=Math}"/>
    </StackPanel>
</Border>
```

最後，將兩個 `SubView` 放入 `MainWindow` 中，並且設定 `DataContext` 並綁定對應的 ViewModel，這樣每個 `SubView` 都會擁有自己獨立的 ViewModel。

```xml title="MainWindow"
<Window.DataContext>
    <local:MainViewModel/>
</Window.DataContext>

<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <local:SubView01 Grid.Row="0" DataContext="{Binding Path=SubViewModel01}"/>
    <local:SubView02 Grid.Row="1" DataContext="{Binding Path=SubViewModel02}"/>
</Grid>
```

從 1 個 View 和 1 個 ViewModel 變成 3 個 View 和 3 個 ViewModel，複雜度增加了不少，但從 Main 的角度來看，整體程式碼變得整潔了一些。

## 組合 4 - 多個 View 與 單一 ViewModel 的奇特組合

前面的三種組合，無論是學習還是實務應用，都有機會遇到並使用。
本以為已經涵蓋了全部的架構變化，但某次因應 UI 需求，需要拆分成多個 View，然而這些 View 需要綁定的內容並不多，覺得懶得拆分多個 ViewModel，於是嘗試了「多個 View 共用單一 ViewModel」的組合。

![[View與ViewModel組合變化_多個 View 與單一 ViewModel 架構圖.png]]

ViewModel 的部分只有一個，跟範例 1 是一模一樣的，內容就不重複說明。

```csharp title="MainViewModel"
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace WpfViewAndVmSample04
{
    public partial class MainViewModel : ObservableObject
    {
        [ObservableProperty]
        private string? _class;

        [ObservableProperty]
        private int _number;

        [ObservableProperty]
        private int _english;

        [ObservableProperty]
        private int _math;

        public MainViewModel()
        { }

        [RelayCommand]
        private void Click()
        {
            var random = new Random();

            Class = $"{(char)random.Next('A', 'F')}";
            Number = random.Next(1, 31);
            English = random.Next(0, 101);
            Math = random.Next(0, 101);
        }
    }
}
```

View 的部分 99 % 則跟範例 3 相同，`SubView01` 與 `SubView01` 是完全相同的。

```xml title="SubView01"
<Border Style="{StaticResource BorderStyle}">
    <StackPanel Margin="5">
        <!--Class-->
        <TextBlock Text="Class:"/>
        <TextBox Text="{Binding Path=Class}"/>

        <!--Number-->
        <TextBlock Margin="{StaticResource Margin.Top}" Text="Number:"/>
        <TextBox Text="{Binding Path=Number}"/>

        <!--Click-->
        <Button Margin="{StaticResource Margin.Top}" Height="50" Content="Click"
            Command="{Binding Path=ClickCommand}"/>
    </StackPanel>
</Border>
```

```xml title="SubView02"
<Border Style="{StaticResource BorderStyle}">
    <StackPanel Margin="5">
        <!--English-->
        <TextBlock Text="English:"/>
        <TextBox Text="{Binding Path=English}"/>

        <!--Math-->
        <TextBlock Margin="{StaticResource Margin.Top}" Text="Math:"/>
        <TextBox Text="{Binding Path=Math}"/>
    </StackPanel>
</Border>
```

`MainWindow` 基本上與範例 3 相同，唯一的不同在於 `DataContext` 的設定。
在 MainWindow 中使用 `SubView01` 和 `SubView02` 時，其中的 `DataContext` 設定為 `{Binding}`，這表示 直接綁定到 `MainViewModel`，而不是各自綁定到獨立的 `SubViewModel`。

```xml title="MainWindow"
<Window.DataContext>
    <local:MainViewModel/>
</Window.DataContext>

<Grid Margin="10">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <local:SubView01 Grid.Row="0" DataContext="{Binding}"/>
    <local:SubView02 Grid.Row="1" DataContext="{Binding}"/>
</Grid>
```

當 `DataContext=”{Binding}”` 時，該項目會繼承父級的 `DataContext`，也就是會綁定到與父級相同的 ViewModel。
這可減少的 `DataContext` 設定，使綁定更簡潔，但如果父級 ViewModel 沒有對應的屬性，則會導致 Binding 失效。
如果有些 View 拆成較小的組件時，可藉由這種多個 View 單個 ViewModel 的組合方式，讓 View 更加模組化與簡潔，同時 ViewModel 能不這麼複雜，也是個不錯的設計選擇。

## 總結

透過這 4 種組合範例，大致可以掌握 View 與 ViewModel 之間的不同搭配方式。
在 ViewModel 的設計上，若熟悉物件導向概念，通常會習慣將物件封裝並模組化，讓程式結構更清晰，不過需要開放屬性作為對外接口。
而 View 的部分，同樣可以透過模組化設計來提升可維護性，根據需求選擇適合的方式，例如繼承 `Control` 自訂元件，或是使用 `UserControl` 來拆分 UI。
比較關鍵的地方在於，如何讓 View 與 ViewModel 配合良好，其中 Binding 設定是否正確，將直接影響資料的傳遞與顯示。

為了理解 `DataContext` 的運作，這裡整理了三種情況：
- **未**設定 `DataContext`：無法找到 Binding 來源，導致綁定無效。
- 設定特定的 Binding 來源：直接指定特定的 ViewModel 或物件，例如 `DataContext=”{Binding Path=SubViewModel01}”`。
- 設定 `{Binding}`：沿用父級的 `DataContext`，從視覺樹向上尋找最近的可用 `DataContext`。

理解這些概念，能讓 View 與 ViewModel 的搭配更靈活，在專案開發中選擇更適合的方式，提升程式的可讀性與擴展性。

## 自言自語 543

個人初接觸 C# 時， UI 方面是從 Winform 開始入門的，當時在做開發時，並沒有使用什麼特別的架構，且 Winform 也難以自製 UI 樣式。
某天得知了 WPF ，並且也得知可以製作華麗的 UI ，於是開始嘗試使用 WPF，剛開始學習時，仍然習慣像在 Winform 那樣，直接透過 UI 的事件處理邏輯，來控制程式行為。
隨著學習的深入，才接觸到 MVVM 架構，也了解到 WPF 是專為這種架構設計的，於是開始轉向這種方式來撰寫程式。
從最初簡單的單個 View 搭配單個 ViewModel，到現在因應專案需求，逐漸習慣將類別拆分，並變化不同的 View 與 ViewModel 數量組合，當初還真沒想過這些狀況呢。
