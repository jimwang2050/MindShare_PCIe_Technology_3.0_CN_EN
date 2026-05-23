# Chapter 12: Physical Layer — Logical (Gen3)
# 第12章：物理层——逻辑子层（Gen3）

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 466–506 (41 pages)

---

## Gen3 Overview: Eliminating 8b/10b / Gen3概述：消除8b/10b编码

The most significant change for Gen3 (8.0 GT/s) is the elimination of 8b/10b encoding. Instead, Gen3 uses **128b/130b encoding** with scrambling. This reduces the encoding overhead from 20% to approximately 1.54% (2 Sync Header bits per 128-bit block), nearly doubling the effective bandwidth per Lane to 8.0 Gbps.

> Gen3(8.0 GT/s)最重大的变化是消除了8b/10b编码，改用带扰码的**128b/130b编码**。这将编码开销从20%降至约1.54%(每128位块2位Sync Header)，每通道有效带宽几乎翻倍至8.0 Gbps。

The key motivations for dropping 8b/10b encoding:
1. **Bandwidth efficiency:** 8b/10b wastes 20% of raw bandwidth. 128b/130b achieves ~98.5% efficiency.
2. **Frequency scaling:** To double bandwidth from 5.0 to 10.0 GT/s (with 8b/10b) would require doubling the frequency — extremely challenging for PCB design. At 8.0 GT/s with 128b/130b, the effective bandwidth is higher than 10 GT/s 8b/10b would have provided, but at only 8.0 GT/s line rate.
3. **Scrambling replaces DC balance:** The scrambler provides statistically DC-balanced output, eliminating the need for disparity tracking.

> 放弃8b/10b编码的关键动机：
> 1. **带宽效率：** 8b/10b浪费20%原始带宽，128b/130b实现约98.5%效率。
> 2. **频率缩放：** 用8b/10b从5.0翻到10.0 GT/s需要频率翻倍——对PCB设计极难。8.0 GT/s + 128b/130b提供比10 GT/s 8b/10b更高的有效带宽，但仅需8.0 GT/s线速率。
> 3. **扰码替代DC平衡：** 扰码器提供统计上DC平衡的输出，无需差异跟踪。

---

## 128b/130b Block Structure / 128b/130b块结构

Each transmitted block is 130 bits (16.25 bytes): a 2-bit **Sync Header** followed by a 128-bit **Payload**. The Sync Header indicates:
- **01b:** Payload contains a Data Block (TLP/DLLP data or logical idle)
- **10b:** Payload contains an Ordered Set Block (TS1, TS2, SKP, EIOS, EIEOS, etc.)

The Sync Header values 00b and 11b are illegal — their detection indicates a framing error.

> 每个发送块为130位(16.25字节)：2位**Sync Header**后跟128位**Payload**。Sync Header指示：01b=数据块(TLP/DLLP/逻辑空闲)，10b=有序集块(TS1/TS2/SKP/EIOS/EIEOS等)。00b和11b为非法值——其检测指示定帧错误。

---

## Block-Level Framing: Ordered Set Blocks vs Data Blocks / 块级定帧

This is fundamentally different from Gen1/Gen2, where the COM character (K28.5) distinguished Ordered Sets from TLPs/DLLPs at the Symbol level. In Gen3, the Sync Header provides this distinction at the Block level. The receiver first achieves Block Lock (identifying 130-bit block boundaries), then examines the Sync Header to determine the block type.

> 这与Gen1/Gen2根本不同——Gen1/Gen2中COM字符(K28.5)在符号级区分有序集与TLP/DLLP。Gen3中Sync Header在块级提供此区分。接收端首先实现块锁定（识别130位块边界），然后检查Sync Header确定块类型。

Within a Data Block (01b), the 128-bit payload is further divided into framing tokens that indicate the start/end of TLPs, DLLPs, and logical idle segments. This is analogous to the STP/SDP/END/IDL Symbols in Gen1/Gen2 but implemented as bit-level tokens within the 128-bit payload.

> 数据块(01b)内128位payload进一步划分为定帧令牌，指示TLP/DLLP/逻辑空闲片段的开始/结束。这类似于Gen1/Gen2中的STP/SDP/END/IDL Symbol，但实现为128位payload内的位级令牌。

---

## Scrambling in Gen3 / Gen3扰码

Gen3 uses a 23-bit LFSR (polynomial: X^23 + X^21 + X^16 + X^8 + X^5 + X^2 + 1) applied to the entire 128-bit payload. The Sync Header is NOT scrambled. The scrambler provides adequate DC balance statistically, eliminating the need for Running Disparity tracking. A **Precoder** (1/(1+D) filter) is applied after scrambling at 8.0 GT/s to shape the output spectrum for better channel characteristics.

> Gen3使用23位LFSR(多项式：X^23+X^21+X^16+X^8+X^5+X^2+1)对整个128位payload进行扰码。Sync Header不扰码。扰码器在统计上提供足够的DC平衡，无需运行差异跟踪。8.0 GT/s下扰码后应用**预编码器**(1/(1+D)滤波器)整形输出频谱。

---

## Ordered Sets in Gen3 / Gen3中的有序集

Gen3 Ordered Sets are fundamentally different from Gen1/Gen2:
- **No COM character:** Ordered Sets are identified by Sync Header = 10b, not by a special starting character
- **Block-aligned:** Each Ordered Set is a complete 130-bit block
- **TS1/TS2 redesign:** The 16-Symbol TS1/TS2 structure is replaced by a block-based format with equivalent fields (Link#, Lane#, N_FTS, Rate ID, Training Control, EQ Info)
- **EIEOS (Electrical Idle Exit Ordered Set):** A new Ordered Set type used specifically for exiting Electrical Idle at Gen3 speeds. EIEOS provides a longer, more distinctive pattern for reliable detection at the higher data rate.
- **SDS (Start of Data Stream):** Sent at the end of training to signal the transition to normal data transmission

> Gen3有序集与Gen1/Gen2根本不同：无序集由Sync Header=10b标识而非特殊起始字符；每个有序集是一个完整的130位块；TS1/TS2的16-Symbol结构被替换为包含等效字段的块格式；EIEOS是Gen3速度下专门用于退出电气空闲的新有序集类型；SDS在训练结束时发送，信号转换到正常数据传输。

---

## Gen3 Key Features Summary / Gen3关键特性总结

| Feature | Gen1/Gen2 | Gen3 |
|---------|-----------|------|
| Encoding | 8b/10b | 128b/130b |
| Line rate | 2.5 / 5.0 GT/s | 8.0 GT/s |
| Overhead | 20% | ~1.54% |
| DC Balance | Running Disparity | Scrambler + Precoder |
| Ordered Set ID | COM character | Sync Header = 10b |
| Block size | 10 bits (Symbol) | 130 bits (Block) |
| De-skew reference | COM character | Sync Header + TS patterns |
| Equalization | De-emphasis only | Full TX EQ with coefficient exchange |
| L0s exit | FTS | FTS via blocks |
| Electrical Idle | EIOS | EIOS block + EIEOS |
