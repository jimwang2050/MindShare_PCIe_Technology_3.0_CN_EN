# Chapter 14: Link Initialization & Training
# 第14章：链路初始化与训练

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 564–702 (139 pages)

---

## Overview / 概述

Link initialization and training is a hardware-based (not software) process controlled by the Physical Layer. The process configures and initializes a device's Link and Port so that normal packet traffic proceeds on the Link. The Link Training and Status State Machine (LTSSM) is the core state machine that orchestrates this entire process — from Power-On or Reset until the Link reaches the fully operational L0 state.

> 链路初始化与训练是由物理层控制的基于硬件（非软件）的过程。该过程配置和初始化设备的链路和端口，使正常的数据包流量能够在链路上进行。链路训练与状态状态机（LTSSM）是编排整个流程的核心状态机——从上电或复位直到链路达到完全可操作的L0状态。

The LTSSM consists of 11 top-level states: Detect, Polling, Configuration, Recovery, L0, L0s, L1, L2, Hot Reset, Disabled, and Loopback. Each state contains multiple substates. The machine operates on main power (not auxiliary power), so when main power is removed, the LTSSM stops operating and resets.

> LTSSM由11个顶级状态组成：Detect、Polling、Configuration、Recovery、L0、L0s、L1、L2、Hot Reset、Disabled和Loopback。每个状态包含多个子状态。该状态机在主电源（非辅助电源）上运行，因此当主电源移除时LTSSM停止运行并复位。

The LTSSM resides within the Physical Layer of each PCIe Port. During initialization, the two LTSSMs on opposite ends of a Link communicate via Training Sequence Ordered Sets (TS1 and TS2) to negotiate Link parameters. Software is not involved in this hardware-autonomous process, though software can trigger certain transitions (e.g., Hot Reset, Link Disable, retraining).

> LTSSM驻留在每个PCIe端口的物理层内。在初始化期间，链路两端的两个LTSSM通过训练序列有序集（TS1和TS2）通信以协商链路参数。软件不参与此硬件自主过程，但可触发某些转换（如Hot Reset、Link Disable、重训练）。

---

## Training Sequences: TS1 and TS2 / 训练序列：TS1与TS2

Training Sequences are Ordered Sets used to exchange configuration and status information between Link partners during initialization and training.

> 训练序列是有序集，用于在初始化和训练期间在链路伙伴之间交换配置和状态信息。

### TS1/TS2 Symbol Format (Gen1/Gen2 vs Gen3)

For Gen1 and Gen2 (2.5 and 5.0 GT/s), each Ordered Set starts with the COM (K28.5) character. Receivers use COM to acquire Symbol Lock. Since COM must appear on all Lanes at the same time, it is also used for Lane-to-Lane de-skew.

For Gen3 (8.0 GT/s), an Ordered Set is identified by the 2-bit Sync Header that must precede the Block, and the first Symbol after that indicates which Ordered Set will follow. For a TS1, the first Symbol is 1Eh, and for a TS2, it is 2Dh.

> 对于Gen1和Gen2（2.5和5.0 GT/s），每个有序集以COM (K28.5)字符开头。接收端使用COM获取符号锁定。由于COM必须在所有通道上同时出现，它也用于通道间去偏斜。对于Gen3（8.0 GT/s），有序集由块前的2位Sync Header标识，其后的第一个Symbol指示后续有序集类型：TS1为1Eh，TS2为2Dh。

### Detailed TS1/TS2 Symbol Fields

**Symbol 0 — COM / TS ID:**
- Gen1/Gen2: The COM character (K28.5) used for Symbol Lock acquisition and de-skew
- Gen3: TS ID — 1Eh for TS1, 2Dh for TS2

> Gen1/Gen2为COM字符(K28.5)，用于获取符号锁定和去偏斜。Gen3中TS ID为1Eh(TS1)或2Dh(TS2)。

**Symbol 1 — Link Number:** In the Polling state this field contains the PAD Symbol (F7h), but in other states a Link Number is assigned (0-31). This uniquely identifies the Link within the PCIe hierarchy. Components compare the received Link Number against their expectation to verify they are communicating with the correct Link partner.

> 在Polling状态此字段为PAD Symbol (F7h)，其他状态中分配Link Number (0-31)。它在PCIe层次结构内唯一标识链路。组件将接收到的Link Number与预期值比较，以验证其与正确的链路伙伴通信。

**Symbol 2 — Lane Number:** In the Polling state this field contains PAD (F7h). In other states, each Lane is assigned a unique Lane Number (0-31). This identifies each Lane within a multi-Lane Link and ensures each Lane's data is correctly assembled at the receiver.

> 在Polling状态此字段为PAD (F7h)。其他状态中，每条通道分配唯一的Lane Number (0-31)。这在多通道链路内标识每条通道，确保接收端正确组装每条通道的数据。

**Symbol 3 — N_FTS (Number of Fast Training Sequences):** Indicates the number of Fast Training Sequences the Receiver will need to achieve the L0 state when exiting from the L0s power state at the current speed. Transmitters will send at least this many FTSs when exiting L0s. The amount of time needed depends on the data rate — at 2.5 GT/s each Symbol takes 4 ns, so if 200 FTSs are needed, the time would be 200 FTS × 4 Symbols per FTS × 4 ns/Symbol = 3,200 ns. If the Extended Synch bit is set in the transmitter's Link Control Register, a total of 4096 FTSs must be sent. This large number provides enough time for external Link monitoring tools (e.g., protocol analyzers) to acquire Bit and Symbol Lock.

> 指示接收端从L0s退出到L0所需的快速训练序列数量。发送端退出L0s时将发送至少此数量的FTS。所需时间取决于数据速率——2.5 GT/s时每Symbol为4ns，若需200个FTS则时间为200×4×4=3200ns。若发送端Link Control Register中Extended Synch位置位，必须发送4096个FTS——这个较大数量为外部链路监控工具（如协议分析仪）获取位和符号锁定提供足够时间。

**Symbol 4 — Rate ID:** Reports which data rates the device supports. Key sub-fields:
- **Data Rate bits:** Identify supported data rates. 2.5 GT/s must always be supported and the Link will train to 2.5 GT/s automatically after reset. If 8.0 GT/s is supported, 5.0 GT/s must also be available.
- **Autonomous Change bit:** If set, indicates the bandwidth change was initiated for power-management reasons. If cleared, the change was due to unreliable operation at the higher speed/wider Link.
- **Selectable De-emphasis bit (at 5.0 GT/s):** Upstream Ports set this to indicate their desired de-emphasis level. Downstream/Root Ports set this bit to match the Selectable De-emphasis field in the Link Control 2 Register.
- **Link Upconfigure Capability:** Reports whether a wide Link whose width is reduced will be capable of going back to the full width. If both sides of a Link report this during Configuration.Complete, an upconfigure_capable bit is set internally.

> 报告设备支持的数据速率。关键子字段：
> - **Data Rate位：** 标识支持的数据速率。2.5 GT/s必须始终支持，复位后链路将自动训练到2.5 GT/s。若支持8.0 GT/s则也必须支持5.0 GT/s。
> - **Autonomous Change位：** 置位表示因电源管理原因发起带宽变更。清除表示因检测到较高速度或较宽链路下的不可靠操作而请求变更。
> - **Selectable De-emphasis位 (5.0 GT/s)：** Upstream Port置位表示期望的去加重级别。Downstream/Root Port根据Link Control 2 Register设置此位。
> - **Link Upconfigure Capability：** 报告宽度降低的宽链路是否能恢复到全宽。若在Configuration.Complete期间双方都报告此能力，内部设置upconfigure_capable位。

**Symbol 5 — Training Control:** Communicates special conditions through individual bit settings:
- **Hot Reset bit:** When set, signals an in-band Hot Reset to the Link partner
- **Enable Loopback bit:** When set, requests the Link partner to enter Loopback mode
- **Disable Link bit:** When set, requests the Link partner to enter the Disabled state
- **Disable Scrambling bit:** When set, disables data scrambling for test/debug purposes
- **Compliance Receive bit:** When set, directs the receiver to enter Compliance test mode

> 通过各个位的设置传达特殊条件：
> - **Hot Reset位：** 置位时向链路伙伴信号带内Hot Reset
> - **Enable Loopback位：** 置位时请求链路伙伴进入Loopback模式
> - **Disable Link位：** 置位时请求链路伙伴进入Disabled状态
> - **Disable Scrambling位：** 置位时为测试/调试禁用数据扰码
> - **Compliance Receive位：** 置位时指示接收端进入合规测试模式

**Symbols 6-9 — TS Identifier:** Identifies the training sequence type (TS1 ID = 4Ah, TS2 ID = 45h). These DWORDs are used in Gen1/Gen2 to differentiate TS1 from TS2. In Gen3, this function is handled by the first Symbol (TS ID field).

> 标识训练序列类型（TS1 ID=4Ah，TS2 ID=45h）。这些DWORD在Gen1/Gen2中用于区分TS1和TS2。Gen3中此功能由第一个Symbol（TS ID字段）处理。

**Symbols 10-13 — EQ Info (Gen3 only):** Equalization information that carries transmitter preset values and coefficient requests during the Recovery.Equalization state at 8.0 GT/s. This enables the iterative equalization process where Link partners exchange preset suggestions and coefficient adjustments.

> 均衡信息，在8.0 GT/s的Recovery.Equalization状态期间携带发送端预设值和系数请求。这使得链路伙伴能够通过交换预设建议和系数调整进行迭代均衡过程。

**Symbols 14-15 — DC Balance Symbols:** Used to maintain DC balance on the Link. These symbols are selected to ensure that over time, the number of 1s and 0s transmitted remains approximately equal, preventing DC drift.

> 用于维持链路上的DC平衡。这些符号的选取确保长期来看发送的1和0数量保持大致相等，防止DC漂移。

---

## The LTSSM States in Detail / LTSSM状态详解

### 1. Detect State / Detect状态

The Detect state is the entry point after Fundamental Reset, Hot Reset, or when exiting L2. Its sole purpose is to detect whether a Link partner (a far-end receiver termination) is present on the Lanes. No communication occurs in this state — it is purely about receiver detection.

> Detect状态是基本复位、热复位或退出L2后的入口点。其唯一目的是检测通道上是否存在链路伙伴（远端接收端接）。此状态不发生通信——纯粹是接收端检测。

**Detect.Quiet (substate 1):** The transmitter is held in Electrical Idle. The receiver monitors each unconfigured Lane for an Electrical Idle exit condition, which would indicate that a far-end transmitter has become active. This substate is typically entered for a minimum of 12 ms or until Electrical Idle exit is detected on any Lane.

> 发送端保持电气空闲。接收端监控每条未配置通道的电气空闲退出条件（这将指示远端发送端已活跃）。此子状态通常进入至少12ms或直到在任一通道上检测到电气空闲退出。

**Detect.Active (substate 2):** The transmitter remains in Electrical Idle. The receiver drives a common-mode voltage transition on each unconfigured Lane and monitors the resultant voltage level. By measuring the RC time constant of the interconnect, the receiver can determine whether a 50Ω termination to ground is present at the far end. If a far-end receiver is detected on any Lane, the LTSSM transitions to Polling. If no receiver is detected, the LTSSM remains in Detect (cycling between Quiet and Active). A Lane where a receiver has been detected is marked as "detected" and will participate in subsequent training.

> 发送端保持电气空闲。接收端在每条未配置通道上驱动共模电压转换并监测结果电压水平。通过测量互连的RC时间常数，接收端可确定远端是否存在50Ω到地的端接。若在任一通道上检测到远端接收端，LTSSM转换到Polling。若未检测到接收端，LTSSM保持在Detect（在Quiet和Active之间循环）。检测到接收端的通道标记为"已检测"，将参与后续训练。

The transition from Detect to Polling occurs when at least one Lane detects a far-end receiver and the 12 ms timeout (minimum) has elapsed. At this point, the set of Lanes that will form the Link is not yet fully determined — that happens during Configuration.

> 从Detect到Polling的转换发生在至少一条通道检测到远端接收端且12ms超时（至少）已过时。此时，将形成链路的通道集尚未完全确定——这在Configuration期间进行。

---

### 2. Polling State / Polling状态

The Polling state establishes Bit Lock and Symbol Lock (or Block Lock at Gen3 speeds) on each Lane, determines Lane polarity, and allows both Link partners to identify each other's data rate capabilities. This is the first state where the transmitter becomes active.

> Polling状态在每条通道上建立位锁定和符号锁定（或Gen3速度下的块锁定），确定通道极性，并允许双方链路伙伴识别彼此的数据速率能力。这是发送端首次变为活跃的状态。

**Polling.Active:** The transmitter begins sending TS1 Ordered Sets on all Lanes where a receiver was detected during Detect. TS1s are sent at the highest common data rate supported by both sides (determined by exchanging data rate information within the TS1s). The receiver attempts to acquire Bit Lock from the incoming data stream — it must determine the boundaries of each bit. After achieving Bit Lock, Symbol Lock (Gen1/Gen2) or Block Lock (Gen3) is acquired by searching for the COM character or Sync Header. Lane polarity is also determined here: if the positive and negative signals of a differential pair are accidentally swapped (due to board routing), the receiver detects this inversion from the TS1 COM character and automatically compensates by inverting the received data.

> 发送端开始在Detect期间检测到接收端的所有通道上发送TS1有序集。TS1以双方支持的最高公共数据速率发送（通过在TS1内交换数据速率信息确定）。接收端尝试从输入数据流获取位锁定——必须确定每个位的边界。实现位锁定后，通过搜索COM字符或Sync Header获取符号锁定(Gen1/Gen2)或块锁定(Gen3)。通道极性也在此确定：若差分对的正负信号因电路板布线而意外交换，接收端从TS1 COM字符检测此反转并自动通过反转接收数据补偿。

**Polling.Configuration:** Once Bit Lock and Symbol Lock are achieved on all detected Lanes, the transmitter continues sending TS1s while the LTSSM transitions toward Configuration. The receiver stores the received values of the TS1 fields for use during Configuration. This substate is short and primarily serves as a handshake confirming that both sides are ready to proceed.

> 一旦所有检测到的通道实现位锁定和符号锁定，发送端继续发送TS1，同时LTSSM向Configuration转换。接收端存储收到的TS1字段值以供Configuration期间使用。此子状态较短暂，主要作为确认双方准备好继续的握手。

**Polling.Compliance:** If a device is a compliance test load (determined by board strapping or test equipment), or if directed by system software, the LTSSM enters this substate. In Compliance mode, the transmitter sends a predefined compliance pattern (a repeating pattern of specific symbols) instead of TS1s. This allows test equipment (such as oscilloscopes or BERT testers) to measure transmitter electrical characteristics — voltage levels, jitter, eye opening — without needing a trained Link. The compliance pattern includes both "stressed" and "clean" symbol sequences for comprehensive testing.

> 若设备是合规测试负载（由电路板配置或测试设备确定），或由系统软件指示，LTSSM进入此子状态。在合规模式下，发送端发送预定义的合规模式（特定符号的重复模式）而非TS1。这允许测试设备（如示波器或BERT测试仪）在无需训练链路的情况下测量发送端电气特性——电压水平、抖动、眼图开口。合规模式包括"受压"和"干净"的符号序列以进行全面测试。

After Polling, the Link transitions to Configuration. If at any point Bit Lock or Symbol Lock is lost, the LTSSM may transition back to Detect.

> Polling之后链路转换到Configuration。若在任何时刻位锁定或符号锁定丢失，LTSSM可转换回Detect。

---

### 3. Configuration State / Configuration状态

The Configuration state is where the Link is fully configured: Link numbers and Lane numbers are assigned, the final Link width is negotiated, Lane-to-Lane de-skew is performed, and both sides confirm they are ready to enter normal operation. This state has six substates that must be traversed in order.

> Configuration状态是链路被完全配置的地方：分配链路号和通道号，协商最终链路宽度，执行通道间去偏斜，双方确认准备好进入正常运行。此状态有六个必须按序经过的子状态。

**Configuration.Linkwidth.Start:** The Downstream Port (the component whose downstream-facing Port is being configured) sends TS1s containing its proposed Link Number and initial Lane Numbers for each Lane. The set of Lanes proposed for the Link is the maximum set that were detected in both Detect and Polling — but the Downstream Port may propose a reduced set if some Lanes exhibited problems during Polling. The Upstream Port echoes back the TS1s with the same Link Number.

> Downstream Port发送包含其建议的Link Number和每条通道初始Lane Number的TS1。为链路建议的通道集是Detect和Polling中都检测到的最大集——但若某些通道在Polling期间表现出问题，Downstream Port可建议缩减集。Upstream Port以相同Link Number回显TS1。

**Configuration.Linkwidth.Accept:** The Downstream Port confirms the Link width by continuing to send TS1s with the final width selection. The Upstream Port echoes back. At this point, the Link width is locked — Lanes not included in the final width are marked as "inactive" and will not participate in L0 operation.

> Downstream Port通过继续发送带最终宽度选择的TS1确认链路宽度。Upstream Port回显。此时链路宽度被锁定——未包含在最终宽度中的通道标记为"不活跃"，将不参与L0操作。

**Configuration.Lanenum.Wait:** The Downstream Port transitions from sending TS1s to TS2s. This is a significant signal: it indicates that the Downstream Port is ready to finalize the Lane numbering. In the TS2s, each Lane is assigned a unique Lane Number (0 to N-1, where N is the negotiated Link width). The TS2 contains the same Link Number and Lane Number information but signals a different substate of the negotiation.

> Downstream Port从发送TS1转换到TS2。这是一个重要信号：它表示Downstream Port准备完成通道编号。在TS2中，每条通道被分配唯一的Lane Number（0到N-1，其中N为协商的链路宽度）。TS2包含相同的Link Number和Lane Number信息，但信号协商的不同子状态。

**Configuration.Lanenum.Accept:** The Upstream Port confirms the Lane Number assignments by echoing back the TS2s. Both sides now agree on exactly which Lane is Lane 0, Lane 1, etc.

> Upstream Port通过回显TS2确认Lane Number分配。双方现在对哪条通道是Lane 0、Lane 1等达成完全一致。

**Configuration.Complete:** Both sides have fully agreed on the Link configuration. TS2s continue to be exchanged while the LTSSM verifies that all lanes are properly de-skewed and that no configuration mismatches exist. The Link Number, Lane Numbers, N_FTS value, and data rate are all latched into internal registers. The upconfigure_capable flag is set if both sides indicated Link Upconfigure Capability.

> 双方已就链路配置完全达成一致。继续交换TS2，同时LTSSM验证所有通道正确去偏斜且不存在配置不匹配。Link Number、Lane Number、N_FTS值和数据速率全部锁存到内部寄存器。若双方指示了Link Upconfigure Capability，则设置upconfigure_capable标志。

**Configuration.Idle:** The final substep. The transmitter stops sending TS2s and instead sends Idle data (logical idle sequences). After a specified number of idle symbols are transmitted and received correctly, the Link transitions to L0 — the fully operational state. The transition to L0 is a carefully timed event: both sides must see idle data for a minimum period (typically several hundred Symbol Times) to confirm the Link is stable before allowing packet traffic.

> 最后子步骤。发送端停止发送TS2，改为发送空闲数据（逻辑空闲序列）。在指定数量的空闲符号被正确收发后，链路转换到L0——完全操作状态。L0的转换是一个精心定时的事件：双方必须在允许数据包流量之前看到空闲数据至少一段时间（通常数百个Symbol Time），以确认链路稳定。

**Lane-to-Lane De-skew:** During Configuration, the receiver must de-skew the data from multiple Lanes. Because the physical traces on a PCB for different Lanes may have slightly different lengths, data on different Lanes arrives at different times. The TS1/TS2 COM characters (or Sync Headers in Gen3) serve as reference markers. The receiver measures the arrival time differences (skew) between the first Lane to receive COM and all other Lanes, then delays the earlier-arriving Lanes to align all data streams. The maximum skew that must be tolerated is specified by the form factor or system design.

> **通道间去偏斜：** 在Configuration期间，接收端必须对来自多条通道的数据进行去偏斜。由于PCB上不同通道的物理走线长度可能略有不同，不同通道上的数据以不同时间到达。TS1/TS2的COM字符（或Gen3中的Sync Header）作为参考标记。接收端测量第一条收到COM的通道与所有其他通道之间的到达时间差（偏斜），然后延迟较早到达的通道以对齐所有数据流。必须容忍的最大偏斜由外形规格或系统设计指定。

---

### 4. L0 State — Fully Operational / L0状态——完全操作

L0 is the fully operational state where normal packet traffic flows. TLPs carry read/write requests and completions; DLLPs handle acknowledgments, flow control updates, and power management; and SKP Ordered Sets perform clock tolerance compensation. The Link remains in L0 for the vast majority of its operational lifetime.

> L0是完全操作状态，正常数据包流量在此流动。TLP承载读写请求和完成；DLLP处理确认、流控更新和电源管理；SKP有序集执行时钟容差补偿。链路在其绝大多数操作生命周期中保持在L0。

When in L0, the Physical Layer continuously transmits either packets (TLPs or DLLPs) or logical idle sequences on each Lane. This continuous transmission keeps the receiver's Bit Lock and Symbol Lock active. If no packets need to be sent, the Physical Layer sends logical idle — a repeating D symbol (typically D0.0 for 8b/10b encoding) that maintains DC balance.

> 在L0时，物理层持续在每条通道上发送数据包（TLP或DLLP）或逻辑空闲序列。这种持续发送保持接收端的位锁定和符号锁定活跃。若无数据包需发送，物理层发送逻辑空闲——一种重复的D符号（8b/10b编码通常为D0.0），维持DC平衡。

**Electrical Idle in L0:** The L0 state can transition to a low-power Link state by placing the transmitter in Electrical Idle. The Electrical Idle condition is characterized by differential peak-to-peak voltage below a specified threshold (typically < 20 mV). Before entering Electrical Idle, the transmitter sends an Electrical Idle Ordered Set (EIOS) to warn the receiver of the impending idle condition.

> **L0中的电气空闲：** L0状态可通过将发送端置于电气空闲转换到低功耗链路状态。电气空闲条件的特点是差分峰峰值电压低于指定阈值（通常<20 mV）。在进入电气空闲前，发送端发送电气空闲有序集（EIOS）以警告接收端即将到来的空闲状态。

---

### 5. L0s State — Standby / L0s状态——待机

L0s is a low-power standby state with very short entry and exit latencies. It is designed for brief idle periods and operates independently per direction — one direction of a Link can be in L0s while the other remains fully active in L0.

> L0s是低功耗待机状态，具有非常短的进入和退出延迟。它针对短暂的闲置期间设计，且按方向独立运行——链路的一个方向可在L0s，而另一个方向保持在L0完全活跃。

**Rx_L0s.Entry:** The receiver detects that the far-end transmitter has entered Electrical Idle (after receiving an EIOS). The receiver enters Rx_L0s, powering down portions of its receive circuitry while maintaining the ability to quickly detect Electrical Idle exit.

> 接收端检测到远端发送端已进入电气空闲（收到EIOS后）。接收端进入Rx_L0s，关闭部分接收电路，同时保持快速检测电气空闲退出的能力。

**Rx_L0s.Idle:** The receiver is in the low-power idle state, monitoring for Electrical Idle exit.

> 接收端处于低功耗空闲状态，监控电气空闲退出。

**Rx_L0s.FTS:** The receiver detects the Electrical Idle exit and begins receiving FTSs from the far-end transmitter. The FTSs provide a known bit pattern that the receiver uses to re-acquire Bit Lock and Symbol Lock. After the programmed number of FTSs (based on the N_FTS value exchanged during Configuration) plus an SKP Ordered Set, the receiver returns to L0.

> 接收端检测到电气空闲退出并开始接收来自远端发送端的FTS。FTS提供已知的位置模式，接收端用于重新获取位锁定和符号锁定。在已编程数量的FTS（基于Configuration期间交换的N_FTS值）加上一个SKP有序集之后，接收端返回L0。

**Tx_L0s.Entry:** The transmitter has been idle (no TLPs or DLLPs to send) for a period determined by the implementation's L0s invocation policy. The transmitter sends an EIOS and then places its drivers in Electrical Idle.

> 发送端已空闲（无TLP或DLLP要发送）一段由实现L0s调用策略决定的时间。发送端发送EIOS，然后将其驱动器置于电气空闲。

**Tx_L0s.Idle:** The transmitter is in Electrical Idle. No data is being transmitted.

> 发送端处于电气空闲。无数据传输。

**Tx_L0s.FTS:** The transmitter needs to resume operation (has a packet to send, or receives a wakeup signal). It exits Electrical Idle by driving the Lanes with FTSs, followed by an SKP Ordered Set, then resumes normal packet transmission.

> 发送端需要恢复操作（有数据包要发送，或收到唤醒信号）。它通过驱动通道发送FTS退出电气空闲，后跟SKP有序集，然后恢复正常数据包发送。

> **注意：** L0s仅在Non-Flit Mode且无Retimer时可用。Retimer硬件不支持L0s。Flit Mode（64.0 GT/s及以上）使用L0p替代L0s。

---

### 6. Recovery State / Recovery状态

Recovery is arguably the most important LTSSM state besides L0. It is the state where the Link re-establishes Bit Lock and Symbol Lock (or Block Lock), re-configures if necessary, handles data rate changes, and performs transmitter equalization. The Link enters Recovery from L0, L0s, or L1 whenever there is a need to re-synchronize.

> Recovery可以说是除L0外最重要的LTSSM状态。它是链路重新建立位锁定和符号锁定（或块锁定）、必要时重新配置、处理数据速率变更以及执行发送端均衡的状态。每当需要重新同步时，链路从L0、L0s或L1进入Recovery。

**Recovery.RcvrLock:** The transmitter starts sending TS1 Ordered Sets on all configured Lanes (and also on previously inactive Lanes that may be re-activated for width upconfiguration). The receiver uses these TS1s to re-acquire Bit Lock and Symbol Lock (or Block Lock). Lane-to-Lane de-skew is re-established. The receiver also checks that the received Link Number and Lane Number match the previously configured values. If they do not match (e.g., the Link partner has changed), the receiver may signal an error or transition to Detect. All configured Lanes must achieve lock before the substate can advance. If lock cannot be achieved within a timeout period (typically 24 ms for Gen1/Gen2), the LTSSM transitions to Detect — the Link is considered down.

> 发送端开始在所有已配置通道上（以及可能为宽度升级重新激活的之前非活跃通道上）发送TS1有序集。接收端使用这些TS1重新获取位锁定和符号锁定（或块锁定）。通道间去偏斜重新建立。接收端还检查接收到的Link Number和Lane Number是否与先前配置值匹配。若不匹配（如链路伙伴已更改），接收端可信号错误或转换到Detect。所有已配置通道必须在子状态可前进前实现锁定。若在超时期间内（Gen1/Gen2通常24ms）无法实现锁定，LTSSM转换到Detect——链路被视为断开。

**Recovery.RcvrCfg:** TS2s are exchanged instead of TS1s. This substate communicates the current Link and Lane number configuration and confirms that both sides agree. It also provides a final check that the Link is operating correctly before returning to L0. If the Downstream Port intends to change the Link width or initiate a data rate change, this information is communicated through the TS1/TS2 fields.

> 交换TS2而非TS1。此子状态通信当前链路和通道号配置，确认双方一致。它还提供返回L0前链路正确操作的最后检查。若Downstream Port打算变更链路宽度或发起数据速率变更，此信息通过TS1/TS2字段通信。

**Recovery.Speed:** Entered when a Link data rate change is requested. This can be a downgrade (e.g., due to excessive errors at higher speed) or an upgrade. Both sides must support the target data rate. The transmitter sends TS1s (and TS2s in RcvrCfg) at the new data rate. If the receiver successfully achieves lock at the new rate on all Lanes, the speed change is committed. If lock fails, the Link falls back to the previous data rate or, if that also fails, transitions to Detect. The speed change process is carefully orchestrated: the Downstream Port proposes the new rate in Recovery.RcvrCfg; both sides then simultaneously switch to the new rate in Recovery.Speed.

> 当请求链路数据速率变更时进入。这可以是降级（如因较高速度下过多错误）或升级。双方必须支持目标数据速率。发送端以新数据速率发送TS1（和RcvrCfg中的TS2）。若接收端在所有通道上以新速率成功锁定，速率变更提交。若锁定失败，链路退回到先前数据速率，若也失败则转换到Detect。速率变更是精心编排的：Downstream Port在Recovery.RcvrCfg中提议新速率；然后双方在Recovery.Speed中同时切换到新速率。

**Recovery.Equalization (Gen3 — 8.0 GT/s and higher):** At 8.0 GT/s data rates, the transmitter must be equalized to compensate for channel frequency-dependent losses. This involves adjusting the transmitter's pre-cursor, main cursor, and post-cursor coefficients to optimize the signal eye at the receiver. The equalization process proceeds through four phases:

- **Phase 0:** The Upstream component (the one whose transmitter is being equalized) advertises its transmitter preset and initial coefficients in TS1s. The Downstream component (the one making equalization requests) evaluates the received signal quality and responds by requesting a specific preset or specific coefficient adjustments (increasing/decreasing pre-cursor, post-cursor by defined steps).

- **Phase 1:** The Downstream component's transmitter is now equalized. The Upstream component evaluates and makes adjustment requests. This phase fine-tunes the Downstream transmitter.

- **Phase 2:** The Upstream component fine-tunes its transmitter based on feedback from Phase 2. Further coefficient optimization occurs.

- **Phase 3:** Final adjustments and lock. Both sides confirm their equalization settings. If equalization is successful at the target data rate and within the specified bit error rate, the settings are committed.

> **Recovery.Equalization (Gen3 — 8.0 GT/s及以上)：** 在8.0 GT/s数据速率下，必须均衡发送端以补偿信道频率相关损耗。这涉及调整发送端的前标、主标和后标系数以优化接收端信号眼图。均衡过程通过四个阶段进行：Phase 0交换预设和初始系数；Phase 1均衡下游发送端；Phase 2均衡上游发送端；Phase 3最终调整与锁定。若在目标数据速率和规定误码率内均衡成功，设置被提交。

**Recovery.Idle:** The final substep before returning to L0. Idle data is transmitted, and after a specified minimum period of successful idle reception, the Link returns to L0. If the Exit Latency from L0s has been exceeded, the Link may also exit through Recovery directly to L0.

> 返回L0前的最后子步骤。发送空闲数据，在成功接收空闲数据一段指定最小时间后，链路返回L0。若已超过L0s的退出延迟，链路也可通过Recovery直接退出到L0。

---

### 7. L1 State — Deeper Low-Power Standby / L1状态——更深低功耗待机

**Entry Sequence:** See Chapter 5 (Power Management) and Chapter 16 for the detailed L1 entry negotiation involving PM_Enter_L1 and PM_Request_Ack DLLPs. After the handshake, both sides bring their transmitters to Electrical Idle. Internal PLLs may be turned off.

**Exit:** See Chapter 5. Exit is triggered by either component needing to transmit. The exiting component sends EIEOS (Electrical Idle Exit Ordered Set) signals, transitions through Recovery to re-establish lock, and the Link returns to L0.

> **进入序列：** 详见第5章（电源管理）和第16章，涉及PM_Enter_L1和PM_Request_Ack DLLP的详细L1进入协商。握手后双方将发送端置于电气空闲。内部PLL可关闭。
> **退出：** 详见第5章。由任一组件的发送需求触发。退出组件发送EIEOS（电气空闲退出有序集）信号，通过Recovery重新建立锁定，链路返回L0。

---

### 8. L2/L3 Ready, Hot Reset, Disabled, Loopback / 其他状态

**L2:** Deep sleep. Main power and refclk removed; wakeup logic powered by Vaux. Exit through Detect via Fundamental Reset.

> L2：深度睡眠。主电源和参考时钟移除；唤醒逻辑由Vaux供电。通过基本复位经Detect退出。

**Hot Reset:** In-band reset propagated by TS1s with the Hot Reset bit set. Used for targeted subsystem resets without affecting the entire platform.

> Hot Reset：通过Hot Reset位置位的TS1传播的带内复位。用于目标子系统复位，不影响整个平台。

**Disabled:** Link is electrically idle and not operational. Entered via setting the Link Disable bit or after too many Recovery retries. Exit requires software to clear Link Disable and trigger retraining.

> Disabled：链路电气空闲，不可操作。通过置位Link Disable位或在过多Recovery重试后进入。退出需要软件清除Link Disable并触发起重训。

**Loopback:** Test/debug mode. Master sends data; Slave loops it back. Used for BERT (Bit Error Rate Testing).

> Loopback：测试/调试模式。Master发送数据；Slave环回。用于BERT（误码率测试）。

---

## Link Data Rate and Width Changes / 链路速率与宽度变更

Data rate re-negotiation (e.g., 2.5→5.0→8.0 GT/s) and width re-negotiation (e.g., x16→x8) both occur through the Recovery state. The Downstream Port proposes changes; both sides must support the target configuration.

> 数据速率重新协商（如2.5→5.0→8.0 GT/s）和宽度重新协商（如x16→x8）均通过Recovery状态进行。Downstream Port提议变更；双方必须支持目标配置。

**Directed Speed Change:** Software writes to the Link Control 2 Register to request a specific Target Link Speed. The hardware performs the speed change at the next opportunity (entering Recovery).

> **定向速率变更：** 软件写入Link Control 2 Register请求特定的Target Link Speed。硬件在下次机会（进入Recovery）时执行速率变更。

**Autonomous Speed Change:** Hardware detects that a higher speed is unreliable (based on error rates exceeding a threshold) and autonomously downgrades. This is a RAS feature — the Link remains operational at the lower speed rather than failing entirely.

> **自主速率变更：** 硬件检测到较高速率不可靠（基于错误率超过阈值）并自主降级。这是RAS特性——链路在较低速率保持运行而非完全失败。

---

## Summary of State Transitions / 状态转换总结

| From | To | Triggering Event |
|------|----|-----------------|
| Detect | Polling | Receiver detected on any Lane after 12ms minimum |
| Polling | Configuration | Bit Lock and Symbol Lock (or Block Lock) achieved on all detected Lanes |
| Configuration | L0 | Link width and Lane numbering completed; idle symbols exchanged |
| L0 | Recovery | Retraining requested, error detected, or speed/width change needed |
| L0 | L0s (per direction) | Idle condition met (no TLPs/DLLPs pending) |
| L0 | L1 | PM_Enter_L1 handshake completes |
| L0s/L1 | Recovery | Transmission needed, Electrical Idle exit detected |
| Recovery | L0 | Lock re-established, no configuration changes pending |
| Recovery | Detect | Lock timeout (24ms for Gen1/2) or configuration mismatch |
| Recovery | Configuration | Link width change or re-configuration required |
| L2 | Detect | Main power restored, Fundamental Reset (PERST# de-asserted) |
| Any | Hot Reset | TS1 received with Hot Reset bit set in Training Control |
| Any | Disabled | Link Disable bit set in Link Control Register |
| Disabled | Detect | Link Disable bit cleared, retraining initiated |
| L0 | Loopback | Enable Loopback bit set in TS1 during Configuration |
| Loopback | Detect | Loopback exit requested or timeout |

> | 从 | 到 | 触发事件 |
> |------|----|-----------------|
> | Detect | Polling | 12ms后任一通道检测到接收端 |
> | Polling | Configuration | 所有检测到的通道实现位锁定和符号锁定 |
> | Configuration | L0 | 链路宽度和通道号完成，交换空闲符号 |
> | L0 | Recovery | 请求重训练、检测到错误或需速率/宽度变更 |
> | L0 | L0s（按方向） | 满足空闲条件（无TLP/DLLP待发送） |
> | L0 | L1 | PM_Enter_L1握手完成 |
> | L0s/L1 | Recovery | 需发送，检测到电气空闲退出 |
> | Recovery | L0 | 锁定重建立，无待处理配置变更 |
> | Recovery | Detect | 锁定超时(24ms)或配置不匹配 |
> | Recovery | Configuration | 需要链路宽度变更或重新配置 |
> | L2 | Detect | 主电源恢复，基本复位(PERST#取消断言) |
> | 任意 | Hot Reset | 收到Training Control中Hot Reset位置位的TS1 |
> | 任意 | Disabled | Link Control Register中Link Disable位置位 |
> | Disabled | Detect | Link Disable位清除，发起重训练 |
> | L0 | Loopback | Configuration期间TS1中Enable Loopback位置位 |
> | Loopback | Detect | 请求退出Loopback或超时 |
