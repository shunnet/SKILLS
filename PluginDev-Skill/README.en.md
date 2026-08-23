# PluginDev-Skill — Snet Daq Plugin Development Contract (IDaq + IMq)

**Version:** 1.0.1.4  
**Author:** Shun  
**License:** MIT  
**Framework:** .NET 10.0

🌐 [中文 README](README.md) | [English README](README.en.md)

## Core Principles

> **What to acquire and how, what to send and how — the AI decides freely.**
> **How data is wrapped and what methods return — a hard contract that must be followed.**

**This skill covers two plugin types:**
- **IDAq plugins** (SKILL.md chapters 1–7): data acquisition — extend `DaqAbstract<O,D>` and implement 8 abstract methods
- **IMq plugins** (SKILL.md chapter 8): messaging middleware — extend `MqAbstract<O,D>` and implement 6 abstract methods

## Interaction Flow

> The user speaks in plain language; the AI asks questions one by one and generates code after confirmation.

**IDAq plugins (data acquisition):**

| # | What the AI asks | How the user answers | What the AI maps it to |
|---|-----------|---------------|--------------|
| 1 | What is the device? | "Temperature/humidity sensor" | → communication method |
| 2 | How does it connect? | "Ethernet" "Serial" | → TcpClient / Serial |
| 3 | What is the address? | "192.168.0.100" | → IpAddress + Port |
| 4 | What are the protocol rules? | "Send 01 03 00 00 00 02" | → request frame construction |
| 5 | What does the response look like? | "7 bytes, bytes 4–5 are temperature" | → response parsing logic |
| 6 | Need caching? | "No" | → ProcessCache |
| 7 | Need to load a DLL? | "No" | → Reflection |

**IMq plugins (messaging middleware):**

| # | What the AI asks | How the user answers | What the AI maps it to |
|---|-----------|---------------|--------------|
| 1 | What messaging system? | "Custom TCP message service" | → broker protocol |
| 2 | Broker address? | "192.168.0.1:6688" | → IpAddress + Port |
| 3 | Produce or consume? | "Both" | → Produce / Consume |
| 4 | Message format? | "JSON string" "Byte stream" | → Produce(string/byte[]) + ResponseType |

**After information gathering, output a confirmation summary and generate code only after the user confirms.**

## Plugin Architecture

```
IDAq plugin:
Your Operate class
  └─ DaqAbstract<O, D> (8 abstract methods that must be implemented)
       └─ CoreUnify<O, D> (auto-provided: singleton, events, logging, multi-language, args, WebAPI)

Your Data class
  └─ Basics extends SubscribeData.SCData (subscription fields: HandleInterval/ChangeOut/AllOut/TaskNumber)
       └─ ProtocolType enum + Attribute annotations

IMq plugin:
Your Mq Operate class
  └─ MqAbstract<O, D> (6 abstract methods that must be implemented)
       └─ CoreUnify<O, D> (same auto-provided features as above)

Your MqData class
  └─ Basics standalone class (no subscription fields, contains SN + connection parameters; no ProtocolType enum needed — that is an IDAQ concept)
```

## IDaq Plugin: 8 Abstract Methods to Implement

| Method | Parameters | Returns | Core constraints |
|------|------|------|----------|
| `OnAsync()` | — | `Task<OperateResult>` | First line `BegOperateAsync`; on failure catch → `OffAsync(true)` |
| `OffAsync(bool)` | hardClose | `Task<OperateResult>` | Release all resources |
| `GetStatusAsync()` | — | `Task<OperateResult>` | Status = connection state (no try/catch) |
| `GetBaseObjectAsync()` | — | `Task<OperateResult>` | ResultData = underlying object (no try/catch) |
| **`ReadAsync(Address)`** | address config | `Task<OperateResult>` | **ResultData = `ConcurrentDictionary<string, AddressValue>`** |
| `WriteAsync(dict)` | `(object, EncodingType?)` tuple dictionary | `Task<OperateResult>` | — |
| `SubscribeAsync(Address)` | address config | `Task<OperateResult>` | Subscription lifecycle via `SubscribeOperate` |
| `UnSubscribeAsync(Address)` | address config | `Task<OperateResult>` | Cancel token |

## IMq Plugin: 6 Abstract Methods to Implement

| Method | Parameters | Returns | Core constraints |
|------|------|------|----------|
| `OnAsync()` | — | `Task<OperateResult>` | Connect to broker; on failure catch → `OffAsync(true)` |
| `OffAsync(bool)` | hardClose | `Task<OperateResult>` | Release producer/consumer |
| `GetStatusAsync()` | — | `Task<OperateResult>` | Connection state (no try/catch) |
| **`ProduceAsync(topic, byte[])`** | topic + message | `Task<OperateResult>` | Publish message (string overload is a base-class virtual, may inherit) |
| **`ConsumeAsync(topic)`** | topic | `Task<OperateResult>` | Subscribe + push consumption data via `OnDataEventHandlerAsync` |
| `UnConsumeAsync(topic)` | topic | `Task<OperateResult>` | Cancel token, release consumer |

## Event Model

| Event | Trigger | Use case |
|------|----------|----------|
| `OnDataEvent` | Subscription data arrives | Synchronous processing |
| `OnDataEventAsync` | Subscription data arrives | Async I/O (consumption data push) |
| `OnInfoEvent` | Status change / alarm | Connection monitoring |

## Optional Built-in Components

| Component | Namespace | Purpose |
|------|---------|------|
| `ProcessCacheOperate` | `Snet.Core.cache.process` | In-process memory cache (auto-expiry, thread-safe) |
| `ShareCacheOperate` | `Snet.Core.cache.share` | Cross-process shared memory cache (MemoryMappedFile + Mutex) |
| `ReflectionOperate` | `Snet.Core.reflection` | Dynamically load DLLs, invoke methods, register events |
| `BytesHandler` | `Snet.Core.handler` | `TransformAsync` parses raw bytes into address values per BytesModel |
| `BytesTransformHandler` | `Snet.Core.handler` | Low-level byte↔value conversion (14 types + 4 byte orders) |
| `ChannelOperate` | `Snet.Core.channel` | High-performance async data pipeline (System.Threading.Channels) |

## The Single Data Flow for Read

```
Acquired raw data (method decided by the AI)
  → AddressHandler.ExecuteDispose(item, rawValue, message)
    → automatic: null detection + type conversion + reflection parsing + MQ forwarding
    → returns AddressValue
  → ConcurrentDictionary<string, AddressValue>
  → EndOperate(true, resultData: param)
```

## Attribute Annotations

| Attribute | Location | Purpose |
|-----------|------|------|
| `[Category][Description]` | Every Basics field | UI display |
| `[Display(use,show,mustFillIn,cate)]` | Every Basics field | UI control (parameter order: Use/Show/MustFillIn/DataCate) |
| `[AutoAllocatingTag(typeof(Enum))]` | ProtocolType property | Protocol marker |
| `[AutoAllocating(string[])]` | Every enum value | Parameter list |

## Packaging & Deployment

```bash
dotnet publish -c Release -o ./publish
Compress-Archive -Path ./publish/* -DestinationPath MyPlugin.zip
# → Daq tool → Plugin settings → Upload ZIP
```

> **MQ plugins also need configuration registration after deployment:** create `config/mq/{namespace}.{class name}.{SN}.Mq.Config.json` in the Daq tool's runtime directory (content = JSON of Basics); MqOperate loads it automatically (see SKILL.md §8.6).

## Reference Implementations

**IDAq plugins:** `Snet.Siemens` `Snet.Modbus` `Snet.Mitsubishi` `Snet.Omron` `Snet.Opc` `Snet.AllenBradley` `Snet.DB` `Snet.Sim` `Snet.Freedom` `Snet.TEP` `Snet.PQDIF`

**IMq plugins:** `Snet.Mqtt` `Snet.Kafka` `Snet.RabbitMQ` `Snet.RocketMQ` `Snet.NetMQ` `Snet.Netty`

> **📌 On versions:** all `dotnet add package` commands **must** pin `-v` with a version (current: 26.235.2).
> Get the latest version at `https://www.nuget.org/packages/<package-name>`, for example:
> - `https://www.nuget.org/packages/Snet.DB`
> - `https://www.nuget.org/packages/Snet.TEP`
