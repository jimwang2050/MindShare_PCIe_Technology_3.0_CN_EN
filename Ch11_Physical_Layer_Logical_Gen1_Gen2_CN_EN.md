# Chapter 11: Physical Layer — Logical (Gen1 and Gen2)
# 第11章：物理层——逻辑子层（Gen1与Gen2）

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 420–465 (46 pages)

---

## Physical Layer Overview / 物理层概述

The Physical Layer resides at the bottom of the interface between the external physical Link and the Data Link Layer. It converts outbound packets from the Data Link Layer into a serialized bit stream clocked onto all Lanes of the Link. The receive logic de-serializes the bits back into a Symbol stream, re-assembles the packets, and forwards TLPs and DLLPs up to the Data Link Layer.

> 物理层位于外部物理链路与数据链路层之间接口的底层。它将数据链路层的输出数据包转换为串行位流，按时钟发送到链路的所有通道上。接收逻辑将位反序列化为符号流，重新组装数据包，并将TLP和DLLP向上转发到数据链路层。

For Gen1 (2.5 GT/s) and Gen2 (5.0 GT/s), the Physical Layer uses **8b/10b encoding**. Each 8-bit byte is encoded into a 10-bit Symbol before transmission. The encoding guarantees: sufficient transitions (edges) for the receiver to recover the embedded clock; DC balance (equal numbers of 1s and 0s over time); special control characters (K-codes) used for packet framing and Ordered Set identification.

> Gen1 (2.5 GT/s)和Gen2 (5.0 GT/s)的物理层使用**8b/10b编码**。每个8位字节在发送前编码为10位Symbol。该编码保证：足够的跳变（边沿）供接收端恢复嵌入时钟；直流平衡（长期1和0数量相等）；用于数据包定帧和有序集识别的特殊控制字符（K码）。

---

## 8b/10b Encoding Details / 8b/10b编码详解

The 8b/10b encoder maps each 8-bit byte to a 10-bit Symbol from a table of 256 Data (D) symbols. The encoder maintains a **Running Disparity** (RD) counter tracking cumulative DC balance. For each byte, the encoder selects either the positive-disparity or negative-disparity version of the Symbol to drive RD toward zero. The RD toggles between negative (−1) and positive (+1).

> 8b/10b编码器将每个8位字节映射到256个数据(D)符号表中的10位Symbol。编码器维护**运行差异**(RD)计数器跟踪累积DC平衡。对于每个字节，编码器选择Symbol的正差异或负差异版本以将RD驱向零。RD在负(−1)和正(+1)之间切换。

**Special K-codes:** Beyond the 256 D symbols, 12 special K (control) symbols provide essential PCIe protocol functions:
- **K28.5 (COM):** First character of ALL Ordered Sets; used for Symbol Lock and de-skew
- **K28.3 (IDL):** Logical idle sent during L0 when no packets are pending
- **K28.7 (EIE):** Electrical Idle Exit signaling character
- **K28.1 (FTS):** Fast Training Sequence character for L0s exit
- **K28.0, K28.2, K28.4, K28.6, K23.7, K27.7, K29.7, K30.7:** SKP, EIOS, and training sequence characters

These K-codes are never used for data bytes. Their unique bit patterns make them unambiguously identifiable even in the presence of bit errors.

> **特殊K码：** 256个D符号外的12个特殊K(控制)符号：K28.5(COM)为所有有序集起始字符；K28.3(IDL)用于L0空闲；K28.7(EIE)电气空闲退出；K28.1(FTS)用于L0s退出；其他K码用于SKP、EIOS和训练序列。K码永不用于数据字节，其唯一位模式使其即使在位错误下也能被无误识别。

---

## Symbol Time and Effective Bandwidth / 符号时间与有效带宽

At 2.5 GT/s, each Symbol (10 bits) = 4 ns (10 bits / 2.5 Gbps). At 5.0 GT/s, each Symbol = 2 ns. The 8b/10b encoding overhead is 20% (8 data bits → 10 transmitted bits). Therefore:
- Gen1 effective bandwidth: 2.5 GT/s × 8/10 = 2.0 Gbps per Lane
- Gen2 effective bandwidth: 5.0 GT/s × 8/10 = 4.0 Gbps per Lane
- x16 Gen2: 4.0 × 16 = 64 Gbps ≈ 8 GB/s (both directions combined: ~16 GB/s)

> 2.5 GT/s下每Symbol(10位)=4ns；5.0 GT/s下每Symbol=2ns。8b/10b编码20%开销。Gen1有效带宽2.0 Gbps/通道，Gen2 4.0 Gbps/通道。x16 Gen2双向合计约16 GB/s。

---

## Packet Framing / 数据包定帧

The Physical Layer adds framing Symbols so the receiver can locate packet boundaries:
- **TLP Framing:** STP Symbol → TLP Data (with LCRC appended by DLL) → END (good) or EDB (LCRC failed)
- **DLLP Framing:** SDP Symbol → 6-byte DLLP → END

> 物理层添加定帧Symbol使接收端能定位数据包边界：TLP使用STP起始和END/EDB结束；DLLP使用SDP起始和END结束。EDB表示TLP未通过LCRC校验。

---

## Ordered Sets / 有序集

Ordered Sets begin with COM (K28.5) and serve Link management functions:

**TS1/TS2 (Training Sequences):** 16-Symbol sequences exchanged during training (see Ch14).

**SKP (Skip) Ordered Set:** Compensates for clock frequency differences (up to ±300 ppm). SKP sets are transmitted at fixed intervals — every 1180 Symbol Times for Gen1, every 590 for Gen2. If the receiver clock is faster than the transmitter, SKP symbols are discarded; if slower, extra SKP symbols are inserted. This prevents receiver elastic buffer overflow/underflow.

**EIOS (Electrical Idle Ordered Set):** Warns the receiver that the transmitter is about to enter Electrical Idle.

**FTS (Fast Training Sequence):** Sent when exiting L0s. The pattern helps the receiver re-acquire Bit Lock quickly.

> 有序集以COM(K28.5)开头：TS1/TS2用于训练(见Ch14)；SKP有序集补偿时钟频率差异(±300 ppm)，Gen1每1180 Symbol Time/Gen2每590发送，防止弹性缓冲器溢出/欠载；EIOS警告接收端发送端即将进入电气空闲；FTS帮助接收端从L0s快速退出。

---

## Scrambling / 扰码

To avoid repetitive patterns that cause EMI peaks and hamper clock recovery, all TLP/DLLP data and logical idle are scrambled using a 16-bit LFSR implementing the polynomial G(x) = X^16 + X^5 + X^4 + X^3 + 1. The LFSR is initialized to FFFFh and advanced once per Symbol (except during Ordered Sets, which bypass scrambling — they must be received unmodified). The receiver synchronizes its LFSR using the COM character.

> 为避免产生EMI峰值和阻碍时钟恢复的重复模式，使用16位LFSR实现多项式X^16+X^5+X^4+X^3+1对数据进行扰码。LFSR初始化为FFFFh，每Symbol前进一次（有序集不扰码）。接收端使用COM字符同步其LFSR。

---

## Byte Striping / 字节条带化

In multi-Lane Links, data bytes are distributed round-robin across Lanes. For an x4 Link, byte N goes to Lane (N mod 4). This parallel transmission is the basis for Link bandwidth scaling with width.

> 多通道链路中数据字节以轮询方式分布：x4链路中byte N去Lane (N mod 4)。这种并行传输是链路带宽随宽度缩放的基础。

---

## Physical Layer Logical Functions Summary / 物理层逻辑功能总结

| Function | Description |
|----------|-------------|
| 8b/10b Encode/Decode | Convert bytes ↔ 10b Symbols with DC balance |
| Scrambling/De-scrambling | LFSR-based to avoid repetitive patterns |
| Packet Framing | STP/SDP start markers, END/EDB end markers |
| Ordered Set Handling | Generate/recognize TS1, TS2, SKP, EIOS, FTS, EIEOS |
| Clock Tolerance Comp | SKP insertion/deletion for ±300 ppm |
| Byte Striping | Distribute bytes across Lanes |
| Lane-to-Lane De-skew | Align data from different Lanes |
| Polarity Inversion | Detect and correct swapped differential pairs |
| Electrical Idle Detect | Monitor for Electrical Idle entry/exit |
| LTSSM | Link Training and Status State Machine (see Ch14) |
