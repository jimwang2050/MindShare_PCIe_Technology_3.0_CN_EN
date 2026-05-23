# Chapter 9: DLLP Elements
# 第9章：DLLP元素

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 366–375 (10 pages)

---

## General
## 概述

The Data Link Layer manages the lower-level Link protocol. Its primary responsibility is assuring the integrity of TLPs moving between devices, but it also plays a part in TLP flow control, Link initialization, power management, and conveys information between the Transaction Layer and Physical Layer.

> 数据链路层管理底层链路协议。其主要职责是确保设备间TLP传输的完整性，但也参与TLP流控、链路初始化、电源管理，并在事务层与物理层之间传递信息。

DLLPs are always 8 Symbols (6 bytes) in size and are communicated between the Data Link Layers of each device. Unlike TLPs (which are routed), DLLPs terminate at the receiver's DLL — they never cross a Switch. DLLP types include: **Ack/Nak** (TLP acknowledgement), **InitFC1/InitFC2** (FC initialization), **UpdateFC** (FC credits), and **PM DLLPs** (power management).

> DLLP总是8个符号（6字节），在每台设备的数据链路层之间传递。与TLP不同（TLP被路由），DLLP在接收端DLL处终止——它们从不穿越Switch。DLLP类型包括：Ack/Nak（TLP确认）、InitFC1/InitFC2（FC初始化）、UpdateFC（FC信用更新）和PM DLLP（电源管理）。

---

## DLLP Format
## DLLP格式

Each DLLP (8 Symbols) contains:
- **Byte 0**: DLLP Type field (bits 7:0) — identifies the DLLP type / 标识DLLP类型
- **Bytes 1–3**: Type-specific information (24 bits) — varies per DLLP type / 类型特定信息
- **Bytes 4–5**: 16-bit CRC (same polynomial as TLP LCRC: 04C11DB7h, seed: FFFFh) / 16位CRC
- **Bytes 6–7**: Reserved / 保留

> DLLP Type编码见第5章表5-5。关键DLLP：Ack (0000_0000b)确认TLP序列号；Nak (0001_0000b)表示错误需重放；InitFC1/InitFC2初始化流控每个VC每种类型6对信用；UpdateFC运行时发布已释放信用。

---

## Power Management DLLP
## 电源管理DLLP

PM DLLPs request Link power state transitions: **PM_Enter_L1**, **PM_Enter_L23**, **PM_Active_State_Request_L1**, **PM_Request_Ack**. All PM DLLPs use the encoding 0010_0xxxxb. They are transmitted and received by the DLL but the request originates from the Transaction Layer's power management logic. See Chapter 16 for full PM details.

> PM DLLP请求链路电源状态转换：PM_Enter_L1、PM_Enter_L23、PM_Active_State_Request_L1、PM_Request_Ack。所有PM DLLP使用0010_0xxxxb编码。由DLL收发，但请求源于事务层的电源管理逻辑。

---

## DLLP CRC and Error Handling
## DLLP CRC与错误处理

The DLLP CRC uses polynomial 04C11DB7h (same 16-bit CRC as other protocols) with seed FFFFh. CRC covers bytes 0–3. Receivers calculate CRC over the received DLLP and compare it to the transmitted CRC. If they don't match, the DLLP is corrupt and discarded — this is a **Bad DLLP** error. Corrupt Ack/Nak DLLPs are simply dropped; the Ack/Nak protocol's timeout mechanism ensures recovery.

> DLLP CRC使用多项式04C11DB7h，种子FFFFh。CRC覆盖字节0–3。接收端计算已接收DLLP的CRC并与传输的CRC比较。不匹配则DLLP损坏并被丢弃——**Bad DLLP**错误。损坏的Ack/Nak DLLP被简单丢弃；Ack/Nak协议的超时机制确保恢复。

| English | 中文 | Notes |
|---------|------|-------|
| DLLP | 数据链路层包 | 8符号, 6字节 |
| Ack DLLP | 确认DLLP | TLP序列号确认 |
| Nak DLLP | 否定确认DLLP | 错误重放请求 |
| InitFC1/InitFC2 | 流控初始化 | 交换信用大小 |
| UpdateFC | 流控更新 | 运行时信用释放 |
| PM DLLP | 电源管理DLLP | 链路电源状态 |
| Bad DLLP | 坏DLLP | CRC不匹配错误 |
