# Appendix C: Implementing Intelligent Adapters and Multi-Host Systems / 附录C：实现智能适配器和多主机系统

> **来源：** MindShare PCI Express Technology 3.0
> **PDF页码：** 1002–1020 (共19页)
> **格式：** 中英文段落对照 (Chinese-English Parallel)

---

## Introduction / 引言

This appendix describes techniques for implementing intelligent adapters (devices that can initiate transactions autonomously), host failover configurations, and multi-host systems using standard PCI Express components and non-transparent bridging techniques.

> 本附录描述使用标准PCI Express组件和非透明桥接技术实现智能适配器（可自主发起事务的设备）、主机故障切换配置和多主机系统的技术。

---

## Usage Models / 使用模型

### Intelligent Adapters

An intelligent adapter contains its own processor and can initiate DMA transactions to/from host memory without host CPU involvement. This is the foundation of SmartNICs, GPGPU accelerators, and computational storage devices. Key to implementation is the **Non-Transparent Bridge (NTB)** , which creates an address domain boundary between the host and the adapter.

> 智能适配器包含自己的处理器，可在无需主机CPU参与的情况下向/从主机内存发起DMA事务。这是SmartNIC、GPGPU加速器和计算存储设备的基础。实现的关键是**非透明桥（NTB）**，它在主机与适配器之间创建地址域边界。

### Host Failover

Two hosts can connect to a shared device through NTBs. If the primary host fails, the secondary host takes over, accessing the shared device through its NTB. This is used in high-availability storage and networking systems.

> 两个主机可通过NTB连接到共享设备。若主主机故障，备主机接管，通过其NTB访问共享设备。这用于高可用存储和网络系统。

### Multiprocessor Systems

Multiple independent processor domains can share PCIe resources through a combination of transparent bridges (within a domain) and non-transparent bridges (between domains).

> 多个独立处理器域可通过透明桥（域内）和非透明桥（域间）组合共享PCIe资源。

---

## Address Translation / 地址翻译

### Direct Address Translation

Each NTB contains BARs on both sides that map address ranges from one domain to another. A transaction arriving on the primary side with an address falling within an NTB BAR is translated and forwarded to the secondary side.

> 每个NTB两侧包含BAR，将地址范围从一个域映射到另一个域。在初级侧以落入NTB BAR的地址到达的事务被翻译并转发到次级侧。

### Lookup Table Based Address Translation

For more complex scenarios, lookup tables provide flexible mapping of Requester IDs and address ranges. This allows many-to-many mapping between host domains and device functions.

> 对于更复杂的场景，查找表提供Requester ID和地址范围的灵活映射，允许主机域与设备Function之间的多对多映射。

### Downstream BAR Limit Registers

BAR limit registers define the upper bounds of address windows, preventing transactions from exceeding allocated memory regions and ensuring isolation between domains.

> BAR限制寄存器定义地址窗口的上限，防止事务超出分配的内存区域，确保域间隔离。
