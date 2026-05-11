# PluginDev-Skill — Snet Daq 插件开发契约

**版本:** 1.0.0.1
**作者:** Shun
**许可证:** MIT
**框架:** .NET 10.0

## 核心原则

> **采集什么、用什么方式 —— AI 自行决定。**
> **数据怎么封装、方法返回什么 —— 硬性契约，必须遵守。**

## 交互流程

> 用户说大白话就行，AI 一个一个问题问清楚，确认后再生成代码。

| # | 问用户什么 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | 设备是什么？ | "温湿度传感器" | → 确定通信方式 |
| 2 | 怎么连的？ | "网线" "串口" | → TcpClient / Serial |
| 3 | 地址多少？ | "192.168.0.100" | → IpAddress + Port |
| 4 | 通信规矩是什么？ | "发01 03 00 00 00 02" | → 请求帧构建 |
| 5 | 返回数据长什么样？ | "7字节，第4-5是温度" | → 响应解析逻辑 |
| 6 | 要缓存吗？ | "不用" | → ProcessCache |
| 7 | 要加载DLL吗？ | "不用" | → Reflection |

**信息收集完毕后，输出确认摘要给用户，确认后再生成代码。**

## 插件架构

```
你的 Operate 类
  └─ DaqAbstract<O, D>（8 个必须实现的抽象方法）
       └─ CoreUnify<O, D>（自动提供：单例、事件、日志、多语言、参数、WebApi）

你的 Data 类
  └─ Basics 继承 SubscribeData.SCData
       └─ ProtocolType 枚举 + Attribute 标注
```

## 8 个必须实现的抽象方法

| 方法 | 入参 | 返回 | 核心约束 |
|------|------|------|----------|
| `On()` | — | `OperateResult` | 第一行 `BegOperate()`，返回 `EndOperate()` |
| `Off(bool)` | hardClose | `OperateResult` | 释放所有资源 |
| `GetStatus()` | — | `OperateResult` | Status=连接状态 |
| `GetBaseObject()` | — | `OperateResult` | ResultData=底层对象 |
| **`Read(Address)`** | 地址配置 | `OperateResult` | **ResultData=`ConcurrentDictionary<string, AddressValue>`** |
| `Write(dict)` | `(object, EncodingType?)` 元组字典 | `OperateResult` | — |
| `Subscribe(Address)` | 地址配置 | `OperateResult` | 调用 Read + 触发 OnDataEvent |
| `UnSubscribe(Address)` | 地址配置 | `OperateResult` | 取消 Token |

## 继承字段默认值（SubscribeData.SCData）

| 字段 | 默认值 | 说明 |
|------|--------|------|
| `HandleInterval` | 1000 | 订阅轮询间隔(ms) |
| `ChangeOut` | **true** | 仅变化时输出（true=只抛变化项） |
| `AllOut` | **false** | 变化项与未变项一同抛出 |
| `TaskNumber` | **5** | 并行任务数 |

## 事件模型

| 事件 | 触发时机 | 适用场景 |
|------|----------|----------|
| `OnDataEvent` | 订阅数据到达 | 同步处理 |
| `OnDataEventAsync` | 订阅数据到达 | 异步 I/O |
| `OnInfoEvent` | 状态变化/告警 | 连接监控 |

## 可选内置组件

| 组件 | 命名空间 | 用途 |
|------|---------|------|
| `ProcessCacheOperate` | `Snet.Core.cache.process` | 进程内内存缓存（自动过期、线程安全）|
| `ShareCacheOperate` | `Snet.Core.cache.share` | 跨进程共享内存缓存（MemoryMappedFile + Mutex）|
| `ReflectionOperate` | `Snet.Core.reflection` | 动态加载 DLL、调用方法、注册事件 |

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
| `[Display(show,use,edit,cate)]` | Basics 每个字段 | UI 控制 |
| `[AutoAllocatingTag(typeof(Enum))]` | ProtocolType 属性 | 协议标记 |
| `[AutoAllocating(string[])]` | 枚举每个值 | 参数列表 |

## 参考实现

`Snet.Siemens` `Snet.Modbus` `Snet.Mitsubishi` `Snet.DB` `Snet.Sim` `Snet.Freedom` `Snet.TEP` `Snet.PQDIF`

> **📌 关于版本号：** 所有 `dotnet add package` 命令**必须**指定 `-v` 版本号。
> 获取最新版本：打开 `https://www.nuget.org/packages/<包名>` 查看，例如：
> - `https://www.nuget.org/packages/Snet.DB`
> - `https://www.nuget.org/packages/Snet.TEP`
