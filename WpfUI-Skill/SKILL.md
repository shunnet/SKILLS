---
name: wpfui-skill
description: WPF 现代化界面开发技能，基于 Snet.Windows.Core 和 Snet.Windows.Controls 库，支持自定义窗口、MVVM 架构、深色/浅色主题切换、中英文多语言、PropertyGrid 属性编辑器、拖拽控件、LED 指示灯、分页栏、系统托盘、消息对话框等完整 WPF 桌面应用开发能力。支持"一句话"生成完整 WPF 界面。
version: 1.0.0.7
metadata:
  hermes:
    tags: [wpf, desktop, mvvm, ui, theme, localization, property-grid, drag-drop, dotnet]
    related_skills: [daq-skill, plugindev-skill]
    homepage: https://shunnet.top
---

# 🖥️ WpfUI-Skill — WPF 现代化界面开发

> **一句话：** 用户描述界面需求 → AI 自动生成完整可运行的 WPF 应用程序

## 🎯 用途

帮助用户基于 Snet WPF 框架快速构建现代化桌面应用，涵盖窗口管理、MVVM 绑定、主题切换、多语言、内置控件、拖拽等完整能力。

## ✨ 能力

- 🪟 **现代化窗口基类** — `WindowBase` 自定义窗口，含标题栏按钮、DPI 感知、加载动画
- 🎨 **深色/浅色主题** — Material Design + Wpf.Ui 双主题引擎，一键切换
- 🌍 **中英文多语言** — `LocExtension` / `BLoc` 标记扩展，运行时热切换
- 🧩 **MVVM 架构** — `BindNotify` 基类 + `InjectionWpf` DI 容器 + `EventCommand` 行为
- 🎛️ **内置控件库** — Button / ComboBox / TextBox / PropertyControl / LedGauge / PageBar
- 🔧 **PropertyGrid** — 属性编辑器，含 ColorPicker、FilePicker、DataGrid、TreeListBox
- 🖱️ **拖拽控件** — 8 向缩放手柄 + 拖拽移动 + 动画拖拽创建
- 💬 **消息对话框** — OK / OKCancel / Yes / YesNo 四种模式，Material Design 风格
- 🔔 **系统托盘** — `NotifyIcon` 最小化到托盘，右键菜单
- 📐 **Win32 集成** — 窗口消息处理、DWM 背景色同步、最大化补偿、窗口抖动

## 🗣️ 交互流程

AI 先用大白话问用户：

1. 🏷️ **窗口需求** — 窗口标题、是否需要皮肤/语言切换、有无图标
2. 🎨 **主题偏好** — 默认深色还是浅色
3. 🌍 **语言需求** — 中文 / 英文 / 中英切换
4. 🎛️ **需要哪些控件** — 按钮、下拉框、输入框、属性编辑、LED、分页
5. 📊 **数据绑定** — ViewModel 结构、命令、数据源

确认后生成完整代码。

---

## ⚡ 快速开始

### NuGet 安装

```bash
# 核心库（必装）
dotnet add package Snet.Windows.Core -v 1.0.0.1

# 控件库（按需）
dotnet add package Snet.Windows.Controls -v 1.0.0.1
```

### NuGet 依赖关系

```
Snet.Windows.Controls
  ├── Snet.Windows.Core (>= 1.0.0.1)
  │     ├── MaterialDesignThemes (>= 5.3.2)
  │     ├── WPF-UI (>= 4.3.0)
  │     ├── CommunityToolkit.Mvvm (>= 8.4.2)
  │     └── Snet.Core (>= 1.0.0.1)
  └── AvalonEdit (>= 6.3.1.120)
```

---

## 🪟 第一章：WindowBase 窗口基类

### 概述

`WindowBase` 继承 `System.Windows.Window`，提供以下能力：

| 功能 | 属性 | 默认值 | 说明 |
|:---|:---|:---|:---|
| 皮肤切换 | `SkinEnabled` | `true` | 标题栏显示皮肤切换按钮 |
| 语言切换 | `LanguageEnabled` | `true` | 标题栏显示语言切换按钮 |
| 标题靠左 | `TitleLeft` | `true` | `false` 时标题居中 |
| 版本显示 | `VerEnabled` | `true` | 底部状态栏显示版本号 |
| 加载动画 | `LoadAnimationEnabled` | `false` | 淡入淡出过渡效果 |
| 动画时长 | `AnimationTime` | `2000` | 毫秒 |
| 最大化补偿 | `MaximizeBorderThickness` | `0,0,0,0` | 最大化时的内边距补偿 |

### 最小示例

**App.xaml:**
```xml
<Application x:Class="MyApp.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!-- 深色主题 -->
                <ResourceDictionary Source="pack://application:,,,/Snet.Windows.Core;component/themes/DarkTheme.xaml" />
                <!-- 或浅色主题 -->
                <!-- <ResourceDictionary Source="pack://application:,,,/Snet.Windows.Core;component/themes/LightTheme.xaml" /> -->
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

**MainWindow.xaml:**
```xml
<snet:WindowBase
    x:Class="MyApp.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:snet="https://shunnet.top"
    Title="我的应用"
    Icon="/Icon.ico"
    LanguageEnabled="True"
    SkinEnabled="True"
    WindowStartupLocation="CenterScreen"
    Height="600"
    Width="900">
    <Grid>
        <TextBlock Text="Hello, Snet WPF!" FontSize="24"
                   HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Grid>
</snet:WindowBase>
```

**MainWindow.xaml.cs:**
```csharp
namespace MyApp;

public partial class MainWindow : Snet.Windows.Core.WindowBase
{
    public MainWindow()
    {
        InitializeComponent();
    }
}
```

### 窗口抖动效果

```csharp
// 调用静态方法产生抖动（如输入验证失败时）
WindowBase.WindowShake(this);
// 或对指定窗口抖动
WindowBase.WindowShake(someWindow);
```

---

## 🧩 第二章：MVVM 架构

### BindNotify 基类

`BindNotify` 继承 `ObservableObject`，提供基于**表达式**的类型安全属性系统，无需手动声明后备字段：

```csharp
using Snet.Windows.Core.mvvm;

public class MainViewModel : BindNotify
{
    // 获取属性
    public string UserName => GetProperty(() => UserName);

    // 设置属性（仅通知）
    public void SetUserName(string value)
        => SetProperty(() => UserName, value);

    // 设置属性（带回调 — 接收旧值）
    public void SetUserName(string value)
        => SetProperty(() => UserName, value, oldValue =>
        {
            Console.WriteLine($"用户名从 {oldValue} 变更为 {value}");
        });

    // 设置属性（带回调 — 无参）
    public void SetUserName(string value)
        => SetProperty(() => UserName, value, () =>
        {
            Console.WriteLine("用户名已更新");
        });
}
```

**优势：** 无需手动声明 `private string _userName` 后备字段，属性值存储在内部线程安全的 `Dictionary<string, object>` 中，`SetProperty` 自动触发 `OnPropertyChanged`。

### EventCommand 行为

将任意 WPF 事件绑定到 ViewModel 中的 ICommand：

```xml
<snet:TextBoxControl ...>
    <b:Interaction.Triggers>
        <b:EventTrigger EventName="TextChanged">
            <mvvm:EventCommand Command="{Binding TextChangedCommand}" />
        </b:EventTrigger>
    </b:Interaction.Triggers>
</snet:TextBoxControl>
```

### InjectionWpf 依赖注入

自动创建 View 并注入 ViewModel 作为 DataContext：

```csharp
using Snet.Windows.Core.handler;

// 创建 Window 并注入 ViewModel（Transient 默认不缓存）
var window = InjectionWpf.Window<MainWindow, MainViewModel>();

// 创建 UserControl 并注入 ViewModel
var control = InjectionWpf.UserControl<MyControl, MyViewModel>();

// 缓存模式（Singleton）
var cachedWindow = InjectionWpf.Window<MainWindow, MainViewModel>(cache: true);
```

### 窗口注入示例

```csharp
// 在 App.xaml.cs 中重写 OnStartup
protected override void OnStartup(StartupEventArgs e)
{
    var mainWindow = InjectionWpf.Window<MainWindow, MainViewModel>();
    mainWindow.Show();
}
```

---

## 🎨 第三章：主题系统

### SkinHandler 皮肤管理

```csharp
using Snet.Windows.Core.handler;
using Snet.Windows.Core.@enum;

// 获取当前皮肤
SkinType current = SkinHandler.GetSkin();           // 同步
SkinType current = await SkinHandler.GetSkinAsync(); // 异步

// 设置皮肤
SkinHandler.SetSkin(SkinType.Dark);   // 深色
SkinHandler.SetSkin(SkinType.Light);  // 浅色
```

**持久化：** 皮肤设置自动保存到 `{AppContext.BaseDirectory}/config/skin.json`。

**事件通知：**
```csharp
// 同步事件
SkinHandler.OnSkinEvent += (sender, e) =>
{
    Console.WriteLine($"皮肤切换到: {e.SkinType}");
};

// 异步事件（自动顺序执行多个异步处理器）
SkinHandler.OnSkinEventAsync += async (sender, e) =>
{
    await Task.Delay(100);
    Console.WriteLine($"主题已切换: {e.SkinType}");
};
```

### 主题资源

| 资源 Key | 用途 |
|:---|:---|
| `WindowBackgroundBrush` | 窗口背景色 |
| `CaptionActiveBackgroundBrush` | 标题栏背景渐变 |
| `TitleForeground` | 标题字体颜色 |
| `VerBackgroundBrush` | 底部版本栏背景 |
| `ImageColor` | 图标颜色 |
| `ButtonMouseOverBackground` | 按钮悬停背景 |
| `CloseButtonMouseOverBackground` | 关闭按钮悬停背景（粉红色） |

### 自定义皮肤颜色

在 `App.xaml` 的 `WindowBaseStyle` 应用前覆盖资源：

```xml
<SolidColorBrush x:Key="WindowBackgroundBrush" Color="#1E1E2E" />
<SolidColorBrush x:Key="TitleForeground" Color="#CDD6F4" />
```

---

## 🌍 第四章：多语言系统

### LanguageHandler 语言管理

```csharp
using Snet.Windows.Core.handler;
using Snet.Model.@enum;

// 获取当前语言
LanguageType current = LanguageHandler.GetLanguage();

// 设置语言
LanguageHandler.SetLanguage(LanguageType.zh);  // 中文
LanguageHandler.SetLanguage(LanguageType.en);  // 英文

// 获取翻译文本
string? text = LanguageHandler.GetLanguageValue("Hello");
string? text = await LanguageHandler.GetLanguageValueAsync("Hello");

// 使用指定程序集的资源
var model = new LanguageModel("MyApp", "Language", "MyApp.dll");
string? text = LanguageHandler.GetLanguageValue("Hello", model);
```

### LocExtension XAML 标记扩展

```xml
<!-- 基础用法：通过 Key 查找翻译 -->
<TextBlock Text="{snet:Loc Hello}" />

<!-- 带类型转换（如 SolidColorBrush） -->
<TextBlock Foreground="{snet:Loc Accent, Converter={StaticResource ColorBrushConverter}}" />

<!-- 强制指定语言 -->
<TextBlock Text="{snet:Loc Hello, ForceCulture=zh}" />
```

### BLoc 绑定扩展

```xml
<!-- BLoc 继承自 Binding，可作为绑定源使用 -->
<TextBlock Text="{snet:BLoc Hello}" />

<!-- 强制指定语言 -->
<TextBlock Text="{snet:BLoc Hello, ForceCulture=zh}" />
```

### 资源文件配置

在 XAML 根元素上声明默认程序集和字典：

```xml
<snet:WindowBase
    xmlns:snet="https://shunnet.top"
    snet:ResxLocalizationProvider.DefaultAssembly="MyApp"
    snet:ResxLocalizationProvider.DefaultDictionary="Language"
    ...>
```

对应的 `.resx` 资源文件：
```
Language.resx         (默认 — 中文)
Language.en.resx      (英文翻译)
```

---

## 🎛️ 第五章：内置控件

### ButtonControl — 图标按钮

```xml
<snet:ButtonControl
    Margin="10"
    HorizontalAlignment="Center"
    Command="{Binding SaveCommand}"
    Content="{snet:Loc Save}"
    Icon="{DynamicResource SaveIcon}"
    CornerRadius="8" />
```

| 属性 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `Content` | `string` | — | 按钮文本 |
| `Icon` | `ImageSource` | `null` | 图标 |
| `Command` | `ICommand` | `null` | 绑定命令 |
| `CornerRadius` | `CornerRadius` | `new CornerRadius(8)` | 圆角 |

### ComboBoxControl — 下拉选择框

```xml
<snet:ComboBoxControl
    Width="200"
    DisplayMemberPath="Key"
    Hint="请选择..."
    Icon="{DynamicResource DeviceIcon}"
    ItemsSource="{Binding DeviceList}"
    SelectedItem="{Binding SelectedDevice}" />
```

| 属性 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `Hint` | `object` | — | 占位提示文本 |
| `ItemsSource` | `IEnumerable` | `null` | 数据源 |
| `SelectedItem` | `object` | `null` | 选中项（双向绑定） |
| `DisplayMemberPath` | `string` | — | 显示路径 |
| `Icon` | `ImageSource` | `null` | 图标 |

### TextBoxControl — 文本输入框

```xml
<snet:TextBoxControl
    Width="200"
    Hint="请输入名称"
    ClearButtonEnabled="True"
    Icon="{DynamicResource SearchIcon}"
    Text="{Binding SearchText}" />
```

| 属性 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `Text` | `object` | `null` | 输入文本（双向绑定） |
| `Hint` | `string` | — | 占位提示文本 |
| `ClearButtonEnabled` | `bool` | `true` | 是否显示清除按钮 |
| `Icon` | `ImageSource` | `null` | 图标 |

### LedGaugeControl — LED 指示灯

```xml
<snet:LedGaugeControl
    Width="30" Height="30"
    Color="Red"
    IsOn="{Binding AlarmActive}"
    IsFlashing="True"
    FlashingInterval="300"
    OnLightness="0.8"
    OffLightness="0.2" />
```

| 属性 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `Color` | `Color` | `Green` | LED 颜色 |
| `IsOn` | `bool` | `false` | 亮/灭状态 |
| `IsFlashing` | `bool` | `false` | 闪烁模式 |
| `FlashingInterval` | `double` | `500` | 闪烁间隔（毫秒） |
| `OnLightness` | `double` | `0.5` | 点亮时的亮度 |
| `OffLightness` | `double` | `0.3` | 熄灭时的亮度 |
| `IsFlat` | `bool` | `false` | 扁平模式（移除外阴影） |

### PageBarControl — 分页栏

```xml
<snet:PageBarControl
    PageIndex="{Binding PageIndex, Mode=TwoWay}"
    PageSize="{Binding PageSize, Mode=TwoWay}"
    Total="{Binding TotalCount, Mode=TwoWay}"
    MaxDisplayedPageCount="7" />
```

| 属性 | 类型 | 默认值 | 说明 |
|:---|:---|:---|:---|
| `PageIndex` | `int` | `1` | 当前页码（1-based） |
| `PageSize` | `int` | — | 每页条数 |
| `Total` | `int` | — | 总条目数 |
| `MaxDisplayedPageCount` | `int` | `7` | 最大显示页码数 |

```csharp
// ViewModel 中处理分页变更
partial class MainViewModel : BindNotify
{
    public int PageIndex => GetProperty(() => PageIndex);
    public int PageSize => GetProperty(() => PageSize);
    public int Total => GetProperty(() => Total);

    // 页码变化命令
    public ICommand PageChangedCommand => new RelayCommand<int>(async (page) =>
    {
        await LoadPageAsync(page);
    });
}
```

---

## 🔧 第六章：PropertyControl 属性编辑器

### 基础用法

```xml
<snet:PropertyControl BasicsData="{Binding SettingsObject}" />
```

### 配置对象

```csharp
using Snet.Windows.Controls.property.wpf.PropertyGrid;

// 属性对象
public class DeviceSettings
{
    [DisplayName("设备名称")]
    [Category("基本信息")]
    public string DeviceName { get; set; } = "PLC-001";

    [DisplayName("IP 地址")]
    [Category("连接")]
    public string IpAddress { get; set; } = "192.168.0.1";

    [DisplayName("端口")]
    [Category("连接")]
    [Slidable(0, 65535)]
    public int Port { get; set; } = 6688;

    [DisplayName("颜色")]
    [Category("外观")]
    public Color IndicatorColor { get; set; } = Colors.Green;

    [DisplayName("启用日志")]
    [Category("高级")]
    public bool EnableLog { get; set; } = true;

    [DisplayName("协议类型")]
    [Category("连接")]
    [ItemsSourceProperty(typeof(ProtocolTypes))]
    public string ProtocolType { get; set; }

    [DisplayName("固件文件")]
    [Category("高级")]
    [InputFilePath("Firmware Files|*.hex;*.bin")]
    public string FirmwarePath { get; set; }

    [Browsable(false)]
    public string InternalId { get; set; } // 不会在界面中显示
}
```

### PropertyGrid 注解一览

| Attribute | 用途 |
|:---|:---|
| `[DisplayName]` | 显示名称 |
| `[Category]` | 分类分组 |
| `[Description]` | 提示描述 |
| `[Browsable(false)]` | 隐藏属性 |
| `[ReadOnly(true)]` | 只读属性 |
| `[Editable(false)]` | 不可编辑 |
| `[Slidable(min, max)]` | 滑块控件 |
| `[Spinnable(min, max, step)]` | 数字微调控件 |
| `[SelectorStyle(ComboBox)]` | 下拉选择样式 |
| `[ItemsSourceProperty(typeof(EnumType))]` | 枚举数据源 |
| `[InputFilePath("filter")]` | 文件输入选择器 |
| `[OutputFilePath("filter")]` | 文件输出选择器 |
| `[DirectoryPath]` | 目录选择器 |
| `[FontFamilySelector]` | 字体选择器 |
| `[Width(min, max)]` | 宽度 |
| `[Height(min, max)]` | 高度 |
| `[FormatString("{0:F2}")]` | 格式化字符串 |
| `[Progress]` | 进度条显示 |
| `[CheckableItems]` | 可勾选列表 |
| `[EnableByRadioButton]` | 单选框条件启用 |
| `[VisibleBy("OtherProperty")]` | 条件可见 |
| `[EnableBy("OtherProperty")]` | 条件启用 |
| `[SortIndex(n)]` | 排序 |
| `[HeaderPlacement(Left)]` | 标题位置 |
| `[IndentationLevel(n)]` | 缩进层级 |

---

## 🖱️ 第七章：拖拽控件

### DragControlsBase — 缩放+移动

```csharp
using Snet.Windows.Controls.drag;

// 创建可拖拽的控件
var dragControl = new DragControlsBase(
    Controls: targetElement,         // 被装饰的 UI 元素
    LlayoutContainer: parentGrid,    // 父布局容器
    Move: true,                      // 启用拖拽移动
    DragSize: true                   // 启用 8 向缩放手柄
);

// 自定义样式
dragControl.BorderColor = new SolidColorBrush(Colors.DodgerBlue);
dragControl.ThumbInnerColor = new SolidColorBrush(Colors.White);
dragControl.ThumbOuterColor = new SolidColorBrush(Colors.DodgerBlue);
dragControl.MinWidths = 50;
dragControl.MaxWidths = 800;

// 将装饰器层应用到目标元素
var adornerLayer = AdornerLayer.GetAdornerLayer(targetElement);
adornerLayer.Add(dragControl);
```

### DragControlsAnimate — 拖拽创建新控件

```csharp
var dragAnimate = new DragControlsAnimate(
    Windows: this,                   // 包含窗口
    LlayoutContainer: canvas,        // 画布或网格
    HeightOffset: 0,
    WidthOffset: 0
);

// 注册拖拉源控件
dragAnimate.Insert(sourceControl);

// 拖拽回调 — 返回新控件实例
dragAnimate.dragEvenTrigger = () =>
{
    var newControl = new Button { Content = "新控件" };
    return (newControl, IsMove: true, IsDragSize: true);
};

// 移除拖拉源
dragAnimate.Remove(sourceControl);
```

---

## 💬 第八章：消息对话框

### 四种按钮模式

```csharp
using Snet.Windows.Controls.message;
using Snet.Windows.Controls.@enum;

// OK — 仅确认
bool ok = await MessageBox.Show("操作成功", "提示");

// OK Cancel — 确认/取消
bool confirmed = await MessageBox.Show(
    "确定要删除吗？", "确认", 
    MessageBoxButton.OKCancel, 
    MessageBoxImage.Warning);

// Yes — 仅是（返回 true）
bool yes = await MessageBox.Show("是否继续？", "提示",
    MessageBoxButton.Yes, MessageBoxImage.Question);

// Yes No — 是/否
bool yesNo = await MessageBox.Show("是否保存？", "提示",
    MessageBoxButton.YesNo, MessageBoxImage.Question);
```

| 按钮枚举 | 显示按钮 | 返回 |
|:---|:---|:---|
| `MessageBoxButton.OK` | 确定 | `true` |
| `MessageBoxButton.OKCancel` | 确定 / 取消 | `true` / `false` |
| `MessageBoxButton.Yes` | 是 | `true` |
| `MessageBoxButton.YesNo` | 是 / 否 | `true` / `false` |

| 图标枚举 | 图标类型 |
|:---|:---|
| `MessageBoxImage.Information` | 信息 |
| `MessageBoxImage.Warning` | 警告 |
| `MessageBoxImage.Error` | 错误 |
| `MessageBoxImage.Question` | 询问 |

---

## 🔔 第九章：系统托盘

### NotifyIcon 托盘图标

```csharp
using Snet.Windows.Controls.tray;
using System.Windows.Media.Imaging;

// 创建托盘管理
var trayManager = new TrayManager();

// 添加托盘图标
var notifyIcon = trayManager.AddIcon("MyAppTray", () =>
{
    // 点击图标时恢复窗口
    this.Show();
    this.WindowState = WindowState.Normal;
    this.Activate();
});

// 设置图标（必须在显示后设置）
notifyIcon.Show();
notifyIcon.Icon = new BitmapImage(new Uri("pack://application:,,,/Icon.ico"));

// 设置托盘提示文字
notifyIcon.ToolTipText = "我的应用";

// 隐藏窗口到托盘
notifyIcon.OnMinimiseToTray = () =>
{
    this.Hide();
};

// 清理
trayManager.RemoveIcon("MyAppTray");
```

---

## 🧬 第十章：完整示例 — 数采监控面板

下面是一个完整的 WPF 监控面板示例，结合了窗口基类、MVVM、多语言、PropertyGrid、LED 指示灯、分页等：

### MainViewModel.cs

```csharp
using CommunityToolkit.Mvvm.Input;
using Snet.Windows.Core.mvvm;
using System.Collections.ObjectModel;
using System.Windows.Media;

namespace MyApp;

public partial class MonitorViewModel : BindNotify
{
    // 属性通过表达式访问
    public string DeviceName => GetProperty(() => DeviceName);
    public bool IsConnected => GetProperty(() => IsConnected);
    public int PageIndex => GetProperty(() => PageIndex);
    public int PageSize => GetProperty(() => PageSize);
    public int Total => GetProperty(() => Total);
    public Color LedColor => GetProperty(() => LedColor);

    public ObservableCollection<DeviceModel> Devices { get; } = new();
    public DeviceSettings Settings => GetProperty(() => Settings);

    // 初始化
    public MonitorViewModel()
    {
        SetProperty(() => DeviceName, "S7-1200-01", null);
        SetProperty(() => PageSize, 20, null);
        SetProperty(() => LedColor, Colors.Green, null);
        SetProperty(() => Settings, new DeviceSettings(), null);
    }

    // 连接命令
    [RelayCommand]
    private async Task ConnectAsync()
    {
        SetProperty(() => IsConnected, true, old =>
        {
            SetProperty(() => LedColor, Colors.Red, null);
        });
        await LoadDataAsync();
    }

    // 断开命令
    [RelayCommand]
    private void Disconnect()
    {
        SetProperty(() => IsConnected, false, null);
        SetProperty(() => LedColor, Colors.Gray, null);
    }

    // 搜索命令
    [RelayCommand]
    private async Task SearchAsync(string keyword)
    {
        SetProperty(() => PageIndex, 1, null);
        await LoadDataAsync(keyword);
    }

    private async Task LoadDataAsync(string? keyword = null)
    {
        // 模拟数据加载
        await Task.Delay(500);
        Devices.Clear();
        for (int i = 0; i < PageSize; i++)
            Devices.Add(new DeviceModel { Id = i + 1, Name = $"点位 {i + 1}" });
        SetProperty(() => Total, 200, null);
    }
}

public class DeviceModel
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

public class DeviceSettings
{
    [DisplayName("设备名称")]
    [Category("基本信息")]
    public string Name { get; set; } = "PLC-001";

    [DisplayName("IP 地址")]
    [Category("连接")]
    public string Ip { get; set; } = "192.168.0.1";

    [DisplayName("端口")]
    [Category("连接")]
    [Spinnable(0, 65535, 1)]
    public int Port { get; set; } = 502;

    [DisplayName("采样间隔")]
    [Category("采集")]
    [Slidable(100, 10000)]
    public int Interval { get; set; } = 1000;
}
```

### MainWindow.xaml

```xml
<snet:WindowBase
    x:Class="MyApp.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:snet="https://shunnet.top"
    xmlns:uis="http://schemas.lepo.co/wpfui/2022/xaml"
    xmlns:local="clr-namespace:MyApp"
    Title="{snet:Loc AppTitle}"
    snet:ResxLocalizationProvider.DefaultAssembly="MyApp"
    snet:ResxLocalizationProvider.DefaultDictionary="Language"
    LanguageEnabled="True"
    SkinEnabled="True"
    Icon="/Icon.ico"
    WindowStartupLocation="CenterScreen"
    Height="700"
    Width="1100">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="auto" />
            <RowDefinition />
            <RowDefinition Height="auto" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="300" />
        </Grid.ColumnDefinitions>

        <!-- 顶部工具栏 -->
        <StackPanel Grid.Row="0" Grid.ColumnSpan="2"
                    Orientation="Horizontal" Margin="0,0,0,10">
            <!-- 连接状态 LED -->
            <snet:LedGaugeControl Width="24" Height="24"
                Color="{Binding LedColor}" IsOn="{Binding IsConnected}"
                VerticalAlignment="Center" Margin="5" />

            <TextBlock Text="{Binding DeviceName}" FontSize="16"
                       VerticalAlignment="Center" Margin="10,0" />

            <snet:ButtonControl
                Content="{snet:Loc Connect}"
                Command="{Binding ConnectCommand}"
                Icon="{DynamicResource ConnectIcon}" Margin="5" />

            <snet:ButtonControl
                Content="{snet:Loc Disconnect}"
                Command="{Binding DisconnectCommand}"
                Icon="{DynamicResource DisconnectIcon}" Margin="5" />

            <snet:TextBoxControl Width="200"
                Hint="搜索..." ClearButtonEnabled="True"
                Icon="{DynamicResource SearchIcon}" Margin="20,0,0,0">
                <b:Interaction.Triggers>
                    <b:EventTrigger EventName="TextChanged">
                        <b:InvokeCommandAction
                            Command="{Binding SearchCommand}"
                            CommandParameter="{Binding Text, RelativeSource={RelativeSource AncestorType=snet:TextBoxControl}}" />
                    </b:EventTrigger>
                </b:Interaction.Triggers>
            </snet:TextBoxControl>
        </StackPanel>

        <!-- 数据表格 -->
        <DataGrid Grid.Row="1" Grid.Column="0"
                  ItemsSource="{Binding Devices}"
                  AutoGenerateColumns="True" Margin="0,0,10,0" />

        <!-- 右侧属性面板 -->
        <Border Grid.Row="1" Grid.Column="1"
                BorderBrush="{DynamicResource MaterialDesignBody}"
                BorderThickness="1" CornerRadius="4">
            <snet:PropertyControl BasicsData="{Binding Settings}" />
        </Border>

        <!-- 底部分页栏 -->
        <snet:PageBarControl Grid.Row="2" Grid.ColumnSpan="2"
            PageIndex="{Binding PageIndex, Mode=TwoWay}"
            PageSize="{Binding PageSize, Mode=TwoWay}"
            Total="{Binding Total, Mode=TwoWay}"
            MaxDisplayedPageCount="7" Margin="0,10,0,0" />
    </Grid>
</snet:WindowBase>
```

---

## 📐 第十一章：XAML 命名空间与资源

### XML 命名空间

```xml
xmlns:snet="https://shunnet.top"
```

### 控件命名空间映射

| 前缀 | 命名空间 | 程序集 |
|:---|:---|:---|
| `snet:` | `Snet.Windows.Core` 和 `Snet.Windows.Controls` | 两程序集共享同一 XMLNS |

### 内置图标资源

```xml
Icon="{DynamicResource Hello}"
Icon="{DynamicResource AutoHandler}"
Icon="{DynamicResource SaveIcon}"
Icon="{DynamicResource SearchIcon}"
```

自定义图标加载：
```csharp
// 从外部 ResourceDictionary 文件加载
IconsHandler.Loading("pack://application:,,,/MyApp;component/Resources/Icons.xaml");

// 获取图标
var icon = IconsHandler.GetIcon("MyCustomIcon");
```

### 常用 DynamicResource Key

| Key | 说明 |
|:---|:---|
| `WindowBackgroundBrush` | 窗口背景 |
| `MaterialDesignBody` | 文本/边框色 |
| `UiScrollBar` | 滚动条样式 |
| `DefaultComboBoxStyle` | 下拉框默认样式 |
| `DefaultComboBoxItemStyle` | 下拉框项样式 |
| `MaterialDesignRaisedButton` | 凸起按钮样式 |
| `MaterialDesignFlatButton` | 扁平按钮样式 |
| `MaterialDesignCircularProgressBar` | 圆形进度条 |

---

## ⚙️ 第十二章：自定义控件开发

### 创建自定义 UserControl

```csharp
// MyCustomControl.xaml.cs
using Snet.Windows.Core.mvvm;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;

namespace MyApp.Controls;

public partial class MyCustomControl : UserControl
{
    public static readonly DependencyProperty HeaderProperty =
        DependencyProperty.Register(nameof(Header), typeof(string),
            typeof(MyCustomControl), new PropertyMetadata(""));

    public static readonly DependencyProperty ValueProperty =
        DependencyProperty.Register(nameof(Value), typeof(double),
            typeof(MyCustomControl), new PropertyMetadata(0.0));

    public string Header
    {
        get => (string)GetValue(HeaderProperty);
        set => SetValue(HeaderProperty, value);
    }

    public double Value
    {
        get => (double)GetValue(ValueProperty);
        set => SetValue(ValueProperty, value);
    }

    public MyCustomControl()
    {
        InitializeComponent();
    }
}
```

### 继承 BindNotify 创建 ViewModel

```csharp
public class MyControlViewModel : BindNotify
{
    public int Count => GetProperty(() => Count);

    [RelayCommand]
    private void Increment()
    {
        SetProperty(() => Count, Count + 1, null);
    }
}
```

---

## 📋 第十三章：常见问题排查

| 问题 | 原因 | 解决 |
|:---|:---|:---|
| 窗口最大化有白色边框 | GlassFrameThickness 溢出 | 设置 `MaximizeBorderThickness` 属性 |
| 皮肤切换后图标不变色 | 图标未使用 DynamicResource | 使用 `{DynamicResource ImageColor}` 作为图标颜色 |
| 语言切换后文本不刷新 | 绑定未使用 LocExtension | 使用 `{snet:Loc Key}` 代替硬编码文本 |
| 拖拽控件位置漂移 | 父容器不是 Grid | 确保 LlayoutContainer 为 Grid（拖拽使用 Margin） |
| LED 闪烁不流畅 | 闪烁间隔过短 | 建议 `FlashingInterval >= 200` |
| 属性编辑器中属性不显示 | 缺少 public get/set | 确保属性有 public getter 和 setter |

---

## 🧭 第十四章：NavigationView 导航壳模式

基于 [WPF-UI](https://github.com/lepoco/wpfui) 的 `NavigationView` 控件，构建带侧边导航的多页签桌面应用壳。

### 基础布局

```xml
<snet:WindowBase
    x:Class="MyApp.MainWindow"
    xmlns:wpfui="http://schemas.lepo.co/wpfui/2022/xaml"
    Title="{snet:Loc SystemTitle}"
    Height="900" Width="1600"
    Icon="/icon.ico">
    <Grid>
        <wpfui:NavigationView
            x:Name="NavigationViewControls"
            FooterMenuItemsSource="{Binding FooterMenuItemsSource}"
            IsBackButtonVisible="Auto"
            IsPaneToggleVisible="True"
            MenuItemsSource="{Binding MenuItemsSource}"
            OpenPaneLength="260" />
    </Grid>
</snet:WindowBase>
```

### MainWindowViewModel — 菜单数据源

```csharp
using Snet.Windows.Core.mvvm;
using Snet.Windows.Controls.handler;
using Wpf.Ui.Controls;
using System.Collections.ObjectModel;

public class MainWindowViewModel : BindNotify
{
    public ICollection<object> MenuItemsSource
    {
        get => GetProperty(() => MenuItemsSource);
        set => SetProperty(() => MenuItemsSource, value);
    }

    public ICollection<object> FooterMenuItemsSource
    {
        get => GetProperty(() => FooterMenuItemsSource);
        set => SetProperty(() => FooterMenuItemsSource, value);
    }

    public MainWindowViewModel()
    {
        MenuItemsSource = new ObservableCollection<object>
        {
            new NavigationViewItem()
            {
                NavigationCacheMode = NavigationCacheMode.Required,
                Content = "Daq",
                Icon = new SymbolIcon { Symbol = SymbolRegular.CatchUp24 },
                MenuItemsSource = new object[]
                {
                    WpfUiHandler.CreationControl(
                        "西门子", SymbolRegular.Molecule28,
                        typeof(SiemensView), true, App.LanguageOperate),
                    WpfUiHandler.CreationControl(
                        "Modbus", SymbolRegular.Molecule28,
                        typeof(ModbusView), true, App.LanguageOperate),
                    // ... 更多协议页 ...
                }
            },
            new NavigationViewItem()
            {
                NavigationCacheMode = NavigationCacheMode.Required,
                Content = "Mq",
                Icon = new SymbolIcon { Symbol = SymbolRegular.Flowchart24 },
                MenuItemsSource = new object[]
                {
                    WpfUiHandler.CreationControl(
                        "Mqtt", SymbolRegular.PlayMultiple16,
                        typeof(MqttClientView), true, App.LanguageOperate),
                    // ...
                }
            },
            new NavigationViewItem()
            {
                NavigationCacheMode = NavigationCacheMode.Required,
                Content = "通信",
                Icon = new SymbolIcon { Symbol = SymbolRegular.Communication16 },
                MenuItemsSource = new object[]
                {
                    WpfUiHandler.CreationControl(
                        "Tcp", SymbolRegular.ArrowTrendingSparkle24,
                        typeof(TcpClientView), true, App.LanguageOperate),
                    WpfUiHandler.CreationControl(
                        "Serial", SymbolRegular.ArrowTrendingSparkle24,
                        typeof(SerialView), true, App.LanguageOperate),
                }
            },
            new NavigationViewItem()
            {
                NavigationCacheMode = NavigationCacheMode.Required,
                Content = "工具",
                Icon = new SymbolIcon { Symbol = SymbolRegular.Toolbox28 },
                MenuItemsSource = new object[]
                {
                    WpfUiHandler.CreationControl(
                        "Svg", SymbolRegular.SplitVertical20,
                        typeof(SvgView), true, App.LanguageOperate),
                }
            },
        };

        FooterMenuItemsSource = new ObservableCollection<object>
        {
            WpfUiHandler.CreationControl(
                "关于", SymbolRegular.Info28,
                typeof(AboutView), true, App.LanguageOperate),
        };
    }
}
```

### WpfUiHandler.CreationControl 辅助方法

```csharp
// 位于 Snet.Windows.Controls.handler.WpfUiHandler
// 创建带路由的导航项，自动绑定多语言标题
public static NavigationViewItem CreationControl(
    string name,                    // 显示名称（多语言 Key）
    SymbolRegular symbol,           // WPF-UI 内置图标
    Type type,                      // 目标 View 类型
    bool isNavigationCache,         // 是否缓存页面
    LanguageModel model)            // 语言模型
```

**可选搜索框：**

```xml
<wpfui:NavigationView AutoSuggestBox>
    <wpfui:AutoSuggestBox x:Name="AutoSuggestBox" PlaceholderText="Search">
        <wpfui:AutoSuggestBox.Icon>
            <wpfui:IconSourceElement>
                <wpfui:SymbolIconSource Symbol="Search24" />
            </wpfui:IconSourceElement>
        </wpfui:AutoSuggestBox.Icon>
    </wpfui:AutoSuggestBox>
</wpfui:NavigationView.AutoSuggestBox>
```

---

## 🧩 第十五章：模板化视图架构

Daq 大规模使用 **DataTemplate + ViewModel 继承链** 模式：共享 UI 布局通过 ResourceDictionary 中的 DataTemplate 定义，各协议页面仅需 5 行 XAML + 5 行 C#。

### 五种内置模板

| 模板 Key | 用途 | 对应 VM 基类 | 主要控件 |
|:---|:---|:---|:---|
| `DAQ` | 采集协议调试 | `DaqTemplateViewModel<T>` | PropertyGrid侧栏 + 数据类型RadioButton + 地址/读/写/订阅 + 三区日志 |
| `MQ` | 消息中间件调试 | `MqTemplateViewModel<T>` | PropertyGrid侧栏 + Topic/Content + 发布/订阅 + 双区日志 |
| `CommunicationClient` | TCP/WS/Serial 客户端 | `CommunicationClientTemplateViewModel<T>` | 发送格式(ASCII/HEX) + 数据输入 + 发送 + 单区日志 |
| `CommunicationService` | TCP/WS 服务端 | `CommunicationServiceTemplateViewModel<T>` | 同上 + 服务端管理 |
| `MqService` | MQ 服务端 | `MqServiceTemplateViewModel<T>` | 同上 + 服务端管理 |

### DAQ 模板 — 采集协议面板结构

```
┌─ DrawerHost ─────────────────────────────────────────────┐
│  ┌─ 左抽屉: PropertyControl (BasicsData 属性编辑) ────┐  │
│  └────────────────────────────────────────────────────┘  │
│  ┌─ 功能区 ────────────────────────────────────────────┐  │
│  │  打开 | 关闭                    获取对应类型参数     │  │
│  │  ┌─ 数据类型 (20 个 RadioButton) ─────────────────┐ │  │
│  │  │ String | Double | Int | Bool | Byte[] | ...    │ │  │
│  │  └───────────────────────────────────────────────┘ │  │
│  │  [地址] [长度] [读取] [订阅]                        │  │
│  │  [数据] [编码类型▼] [写入] [取消订阅]               │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌─ 信息事件 ───────┐ ┌─ 数据事件 ─────────────────────┐ │
│  │ [清空]           │ │ [清空]                         │ │
│  │ (滚动日志框)     │ │ (滚动日志框)                   │ │
│  └──────────────────┘ └────────────────────────────────┘ │
│  ┌─ 交互数据（原始帧收发日志）─────────────────────────┐ │
│  │ [清空]                                              │ │
│  │ (滚动日志框 — 可折叠)                              │ │
│  └────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### 采集协议 View 示例（SiemensView）

**SiemensView.xaml** — 仅 23 行：

```xml
<UserControl x:Class="Daq.view.SiemensView"
    xmlns:snet="https://shunnet.top"
    xmlns:ui="http://materialdesigninxaml.net/winfx/xaml/themes"
    xmlns:vm="clr-namespace:Daq.viewModel"
    snet:ResxLocalizationProvider.DefaultAssembly="Daq"
    snet:ResxLocalizationProvider.DefaultDictionary="Language">
    <UserControl.Resources>
        <ResourceDictionary Source=
            "pack://application:,,,/Daq;component/template/DaqTemplateView.xaml" />
    </UserControl.Resources>
    <UserControl.DataContext>
        <vm:SiemensViewModel />
    </UserControl.DataContext>
    <ui:Card UniformCornerRadius="{DynamicResource WindowCornerRadius_Double}">
        <ContentControl Content="{Binding}" ContentTemplate="{DynamicResource DAQ}" />
    </ui:Card>
</UserControl>
```

**SiemensViewModel.cs** — 仅 22 行：

```csharp
using Daq.template;
using Snet.Siemens;
using static Snet.Siemens.SiemensData;

namespace Daq.viewModel;

public class SiemensViewModel : DaqTemplateViewModel<Basics>
{
    public SiemensViewModel()
    {
        BasicsData = new Basics();
        FileName = typeof(SiemensData).Name;
        Daq = SiemensOperate.Instance(BasicsData);
        GetAutoAllocatingParamVisibility = Daq.ExistsAutoAllocatingParam().Status
            ? Visibility.Visible : Visibility.Collapsed;
        Key = Daq.GetParam()?.GetSource<ParamModel>()?.Name;
    }
}
```

### 消息中间件 View 示例（MqttClientViewModel）

```csharp
using Daq.template;
using Snet.Mqtt.client;
using static Snet.Mqtt.client.MqttClientData;

namespace Daq.viewModel;

public class MqttClientViewModel : MqTemplateViewModel<Basics>
{
    public MqttClientViewModel()
    {
        BasicsData = new Basics();
        FileName = typeof(MqttClientData).Name;
        Mq = MqttClientOperate.Instance(BasicsData);
        Key = Mq.GetParam()?.GetSource<ParamModel>()?.Name;
    }
}
```

### 通信客户端 View 示例（TcpClientViewModel）

```csharp
using Daq.template;
using Snet.Core.communication.net.tcp.client;
using static Snet.Core.communication.net.tcp.client.TcpClientData;

namespace Daq.viewModel;

public class TcpClientViewModel : CommunicationClientTemplateViewModel<Basics>
{
    public TcpClientViewModel()
    {
        BasicsData = new Basics();
        FileName = typeof(TcpClientData).Name;
        Communication = TcpClientOperate.Instance(BasicsData);
        Key = "TcpClient";
        DataFormat = 1;  // 1 = HEX, 0 = ASCII
    }
}
```

### ViewModel 继承链总结

```
BindNotify
  ├── DaqTemplateViewModel<T>          ← 采集协议基类
  │     ├── SiemensViewModel
  │     ├── ModbusViewModel
  │     ├── OmronViewModel
  │     ├── OPC UA Client VM
  │     └── ... (30+ 协议)
  ├── MqTemplateViewModel<T>           ← MQ 客户端基类
  │     ├── MqttClientViewModel
  │     ├── KafkaViewModel
  │     └── RabbitMQ / NetMQ / Netty VM
  ├── MqServiceTemplateViewModel<T>    ← MQ 服务端基类
  │     ├── MqttServiceViewModel
  │     └── MqttWebSocketServiceViewModel
  ├── CommunicationClientTemplateViewModel<T> ← 通信客户端基类
  │     ├── TcpClientViewModel
  │     ├── WsClientViewModel
  │     └── SerialViewModel
  ├── CommunicationServiceTemplateViewModel<T> ← 通信服务端基类
  │     ├── TcpServiceViewModel
  │     └── WsServiceViewModel
  └── 独立 ViewModel
        ├── SvgViewModel
        ├── GifViewModel
        ├── AboutViewModel
        └── ...
```

---

## 🛡️ 第十六章：全局异常处理

WPF 应用中未捕获的异常会导致进程静默退出。需要三层全局捕获：

```csharp
// App.xaml.cs
public partial class App : Application
{
    private void OnStartup(object sender, StartupEventArgs e)
    {
        RegisterEvents();
        // ...
    }

    private void RegisterEvents()
    {
        // 第一层：Task 线程内未捕获异常
        TaskScheduler.UnobservedTaskException += (sender, e) =>
        {
            var ex = e.Exception as Exception;
            if (ex?.HResult == -2146233088) return; // 忽略特定错误码
            HandleException(ex);
            e.SetObserved();
        };

        // 第二层：UI 主线程未捕获异常
        this.DispatcherUnhandledException += (sender, e) =>
        {
            HandleException(e.Exception);
            e.Handled = true; // 标记已处理，防止进程退出
        };

        // 第三层：非 UI 子线程未捕获异常
        AppDomain.CurrentDomain.UnhandledException += (sender, e) =>
        {
            if (e.ExceptionObject is Exception ex)
                HandleException(ex);
        };
    }

    private async Task HandleException(Exception e)
    {
        // 构建完整错误消息
        string msg = $"{e.Source}\r\n{e.Message}\r\n\r\n{e.StackTrace}";

        // 弹出错误对话框
        await Application.Current.Dispatcher.InvokeAsync(async () =>
        {
            await MessageBox.Show(msg, "全局异常捕获",
                MessageBoxButton.OK, MessageBoxImage.Exclamation);
        }, DispatcherPriority.Loaded);

        // 写入本地日志文件
        LogHelper.Error(msg, "App.log", e);
    }
}
```

### 退出清理

```csharp
private void OnExit(object sender, ExitEventArgs e)
{
    InjectionWpf.ClearService();
    GC.SuppressFinalize(this);
    GC.Collect();
}
```

---

## 📝 第十七章：异步日志流水线

Daq 的生产级日志模式：`ConcurrentQueue` 入队 + 定时 50ms 批处理刷新 + UI 线程调度 + 10KB 自动裁剪。

### 日志流水线实现

```csharp
using System.Collections.Concurrent;
using System.Text;

public class LogPipeline
{
    private readonly ConcurrentQueue<string> _logQueue = new();
    private CancellationTokenSource? _logCts;

    /// <summary>启动日志消费循环</summary>
    /// <param name="flushIntervalMs">刷新间隔（毫秒），默认 50ms</param>
    public void Start(int flushIntervalMs = 50)
    {
        _logCts = new CancellationTokenSource();
        var token = _logCts.Token;

        Task.Run(async () =>
        {
            while (!token.IsCancellationRequested)
            {
                await Task.Delay(flushIntervalMs, token);
                FlushLogsToUI();
            }
        }, token);
    }

    /// <summary>停止日志循环</summary>
    public void Stop() => _logCts?.Cancel();

    /// <summary>入队一条日志</summary>
    public void Enqueue(string? msg, DateTime timestamp, bool showTime = true)
    {
        if (string.IsNullOrWhiteSpace(msg)) return;
        string log = showTime
            ? $"{timestamp:yyyy-MM-dd HH:mm:ss.ffffff} {msg}\r\n"
            : $"{msg}\r\n";
        _logQueue.Enqueue(log);
    }

    /// <summary>批量刷新到 UI 线程</summary>
    private void FlushLogsToUI()
    {
        if (Application.Current == null) return;

        StringBuilder sb = new();
        while (_logQueue.TryDequeue(out var log))
            sb.Append(log);

        if (sb.Length > 0)
        {
            Application.Current.Dispatcher.Invoke(() =>
            {
                // 超出 10KB 自动清空（防止 UI 卡顿）
                if (LogText?.Length > 10000)
                    LogText = string.Empty;
                LogText += sb.ToString();
            });
        }
    }

    /// <summary>绑定到 ViewModel 的日志文本属性</summary>
    public string LogText { get; set; }
}
```

### 滚动条自动跟底

配合 `EventCommand` 让 TextBox 始终显示最新日志：

```xml
<TextBox Text="{Binding InfoEvent}" AcceptsReturn="True" IsReadOnly="True"
         TextWrapping="Wrap" VerticalScrollBarVisibility="Hidden">
    <b:Interaction.Triggers>
        <b:EventTrigger EventName="TextChanged">
            <snet:EventCommand Command="{Binding InfoEventTextChanged}" />
        </b:EventTrigger>
    </b:Interaction.Triggers>
</TextBox>
```

```csharp
// ViewModel 中：TextChanged 触发 ScrollToEnd
public IAsyncRelayCommand InfoEventTextChanged =>
    new AsyncRelayCommand<TextChangedEventArgs>(e =>
    {
        var tb = e.Source.GetSource<TextBox>();
        tb.SelectionStart = tb.Text.Length;
        tb.ScrollToEnd();
        return Task.CompletedTask;
    });
```

---

## 📂 第十八章：文件对话框

`Win32Handler` 提供原生 Windows 文件打开/保存/目录选择对话框，支持文件类型过滤。

### 选择文件

```csharp
using Snet.Windows.Controls.handler;

// 单文件选择
var filters = new Dictionary<string, string>
{
    { "JSON 文件 (*.json)", "*.json" },
    { "所有文件 (*.*)", "*.*" },
};
string? filePath = Win32Handler.Select("请选择文件", false, filters);

// 多文件选择
string? filePath = Win32Handler.Select("请选择文件", true, filters);
```

### 选择文件夹

```csharp
string? folderPath = Win32Handler.Select("请选择文件夹", isFolder: true);
```

### 导入/导出模式示例

```csharp
// 导出 JSON 配置
public async Task ExportAsync()
{
    string path = Win32Handler.Select("请选择保存目录", isFolder: true);
    if (!string.IsNullOrEmpty(path))
    {
        string file = Path.Combine(path,
            $"Config[{DateTime.Now:yyyyMMddHHmmss}].json");
        File.WriteAllText(file, ConfigData.ToJson());
        await InfoEventLogShow("导出成功");
    }
}

// 导入 JSON 配置
public async Task ImportAsync()
{
    string? file = Win32Handler.Select("请选择配置文件", false,
        new() { { "JSON (*.json)", "*.json" } });
    if (!string.IsNullOrEmpty(file))
    {
        var data = File.ReadAllText(file).ToJsonEntity<T>();
        if (data != null)
            BasicsData = data;
    }
}
```

---

## 🎬 第十九章：FFmpeg GIF 转换器

`GifHandler` 基于 FFmpeg 将视频文件转换为高质量 GIF（palettegen + paletteuse 滤镜）。

### 使用方式

```csharp
using Daq.handler;

var gifHandler = GifHandler.Instance();

// 监听输出行（显示 FFmpeg 进度信息）
gifHandler.OnResponse = (line) =>
{
    Console.WriteLine($"FFmpeg: {line}");
};

// 监听转换结束
gifHandler.OnEnd = (success) =>
{
    if (success)
        Console.WriteLine("GIF 转换完成！");
    else
        Console.WriteLine("转换失败");
};

// 指定 FFmpeg 路径（默认: AppContext.BaseDirectory/lib/ffmpeg/ffmpeg.exe）
gifHandler.FFmpegTool = @"C:\Tools\ffmpeg.exe";

// 执行转换
gifHandler.RunConverter("input.mp4", "output.gif");
```

### 转换参数

```csharp
// 内部使用的 FFmpeg 命令（无需手动设置）：
// ffmpeg -i "input.mp4" -filter_complex
//   "fps=25,split [a][b];[a] palettegen=stats_mode=diff [p];[b][p] paletteuse=dither=bayer"
//   -y "output.gif"
```

---

## 🎨 第二十章：SVG 代码转换器

`SvgHandler.SvgCodeConverter` 将原始 SVG 内容转换为 WPF 兼容的 DrawingImage XAML 代码。

### 使用方式

```csharp
using Snet.Utility;

// 输入参数
string name = "MyIcon";          // 图标名称
string annotation = "我的图标";   // 注释
string svgContent = "<svg>...</svg>"; // 原始 SVG
string color = "{DynamicResource ImageColor}"; // 颜色绑定

// 转换
if (SvgHandler.SvgCodeConverter(name, annotation, svgContent, out string xamlCode, color))
{
    // xamlCode 即为可用的 DrawingImage XAML，可直接粘贴到 ResourceDictionary
    Console.WriteLine(xamlCode);
}
```

### 完整 SvgView 示例

```xml
<!-- 参数区 -->
<snet:TextBoxControl Hint="名称" Text="{Binding Name}" />
<snet:TextBoxControl Hint="注释" Text="{Binding Annotation}" />
<snet:TextBoxControl Hint="颜色" Text="{Binding Color}" />
<snet:ButtonControl Content="转换" Command="{Binding Transition}" />
<snet:ButtonControl Content="复制" Command="{Binding Copy}" />

<!-- 输入区 -->
<TextBox Text="{Binding InputData}" AcceptsReturn="True" />

<!-- 输出区 -->
<TextBox Text="{Binding OutData}" AcceptsReturn="True" IsReadOnly="True" />
```

---

## 🌳 第二十一章：OPC UA 节点浏览数据模型

用于构建 OPC UA 树形节点浏览器的结构化数据模型。

### OpcUaNodeBrowseStructuralBody

```csharp
// F:/Snet/Shunnet/Snet/Daq/data/OpcUaNodeBrowseStructuralBody.cs
public class OpcUaNodeBrowseStructuralBody
{
    /// <summary>节点 ID</summary>
    public string NodeId { get; set; }

    /// <summary>显示名称</summary>
    public string DisplayName { get; set; }

    /// <summary>节点类别</summary>
    public string NodeClass { get; set; }

    /// <summary>是否已展开（UI 绑定）</summary>
    public bool IsExpanded { get; set; }

    /// <summary>是否已选中（UI 绑定）</summary>
    public bool IsSelected { get; set; }

    /// <summary>子节点列表</summary>
    public List<OpcUaNodeBrowseStructuralBody> Children { get; set; } = new();
}
```

### OpcUaNodeBrowseMessageStructuralBody

```csharp
// 消息级别的节点浏览结果
public class OpcUaNodeBrowseMessageStructuralBody
{
    public string StatusMessage { get; set; }
    public List<OpcUaNodeBrowseStructuralBody> Nodes { get; set; } = new();
}
```

### 分页扩展（OPC UA ReferenceDescription）

```csharp
// 对 OPC UA ReferenceDescriptionCollection 进行分页
public static class PageHandler
{
    public static PagedResult<ReferenceDescription> ToPagedResult(
        this ReferenceDescriptionCollection source,
        int pageIndex,
        int pageSize = 25)
    {
        int skip = pageIndex * pageSize;
        var items = skip >= source.Count
            ? new List<ReferenceDescription>()
            : source.Skip(skip).Take(pageSize).ToList();

        return new PagedResult<ReferenceDescription>
        {
            Items = items,
            TotalCount = source.Count,
            PageIndex = pageIndex,
            PageSize = pageSize
        };
    }
}

// 配合 PageBarControl 实现 OPC UA 节点分页浏览
```

---

## 🏗️ 第二十二章：生产级应用架构实战

综合运用所有章节的技术，构建与 [Daq](https://github.com/shunnet) 同级的完整工业物联网桌面应用。

### 项目结构

```
MyApp/
├── App.xaml                  # 启动入口，加载主题
├── App.xaml.cs               # 全局异常处理 + 图标加载
├── MainWindow.xaml           # NavigationView 壳
├── MainWindowViewModel.cs    # 菜单数据源
├── Language.resx             # 中文默认资源
├── Language.en.resx          # 英文翻译资源
├── resources/
│   └── icons.xaml            # 自定义图标
├── template/                 # 可复用 DataTemplate
│   ├── DaqTemplateView.xaml
│   ├── MqTemplateView.xaml
│   ├── CommunicationClientTemplateView.xaml
│   ├── CommunicationServiceTemplateView.xaml
│   ├── MqServiceTemplateView.xaml
│   ├── DaqTemplateViewModel.cs
│   ├── MqTemplateViewModel.cs
│   ├── CommunicationClientTemplateViewModel.cs
│   ├── CommunicationServiceTemplateViewModel.cs
│   └── MqServiceTemplateViewModel.cs
├── view/                     # View（每个协议/功能一个）
│   ├── SiemensView.xaml + .cs
│   ├── ModbusView.xaml + .cs
│   ├── SvgView.xaml + .cs
│   └── ...
├── viewModel/                # ViewModel（对应 View）
│   ├── SiemensViewModel.cs
│   ├── ModbusViewModel.cs
│   ├── SvgViewModel.cs
│   └── ...
└── data/                     # 数据模型
    ├── ItemsControlBody.cs
    └── ...
```

### App.xaml — 启动配置

```xml
<Application x:Class="MyApp.App"
    xmlns:wpfui="http://schemas.lepo.co/wpfui/2022/xaml"
    Exit="OnExit" Startup="OnStartup">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!-- 深色主题 -->
                <ResourceDictionary Source=
                    "pack://application:,,,/Snet.Windows.Core;component/themes/DarkTheme.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

### App.xaml.cs — 全局配置

```csharp
public partial class App : Application
{
    public static readonly LanguageModel LanguageOperate = new(
        "MyApp", "Language", "MyApp.dll");

    private void OnStartup(object sender, StartupEventArgs e)
    {
        RegisterEvents();  // 三层全局异常捕捉
        IconsHandler.Loading(
            "pack://application:,,,/MyApp;component/resources/icons.xaml");
        InjectionWpf.Window<MainWindow, MainWindowViewModel>(true).Show();
    }

    private void OnExit(object sender, ExitEventArgs e)
    {
        InjectionWpf.ClearService();
        GC.Collect();
    }
}
```

### 完整的 DaqTemplateViewModel.cs

基于 Daq 实际代码的关键架构模式：

```csharp
public class DaqTemplateViewModel<T> : BindNotify
{
    // === 生命周期事件 ===
    public DaqTemplateViewModel()
    {
        // 订阅语言切换事件（响应标题刷新）
        LanguageHandler.OnLanguageEventAsync -= OnLanguageChangedAsync;
        LanguageHandler.OnLanguageEventAsync += OnLanguageChangedAsync;

        // UI 线程延迟初始化默认值
        Application.Current.Dispatcher.InvokeAsync(async () =>
        {
            ComboBoxSelectedItem = ComboBoxItemsSource[0];
            DataType = 20;       // 默认 String
            Length = 1;
            ToolTitle = await GetTitleAsync();
        }, DispatcherPriority.Loaded);

        StartLogLoop(); // 启动底层日志循环
    }

    // === 核心对象 ===
    public IDaq Daq { get; set; }          // 采集接口
    public T BasicsData { get; set; }       // 协议配置对象
    public string FileName { get; set; }    // 导出文件名
    public string Key { get; set; }         // 多语言 Key

    // === 双向绑定属性（Panel 可见性控制）===
    public Visibility GetAutoAllocatingParamVisibility { get; set; }
    public Visibility InteractionVisibility { get; set; } = Visibility.Visible;

    // === 命令：On / Off / Read / Write / Subscribe / UnSubscribe ===
    // === 命令：Inc (导入) / Exp (导出) ===
    // === 命令：InfoClear / DataClear / InteractionClear ===
    // === 命令：GetAutoAllocatingParam (获取协议类型参数) ===
    // === 三个日志区：InfoEvent / DataEvent / InteractionEvent ===

    // === 事件处理: On → 注册 Info/Data 异步事件 → 推送日志 ===
    // === 事件处理: 底层交互数据 LogNet_BeforeSaveToFile → 入队 ===
    // === 语言切换: 重新生成 ToolTitle (中/英) ===
    // === 导入: 文件选择 → 反序列化 → Dispose 旧对象 → 创建新实例 ===
    // === 导出: 文件夹选择 → BasicsData.ToJson() → 写入文件 ===
}
```

### ItemsControlBody — 通用列表项数据模型

```csharp
public class ItemsControlBody : BindNotify
{
    public string Name
    {
        get => GetProperty(() => Name);
        set => SetProperty(() => Name, value);
    }

    public string Description
    {
        get => GetProperty(() => Description);
        set => SetProperty(() => Description, value);
    }

    public string Path
    {
        get => GetProperty(() => Path);
        set => SetProperty(() => Path, value);
    }

    public bool IsSelected
    {
        get => GetProperty(() => IsSelected);
        set => SetProperty(() => IsSelected, value);
    }
}
```

### 应用启动流程总览

```
1. App.OnStartup
   ├── RegisterEvents()           ← 三层全局异常捕获
   ├── IconsHandler.Loading(...)  ← 加载自定义图标集
   └── InjectionWpf.Window<MainWindow, MainWindowViewModel>()
         └── WindowBase.SourceInitialized
               └── 加载 DarkTheme / LightTheme
                     └── WindowBase 模板渲染
2. MainWindowViewModel 构造函数
   ├── 构建 NavigationView 菜单树（Daq / Mq / 通信 / 工具）
   └── 构建底部菜单（关于）
3. 用户点击菜单项 → NavigationView 路由到对应的 View
   └── View 构造 → 设置 DataContext = new ViewModel()
         └── ViewModel 构造 → 初始化 BasicsData + Daq/Mq 实例
               └── 加载 PropertyControl → 用户可编辑参数
```

---

## 🔌 第二十三章：插件热插拔架构

基于 .NET `AssemblyLoadContext(isCollectible: true)` 实现完整的运行时插件热加载/卸载，无需重启进程。Daq 同时支持 **DAQ 采集插件** 和 **MQ 消息插件** 两种类型。

### 加载流程

```
上传 ZIP 插件包 → 自动解压到插件目录 → 创建可回收 AssemblyLoadContext
→ 通过 MemoryStream 流式加载 DLL（无文件锁）
→ 扫描并实例化 IDaq / IMq 接口 → 注册到 IOC 容器 → 开始采集
```

### 卸载流程

```
停止采集 → 释放插件实例（IAsyncDisposable）
→ 移除 IOC 注册 → 卸载 AssemblyLoadContext
→ GC 回收 → 删除插件文件
```

### 技术亮点

| 特性 | 说明 |
|:---|:---|
| 可回收上下文 | `AssemblyLoadContext(isCollectible: true)`，卸载后可被 GC 完整回收 |
| 流式加载 | `MemoryStream` + `LoadFromStream` 加载 DLL，避免文件锁定，卸载后立即可删除 |
| 类型一致性 | 共享接口程序集（`IDaq`、`IMq`）始终从默认上下文加载，确保 `as` 类型转换正确 |
| 并发安全 | `ConcurrentDictionary` 管理插件实例与上下文，支持多插件并发操作 |
| 双插件类型 | 同时支持 **DAQ** 和 **MQ** 两种插件接口 |
| PDB 符号 | 加载时自动附带 PDB 调试符号文件 |

### 插件契约

```csharp
// DAQ 采集插件 — 必须实现 IDaq 接口
public interface IDaq : IOn, IOff, IRead, IWrite, ISubscribe,
    IGetStatus, IEvent, IGetParam, ICreateInstance, ILog,
    IWA, IGetObject, ILanguage, IDisposable, IAsyncDisposable { }

// MQ 消息插件 — 必须实现 IMq 接口
public interface IMq : IOn, IOff, IProducer, IConsumer,
    IGetStatus, IEvent, IGetParam, ICreateInstance, ILog,
    ILanguage, IDisposable, IAsyncDisposable { }
```

### PluginHandlerCore 核心 API

```csharp
// 获取插件 UI 配置
var plugins = PluginHandlerCore.GetPluginUIConfig<ObservableCollection<PluginListModel>>(configPath);

// 初始化一个插件实例
PluginHandlerCore.PluginOperate.InitPlugin(
    pluginPath,           // ZIP/DLL 路径
    interfaceFullName);   // "IDaq" 或 "IMq"
```

---

## 📊 第二十四章：ScottPlot 实时图表

基于 [ScottPlot](https://scottplot.net) 的多曲线实时图表系统，支持主题跟随、皮肤切换、历史数据。

### ChartHandler 封装扩展

```
// WpfPlot 扩展方法，位于 Snet.Iot.Daq.chart
```

```csharp
using ScottPlot;
using ScottPlot.WPF;

// 创建散点连线（X + Y 数组）
IPlottable line1 = wpfPlot.Create(xs: times, ys: values, "温度", Colors.Red);

// 创建信号线（仅 Y 数组，自动生成 X 索引）
IPlottable line2 = wpfPlot.Create(ys: samples, "压力", Colors.Blue);

// 移除指定线条
wpfPlot.Remove(line1, out Exception? ex);

// 移除所有线条
wpfPlot.RemoveAll(out ex);

// 自动调整坐标轴并刷新
wpfPlot.Adjust(out ex);
```

### ChartLine 自定义控件

```xml
<!-- ChartLine.xaml — 对 WpfPlot 的封装，添加图例、工具栏等 -->
<local:ChartLine x:Name="TemperatureChart" />
```

```csharp
// ChartData — 包含图表配置和运行时数据
public class ChartData
{
    public string Title { get; set; }
    public Color LineColor { get; set; }
    public List<double> XValues { get; set; }
    public List<double> YValues { get; set; }
    public double MaxPoints { get; set; } = 1000;  // 最大数据点数（防内存溢出）
}
```

### XAML 使用

```xml
<local:ChartLine x:Name="MainChart" Margin="5" />
```

```csharp
// 动态添加数据（滚动窗口模式）
MainChart.ScrollData(temperatureSeries, DateTime.Now, 25.6);
MainChart.ScrollData(pressureSeries, DateTime.Now, 1.2);
```

---

## 🖥️ 第二十五章：系统监控仪表盘

基于 [LibreHardwareMonitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor) 的 CPU / GPU / RAM 实时硬件监控。

### SystemMonitoring 核心

```csharp
// F:/Snet/Daq/Snet.Iot.Daq/utility/SystemMonitoring.cs
public class SystemMonitoring : IDisposable
{
    public float CpuLoad { get; private set; }        // CPU 使用率 (%)
    public float CpuTemperature { get; private set; }  // CPU 温度 (°C)
    public float GpuLoad { get; private set; }         // GPU 使用率 (%)
    public float GpuTemperature { get; private set; }  // GPU 温度 (°C)
    public float RamUsed { get; private set; }         // 已用内存 (GB)
    public float RamTotal { get; private set; }        // 总内存 (GB)
    public float RamLoad { get; private set; }         // 内存使用率 (%)

    public void Start(int intervalMs = 1000) { /* 启动定时采集 */ }
    public void Stop() { /* 停止定时采集 */ }

    // 更新回调（每 intervalMs 触发一次）
    public event Action<SystemMonitoring> OnDataUpdated;
}
```

### 配合 LedGaugeControl 的仪表盘面板

```xml
<!-- CPU 面板 -->
<GroupBox Header="CPU">
    <StackPanel>
        <snet:LedGaugeControl Color="Blue" IsOn="{Binding CpuAlive}"
            IsFlashing="{Binding CpuFlashing}" />
        <TextBlock Text="{Binding CpuLoad, StringFormat={}{0:F1}%}" />
        <TextBlock Text="{Binding CpuTemperature, StringFormat={}{0:F1}°C}" />
    </StackPanel>
</GroupBox>

<!-- RAM 面板 -->
<GroupBox Header="RAM">
    <StackPanel>
        <TextBlock Text="{Binding RamUsed, StringFormat={}{0:F1} GB}" />
        <ProgressBar Value="{Binding RamLoad}" Maximum="100" />
    </StackPanel>
</GroupBox>
```

---

## ❄️ 第二十六章：雪花粒子特效

WPF 纯渲染实现的雪花飘落动画，支持主题色跟随。

### SnowflakeEffect 核心

```csharp
// 使用 Canvas 作为容器，按 50ms 定时器驱动逐帧动画
public class SnowflakeEffect : IDisposable
{
    private readonly Canvas _canvas;
    private readonly List<SnowFlake> _snowflakes = new(200);
    private readonly Color _baseColor;      // 基础颜色（跟随主题）
    private Timer _timer;

    public SnowflakeEffect(Canvas canvas, int count = 200, Color? color = null)
    {
        _canvas = canvas;
        _baseColor = color ?? Colors.White;
        // 随机初始化雪花：位置、大小、透明度、速度、摆动角度
        InitializeSnowflakes(count);
        // 启动 50ms 定时器驱动动画
        _timer = new Timer(Tick, null, 0, 50);
    }

    private void Tick(object? state)
    {
        // 每帧更新每个雪花：Y += Speed, X += sin(Angle) * Step
        // 触底回归顶部，利用 RenderTransform 避免重建 UI 元素
    }

    public void UpdateColor(Color newColor) { /* 主题切换时更新颜色 */ }
    public void Dispose() { _timer?.Dispose(); ClearSnowflakes(); }
}
```

### XAML 集成

```xml
<Grid>
    <!-- 主页内容 -->
    <Grid x:Name="HomeContent"> ... </Grid>

    <!-- 雪花覆盖层（IsHitTestVisible=False 不拦截鼠标事件） -->
    <Canvas x:Name="SnowCanvas" IsHitTestVisible="False" />
</Grid>
```

```csharp
// 在 Home 页面加载时启动
var effect = new SnowflakeEffect(SnowCanvas, count: 200);

// 皮肤切换时同步颜色
SkinHandler.OnSkinEvent += (sender, e) =>
{
    effect.UpdateColor(e.SkinType == SkinType.Dark
        ? Colors.White : Colors.DarkGray);
};
```

---

## 🔒 第二十七章：单实例保护

基于命名 Mutex + NamedPipe 的单实例管理器，第二实例启动时自动激活已有窗口。

### SingleInstanceHandler

```csharp
// 构造函数 — 创建全局 Mutex 判重
var single = new SingleInstanceHandler("MyApp", out bool isFirst);

if (!isFirst)
{
    // 第二实例：发送参数给第一实例，然后退出
    single.SignalFirstInstance(new[] { "--file", "config.json" });
    single.Dispose();
    Application.Current.Shutdown(0);
    return;
}

// 第一实例：监听管道消息
single.SignalReceived += OnWakeup;
single.RegisterMainWindow(mainWindow);
```

### 技术实现

| 组件 | 用途 |
|:---|:---|
| `Global\{AppName}_{UserName}` Mutex | 进程唯一性判定，用户级隔离 |
| `NamedPipeServerStream` (异步) | 接收第二实例的启动参数 |
| `NamedPipeClientStream` | 向第一实例发送消息 |
| `BringToFront()` | 恢复最小化、Show、Focus 三段激活 |

### App.xaml.cs 完整集成

```csharp
private void OnStartup(object sender, StartupEventArgs e)
{
    SingleInstance(e);                    // 判重
    Init();                               // IOC / 数据库 / 插件
    RegisterEvents();                     // 全局异常
    IconsHandler.Loading("...");          // 图标

    MainWindow window = InjectionWpf.Window<MainWindow, MainWindowModel>(true);
    window.Show();
    singleInstance.RegisterMainWindow(window); // Show() 之后注册 HWND
}

private void Init()
{
    // 注册 PropertyControl 单例到 IOC
    var control = new PropertyControl { ButtonVisibility = Visibility.Visible };
    InjectionWpf.AddService(s => s.AddSingleton(control));

    // 初始化 SQLite 数据库
    GlobalConfigModel.sqliteOperate.CreateTable<AddressModel>();

    // 加载已安装插件并启动
    var plugins = PluginHandlerCore.GetPluginUIConfig<...>(configPath) ?? new();
    foreach (var plugin in plugins)
        PluginHandlerCore.PluginOperate.InitPlugin(plugin.Path, interfaceName);

    // 加载配置数据
    PluginHandler.GetAllPlugin();
    AddressHandler.GetAllAddress();
    ProjectHandler.GetAllProject();
}
```

---

## 🏠 第二十八章：Console 控制台日志系统

Daq 内置的彩色标签日志控制台，通过正则匹配日志标签自动标注颜色。

### 颜色标签映射

```csharp
// App.xaml.cs / App.EditModels
private static List<EditModel> GetEditModels() =>
[
    new() { Name = "[ Info ]",                Color = "#4CAF50" },  // 绿色
    new() { Name = "[ Error ]",               Color = "#F44336" },  // 红色
    new() { Name = "异常",                    Color = "#F44336" },
    new() { Name = "Exception",               Color = "#F44336" },
    new() { Name = "[ Mq ]",                  Color = "#2196F3" },  // 蓝色
    new() { Name = "[ Daq ]",                 Color = "#2196F3" },
    new() { Name = "[ MqttService ]",         Color = "#2196F3" },
    new() { Name = "[ OpcUaService ]",        Color = "#2196F3" },
    new() { Name = "TRUE",                    Color = "#4CAF50" },
    new() { Name = "FALSE",                   Color = "#F44336" },
    new() { Name = ">",                       Color = "#FBC31D" },  // 黄色
    new() { Name = "<",                       Color = "#FBC31D" },
];
```

### Console.xaml 控制台视图

```xml
<!-- 设备选择 + 日志输出 TextBox -->
<ComboBox ItemsSource="{Binding Devices}" SelectedItem="{Binding SelectedDevice}" />
<TextBox Text="{Binding ConsoleOutput}"
         AcceptsReturn="True" IsReadOnly="True"
         TextWrapping="Wrap" VerticalScrollBarVisibility="Auto"
         TextChanged="{snet:EventCommand Command={Binding TextChanged}}" />
```

### EditBindingHandler 彩色渲染

```
// F:/Snet/WpfMUI/Snet.Windows.Controls/handler/EditBindingHandler.cs
// 通过正则扫描 TextBox.Text，匹配 EditModel.Name 标签，
// 在 RichTextBox/FlowDocument 中对匹配行应用对应 Color 的高亮效果。
```

---

## ⚙️ 第二十九章：地址字节级解析器

`AddressSettings` 和 `AutoPackHandler` 实现可视化的字节/位/编码/数据格式配置，支持自定义协议解析。

### 核心概念

| 概念 | 说明 |
|:---|:---|
| **单地址** | 一个 `AddressDetails` 对应一个读取点位 |
| **自动组包** | 离散地址智能合并为批量读取，减少通信开销 |
| **字节解析** | `BytesHandler.TransformAsync` 根据 DataType/EncodingType/BitOffset 自动转换 |
| **数据格式** | String / Int / Float / Bool / Byte[] 及各自数组类型共 20 种 |

### AddressModel 核心字段

```csharp
// F:/Snet/Daq/Snet.Iot.Daq.Core/data/AddressModelCore.cs
public class AddressModelCore
{
    public string SN { get; set; }              // 唯一标识（机台号）
    public string AddressName { get; set; }     // 设备地址（如 DB1.0、40001）
    public DataType DataType { get; set; }      // 数据类型（String/Int/Float/Bool...）
    public int Length { get; set; }             // 读取长度
    public EncodingType EncodingType { get; set; } // 编码格式（ASCII/UTF8/HEX...）
    public AddressType AddressType { get; set; }   // 地址类型（实数/虚拟）
    public bool IsEnable { get; set; }          // 是否启用
    public AddressMq? AddressMqParam { get; set; } // MQ 自动转发配置
}
```

### 自动组包算法

```csharp
// F:/Snet/Daq/Snet.Iot.Daq.Core/handler/AutoPackHandler.cs
// 将离散地址列表按协议类型分组，合并连续/临近地址为批量读取
public class AutoPackHandler
{
    // 输入: List<AddressModel> 离散地址
    // 输出: List<AddressBatch> 组包后的批次
    // 策略: 按 (DeviceSN, ProtocolType) 分组，按 AddressName 排序后合并连续地址
}
```

### AddressSettings 视图

```xml
<!-- AddressSettings.xaml — 可视化地址配置面板 -->
<GroupBox Header="地址配置">
    <DataGrid ItemsSource="{Binding Addresses}" AutoGenerateColumns="False">
        <DataGrid.Columns>
            <DataGridTextColumn Header="SN" Binding="{Binding SN}" />
            <DataGridTextColumn Header="地址" Binding="{Binding AddressName}" />
            <snet:ComboBoxControl Header="数据类型" ItemsSource="{Binding DataTypes}"
                SelectedItem="{Binding DataType}" />
            <DataGridTextColumn Header="长度" Binding="{Binding Length}" />
            <DataGridCheckBoxColumn Header="启用" Binding="{Binding IsEnable}" />
        </DataGrid.Columns>
    </DataGrid>
    <snet:ButtonControl Content="自动组包" Command="{Binding AutoPackCommand}" />
</GroupBox>
```

---

## 🔗 相关链接

| 链接 | 说明 |
|:---|:---|
| [WpfMUI 仓库](https://github.com/shunnet/WpfMUI) | Snet.Windows.Core / Controls 源代码 |
| [Daq 仓库](https://github.com/shunnet/Daq) | 基于本框架的完整生产级桌面应用（插件热插拔、OPC UA Server、MQTT Broker、图表、系统监控） |
| [NuGet 包](https://www.nuget.org/profiles/shun) | Snet.Windows.Core / Controls |
| [Snet 官网](https://shunnet.top) | 文档和演示视频 |
| [演示视频](https://shunnet.top/7EUf6) | 框架功能演示 |

---

## 📦 版本历史

| 版本 | 日期 | 变更 |
|:---|:---|:---|
| 1.0.0.7 | 2026-07-14 | 初始版本：WindowBase、6 大控件、MVVM、主题、多语言、拖拽、托盘、PropertyGrid、NavigationView、模板视图架构、全局异常处理、异步日志、文件对话框、GIF/SVG 转换、OPC UA 节点浏览、完整架构实战、Daq 插件热插拔、ScottPlot 图表、系统监控、雪花特效、单实例保护、Console 控制台、地址字节级解析器（共 29 章） |
