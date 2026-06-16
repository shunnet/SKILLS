# DAQ-Skill — 工业物联网数据采集技能

**版本:** 1.0.0.5  
**作者:** Shun  
**许可证:** MIT  
**框架:** .NET 10.0

## 一句话采集+转发

本技能支持用自然语言描述需求，自动生成完整代码：

> **"连接西门子S7-1500，IP 192.168.0.1，读取DB1.0（Int32），通过MQTT转发到127.0.0.1:1883，topic为sensors/siemens"**

→ 自动生成包含：项目创建 + NuGet安装 + DAQ配置 + MQ配置 + 自动转发 + 日志的完整可运行代码

**核心：** 在 `AddressDetails` 中配置 `AddressMqParam`，框架自动完成数据转发（读→格式→MQ生产），无需手动编码转发逻辑。

## 简介

基于 [Snet](https://www.nuget.org/profiles/Shun) 框架的工业物联网全栈式数据采集通信库。支持 30+ 种工业协议（PLC/工控/电力/机器人），消息中间件（Kafka/MQTT/RabbitMQ/NetMQ/Netty）数据转发，内置 WebAPI 远程控制。

## 交互流程

> AI 先用大白话问用户，确认后再生成代码。用户说一句"帮我连PLC读点数据"，AI 一步步问清楚：

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 设备是什么牌子什么型号？ | "西门子S7-1500" | → ProtocolType |
| 2 | 网线连的还是串口线连的？ | "网线" | → TCP / Serial |
| 3 | IP和端口/COM口和波特率？ | "192.168.0.1" | → 连接参数 |
| 4 | 要读哪些数据？地址和类型？ | "DB1.0是整数" | → AddressDetails |
| 5 | 数据发到哪里？ | "MQTT 127.0.0.1:1883" | → MQ 配置 |
| 6 | 多久读一次？ | "实时，变了就发" | → HandleInterval |

## 核心组件

| 组件 | 说明 |
|------|------|
| `Snet.Core` | 核心引擎（抽象基类 · TCP/UDP/HTTP/WS/串口 · 订阅 · 缓存 · 反射 · 脚本 · WebAPI） |
| `Snet.Model` | 数据模型层（特性 · 数据结构 · 枚举 · `IDaq`/`IMq` 接口） |
| `Snet.Log` | 日志系统（6级 · 文件/数据库/控制台多通道） |
| `Snet.Utility` | 工具集（字节 · 枚举 · 文件 · 字符串 · JSON · XML · Protobuf · FTP） |
| `Snet.Driver` | 底层硬件通信驱动 |

## 采集协议一览（30+ 种协议，150+ ProtocolType）

| 包名 | 协议 | Operate 类 | ProtocolType 数量 |
|------|------|-----------|-------------------|
| `Snet.Siemens` | 西门子 S7/PPI/S7Plus/FetchWrite | `SiemensOperate` | 10 种 |
| `Snet.Modbus` | Modbus TCP/UDP/RTU/ASCII/RTUoTCP/ASCIIoTCP | `ModbusOperate` | 6 种 |
| `Snet.Mitsubishi` | 三菱 MC/FX/A1E/A3C/CIP/Links | `MitsubishiOperate` | 14 种 |
| `Snet.Omron` | 欧姆龙 Fins/CIP/HostLink/CMode | `OmronOperate` | 8 种 |
| `Snet.Inovance` | 汇川 TCP/Serial/CIP/Easy/ComputerLink | `InovanceOperate` | 6 种 |
| `Snet.Opc` | OPC UA Client/Server + DA Client/HTTP | `OpcUaClientOperate` / `OpcUaServiceOperate` / `OpcDaClientOperate` / `OpcDaHttpOperate` | 4 个类 |
| `Snet.AllenBradley` | 罗克韦尔 CIP/PCCC/SLC/DF1 | `AllenBradleyOperate` | 6 种 |
| `Snet.Delta` | 台达 TCP/Serial/ASCII | `DeltaOperate` | 5 种 |
| `Snet.Keyence` | 基恩士 MC/Nano/KvOld | `KeyenceOperate` | 5 种 |
| `Snet.Panasonic` | 松下 MC/Mewtocol | `PanasonicOperate` | 3 种 |
| `Snet.Invt` | 英威腾（Modbus） | `InvtOperate` | 2 种 |
| `Snet.MegMeet` | 麦格米特 TCP/Serial | `MegMeetOperate` | 3 种 |
| `Snet.Beckhoff` | 倍福 ADS | `BeckhoffOperate` | 1 种 |
| `Snet.GE` | 通用电气 SRTP | `GEOperate` | 1 种 |
| `Snet.Yaskawa` | 安川 Memobus TCP/UDP | `YaskawaOperate` | 2 种 |
| `Snet.Cimon` | 西蒙 PLC | `CimonOperate` | 1 种 |
| `Snet.Fanuc` | 发那科 CNC/机器人 | `FanucOperate` | 1 种 |
| `Snet.Fatek` | 永宏 PLC | `FatekOperate` | 2 种 |
| `Snet.Fuji` | 富士 SPH/SPB | `FujiOperate` | 4 种 |
| `Snet.LSis` | LS产电 PLC | `LSisOperate` | 4 种 |
| `Snet.RKC` | 理化温控器 | `RKCOperate` | 2 种 |
| `Snet.Toyota` | 丰田机器人 | `ToyotaOperate` | 1 种 |
| `Snet.Turck` | 图尔克 IO-Link | `TurckOperate` | 1 种 |
| `Snet.Vigor` | 丰炜 PLC | `VigorOperate` | 2 种 |
| `Snet.WeCon` | 维控 PLC | `WeConOperate` | 2 种 |
| `Snet.XinJE` | 信捷（另一系列） | `XinJEOperate` | 4 种 |
| `Snet.Yamatake` | 山武（AZBIL） | `YamatakeOperate` | 2 种 |
| `Snet.Yokogawa` | 横河 PLC | `YokogawaOperate` | 1 种 |
| `Snet.PQDIF` | 电力通讯规约（DLT645/DLT698/CJT188/DTSU6606） | `PQDIFOperate` | 10 种 |
| `Snet.DB` | 数据库采集（SqlServer/MySQL/Oracle/SQLite） | `DBOperate` | 4 种 DB |
| `Snet.TEP` | TCP 扩展插件（非标设备采集） | `TepMasterOperate` / `TepSlaveOperate` | 自定义 |
| `Snet.Freedom` | 自由协议（自定义报文） | `FreedomOperate` | 3 种 |
| `Snet.Sim` | 模拟库（无硬件测试） | `SimOperate` | 5 种虚拟地址 |
| *(Snet.Driver)* | Knx / OpenProtocol / Sick / Geniitek / IDCard / OrientalMotor / Toledo / IEC104 / Light / AIBus / DAM3601 | 驱动层（无独立 NuGet 包） | 11 种 |

## 消息中间件

| 包名 | 能力 | ISns 格式 |
|------|------|-----------|
| `Snet.Mqtt` | Client（连接 Broker）+ Service（内置 Broker）+ WSService（WebSocket） | `Snet.Mqtt.client.MqttClientOperate.{SN}` |
| `Snet.Kafka` | AdminClient/Producer/Consumer | `Snet.Kafka.KafkaOperate.{SN}` |
| `Snet.RabbitMQ` | Publish/Subscribe | `Snet.RabbitMQ.RabbitMQOperate.{SN}` |
| `Snet.NetMQ` | Publish/Subscribe | `Snet.NetMQ.NetMQOperate.{SN}` |
| `Snet.Netty` | Client/Service | `Snet.Netty.client.NettyClientOperate.{SN}` |

## 数据转发机制

```
读取/订阅获取数据
  → AddressDetails.AddressMqParam 配置
    → 框架自动调用 mqOperate.Produce(Topic, Content, ISns)
      → 发送到指定 MQ 实例
```

**用户只需三步：**
1. 创建 MQ 实例并设置 `Basics.SN`
2. 在 `AddressDetails.AddressMqParam` 中配置 `ISns`（指向 MQ 的 SN）、`Topic`、`ContentFormat`
3. 启动订阅 → 数据自动转发

## 关键注意事项

| 项目 | 说明 |
|------|------|
| 状态检查 | `result.Status`（不是 `IsSuccess`！） |
| Write 签名 | `(object value, EncodingType? encodingType)` 元组 |
| 端口默认 | 多数库默认 6688，务必显式设置标准端口 |
| MQTT 属性 | `IpAddress`（不是 `Ip`！） |
| ISns 格式 | `完整命名空间.Operate类名.Basics.SN` |
| 数据质量 | `QualityType`: None(-1) / Exception(0) / **Normal(1)** / DataTypeError(2) / ParseUnknown(3) / ParseError(4) |
| GetSource | `result.GetSource<T>()` 获取泛型结果数据（不是 `GetRData`！） |

## 快速开始

```bash
dotnet new console -n DaqDemo && cd DaqDemo

# ✅ 只需引用驱动包 + MQ 包，其他库（Core/Model/Log/Utility）会作为传递依赖自动引入
# ⚠️ 必须指定版本号！到 nuget.org 查找最新版本：
#   https://www.nuget.org/packages/Snet.Siemens
#   https://www.nuget.org/packages/Snet.Mqtt
dotnet add package Snet.Siemens -v 1.0.0.1
dotnet add package Snet.Mqtt -v 1.0.0.1

# 电力协议示例
# dotnet add package Snet.PQDIF -v <最新版本>

# 编写代码 → 参考 SKILL.md 场景模板
dotnet run
```

> **📌 关于版本号：** 所有 `dotnet add package` 命令**必须**指定 `-v` 版本号。
> 获取最新版本：打开 `https://www.nuget.org/packages/<包名>` 查看，例如：
> - `https://www.nuget.org/packages/Snet.Siemens`
> - `https://www.nuget.org/packages/Snet.Modbus`
> - `https://www.nuget.org/packages/Snet.Mqtt`
> 
> **📌 关于传递依赖：** 只需引用顶层驱动包（如 `Snet.Siemens`、`Snet.Modbus`），
> `Snet.Core`、`Snet.Model`、`Snet.Log`、`Snet.Utility`、`Snet.Driver` 等会自动引入，**无需手动添加**。

## 相关链接

- 🚀 [官网](https://shunnet.top) · 📚 [NuGet](https://www.nuget.org/profiles/shun) · 💻 [GitHub](https://github.com/shunnet)
- 📝 [博客](https://blog.shunnet.top/details/d46c5e6192d9476e910a0e18145faeba) · 🔧 [Git](https://git.shunnet.top)
