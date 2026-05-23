# Chapter 6: Flow Control
# 第6章：流控

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 274–303 (30 pages)

---

## Flow Control Concept
## 流控概念

Ports at each end of every PCIe Link must implement Flow Control. Before a packet can be transmitted, flow control checks verify that the receiving port has sufficient buffer space. In parallel PCI, transactions are attempted blindly and then retried if rejected — inefficient. PCIe's credit-based flow control eliminates this: the transmitter knows the receiver's buffer state before sending.

> 每个PCIe链路两端的端口必须实现流控。在包被发送之前，流控检查验证接收端口是否有足够缓冲空间。在并行PCI中，事务被盲目尝试，若被拒绝则重试——效率低。PCIe基于信用的流控消除了这一问题：发送端在发送前就知道接收端的缓冲状态。

Flow control logic is shared between the Transaction Layer (maintains counters) and the Data Link Layer (sends/receives FC DLLPs). PCIe supports up to 8 Virtual Channels (VCs), each with independent flow control buffers — a blocked VC doesn't affect others.

> 流控逻辑由事务层（维护计数器）和数据链路层（收发FC DLLP）共享。PCIe最多支持8个虚拟通道（VC），每个都有独立的流控缓冲区——一个VC被阻塞不影响其他VC。

**Credit types / 信用类型：**
- **Posted Headers (PH)** — Memory Write, Message headers / Posted头
- **Posted Data (PD)** — Data payload for Posted TLPs / Posted数据
- **Non-Posted Headers (NPH)** — Read, Config/IO Write headers / Non-Posted头
- **Non-Posted Data (NPD)** — Data payload for Non-Posted requests / Non-Posted数据
- **Completion Headers (CplH)** — Completion headers / 完成头
- **Completion Data (CplD)** — Data payload for Completions / 完成数据

**Initialization**: InitFC1/InitFC2 DLLPs exchange credit sizes. **Runtime**: UpdateFC DLLPs advertise freed credits as TLPs are consumed.

> 初始化：InitFC1/InitFC2 DLLP交换信用大小。运行时：UpdateFC DLLP在TLP被消费时公布释放的信用。接收端必须持续正确跟踪已公布的信用——任何传送中更新信用计数器的错误都将导致不可恢复的链路错误。

---

## Flow Control Buffer Sizing and Error Check
## 流控缓冲区大小与错误检查

Minimum FC buffer sizes are specified to guarantee that compliant devices can always make forward progress. The **FC Buffer Overflow Error Check** catches cases where the transmitter sends a TLP exceeding the receiver's advertised credits — this is a reported error. The **Error Detection Timer** acts as a pseudo-requirement: if a receiver doesn't receive any UpdateFC DLLPs for an extended period, it may suspect a lost UpdateFC and report an error.

> 最小FC缓冲区大小为规范规定，以保证兼容设备始终能向前推进。**FC Buffer Overflow Error Check**捕获发送端超过接收端公布信用的发送——这是报告性错误。**Error Detection Timer**充当伪需求：若接收端长期未收到UpdateFC DLLP，可怀疑UpdateFC丢失并报告错误。

| English Term | 中文 | 描述 |
|-------------|------|------|
| Credit | 信用 | 代表接收缓冲空间的单位 |
| InitFC1 / InitFC2 | 流控初始化DLLP | 初始化期间交换信用大小 |
| UpdateFC | 流控更新DLLP | 运行时公布已释放信用 |
| PH / PD / NPH / NPD / CplH / CplD | 六种信用类型 | Posted/Non-Posted/Completion × Header/Data |
| VC (Virtual Channel) | 虚拟通道 | 最多8个，独立流控 |
| Infinite Credits | 无限信用 | 需特殊编码 |
