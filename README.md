<h1 align="center">📦 SKILLS</h1>

<p align="center">
  <img width="120" height="120" src="https://api.shunnet.top/pic/nuget.png" alt="Snet Logo"/>
</p>

<p align="center">
  <b>🛠️ Snet 工业物联网技能集</b>
</p>

<p align="center">

  <img src="https://img.shields.io/badge/.NET-10.0%2B-purple.svg"/>
  <img src="https://img.shields.io/badge/license-MIT-green"/>
  <img src="https://img.shields.io/badge/version-1.0.0.7-blue"/>
  <img src="https://img.shields.io/badge/skills-3-orange"/>
  <img src="https://img.shields.io/github/stars/shunnet/SKILLS?style=social"/>

</p>

<p align="center">
  🔄 覆盖从「<b>采集转发</b>」→「<b>插件开发</b>」→「<b>桌面应用</b>」的完整 Snet 生态
</p>

<p align="center">
  <a href="https://shunnet.top"><b>🌐 官方网站</b></a> ·
  <a href="https://www.nuget.org/profiles/shun"><b>📚 NuGet</b></a> ·
  <a href="https://github.com/shunnet"><b>💻 GitHub</b></a> ·
  <a href="https://github.com/shunnet/Daq"><b>🔌 Daq 工具</b></a>
</p>



## 📋 技能一览

| 技能 | 目录 | 定位 | 🗣️ 用户说一句话 → |
|------|------|------|------|
| **🔌 DAQ-Skill** | `DAQ-Skill/` | 🏭 使用库 — 数据采集与转发 | "连接西门子S7-1500，IP 192.168.0.1，读取DB1.0，MQTT转发到broker" |
| **🧩 PluginDev-Skill** | `PluginDev-Skill/` | ⚙️ 开发插件 — 自定义协议对接 Daq 工具 | "开发一个温湿度传感器插件，TCP发送 01 03 00 00 00 02，解析响应字节" |
| **🖥️ WpfUI-Skill** | `WpfUI-Skill/` | 🪟 界面开发 — WPF 桌面应用 | "创建一个监控面板，深色主题，中英文切换，LED状态灯，属性编辑器" |



## 🔌 DAQ-Skill — 工业物联网数据采集

> **一句话：** 用户描述需求 → AI 自动生成完整可运行代码

### 🎯 用途

帮助用户直接使用 Snet 框架完成数据采集与转发，无需开发插件。

### ✨ 能力

- 🏭 **30+ 种工业协议** — 西门子/Modbus/三菱/欧姆龙/OPC UA/罗克韦尔/台达/基恩士/松下/倍福/汇川/英威腾/麦格米特
- 🗄️ **数据库采集** — SqlServer/MySQL/Oracle/SQLite + TEP 非标设备
- 📡 **5 种消息中间件转发** — MQTT/Kafka/RabbitMQ/NetMQ/Netty
- 🎮 **内置模拟库** — 无需真实 PLC 即可测试
- ⚡ **自动数据转发** — 配置 `AddressMqParam`，框架自动完成 读取→格式化→MQ生产

### 💬 场景示例

```
🗣️ "连接西门子S7-1500，读DB1.0，MQTT转发到1883端口"
🗣️ "连接Modbus TCP 192.168.0.2:502，读取40001-40010，转发到Kafka"
🗣️ "连接OPC UA，读取所有Tag，通过MQTT转发"
🗣️ "从SqlServer数据库查询温度数据，MQTT转发"
```

### 🗣️ 交互流程

AI 先用大白话问用户（🏷️ 牌子型号、🔌 怎么连的、📍 IP多少、📊 读什么数据、📨 发到哪里），确认后再生成代码。

**📄 文件：** `SKILL.md`（55KB，14章）



## 🧩 PluginDev-Skill — Daq 工具插件开发

> **一句话：** 用户描述设备 → AI 按契约生成插件代码 → 打包 ZIP 上传 Daq 工具

### 🎯 用途

帮助用户开发自定义协议插件，部署到 [Snet.Iot.Daq](https://github.com/shunnet/Daq) 工具中使用。

### ✨ 能力

- 📋 **严格插件契约** — 8 个抽象方法 + 3 个必须属性
- 📡 **5 种内置通信类** — TCP/UDP/WebSocket/Serial/HTTP
- 💾 **数据缓存** — 进程缓存 `ProcessCacheOperate` + 跨进程共享缓存 `ShareCacheOperate`
- 🔍 **反射调用** — `ReflectionOperate` 动态加载外部 DLL、调用方法、注册事件
- 📐 **数据类规范** — Basics 继承 SCData、ProtocolType 枚举、Attribute 标注
- 🔄 **核心数据链** — `采集原始数据 → AddressHandler.ExecuteDispose → 自动类型转换+反射解析+MQ转发`
- 📦 **打包热插拔** — ZIP → 上传 Daq 工具 → 运行时加载/卸载

### 🔑 核心原则

| 🧠 AI 自行决定 | 📏 硬性契约 |
|-|-|
| 采集什么、用什么方式 | 数据怎么封装、方法返回什么 |

### 💬 场景示例

```
🗣️ "开发一个温湿度传感器插件，TCP 发送 Modbus 帧，解析温度湿度"
🗣️ "开发一个 HTTP JSON 传感器插件，GET API 返回 JSON 数据"
🗣️ "开发一个文件监控插件，FileSystemWatcher 监控日志文件"
```

### 🗣️ 交互流程

AI 先用大白话问用户（🏷️ 设备是什么、🔌 怎么连的、📋 协议规矩是什么、📊 数据长什么样），确认后再生成代码。

**📄 文件：** `SKILL.md`（41KB，10章）



## 🖥️ WpfUI-Skill — WPF 桌面应用开发

> **一句话：** 用户描述界面 → AI 生成完整的 WPF 桌面应用程序

### 🎯 用途

帮助用户基于 Snet WPF 框架快速构建现代化桌面应用，覆盖窗口管理、MVVM绑定、皮肤切换、多语言、内置控件等。

### ✨ 能力

- 🪟 **WindowBase 窗口基类** — 自定义标题栏、DPI感知、加载动画、Win32 集成
- 🎨 **深色/浅色主题** — Material Design + Wpf.Ui 双引擎，一键切换，JSON 持久化
- 🌍 **中英文多语言** — `LocExtension` / `BLoc` XAML 标记扩展，运行时热切换
- 🧩 **MVVM 架构** — `BindNotify` 表达式属性基类 + `InjectionWpf` DI容器 + EventCommand
- 🎛️ **6 大内置控件** — ButtonControl / ComboBoxControl / TextBoxControl / LedGaugeControl / PageBarControl / PropertyControl
- 🔧 **PropertyGrid 属性编辑器** — 40+ 注解，分类/可见性/条件启用/滑块/文件选择器/颜色选择器
- 🖱️ **拖拽控件** — 8向缩放手柄 + 拖拽移动 + 动画拖拽创建
- 💬 **消息对话框** — OK / OKCancel / Yes / YesNo，Material Design 风格
- 🔔 **系统托盘** — NotifyIcon 最小化到托盘，右键菜单

### 💬 场景示例

```
🗣️ "创建一个WPF监控面板，深色主题，中英文切换，左边数据表格，右边属性编辑器"
🗣️ "做一个PLC调试工具，有连接按钮、LED状态灯、参数配置面板"
🗣️ "做一个可拖拽的设备布局编辑器，控件可以拖动和缩放"
🗣️ "做一个多页签的设置窗口，用PropertyGrid编辑配置"
```

### 🗣️ 交互流程

AI 先用大白话问用户（🏷️ 窗口需求、🎨 主题偏好、🌍 语言需求、🎛️ 需要哪些控件、📊 数据绑定结构），确认后生成完整 XAML + C# 代码。

**📄 文件：** `SKILL.md`（13章）



## 🔗 技能关系

```
┌─ 🔌 DAQ-Skill ───────────────────────────────────────┐
│  使用现成协议库（Snet.Siemens / Modbus / OPC ...）    │
│  → 配置连接参数 + 地址 → 采集 → 订阅 → 自动转发      │
└───────────────────────┬───────────────────────────────┘
                        │
                        │ ⬇️ 如果现成协议不满足需求...
                        ↓
┌─ 🧩 PluginDev-Skill ─────────────────────────────────┐
│  开发自定义插件（继承 DaqAbstract，实现 8 个方法）     │
│  → 用内置通信类(TcpClientOperate等) → Read/Write     │
│  → 数据经 ExecuteDispose 处理 → 自动类型转换+转发     │
│  → 打包 ZIP → 上传 Daq 工具 → 热插拔加载             │
└───────────────────────┬───────────────────────────────┘
                        │
                        │ ⬇️ 需要桌面应用管理采集...
                        ↓
┌─ 🖥️ WpfUI-Skill ─────────────────────────────────────┐
│  构建 WPF 桌面监控/管理应用                            │
│  → WindowBase 窗口 + MVVM 架构                        │
│  → 深色/浅色主题 + 中英文多语言                       │
│  → Button/ComboBox/TextBox/LedGauge/PageBar          │
│  → PropertyGrid 属性编辑器 + 拖拽控件                 │
│  → 消息对话框 + 系统托盘                              │
└───────────────────────────────────────────────────────┘
```

**💡 一句话总结：** DAQ-Skill 用好现成协议，PluginDev-Skill 开发新协议，WpfUI-Skill 构建管理界面——覆盖从数据采集到人机交互的完整链路。


## 🌍 相关链接

| 🔗 | 链接 | 说明 |
|---|---|---|
| 🌐 | [官网](https://shunnet.top) | 官方网站 |
| 📚 | [NuGet](https://www.nuget.org/profiles/shun) | NuGet 包仓库 |
| 💻 | [GitHub](https://github.com/shunnet) | 源代码仓库 |
| 🔌 | [Daq 工具](https://github.com/shunnet/Daq) | 插件化数采工具 |
| 📝 | [博客](https://blog.shunnet.top) | 技术博客 |



<p align="center">
  <b>🙏 感谢使用 Snet 工业物联网技能集</b>
</p>
