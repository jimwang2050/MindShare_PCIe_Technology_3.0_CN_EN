# Chapter 20: Updates for Spec Revision 2.1 / 第20章：规范2.1版更新

> **来源：** MindShare PCI Express Technology 3.0
> **PDF页码：** 946–973 (共28页)
> **格式：** 中英文段落对照 (Chinese-English Parallel)

---

## System Redundancy Improvement: Multi-casting / 系统冗余改进：多播

Multicast enables a single TLP to be delivered to multiple target devices simultaneously. This is especially useful for redundant storage, network mirroring, and FPGA acceleration clusters.

> 多播使单个TLP能够同时传递到多个目标设备。这对于冗余存储、网络镜像和FPGA加速集群特别有用。

### Multicast Capability Registers

| Register | Description |
|----------|-------------|
| **Multicast Capability** | MC_Max_Group, Window Size Requested |
| **Multicast Control** | MC_Enable, Multicast Group number |
| **MC_Base_Address** | Base address for multicast window |
| **MC_Receive** | Indicates which multicast groups the Function receives |
| **MC_Block_All** | Block all multicast traffic |
| **MC_Block_Untranslated** | Block multicast traffic not matching address translation |

> | 寄存器 | 描述 |
> |--------|------|
> | **Multicast Capability** | MC_Max_Group、Window Size Requested |
> | **Multicast Control** | MC_Enable、Multicast Group号 |
> | **MC_Base_Address** | 多播窗口的基址 |
> | **MC_Receive** | 指示Function接收哪些多播组 |
> | **MC_Block_All** | 阻止所有多播流量 |
> | **MC_Block_Untranslated** | 阻止不匹配地址翻译的多播流量 |

### MC Overlay BAR

The Overlay BAR mechanism allows the multicast window to overlay part of a Function's normal memory address range, enabling a device to transparently receive multicast traffic within its existing BAR space without dedicating a separate BAR.

> Overlay BAR机制允许多播窗口覆盖Function正常内存地址范围的一部分，使设备能够在其现有BAR空间内透明接收多播流量，无需专用BAR。

---

## Performance Improvements / 性能改进

### AtomicOps

Atomic Operations allow a device to perform read-modify-write operations on host memory atomically. Three types:
- **FetchAdd** — atomically add a value and return the original
- **Swap** — atomically exchange a value
- **CAS (Compare and Swap)** — atomically compare and conditionally replace

> 原子操作允许设备在主机内存上原子执行读-改-写操作。三种类型：
> - **FetchAdd** — 原子加一个值并返回原值
> - **Swap** — 原子交换一个值
> - **CAS (Compare and Swap)** — 原子比较并有条件替换

### TPH (TLP Processing Hints)

TLP Processing Hints allow a Requester to provide hints about how its data should be cached or placed in the system's cache hierarchy. TPH supports **Steering Tags** that associate TLPs with specific processing resources (cache sets).

> TLP Processing Hints允许请求者提供关于其数据应如何在系统缓存层次结构中缓存或放置的提示。TPH支持**Steering Tags**，将TLP与特定处理资源（缓存集）关联。

### IDO (ID-based Ordering)

ID-based Ordering allows TLPs with different Requester IDs to bypass each other in ordering queues, improving concurrency by relaxing the traditional PCI producer-consumer ordering model when different Functions are involved.

> ID-based Ordering允许具有不同Requester ID的TLP在排序队列中相互绕过，通过在涉及不同Function时放宽传统PCI生产者-消费者排序模型来提高并发性。

### ARI (Alternative Routing-ID Interpretation)

ARI reinterprets the traditional Bus/Device/Function addressing by merging the 5-bit Device Number and 3-bit Function Number into an 8-bit Function Number, allowing up to 256 Functions per Device (versus 8 in non-ARI mode).

> ARI通过将传统5位Device Number和3位Function Number合并为8位Function Number，重新解释传统Bus/Device/Function寻址，每个Device最多支持256个Function（而非非ARI模式下的8个）。

---

## Power Management Improvements / 电源管理改进

| Feature | Description |
|---------|-------------|
| **DPA (Dynamic Power Allocation)** | Allows runtime switching between multiple power allocation states |
| **LTR (Latency Tolerance Reporting)** | Devices report their maximum tolerable latency to the platform PM controller |
| **OBFF (Optimized Buffer Flush and Fill)** | Platform informs devices of optimal times for DMA activity to reduce power |
| **ASPM Options** | Enhanced reporting of L0s/L1 latencies, optional L1 support without L0s |

> | 特性 | 描述 |
> |------|------|
> | **DPA (Dynamic Power Allocation)** | 允许在多个功率分配状态之间运行时切换 |
> | **LTR (Latency Tolerance Reporting)** | 设备向平台PM控制器报告其最大可容忍延迟 |
> | **OBFF (Optimized Buffer Flush and Fill)** | 平台告知设备DMA活动的最佳时机以减少功耗 |
> | **ASPM Options** | 增强的L0s/L1延迟报告，可选L1支持（无需L0s） |

---

## Configuration Improvements / 配置改进

### Internal Error Reporting

Functions can now report internal errors (uncorrectable internal errors that are not associated with a received TLP) through the AER capability, enabling better RAS (Reliability, Availability, Serviceability).

> Function现可通过AER能力报告内部错误（非关联接收TLP的不可纠正内部错误），实现更好的RAS。

### Resizable BARs

Resizable BAR capability allows system software to renegotiate the size of a Function's BAR after initial enumeration. This enables use cases where a device needs to map a large memory region (e.g., GPU frame buffer) but the exact size is not known at boot or needs to change at runtime.

> Resizable BAR能力允许系统软件在初始枚举后重新协商Function BAR的大小。这适用于设备需要映射大内存区域（如GPU帧缓冲）但确切大小在启动时未知或需要在运行时更改的用例。

Each BAR that supports resizing has a **Capability Register** (lists supported sizes) and a **Control Register** (selects the current size). The number of Resizable BARs and their supported sizes are enumerated through the Resizable BAR Extended Capability (ECAP ID 0015h).

> 每个支持调整大小的BAR都有一个**Capability Register**（列出支持的大小）和一个**Control Register**（选择当前大小）。Resizable BAR的数量及其支持的大小通过Resizable BAR Extended Capability（ECAP ID 0015h）枚举。

### Simplified Ordering Table

The ordering rules table was simplified in PCIe 2.1 to make the specification easier to understand and implement, while maintaining full backward compatibility.

> PCIe 2.1中简化了排序规则表，使其更易于理解和实现，同时保持完全的向后兼容。
