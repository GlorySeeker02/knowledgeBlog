---
title: "A 224 Gb/s 3 pJ/bit 40 dB Insertion Loss Transceiver in 3-nm FinFET CMOS"
authors: ["Dirk Pfaff", "Muhammad Nummer", "Noman Hai", "Jingjing Xia", "Marc-Andre LaCroix", "David A. Yokoyama-Martin", "et al. (24 人)"]
year: 2025
venue: "IEEE Journal of Solid-State Circuits (JSSC), vol. 60, no. 1, pp. 9-18, Jan. 2025"
abstract: "Synopsys 提出的首款完整 3nm 224Gb/s PAM-4 长距收发机：40dB 插入损耗下 BER<1e-6，模拟能效 3pJ/b（TX 1.0 + RX 2.0，不含 DSP）。TX 为 7bit 电流模 DAC（1/8 率、TDC 校准时钟）；RX 为 20dB 峰值反相器 AFE + 7bit 时间交织 SAR ADC（112GS/s）+ 44-tap FFE 与 DFE/MLSD DSP；共享 14GHz 全数字 bang-bang PLL。"
keywords: ["224Gb/s", "PAM-4", "transceiver", "MLSD", "TI-ADC", "DFE", "3nm"]
tags: ["论文", "高速接口"]
doi: "10.1109/JSSC.2024.3466092"
---

# 224 Gb/s 3 pJ/bit 40 dB 损耗收发机（3nm FinFET）

**一句话贡献**：业界首篇**完整的 3nm 224Gb/s 长距 PAM-4 收发机**（含 TX+RX+时钟+DSP），40dB 损耗下 BER<1e-6、3pJ/b 模拟能效——把上一代 112G SerDes 的比特率翻倍的同时能效不倒退。

## 定位与设计挑战

- 224Gb/s PAM-4 = 波特率 112GBaud = **56GHz 模拟带宽**、112GS/s 数据转换，信道损耗 >35dB（需高阶数字均衡 + MLSD）
- 应对思路：用 **3nm 逻辑密度** 扛 DSP 复杂度；数据总线加宽（RX 80 路并行采样、TX 64 符号并行）把 DSP/FFE 时钟压到 1.4/1.75GHz

## 核心架构（Fig. 1）

```
RX: AFE(CTLE+VGA, 20dB峰值@50GHz) → 7bit TI-SAR ADC (8路 1/8率, 112GS/s) → DSP(80路并行)
    DSP: 44-tap FFE + DFE 或 RS-MLSD / FS-MLSD
TX: 64符号并行 FFE → 串化 → 7bit 电流模 DAC (1/8率, 8相) → 驱动
时钟: CMU = 全数字 bang-bang PLL（双 LC-DCO 12.7/10.4GHz + 1.5 分频覆盖 7-14GHz）
      → CCU（clock conditioner）8 相生成：注入锁定环形振荡器 ILO1/ILO2 + 7bit 相位旋转器(PR)
```

**核心时钟创新**：全局只用 **1/8 率时钟（14GHz）**——比半/四分之一率更低，允许**全 CMOS、无电感**时钟生成与分布；代价是要生成 8 路高精度相位，由新模块 **CCU** 解决（TX 侧还用 1/8 率的全新 baud-rate 数据 MUX）。

## 关键电路细节

- **全数字 bang-bang PLL**：LC-DCO（0.5MHz/LSB 细分辨率）+ 2 阶 ΔΣ + 数字环路滤波；比模拟 PLL 更抗 3nm 工艺的闪烁/热噪声；数字漂移校正环覆盖 -40~+125°C
- **CCU 相位生成**：两级注入锁定环形振荡器 + 相位旋转器（7bit、45° 相邻相插值）
- **RX AFE**：反相器型增益级，CTLE 峰值 20dB@50GHz，VGA 6dB 范围
- **RX DSP**：44-tap FFE + 单 tap DFE，可选 **RS-MLSD / 全状态 FS-MLSD**（高损耗场景关键）
- **TX DAC**：7bit 电流模、1/8 率 8 相；TDC 辅助时序校准（14fs 步进相位缓冲）

## 测量结果

| 指标 | 结果 |
|---|---|
| 工艺/面积 | 3nm FinFET；**0.5mm²/lane**（含 DSP，4-lane macro） |
| 能效 | **TX 1.0 + RX 2.0 = 3.0 pJ/b**（含片上 LDO，不含 DSP 功耗） |
| PLL 抖动 | -113 dBc/Hz@100MHz，**90 fsrms**（1k-100MHz）；全温变化仅 12fs |
| TX | 55 fsrms RJ + 170 fs DJ；960mVdiffpp；**RLM 0.992**；DNL ±0.2 LSB |
| AFE/ADC | 50GHz 峰值/20dB 增益；7bit TI-SAR @112GS/s，ENOB **5.5bit**@1.2GHz / **4.0bit**@50.5GHz |
| BER | 224Gb/s @**40dB** 损耗需 **FS-MLSD** 达 1e-6；FS-MLSD 较 DFE 降 BER **24-75×** |

## 权衡与启示

- **1/8 率时钟**是全文最关键的架构赌注：用"8 相生成难度"换"无电感全 CMOS 时钟 + 低压摆率路径"，依赖 CCU + 注入锁定环形振荡器成熟度
- **MLSD 的价值**：40dB 损耗场景 DFE 不够用（BER 达不到 1e-6），FS-MLSD 成为长距必选——数字复杂度换模拟指标
- **模拟 vs DSP 分工**：AFE 先做 20dB 峰值减轻 ADC 压力，剩下靠 DSP（44-tap FFE + MLSD）——呼应 8.6 的"均衡资源分工"思路，但 224G 时代数字成本已可接受
- 定位差异：**8.6（Broadcom ISSCC 2026）是 5nm 模拟 RX 短中距 35dB/2.5pJ/b**；本文是 **3nm 长距 40dB/3pJ/b 带 MLSD**——长距数字化、短距模拟化的路线分野很清楚

## 相关

- [[wiki/论文/高速接口/Pfaff2025-224G-Transceiver-DeepDive]] — 深度分析（1/8 率时钟/逆变器 AFE/8:1 MUX/MLSD 权衡）
- [[wiki/论文/高速接口/ISSCC2026-Session8-D2D-Transceivers]] — ISSCC 2026 Session 8（同领域综述）
- [[wiki/论文/高速接口/Broadcom2026-280mW-112G-Transceiver-Analysis]] — 112G 模拟 RX 方案对比
- Source: `raw/论文/高速接口/A 224 Gb-s 3 pJ-bit 40 dB Insertion Loss Transceiver in 3-nm FinFET CMOS/`（原文 md + 插图）
