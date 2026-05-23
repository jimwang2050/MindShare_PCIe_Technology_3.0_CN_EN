# Chapter 7: Quality of Service
# 第7章：服务质量 (QoS)

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 304–343 (40 pages)

---

## Motivation
## 动机

Many computer systems don't include mechanisms to manage bandwidth for peripheral traffic, but some applications need it — streaming video, embedded guidance control, isochronous data delivery. The PCIe spec includes QoS mechanisms for **Differentiated Service**: packets are treated differently based on assigned priority, allowing a range from best-effort to guaranteed (isochronous) performance.

> 许多计算机系统不包含外设流量带宽管理机制，但某些应用需要——流视频、嵌入式制导控制、等时数据交付。PCIe规范包含了QoS机制用于**差异化服务（Differentiated Service）**：基于优先级区别对待包，允许从尽力而为到保证（等时）性能的各种级别。

---

## Basic Elements: TC, VC, and Arbitration
## 基本元素：TC、VC与仲裁

QoS in PCIe rests on three pillars:

### Traffic Class (TC)
### 流量类别 (TC)

Every TLP carries a 3-bit **TC** label (0–7) assigned by software based on the application's priority. TC 0 is the default. TC labels travel with the TLP through the entire fabric.

> 每个TLP承载由软件根据应用优先级分配的3位**TC**标签（0–7）。TC 0为默认。TC标签随TLP贯穿整个fabric。

### Virtual Channel (VC)
### 虚拟通道 (VC)

VCs are independent hardware buffer/fabric resources. Each VC has its own flow control buffers, ordering domain, and arbitration mechanism. TC-to-VC mapping allows multiple TCs to map to one VC (for simpler designs) or one-to-one (for maximum differentiation). Most systems implement VC0 only.

> VC是独立的硬件缓冲/fabric资源。每个VC有独立的流控缓冲区、排序域和仲裁机制。TC到VC映射允许多个TC映射到一个VC（简单设计）或一对一（最大差异化）。多数系统仅实现VC0。

### Port Arbitration
### 端口仲裁

When multiple VCs are ready to transmit, the **Port Arbiter** selects which VC goes next. Two arbitration schemes:
- **Strict Priority (Hardware Fixed)**: Higher-numbered VCs always win. Simple but can starve lower VCs.
- **Round Robin (WSP/Weighted Round Robin)**: Programmable weights (1–127) assigned to each VC. VCs get proportional bandwidth.

> 当多个VC就绪可发送时，**端口仲裁器（Port Arbiter）**选择下一个发送的VC。两种仲裁方案：Strict Priority（硬件固定，高编号VC总是赢——简单但可能饿死低VC）；Round Robin（加权轮询，可编程权重1–127，VC获得成比例带宽）。

**VC Capability Structure**: Configuration registers (Figure 7-1) define:
- VC count / VC数量
- TC/VC mapping: Each TC maps to a specific VC
- Port Arbitration Table: Weighted Round Robin weights, or Strict Priority
- Extended VC Count (PCIe 3.0): Supports up to 8 VCs in extended capability format

> VC能力结构配置寄存器（图7-1）定义了VC数量、TC/VC映射（每个TC映射到特定VC）、端口仲裁表（加权轮询权重或Strict Priority）和扩展VC数量（PCIe 3.0：在扩展能力格式中支持最多8个VC）。

---

## Isochronous Support
## 等时支持

PCIe 3.0 enhanced QoS with **Isochronous** support: deterministic, time-based data delivery. Key additions include Isochronous VC Arbitration capability and mechanisms to reserve bandwidth for isochronous traffic. Combined with LTR (Latency Tolerance Reporting), the system can optimally manage power while meeting isochronous deadlines.

> PCIe 3.0通过**等时（Isochronous）**支持增强了QoS：确定性的、基于时间的数据交付。关键新增包括等时VC仲裁能力和为等时流量预留带宽的机制。结合LTR（延迟容忍报告），系统可在满足等时截止时间的同时最优管理功耗。

---

| English | 中文 | Notes |
|---------|------|-------|
| TC (Traffic Class) | 流量类别 | 3-bit, 0–7 |
| VC (Virtual Channel) | 虚拟通道 | 独立缓冲与排序 |
| TC/VC Mapping | TC/VC映射 | 灵活映射 |
| Port Arbitration | 端口仲裁 | 选择下一个发送的VC |
| Strict Priority | 严格优先级 | 高编号VC优先 |
| WRR (Weighted Round Robin) | 加权轮询 | 可编程权重 1–127 |
| Isochronous | 等时 | 确定性数据交付 |
| LTR (Latency Tolerance Reporting) | 延迟容忍报告 | |
