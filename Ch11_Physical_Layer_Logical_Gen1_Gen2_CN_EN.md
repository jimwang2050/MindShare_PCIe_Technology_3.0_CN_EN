# Chapter 11: Physical Layer — Logical (Gen1 and Gen2)
# 第11章：物理层 — 逻辑子层 (Gen1/Gen2)

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 420–465 (46 pages)

---

## 8b/10b Encoding
## 8b/10b编码

PCIe Gen1 (2.5 GT/s) and Gen2 (5.0 GT/s) use **8b/10b encoding**. Every 8-bit data character is encoded into a 10-bit Symbol. The encoding guarantees:
- **DC Balance**: Equal numbers of 1s and 0s (or close), preventing baseline wander
- **Sufficient Transitions**: Enough bit transitions for clock recovery (max run length = 5)
- **Special Characters**: 12 K-Codes for control (framing, Ordered Sets, comma detection)

> PCIe Gen1 (2.5 GT/s) 和 Gen2 (5.0 GT/s) 使用**8b/10b编码**。每8位数据字符被编码为10位符号。编码保证：DC平衡（1和0数量相等或接近，防止基线漂移）、足够跳变（时钟恢复用，最大运行长度=5）、12个特殊K码用于控制（成帧、有序集、逗号检测）。

**Running Disparity (RD)**: Tracks the cumulative DC imbalance. Encoders choose the positive or negative disparity version of each 10-bit Symbol to maintain overall balance. Started negative after reset.

> **运行差异 (RD)**：跟踪累积DC不平衡。编码器选择每个10位符号的正差异或负差异版本以维持总体平衡。复位后从负差异开始。

## Symbol Encoding / 符号编码

8-bit data: HGF EDCBA → 10-bit Symbols: abcdei fghj. Control bit (Z) distinguishes D-characters (data) from K-characters (special). COM symbol (K28.5, 0011111010b or 1100000101b) is the comma for symbol/word alignment — its unique 7-bit pattern never appears in any other Symbol or across Symbol boundaries.

> 8位数据HGF EDCBA映射到10位符号abcdei fghj。控制位(Z)区分D字符（数据）和K字符（特殊）。COM符号（K28.5，0011111010b或1100000101b）是符号/字对齐的逗号——其独特的7位模式不会出现在任何其他符号或跨符号边界。

---

## Framing: STP, SDP, END, EDB
## 成帧：STP、SDP、END、EDB

TLPs are framed with:
- **STP** (Start of TLP, K27.7): Marks TLP beginning / 标记TLP开始
- **SDP** (Start of DLLP, K28.2): Marks DLLP beginning / 标记DLLP开始
- **END** (End Good, K29.7): Marks normal end / 标记正常结束
- **EDB** (End Bad, K30.7): Marks end of nullified TLP / 标记废弃TLP结束

Logical Idle (00h with D character control) fills gaps when no packets are being transmitted.

> 逻辑空闲（00h，D字符控制）在无包传输时填充间隙。成帧符号使得接收端能够定位每个TLP/DLLP的精确边界而无需事先知道长度。

---

## Data Scrambling
## 数据加扰

Gen1/Gen2 use a 16-bit LFSR (Linear Feedback Shift Register) with polynomial: **G(x) = X¹⁶ + X⁵ + X⁴ + X³ + 1**. Scrambling reduces EMI by eliminating repetitive patterns. The LFSR advances 8 bits per character; COM symbol resets/reloads the scrambler to FFFFh.

> Gen1/Gen2使用16位LFSR（线性反馈移位寄存器），多项式**G(x) = X¹⁶ + X⁵ + X⁴ + X³ + 1**。加扰通过消除重复模式降低EMI。LFSR每字符推进8位；COM符号复位/重载加扰器至FFFFh。

---

## 8b/10b Error Detection
## 8b/10b错误检测

Three classes of 8b/10b errors:
1. **Code Violation**: Received 10-bit pattern doesn't match any valid Symbol / 接收的10位模式不匹配任何有效符号
2. **Disparity Error**: Symbol has correct format but wrong disparity for current RD / 符号格式正确但差异与当前RD不符
3. **K-Code Error**: Non-K Symbol received where a K Symbol was expected / 在期望K符号的位置收到非K符号

> 三类8b/10b错误：Code Violation（无效10位模式）、Disparity Error（差异不匹配）、K-Code Error（预期K码处收到非K码）。错误被报告为物理层错误，可能触发LTSSM恢复或链路重训练。

---

## Serialization and De-serialization
## 串行化与反串行化

Symbols are serialized and transmitted LSB-first on each Lane. In a multi-Lane Link, bytes are distributed across Lanes byte-by-byte (byte striping) for even loading and to minimize per-Lane buffering.

> 符号被串行化并以LSB优先在每个Lane上传输。多Lane链路中，字节按逐字节方式跨Lane分配（字节条带化/byte striping），以实现均匀加载并最小化每Lane缓冲。

---

| English | 中文 | Notes |
|---------|------|-------|
| 8b/10b Encoding | 8b/10b编码 | 25% overhead |
| RD (Running Disparity) | 运行差异 | DC平衡 |
| COM (K28.5) | 逗号符号 | 符号对齐 |
| STP / SDP | TLP/DLLP开始 | K27.7/K28.2 |
| END / EDB | 结束良好/坏 | K29.7/K30.7 |
| LFSR | 线性反馈移位寄存器 | 加扰 |
| Scrambling | 加扰 | 降低EMI |
| Code Violation | 编码违例 | 无效10位模式 |
| Disparity Error | 差异错误 | RD不匹配 |
| Byte Striping | 字节条带化 | 跨Lane分布 |
