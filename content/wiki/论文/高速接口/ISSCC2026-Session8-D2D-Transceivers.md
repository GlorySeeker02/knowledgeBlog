---
title: "ISSCC 2026 Session 8: Die-to-Die and High-Speed Electrical Transceivers"
authors: ["Session Chairs: Didem Turker Melek (Cadence)", "Kenny Hsieh (TSMC)"]
year: 2026
venue: "ISSCC 2026, Session 8 (Wireline Subcommittee), 2026-02-16"
abstract: "ISSCC 2026 Session 8 会议综述：11 篇论文覆盖 Die-to-Die 与高速电收发机，方向包括 UCIe 兼容芯片间互连、单端双向（SBD）收发机、无参考 PAM-4 CDR、超低功耗模拟密集型 TX，目标应用为 chiplet 与 AI 数据中心互连。"
keywords: ["die-to-die", "transceiver", "SerDes", "UCIe", "PAM-4", "CDR", "SBD", "chiplet"]
tags: ["论文", "高速接口"]
---

# ISSCC 2026 Session 8: Die-to-Die and High-Speed Electrical Transceivers

**一句话贡献**：Session 8 集中展示了面向 AI/数据中心场景的 D2D 与高速电互连前沿——从 UCIe 兼容芯片间链路（48-56Gb/s/lane、0.23-1.2pJ/b）到 112Gb/s 级单端双向收发机与无参考 PAM-4 CDR，再到 240Gb/s 级模拟密集型 TX，全面比拼带宽密度与能效（pJ/b）。

## 论文清单（11 篇）

| # | 标题（简） | 第一作者 / 单位 | 核心指标 |
|---|---|---|---|
| 8.1 | UCIe 兼容 D2D 链路，30mm 有机封装 | Susnata Mondal (Intel) | 48Gb/s/lane × 16，1.24Tb/s/mm，1.2pJ/b，22nm，可扩展 56GT/s |
| 8.2 | 边沿触发收发机（ETT）D2D 接口 | Wei-Chih Chen (TSMC) | 32Gb/s，12.35Tb/s/mm²，0.36pJ/b，3nm + 有源 LSI 封装 |
| 8.3 | 零唤醒惩罚时钟门控模块化 D2D | Ravi Shivnaraine (Microsoft) | 24Gb/s，0.23pJ/b，4.2ns 端到端延迟，3nm |
| 8.4 | 112Gb/s/wire 单端双向收发机 + 动态均衡器 | Zhiwen Huang (北京大学) | 112Gb/s/wire NRZ SBD，BER<1E-14，1.01pJ/b，28nm |
| 8.5 | 112Gb/s 无参考混合信号 PAM-4 CDR | Yiqing Xu (中科院半导体所) | 112Gb/s，0.76pJ/b，恢复时钟抖动 343fsrms，0.11UIPP@1E-12 |
| 8.6 | 280mW 112Gb/s PAM-4/NRZ 低功耗收发机 | Ullas Singh (Broadcom) | 280mW，补偿 35dB 信道损耗，BER<1E-6，最佳 pJ/b/dB FOM，5nm（[深度分析](Broadcom2026-280mW-112G-Transceiver-Analysis.md)） |
| 8.7 | 112Gb/s PAM-4 SBD + 失配补偿 2×VDD 混合 | Huanfa Sun (西安交通大学) | TX swing 1.27V，1.73pJ/b，FoM 0.14pJ/b/dB，28nm |
| 8.8 | 56Gb/s/wire 电容驱动 SBD，PVT/失配跟踪 | Kahyun Kim (首尔大学) | 0.292pJ/b，20.7Tb/s/mm，56Gb/s/wire，28nm，XSR/D2D |
| 8.9 | 72Gb/s/pin 单端 SBD + C-Peaking 泄漏消除 | Xuxu Cheng (南方科技大学) | 72Gb/s/pin，CPLC 抑制泄漏 63%，1.5pJ/b，28nm |
| 8.10 | 180-240Gb/s 模拟密集型 PAM-4 TX | Ziyi Lin (清华大学) | 240Gb/s 时 0.70pJ/b 模拟能效，首款 >240Gb/s/ch SerDes TX，65nm |
| 8.11 | 无电阻 7bit SST DAC TX（PAM-4/PAM-8） | Yao-Hung Tsai (台湾大学) | 112Gb/s PAM-4：1.59pJ/b；168Gb/s PAM-8：1.06pJ/b，8-tap FFE，28nm |

## 四个技术方向

1. **Die-to-Die 互连（8.1-8.3）**：UCIe-S 兼容、高带宽密度（1.24Tb/s/mm）、超低功耗（0.23-1.2pJ/b）、模块化低延迟，面向 chiplet 与数据中心。关键电路创新：阻抗不变型 TX CTLE、高摆幅 N/N 驱动器、紧凑谐振时钟分布、边沿触发收发机（ETT）、零唤醒惩罚时钟门控。
2. **单端同时双向（SBD）收发机（8.4, 8.7, 8.8, 8.9）**：112Gb/s/wire 级别，核心难点是**自身串扰/反射消除**——动态均衡器、两步回波消除器、电容驱动（CDSBD）降低自干扰、C-Peaking 泄漏消除。
3. **PAM-4 CDR（8.5）**：无参考时钟混合信号 CDR，对称线性 PD + bang-bang PFD 混合架构，恢复时钟抖动 343fsrms，0.11UIPP 抖动容限。
4. **低功耗/高速 TX（8.6, 8.10, 8.11）**：模拟密集型架构（三级级联 2:1 AMUX、双 T-coil）、无电阻 SST DAC（寄生电容降 1.84×）、8-tap FFE + 双反馈均衡器抑制 ISI 抖动。

## 关键趋势观察

- **UCIe/chiplet 生态落地**：Intel/TSMC/Microsoft 的 D2D 链路都强调 UCIe 兼容与标准封装（30mm 有机基板 2 层布线）
- **能效竞赛**：D2D 已到 0.2-0.4pJ/b 量级；SBD 用单端+双向把 pin 效率翻倍
- **SBD 成为主流选择**：4/11 篇是单端同时双向，均衡与回波消除是共性难题
- **国内高校活跃**：北大、中科院、西交、南科大、清华、台大共 6 篇

## 与知识库的联系

- 抖动/时钟：[[wiki/抖动与相位噪声/抖动与相位噪声的关系]]、[[wiki/抖动与相位噪声/数据转换器中的抖动]] — CDR 抖动（343fsrms）、时钟分布
- 电路实现：[[wiki/模拟集成电路/Allen/Ch06 - CMOS Operational Amplifiers]]、[[wiki/模拟集成电路/Razavi/Ch09 - Operational Amplifiers]] — 均衡器/驱动器/混合电路基础
- 信号处理：[[wiki/数字信号处理/自适应滤波器]] — 均衡与回波消除算法
- 系统框架：[[wiki/数字信号处理/数字环路滤波器与ADPLL]] — CDR 环路与相位插值

## 个人评价

- 亮点：数据密度与能效指标全面刷新（UCIe 3× 速率、0.23pJ/b、240Gb/s TX）；Session 综述形式适合快速建立领域地图
- 局限：这是会议 digest（每篇 2 页摘要），电路细节有限；需要单篇全文时再从 raw 源文件按需展开
- Source: `raw/论文/高速接口/8 Die-to-Die and High-Speed Electrical Transceivers/`（原文 md + 全部插图）

## See Also

- [[wiki/论文/高速接口/Broadcom2026-280mW-112G-Transceiver-Analysis]] — 8.6（Broadcom 低功耗收发机）深度分析
