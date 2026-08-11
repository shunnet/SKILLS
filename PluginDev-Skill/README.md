# PluginDev-Skill — Snet Daq 插件开发契约（IDaq + IMq）

**版本:** 1.0.0.9  
**作者:** Shun  
**许可证:** MIT  
**框架:** .NET 10.0

## 核心原则

> **采集什么/怎么采集、发什么/怎么发 —— AI 自行决定。**
> **数据怎么封装、方法返回什么 —— 硬性契约，必须遵守。**

**本技能覆盖两类插件：**
- **IDAq 插件**（SKILL.md 第 1-7 章）：数据采集，继承 `DaqAbstract<O,D>`，实现 8 个抽象方法
- **IMq 插件**（SKILL.md 第 8 章）：消息中间件，继承 `MqAbstract<O,D>`，实现 6 个抽象方法

## 交互流程

> 用户说大白话就行，AI 一个一个问题问清楚，确认后再生成代码。

**IDAq 插件（数据采集）：**

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 设备是什么？ | "温湿度传感器" | → 确定通信方式 |
| 2 | 怎么连的？ | "网线" "串口" | → TcpClient / Serial |
| 3 | 地址多少？ | "192.168.0.100" | → IpAddress + Port |
| 4 | 通信规矩是什么？ | "发01 03 00 00 00 02" | → 请求帧构建 |
| 5 | 返回数据长什么样？ | "7字节，第4-5是温度" | → 响应解析逻辑 |
| 6 | 要缓存吗？ | "不用" | → ProcessCache |
| 7 | 要加载DLL吗？ | "不用" | → Reflection |

**IMq 插件（消息中间件）：**

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 对接什么消息系统？ | "自定义TCP消息服务" | → 确定 Broker 协议 |
| 2 | Broker 地址？ | "192.168.0.1:6688" | → IpAddress + Port |
| 3 | 生产还是消费？ | "都做" | → Produce / Consume |
| 4 | 消息格式？ | "JSON字符串" "字节流" | → Produce(string/byte[]) + ResponseType |

**信息收集完毕后，输出确认摘要给用户，确认后再生成代码。**

## 插件架构

```
IDAq 插件：
你的 Operate 类
  └─ DaqAbstract<O, D>（8 个必须实现的抽象方法）
       └─ CoreUnify<O, D>（自动提供：单例、事件、日志、多语言、参数、WebApi）

你的 Data 类
  └─ Basics 继承 SubscribeData.SCData（订阅字段：HandleInterval/ChangeOut/AllOut/TaskNumber）
       └─ ProtocolType 枚举 + Attribute 标注

IMq 插件：
你的 Mq Operate 类
  └─ MqAbstract<O, D>（6 个必须实现的抽象方法）
       └─ CoreUnify<O, D>（同上自动提供）

你的 MqData 类
  └─ Basics 独立类（无订阅字段，含 SN + 连接参数）
       └─ ProtocolType 枚举 + Attribute 标注
```

## IDaq 插件：8 个必须实现的抽象方法

| 方法 | 入参 | 返回 | 核心约束 |
|------|------|------|----------|
| `OnAsync()` | — | `Task<OperateResult>` | 第一行 `BegOperateAsync`，失败 catch 调 `OffAsync(true)` |
| `OffAsync(bool)` | hardClose | `Task<OperateResult>` | 释放所有资源 |
| `GetStatusAsync()` | — | `Task<OperateResult>` | Status=连接状态（禁止 try/catch）|
| `GetBaseObjectAsync()` | — | `Task<OperateResult>` | ResultData=底层对象（禁止 try/catch）|
| **`ReadAsync(Address)`** | 地址配置 | `Task<OperateResult>` | **ResultData=`ConcurrentDictionary<string, AddressValue>`** |
| `WriteAsync(dict)` | `(object, EncodingType?)` 元组字典 | `Task<OperateResult>` | — |
| `SubscribeAsync(Address)` | 地址配置 | `Task<OperateResult>` | 经 `SubscribeOperate` 管理订阅生命周期 |
| `UnSubscribeAsync(Address)` | 地址配置 | `Task<OperateResult>` | 取消 Token |

## IMq 插件：6 个必须实现的抽象方法

| 方法 | 入参 | 返回 | 核心约束 |
|------|------|------|----------|
| `OnAsync()` | — | `Task<OperateResult>` | 连接 Broker，失败 catch 调 `OffAsync(true)` |
| `OffAsync(bool)` | hardClose | `Task<OperateResult>` | 释放生产者/消费者 |
| `GetStatusAsync()` | — | `Task<OperateResult>` | 连接状态（禁止 try/catch）|
| **`ProduceAsync(topic, byte[])`** | 主题+消息 | `Task<OperateResult>` | 发布消息（string 版是基类 virtual，可继承）|
| **`ConsumeAsync(topic)`** | 主题 | `Task<OperateResult>` | 订阅 + 经 `OnDataEventHandlerAsync` 推送消费数据 |
| `UnConsumeAsync(topic)` | 主题 | `Task<OperateResult>` | 取消 Token，释放消费者 |

## 事件模型

| 事件 | 触发时机 | 适用场景 |
|------|----------|----------|
| `OnDataEvent` | 订阅数据到达 | 同步处理 |
| `OnDataEventAsync` | 订阅数据到达 | 异步 I/O（消费数据推送）|
| `OnInfoEvent` | 状态变化/告警 | 连接监控 |

## 可选内置组件

| 组件 | 命名空间 | 用途 |
|------|---------|------|
| `ProcessCacheOperate` | `Snet.Core.cache.process` | 进程内内存缓存（自动过期、线程安全）|
| `ShareCacheOperate` | `Snet.Core.cache.share` | 跨进程共享内存缓存（MemoryMappedFile + Mutex）|
| `ReflectionOperate` | `Snet.Core.reflection` | 动态加载 DLL、调用方法、注册事件 |
| `BytesHandler` | `Snet.Core.handler` | `TransformAsync` 把原始字节按 BytesModel 解析为地址值 |
| `BytesTransformHandler` | `Snet.Core.handler` | 底层字节↔值转换（14 种类型 + 4 种字节序）|
| `ChannelOperate` | `Snet.Core.channel` | 高性能异步数据管道（基于 System.Threading.Channels）|

## Read 方法唯一数据流

```
采集原始数据（AI 决定方式）
  → AddressHandler.ExecuteDispose(item, rawValue, message)
    → 自动：空值检测 + 类型转换 + 反射解析 + MQ转发
    → 返回 AddressValue
  → ConcurrentDictionary<string, AddressValue>
  → EndOperate(true, resultData: param)
```

## Attribute 标注

| Attribute | 位置 | 用途 |
|-----------|------|------|
| `[Category][Description]` | Basics 每个字段 | UI 显示 |
| `[Display(use,show,mustFillIn,cate)]` | Basics 每个字段 | UI 控制（参数顺序：Use/Show/MustFillIn/DataCate）|
| `[AutoAllocatingTag(typeof(Enum))]` | ProtocolType 属性 | 协议标记 |
| `[AutoAllocating(string[])]` | 枚举每个值 | 参数列表 |

## 打包与部署

```bash
dotnet publish -c Release -o ./publish
Compress-Archive -Path ./publish/* -DestinationPath MyPlugin.zip
# → Daq 工具 → 插件设置 → 上传 ZIP
```

> **MQ 插件部署后还需配置注册：** 在 Daq 工具运行目录创建 `config/mq/{命名空间}.{类名}.{SN}.Mq.Config.json`（内容 = Basics 的 JSON），MqOperate 自动加载（详见 SKILL.md §8.6）。

## 参考实现

**IDAq 插件：** `Snet.Siemens` `Snet.Modbus` `Snet.Mitsubishi` `Snet.Omron` `Snet.Opc` `Snet.AllenBradley` `Snet.DB` `Snet.Sim` `Snet.Freedom` `Snet.TEP` `Snet.PQDIF`

**IMq 插件：** `Snet.Mqtt` `Snet.Kafka` `Snet.RabbitMQ` `Snet.NetMQ` `Snet.Netty`

> **📌 关于版本号：** 所有 `dotnet add package` 命令**必须**指定 `-v` 版本号（当前 26.222.1）。
> 获取最新版本：打开 `https://www.nuget.org/packages/<包名>` 查看，例如：
> - `https://www.nuget.org/packages/Snet.DB`
> - `https://www.nuget.org/packages/Snet.TEP`
