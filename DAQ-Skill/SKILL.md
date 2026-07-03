---
name: daq-skill
description: 工业物联网数据采集通信库，基于 Snet 框架，支持 PLC/工控/电力/机器人 等 30+ 种工业协议的数据读取、写入、订阅、状态获取，以及 Kafka/MQTT/RabbitMQ/NetMQ/Netty 消息中间件转发。所有采集库通过 ProtocolType 枚举自动选择底层驱动。支持"一句话"完成采集+转发。
version: 1.0.0.6
metadata:
  hermes:
    tags: [daq, iot, plc, industrial-automation, modbus, siemens, opc-ua, mqtt, kafka]
    related_skills: [plugindev-skill]
    homepage: https://shunnet.top
---

# DAQ-Skill — 工业物联网数据采集技能

> **运行环境：** .NET 10.0 SDK，所有 NuGet 包通过 `dotnet add package` 安装

## 一句话生成采集代码

本技能支持用户用一句自然语言描述需求，自动生成完整可编译运行的代码：

> "连接西门子S7-1500，IP 192.168.0.1，读取DB1.0（Int32）、DB1.4（Float）、M0.0（Bool），每500ms通过MQTT转发到broker 127.0.0.1:1883，topic为factory/line1"

→ 自动生成包含：项目创建 + NuGet安装 + DAQ配置 + MQ配置 + 数据转发 + 日志 + 错误处理的完整代码

→ 核心要点：**地址配置中的 `AddressMqParam` 实现订阅数据自动转发，无需手动编码转发逻辑**

---

## ⚡ 异步架构（重要！v1.0.0.5+）

> **Snet 框架底层已全面切换为异步实现。** 所有核心方法（On/Off/Read/Write/Subscribe/UnSubscribe/GetStatus/GetBaseObject）的真实实现均为 `*Async` 版本，同步方法仅是 `.GetAwaiter().GetResult()` 的薄封装。

### 异步优先原则

| 层级 | 异步方法（真实实现） | 同步方法（薄封装） |
|------|---------------------|-------------------|
| **接口** | `Task<OperateResult> XxxAsync(CancellationToken)` | `OperateResult Xxx()` |
| **抽象基类** | `abstract Task<OperateResult> XxxAsync(...)` | `Xxx()` → `XxxAsync().GetAwaiter().GetResult()` |
| **协议实现** | `override async Task<OperateResult> XxxAsync(...)` | 继承自基类 |
| **消息中间件** | `override async Task<OperateResult> XxxAsync(...)` | 继承自基类 |

### 关键方法对照表

| Sync（旧，不推荐） | Async（新，推荐） | 新增参数 |
|---------------------|-------------------|----------|
| `operate.On()` | `await operate.OnAsync()` | `CancellationToken` |
| `operate.Off()` | `await operate.OffAsync()` | `CancellationToken` |
| `operate.Off(true)` | `await operate.OffAsync(true)` | `CancellationToken` |
| `operate.Read(address)` | `await operate.ReadAsync(address)` | `CancellationToken` |
| `operate.Write(values)` | `await operate.WriteAsync(values)` | `CancellationToken` |
| `operate.Subscribe(address)` | `await operate.SubscribeAsync(address)` | `CancellationToken` |
| `operate.UnSubscribe(address)` | `await operate.UnSubscribeAsync(address)` | `CancellationToken` |
| `operate.GetStatus()` | `await operate.GetStatusAsync()` | `CancellationToken` |
| `mq.Produce(topic, content)` | `await mq.ProduceAsync(topic, content)` | `CancellationToken` |
| `mq.Consume(topic)` | `await mq.ConsumeAsync(topic)` | `CancellationToken` |
| `operate.WAOn(model)` | `await operate.WAOnAsync(model)` | `CancellationToken` |
| `operate.WAOff()` | `await operate.WAOffAsync()` | `CancellationToken` |

### 代码风格选择

```csharp
// ✅ 推荐：async/await（控制台/Web/服务端）
await operate.OnAsync();
var result = await operate.ReadAsync(address);
await operate.OffAsync();

// ⚠️ 可用但不推荐：同步包装（仅适合简单脚本）
operate.On();
var result = operate.Read(address);
operate.Off();

// ❌ 禁止：Task.Run 包裹同步方法（使用同步包装器即可）
```

> **生成代码默认使用 async/await 模式**，除用户明确要求同步代码外。

---

## NuGet 引用规范

> **⚠️ 以下规则在生成代码时必须遵守，违反会导致编译或运行时错误。**

### 版本号：必须指定

所有 `dotnet add package` 命令**必须**带 `-v` 版本号，**禁止使用 `*` 通配符**（运行时会报版本冲突错误）。

**获取最新版本的方式：** 访问 `https://www.nuget.org/packages/<包名>`，页面顶部显示的版本即为最新稳定版。

示例：
- `https://www.nuget.org/packages/Snet.Siemens` → 最新版本
- `https://www.nuget.org/packages/Snet.Modbus` → 最新版本
- `https://www.nuget.org/packages/Snet.Mqtt` → 最新版本

```bash
# ✅ 正确：指定版本号
dotnet add package Snet.Siemens -v 1.0.0.1
dotnet add package Snet.Mqtt -v 1.0.0.1

# ❌ 错误：不带版本号（使用 * 通配符，运行时报错）
dotnet add package Snet.Siemens
dotnet add package Snet.Mqtt
```

### 传递依赖：只引用顶层驱动包

各驱动包已内置传递依赖，**只需引用顶层驱动包 + MQ 包**，以下库会自动引入，**不要手动添加：**

| 不要手动添加的包 | 原因 |
|------------------|------|
| `Snet.Core` | 所有驱动包的传递依赖 |
| `Snet.Model` | 所有驱动包的传递依赖 |
| `Snet.Log` | 所有驱动包的传递依赖 |
| `Snet.Utility` | 所有驱动包的传递依赖 |
| `Snet.Driver` | 所有驱动包的传递依赖 |

```bash
# ✅ 正确：只引用驱动包 + MQ 包
dotnet add package Snet.Siemens -v 1.0.0.1
dotnet add package Snet.Mqtt -v 1.0.0.1

# ❌ 错误：多引用了传递依赖包
dotnet add package Snet.Siemens -v 1.0.0.1
dotnet add package Snet.Mqtt -v 1.0.0.1
dotnet add package Snet.Core -v 1.0.0.1      # 不需要！
dotnet add package Snet.Model -v 1.0.0.1     # 不需要！
dotnet add package Snet.Log -v 1.0.0.1       # 不需要！
dotnet add package Snet.Utility -v 1.0.0.1   # 不需要！
```

### using 语句 vs NuGet 引用

`using` 是 C# 命名空间导入，不是 NuGet 引用。代码中的 `using Snet.Log;` 等语句**必须保留**（库的命名空间不等于包名）：

```csharp
// ✅ 这些 using 语句必须保留（命名空间导入）
using Snet.Siemens;          // ← 来自 Snet.Siemens 包
using Snet.Mqtt.client;      // ← 来自 Snet.Mqtt 包
using Snet.Model.data;       // ← 来自传递依赖 Snet.Model
using Snet.Log;              // ← 来自传递依赖 Snet.Log
using Snet.Utility;          // ← 来自传递依赖 Snet.Utility
```

---

## 0. 交互流程 — 生成代码前先问清楚

> **原则：用户说大白话就行，AI 负责翻译成技术参数。**
> 用户说不清的就问，一个问题一个问题来，别一股脑问太多。

### 0.1 DAQ 采集：需要问用户什么

用户可能只说一句："帮我连一下西门子PLC读点数据"，AI 必须先搞清楚以下信息再动手写代码：

| # | 大白话问题 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | **你的设备是什么牌子什么型号？** | "西门子S7-1500" "三菱Q系列" "欧姆龙NX" "一个温湿度传感器" | → ProtocolType + Operate 类 |
| 2 | **设备是用网线连的还是用线（USB/串口线）连的？** | "网线" "USB转串口" "没连，走HTTP接口" | → TCP / Serial / HTTP |
| 3 | **网线的话IP和端口是多少？串口的话COM口和波特率是多少？** | "192.168.0.1" "COM3,9600" | → IpAddress+Port / SerialPortInfo |
| 4 | **你要读哪些数据？能告诉我地址和类型吗？** | "DB1.0是整数，DB1.4是小数" "40001-40010是寄存器" | → AddressDetails 配置 |
| 5 | **读到的数据要发到哪里？** | "发到MQTT 127.0.0.1:1883" "发到Kafka" "不用转发，就看看" | → MQ 配置 + AddressMqParam |
| 6 | **要多久读一次？** | "实时，变了就发" "每5秒读一次" | → 订阅间隔 HandleInterval |

**注意：**
- 有些用户可能一句话说全了（"西门子S7-1500，IP 192.168.0.1，读DB1.0"），那就不需要再问
- 有些用户可能只知道"设备是西门子的"，那就一个个问
- 用户说"传感器"说不清型号，就问"能拍个设备照片或者告诉我型号吗？"
- 用户说"读那个温度数据"，就问"温度数据在哪个地址？是什么类型（整数还是小数）？"

### 0.2 数据转发：需要问用户什么

| # | 大白话问题 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | **读到的数据要发到哪里？** | "发到MQTT" "发到Kafka" "发到两个地方" "不用转发" | → 选择 MQ 类型 |
| 2 | **MQTT/Kafka 的地址是多少？** | "127.0.0.1:1883" | → IpAddress + Port |
| 3 | **需要账号密码吗？** | "要，admin/123" "不用" | → UserName + Password |
| 4 | **发到哪个主题（topic）？** | "factory/line1" "温度数据" | → Topic |
| 5 | **发出去的数据想长什么样？** | "直接发值就行" "带上地址名，比如 DB1.0=25.6" | → ContentFormat |

### 0.3 用户说不清楚时怎么办

| 用户说 | AI 追问 |
|--------|--------|
| "帮我连PLC" | "好的！PLC是什么牌子的？西门子、三菱、欧姆龙、还是别的？" |
| "西门子的" | "型号知道吗？比如S7-200、S7-1200、S7-1500？不确定的话告诉我大概什么时候买的也行" |
| "读温度" | "温度数据在PLC里的地址知道吗？比如DB1.0这种？不知道的话告诉我温度显示在PLC的哪个画面也行" |
| "传感器" | "传感器是什么牌子型号？有说明书吗？通信方式是网口还是串口？" |
| "数据发到服务器" | "服务器用的是什么协议？MQTT、Kafka、还是HTTP接口？地址是多少？" |

---

## 1. 核心架构

### 1.1 DAQ ↔ MQ 转发机制

```
订阅 → DAQ收到数据 → AddressHandler自动匹配
                     ↓
              AddressDetails.AddressMqParam
                     ↓
              自动调用 mqOperate.Produce(Topic, Content, ISns)
                     ↓
                所有匹配的 MQ 实例并发发送
```

**框架自动完成转发，用户只需：**
1. 启动 DAQ 并订阅地址
2. 启动 MQ 客户端（设置好 SN）
3. 在 AddressDetails 中配置 AddressMqParam（指定 ISns、Topic、ContentFormat）

### 1.2 ProtocolType 驱动机制

```
XxxData.Basics.ProtocolType → switch-case → 自动实例化对应底层驱动
```

---

## 2. 场景驱动代码生成（一句话采集）

### 2.1 场景模板：采集 + MQTT 转发

**用户输入：** "连接西门子S7-1500，IP 192.168.0.1，读取DB1.0（Int32）、DB1.4（Float），通过MQTT转发到127.0.0.1:1883，topic为factory/siemens"

**生成代码：**

> **📌 生成代码前，先创建项目并安装 NuGet 包（版本号到 nuget.org 查询）：**
> ```bash
> dotnet new console -n DaqDemo && cd DaqDemo
> dotnet add package Snet.Siemens -v <最新版本>
> dotnet add package Snet.Mqtt -v <最新版本>
> ```
> 只需安装驱动包 + MQ 包，其他库（Core/Model/Log/Utility）作为传递依赖自动引入。
>
> **📌 注意：** 控制台项目需要在 `Program.cs` 顶层使用 `await`，或包裹在 `static async Task Main()` 中。推荐使用 .NET 6.0+ 的顶级语句，直接在文件顶层使用 `await`。

```csharp
using System.Collections.Concurrent;
using Snet.Siemens;
using Snet.Mqtt.client;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;

// .NET 控制台顶级语句直接支持 await（无需 Main 方法）

#region 日志配置
LogHelper.Get().FolderName = "./logs";
LogHelper.Get().ConsoleOut = true;
#endregion

#region 1. 启动 MQTT 客户端（转发目标）
var mqConfig = new MqttClientData.Basics
{
    SN = "mqtt-target",              // 重要：设置 SN 用于 AddressMqParam.ISns 匹配
    IpAddress = "127.0.0.1",
    Port = 1883,                      // 库默认 6688，需改标准端口
    UserName = null,                   // 默认 "sample"，不需要认证时设 null
    Password = null,                   // 默认 "sample"，不需要认证时设 null
    ClientID = null,                   // 客户端ID，null 则自动生成随机
    MessageExpirationTime = 86400000,  // 消息过期时间(ms)，默认 24h
    QualityOfServiceLevel = MqttQualityOfServiceLevel.AtMostOnce, // QoS: AtMostOnce/AtLeastOnce/ExactlyOnce
};
using var mqClient = new MqttClientOperate(mqConfig);
var mqResult = await mqClient.OnAsync();
if (!mqResult.Status)
{
    LogHelper.Fatal($"MQTT 连接失败: {mqResult.Message}");
    return;
}
LogHelper.Info("MQTT 客户端已连接");
#endregion

#region 2. 配置 DAQ 采集协议
var daqConfig = new SiemensData.Basics
{
    IpAddress = "192.168.0.1",
    Port = 102,
    ProtocolType = SiemensData.ProtocolType.SiemensS7Net_S1500,
    Rack = 0,
    Slot = 1,
    ConnectTimeOut = 3000,
    ReceiveTimeOut = 3000,
};

using var siemens = new SiemensOperate(daqConfig);
var daqResult = await siemens.OnAsync();
if (!daqResult.Status)
{
    LogHelper.Fatal($"西门子连接失败: {daqResult.Message}");
    return;
}
LogHelper.Info("西门子 PLC 已连接");
#endregion

#region 3. 配置采集地址 + 数据转发
Address address = new Address
{
    SN = "工厂产线1",
    AddressArray = new List<AddressDetails>
    {
        new AddressDetails
        {
            SN = "工艺参数",
            AddressName = "DB1.0",
            AddressDataType = DataType.Int32,
            IsEnable = true,
            // ── 数据转发配置（关键！框架自动转发，无需手动编码）──
            AddressMqParam = new AddressMq
            {
                // ISns 格式: "{完整命名空间}.{类名}.{Basics.SN}"
                // 空则广播给所有已打开 MQ 实例
                ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.mqtt-target" },
                Topic = "factory/siemens",
                // ContentFormat: {0} = 点位值，可包含地址前缀
                ContentFormat = "DB1.0 = {0}"
            }
        },
        new AddressDetails
        {
            SN = "温度值",
            AddressName = "DB1.4",
            AddressDataType = DataType.Float,
            IsEnable = true,
            AddressMqParam = new AddressMq
            {
                ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.mqtt-target" },
                Topic = "factory/siemens",
                ContentFormat = "DB1.4 = {0}"
            }
        }
    }
};
#endregion

#region 4. 启动订阅（数据变化自动触发转发）
// 异步信息事件（状态变化/告警）
siemens.OnInfoEventAsync += async (sender, e) =>
{
    if (!e.Status)
        LogHelper.Warning($"事件告警: {e.Message}");
    await Task.CompletedTask;
};

// 异步数据事件（可选：即使有 AddressMqParam 自动转发，也可同时监听）
siemens.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    var data = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
    if (data != null)
    {
        foreach (var kv in data)
        {
            if (kv.Value.Quality == QualityType.Normal)
                LogHelper.Info($"📡 {kv.Key} = {kv.Value.ResultValue}");
        }
    }
    await Task.CompletedTask;
};

daqResult = await siemens.SubscribeAsync(address);
if (!daqResult.Status)
{
    LogHelper.Error($"订阅失败: {daqResult.Message}");
    return;
}
LogHelper.Info($"订阅已启动，数据自动转发到 MQTT topic: factory/siemens");
#endregion

#region 5. 保持运行
Console.WriteLine("采集+转发已启动，按任意键停止...");
Console.ReadKey();

await siemens.OffAsync();
await mqClient.OffAsync();
#endregion
```

### 2.2 场景模板：采集 + 多点转发

**用户输入：** "连接Modbus TCP 192.168.0.2:502，读取40001（Int16）、40002（Int16），转发到2个MQTT broker"

> **📌 项目创建：**
> ```bash
> dotnet new console -n DaqDemo && cd DaqDemo
> dotnet add package Snet.Modbus -v <最新版本>
> dotnet add package Snet.Mqtt -v <最新版本>
> ```

```csharp
using System.Collections.Concurrent;
using Snet.Modbus;
using Snet.Driver.Core;  // DataFormat 枚举（来自传递依赖，无需手动安装）
using Snet.Mqtt.client;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;

using var mq1 = new MqttClientOperate(new MqttClientData.Basics
{
    SN = "broker1", IpAddress = "192.168.0.10", Port = 1883,
});
await mq1.OnAsync();

using var mq2 = new MqttClientOperate(new MqttClientData.Basics
{
    SN = "broker2", IpAddress = "192.168.0.11", Port = 1883,
});
await mq2.OnAsync();

// Modbus 配置
using var modbus = new ModbusOperate(new ModbusData.Basics
{
    IpAddress = "192.168.0.2", Port = 502,
    ProtocolType = ModbusData.ProtocolType.ModbusTcpNet,
    Station = 1, DataFormat = DataFormat.CDAB,
});
await modbus.OnAsync();

// 地址 + 多点转发（AddressMqParam.ISns 指定多个目标）
Address address = new Address
{
    SN = "MultiForward",
    AddressArray = new List<AddressDetails>
    {
        new AddressDetails
        {
            SN = "reg1", AddressName = "40001", AddressDataType = DataType.Int16,
            IsEnable = true,
            AddressMqParam = new AddressMq
            {
                ISns = new List<string>
                {
                    "Snet.Mqtt.client.MqttClientOperate.broker1",
                    "Snet.Mqtt.client.MqttClientOperate.broker2",
                },
                Topic = "modbus/registers",
            }
        },
        new AddressDetails
        {
            SN = "reg2", AddressName = "40002", AddressDataType = DataType.Int16,
            IsEnable = true,
            AddressMqParam = new AddressMq
            {
                ISns = new List<string>
                {
                    "Snet.Mqtt.client.MqttClientOperate.broker1",
                    "Snet.Mqtt.client.MqttClientOperate.broker2",
                },
                Topic = "modbus/registers",
            }
        }
    }
};

await modbus.SubscribeAsync(address);
Console.WriteLine("多点转发已启动，按任意键停止...");
Console.ReadKey();
await modbus.OffAsync();
```

### 2.3 场景模板：采集 + 广播所有 MQ

**原理：** `ISns = null` 或空列表 = 转发到所有已打开的 MQ 实例

```csharp
AddressMqParam = new AddressMq
{
    // ISns 留空 → 广播到所有 MQ 实例
    ISns = null,
    Topic = "broadcast/all",
}
```

---

## 3. AddressMq 转发配置详解

### 3.1 AddressMq 属性

```csharp
public class AddressMq
{
    public string Topic { get; set; }           // MQTT/Kafka 主题（必填）
    public string? ContentFormat { get; set; }   // 内容格式化模板（{0}=点位值）
    public List<string>? ISns { get; set; }     // 目标 MQ 实例 SN 列表（null=广播所有）
}
```

### 3.2 ISns 格式

```
"{完整命名空间}.{Operate类名}.{Basics.SN}"
```

| MQ 类型 | ISns 格式示例 |
|---------|---------------|
| MQTT Client | `"Snet.Mqtt.client.MqttClientOperate.my-mqtt-1"` |
| Kafka | `"Snet.Kafka.KafkaOperate.my-kafka-1"` |
| RabbitMQ | `"Snet.RabbitMQ.RabbitMQOperate.my-rabbit-1"` |
| NetMQ | `"Snet.NetMQ.NetMQOperate.my-netmq-1"` |
| Netty | `"Snet.Netty.client.NettyClientOperate.my-netty-1"` |

### 3.3 ContentFormat 模板

| 模板 | 输出示例 |
|------|----------|
| `{0}`（默认） | `123` |
| `"值: {0}"` | `值: 123` |
| `"DB1.0 = {0}"` | `DB1.0 = 123` |
| `"{\"addr\":\"DB1.0\",\"val\":{0}}"` | `{"addr":"DB1.0","val":123}` |
| `null` 或空 | 直接发送原始值的 ToString() |

### 3.4 两种转发模式

| 模式 | ISns 值 | 效果 |
|------|---------|------|
| 指定转发 | `["Snet.Mqtt...broker1"]` | 只发给指定MQ实例 |
| 广播模式 | `null` 或 `[]` | 发给所有已打开的MQ实例 |

---

## 4. 数据模型 API 速查

### 4.1 OperateResult（所有操作返回值）

```csharp
// 同步调用
OperateResult result = operate.On();

// ✅ 推荐：异步调用
OperateResult result = await operate.OnAsync();

// 检查成功
bool ok = result.Status;         // true=成功, false=失败
string msg = result.Message;     // 状态描述
int ms = result.RunTime;         // 执行耗时(ms)

// 获取泛型结果数据
var data = result.GetSource<ConcurrentDictionary<string, AddressValue>>();

// 获取详情
if (result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var dict))
{
    // dict 包含所有点位数据
}

// 所有 Async 方法都接受 CancellationToken
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
OperateResult result = await operate.ReadAsync(address, cts.Token);
```

### 4.2 AddressValue（单个点位的值）

```csharp
AddressValue av = data["DB1.0"];

object? value = av.ResultValue;   // 解析后的值
object? raw = av.OriginalValue;   // 原始字节
QualityType q = av.Quality;       // 数据质量
string? msg = av.Message;         // 详细信息
DateTime t = av.Time;             // 采集时间

// 类型转换
int i = Convert.ToInt32(av.ResultValue);
float f = Convert.ToSingle(av.ResultValue);
bool b = Convert.ToBoolean(av.ResultValue);
string s = av.ResultValue?.ToString();

// 精简传输模型
AddressValueSimplify sim = av.GetSimplify();
// sim.Add=地址, sim.VAL=值(JSON), sim.Q=质量, sim.Ts=时间戳Ticks
```

### 4.3 QualityType（数据质量）

| 值 | 枚举 | 含义 | 是否可用 |
|----|------|------|----------|
| -1 | `None` | 尚未处理 | ❌ |
| 0 | `Exception` | 异常 | ❌ |
| 1 | `Normal` | **正常**（自动触发 MQ 转发） | ✅ |
| 2 | `DataTypeError` | 数据类型错误 | ❌ |
| 3 | `ParseUnknown` | 解析成功但正确性未知（也会触发 MQ 转发） | ⚠️ |
| 4 | `ParseError` | 解析错误 | ❌ |

### 4.4 AddressDetails（地址配置）

```csharp
new AddressDetails
{
    SN = "唯一标识",                // 机台号/组名/车间/厂
    AddressName = "DB1.0",           // PLC 地址（必填！）
    AddressDataType = DataType.Int32,// 数据类型
    AddressType = AddressType.Reality, // Reality=实际, VirtualStatic/DynamicRandom...
    IsEnable = true,                 // false=跳过采集
    Length = 1,                      // 数组长度
    EncodingType = EncodingType.ANSI, // 编码（String时使用）
    AddressDescribe = "描述说明",    // 描述
    AddressAnotherName = "别名",     // 别名

    // 消息中间件转发（订阅时自动转发）
    AddressMqParam = new AddressMq { ... },

    // 反射解析（对采集值进行二次加工）
    AddressParseParam = new AddressParse { ... },
}
```

### 4.5 DataType（数据类型）

| 基本类型 | 数组类型 | C# 类型 | 字节 |
|----------|----------|---------|------|
| `Bool` | `BoolArray` | `bool` | 1 |
| `Int16`/`Short` | `Int16Array`/`ShortArray` | `short` | 2 |
| `UInt16`/`Ushort` | `UInt16Array`/`UshortArray` | `ushort` | 2 |
| `Int32`/`Int` | `Int32Array`/`IntArray` | `int` | 4 |
| `UInt32`/`Uint` | `UInt32Array`/`UintArray` | `uint` | 4 |
| `Int64`/`Long` | `Int64Array`/`LongArray` | `long` | 8 |
| `UInt64`/`Ulong` | `UInt64Array`/`UlongArray` | `ulong` | 8 |
| `Float`/`Single` | `FloatArray`/`SingleArray` | `float` | 4 |
| `Double` | `DoubleArray` | `double` | 8 |
| `String` | — | `string` | n×2 |
| `ByteArray` | — | `byte[]` | n |
| `Char` | — | `char` | 2 |
| `Date` / `Time` / `DateTime` | — | `DateTime` | — |
| `None` | — | `object`（原始值直通） | — |

### 4.6 DataFormat（字节序）

```csharp
DataFormat.CDAB  // 默认，最常见
DataFormat.ABCD  // 大端序
DataFormat.BADC  // 字节交换
DataFormat.DCBA  // 完全反转
```

---

## 5. 协议类型自然语言映射

### 5.1 映射规则

当用户描述 PLC 型号时，按以下规则自动映射 ProtocolType：

| 用户描述关键词 | 使用的 Operate 类 | ProtocolType |
|----------------|-------------------|--------------|
| "西门子" "S7-200" "200" | `SiemensOperate` | `SiemensS7Net_S200` |
| "西门子" "S7-200Smart" "200Smart" | `SiemensOperate` | `SiemensS7Net_S200Smart` |
| "西门子" "S7-300" "300" | `SiemensOperate` | `SiemensS7Net_S300` |
| "西门子" "S7-400" "400" | `SiemensOperate` | `SiemensS7Net_S400` |
| "西门子" "S7-1200" "1200" | `SiemensOperate` | `SiemensS7Net_S1200` |
| "西门子" "S7-1500" "1500" | `SiemensOperate` | `SiemensS7Net_S1500` |
| "西门子" "PPI" "串口" | `SiemensOperate` | `SiemensPPI` |
| "西门子" "PPI" "TCP" "PPI以太网" | `SiemensOperate` | `SiemensPPIOverTcp` |
| "西门子" "S7Plus" "1500R" | `SiemensOperate` | `SiemensS7Plus` |
| "西门子" "FetchWrite" | `SiemensOperate` | `SiemensFetchWriteNet` |
| "Modbus" "TCP" | `ModbusOperate` | `ModbusTcpNet` |
| "Modbus" "RTU" "串口" | `ModbusOperate` | `ModbusRtu` |
| "Modbus" "ASCII" | `ModbusOperate` | `ModbusAscii` |
| "Modbus" "UDP" | `ModbusOperate` | `ModbusUdpNet` |
| "Modbus" "RTU" "TCP" "RTU over TCP" | `ModbusOperate` | `ModbusRtuOverTcp` |
| "Modbus" "ASCII" "TCP" "ASCII over TCP" | `ModbusOperate` | `ModbusAsciiOverTcp` |
| "三菱" "MC" "Q系列" | `MitsubishiOperate` | `MelsecMcNet` |
| "三菱" "MC" "UDP" | `MitsubishiOperate` | `MelsecMcUdp` |
| "三菱" "MC" "ASCII" | `MitsubishiOperate` | `MelsecMcAsciiNet` |
| "三菱" "MC" "ASCII" "UDP" | `MitsubishiOperate` | `MelsecMcAsciiUdp` |
| "三菱" "R系列" "MC R" | `MitsubishiOperate` | `MelsecMcRNet` |
| "三菱" "FX" "串口" | `MitsubishiOperate` | `MelsecFxSerial` |
| "三菱" "FX" "串口" "TCP" | `MitsubishiOperate` | `MelsecFxSerialOverTcp` |
| "三菱" "Links" "计算机链接" | `MitsubishiOperate` | `MelsecFxLinks` |
| "三菱" "Links" "TCP" | `MitsubishiOperate` | `MelsecFxLinksOverTcp` |
| "三菱" "A1E" | `MitsubishiOperate` | `MelsecA1ENet` |
| "三菱" "A1E" "ASCII" | `MitsubishiOperate` | `MelsecA1EAsciiNet` |
| "三菱" "A3C" | `MitsubishiOperate` | `MelsecA3CNet` |
| "三菱" "A3C" "TCP" | `MitsubishiOperate` | `MelsecA3CNetOverTcp` |
| "三菱" "CIP" | `MitsubishiOperate` | `MelsecCipNet` |
| "欧姆龙" "Fins" "TCP" | `OmronOperate` | `OmronFinsNet` |
| "欧姆龙" "Fins" "UDP" | `OmronOperate` | `OmronFinsUdp` |
| "欧姆龙" "CIP" "NJ" "NX" "NY" | `OmronOperate` | `OmronCipNet` |
| "欧姆龙" "Connected CIP" | `OmronOperate` | `OmronConnectedCipNet` |
| "欧姆龙" "HostLink" | `OmronOperate` | `OmronHostLink` |
| "欧姆龙" "HostLink" "TCP" | `OmronOperate` | `OmronHostLinkOverTcp` |
| "欧姆龙" "HostLink" "CMode" | `OmronOperate` | `OmronHostLinkCMode` |
| "欧姆龙" "HostLink" "CMode" "TCP" | `OmronOperate` | `OmronHostLinkCModeOverTcp` |
| "东方马达" "OrientalMotor" "步进" "EIP" | `OrientalMotorOperate` | `OrientalMotorEipNet` |
| "汇川" "AM" "AC" "H3U" "H5U" | `InovanceOperate` | `InovanceTcpNet` |
| "汇川" "Easy" | `InovanceOperate` | `InovanceEasyNet` |
| "汇川" "串口" | `InovanceOperate` | `InovanceSerial` |
| "汇川" "串口" "TCP" | `InovanceOperate` | `InovanceSerialOverTcp` |
| "汇川" "ComputerLink" | `InovanceOperate` | `InovanceComputerLink` |
| "汇川" "Connected CIP" | `InovanceOperate` | `InovanceConnectedCipNet` |
| "OPC UA" "OPCUA" | `OpcUaClientOperate` | (无需 ProtocolType) |
| "罗克韦尔" "AB" "Allen Bradley" "CIP" | `AllenBradleyOperate` | `AllenBradleyNet` |
| "罗克韦尔" "Connected CIP" | `AllenBradleyOperate` | `AllenBradleyConnectedCipNet` |
| "罗克韦尔" "Micro800" | `AllenBradleyOperate` | `AllenBradleyMicroCip` |
| "罗克韦尔" "PCCC" "SLC" | `AllenBradleyOperate` | `AllenBradleyPcccNet` |
| "罗克韦尔" "SLC500" | `AllenBradleyOperate` | `AllenBradleySLCNet` |
| "罗克韦尔" "DF1" "串口" | `AllenBradleyOperate` | `AllenBradleyDF1Serial` |
| "台达" "Delta" | `DeltaOperate` | `DeltaTcpNet` |
| "台达" "Delta" "串口" | `DeltaOperate` | `DeltaSerial` |
| "台达" "Delta" "串口" "TCP" | `DeltaOperate` | `DeltaSerialOverTcp` |
| "台达" "Delta" "ASCII" | `DeltaOperate` | `DeltaSerialAscii` |
| "台达" "Delta" "ASCII" "TCP" | `DeltaOperate` | `DeltaSerialAsciiOverTcp` |
| "基恩士" "Keyence" "MC" | `KeyenceOperate` | `KeyenceMcNet` |
| "基恩士" "Keyence" "MC" "ASCII" | `KeyenceOperate` | `KeyenceMcAsciiNet` |
| "基恩士" "Keyence" "Nano" "串口" | `KeyenceOperate` | `KeyenceNanoSerial` |
| "基恩士" "Keyence" "Nano" "TCP" | `KeyenceOperate` | `KeyenceNanoSerialOverTcp` |
| "基恩士" "Keyence" "KV" "旧版" | `KeyenceOperate` | `KeyenceKvOld` |
| "科伺" "Kossi" | `KossiOperate` | `OmronCipNet`（Kossi PLC 使用欧姆龙 CIP 协议） |
| "松下" "Panasonic" "MC" | `PanasonicOperate` | `PanasonicMcNet` |
| "松下" "Panasonic" "Mewtocol" | `PanasonicOperate` | `PanasonicMewtocol` |
| "松下" "Panasonic" "Mewtocol" "TCP" | `PanasonicOperate` | `PanasonicMewtocolOverTcp` |
| "麦格米特" "MegMeet" | `MegMeetOperate` | `MegMeetTcpNet` |
| "麦格米特" "MegMeet" "串口" | `MegMeetOperate` | `MegMeetSerial` |
| "麦格米特" "MegMeet" "串口" "TCP" | `MegMeetOperate` | `MegMeetSerialOverTcp` |
| "英威腾" "Invt" | `InvtOperate` | `ModbusTcpNet` |
| "英威腾" "Invt" "串口" | `InvtOperate` | `ModbusRtu` |
| "倍福" "Beckhoff" "ADS" | `BeckhoffOperate` | `BeckhoffAdsNet` |
| "通用电气" "GE" "SRTP" | `GEOperate` | `GeSRTPNet` |
| "安川" "Yaskawa" "Memobus" | `YaskawaOperate` | `MemobusTcpNet` |
| "安川" "Yaskawa" "Memobus" "UDP" | `YaskawaOperate` | `MemobusUdpNet` |
| "数据库" "DB" "SqlServer" "MySQL" "Oracle" "SQLite" | `DBOperate` | (DBType 枚举: SqlServer/MySql/Oracle/SQLite) |
| "TEP" "TCP扩展" "非标设备" "自定义协议" | `TepMasterOperate` | (无需 ProtocolType) |
| "Cimon" "西蒙" | `CimonOperate` | `CimonHmiProtocol` |
| "发那科" "Fanuc" "CNC" | `FanucOperate` | `FanucInterfaceNet` |
| "永宏" "Fatek" | `FatekOperate` | `FatekProgram` / `FatekProgramOverTcp` |
| "富士" "Fuji" "SPH" "SPB" | `FujiOperate` | `FujiSPHNet` / `FujiSPB` / `FujiSPBOverTcp` / `FujiCommandSettingType` |
| "LS产电" "LSis" "LG" | `LSisOperate` | `LSFastEnet` / `LSCnet` / `LSCpu` / `LSCnetOverTcp` |
| "RKC" "理化" | `RKCOperate` | `TemperatureControllerOverTcp` / `TemperatureController` |
| "丰田" "Toyota" | `ToyotaOperate` | `ToyoPuc` |
| "图尔克" "Turck" "IO-Link" | `TurckOperate` | `ReaderNet` |
| "丰炜" "Vigor" | `VigorOperate` | `VigorSerialOverTcp` / `VigorSerial` |
| "维控" "WeCon" | `WeConOperate` | `ModbusTcpNet` / `ModbusRtu` |
| "信捷" "XinJE" | `XinJEOperate` | `XinJETcpNet` / `XinJESerial` / `XinJESerialOverTcp` / `XinJEInternalNet` |
| "山武" "Yamatake" "AZBIL" | `YamatakeOperate` | `DigitronCPLOverTcp` / `DigitronCPL` |
| "横河" "Yokogawa" | `YokogawaOperate` | `YokogawaLinkTcp` |
| "宇电" "YuDian" "AIBus" "温控" | `YuDianOperate` | `YuDianAIBus` |
| "电力" "电表" "DLT" "PQDIF" "698" "645" "CJT188" "DTSU6606" "国网" | `PQDIFOperate` | `DLT698TcpNet` / `DLT645` / `DLT645With1997` / `CJT188` / `DTSU6606Serial` 等（NuGet: `Snet.PQDIF`） |
| "自由协议" "Freedom" "自定义报文" "raw" | `FreedomOperate` | `FreedomTcpNet` / `FreedomUdpNet` / `FreedomSerial` |
| "模拟" "Sim" "测试" | `SimOperate` | (无需 ProtocolType) |

### 5.2 端口默认值

**多数协议库默认端口为 6688（框架设计偏好），务必根据实际设备显式指定标准端口：**

| 协议 | 标准端口 | 库默认 |
|------|----------|--------|
| 西门子 S7 | 102 | 6688 |
| Modbus TCP | 502 | 6688 |
| OPC UA | 4840 | 6688 |
| 三菱 MC | 自适应 | 6688 |
| 欧姆龙 Fins | 9600 | 6688 |
| AB CIP | 44818 | 6688 |
| MQTT | 1883 | 6688 |

### 5.3 各协议 Basics 核心属性速查

> **完整属性列表请参考源码 `XxxData.cs`，以下仅列出关键配置项。**

| 协议 | 关键属性（除公共属性外） | 源码文件 |
|------|------------------------|----------|
| **西门子** | `Rack`, `Slot`, `PDULength`, `LocalTSAP` | `SiemensData.cs` |
| **Modbus** | `Station`, `DataFormat`, `AddressStartWithZero`, `IsCheckMessageId` | `ModbusData.cs` |
| **三菱** | `Slot`, `NetworkNumber`, `NetworkStationNumber`, `EnableWriteBitToWordRegister` | `MitsubishiData.cs` |
| **欧姆龙** | `OmronPlcType`, `SA1`, `DA1`, `DA2`, `GCT`, `DataFormat` | `OmronData.cs` |
| **东方马达** | `RunIdleHeader`, `RPITime`, `ActualTimeout`（EIP 协议） | `OrientalMotorData.cs` |
| **汇川** | `Station`, `InovanceSeries`(AM/AC/H3U/H5U), `DataFormat`, `AddressStartWithZero` | `InovanceData.cs` |
| **OPC UA** | `ServerUrl`(代替Ip+Port), `AType`(认证方式), `SamplingInterval`, `PublishingInterval` | `OpcUaClientData.cs` |
| **罗克韦尔** | `Slot`, `ReadArrayUseSegment`, `ContextCheck`, `DstNode` | `AllenBradleyData.cs` |
| **台达** | `Station`, `DeltaSeries` | `DeltaData.cs` |
| **基恩士** | `UseStation`, `Station`, `EnableWriteBitToWordRegister` | `KeyenceData.cs` |
| **科伺（Kossi）** | `Slot`（Kossi PLC 使用欧姆龙 CIP 协议） | `KossiData.cs` |
| **松下** | `Station` | `PanasonicData.cs` |
| **麦格米特** | `Station`, `DataFormat`, `AddressStartWithZero` | `MegMeetData.cs` |
| **英威腾** | `Station`, `DataFormat`, `StationCheckMatch`, `AddressStartWithZero` | `InvtData.cs` |
| **倍福** | `SenderAMSNetId`, `TargetAMSNetId`, `UseAutoAmsNetID`, `UseTagCache` | `BeckhoffData.cs` |
| **安川** | `CpuFrom`, `CpuTo`, `DataFormat` | `YaskawaData.cs` |
| **西蒙** | `FrameNo` | `CimonData.cs` |
| **发那科** | `StringEncoding` | `FanucData.cs` |
| **永宏** | `Station`, `DataFormat` | `FatekData.cs` |
| **富士** | `ConnectionID`, `DataFormat`, `DataSwap`, `Station` | `FujiData.cs` |
| **LS产电** | `SlotNo`, `CpuType`, `CompanyID`, `Station` | `LSisData.cs` |
| **理化** | `Station` | `RKCData.cs` |
| **丰田** | 无特殊属性 | `ToyotaData.cs` |
| **图尔克** | 无特殊属性 | `TurckData.cs` |
| **丰炜** | `Station` | `VigorData.cs` |
| **维控** | `Station`, `DataFormat`, `AddressStartWithZero`, `IsCheckMessageId` | `WeConData.cs` |
| **信捷** | `Station`, `XinJESeries`, `DataFormat`, `AddressStartWithZero` | `XinJEData.cs` |
| **山武** | `Station` | `YamatakeData.cs` |
| **横河** | `CpuNumber` | `YokogawaData.cs` |
| **宇电** | `Station`, `SerialPortInfo`（AIBus 串口协议） | `YuDianData.cs` |
| **PQDIF（电力）** | `StationStr`, `EnableCodeFE`, `UseSecurityResquest`, `CA`, `Password`, `OpCode`, `CheckDataId`, `InstrumentType`, `AddressStartWithZero`, `IsStringReverse`, `Crc16CheckEnable` | `PQDIFData.cs` |
| **自由协议** | `DataFormat`, `IsStringReverseByteWord` | `FreedomData.cs` |

**公共属性（所有协议共有）：** `IpAddress`, `Port`, `ConnectTimeOut`, `ReceiveTimeOut`, `SleepTime`, `SocketKeepAliveTime`, `IsPersistentConnection`, `SerialPortInfo`, `RtsEnable`, `DtrEnable`, `ProtocolType`

---

## 6. 地址格式速查

> **地址名称 `AddressName` 就是表格中的示例值，AI 根据用户描述自动填写。**

| PLC/协议 | 位地址 | 字地址 | 示例 AddressName | 数据类型 |
|----------|--------|--------|-----------------|----------|
| 西门子 S7 | `M0.0`, `I0.0`, `Q0.0` | `DB1.0`, `MW0`, `IW0` | `DB1.0` | Int32/Float/Bool |
| Modbus | `00001`(线圈) `10001`(输入) | `40001`(保持) `30001`(输入) | `40001` | Int16/Int32/Float |
| 三菱 MC | `M0`, `X0`, `Y0` | `D100`, `W100`, `R100` | `D100` | Int16/Int32/Float |
| 三菱 FX | `M0`, `X0`, `Y0` | `D100` | `D100` | Int16/Int32 |
| 欧姆龙 Fins | `CIO0.0`, `W0.0`, `H0.0` | `D100`, `A0` | `D100` | Int16/Int32/Float |
| 欧姆龙 CIP | Tag名 | `MyTag` | `Program:MainProgram.Var` | 任意 |
| 罗克韦尔 | Tag名 | `MyTag` | `Program:MainProgram.Var` | 任意 |
| 汇川 | `M0`, `X0`, `Y0` | `D100`, `SD100` | `D100` | Int16/Int32/Float |
| 台达 | `M0`, `X0`, `Y0` | `D100`, `S100` | `D100` | Int16/Int32 |
| 基恩士 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 松下 | `R0`, `X0`, `Y0` | `DT100`, `LD100` | `DT100` | Int16/Int32 |
| OPC UA | NodeId | `ns=0;i=2258` | `ns=2;s=MyVariable` | 任意 |
| 丰炜 | `M0`, `X0`, `Y0` | `D100` | `D100` | Int16/Int32 |
| 维控 | `M0`, `X0`, `Y0` | `D100` | `D100` | Int16/Int32/Float |
| 信捷 | `M0`, `X0`, `Y0` | `D100`, `SD100` | `D100` | Int16/Int32/Float |
| LS产电 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 安川 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 永宏 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 富士 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 山武 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 横河 | `M0`, `X0`, `Y0` | `D100`, `R100` | `D100` | Int16/Int32 |
| 理化温控器 | — | 寄存器地址 | `100` | Float |

---

## 7. 完整协议示例

### 7.1 西门子 PLC

```csharp
using Snet.Siemens;

var config = new SiemensData.Basics
{
    IpAddress = "192.168.0.1",
    Port = 102,
    ProtocolType = SiemensData.ProtocolType.SiemensS7Net_S1500,
    Rack = 0,
    Slot = 1,
    ConnectTimeOut = 3000,
    ReceiveTimeOut = 3000,
};
using var operate = new SiemensOperate(config);
await operate.OnAsync();
// 地址: DB1.0, M0.0, I0.0, Q0.0, MW0, IW0 等
// 读取: await operate.ReadAsync(address);
// 写入: await operate.WriteAsync(values);
await operate.OffAsync();
```

### 7.2 Modbus

```csharp
using Snet.Modbus;
using Snet.Driver.Core; // DataFormat

var config = new ModbusData.Basics
{
    IpAddress = "192.168.1.100",
    Port = 502,
    ProtocolType = ModbusData.ProtocolType.ModbusTcpNet,
    Station = 1,
    AddressStartWithZero = true,
    DataFormat = DataFormat.CDAB,
};
// ModbusRtu 用 SerialPortInfo 替代 IpAddress/Port
// ModbusRtuOverTcp 用 IpAddress/Port
// ModbusUdpNet 用 IpAddress/Port
using var operate = new ModbusOperate(config);
await operate.OnAsync();
// 读取: await operate.ReadAsync(address);
```

### 7.3 OPC UA / OPC DA

> NuGet: `dotnet add package Snet.Opc -v <最新版本>`
> OPC 项目提供 4 种操作类：UA 客户端、UA 服务端、DA 客户端（COM/DCOM）、DA HTTP 转发

**OPC UA Client（最常用）：**

```csharp
using Snet.Opc.ua.client;

var config = new OpcUaClientData.Basics
{
    ServerUrl = "opc.tcp://192.168.0.1:4840",
    AType = Snet.Opc.core.Data.AuType.UserName,
    UserName = "user",
    Password = "password",
};
using var operate = new OpcUaClientOperate(config);
await operate.OnAsync();
// 地址: ns=0;i=2258 或 ns=2;s=MyVariable
// 读取: await operate.ReadAsync(address);
await operate.OffAsync();
```

**OPC UA Server（提供 OPC UA 服务端）：**

```csharp
using Snet.Opc.ua.service;

using (OpcUaServiceOperate operate = new OpcUaServiceOperate(new OpcUaServiceData.Basics
{
    Port = 6688,
    AType = Snet.Opc.core.Data.AuType.UserName,
    UserName = "user",
    Password = "password",
    AutoCreateAddress = true,   // 自动创建地址空间
    AddressSpaceName = "Snet",  // 地址空间名称
}))
{
    await operate.OnAsync();
    // 客户端可连接 opc.tcp://127.0.0.1:6688/Opc.Ua.Service
}
```

**OPC DA Client（COM/DCOM，传统 OPC DA 服务器）：**

```csharp
using Snet.Opc.da.client;

using (OpcDaClientOperate operate = new OpcDaClientOperate(new OpcDaClientData.Basics
{
    ServerUrl = "opcda://192.168.0.1/OpcDaServer",
}))
{
    await operate.OnAsync();
}
```

**OPC DA HTTP（通过 HTTP 转发 OPC DA 操作）：**

```csharp
using Snet.Opc.da.http;

using (OpcDaHttpOperate operate = new OpcDaHttpOperate(new OpcDaHttpData.Basics
{
    ServerUrl = "http://192.168.0.1:8080/opcda",
}))
{
    await operate.OnAsync();
}
```

### 7.4 三菱 PLC

```csharp
using Snet.Mitsubishi;

var config = new MitsubishiData.Basics
{
    IpAddress = "192.168.1.100",
    ProtocolType = MitsubishiData.ProtocolType.MelsecMcNet,
    NetworkNumber = 0,
    NetworkStationNumber = 0,
    TargetIOStation = 1023,
};
using var operate = new MitsubishiOperate(config);
await operate.OnAsync();
// 读取: await operate.ReadAsync(address);
```

### 7.5 模拟库（无硬件测试）

```csharp
using Snet.Sim;

using var operate = new SimOperate(new SimData.Basics { SN = "test" });
await operate.OnAsync();
// 5 种虚拟地址: VirtualStatic, VirtualDynamic_Random, VirtualDynamic_RandomScope,
//               VirtualDynamic_Order, VirtualDynamic_OrderScope
Address address = new Address(new AddressDetails
{
    SN = "v1", AddressName = "SimVal{1000,1^100}", AddressDataType = DataType.Int32,
    AddressType = AddressType.VirtualDynamic_RandomScope,
    // 虚拟地址可在 AddressName 中嵌入参数：
    // {间隔} / {间隔,最小值^最大值} / {间隔,增长比例} / {间隔,增长比例,最小值^最大值}
});
var result = await operate.ReadAsync(address);
await operate.OffAsync();
```

### 7.6 数据库采集（Snet.DB）

> NuGet: `dotnet add package Snet.DB -v <最新版本>`
> Operate: `DBOperate` | Config: `DBData.Basics`
> 地址: SQL 查询语句 | DBType: SqlServer / MySql / Oracle / SQLite
> HandlerType: Daq（采集，用 Dapper）/ Default（增删改查，用 SqlSugar）

**场景：从数据库查询数据并转发到 MQTT**

```csharp
using Snet.DB;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;

using (DBOperate operate = new DBOperate(new DBData.Basics
{
    // 数据库类型
    DBType = DBData.DBType.SqlServer,  // 或 MySql / Oracle / SQLite
    // 连接字符串
    ConnectStr = "Server=192.168.0.100;Database=IoTData;User Id=sa;Password=xxx;",
    // 处理类型: Daq=数据采集, Default=增删改查
    HandlerType = DBData.DBHandlerType.Daq,
    // 订阅间隔（ms）
    HandleInterval = 5000,
}))
{
    OperateResult result = await operate.OnAsync();
    if (!result.Status)
    {
        LogHelper.Fatal($"数据库连接失败: {result.Message}");
        return;
    }
    LogHelper.Info("数据库已连接");

    // AddressName = SQL 查询语句
    Address address = new Address
    {
        SN = "数据库采集",
        AddressArray = new List<AddressDetails>
        {
            // 第一条 SQL 查询
            new AddressDetails
            {
                SN = "温度数据",
                AddressName = "SELECT TOP 1 Temperature, Humidity FROM SensorData WHERE DeviceId='D001' ORDER BY Timestamp DESC",
                AddressDataType = DataType.String,  // 查询结果以 JSON 返回
                IsEnable = true,
                // 转发配置
                AddressMqParam = new AddressMq
                {
                    ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.my-mqtt" },
                    Topic = "db/sensors",
                    ContentFormat = "{0}"  // JSON 格式的查询结果
                }
            },
            // 第二条 SQL 查询
            new AddressDetails
            {
                SN = "产量统计",
                AddressName = "SELECT SUM(Quantity) AS TotalOutput FROM ProductionLog WHERE Date = CAST(GETDATE() AS DATE)",
                AddressDataType = DataType.String,
                IsEnable = true,
                AddressMqParam = new AddressMq
                {
                    ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.my-mqtt" },
                    Topic = "db/production",
                }
            }
        }
    };

    // 读取（执行 SQL 查询）
    result = await operate.ReadAsync(address);
    if (result.Status && result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    {
        foreach (var kv in data)
        {
            Console.WriteLine($"{kv.Key}: {kv.Value.ResultValue}");
        }
    }

    // 订阅（定时执行 SQL + 自动转发）
    await operate.SubscribeAsync(address);
    Console.WriteLine("数据库定时采集已启动，按任意键停止...");
    Console.ReadKey();

    await operate.OffAsync();
}
```

**Default 模式（增删改查）：**

```csharp
using (DBOperate operate = new DBOperate(new DBData.Basics
{
    DBType = DBData.DBType.SqlServer,
    ConnectStr = "Server=192.168.0.100;Database=IoTData;User Id=sa;Password=xxx;",
    HandlerType = DBData.DBHandlerType.Default,  // ← 增删改查模式
}))
{
    await operate.OnAsync();

    // 查询（Dapper）
    var list = await operate.QueryAsync<SensorData>("SELECT * FROM SensorData WHERE DeviceId='D001'");

    // 增删改（Dapper）
    await operate.ExecuteAsync("INSERT INTO SensorData (DeviceId, Temperature) VALUES (@DId, @Temp)",
        new { DId = "D001", Temp = 25.6 });

    await operate.OffAsync();
}
```

**数据库连接字符串示例：**

| DBType | 连接字符串模板 |
|--------|----------------|
| `SqlServer` | `"Server=IP;Database=DBName;User Id=sa;Password=pwd;"` |
| `MySql` | `"Server=IP;Port=3306;Database=DBName;User=root;Password=pwd;"` |
| `Oracle` | `"Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=IP)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=ORCL)));User Id=user;Password=pwd;"` |
| `SQLite` | `"Data Source=/path/to/database.db"` |

### 7.7 TEP 非标设备采集（Snet.TEP）

> NuGet: `dotnet add package Snet.TEP -v <最新版本>`
> TEP = TCP 扩展插件，用于采集**非标准协议设备**（自定义 TCP 协议）
> 架构: Master（服务端，实现 IDaq）← TCP 私有协议 → Slave（客户端，设备端）

**TEP 协议流程：** 握手(0x30) → 身份认证(0x31) → 心跳(0x32) → 数据上传(0x34) / 数据写入(0x35)

**包格式：** `{0x7B,0x5B,0x28,0x3C} + 方向 + 命令 + 长度 + 数据 + CRC + {0x3E,0x29,0x5D,0x7D}`

#### 7.7.1 TEP Master（服务端 — 采集非标设备数据）

```csharp
using Snet.TEP.master;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;
using System.Collections.Concurrent;

// TepMasterOperate 实现了 IDaq，用法与西门子/Modbus 完全一致
using (TepMasterOperate operate = new TepMasterOperate(new TepMasterData.Basics
{
    IpAddress = "0.0.0.0",
    Port = 6688,
    MaxNumber = 1000,       // 最大客户端连接数
    UserName = "samples",   // 身份认证账号
    Password = "samples",   // 身份认证密码
}))
{
    OperateResult result = await operate.OnAsync();
    if (!result.Status)
    {
        LogHelper.Fatal($"TEP 服务端启动失败: {result.Message}");
        return;
    }
    LogHelper.Info("TEP 服务端已启动，等待非标设备连接...");

    // 地址格式: "{设备名称}.{设备ID}.{点位名称}"
    // 例如: "我的非标设备.10001.温度"
    Address address = new Address
    {
        SN = "非标设备采集",
        AddressArray = new List<AddressDetails>
        {
            new AddressDetails
            {
                SN = "param1",
                AddressName = "测试设备.10086.温度",  // DevName.DevID.PointName
                AddressDataType = DataType.Float,
                IsEnable = true,
                AddressMqParam = new AddressMq
                {
                    ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.my-mqtt" },
                    Topic = "tep/sensors",
                    ContentFormat = "温度 = {0}"
                }
            },
            new AddressDetails
            {
                SN = "param2",
                AddressName = "测试设备.10086.状态",
                AddressDataType = DataType.Bool,
                IsEnable = true,
            }
        }
    };

    // 读取所有已连接设备的数据
    result = await operate.ReadAsync(address);
    if (result.Status && result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    {
        foreach (var kv in data)
            Console.WriteLine($"{kv.Key} = {kv.Value.ResultValue}");
    }

    // 订阅（设备主动上传数据时自动接收 + 转发）
    operate.OnDataEventAsync += async (sender, e) =>
    {
        if (!e.Status) return;
        var d = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
        if (d != null)
        {
            foreach (var kv in d)
                Console.WriteLine($"📡 TEP: {kv.Key} = {kv.Value.ResultValue}");
        }
        await Task.CompletedTask;
    };
    await operate.SubscribeAsync(address);

    // 写入数据到设备
    var writeValues = new ConcurrentDictionary<string, (object value, EncodingType? encodingType)>
    {
        ["测试设备.10086.温度"] = (25.6f, null),
        ["测试设备.10086.状态"] = (true, null)
    };
    await operate.WriteAsync(writeValues);

    Console.WriteLine("按任意键停止...");
    Console.ReadKey();
    await operate.OffAsync();
}
```

#### 7.7.2 TEP Slave（客户端 — 设备端上报数据）

```csharp
using Snet.TEP.slave;
using Snet.TEP.core;
using Snet.Model.data;

// TepSlaveOperate 运行在非标设备上，连接 Master 并上报数据
TepSlaveOperate clientOperate = TepSlaveOperate.Instance(new TepSlaveData.Basics
{
    IpAddress = "127.0.0.1",   // Master IP
    Port = 6688,                // Master Port
    DevName = "测试设备",        // 设备名称（Master 用它定位设备）
    DevID = "10086",            // 设备 ID
    UserName = "samples",       // 认证账号（与 Master 一致）
    Password = "samples",       // 认证密码
    ViolenceUpload = false,     // false=等待响应, true=只上传不等结果
});

// 设置设备状态回调（Master 查询设备状态时触发）
bool GetDeviceStatus()
{
    return true;  // 设备正常
}

// 设置数据写入回调（Master 向设备写数据时触发）
OperateResult HandleWrite(List<CoreBasicsData.KeyValue> keyValues)
{
    foreach (var kv in keyValues)
        Console.WriteLine($"收到写入指令: {kv.Key} = {kv.Value}");
    return OperateResult.CreateSuccessResult("写入成功");
}

clientOperate.SetBaseStateFunc(GetDeviceStatus);
clientOperate.SetBaseWriteFunc(HandleWrite);

// 连接 Master
OperateResult result = await clientOperate.OnAsync();
if (!result.Status)
{
    Console.WriteLine($"连接 Master 失败: {result.Message}");
    return;
}
Console.WriteLine("已连接到 TEP Master");

// 上传数据到 Master
var data = new List<CoreBasicsData.KeyValue>
{
    new CoreBasicsData.KeyValue { Key = "温度", Value = 25.6 },
    new CoreBasicsData.KeyValue { Key = "湿度", Value = 60.2 },
    new CoreBasicsData.KeyValue { Key = "状态", Value = true },
};
result = await clientOperate.DataUploadAsync(data);
Console.WriteLine($"数据上传: {(result.Status ? "成功" : result.Message)}");

await clientOperate.OffAsync();
```

### 7.8 电力通讯规约 PQDIF（电表/电力采集）

> NuGet: `dotnet add package Snet.PQDIF -v <最新版本>`
> Operate: `PQDIFOperate` | Config: `PQDIFData.Basics`
> 统一电力协议库，通过 ProtocolType 覆盖 DLT645/DLT698/CJT188/DTSU6606 等 10 种电表协议

**协议子类型速查：**

| 用户描述 | ProtocolType | 通信方式 |
|----------|-------------|----------|
| "698.45" "面向对象" "DLT698 TCP" | `DLT698TcpNet` | TCP |
| "698.45" "DLT698 串口" | `DLT698` | 串口 |
| "698.45" "DLT698 透传" | `DLT698OverTcp` | 串口转网口透传 |
| "DLT645-2007" "645 串口" | `DLT645` | 串口 |
| "DLT645-2007" "645 TCP" | `DLT645OverTcp` | 串口转网口透传 |
| "DLT645-1997" "老版645 串口" | `DLT645With1997` | 串口 |
| "DLT645-1997" "老版645 TCP" | `DLT645With1997OverTcp` | 串口转网口透传 |
| "CJT188" "户表" "188协议 串口" | `CJT188` | 串口 |
| "CJT188" "户表" "188协议 TCP" | `CJT188OverTcp` | 串口转网口透传 |
| "DTSU6606" "德力西" "Modbus电表" | `DTSU6606Serial` | 串口(Modbus-RTU) |

**关键属性（除公共属性外）：** `StationStr`(站号，字符串), `EnableCodeFE`, `UseSecurityResquest`, `CA`(客户机地址), `Password`, `OpCode`, `CheckDataId`, `InstrumentType`, `AddressStartWithZero`, `IsStringReverse`

**示例：读取 DLT645-2007 电表数据并转发 MQTT**

```csharp
using Snet.PQDIF;
using Snet.Mqtt.client;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using System.Collections.Concurrent;
using static Snet.PQDIF.PQDIFData;

// MQTT 客户端
var mq = new MqttClientOperate(new MqttClientData.Basics
{
    SN = "power-mqtt", IpAddress = "127.0.0.1", Port = 1883,
});
await mq.OnAsync();

// PQDIF 电表 — DLT645-2007 TCP
using var meter = new PQDIFOperate(new Basics
{
    IpAddress = "192.168.0.100",
    Port = 2404,                       // DLT645 常用端口
    ProtocolType = ProtocolType.DLT645OverTcp,
    StationStr = "123456789012",       // 12位表号
    Password = "00000000",
    OpCode = "00000000",
    CheckDataId = true,
});
await meter.OnAsync();

Address address = new Address
{
    SN = "电表1",
    AddressArray = new List<AddressDetails>
    {
        new AddressDetails
        {
            SN = "电压", AddressName = "02010100",     // A相电压 数据标识
            AddressDataType = DataType.Float, IsEnable = true,
            AddressMqParam = new AddressMq
            {
                ISns = new List<string> { "Snet.Mqtt.client.MqttClientOperate.power-mqtt" },
                Topic = "meter/voltage",
                ContentFormat = "电压 = {0} V"
            }
        },
        new AddressDetails
        {
            SN = "电流", AddressName = "02020100",     // A相电流
            AddressDataType = DataType.Float, IsEnable = true,
        },
        new AddressDetails
        {
            SN = "有功功率", AddressName = "02030100", // 总有功功率
            AddressDataType = DataType.Float, IsEnable = true,
        },
    }
};

// 读取
var result = await meter.ReadAsync(address);
if (result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    foreach (var kv in data)
        Console.WriteLine($"{kv.Key} = {kv.Value.ResultValue}");

// 订阅（定时读取 + 自动MQTT转发）
await meter.SubscribeAsync(address);
Console.WriteLine("电表采集已启动，按任意键停止...");
Console.ReadKey();
await meter.OffAsync(); await mq.OffAsync();
```

> **地址格式说明：** DLT645 使用 4 字节数据标识（DI0-DI3），格式为 `DDDDDDDD`（8 位十六进制）。例如 `02010100` = A相电压。DLT698 使用 OAD（对象属性描述符）字符串格式。

### 7.9 科伺 Kossi PLC

> NuGet: `dotnet add package Snet.Kossi -v <最新版本>`
> Operate: `KossiOperate` | Config: `KossiData.Basics`
> 科伺 PLC 使用欧姆龙 CIP 协议通信

```csharp
using Snet.Kossi;

var config = new KossiData.Basics
{
    IpAddress = "192.168.1.100",
    ProtocolType = KossiData.ProtocolType.OmronCipNet,
    Slot = 0,
    ConnectTimeOut = 3000,
    ReceiveTimeOut = 3000,
};
using var operate = new KossiOperate(config);
await operate.OnAsync();
// 地址: 参考欧姆龙 CIP Tag 格式
```

### 7.10 东方马达 OrientalMotor

> NuGet: `dotnet add package Snet.OrientalMotor -v <最新版本>`
> Operate: `OrientalMotorOperate` | Config: `OrientalMotorData.Basics`
> 东方马达步进驱动器，使用 EIP（EtherNet/IP）协议

```csharp
using Snet.OrientalMotor;

var config = new OrientalMotorData.Basics
{
    IpAddress = "192.168.1.100",
    ProtocolType = OrientalMotorData.ProtocolType.OrientalMotorEipNet,
    RunIdleHeader = 1,         // RunIdle 头
    RPITime = 100,             // RPI 时间(ms)
    ActualTimeout = 2,         // EIP 超时(秒)，必须为 2 的幂
    ConnectTimeOut = 3000,
    ReceiveTimeOut = 3000,
};
using var operate = new OrientalMotorOperate(config);
await operate.OnAsync();
// 读取: await operate.ReadAsync(address);
```

### 7.11 宇电 YuDian AIBus

> NuGet: `dotnet add package Snet.YuDian -v <最新版本>`
> Operate: `YuDianOperate` | Config: `YuDianData.Basics`
> 宇电 AIBus 温控器，基于串口通信（RS-485）

```csharp
using Snet.YuDian;

var config = new YuDianData.Basics
{
    SerialPortInfo = "COM3-9600-8-N-1",
    ProtocolType = YuDianData.ProtocolType.YuDianAIBus,
    Station = 1,
    ReceiveTimeOut = 1000,
};
using var operate = new YuDianOperate(config);
await operate.OnAsync();
// 读取: await operate.ReadAsync(address);
```

### 7.12 Snet.Driver 内置协议（无需单独 NuGet 包）

> 以下协议已在 `Snet.Driver` 中实现底层驱动，但暂无独立的 NuGet 包和 Operate 封装类。
> 如需使用，可通过 `Snet.Freedom`（自由协议）间接调用，或参考 [PluginDev-Skill](../PluginDev-Skill) 自行封装 Operate。
> 这些协议由 `Snet.Driver` 包作为传递依赖自动引入。

| 协议 | 驱动目录 | 说明 | 通信方式 |
|------|---------|------|----------|
| **Knx** | `Snet.Driver.Profinet.Knx` | KNXnet/IP 楼宇自动化协议（ISO/IEC 14543） | UDP |
| **OpenProtocol** | `Snet.Driver.Profinet.OpenProtocol` | Atlas Copco 拧紧枪协议（MID 指令） | TCP (默认 4545) |
| **Sick ICR** | `Snet.Driver.Profinet.Sick` | SICK/海康/Keyence/Datalogic 条码扫描器 | TCP Server |
| **Geniitek** | `Snet.Driver.Profinet.Geniitek` | 捷杰 VB31 无线振动传感器 | TCP (默认 3001) |
| **IDCard** | `Snet.Driver.Profinet.IDCard` | SAM 身份证读卡器 | 串口/TCP |
| **Toledo** | `Snet.Driver.Profinet.Toledo` | 梅特勒-托利多称重仪表 | TCP |
| **IEC 60870-5-104** | `Snet.Driver.Instrument.IEC` | IEC 104 电力远动协议 | TCP (默认 2404) |
| **ShineIn Light** | `Snet.Driver.Instrument.Light` | 昱行智造光源控制器（私有协议） | 串口 (57600-8-E-1) |
| **DAM3601** | `Snet.Driver.Instrument.Temperature` | 阿尔泰科技 DAM3601 温控模块（Modbus RTU 变体） | 串口 |

---

## 8. 消息中间件

### 8.1 MQTT Client / Service / WebSocket

**MQTT Client（最常用 — 连接外部 Broker）：**

```csharp
using Snet.Mqtt.client;

var config = new MqttClientData.Basics
{
    SN = "my-mqtt",              // 重要：用于 ISns 匹配
    IpAddress = "127.0.0.1",     // ← 属性名是 IpAddress，不是 Ip！
    Port = 1883,                  // 库默认 6688，需改标准端口
    UserName = "admin",           // 默认 "sample"
    Password = "password",        // 默认 "sample"
    ClientID = null,              // 客户端ID，null 则自动生成随机
    MessageExpirationTime = 86400000,  // 消息过期时间(ms)，默认 24h
    QualityOfServiceLevel = MqttQualityOfServiceLevel.AtMostOnce, // QoS: AtMostOnce/AtLeastOnce/ExactlyOnce
    ResponseType = ResponseType.Content,  // Content/Bytes/ContentWithTopic
};
using var mq = new MqttClientOperate(config);
await mq.OnAsync();

// 生产
await mq.ProduceAsync("topic/hello", "hello world");

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

**MQTT Service（内置 Broker — 无需外部 MQTT 服务器）：**

```csharp
using Snet.Mqtt.service;

var config = new MqttServiceData.Basics
{
    Port = 6688,
    MaxNumber = 10000,    // 最大客户端连接数
};
using (MqttServiceOperate mqttService = new MqttServiceOperate(config))
{
    await mqttService.OnAsync();
    // 内置 MQTT Broker 已启动，客户端可连接 mqtt://127.0.0.1:6688
}
```

**MQTT WebSocket Service（支持浏览器 WebSocket 连接）：**

```csharp
using Snet.Mqtt.service.websocket;

var config = new MqttWebSocketServiceData.Basics
{
    Port = 6688,          // MQTT 端口
    WsPort = 8866,        // WebSocket 端口
};
using (MqttWebSocketServiceOperate wsService = new MqttWebSocketServiceOperate(config))
{
    await wsService.OnAsync();
    // 浏览器可通过 ws://127.0.0.1:8866 连接
}
```

### 8.2 Kafka / RabbitMQ / NetMQ / Netty

```
// Kafka: dotnet add package Snet.Kafka -v <最新版本>
//   Operate: KafkaOperate, Config: KafkaData.Basics
//   ISns: "Snet.Kafka.KafkaOperate.my-kafka"
//   注意: Kafka 用 BootstrapServers（非 IpAddress+Port），支持 SASL 认证
//
// RabbitMQ: dotnet add package Snet.RabbitMQ -v <最新版本>
//   Operate: RabbitMQOperate, Config: RabbitMQData.Basics
//   ISns: "Snet.RabbitMQ.RabbitMQOperate.my-rabbit"
//   注意: RabbitMQ 有 ExChangeName（交换机名称）属性
//
// NetMQ: dotnet add package Snet.NetMQ -v <最新版本>
//   Operate: NetMQOperate, Config: NetMQData.Basics
//   ISns: "Snet.NetMQ.NetMQOperate.my-netmq"
//   注意: NetMQ 用 Address（如 "tcp://127.0.0.1:8866"），有 UModel（PubModel/SubModel）
//
// Netty: dotnet add package Snet.Netty -v <最新版本>
//   Operate: NettyClientOperate, Config: NettyClientData.Basics
//   ISns: "Snet.Netty.client.NettyClientOperate.my-netty"
//   注意: Netty 支持 SSL（SslFilePath/SslFilePassword）
```

**各 MQ 关键属性速查：**

| MQ 类型 | 连接参数 | 特殊属性 |
|---------|---------|---------|
| **MQTT** | `IpAddress` + `Port`(1883) | `ClientID`, `QualityOfServiceLevel`, `MessageExpirationTime` |
| **Kafka** | `BootstrapServers`（如 `"192.168.0.1:9092"`） | `SecurityProtocol`, `SaslMechanism`, `AutoOffsetReset` |
| **RabbitMQ** | `IpAddress` + `Port`(5672) | `ExChangeName`, `MessageExpirationTime` |
| **NetMQ** | `Address`（如 `"tcp://127.0.0.1:8866"`） | `UModel`(PubModel/SubModel), `TimeOut` |
| **Netty** | `IpAddress` + `Port` | `SslFilePath`, `SslFilePassword`, `TaskNumber` |

---

## 9. 日志配置

```csharp
// 程序启动时一次性配置
LogHelper.Get().FolderName = "./logs";    // 日志文件目录
LogHelper.Get().ConsoleOut = true;        // 控制台输出（默认 false）
LogHelper.Get().FileOut = true;           // 文件输出

// 日志级别
LogHelper.Verbose("详细调试");
LogHelper.Debug("调试");
LogHelper.Info("信息");
LogHelper.Warning("警告");
LogHelper.Error("错误");
LogHelper.Fatal("致命");
```

---

## 10. WebAPI 远程控制

> DaqAbstract 内置 WebAPI 服务端，可通过 HTTP 接口远程控制采集设备。

### 10.1 启动 WebAPI

```csharp
using (var operate = new XxxOperate(config))
{
    await operate.OnAsync();
    
    // 启动 WebAPI（可选）
    var waResult = await operate.WAOnAsync(new WAModel
    {
        Port = 5000,  // WebAPI 端口
    });
    
    if (waResult.Status)
        Console.WriteLine("WebAPI 已启动，端口: 5000");
}
```

### 10.2 WebAPI 接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/on` | POST | 打开连接 |
| `/api/off` | POST | 关闭连接 |
| `/api/read` | POST | 读取数据（需传 Address JSON） |
| `/api/write` | POST | 写入数据（需传 WriteModel JSON） |
| `/api/getstatus` | POST | 获取连接状态 |
| `/api/switchlanguage` | POST | 切换中英文 |

### 10.3 WebAPI 方法

```csharp
// 启动 WebAPI
OperateResult WAOn(WAModel wAModel);

// 关闭 WebAPI
OperateResult WAOff();

// 获取 WebAPI 状态
OperateResult WAStatus();

// 获取请求示例（返回各接口的请求格式）
OperateResult WARequestExample();

// 异步版本
Task<OperateResult> WAOnAsync(WAModel wAModel, CancellationToken token = default);
Task<OperateResult> WAOffAsync(CancellationToken token = default);
Task<OperateResult> WAStatusAsync(CancellationToken token = default);
Task<OperateResult> WARequestExampleAsync(CancellationToken token = default);
```

---

## 11. 故障排查

### 11.1 编译错误

| 错误 | 正确写法 |
|------|----------|
| `result.IsSuccess` | `result.Status` ✅ |
| `Write(dict<string, object>)` | `Write(dict<string, (object, EncodingType?)>)` ✅ |
| MQTT `.Ip = ""` | `.IpAddress = ""` ✅ |
| 缺少 `using System.Collections.Concurrent;` | 添加到文件顶部 ✅ |
| 缺少 `using Snet.Driver.Core;` (DataFormat) | 添加到文件顶部 ✅ |

### 11.2 运行时问题

| 症状 | 原因 | 解决 |
|------|------|------|
| 连接超时 | 端口不对（默认 6688） | 显式设置标准端口（西门子=102, Modbus=502） |
| 读取无数据 | AddressName 格式错 | 对照地址格式速查表 |
| 转发不生效 | ISns 格式错 | 格式: `完整命名空间.类名.SN` |
| 转发不生效 | MQ 未设置 SN | MQ Basics.SN 必须设置用于 ISns 匹配 |
| 数据全 0 | AddressStartWithZero | 有些设备从 0 开始，有些从 1 |
| 数据乱码 | DataFormat 字节序 | 尝试 CDAB→ABCD→BADC→DCBA |
| 看不到日志 | ConsoleOut 未开启 | `LogHelper.Get().ConsoleOut = true` |
| 订阅无回调 | IsEnable=false | 设置 `IsEnable = true` |

---

## 12. 事件模型

> 采集设备通过事件机制推送数据，外部通过订阅事件接收数据。
> **每个事件都有同步和异步两个版本**，推荐使用异步事件进行 I/O 密集型处理。

### 12.1 事件类型

| 同步事件 | 异步事件 | 触发时机 | 数据类型 |
|----------|----------|----------|----------|
| `OnDataEvent` | `OnDataEventAsync` | 订阅数据到达时 | `EventDataResult` |
| `OnInfoEvent` | `OnInfoEventAsync` | 状态变化/告警时 | `EventDataResult` |
| `OnLanguageEvent` | `OnLanguageEventAsync` | 语言切换时 | `EventLanguageResult` |

> **推荐：** 同步事件用于简单日志/打印，异步事件用于数据库写入、HTTP 转发等 I/O 操作。

### 12.2 EventDataResult

```csharp
public class EventDataResult : ResultModel
{
    public bool Status { get; set; }      // 状态
    public string Message { get; set; }   // 消息
    public object ResultData { get; set; } // 结果数据
    
    // 获取泛型数据
    public T? GetSource<T>();
    
    // 获取详情
    public bool GetDetails(out EventDataResult result);
}
```

### 12.3 订阅数据示例（异步事件 — 推荐）

```csharp
// ✅ 推荐：异步数据事件（I/O 密集型处理）
siemens.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    
    // 获取数据字典
    var data = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
    if (data == null) return;
    
    foreach (var kv in data)
    {
        if (kv.Value.Quality == QualityType.Normal)
        {
            // 异步处理：写数据库、HTTP 转发等
            await SaveToDatabaseAsync(kv.Key, kv.Value.ResultValue);
        }
    }
};

// 异步信息事件
siemens.OnInfoEventAsync += async (sender, e) =>
{
    if (!e.Status)
        await LogHelper.WarningAsync($"告警: {e.Message}");
};
```

### 12.4 同步事件（简单场景）

```csharp
// 同步数据事件（仅适合简单日志/打印）
siemens.OnDataEvent += (sender, e) =>
{
    if (!e.Status) return;
    
    var data = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
    if (data == null) return;
    
    foreach (var kv in data)
    {
        if (kv.Value.Quality == QualityType.Normal)
        {
            Console.WriteLine($"{kv.Key} = {kv.Value.ResultValue}");
        }
    }
};

// 同步信息事件（状态告警）
siemens.OnInfoEvent += (sender, e) =>
{
    if (!e.Status)
        LogHelper.Warning($"告警: {e.Message}");
};
```

### 12.5 异步事件调度机制

> 异步事件处理器由框架通过 `EventingWrapperAsync<T>` 顺序 `await` 执行，每个处理器异常被独立捕获，不会影响其他订阅者或轮询循环。

---

## 13. 最佳实践

### 13.1 错误处理模板

```csharp
using var operate = new XxxOperate(config);
OperateResult result = await operate.OnAsync();
if (!result.Status)
{
    LogHelper.Fatal($"连接失败: {result.Message}");
    return;
}

try
{
    result = await operate.ReadAsync(address);
    if (result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    {
        foreach (var kv in data)
        {
            if (kv.Value.Quality == QualityType.Normal)
                ProcessData(kv.Key, kv.Value.ResultValue);
            else
                LogHelper.Warning($"数据异常: {kv.Key} Q={kv.Value.Quality}");
        }
    }

    operate.OnDataEventAsync += async (sender, e) =>
    {
        if (!e.Status) return;
        var d = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
        if (d != null)
        {
            foreach (var kv in d)
                ProcessData(kv.Key, kv.Value.ResultValue);
        }
        await Task.CompletedTask;
    };
    await operate.SubscribeAsync(address);
}
finally
{
    await operate.OffAsync();
}
```

### 13.2 ISns 格式速查

```
DAQ → MQTT:    "Snet.Mqtt.client.MqttClientOperate.{SN}"
DAQ → Kafka:   "Snet.Kafka.KafkaOperate.{SN}"
DAQ → RabbitMQ:"Snet.RabbitMQ.RabbitMQOperate.{SN}"
DAQ → NetMQ:   "Snet.NetMQ.NetMQOperate.{SN}"
DAQ → Netty:   "Snet.Netty.client.NettyClientOperate.{SN}"
```

### 13.3 性能建议

- 频繁采集用 `IsPersistentConnection = true`（长连接）
- 实时数据用订阅模式优于轮询
- 数据量大时用 `GetSimplify()` 减少序列化开销
- `ContentFormat` 尽量简洁，减少网络传输

---

## 14. 技能关联

> **本技能覆盖不了的协议？** → 用 [PluginDev-Skill](../PluginDev-Skill) 开发自定义插件，打包 ZIP 上传 Daq 工具热插拔加载。
>
> **已覆盖 NuGet 包：** 西门子/Modbus/三菱/欧姆龙/东方马达/汇川/OPC UA Client+Server/OPC DA/罗克韦尔/台达/基恩士/科伺(Kossi)/松下/倍福/GE/安川/英威腾/麦格米特/宇电(YuDian)/数据库/TEP/自由协议/模拟库/PQDIF(电力电表：DLT645/DLT698/CJT188/DTSU6606)。
>
> **Snet.Driver 内置（无独立 NuGet 包）：** Knx(楼宇自动化)/OpenProtocol(拧紧枪)/Sick(条码扫描)/Geniitek(振动传感器)/IDCard(身份证读卡器)/Toledo(称重)/IEC104(电力远动)/ShineIn Light(光源)/DAM3601(温控)。**不在列表中？** PluginDev-Skill 定义插件开发契约，AI 根据协议描述自动生成插件代码。
