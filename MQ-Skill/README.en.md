# MQ-Skill — Snet Messaging Middleware Skill

**Version:** 1.0.1.0  
**Author:** Shun  
**License:** MIT  
**Framework:** .NET 10.0

🌐 [中文 README](README.md) | [English README](README.en.md)

## One-Sentence Messaging

This skill generates complete code from natural-language requirements:

> **"Connect to MQTT at 127.0.0.1:1883, topic factory/line1, receive and print messages"**

→ Automatically generates complete runnable code including: project creation + NuGet installation + middleware configuration + produce/consume + logging

**Core:** all 6 middleware (MQTT/Kafka/RabbitMQ/RocketMQ/NetMQ/Netty) implement the unified `IMq` interface — `Produce`/`Consume`/`OnDataEventAsync` usage is identical, and switching middleware requires no business-logic changes.

## Introduction

A messaging middleware skill based on the [Snet](https://www.nuget.org/profiles/Shun) framework, covering:

| Middleware | Operate class | Highlights |
|--------|--------|------|
| **MQTT** | `MqttClientOperate` / `MqttServiceOperate` / `MqttWebSocketServiceOperate` | Client + embedded broker + WebSocket, QoS 0/1/2 |
| **Kafka** | `KafkaOperate` | High throughput, SASL auth, `BootstrapServers` connection |
| **RabbitMQ** | `RabbitMQOperate` | AMQP protocol, 4 exchange types (direct/topic/fanout/headers) |
| **RocketMQ** | `RocketMQOperate` | Apache RocketMQ 5.x, gRPC to Proxy, ACL/SSL |
| **NetMQ** | `NetMQOperate` | ZeroMQ Pub/Sub, brokerless, <1ms latency |
| **Netty** | `NettyClientOperate` / `NettyServiceOperate` | DotNetty high-performance TCP, SSL support |

## Interaction Flow

> The AI asks the user in plain language and generates code after confirmation:

| # | What the AI asks | How the user answers | What the AI maps it to |
|---|-----------|---------------|--------------|
| 1 | Which message middleware? | "MQTT" "Kafka" | → middleware package + Operate class |
| 2 | What is the broker address? | "127.0.0.1:1883" | → connection parameters |
| 3 | Username/password needed? | "No" | → UserName/Password |
| 4 | Send or receive messages? | "Send" "Receive" "Both" | → Produce / Consume |
| 5 | What is the topic? | "factory/line1" | → Topic |
| 6 | Text or binary messages? | "String" | → Produce(string) + ResponseType |

## Unified API (IMq Interface)

```csharp
await mq.OnAsync();                              // connect
await mq.ProduceAsync(topic, content);           // produce
mq.OnDataEventAsync += async (s, e) => { ... };  // consumption data event
await mq.ConsumeAsync(topic);                    // subscribe
await mq.OffAsync();                             // disconnect
```

## DAQ Integration

`AddressMqParam` configuration + `config/mq/*.Mq.Config.json` registration → DAQ-acquired data is automatically forwarded to the middleware (see SKILL.md §6).

## Related Skills

- [DAQ-Skill](../DAQ-Skill) — data acquisition + auto-forwarding
- [PluginDev-Skill](../PluginDev-Skill) — develop custom IMq plugins (chapter 8)

**📄 File:** `SKILL.md` (9 chapters: unified IMq interface / 6 middleware guides / DAQ-linked forwarding / troubleshooting)
