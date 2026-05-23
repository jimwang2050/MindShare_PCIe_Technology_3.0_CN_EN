# Chapter 14: Link Initialization & Training
# 第14章：链路初始化与训练

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 564–703 (140 pages)

---

## LTSSM Overview
## LTSSM概述

The **Link Training and Status State Machine (LTSSM)** is the heart of PCIe Link management. It brings up, maintains, recovers, and powers down the Link. The LTSSM has 11 major states, each with multiple sub-states.

> **链路训练与状态机（LTSSM）**是PCIe链路管理的核心。它启动、维护、恢复和关闭链路。LTSSM有11个主要状态，每个都有多个子状态。

---

## LTSSM State Descriptions
## LTSSM状态描述

### Detect
### 检测

**Detect.Quiet**: Initial state — waits 2 ms then transitions to Detect.Active. / 初始状态——等待2 ms后转换到Detect.Active。
**Detect.Active**: Performs **Receiver Detection** (charge/discharge Lane, measure RC time constant). Rx detected → Polling; not detected → Detect.Quiet.

> Detect.Active：执行接收端检测（充放电Lane、测量RC时间常数）。检测到Rx→Polling；未检测到→Detect.Quiet。

### Polling
### 轮询

**Polling.Active**: Transmits TS1 Ordered Sets at the highest supported data rate. Achieves bit lock (PLL locks to incoming data), Symbol/Block lock (COM detection or Sync Header alignment), and Lane polarity inversion (if needed). Exits to Configuration after receiving 8 consecutive matching TS1 Ordered Sets with valid Lane numbers.

**Polling.Compliance**: Entered when directed (test equipment) or if Configuration times out — transmits compliance patterns for testing.

> Polling.Active：以最高支持数据速率发送TS1有序集。实现位锁定（PLL锁定到输入数据）、符号/Block锁定（COM检测或Sync Header对齐）和Lane极性反转（如需要）。收到8个连续匹配TS1有序集（带有效Lane编号）后退出到Configuration。Polling.Compliance：被测试设备指示时或Configuration超时时进入——发送合规码型。

### Configuration
### 配置

**Configuration.Linkwidth.Start**: Transmits TS1 with proposed Link width. Negotiates actual width. / 发送提议链路宽度的TS1，协商实际宽度。
**Configuration.Linkwidth.Accept**: Acknowledges negotiated width. / 确认协商的宽度。
**Configuration.Lanenum.Wait**: Waits for Lane number assignment from upstream. / 等待上游分配Lane编号。
**Configuration.Lanenum.Accept**: Accepts Lane numbers; performs **Lane-to-Lane De-skew** — aligns received data across multiple Lanes to compensate for different trace lengths. / 接受Lane编号；执行Lane-to-Lane去偏移——对齐多Lane接收数据以补偿不同走线长度。
**Configuration.Complete**: Transmits TS2 Ordered Sets; confirms readiness; exits to L0. / 发送TS2有序集；确认就绪；退出到L0。
**Configuration.Idle**: Additional state for higher data rates (Gen3). / 更高数据速率的附加状态。

### Recovery
### 恢复

Recovery is entered to retrain the Link after errors, or for directed speed/width changes:
- **Recovery.RcvrLock**: Re-establish bit/symbol lock / 重建位/符号锁定
- **Recovery.RcvrCfg**: Re-establish Lane numbering and de-skew / 重建Lane编号和去偏移
- **Recovery.Speed**: Entered for directed data rate change / 定向速率变更时进入
- **Recovery.Equalization** (Gen3): Execute Link Equalization Procedure (Phase 0–3) / 执行链路均衡过程

### L0 / L0s / L1 / L2
### L0/L0s/L1/L2

- **L0**: Normal operation — all TLPs, DLLPs, and Logical Idle flow / 正常运行
- **L0s**: Tx-only standby, fast exit (tens of ns). TX sends Electrical Idle; RX stays active. Exit via FTS Ordered Set training. / 仅TX待机，快速退出（数十ns）
- **L1**: Bidirectional low power. Enter via PM DLLP handshake. Exit via Electrical Idle exit + FTS/TS1 retraining. CLKREQ# signal manages Refclk. / 双向低功耗
- **L2**: Deep power saving. Power/clocks may be removed. Wake via Beacon or WAKE#. / 深度省电。可移除电源/时钟。通过Beacon或WAKE#唤醒

### Disabled / Loopback / Hot Reset
### 禁用/环回/热复位

- **Disabled**: Link disabled by software (Link Disable bit Set) / 软件禁用
- **Loopback**: Test/debug mode — Entry (TS1 with Loopback bit) → Active (loop data back) → Exit (timeout/EI). Retimer loopback also supported. / 测试/调试模式
- **Hot Reset**: In-band reset via TS1 Ordered Sets (Hot Reset bit Set), propagated upstream through hierarchy. Duration: 2 ms. / 带内复位（通过TS1有序集，Hot Reset位置位），沿层级向上传播

---

## Training Sequences: TS1 and TS2
## 训练序列：TS1与TS2

TS1/TS2 Ordered Sets carry Link training information: Link Number, Lane Number, Data Rate Identifier, Training Control bits (Hot Reset, Disable Link, Loopback, Compliance Receive), and Equalization Control (presets, coefficients, FS/LF request).

> TS1/TS2有序集承载链路训练信息：链路编号、Lane编号、数据速率标识符、训练控制位和均衡控制。

---

## Link Equalization (Gen3)
## 链路均衡 (Gen3)

At 8.0 GT/s, equalization compensates for channel loss to open the eye at the receiver. Four phases:
- **Phase 0**: Upstream Port transmits with initial presets (full swing) / 上行端口以初始预设值发送
- **Phase 1**: Downstream Port adjusts upstream TX coefficients / 下行端口调整上行TX系数
- **Phase 2**: Upstream Port adjusts downstream TX coefficients / 上行端口调整下游TX系数
- **Phase 3**: Fine-tuning and finalization / 微调与完成

Each phase uses TS1 Ordered Sets with Equalization Control fields. The receiver evaluates the eye and requests coefficient changes (increment/decrement pre-cursor c-1 or post-cursor c+1) to optimize the eye opening.

> 每个阶段使用带均衡控制字段的TS1有序集。接收端评估眼图并请求系数变更（增量/减量前标c-1或后标c+1）以优化眼图张开度。

---

| English | 中文 | Notes |
|---------|------|-------|
| LTSSM | 链路训练与状态机 | 11个主要状态 |
| Detect | 检测 | Rx存在检测 |
| Polling | 轮询 | 位/符号锁定 |
| Configuration | 配置 | 宽度/Lane/去偏移 |
| Recovery | 恢复 | 重训练/均衡 |
| TS1 / TS2 | 训练序列1/2 | |
| Lane De-skew | Lane去偏移 | 多Lane对齐 |
| FTS (Fast Training Sequence) | 快速训练序列 | L0s退出 |
| Equalization | 均衡 | Gen3, Phase 0–3 |
| Hot Reset | 热复位 | 带内复位, 2ms |
| Loopback | 环回 | 测试模式 |
| CLKREQ# | 时钟请求信号 | L1 Refclk管理 |
| Beacon | 信标 | L2唤醒信号 |
