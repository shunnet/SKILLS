# DAQ-Skill — Industrial IoT Data Acquisition Skill

**Version:** 1.0.1.3  
**Author:** Shun  
**License:** MIT  
**Framework:** .NET 10.0

🌐 [中文 README](README.md) | [English README](README.en.md)

## One-Sentence Acquisition + Forwarding

This skill generates complete code from natural-language requirements:

> **"Connect to Siemens S7-1500 at 192.168.0.1, read DB1.0 (Int32), forward via MQTT to 127.0.0.1:1883, topic sensors/siemens"**

→ Automatically generates complete runnable code including: project creation + NuGet installation + DAQ configuration + MQ configuration + automatic forwarding + logging

**Core:** Configure `AddressMqParam` in `AddressDetails` and the framework handles forwarding automatically (read → format → MQ produce) — no manual forwarding code.

## Introduction

A full-stack industrial IoT data acquisition library based on the [Snet](https://www.nuget.org/profiles/Shun) framework. Supports 30+ industrial protocols (PLC/industrial control/power/robotics), messaging middleware (Kafka/MQTT/RabbitMQ/RocketMQ/NetMQ/Netty) for data forwarding, and a built-in WebAPI for remote control.

## Interaction Flow

> The AI asks the user in plain language and generates code after confirmation. When the user says "help me connect to a PLC and read some data", the AI asks step by step:

| # | What the AI asks | How the user answers | What the AI maps it to |
|---|-----------|---------------|--------------|
| 1 | What brand and model is the device? | "Siemens S7-1500" | → ProtocolType |
| 2 | Ethernet or serial connection? | "Ethernet" | → TCP / Serial |
| 3 | IP and port / COM port and baud rate? | "192.168.0.1" | → connection parameters |
| 4 | What data to read? Addresses and types? | "DB1.0 is an integer" | → AddressDetails |
| 5 | Where to send the data? | "MQTT 127.0.0.1:1883" | → MQ configuration |
| 6 | How often to read? | "Real-time, send on change" | → HandleInterval |

## Core Components

| Component | Description |
|------|------|
| `Snet.Core` | Core engine (abstract base classes · TCP/UDP/HTTP/WS/Serial · subscription · cache · reflection · script · WebAPI) |
| `Snet.Model` | Data model layer (attributes · data structures · enums · `IDaq`/`IMq` interfaces) |
| `Snet.Log` | Logging system (6 levels · file/database/console multi-channel) |
| `Snet.Utility` | Utilities (bytes · enums · files · strings · JSON · XML · Protobuf · FTP) |
| `Snet.Driver` | Low-level hardware communication drivers |

## Protocol Overview (36 protocols, 150+ ProtocolTypes)

| Package | Protocol | Operate class | ProtocolType count |
|------|------|-----------|-------------------|
| `Snet.Siemens` | Siemens S7/PPI/S7Plus/FetchWrite | `SiemensOperate` | 10 |
| `Snet.Modbus` | Modbus TCP/UDP/RTU/ASCII/RTUoTCP/ASCIIoTCP | `ModbusOperate` | 6 |
| `Snet.Mitsubishi` | Mitsubishi MC/FX/A1E/A3C/CIP/Links | `MitsubishiOperate` | 14 |
| `Snet.Omron` | Omron Fins/CIP/HostLink/CMode | `OmronOperate` | 8 |
| `Snet.OrientalMotor` | OrientalMotor EIP stepper drive | `OrientalMotorOperate` | 1 |
| `Snet.Inovance` | Inovance TCP/Serial/CIP/Easy/ComputerLink | `InovanceOperate` | 6 |
| `Snet.Opc` | OPC UA Client/Server + DA Client/HTTP | `OpcUaClientOperate` / `OpcUaServiceOperate` / `OpcDaClientOperate` / `OpcDaHttpOperate` | 4 classes |
| `Snet.AllenBradley` | Allen-Bradley CIP/PCCC/SLC/DF1 | `AllenBradleyOperate` | 6 |
| `Snet.Delta` | Delta TCP/Serial/ASCII | `DeltaOperate` | 5 |
| `Snet.Keyence` | Keyence MC/Nano/KvOld | `KeyenceOperate` | 5 |
| `Snet.Kossi` | Kossi PLC (Omron CIP) | `KossiOperate` | 1 |
| `Snet.Panasonic` | Panasonic MC/Mewtocol | `PanasonicOperate` | 3 |
| `Snet.Invt` | Invt (Modbus) | `InvtOperate` | 2 |
| `Snet.MegMeet` | MegMeet TCP/Serial | `MegMeetOperate` | 3 |
| `Snet.Beckhoff` | Beckhoff ADS | `BeckhoffOperate` | 1 |
| `Snet.GE` | GE SRTP | `GEOperate` | 1 |
| `Snet.Yaskawa` | Yaskawa Memobus TCP/UDP | `YaskawaOperate` | 2 |
| `Snet.Cimon` | Cimon PLC | `CimonOperate` | 1 |
| `Snet.Fanuc` | FANUC CNC/robot | `FanucOperate` | 1 |
| `Snet.Fatek` | Fatek PLC | `FatekOperate` | 2 |
| `Snet.Fuji` | Fuji SPH/SPB | `FujiOperate` | 4 |
| `Snet.LSis` | LS Industrial Systems PLC | `LSisOperate` | 4 |
| `Snet.RKC` | RKC temperature controller | `RKCOperate` | 2 |
| `Snet.Toyota` | Toyota robot | `ToyotaOperate` | 1 |
| `Snet.Turck` | Turck IO-Link | `TurckOperate` | 1 |
| `Snet.Vigor` | Vigor PLC | `VigorOperate` | 2 |
| `Snet.WeCon` | WeCon PLC | `WeConOperate` | 2 |
| `Snet.XinJE` | XinJE | `XinJEOperate` | 4 |
| `Snet.Yamatake` | Yamatake (AZBIL) | `YamatakeOperate` | 2 |
| `Snet.Yokogawa` | Yokogawa PLC | `YokogawaOperate` | 1 |
| `Snet.YuDian` | YuDian AIBus temperature controller | `YuDianOperate` | 1 |
| `Snet.PQDIF` | Power communication protocols (DLT645/DLT698/CJT188/DTSU6606) | `PQDIFOperate` | 10 |
| `Snet.DB` | Database acquisition (SqlServer/MySQL/Oracle/SQLite) | `DBOperate` | 4 DB types |
| `Snet.TEP` | TCP extension plugin (non-standard devices) | `TepMasterOperate` / `TepSlaveOperate` | custom |
| `Snet.Freedom` | Free protocol (custom frames) | `FreedomOperate` | 3 |
| `Snet.Sim` | Simulation library (hardware-free testing) | `SimOperate` | 5 virtual address types |
| *(Snet.Driver)* | Knx / OpenProtocol / Sick / Geniitek / IDCard / Toledo / IEC104 / ShineIn Light / DAM3601 | Driver layer (no standalone NuGet package) | 9 |

## Message Middleware

| Package | Capability | ISns format |
|------|------|-----------|
| `Snet.Mqtt` | Client (connect to broker) + Service (embedded broker) + WSService (WebSocket) | `Snet.Mqtt.client.MqttClientOperate.{SN}` |
| `Snet.Kafka` | AdminClient/Producer/Consumer | `Snet.Kafka.KafkaOperate.{SN}` |
| `Snet.RabbitMQ` | Publish/Subscribe | `Snet.RabbitMQ.RabbitMQOperate.{SN}` |
| `Snet.RocketMQ` | Publish/Subscribe (Apache RocketMQ 5.x, gRPC to Proxy) | `Snet.RocketMQ.RocketMQOperate.{SN}` |
| `Snet.NetMQ` | Publish/Subscribe | `Snet.NetMQ.NetMQOperate.{SN}` |
| `Snet.Netty` | Client/Service | `Snet.Netty.client.NettyClientOperate.{SN}` |

## Data Forwarding Mechanism

```
Read/subscribe to get data
  → configure AddressDetails.AddressMqParam
    → framework automatically calls mqOperate.Produce(Topic, Content, ISns)
      → send to the designated MQ instance
```

**The user only needs three steps:**
1. **Register the MQ instance via a `config/mq/` config file** (⚠️ critical prerequisite, see below)
2. Configure `ISns` (pointing to the MQ's SN), `Topic`, and `ContentFormat` in `AddressDetails.AddressMqParam`
3. Start the subscription → data is forwarded automatically

> **⚠️ Automatic forwarding prerequisite (important):** the forwarding mechanism only recognizes MQ instances loaded from `{full namespace}.{class name}.{SN}.Mq.Config.json` files under `config/mq/` (the `MqOperate.InstanceIoc` registry). Instances created with `new` in user code are not registered, and forwarding fails with "instance not found". The config file content = JSON serialization of MQ Basics (see SKILL.md §1.1 for JSON examples). **Alternative:** without a config file, manually call `mqClient.ProduceAsync(topic, content)` inside the subscription data event `OnDataEventAsync`.

## Key Notes

| Item | Description |
|------|------|
| Status check | `result.Status` (not `IsSuccess`!) |
| Write signature | `(object value, EncodingType? encodingType)` tuple |
| Default port | Most libraries default to 6688 — always set the standard port explicitly |
| MQTT property | `IpAddress` (not `Ip`!) |
| ISns format | `full namespace.OperateClassName.Basics.SN` |
| Data quality | `QualityType`: None(-1) / Exception(0) / **Normal(1)** / DataTypeError(2) / ParseUnknown(3) / ParseError(4) |
| GetSource | `result.GetSource<T>()` to get generic result data (not `GetRData`!) |

## Quick Start

```bash
dotnet new console -n DaqDemo && cd DaqDemo

# ✅ Only the driver package + MQ package are needed; other libraries (Core/Model/Log/Utility) come in as transitive dependencies
# ⚠️ Always pin the version! Check nuget.org for the latest:
#   https://www.nuget.org/packages/Snet.Siemens
#   https://www.nuget.org/packages/Snet.Mqtt
dotnet add package Snet.Siemens -v 26.236.1
dotnet add package Snet.Mqtt -v 26.236.1

# Power protocol example
# dotnet add package Snet.PQDIF -v <latest version>

# Write your code → see SKILL.md scenario templates
dotnet run
```

> **📌 On versions:** all `dotnet add package` commands **must** pin `-v` with a version.
> Get the latest version at `https://www.nuget.org/packages/<package-name>`, for example:
> - `https://www.nuget.org/packages/Snet.Siemens`
> - `https://www.nuget.org/packages/Snet.Modbus`
> - `https://www.nuget.org/packages/Snet.Mqtt`
>
> **📌 On transitive dependencies:** only the top-level driver package (e.g. `Snet.Siemens`, `Snet.Modbus`) is needed;
> `Snet.Core`, `Snet.Model`, `Snet.Log`, `Snet.Utility`, `Snet.Driver` come in automatically — **no manual adds**.

## Related Links

- 🚀 [Official Site](https://shunnet.top) · 📚 [NuGet](https://www.nuget.org/profiles/shun) · 💻 [GitHub](https://github.com/shunnet)
- 📝 [Blog](https://blog.shunnet.top/details/d46c5e6192d9476e910a0e18145faeba) · 🔧 [Git](https://git.shunnet.top)
