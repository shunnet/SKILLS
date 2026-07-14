---
name: plugindev-skill
description: Snet.Iot.Daq 插件开发技能。严格定义插件开发契约：必须实现的抽象方法、必须遵循的返回类型、必须使用的数据标注、必须调用的框架方法。AI 自行决定采集方式（TCP/HTTP/文件/串口），但必须遵守契约。
version: 1.0.0.7
metadata:
  hermes:
    tags: [plugin-development, daq, iot, dotnet, contract, code-generation]
    related_skills: [daq-skill]
    homepage: https://shunnet.top
---

# PluginDev-Skill — Snet Daq 插件开发契约

> **运行环境：** .NET 10.0 SDK，插件引用 `Snet.Core` NuGet 包

## 核心原则

> **采集什么、用什么方式采集 —— 由 AI 根据用户描述自行决定。**
> **但数据如何封装、方法返回什么类型、生命周期如何管理 —— 这是硬性契约，必须严格遵守。**

---

## NuGet 引用规范

> **⚠️ 以下规则在生成代码时必须遵守，违反会导致编译或运行时错误。**

### 版本号：必须指定

所有 `dotnet add package` 命令**必须**带 `-v` 版本号，**禁止使用 `*` 通配符**（运行时会报版本冲突错误）。

**获取最新版本的方式：** 访问 `https://www.nuget.org/packages/<包名>`，页面顶部显示的版本即为最新稳定版。

```bash
# ✅ 正确：指定版本号
dotnet add package Snet.Core -v 1.0.0.1

# ❌ 错误：不带版本号（使用 * 通配符，运行时报错）
dotnet add package Snet.Core
```

### 传递依赖：只引用顶层包

插件开发通常只需引用 `Snet.Core`（提供 `DaqAbstract`、`CoreUnify`、`IDaq` 等基类和接口），
`Snet.Model`、`Snet.Log`、`Snet.Utility`、`Snet.Driver` 等会作为传递依赖自动引入，**不要手动添加**。

---

## 0. 交互流程 — 开发插件前先问清楚

> **原则：用户说大白话就行，AI 负责翻译成技术参数。**
> 用户可能只说"我要做一个XX设备的插件"，AI 必须先搞清楚以下信息再动手写代码。

### 0.1 开发插件：需要问用户什么

| # | 大白话问题 | 用户会怎么回答 | AI 翻译成什么 |
|---|-----------|---------------|--------------|
| 1 | **你要对接的是什么设备？** | "一个温湿度传感器" "一台激光切割机" "一个扫码枪" | → 确定协议类型和通信方式 |
| 2 | **设备是用网线连的还是用线（USB/串口线）连的？** | "网线" "串口" "走HTTP接口" | → TcpClientOperate / SerialOperate / HttpClientOperate |
| 3 | **设备的地址是多少？** | "192.168.0.100" "COM3,9600,8,N,1" | → 连接参数 |
| 4 | **和设备通信的规矩是什么？** | "发01 03 00 00 00 02就能读" "发GET /api/data返回JSON" "发ASCII命令" | → 请求帧构建 |
| 5 | **设备返回的数据长什么样？** | "返回7个字节，第4-5字节是温度" "返回JSON `{"temp":25.6}`" | → 响应解析逻辑 |
| 6 | **读到的数据要存起来吗？** | "不用" "要，下次读的时候用上一次的值对比" | → 是否用 ProcessCacheOperate |
| 7 | **需要加载外部DLL处理数据吗？** | "不用" "有个解析库，DLL放在同目录下" | → 是否用 ReflectionOperate |
| 8 | **读到的数据要发到哪里？** | "发到MQTT" "不用转发" | → AddressMqParam 配置 |

### 0.2 用户说不清楚时怎么办

| 用户说 | AI 追问 |
|--------|--------|
| "我要做一个传感器插件" | "好的！传感器是什么牌子型号？有说明书吗？通信方式是网口还是串口？" |
| "设备用网线连的" | "IP地址和端口知道吗？一般是设备上贴的标签或者说明书上有" |
| "发命令读数据" | "能告诉我发送的命令是什么吗？是十六进制还是ASCII？比如 `01 03 00 00 00 02` 这种" |
| "返回一堆字节" | "能告诉我返回数据的格式吗？比如前2个字节是什么，后面是什么？或者发个示例给我看看" |
| "我也不知道协议" | "没关系，能拍个设备的照片或者告诉我型号吗？我帮你查一下" |

### 0.3 信息收集完毕后的输出

收集完信息后，AI 应该输出一个确认摘要，让用户确认：

```
好的，我整理一下你的需求：

设备：温湿度传感器，型号XXX
连接：网线，IP 192.168.0.100，端口 502
协议：发送 01 03 00 00 00 02，返回7字节
  → 第4-5字节：温度（÷10）
  → 第6-7字节：湿度（÷10）
转发：MQTT 到 127.0.0.1:1883，topic: sensor/temp

没问题的话我开始生成代码。
```

用户确认后再生成代码，避免返工。

---

## 1. 继承链契约

```
你的 Operate 类
  └─ 必须继承 DaqAbstract<你的Operate类, 你的Data.Basics>
       └─ DaqAbstract 继承 CoreUnify（自动提供：单例、事件、日志、多语言、参数、WebApi）
            └─ 必须实现 8 个抽象异步方法 + 3 个属性
            └─ 同步方法是薄封装：`=> XxxAsync().GetAwaiter().GetResult()`，从基类自动继承
```

```csharp
public class XxxOperate : DaqAbstract<XxxOperate, XxxData.Basics>, IDaq
{
    // ============ 3 个必须属性 ============
    protected override string CN => "中文名称";
    protected override string CD => "中文描述";
    protected override List<propertie> AP => new List<propertie>
    {
        new propertie { PropertyName = "ServiceName", Description = "命名空间", Default = this.GetType().FullName }
    };

    // ============ 构造函数（必须调用 LanguageHandler）============
    public XxxOperate() : base() { LanguageHandler(); }
    public XxxOperate(XxxData.Basics basics) : base(basics) { LanguageHandler(); }

    // ============ 8 个抽象异步方法（核心实现）============
    public override async Task<OperateResult> OnAsync(CancellationToken token = default) { ... }
    public override async Task<OperateResult> OffAsync(bool hardClose = false, CancellationToken token = default) { ... }
    public override async Task<OperateResult> GetStatusAsync(CancellationToken token = default) { ... }
    public override async Task<OperateResult> GetBaseObjectAsync(CancellationToken token = default) { ... }
    public override async Task<OperateResult> ReadAsync(Address address, CancellationToken token = default) { ... }
    public override async Task<OperateResult> WriteAsync(ConcurrentDictionary<string, (object, EncodingType?)> values, CancellationToken token = default) { ... }
    public override async Task<OperateResult> SubscribeAsync(Address address, CancellationToken token = default) { ... }
    public override async Task<OperateResult> UnSubscribeAsync(Address address, CancellationToken token = default) { ... }
    
    // 同步方法自动从基类继承（无需实现）：
    // On(), Off(), Read(), Write(), Subscribe(), UnSubscribe(), GetStatus(), GetBaseObject()
}
```

---

## 2. 方法契约详解

> **核心约束：** 8 个抽象异步方法必须 `await BegOperateAsync → try → await GetStatusAsync检查 → 实现 → catch`，2 个方法禁止 try/catch。

### 2.1 OnAsync — 打开连接

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| 状态检查 | 已连接则返回失败（与其它方法相反） |
| 失败处理 | **必须在 catch 中调用 `await OffAsync(true, token)` 清理** |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> OnAsync(CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        var status = await GetStatusAsync(token);
        if (status.GetDetails(out string? message))
            return await EndOperateAsync(false, message, token);  // 已连接，返回失败

        // AI 自行决定如何连接（TCP、文件、HTTP...）

        return await EndOperateAsync(true, token: token);
    }
    catch (Exception ex)
    {
        await OffAsync(true, token);   // ← 必须：失败时清理
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.2 OffAsync(bool hardClose) — 关闭连接

| 约束 | 值 |
|------|-----|
| hardClose=false | 未连接则返回失败 |
| hardClose=true | 跳过状态检查，强制关闭 |
| 必须释放 | SubscribeOperate、VAM、通信对象、日志 |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> OffAsync(bool hardClose = false, CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        if (!hardClose)
        {
            var status = await GetStatusAsync(token);
            if (!status.GetDetails(out string? message))
                return await EndOperateAsync(false, message, token);
        }

        // 释放顺序：订阅 → 虚拟地址 → 通信对象 → 日志
        if (subscribeOperate != null)
        {
            await subscribeOperate.OffAsync(token);
            subscribeOperate = null;
        }
        VAM?.Dispose();

        // AI 自行决定如何断开连接

        return await EndOperateAsync(true, token: token);
    }
    catch (Exception ex)
    {
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.3 GetStatusAsync — 获取状态

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| Status=true | 已连接 |
| Status=false | 未连接 |
| try/catch | **禁止**（简单状态判断，无需异常捕获） |
| consoleOutput | 建议 `false`（避免高频日志刷屏） |

```csharp
public override async Task<OperateResult> GetStatusAsync(CancellationToken token = default)
{
    await BegOperateAsync(token);
    // AI 判断连接状态，直接返回
    return await EndOperateAsync(/* 已连接? */, consoleOutput: false, token: token);
}
```

### 2.4 GetBaseObjectAsync — 获取底层对象

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| ResultData | 底层连接对象（TcpClient、SerialPort、HttpClient 等） |
| try/catch | **禁止** |
| 前置检查 | 必须先 GetStatusAsync 确认已连接 |

```csharp
public override async Task<OperateResult> GetBaseObjectAsync(CancellationToken token = default)
{
    await BegOperateAsync(token);
    var status = await GetStatusAsync(token);
    if (!status.GetDetails(out string? message))
        return await EndOperateAsync(false, message, token);

    return await EndOperateAsync(true, resultData: /* 底层对象 */, token: token);
}
```

### 2.5 ReadAsync(Address address) — 读取数据（核心）

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| ResultData 类型 | **`ConcurrentDictionary<string, AddressValue>`** |
| 前置检查 | GetStatusAsync + `address.CheckAddress()` |
| 每个点位 | `AddressHandler.ExecuteDispose(item, rawValue, message)` |
| 跳过禁用 | `if (!item.IsEnable) continue` |
| 虚拟地址 | `VAM.InitVirtualAddress(item, out bool IsVA)` |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> ReadAsync(Address address, CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        var status = await GetStatusAsync(token);
        if (!status.GetDetails(out string? message))
            return await EndOperateAsync(false, message, token);

        if (!address.CheckAddress())
            return await EndOperateAsync(false, "存在无效点位数据，操作失败", token);

        ConcurrentDictionary<string, AddressValue> param = new();
        foreach (var item in address.AddressArray)
        {
            if (!item.IsEnable) continue;

            bool IsVA = false;
            VAM.InitVirtualAddress(item, out IsVA);

            object? value = null;
            string resultMsg = "失败";

            if (IsVA)
            {
                value = VAM.Read(item);
            }
            else
            {
                // ═══ AI 在此实现采集逻辑 ═══
                value = /* AI 自行实现的采集 */;
                resultMsg = value != null ? "成功" : "读取失败";
            }

            AddressValue? av = AddressHandler.ExecuteDispose(item, value, resultMsg);
            if (av != null)
                param.AddOrUpdate(item.AddressName, av, (k, v) => av);
        }
        return param.Count > 0
            ? await EndOperateAsync(true, resultData: param, token: token)
            : await EndOperateAsync(false, "读取失败", token);
    }
    catch (Exception ex)
    {
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.6 WriteAsync(values) — 写入数据

| 约束 | 值 |
|------|-----|
| 入参 | `ConcurrentDictionary<string, (object value, EncodingType? encodingType)>` |
| 返回类型 | `Task<OperateResult>` |
| 前置检查 | GetStatusAsync |
| 虚拟地址 | `VAM.IsVirtualAddress(key)` → `VAM.Write(key, value)` |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> WriteAsync(
    ConcurrentDictionary<string, (object value, EncodingType? encodingType)> values,
    CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        var status = await GetStatusAsync(token);
        if (!status.GetDetails(out string? message))
            return await EndOperateAsync(false, message, token);

        foreach (var (addressName, (value, encoding)) in values)
        {
            if (VAM.IsVirtualAddress(addressName))
            {
                VAM.Write(addressName, value);
            }
            else
            {
                // AI 自行实现写入逻辑
            }
        }
        return await EndOperateAsync(true, token: token);
    }
    catch (Exception ex)
    {
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.7 SubscribeAsync(Address address) — 订阅数据

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| 前置检查 | GetStatusAsync + `address.CheckAddress()` |
| 必须创建 | `SubscribeOperate.InstanceAsync(basics)` 管理订阅生命周期 |
| 数据事件 | 通过 `OnDataEvent?.Invoke` / `OnDataEventAsync?.Invoke` 推送 |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> SubscribeAsync(Address address, CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        var status = await GetStatusAsync(token);
        if (!status.GetDetails(out string? message))
            return await EndOperateAsync(false, message, token);

        if (!address.CheckAddress())
            return await EndOperateAsync(false, "存在无效点位数据，操作失败", token);

        subscribeToken = new CancellationTokenSource();
        subscribeOperate = SubscribeOperate.InstanceAsync(basics);
        await subscribeOperate.OnAsync(token);

        _ = Task.Run(async () =>
        {
            while (!subscribeToken.Token.IsCancellationRequested)
            {
                OperateResult r = await ReadAsync(address, subscribeToken.Token);
                if (r.Status)
                {
                    var e = new EventDataResult(true, "订阅数据", r.ResultData);
                    OnDataEvent?.Invoke(this, e);
                    if (OnDataEventAsync != null)
                        await OnDataEventAsync.Invoke(this, e);
                }
                await Task.Delay(
                    basics.HandleInterval > 0 ? basics.HandleInterval : 1000,
                    subscribeToken.Token);
            }
        }, subscribeToken.Token);

        return await EndOperateAsync(true, token: token);
    }
    catch (Exception ex)
    {
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.8 UnSubscribeAsync(Address address) — 取消订阅

| 约束 | 值 |
|------|-----|
| 返回类型 | `Task<OperateResult>` |
| 前置检查 | GetStatusAsync |
| 必须操作 | 取消 Token，关闭 SubscribeOperate |
| try/catch | **必须** |

```csharp
public override async Task<OperateResult> UnSubscribeAsync(Address address, CancellationToken token = default)
{
    await BegOperateAsync(token);
    try
    {
        var status = await GetStatusAsync(token);
        if (!status.GetDetails(out string? message))
            return await EndOperateAsync(false, message, token);

        subscribeToken?.Cancel();
        if (subscribeOperate != null)
        {
            await subscribeOperate.OffAsync(token);
            subscribeOperate = null;
        }
        return await EndOperateAsync(true, token: token);
    }
    catch (Exception ex)
    {
        return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
    }
}
```

### 2.9 事件模型 — OnDataEvent / OnDataEventAsync

`CoreUnify` 提供同步和异步两种数据事件，订阅数据到达时自动触发：

```csharp
// 同步事件（推荐用于简单日志/打印）
operate.OnDataEvent += (sender, e) =>
{
    if (!e.Status) return;
    var data = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
    // 处理数据...
};

// 异步事件（推荐用于 I/O 密集型处理，如写数据库/HTTP）
operate.OnDataEventAsync += async (sender, e) =>
{
    if (!e.Status) return;
    var data = e.GetSource<ConcurrentDictionary<string, AddressValue>>();
    await SaveToDatabaseAsync(data);
};
```

| 事件 | 触发时机 | 数据类型 | 适用场景 |
|------|----------|----------|----------|
| `OnDataEvent` | 订阅数据到达时 | `EventDataResult` | 同步处理（日志/打印/内存操作） |
| `OnInfoEvent` | 状态变化/告警时 | `EventDataResult` | 连接状态监控 |
| `OnDataEventAsync` | 订阅数据到达时 | `EventDataResult` | 异步 I/O（数据库写入/HTTP 转发） |

---

## 3. 数据类契约

### 3.1 基础数据类

> **异步兼容：** Basics 字段同时支持同步和异步事件处理模式，`HandleInterval` 等字段在 async 订阅轮询中同样生效。

```csharp
public class XxxData
{
    public class Basics : SubscribeData.SCData   // ← 必须继承
    {
        // ── 必须：唯一标识符 ──
        [Category("基础数据")]
        [Description("唯一标识符")]
        public string SN { get; set; } = Guid.NewGuid().ToUpperNString();

        // ── 连接参数（AI 根据协议自行定义）──
        [Description("Ip地址")]
        [Verify(@"正则", "提示")]
        [Display(true, true, true, ParamModel.dataCate.text)]
        public string? IpAddress { get; set; } = "127.0.0.1";

        // ── 必须：协议类型 ──
        [Description("协议类型")]
        [JsonConverter(typeof(JsonStringEnumConverter))]
        [AutoAllocatingTag(typeof(ProtocolType))]  // ← 用此属性标注
        [Display(true, true, true, ParamModel.dataCate.select)]
        public ProtocolType ProtocolType { get; set; } = ProtocolType.XxxProtocol;
    }

    // ── 必须：协议枚举 ──
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public enum ProtocolType
    {
        [Description("协议中文描述")]
        [AutoAllocating(new string[] {
            "SN",              // 必须
            "HandleInterval",  // 必须
            "ChangeOut",       // 必须
            "AllOut",          // 必须
            "TaskNumber",      // 必须
            // ... AI 定义的连接参数
        })]
        XxxProtocol,
    }
}
```

### 3.2 继承得来的字段（无需声明）

`SubscribeData.SCData` 自动提供：

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `HandleInterval` | `int` | 1000 | 订阅轮询间隔(ms) |
| `ChangeOut` | `bool` | true | 仅变化时输出；true=只抛变化项，false=实时全量输出 |
| `AllOut` | `bool` | false | 变化项与未变项一同抛出；当 ChangeOut 为 true 时生效，确保此批数据完整性 |
| `TaskNumber` | `int` | 5 | 并行任务数，一个任务属于一个队列 |

### 3.3 属性标注速查

| Attribute | 用途 | 示例 |
|-----------|------|------|
| `[Category("分类")]` | Daq UI 分组 | `[Category("基础数据")]` |
| `[Description("描述")]` | 字段标签+提示 | `[Description("Ip地址")]` |
| `[Display(show,use,edit,cate)]` | UI 显示控制 | `[Display(true,true,true,text)]` |
| `[Unit("单位")]` | 字段单位 | `[Unit("ms")]` |
| `[Verify(@"正则","提示")]` | 输入验证 | `[Verify(@"^\d+$","必须为数字")]` |
| `[AutoAllocatingTag(typeof(Enum))]` | 协议类型标记 | 仅 ProtocolType 用 |
| `[AutoAllocating(string[])]` | 协议参数列表 | 协议枚举每个值用 |
| `[JsonConverter(typeof(JsonStringEnumConverter))]` | JSON枚举序列化 | 枚举类型用 |

---

## 4. SubscribeOperate — 订阅管理

由 `Snet.Core.subscription.SubscribeOperate` 提供：

```csharp
private SubscribeOperate? subscribeOperate;

// 异步启动订阅时
subscribeOperate = SubscribeOperate.InstanceAsync(basics);
await subscribeOperate.OnAsync();

// 停止订阅时
subscribeOperate?.Off();
subscribeOperate = null;
```

---

## 5. VirtualAddressManage — 虚拟地址

```csharp
private VirtualAddressManage VAM = new VirtualAddressManage();

// Read 中使用
bool IsVA = false;
VAM.InitVirtualAddress(item, out IsVA);
if (IsVA)
{
    value = VAM.Read(item);  // 读取虚拟地址值
}
else
{
    // 读取实际设备
}

// Write 中使用
if (VAM.IsVirtualAddress(addressName))
    VAM.Write(addressName, value);
```

---

## 6. 可选内置组件（按需使用）

以下组件需手动实例化，不是自动提供的。根据插件需求按需使用：

| 方法 | 用途 |
|------|------|
| `BegOperateAsync(token)` | 每个异步方法首行调用，初始化操作上下文 |
| `BegOperate()` | 同步版本（同步封装方法使用） |
| `EndOperateAsync(bool status, string? message=null, object? resultData=null, Exception? exception=null, bool logOutput=true, bool consoleOutput=true, CancellationToken token=default)` | 异步统一返回 OperateResult，自动记录耗时和日志。`consoleOutput: false` 可静默调用（如 GetStatusAsync 高频检查）。**重载：** `EndOperateAsync(OperateResult result, token)` 传递已有结果 |
| `EndOperate(...)` | 同步版本（同步封装方法使用） |
| `Instance(basics)` | 单例模式获取实例 |
| `CreateInstance(basics)` | 创建新实例 |
| `GetParam()` | 获取配置参数（自动反射读取所有属性） |
| `GetSource<T>()` | 从 OperateResult 获取泛型结果数据 |
| `WAOn(WAModel)` | 启动内置 WebApi |
| `WAOff()` | 停止内置 WebApi |
| `LogOperateGet()` | 获取日志配置 |

### 6.1 进程缓存 ProcessCacheOperate

> 命名空间: `Snet.Core.cache.process`
> 进程内内存缓存，基于 `MemoryCache`，线程安全，自动过期。
> 适用于：插件内数据暂存、采集结果缓存、热点数据加速。

```csharp
using Snet.Core.cache.process;

// 创建（传入配置，可设过期时间）
ProcessCacheOperate cache = new ProcessCacheOperate(new ProcessCacheData
{
    AbsoluteExpiration = 60,   // 绝对过期（分钟），从写入算起
    SlidingExpiration = 20,    // 滑动过期（分钟），每次读取刷新
    Priority = CacheItemPriority.Normal
});

// 写入
var setResult = cache.SetCache("lastTemperature", 25.6f);

// 读取
var getResult = cache.GetCache<float>("lastTemperature");
if (getResult.Status)
{
    float temp = getResult.GetSource<float>();
}

// 删除
var removeResult = cache.RemoveCache("lastTemperature");

// 清空全部
var clearResult = cache.ClearCache();

// 释放（清空缓存 + 释放资源）
cache.Dispose();
```

**API 速查：**

| 方法 | 说明 | 返回 |
|------|------|------|
| `SetCache<T>(key, value)` | 写入/覆盖缓存 | `OperateResult` |
| `GetCache<T>(key)` | 读取缓存，`GetSource<T>()` 取值 | `OperateResult` |
| `RemoveCache(key)` | 删除指定缓存 | `OperateResult` |
| `ClearCache()` | 清空所有缓存 | `OperateResult` |
| `Dispose()` | 清空并释放 | `void` |

### 6.2 共享缓存 ShareCacheOperate

> 命名空间: `Snet.Core.cache.share`
> 跨进程共享内存缓存，基于 `MemoryMappedFile` + 全局 Mutex。
> 适用于：多个插件进程间数据共享、跨进程热数据交换。

```csharp
using Snet.Core.cache.share;

// 创建（指定缓存文件路径和大小）
ShareCacheOperate share = new ShareCacheOperate(new ShareCacheData
{
    Path = System.IO.Path.GetTempPath(),  // 缓存文件目录
    Name = "MyPlugin-Cache.dat",          // 缓存文件名
    MapName = "MyPlugin-Cache",            // 内存映射名称（跨进程标识）
    WholeSize = 1024 * 1024 * 10,         // 总大小 10MB
    IndexSize = 1024 * 1024 / 2,          // 索引区大小 512KB
});

// 写入
byte[] data = System.Text.Encoding.UTF8.GetBytes("共享数据");
share.SetCache("sharedKey", data);

// 读取
var getResult = share.GetCache("sharedKey");
if (getResult.Status)
{
    byte[]? bytes = getResult.GetSource<byte[]>();
}

// 删除 / 清空
share.RemoveCache("sharedKey");
share.ClearCache();

// 释放
share.Dispose();
```

**API 速查：**

| 方法 | 说明 |
|------|------|
| `SetCache(key, byte[])` | 写入字节数据到共享内存 |
| `GetCache(key)` | 读取共享内存中的字节数据 |
| `RemoveCache(key)` | 删除指定缓存项 |
| `ClearCache()` | 清空所有共享缓存 |
| `Dispose()` | 释放内存映射文件 |

**选择指南：**

| 需求 | 选择 |
|------|------|
| 插件内部缓存（单进程） | `ProcessCacheOperate` |
| 多插件进程间数据共享 | `ShareCacheOperate` |

### 6.3 字节处理 BytesHandler / BytesTransformHandler

> 命名空间: `Snet.Core.handler`
> 将原始字节数组自动转换为 `ConcurrentDictionary<string, AddressValue>`，支持 14 种数据类型和字节序转换。
> 适用于：协议响应为字节流，需要按配置自动解析为结构化数据。

```csharp
using Snet.Core.handler;
using static Snet.Core.handler.BytesHandler;

// BytesHandler 自动根据 AddressDetails 中的配置（DataType, Length, DataFormat）解析字节
// 在 Read 方法中使用：
byte[] response = /* 从设备收到的原始字节 */;
Address address = /* 用户配置的地址 */;

// BytesHandler 内部会遍历 address.AddressArray，按每个 item 的配置解析 response
// 返回 ConcurrentDictionary<string, AddressValue>
// 注意：BytesHandler.Execute 是同步处理器，在 ReadAsync 异步方法内部同步调用
var result = BytesHandler.Execute(address, response);
```

**BytesTransformHandler** 提供底层的字节↔值转换：
- 支持 ABCD/BADC/CDAB/DCBA 四种字节序
- 覆盖 Bool/Byte/Int16/UInt16/Int32/UInt32/Int64/UInt64/Single/Double/String 全部类型
- 自动处理数组类型的字节切分和重组

### 6.4 数据通道 ChannelOperate

> 命名空间: `Snet.Core.channel`
> 基于 `System.Threading.Channels` 的高性能异步数据管道，支持背压控制。
> 适用于：生产者-消费者模式、数据缓冲队列、异步流水线处理。

```csharp
using Snet.Core.channel;

// 创建通道（支持泛型）
ChannelOperate<AddressValue> channel = new ChannelOperate<AddressValue>(new ChannelData
{
    IsSync = false,  // false=后台自动消费并触发事件, true=手动调用 ReadAsync
});

// 写入数据
channel.TryWrite(new AddressValue { ... });

// 同步模式下手动读取
var result = await channel.ReadAsync(token);

// 通过事件消费（异步模式）
channel.OnDataEvent += (sender, e) =>
{
    var item = e.GetSource<AddressValue>();
    // 处理数据...
};

// 释放
channel.Dispose();
```

### 6.5 反射 ReflectionOperate

> 命名空间: `Snet.Core.reflection`
> 动态加载 DLL、调用方法、注册事件，用于运行时扩展能力。
> 适用于：插件加载外部解析库、动态调用用户自定义函数、数据二次加工。

```csharp
using Snet.Core.reflection;
using static Snet.Core.reflection.ReflectionData;

// ① 构建配置：指定 DLL、命名空间、类、方法
var basics = new Basics
{
    DllDatas = new List<DllData>
    {
        new DllData
        {
            DllPath = "MyParser.dll",   // DLL 路径
            IsAbsolutePath = false,     // false=相对路径（从程序运行目录找）
            NamespaceDatas = new List<NamespaceData>
            {
                new NamespaceData
                {
                    Namespace = "MyParser",
                    ClassDatas = new List<ClassData>
                    {
                        new ClassData
                        {
                            SN = "parser-instance",
                            ClassName = "Parser",
                            MethodDatas = new List<MethodData>
                            {
                                new MethodData
                                {
                                    SN = "parse-temp",
                                    MethodName = "ParseTemperature",
                                    WhetherExecute = false,
                                }
                            },
                            EventDatas = new List<EventData>
                            {
                                new EventData
                                {
                                    SN = "on-data",
                                    EventName = "OnDataChanged"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
};

// ② 初始化
var reflect = new ReflectionOperate(basics);
var initResult = reflect.Init();
if (!initResult.Status)
{
    LogHelper.Error($"反射初始化失败: {initResult.Message}");
    return;
}

// ③ 调用方法（返回 object?，不是 OperateResult）
object? methodResult = reflect.ExecuteMethod("parse-temp", new object[] { "25.6" });
if (methodResult != null)
{
    // 结果可能是 OperateResult 或直接值，需判断类型
    if (methodResult is OperateResult op && op.Status)
        object? parsed = op.GetSource<object>();
    else
        object? parsed = methodResult;  // 直接返回值
}

// ④ 注册事件（签名：RegisterEvent(SN, Register, P1?, P2?, ..., P6?)）
// Register=true 注册，Register=false 移除；P1~P6 对应 1~6 个参数的 Action
reflect.RegisterEvent("on-data", true,
    P1: (sender) =>
    {
        LogHelper.Info($"反射事件触发: {sender}");
    });

// ⑤ 释放
reflect.Dispose();
```

**配置结构：**

```
ReflectionData.Basics
└─ DllDatas: List<DllData>           # 可同时加载多个 DLL
   ├─ DllPath: string                # DLL 路径
   ├─ IsAbsolutePath: bool           # true=绝对路径, false=相对路径
   └─ NamespaceDatas: List<NamespaceData>
      ├─ Namespace: string           # 命名空间
      └─ ClassDatas: List<ClassData>
         ├─ SN: string               # 实例标识（用于调用时引用）
         ├─ ClassName: string        # 类名
         ├─ ConstructorParam: object[]?  # 构造函数参数（null=无参）
         ├─ MethodDatas: List<MethodData>    # 要加载的方法
         │   ├─ SN: string           # 调用标识
         │   ├─ MethodName: string   # 方法名
         │   └─ WhetherExecute: bool # 初始化时是否自动执行
         └─ EventDatas: List<EventData>      # 要注册的事件
             ├─ SN: string           # 注册标识
             └─ EventName: string    # 事件名
```

**API 速查：**

| 方法 | 说明 |
|------|------|
| `Init()` | 加载 DLL，实例化类，绑定方法/事件 |
| `ExecuteMethod(sn, params)` | 调用方法（sn = MethodData.SN），返回 `object?`（可能是 OperateResult 或直接值） |
| `RegisterEvent(sn, register, P1?, P2?, ..., P6?)` | 注册(`true`)/注销(`false`) 事件处理器（sn = EventData.SN），P1~P6 对应 1~6 个参数的 Action |
| `GetStatus()` | 反射是否已初始化（返回 `bool`，非 `OperateResult`） |
| `ReflectionInstance(sn)` | 获取指定 SN 的实例对象 |
| `GetMethod(sn?)` | 获取所有方法或指定 SN 的方法 |
| `GetEvent(sn?)` | 获取所有事件或指定 SN 的事件 |
| `Dispose()` | 释放资源 |

**在 Read 方法中使用反射：**

```csharp
// Read 中调用反射方法做数据二次加工
object? rawValue = /* 原始采集值 */;
if (reflect.GetStatus())
{
    object? result = reflect.ExecuteMethod("parse-temp", new object[] { rawValue });
    if (result is OperateResult op && op.Status)
        rawValue = op.GetSource<object>();  // 用解析后的值替换原始值
    else if (result != null)
        rawValue = result;                   // 直接返回值
}
// 然后交给框架
var av = AddressHandler.ExecuteDispose(item, rawValue, "成功");
```

---

## 7. 完整模板（AI 填写采集逻辑）

> **异步架构说明：** DaqAbstract 的 8 个抽象方法现全部为异步（`Task<OperateResult>` + `CancellationToken`）。
> 同步方法（`On()`, `Off()`, `Read()`, `Write()`, `Subscribe()`, `UnSubscribe()`, `GetStatus()`, `GetBaseObject()`）
> 是基类中的薄封装 `=> XxxAsync().GetAwaiter().GetResult()`，从基类自动继承，**插件无需实现**。

```csharp
using Snet.Core.@abstract;
using Snet.Core.handler;
using Snet.Core.subscription;
using Snet.Core.virtualAddress;
using Snet.Core.cache.process;    // 按需：进程缓存
using Snet.Core.cache.share;      // 按需：跨进程共享缓存
using Snet.Core.reflection;       // 按需：反射调用外部 DLL
using Snet.Log;
using Snet.Model.data;
using Snet.Model.@enum;
using Snet.Model.@interface;
using Snet.Utility;
using System.Collections.Concurrent;
using static XxxNamespace.XxxData;
using static Snet.Model.data.ParamModel;
using static Snet.Core.reflection.ReflectionData;  // 按需：反射配置

namespace XxxNamespace
{
    public class XxxOperate : DaqAbstract<XxxOperate, Basics>, IDaq
    {
        // ═══ 3 个必须属性 ═══
        protected override string CN => "【AI 填写中文名称】";
        protected override string CD => "【AI 填写中文描述】";
        protected override List<propertie> AP => new List<propertie>
        {
            new propertie { PropertyName = "ServiceName", Description = "命名空间", Default = this.GetType().FullName }
        };

        // ═══ 构造函数 ═══
        public XxxOperate() : base() { LanguageHandler(); }
        public XxxOperate(Basics basics) : base(basics) { LanguageHandler(); }

        // ═══ 内部字段 ═══
        private SubscribeOperate? subscribeOperate;
        private VirtualAddressManage VAM = new();
        private CancellationTokenSource? subscribeToken;
        private ProcessCacheOperate? cache;    // 按需：进程缓存
        private ReflectionOperate? reflect;    // 按需：反射

        // ═══ OnAsync — 必须 try/catch，失败调用 OffAsync(true) ═══
        public override async Task<OperateResult> OnAsync(CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                var status = await GetStatusAsync(token);
                if (status.GetDetails(out string? message))
                    return await EndOperateAsync(false, message, token);

                // 【AI 在此实现连接逻辑】

                return await EndOperateAsync(true, token: token);
            }
            catch (Exception ex)
            {
                await OffAsync(true, token);
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }

        // ═══ OffAsync — 必须 try/catch ═══
        public override async Task<OperateResult> OffAsync(bool hardClose = false, CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                if (!hardClose)
                {
                    var status = await GetStatusAsync(token);
                    if (!status.GetDetails(out string? message))
                        return await EndOperateAsync(false, message, token);
                }

                subscribeToken?.Cancel(); subscribeToken?.Dispose();
                if (subscribeOperate != null)
                {
                    await subscribeOperate.OffAsync(token);
                    subscribeOperate = null;
                }
                VAM?.Dispose();

                // 【AI 在此实现断开逻辑】

                return await EndOperateAsync(true, token: token);
            }
            catch (Exception ex)
            {
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }

        // ═══ GetStatusAsync — 禁止 try/catch，consoleOutput: false ═══
        public override async Task<OperateResult> GetStatusAsync(CancellationToken token = default)
        {
            await BegOperateAsync(token);
            // 【AI 判断连接状态】
            return await EndOperateAsync(/* 已连接? */, consoleOutput: false, token: token);
        }

        // ═══ GetBaseObjectAsync — 禁止 try/catch，必须先 GetStatusAsync ═══
        public override async Task<OperateResult> GetBaseObjectAsync(CancellationToken token = default)
        {
            await BegOperateAsync(token);
            var status = await GetStatusAsync(token);
            if (!status.GetDetails(out string? message))
                return await EndOperateAsync(false, message, token);

            return await EndOperateAsync(true, resultData: /* 【AI 返回底层对象】 */, token: token);
        }

        // ═══ ReadAsync — 必须 try/catch ═══
        public override async Task<OperateResult> ReadAsync(Address address, CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                var status = await GetStatusAsync(token);
                if (!status.GetDetails(out string? message))
                    return await EndOperateAsync(false, message, token);

                if (!address.CheckAddress())
                    return await EndOperateAsync(false, "存在无效点位数据，操作失败", token);

                ConcurrentDictionary<string, AddressValue> param = new();
                foreach (var item in address.AddressArray)
                {
                    if (!item.IsEnable) continue;

                    bool IsVA = false;
                    VAM.InitVirtualAddress(item, out IsVA);

                    object? value = null;
                    string resultMsg = "失败";

                    if (IsVA)
                    {
                        value = VAM.Read(item);
                    }
                    else
                    {
                        // ═══ 【AI 在此实现采集逻辑】 ═══
                        // item.AddressName  = 用户配置的地址
                        // item.AddressDataType = 用户配置的数据类型
                        // 返回采集到的原始值 value，成功时设 resultMsg = "成功"
                    }

                    AddressValue? av = AddressHandler.ExecuteDispose(item, value, resultMsg);
                    if (av != null)
                        param.AddOrUpdate(item.AddressName, av, (k, v) => av);
                }
                return param.Count > 0
                    ? await EndOperateAsync(true, resultData: param, token: token)
                    : await EndOperateAsync(false, "读取失败", token);
            }
            catch (Exception ex)
            {
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }

        // ═══ WriteAsync — 必须 try/catch ═══
        public override async Task<OperateResult> WriteAsync(
            ConcurrentDictionary<string, (object value, EncodingType? encodingType)> values,
            CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                var status = await GetStatusAsync(token);
                if (!status.GetDetails(out string? message))
                    return await EndOperateAsync(false, message, token);

                foreach (var (addressName, (value, encoding)) in values)
                {
                    if (VAM.IsVirtualAddress(addressName))
                    {
                        VAM.Write(addressName, value);
                        continue;
                    }
                    // 【AI 在此实现写入逻辑】
                }
                return await EndOperateAsync(true, token: token);
            }
            catch (Exception ex)
            {
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }

        // ═══ SubscribeAsync — 必须 try/catch ═══
        public override async Task<OperateResult> SubscribeAsync(Address address, CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                var status = await GetStatusAsync(token);
                if (!status.GetDetails(out string? message))
                    return await EndOperateAsync(false, message, token);

                if (!address.CheckAddress())
                    return await EndOperateAsync(false, "存在无效点位数据，操作失败", token);

                subscribeToken = new CancellationTokenSource();
                subscribeOperate = SubscribeOperate.InstanceAsync(basics);
                await subscribeOperate.OnAsync(token);

                _ = Task.Run(async () =>
                {
                    while (!subscribeToken.Token.IsCancellationRequested)
                    {
                        OperateResult r = await ReadAsync(address, subscribeToken.Token);
                        if (r.Status)
                        {
                            var e = new EventDataResult(true, "订阅数据", r.ResultData);
                            OnDataEvent?.Invoke(this, e);
                            if (OnDataEventAsync != null)
                                await OnDataEventAsync.Invoke(this, e);
                        }
                        await Task.Delay(
                            basics.HandleInterval > 0 ? basics.HandleInterval : 1000,
                            subscribeToken.Token);
                    }
                }, subscribeToken.Token);

                return await EndOperateAsync(true, token: token);
            }
            catch (Exception ex)
            {
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }

        // ═══ UnSubscribeAsync — 必须 try/catch ═══
        public override async Task<OperateResult> UnSubscribeAsync(Address address, CancellationToken token = default)
        {
            await BegOperateAsync(token);
            try
            {
                var status = await GetStatusAsync(token);
                if (!status.GetDetails(out string? message))
                    return await EndOperateAsync(false, message, token);

                subscribeToken?.Cancel();
                if (subscribeOperate != null)
                {
                    await subscribeOperate.OffAsync(token);
                    subscribeOperate = null;
                }
                return await EndOperateAsync(true, token: token);
            }
            catch (Exception ex)
            {
                return await EndOperateAsync(false, ex.Message, exception: ex, token: token);
            }
        }
    }
}
```

---

## 8. 打包为 Daq ZIP 插件

```bash
dotnet publish -c Release -o ./publish
Compress-Archive -Path ./publish/* -DestinationPath MyPlugin.zip
# → Daq 工具 → 插件设置 → 上传 ZIP
```

---

## 9. 内置通信类（TCP/UDP/WebSocket/Serial/HTTP）

Snet.Core 提供 5 种内置通信类，实现了 `ICommunication` 接口。**开发插件时应使用这些类，不要自己写 Socket。**

### 9.1 统一接口

```csharp
// 所有通信类共享的接口
public interface ICommunication : IOn, IOff, ISend, ISendWait, IGetObject, IGetStatus, IEvent, ...
```

| 方法 | 说明 | 适用 |
|------|------|------|
| `On()` / `Off(bool)` | 打开/关闭连接（同步封装） | 全部 |
| `OnAsync(token)` / `OffAsync(bool, token)` | 打开/关闭连接（异步） | 全部 |
| `GetStatus()` | 连接状态 | 全部 |
| `GetBaseObject()` | 底层对象 | 全部 |
| `Send(byte[])` | 发送字节（自动分包） | TCP/Serial/UDP/WS |
| `SendWaitAsync(byte[], CancellationToken)` | 发送并等待响应（异步） | TCP/Serial/UDP/WS |
| `OnDataEvent` / `OnDataEventAsync` | 收到数据触发 → `e.GetSource<byte[]>()` | TCP/Serial/UDP/WS |
| `OnInfoEvent` | 状态变化触发 | 全部 |

### 9.2 通信类速查

| 类 | 命名空间 | Config | 连接参数 | 特殊属性 |
|----|---------|--------|----------|----------|
| `TcpClientOperate` | `Snet.Core.communication.net.tcp.client` | `TcpClientData.Basics` | `IpAddress`, `Port`, `Timeout` | `InterruptReconnection`, `ReconnectionInterval`, `SendWaitInterval`, `MaxChunkSize`, `RetrySendCount`, `BufferSize` |
| `TcpServiceOperate` | `Snet.Core.communication.net.tcp.service` | `TcpServiceData.Basics` | `IpAddress`, `Port` | 多客户端管理，`Send(byte[], string[])` 定向/广播发送 |
| `UdpOperate` | `Snet.Core.communication.net.udp` | `UdpData.Basics` | `IpAddress`, `Port` | 支持广播模式、远程定向模式 |
| `SerialOperate` | `Snet.Core.communication.serial` | `SerialData.Basics` | `PortName`, `BaudRate`, `ParityBit`, `DataBit`, `StopBit` | `WriteTimeout`, `ReadTimeout`, `ReceivedBytesThreshold`, `SendWaitInterval`, `MaxChunkSize`, `RetrySendCount`, `BufferSize` |
| `WsClientOperate` | `Snet.Core.communication.net.ws.client` | `WsClientData.Basics` | `Url`（`"ws://IP:Port/path"`） | 支持断线重连 |
| `WsServiceOperate` | `Snet.Core.communication.net.ws.service` | `WsServiceData.Basics` | `Port` | WebSocket 服务端，管理多客户端连接 |
| `HttpClientOperate` | `Snet.Core.communication.net.http.client` | `HttpClientData.Basics` | 无连接，每次 `RequestAsync(RequestData, token)` | 支持 FormData/Raw/Cookie/Proxy |
| `HttpServiceOperate` | `Snet.Core.communication.net.http.service` | `HttpServiceData.Basics` | `Port` | REST API 服务端，支持 CORS、路由注册 |

> **注意：** `SerialData.Basics` 使用独立属性（`PortName`/`BaudRate`/`ParityBit`/`DataBit`/`StopBit`），不是 `SerialPortInfo` 字符串格式。串口驱动库（如 Snet.Modbus 的 ModbusRtu）使用 `SerialPortInfo` 字符串（`"COM3-9600-8-N-1"`），但内置通信类 `SerialOperate` 使用独立属性。

### 9.3 使用模板（TCP/Serial/UDP/WebSocket）

```csharp
using Snet.Core.communication.net.tcp.client;  // 按需替换命名空间

public class MyPluginOperate : DaqAbstract<MyPluginOperate, MyPluginData.Basics>, IDaq
{
    private TcpClientOperate? comm;  // ← 用内置通信类

    public override async Task<OperateResult> OnAsync(CancellationToken token = default)
    {
        await BegOperateAsync(token);
        comm = new TcpClientOperate(new TcpClientData.Basics
        {
            IpAddress = basics.IpAddress, Port = basics.Port,
            Timeout = basics.ConnectTimeOut,
            InterruptReconnection = true,  // 自动断线重连
        });
        // 事件驱动接收（可选，也可用 SendWaitAsync）
        comm.OnDataEventAsync += async (_, e) =>
        {
            byte[] received = e.GetSource<byte[]>();
            // AI 缓存/处理接收数据
        };
        return await comm.OnAsync(token);
    }

    public override async Task<OperateResult> OffAsync(bool hardClose = false, CancellationToken token = default)
    {
        if (comm != null)
            return await comm.OffAsync(hardClose, token);
        return await EndOperateAsync(true, token: token);
    }

    public override async Task<OperateResult> GetStatusAsync(CancellationToken token = default)
    {
        if (comm != null)
            return await comm.GetStatusAsync(token);
        return await EndOperateAsync(false, token: token);
    }

    public override async Task<OperateResult> GetBaseObjectAsync(CancellationToken token = default)
        => await EndOperateAsync(true, resultData: comm, token: token);

    public override async Task<OperateResult> ReadAsync(Address address, CancellationToken token = default)
    {
        await BegOperateAsync(token);
        var status = await GetStatusAsync(token);
        if (!status.GetDetails(out var msg)) return await EndOperateAsync(false, msg, token);

        ConcurrentDictionary<string, AddressValue> param = new();
        foreach (var item in address.AddressArray)
        {
            if (!item.IsEnable) continue;

            // 1. 构建请求帧
            byte[] request = /* AI 构建 */;

            // 2. 发送并等待响应
            using var cts = new CancellationTokenSource(basics.ReceiveTimeOut);
            var recv = await comm.SendWaitAsync(request, cts.Token);
            if (!recv.Status) continue;  // SendWaitAsync 失败跳过
            byte[] response = recv.GetSource<byte[]>();

            // 3. 解析响应
            object? value = /* AI 从 response 解析 */;

            // 4. 交给框架
            var av = AddressHandler.ExecuteDispose(item, value, "成功");
            if (av != null) param.AddOrUpdate(item.AddressName, av, (k, v) => av);
        }
        return param.Count > 0
            ? await EndOperateAsync(true, resultData: param, token: token)
            : await EndOperateAsync(false, "读取失败", token);
    }

    // WriteAsync / SubscribeAsync 同上模式
}
```

### 9.4 HTTP 模式

```csharp
using Snet.Core.communication.net.http.client;
using static Snet.Core.communication.net.http.client.HttpClientData;

public override async Task<OperateResult> ReadAsync(Address address, CancellationToken token = default)
{
    await BegOperateAsync(token);
    using var http = new HttpClientOperate(new Basics());
    var result = await http.RequestAsync(new RequestData
    {
        Url = basics.ApiUrl, Method = "GET",
        BType = BType.Json,
    }, token);
    if (!result.Status) return await EndOperateAsync(false, result.Message, token);
    string json = result.GetSource<string>();
    // AI 解析 JSON → ExecuteDispose
}
```

---

## 10. 已有实现参考

| 插件 | 数据源 | Read 采集方式 | 通信层 |
|------|--------|---------------|--------|
| `Snet.Siemens` | TCP | 发送 S7 协议 → 解析响应 | Driver.PipeTcpNet |
| `Snet.Modbus` | TCP/UDP/串口 | 发送 Modbus 帧 → 解析 | Driver.PipeTcpNet |
| `Snet.Mitsubishi` | TCP/UDP/串口 | 发送 MELSEC 帧 → 解析 | Driver.PipeTcpNet |
| `Snet.Omron` | TCP/UDP/串口 | 发送 FINS/HostLink/CIP 帧 | Driver.PipeTcpNet |
| `Snet.Opc` | TCP/COM | UA 二进制协议 / DA COM 调用 | OPC Foundation SDK |
| `Snet.AllenBradley` | TCP/串口 | 发送 CIP/PCCC/DF1 帧 | Driver.PipeTcpNet |
| `Snet.Freedom` | TCP/UDP/串口 | 发送自定义报文 → 偏移解析 | Driver.PipeTcpNet |
| `Snet.DB` | 数据库 | 执行 SQL → JSON/Dapper | Dapper/SqlSugar |
| `Snet.Sim` | 内存 | 虚拟地址随机/序列 | 无 |
| `Snet.TEP` | TCP | 私有协议握手→认证→数据流 | 内置 TcpServiceOperate |
| `Snet.PQDIF` | TCP/串口 | DLT645/DLT698/CJT188 帧 | Driver.PipeTcpNet |
| `Snet.Mqtt` | TCP | MQTT 协议（Client/Service/WS） | MQTTnet |
| `Snet.Kafka` | TCP | Kafka 协议（Producer/Consumer） | Confluent.Kafka |
