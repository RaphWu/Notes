---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Main Window

```xml
<Window

    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"

    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"

    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"

    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"

    mc:Ignorable="d"

  

    xmlns:system="clr-namespace:System;assembly=System.Runtime"

    xmlns:sys="clr-namespace:System;assembly=System.Runtime"

  

    xmlns:system="clr-namespace:System;assembly=mscorlib"

    xmlns:sys="clr-namespace:System;assembly=mscorlib"

  

    xmlns:prism="http://prismlibrary.com/"

    prism:ViewModelLocator.AutoWireViewModel="True"

  

    xmlns:md="http://materialdesigninxaml.net/winfx/xaml/themes"

  

    xmlns:mah="http://metro.mahapps.com/winfx/xaml/controls"

    xmlns:mah="clr-namespace:MahApps.Metro.Controls;assembly=MahApps.Metro"

    xmlns:iconPacks="http://metro.mahapps.com/winfx/xaml/iconpacks"

  

    xmlns:i="http://schemas.microsoft.com/xaml/behaviors"

        xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"

        xmlns:i="http://schemas.microsoft.com/expression/2010/interactions"

        xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity"

  

    Loaded="OnLoaded"

    Unloaded="OnUnloaded"

  

    xmlns:behavior="clr-namespace:Prism.Behaviors;"

    xmlns:behavior="http://schemas.microsoft.com/xaml/behaviors"

  

    xmlns:lex="http://wpflocalizeextension.codeplex.com"

    lex:LocalizeDictionary.DesignCulture="zh-Hant"

    lex:ResxLocalizationProvider.DefaultAssembly="Machine"

    lex:ResxLocalizationProvider.DefaultDictionary="Resources"

  

    d:DesignHeight="807"

    d:DesignWidth="1280"

    Style="{StaticResource BaseUserControlStyle}"

  

    xmlns:behaviors="http://wpfwindowtoolkit.org/behaviors"

    xmlns:helpers="http://wpfwindowtoolkit.org/helpers"

  

    MouseRightButtonDown="MouseRightButtonDown"

>

  

    <i:Interaction.Triggers>

        <!--  視窗載入  -->

        <i:EventTrigger EventName="Loaded">

            <i:InvokeCommandAction Command="{Binding LoadedCommand}" />

        </i:EventTrigger>

  

        <!--  視窗卸載  -->

        <i:EventTrigger EventName="Unloaded">

            <i:InvokeCommandAction Command="{Binding UnloadedCommand}" />

        </i:EventTrigger>

  

        <!--  視窗顯示/隱藏狀態變更  -->

        <i:PropertyChangedTrigger Binding="{Binding Visibility, ElementName=ObjectMoveWindow, Mode=OneWay}">

            <prism:InvokeCommandAction Command="{Binding VisibleChangedCommand}" />

        </i:PropertyChangedTrigger>

    </i:Interaction.Triggers>

  

</Window>

```

# Main ViewModel

```csharp

using OEP520G.Core;

using OEP520G.Parameter;

using Prism;

using Prism.Commands;

using Prism.Mvvm;

using System;

using System.Collections.Generic;

using System.Linq;

  

public class xxxViewModel : BindableBase, IActiveAware

  

/********************

 * IActiveAware & ApplicationCommands

 ********************/

private bool _isActive = false;

public bool IsActive

{

    get { return _isActive; }

    set

    {

        _isActive = value;

        OnIsActiveChanged();

  

        if (value)

        {

        }

        else

        {

        }

    }

}

public event EventHandler IsActiveChanged;

public DelegateCommand SaveDataCommand { get; private set; }

private void OnIsActiveChanged()

{

    SaveDataCommand.IsActive = IsActive;

    IsActiveChanged?.Invoke(this, new EventArgs());

}

  

/********************

 * ctor

 ********************/

public xxxViewModel(IApplicationCommands applicationCommands)

{

    SaveDataCommand = new DelegateCommand(ExecuteSaveDataCommand).ObservesCanExecute(() => CanApply);

    // TODO: UnregisterCommand

    SaveDataCommand = new DelegateCommand(ExecuteSaveDataCommand);

    applicationCommands.SaveDataCommand.RegisterCommand(SaveDataCommand);

}

  

/********************

 * System Commands

 ********************/

/// <summary>

/// 將資料寫入資料庫

/// 將Model資料寫入資料庫

/// 將View資料寫入Model及資料庫

/// </summary>

public void ExecuteSaveDataCommand() => WriteToDb();

  

/********************

 * Database

 ********************/

/// <summary>

/// 將Model資料寫入資料庫

/// </summary>

public bool WriteToDb()

{

    if (!_pm.IsProductActive)

        return;

  

    if (IsActive)

    {

        SaveData();

    }

}

  

/// <summary>

/// 將View資料存回Model

/// </summary>

public bool SaveData()

{

    if (!_pm.IsProductActive)

        return;

  

    if (IsActive)

    {

    }

}

  

/// <summary>

/// 從資料庫讀取資料至Model

/// </summary>

public bool ReadFromDb()

{

    if (!_pm.IsProductActive)

        return;

  

    if (IsActive)

    {

        _shape.ReadFromDb();

        LoadData();

    }

}

  

/// <summary>

/// 從Model載入資料至View

/// </summary>

public bool LoadData()

{

    if (!_pm.IsProductActive)

        return;

  

    if (IsActive)

    {

    }

}

```

[[回目錄]](#目錄)

## Database

```csharp

/********************

 * Database

 ********************/

/// <summary>

/// 將Model資料寫入資料庫

/// 將View資料寫入Model及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

bool WriteToDb();

  

/// <summary>

/// 從資料庫讀取資料至Model

/// 從資料庫讀取資料至Model及View

/// </summary>

/// <returns>是否讀取成功</returns>

bool ReadFromDb();

  
  

/// <summary>

/// 將Model資料寫入資料庫

/// 將View資料寫入Model及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

public bool WriteData()

{

}

  
  

/// <summary>

/// 從資料庫讀取資料至Model及View

/// </summary>

/// <returns>是否讀取成功</returns>

public bool ReadData()

{

}

  

/********************

 * Database

 ********************/

/// <summary>

/// 將Model資料寫入資料庫

/// 將View資料寫入Model及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

private bool SaveData()

{

    if (IsActive)

    {

    }

}

  

/// <summary>

/// 從Model載入資料至View

/// </summary>

/// <returns>是否讀取成功</returns>

public bool LoadData()

{

}

  
  

/********************

 * Database

 ********************/

/// <inheritdoc/>

public bool WriteToDb()

{

    using var conn = _sqlite.OpenConnection(_pm.DB_NAME_PRODUCT);

    if (conn == null)

    {

        Console.WriteLine($"資料庫 '{_pm.DB_NAME_PRODUCT}' 開啟失敗！");

        return false;

    }

  

    using var tran = conn.BeginTransaction();

    try

    {

        // 旋轉中心

        conn.UpdateAsync(StageParameters.StageCenter);

  

        // 旋轉位移

        conn.Execute($"DELETE FROM {DB.TABLE_NAME_STAGE_ECCENTRIC}");

        conn.InsertAsync(StageParameters.StageEccentric);

    }

    catch

    {

        Console.WriteLine($"Stage資料庫(旋轉中心/旋轉位移)寫入失敗！");

        return false;

    }

    tran.Commit();

  

    return true;

}

  

/// <inheritdoc/>

public bool ReadFromDb()

{

    using var conn = _sqlite.OpenConnection(_pm.DB_NAME_PRODUCT);

    if (conn == null)

    {

        Console.WriteLine($"資料庫 '{_pm.DB_NAME_PRODUCT}' 開啟失敗！");

        return false;

    }

  

    using var tran = conn.BeginTransaction();

    try

    {

        if (_sqlite.IsTableExist(conn, DB.TABLE_NAME_CAMERA))

        {

            CameraParameters.Camera = conn.GetAll<CameraDefine>().ToList();

        }

        else

        {

            _sqlite.CreateTable(conn, DB.CREATE_TABLE_CAMERA);

  

            CameraParameters.Camera = new List<CameraDefine>

            {

                new CameraDefine() { Id = CameraId.FixCamera },

                new CameraDefine() { Id = CameraId.MoveCamera }

            };

            WriteToDb();

        }

    }

    catch

    {

        Console.WriteLine("Camera資料庫載入失敗！");

        return false;

    }

    tran.Commit();

  

    return true;

}

```

[[回目錄]](#目錄)

# Command Binding

```csharp

/********************

 * Command Binding

 ********************/

  

private DelegateCommand _ACommand;

public DelegateCommand ACommand

    => _ACommand ??= new DelegateCommand(Execute);

private void Execute()

{

}

  

private DelegateCommand<string> _ACommand;

public DelegateCommand<string> ACommand

    => _ACommand ??= new DelegateCommand<string>(Execute);

private void Execute(string para)

{

}

  
  
  

public DelegateCommand ACommand { get; set; }

ACommand = new DelegateCommand(Execute);

private void Execute()

{

}

  

public DelegateCommand<string> ACommand { get; set; }

ACommand = new DelegateCommand<string>(Execute);

private void Execute(string para)

{

}

```

[[回目錄]](#目錄)

# Data Binding

```csharp

/********************

 * Data Binding

 ********************/

  

// 是否允許執行ApplyCommand

private bool _AAA = true;

public bool AAA

{

    get => _AAA;

    set => SetProperty(ref _AAA, value);

}

  

private int _AAA;

public int AAA

{

    get { return _AAA; }

    set { SetProperty(ref _AAA, value); }

}

  

private ObservableCollection<int> _AAA;

public ObservableCollection<int> AAA

{

    get { return _AAA; }

    set { SetProperty(ref _AAA, value); }

}

  

private ObservableCollection<double> _AAA;

public ObservableCollection<double> AAA

{

    get => _AAA;

    set => SetProperty(ref _AAA, value);

}

```

[[回目錄]](#目錄)

# Mouse

```xml

<!-- Mouse -->

<i:Interaction.Triggers>

    <i:EventTrigger EventName="MouseRightButtonDown">

        <i:InvokeCommandAction Command="{Binding SpeedCycleCommand}"            TriggerParameterPath="x" CommandParameter="x" />

    </i:EventTrigger>

</i:Interaction.Triggers>

```

```csharp

/********************

 * Mouse Binding

 ********************/

  

```

[[回目錄]](#目錄)

# Key

```csharp

/********************

 * Key Binding

 ********************/

```

[[回目錄]](#目錄)

# ContextMenu

```xml

<!-- ContextMenu -->

<TextBox.ContextMenu>

    <ContextMenu>

        <MenuItem

            Command="{Binding GetCoorCommand}"

            CommandParameter="Z,START,SETTING"

            Header="取得設定座標" />

    </ContextMenu>

</TextBox.ContextMenu>

```

[[回目錄]](#目錄)

# ToolTip

```xml

<!-- ToolTip -->

<Button.ToolTip>

    <ToolTip>

        <TextBlock HorizontalAlignment="Left" Text="{lex:Loc MouseButton_ToolTip}" />

    </ToolTip>

</Button.ToolTip>

```

```csharp

/********************

 * Object override

 ********************/

  

public override bool Equals(Object obj)

{

}

  

public override int GetHashCode()

{

    return (base.GetHashCode() << 2) ^ z;

    return Tuple.Create(x, y).GetHashCode();

}

  

public override string ToString()

{

}

```

[[回目錄]](#目錄)

# Window 事件

```csharp

// Window Loaded

public DelegateCommand HandleLoadedCommand { get; private set; }

  

// Window Closing

public DelegateCommand HandleClosingCommand { get; private set; }

  

// 全域Save事件

public DelegateCommand WriteDataCommand { get; private set; }

  
  

// Window Loaded

HandleLoadedCommand = new DelegateCommand(HandleLoaded);

  

// Window Closing

HandleClosingCommand = new DelegateCommand(HandleClosing);

  

// 全域Save事件

WriteDataCommand = new DelegateCommand(WriteData);

ApplicationCommands.WriteCommand.RegisterCommand(WriteDataCommand);

  
  

/// <summary>

/// Window Loaded

/// </summary>

private void HandleLoaded()

{

}

  

/// <summary>

/// Window Closing

/// </summary>

private void HandleClosing()

{

}

  

/// <summary>

/// 全域Save事件

/// </summary>

private void WriteData()

{

    if (IsActive)

    {

    }

}

```

[[回目錄]](#目錄)

# 參數作業

```csharp

/********************

 * 參數作業

 ********************/

/// <summary>

/// 存回資料

/// </summary>

public void SaveData()

{

}

  

/// <summary>

/// 取得資料

/// </summary>

public void ReadData()

{

}

  
  

/// <summary>

/// 存回參數值

/// </summary>

public void SaveParameter()

{

}

  

/// <summary>

/// 取得參數值

/// </summary>

public void GetParameter()

{

}

```

# 子視窗位置儲存、不關閉只隱藏

```xml

/********************

 * 子視窗位置儲存、不關閉只隱藏

 ********************/

<mah:MetroWindow

    x:Class="OEP520G.Manual.Views.xxxxx"

        ...

    Closing="OnClosing"

    IsVisibleChanged="OnVisibleChanged"

    Loaded="OnLoaded"

    Style="{StaticResource CustomMetroWindow}">

```

```csharp

public partial class xxxxx : MetroWindow

{

    private readonly IFileService _fileService;

    private const string paraFileName = "xxxxx.json";

  

    public xxxxx(IFileService fileService)

    {

        InitializeComponent();

        _fileService = fileService;

    }

  

    private void OnLoaded(object sender, RoutedEventArgs e)

    {

        var windowCoordinate = _fileService.Read<LastWindowsStatus>(DbCommon.WorkPath, paraFileName);

        if (windowCoordinate != null)

        {

            this.Left = windowCoordinate.Left;

            this.Top = windowCoordinate.Top;

            this.AdvancePanelTiggle.IsChecked = windowCoordinate.CoorPanel; // 自己加的

        }

    }

  

    private void OnClosing(object sender, CancelEventArgs e)

    {

        if (this != null)

        {

            this.Hide();

            if (this.Owner.IsLoaded)

            {

                this.Owner.Activate();

                e.Cancel = true;

            }

        }

    }

  

    private void OnVisibleChanged(object sender, DependencyPropertyChangedEventArgs e)

    {

        if (this != null)

        {

            if (this.Visibility != Visibility.Visible)

                _fileService.Save(DbCommon.WorkPath, paraFileName, new LastWindowsStatus()

                {

                    Left = this.Left,

                    Top = this.Top,

                });

        }

    }

}

  

internal class LastWindowsStatus

{

    public double Left { get; set; }

    public double Top { get; set; }

    public bool CoorPanel { get; set; } // 自己加的

}

```

[[回目錄]](#目錄)

# .ini 檔作業

```csharp

/********************

 * .ini檔作業

 ********************/

```

[[回目錄]](#目錄)

# 檔案作業

```csharp

/********************

 * 檔案作業

 ********************/

private readonly string FileName = FileList.NOZZLE_INI;

private string sectionName;

  

/// <summary>

/// 將參數寫入參數檔

/// </summary>

/// <remarks>

/// iniFile.WriteIniFile(SectionName, "[屬性名稱]", [屬性值]));

/// </remarks>

public void WriteParameter()

{

    // 參數檔檔案名稱

    IniFile iniFile = new IniFile(FileName);

}

  

/// <summary>

/// 從參數檔讀取參數

/// </summary>

/// <remarks>

/// [屬性名稱] = [Type].Parse(iniFile.ReadIniFile(SectionName, "[屬性名稱]", "[預設值]"));

/// </remarks>

public void ReadParameter()

{

    // 參數檔檔案名稱

    IniFile iniFile = new IniFile(FileName);

}

  

/********************

 * 檔案作業

 ********************/

private const string TABLE_NAME_EPCIO6000 = "EPCIO6000" + GlobalSQLite.DB_FILE_EXT_NAME;

  

/// <summary>

/// 將參數寫入參數檔

/// </summary>

public void WriteParameter()

{

}

  

/// <summary>

/// 從參數檔讀取參數

/// </summary>

public void ReadParameter()

{

}

  
  

/********************

 *

 ********************/

private readonly Timer _scanIoTimer = new Timer { Interval = 200 };

  

_scanIoTimer.Elapsed += new ElapsedEventHandler(UpdateIoEvent);

_scanIoTimer.AutoReset = false;

  

/// <summary>

/// IO狀態更新Timer Event

/// </summary>

private void UpdateIoEvent(object sender, ElapsedEventArgs e)

{

    foreach (var nozzle in NozzleList)

        UpdateNozzleState(nozzle);

  

    foreach (var clamper in ClamperList)

        UpdateClamperState(clamper);

}

  

_scanIoTimer.Start();

  

/********************

 * 繫結

 ********************/

public bool NozzleSelect1

{

    get { return _nozzleSelect1; }

    set { SetProperty(ref _nozzleSelect1, value); }

}

private bool _nozzleSelect1;

```

```xml

<StackPanel>

    <!--  上版面 Start  -->

    <!--  上版面 End  -->

  

    <!--  下版面 Start  -->

    <!--  下版面 End  -->

</StackPanel>

  
  

<StackPanel Orientation="Horizontal">

    <!--  左版面 Start  -->

    <!--  左版面 End  -->

  

    <!--  右版面 Start  -->

    <!--  右版面 End  -->

</StackPanel>

  
  

<!--#region 上版面-->

<!--#endregion-->

  

<!--#region 下版面-->

<!--#endregion-->

  
  
  

<!--  分隔線  -->

<Rectangle />

  

<!--  垂直分隔線  -->

<Style x:Key="VerticalDivider" TargetType="{x:Type Rectangle}">

    <Setter Property="Width" Value="5" />

    <Setter Property="Height" Value="Auto" />

    <Setter Property="Margin" Value="0" />

    <Setter Property="VerticalAlignment" Value="Center" />

    <Setter Property="Fill" Value="Transparent" />

</Style>

  

<!--  水平分隔線  -->

<Style x:Key="HorizontalDivider" TargetType="{x:Type Rectangle}">

    <Setter Property="Width" Value="Auto" />

    <Setter Property="Height" Value="5" />

    <Setter Property="Margin" Value="0" />

    <Setter Property="VerticalAlignment" Value="Center" />

    <Setter Property="Fill" Value="Transparent" />

</Style>

  
  

<!--  分隔線  -->

<Border

    VerticalAlignment="Center"

    BorderBrush="LightGray"

    BorderThickness="2,2,0,0" />

  
  

<Label

    Content="點膠參數"

    FontSize="16"

    FontWeight="Bold"

    Foreground="Brown" />

  
  

<Label

    HorizontalAlignment="Left"

    Content="照射位置"

    Foreground="Brown" />

  
  

<!--  ListView  -->

<Style BasedOn="{StaticResource MahApps.Styles.ListView.Virtualized}" TargetType="ListView">

    <Setter Property="Margin" Value="0" />

</Style>

  

<Style TargetType="GridViewColumnHeader">

    <Setter Property="Margin" Value="0" />

    <Setter Property="Padding" Value="4,2,4,1" />

</Style>

  

<Style TargetType="ListViewItem">

    <Setter Property="Control.Margin" Value="5,0,5,0" />

    <Setter Property="Control.Padding" Value="10,0,10,0" />

    <Setter Property="Control.VerticalAlignment" Value="Center" />

</Style>

  
  

<GroupBox

    x:Name="GroupBox_InputIo"

    Grid.Column="1"

    Header="{lex:Loc}">

    <ListView ItemsSource="{Binding RemoteIoInputSource}">

        <ListView.View>

            <GridView AllowsColumnReorder="true">

                <GridViewColumn Width="40" Header="No">

                    <GridViewColumn.CellTemplate>

                        <DataTemplate>

                            <CheckBox IsChecked="{Binding Value, Mode=OneWay}" />

                        </DataTemplate>

                    </GridViewColumn.CellTemplate>

                </GridViewColumn>

                <GridViewColumn

                    Width="40"

                    DisplayMemberBinding="{Binding Id}"

                    Header="ID" />

                <GridViewColumn

                    Width="50"

                    DisplayMemberBinding="{Binding IoCode}"

                    Header="Code" />

                <GridViewColumn DisplayMemberBinding="{Binding Title}" Header="Title" />

            </GridView>

        </ListView.View>

    </ListView>

</GroupBox>

  
  
  

<!--  模擬GroupBox Header，搭配無標題的DataGrid當標題使用  -->

<Border Grid.Row="1" Style="{StaticResource GroupBoxWithDataGrid}">

    <DockPanel LastChildFill="True">

        <TextBlock

            x:Name="GroupBox_SensorOnMotionAxis"

            Margin="0"

            DockPanel.Dock="Top"

            Style="{StaticResource SubtitleTextSimilarGroupBoxStyle}"

            Text="{lex:Loc}" />

        <DataGrid ItemsSource="{Binding AxisSensorSource}">

            <DataGrid.Columns>

                <DataGridCheckBoxColumn

                    Binding="{Binding Value}"

                    CellStyle="{StaticResource ArrangeCenter}"

                    Header="{lex:Loc OEP520G.EPCIO6000:Resources:IoListHeader_Value}" />

                <DataGridTextColumn

                    Binding="{Binding Code}"

                    CellStyle="{StaticResource ArrangeCenter}"

                    Header="{lex:Loc OEP520G.EPCIO6000:Resources:IoListHeader_IoCode}" />

                <DataGridTextColumn

                    Binding="{Binding Title}"

                    CellStyle="{StaticResource ArrangeLeft}"

                    Header="{lex:Loc OEP520G.EPCIO6000:Resources:IoListHeader_Description}" />

            </DataGrid.Columns>

        </DataGrid>

    </DockPanel>

</Border>

<!--  模擬GroupBox Header，搭配無標題的DataGrid當標題使用  -->

  
  

<TextBox

    MinWidth="50"

    Margin="5,0,5,0"

    HorizontalContentAlignment="Center"

    VerticalContentAlignment="Center"

    Text="0" />

  
  
  

<Button

    MinWidth="100"

    MinHeight="40"

    Margin="0,20,0,0"

    HorizontalAlignment="Center"

    Content="測試" />

  
  

<ComboBox

    Width="120"

    HorizontalContentAlignment="Center"

    VerticalContentAlignment="Center"

    md:ComboBoxAssist.ClassicMode="True"

    md:ComboBoxAssist.ShowSelectedItem="True"

    Foreground="Black">

    <ComboBoxItem Content="DIS" />

</ComboBox>

  
  
  

<Style x:Key="Block" TargetType="{x:Type StackPanel}">

    <Setter Property="Margin" Value="0,7,0,0" />

    <Setter Property="VerticalAlignment" Value="Center" />

    <Setter Property="HorizontalAlignment" Value="Center" />

</Style>

  
  
  
  
  

<RadioButton

    Content="{Binding ResoultContent0}"

    GroupName="ResoultSelector"

    IsChecked="{Binding ResoultOption, Converter={StaticResource RadioButtonConverter}, ConverterParameter=0}"

    Style="{StaticResource ResoultStyle}" />

  

<Style

    x:Key="ResoultStyle"

    BasedOn="{StaticResource MaterialDesignTabRadioButton}"

    TargetType="{x:Type RadioButton}">

    <Setter Property="Width" Value="38" />

    <Setter Property="Height" Value="40" />

    <Setter Property="Margin" Value="1,0,1,0" />

    <Setter Property="Padding" Value="0" />

    <Setter Property="Cursor" Value="Hand" />

</Style>

  
  
  
  

<Style

    x:Key="CoorToggleButtonStyle"

    BasedOn="{StaticResource MaterialDesignActionToggleButton}"

    TargetType="{x:Type ToggleButton}">

    <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

    <Setter Property="Width" Value="48" />

    <Setter Property="Height" Value="48" />

    <Setter Property="Margin" Value="3" />

    <Setter Property="Padding" Value="0" />

</Style>

  

<UserControl.Resources>

    <ResourceDictionary>

        <Style

            x:Key="SelectButton"

            BasedOn="{StaticResource MaterialDesignRaisedButton}"

            TargetType="{x:Type Button}">

            <Setter Property="Width" Value="110" />

            <Setter Property="Height" Value="40" />

            <Setter Property="Margin" Value="5,3,5,3" />

        </Style>

  

        <!--  屬性欄  -->

  

            <Style x:Key="ItemUnit" TargetType="{x:Type Label}">

                <Setter Property="Width" Value="70" />

                <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

                <Setter Property="VerticalContentAlignment" Value="Center" />

                <Setter Property="HorizontalContentAlignment" Value="Left" />

                <Setter Property="DockPanel.Dock" Value="Right" />

            </Style>

  

            <Style x:Key="ItemTitle" TargetType="{x:Type Label}">

                <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

                <Setter Property="VerticalContentAlignment" Value="Center" />

                <Setter Property="HorizontalContentAlignment" Value="Right" />

                <Setter Property="DockPanel.Dock" Value="Left" />

            </Style>

  

            <Style

                x:Key="ItemField"

                BasedOn="{StaticResource MahApps.Styles.TextBox}"

                TargetType="{x:Type TextBox}">

                <Setter Property="Width" Value="140" />

                <Setter Property="Margin" Value="0,3,0,3" />

                <Setter Property="Foreground" Value="Black" />

                <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

                <Setter Property="HorizontalContentAlignment" Value="Center" />

                <Setter Property="VerticalContentAlignment" Value="Center" />

                <Setter Property="VerticalAlignment" Value="Top" />

                <Setter Property="DockPanel.Dock" Value="Right" />

            </Style>

  

            <Style

                x:Key="ItemComboBox"

                BasedOn="{StaticResource MahApps.Styles.ComboBox}"

                TargetType="{x:Type ComboBox}">

                <Setter Property="Width" Value="140" />

                <Setter Property="Margin" Value="0,3,0,3" />

                <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

                <Setter Property="HorizontalContentAlignment" Value="Center" />

                <Setter Property="VerticalContentAlignment" Value="Center" />

                <Setter Property="Foreground" Value="Black" />

                <Setter Property="DockPanel.Dock" Value="Right" />

            </Style>

  

        <Style BasedOn="{StaticResource MaterialDesignSwitchToggleButton}" TargetType="{x:Type ToggleButton}">

            <Setter Property="Margin" Value="30,1,138,1" />

            <Setter Property="DockPanel.Dock" Value="Right" />

        </Style>

  

        <Style x:Key="ItemTitle" TargetType="{x:Type Label}">

            <Setter Property="Margin" Value="0,1,0,1" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

            <Setter Property="HorizontalContentAlignment" Value="Right" />

            <Setter Property="DockPanel.Dock" Value="Left" />

        </Style>

  

        <Style x:Key="ItemInfo" TargetType="{x:Type Label}">

            <Setter Property="Width" Value="150" />

            <Setter Property="Margin" Value="0,1,9,1" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

            <Setter Property="HorizontalContentAlignment" Value="Left" />

            <Setter Property="DockPanel.Dock" Value="Right" />

        </Style>

  

        <!--  Channel選擇  -->

        <Style

            x:Key="ChannelSelectComboBox"

            BasedOn="{StaticResource MaterialDesignComboBox}"

            TargetType="{x:Type ComboBox}">

            <Setter Property="Width" Value="120" />

            <Setter Property="Margin" Value="9,0,80,0" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

            <Setter Property="md:ComboBoxAssist.ClassicMode" Value="True" />

            <Setter Property="md:ComboBoxAssist.ShowSelectedItem" Value="True" />

            <Setter Property="Foreground" Value="Black" />

            <Setter Property="DockPanel.Dock" Value="Right" />

        </Style>

  

        <Style x:Key="ChannelSelectLabel" TargetType="{x:Type Label}">

            <Setter Property="Margin" Value="0,1,9,1" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

            <Setter Property="HorizontalContentAlignment" Value="Right" />

            <Setter Property="DockPanel.Dock" Value="Right" />

        </Style>

  

        <!--  框  -->

  

        <Style TargetType="{x:Type Border}">

            <Setter Property="Margin" Value="0,9,0,0" />

            <Setter Property="Padding" Value="5" />

            <Setter Property="BorderBrush" Value="{StaticResource PrimaryHueMidBrush}" />

            <Setter Property="BorderThickness" Value="1" />

            <Setter Property="CornerRadius" Value="5" />

        </Style>

  

        <Style x:Key="DataGridBorder"  TargetType="{x:Type Border}">

            <Setter Property="Margin" Value="9" />

            <Setter Property="Padding" Value="1.5" />

            <Setter Property="BorderBrush" Value="LightGray" />

            <Setter Property="BorderThickness" Value="2" />

            <Setter Property="CornerRadius" Value="5" />

        </Style>

  

        <Style x:Key="Gap" TargetType="{x:Type Border}">

            <Setter Property="Margin" Value="8" />

            <Setter Property="BorderThickness" Value="0" />

        </Style>

    </ResourceDictionary>

</UserControl.Resources>

  
  

<UserControl.Resources>

    <ResourceDictionary>

  

        <!--  GroupBox  -->

        <Style BasedOn="{StaticResource MahApps.Styles.GroupBox}" TargetType="{x:Type GroupBox}">

            <Setter Property="Margin" Value="10,5,10,5" />

        </Style>

  

        <!--  StackPanel  -->

        <Style x:Key="NozzleSelect_StackPanel" TargetType="{x:Type StackPanel}">

            <Setter Property="Margin" Value="4,1,0,1" />

            <Setter Property="Orientation" Value="Horizontal" />

        </Style>

  

        <!--  CheckBox  -->

  

        <Style BasedOn="{StaticResource MaterialDesignCheckBox}" TargetType="{x:Type CheckBox}">

            <Setter Property="HorizontalContentAlignment" Value="Center" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

            <Setter Property="md:CheckBoxAssist.CheckBoxSize" Value="20" />

        </Style>

  

        <!--  Label  -->

        <Style x:Key="NozzleSelect_Label" TargetType="{x:Type Label}">

            <Setter Property="Width" Value="60" />

            <Setter Property="HorizontalContentAlignment" Value="Left" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

        </Style>

  

        <!--  ComboBox  -->

  

        <Style BasedOn="{DynamicResource MahApps.Styles.ComboBox.Virtualized}" TargetType="{x:Type ComboBox}">

            <Setter Property="Width" Value="110" />

            <Setter Property="Height" Value="30" />

            <Setter Property="Margin" Value="0,4" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

        </Style>

  

        <Style BasedOn="{StaticResource MaterialDesignComboBox}" TargetType="{x:Type ComboBox}">

            <Setter Property="Foreground" Value="Black" />

            <Setter Property="Width" Value="100" />

            <Setter Property="Margin" Value="0,0,4,0" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

            <Setter Property="md:ComboBoxAssist.ClassicMode" Value="True" />

            <Setter Property="md:ComboBoxAssist.ShowSelectedItem" Value="True" />

        </Style>

  

        <!--  Button  -->

  

        <Style BasedOn="{StaticResource MaterialDesignRaisedButton}" TargetType="{x:Type Button}">

            <Setter Property="Width" Value="110" />

            <Setter Property="Height" Value="40" />

            <Setter Property="Margin" Value="5" />

            <Setter Property="FontWeight" Value="Regular" />

        </Style>

  

        <Style

            x:Key="NozzleSelect_VisualButton"

            BasedOn="{StaticResource MaterialDesignRaisedButton}"

            TargetType="{x:Type Button}">

            <Setter Property="FontWeight" Value="Regular" />

            <Setter Property="Width" Value="65" />

            <Setter Property="Height" Value="26" />

            <Setter Property="Margin" Value="6,0,0,0" />

            <Setter Property="Padding" Value="0" />

        </Style>

  

        <Style

            x:Key="NozzleSelect_ActionButton"

            BasedOn="{StaticResource MaterialDesignRaisedButton}"

            TargetType="{x:Type Button}">

            <Setter Property="FontWeight" Value="Regular" />

            <Setter Property="Width" Value="35" />

            <Setter Property="Height" Value="26" />

            <Setter Property="Margin" Value="4,0,0,0" />

            <Setter Property="Padding" Value="0" />

        </Style>

  

        <!--  ToggleButton  -->

  

        <Style BasedOn="{StaticResource MaterialDesignActionToggleButton}" TargetType="{x:Type ToggleButton}">

            <Setter Property="MinWidth" Value="80" />

            <Setter Property="Height" Value="35" />

            <Setter Property="Margin" Value="0,2,0,2" />

            <Setter Property="Padding" Value="0" />

            <Setter Property="IsChecked" Value="True" />

        </Style>

  

        <!--  DataGridComboBoxColumn  -->

  

        <DataGridTemplateColumn MinWidth="100" Header="State">

            <DataGridTemplateColumn.CellTemplate>

                <DataTemplate>

                    <TextBlock Style="{StaticResource DataGrid_TextBlock}" Text="{Binding ConfigureHead, Mode=OneWay, UpdateSourceTrigger=PropertyChanged}" />

                </DataTemplate>

            </DataGridTemplateColumn.CellTemplate>

            <DataGridTemplateColumn.CellEditingTemplate>

                <DataTemplate>

                    <ComboBox

                        Background="White"

                        ItemsSource="{Binding Source={converters:EnumBindingSource {x:Type const_Local:ConfigureHeadType}}}"

                        SelectedItem="{Binding ConfigureHead}"

                        Style="{StaticResource DataGrid_ComboBox}" />

                </DataTemplate>

            </DataGridTemplateColumn.CellEditingTemplate>

        </DataGridTemplateColumn>

  
  

        <DataGridComboBoxColumn

            x:Name="DataGrid_PartsId"

            Header="{lex:Loc OEP520G.AutomaticOperation:Resources:DataGrid_PartsId_Header}"

            ItemsSource="{Binding Source={converters:EnumBindingSource {x:Type const_Local:ConfigureHeadType}}}"

            SelectedValueBinding="{Binding ConfigureHead}" />

  
  

        <DataGridComboBoxColumn

            DisplayMemberPath="Name"

            Header="Artist"

            ItemsSource="{x:Static models:SampleData.Artists}"

            SelectedItemBinding="{Binding Artist}" />

  
  
  

        <!--  DataGrid  -->

  

        <Style BasedOn="{StaticResource MahApps.Styles.DataGrid.Azure}" TargetType="{x:Type DataGrid}">

            <Setter Property="Margin" Value="0" />

            <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

            <Setter Property="AutoGenerateColumns" Value="False" />

            <Setter Property="CanUserAddRows" Value="False" />

            <Setter Property="CanUserDeleteRows" Value="False" />

            <Setter Property="IsReadOnly" Value="True" />

            <Setter Property="SelectionMode" Value="Extended" />

            <Setter Property="SelectionUnit" Value="FullRow" />

            <Setter Property="BorderBrush" Value="{StaticResource PrimaryHueDarkBrush}" />

            <Setter Property="BorderThickness" Value="1" />

            <Setter Property="GridLinesVisibility" Value="Horizontal" />

        </Style>

  

        <Style BasedOn="{StaticResource MahApps.Styles.DataGridColumnHeader}" TargetType="{x:Type DataGridColumnHeader}">

            <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

            <Setter Property="FontWeight" Value="Regular" />

            <Setter Property="Foreground" Value="Brown" />

            <Setter Property="mah:ControlsHelper.ContentCharacterCasing" Value="Normal" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

        </Style>

  

        <Style

            x:Key="ArrangeCenter"

            BasedOn="{StaticResource MahApps.Styles.DataGridCell}"

            TargetType="{x:Type DataGridCell}">

            <Setter Property="Padding" Value="2,6" />

            <Setter Property="TextBlock.FontSize" Value="{StaticResource SmallFontSize}" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

            <Setter Property="VerticalContentAlignment" Value="Center" />

        </Style>

  

        <Style

            x:Key="ArrangeRight"

            BasedOn="{StaticResource ArrangeCenter}"

            TargetType="{x:Type DataGridCell}">

            <Setter Property="Padding" Value="6,0" />

            <Setter Property="HorizontalContentAlignment" Value="Right" />

        </Style>

  
  
  

        <!--  DataGrid  -->

  

        <Style

            x:Key="BaseDataGridStyle"

            BasedOn="{StaticResource MaterialDesignDataGrid}"

            TargetType="{x:Type DataGrid}">

            <Setter Property="AutoGenerateColumns" Value="False" />

            <Setter Property="IsReadOnly" Value="True" />

            <Setter Property="Margin" Value="10,0,10,10" />

            <Setter Property="Padding" Value="1,2,1,2" />

  

            <Setter Property="CanUserAddRows" Value="False" />

            <Setter Property="CanUserDeleteRows" Value="False" />

            <Setter Property="CanUserResizeRows" Value="False" />

  

            <Setter Property="CanUserSortColumns" Value="True" />

            <Setter Property="CanUserResizeColumns" Value="False" />

            <Setter Property="CanUserReorderColumns" Value="False" />

  

            <Setter Property="SelectionMode" Value="Single" />

            <Setter Property="SelectionUnit" Value="FullRow" />

  

            <Setter Property="md:DataGridAssist.CellPadding" Value="6 6 6 6" />

            <Setter Property="md:DataGridAssist.ColumnHeaderPadding" Value="6 6 6 6" />

            <Setter Property="Background" Value="{DynamicResource MahApps.Brushes.ThemeBackground}" />

            <Setter Property="BorderBrush" Value="{StaticResource PrimaryHueMidBrush}" />

            <Setter Property="BorderThickness" Value="1" />

            <Setter Property="GridLinesVisibility" Value="Horizontal" />

            <Setter Property="HorizontalGridLinesBrush" Value="Gainsboro" />

            <Setter Property="VerticalGridLinesBrush" Value="Gainsboro" />

            <Setter Property="AlternatingRowBackground" Value="LightGreen" />

        </Style>

  

        <Style TargetType="{x:Type DataGridColumnHeader}">

            <Setter Property="FontSize" Value="{StaticResource SmallFontSize}" />

            <Setter Property="Foreground" Value="Brown" />

            <Setter Property="FontWeight" Value="Regular" />

            <Setter Property="HorizontalContentAlignment" Value="Center" />

        </Style>

  

        <Style BasedOn="{StaticResource MaterialDesignDataGridCell}" TargetType="{x:Type DataGridCell}">

            <Setter Property="TextBlock.FontSize" Value="{StaticResource SmallFontSize}" />

            <Setter Property="TextBlock.VerticalAlignment" Value="Center" />

            <Setter Property="TextBlock.TextAlignment" Value="Left" />

        </Style>

  

    </ResourceDictionary>

</UserControl.Resources>

  
  
  

<Border Style="{StaticResource DataGridBorder}">

    <DataGrid Name="FlyingCameraDgv">

        <DataGrid.Columns>

            <DataGridTextColumn

                Binding="{Binding NozzleNo}"

                Header="吸嘴"

                IsReadOnly="True" />

            <DataGridTextColumn

                Binding="{Binding ShiftX, StringFormat='0.000'}"

                Header="X位移" />

            <DataGridTextColumn

                Binding="{Binding ShiftY}"

                Header="Y位移" />

            <DataGridTextColumn

                Binding="{Binding NewX}"

                Header="新X" />

            <DataGridTextColumn

                Binding="{Binding NewY}"

                CanUserSort="False"

                Header="新Y" />

            <DataGridCheckBoxColumn

                Binding="{Binding IsUpdate}"

                EditingElementStyle="{StaticResource MaterialDesignDataGridCheckBoxColumnEditingStyle}"

                ElementStyle="{StaticResource MaterialDesignDataGridCheckBoxColumnStyle}"

                Header="更新" />

        </DataGrid.Columns>

    </DataGrid>

</Border>

  
  
  

<Border Style="{StaticResource DataGridBorder}">

    <DataGrid

        x:Name="TrayDgv"

        DockPanel.Dock="Bottom"

        ItemsSource="{Binding TraySource}"

        SelectedIndex="{Binding SelectedIndex}"

        SelectedItem="{Binding SelectedItem}">

        <DataGrid.Columns>

            <DataGridTextColumn

                MinWidth="200"

                Binding="{Binding Name}"

                Header="托盤資料ID" />

            <DataGridTextColumn

                MinWidth="250"

                Binding="{Binding Memo}"

                Header="註解" />

        </DataGrid.Columns>

    </DataGrid>

</Border>

  

<Style BasedOn="{StaticResource MaterialDesignDataGrid}" TargetType="{x:Type DataGrid}">

    <Setter Property="md:DataGridAssist.CellPadding" Value="6 2 6 2" />

    <Setter Property="md:DataGridAssist.ColumnHeaderPadding" Value="6 2 6 2" />

    <Setter Property="Margin" Value="0" />

    <Setter Property="Background" Value="WhiteSmoke" />

    <Setter Property="BorderThickness" Value="0" />

    <Setter Property="AutoGenerateColumns" Value="False" />

    <Setter Property="CanUserAddRows" Value="False" />

    <Setter Property="CanUserDeleteRows" Value="False" />

    <Setter Property="CanUserReorderColumns" Value="False" />

    <Setter Property="CanUserResizeColumns" Value="False" />

    <Setter Property="CanUserResizeRows" Value="False" />

    <Setter Property="CanUserSortColumns" Value="True" />

    <Setter Property="IsReadOnly" Value="True" />

    <Setter Property="GridLinesVisibility" Value="All" />

    <Setter Property="HorizontalGridLinesBrush" Value="Gainsboro" />

    <Setter Property="VerticalGridLinesBrush" Value="Gainsboro" />

    <Setter Property="SelectionMode" Value="Single" />

    <Setter Property="SelectionUnit" Value="FullRow" />

</Style>

  

<Style BasedOn="{StaticResource MaterialDesignDataGridColumnHeader}" TargetType="{x:Type DataGridColumnHeader}">

    <Setter Property="FontSize" Value="14" />

    <Setter Property="Foreground" Value="Brown" />

    <Setter Property="FontWeight" Value="Regular" />

    <Setter Property="HorizontalAlignment" Value="Center" />

</Style>

  

<Style BasedOn="{StaticResource MaterialDesignDataGridCell}" TargetType="{x:Type DataGridCell}">

    <Setter Property="TextBlock.FontSize" Value="14" />

    <Setter Property="TextBlock.VerticalAlignment" Value="Center" />

    <Setter Property="TextBlock.TextAlignment" Value="Left" />

</Style>

```

```csharp

DataTable dt;

DataRow row;

  

dt = new DataTable();

dt.Columns.Add("No", typeof(long));

dt.Columns.Add("MachineId", typeof(string));

dt.Columns.Add("ProductId", typeof(string));

dt.Columns.Add("StartTime", typeof(string));

dt.Columns.Add("StopTime", typeof(string));

dt.Columns.Add("CycleCount", typeof(long));

dt.Columns.Add("PickCount", typeof(long));

dt.Columns.Add("AbandonCount", typeof(long));

dt.Columns.Add("AbandonRate", typeof(double));

dt.Columns.Add("CycleTime", typeof(double));

  

dt = new DataTable();

dt.Columns.Add("No", typeof(int));

dt.Columns.Add("IsEnable", typeof(bool));

dt.Columns.Add("AbandonBoxName", typeof(string));

dt.Columns.Add("MinX", typeof(double));

dt.Columns.Add("MaxX", typeof(double));

dt.Columns.Add("MinY", typeof(double));

dt.Columns.Add("MaxY", typeof(double));

dt.Columns.Add("AbandonHigh", typeof(double));

  

row = dt.NewRow();

row["No"] = 1;

row["IsEnable"] = true;

row["AbandonBoxName"] = "Auto Run";

row["MinX"] = 0.0;

row["MaxX"] = 0.0;

row["MinY"] = 0.0;

row["MaxY"] = 0.0;

row["AbandonHigh"] = 5.0;

dt.Rows.Add(row);

  

ProdictionDataGrid.ItemsSource = dt.DefaultView;

  
  
  
  

Random rnd = new Random();

  

row["ShiftX"] = (double)(rnd.Next(-20 * speed, -20 * (speed - 1))) / 1000;

row["ShiftY"] = (double)(rnd.Next(-20 * speed, -20 * (speed - 1))) / 1000;

row["MirrorX"] = (double)(rnd.Next(-20 * speed, -20 * (speed - 1))) / 1000;

row["MirrorY"] = (double)(rnd.Next(-20 * speed, -20 * (speed - 1))) / 1000;

  
  

/********************

 *

 ********************/

  
  

LocalizationProvider.GetValue<string>("");

```

```xml

<ComboBox

    x:Name="DataNameIdAtComboBox"

    DisplayMemberPath="Value"

    ItemsSource="{Binding DataContext.WfsDataNameList, RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type DataGrid}}}"

    SelectedValue="{Binding Path=DataNameId}"

    SelectedValuePath="Key"

    Style="{StaticResource ParaNameSelector}" />

  

<TextBox

    Grid.Row="1"

    Grid.Column="1"

    Style="{StaticResource Coordinate}"

    Text="{Binding RelativeSource={x:Static RelativeSource.Self}, Path=DataContext.MachineDatas.DatumPointX, Mode=TwoWay, StringFormat=F3}" />

  

<RadioButton

    Command="{Binding RibbonPageNavigateCommand}"

    CommandParameter="Teaching"

    Content="{lex:Loc WFS:Resources:MainMenu_Teaching}"

    DockPanel.Dock="Left"

    GroupName="MainMenu"

    IsEnabled="{Binding Source={x:Static SysFuncs:SystemSwitcher.Instance}, Path=SystemReady, Mode=OneWay}"

    Style="{StaticResource WFS.MainMenuRadioButton}" />

  

<!--  暗色  -->

<RadioButton

    Content="{lex:Loc WFS.Core:Resources:Caption_Theme_DarkTheme}"

    GroupName="AppTheme"

    IsChecked="{Binding Source={x:Static sysModels:SysParameters.Instance}, Path=SystemSetting.AppTheme, Mode=TwoWay, Converter={StaticResource EnumToBooleanConverter}, ConverterParameter={x:Static sysModels:AppThemeList.Dark}}">

    <i:Interaction.Triggers>

        <i:EventTrigger EventName="Checked">

            <i:InvokeCommandAction Command="{Binding SetThemeCommand}" CommandParameter="Dark" />

        </i:EventTrigger>

    </i:Interaction.Triggers>

</RadioButton>

```

```csharp

/********************

 * 取消權杖

 ********************/

  

private CancellationTokenSource _cts;

private CancellationToken _token;

  

if (_cts != null)

    _cts.Dispose();

_cts = new();

_token = _cts.Token;

  
  

// Cancel

if (_cts != null && !_cts.IsCancellationRequested)

    _cts.Cancel();

  
  
  

// 實作1

_cts.Dispose();

_cts = new CancellationTokenSource();

_token = _cts.Token;

  

await Task.Run(async () =>

{

    try

    {

        // 作業是否取消

        if (_token.IsCancellationRequested)

            _token.ThrowIfCancellationRequested();

  

    }

    catch (OperationCanceledException ex)

    {

    }

    catch

    {

    }

    finally

    {

    }

}, _token).ContinueWith(x =>

{

});

  
  

/********************

 * Singleton

 ********************/

private xxxClass() { }

private static readonly Lazy<xxxClass> _instance = new Lazy<xxxClass>(() => new xxxClass());

public static xxxClass Instance => _instance.Value;

  
  

// Singleton單例模式

private xxxClass() { }

public static xxxClass Instance => Nested.instance;

private class Nested

{

    static Nested() { }

    internal static readonly xxxClass instance = new xxxClass();

}

  
  

/********************

 * Singleton & INotifyPropertyChanged

 ********************/

private xxxClass() { }

private static readonly Lazy<xxxClass> _instance = new Lazy<xxxClass>(() => new xxxClass());

public static xxxClass Instance => _instance.Value;

  

public event PropertyChangedEventHandler PropertyChanged;

private void OnPropertyChanged([CallerMemberName] string name = null)

{

    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));

}

  
  

OnPropertyChanged();

  
  

/********************

 * OEP520G

 ********************/

  

private readonly IRegionManager         _regionManager          ;

private readonly IThemeSelectorService  _themeSelectorService   ;

private readonly IEventAggregator       _ea                     ;

private readonly IFileService           _fileService            ;

private readonly ISqliteService         _sqlite                 ;

private readonly IDialogService         _dialogService          ;

private readonly ICrudDialog            _crudDialog             ;

private readonly IStatusBar             _statusBar              ;

private readonly IOepSystem             _oepSystem              ;

private readonly IAuthority             _authority              ;

private readonly IProductManager        _pm                     ;

private readonly IProductCrud           _pmCrudDialog           ;

  

private readonly IServo                 _servo                  ;

private readonly IIo                    _io                     ;

private readonly IVisionCommunication   _visionComm             ;

private readonly IImageProcessing       _ip                     ; x

private readonly ICamera                _camera                 ; x

private readonly ILighting              _lighting               ; x

  

private readonly IDatumPoint            _dp                     ;

private readonly IMachineInfo           _machineInfo            ;

private readonly IEpr                   _epr                    ;

private readonly INozzle                _nozzle                 ;

private readonly IStage                 _stage                  ;

private readonly IClamper               _clamper                ;

private readonly IFeeder                _feeder                 ;

private readonly IShape                 _shape                  ; x

private readonly IDispensing            _dispensing             ;

  

private readonly IMechanismMotion       _mechanismMotion        ;

private readonly IMachineFunction       _machineFunction        ;

private readonly IOnflyOffset           _onflyOffset            ;

private readonly IDetectHeight          _detectHeight           ;

private readonly ITeaching              _teaching               ;

private readonly IAutomaticOperation    _auto                   ;

  
  
  

IRegionManager          regionManager           ,

IThemeSelectorService   themeSelectorService    ,

IApplicationCommands    applicationCommands     ,

IEventAggregator        ea                      ,

IFileService            fileService             ,

ISqliteService          sqlite                  ,

IDialogService          dialogService           ,

ICrudDialog             crudDialog              ,

IStatusBar              statusBar               ,

IOepSystem              oepSystem               ,

IAuthority              authority               ,

IProductManager         pm                      ,

IProductCrud            pmCrudDialog            ,

  

IServo                  servo                   ,

IIo                     io                      ,

IVisionCommunication    visionComm              ,

IImageProcessing        ip                      , x

ICamera                 camera                  , x

ILighting               lighting                , x

  

IDatumPoint             dp                      ,

IMachineInfo            machineInfo             ,

IEpr                    epr                     ,

INozzle                 nozzle                  ,

IStage                  stage                   ,

IClamper                clamper                 ,

IFeeder                 feeder                  ,

IShape                  shape                   , x

IDispensing             dispensing              ,

  

IMechanismMotion        mechanismMotion         ,

IMachineFunction        machineFunction         ,

IOnflyOffset            onflyOffset             ,

IDetectHeight           detectHeight            ,

ITeaching               teaching                ,

IAutomaticOperation     auto                    ,

  
  
  

_regionManager          =   regionManager           ;

_themeSelectorService   =   themeSelectorService    ;

_ea                     =   ea                      ;

_fileService            =   fileService             ;

_sqlite                 =   sqlite                  ;

_dialogService          =   dialogService           ;

_crudDialog             =   crudDialog              ;

_statusBar              =   statusBar               ;

_oepSystem              =   oepSystem               ;

_authority              =   authority               ;

_pm                     =   pm                      ;

_pmCrudDialog           =   pmCrudDialog            ;

  

_servo                  =   servo                   ;

_io                     =   io                      ;

_visionComm             =   visionComm              ;

_ip                     =   ip                      ; x

_lighting               =   lighting                ; x

  

_dp                     =   dp                      ;

_machineInfo            =   machineInfo             ;

_epr                    =   epr                     ;

_nozzle                 =   nozzle                  ;

_stage                  =   stage                   ;

_clamper                =   clamper                 ;

_feeder                 =   feeder                  ;

_shape                  =   shape                   ; x

_dispensing             =   dispensing              ;

  

_mechanismMotion        =   mechanismMotion         ;

_machineFunction        =   machineFunction         ;

_onflyOffset            =   onflyOffset             ;

_detectHeight           =   detectHeight            ;

_teaching               =   teaching                ;

_auto                   =   auto                    ;

  
  

/********************

 * WFS

 ********************/

  

xmlns:sysModels="clr-namespace:WFS.Components.Systems.Models;assembly=WFS.Components"

IsEnabled="{Binding Source={x:Static sysModels:SystemStateHandle.Instance}, Path=SystemStationary, Mode=OneWay}"

  
  

private readonly SystemStateHandle _sysState;

private readonly WfsParameters _wfsParas;

private readonly BigDataContent _bdContent;

  

_sysState = SystemStateHandle.Instance;

_wfsParas = WfsParameters.Instance;

_bdContent = BigDataContent.Instance;

  
  

private readonly IRegionManager         _regionManager          ;

private readonly IThemeSelectorService  _themeSelectorService   ;

private readonly IEventAggregator       _ea                     ;

private readonly ISqliteService         _sqlite                 ;

private readonly IFileService           _fileService            ;

private readonly IPrismMessageBox       _prismMessageBox        ;

private readonly ICrudDialog            _crudDialog             ;

private readonly ISystem                _sys                    ;

private readonly ISystemMessenger       _sysMessenger           ;

private readonly IAuthority             _authority              ;

private readonly IMachine               _machine                ;

private readonly IProductManager        _pm                     ;

private readonly ITray                  _tray                   ;

private readonly IWfs                   _wfs                    ;

private readonly IPlc                   _plc                    ;

private readonly IVision                _vision                 ;

private readonly IDisplacement          _displacement           ;

private readonly IMeasuring             _measuring              ;

private readonly IBigData               _bigData                ;

  
  
  

IRegionManager          regionManager           ,

IThemeSelectorService   themeSelectorService    ,

IApplicationCommands    applicationCommands     ,

IEventAggregator        ea                      ,

ISqliteService          sqliteService           ,

IFileService            fileService             ,

IPrismMessageBox        prismMessageBox         ,

ICrudDialog             crudDialog              ,

ISystem                 sys                     ,

ISystemMessenger        sysMessenger            ,

IAuthority              authority               ,

IMachine                machine                 ,

IProductManager         pm                      ,

ITray                   tray                    ,

IWfs                    wfs                     ,

IPlc                    plc                     ,

IVision                 vision                  ,

IDisplacement           displacement            ,

IMeasuring              measuring               ,

IBigData                bigData                 ,

  
  

_regionManager          =   regionManager           ;

_themeSelectorService   =   themeSelectorService    ;

_ea                     =   ea                      ;

_sqlite                 =   sqliteService           ;

_fileService            =   fileService             ;

_prismMessageBox        =   prismMessageBox         ;

_crudDialog             =   crudDialog              ;

_sys                    =   sys                     ;

_sysMessenger           =   sysMessenger            ;

_authority              =   authority               ;

_machine                =   machine                 ;

_pm                     =   pm                      ;

_tray                   =   tray                    ;

_wfs                    =   wfs                     ;

_plc                    =   plc                     ;

_vision                 =   vision                  ;

_displacement           =   displacement            ;

_measuring              =   measuring               ;

_bigData                =   bigData                 ;

```

---

# DataTrigger (XAML)

```xml

<DataGridTextColumn

    Binding="{Binding Path=Numbers[00]}"

    Header="01">

    <DataGridTextColumn.CellStyle>

        <Style

            BasedOn="{StaticResource NumberStyle}"

            TargetType="{x:Type DataGridCell}">

            <Style.Triggers>

                <DataTrigger

                    Binding="{Binding Path=Type[00]}"

                    Value="{x:Static constants:WaterfallType.WinningNumber}">

                    <Setter Property="Background" Value="DarkBlue" />

                    <Setter Property="Foreground" Value="Yellow" />

                </DataTrigger>

                <DataTrigger

                    Binding="{Binding Path=Type[00]}"

                    Value="{x:Static constants:WaterfallType.BonusNumber}">

                    <Setter Property="Background" Value="DarkRed" />

                    <Setter Property="Foreground" Value="Yellow" />

                </DataTrigger>

                <DataTrigger

                    Binding="{Binding Path=Type[00]}"

                    Value="{x:Static constants:WaterfallType.NoDraw}">

                    <Setter Property="Foreground" Value="LightGray" />

                </DataTrigger>

                <DataTrigger

                    Binding="{Binding Path=Type[00]}"

                    Value="{x:Static constants:WaterfallType.NoDrawNear}">

                    <Setter Property="Background" Value="PowderBlue" />

                    <Setter Property="Foreground" Value="Green" />

                </DataTrigger>

                <DataTrigger

                    Binding="{Binding Path=Type[00]}"

                    Value="{x:Static constants:WaterfallType.NoDrawFar}">

                    <Setter Property="Background" Value="PeachPuff" />

                    <Setter Property="Foreground" Value="Crimson" />

                </DataTrigger>

            </Style.Triggers>

        </Style>

    </DataGridTextColumn.CellStyle>

</DataGridTextColumn>

```
