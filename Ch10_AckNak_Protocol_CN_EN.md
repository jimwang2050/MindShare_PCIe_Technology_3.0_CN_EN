# Chapter 10: Ack/Nak Protocol
# 第10章：Ack/Nak协议

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 376–417 (42 pages)

---

## Introduction
## 引言

The Ack/Nak protocol is the reliable delivery mechanism of the Data Link Layer. The transmitter assigns each TLP a 12-bit sequence number and retains a copy in the Retry Buffer until acknowledged. The receiver checks LCRC and sequence numbers, sends Ack DLLPs for successfully received TLPs, and Nak DLLPs (or allows the REPLAY_TIMER to expire) to trigger retransmission.

> Ack/Nak协议是数据链路层的可靠交付机制。发送端为每个TLP分配12位序列号并在Retry Buffer中保留副本直到被确认。接收端检查LCRC和序列号，为成功接收的TLP发送Ack DLLP，通过Nak DLLP（或REPLAY_TIMER超时）触发重传。

---

## Key Counters and Timers
## 关键计数器与定时器

**TLP Transmitter (TX) side / 发送端：**
- **NEXT_TRANSMIT_SEQ** (12-bit): Sequence number assigned to the next TLP / 分配给下一个TLP的序列号
- **ACKD_SEQ** (12-bit): Latest acknowledged sequence number from Ack/Nak / 最近确认的序列号
- **REPLAY_NUM** (3-bit): Counts replay attempts; rolls over → LTSSM Recovery / 重放尝试次数
- **REPLAY_TIMER**: Counts time since last Ack; expires → replay / 自上次Ack以来的时间

**TLP Receiver (RX) side / 接收端：**
- **NEXT_RCV_SEQ** (12-bit): Expected sequence number for the next TLP / 预期下一个TLP的序列号
- **NAK_SCHEDULED**: Flag: Nak DLLP pending transmission / Nak DLLP待发送标志
- **AckNak_LATENCY_TIMER**: Limits time before Ack must be sent / 限制Ack必须发送前的时间

> TX side: NEXT_TRANSMIT_SEQ和ACKD_SEQ之间的差值限制未确认TLP的最大数量——若等式(NEXT_TRANSMIT_SEQ - ACKD_SEQ) mod 4096 >= 2048为真，发送端必须停止接受新TLP。RX side: NEXT_RCV_SEQ跟踪预期序列号；收到的TLP序列号与之比较以检测丢失/重复TLP。

---

## Normal Operation (No Errors)
## 正常操作（无错误）

1. TX accepts TLP from Transaction Layer, assigns NEXT_TRANSMIT_SEQ, increments counter
2. TX calculates and appends 32-bit LCRC (polynomial 04C11DB7h, seed FFFFFFFFh)
3. TX stores a copy in Retry Buffer, passes TLP to Physical Layer
4. RX receives TLP, checks LCRC and sequence number:
   - If SeqNum == NEXT_RCV_SEQ: Good TLP → forward to RX Transaction Layer; increment NEXT_RCV_SEQ; clear NAK_SCHEDULED; schedule Ack (within AckNak_LATENCY_TIMER limit)
   - If SeqNum < NEXT_RCV_SEQ (within 2048 mod range): Duplicate → discard; schedule Ack
5. TX receives Ack: purge all TLPs from Retry Buffer up to AckNak_Seq_Num; load ACKD_SEQ; reset REPLAY_NUM and REPLAY_TIMER

> 正常操作：TX从事务层接受TLP，分配序列号，增量计数器；计算并追加32位LCRC；在Retry Buffer中保留副本；传递给物理层。RX接收TLP，检查LCRC和序列号；若匹配则转发至接收事务层，增量NEXT_RCV_SEQ，清零NAK_SCHEDULED，调度Ack。TX收到Ack：清除Retry Buffer中直至AckNak_Seq_Num的所有TLP；加载ACKD_SEQ；复位REPLAY_NUM和REPLAY_TIMER。

---

## TLP Transmission Errors
## TLP传输错误

Error scenarios handled by Ack/Nak:
- **LCRC Error (Bad TLP)**: RX detects LCRC mismatch → discard TLP, schedule Nak / RX检测LCRC不匹配→丢弃TLP，调度Nak
- **Sequence Number Error**: TLP lost or out-of-order → RX detects SeqNum ≠ NEXT_RCV_SEQ → if duplicate: Ack; otherwise: Nak, Bad TLP error / TLP丢失或乱序→RX检测到SeqNum≠NEXT_RCV_SEQ→若重复：Ack；否则：Nak，Bad TLP错误
- **REPLAY_TIMER Expiry**: TX times out waiting for Ack → replay all unacknowledged TLPs / TX等待Ack超时→重放所有未确认TLP
- **Nullified TLP**: TX intentionally corrupts a TLP (inverted LCRC) to cancel it → RX discards, no error / TX有意破坏TLP（反转LCRC）以取消→RX丢弃，无错误
- **RX Physical Layer Error**: Physical Layer reports error → discard TLP, schedule Nak / 物理层报告错误→丢弃TLP，调度Nak

---

## Replay Process
## 重放过程

When a Nak is received or REPLAY_TIMER expires:
1. TX blocks new TLPs from Transaction Layer
2. Completes any TLP currently being transmitted
3. Increments **REPLAY_NUM** (by 2 in Non-Flit Mode). If REPLAY_NUM rolls over from 110b/111b → 000b/001b, signals Physical Layer to retrain Link (reported error)
4. Retransmits all unacknowledged TLPs from the Retry Buffer, starting with the oldest
5. If Ack/Nak received during replay, TX may complete the replay regardless, or skip newly-acknowledged TLPs
6. Once all unacknowledged TLPs retransmitted, resumes normal operation

> 重放过程：TX阻断新TLP；完成当前传输；增量REPLAY_NUM（非Flit模式+2）——若翻转→通知物理层重训链路；从Retry Buffer最旧的未确认TLP开始重传。重放期间收到的Ack/Nak可被合并处理。

---

## REPLAY_TIMER Limits
## REPLAY_TIMER限值

- **Simplified REPLAY_TIMER** (8.0 GT/s+, all data rates recommended):
  - Normal: 24,000–31,000 Symbol Times
  - Extended Synch: 80,000–100,000 Symbol Times

The Simplified limits remove dependencies on L0s exit latency — making compliance testing simpler.

> 简化REPLAY_TIMER（8.0 GT/s+，建议所有速率）：Normal 24000-31000符号时间；Extended Synch 80000-100000符号时间。简化限值消除了对L0s退出延迟的依赖。

---

## Ack Latency Limits
## Ack延迟限值

Acks must be transmitted within AckNak_LATENCY_TIMER values (defined per data rate, link width, and Rx_MPS_Limit). These limits ensure the transmitter's Retry Buffer doesn't overflow. If the Ack Latency limit is exceeded and the transmitter runs out of Retry Buffer space, performance stalls.

> Ack必须在AckNak_LATENCY_TIMER值内传输（每数据速率、链路宽度和Rx_MPS_Limit定义）。这些限值确保发送端的Retry Buffer不溢出。若超过Ack Latency限值且发送端Retry Buffer空间耗尽，性能将停滞。

---

| English | 中文 | Notes |
|---------|------|-------|
| Ack / Nak | 正确认/否定确认 | DLLP |
| NEXT_TRANSMIT_SEQ | 下一发送序列号 | 12-bit |
| ACKD_SEQ | 已确认序列号 | 12-bit |
| NEXT_RCV_SEQ | 下一接收序列号 | 12-bit |
| REPLAY_NUM | 重放次数 | 3-bit |
| REPLAY_TIMER | 重放定时器 | |
| AckNak_LATENCY_TIMER | Ack延迟定时器 | |
| Retry Buffer | 重试缓冲区 | TX保留TLP副本 |
| LCRC (Link CRC) | 链路CRC | 32-bit, 多项式 04C11DB7h |
| Bad TLP / Bad DLLP | 坏TLP/DLLP | 报告性错误 |
| NAK_SCHEDULED | Nak已调度 | 标志 |
| Nullified TLP | 废弃TLP | 有意损坏以取消 |
