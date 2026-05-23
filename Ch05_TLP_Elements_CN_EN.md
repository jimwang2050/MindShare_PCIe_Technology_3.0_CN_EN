# Chapter 5: TLP Elements
# 第5章：TLP元素

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 228–273 (46 pages)
> 来源：MindShare PCI Express 技术 3.0 | 页码：228–273（共46页）

---

## 快速导航 | Quick Navigation

- [Packet-Based Protocol — 基于包的协议](#introduction-to-packet-based-protocol)
- [TLP Structure — TLP结构](#tlp-structure)
- [Generic TLP Header — 通用TLP头](#generic-tlp-header-format)
- [ECRC / Digest — 可选的CRC](#digest--ecrc-field)
- [Byte Enables — 字节使能](#using-byte-enables)
- [Transaction Descriptor — 事务描述符](#transaction-descriptor-fields)
- [Specific TLP Formats — 特定TLP格式](#specific-tlp-formats-request--completion-tlps)
  - [IO / Memory / Config Requests](#io-requests)
  - [Completions](#completions)
  - [Messages](#message-requests)
- [术语附录 | Terminology Appendix](#术语附录-terminology-appendix)

---

## Introduction to Packet-Based Protocol
## 基于包的协议

Unlike parallel buses, serial transports like PCIe use no control signals to identify what's happening on the Link. Instead, the bit stream must have a recognizable format. Information moves across an active PCIe Link in fundamental chunks called **packets** comprised of symbols. Two major classes: high-level **TLPs** (Transaction Layer Packets) and low-level **DLLPs** (Data Link Layer Packets). Ordered Sets are packets too, but are replicated on all Lanes (not byte-striped).

> 不同于并行总线，串行传输如PCIe不使用控制信号识别链路上正在发生什么。相反，比特流必须具有可识别的格式。信息通过活跃PCIe链路以称为**包（packet）**的基本块传输，包由符号组成。两大类：高层**TLP**（事务层包）和低层**DLLP**（数据链路层包）。有序集也是包，但在所有Lane上复制（不按字节条带化）。

Benefits of packet-based protocol: (1) Packets have well-defined formats; (2) Framing symbols (STP/SDP/END) define packet boundaries; (3) CRC protects the entire packet.

> 基于包协议的好处：(1) 包有明确定义的格式；(2) 成帧符号（STP/SDP/END）定义包边界；(3) CRC保护整个包。

---

## TLP Structure
## TLP结构

A TLP consists of:
- **Header** (3 or 4 DWs): Transaction type, address/ID, length, attributes
- **Data Payload** (optional, 0–1024 DWs): The data being transferred
- **Digest** (optional, 1 DW): End-to-End CRC (ECRC)

The header defines the packet format and type via the **Fmt[1:0] and Type[4:0]** fields. These fields encode whether the TLP has data, the header size (3 or 4 DW), and the transaction type.

> TLP结构：**Header**（3或4 DW：事务类型、地址/ID、长度、属性）；**Data Payload**（可选，0–1024 DW：传输的数据）；**Digest**（可选，1 DW：端到端CRC/ECRC）。Header通过**Fmt[1:0]和Type[4:0]**字段定义包格式和类型。

---

## Generic TLP Header Format
## 通用TLP Header格式

Key header fields common to most TLPs:

| Field | Bits | Description | 描述 |
|-------|------|-------------|------|
| Fmt[1:0] | Byte0[6:5] | Header size: 00=3DW no data, 01=4DW no data, 10=3DW+data, 11=4DW+data | 头部大小+有无数据 |
| Type[4:0] | Byte0[4:0] | Transaction type (Memory Rd/Wr, Cfg Rd/Wr, IO, Msg, Cpl, etc.) | 事务类型 |
| TC[2:0] | Byte0[6:4]/B1 | Traffic Class (0–7) | 流量类别 |
| Attr[2:0] | Byte1[5:3] | Attributes (No Snoop, Relaxed Ordering, ID-based Ordering) | 事务属性 |
| Length[9:0] | Byte2/B3 | Data payload length in DW | 数据载荷长度（DW） |
| Requester ID[15:0] | Byte4/B5 | Bus[7:0], Device[4:0], Function[2:0] | 请求者BDF |
| Tag[7:0] | Byte6 | Transaction tag for matching Completions | 事务标签 |
| Last/First DW BE | Byte7 | Byte Enables for 1st and last DW of data | 首/末DW字节使能 |
| Address[31:2] or [63:2] | Byte8+ | Target address (memory or IO) | 目标地址 |

> Fmt+Type字段编码了TLP格式和类型。TC（Traffic Class）用于QoS。Attr（Attributes）包括No Snoop（非窥探）和Relaxed Ordering（宽松排序）提示。Requester ID标识TLP发起者。Tag用于匹配Non-Posted请求与相应的完成。Length以DW（双字，4字节）为单位。

---

## Digest / ECRC Field
## 摘要 / ECRC字段

The optional TLP Digest (1 DW) provides End-to-End CRC (ECRC) protection. ECRC is calculated and checked by the Transaction Layer (end-to-end), unlike LCRC which is per-Link. ECRC covers the entire TLP (header + data). Devices that support ECRC set the ECRC Check/Generate enables in the AER Capability.

> 可选的TLP摘要（1 DW）提供端到端CRC（ECRC）保护。ECRC由事务层计算和检查（端到端），不同于逐链路的LCRC。ECRC覆盖整个TLP（头+数据）。支持ECRC的设备在AER能力中设置ECRC检查/生成使能位。

## Byte Enables
## 字节使能

Byte Enables (BE) control which bytes in the first and last DW of a data payload are valid. Rules: BEs must be contiguous (no gaps); the First DW BE cannot be all-zero; for reads, the Last DW BE being all-zero indicates the last DW is empty.

> 字节使能（Byte Enable/BE）控制数据载荷中第一个和最后一个DW中哪些字节有效。规则：BE必须连续（无间隙）；First DW BE不能全为零；对于读取，Last DW BE全零表示最后一个DW为空。

---

## Transaction Descriptor Fields
## 事务描述符字段

The Transaction Descriptor comprises:
- **Transaction ID**: Requester ID (16-bit BDF) + Tag (8-bit) — uniquely identifies each outstanding Non-Posted request
- **Traffic Class (TC)**: 3-bit QoS label, maps to Virtual Channels
- **Transaction Attributes**: No Snoop, Relaxed Ordering, ID-based Ordering — optimization hints

> 事务描述符包括：Transaction ID（Requester ID 16位BDF + Tag 8位）唯一标识每个未完成的Non-Posted请求；Traffic Class（TC）3位QoS标签映射到虚拟通道；事务属性——优化提示。

---

## Specific TLP Formats: Request & Completion TLPs
## 特定TLP格式：请求与完成TLP

### IO Requests
### IO请求

IO Requests use 32-bit IO addressing. Header format: Fmt=00b, Type=00010b (IO Rd) or 00001b (IO Wr). IO space is legacy (discouraged in PCIe). Requires Completion for reads; writes are Non-Posted unlike Memory Writes.

> IO请求使用32位IO寻址。Header：Fmt=00b, Type=00010b (IO读) 或 00001b (IO写)。IO空间为传统（PCIe不鼓励）。读需要完成；写也是Non-Posted（不同于Memory Write）。

### Memory Requests
### 内存请求

Memory Requests: Fmt=00b/01b (32/64-bit addr), Type=00000b (Rd) or 00001b (Wr). Memory Writes are Posted (no Completion) — most efficient for bulk data. Memory Reads are Non-Posted — Completion(s) return data. Supports up to 1024 DW payload (4KB). Address[1:0] bits determine address type: 00b=memory, other=Reserved.

> 内存请求：Fmt=00b/01b（32/64位地址），Type=00000b（读）或00001b（写）。内存写是Posted（无完成）——大块数据最高效。内存读是Non-Posted——完成返回数据。支持最多1024 DW载荷（4KB）。Address[1:0]确定地址类型：00b=memory。

### Configuration Requests
### 配置请求

Configuration Requests: Fmt=00b, Type=00100b (Type 0 Rd), 00101b (Type 1 Rd), 00100b (Type 0 Wr), 00101b (Type 1 Wr). Requester ID identifies the initiator. Target identified by Bus/Device/Function numbers and Register Number (byte offset within config space). The four Register Number bits are shifted left 2 to form byte-aligned addresses.

> 配置请求：Fmt=00b。Type 0针对同总线功能，Type 1针对下游总线功能。Requester ID标识发起者。目标由Bus/Device/Function编号和Register Number（配置空间内字节偏移）标识。4位Register Number左移2位构成字节对齐地址。

### Completions
### 完成

Completions: Fmt=00b/10b (no data / with data), Type=01010b (Cpl) or 01001b (CplD). Completions use ID routing — the Completer ID and Requester ID guide the Completion back to the original requester. Completion Status: 000b=Successful, 001b=Unsupported Request (UR), 010b=Configuration Request Retry Status (CRS), 100b=Completer Abort (CA).

> 完成：使用ID路由——Completer ID和Requester ID引导完成返回到原始请求者。Completion Status编码：000b=成功，001b=不支持的请求(UR)，010b=配置请求重试状态(CRS)，100b=完成者中止(CA)。对于Read Completions，Byte Count字段反映剩余待传输的字节数。

### Message Requests
### 消息请求

Messages: Fmt=01b (4DW header, typically no data). Type field = 10rrr₂ (r[2:0] = routing field). The Message Code field (8-bit) defines the message function:
- **INTx Interrupt Messages**: Assert_INTx/Deassert_INTx (Route to Root Complex)
- **Power Management Messages**: PM_Active_State_Nak, PM_PME, PME_Turn_Off/PME_TO_Ack (Route to Root Complex)
- **Error Messages**: ERR_COR, ERR_NONFATAL, ERR_FATAL (Route to Root Complex)
- **Unlock Message**: For locked transaction support
- **Set Slot Power Limit**: Broadcast from Root Complex
- **Vendor-Defined Messages**: Type 0 and Type 1
- **Latency Tolerance Reporting (LTR)**: Devices report acceptable latency
- **Optimized Buffer Flush/Fill (OBFF)**: Power optimization mechanism

> 消息：Fmt=01b（4DW header，通常无数据）。Message Code（8位）定义消息功能。消息替代PCI边带信号——中断(INTx→MSI/MSI-X)、电源管理、错误报告等均通过消息TLP在带内传输。Routing字段决定消息传播方式（到根联合体、按ID等）。

---

## 术语附录 | Terminology Appendix

| English | 中文 | Notes |
|---------|------|-------|
| Attr (Attributes) | 事务属性 | NS, RO, IDO |
| Byte Enable (BE) | 字节使能 | 控制有效字节 |
| CA (Completer Abort) | 完成者中止 | 完成状态 |
| Completion | 完成TLP | Non-Posted的响应 |
| CRS (Configuration Retry Status) | 配置重试状态 | |
| Digest | 摘要 | 可选ECRC |
| ECRC (End-to-End CRC) | 端到端CRC | 事务层检查 |
| Fmt[1:0] | 格式字段 | Header大小+数据指示 |
| IO Request | IO请求 | 传统32位IO |
| Length[9:0] | 长度字段 | DW为单位 |
| Message | 消息TLP | 替代边带信号 |
| Non-Posted | 非发布 | 需完成 |
| NS (No Snoop) | 非窥探 | 事务属性 |
| OBFF | 优化缓冲刷新/填充 | 电源优化 |
| Posted | 已发布 | 无需完成 |
| Requester ID | 请求者ID | BDF标识 |
| RO (Relaxed Ordering) | 宽松排序 | 事务属性 |
| Tag | 标签 | 匹配请求与完成 |
| TC (Traffic Class) | 流量类别 | 0–7, QoS |
| UR (Unsupported Request) | 不支持的请求 | 完成状态 |
