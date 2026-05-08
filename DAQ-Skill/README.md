# DAQ-Skill — 工业物联网数据采集技能

**版本:** 1.0.0.0  
**作者:** Shun  
**许可证:** MIT  
**框架:** .NET 8.0 / 9.0 / 10.0

## 一句话采集+转发

本技能支持用自然语言描述需求，自动生成完整代码：

> **"连接西门子S7-1500，IP 192.168.0.1，读取DB1.0（Int32），通过MQTT转发到127.0.0.1:1883，topic为sensors/siemens"**

→ 自动生成包含：项目创建 + NuGet安装 + DAQ配置 + MQ配置 + 自动转发 + 日志的完整可运行代码

**核心：** 在 `AddressDetails` 中配置 `AddressMqParam`，框架自动完成数据转发（读→格式→MQ生产），无需手动编码转发逻辑。

## 简介

基于 [Snet](https://www.nuget.org/profiles/Shun) 框架的工业物联网全栈式数据采集通信库。支持 20+ 种工业协议（PLC/工控/电力/机器人），消息中间件（Kafka/MQTT/RabbitMQ/NetMQ/Netty）数据转发，内置 WebAPI 远程控制。

## 交互流程

> AI 先用大白话问用户，确认后再生成代码。用户说一句“帮我连PLC读点数据”，AI 一步步问清楚：

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 设备是什么牌子什么型号？ | “西门子S7-1500” | → ProtocolType |
| 2 | 网线连的还是串口线连的？ | “网线” | → TCP / Serial |
| 3 | IP和端口/COM口和波特率？ | “192.168.0.1” | → 连接参数 |
| 4 | 要读哪些数据？地址和类型？ | “DB1.0是整数” | → AddressDetails |
| 5 | 数据发到哪里？ | “MQTT 127.0.0.1:1883” | → MQ 配置 |
| 6 | 多久读一次？ | “实时，变了就发” | → HandleInterval |

## 核心组件

| 组件 | 说明 |
|------|------|
| `Snet.Core` | 核心引擎（抽象基类 · TCP/UDP/HTTP/WS/串口 · 订阅 · 缓存 · 反射 · 脚本 · WebAPI） |
| `Snet.Model` | 数据模型层（特性 · 数据结构 · 枚举 · `IDaq`/`IMq` 接口） |
| `Snet.Log` | 日志系统（6级 · 文件/数据库/控制台多通道） |
| `Snet.Utility` | 工具集（字节 · 枚举 · 文件 · 字符串 · JSON · XML · Protobuf · FTP） |
| `Snet.Driver` | 底层硬件通信驱动 |

## 采集协议一览

| 包名 | 协议 | Operate 类 | ProtocolType 数量 |
|------|------|-----------|-------------------|
| `Snet.Siemens` | 西门子 S7/PPI/S7Plus | `SiemensOperate` | 10 种 |
| `Snet.Modbus` | Modbus TCP/UDP/RTU/ASCII | `ModbusOperate` | 6 种 |
| `Snet.Mitsubishi` | 三菱 MC/FX/A1E/A3C/CIP | `MitsubishiOperate` | 14 种 |
| `Snet.Omron` | 欧姆龙 Fins/CIP/HostLink | `OmronOperate` | 8 种 |
| `Snet.Inovance` | 汇川 TCP/Serial/CIP/Easy | `InovanceOperate` | 6 种 |
| `Snet.Opc` | OPC UA/DA/DAHttp | `OpcUaClientOperate` | 3 种 |
| `Snet.Delta` | 台达 | `DeltaOperate` | 5 种 |
| `Snet.Keyence` | 基恩士 MC/Nano/KvOld | `KeyenceOperate` | 5 种 |
| `Snet.AllenBradley` | 罗克韦尔 CIP/PCCC/DF1 | `AllenBradleyOperate` | 6 种 |
| `Snet.Panasonic` | 松下 MC/Mewtocol | `PanasonicOperate` | 3 种 |
| `Snet.Invt` | 英威腾 | `InvtOperate` | 2 种 |
| `Snet.MegMeet` | 麦格米特 | `MegMeetOperate` | 3 种 |
| `Snet.DB` | 数据库采集（SqlServer/MySQL/Oracle/SQLite） | `DBOperate` | 4 种 DB + Daq/Default 模式 |
| `Snet.TEP` | TCP 扩展插件（非标设备采集） | `TepMasterOperate` / `TepSlaveOperate` | 自定义 TCP 协议 |
| `Snet.Freedom` | 自由协议（自定义报文） | `FreedomOperate` | TCP/UDP/串口 |
| `Snet.Sim` | 模拟库（无硬件测试） | `SimOperate` | 5 种虚拟地址 |
| 更多... | Beckhoff, GE, Yaskawa, Yokogawa, Fanuc, Fatek, Fuji, Vigor, WeCon, XinJE, LSis, Cimon, RKC, Turck, Yamatake | | |

## 消息中间件

| 包名 | 能力 | ISns 格式 |
|------|------|-----------|
| `Snet.Mqtt` | Client/Service/WSService | `Snet.Mqtt.client.MqttClientOperate.{SN}` |
| `Snet.Kafka` | AdminClient/Producer/Consumer | `Snet.Kafka.KafkaOperate.{SN}` |
| `Snet.RabbitMQ` | Publish/Subscribe | `Snet.RabbitMQ.RabbitMqOperate.{SN}` |
| `Snet.NetMQ` | Publish/Subscribe | `Snet.NetMQ.NetMqOperate.{SN}` |
| `Snet.Netty` | Client/Service | `Snet.Netty.NettyOperate.{SN}` |

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
| 端口默认 | 所有库默认 6688，务必显式设置标准端口 |
| MQTT 属性 | `IpAddress`（不是 `Ip`） |
| ISns 格式 | `完整命名空间.Operate类名.Basics.SN` |
| 数据质量 | `QualityType`: None(-1) / Exception(0) / **Normal(1)** / DataTypeError(2) / ParseUnknown(3) / ParseError(4) |

## 快速开始

```bash
dotnet new console -n DaqDemo && cd DaqDemo
dotnet add package Snet.Siemens Snet.Mqtt Snet.Core Snet.Model Snet.Log Snet.Utility

# 编写代码 → 参考 SKILL.md 场景模板
dotnet run
```

## 相关链接

- 🚀 [官网](https://shunnet.top) · 📚 [NuGet](https://www.nuget.org/profiles/shun) · 💻 [GitHub](https://github.com/shunnet)
- 📝 [博客](https://blog.shunnet.top/details/d46c5e6192d9476e910a0e18145faeba) · 🔧 [Git](https://git.shunnet.top)
