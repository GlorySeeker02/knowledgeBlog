---
title: "Synopsys 224Gb/s 3nm 收发机 · 深度分析"
authors: ["Dirk Pfaff", "Muhammad Nummer", "Marc-Andre LaCroix", "David A. Yokoyama-Martin", "et al. (Synopsys)"]
year: 2025
venue: "IEEE JSSC, vol. 60, no. 1, Jan. 2025"
abstract: "对 'A 224 Gb/s 3 pJ/bit 40 dB Insertion Loss Transceiver in 3-nm FinFET CMOS' 的深度技术分析：1/8 率时钟赌注、电压模式逆变器 AFE 的三阶失真消隐论证、双 T-coil、串联峰值 CTLE 独立 Av/AHF、8+10 两级 TI-SAR ADC、FFE/DFE 分割优化与 MLSD、首创 8:1 全速率 MUX、修正温度计分段。"
keywords: ["224Gb/s", "transceiver", "1/8-rate clocking", "inverter-based AFE", "MLSD", "8:1 MUX", "analysis"]
tags: ["论文", "高速接口"]
doi: "10.1109/JSSC.2024.3466092"
---

# Synopsys 224Gb/s 3nm 收发机 · 深度分析

> 归档分析（2026-08-22）。原始收录见 [[wiki/论文/高速接口/Pfaff2025-224G-Transceiver-3nm]]，原文在 `raw/论文/高速接口/A 224 Gb-s 3 pJ-bit 40 dB Insertion Loss Transceiver in 3-nm FinFET CMOS/`。

**定位**：业界首篇**完整（TX+RX+时钟+DSP）3nm 224Gb/s 长距 PAM-4 收发机**，40dB 损耗 BER<1e-6、3pJ/b。全篇在回答一个问题：**波特率翻倍（112GBaud / 56GHz 带宽）时，112G 时代的哪些"理所当然"要重新发明？**

## 一、核心架构赌注：1/8 率时钟（14GHz）⭐

- **动机**：1/2 率（56GHz）/1/4 率（28GHz）的 CMOS 时钟链需要电感/传输线；降到 **1/8 率（14GHz）后整个时钟系统可全 CMOS 无电感**实现
- **代价**：要生成 8 路高精度、50% 占空比相位（RX ADC 采样 + TX DAC 驱动）——把"频率难度"换成"相位精度难度"
- **解决**：新模块 **CCU（Clock Conditioner）** = 注入锁定环形振荡器 ILO1/ILO2 + 7bit 相位旋转器。两个关键洞察：
  - ILO 把 CMOS 时钟链抬高的**相噪基底"重置"回时钟源水平**（环形振荡器相噪比 LC 高 ~20dB，CMOS 缓冲每级把热噪声翻成平直相噪并逐级累积——ILO 是"噪声重置器"）
  - 代价：整条 CMOS 时钟路径必须加 **LDO 稳压**，否则电源噪声毁掉抖动

## 二、接收机：三个反常规设计

### ① 放弃差分对，逆变器化（电压模式）⭐
224G 要求每级 3dB 带宽 75-80GHz：
- 双差分对翻倍 gm（gmN+gmP），**去掉尾电流源**、源端直连电源/地 → 寄生退化最小，增益/带宽顶到工艺极限
- 二极管连接逆变器做负载 → `Av = gm/gmd`，**增益对 PVT 不敏感**（两逆变器互跟踪）
- **线性度论证**：平方律模型下电压模式逆变器输出电流**无三阶失真项**，恒流差分对则有三阶项；二阶靠 NMOS/PMOS 等强 + 差分平衡消除
- 代价：需片内 LDO 校准逆变器电源 `V_AFE`（恒定电流→恒定带宽）+ 提供电压模式必需的 PSRR

### ② 双 T-coil 端接
单 T-coil 只到 30GHz（ESD 主二极管 180fF + CTLE 输入 110fF 拖累）。**第二个 T-coil 把主 ESD 二极管隔离出信号路径** → 带宽由 CTLE 输入电容单独决定：78GHz、RL −9dB 到奈奎斯特。

### ③ "被动增益" CTLE（变压器串联峰值）⭐
20dB 峰值（2 级 ×10dB）在 56GHz 用有源增益太耗电 → 变压器**串联峰值**：
- 变压器形成共轭复极点对；额外零/极点对（RC 耦合）把窄带峰化展宽到全带
- **直流增益 Av 与高频峰化 AHF 独立可调**（vs 分流峰化 CTLE 无此自由度）——3 个可编程参数：`RQ` 控 Q、`RC` 控 Av、`gmd2` 控零点

### ADC：8+10 两级交织取舍
- 一级 **8 路交织**（直接用 8 个 CCU 相位、50% 占空比）→ 省去传统 16 路的时钟去偏斜；代价是一二级间带宽要求更高（T/H 输出阻抗低→加重 AFE/时钟负载），作者明说"用复杂度换简洁"
- 二级 **10 路交织** → SAR 只需 1.4GS/s、0.75V；顶板采样（7bit 仅 6 次开关）、全二进制 CDAC 劈半保共模、StrongARM；offset/gain 全数字修正

### DSP：均衡分工 + MLSD
- **FFE/DFE 分割优化**：第一个后标由 FFE+DFE 分担，**FFE 故意部分均衡、残差交 DFE**——最佳分割随损耗/波特率变化
- **CDR 低延迟旁路**：主路径 44-tap FFE 延迟大，CDR 用独立 **15-tap FFE** 喂 Mueller-Muller PED；type-I 二阶环保证稳态锁定点与频偏无关
- **Skew 检测环路**带宽远低于 CDR（避免互扰）
- **MLSD**：RS-MLSD 只算 4/16 转移（DFE 预选省功耗）；FS-MLSD 全状态。40dB 损耗下 DFE 受突发错误拖累，**必须 FS-MLSD 才达 1e-6**

## 三、发射机

### ① 首创直接 8:1 全速率数据 MUX ⭐
1/8 率时钟 → 需 8:1 MUX（他人都是 4:1）。脉冲发生器法：NAND/NOR/INV 从 1/8 率数据生成全速率脉冲，NMOS 源侧开关转电流；8 个发生器交错拼成 8:1：
- **共享电流源**（两开关共用一个）→ 输出电容与 4:1 方案相当
- **rail-to-rail 内部信号**（此前 224G TX 用低摆幅需辅助电路）；开关只流过 4:1 一半电流 → 可激进缩小（EM 极限内）

### ② 修正温度计分段
全二进制 7 slice：强度比 64:1 太大；全温度计 18 slice：电容爆。**折中 = 4bit 二进制 + 8×12-LSB slice（配对共享 MUX）**：14 slice、步进 16，DAC 输出电容压到 **110fF**。

### ③ 3 LDO + AVS + TDC 校准
串行器/DAC/时钟脊柱各一个 LDO；测量环形振荡器 + 9bit R2R DAC 做 AVS 维持全温/工艺时序裕量；时钟偏斜由 TDC 连续后台校准（14fs 步进）。

## 四、测量解读

| 维度 | 数据 | 点评 |
|---|---|---|
| PLL | −113dBc/Hz@100MHz、90fsrms、全温漂移 12fs | 全数字 PLL + 双 DCO 成果 |
| TX | 55fs RJ + 170fs DJ、RLM 0.992、DNL ±0.2LSB | 首创 8:1 MUX 未牺牲质量 |
| ADC | ENOB 5.5bit@1.2GHz / 4.0bit@50.5GHz，SNDR 仅差 SNR 1-1.5dB | 线性度由噪声主导 = AFE 无三阶失真的实验证据 |
| BER | 40dB/224G：DFE 不够 → FS-MLSD 降 BER 24-75× | MLSD 在长距是刚需 |
| 能效 | TX 1.0 + RX 2.0 = 3.0 pJ/b（**不含 DSP 功耗**） | 口径要留意 |

## 五、批判性评价

**可借鉴**：
1. **系统级时钟决策**：把频率压力换成相位精度压力（1/8 率 + CCU）——"选对战场"
2. 电压模式逆变器 AFE 的三阶失真消隐论证扎实 + V_AFE 校准 LDO = 可复用设计模式
3. CTLE 独立 Av/AHF、FFE/DFE 分割优化、CDR 独立低延迟路径——成熟工程权衡方法论
4. 共享电流源 8:1 MUX 是优雅电路发明

**可质疑**：
1. **3pJ/b 不含 DSP**（44-tap FFE + Viterbi 功耗不透明），对比其他论文要核对口径
2. 8 路一级交织的带宽压力被"作者认为值得"带过，代价未量化
3. 双 DCO 本质是面积换调谐范围，0.5mm²/lane 是否含两个 LC-DCO 未说明
4. **与 8.6（Broadcom 112G/5nm/2.5pJ/b 含全部功耗）不可直接对比**：工艺/速率/口径全不同；领域里"长距必须 DSP+MLSD、短距可模拟"的分野在此得到明确答案

## 六、一句话总结

**用 1/8 率时钟换全 CMOS 无电感、用电压模式换线性度、用首创 8:1 MUX 换 TX 带宽、用 MLSD 换 40dB 长距**——四项"换"构成 224G 世代答卷；内核是**把困难转移到自己有工艺优势的地方**（3nm 逻辑密度扛 DSP）。

## See Also

- [[wiki/论文/高速接口/Pfaff2025-224G-Transceiver-3nm]] — 原始收录笔记
- [[wiki/论文/高速接口/Broadcom2026-280mW-112G-Transceiver-Analysis]] — 112G 模拟 RX 路线（对比对象）
- [[wiki/论文/高速接口/ISSCC2026-Session8-D2D-Transceivers]] — 同领域 Session 综述
- [[wiki/论文/论文数据库]] — 论文数据库
