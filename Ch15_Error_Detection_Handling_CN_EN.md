# Chapter 15: Error Detection and Handling
# 第15章：错误检测与处理

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 706–761 (56 pages)

---

## Error Classification
## 错误分类

PCIe errors fall into three severity categories:
- **Correctable Errors**: Hardware-corrected (e.g., correctable ECC error). Reported via ERR_COR Message. / 可纠正错误：硬件已纠正。通过ERR_COR消息报告。
- **Non-Fatal Errors**: Transaction failed but Link remains operational. Data may be lost. Reported via ERR_NONFATAL Message. / 非致命错误：事务失败但链路仍可运行。数据可能丢失。通过ERR_NONFATAL消息报告。
- **Fatal Errors**: Link reliability compromised. Reported via ERR_FATAL Message. May trigger Link retraining. / 致命错误：链路可靠性受损。通过ERR_FATAL消息报告。可能触发链路重训练。

---

## Error Sources
## 错误来源

### Physical Layer Errors / 物理层错误
- **8b/10b Code Violation or Disparity Error** (Gen1/2) / 8b/10b编码违例或差异错误
- **128b/130b Sync Header Error** (>1% invalid Sync Headers, Gen3) / 128b/130b同步头错误
- **Receiver Error** (analog front-end detects invalid signal) / 接收器错误（模拟前端检测无效信号）
- **Framing Error** (STP where END expected, etc.) / 成帧错误
- **Loss of Symbol/Block Lock** / 符号/Block锁定丢失

### Data Link Layer Errors / 数据链路层错误
- **Bad TLP**: LCRC mismatch / LCRC不匹配
- **Bad DLLP**: DLLP CRC mismatch / DLLP CRC不匹配
- **Sequence Number Error**: Lost or out-of-order TLP / 序列号错误：丢失或乱序TLP
- **Replay Timeout**: REPLAY_TIMER expired / 重放定时器超时
- **Replay Number Rollover**: Too many replay attempts / 重放次数翻转

### Transaction Layer Errors / 事务层错误
- **Poisoned TLP**: EP bit set in header — data payload is invalid / EP位（中毒位）置位——数据载荷无效
- **ECRC Error**: End-to-End CRC mismatch / 端到端CRC不匹配
- **Unsupported Request (UR)**: Request type not supported by the receiver / 接收端不支持的请求类型
- **Completer Abort (CA)**: Receiver can't complete the request / 接收端无法完成请求
- **Completion Timeout**: Requester never received Completion / 请求者从未收到完成
- **Malformed TLP**: Header contains invalid field values / Header包含无效字段值

---

## Error Reporting Mechanisms
## 错误报告机制

PCIe provides two error reporting models:
- **Baseline**: Simple registers (PCI-compatible Command/Status registers)
- **Advanced Error Reporting (AER)**: Extended Capability with per-error-type mask/severity/status registers. Mandatory for Root Ports and recommended for Endpoints. Provides much richer error tracking — Header Log captures the failing TLP's header for debugging.

> PCIe提供两种错误报告模型：基线（PCI兼容Command/Status寄存器简单报告）和高级错误报告（AER——每错误类型mask/severity/status寄存器的扩展能力。Root Port强制，端点推荐。Header Log捕获故障TLP的header以供调试）。

### Error Signaling Messages
### 错误信令消息

- **ERR_COR**: Correctable error detected / 检测到可纠正错误
- **ERR_NONFATAL**: Non-fatal error / 非致命错误
- **ERR_FATAL**: Fatal error / 致命错误

All three are routed to the Root Complex (Implicit Routing). The Root Complex logs the error, may signal an interrupt, and optionally signals the system error (SERR#/NMI) for fatal errors.

> 三种消息均隐式路由到根联合体。根联合体记录错误，可发信号中断，对于致命错误可选发信号系统错误（SERR#/NMI）。

---

## Error Handling Flow
## 错误处理流程

1. Hardware detects error / 硬件检测错误
2. Error is logged in the device's AER (or baseline) registers / 错误记录在设备AER（或基线）寄存器中
3. If error is enabled (unmasked) and severity matches, device sends ERR_COR/ERR_NONFATAL/ERR_FATAL Message to Root Complex / 若错误已启用（非屏蔽）且严重度匹配，设备向根联合体发送错误消息
4. Root Complex processes the error: logs, interrupts, optionally signals SERR#/NMI / 根联合体处理错误：记录、中断、可选发信号系统错误
5. System software (OS/driver) reads error logs and takes appropriate action / 系统软件（OS/驱动）读取错误日志并采取适当行动

---

## Error Poisoning (EP Bit)
## 错误中毒 (EP位)

When a device detects uncorrectable data in a TLP payload, it sets the **EP (Error Poisoned)** bit in the TLP header. This indicates "the data may be bad but use it if you can." Receivers must not crash on poisoned data — they may use it (e.g., writes), forward it (through a Switch), or discard it (reads with EP set + ECRC error).

> 当设备在TLP载荷中检测到无法纠正的数据时，设置TLP header中的**EP（错误中毒/Error Poisoned）**位。这表示"数据可能有问题，但尽可能使用它"。接收端不得因中毒数据崩溃——可使用它（如写入）、转发它（通过Switch）或丢弃它（EP+ECRC错误时读取）。

---

| English | 中文 | Notes |
|---------|------|-------|
| AER | 高级错误报告 | Extended Capability |
| CA (Completer Abort) | 完成者中止 | |
| Correctable / Non-Fatal / Fatal | 可纠正/非致命/致命 | 三种严重度 |
| Completion Timeout | 完成超时 | |
| ECRC | 端到端CRC | 可选 |
| EP (Error Poisoned) | 错误中毒位 | TLP Header |
| ERR_COR/NONFATAL/FATAL | 错误信令消息 | 隐式路由到RC |
| Header Log | 头部日志 | AER：捕获故障TLP |
| Malformed TLP | 畸形TLP | 无效字段值 |
| Poisoned TLP | 中毒TLP | EP=1 |
| SERR# / NMI | 系统错误/不可屏蔽中断 | 致命错误 |
| UR (Unsupported Request) | 不支持的请求 | |
