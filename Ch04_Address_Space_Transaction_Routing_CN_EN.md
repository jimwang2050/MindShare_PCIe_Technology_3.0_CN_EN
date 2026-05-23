# Chapter 4: Address Space & Transaction Routing
# 第4章：地址空间与事务路由

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 180–225 (46 pages)
> 来源：MindShare PCI Express 技术 3.0 | 页码：180–225（共46页）

---

## 快速导航 | Quick Navigation

- [Address Spaces — 地址空间](#i-need-an-address)
- [Base Address Registers (BARs) — 基地址寄存器](#base-address-registers-bars)
- [Base and Limit Registers — 基础与限制寄存器](#base-and-limit-registers)
- [TLP Routing Basics — TLP路由基础](#tlp-routing-basics)
  - [ID Routing — ID路由](#id-routing)
  - [Address Routing — 地址路由](#address-routing)
  - [Implicit Routing — 隐式路由](#implicit-routing)
- [术语附录 | Terminology Appendix](#术语附录-terminology-appendix)

---

## I Need An Address
## 我需要一个地址

Almost all devices have internal registers or storage locations that software needs to access. PCI Express supports the same three address spaces as PCI: **Configuration**, **Memory**, and **IO**. Configuration space is used for device control and status in a standardized way. Memory-mapped IO (MMIO) is the modern preferred method — the PCIe spec discourages IO address space use (legacy only, may be deprecated in future).

> 几乎所有设备都有软件需要访问的内部寄存器或存储位置。PCIe支持与PCI相同的三种地址空间：**配置（Configuration）**、**内存（Memory）**和**IO**。配置空间用于以标准化方式控制设备和获取状态。内存映射IO（MMIO）是现代首选方式——PCIe规范不鼓励使用IO地址空间（仅用于传统兼容，未来版本可能废弃）。

### Prefetchable vs. Non-Prefetchable Memory
### 可预取 vs. 不可预取内存

Memory space is subdivided into **prefetchable** (P-MMIO) and **non-prefetchable** (NP-MMIO):
- **Prefetchable**: Reads have no side effects; can be aggressively prefetched. Used for buffers, large data transfers.
- **Non-Prefetchable**: Reads may have side effects (e.g., reading a status register clears it). Must not be speculatively read.

> 内存空间分为**可预取（Prefetchable/P-MMIO）**和**不可预取（Non-Prefetchable/NP-MMIO）**：
> - **可预取**：读取无副作用；可被激进预取。用于缓冲区、大数据传输。
> - **不可预取**：读取可能有副作用（如读状态寄存器会清除它）。不能投机性读取。

---

## Base Address Registers (BARs)
## 基地址寄存器 (BAR)

BARs are registers in a Function's configuration space that request memory or IO address space. Key aspects:
- Each Function may have up to six 32-bit BARs
- BARs can pair up to form 64-bit memory BARs
- The size is determined by writing all-1s and reading back — the number of low-order zeros indicates the size
- Software writes unique base addresses into BARs during enumeration

> BAR是功能配置空间中请求内存或IO地址空间的寄存器：
> - 每个功能最多六个32位BAR
> - BAR可配对形成64位内存BAR
> - 大小通过写全1后读回确定——低位零的个数指示大小
> - 枚举期间软件将唯一基地址写入BAR

**BAR Example**: If a device needs 4KB of memory space: write FFFFF000h to BAR, read back FFFFF000h → lower 12 bits are writable zeros, indicating a 4KB (2¹²) request. Software then writes the assigned base address (e.g., F1000000h) into the BAR.

> **BAR示例**：设备需4KB内存空间时，向BAR写FFFFF000h，读回FFFFF000h → 低12位为可写零，表明4KB（2¹²）请求。软件随后将分配的基地址（如F1000000h）写入BAR。

**Resizable BARs**: PCIe 3.0 introduced resizable BARs — a capability that allows software to negotiate a BAR size with the device (larger than the original request).

> **可调整BAR（Resizable BAR）**：PCIe 3.0引入可调整BAR——允许软件与设备协商BAR大小（大于原始请求的能力）。

---

## Base and Limit Registers
## 基础与限制寄存器

Bridges (and Switch Ports) use **Base/Limit registers** to define address ranges that are routed downstream:
- **P-MMIO Base/Limit**: Prefetchable memory range
- **NP-MMIO Base/Limit**: Non-prefetchable memory range
- **IO Base/Limit**: IO address range

During enumeration, software programs these registers after all downstream devices' BARs have been assigned, creating address windows that encompass all downstream device needs.

> 桥（和Switch端口）使用**Base/Limit寄存器**定义路由到下游的地址范围。枚举期间，软件在下游所有设备BAR分配完毕后编程这些寄存器，创建涵盖所有下游设备需求的地址窗口。

---

## TLP Routing Basics
## TLP路由基础

PCIe uses three routing methods:

| Method | 方法 | Mechanism | Used For |
|--------|------|-----------|----------|
| **Address Routing** | 地址路由 | Target address in TLP header compared against BARs and Base/Limit ranges | Memory & IO Requests |
| **ID Routing** | ID路由 | Bus/Device/Function number in TLP header | Configuration Requests, Completions |
| **Implicit Routing** | 隐式路由 | TLP Type/Message Code interpreted by receiver | Messages (interrupts, errors, PM, etc.) |

> PCIe使用三种路由方法。所有TLP接收端检查三类流量：(1) 与自身地址匹配的流量；(2) ID路由流量——需比较Bus/Dev/Func#；(3) 隐式路由流量——基于Message Code解释。

### ID Routing
### ID路由

ID routing uses the **Bus Number, Device Number, Function Number** fields in the TLP header:
- Endpoints: Single check — does the target BDF match my BDF?
- Switches (Bridges): Two checks per port — upstream check (is target downstream from me?) and downstream check (is target on my secondary bus or further below?)

> ID路由使用TLP header中的Bus/Device/Function编号。端点仅一次检查；Switch每端口两次检查（上游/下游方向）。

### Address Routing
### 地址路由

Address routing uses 32-bit or 64-bit addresses in the TLP header:
- **Endpoints**: Check if the address falls within any of their programmed BARs
- **Switches**: Compare against Base/Limit register pairs. If the address falls within a downstream port's range, route there; if not and received from a downstream port, route upstream toward the Root Complex

> 地址路由使用TLP header中32位或64位地址。端点检查地址是否落在任何已编程BAR内；Switch与Base/Limit寄存器对比较。

### Implicit Routing
### 隐式路由

Implicit routing is only for **Messages**. The TLP header contains no explicit address or ID — instead, a 3-bit Routing field and an 8-bit Message Code determine how the TLP is handled. Examples:
- **Route to Root Complex** (r[2:0] = 000b): Error messages, PME messages
- **Route by ID** (r[2:0] = 010b): Vendor-defined messages
- **Broadcast from Root Complex** (r[2:0] = 011b): Set Slot Power Limit
- **Local — Terminate at Receiver** (r[2:0] = 100b): INTx interrupt messages from Endpoints

> 隐式路由仅用于**消息（Messages）**。TLP header不含显式地址或ID——而是3位Routing字段和8位Message Code决定TLP的处理。消息替代了PCI中的边带信号：中断（INTx→MSI/MSI-X）、错误、电源管理等。

Messages were introduced to replace the side-band signals (interrupts, errors, power management) that were needed in the PCI bus model. By using in-band messages, PCIe eliminates the need for extra pins and provides a cleaner, more efficient signaling method.

> 消息的引入替代了PCI总线模型中需要的边带信号（中断、错误、电源管理）。通过使用带内消息，PCIe消除了对额外引脚的需求，提供了更清晰、更高效的信令方法。

---

## 术语附录 | Terminology Appendix

| English | 中文 | Notes |
|---------|------|-------|
| Address Routing | 地址路由 | 基于内存/IO地址 |
| BAR (Base Address Register) | 基地址寄存器 | 最多6个32位BAR |
| Base/Limit Registers | 基础/限制寄存器 | 桥地址窗口 |
| Completion | 完成 | Non-Posted事务的响应TLP |
| ID Routing | ID路由 | Bus/Dev/Func路由 |
| Implicit Routing | 隐式路由 | Message TLP路由 |
| MMIO (Memory-Mapped IO) | 内存映射IO | 现代首选 |
| NP-MMIO | 不可预取内存映射IO | 可能有副作用 |
| P-MMIO | 可预取内存映射IO | 无副作用 |
| Prefetchable | 可预取的 | 可安全投机读取 |
| Resizable BAR | 可调整BAR | PCIe 3.0特性 |
| Routing Field (r[2:0]) | 路由字段 | Message TLP |
| Split Transaction | 分裂事务 | 请求/完成解耦 |
| TLP (Transaction Layer Packet) | 事务层包 | |
