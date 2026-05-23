# Chapter 6: Flow Control
# 第6章：流控

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 274–303 (30 pages)

---

## Flow Control Concept / 流控概念

Ports at each end of every PCIe Link must implement Flow Control. Before a packet can be transmitted, flow control checks verify that the receiving port has sufficient buffer space. PCIe's credit-based flow control ensures the transmitter knows the receiver's buffer state before sending — eliminating the blind transmission/retry inefficiency of parallel PCI.

> 每个PCIe链路两端的端口必须实现流控。PCIe基于信用的流控确保发送端在发送前知道接收端的缓冲状态——消除了并行PCI盲目发送/重试的低效。

Flow control operates at the **Transaction Layer**, but the actual credit information is conveyed by **Data Link Layer** DLLPs (InitFC1, InitFC2, UpdateFC). FC logic maintains counters tracking available credits. Importantly, PCIe supports up to 8 Virtual Channels (VCs), each with **independent flow control buffers** — a blocked VC does not affect traffic on other VCs.

> 流控在事务层运行，但实际信用信息由数据链路层DLLP(InitFC1/InitFC2/UpdateFC)传递。FC逻辑维护跟踪可用信用的计数器。PCIe最多支持8个虚拟通道(VC)，每个都有独立的流控缓冲——一个VC被阻塞不影响其他VC的流量。

---

## The Six Credit Types / 六种信用类型

Flow control tracks six independent credit pools (Header + Data × 3 TLP types):

| Credit Type | Applies To | Description |
|-------------|-----------|-------------|
| **PH** (Posted Headers) | Memory Write, Message | Header credits for Posted requests |
| **PD** (Posted Data) | Memory Write, Message | Data payload credits for Posted requests |
| **NPH** (Non-Posted Headers) | Memory Read, Config/IO Read/Write | Header credits for Non-Posted requests |
| **NPD** (Non-Posted Data) | Configuration/IO Write | Data payload credits (non-posted writes only) |
| **CplH** (Completion Headers) | Cpl, CplD | Header credits for Completions |
| **CplD** (Completion Data) | CplD | Data payload credits for Completions with data |

> 六种独立信用池分别跟踪Posted/Non-Posted/Completion的Header和Data。PH/PD用于Memory Write和Message；NPH/NPD用于Memory Read和Config/IO；CplH/CplD用于Completion。

The separation into six types is essential because PCIe ordering rules treat these categories differently. For example, a Posted request can pass a Non-Posted request, but not vice versa. Independent credit pools prevent head-of-line blocking between categories.

> 分成六种类型是必要的，因为PCIe排序规则以不同方式处理这些类别。例如Posted请求可超越Non-Posted请求但反之不可。独立信用池防止类别间的头阻塞。

---

## Credit Initialization (InitFC1/InitFC2) / 信用初始化

During Link initialization (Configuration.Idle state), the FC buffers must be initialized before any TLPs can be transmitted. The process uses two DLLP types:
1. **InitFC1:** First-phase initialization. The receiver advertises the size of each credit pool.
2. **InitFC2:** Second-phase initialization. Confirms the credit values and signals readiness.

Both sides must complete InitFC1 and InitFC2 exchange for all enabled VCs and all six credit types before entering L0 and beginning TLP traffic.

> 链路初始化(Configuration.Idle状态)期间必须初始化FC缓冲才能传输TLP。两个DLLP类型：InitFC1广告每个信用池大小；InitFC2确认信用值并信号就绪。双方必须完成所有已使能VC和全部六种信用类型的InitFC1/InitFC2交换后才能进入L0并开始TLP流量。

---

## Runtime Credit Tracking (UpdateFC DLLPs) / 运行时信用跟踪

Once in L0, the receiver continuously advertises freed credits via **UpdateFC DLLPs**. As TLPs are consumed from the receiver's buffer and forwarded to the upper layers, the corresponding credits are released. The UpdateFC DLLP carries an updated credit value for one credit type at a time — multiple UpdateFC DLLPs may be needed to update all six types.

> 进入L0后接收端持续通过UpdateFC DLLP广告已释放的信用。当TLP从接收端缓冲被消费并转发到上层时释放对应信用。UpdateFC DLLP一次携带一种信用类型的更新值——可能需多个UpdateFC DLLP更新全部六种类型。

The transmitter maintains a running count of available credits for each type. Before sending a TLP, it checks: Do I have enough credits? If yes, the TLP can be transmitted immediately and the credit count is decremented. If no, the TLP is buffered until credits become available. This ensures the receiver's buffer never overflows.

> 发送端维护每种类型的可用信用运行计数。发送TLP前检查是否有足够信用。若是则立即发送并减少信用计数；若否则缓冲直到信用可用。这确保接收端缓冲永不溢出。

**Infinite Credits:** For certain credit types that a receiver never wants to constrain (e.g., Completion Data for a device that always has space), the receiver advertises "Infinite" credits (a special encoding). The transmitter treats this as unlimited — it never waits for credits of that type.

> **无限信用：** 接收端对某些永不期望限制的信用类型（如始终有空间的设备的Completion Data）广告"无限"信用（特殊编码）。发送端视作无限——永不等待该类型信用。

---

## Minimum FC Buffer Sizes / 最小FC缓冲大小

The specification defines minimum buffer sizes for each credit type to guarantee forward progress. For example, the minimum Posted Data buffer must accommodate the largest possible Memory Write TLP (including the maximum payload size of 4096 bytes). If a receiver advertises fewer credits than the minimum, the transmitter is not guaranteed to be able to send compliant TLPs.

> 规范定义每种信用类型的最小缓冲大小以保证前向推进。例如最小Posted Data缓冲必须容纳最大可能的Memory Write TLP(含4096字节最大payload)。若接收端广告的信用少于最小值，发送端不能保证能发送合规的TLP。

---

## FC Buffer Overflow Error Check / 流控缓冲溢出错误检查

The Data Link Layer continuously monitors for FC protocol violations. If a transmitter attempts to send a TLP that exceeds the receiver's advertised credits (e.g., due to a corrupted UpdateFC DLLP or a logic error), the receiver detects the overflow and reports a **Flow Control Protocol Error**. This is a fatal error that typically causes the Link to enter Recovery.

> 数据链路层持续监控FC协议违规。若发送端尝试发送超过接收端广告信用的TLP（如因UpdateFC DLLP损坏或逻辑错误），接收端检测溢出并报告**流控协议错误**。这是一个通常导致链路进入Recovery的严重错误。

---

## FC Update Timer and Error Detection Timer / FC更新定时器与错误检测定时器

**UpdateFC Timer:** The receiver must send UpdateFC DLLPs periodically (even if no new credits are available) so the transmitter knows the Link is functioning. The timer period is implementation-specific but bounded.

**Error Detection Timer:** If a transmitter receives no UpdateFC DLLPs for an extended period (suggesting a lost UpdateFC or a broken Link), it may report an error. This acts as a keep-alive mechanism for FC.

> **UpdateFC定时器：** 接收端必须周期性发送UpdateFC DLLP（即使无新信用可用），使发送端知道链路正常工作。
> **错误检测定时器：** 若发送端长期未收到任何UpdateFC DLLP（暗示UpdateFC丢失或链路中断），可报告错误。这充当FC的保活机制。
