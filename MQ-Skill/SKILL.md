---
name: mq-skill
description: Snet 消息中间件应用技能，覆盖 MQTT（Client/Service/WebSocket）、Kafka、RabbitMQ、RocketMQ、NetMQ、Netty 六种 IMq 传输协议的消息生产、消费、统一 API 与数据转发。支持"一句话"完成消息收发配置。配合 PluginDev-Skill 开发自定义 IMq 插件。
version: 1.0.1.2
metadata:
  hermes:
    tags: [mq, middleware, mqtt, kafka, rabbitmq, rocketmq, netmq, netty, iot, dotnet, message-queue]
    related_skills: [daq-skill, plugindev-skill]
    homepage: https://shunnet.top
---

# MQ-Skill — Snet 消息中间件应用技能

> **一句话：** 用户描述消息需求 → AI 自动生成完整可运行的中间件收发代码

## 一句话生成消息代码

本技能支持用户用一句自然语言描述需求，自动生成完整可编译运行的代码：

> "通过MQTT连接到 127.0.0.1:1883，topic为factory/line1，接收温度数据并打印" 或 "把采集的数据通过Kafka发送到集群 192.168.0.1:9092"

→ 自动生成包含：项目创建 + NuGet安装 + 中间件配置 + 生产/消费 + 日志 + 错误处理的完整代码

---

## ⚡ 统一 IMq 接口（所有中间件共享）

Snet 的 6 种消息中间件（MQTT/Kafka/RabbitMQ/RocketMQ/NetMQ/Netty）全部实现统一 **`IMq`** 接口（Snet.Model/interface/IMq.cs），上层代码无需区分具体中间件：

```csharp
public interface IMq : IOn, IOff, IProducer, IConsumer, IStatus,
    IEvent, IArgs, IInstance, ILog, ILanguage, IClone, IDisposable, IAsyncDisposable { }
```

### 统一 API 对照表

| 操作 | 同步 | 异步 | 说明 |
|------|------|------|------|
| 连接 | `mq.On()` | `await mq.OnAsync()` | 连接 Broker |
| 断开 | `mq.Off()` | `await mq.OffAsync()` | 断开连接 |
| 状态 | `mq.GetStatus()` | `await mq.GetStatusAsync()` | 连接状态 |
| 生产(字符串) | `mq.Produce(topic, content)` | `await mq.ProduceAsync(topic, content)` | 发布字符串消息 |
| 生产(字节) | `mq.Produce(topic, bytes)` | `await mq.ProduceAsync(topic, bytes)` | 发布字节消息 |
| 消费 | `mq.Consume(topic)` | `await mq.ConsumeAsync(topic)` | 订阅主题 |
| 取消消费 | `mq.UnConsume(topic)` | `await mq.UnConsumeAsync(topic)` | 取消订阅 |
| 获取参数 | `mq.GetBasicsArgs()` | `await mq.GetBasicsArgsAsync()` | 获取当前 Basics 配置 |
| 更新参数 | `mq.UpdateArgs(config)` | `await mq.UpdateArgsAsync(config)` | 运行期替换单例参数（原子换键，失败回滚） |
| 克隆实例 | `mq.CloneThis()` | `await mq.CloneThisAsync()` | 克隆新实例（不入单例池，独立连接） |

> **⚠️ 已知缺陷（v26.226.1）：** `UpdateArgsAsync` 成功路径返回 `Status=false`（底层 `EndOperateAsync(false, "参数修改成功")`）——判断结果请用 `GetDetails` 消息内容；`Off(true)` 同步版不透传 `hardClose`（未连接时仍报"未连接"），强制关闭请用 `await OffAsync(true)`。

### 消费数据接收（统一事件）

所有中间件消费到的消息统一经 **`OnDataEventAsync`** 事件推送，`EventDataResult.ResultData` 的类型由 `ResponseType` 配置决定：

```csharp
mq.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    // ResponseType.Content（默认）→ string
    // ResponseType.Bytes → byte[]
    // ResponseType.ContentWithTopic → ResponseModel(Topic, Content)
    Console.WriteLine($"收到: {e.GetSource<string>()}");
    await Task.CompletedTask;
};
await mq.ConsumeAsync("my/topic");
```

| ResponseType | ResultData 类型 | 适用 |
|-------------|----------------|------|
| `Content`（默认） | `string` | 文本消息 |
| `Bytes` | `byte[]` | 二进制消息 |
| `ContentWithTopic` | `ResponseModel(Topic, Content)` | 需要携带主题 |

---

## NuGet 引用规范

> **⚠️ 以下规则在生成代码时必须遵守，违反会导致编译或运行时错误。**

### 版本号：必须指定

所有 `dotnet add package` 命令**必须**带 `-v` 版本号（当前统一版本 **26.236.1**），**禁止使用 `*` 通配符**。安装前到 nuget.org 确认最新版本。

```bash
# ✅ 正确：指定版本号
dotnet add package Snet.Mqtt -v 26.236.1

# ❌ 错误：不带版本号（使用 * 通配符，运行时报错）
dotnet add package Snet.Mqtt
```

### 传递依赖：只引用中间件包

| 不要手动添加的包 | 原因 |
|------------------|------|
| `Snet.Core` | 中间件包的传递依赖 |
| `Snet.Model` | 中间件包的传递依赖 |
| `Snet.Log` | 中间件包的传递依赖 |
| `Snet.Utility` | 中间件包的传递依赖 |

```bash
# ✅ 正确：只引用中间件包
dotnet add package Snet.Mqtt -v 26.236.1
```

### using 语句 vs NuGet 引用

`using` 是命名空间导入，必须保留（命名空间 ≠ 包名）：

```csharp
using Snet.Mqtt.client;              // ← 来自 Snet.Mqtt 包
using MQTTnet.Protocol;              // ← MqttQualityOfServiceLevel 枚举所在命名空间（QoS 配置时需要）
using Snet.Model.data;               // ← 来自传递依赖 Snet.Model
using Snet.Model.@enum;              // ← ResponseType 等
```

---

## 0. 交互流程 — 生成代码前先问清楚

> **原则：用户说大白话就行，AI 负责翻译成技术参数。**

| # | 大白话问题 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | **用什么消息中间件？** | "MQTT" "Kafka" "RabbitMQ" "RocketMQ" "ZeroMQ" "Netty" | → 中间件包 + Operate 类 |
| 2 | **Broker 地址是多少？** | "127.0.0.1:1883" "192.168.0.1:9092" | → IpAddress+Port / BootstrapServers / Address |
| 3 | **需要账号密码吗？** | "要，admin/123" "不用" | → UserName + Password |
| 4 | **发消息还是收消息？** | "发" "收" "都做" | → Produce / Consume / 两者 |
| 5 | **主题（topic）叫什么？** | "factory/line1" "温度数据" | → Topic |
| 6 | **消息是文本还是二进制？** | "就是字符串" "是协议字节" | → Produce(string) / Produce(byte[]) + ResponseType |

**用户说不清楚时：**

| 用户说 | AI 追问 |
|--------|--------|
| "搞个消息队列" | "好的！用哪种中间件？MQTT、Kafka、RabbitMQ、RocketMQ、还是别的？" |
| "MQTT" | "Broker 地址是多少？一般是 IP:端口 格式，比如 127.0.0.1:1883" |
| "发消息" | "发到哪个主题？消息内容是什么格式？" |
| "收消息" | "订阅哪个主题？收到后要怎么处理？" |

---

## 1. MQTT — 最常用的轻量消息协议

> NuGet: `dotnet add package Snet.Mqtt -v 26.236.1`
> 三种操作类：`MqttClientOperate`（连接外部 Broker，继承 MqAbstract）/ `MqttServiceOperate`（内置 Broker）/ `MqttWebSocketServiceOperate`（浏览器 WebSocket）

### 1.1 MQTT Client（连接外部 Broker）

```csharp
using Snet.Mqtt.client;
using MQTTnet.Protocol;   // MqttQualityOfServiceLevel 枚举所在命名空间
using Snet.Model.@enum;  // ResponseType 枚举

var config = new MqttClientData.Basics
{
    SN = "my-mqtt",              // 重要：用于 ISns 匹配
    IpAddress = "127.0.0.1",     // ← 属性名是 IpAddress，不是 Ip！
    Port = 1883,                  // 库默认 6688，需改标准端口
    UserName = "admin",           // 默认 "shunnet"
    Password = "password",        // 默认 "shunnet"
    ClientID = null,              // 客户端ID，null 则自动生成随机
    MessageExpirationTime = 86400000,  // 消息过期时间(ms)，默认 24h
    QualityOfServiceLevel = MqttQualityOfServiceLevel.AtMostOnce, // ⚠️ 该值仅用于遗嘱(Will)消息；IMq 生产/消费固定 QoS0（ProduceAsync→PublishAsync 硬编码 QoSLevel 默认 0）
    ResponseType = ResponseType.Content,  // Content/Bytes/ContentWithTopic
};
using var mq = await MqttClientOperate.InstanceAsync(config);
await mq.OnAsync();

// 生产（字符串）
await mq.ProduceAsync("topic/hello", "hello world");

// 生产（字节）
await mq.ProduceAsync("topic/hello", new byte[] { 0x01, 0x02, 0x03 });

// 消费（异步事件）
mq.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    string content = e.GetSource<string>();
    Console.WriteLine($"收到: {content}");
    await Task.CompletedTask;
};
await mq.ConsumeAsync("topic/hello");

Console.ReadKey();
await mq.OffAsync();
```

### 1.2 MQTT Service（内置 Broker — 无需外部 MQTT 服务器）

```csharp
using Snet.Mqtt.service;

var config = new MqttServiceData.Basics
{
    Port = 6688,          // 监听端口
    MaxNumber = 10000,    // TCP 连接积压数（backlog，非客户端数量上限）
};
using (var mqttService = await MqttServiceOperate.InstanceAsync(config))
{
    await mqttService.OnAsync();
    // 内置 MQTT Broker 已启动，客户端可连接 mqtt://127.0.0.1:6688
}
```

> **注意：** `MqttServiceOperate` 继承 `CoreUnify`（非 MqAbstract），只有 `On/Off/GetStatus`，**没有** `Produce/Consume`——它是 Broker 服务端，不是客户端。

### 1.3 MQTT WebSocket Service（浏览器 WebSocket 连接）

```csharp
using Snet.Mqtt.service.websocket;

var config = new MqttWebSocketServiceData.Basics
{
    Port = 6688,          // MQTT 端口
    WsPort = 8866,        // WebSocket 端口
};
using (var wsService = await MqttWebSocketServiceOperate.InstanceAsync(config))
{
    await wsService.OnAsync();
    // 浏览器连接地址: ws://127.0.0.1:8866/shun（Uri 默认 "shun"）
}
```

> **📌 WebSocket 连接路径：** `MqttWebSocketServiceData.Basics.Uri` 默认 `"shun"`，浏览器必须连 `ws://{IpAddress}:{WsPort}/{Uri}`（如 `ws://127.0.0.1:8866/shun`），连根路径会 404。
> **⚠️ 单实例限制：** 该类配置为静态字段，多个实例会互相覆盖——一个进程内只能创建一个 WebSocket Broker。

### 1.4 MQTT 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `SN` | `string` | 随机 GUID | 实例标识（ISns 匹配用） |
| `IpAddress` | `string` | `"127.0.0.1"` | Broker 地址 |
| `Port` | `int` | `6688` | Broker 端口（标准 1883） |
| `UserName` / `Password` | `string?` | `"shunnet"` | 认证凭据 |
| `ClientID` | `string?` | `null` | 客户端 ID（null 自动生成） |
| `MessageExpirationTime` | `int` | `86400000` | 消息过期(ms) |
| `QualityOfServiceLevel` | `MqttQualityOfServiceLevel` | `AtMostOnce` | ⚠️ 仅用于**遗嘱(Will)消息** QoS；IMq 生产/消费固定 QoS0（源码硬编码），设 QoS2 对收发无效果 |
| `ResponseType` | `ResponseType` | `Content` | 消费数据格式 |

> **需要 QoS1/2 的订阅？** IMq 的 `ConsumeAsync` 固定 QoS0，需更高级别时用 `MqttClientOperate` 直连 API：`await mq.AddSubscribeAsync(topic, qosLevel)` / `await mq.RemoveSubscribeAsync(topic)`（QoSLevel 默认 0）。

## 2. Kafka — 分布式高吞吐消息队列

> NuGet: `dotnet add package Snet.Kafka -v 26.236.1`
> Operate: `KafkaOperate`（继承 MqAbstract）| Config: `KafkaData.Basics`
> **注意：Kafka 用 `BootstrapServers`（非 IpAddress+Port）**

```csharp
using Snet.Kafka;
using Confluent.Kafka;        // SecurityProtocol / AutoOffsetReset 枚举所在命名空间
using Snet.Model.data;        // ResponseModel
using Snet.Model.@enum;       // ResponseType

var config = new KafkaData.Basics
{
    SN = "my-kafka",
    BootstrapServers = "192.168.0.1:9092",   // ← 属性名是 BootstrapServers，不是 IpAddress！
    SecurityProtocol = SecurityProtocol.Plaintext,  // Plaintext/Ssl/SaslPlaintext/SaslSsl
    // ⚠️ SASL 认证：Basics 只有 SaslMechanism（默认 Gssapi/Kerberos）+ SaslKerberosServiceName（默认 "snet"）
    // 无 SaslUsername/SaslPassword 属性！SASL 用户密码认证当前无法启用（除非源码补属性）。
    // 设 SaslPlaintext/SaslSsl 且不配 Kerberos 环境 → 认证必然失败。
    AutoOffsetReset = AutoOffsetReset.Latest,       // Earliest/Latest
    ResponseType = ResponseType.ContentWithTopic,   // Kafka 通常带 Topic
};
using var kafka = await KafkaOperate.InstanceAsync(config);
await kafka.OnAsync();

// 生产
await kafka.ProduceAsync("factory/line1", "{\"temp\":25.6}");

// 消费（手动 Offset 提交由框架处理）
kafka.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    var msg = e.GetSource<ResponseModel>();   // ResponseType.ContentWithTopic → ResponseModel
    Console.WriteLine($"[{msg.Topic}] {msg.Content}");
    await Task.CompletedTask;
};
await kafka.ConsumeAsync("factory/line1");

Console.ReadKey();
await kafka.OffAsync();
```

### Kafka 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `BootstrapServers` | `string?` | `null` | Kafka 集群地址（`"host:9092"`） |
| `SecurityProtocol` | `SecurityProtocol` | `Plaintext` | 安全协议 |
| `SaslMechanism` | `SaslMechanism` | `Gssapi` | SASL 机制（默认 Kerberos；⚠️ 无 SaslUsername/SaslPassword 属性，用户密码认证不可用） |
| `SaslKerberosServiceName` | `string` | `"snet"` | Kerberos 服务名 |
| `AutoOffsetReset` | `AutoOffsetReset` | `Latest` | 无 Offset 时从哪开始 |
| `ResponseType` | `ResponseType` | `Content` | 消费数据格式 |

### Kafka 扩展能力（v26.226.1+）

**带 Key 生产（分区路由）：**
```csharp
// 三参重载: ProduceAsync(topic, key, content) — Key 决定消息进入哪个分区
await kafka.ProduceAsync("factory/line1", "device-001", "{\"temp\":25.6}");
```

**AdminClient 主题管理（自动建主题）：**
```csharp
await kafka.OnAsync();                      // 必须先打开
var created = await kafka.CreateTopicsAsync(new List<string> { "factory/line1", "factory/line2" });
// 内部: GetMetadata(5s) 取已有主题求差集 → 仅创建不存在的主题 → Wait(5s)
// 重复调用幂等，已存在的主题跳过
```

---

## 3. RabbitMQ — AMQP 协议

> NuGet: `dotnet add package Snet.RabbitMQ -v 26.236.1`
> Operate: `RabbitMQOperate`（继承 MqAbstract）| Config: `RabbitMQData.Basics`
> **注意：RabbitMQ 有 `ExChangeName`（交换机名称）+ `Publish/Consume` 用 `Type` 参数指定交换机类型（direct/topic/fanout/headers）**

```csharp
using Snet.RabbitMQ;
using Snet.Model.@enum;   // ResponseType

var config = new RabbitMQData.Basics
{
    SN = "my-rabbit",
    ExChangeName = "exchang",      // 交换机名称
    IpAddress = "127.0.0.1",
    Port = 5672,                    // 库默认 6688，需改 AMQP 标准端口 5672
    UserName = "shunnet",           // 默认 "shunnet"
    Password = "shunnet",
    MessageExpirationTime = 86400000,
    ResponseType = ResponseType.Content,
};
using var rabbit = await RabbitMQOperate.InstanceAsync(config);
await rabbit.OnAsync();

// 生产（Publish 方法，Type 参数 = 交换机类型）
await rabbit.PublishAsync("hello content", Topic: "my.topic", Type: "topic");

// 消费
rabbit.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    Console.WriteLine($"收到: {e.GetSource<string>()}");
    await Task.CompletedTask;
};
await rabbit.ConsumeAsync(Topic: "my.topic", Type: "topic");

Console.ReadKey();
await rabbit.OffAsync();
```

### RabbitMQ 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ExChangeName` | `string` | `"exchang"` | 交换机名称 |
| `IpAddress` / `Port` | `string` / `int` | `"127.0.0.1"` / `6688` | Broker 地址（AMQP 标准端口 5672） |
| `UserName` / `Password` | `string?` | `"shunnet"` | 认证凭据 |
| `MessageExpirationTime` | `int` | `86400000` | 消息过期(ms) |
| `ResponseType` | `ResponseType` | `Content` | 消费数据格式 |

**Publish/Consume 参数：** `Topic`（主题）/ `Queue`（队列）/ `RoutingKey`（路由键）/ `Type`（交换机类型：`"direct"`/`"topic"`/`"fanout"`/`"headers"`，默认 `"topic"`）/ `Durable`/`Exclusive`/`AutoDelete`（队列属性）/ `AutoAck`（消费自动确认）。

---

## 4. NetMQ — ZeroMQ 无 Broker 模式

> NuGet: `dotnet add package Snet.NetMQ -v 26.236.1`
> Operate: `NetMQOperate`（继承 MqAbstract）| Config: `NetMQData.Basics`
> **注意：NetMQ 用 `Address`（ZMQ 地址格式）+ `UModel`（PubModel/SubModel）**

```csharp
using Snet.NetMQ;
using static Snet.NetMQ.NetMQData;   // UseModel 是 NetMQData 的嵌套枚举，必须 using static 才能裸用

// ⚠️ 一个实例一个角色：PubModel 只能生产，SubModel 只能消费。
// socket 类型在 OnAsync 时定型，事后改 UModel 不会重建 socket：消费路径会空引用崩溃
// 或静默收不到消息（GetSource<T> 类型不符返回 null，不抛 InvalidCastException）。
// 正确做法：生产/消费各用一个实例；确需改角色必须 Off→改 UModel→On 重建。

// 发布者（PubModel）
var pubConfig = new NetMQData.Basics
{
    SN = "my-netmq-pub",
    Address = "tcp://127.0.0.1:8866",   // ZMQ 地址格式
    UModel = UseModel.PubModel,          // 发布者
};
using var pub = await NetMQOperate.InstanceAsync(pubConfig);
await pub.OnAsync();
await pub.ProduceAsync("topic1", "hello zeromq");

// 订阅者（SubModel）——需另开进程/实例，同一 Address 两端绑定
var subConfig = new NetMQData.Basics
{
    SN = "my-netmq-sub",
    Address = "tcp://127.0.0.1:8866",
    UModel = UseModel.SubModel,          // 订阅者
};
using var sub = await NetMQOperate.InstanceAsync(subConfig);
await sub.OnAsync();
sub.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    Console.WriteLine($"收到: {e.GetSource<string>()}");
    await Task.CompletedTask;
};
await sub.ConsumeAsync("topic1");

Console.ReadKey();
await sub.OffAsync();
await pub.OffAsync();
```

### NetMQ 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `Address` | `string?` | `"tcp://127.0.0.1:8866"` | ZMQ 地址 |
| `UModel` | `UseModel` | `PubModel` | `PubModel`=发布 / `SubModel`=订阅 |
| `TimeOut` | `int` | `1000` | 超时(ms) |

> **注意：** `ConsumeAsync` 要求 `UModel == SubModel`，否则返回"未使用订阅模式"；socket 类型在 `OnAsync` 时定型，事后改 UModel 不会重建 socket（消费路径空引用崩溃或静默收不到消息）——**角色切换需 `OffAsync` → 改 UModel → `OnAsync` 重建**，生产/消费建议各用一个实例。

---

## 5. Netty — 高性能 TCP 框架

> NuGet: `dotnet add package Snet.Netty -v 26.236.1`
> 两种操作类：`NettyClientOperate`（客户端，继承 MqAbstract）/ `NettyServiceOperate`（服务端）
> **注意：Netty 支持 SSL 加密（SslFilePath/SslFilePassword）**

### 5.1 Netty Client

```csharp
using Snet.Netty.client;

var config = new NettyClientData.Basics
{
    SN = "my-netty",
    IpAddress = "127.0.0.1",
    Port = 6688,
    // SSL（如需要）：
    // SslFilePath = "server.pfx",
    // SslFilePassword = "password",
    TaskNumber = 5,        // 并发消费任务数
};
using var client = await NettyClientOperate.InstanceAsync(config);
await client.OnAsync();

// 生产
await client.ProduceAsync("cmd", new byte[] { 0x01, 0x03, 0x00, 0x00 });

// 消费
client.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    Console.WriteLine($"收到: {e.GetSource<string>()}");
    await Task.CompletedTask;
};
await client.ConsumeAsync("cmd");

Console.ReadKey();
await client.OffAsync();
```

### 5.2 Netty Service（服务端）

```csharp
using Snet.Netty.service;

var config = new NettyServiceData.Basics
{
    Port = 6688,
    // SslFilePath / SslFilePassword（如需要 SSL）
};
using (var service = await NettyServiceOperate.InstanceAsync(config))
{
    await service.OnAsync();
    // TCP 服务端已启动

    byte[] bytes = System.Text.Encoding.UTF8.GetBytes("hello");

    // 定向发送给指定客户端（"IP:Port" 标识，不存在则报"终端不存在"）
    await service.SendAsync(bytes, new[] { "192.168.1.50:5000" });
    // IpPort 传 null 则群发全部已连接客户端
    await service.SendAsync(bytes);

    // 带主题的消息重载（内部组织 JSON {T,C} 发送）
    await service.SendAsync("factory/line1", "hello", new[] { "192.168.1.50:5000" });
}
```

> **注意：** `NettyServiceOperate` 继承 `CoreUnify`（非 MqAbstract），没有 `Produce/Consume`（IMq 生产/消费），但提供 `SendAsync(byte[] Data, string[]? IpPort = null)` 定向/群发与 `SendAsync(string Topic, string Content, string[]? IpPort)` 带主题重载。

### 5.3 Netty 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `IpAddress` / `Port` | `string` / `int` | `"127.0.0.1"` / `6688` | 服务端地址 |
| `SslFilePath` | `string?` | `null` | SSL 证书路径 |
| `SslFilePassword` | `string?` | `null` | SSL 证书密码 |
| `TaskNumber` | `int` | `5` | 并发消费任务数 |

---

## 6. RocketMQ — Apache RocketMQ 5.x 消息队列

> NuGet: `dotnet add package Snet.RocketMQ -v 26.236.1`
> Operate: `RocketMQOperate`（继承 MqAbstract）| Config: `RocketMQData.Basics`
> 底层依赖 RocketMQ.Client 5.2.1（Apache RocketMQ 5.x .NET 客户端，**经 gRPC 协议连接 Proxy**）

```csharp
using Snet.RocketMQ;
using Snet.Model.@enum;   // ResponseType

var config = new RocketMQData.Basics
{
    SN = "my-rocketmq",         // 实例标识（ISns 匹配用）
    IpAddress = "127.0.0.1",
    Port = 8081,                // ⚠️ RocketMQ 5.x 客户端连 Proxy（gRPC），标准端口 8081（库默认 6688 需改）
    ConsumerGroup = "snet",     // 消费组（默认 "snet"）
    SslEnabled = false,         // ⚠️ SDK 默认开启 TLS；本地无 TLS Broker 保持 false
    AccessKey = null,           // ACL 认证（与 SecretKey 必须成对配置）
    SecretKey = null,
    ResponseType = ResponseType.Content,  // Content/Bytes/ContentWithTopic
};
using var rmq = await RocketMQOperate.InstanceAsync(config);
await rmq.OnAsync();

// 消费（异步事件，先绑定再订阅）
rmq.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    string content = e.GetSource<string>();
    Console.WriteLine($"收到: {content}");
    await Task.CompletedTask;
};
await rmq.ConsumeAsync("my/topic");   // 首次订阅懒创建 PushConsumer

// 生产（字节）
await rmq.ProduceAsync("my/topic", System.Text.Encoding.UTF8.GetBytes("hello rocketmq"));

Console.ReadKey();
await rmq.OffAsync();
```

### RocketMQ 配置速查

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `SN` | `string` | 随机 GUID | 实例标识（ISns 匹配用） |
| `IpAddress` | `string` | `"127.0.0.1"` | Proxy 地址 |
| `Port` | `int` | `6688` | ⚠️ gRPC 连 Proxy——标准 RocketMQ 5.x 部署须设 **8081** |
| `AccessKey` | `string?` | `null` | ACL AccessKey（与 SecretKey 必须成对） |
| `SecretKey` | `string?` | `null` | ACL SecretKey（与 AccessKey 必须成对） |
| `ConsumerGroup` | `string` | `"snet"` | 消费组 |
| `SslEnabled` | `bool` | `false` | 启用 SSL（SDK 默认 TLS，本地 Broker 关） |
| `ResponseType` | `ResponseType` | `Content` | 消费数据格式 |

### RocketMQ 关键行为

| 行为 | 说明 |
|------|------|
| **消费者懒创建** | `OnAsync` 只建 Producer；首个 `ConsumeAsync` 才建 PushConsumer（SDK Build 要求非空初始订阅，以首个主题初始化），后续订阅动态追加 |
| **重复订阅** | 已订阅的主题再次 `ConsumeAsync` → 失败「此主题已订阅」 |
| **取消订阅** | 未订阅的主题 `UnConsumeAsync` → 失败「此主题不存在」；最后一个主题取消后销毁消费者 |
| **消息保序** | `ProduceAsync` 将消息 Key 设为 topic，同主题消息同分区保序；成功返回消息 ID |
| **事件推送** | fire-and-forget（不阻塞 SDK 消费线程）；handler 抛异常返回 FAILURE → RocketMQ 按重试策略自动重投 |

> **故障排查：** 连不上先查 `Port`（库默认 6688 vs 标准 Proxy 8081）与 `SslEnabled` 是否与 Broker 一致；收不到消息查 `ConsumerGroup` 是否与其他消费组冲突。

---

## 7. 与 DAQ 联动 — 采集数据自动转发

DAQ 采集的数据可自动转发到任意中间件（**关键机制，详见 DAQ-Skill §1.1**）：

### 7.1 前提：config/mq 配置文件注册实例

> 自动转发只识别 **`config/mq/` 目录下 `{完整命名空间}.{类名}.{SN}.Mq.Config.json` 配置文件加载的实例**（`MqOperate.InstanceIoc` 注册表）。用户代码 `new` 创建的实例不注册，转发会报 "实例未找到"。

> **⚠️ 热加载 key 不一致（已知缺陷，v26.226.1）：** `MqOperate` **启动时扫描**注册的实例 key 与 ISns 格式一致（`命名空间.类名.SN`）；但**运行期间新增/修改**配置文件（watcher 热加载路径）注册的 key 会多保留 `.Mq.Config` 后缀，此时 ISns 定向转发仍报"实例未找到"。**规避：** 在 MqOperate 启动前放好配置文件；运行期动态加 MQ 实例后请重启应用或改用 §6.2 事件内手动 `ProduceAsync`。

```json
// config/mq/Snet.Mqtt.client.MqttClientOperate.mqtt-target.Mq.Config.json
{
  "SN": "mqtt-target",
  "IpAddress": "127.0.0.1",
  "Port": 1883,
  "UserName": "shunnet",
  "Password": "shunnet",
  "ClientID": null,
  "MessageExpirationTime": 86400000,
  "QualityOfServiceLevel": "AtMostOnce",
  "ResponseType": "Content"
}
```

### 7.2 AddressMqParam 转发配置

```csharp
new AddressDetails
{
    SN = "温度",
    AddressName = "DB1.0",
    AddressDataType = DataType.Float,
    AddressMqParam = new AddressMq
    {
        ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.mqtt-target" },  // 命名空间.类名.SN
        Topic = "factory/siemens",
        ContentFormat = "DB1.0 = {0}",   // {0} = 点位值
    }
}
```

**ISns 格式速查：**

| MQ 类型 | ISns 格式 |
|---------|----------|
| MQTT Client | `"Snet.Mqtt.client.MqttClientOperate.{SN}"` |
| Kafka | `"Snet.Kafka.KafkaOperate.{SN}"` |
| RabbitMQ | `"Snet.RabbitMQ.RabbitMQOperate.{SN}"` |
| RocketMQ | `"Snet.RocketMQ.RocketMQOperate.{SN}"` |
| NetMQ | `"Snet.NetMQ.NetMQOperate.{SN}"` |
| Netty | `"Snet.Netty.client.NettyClientOperate.{SN}"` |

---

## 8. 故障排查

| 症状 | 原因 | 解决 |
|------|------|------|
| 连接超时 | 端口不对（库默认 6688） | 显式设置标准端口（MQTT=1883, Kafka=9092, RabbitMQ=5672） |
| Kafka 连不上 | 用了 IpAddress 而非 BootstrapServers | 用 `BootstrapServers = "host:9092"` |
| NetMQ 消费失败 | UModel 不是 SubModel | 消费实例设 `UModel = UseModel.SubModel` |
| 收不到消息 | 未订阅/未设置 OnDataEventAsync | 先 `ConsumeAsync(topic)` 再等事件 |
| 收到的是字节 | ResponseType=Bytes | 设 `ResponseType = ResponseType.Content` 或按字节处理 |
| 自动转发报"实例未找到" | MQ 实例未经 config/mq 注册 | 创建配置文件（见 §7.1） |
| 缺 using MQTTnet.Protocol | QoS 枚举命名空间 | 添加 `using MQTTnet.Protocol;` |
| 新订阅者立即收到旧消息 | MQTT 生产默认 `Retain=true`（IMq ProduceAsync 硬编码）| 需精确控制时用 `PublishAsync(topic, content, Retain: false)` 直连 API |
| RabbitMQ 取消订阅后数据丢失 | ~~`UnConsumeAsync` 会**删除队列**~~（已修复）—— 现为 `BasicCancel`，**队列与积压消息保留** | 未消费主题返回失败「此主题未在消费」；删除队列需走 Broker 管理 API |
| 一处 `using var` 释放后全局断连 | `InstanceAsync` 同 SN/配置返回**共享实例**，Dispose 即 Off+出池 | 勿在多处 using 同一实例；确认无其他持有者再释放 |
| Kafka 断线重连后收不到旧消息 | ~~每次 `OnAsync` 生成新消费组~~（已修复）—— `GroupId` 为**固定配置**（默认 `"snet"`），偏移量**手动提交**（EnableAutoCommit=false），断线重连后从上次提交偏移续读 | 同组多实例分摊消息；需全新消费时换一个 GroupId |
| `Off(true)` 未连接时仍报"未连接" | 基类同步 `Off(bool)` 不透传 `hardClose`（MqAbstract.cs:34，已知缺陷）| 强制关闭用 `await mq.OffAsync(true)` |
| `UpdateArgsAsync` 返回 Status=false | 底层成功路径误用 `EndOperateAsync(false, "参数修改成功")`（已知缺陷）| 按 `GetDetails` 消息内容判断，勿只看 Status |
| 运行期新增 config/mq 文件后 ISns 失效 | 热加载注册 key 多保留 `.Mq.Config` 后缀（MqOperate.Monitor.cs，已知缺陷）| 启动前放好配置文件；动态新增后重启（见 §7.1） |
| 断线后收不到消息 | MQTT 自动重连 5 次（5/10/15/20/30 秒退避）后仍失败才 Off；但 CleanSession 丢订阅，**重连成功不自动恢复订阅** | 在「重连成功」信息事件回调里重新 `ConsumeAsync(topic)` |
| 消费 handler 异常后消息丢失 | 异常只发 `OnInfoEventAsync`（不落盘）| 订阅 `OnInfoEventAsync` 并在 handler 内 try-catch |
| 实例重复创建 | 直接 new 绕过单例池 | 用 `await XxxOperate.InstanceAsync(config)` |

---

## 9. 相关技能

> **开发自定义 IMq 插件（新协议中间件）？** → 用 [PluginDev-Skill](../PluginDev-Skill) 第 8 章（MqAbstract 契约，6 个抽象方法）。
>
> **采集数据自动转发？** → 用 [DAQ-Skill](../DAQ-Skill)（AddressMqParam 配置）。

---

## 📅 版本历史

| 版本 | 日期 | 变更 |
|:---|:---|:---|
| 1.0.1.2 | 2026-08-24 | 对照源码升级（Shunnet @46ef840，NuGet 26.235.x→**26.236.1** 全包统一）：安装命令与统一版本号更新（六种 IMq 传输协议一律 26.236.1）；已知缺陷注记保留（UpdateArgsAsync Status=false、热加载 key 不一致、Off(true) 不透传 hardClose）——26.236 源码核对仍成立 |
| 1.0.1.1 | 2026-08-24 | 对照源码升级（Shunnet @ffd40cd，NuGet 26.226.1→**26.235.2**）：安装命令与统一版本号更新（六种 IMq 传输协议）；已知缺陷注记保留（UpdateArgsAsync Status=false、热加载 key 不一致、Off(true) 不透传 hardClose）——26.235 源码核对仍成立 |
| 1.0.1.0 | 2026-08-14 | 对照源码升级（26.222.1→**26.226.1**）：**新增 §6 RocketMQ 章节**（Apache RocketMQ 5.x gRPC 连 Proxy 8081、消费者懒创建、ACL/SSL、配置速查、关键行为表），中间件 5 种→**6 种**（frontmatter/tags/统一接口/交互选项/ISns 表同步）；§7 故障排查 3 条行为改写：RabbitMQ 取消订阅不再删队列（BasicCancel 保留队列）、Kafka GroupId 固定 "snet"（偏移手动提交、重连续读）、MQTT 自动重连 5 次（CleanSession 重连后需重新订阅）；补 §2 Kafka 章节标题（原缺失）；版本历史与版本号标注统一 26.226.1 |
| 1.0.0.9 | 2026-08-11 | 对照源码升级：NuGet 版本 26.214.1→26.222.1（8 处）；统一 API 对照表补 GetBasicsArgs/UpdateArgs/CloneThis 三行 + 已知缺陷警告（UpdateArgsAsync Status=false、Off(true) 不透传 hardClose）；§1.3 WebSocket 补 Uri 默认 "shun"（连接地址 ws://host:8866/shun）与单实例限制；§2 Kafka 补 AdminClient CreateTopicsAsync 与带 Key 三参 ProduceAsync；§5.2 Netty Service 补 SendAsync 定向/群发；§6.1 补热加载 key 不一致缺陷警告；§7 故障排查表 +3 条；code-review 修正：§1.1 补 using Snet.Model.@enum、§1.4 补 AddSubscribeAsync QoS、§4 NetMQ 改 UModel 不抛 InvalidCastException（空引用/静默）、§5.2 Netty 示例补 bytes 声明、§7 补 Kafka 消费组随机警告、MaxNumber 注释改为 backlog |
| 1.0.0.8 | 2026-08-02 | 初始版本：MQTT/Kafka/RabbitMQ/NetMQ/Netty 五种中间件完整应用指南（统一 IMq 接口/配置速查/生产消费示例/与 DAQ 联动转发/config 注册前提/故障排查） |
