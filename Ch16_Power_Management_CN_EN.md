# Chapter 16: Power Management
# 第16章：电源管理

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 762–851 (90 pages)

---

## Power Management Overview
## 电源管理概述

PCIe defines two independent power management mechanisms: **PCI-compatible PM** (legacy, register-based) and **PCIe Native PM** (Active State Power Management — ASPM). They operate independently but must coordinate.

> PCIe定义了两种独立的电源管理机制：**PCI兼容PM**（传统，基于寄存器）和**PCIe原生PM**（活跃状态电源管理——ASPM）。两者独立运行但必须协调。

---

## Device Power States (D-States)
## 设备电源状态 (D状态)

| State | 状态 | Power | Wake-up Latency | Context |
|-------|------|-------|-----------------|---------|
| **D0** | 全功率 | Maximum | N/A | Full context maintained |
| **D1** | 中低功率 | Less than D0 | Medium | Partial context lost |
| **D2** | 更低功率 | Less than D1 | Slower | More context lost |
| **D3hot** | 极低功率 | Minimal | Longest | Most context lost; PME enabled |
| **D3cold** | 断电 | Power removed | Cold boot | All context lost; Aux power for wake |

D-state transitions are initiated by software writing to the Power Management Control/Status Register (PMCSR) in PCI-compatible configuration space. D3hot → D0 transition is a full device re-initialization.

> D状态转换由软件写PCI兼容配置空间中的电源管理控制/状态寄存器（PMCSR）发起。D3hot→D0转换需要完整的设备重新初始化。

---

## ASPM: Active State Power Management
## ASPM：活跃状态电源管理

ASPM is hardware-managed (no software involvement beyond enabling). Two Link power states:

**L0s**: Unidirectional low-power standby. TX enters Electrical Idle while RX remains active. Entry: when no TLPs/DLLPs pending. Exit: RX detects Electrical Idle exit on receive Lane → sends FTS Ordered Sets → TX resumes. Exit latency: tens of ns. Very fast but limited power savings.

**L1**: Bidirectional low power. Both directions enter Electrical Idle. Entry via PM DLLP handshake (PM_Active_State_Request_L1 / PM_Request_Ack). Exit: similar to L0s but involves TS1 retraining. Exit latency: microseconds (slower than L0s but greater power savings). CLKREQ# signal used to stop/restart Refclk during L1 for additional power savings.

> ASPM是硬件管理的（除启用外无需软件参与）。L0s：单向低功耗待机，TX进入电气空闲而RX保持活跃，退出延迟数十ns。L1：双向低功耗，双方进入电气空闲，通过PM DLLP握手进入/退出，退出延迟微秒级（功耗更低）。CLKREQ#信号用于在L1期间停止/重启Refclk以进一步省电。

---

## L1 PM Substates (PCIe 2.1+)
## L1 PM子状态 (PCIe 2.1+)

PCIe 2.1 introduced L1 PM Substates for even finer power granularity:
- **L1.1**: Refclk stopped but common mode maintained / Refclk停止但共模维持
- **L1.2**: Refclk stopped, common mode not maintained (deeper power savings) / Refclk停止，共模不维持（更深省电）

Entry/exit controlled via CLKREQ# and per-Lane signaling. Exit latency longer than L1 but dramatically lower power.

> L1.1/L1.2提供比L1更精细的电源粒度。通过CLKREQ#和逐Lane信令控制进入/退出。退出延迟长于L1但功耗大幅降低。

---

## Power Management Messages
## 电源管理消息

- **PM_Active_State_Nak**: Refuses L1 entry request from Link partner / 拒绝链路伙伴的L1进入请求
- **PM_PME**: Power Management Event — wake-up notification / 唤醒通知
- **PME_Turn_Off / PME_TO_Ack**: Protocol to turn off main power while preserving wake capability / 关闭主电源同时保留唤醒能力的协议
- **PM_Active_State_Request_L1 / PM_Request_Ack**: L1 ASPM entry handshake / L1 ASPM进入握手

---

## Power Management Event (PME)
## 电源管理事件 (PME)

PME is the mechanism for a device to request wake-up from a low-power state. The device asserts PME via a PM_PME Message (if Link is up) or via the WAKE# sideband signal (if Link is down, e.g., L2/L3). The Root Complex relays the PME to system software which restores power and re-enumerates the device.

> PME是设备从低功耗状态请求唤醒的机制。设备通过PM_PME消息（链路活跃时）或WAKE#边带信号（链路关闭时，如L2/L3）断言PME。根联合体中继PME到系统软件，系统软件恢复电源并重新枚举设备。

---

| English | 中文 | Notes |
|---------|------|-------|
| D0/D1/D2/D3hot/D3cold | 设备电源状态 | D0→D3功耗递减 |
| ASPM | 活跃状态电源管理 | 硬件自动 |
| L0s / L1 | 链路电源状态 | 非双向/双向 |
| L1.1 / L1.2 | L1 PM子状态 | PCIe 2.1+ |
| CLKREQ# | 时钟请求 | Refclk门控 |
| PME | 电源管理事件 | 唤醒通知 |
| PMCSR | PM控制/状态寄存器 | D状态控制 |
| PM_PME / PME_Turn_Off | PM消息 | |
| WAKE# | 唤醒信号 | 边带信号 |
