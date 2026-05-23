# Chapter 13: Physical Layer — Electrical
# 第13章：物理层——电气子层

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 506–560 (55 pages)

---

## Overview / 概述

The Electrical sub-block of the Physical Layer defines the analog characteristics of PCIe transmitters and receivers, including voltage levels, jitter budgets, rise/fall times, return loss, and the physical channel (PCB traces, connectors, cables). This chapter covers Gen1, Gen2, and Gen3 electrical parameters.

> 物理层的电气子层定义PCIe发送端和接收端的模拟特性，包括电压水平、抖动预算、上升/下降时间、回波损耗和物理信道（PCB走线、连接器、线缆）。本章涵盖Gen1、Gen2和Gen3电气参数。

---

## Differential Signaling / 差分信令

PCIe uses differential signaling on each Lane: two wires carrying equal and opposite signals (D+ and D−). Benefits: common-mode noise rejection, reduced EMI (the opposite fields cancel), and the ability to detect a signal even at very low amplitudes. The differential peak-to-peak voltage at the transmitter is typically 800-1200 mV for Gen1/Gen2.

> PCIe在每条通道上使用差分信令：两条线承载相等且相反的信号(D+和D−)。优势：共模噪声抑制、减少EMI（相反场抵消）、即使在极低幅度下也能检测信号。Gen1/Gen2发送端差分峰峰值电压通常为800-1200 mV。

---

## AC Coupling / 交流耦合

PCIe Links are AC-coupled: capacitors (typically 75-200 nF) are placed in series on each differential pair to block DC. This allows the transmitter and receiver to operate at different common-mode voltages. The capacitors must be located on the transmitter side for add-in cards and on the receiver side for system boards.

> PCIe链路为交流耦合：在每个差分对上串联电容（通常75-200 nF）以隔离直流。这允许发送端和接收端以不同的共模电压运行。电容必须位于扩展卡发送端侧和系统板接收端侧。

---

## Gen1/Gen2 Electrical Parameters / Gen1/Gen2电气参数

| Parameter | Gen1 (2.5 GT/s) | Gen2 (5.0 GT/s) |
|-----------|-----------------|------------------|
| Unit Interval (UI) | 400 ps | 200 ps |
| Differential TX Voltage | 800-1200 mVpp | 800-1200 mVpp |
| TX Rise/Fall Time | 0.125 UI min | 0.125 UI min |
| TX Total Jitter | 0.25 UI max | 0.25 UI max |
| RX Total Jitter Tolerance | 0.55 UI | 0.55 UI |
| Differential Return Loss | ≥ 10 dB | ≥ 10 dB |

> | 参数 | Gen1 (2.5 GT/s) | Gen2 (5.0 GT/s) |
> |------|-----------------|------------------|
> | 单位间隔(UI) | 400 ps | 200 ps |
> | 差分TX电压 | 800-1200 mVpp | 800-1200 mVpp |
> | TX上升/下降时间 | ≥0.125 UI | ≥0.125 UI |
> | TX总抖动 | ≤0.25 UI | ≤0.25 UI |
> | RX总抖动容限 | 0.55 UI | 0.55 UI |
> | 差分回波损耗 | ≥10 dB | ≥10 dB |

---

## De-emphasis (Gen2) / 去加重

At 5.0 GT/s, signal degradation due to channel losses becomes significant. **De-emphasis** reduces the amplitude of subsequent bits after a transition, pre-compensating for the low-pass filtering effect of the channel. Gen2 specifies -3.5 dB de-emphasis: the voltage of non-transition bits is reduced by 3.5 dB relative to transition bits. The receiver can be tuned to expect this de-emphasis level.

> 5.0 GT/s下信道损耗导致的信号退化变得显著。**去加重**降低跳变后后续位的幅度，预补偿信道的低通滤波效应。Gen2指定-3.5 dB去加重：非跳变位电压相对跳变位降低3.5 dB。接收端可调谐以期望此去加重水平。

---

## Equalization (Gen3) / 均衡

At 8.0 GT/s, simple de-emphasis is insufficient. Gen3 introduces full transmitter equalization with programmable coefficients:
- **Pre-cursor (C-1):** Adjusts the signal before the main transition to pre-compensate for ISI
- **Main cursor (C0):** The primary signal level
- **Post-cursor (C+1):** Adjusts the signal after the main transition to compensate for reflections and tail effects

The coefficients are negotiated during Recovery.Equalization using the EQ Info fields in TS1/TS2 blocks. Multiple presets are defined for common channel characteristics, with the ability to fine-tune via coefficient requests.

> 8.0 GT/s下简单去加重不足够。Gen3引入带可编程系数的全发送端均衡：前标(C-1)预补偿ISI；主标(C0)主信号水平；后标(C+1)补偿反射和尾部效应。系数在Recovery.Equalization期间使用TS1/TS2块中的EQ Info字段协商。多个预设针对常见信道特性定义，可通过系数请求微调。

---

## Receiver Characteristics / 接收端特性

The receiver must reliably extract data from a signal degraded by channel losses, reflections, crosstalk, and jitter. Key receiver requirements:
- **Continuous Time Linear Equalizer (CTLE):** Amplifies high-frequency components to compensate for channel roll-off
- **Decision Feedback Equalizer (DFE) (optional, Gen3):** Uses previously detected bits to cancel ISI on the current bit
- **Clock and Data Recovery (CDR):** Extracts a sampling clock from the incoming data stream
- **Jitter tolerance:** Must handle the transmitter's jitter plus channel-induced jitter

> 接收端必须从因信道损耗、反射、串扰和抖动而降质的信号中可靠提取数据。关键要求：CTLE放大高频分量补偿信道衰减；DFE(Gen3可选)用先前检测的位消除当前位的ISI；CDR从输入数据流提取采样时钟；抖动容限必须处理发送端加信道引入的抖动。

---

## Jitter Budget / 抖动预算

The total jitter budget is divided among the transmitter, channel (interconnect), and receiver. At Gen3 speeds, jitter margins are tighter because the UI is only 125 ps. The system design must ensure that total jitter at the receiver does not exceed 0.55 UI for Gen1/Gen2 and a tighter spec for Gen3. Jitter components include:
- **Random Jitter (RJ):** Gaussian, unbounded, caused by thermal noise
- **Deterministic Jitter (DJ):** Bounded, caused by ISI, duty-cycle distortion, crosstalk
- **Total Jitter (TJ) at BER 10^-12:** TJ = DJ + 14.069 × RJ (for Gen1/Gen2)

> 总抖动预算在发送端、信道（互连）和接收端之间分配。Gen3速度下抖动裕量更紧，因为UI仅为125 ps。抖动分量包括：随机抖动(RJ)为高斯无界抖动由热噪声引起；确定性抖动(DJ)为有界抖动由ISI、占空比失真、串扰引起；总抖动(TJ)=DJ+14.069×RJ(Gen1/Gen2)。

---

## The Physical Channel / 物理信道

The PCIe channel consists of PCB traces, vias, connectors, and possibly cables. Key channel parameters:
- **Insertion Loss:** Signal attenuation vs frequency. At Gen3 Nyquist (4 GHz), typical loss is 15-25 dB for a reasonable channel length
- **Return Loss:** Signal energy reflected back due to impedance discontinuities. Minimum 10 dB for Gen1/Gen2
- **Crosstalk:** Unwanted coupling between adjacent Lanes. Must be below specified limits

Channel reach decreases as data rate increases. Gen1 can typically drive 20+ inches of FR4; Gen2 ~15 inches; Gen3 ~10 inches (with equalization).

> PCIe信道由PCB走线、过孔、连接器和可能的线缆组成。关键参数：插入损耗(信号衰减vs频率)，Gen3 Nyquist(4 GHz)典型损耗15-25 dB；回波损耗(阻抗不连续反射的能量)，Gen1/Gen2 ≥10 dB；串扰(相邻通道间不期望的耦合)。随数据速率增加信道延伸减少：Gen1~20+英寸FR4，Gen2~15英寸，Gen3~10英寸(使用均衡)。

---

## Spread Spectrum Clocking (SSC) / 展频时钟

SSC modulates the reference clock frequency by typically 0.5% down-spread at 30-33 kHz. This reduces EMI peak amplitude by spreading the clock energy over a wider frequency band. PCIe supports SSC but requires that both ends of a Link be configured consistently (common clock with SSC or separate clocks without SSC).

> SSC通常以30-33 kHz、0.5%向下展频调制参考时钟频率。通过在更宽频带上展布时钟能量降低EMI峰值。PCIe支持SSC但要求链路两端一致配置（公共时钟带SSC或独立时钟不带SSC）。

---

## Low-Power Electrical States / 低功耗电气状态

- **Electrical Idle:** Differential voltage < 20 mVpp. Transmitter drivers are in high-impedance state. The receiver must detect exit (voltage rising above threshold) reliably.
- **L0s/L1:** Transmitter in Electrical Idle. PLLs may remain active (L0s) or may be disabled (L1).
- **Beacon:** Periodic signal on idle Lanes for L2 wakeup. Must be detectable even with significant signal attenuation.

> 电气空闲(差分电压<20mVpp，发送端驱动器高阻态)；L0s/L1(发送端电气空闲)；Beacon(L2唤醒的周期信号，即使信号显著衰减也必须可检测)。
