# MindShare PCI Express Technology 3.0 — 中英文对照翻译

**MindShare PCI Express Technology 3.0** — *Comprehensive Guide to Generations 1.x, 2.x, 3.0*
<br>Authors: Mike Jackson, Ravi Budruk | MindShare Press, 2012 | 1,057 pages

全书 **逐段中英文双语对照翻译**，保留原始图表。覆盖 PCIe 1.x / 2.x / 3.0 三代规范，从 PCI/PCI-X 演进背景到 Gen3 物理层电气特性，是学习和查阅 PCIe 体系结构的系统性参考。

> **说明：** 本翻译由 AI 辅助生成，仅供学习参考。技术细节请以英文原文为准，请注意甄别分析的准确性。欢迎提交 Issue/PR 指正错误。

---

## 项目概览 | Project Overview

| 项目 | 数量 |
|------|------|
| 翻译章节 | 20 章 (Ch01–Ch20) |
| 附录 | 4 篇 (AppA–AppD) |
| 术语表 | 1 篇 (350+ 术语) |
| 总行数 | ~3,500 行 |
| 配图 | 90+ 张 |
| 覆盖页码 | 68–1057 |

---

## 目录 | Table of Contents

### Part 1: The Big Picture · 第一部分：全景

| Chapter | Title | 标题 | Pages |
|---------|-------|------|-------|
| [Ch01](Ch01_Background_CN_EN.md) | Background | 背景知识 | 68–97 |
| [Ch02](Ch02_PCIe_Architecture_Overview_CN_EN.md) | PCIe Architecture Overview | PCIe架构概述 | 98–143 |
| [Ch03](Ch03_Configuration_Overview_CN_EN.md) | Configuration Overview | 配置概述 | 144–179 |
| [Ch04](Ch04_Address_Space_Transaction_Routing_CN_EN.md) | Address Space & Transaction Routing | 地址空间与事务路由 | 180–225 |

### Part 2: Transaction Layer · 第二部分：事务层

| Chapter | Title | 标题 | Pages |
|---------|-------|------|-------|
| [Ch05](Ch05_TLP_Elements_CN_EN.md) | TLP Elements | TLP元素 | 228–273 |
| [Ch06](Ch06_Flow_Control_CN_EN.md) | Flow Control | 流控 | 274–303 |
| [Ch07](Ch07_Quality_of_Service_CN_EN.md) | Quality of Service | 服务质量 | 304–343 |
| [Ch08](Ch08_Transaction_Ordering_CN_EN.md) | Transaction Ordering | 事务排序 | 344–363 |

### Part 3: Data Link Layer · 第三部分：数据链路层

| Chapter | Title | 标题 | Pages |
|---------|-------|------|-------|
| [Ch09](Ch09_DLLP_Elements_CN_EN.md) | DLLP Elements | DLLP元素 | 366–375 |
| [Ch10](Ch10_AckNak_Protocol_CN_EN.md) | Ack/Nak Protocol | Ack/Nak协议 | 376–417 |

### Part 4: Physical Layer · 第四部分：物理层

| Chapter | Title | 标题 | Pages |
|---------|-------|------|-------|
| [Ch11](Ch11_Physical_Layer_Logical_Gen1_Gen2_CN_EN.md) | Physical Layer — Logical (Gen1 & Gen2) | 物理层—逻辑子层 (Gen1/Gen2) | 420–465 |
| [Ch12](Ch12_Physical_Layer_Logical_Gen3_CN_EN.md) | Physical Layer — Logical (Gen3) | 物理层—逻辑子层 (Gen3) | 466–505 |
| [Ch13](Ch13_Physical_Layer_Electrical_CN_EN.md) | Physical Layer — Electrical | 物理层—电气子层 | 506–563 |
| [Ch14](Ch14_Link_Initialization_Training_CN_EN.md) | Link Initialization & Training | 链路初始化与训练 | 564–703 |

### Part 5: Additional System Topics · 第五部分：其他系统主题

| Chapter | Title | 标题 | Pages |
|---------|-------|------|-------|
| [Ch15](Ch15_Error_Detection_Handling_CN_EN.md) | Error Detection and Handling | 错误检测与处理 | 706–761 |
| [Ch16](Ch16_Power_Management_CN_EN.md) | Power Management | 电源管理 | 762–851 |
| [Ch17](Ch17_Interrupt_Support_CN_EN.md) | Interrupt Support | 中断支持 | 852–891 |
| [Ch18](Ch18_System_Reset_CN_EN.md) | System Reset | 系统复位 | 892–905 |
| [Ch19](Ch19_Hot_Plug_Power_Budgeting_CN_EN.md) | Hot Plug and Power Budgeting | 热插拔与功耗预算 | 906–945 |
| [Ch20](Ch20_Spec_Rev_2_1_Updates_CN_EN.md) | Updates for Spec Revision 2.1 | 规范2.1修订更新 | 946–1031 |

### Appendices & Glossary · 附录与术语表

| Section | Title | 标题 | Pages |
|---------|-------|------|-------|
| [AppA](AppA_Debugging_PCIe_Traffic_CN_EN.md) | Debugging PCIe Traffic | PCIe流量调试 | — |
| [AppB](AppB_Markets_Applications_CN_EN.md) | Markets & Applications | 市场与应用 | — |
| [AppC](AppC_Intelligent_Adapters_Multi_Host_CN_EN.md) | Intelligent Adapters & Multi-Host | 智能适配器与多主机 | — |
| [AppD](AppD_Locked_Transactions_CN_EN.md) | Locked Transactions | 锁定事务 | — |
| [Glossary](Glossary_CN_EN.md) | Glossary | 术语表 (350+ 术语) | 1032–1057 |

---

## 翻译格式 | Translation Format

每章采用 **段落级双语对照** 排版：

- **英文原文**：普通段落（GitHub 白色背景）
- **中文翻译**：块引用 `>` （GitHub 灰色背景，视觉分层）
- 技术术语、寄存器名、信号名、协议名保留原文不译
- 数值与表格数据保持原样
- 章节内嵌快速导航锚点，方便跳转

示例效果：

```
The LTSSM consists of 11 top-level states: Detect, Polling, Configuration,
Recovery, L0, L0s, L1, L2, Hot Reset, Disabled, and Loopback.

> LTSSM由11个顶级状态组成：Detect、Polling、Configuration、Recovery、
  L0、L0s、L1、L2、Hot Reset、Disabled 和 Loopback。
```

---

## 插图目录 | Images

所有原始插图按章节组织在 `images/` 目录下：

```
images/
├── ch01/    (35 files)  — Ch01 背景知识
├── ch02/    (46 files)  — Ch02 PCIe架构概述
├── ch04/    (49 files)  — Ch04 地址空间与事务路由
├── ch05/    (50 files)  — Ch05 TLP元素
├── ch17/    (82 files)  — Ch17 中断支持
├── ch18/    (19 files)  — Ch18 系统复位
├── ch19/    (44 files)  — Ch19 热插拔与功耗预算
├── ch20/    (33 files)  — Ch20 规范2.1修订更新
├── appA/    (34 files)  — AppA PCIe流量调试
├── appB/    (14 files)  — AppB 市场与应用
├── appC/    (30 files)  — AppC 智能适配器与多主机
├── appD/    (10 files)  — AppD 锁定事务
└── glossary/(49 files)  — 术语表
```

> 部分章节（Ch03, Ch06–Ch16 部分段落）持续补充插图中。

---

## 来源 | Source

- **书名：** MindShare PCI Express Technology 3.0 (2012)
- **作者：** Mike Jackson, Ravi Budruk
- **出版社：** MindShare Press
- **ISBN：** 978-0-9770878-6-0
- **PDF 总页数：** 1,057 页

---

## 贡献 | Contributing

欢迎任何形式的改进：

- **指正翻译错误** — 提交 Issue 描述问题所在章节及修正建议
- **补充遗漏段落** — 提交 PR 追加翻译
- **补充插图** — 为未配图的章节补充图表截图
- **统一术语** — 术语表中存在 AI 翻译不一致之处，欢迎标准化

提交 PR 时请保持段落级中英对照格式不变。

---

## 许可 | License

原始英文内容版权归 MindShare Press 及原作者所有。中文翻译仅供个人学习研究使用，不得用于商业目的。
