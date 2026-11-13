# WpfUI-Skill — Modern WPF UI Development Skill

**Version:** 1.0.1.10
**Author:** Shun
**License:** MIT

🌐 [中文 README](README.md) | [English README](README.en.md)

## 🎯 Positioning

🖥️ **UI development** — build modern desktop applications on the Snet WPF framework

This skill requires `Snet.Windows.Controls` first, then `Wpf.Ui.Controls` only for missing capabilities. Native interactive-control fallback is prohibited. Layout panels and resource/template primitives follow the explicit SKILL.md boundary. Consult [Debug](https://github.com/shunnet/Debug) and [the Daq WPF frontend](https://github.com/shunnet/Daq/tree/main/Snet.Iot.Daq).

## ✨ Capabilities

- 🪟 `WindowBase` custom windows (title-bar buttons, DPI awareness, loading animation)
- 🎨 Dark/light theme one-click switch (Material Design + Wpf.Ui)
- 🌍 Chinese/English multi-language (`LocExtension` / `BLoc` markup extensions, runtime hot-switch)
- 🧩 MVVM architecture (`BindNotify` + `InjectionWpf` + `EventCommand`)
- 🎛️ Built-in controls (full): ButtonControl / ComboBoxControl / TextBoxControl / LedGaugeControl / PageBarControl / TextEditorControl / PropertyControl / NotifyIcon / message dialogs
- 🧰 property/wpf sub-controls (20+): ColorPicker / FilePicker / DirectoryPicker / RadioButtonList / SpinControl / FormattingTextBox / HeaderedEntrySlider / TextBoxEx / EditableTextBlock / CheckMark / EnumMenuItem / DockPanelSplitter / StackPanelEx / PopupBox / LinkBlock / PropertyGrid / DataGrid (embedded) / TreeListBox / ItemsBag / PropertyDialog / WizardDialog (see SKILL.md §6.2)
- 🔧 PropertyGrid property editor (40+ annotations, ColorPicker/FilePicker/DataGrid)
- 🖱️ Drag controls (8-direction resize + drag to move + animated creation)
- 💬 Message dialogs (OK / OKCancel / Yes / YesNo — ⚠️ same name as System.Windows.MessageBox, use an alias; see SKILL.md §8)
- 🔔 System tray (NotifyIcon XAML element + Register/click events; TrayManager is internal and not accessible; runtime updates and Closed cleanup)

## 💬 Example Scenarios

```
🗣️ "Create a WPF monitoring panel — dark theme, zh/en switching, data table on the left, property editor on the right"
🗣️ "Build a PLC debugging tool with a connect button, LED status lights, and a parameter configuration panel"
🗣️ "Build a draggable device layout editor with movable and resizable controls"
🗣️ "Build a multi-tab settings window that edits configuration with PropertyGrid"
```

- 💬 `DialogHandler.ShowAsync` custom dialogs with per-window hosts and the shared message-box queue; see SKILL.md §8 for close/retemplate/cancellation semantics

## 📄 Files

- `SKILL.md` — main skill file (29 chapters, incl. §6.2 built-in sub-control overview)
