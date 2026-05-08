---
name: daq-skill
description: 工业物联网数据采集通信库，基于 Snet 框架，支持 PLC/工控/电力/机器人 等 20+ 种工业协议的数据读取、写入、订阅、状态获取，以及 Kafka/MQTT/RabbitMQ/NetMQ/Netty 消息中间件转发。所有采集库通过 ProtocolType 枚举自动选择底层驱动。支持"一句话"完成采集+转发。
version: 1.0.0.0
metadata:
  openclaw:
    requires:
      bins: []
      dotnet: "8.0"
---

# DAQ-Skill — 工业物联网数据采集技能

## 一句话采集中间件

本技能支持用户用一句自然语言描述需求，自动生成完整可编译运行的代码：

> "连接西门子S7-1500，IP 192.168.0.1，读取DB1.0（Int32）、DB1.4（Float）、M0.0（Bool），每500ms通过MQTT转发到broker 127.0.0.1:1883，topic为factory/line1"

→ 自动生成包含：项目创建 + NuGet安装 + DAQ配置 + MQ配置 + 数据转发 + 日志 + 错误处理的完整代码

→ 核心要点：**地址配置中的 `AddressMqParam` 实现订阅数据自动转发，无需手动编码转发逻辑**

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

```csharp
using System.Collections.Concurrent;
using Snet.Siemens;
using Snet.Mqtt.client;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;

#region 日志配置
LogHelper.Get().FolderName = "./logs";
LogHelper.Get().ConsoleOut = true;
#endregion

#region 1. 启动 MQTT 客户端（转发目标）
var mqConfig = new MqttClientData.Basics
{
    SN = "mqtt-target",              // 重要：设置 SN 用于 AddressMqParam.ISns 匹配
    IpAddress = "127.0.0.1",
    Port = 1883,
    UserName = null,
    Password = null,
};
using (MqttClientOperate mqClient = new MqttClientOperate(mqConfig))
{
    var mqResult = mqClient.On();
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

    using (SiemensOperate siemens = new SiemensOperate(daqConfig))
    {
        var daqResult = siemens.On();
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
        siemens.OnInfoEvent += (sender, e) =>
        {
            if (!e.Status)
                LogHelper.Warning($"事件告警: {e.Message}");
        };

        // OnDataEvent 可选：手动监听数据（即使有 AddressMqParam 自动转发，也可同时监听）
        siemens.OnDataEvent += (sender, e) =>
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
        };

        daqResult = siemens.Subscribe(address);
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

        siemens.Off();
        #endregion
    }
    mqClient.Off();
}
```

### 2.2 场景模板：采集 + 多点转发

**用户输入：** "连接Modbus TCP 192.168.0.2:502，读取40001（Int16）、40002（Int16），转发到2个MQTT broker"

```csharp
using System.Collections.Concurrent;
using Snet.Modbus;
using Snet.Driver.Core;  // DataFormat 枚举
using Snet.Mqtt.client;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Log;
using Snet.Utility;

using var mq1 = new MqttClientOperate(new MqttClientData.Basics
{
    SN = "broker1", IpAddress = "192.168.0.10", Port = 1883,
});
mq1.On();

using var mq2 = new MqttClientOperate(new MqttClientData.Basics
{
    SN = "broker2", IpAddress = "192.168.0.11", Port = 1883,
});
mq2.On();

// Modbus 配置
using var modbus = new ModbusOperate(new ModbusData.Basics
{
    IpAddress = "192.168.0.2", Port = 502,
    ProtocolType = ModbusData.ProtocolType.ModbusTcpNet,
    Station = 1, DataFormat = DataFormat.CDAB,
});
modbus.On();

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

modbus.Subscribe(address);
Console.WriteLine("多点转发已启动，按任意键停止...");
Console.ReadKey();
modbus.Off();
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
| RabbitMQ | `"Snet.RabbitMQ.RabbitMqOperate.my-rabbit-1"` |
| NetMQ | `"Snet.NetMQ.NetMqOperate.my-netmq-1"` |
| Netty | `"Snet.Netty.NettyOperate.my-netty-1"` |

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
OperateResult result = operate.On();

// 检查成功
bool ok = result.Status;         // true=成功, false=失败
string msg = result.Message;     // 状态描述
int ms = result.RunTime;         // 执行耗时(ms)

// 获取泛型结果数据
var data = result.GetRData<ConcurrentDictionary<string, AddressValue>>();

// 获取详情
if (result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var dict))
{
    // dict 包含所有点位数据
}
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
| "Modbus" "TCP" | `ModbusOperate` | `ModbusTcpNet` |
| "Modbus" "RTU" "串口" | `ModbusOperate` | `ModbusRtu` |
| "Modbus" "ASCII" | `ModbusOperate` | `ModbusAscii` |
| "三菱" "MC" "Q系列" | `MitsubishiOperate` | `MelsecMcNet` |
| "三菱" "FX" "串口" | `MitsubishiOperate` | `MelsecFxSerial` |
| "三菱" "Links" "计算机链接" | `MitsubishiOperate` | `MelsecFxLinks` |
| "三菱" "R系列" "MC R" | `MitsubishiOperate` | `MelsecMcRNet` |
| "欧姆龙" "Fins" "TCP" | `OmronOperate` | `OmronFinsNet` |
| "欧姆龙" "CIP" "NJ" "NX" "NY" | `OmronOperate` | `OmronCipNet` |
| "欧姆龙" "HostLink" | `OmronOperate` | `OmronHostLink` |
| "自由协议" "Freedom" "自定义报文" "raw" | `FreedomOperate` | `FreedomTcpNet` / `FreedomUdpNet` / `FreedomSerial` |
| "汇川" "AM" "AC" "H3U" "H5U" | `InovanceOperate` | `InovanceTcpNet` |
| "汇川" "Easy" | `InovanceOperate` | `InovanceEasyNet` |
| "汇川" "串口" | `InovanceOperate` | `InovanceSerial` |
| "OPC UA" "OPCUA" | `OpcUaClientOperate` | (无需 ProtocolType) |
| "罗克韦尔" "AB" "Allen Bradley" "CIP" | `AllenBradleyOperate` | `AllenBradleyNet` |
| "罗克韦尔" "Micro800" | `AllenBradleyOperate` | `AllenBradleyMicroCip` |
| "台达" "Delta" | `DeltaOperate` | `DeltaTcpNet` |
| "基恩士" "Keyence" "MC" | `KeyenceOperate` | `KeyenceMcNet` |
| "松下" "Panasonic" "MC" | `PanasonicOperate` | `PanasonicMcNet` |
| "麦格米特" "MegMeet" | `MegMeetOperate` | `MegMeetTcpNet` |
| "英威腾" "Invt" | `InvtOperate` | `ModbusTcpNet` |
| "倍福" "Beckhoff" "ADS" | `BeckhoffOperate` | `BeckhoffAdsNet` |
| "通用电气" "GE" "SRTP" | `GEOperate` | `GeSRTPNet` |
| "安川" "Yaskawa" "Memobus" | `YaskawaOperate` | `MemobusTcpNet` |
| "数据库" "DB" "SqlServer" "MySQL" "Oracle" "SQLite" | `DBOperate` | (DBType 枚举: SqlServer/MySql/Oracle/SQLite) |
| "TEP" "TCP扩展" "非标设备" "自定义协议" | `TepMasterOperate` | (无需 ProtocolType) |
| "模拟" "Sim" "测试" | `SimOperate` | (无需 ProtocolType) |

### 5.2 端口默认值

**所有协议库默认端口都是 6688**，务必根据实际设备显式指定：

| 协议 | 标准端口 | 库默认 |
|------|----------|--------|
| 西门子 S7 | 102 | 6688 |
| Modbus TCP | 502 | 6688 |
| OPC UA | 4840 | 6688 |
| 三菱 MC | 自适应 | 6688 |
| 欧姆龙 Fins | 9600 | 6688 |
| AB CIP | 44818 | 6688 |
| MQTT | 1883 | 6688 |

---

## 6. 地址格式速查

| PLC/协议 | 位地址 | 字地址 | 示例 |
|----------|--------|--------|------|
| 西门子 S7 | `M0.0`, `I0.0`, `Q0.0` | `DB1.0`, `MW0`, `IW0` | `DB1.DBD0` |
| Modbus | `00001`(线圈) `10001`(输入) | `40001`(保持) `30001`(输入) | `40001` |
| 三菱 MC | `M0`, `X0`, `Y0` | `D100`, `W100`, `R100` | `D100` |
| 三菱 FX | `M0`, `X0`, `Y0` | `D100` | `D100` |
| 欧姆龙 Fins | `CIO`, `W`, `H`, `D`, `A` | 按区域 | `D100`, `CIO100` |
| 欧姆龙 CIP | Tag名 | `MyTag` | `Program:MainProgram.Var` |
| 罗克韦尔 | Tag名 | `MyTag` | `Program:MainProgram.Var` |
| 汇川 | `M0`, `X0`, `Y0` | `D100`, `SD100` | 兼容三菱/Modbus |
| OPC UA | NodeId | `ns=0;i=2258` | `ns=2;s=MyVariable` |

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
using (SiemensOperate operate = new SiemensOperate(config))
{
    operate.On();
    // 地址: DB1.0, M0.0, I0.0, Q0.0, MW0, IW0 等
}
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
using (ModbusOperate operate = new ModbusOperate(config)) { operate.On(); }
```

### 7.3 OPC UA

```csharp
using Snet.Opc.ua.client;

var config = new OpcUaClientData.Basics
{
    ServerUrl = "opc.tcp://192.168.0.1:4840",
    AType = Snet.Opc.core.Data.AuType.UserName,
    UserName = "user",
    Password = "password",
};
using (OpcUaClientOperate operate = new OpcUaClientOperate(config))
{
    operate.On();
    // 地址: ns=0;i=2258 或 ns=2;s=MyVariable
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
using (MitsubishiOperate operate = new MitsubishiOperate(config)) { operate.On(); }
```

### 7.5 模拟库（无硬件测试）

```csharp
using Snet.Sim;

using (SimOperate operate = new SimOperate(new SimData.Basics { SN = "test" }))
{
    operate.On();
    // 5 种虚拟地址: VirtualStatic, VirtualDynamicRandom, VirtualDynamicRandomRange,
    //               VirtualDynamicSequential, VirtualDynamicSequentialRange
    Address address = new Address(new AddressDetails
    {
        SN = "v1", AddressName = "SimVal", AddressDataType = DataType.Int32,
        AddressType = AddressType.VirtualDynamicRandom,
        AddressExtendParam = (0, 1000)  // 范围
    });
    var result = operate.Read(address);
}
```

### 7.6 数据库采集（Snet.DB）

> NuGet: `dotnet add package Snet.DB`
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
    OperateResult result = operate.On();
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
    result = operate.Read(address);
    if (result.Status && result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    {
        foreach (var kv in data)
        {
            Console.WriteLine($"{kv.Key}: {kv.Value.ResultValue}");
        }
    }

    // 订阅（定时执行 SQL + 自动转发）
    operate.Subscribe(address);
    Console.WriteLine("数据库定时采集已启动，按任意键停止...");
    Console.ReadKey();

    operate.Off();
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
    operate.On();

    // 查询（Dapper）
    var list = operate.Query<SensorData>("SELECT * FROM SensorData WHERE DeviceId='D001'");

    // 增删改（Dapper）
    operate.Execute("INSERT INTO SensorData (DeviceId, Temperature) VALUES (@DId, @Temp)",
        new { DId = "D001", Temp = 25.6 });

    operate.Off();
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

> NuGet: `dotnet add package Snet.TEP`
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
    OperateResult result = operate.On();
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
    result = operate.Read(address);
    if (result.Status && result.GetDetails<ConcurrentDictionary<string, AddressValue>>(out var data))
    {
        foreach (var kv in data)
            Console.WriteLine($"{kv.Key} = {kv.Value.ResultValue}");
    }

    // 订阅（设备主动上传数据时自动接收 + 转发）
    operate.OnDataEvent += (sender, e) =>
    {
        if (!e.Status) return;
        var d = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
        if (d != null)
        {
            foreach (var kv in d)
                Console.WriteLine($"📡 TEP: {kv.Key} = {kv.Value.ResultValue}");
        }
    };
    operate.Subscribe(address);

    // 写入数据到设备
    var writeValues = new ConcurrentDictionary<string, (object value, EncodingType? encodingType)>
    {
        ["测试设备.10086.温度"] = (25.6f, null),
        ["测试设备.10086.状态"] = (true, null)
    };
    operate.Write(writeValues);

    Console.WriteLine("按任意键停止...");
    Console.ReadKey();
    operate.Off();
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
OperateResult result = clientOperate.On();
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
result = clientOperate.DataUpload(data);
Console.WriteLine($"数据上传: {(result.Status ? "成功" : result.Message)}");

Console.ReadKey();
clientOperate.Off();
```

---

## 8. 消息中间件

### 8.1 MQTT Client

```csharp
using Snet.Mqtt.client;

var config = new MqttClientData.Basics
{
    SN = "my-mqtt",              // 重要：用于 ISns 匹配
    IpAddress = "127.0.0.1",     // ← 属性名是 IpAddress，不是 Ip！
    Port = 1883,                  // 库默认 6688，需改标准端口
    UserName = "admin",
    Password = "password",
    ResponseType = ResponseType.Content,  // Content/Bytes/ContentWithTopic
};
using (MqttClientOperate mq = new MqttClientOperate(config))
{
    mq.On();

    // 生产
    mq.Produce("topic/hello", "hello world");

    // 消费
    mq.OnDataEvent += (sender, e) =>
    {
        if (!e.Status) return;
        string content = e.GetRData<string>();
        Console.WriteLine($"收到: {content}");
    };
    mq.Consume("topic/hello");

    Console.ReadKey();
}
```

### 8.2 Kafka / RabbitMQ / NetMQ / Netty

```
// Kafka: dotnet add package Snet.Kafka
//   Operate: KafkaOperate, Config: KafkaData.Basics
//   ISns: "Snet.Kafka.KafkaOperate.my-kafka"
//
// RabbitMQ: dotnet add package Snet.RabbitMQ
//   Operate: RabbitMqOperate, Config: RabbitMqData.Basics
//   ISns: "Snet.RabbitMQ.RabbitMqOperate.my-rabbit"
//
// NetMQ: dotnet add package Snet.NetMQ
//   Operate: NetMqOperate, Config: NetMqData.Basics
//   ISns: "Snet.NetMQ.NetMqOperate.my-netmq"
//
// Netty: dotnet add package Snet.Netty
//   Operate: NettyOperate, Config: NettyData.Basics
//   ISns: "Snet.Netty.NettyOperate.my-netty"
```

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

## 10. 故障排查

### 10.1 编译错误

| 错误 | 正确写法 |
|------|----------|
| `result.IsSuccess` | `result.Status` ✅ |
| `Write(dict<string, object>)` | `Write(dict<string, (object, EncodingType?)>)` ✅ |
| MQTT `.Ip = ""` | `.IpAddress = ""` ✅ |
| 缺少 `using System.Collections.Concurrent;` | 添加到文件顶部 ✅ |
| 缺少 `using Snet.Driver.Core;` (DataFormat) | 添加到文件顶部 ✅ |

### 10.2 运行时问题

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

## 11. 最佳实践

### 11.1 错误处理模板

```csharp
using (var operate = new XxxOperate(config))
{
    OperateResult result = operate.On();
    if (!result.Status)
    {
        LogHelper.Fatal($"连接失败: {result.Message}");
        return;
    }

    try
    {
        result = operate.Read(address);
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

        operate.OnDataEvent += OnDataReceived;
        operate.Subscribe(address);
    }
    finally
    {
        operate.Off();
    }
}
```

### 11.2 ISns 格式速查

```
DAQ → MQTT:    "Snet.Mqtt.client.MqttClientOperate.{SN}"
DAQ → Kafka:   "Snet.Kafka.KafkaOperate.{SN}"
DAQ → RabbitMQ:"Snet.RabbitMQ.RabbitMqOperate.{SN}"
DAQ → NetMQ:   "Snet.NetMQ.NetMqOperate.{SN}"
DAQ → Netty:   "Snet.Netty.NettyOperate.{SN}"
```

### 11.3 性能建议

- 频繁采集用 `IsPersistentConnection = true`（长连接）
- 实时数据用订阅模式优于轮询
- 数据量大时用 `GetSimplify()` 减少序列化开销
- `ContentFormat` 尽量简洁，减少网络传输

---

## 12. 技能关联

> **本技能覆盖不了的协议？** → 用 [PluginDev-Skill](../PluginDev-Skill) 开发自定义插件，打包 ZIP 上传 Daq 工具热插拔加载。
>
> **已覆盖：** 西门子/Modbus/三菱/欧姆龙/汇川/OPC UA/罗克韦尔/台达/基恩士/松下/倍福/GE/安川/英威腾/麦格米特/数据库/TEP/自由协议/模拟库。**不在列表中？** PluginDev-Skill 定义插件开发契约，AI 根据协议描述自动生成插件代码。
