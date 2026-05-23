# Chapter 8: Transaction Ordering
# 第8章：事务排序

> 中英文对照翻译 | Chinese-English Parallel Translation
> Source: MindShare PCI Express Technology 3.0 | Pages: 344–363 (20 pages)

---

## Introduction
## 引言

PCI Express imposes ordering rules on transactions of the same Traffic Class (TC). Transactions with different TCs have no ordering relationship. Reasons for ordering rules: maintaining compatibility with legacy buses (PCI/PCI-X/AGP); ensuring deterministic completion order; avoiding deadlock conditions; maximizing performance by minimizing read latencies.

> PCIe对同一流量类别（TC）的事务施加排序规则。不同TC的事务没有排序关系。排序规则的原因：保持与传统总线（PCI/PCI-X/AGP）兼容；确保确定的完成顺序；避免死锁条件；通过最小化读延迟最大化性能。

---

## Producer/Consumer Model
## 生产者/消费者模型

The ordering rules are largely motivated by the **Producer/Consumer programming model**: one entity (Producer) writes data to a buffer, then sets a flag; another entity (Consumer) polls the flag, and when set, reads the data. This requires that the flag update (write) not pass the data write — otherwise the Consumer could read stale data.

> 排序规则主要由**生产者/消费者编程模型**驱动：一个实体（生产者）将数据写入缓冲区，然后设置标志；另一个实体（消费者）轮询标志，置位后读取数据。这要求标志更新（写）不能超越数据写入——否则消费者可能读到过期数据。

The reverse (read completions must be able to pass writes) is also critical for performance: if a long burst of writes blocks a read completion from returning, the application stalls waiting for data that's already available.

> 反向规则（读取完成必须能超越写入）对性能同样关键：若长写突发阻止读取完成返回，应用程序将阻塞等待实际已可用的数据。

---

## Ordering Rules Summary
## 排序规则概览

| Transaction Type | Can Pass Posted? | Can Pass Non-Posted? | Can Pass Completions? |
|-----------------|------------------|---------------------|----------------------|
| **Posted (Memory Writes, Messages)** | Yes/No (must not pass other Posted) | Yes | Yes |
| **Non-Posted Requests (Reads, IO/Config Writes)** | No | Yes/No | Yes |
| **Completions (Read Cpls, IO/Config Cpls)** | No (must not pass Posted) | Yes | Yes/No |

Key rules:
1. **Posted can pass everything** (except other Posted requests) — ensures writes don't wait for reads
2. **Completions cannot pass Posted** — prevents deadlocks (the Producer/Consumer case)
3. **Completions can pass Non-Posted** — read completions can bypass pending reads
4. **Relaxed Ordering (RO) bit**: Set by software to allow a TLP to bypass ordering rules. If Set, a Posted write can pass other Posted writes; a Completion can pass Posted writes.

> 关键规则：
> 1. Posted可超越一切（除其他Posted）——确保写入不等待读取
> 2. 完成不能超越Posted——防止死锁（生产者/消费者场景）
> 3. 完成可超越Non-Posted——读取完成可绕过等待中的读取
> 4. 宽松排序（RO/Relaxed Ordering）位：软件设置以允许TLP绕过排序规则。置位后Posted写可超越其他Posted写；完成可超越Posted写。

**ID-Based Ordering (IDO)**: PCIe 3.0 attribute. When Set, ordering rules are relaxed within a single Requester ID. Multiple independent streams from the same device can proceed without blocking each other.

> **基于ID排序（IDO）**：PCIe 3.0属性。置位后，单一Requester ID内的排序规则放宽。同一设备的多个独立流可互不阻塞地推进。

---

## Deadlock Avoidance
## 死锁避免

The ordering rules prevent the classic PCI deadlock: if completions could be blocked behind posted writes, and the device needing the completion must first drain posted writes to make buffer space, a circular dependency forms. The rule "Completions can pass Non-Posted but not Posted" resolves this: completions from one VC can drain even if posted writes are still waiting.

> 排序规则防止经典的PCI死锁：若完成被Posted写阻塞，而需要完成的设备必须先排空Posted写以释放缓冲空间，则形成循环依赖。规则"完成可超越Non-Posted但不能超越Posted"解决了此问题。

| English | 中文 | Notes |
|---------|------|-------|
| Producer/Consumer | 生产者/消费者 | 编程模型 |
| Posted / Non-Posted / Completion | 发布/非发布/完成 | 三种事务类型 |
| RO (Relaxed Ordering) | 宽松排序 | 排序属性 |
| IDO (ID-Based Ordering) | 基于ID排序 | PCIe 3.0 |
| Deadlock | 死锁 | 循环依赖 |
