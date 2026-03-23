---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

/********************

 * System Commands

 ********************/

public DelegateCommand LoadedCommand

    => _LoadedCommand ?? (_LoadedCommand = new DelegateCommand(ExecuteLoadedCommand));

// 視窗載入

private DelegateCommand _LoadedCommand;

public DelegateCommand LoadedCommand

    => _LoadedCommand ??= new DelegateCommand(ExecuteLoadedCommand);

private void ExecuteLoadedCommand()

{

}

// 視窗卸載

private DelegateCommand _UnloadedCommand;

public DelegateCommand UnloadedCommand

    => _UnloadedCommand ??= new DelegateCommand(ExecuteUnloadedCommand);

private void ExecuteUnloadedCommand()

{

}

// 視窗顯示/隱藏狀態變更

private DelegateCommand<object> _VisibleChangedCommand;

public DelegateCommand<object> VisibleChangedCommand

    => _VisibleChangedCommand ??= new DelegateCommand<object>(ExecuteVisibleChangedCommand);

private void ExecuteVisibleChangedCommand(object visibility)

{

    if (visibility != null && visibility is DependencyPropertyChangedEventArgs args)

    {

        if ((Visibility)args.NewValue == Visibility.Visible)

        {

        }

        else

        {

        }

    }

}

//

// 視窗載入

//

private DelegateCommand _loadedCommand;

public DelegateCommand LoadedCommand

    => _loadedCommand ?? (_loadedCommand = new DelegateCommand(ExecuteLoadedCommand));

private void ExecuteLoadedCommand()

{

}

//

// 視窗卸載

//

private DelegateCommand _unloadedCommand;

public DelegateCommand UnloadedCommand

    => _unloadedCommand ?? (_unloadedCommand = new DelegateCommand(ExecuteUnloadedCommand));

private void ExecuteUnloadedCommand()

{

}

//

// 頁面載入

//

private DelegateCommand _loadedCommand;

public DelegateCommand LoadedCommand

    => _LoadedCommand ??= new DelegateCommand(ExecuteLoadedCommand);

private void ExecuteLoadedCommand()

{

    // 調整所有 ToolTip 顯示行為

    ToolTipService.ShowDurationProperty.OverrideMetadata(typeof(DependencyObject), new FrameworkPropertyMetadata(Int32.MaxValue));

    ToolTipService.InitialShowDelayProperty.OverrideMetadata(typeof(DependencyObject), new FrameworkPropertyMetadata(2));

    ToolTipService.BetweenShowDelayProperty.OverrideMetadata(typeof(DependencyObject), new FrameworkPropertyMetadata(0));

}

//

// 頁面卸載

//

private DelegateCommand _unloadedCommand;

public DelegateCommand UnloadedCommand

    => _unloadedCommand ?? (_unloadedCommand = new DelegateCommand(ExecuteUnloadedCommand));

private void ExecuteUnloadedCommand()

{

}

//////////////////////////////////////////////

using OEP520G.Core;

using OEP520G.Parameter;

using Prism;

using Prism.Commands;

using Prism.Mvvm;

using System;

using System.Collections.Generic;

using System.Linq;

public class xxxViewModel : BindableBase, IActiveAware, INavigationAware

/********************

 * INavigationAware

 ********************/

public void OnNavigatedTo(NavigationContext navigationContext)

{

}

public void OnNavigatedFrom(NavigationContext navigationContext)

{

}

public bool IsNavigationTarget(NavigationContext navigationContext) => true;

/********************

 * IActiveAware

 ********************/

private bool _isActive = false;

public bool IsActive

{

    get { return _isActive; }

    set

    {

        _isActive = value;

        IsActiveChanged?.Invoke(this, new EventArgs());

    }

}

public event EventHandler IsActiveChanged;

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

/// 將 Model 資料寫入資料庫

/// 將 View 資料寫入 Model 及資料庫

/// </summary>

public void ExecuteSaveDataCommand() => WriteToDb();

/********************

 * Database

 ********************/

/// <summary>

/// 將 Model 資料寫入資料庫

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

/// 將 View 資料存回 Model

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

/// 從資料庫讀取資料至 Model

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

/// 從 Model 載入資料至 View

/// </summary>

public bool LoadData()

{

    if (!_pm.IsProductActive)

        return;

    if (IsActive)

    {

    }

}

/********************

 * Database

 ********************/

/// <summary>

/// 將 Model 資料寫入資料庫

/// 將 View 資料寫入 Model 及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

bool WriteToDb();

/// <summary>

/// 從資料庫讀取資料至 Model

/// 從資料庫讀取資料至 Model 及 View

/// </summary>

/// <returns>是否讀取成功</returns>

bool ReadFromDb();

/// <summary>

/// 將 Model 資料寫入資料庫

/// 將 View 資料寫入 Model 及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

public bool WriteData()

{

}

/// <summary>

/// 從資料庫讀取資料至 Model 及 View

/// </summary>

/// <returns>是否讀取成功</returns>

public bool ReadData()

{

}

/********************

 * Database

 ********************/

/// <summary>

/// 將 Model 資料寫入資料庫

/// 將 View 資料寫入 Model 及資料庫

/// </summary>

/// <returns>是否寫入成功</returns>

private bool SaveData()

{

    if (IsActive)

    {

    }

}

/// <summary>

/// 從 Model 載入資料至 View

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

        Console.WriteLine($" 資料庫 '{_pm.DB_NAME_PRODUCT}' 開啟失敗！");

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

        Console.WriteLine($"Stage 資料庫 (旋轉中心/旋轉位移) 寫入失敗！");

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

        Console.WriteLine($" 資料庫 '{_pm.DB_NAME_PRODUCT}' 開啟失敗！");

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

        Console.WriteLine("Camera 資料庫載入失敗！");

        return false;

    }

    tran.Commit();

    return true;

}

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

/********************

 * Data Binding

 ********************/

// 是否允許執行 ApplyCommand

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

/********************

 * Mouse Binding

 ********************/

/********************

 * Key Binding

 ********************/

/********************

 * ContextMenu

 ********************/

<TextBox.ContextMenu>

    <ContextMenu>

        <MenuItem

            Command="{Binding GetCoorCommand}"

            CommandParameter="Z,START,SETTING"

            Header=" 取得設定座標 " />

    </ContextMenu>

</TextBox.ContextMenu>

/********************

 * Mouse

 ********************/

<i:Interaction.Triggers>

    <i:EventTrigger EventName="MouseRightButtonDown">

        <i:InvokeCommandAction Command="{Binding SpeedCycleCommand}" TriggerParameterPath="x" CommandParameter="x" />

    </i:EventTrigger>

</i:Interaction.Triggers>

/********************

 * ToolTip

 ********************/

<Button.ToolTip>

    <ToolTip>

        <TextBlock HorizontalAlignment="Left" Text="{lex:Loc MouseButton_ToolTip}" />

    </ToolTip>

</Button.ToolTip>

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

//////////////////////////////////////////////

// Window Loaded

public DelegateCommand HandleLoadedCommand { get; private set; }

// Window Closing

public DelegateCommand HandleClosingCommand { get; private set; }

// 全域 Save 事件

public DelegateCommand WriteDataCommand { get; private set; }

// Window Loaded

HandleLoadedCommand = new DelegateCommand(HandleLoaded);

// Window Closing

HandleClosingCommand = new DelegateCommand(HandleClosing);

// 全域 Save 事件

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

/// 全域 Save 事件

/// </summary>

private void WriteData()

{

    if (IsActive)

    {

    }

}

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

public partial class xxxxx : MetroWindow

{

    private readonly IIoService _io;

    private const string paraFileName = "xxxxx.json";

    public xxxxx(IIoService io)

    {

        InitializeComponent();

        _io = io;

    }

    private void OnLoaded(object sender, RoutedEventArgs e)

    {

        var windowCoordinate = _io.Read<LastWindowsStatus>(DbCommon.WorkPath, paraFileName);

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

                _io.Save(DbCommon.WorkPath, paraFileName, new LastWindowsStatus()

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

/********************

 * .ini 檔作業

 ********************/

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

/// IO 狀態更新 Timer Event

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

    set { SetProperty(ref _nozzleSelect1, value); }}

private bool _nozzleSelect1;

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

    Content=" 點膠參數 "

    FontSize="16"

    FontWeight="Bold"

    Foreground="Brown" />

<Label

    HorizontalAlignment="Left"

    Content=" 照射位置 "

    Foreground="Brown" />

/********************

 * ListView

 ********************/

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

 ********************/

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

    Content=" 測試 " />

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

                Header=" 吸嘴 "

                IsReadOnly="True" />

            <DataGridTextColumn

                Binding="{Binding ShiftX, StringFormat='0.000'}"

                Header="X 位移 " />

            <DataGridTextColumn

                Binding="{Binding ShiftY}"

                Header="Y 位移 " />

            <DataGridTextColumn

                Binding="{Binding NewX}"

                Header=" 新 X" />

            <DataGridTextColumn

                Binding="{Binding NewY}"

                CanUserSort="False"

                Header=" 新 Y" />

            <DataGridCheckBoxColumn

                Binding="{Binding IsUpdate}"

                EditingElementStyle="{StaticResource MaterialDesignDataGridCheckBoxColumnEditingStyle}"

                ElementStyle="{StaticResource MaterialDesignDataGridCheckBoxColumnStyle}"

                Header=" 更新 " />

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

                Header=" 托盤資料 ID" />

            <DataGridTextColumn

                MinWidth="250"

                Binding="{Binding Memo}"

                Header=" 註解 " />

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

<ComboBox

    x:Name="DataNameIdAtComboBox"

    DisplayMemberPath="Value"

    SelectedValuePath="Key"

    ItemsSource="{Binding DataContext.WfsDataNameList, RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type DataGrid}}}"

    SelectedValue="{Binding Path=DataNameId}"

    Style="{StaticResource ParaNameSelector}" />

<TextBox

    Grid.Row="1"

    Grid.Column="1"

    Style="{StaticResource Coordinate}"

    Text="{Binding RelativeSource={x:Static RelativeSource.Self}, Path=DataContext.MachineDatas.DatumPointX, Mode=TwoWay, StringFormat=F3}" />

        Binding="{Binding RelativeSource={x:Static RelativeSource.Self}, Path=IsChecked}"

ContentCharacterCasing="{Binding RelativeSource={RelativeSource TemplatedParent}, Path=(mah:ControlsHelper.ContentCharacterCasing)}"

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

/********************

 * 取消權杖

 ********************/

private CancellationTokenSource _cts;

private CancellationToken _token;

if (_cts != null)

    _cts.Dispose();

_cts?.Dispose();

_cts = new();

_token = _cts.Token;

// Cancel

if (_cts != null && !_cts.IsCancellationRequested)

    _cts.Cancel();

// 實作 1

_cts?.Dispose();

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

private static readonly Lazy<xxxClass> _instance = new(() => new xxxClass());

public static xxxClass Instance => _instance.Value;

// Singleton 單例模式

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

private static readonly Lazy<xxxClass> _instance = new(() => new xxxClass());

public static xxxClass Instance => _instance.Value;

public event PropertyChangedEventHandler? PropertyChanged;

private void RaisePropertyChanged([CallerMemberName] string propertyName = "")

{

    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));

}

/********************

 * MultiBinding

 ********************/

authorityModels

Binding Source={x:Static authorityModels:AuthorityData.Instance}, Path=CurrentAuthotization.Menu_MainConsole, Mode=OneWay, Converter={StaticResource BooleanToTrueFalseConverter}

<StackPanel.Visibility>

    <MultiBinding

        Converter="{StaticResource VisibilityByMultiBinding}"

        ConverterParameter="OR">

        <Binding

            Mode="OneWay" Path="SystemMode_WfsOnline"

            Source="{x:Static coreModels:SystemStateHandle.Instance}" />

        <Binding

            Mode="OneWay" Path="SystemMode_DockingMode_DoubleCylinder"

            Source="{x:Static coreModels:SystemStateHandle.Instance}" />

    </MultiBinding>

</StackPanel.Visibility>

<GroupBox.IsEnabled>

    <MultiBinding

        Converter="{StaticResource IsEnabledByMultiBinding}"

        ConverterParameter="AND">

        <Binding

            Mode="OneWay"

            Path="CurrentAuthotization.Menu_PalletParameters"

            Source="{x:Static coreModels:AuthorityData.Instance}" />

        <Binding

            Path="DataContext.ScreenEnabled"

            RelativeSource="{RelativeSource Mode=FindAncestor,

                                            AncestorType=UserControl}" />

        <Binding

            Converter="{StaticResource InvertBooleanConverter}"

            ElementName="PalletFuncLocker" Mode="OneWay"

            Path="IsChecked" />

        <Binding

            ElementName="ReportsToDisplay" Mode="OneWay" Path="Tag" />

        <Binding

            Mode="OneWay" Path="Tag" />

    </MultiBinding>

</GroupBox.IsEnabled>

/********************

 * Prism

 ********************/

<!--

    <<< Prism IDialogAware 專用旳 UserControl Style >>>

    檔首引入:

        xmlns:prism="http://prismlibrary.com/"

        prism:Dialog.WindowStyle="{DynamicResource PrismDialogStyleTheme}"

    增加 Title Property:

        public bool ShowTitle => 是否顯示標題列

        public string Title => 標題文字

        public SolidColorBrush TitleBackground => 標題顏色

-->

<Style

    x:Key="PrismDialogStyleTheme"

    TargetType="prism:DialogWindow">

    <Setter Property="WindowStyle" Value="None" />

    <Setter Property="ResizeMode" Value="NoResize" />

    <Setter Property="SizeToContent" Value="WidthAndHeight" />

    <Setter Property="AllowsTransparency" Value="True" />

    <Setter Property="ShowInTaskbar" Value="False" />

    <Setter Property="prism:Dialog.WindowStartupLocation" Value="CenterScreen" />

    <Setter Property="Template">

        <Setter.Value>

            <ControlTemplate TargetType="prism:DialogWindow">

                <Border x:Name="MainBorder"

                    Padding="1.5"

                    Background="Cyan"

                    BorderBrush="{StaticResource MahApps.Brushes.AccentBase}"

                    BorderThickness="4" CornerRadius="5">

                    <DockPanel

                        Margin="1"

                        Background="{TemplateBinding Background}"

                        LastChildFill="True">

                        <!--  標題欄 START  -->

                        <StackPanel

                            Height="34"

                            Background="{Binding TitleBackground}"

                            DockPanel.Dock="Top" Orientation="Horizontal"

                            Visibility="{Binding ShowTitle, Mode=OneWay, Converter={StaticResource booleanToVisibleCollapsedConverter}}">

                            <!--  Icon  -->

                            <!--<iconPacks:PackIconMaterial

                                Width="16" Height="16"

                                Margin="20,0,10,0" HorizontalAlignment="Center"

                                VerticalAlignment="Center"

                                Foreground="{DynamicResource MahApps.Brushes.ThemeForeground}"

                                Kind="MessageTextOutline" />-->

                            <!--  Title  -->

                            <TextBlock

                                Margin="20,2,0,0" VerticalAlignment="Center"

                                Text="{Binding Title, Mode=OneWay}">

                                <TextBlock.Style>

                                    <Style TargetType="TextBlock">

                                        <Setter Property="Foreground" Value="{DynamicResource MahApps.Brushes.ThemeForeground}" />

                                        <Setter Property="FontSize" Value="{StaticResource LargeFontSize}" />

                                    </Style>

                                </TextBlock.Style>

                            </TextBlock>

                        </StackPanel>

                        <!--  標題欄 END  -->

                        <!--  內容區  -->

                        <ContentPresenter x:Name="ContentPresenter"

                            ClipToBounds="True" DockPanel.Dock="Top" />

                    </DockPanel>

                </Border>

            </ControlTemplate>

        </Setter.Value>

    </Setter>

</Style>

<prism:Dialog.WindowStyle>

    <Style TargetType="prism:DialogWindow">

        <Setter Property="ShowInTaskbar" Value="False" />

        <Setter Property="SizeToContent" Value="WidthAndHeight" />

        <Setter Property="ResizeMode" Value="NoResize" />

        <Setter Property="WindowStyle" Value="None" />

        <Setter Property="prism:Dialog.WindowStartupLocation" Value="CenterOwner" />

    </Style>

</prism:Dialog.WindowStyle>

<md:Card

    Margin="0" Padding="5,15,5,10"

    md:ShadowAssist.ShadowDepth="Depth3"

    Background="{StaticResource MahApps.Brushes.Accent3}"

    UniformCornerRadius="10">

</md:Card>

/********************

 * OEP520G

 ********************/

private readonly IRegionManager         _regionManager          ;

private readonly IThemeSelectorService  _themeSelectorService   ;

private readonly IEventAggregator       _ea                     ;

private readonly IIoService         _io         ;

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

IIoService          io              ,

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

_io         =   io              ;

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

private readonly IIoService             _ioService              ;

private readonly IDialogService         _dialogService          ;

private readonly IPrismMessageBox       _prismMessageBox        ;

private readonly ICrudDialog            _crudDialog             ;

private readonly ISystem                _sys                    ;

private readonly ISystemMessenger       _sysMessenger           ;

private readonly IAuthority             _authority              ;

private readonly IMachine               _machine                ;

private readonly IProductManager        _pm                     ;

private readonly IStage                 _stage                  ;

private readonly ITray                  _tray                   ;

private readonly IWfs                   _wfs                    ;

private readonly IPlc                   _plc                    ;

private readonly IAvailability          _availability           ;

private readonly IVision                _vision                 ;

private readonly IDisplacement          _displacement           ;

private readonly IMeasuring             _measuring              ;

private readonly IBigData               _bigData                ;

echo9051@taichung.com.tw

IRegionManager          regionManager           ,

IThemeSelectorService   themeSelectorService    ,

IApplicationCommands    applicationCommands     ,

IEventAggregator        ea                      ,

ISqliteService          sqliteService           ,

IIoService              ioService               ,

IDialogService          dialogService           ,

IPrismMessageBox        prismMessageBox         ,

ICrudDialog             crudDialog              ,

ISystem                 sys                     ,

ISystemMessenger        sysMessenger            ,

IAuthority              authority               ,

IMachine                machine                 ,

IProductManager         pm                      ,

IStage                  stage                   ,

ITray                   tray                    ,

IWfs                    wfs                     ,

IPlc                    plc                     ,

IAvailability           availability            ,

IVision                 vision                  ,

IDisplacement           displacement            ,

IMeasuring              measuring               ,

IBigData                bigData                 ,

_regionManager          =   regionManager           ;

_themeSelectorService   =   themeSelectorService    ;

_ea                     =   ea                      ;

_sqlite                 =   sqliteService           ;

_ioService              =   ioService               ;

_dialogService          =   dialogService           ;

_prismMessageBox        =   prismMessageBox         ;

_crudDialog             =   crudDialog              ;

_sys                    =   sys                     ;

_sysMessenger           =   sysMessenger            ;

_authority              =   authority               ;

_machine                =   machine                 ;

_pm                     =   pm                      ;

_stage                  =   stage                   ;

_tray                   =   tray                    ;

_wfs                    =   wfs                     ;

_plc                    =   plc                     ;

_availability           =   availability            ;

_vision                 =   vision                  ;

_displacement           =   displacement            ;

_measuring              =   measuring               ;

_bigData                =   bigData                 ;
