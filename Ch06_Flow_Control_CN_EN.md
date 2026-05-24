# Chapter 6: Flow Control
# 第6章：流控

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 — Comprehensive Guide to Generations 1.x, 2.x, 3.0
> Authors: Mike Jackson, Ravi Budruk | Pages: 274–303 (30 pages)
> 来源：MindShare PCI Express 技术 3.0 — 1.x、2.x、3.0代全面指南
> 作者：Mike Jackson, Ravi Budruk | 页码：274–303（共30页）

---

## 快速导航 | Quick Navigation

- [The Previous Chapter / 前一章](#the-previous-chapter)
- [This Chapter / 本章](#this-chapter)
- [The Next Chapter / 下一章](#the-next-chapter)
- [Flow Control Concept / 流控概念](#flow-control-concept)
- [Flow Control Buffers and Credits / 流控缓冲与信用](#flow-control-buffers-and-credits)
- [Flow Control Credits / 流控信用](#flow-control-credits)
- [Flow Control Initialization / 流控初始化](#flow-control-initialization)
- [Introduction to the Flow Control Mechanism / 流控机制介绍](#introduction-to-the-flow-control-mechanism)
- [Flow Control Example / 流控示例](#flow-control-example)
- [Flow Control Updates / 流控更新](#flow-control-updates)
- [Flow Control Update Frequency / 流控更新频率](#flow-control-update-frequency)
- [Error Detection Timer / 错误检测定时器](#error-detection-timer--a-pseudo-requirement)

---

## The Previous Chapter
## 前一章

The previous chapter discusses the three major classes of packets: Transaction Layer Packets (TLPs), Data Link Layer Packets (DLLPs) and Ordered Sets. This chapter describes the use, format, and definition of the variety of TLPs and the details of their related fields. DLLPs are described separately in Chapter 9, entitled "DLLP Elements," on page 307.

> 前一章讨论了三大类数据包：事务层数据包（TLP）、数据链路层数据包（DLLP）和有序集。该章描述了各类TLP的用途、格式定义及相关字段的细节。DLLP将在第9章"DLLP元素"中单独描述（第307页）。

## This Chapter
## 本章

This chapter discusses the purposes and detailed operation of the Flow Control Protocol. Flow control is designed to ensure that transmitters never send Transaction Layer Packets (TLPs) that a receiver can't accept. This prevents receive buffer over-runs and eliminates the need for PCI-style inefficiencies like disconnects, retries, and wait-states.

> 本章讨论流控协议的目的和详细操作。流控旨在确保发送端永远不会发送接收端无法接受的事务层数据包（TLP）。这防止了接收缓冲溢出，并消除了PCI式低效问题——如断开、重试和等待状态。

## The Next Chapter
## 下一章

The next chapter discusses the mechanisms that support Quality of Service and describes the means of controlling the timing and bandwidth of different packets traversing the fabric. These mechanisms include application-specific software that assigns a priority value to every packet, and optional hardware that must be built into each device to enable managing transaction priority.

> 下一章讨论支持服务质量（QoS）的机制，并描述控制不同数据包在交换结构中传输时序和带宽的方法。这些机制包括为每个数据包分配优先级值的应用特定软件，以及每个设备中必须内置的可选硬件以管理事务优先级。

---

## Flow Control Concept
## 流控概念

Ports at each end of every PCIe Link must implement Flow Control. Before a packet can be transmitted, flow control checks must verify that the receiving port has sufficient buffer space to accept it. In parallel bus architectures like PCI, transactions are attempted without knowing whether the target is prepared to handle the data. If the request is rejected due to insufficient buffer space, the transaction is repeated (retried) until it completes. This is the "Delayed Transaction Model" of PCI and while it works the efficiency is poor.

> 每个PCIe链路两端的端口必须实现流控。数据包传输前，流控检查必须验证接收端口有足够的缓冲空间来接受它。在PCI等并行总线架构中，事务在不知道目标是否准备好处理数据的情况下被尝试。如果请求因缓冲空间不足而被拒绝，事务将被重复（重试）直到完成。这就是PCI的"延迟事务模型"，虽然可行但效率低下。

Flow Control mechanisms can improve transmission efficiency if multiple Virtual Channels (VCs) are used. Each Virtual Channel carries transactions that are independent from the traffic flowing in other VCs because flow-control buffers are maintained separately. Therefore, a full Flow Control buffer in one VC will not block access to other VC buffers. PCIe supports up to 8 Virtual Channels.

> 如果使用多个虚拟通道（VC），流控机制可以提高传输效率。每个虚拟通道承载独立于其他VC流量的传输，因为流控缓冲是独立维护的。因此，一个VC中流控缓冲满不会阻塞对其他VC缓冲的访问。PCIe最多支持8个虚拟通道。

The Flow Control mechanism uses a credit-based mechanism that allows the transmitting port to be aware of buffer space available at the receiving port. As part of its initialization, each receiver reports the size of its buffers to the transmitter on the other end of the Link, and then during run-time it regularly updates the number of credits available using Flow Control DLLPs. Technically, of course, DLLPs are overhead because they don't convey any data payload, but they are kept small (always 8 symbols in size) to minimize their impact on performance.

> 流控机制使用基于信用的方法，允许发送端口了解接收端口的可用缓冲空间。作为初始化的一部分，每个接收端向链路另一端的发送端报告其缓冲大小，然后在运行时使用流控DLLP定期更新可用信用数量。当然，从技术上讲DLLP是开销，因为它们不携带任何数据载荷，但保持较小（始终为8个符号大小）以最小化对性能的影响。

Flow control logic is actually a shared responsibility between two layers: the Transaction Layer contains the counters, but the Link Layer sends and receives the DLLPs that convey the information. Figure 6-1 illustrates that shared responsibility. In the process of making flow control work:

> 流控逻辑实际上是两层之间的共享职责：事务层包含计数器，但链路层发送和接收传递信息的DLLP。图6-1说明了这种共享职责。在使流控工作的过程中：

- **Devices Report Available Buffer Space** — The receiver of each port reports the size of its Flow Control buffers in units called credits. The number of credits within a buffer is sent from the receive-side transaction layer to the transmit-side of the Link Layer. At the appropriate times, the Link Layer creates a Flow Control DLLP to forward this credit information to the receiver at the other end of the Link for each Flow Control Buffer.

> - **设备报告可用缓冲空间** — 每个端口的接收端以称为信用的单位报告其流控缓冲大小。缓冲内的信用数从接收侧事务层发送到链路层的发送侧。在适当时机，链路层创建流控DLLP将此信用信息转发给链路另一端的接收端，针对每个流控缓冲。

- **Receivers Register Credits** — The receiver gets Flow Control DLLPs and transfers the credit values to the transmit-side of the transaction layer. This completes the transfer of credits from one link partner to the other. These actions are performed in both directions until all flow control information has been exchanged.

> - **接收端记录信用** — 接收端获取流控DLLP并将信用值传输到事务层的发送侧。这完成了信用从一个链路伙伴到另一个的传输。这些操作在双向上执行，直到所有流控信息已交换完毕。

- **Transmitters Check Credits** — Before it can send a TLP, a transmitter checks the Flow Control Counters to learn whether sufficient credits are available. If so, the TLP is forwarded to the Link Layer but, if not, the transaction is blocked until more Flow Control credits are reported.

> - **发送端检查信用** — 在发送TLP之前，发送端检查流控计数器以了解是否有足够的信用可用。若有，TLP被转发到链路层；若否，事务被阻塞直到更多流控信用被报告。

---

## Flow Control Buffers and Credits
## 流控缓冲与信用

Flow control buffers are implemented for each VC resource supported by a port. Recall that ports at each end of the Link may not support the same number of VCs, therefore the maximum number of VCs configured and enabled by software is the highest common number between the two ports.

> 流控缓冲为端口支持的每个VC资源实现。回想链路两端的端口可能支持不同数量的VC，因此由软件配置和使能的最大VC数量是两个端口之间最高的共同数量。

![Figure 6-1: Location of Flow Control Logic](images/ch06/ch06_p276.png)

*Figure 6-1: Location of Flow Control Logic — FC is a shared responsibility between the Transaction Layer (counters) and Data Link Layer (DLLPs).*
*图6-1：流控逻辑位置 — FC是事务层（计数器）和数据链路层（DLLP）之间的共享职责。*

### VC Flow Control Buffer Organization
### VC流控缓冲组织

Each VC Flow Control buffer at the receiver is managed for each category of transaction flowing through the virtual channel. These categories are:

> 接收端的每个VC流控缓冲针对流经虚拟通道的每类事务进行管理。这些类别包括：

- **Posted Transactions** — Memory Writes and Messages
- **Non-Posted Transactions** — Memory Reads, Configuration Reads and Writes, and I/O Reads and Writes
- **Completions** — Read and Write Completions

> - **Posted事务** — 存储器写和消息
> - **Non-Posted事务** — 存储器读、配置读写和I/O读写
> - **完成** — 读和写完成

In addition, each of these categories is separated into header and data portions for transactions that have both header and data. This yields six different buffers each of which implements its own flow control (see Figure 6-2). Some transactions, like read requests, consist of a header only while others, like write requests, have both a header and data. The transmitter must ensure that both header and data buffer space is available as needed for a transaction before it can be sent. Note that transaction ordering must be maintained within a VC Flow Control buffer when the transactions are forwarded to software or to an egress port in the case of a switch. Consequently, the receiver must also track the order of header and data components within the buffer.

> 此外，对于同时具有头部和数据的事务，每个类别被分为头部和数据部分。这产生了六个不同的缓冲，每个都实现自己的流控（见图6-2）。某些事务（如读请求）仅由头部组成，而其他事务（如写请求）同时具有头部和数据。发送端必须确保事务所需的头部和数据缓冲空间都可用才能发送。注意，当事务被转发到软件或交换机的出口端口时，必须在VC流控缓冲内维护事务排序。因此，接收端还必须跟踪缓冲内头部和数据组件的顺序。

![Figure 6-2: Flow Control Buffer Organization](images/ch06/ch06_p277.png)

*Figure 6-2: Flow Control Buffer Organization — Six independent credit pools: PH, PD, NPH, NPD, CPLH, CPLD.*
*图6-2：流控缓冲组织 — 六个独立信用池：PH、PD、NPH、NPD、CPLH、CPLD。*

---

## Flow Control Credits
## 流控信用

Buffer space is reported by the receiver in units called Flow Control credits. The unit value of Flow Control Credits (FCCs) for header and data buffers are:

> 接收端以称为流控信用的单位报告缓冲空间。头部和数据缓冲的流控信用（FCC）单位值为：

- **Header credits** — maximum header size + digest
  - 4 DWs for completions
  - 5 DWs for requests
- **Data credits** — 4 DWs (aligned 16 bytes)

> - **头部信用** — 最大头部大小 + 摘要
>   - 完成：4 DW
>   - 请求：5 DW
> - **数据信用** — 4 DW（16字节对齐）

Flow Control DLLPs communicate this information, and do not require Flow Control credits themselves. That's because they originate and terminate at the Link Layer and don't use the Transaction Layer buffers.

> 流控DLLP传递此信息，并且它们自身不需要流控信用。这是因为它们发起和终止于链路层，不使用事务层缓冲。

### Initial Flow Control Advertisement
### 初始流控广告

During Flow Control initialization, PCIe devices communicate their buffer sizes by "advertising" their buffer space via flow control credits. PCIe also defines an infinite Flow Control credit value that is required for some buffers. A receiver that advertises infinite buffer space is effectively guaranteeing that its buffer space will never overflow.

> 在流控初始化期间，PCIe设备通过流控信用"广告"其缓冲空间来沟通其缓冲大小。PCIe还定义了一个无限流控信用值，某些缓冲需要此值。广告无限缓冲空间的接收端实际上保证了其缓冲空间永远不会溢出。

### Minimum and Maximum Flow Control Advertisement
### 最小与最大流控广告

The specification defines the minimum number of credits that can be reported for the different Flow Control buffer types as listed in Table 6-1. However, devices normally advertise considerably more credits than the minimum. Table 6-2 lists the maximum advertisement allowed by the specification.

> 规范定义了不同流控缓冲类型可报告的最小信用数，如表6-1所列。然而，设备通常广告的信用远多于最小值。表6-2列出了规范允许的最大广告值。

**Table 6-1: Required Minimum Flow Control Advertisements / 表6-1：要求的最小流控广告**

| Credit Type / 信用类型 | Minimum Advertisement / 最小广告 |
|-------------|----------------------|
| Posted Request Header (PH) | 1 unit. Credit Value = one 4DW HDR + Digest = 5DW. / 1单位。信用值 = 1个4DW头部 + 摘要 = 5DW。 |
| Posted Request Data (PD) | Largest possible setting of the Max_Payload_Size in credits. Example: If the largest Max_Payload_Size value supported is 1024 bytes, the smallest permitted initial credit value would be 040h. / Max_Payload_Size最大可能设置的信用数。例如：若支持的最大Max_Payload_Size为1024字节，则最小允许初始信用值为040h。 |
| Non-Posted Request HDR (NPH) | 1 unit. Credit Value = one 4DW HDR + Digest = 5DW. / 1单位。信用值 = 1个4DW头部 + 摘要 = 5DW。 |
| Non-Posted Request Data (NPD) | 1 unit. Credit Value = 4DW. 2 units for receivers supporting AtomicOp routing or AtomicOp Completer capability (credit value of 02h). / 1单位。信用值 = 4DW。支持AtomicOp路由或AtomicOp Completer能力的接收端为2单位（信用值02h）。 |
| Completion HDR (CPLH) | 1 unit. Credit Value = one 3DW HDR + Digest = 4DW; for Root Complex with peer-to-peer support and Switches. **Infinite** units (Initial Credit Value = all 0's) for Root Complex with no peer-to-peer support and Endpoints. / 1单位。信用值 = 1个3DW头部 + 摘要 = 4DW；适用于支持点对点的根复合体和交换机。**无限**单位（初始信用值 = 全0）适用于无点对点支持的根复合体和端点。 |
| Completion Data (CPLD) | n units. Value of largest possible setting of Max_Payload_Size or size of largest Read Request (whichever is smaller) divided by FC Unit Size (4DW); for Root Complex with peer-to-peer support and Switches. **Infinite** units (Initial Credit Value = all 0's) for Root Complex with no peer-to-peer support and Endpoints. / n单位。Max_Payload_Size最大可能设置值或最大读请求大小（取较小者）除以FC单位大小（4DW）；适用于支持点对点的根复合体和交换机。**无限**单位（初始信用值 = 全0）适用于无点对点支持的根复合体和端点。 |

**Table 6-2: Maximum Flow Control Advertisements / 表6-2：最大流控广告**

| Credit Type / 信用类型 | Maximum Advertisement / 最大广告 |
|-------------|----------------------|
| Posted Request Header (PH) | 128 units. 128 credits @ 5 DWs = 2,560 bytes. / 128单位。128信用 × 5 DW = 2,560字节。 |
| Posted Request Data (PD) | 2048 units. Value of the Max_Payload_Size (4096 bytes) including all functions supported by device (8) divided by the credit size (4 DWs) = 32,768 bytes. / 2048单位。Max_Payload_Size值（4096字节）包含设备支持的所有功能（8）除以信用大小（4 DW）= 32,768字节。 |
| Non-Posted Request HDR (NPH) | 128 units. 128 credits @ 5 DWs = 2,560 bytes. / 128单位。128信用 × 5 DW = 2,560字节。 |
| Non-Posted Request Data (NPD) | The authors could not find a precise value. The maximum number of credits listed for Data is 2048. However, a more reasonable approach might use the Non-Posted header limit of 128 credits, because Non-Posted Data is always associated with Non-Posted Headers. / 作者未能找到精确值。数据的最大信用数列为2048。然而，更合理的方法可能使用Non-Posted头部限制128信用，因为Non-Posted数据始终与Non-Posted头部关联。 |
| Completion HDR (CPLH) | 128 units. 128 credits @ 5 DWs = 2,560 bytes. This is the limit for ports that do not originate transactions (e.g., Root Complex with peer-to-peer support and Switches). **Infinite** units for ports that originate transactions (e.g., Root Complex with no peer-to-peer support and Endpoints). / 128单位。128信用 × 5 DW = 2,560字节。这是不发起事务的端口的限制（如支持点对点的根复合体和交换机）。**无限**单位适用于发起事务的端口（如无点对点支持的根复合体和端点）。 |
| Completion Data (CPLD) | 2048 units. 2048 credits @ 4 DWs = 32,768 bytes. **Infinite** units for ports that originate transactions (e.g., Root Complex with no peer-to-peer support and Endpoints). / 2048单位。2048信用 × 4 DW = 32,768字节。**无限**单位适用于发起事务的端口（如无点对点支持的根复合体和端点）。 |

### Infinite Credits
### 无限信用

Note that a flow control value of 00h will be understood to mean infinite credits during initialization. Following Flow-Control initialization no further advertisements are made. Devices that originate transactions must reserve buffer space for the data or status information that will return during split transactions. These transaction combinations include:

> 注意，流控值00h在初始化期间将被理解为无限信用。流控初始化完成后不再进行进一步的广告。发起事务的设备必须为拆分事务中返回的数据或状态信息预留缓冲空间。这些事务组合包括：

- Non-posted Read requests and return of Completion Data / Non-posted读请求与完成数据返回
- Non-posted Read requests and return of Completion Status / Non-posted读请求与完成状态返回
- Non-posted Write requests and return of Completion Status / Non-posted写请求与完成状态返回

**Special Use for Infinite Credit Advertisements.** The specification points out a special consideration for devices that implement only VC0. For example, the only Non-Posted writes are I/O Writes and Configuration Writes both of which are permitted only on VC0. Thus, Non-Posted data buffers are not used for VC1 – VC7 and an infinite value can be advertised for those values. However, the Non-Posted Header must still operate and header credits must still need to be updated.

> **无限信用广告的特殊用途。** 规范指出了仅实现VC0的设备的特殊考虑。例如，唯一的Non-Posted写是I/O写和配置写，两者仅在VC0上允许。因此，Non-Posted数据缓冲不用于VC1–VC7，可以为这些VC广告无限值。然而，Non-Posted头部仍必须运行，头部信用仍需更新。

---

## Flow Control Initialization
## 流控初始化

### General
### 概述

Prior to sending any transactions, flow control initialization is needed. In fact, TLPs cannot be sent across the Link until Flow Control Initialization is performed successfully. Initialization occurs on every Link in the system and involves a handshake between the devices at each end of a link. This process begins as soon as the Physical Layer link training has completed. The Link Layer knows the Physical Layer is ready when it observes the LinkUp signal is active, as illustrated in Figure 6-3.

> 在发送任何事务之前，需要进行流控初始化。实际上，在流控初始化成功执行之前TLP不能跨链路发送。初始化发生在系统中的每条链路上，并涉及链路两端设备之间的握手。此过程在物理层链路训练完成后立即开始。链路层在观察到LinkUp信号有效时知道物理层已就绪，如图6-3所示。

![Figure 6-3: Physical Layer Reports That It's Ready](images/ch06/ch06_p281.png)

*Figure 6-3: Physical Layer Reports That It's Ready — The LTSSM asserts LinkUp to the DLCMSM.*
*图6-3：物理层报告其已就绪 — LTSSM向DLCMSM断言LinkUp。*

Once started, the Flow Control initialization process is fundamentally the same for all Virtual Channels and is controlled by hardware once a VC has been enabled. VC0 is always enabled by default, so its initialization is automatic. That allows configuration transactions to traverse the topology and carry out the enumeration process. Other VCs only initialize when configuration software has set up and enabled them at both ends of the Link.

> 一旦开始，流控初始化过程对所有虚拟通道基本相同，并在VC被使能后由硬件控制。VC0始终默认使能，因此其初始化是自动的。这允许配置事务穿越拓扑并执行枚举过程。其他VC仅在配置软件在链路两端设置并使能它们时才初始化。

### The FC Initialization Sequence
### FC初始化序列

The flow control initialization process involves the Link Layer's DLCMSM (Data Link Control and Management State Machine). As shown in Figure 6-4, a reset puts the state machine into the DL_Inactive state. While in the DL_Inactive state, DL_Down is signaled to both the Link and Transaction Layers. Meanwhile, it waits to see LinkUp from the Physical Layer to indicate that the LTSSM has finished its work and the Physical Layer is ready. That causes a transition to the DL_Init sub-state, which contains two stages that handle flow control initialization: FC_INIT1 and FC_INIT2.

> 流控初始化过程涉及链路层的DLCMSM（数据链路控制与管理状态机）。如图6-4所示，复位将状态机置于DL_Inactive状态。在DL_Inactive状态下，DL_Down被发信号给链路层和事务层。同时，它等待来自物理层的LinkUp以指示LTSSM已完成其工作且物理层就绪。这导致转换到DL_Init子状态，该子状态包含处理流控初始化的两个阶段：FC_INIT1和FC_INIT2。

![Figure 6-4: The Data Link Control & Management State Machine](images/ch06/ch06_p282.png)

*Figure 6-4: The DLCMSM — DL_Inactive → DL_Init (FC_Init1 → FC_Init2) → DL_Active.*
*图6-4：DLCMSM状态机 — DL_Inactive → DL_Init（FC_Init1 → FC_Init2）→ DL_Active。*

### FC_Init1 Details
### FC_Init1详情

During the FC_INIT1 state, devices continuously send a sequence of 3 InitFC1 Flow Control DLLPs advertising their receiver buffer sizes (see Figure 6-5). According to the spec, the packets must be sent in this order: Posted, Non-posted, and Completions as illustrated in Figure 6-6. The specification strongly encourages that these be repeated frequently to make it easier for the receiving device to see them, especially if there are no TLPs or DLLPs to send. Each device should also receive this sequence from its neighbor so it can register the buffer sizes. Once a device has sent its own values and received the complete sequence enough times to be confident that the values were seen correctly, it's ready to exit FC_INIT1. To do that, it records the received values in its transmit counters, sets an internal flag (FL1), and changes to the FC_INIT2 state to begin the second initialization step.

> 在FC_INIT1状态期间，设备持续发送3个InitFC1流控DLLP序列，广告其接收缓冲大小（见图6-5）。根据规范，数据包必须按以下顺序发送：Posted、Non-Posted和Completion，如图6-6所示。规范强烈建议频繁重复这些DLLP，以便接收设备更容易看到它们，尤其是没有TLP或DLLP可发送时。每个设备还应从其邻居接收此序列，以便记录缓冲大小。一旦设备已发送自己的值并多次接收到完整序列足以确信值已被正确看到，它就可以退出FC_INIT1。为此，它将接收到的值记录在其发送计数器中，设置内部标志（FL1），并切换到FC_INIT2状态以开始第二个初始化步骤。

![Figure 6-5: INIT1 Flow Control DLLP Format and Contents](images/ch06/ch06_p283.png)

*Figure 6-5: InitFC1 DLLP Format — DLLP Type encodings: 0100 (Init1 Posted), 0101 (Init1 Non-Posted), 0110 (Init1 Completion).*
*图6-5：InitFC1 DLLP格式 — DLLP类型编码：0100（Init1 Posted）、0101（Init1 Non-Posted）、0110（Init1 Completion）。*

### FC_Init2 Details
### FC_Init2详情

In this state a device continuously sends InitFC2 DLLPs. These are sent in the same sequence as the InitFC1s and contain the same credit information, but they also confirm that FC initialization has succeeded at the sender. Since the device has already registered the values from the neighbor it doesn't need any more credit information and will ignore any incoming InitFC1s while it waits to see InitFC2s. It can even send TLPs at this point, even though initialization hasn't completed for the other side of the Link, and this is indicated to the Transaction Layer by the DL_Up signal (See Figure 6-7).

> 在此状态下，设备持续发送InitFC2 DLLP。这些DLLP以与InitFC1相同的顺序发送，包含相同的信用信息，但它们还确认FC初始化已在发送端成功。由于设备已从邻居记录了值，它不再需要更多信用信息，并将忽略任何传入的InitFC1，同时等待看到InitFC2。此时它甚至可以发送TLP，即使链路的另一端初始化尚未完成，这通过DL_Up信号指示给事务层（见图6-7）。

**Why is this second initialization step needed?** The simple answer is that neighboring devices may finish FC initialization at different times and this method ensures that the late one will continue to receive the FC information it needs even if the neighbor finishes early. Once a device receives an FC_INIT2 packet for any buffer type, it sets an internal flag (FL2). (It doesn't wait to receive an FC_Init2 for each type.) Note that FL2 is also set upon receipt of an UpdateFC packet or TLP. When both sides are done and have sent InitFC2s, the DLCMSM transitions to the DL_Active state and the Link Layer is ready for normal operation.

> **为什么需要这第二个初始化步骤？** 简单回答是，相邻设备可能在不同时间完成FC初始化，此方法确保较晚完成的一方即使邻居提前完成也能继续接收所需的FC信息。一旦设备接收到任何缓冲类型的FC_INIT2数据包，它设置内部标志（FL2）。（它不等待接收每种类型的FC_Init2。）注意，FL2也在接收到UpdateFC数据包或TLP时被设置。当双方都完成并已发送InitFC2后，DLCMSM转换到DL_Active状态，链路层准备好进行正常操作。

![Figure 6-6: Devices Send InitFC1 in the DL_Init State](images/ch06/ch06_p284.png)

*Figure 6-6: Devices Send InitFC1 in the DL_Init State — Required order: InitFC1-P → InitFC1-NP → InitFC1-Cpl.*
*图6-6：设备在DL_Init状态发送InitFC1 — 要求顺序：InitFC1-P → InitFC1-NP → InitFC1-Cpl。*

![Figure 6-7: FC Values Registered — Send InitFC2s, Report DL_Up](images/ch06/ch06_p285.png)

*Figure 6-7: FC Values Registered — Send InitFC2s, Report DL_Up to Transaction Layer.*
*图6-7：FC值已记录 — 发送InitFC2，向事务层报告DL_Up。*

### Rate of FC_INIT1 and FC_INIT2 Transmission
### FC_INIT1和FC_INIT2发送速率

The specification defines the latency between sending FC_INIT DLLPs as follows:

> 规范定义了发送FC_INIT DLLP之间的延迟如下：

- **VC0.** Hardware-initiated flow control of VC0 requires that FC_INIT1 and FC_INIT2 packets be transmitted "continuously at the maximum rate possible." That is, the resend timer is set to a value of zero.

> - **VC0。** 硬件发起的VC0流控要求FC_INIT1和FC_INIT2数据包"以尽可能的最大速率连续"发送。即重发定时器设置为零值。

- **VC1–VC7.** When software initiates flow control initialization for other VCs, the FC_INIT sequence is repeated "when no other TLPs or DLLPs are available for transmission." However, the latency between the beginning of one sequence to the next can be no greater than 17 μs.

> - **VC1–VC7。** 当软件为其他VC发起流控初始化时，FC_INIT序列在"没有其他TLP或DLLP可用于发送时"重复。然而，从一个序列开始到下一个序列开始的延迟不能大于17 μs。

### Violations of the Flow Control Initialization Protocol
### 流控初始化协议违规

A violation of the flow control initialization protocol can be optionally checked by a device. An error detected can be reported as a Data Link Layer protocol error.

> 流控初始化协议的违规可由设备可选地检查。检测到的错误可作为数据链路层协议错误报告。

---

## Introduction to the Flow Control Mechanism
## 流控机制介绍

### General
### 概述

The specification defines the requirements of the Flow Control mechanism using registers, counters, and mechanisms for reporting, tracking, and calculating whether a transaction can be sent. These elements are not required and the actual implementation is left to the device designer. This section introduces the specification model and serves to explain the concepts and to define the requirements.

> 规范使用寄存器、计数器和报告、跟踪及计算事务是否可发送的机制定义了流控机制的要求。这些元素不是必需的，实际实现留给设备设计者。本节介绍规范模型，用于解释概念和定义要求。

### The Flow Control Elements
### 流控元素

Figure 6-8 illustrates the elements used for managing flow control. The diagram shows transactions flowing in a single direction across a Link, and another set of these elements supports transfers in the opposite direction. The primary function of each element is listed below. While these Flow Control elements are duplicated for all six receive buffers, for simplicity this example only deals with non-posted header flow control.

> 图6-8说明了用于管理流控的元素。该图显示了事务在链路上单向流动，另一组这些元素支持相反方向上的传输。每个元素的主要功能列在下面。虽然这些流控元素对所有六个接收缓冲重复，但为简单起见，此示例仅处理non-posted头部流控。

One final element associated with managing flow control is the Flow Control Update DLLP. This is the only Flow Control packet that is used during normal transmission. The format of the FC Update packet is illustrated in Figure 6-9.

> 与管理流控相关的最后一个元素是流控更新DLLP。这是在正常传输期间使用的唯一种流控数据包。FC更新数据包的格式如图6-9所示。

![Figure 6-8: Flow Control Elements](images/ch06/ch06_p287.png)

*Figure 6-8: Flow Control Elements — Transmitter side: Transactions Pending Buffer, Credits Consumed (CC), Credit Limit (CL), FC Gating Logic. Receiver side: FC Buffer, Credits Allocated (CrAl), Credits Received (CrRcv, optional).*
*图6-8：流控元素 — 发送侧：事务等待缓冲、已消费信用（CC）、信用限制（CL）、FC门控逻辑。接收侧：FC缓冲、已分配信用（CrAl）、已接收信用（CrRcv，可选）。*

### Transmitter Elements
### 发送端元素

- **Transactions Pending Buffer** — holds transactions that are waiting to be sent in the same virtual channel.

> - **事务等待缓冲** — 保存在同一虚拟通道中等待发送的事务。

- **Credits Consumed counter** — contains the credit sum of all transactions sent for this buffer. This count is abbreviated "CC."

> - **已消费信用计数器** — 包含为此缓冲发送的所有事务的信用总和。此计数缩写为"CC"。

- **Credit Limit counter** — initialized by the receiver with the size of the corresponding Flow Control buffer. After initialization, Flow Control update packets are sent periodically to update the Flow Control credits as they become available at the receiver. This value is abbreviated "CL."

> - **信用限制计数器** — 由接收端用相应流控缓冲的大小初始化。初始化后，流控更新数据包被定期发送以更新接收端变得可用的流控信用。此值缩写为"CL"。

- **Flow Control Gating Logic** — performs the calculations to determine if the receiver has sufficient Flow Control credits to accept the pending TLP (PTLP). In essence, this logic checks that the CREDITS_CONSUMED (CC) plus the credits required for the next Pending TLP (PTLP) does not exceed the CREDIT_LIMIT (CL). This specification defines the following equation for performing the check, with all values represented in credits:

> - **流控门控逻辑** — 执行计算以确定接收端是否有足够的流控信用来接受待处理的TLP（PTLP）。本质上，此逻辑检查CREDITS_CONSUMED（CC）加上下一个待处理TLP（PTLP）所需的信用不超过CREDIT_LIMIT（CL）。规范定义了以下方程用于执行此检查，所有值以信用表示：

```
(CL - (CC + PTLP)) mod 2^FieldSize ≤ 2^(FieldSize/2)
```

### Receiver Elements
### 接收端元素

- **Flow Control Buffer** — stores incoming headers or data.

> - **流控缓冲** — 存储传入的头部或数据。

- **Credit Allocated** — tracks the total Flow Control credits that have been allocated (made available). It's initialized by hardware to reflect the size of the associated Flow Control buffer. The buffer fills as transactions arrive but then they are eventually removed from the buffer by the core logic at the receiver. When they are removed, the number of Flow Control credits is added to the CREDIT_ALLOCATED counter. Thus the counter tracks the number of credits currently available.

> - **已分配信用** — 跟踪已分配（使可用）的总流控信用。由硬件初始化以反映相关流控缓冲的大小。事务到达时缓冲填满，但最终由接收端的核心逻辑从缓冲中移除。当它们被移除时，流控信用数被添加到CREDIT_ALLOCATED计数器。因此，该计数器跟踪当前可用的信用数。

- **Credits Received counter (optional)** — tracks the total credits of all TLPs received into the Flow Control buffer. When flow control is functioning properly, the CREDITS_RECEIVED count should be equal to or less than the CREDIT_ALLOCATED count. If this test ever becomes false, a flow control buffer overflow has occurred and an error is detected. The spec recommends that this optional mechanism be implemented and notes that a failure here will be considered a fatal error.

> - **已接收信用计数器（可选）** — 跟踪接收到流控缓冲中的所有TLP的总信用。当流控正常工作时，CREDITS_RECEIVED计数应等于或小于CREDIT_ALLOCATED计数。如果此测试变为false，则发生了流控缓冲溢出并检测到错误。规范建议实现此可选机制，并注意此处的故障将被视为致命错误。

![Figure 6-9: Types and Format of Flow Control DLLPs](images/ch06/ch06_p288.png)

*Figure 6-9: Types and Format of Flow Control DLLPs — DLLP Type encodings: 1000 (Update Posted), 1001 (Update Non-Posted), 1010 (Update Completion).*
*图6-9：流控DLLP的类型和格式 — DLLP类型编码：1000（Update Posted）、1001（Update Non-Posted）、1010（Update Completion）。*

---

## Flow Control Example
## 流控示例

The following example describes the non-posted header Flow Control buffer, and attempts to capture the nuances of the flow control implementation in several situations. The discussion of Flow Control is described with a series of basic stages as follows:

> 以下示例描述了non-posted头部流控缓冲，并尝试捕捉几种情况下流控实现的细微差别。流控的讨论通过一系列基本阶段描述如下：

- **Stage One** — Immediately following initialization a transaction is transmitted and tracked to explain the basic operation of the counters and registers.
- **Stage Two** — The transmitter sends transactions faster than the receiver can process them and the buffer becomes full.
- **Stage Three** — When counters roll over to zero, the mechanism still works but there are a couple of issues to consider.
- **Stage Four** — The optional receiver error check for a buffer overflow.

> - **阶段一** — 初始化后立即发送一个事务并跟踪，以解释计数器和寄存器的基本操作。
> - **阶段二** — 发送端发送事务的速度超过接收端的处理能力，缓冲变满。
> - **阶段三** — 当计数器回绕到零时，机制仍然有效但需考虑几个问题。
> - **阶段四** — 可选接收端缓冲溢出错误检查。

### Stage 1 — Flow Control Following Initialization
### 阶段一 — 初始化后的流控

Once flow control initialization has completed, the devices are ready for normal operation. The Flow Control buffer in our example is 2KB in size. We're describing the non-posted header buffer, and each credit is 5 dwords in size or 20 bytes. That means 102d (66h) Flow Control units are available. Figure 6-10 illustrates the elements involved, including the values that would be in each counter and register following flow control initialization.

> 流控初始化完成后，设备已准备好进行正常操作。我们示例中的流控缓冲大小为2KB。我们描述的是non-posted头部缓冲，每个信用为5 dwords或20字节。这意味着有102d（66h）个流控单元可用。图6-10说明了所涉及的元素，包括流控初始化后每个计数器和寄存器中的值。

When the transmitter is ready to send a TLP, it must first check Flow Control credits. Our example is simple because a non-posted header is the only packet being sent and it always requires just one Flow Control credit, and we are also assuming that no data is included in the transaction.

> 当发送端准备发送TLP时，必须首先检查流控信用。我们的示例很简单，因为non-posted头部是唯一要发送的数据包，且始终只需一个流控信用，我们还假设事务中不包含数据。

![Figure 6-10: Flow Control Elements Following Initialization](images/ch06/ch06_p290.png)

*Figure 6-10: Flow Control Elements Following Initialization — CL = 66h, CC = 00h, CrAl = 66h, CrRcv = 00h.*
*图6-10：初始化后的流控元素 — CL = 66h，CC = 00h，CrAl = 66h，CrRcv = 00h。*

The header credit check is made using unsigned arithmetic (2's complement), and must satisfy the following formula:

> 头部信用检查使用无符号算术（二进制补码）进行，必须满足以下公式：

```
(CL - (CC + PTLP)) mod 2^FieldSize ≤ 2^(FieldSize/2)
```

Substituting values from Figure 6-10 yields:

> 代入图6-10的值得到：

```
(66h - (00h + 01h)) mod 256 ≤ 80h
(66h - 01h) mod 256 ≤ 80h
65h ≤ 80h → Yes, sufficient credits! / 是，信用充足！
```

In this case, the current CREDITS_CONSUMED count (CC) is added to the PTLP credits required, to determine the CREDITS_REQUIRED (CR), and that gives 00h + 01h = 01h. The CREDITS_REQUIRED count is subtracted from the CREDIT_LIMIT count (CL) to determine whether or not sufficient credits are available.

> 在这种情况下，当前CREDITS_CONSUMED计数（CC）加上所需PTLP信用得到CREDITS_REQUIRED（CR），即00h + 01h = 01h。从CREDIT_LIMIT计数（CL）中减去CREDITS_REQUIRED计数以确定是否有足够的信用可用。

**Credit Check (2's complement subtraction detail): / 信用检查（二进制补码减法细节）：**

```
CL  01100110b (66h)
CR  00000001b (01h)

CR inverted:     11111110b / CR取反
+ 1:             11111111b (2's complement of CR / CR的2的补码)

CL                01100110b
2's comp of CR  + 11111111b / CR的2的补码
                 -----------
Result:          01100101b = 65h (carry bit is dropped / 进位位被丢弃)

Is 65h ≤ 80h? / 65h ≤ 80h？ Yes / 是 → Send TLP / 发送TLP
```

If the subtraction result is equal to or less than half the max value, which is tracked with a modulo 256 counter (128), then we know there is sufficient space in the receiver buffer and this packet can be sent. The decision to use only half the counter value avoids a potential count alias problem (see Stage 3).

> 如果减法结果等于或小于最大值的一半（由模256计数器跟踪，即128），则我们知道接收缓冲中有足够的空间，可以发送此数据包。仅使用一半计数器值的决定避免了潜在的计数别名问题（见阶段三）。

![Figure 6-11: Flow Control Elements After First TLP Sent](images/ch06/ch06_p291.png)

*Figure 6-11: Flow Control Elements After First TLP Sent — CC increments from 00h → 01h, CrRcv increments from 00h → 01h.*
*图6-11：发送第一个TLP后的流控元素 — CC从00h增加到01h，CrRcv从00h增加到01h。*

### Stage 2 — Flow Control Buffer Fills Up
### 阶段二 — 流控缓冲填满

Assume now that the receiver has been unable to remove transactions from the Flow Control buffer for some time. Perhaps the device core logic was temporarily busy and unable to process transactions. Eventually, the Flow Control buffer becomes completely full, as shown in Figure 6-12.

> 现在假设接收端在一段时间内无法从流控缓冲中移除事务。可能是设备核心逻辑暂时繁忙无法处理事务。最终，流控缓冲变得完全满，如图6-12所示。

![Figure 6-12: Flow Control Elements with Flow Control Buffer Filled](images/ch06/ch06_p293.png)

*Figure 6-12: Flow Control Elements with Flow Control Buffer Filled — CL = 66h, CC = 66h, buffer exhausted.*
*图6-12：流控缓冲填满时的流控元素 — CL = 66h，CC = 66h，缓冲耗尽。*

If the transmitter wishes to send another TLP and checks the Flow Control credits:

> 如果发送端希望发送另一个TLP并检查流控信用：

```
Credit Limit (CL)   = 66h
Credits Required (CR) = 67h

CL  01100110b (66h)
CR  10011001b (2's complement of 67h / 67h的2的补码)
    ----------
    11111111b = FFh

Is FFh ≤ 80h? / FFh ≤ 80h？ No / 否 → Don't send packet — BLOCKED / 不发送数据包——已阻塞
```

This channel is blocked until an Update Flow Control DLLP is received with a new CREDIT_LIMIT value of 67h or greater. When the new value is loaded into the CL register the transmitter credit check will pass the test and a TLP can be sent:

> 此通道被阻塞，直到接收到带有新CREDIT_LIMIT值67h或更大的Update Flow Control DLLP。当新值加载到CL寄存器中时，发送端信用检查将通过测试，TLP可以被发送：

```
CL  01100111b (67h)
CR  10011001b (2's complement of 67h / 67h的2的补码)
    ----------
    00000000b = 00h

Is 00h ≤ 80h? / 00h ≤ 80h？ Yes / 是 → Send transaction / 发送事务
```

### Stage 3 — Counters Roll Over
### 阶段三 — 计数器回绕

Since the Credit Limit (CL) and Credits Required (CR) counts only increment upward, they eventually roll over back to zero. When CL rolls over and CR has not, the credit check (CL − CR) results in a small CL value and a large CR value. However, what might appear to be a problem is not when using unsigned arithmetic. As described in the previous examples the results are handled correctly when performing 2's complement subtraction. Figure 6-13 shows the CL and CR counts before and after CL rollover along with the 2's complement results.

> 由于Credit Limit（CL）和Credits Required（CR）计数只向上递增，它们最终会回绕到零。当CL回绕而CR尚未回绕时，信用检查（CL − CR）导致小的CL值和大的CR值。然而，在使用无符号算术时，看似问题实际上不是问题。如前面示例所述，执行二进制补码减法时结果被正确处理。图6-13显示了CL回绕前后的CL和CR计数以及二进制补码结果。

![Figure 6-13: Flow Control Rollover Problem](images/ch06/ch06_p294.png)

*Figure 6-13: Flow Control Rollover Problem — 2's complement arithmetic handles counter rollover correctly.*
*图6-13：流控回绕问题 — 二进制补码算术正确处理计数器回绕。*

**Before CL Rollover / CL回绕前:**
```
CL = F8h, CR = E8h
CL  11111000b (F8h)
CR  00011000b (E8h 2's complement / E8h的2的补码)
    ----------
    00010000b = 10h (Available credit / 可用信用)
```

**After CL Rollover / CL回绕后:**
```
CL = 08h, CR = F8h
CL  00001000b (08h)
CR  00001000b (F8h 2's complement / F8h的2的补码)
    ----------
    00010000b = 10h (Same available credit! / 相同的可用信用！)
```

The result is the same — 2's complement subtraction handles the rollover transparently.

> 结果相同——二进制补码减法透明地处理了回绕。

### Stage 4 — FC Buffer Overflow Error Check
### 阶段四 — FC缓冲溢出错误检查

Although it's optional to do so, the specification recommends implementation of the FC buffer overflow error checking mechanism. Figure 6-14 shows the elements associated with the overflow error check that include:

> 虽然是可选的，但规范建议实现FC缓冲溢出错误检查机制。图6-14显示了与溢出错误检查相关的元素，包括：

- Credits Received (CR) counter / 已接收信用（CR）计数器
- Credits Allocated (CA) counter / 已分配信用（CA）计数器
- Error Check Logic / 错误检查逻辑

This permits the receiver to track Flow Control credits in the same manner as the transmitter. If flow control is working correctly, the transmitter's Credits Consumed count will never exceed its Credit Limit, and the receiver's Credits Received count will never exceed its Credits Allocated count.

> 这允许接收端以与发送端相同的方式跟踪流控信用。如果流控正常工作，发送端的Credits Consumed计数永远不会超过其Credit Limit，接收端的Credits Received计数永远不会超过其Credits Allocated计数。

![Figure 6-14: Buffer Overflow Error Check](images/ch06/ch06_p295.png)

*Figure 6-14: Buffer Overflow Error Check — CL = 69h, CC = 66h, CrAl = 66h. CrRcv = 67h indicates overflow (67h > 66h).*
*图6-14：缓冲溢出错误检查 — CL = 69h，CC = 66h，CrAl = 66h。CrRcv = 67h表示溢出（67h > 66h）。*

An overflow condition is detected if the following formula evaluates true (Field Size is either 8 for headers or 12 for data):

> 如果以下公式评估为真，则检测到溢出条件（字段大小为8（头部）或12（数据））：

```
(CA - CR) mod 2^FieldSize > 2^(FieldSize/2)
```

If it does evaluate true, then more credits have been sent to the FC buffer than were available and an overflow has occurred. Note that the 1.0a version of the specification defines the equation as ≥ rather than > as shown above. That appears to be an error, because when CA = CR no overflow condition exists.

> 如果评估为真，则发送到FC缓冲的信用多于可用信用，发生了溢出。注意，规范的1.0a版本将方程定义为≥而非上述的>。这似乎是一个错误，因为当CA = CR时不存在溢出条件。

---

## Flow Control Updates
## 流控更新

The receiver must regularly update its neighboring device with Flow Control credits that become available when transactions are removed from the buffer. Figure 6-15 illustrates an example where the transmitter was previously blocked from sending header transactions because the buffer was full.

> 接收端必须定期向相邻设备更新当事务从缓冲中移除时变得可用的流控信用。图6-15说明了发送端之前因缓冲满而被阻塞发送头部事务的示例。

In the illustration, the receiver has just removed three headers from the Flow Control buffer. More space is now available, but the neighboring device is unaware of this. As headers are removed from the buffer, the CREDITS_ALLOCATED count increments from 66h to 69h. This new count is reported to the CREDIT_LIMIT register of the neighboring device using a Flow Control update packet. Once the credit limit has been updated, transmission of additional TLPs can proceed.

> 在图中，接收端刚从流控缓冲中移除了三个头部。现在有更多空间可用，但相邻设备不知道这一点。随着头部从缓冲中移除，CREDITS_ALLOCATED计数从66h增加到69h。此新计数使用流控更新数据包报告给相邻设备的CREDIT_LIMIT寄存器。一旦信用限制已更新，额外TLP的传输即可继续。

![Figure 6-15: Flow Control Update Example](images/ch06/ch06_p297.png)

*Figure 6-15: Flow Control Update Example — CrAl increments from 66h → 69h as 3 headers are removed.*
*图6-15：流控更新示例 — CrAl从66h增加到69h，因为移除了3个头部。*

**Why report the absolute value and not the delta?** An interesting note here is that the update reports the actual value of the Credits Allocated register. It would have worked to report just the change in the register, as perhaps "+3 credits on NP Headers" for example, but that represents a potential problem. To understand the risk, consider what would happen if the DLLP containing that increment information was lost for some reason. There is no replay mechanism for DLLPs; if an error occurs the packet is simply dropped. In this case, the increment information would be lost without a means of recovering it.

> **为什么报告绝对值而不是增量？** 这里一个有趣的注意点是，更新报告的是Credits Allocated寄存器的实际值。仅报告寄存器的变化，如"+3个NP头部信用"，也是可行的，但这代表了一个潜在问题。要理解风险，考虑如果包含该增量信息的DLLP由于某种原因丢失会发生什么。DLLP没有重播机制；如果发生错误，数据包被简单地丢弃。在这种情况下，增量信息将丢失且无法恢复。

If, on the other hand, the actual value of the register is reported instead and the DLLP fails, the next DLLP that succeeds will get the counters back in synchronization. In that case some time might be wasted if the transmitter is waiting on the FC credits before it can send the next TLP, but no information is lost.

> 另一方面，如果报告的是寄存器的实际值且DLLP失败，下一个成功的DLLP将使计数器重新同步。在这种情况下，如果发送端在发送下一个TLP之前等待FC信用，可能会浪费一些时间，但不会丢失任何信息。

### FC_Update DLLP Format and Content
### FC_Update DLLP格式和内容

Recall that Flow Control update packets, like the Flow Control initialization packets, contain two credit fields, one for header and one for data, as shown in Figure 6-16. The receiver's credit values reported in the HdrFC and DataFC fields may have been updated many times or not at all since the last update packet was sent.

> 回想一下，流控更新数据包与流控初始化数据包一样，包含两个信用字段，一个用于头部一个用于数据，如图6-16所示。在HdrFC和DataFC字段中报告的接收端信用值可能自上次发送更新数据包以来已更新多次或根本没有更新。

![Figure 6-16: Update Flow Control Packet Format and Contents](images/ch06/ch06_p298.png)

*Figure 6-16: UpdateFC DLLP Format — HdrFC and DataFC fields carry the CREDITS_ALLOCATED count. DLLP Type encodings: 1000 (Update Posted), 1001 (Update Non-Posted), 1010 (Update Completion).*
*图6-16：UpdateFC DLLP格式 — HdrFC和DataFC字段携带CREDITS_ALLOCATED计数。DLLP类型编码：1000（Update Posted）、1001（Update Non-Posted）、1010（Update Completion）。*

---

## Flow Control Update Frequency
## 流控更新频率

The specification defines a variety of rules and suggested implementations that govern when and how often Flow Control Update DLLPs should be sent. These are motivated by a desire to:

> 规范定义了各种规则和建议的实现，用于管理何时以及多频繁应该发送流控更新DLLP。这些规则的动机是希望：

- Notify the transmitting device as early as possible about new credits allocated, especially if any transactions were previously blocked.
- Establish worst-case latency between FC Packets.
- Balance the requirements associated with flow control operation, such as:
  - the need to report credits often enough to prevent transaction blocking
  - the desire to reduce the Link bandwidth needed for FC_Update DLLPs
  - selecting the optimum buffer size
  - selecting the maximum data payload size
- Detect violations of the maximum latency between Flow Control packets.

> - 尽早通知发送设备有关新分配的信用，特别是如果任何事务之前被阻塞。
> - 建立FC数据包之间的最坏情况延迟。
> - 平衡与流控操作相关的需求，如：
>   - 足够频繁地报告信用以防止事务阻塞的需求
>   - 减少FC_Update DLLP所需链路带宽的愿望
>   - 选择最佳缓冲大小
>   - 选择最大数据载荷大小
> - 检测流控数据包之间最大延迟的违规。

Flow Control updates are permitted only when the Link is in the active state (L0 or L0s). All other Link states represent more aggressive power management that have longer recovery latencies.

> 流控更新仅在链路处于活动状态（L0或L0s）时才被允许。所有其他链路状态代表更激进的电源管理，具有更长的恢复延迟。

### Immediate Notification of Credits Allocated
### 信用分配的立即通知

When a Flow Control buffer is so full that maximum-sized packets cannot be sent, the spec requires immediate delivery of a FC_Update DLLP when more space becomes available. Two cases exist:

> 当流控缓冲如此满以至于最大大小的数据包无法发送时，规范要求在更多空间变得可用时立即发送FC_Update DLLP。存在两种情况：

- **Maximum Packet Size = 1 Credit.** When packet transmission is blocked due to a buffer full condition for non-infinite NPH, NPD, PH, and CPLH buffer types, an UpdateFC packet must be scheduled for transmission when one or more credits are made available (allocated) for that buffer type.

> - **最大数据包大小 = 1信用。** 当非无限NPH、NPD、PH和CPLH缓冲类型因缓冲满条件导致数据包传输被阻塞时，必须在为该缓冲类型分配一个或多个信用时调度发送UpdateFC数据包。

- **Maximum Packet Size = Max_Payload_Size.** Flow Control buffer space may decrease to the extent that a maximum-sized packet cannot be sent for non-infinite PD and CPLD credit types. In this case, when one or more additional credits are allocated, an Update FCP must be scheduled for transmission.

> - **最大数据包大小 = Max_Payload_Size。** 流控缓冲空间可能减少到非无限PD和CPLD信用类型无法发送最大大小数据包的程度。在这种情况下，当分配一个或多个额外信用时，必须调度发送Update FCP。

### Maximum Latency Between Update Flow Control DLLPs
### Update Flow Control DLLP之间的最大延迟

The transmission frequency of Update FCPs for each FC credit type (non-infinite) must be scheduled for transmission at least once every 30 μs (−0%/+50%). If the Extended Sync bit within the Control Link register is set, updates must be scheduled no later than every 120 μs (−0%/+50%). Note that Update FCPs may be scheduled for transmission more frequently than is required.

> 每种FC信用类型（非无限）的Update FCP发送频率必须至少每30 μs（−0%/+50%）调度发送一次。如果Control Link寄存器中的Extended Sync位被设置，更新必须不迟于每120 μs（−0%/+50%）调度发送。注意，Update FCP可以比要求的更频繁地调度发送。

### Calculating Update Frequency Based on Payload Size and Link Width
### 基于载荷大小和链路宽度计算更新频率

The specification offers a formula for calculating the frequency at which update packets need to be sent for maximum data payload sizes and Link widths. The formula, shown below, defines FC Update delivery intervals in symbol times. For reference, a symbol time is defined as the time it takes to deliver one symbol: 4ns for Gen1, 2ns for Gen2, 1ns for Gen3.

> 规范提供了一个公式，用于计算最大数据载荷大小和链路宽度下需要发送更新数据包的频率。如下所示公式以符号时间定义了FC更新发送间隔。供参考，一个符号时间定义为发送一个符号所需的时间：Gen1为4ns，Gen2为2ns，Gen3为1ns。

```
(MaxPayloadSize + TLPOverhead) × UpdateFactor
───────────────────────────────────────────────  +  InternalDelay
                  LinkWidth
```

Where / 其中:

- **MaxPayloadSize** = The value in the Max_Payload_Size field of the Device Control register / Device Control寄存器中Max_Payload_Size字段的值
- **TLPOverhead** = the constant value (28 symbols) representing the additional TLP components that consume Link bandwidth (TLP Prefix, Sequence Number, Packet Header, LCRC, Framing Symbols) / 常量值（28个符号），表示消耗链路带宽的额外TLP组件（TLP前缀、序列号、数据包头、LCRC、帧符号）
- **UpdateFactor** = the number of maximum size TLPs sent during the interval between UpdateFC Packets received / 在接收到的UpdateFC数据包间隔期间发送的最大大小TLP的数量
- **LinkWidth** = The number of Lanes the Link is using / 链路使用的通道数
- **InternalDelay** = a constant value of 19 symbol times that represents the internal processing delays for received TLPs and transmitted DLLPs / 常量值19个符号时间，表示接收TLP和发送DLLP的内部处理延迟

The relationship defined by the formula shows that the frequency of update packet delivery decreases as the Link width increases and suggests a timer that triggers scheduling of update packets. Note that this formula does not account for delays associated with the receiver or transmitter being in the L0s power management state.

> 公式定义的关系表明，更新数据包发送频率随链路宽度增加而降低，并建议使用定时器触发更新数据包的调度。注意，此公式不考虑与接收端或发送端处于L0s电源管理状态相关的延迟。

**Table 6-3: Gen1 Unadjusted FC Update LATENCY TIMER Values (Symbol Times) / 表6-3：Gen1未调整的FC更新延迟定时器值（符号时间）**

| Max Payload | ×1 Link | ×2 Link | ×4 Link | ×8 Link | ×12 Link | ×16 Link | ×32 Link |
|-------------|---------|---------|---------|---------|----------|----------|----------|
| 128 Bytes | 237 (UF=1.4) | 128 (UF=1.4) | 73 (UF=1.4) | 67 (UF=2.5) | 58 (UF=3.0) | 48 (UF=3.0) | 33 (UF=3.0) |
| 256 Bytes | 416 (UF=1.4) | 217 (UF=1.4) | 118 (UF=1.4) | 107 (UF=2.5) | 90 (UF=3.0) | 72 (UF=3.0) | 45 (UF=3.0) |
| 512 Bytes | 559 (UF=1.0) | 289 (UF=1.0) | 154 (UF=1.0) | 86 (UF=1.0) | 109 (UF=2.0) | 86 (UF=2.0) | 52 (UF=2.0) |
| 1024 Bytes | 1071 (UF=1.0) | 545 (UF=1.0) | 282 (UF=1.0) | 150 (UF=1.0) | 194 (UF=2.0) | 150 (UF=2.0) | 84 (UF=2.0) |
| 2048 Bytes | 2095 (UF=1.0) | 1057 (UF=1.0) | 538 (UF=1.0) | 278 (UF=1.0) | 365 (UF=2.0) | 278 (UF=2.0) | 148 (UF=2.0) |
| 4096 Bytes | 4143 (UF=1.0) | 2081 (UF=1.0) | 1050 (UF=1.0) | 534 (UF=1.0) | 706 (UF=2.0) | 534 (UF=2.0) | 276 (UF=2.0) |

**Table 6-4: Gen2 Unadjusted FC Update LATENCY TIMER Values (Symbol Times) / 表6-4：Gen2未调整的FC更新延迟定时器值（符号时间）**

| Max Payload | ×1 Link | ×2 Link | ×4 Link | ×8 Link | ×12 Link | ×16 Link | ×32 Link |
|-------------|---------|---------|---------|---------|----------|----------|----------|
| 128 Bytes | 288 (UF=1.4) | 179 (UF=1.4) | 124 (UF=1.4) | 118 (UF=2.5) | 109 (UF=3.0) | 99 (UF=3.0) | 84 (UF=3.0) |
| 256 Bytes | 467 (UF=1.4) | 268 (UF=1.4) | 169 (UF=1.4) | 158 (UF=2.5) | 141 (UF=3.0) | 123 (UF=3.0) | 96 (UF=3.0) |
| 512 Bytes | 610 (UF=1.0) | 340 (UF=1.0) | 205 (UF=1.0) | 137 (UF=1.0) | 160 (UF=2.0) | 137 (UF=2.0) | 103 (UF=2.0) |
| 1024 Bytes | 1122 (UF=1.0) | 596 (UF=1.0) | 333 (UF=1.0) | 201 (UF=1.0) | 245 (UF=2.0) | 201 (UF=2.0) | 135 (UF=2.0) |
| 2048 Bytes | 2146 (UF=1.0) | 1108 (UF=1.0) | 589 (UF=1.0) | 329 (UF=1.0) | 416 (UF=2.0) | 329 (UF=2.0) | 199 (UF=2.0) |
| 4096 Bytes | 4194 (UF=1.0) | 2132 (UF=1.0) | 1101 (UF=1.0) | 585 (UF=1.0) | 757 (UF=2.0) | 585 (UF=2.0) | 327 (UF=2.0) |

**Table 6-5: Gen3 Unadjusted FC Update LATENCY TIMER Values (Symbol Times) / 表6-5：Gen3未调整的FC更新延迟定时器值（符号时间）**

| Max Payload | ×1 Link | ×2 Link | ×4 Link | ×8 Link | ×12 Link | ×16 Link | ×32 Link |
|-------------|---------|---------|---------|---------|----------|----------|----------|
| 128 Bytes | 333 (UF=1.4) | 224 (UF=1.4) | 169 (UF=1.4) | 163 (UF=2.5) | 154 (UF=3.0) | 144 (UF=3.0) | 129 (UF=3.0) |
| 256 Bytes | 512 (UF=1.4) | 313 (UF=1.4) | 214 (UF=1.4) | 203 (UF=2.5) | 186 (UF=3.0) | 168 (UF=3.0) | 141 (UF=3.0) |
| 512 Bytes | 655 (UF=1.0) | 385 (UF=1.0) | 250 (UF=1.0) | 182 (UF=1.0) | 205 (UF=2.0) | 182 (UF=2.0) | 148 (UF=2.0) |
| 1024 Bytes | 1167 (UF=1.0) | 641 (UF=1.0) | 378 (UF=1.0) | 246 (UF=1.0) | 290 (UF=2.0) | 246 (UF=2.0) | 180 (UF=2.0) |
| 2048 Bytes | 2191 (UF=1.0) | 1153 (UF=1.0) | 643 (UF=1.0) | 374 (UF=1.0) | 461 (UF=2.0) | 374 (UF=2.0) | 244 (UF=2.0) |
| 4096 Bytes | 4239 (UF=1.0) | 2177 (UF=1.0) | 1146 (UF=1.0) | 630 (UF=1.0) | 802 (UF=2.0) | 630 (UF=2.0) | 372 (UF=2.0) |

*UF = UpdateFactor*

The specification recognizes that the formula will be inadequate for many applications such as those that stream large blocks of data. These applications may require buffer sizes larger than the minimum specified, as well as a more sophisticated update policy in order to optimize performance and reduce power consumption. Because a given solution is dependent on the particular requirements of an application, no definition for such policies is provided.

> 规范认识到该公式对许多应用（如流式传输大数据块的应用）来说将不够充分。这些应用可能需要比最小规定值更大的缓冲大小，以及更复杂的更新策略以优化性能和降低功耗。因为给定的解决方案取决于应用的特定需求，所以未提供此类策略的定义。

---

## Error Detection Timer — A Pseudo Requirement
## 错误检测定时器 — 一个准要求

The specification defines an optional time-out mechanism for Flow Control packets that is highly recommended and may become a requirement in future versions of the specification. The maximum latency between FC packets for a given credit type is 120 μs, and this timeout has a maximum limit of 200 μs. A separate timer is implemented for each FC credit type (P, NP, Cpl), and each timer is reset when a FC Update DLLP of the corresponding type is received. Note that a timer associated with infinite FC credit values must not report an error.

> 规范为流控数据包定义了一个可选超时机制，该机制被强烈推荐并可能在规范的未来版本中成为要求。给定信用类型的FC数据包之间的最大延迟为120 μs，此超时的最大限制为200 μs。为每种FC信用类型（P、NP、Cpl）实现单独的定时器，每个定时器在接收到相应类型的FC Update DLLP时复位。注意，与无限FC信用值关联的定时器必须不报告错误。

Apart from the infinite case, a timeout implies a serious problem with the Link. If it occurs, the Physical Layer is signaled to go into the Recovery state and retrain the Link in hopes of clearing the error condition. Timer characteristics include:

> 除无限情况外，超时意味着链路存在严重问题。如果发生超时，物理层被发信号进入Recovery状态并重新训练链路，希望清除错误条件。定时器特性包括：

- Operates only when the Link is in an active state (L0 or L0s).
- Max time limited to 200 μs (−0%/+50%).
- Timer is reset when any Init or Update FCP is received, or optionally by receipt of any DLLP.
- Timeout forces the Physical Layer to enter Link Training and Status State Machine (LTSSM) Recovery state.

> - 仅在链路处于活动状态（L0或L0s）时运行。
> - 最大时间限制为200 μs（−0%/+50%）。
> - 定时器在接收到任何Init或Update FCP时复位，或可选地通过接收任何DLLP复位。
> - 超时强制物理层进入链路训练与状态状态机（LTSSM）Recovery状态。

---

## Figures Reference / 图表索引

| Figure / 图 | Description / 描述 | Page / 页码 |
|--------|-------------|------|
| 6-1 | Location of Flow Control Logic / 流控逻辑位置 | 276 |
| 6-2 | Flow Control Buffer Organization (6 credit pools) / 流控缓冲组织（6个信用池） | 277 |
| 6-3 | Physical Layer Reports That It's Ready (LinkUp) / 物理层报告其已就绪（LinkUp） | 281 |
| 6-4 | Data Link Control & Management State Machine (DLCMSM) / 数据链路控制与管理状态机 | 282 |
| 6-5 | InitFC1 Flow Control DLLP Format and Contents / InitFC1流控DLLP格式和内容 | 283 |
| 6-6 | Devices Send InitFC1 in the DL_Init State / 设备在DL_Init状态发送InitFC1 | 284 |
| 6-7 | FC Values Registered — Send InitFC2s, Report DL_Up / FC值已记录—发送InitFC2，报告DL_Up | 285 |
| 6-8 | Flow Control Elements (Transmitter + Receiver) / 流控元素（发送端+接收端） | 287 |
| 6-9 | Types and Format of Flow Control DLLPs / 流控DLLP类型和格式 | 288 |
| 6-10 | Flow Control Elements Following Initialization / 初始化后的流控元素 | 290 |
| 6-11 | Flow Control Elements After First TLP Sent / 发送第一个TLP后的流控元素 | 291 |
| 6-12 | Flow Control Elements with Flow Control Buffer Filled / 流控缓冲填满时的流控元素 | 293 |
| 6-13 | Flow Control Rollover Problem (2's complement) / 流控回绕问题（二进制补码） | 294 |
| 6-14 | Buffer Overflow Error Check / 缓冲溢出错误检查 | 295 |
| 6-15 | Flow Control Update Example / 流控更新示例 | 297 |
| 6-16 | Update Flow Control Packet Format and Contents / UpdateFC数据包格式和内容 | 298 |

## Tables Reference / 表格索引

| Table / 表 | Description / 描述 | Page / 页码 |
|-------|-------------|------|
| 6-1 | Required Minimum Flow Control Advertisements / 要求的最小流控广告 | 278–279 |
| 6-2 | Maximum Flow Control Advertisements / 最大流控广告 | 279–280 |
| 6-3 | Gen1 Unadjusted FC Update LATENCY TIMER Values / Gen1未调整FC更新延迟定时器值 | 300 |
| 6-4 | Gen2 Unadjusted FC Update LATENCY TIMER Values / Gen2未调整FC更新延迟定时器值 | 300–301 |
| 6-5 | Gen3 Unadjusted FC Update LATENCY TIMER Values / Gen3未调整FC更新延迟定时器值 | 301–302 |
