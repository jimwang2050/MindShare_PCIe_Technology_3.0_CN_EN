# Chapter 16: Power Management
# 第16章：电源管理

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 762–852 (91 pages)

---

## Introduction: Four Areas of PM Support / 引言：电源管理四大领域

PCI Express power management (PM) defines four major areas of support:
1. **PCI-Compatible PM (PCI-PM):** Hardware/software compatible with the PCI Bus Power Management Interface Specification. Manages Function D-states (D0-D3) and the PCI Power Management Capability registers.
2. **Native PCIe PM Extensions:** Active State Power Management (ASPM) — hardware-autonomous Link power reduction without software involvement.
3. **L1 PM Substates:** Optional deeper L1 substates (L1.1, L1.2) for maximum idle power savings.
4. **Auxiliary Power and Wakeup:** Mechanisms (Beacon, WAKE#) to wake sleeping Links when main power is removed.

> PCIe电源管理定义四大支持领域：PCI兼容PM(管理D0-D3状态)；原生PCIe PM扩展ASPM(硬件自主链路降功耗)；L1 PM子状态(L1.1/L1.2最大空闲省电)；辅助电源与唤醒机制。

---

## Background: ACPI, OnNow, and the OS / 背景：ACPI、OnNow与操作系统

PCIe PM works within the broader system PM framework defined by **ACPI (Advanced Configuration and Power Interface)**. ACPI defines:
- **System S-states:** S0 (working), S1-S4 (sleep), S5 (soft off)
- **Device D-states:** D0 (fully on), D1, D2 (intermediate), D3<sub>Hot</sub>, D3<sub>Cold</sub> (off)
- **Processor C-states:** C0 (active), C1-Cn (idle)
- **Link L-states:** L0, L0s, L1, L2, L3 (PCIe-specific, derived from D-states)

> PCIe PM在ACPI定义的更广泛系统PM框架内工作。ACPI定义系统S状态、设备D状态、处理器C状态、链路L状态。

The **OnNow Initiative** (Microsoft) aims for "Instant On" — the PC appears off but can wake instantly. PCIe PM contributes by allowing devices and Links to enter low-power states when idle while maintaining the ability to wake quickly.

> OnNow倡议（Microsoft）的目标是即时启动——PC看似关闭但可即时唤醒。PCIe PM通过允许设备和链路在空闲时进入低功耗状态同时保持快速唤醒能力来贡献。

---

## D-States: Device Power Management / D状态：设备电源管理

### D0 State — Fully Operational

D0 is subdivided into **D0<sub>uninitialized</sub>** and **D0<sub>active</sub>**. After reset or FLR, the Function enters D0<sub>uninitialized</sub>. After configuration (Memory Space, I/O Space, or Bus Master Enable set), it transitions to D0<sub>active</sub>.

### D1 and D2 States — Intermediate Power Savings (Optional)

D1 and D2 are optional intermediate states. Functions in D1 or D2 must not initiate Request TLPs (except Messages). Only Configuration and Message Requests are accepted. The driver must quiesce all outstanding transactions before the Function transitions to D1/D2.

### D3<sub>Hot</sub> State — Software-Accessible Low Power

When in D3<sub>Hot</sub>: main power is present but the Function is in very low power. Configuration and Message requests are the only accepted TLPs. If **No_Soft_Reset** is set, functional context is preserved and the Function returns to D0<sub>active</sub> on wake. Otherwise, it returns to D0<sub>uninitialized</sub> and requires full re-initialization. Minimum recovery time from D3<sub>Hot</sub> to D0: **10 ms** (unless Immediate_Readiness_on_Return_to_D0 is set).

### D3<sub>Cold</sub> State — Power Removed

The Function's main power is removed. Wakeup (if supported) requires auxiliary power (Vaux) and a wakeup mechanism (Beacon or WAKE#). Return to D0 requires Fundamental Reset and full re-initialization.

> D0分为D0<sub>uninitialized</sub>和D0<sub>active</sub>。D1/D2可选中间省电状态。D3<sub>Hot</sub>主电源存在但极低功耗，恢复至少10ms。D3<sub>Cold</sub>主电源移除，唤醒需辅助电源和唤醒机制，返回需基本复位。

---

## L-States: Link Power Management / L状态：链路电源管理

| L-State | Description | Power | Exit Latency | Key Feature |
|---------|-------------|-------|-------------|-------------|
| **L0** | Fully active | Highest | N/A | Normal packet traffic |
| **L0s** | Standby | Low | Very short (~100 ns) | Per-direction, no handshake |
| **L1** | Deeper standby | Very low | ~μs | Both directions, PM_Enter_L1 handshake |
| **L1.1** | L1 substate | Lower | Longer | Common mode maintained, CLKREQ# |
| **L1.2** | L1 substate | Lowest | Longest | Common mode not required |
| **L2** | Sleep | ~0 mW | N/A (re-init) | Main power off, Vaux for wakeup |
| **L3** | Off | 0 mW | N/A | No power at all |

The Link state is determined by the D-state of the Downstream component. For Multi-Function Devices, the Link enters L1 only when **all** Functions are programmed to a non-D0 state (non-ARI MFD) or when all are in non-D0 or D0<sub>uninitialized</sub> (ARI device).

> 链路状态由下游组件D状态决定。非ARI MFD仅在所有Function被编程到非D0后才进入L1。L0完全活跃、L0s短待机、L1深待机需握手、L1.1/L1.2更省电需CLKREQ#、L2/L3断电。

---

## ASPM: Active State Power Management / 主动状态电源管理

ASPM is a **hardware-autonomous** mechanism — no software involvement required once enabled. It allows the Link to enter L0s or L1 even when all Functions are in D0, based solely on idle conditions. ASPM significantly reduces power consumption during brief idle periods between transactions.

### ASPM Entry Conditions

- **L0s:** Transmitter has been idle (no TLPs or DLLPs pending) for a configurable period (implementation-specific, typically < 7 μs). Entry is per-direction and unilaterally decided.
- **L1:** Downstream component requests L1 entry via PM_Active_State_Request_L1 DLLP. Upstream component may accept (PM_Request_Ack) or reject (PM_Active_State_Nak). Both directions must agree.

### ASPM Configuration

Software enables/disables ASPM through the **ASPM Control** field in the Link Control Register:
- 00b = Disabled
- 01b = L0s entry enabled
- 10b = L1 entry enabled
- 11b = L0s and L1 entry enabled

Software must read the **ASPM Support** field and L0s/L1 Exit Latency from the Link Capabilities Register before enabling ASPM. Endpoint Functions also report **Endpoint L0s/L1 Acceptable Latency** — the worst-case latency they can tolerate before risking internal FIFO overflow.

> ASPM是硬件自主机制——使能后无需软件参与。即使所有Function在D0，链路空闲时也可进入L0s或L1。软件通过Link Control Register的ASPM Control字段使能/禁用。必须首先读取ASPM Support和Exit Latency，Endpoint报告可容忍的最坏情况延迟。

---

## L1 PM Substates: L1.1 and L1.2 / L1 PM子状态

L1.1 and L1.2 are optional deeper substates of L1. Entry is always through L1.0, with additional conditions:
- **L1.1:** Common-mode voltages maintained. Reference clock may be removed (via CLKREQ#). Lower power than L1.0.
- **L1.2:** Common-mode voltages NOT required. Reference clock may be removed. Deepest idle power, longest exit latency.

Both substates use the CLKREQ# sideband signal for exit: the component asserting CLKREQ# triggers the Link to wake. Entry and exit timing parameters (T_POWER_ON, T_COMMONMODE, LTR_L1.2_THRESHOLD) are configurable through the L1 PM Substates Extended Capability registers.

> L1.1和L1.2是L1的可选更深子状态，始终通过L1.0进入。L1.1保持共模电压；L1.2不要求共模电压。两者均使用CLKREQ#边带信号退出。时序参数通过L1 PM Substates Extended Capability寄存器配置。

---

## The PME Mechanism / PME机制

The Power Management Event (PME) mechanism allows a device to request a system wakeup from a low-power state. PCIe separates the PME task into two components:

### 1. Link Wakeup (Reactivation)

When a Link is in L2 (non-communicating), the device cannot send TLPs. It must first reactivate the Link using one of two wakeup mechanisms:
- **Beacon:** In-band periodic signal sent on idle Lanes. Detected by the Upstream Port, which propagates the wakeup toward the Root Complex.
- **WAKE#:** Sideband open-drain signal routed to the system power controller. Form-factor dependent.

### 2. PM_PME Message

Once the Link is reactivated and trained to L0, the device sends a **PM_PME Message** (a Posted TLP) upstream to the Root Complex. The PM_PME Message contains the Requester ID, identifying which device needs service.

### PME Synchronization: PME_Turn_Off/PME_TO_Ack

Before system sleep, the Root Complex broadcasts a **PME_Turn_Off Message** Downstream, instructing all devices to stop sending PM_PMEs and prepare for power removal. Each device responds with **PME_TO_Ack**. This handshake ensures that all in-flight PME Messages are flushed from the fabric before power is removed. A Switch aggregates PME_TO_Ack from all Downstream Ports and sends a single PME_TO_Ack Upstream.

### PME Service Timeout

If a device's PME is not serviced within **100 ms (+50%/-5%)**, the device re-sends the PM_PME Message. This ensures that PMEs lost due to Root Complex buffer overflow are re-issued.

> PME机制允许设备从低功耗状态请求系统唤醒。分两个组件：链路唤醒（Beacon带内或WAKE#边带）和PM_PME Message（Posted TLP向上游标识唤醒源）。睡眠前PME_Turn_Off/PME_TO_Ack握手确保所有在途PME被刷新。100ms服务超时确保丢失的PME被重发。

---

## Auxiliary Power (Vaux) / 辅助电源

Vaux provides power to a component when its main power rails are off (L2 or D3<sub>Cold</sub>). Only devices that need to support wakeup from these states require Vaux. A device may only consume auxiliary power if enabled to do so by software (PME_En bit in PMCSR, or Aux Power PM Enable bit in Device Control Register). Vaux power is limited — typical budget is 375 mA at 3.3V.

> Vaux在主电源轨关闭(L2或D3<sub>Cold</sub>)时为组件供电。设备仅在软件使能后才可消耗辅助电源。典型预算为3.3V 375mA。

---

## Software Flow for PM Transitions / PM转换的软件流程

### Entering a Low-Power State

1. Device driver saves functional state and quiesces the device (completes/terminates all outstanding transactions)
2. System PM software writes to the Function's PMCSR PowerState field to transition to the target D-state
3. The Downstream component initiates Link state transition (L1 for D1-D3<sub>Hot</sub>)
4. If entering system sleep (S3/S4), the Root Complex broadcasts PME_Turn_Off, waits for PME_TO_Ack, then signals the power controller to remove main power

### Resuming from a Low-Power State

1. Wakeup event (Beacon, WAKE#, or user action) triggers power controller to restore main power and reference clocks
2. Fundamental Reset de-asserted; Link trains through Detect/Polling/Configuration to L0
3. System PM software writes to PMCSR to transition Function to D0
4. After recovery time (10 ms for D3<sub>Hot</sub>→D0, 200 μs for D2→D0), driver restores functional state

> 进入低功耗：驱动保存状态停止设备→PM软件写PMCSR→下游组件发起链路转换→PME_Turn_Off握手。恢复：唤醒事件→电源恢复→基本复位取消→链路训练到L0→PM软件写D0→恢复时间后驱动恢复状态。
