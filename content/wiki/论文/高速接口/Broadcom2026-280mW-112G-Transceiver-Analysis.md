---
title: "A 280mW 112Gb/s PAM-4/NRZ Transceiver for Low-Power IOs in 5nm FinFET Technology"
authors: ["Ullas Singh", "Kumar Thasari", "Nitin Nidhi", "Arvindh Iyer", "Namik Kocaman", "Afshin Momtaz", "et al. (20 人)"]
year: 2026
venue: "ISSCC 2026, Session 8, Paper 8.6"
abstract: "Broadcom 提出的超低功耗紧凑 112Gb/s PAM-4 收发机：1/4 速率接收机（3-tap FFE + 11-tap DFE）+ 半速率 7b SST DAC 发射机。补偿 35dB 信道损耗，BER<1E-6，总功耗 280mW（模拟 250mW），0.43mm²/通道，FOM 0.07 pJ/b/dB（同类最低）。"
keywords: ["transceiver", "PAM-4", "DFE", "FFE", "CTLE", "low-power IO", "CDR", "5nm"]
tags: ["论文", "高速接口"]
---

# Broadcom 280mW 112Gb/s 低功耗收发机（ISSCC 2026 8.6）深度分析

> 归档分析（2026-08-22），原文见 `raw/论文/高速接口/8 Die-to-Die and High-Speed Electrical Transceivers/`（PAGE 18-20）。
> 相关：[[wiki/论文/高速接口/ISSCC2026-Session8-D2D-Transceivers]]

**一句话贡献**：用**模拟接收机**（CTLE+VGA+FFE/DFE）替代 ADC/DSP 方案，在 5nm 工艺上把 112Gb/s 收发机做到 280mW / 0.43mm²，并靠"CTLE 频率控制顶替最耗电的 DFE H1/H2 tap"的系统级均衡分工把功耗压到极致。

## 定位与动机

- 112Gb/s PAM-4 信道 Nyquist 损耗 >20dB，近年主流是 ADC/DSP 接收机（灵活但功耗/面积大）
- 本文瞄准**短中距**（CEI-112G-MR、IEEE 100G VSR/SR，损耗 ≤28dB）：模拟架构 <2.5pJ/b、面积最小，且 **CDR 环路延迟远低于 ADC/DSP** → 抖动容限更好
- 边界：损耗补偿 35dB，长距（>45dB）仍是 DSP 方案的天下

## 系统架构（Fig 8.6.1）

```
RX: 50Ω端接(T-coil+分流电感) → AC耦合 → CTLE(gm-TIA) → VGA(T-coil) → 4路交织S/H
    → 3-tap FFE → 11-tap DFE → Summer → Slicer(两级自定时比较器) → DEMUX(40b) → LMS自适应+CDR
TX: 40:1 MUX(40:2+2:1) → 主动peaking预驱动 → 7b DAC SST驱动(T-coil+并联高通路径)
时钟: 公共PLL(单LC VCO 49-57GHz, 分数-N+3阶ΔΣ) → 驻波谐振分布 → 单半速率PI + ÷2 → 1/4速率I/IB/Q/QB
```

- 数据通路全差分；VCM 由 replica bias CMFB 环路（模拟整条 FFE/S/H/summer 链）控制

## 核心创新

1. **均衡资源分工：CTLE 顶替 DFE H1/H2** ⭐
   - H1（1UI）由 FFE post-cursor 覆盖 → 省掉第 1 个 DFE tap
   - H2（2UI）时序仅 35.6ps、PVT 下极紧 → 用 CTLE 多个 peaking 控制组合消除
   - 依据：CTLE 频率控制跨多个 UI 点作用——`LdeQ(ctle)` 对 1UI 强、对 2UI 反向；`Cm(ctle)` 主要影响 H2
   - 全部纳入同一 LMS 自适应算法联合收敛
2. **单 PI 时钟 + twin-INL 抵消**：传统双 PI 有动态相位失配；本文单电感调谐 PI + CMOS 分频，twin PI 相位码偏移 45° 求和 → INL 反相抵消
3. **谐振时钟分布**：驻波谐振器传输线 + 开漏 CML（电感中心抽头供电），省功耗
4. **TX 省电**：开关/无源电阻阻值比增大降电容负载；2b 温度计+5b 二进制分段抑制 DNL；主动 peaking 预驱动；T-coil + 并联高通锐化边沿

## 测量结果（Fig 8.6.5）

| 指标 | 结果 |
|---|---|
| 链路 | 35dB@28GHz 下 raw BER < 1E-6 |
| 抖动容限 | CEI-112G-MR mask 0.2UI 余量 |
| TX 眼图 | RLM 0.991、J4U 73mUI、Jrms 8.85mUI、EOJ 13.5mUI、SNDR >37.5dB |
| PLL | 49-57GHz，抖动 100fsrms |
| 功耗/面积 | 280mW（模拟 250mW）/ 0.43mm²（模拟 0.33mm²） |
| FOM | 0.07 pJ/b/dB（同类最低） |

## 权衡与可借鉴点

- **优势**：功耗/面积极致、抖动容限好、FOM 最优；DFE 仅 11 tap 靠 CTLE 协同推到 35dB
- **局限**：只覆盖短中距；均衡灵活性低于 DSP；未展示 PAM-8/更高速率扩展
- **设计思维**：① 均衡资源分工先算账（谁做哪段 ISI 最省电）；② 时序紧的 tap 用频域手段替代（把困难从时域挪到频域）；③ 单 PI + INL 抵消减少时钟失配源；④ 模拟 vs DSP 按信道预算取舍

## 关键框图

![[assets/高速接口/ISSCC2026-8.6-fig1.png]]
![[assets/高速接口/ISSCC2026-8.6-fig2.png]]
*(Fig 8.6.1 收发机框图 / Fig 8.6.2 接收机实现与 AFE 参数 ISI 影响，来自原文)*

## See Also

- [[wiki/论文/高速接口/ISSCC2026-Session8-D2D-Transceivers]] — Session 8 综述（11 篇论文）
- [[wiki/抖动与相位噪声/抖动与相位噪声的关系]] — PLL 抖动/相位噪声
- [[wiki/数字信号处理/自适应滤波器]] — LMS 自适应均衡
- [[wiki/数字信号处理/数字环路滤波器与ADPLL]] — CDR/相位插值器
