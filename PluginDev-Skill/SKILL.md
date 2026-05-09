---
name: plugindev-skill
description: Snet.Iot.Daq 插件开发技能。严格定义插件开发契约：必须实现的抽象方法、必须遵循的返回类型、必须使用的数据标注、必须调用的框架方法。AI 自行决定采集方式（TCP/HTTP/文件/串口），但必须遵守契约。
version: 1.0.0.1
metadata:
  openclaw:
    requires:
      bins: []
      dotnet: "8.0"
---

# PluginDev-Skill — Snet Daq 插件开发契约

## 核心原则

> **采集什么、用什么方式采集 —— 由 AI 根据用户描述自行决定。**
> **但数据如何封装、方法返回什么类型、生命周期如何管理 —— 这是硬性契约，必须严格遵守。**

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
            └─ 必须实现 8 个抽象方法 + 3 个属性
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

    // ============ 8 个抽象方法 ============
    public override OperateResult On() { ... }
    public override OperateResult Off(bool hardClose = false) { ... }
    public override OperateResult GetStatus() { ... }
    public override OperateResult GetBaseObject() { ... }
    public override OperateResult Read(Address address) { ... }
    public override OperateResult Write(ConcurrentDictionary<string, (object, EncodingType?)> values) { ... }
    public override OperateResult Subscribe(Address address) { ... }
    public override OperateResult UnSubscribe(Address address) { ... }
}
```

---

## 2. 方法契约详解

### 2.1 On() — 打开连接

| 约束 | 值 |
|------|-----|
| 返回类型 | `OperateResult` |
| 成功条件 | `Status = true` |
| 失败处理 | 内部调用 `Off(true)` 清理，返回失败 |
| 状态检查 | 必须先 `GetStatus().GetDetails(out msg)`，已连接返回失败 |
| 必须调用 | 第一行 `BegOperate()`，返回 `EndOperate(status, message?)` |

```csharp
public override OperateResult On()
{
    BegOperate();
    try
    {
        if (GetStatus().GetDetails(out string? msg))
            return EndOperate(false, msg);  // 已连接

        // AI 自行决定如何连接（TCP、文件、HTTP...）

        return EndOperate(true);
    }
    catch (Exception ex)
    {
        Off(true);
        return EndOperate(false, ex.Message, exception: ex);
    }
}
```

### 2.2 Off(bool hardClose) — 关闭连接

| 约束 | 值 |
|------|-----|
| 返回类型 | `OperateResult` |
| hardClose=false | 先检查 `GetStatus()`，未连接返回失败 |
| hardClose=true | 跳过状态检查，强制关闭 |
| 必须操作 | 释放所有资源（连接、订阅、FileSystemWatcher 等） |

```csharp
public override OperateResult Off(bool hardClose = false)
{
    BegOperate();
    try
    {
        if (!hardClose && !GetStatus().GetDetails(out string? msg))
            return EndOperate(false, msg);

        // AI 自行决定如何断开连接

        return EndOperate(true);
    }
    catch (Exception ex)
    {
        return EndOperate(false, ex.Message, exception: ex);
    }
}
```

### 2.3 GetStatus() — 获取状态

| 约束 | 值 |
|------|-----|
| 返回类型 | `OperateResult` |
| Status=true | 已连接 |
| Status=false | 未连接 |

```csharp
public override OperateResult GetStatus()
    => EndOperate(/* AI 判断连接状态 */);
```

### 2.4 GetBaseObject() — 获取底层对象

| 约束 | 值 |
|------|-----|
| 返回类型 | `OperateResult` |
| ResultData | 底层连接对象（TcpClient、HttpClient 等） |
| 说明 | 返回插件内部使用的通信类实例，如 `TcpClientOperate`、`SerialOperate` 等。供外部诊断/调试用 |

### 2.5 ⭐ Read(Address address) — 读取数据（核心）

| 约束 | 值 |
|------|-----|
| 入参 | `Address address` |
| 返回类型 | `OperateResult` |
| ResultData 类型 | **`ConcurrentDictionary<string, AddressValue>`** |
| 必调方法 | `address.CheckAddress()` 检查点位 |
| 每个点位处理 | `AddressHandler.ExecuteDispose(item, rawValue, message)` |
| 跳过禁用 | `if (!item.IsEnable) continue;` |
| 虚拟地址 | `VAM.InitVirtualAddress(item, out bool IsVA)` → 区分虚实 |
| 成功返回 | `EndOperate(true, resultData: param)` |
| 空结果返回 | `EndOperate(false, "读取失败")` |

**Read 方法的唯一职责：采集原始数据 → 交给 ExecuteDispose 处理 → 返回字典**

```csharp
public override OperateResult Read(Address address)
{
    BegOperate();
    try
    {
        if (!GetStatus().GetDetails(out string? msg))
            return EndOperate(false, msg);

        if (!address.CheckAddress())
            return EndOperate(false, "存在无效点位数据，操作失败");

        ConcurrentDictionary<string, AddressValue> param = new();

        foreach (var item in address.AddressArray)
        {
            if (!item.IsEnable) continue;

            // 判断是否虚拟地址
            bool IsVA = false;
            VAM.InitVirtualAddress(item, out IsVA);

            object? value = null;
            string resultMsg = "失败";

            if (IsVA)
            {
                value = VAM.Read(item);  // 虚拟地址读
            }
            else
            {
                // ═══ AI 在这里自行实现采集逻辑 ═══
                // 采集什么、怎么采集——AI 决定
                // 示例：TCP 发命令收响应 / HTTP GET / 读文件 / 查数据库
                value = /* AI 自行实现的采集 */;
                resultMsg = value != null ? "成功" : "读取失败";
            }

            // ═══ 必须调用！框架处理类型转换+解析+转发 ═══
            AddressValue? av = AddressHandler.ExecuteDispose(item, value, resultMsg);
            if (av != null)
                param.AddOrUpdate(item.AddressName, av, (k, v) => av);
        }

        return param.Count > 0
            ? EndOperate(true, resultData: param)
            : EndOperate(false, "读取失败");
    }
    catch (Exception ex)
    {
        return EndOperate(false, exception: ex);
    }
}
```

**ExecuteDispose 自动完成：**

| 步骤 | 说明 |
|------|------|
| 空值检测 | `value` 为空 → `QualityType.Exception` |
| 类型转换 | 根据 `AddressDataType` 自动转换 `string→int/float/bool...` |
| 反射解析 | 如果 `AddressParseParam` 不为空，调用反射方法二次加工 |
| MQ转发 | 如果 `AddressMqParam` 不为空，Normal/ParseUnknown 时触发 `Produce` |
| 质量标记 | Quality = None / Exception / Normal / DataTypeError / ParseUnknown / ParseError |
| 返回 | 返回完整的 `AddressValue` |

### 2.6 Write(values) — 写入数据

| 约束 | 值 |
|------|-----|
| 入参 | `ConcurrentDictionary<string, (object value, EncodingType? encodingType)>` |
| 返回类型 | `OperateResult` |
| Key | 地址名（AddressName） |
| Value | 元组：(写入的值, 编码类型) |

```csharp
public override OperateResult Write(ConcurrentDictionary<string, (object value, EncodingType? encodingType)> values)
{
    BegOperate();
    try
    {
        if (!GetStatus().GetDetails(out string? msg))
            return EndOperate(false, msg);

        foreach (var (addressName, (value, encoding)) in values)
        {
            bool IsVA = VAM.IsVirtualAddress(addressName);
            if (IsVA)
            {
                VAM.Write(addressName, value);
            }
            else
            {
                // AI 自行实现写入逻辑
            }
        }
        return EndOperate(true);
    }
    catch (Exception ex)
    {
        return EndOperate(false, exception: ex);
    }
}
```

### 2.7 Subscribe(Address address) — 订阅数据

| 约束 | 值 |
|------|-----|
| 必须做的事 | 创建 `SubscribeOperate` 管理订阅生命周期 |
| 数据事件 | 通过 `OnDataEvent?.Invoke(this, eventResult)` 推送 |
| 返回类型 | `OperateResult` |
| 实现方式 | AI 决定（轮询 Read / 事件驱动 / FileSystemWatcher） |

```csharp
public override OperateResult Subscribe(Address address)
{
    BegOperate();
    subscribeToken = new CancellationTokenSource();
    subscribeOperate = new SubscribeOperate();

    _ = Task.Run(async () =>
    {
        while (!subscribeToken.Token.IsCancellationRequested)
        {
            OperateResult r = Read(address);  // ← 重用 Read 方法
            if (r.Status)
            {
                var eventResult = new EventDataResult(true, "订阅数据", r.ResultData);
                OnDataEvent?.Invoke(this, eventResult);
                if (OnDataEventAsync != null)
                    await OnDataEventAsync.Invoke(this, eventResult);
            }
            await Task.Delay(basics.HandleInterval, subscribeToken.Token);
        }
    }, subscribeToken.Token);

    return EndOperate(true);
}
```

### 2.8 UnSubscribe(Address address) — 取消订阅

| 约束 | 值 |
|------|-----|
| 必须操作 | 取消订阅 Token，关闭 SubscribeOperate |
| 返回类型 | `OperateResult` |

---

## 3. 数据类契约

### 3.1 基础数据类

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
| `ChangeOut` | `bool` | false | 仅变化时输出 |
| `AllOut` | `bool` | true | 全量输出 |
| `TaskNumber` | `int` | 1 | 并行任务数 |

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

// 启动订阅时
subscribeOperate = new SubscribeOperate();

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
| `BegOperate()` | 每个 public 方法第一行调用，初始化操作上下文 |
| `EndOperate(status, message?, resultData?, exception?)` | 统一返回 OperateResult，自动记录耗时和日志 |
| `EndOperateAsync(status, ...)` | 异步版本 |
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

### 6.3 反射 ReflectionOperate

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

// ③ 调用方法
var methodResult = reflect.Invoke<object>("parse-temp", new object[] { "25.6" });
if (methodResult.Status)
{
    object? parsed = methodResult.GetSource<object>();
}

// ④ 注册事件
reflect.RegisterEvent("on-data", (sender, args) =>
{
    LogHelper.Info($"反射事件触发: {args}");
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
| `Invoke<T>(sn, params)` | 调用方法（sn = MethodData.SN） |
| `RegisterEvent(sn, handler)` | 注册事件处理器（sn = EventData.SN） |
| `GetStatus()` | 反射是否已初始化 |
| `Dispose()` | 释放资源 |

**在 Read 方法中使用反射：**

```csharp
// Read 中调用反射方法做数据二次加工
object? rawValue = /* 原始采集值 */;
if (reflect.GetStatus())
{
    var result = reflect.Invoke<object>("parse-temp", new object[] { rawValue });
    if (result.Status)
        rawValue = result.GetSource<object>();  // 用解析后的值替换原始值
}
// 然后交给框架
var av = AddressHandler.ExecuteDispose(item, rawValue, "成功");
```

---

## 7. 完整模板（AI 填写采集逻辑）

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

        // ═══ On ═══
        public override OperateResult On()
        {
            BegOperate();
            try
            {
                if (GetStatus().GetDetails(out string? msg))
                    return EndOperate(false, msg);

                // 【AI 在此实现连接逻辑】

                return EndOperate(true);
            }
            catch (Exception ex) { Off(true); return EndOperate(false, ex.Message, exception: ex); }
        }

        // ═══ Off ═══
        public override OperateResult Off(bool hardClose = false)
        {
            BegOperate();
            try
            {
                if (!hardClose && !GetStatus().GetDetails(out string? msg))
                    return EndOperate(false, msg);

                subscribeToken?.Cancel(); subscribeToken?.Dispose();
                subscribeOperate?.Off(); subscribeOperate = null;

                // 【AI 在此实现断开逻辑】

                return EndOperate(true);
            }
            catch (Exception ex) { return EndOperate(false, ex.Message, exception: ex); }
        }

        // ═══ GetStatus ═══
        public override OperateResult GetStatus()
            => EndOperate(/* 【AI 判断连接状态】 */);

        // ═══ GetBaseObject ═══
        public override OperateResult GetBaseObject()
            => EndOperate(true, resultData: /* 【AI 返回底层对象】 */);

        // ═══ Read ═══
        public override OperateResult Read(Address address)
        {
            BegOperate();
            try
            {
                if (!GetStatus().GetDetails(out string? msg))
                    return EndOperate(false, msg);
                if (!address.CheckAddress())
                    return EndOperate(false, "存在无效点位数据");

                ConcurrentDictionary<string, AddressValue> param = new();
                foreach (var item in address.AddressArray)
                {
                    if (!item.IsEnable) continue;

                    bool IsVA = false;
                    VAM.InitVirtualAddress(item, out IsVA);

                    object? value = null;
                    string resultMsg = "失败";

                    if (IsVA) { value = VAM.Read(item); }
                    else
                    {
                        // ═══ 【AI 在此实现采集逻辑】 ═══
                        // item.AddressName  = 用户配置的地址
                        // item.AddressDataType = 用户配置的数据类型
                        // 返回采集到的原始值 value
                    }

                    AddressValue? av = AddressHandler.ExecuteDispose(item, value, resultMsg);
                    if (av != null)
                        param.AddOrUpdate(item.AddressName, av, (k, v) => av);
                }
                return param.Count > 0 ? EndOperate(true, resultData: param) : EndOperate(false, "读取失败");
            }
            catch (Exception ex) { return EndOperate(false, exception: ex); }
        }

        // ═══ Write ═══
        public override OperateResult Write(ConcurrentDictionary<string, (object value, EncodingType? encodingType)> values)
        {
            BegOperate();
            try
            {
                if (!GetStatus().GetDetails(out string? msg)) return EndOperate(false, msg);
                foreach (var (key, (val, enc)) in values)
                {
                    if (VAM.IsVirtualAddress(key)) { VAM.Write(key, val); continue; }
                    // 【AI 在此实现写入逻辑】
                }
                return EndOperate(true);
            }
            catch (Exception ex) { return EndOperate(false, exception: ex); }
        }

        // ═══ Subscribe ═══
        public override OperateResult Subscribe(Address address)
        {
            BegOperate();
            subscribeToken = new CancellationTokenSource();
            subscribeOperate = new SubscribeOperate();
            _ = Task.Run(async () =>
            {
                while (!subscribeToken.Token.IsCancellationRequested)
                {
                    OperateResult r = Read(address);  // ← 复用 Read
                    if (r.Status)
                    {
                        var e = new EventDataResult(true, "订阅数据", r.ResultData);
                        OnDataEvent?.Invoke(this, e);
                        if (OnDataEventAsync != null) await OnDataEventAsync.Invoke(this, e);
                    }
                    await Task.Delay(basics.HandleInterval > 0 ? basics.HandleInterval : 1000, subscribeToken.Token);
                }
            }, subscribeToken.Token);
            return EndOperate(true);
        }

        // ═══ UnSubscribe ═══
        public override OperateResult UnSubscribe(Address address)
        {
            subscribeToken?.Cancel();
            subscribeOperate?.Off();
            return EndOperate(true);
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
| `On()` / `Off(bool)` | 打开/关闭连接 | 全部 |
| `GetStatus()` | 连接状态 | 全部 |
| `GetBaseObject()` | 底层对象 | 全部 |
| `Send(byte[])` | 发送字节（自动分包） | TCP/Serial/UDP/WS |
| `SendWait(byte[], CancellationToken)` | 发送并等待响应 | TCP/Serial/UDP/WS |
| `OnDataEvent` | 收到数据触发 → `e.GetSource<byte[]>()` | TCP/Serial/UDP/WS |
| `OnInfoEvent` | 状态变化触发 | 全部 |

### 9.2 通信类速查

| 类 | 命名空间 | Config | 连接参数 |
|----|---------|--------|----------|
| `TcpClientOperate` | `Snet.Core.communication.net.tcp.client` | `TcpClientData.Basics` | `IpAddress`, `Port`, `Timeout`, `InterruptReconnection` |
| `UdpOperate` | `Snet.Core.communication.net.udp` | `UdpData.Basics` | `IpAddress`, `Port` |
| `SerialOperate` | `Snet.Core.communication.serial` | `SerialData.Basics` | `SerialPortInfo`（`"COM3-9600-8-N-1"`） |
| `WsClientOperate` | `Snet.Core.communication.net.ws.client` | `WsClientData.Basics` | `Url`（`"ws://IP:Port/path"`） |
| `HttpClientOperate` | `Snet.Core.communication.net.http.client` | `HttpClientData.Basics` | 无连接，每次 `Request(RequestData)` |

### 9.3 使用模板（TCP/Serial/UDP/WebSocket）

```csharp
using Snet.Core.communication.net.tcp.client;  // 按需替换命名空间

public class MyPluginOperate : DaqAbstract<MyPluginOperate, MyPluginData.Basics>, IDaq
{
    private TcpClientOperate? comm;  // ← 用内置通信类

    public override OperateResult On()
    {
        BegOperate();
        comm = new TcpClientOperate(new TcpClientData.Basics
        {
            IpAddress = basics.IpAddress, Port = basics.Port,
            Timeout = basics.ConnectTimeOut,
            InterruptReconnection = true,  // 自动断线重连
        });
        // 事件驱动接收（可选，也可用 SendWait）
        comm.OnDataEvent += (_, e) =>
        {
            byte[] received = e.GetSource<byte[]>();
            // AI 缓存/处理接收数据
        };
        return comm.On();
    }

    public override OperateResult Off(bool hardClose = false)
        => comm?.Off(hardClose) ?? EndOperate(true);

    public override OperateResult GetStatus()
        => comm?.GetStatus() ?? EndOperate(false);

    public override OperateResult GetBaseObject()
        => EndOperate(true, resultData: comm);

    public override OperateResult Read(Address address)
    {
        BegOperate();
        if (!GetStatus().GetDetails(out var msg)) return EndOperate(false, msg);

        ConcurrentDictionary<string, AddressValue> param = new();
        foreach (var item in address.AddressArray)
        {
            if (!item.IsEnable) continue;

            // 1. 构建请求帧
            byte[] request = /* AI 构建 */;

            // 2. 发送并等待响应
            using var cts = new CancellationTokenSource(basics.ReceiveTimeOut);
            var recv = comm.SendWait(request, cts.Token);
            if (!recv.Status) continue;  // SendWait 失败跳过
            byte[] response = recv.GetSource<byte[]>();

            // 3. 解析响应
            object? value = /* AI 从 response 解析 */;

            // 4. 交给框架
            var av = AddressHandler.ExecuteDispose(item, value, "成功");
            if (av != null) param.AddOrUpdate(item.AddressName, av, (k, v) => av);
        }
        return param.Count > 0
            ? EndOperate(true, resultData: param)
            : EndOperate(false, "读取失败");
    }

    // Write / Subscribe 同上模式
}
```

### 9.4 HTTP 模式

```csharp
using Snet.Core.communication.net.http.client;
using static Snet.Core.communication.net.http.client.HttpClientData;

public override OperateResult Read(Address address)
{
    BegOperate();
    using var http = new HttpClientOperate(new Basics());
    var result = http.Request(new RequestData
    {
        Url = basics.ApiUrl, Method = "GET",
        BType = BType.Json,
    });
    if (!result.Status) return EndOperate(false, result.Message);
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
| `Snet.Freedom` | TCP/UDP/串口 | 发送自定义报文 → 偏移解析 | Driver.PipeTcpNet |
| `Snet.DB` | 数据库 | 执行 SQL → JSON | Dapper/SqlSugar |
| `Snet.Sim` | 内存 | 虚拟地址随机/序列 | 无 |
| `Snet.TEP` | TCP | 私有协议握手→认证→数据流 | 内置 TcpServiceOperate |
