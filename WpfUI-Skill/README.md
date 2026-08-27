# WpfUI-Skill — WPF 现代化界面开发技能

**版本:** 1.0.1.3
**作者:** Shun
**许可证:** MIT

🌐 [中文 README](README.md) | [English README](README.en.md)

## 🎯 定位

🖥️ **界面开发** — 基于 Snet WPF 框架构建现代化桌面应用

## ✨ 能力

- 🪟 `WindowBase` 自定义窗口（标题栏按钮、DPI 感知、加载动画）
- 🎨 深色/浅色主题一键切换（Material Design + Wpf.Ui）
- 🌍 中英文多语言（`LocExtension` / `BLoc` 标记扩展，运行时热切换）
- 🧩 MVVM 架构（`BindNotify` + `InjectionWpf` + `EventCommand`）
- 🎛️ 内置控件（全）：ButtonControl / ComboBoxControl / TextBoxControl / LedGaugeControl / PageBarControl / TextEditorControl / PropertyControl / NotifyIcon / 消息对话框
- 🧰 property/wpf 子控件（20+）：ColorPicker / FilePicker / DirectoryPicker / RadioButtonList / SpinControl / FormattingTextBox / HeaderedEntrySlider / TextBoxEx / EditableTextBlock / CheckMark / EnumMenuItem / DockPanelSplitter / StackPanelEx / PopupBox / LinkBlock / PropertyGrid / DataGrid（内置版）/ TreeListBox / ItemsBag / PropertyDialog / WizardDialog（见 SKILL.md §6.2）
- 🔧 PropertyGrid 属性编辑器（40+ 注解，ColorPicker/FilePicker/DataGrid）
- 🖱️ 拖拽控件（8 向缩放 + 拖拽移动 + 动画创建）
- 💬 消息对话框（OK / OKCancel / Yes / YesNo，⚠️ 与 System.Windows.MessageBox 同名需用别名，见 SKILL.md §8）
- 🔔 系统托盘（NotifyIcon XAML 元素 + Register/点击事件，TrayManager 为 internal 不可访问）

## 💬 场景示例

```
🗣️ "创建一个 WPF 监控面板，深色主题，中英文切换，左边数据表格，右边属性编辑器"
🗣️ "做一个 PLC 调试工具，有连接按钮、LED 状态灯、参数配置面板"
🗣️ "做一个可拖拽的设备布局编辑器，控件可以拖动和缩放"
🗣️ "做一个多页签的设置窗口，用 PropertyGrid 编辑配置"
```

## 📄 文件

- `SKILL.md` — 主技能文件（29 章，含 §6.2 内置子控件全览）
