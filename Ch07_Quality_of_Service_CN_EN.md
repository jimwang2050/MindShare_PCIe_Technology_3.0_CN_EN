# Chapter 7: Quality of Service
# 第7章：服务质量 (QoS)

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 304–341 (38 pages)

---

## QoS Concept / 服务质量概念

Quality of Service (QoS) in PCIe provides differentiated treatment of traffic flows, ensuring that high-priority traffic (e.g., isochronous audio/video data) is not blocked or excessively delayed by lower-priority traffic (e.g., bulk storage transfers). Without QoS, all TLPs share the same queues and ordering rules — a single congested flow can degrade performance for all traffic on the Link.

> PCIe中的服务质量提供差异化流量处理，确保高优先级流量（如等时音频/视频数据）不被低优先级流量（如大容量存储传输）阻塞或过度延迟。无QoS时所有TLP共享相同队列和排序规则——单个拥塞流可降低链路上所有流量的性能。

QoS is implemented through two orthogonal mechanisms:
1. **Traffic Classes (TC):** Labels on TLPs that identify their QoS requirements
2. **Virtual Channels (VC):** Independent buffer and flow control resources that provide physical separation of traffic

> QoS通过两个正交机制实现：流量类别(TC)——TLP上标识QoS需求的标签；虚拟通道(VC)——独立的缓冲和流控资源，提供流量的物理分离。

---

## Traffic Classes (TC0-TC7) / 流量类别

Each TLP carries a 3-bit **Traffic Class (TC)** field in its header, allowing the Requester to assign one of eight priority levels (TC0-TC7). TC0 is the default — all TLPs must use TC0 unless TC/VC mapping is configured. The TC label travels with the TLP through the entire PCIe fabric.

> 每个TLP在其头部携带3位**Traffic Class (TC)**字段，允许请求者分配8个优先级之一(TC0-TC7)。TC0为默认——除非配置了TC/VC映射，所有TLP必须使用TC0。TC标签随TLP穿越整个PCIe结构。

TC assignment is determined by the device driver or hardware based on the nature of the transaction. For example: TC0 for bulk data, TC1 for control messages, TC2 for isochronous streaming. However, PCIe does NOT specify which TC should be used for which purpose — that is defined by the system software or the device-specific usage model.

> TC分配由设备驱动或硬件根据事务性质决定。例如TC0用于批量数据、TC1用于控制消息、TC2用于等时流。但PCIe不规定哪种TC应用于哪种目的——这由系统软件或设备特定使用模型定义。

---

## Virtual Channels (VC0-VC7) / 虚拟通道

Each Virtual Channel provides an independent set of FC buffers, ordering queues, and arbitration logic. VC0 is the default — it must always be present and carries all traffic before VC configuration. Up to 8 VCs (VC0-VC7) can be configured on a Link.

> 每个虚拟通道提供独立的FC缓冲集、排序列和仲裁逻辑。VC0为默认——它必须始终存在并承载VC配置前的所有流量。链路上最多可配置8个VC(VC0-VC7)。

The VC mechanism provides **physical separation** — a TLP on VC1 uses VC1's flow control credits and does not consume VC0's credits. This means a blocked VC (e.g., its receiver buffer is full) does not affect traffic on other VCs. This is the fundamental value proposition of VCs: head-of-line blocking elimination.

> VC机制提供物理分离——VC1上的TLP使用VC1的流控信用，不消耗VC0的信用。这意味着被阻塞的VC（如接收缓冲满）不影响其他VC上的流量。这是VC的根本价值主张：消除头阻塞。

---

## TC/VC Mapping / TC到VC映射

Software configures the TC/VC mapping through the VC Extended Capability structure (see Chapter 7 of the PCIe spec for register details). The mapping assigns each TC to one VC. Multiple TCs can map to the same VC (consolidation) or each TC can map to its own VC (separation). Unmapped TCs default to VC0.

> 软件通过VC Extended Capability结构配置TC/VC映射。映射将每个TC分配给一个VC。多个TC可映射到同一VC（合并）或每个TC映射到自己的VC（分离）。未映射的TC默认到VC0。

Example mapping: TC0 → VC0 (default traffic), TC1-TC3 → VC0 (consolidated bulk), TC4 → VC1 (isochronous), TC5-TC6 → VC2 (control), TC7 → VC3 (management). The optimal mapping depends on the traffic mix and the number of VCs the hardware supports.

> 映射示例：TC0→VC0（默认）、TC1-TC3→VC0（合并批量）、TC4→VC1（等时）、TC5-TC6→VC2（控制）、TC7→VC3（管理）。最佳映射取决于流量组合和硬件支持的VC数量。

---

## VC Arbitration / VC仲裁

When multiple VCs have packets ready to transmit, the port must decide which VC gets the Link bandwidth. Two arbitration schemes are defined:

**Strict Priority:** Each VC is assigned a priority level. When a higher-priority VC has a packet, it always wins over lower-priority VCs. This provides strong QoS guarantees but can starve low-priority VCs.

**Round-Robin (or Weighted Round-Robin):** VCs are serviced in a cyclic order, optionally with weights that allocate proportional bandwidth. This provides fairness and prevents starvation.

> 当多个VC有数据包就绪时，端口必须决定哪个VC获得链路带宽。两种仲裁方案：严格优先级（高优先级VC总是获胜，提供强QoS保证但可能饿死低优先级VC）；轮询/加权轮询（VC以循环顺序服务，可选加权分配比例带宽，提供公平性并防止饥饿）。

Arbitration is configurable per VC through the VC Resource Control register. The arbitration table can be programmed to implement arbitrary service policies.

> 仲裁可通过VC Resource Control寄存器按VC配置。仲裁表可编程实现任意服务策略。

---

## Isochronous Support / 等时支持

Isochronous traffic requires guaranteed bandwidth and bounded latency — critical for audio/video streaming, real-time control, and sensor data. PCIe supports isochronous contracts through:
1. TC/VC mapping to isolate isochronous traffic on its own VC
2. Strict priority or weighted arbitration for the isochronous VC
3. The Isochronous Virtual Channel Extended Capability

> 等时流量需保证带宽和有界延迟——对音频/视频流、实时控制和传感器数据至关重要。PCIe通过隔离等时流量到其自己的VC、为等时VC设置严格优先级或加权仲裁以及Isochronous Virtual Channel Extended Capability支持等时合约。

---

## QoS and Flow Control Interaction / QoS与流控交互

Since each VC has independent flow control, credit starvation on one VC does not affect others. This is the key advantage over a single-queue model. However, the total Link bandwidth is shared among VCs — arbitration determines the share each VC receives.

> 由于每个VC有独立流控，一个VC上的信用饥饿不影响其他VC。这是相对单队列模型的关键优势。但总链路带宽在VC间共享——仲裁决定每个VC获得的份额。
