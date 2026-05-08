<h1 align="center">SKILL</h1>

<p align="center">
  <img width="120" height="120" src="https://api.shunnet.top/pic/nuget.png" alt="Snet Logo"/>
</p>

<p align="center">
  <b>Snet 工业物联网技能集</b>
</p>

<p align="center">

  <img src="https://img.shields.io/badge/license-MIT-green"/>

</p>

<p align="center">
  本目录包含 Snet 工业物联网生态的两个核心技能，覆盖从"使用现有协议"到"开发自定义插件"的完整流程。
</p>

<p align="center">
  <a href="https://shunnet.top"><b>🌐 官方网站</b></a>
</p>

**版本:** 1.0.0.0  
**作者:** Shun  
**许可证:** MIT  
**框架:** .NET 8.0+  



## 技能一览

| 技能 | 目录 | 定位 | 用户说一句话 → |
||||-|
| **DAQ-Skill** | `DAQ-Skill/` | 使用库 — 数据采集与转发 | "连接西门子S7-1500，IP 192.168.0.1，读取DB1.0，MQTT转发到broker" |
| **PluginDev-Skill** | `PluginDev-Skill/` | 开发插件 — 自定义协议对接 Daq 工具 | "开发一个温湿度传感器插件，TCP发送 01 03 00 00 00 02，解析响应字节" |



## DAQ-Skill — 工业物联网数据采集

**用途：** 帮助用户直接使用 Snet 框架完成数据采集与转发，无需开发插件。

**能力：**
- 30+ 种工业协议（西门子/Modbus/三菱/欧姆龙/OPC UA/罗克韦尔等）
- 数据库采集（SqlServer/MySQL/Oracle/SQLite）+ TEP 非标设备
- 5 种消息中间件转发（MQTT/Kafka/RabbitMQ/NetMQ/Netty）
- 内置模拟库（无需真实 PLC 即可测试）
- 自动数据转发：在 `AddressDetails` 中配置 `AddressMqParam`，框架自动完成读取→格式化→MQ生产

**场景示例：**
- "连接西门子S7-1500，读DB1.0，MQTT转发到1883端口"
- "连接Modbus TCP 192.168.0.2:502，读取40001-40010，转发到Kafka"
- "连接OPC UA，读取所有Tag，通过MQTT转发"
- "从SqlServer数据库查询温度数据，MQTT转发"

**交互流程：** AI 先用大白话问用户（牌子型号、怎么连的、IP多少、读什么数据、发到哪里），确认后再生成代码。

**文件：** `README.md`（5KB）+ `SKILL.md`（38KB，13章）



## PluginDev-Skill — Daq 工具插件开发

**用途：** 帮助用户开发自定义协议插件，部署到 [Snet.Iot.Daq](https://github.com/shunnet/Daq) 工具中使用。

**能力：**
- 严格的插件开发契约（8 个抽象方法 + 3 个必须属性）
- 5 种内置通信类快速集成（TCP/UDP/WebSocket/Serial/HTTP）
- 数据缓存：进程缓存 `ProcessCacheOperate` + 跨进程共享缓存 `ShareCacheOperate`
- 反射调用：`ReflectionOperate` 动态加载外部 DLL、调用方法、注册事件
- 数据类规范（Basics 继承 SCData、ProtocolType 枚举、Attribute 标注）
- 核心数据处理链：`采集原始数据 → AddressHandler.ExecuteDispose → 自动类型转换+反射解析+MQ转发`
- 打包为 ZIP → 上传 Daq 工具 → 热插拔加载

**核心原则：** 采集什么、怎么采集 —— AI 自行决定。数据怎么封裝、方法返回什么 —— 硬性契约，必须遵守。

**场景示例：**
- "开发一个温湿度传感器插件，TCP 发送 Modbus 帧，解析温度湿度"
- "开发一个 HTTP JSON 传感器插件，GET API 返回 JSON 数据"
- "开发一个文件监控插件，FileSystemWatcher 监控日志文件"

**交互流程：** AI 先用大白话问用户（设备是什么、怎么连的、协议规矩是什么、数据长什么样），确认后再生成代码。

**文件：** `README.md`（3KB）+ `SKILL.md`（36KB，11章）



## 技能关系

```
┌─ DAQ-Skill ───────────────────────────────────────┐
│  使用现成协议库（Snet.Siemens / Modbus / OPC ...） │
│  → 配置连接参数 + 地址 → 采集 → 订阅 → 自动转发   │
└───────────────────┬───────────────────────────────┘
                    │
                    │ 如果现成协议不满足需求...
                    ↓
┌─ PluginDev-Skill ─────────────────────────────────┐
│  开发自定义插件（继承 DaqAbstract，实现 8 个方法）  │
│  → 用内置通信类(TcpClientOperate等) → Read/Write  │
│  → 数据经 ExecuteDispose 处理 → 自动类型转换+转发  │
│  → 打包 ZIP → 上传 Daq 工具 → 热插拔加载          │
└───────────────────────────────────────────────────┘
```

**一句话总结：** DAQ-Skill 帮用户用好现成的协议，PluginDev-Skill 帮用户开发新协议插件。



## 依赖关系

两个技能都基于 [Snet NuGet 生态](https://www.nuget.org/profiles/shun)：

```
Snet.Core（抽象基类+通信）
Snet.Model（接口+数据模型）
Snet.Log（6级日志）
Snet.Utility（工具集）
Snet.Driver（硬件驱动）
```



## 相关链接

- 🚀 [官网](https://shunnet.top)
- 📚 [NuGet](https://www.nuget.org/profiles/shun)
- 💻 [GitHub](https://github.com/shunnet)
- 🔌 [Daq 工具](https://github.com/shunnet/Daq)
