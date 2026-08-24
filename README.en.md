<h1 align="center">📦 SKILLS</h1>

<p align="center">
  <img width="120" height="120" src="https://api.shunnet.top/pic/nuget.png" alt="Snet Logo"/>
</p>

<p align="center">
  <b>🛠️ Snet Industrial IoT Skills Collection</b>
</p>

<p align="center">
  <a href="README.md"><b>🌐 中文</b></a> · <a href="README.en.md"><b>🇺🇸 English</b></a>
</p>

<p align="center">

  <img src="https://img.shields.io/badge/.NET-10.0%2B-purple.svg"/>
  <img src="https://img.shields.io/badge/license-MIT-green"/>
  <img src="https://img.shields.io/badge/version-1.0.1.5-blue"/>
  <img src="https://img.shields.io/badge/skills-4-orange"/>
  <img src="https://img.shields.io/github/stars/shunnet/SKILLS?style=social"/>

</p>

<p align="center">
  🔄 Covers the complete Snet ecosystem from <b>acquisition & forwarding</b> → <b>plugin development</b> → <b>desktop applications</b>
</p>

<p align="center">
  <a href="https://snet.cn"><b>🌐 Official Site</b></a> ·
  <a href="https://www.nuget.org/profiles/shun"><b>📚 NuGet</b></a> ·
  <a href="https://github.com/shunnet"><b>💻 GitHub</b></a> ·
  <a href="https://github.com/shunnet/Daq"><b>🔌 Daq Tool</b></a>
</p>



## 📋 Skills at a Glance

| Skill | Directory | Position | 🗣️ One Sentence From the User → |
|------|------|------|------|
| **🔌 DAQ-Skill** | `DAQ-Skill/` | 🏭 Using the library — data acquisition & forwarding | "Connect to Siemens S7-1500 at 192.168.0.1, read DB1.0, forward to MQTT broker" |
| **📡 MQ-Skill** | `MQ-Skill/` | 📨 Using the library — messaging middleware | "Connect to MQTT at 127.0.0.1:1883, subscribe to factory/line1 and print messages" |
| **🧩 PluginDev-Skill** | `PluginDev-Skill/` | ⚙️ Developing plugins — custom IDaq/IMq plugins for the Daq tool | "Develop a temperature/humidity sensor plugin — TCP sends 01 03 00 00 00 02, parse the response bytes" |
| **🖥️ WpfUI-Skill** | `WpfUI-Skill/` | 🪟 UI development — WPF desktop applications | "Create a monitoring panel with dark theme, zh/en switching, LED status lights, and a property editor" |



## 🔌 DAQ-Skill — Industrial IoT Data Acquisition

> **In one sentence:** The user describes the requirement → AI generates complete runnable code

### 🎯 Purpose

Help users use the Snet framework directly for data acquisition and forwarding, without developing plugins.

### ✨ Capabilities

- 🏭 **35+ industrial protocols** — Siemens/Modbus/Mitsubishi/Omron/OPC UA/Allen-Bradley/Delta/Keyence/Panasonic/Beckhoff/Inovance/Invt/MegMeet/Kossi/OrientalMotor/YuDian
- 🗄️ **Database acquisition** — SqlServer/MySQL/Oracle/SQLite + TEP non-standard devices
- 📡 **6 message middleware for forwarding** — MQTT/Kafka/RabbitMQ/RocketMQ/NetMQ/Netty
- 🎮 **Built-in simulation library** — test without real PLC hardware
- ⚡ **Automatic data forwarding** — configure `AddressMqParam` and the framework handles read → format → MQ produce

### 💬 Example Scenarios

```
🗣️ "Connect to Siemens S7-1500, read DB1.0, forward via MQTT to port 1883"
🗣️ "Connect to Modbus TCP 192.168.0.2:502, read 40001-40010, forward to Kafka"
🗣️ "Connect to OPC UA, read all tags, forward via MQTT"
🗣️ "Query temperature data from a SqlServer database, forward via MQTT"
```

### 🗣️ Interaction Flow

The AI asks the user in plain language (🏷️ brand & model, 🔌 how it connects, 📍 IP address, 📊 what data to read, 📨 where to send it), then generates code after confirmation.

**📄 File:** `SKILL.md` (55KB, 15 chapters) · [中文 README](README.md) | [English README](README.en.md)



## 📡 MQ-Skill — Messaging Middleware Applications

> **In one sentence:** The user describes a messaging requirement → AI generates complete runnable middleware code

### 🎯 Purpose

Help users use the Snet framework directly for message production and consumption, covering 6 mainstream message middleware.

### ✨ Capabilities

- 📨 **6 message middleware** — MQTT (Client/Broker/WebSocket), Kafka, RabbitMQ, RocketMQ, NetMQ, Netty
- 🔗 **Unified IMq interface** — Produce/Consume/OnDataEventAsync with identical usage; switch middleware without changing business logic
- 🔄 **DAQ integration** — `AddressMqParam` configuration + `config/mq/` registration, automatic forwarding of acquired data
- 🛠️ **Troubleshooting** — quick reference for ports/auth/ResponseType/instance registration issues

### 💬 Example Scenarios

```
🗣️ "Connect to MQTT at 127.0.0.1:1883, subscribe to factory/line1, print messages"
🗣️ "Send JSON to Kafka cluster at 192.168.0.1:9092, topic sensors"
🗣️ "Send/receive messages with a RabbitMQ topic exchange"
🗣️ "Do pub/sub with NetMQ at tcp://127.0.0.1:8866"
```

### 🗣️ Interaction Flow

The AI asks the user in plain language (📨 which middleware, 📍 broker address, 🔑 authentication, 📬 topic, 📤 send or receive), then generates code after confirmation.

**📄 File:** `SKILL.md` (9 chapters) · [中文 README](README.md) | [English README](README.en.md)



## 🧩 PluginDev-Skill — Plugin Development

> **In one sentence:** The user describes a device → AI generates plugin code per the contract → package as ZIP and upload to the Daq tool

### 🎯 Purpose

Help users develop custom protocol plugins deployed to the [Snet.Iot.Daq](https://github.com/shunnet/Daq) tool. **Covers both IDaq (data acquisition) and IMq (messaging middleware) plugin types.**

### ✨ Capabilities

- 📋 **Strict plugin contracts** — IDaq 8 abstract methods / IMq 6 abstract methods + 2 required properties
- 📡 **5 built-in communication classes** — TCP/UDP/WebSocket/Serial/HTTP
- 💾 **Data caching** — process cache `ProcessCacheOperate` + cross-process shared cache `ShareCacheOperate`
- 🔍 **Reflection invocation** — `ReflectionOperate` dynamically loads external DLLs, invokes methods, registers events
- 📐 **Data class conventions** — Basics extends SCData (DAQ) / standalone class (MQ), ProtocolType enum, Attribute annotations
- 🔄 **Core data chain** — `raw data → AddressHandler.ExecuteDispose → automatic type conversion + reflection parsing + MQ forwarding`
- 📦 **Packaging & hot-swap** — ZIP → upload to the Daq tool → runtime load/unload (MQ plugins require `config/mq/` registration)

### 🔑 Core Principles

| 🧠 AI Decides Freely | 📏 Hard Contract |
|-|-|
| What to acquire, how to acquire it | How data is wrapped, what methods return |

### 💬 Example Scenarios

```
🗣️ "Develop a temperature/humidity sensor plugin — TCP sends Modbus frames, parse temperature and humidity"
🗣️ "Develop an HTTP JSON sensor plugin — GET API returns JSON data"
🗣️ "Develop a file monitoring plugin — FileSystemWatcher watches log files"
```

### 🗣️ Interaction Flow

The AI asks the user in plain language (🏷️ what the device is, 🔌 how it connects, 📋 what the protocol rules are, 📊 what the data looks like), then generates code after confirmation.

**📄 File:** `SKILL.md` (41KB, 12 chapters) · [中文 README](README.md) | [English README](README.en.md)



## 🖥️ WpfUI-Skill — WPF Desktop Application Development

> **In one sentence:** The user describes a UI → AI generates a complete WPF desktop application

### 🎯 Purpose

Help users build modern desktop applications on the Snet WPF framework, covering window management, MVVM binding, theme switching, multi-language support, and built-in controls.

### ✨ Capabilities

- 🪟 **WindowBase window base class** — custom title bar, DPI awareness, loading animation, Win32 integration
- 🎨 **Dark/light themes** — Material Design + Wpf.Ui dual engines, one-click switch, JSON persistence
- 🌍 **Chinese/English multi-language** — `LocExtension` / `BLoc` XAML markup extensions, runtime hot-switch
- 🧩 **MVVM architecture** — `BindNotify` expression-based property class + `InjectionWpf` DI container + EventCommand
- 🎛️ **6 built-in controls** — ButtonControl / ComboBoxControl / TextBoxControl / LedGaugeControl / PageBarControl / PropertyControl
- 🔧 **PropertyGrid property editor** — 40+ annotations, categories/visibility/conditional enable/sliders/file pickers/color pickers
- 🖱️ **Drag controls** — 8-direction resize handles + drag to move + animated drag creation
- 💬 **Message dialogs** — OK / OKCancel / Yes / YesNo, Material Design style
- 🔔 **System tray** — NotifyIcon minimize-to-tray with right-click menu

### 💬 Example Scenarios

```
🗣️ "Create a WPF monitoring panel — dark theme, zh/en switching, data table on the left, property editor on the right"
🗣️ "Build a PLC debugging tool with a connect button, LED status lights, and a parameter configuration panel"
🗣️ "Build a draggable device layout editor with movable and resizable controls"
🗣️ "Build a multi-tab settings window that edits configuration with PropertyGrid"
```

### 🗣️ Interaction Flow

The AI asks the user in plain language (🏷️ window requirements, 🎨 theme preference, 🌍 language needs, 🎛️ which controls, 📊 data binding structure), then generates complete XAML + C# code.

**📄 File:** `SKILL.md` (29 chapters) · [中文 README](README.md) | [English README](README.en.md)



## 🔗 Skill Relationships

```
┌─ 🔌 DAQ-Skill ───────────────────────────────────────┐
│  Use ready-made protocol libraries (Snet.Siemens /   │
│  Modbus / OPC ...) → configure connection + address  │
│  → acquire → subscribe → auto-forward                │
└───────────────────────┬───────────────────────────────┘
                        │
                        │ ⬇️ data forwarding target (AddressMqParam)
                        ↓
┌─ 📡 MQ-Skill ────────────────────────────────────────┐
│  Use ready-made middleware (MQTT / Kafka / RabbitMQ /│
│  RocketMQ / NetMQ / Netty) → produce/consume →       │
│  unified IMq interface                               │
└───────────────────────┬───────────────────────────────┘
                        │
                        │ ⬇️ if existing protocols/middleware don't fit...
                        ↓
┌─ 🧩 PluginDev-Skill ─────────────────────────────────┐
│  Develop custom plugins: IDaq (extends DaqAbstract,  │
│  8 methods) + IMq (extends MqAbstract, 6 methods)    │
│  → use built-in communication classes                │
│  → data via ExecuteDispose → auto conversion+forward │
│  → package ZIP → upload to Daq tool → hot-swap load  │
└───────────────────────┬───────────────────────────────┘
                        │
                        │ ⬇️ need a desktop app to manage acquisition...
                        ↓
┌─ 🖥️ WpfUI-Skill ─────────────────────────────────────┐
│  Build WPF desktop monitoring/management apps        │
│  → WindowBase + MVVM architecture                    │
│  → dark/light themes + zh/en multi-language          │
│  → Button/ComboBox/TextBox/LedGauge/PageBar          │
│  → PropertyGrid editor + drag controls               │
│  → message dialogs + system tray                     │
└───────────────────────────────────────────────────────┘
```

**💡 One-sentence summary:** DAQ-Skill leverages ready-made protocols, MQ-Skill leverages ready-made middleware, PluginDev-Skill develops new protocol/middleware plugins, and WpfUI-Skill builds management UIs — covering the full chain from data acquisition and messaging to human-machine interaction.



## 🌍 Related Links

| 🔗 | Link | Description |
|---|---|---|
| 🌐 | [Official Site](https://snet.cn) | Official website |
| 📚 | [NuGet](https://www.nuget.org/profiles/shun) | NuGet package repository |
| 💻 | [GitHub](https://github.com/shunnet) | Source code repository |
| 🔌 | [Daq Tool](https://github.com/shunnet/Daq) | Plugin-based acquisition tool |
| 📝 | [Blog](https://blog.snet.cn) | Technical blog |



<p align="center">
  <b>🙏 Thanks for using the Snet Industrial IoT Skills Collection</b>
</p>
