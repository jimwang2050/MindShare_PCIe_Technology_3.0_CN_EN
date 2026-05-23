# Chapter 12: Physical Layer — Logical (Gen3)
# 第12章：物理层 — 逻辑子层 (Gen3)

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 466–505 (40 pages)

---

## Gen3: 128b/130b Encoding
## Gen3：128b/130b编码

PCIe Gen3 (8.0 GT/s) achieved double bandwidth without doubling frequency — instead of going to 10 GT/s with 8b/10b, the spec uses 8.0 GT/s with **128b/130b encoding**, reducing overhead from 25% to ~1.5%. Key architecture change: each Block is 130 bits (2-bit Sync Header + 128-bit Payload). The Sync Header (01b=Data Block, 10b=Ordered Set Block) replaces the 8b/10b comma mechanism for block alignment.

> PCIe Gen3 (8.0 GT/s) 在不翻倍频率的情况下实现双倍带宽——不采用10 GT/s 8b/10b，而是使用8.0 GT/s **128b/130b编码**，将开销从25%降至约1.5%。关键架构变化：每个Block为130位（2位Sync Header + 128位载荷）。Sync Header（01b=数据Block，10b=有序集Block）替代8b/10b逗号机制实现Block对齐。

---

## Block Alignment
## Block对齐

Block alignment uses the 2-bit Sync Header pattern. The receiver monitors the bit stream to find the Sync Header boundary. When 01b or 10b appears at regular 130-bit intervals, alignment is achieved. If > 1% Sync Headers are 00b/11b (invalid), the receiver reports an error.

> Block对齐利用2位Sync Header模式。接收端监控比特流，找到Sync Header边界。当01b或10b以规律性的130位间隔出现时，对齐完成。若>1% Sync Header为00b/11b（无效），接收端报告错误。

---

## Data Blocks and Framing Tokens
## 数据Block与成帧令牌

In Gen3 Non-Flit Mode, Data Blocks use **Framing Tokens** (1 Symbol each) to convey TLP/DLLP boundaries:
- **IDL** (1Eh): Logical Idle / 逻辑空闲
- **STP** (FCh): Start of TLP / TLP开始
- **SDP** (5Ch): Start of DLLP / DLLP开始
- **END** (F7h): End good / 结束良好
- **EDB** (F5h): End bad (nullified TLP) / 结束坏

Framing Tokens are placed in the Data Stream to locate packet boundaries. The receiver uses tokens to reconstruct TLPs/DLLPs from the incoming byte stream.

> 成帧令牌放置在数据流中以定位包边界。接收端使用令牌从输入字节流重建TLP/DLLP。

---

## Gen3 Scrambling
## Gen3加扰

Gen3 uses a 23-bit LFSR with polynomial **G(x) = X²³ + X²¹ + X¹⁶ + X⁸ + X⁵ + X² + 1**. The scrambler advances 128 bits per Data Block (held for Ordered Set Blocks). The LFSR is initialized/reloaded at the start of each Data Stream (after SDS Ordered Set).

> Gen3使用23位LFSR，多项式**G(x) = X²³ + X²¹ + X¹⁶ + X⁸ + X⁵ + X² + 1**。加扰器每数据Block推进128位（有序集Block期间保持）。LFSR在每次数据流开始时（SDS有序集之后）初始化/重载。

---

## Ordered Sets in Gen3
## Gen3中的有序集

Gen3 Ordered Sets (TS1/TS2, SKP, EIOS, EIEOS, SDS, FTS) use a 16-Symbol format on each Lane. Key differences from Gen1/2:
- **EIEOS** (Electrical Idle Exit Ordered Set): New for Gen3, mandatory — provides robust Electrical Idle exit detection
- SKP Ordered Set length varies to accommodate clock rate differences between Refclk and data rate
- SDS (Start of Data Stream) marks the transition from Ordered Set blocks to Data blocks

> Gen3有序集（TS1/TS2、SKP、EIOS、EIEOS、SDS、FTS）在每个Lane上使用16符号格式。与Gen1/2关键区别：EIEOS（Gen3新增，强制——提供强健的电气空闲退出检测）；SKP有序集长度可变以容纳Refclk与数据速率之间的时钟速率差异；SDS标记从有序集Block到数据Block的转换。

---

## Gen3 Key Enhancements
## Gen3关键增强

- Eliminated 20% overhead of 8b/10b (128b/130b: ~1.5%) / 消除8b/10b的20%开销
- Same TX/RX design at 8.0 GT/s as 5.0 GT/s (no frequency doubling of analog front-end) / 8.0 GT/s与5.0 GT/s使用相同TX/RX设计
- Transmitter Equalization mandatory for Gen3 / 发送端均衡Gen3强制要求
- Receiver CTLE + DFE equalization for channel compensation / 接收端CTLE+DFE均衡用于通道补偿
- EIEOS for robust Electrical Idle exit / EIEOS用于强健电气空闲退出

| English | 中文 |
|---------|------|
| 128b/130b Encoding | 128b/130b编码 |
| Sync Header | 同步头 |
| Block Alignment | Block对齐 |
| Framing Token | 成帧令牌 |
| IDL / STP / SDP / END / EDB | 空闲/TLP开始/DLLP开始/结束良好/结束坏 |
| EIEOS | 电气空闲退出有序集 |
| SDS | 数据流开始有序集 |
