# MQ-Skill — Snet 消息中间件应用技能

**版本:** 1.0.0.8  
**作者:** Shun  
**许可证:** MIT  
**框架:** .NET 10.0

## 一句话消息收发

本技能支持用自然语言描述需求，自动生成完整代码：

> **"通过MQTT连接 127.0.0.1:1883，topic为factory/line1，接收并打印消息"**

→ 自动生成包含：项目创建 + NuGet安装 + 中间件配置 + 生产/消费 + 日志的完整可运行代码

**核心：** 5 种中间件（MQTT/Kafka/RabbitMQ/NetMQ/Netty）全部实现统一 `IMq` 接口，`Produce`/`Consume`/`OnDataEventAsync` 用法一致，切换中间件无需改业务逻辑。

## 简介

基于 [Snet](https://www.nuget.org/profiles/Shun) 框架的消息中间件应用技能，覆盖：

| 中间件 | 操作类 | 特点 |
|--------|--------|------|
| **MQTT** | `MqttClientOperate` / `MqttServiceOperate` / `MqttWebSocketServiceOperate` | Client+内置 Broker+WebSocket，QoS 0/1/2 |
| **Kafka** | `KafkaOperate` | 高吞吐，SASL 认证，`BootstrapServers` 连接 |
| **RabbitMQ** | `RabbitMQOperate` | AMQP 协议，4 种 Exchange（direct/topic/fanout/headers） |
| **NetMQ** | `NetMQOperate` | ZeroMQ Pub/Sub，无 Broker，<1ms 延迟 |
| **Netty** | `NettyClientOperate` / `NettyServiceOperate` | DotNetty 高性能 TCP，SSL 支持 |

## 交互流程

> AI 先用大白话问用户，确认后再生成代码：

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 用什么消息中间件？ | "MQTT" "Kafka" | → 中间件包 + Operate 类 |
| 2 | Broker 地址是多少？ | "127.0.0.1:1883" | → 连接参数 |
| 3 | 需要账号密码吗？ | "不用" | → UserName/Password |
| 4 | 发消息还是收消息？ | "发" "收" "都做" | → Produce / Consume |
| 5 | 主题叫什么？ | "factory/line1" | → Topic |
| 6 | 消息是文本还是二进制？ | "字符串" | → Produce(string) + ResponseType |

## 统一 API（IMq 接口）

```csharp
await mq.OnAsync();                              // 连接
await mq.ProduceAsync(topic, content);           // 生产
mq.OnDataEventAsync += async (s, e) => { ... };  // 消费数据事件
await mq.ConsumeAsync(topic);                    // 订阅
await mq.OffAsync();                             // 断开
```

## 与 DAQ 联动

`AddressMqParam` 配置 + `config/mq/*.Mq.Config.json` 配置文件注册实例 → DAQ 采集数据自动转发到中间件（详见 SKILL.md §6）。

## 相关技能

- [DAQ-Skill](../DAQ-Skill) — 数据采集 + 自动转发
- [PluginDev-Skill](../PluginDev-Skill) — 开发自定义 IMq 插件（第 8 章）

**📄 文件：** `SKILL.md`（8 章：统一 IMq 接口 / 5 种中间件详解 / DAQ 联动转发 / 故障排查）
