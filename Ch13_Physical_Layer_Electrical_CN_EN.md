# Chapter 13: Physical Layer — Electrical
# 第13章：物理层 — 电气子层

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 506–563 (58 pages)

---

## Differential Signaling
## 差分信令

PCIe uses differential signaling: a pair of wires (D+ and D−) per Lane. Benefits: noise immunity (common mode noise rejection), lower EMI, and ability to use lower voltage swings. The differential peak-to-peak voltage at the transmitter pin is typically 800–1200 mV for Gen1/Gen2.

> PCIe使用差分信令：每个Lane一对导线（D+和D−）。优点：噪声免疫（共模噪声抑制）、低EMI和可使用低电压摆幅。Gen1/Gen2发送端引脚差分峰峰值电压通常800–1200 mV。

---

## AC Coupling
## AC耦合

Each PCIe Lane has AC coupling capacitors (75–200 nF for Gen1/Gen2, 176–265 nF for Gen3) on the Transmitter side. AC coupling blocks DC offsets and allows different common-mode voltages between transmitter and receiver. The capacitors must be rated to handle the highest switching frequency.

> 每个PCIe Lane在发送端一侧具有AC耦合电容（Gen1/Gen2：75–200 nF；Gen3：176–265 nF）。AC耦合阻断DC偏移，允许发送端和接收端之间不同的共模电压。电容必须能承受最高开关频率。

---

## Receiver Detection
## 接收端检测

During the Detect LTSSM state, the transmitter performs **Receiver Detection**: charging the Lane to a known voltage, then measuring the discharge time constant. A receiver presents a known impedance (40–60Ω to ground at each pin), resulting in a measurable RC time constant. This detects whether a receiver (and thus a Link partner) is present.

> 在Detect LTSSM状态，发送端执行**接收端检测**：将Lane充电至已知电压，然后测量放电时间常数。接收端呈现已知阻抗（每引脚40–60Ω到地），产生可测量的RC时间常数。由此检测是否存在接收端（即链路伙伴）。

---

## Electrical Idle
## 电气空闲

Electrical Idle is a low-power state where the transmitter drives both D+ and D− to the same DC common mode voltage (differential voltage ≈ 0V). The receiver detects Electrical Idle when the differential signal amplitude drops below a threshold (typically 65–175 mV peak-to-peak).

> 电气空闲是一种低功耗状态，发送端将D+和D−驱动至相同的DC共模电压（差分电压≈0V）。当差分信号幅度降至阈值（典型65–175 mV峰峰值）以下时，接收端检测到电气空闲。

**Electrical Idle Exit**: The transmitter sends EIOS/EIEOS Ordered Sets before exiting Electrical Idle. The receiver must distinguish the true exit from noise: EIEOS (mandatory at 8.0 GT/s+) provides a robust, known pattern for reliable detection.

> **电气空闲退出**：发送端在退出电气空闲前发送EIOS/EIEOS有序集。接收端必须区分真正的退出与噪声：EIEOS（8.0 GT/s+强制）提供强健的已知模式用于可靠检测。

---

## Transmitter and Receiver Specifications
## 发送端与接收端规范

Key TX parameters: Differential voltage swing, rise/fall time, jitter (total, random, deterministic), AC common mode voltage. Key RX parameters: Input impedance (40–60Ω to ground), input sensitivity, jitter tolerance, return loss.

**De-emphasis (Gen1/Gen2)**: Pre-emphasis of -3.5 dB or -6.0 dB on transitions to compensate for channel loss at higher frequencies.

**Equalization (Gen3)**: Mandatory TX equalization (3-tap FIR: pre-cursor c-1, main c0, post-cursor c+1) and RX adaptive equalization (CTLE + DFE) to open closed eyes at 8.0 GT/s.

> 关键TX参数：差分电压摆幅、上升/下降时间、抖动（总/随机/确定性）、AC共模电压。关键RX参数：输入阻抗、输入灵敏度、抖动容限、回波损耗。Gen1/Gen2去加重：跳变上-3.5 dB或-6.0 dB预加重以补偿高频通道损耗。Gen3均衡：强制TX均衡（3抽头FIR）和RX自适应均衡（CTLE+DFE）以在8.0 GT/s下打开闭眼。

---

## Link Power Management States
## 链路电源管理状态

- **L0**: Fully active / 完全活跃
- **L0s**: Transmitter-only low power (fast entry/exit, tens of ns) / 仅TX低功耗
- **L1**: Bidirectional low power (lower power than L0s, slower exit) / 双向低功耗
- **L2**: Deep power saving (power and clocks may be removed, wake via Beacon or WAKE#) / 深度省电
- **L3**: Power off / 断电

---

| English | 中文 | Notes |
|---------|------|-------|
| AC Coupling | AC耦合 | 75–265 nF |
| Differential Signaling | 差分信令 | D+/D−对 |
| De-emphasis | 去加重 | Gen1/Gen2, -3.5/-6.0 dB |
| Equalization | 均衡 | Gen3, FIR+CTLE+DFE |
| Electrical Idle | 电气空闲 | 低功耗状态 |
| EIOS / EIEOS | 电气空闲(退出)有序集 | |
| Receiver Detection | 接收端检测 | RC时间常数 |
| L0 / L0s / L1 / L2 / L3 | 链路电源状态 | |
| Beacon | 信标 | L2唤醒信号 |
