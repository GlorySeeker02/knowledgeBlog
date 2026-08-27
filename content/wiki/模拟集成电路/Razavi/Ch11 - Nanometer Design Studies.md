---
title: "Ch11 纳米级设计研究"
source: "Razavi - Design of Analog CMOS Integrated Circuits (2nd Edition)"
tags:
  - analog-design
  - CMOS
  - nanometer
  - FinFET
  - design-methodology
  - op-amp
  - transconductance
  - frequency-compensation
  - CMFB
  - Razavi
---

# Ch11 纳米级设计研究

## 本章定位

本章是 Razavi 教材的**方法论转折点**——前 10 章逐步建构了电路分析的理论框架，而从本章开始，作者带领读者走进 **40 nm CMOS 工艺、1 V 供电**的真实设计战场，完整呈现一个模拟设计师在面对纳米级工艺时的思维过程与工程决策链。

本章不是理论的堆砌，而是**设计的"列传"**——从器件特性差异分析开始，经过系统化的晶体管参数标定方法，再到三个完整的运放设计迭代（telescopic → two-stage → high-speed），每一步都展示了"先理解、再调整、手算主导"的模拟设计哲学。

> 本章强烈建议在阅读 Ch9 运放设计实例后再开始。

## 核心概念

### 1. 长沟道模型的崩溃

传统平方律模型假定 $I_D = \frac{1}{2}\mu_n C_{ox}\frac{W}{L}(V_{GS}-V_{TH})^2$，但在 40 nm 器件中，实际 $I_D$-$V_{DS}$ 特性与"最佳拟合"平方律曲线严重偏离（Fig. 11.1）。更糟的是，**饱和区和线性区的分界变得模糊**，沟道长度调制效应极其严重。

### 2. 速度饱和 (Velocity Saturation)

- **成因**：器件沟道长度从 1 $\mu$m 缩至 40 nm（25 倍），但允许的 $V_{DS}$ 仅从 5 V 降至约 1 V，导致横向电场超过临界电场 $E_{crit} \approx$ 1 V/$\mu$m。
- **极端速度饱和下的三个蜕变**：
  1. $I_D \propto v_{sat} W C_{ox}(V_{GS}-V_{TH})$ — 线性依赖于过驱动电压，**与沟道长度无关**。
  2. $I_D$ 在 $V_{DS} < V_{GS}-V_{TH}$ 时就提前饱和（Fig. 11.4）。
  3. $g_m \approx W C_{ox} v_{sat}$ — **几乎恒定**，不再随 $I_D$ 或 $V_{GS}$ 线性增长。

### 3. 垂直电场迁移率退化

随着 $V_{GS}$ 增大，垂直电场增强，载流子迁移率 $\mu$ 下降（Fig. 11.5）。若近似为 $\mu = \mu_0 / [1 + \theta(V_{GS}-V_{TH})]$，则

$$
g_m = \frac{1}{2}\mu_0 C_{ox}\frac{W}{L} \cdot \frac{2(V_{GS}-V_{TH}) + \theta(V_{GS}-V_{TH})^2}{[1+\theta(V_{GS}-V_{TH})]^2}
$$

当 $V_{GS}-V_{TH} \gg 2/\theta$ 时，$g_m$ 趋向饱和常数，不再增长（Fig. 11.6）。

### 4. 参考晶体管法 —— 纳米设计的方法论基石

面对长沟道模型的失效，Razavi 提出了一套优雅的实用方法：**仿真一次参考器件，缩放解决全部设计**。

$$
\text{参考器件：} W_{REF}/L_{min} = 5\ \mu\text{m} / 40\ \text{nm}
$$

对该参考器件做一次完整的 $I_D$-$V_{DS}$ 和 $g_m$-$V_{GS}$ 仿真，之后的晶体管设计只需**根据需求线性缩放** $W$ 和 $I_D$：

$$
\text{缩放因子 } K = \frac{I_{D,target}}{I_{D,REF}}
$$

缩放后：$W = K \cdot W_{REF},\quad g_m = K \cdot g_{m,REF}$

**关键性质**：这种缩放保持了**电流密度** ($I_D/W$) 和 **$g_m/I_D$ 比值**不变，相当于把 $K$ 个完全相同的晶体管并联。

## 关键设计方法论

### 跨导缩放的六种路径

给定长沟道关系 $g_m = \mu_n C_{ox}\frac{W}{L}(V_{GS}-V_{TH}) = \frac{2I_D}{V_{GS}-V_{TH}} = \sqrt{2\mu_n C_{ox}\frac{W}{L}I_D}$，调节 $g_m$ 有三种自由度，产生六种缩放策略（Fig. 11.7）：

| 路径 | 固定参数 | 改变 | 代价 |
|------|---------|------|------|
| (a) | $V_{GS}-V_{TH}$ 恒定 | $\uparrow W/L$ | $\uparrow I_D, \uparrow$ 器件电容 |
| (b) | $W/L$ 恒定 | $\uparrow V_{GS}-V_{TH}$ | $\uparrow I_D, \uparrow V_{DS,min}$ |
| (c) | $I_D$ 恒定 | $\uparrow W/L$ | $\downarrow V_{GS}-V_{TH}$ → 最终进入亚阈值，$g_m$ 不能无限增长 |
| (d) | $W/L$ 恒定 | $\uparrow I_D$ | $\uparrow V_{GS}-V_{TH}, \uparrow V_{DS,min}$ |
| (e) | $V_{GS}-V_{TH}$ 恒定 | $\uparrow I_D, \uparrow W/L$ | 等价于 (a)，保持电流密度 |
| (f) | $I_D$ 恒定 | $\downarrow V_{GS}-V_{TH}, \uparrow W/L$ | $\uparrow$ 器件电容 |

> **在现代低压设计中，最常用的是路径 (c) → (d) → (e) 的渐进推进**：先拉宽器件降低过驱动，不行就加电流，再不行就保持电流密度同时放大尺寸。

### 三种晶体管设计情景

基于 $g_m$-$I_D$ 平面的图形化方法（Fig. 11.14），从原点出发过目标点的射线与参考器件的 $g_m$-$I_D$ 特性曲线交点，即给出所需的过驱动电压和缩放因子。三种典型设计情景（Table 11.1）：

#### 情景一：给定 $I_D$ 和 $V_{DS,min}$

应用场景：已知功耗预算和电压裕度。

1. 先确定参考器件在目标 $V_{DS}=V_{DS,min}$ 时的过驱动电压 $V_{GS}-V_{TH}$ → 读取 $I_{D,REF}$ → 缩放因子 $K = I_D / I_{D,REF}$。
2. 如果 $g_m$ 不够 → 增大 $I_D$ 和 $W/L$（牺牲功耗）。

#### 情景二：给定 $g_m$ 和 $I_D$

应用场景：需要特定 $g_m$，同时有功耗上限。

1. 在 $g_m$-$I_D$ 平面画射线，与参考器件特性交点 → 得到 $(I_{D,REF}, g_{m,REF})$ 和过驱动 → 缩放 $K = g_m/g_{m,REF}$。
2. 如果 $V_{DS,min}$ 太大 → 增大 $W/L$ 降低过驱动。

#### 情景三：给定 $g_m$ 和 $V_{DS,min}$

应用场景：指标给定 $g_m$ 和电压摆幅，没有明确电流约束。

1. 从 $g_m$-$V_{GS}$ 平面找 $V_{GS}-V_{TH}=V_{DS,min}$ 对应的 $g_{m,REF}$ 和 $I_{D,REF}$ → 缩放 $K = g_m/g_{m,REF}$。
2. 如果 $I_D$ 太大 → 转入情景二的流程。

> **亚阈值上限法则**：当仅给定 $g_m$ 时，务必验证 $g_m < I_D/(\xi V_T)$，否则即使用无限大的 $W/L$ 也不可能达到目标 $g_m$。在室温下 $\xi \approx 1.5$，$g_{m,max} \approx I_D/(39\text{ mV})$。

### 沟道长度的选择

当 $r_O$ 不够时，必须增加 $L$。但需注意：**绘制长度不等于有效长度**。$L_{eff} = L_{drawn} - 2L_D$，所以 $L_{drawn}$ 加倍时 $L_{eff}$ 并不严格加倍。实践中应为 60 nm, 80 nm, 100 nm 等不同绘制长度分别仿真 $I_D$-$V_{DS}$、$g_m$ 和 $r_O$ 特性。

## 纳米工艺运放设计实例

目标规格（贯穿 11.5-11.6 节的三个设计都围绕以下指标展开）：

| 参数 | 目标值 |
|------|--------|
| 差分输出摆幅 | 1 V$_{pp}$ |
| 功耗 | 2 mW |
| 电压增益 | 500 |
| 供电电压 | 1 V |

### 设计一：Telescopic Cascode 运放 —— 失败的宝贵教训

尝试用单级 telescopic cascode 实现增益 500（Fig. 11.19）。

- **根本矛盾**：40 nm 工艺下 $g_m r_O$ 仅 $\approx$ 7-10（NMOS）和 5-7（PMOS），远远不够产生 $R_{out} > 26\text{ k}\Omega$。
- **仿真验证**：$(g_m r_O)_{NMOS} r_O \approx 5.3\text{ k}\Omega$，差分增益仅约 30（目标 500 的 6%）。
- **为什么继续设计**：虽然增益目标注定无法达成，但这个过程完整演练了：
  1. **偏置电路设计**（Fig. 11.24-11.26）：电流镜 + 低电压 cascode 偏置 + 电平移位器
  2. **共模反馈 (CMFB) 设计**：互补源跟随器感知 CM 电平（Fig. 11.27），加权求和消除 $V_{GS}$ 项
  3. **CMFB 稳定性补偿**：用误差放大器输出端对地电容实现（因为误差放大器无信号反相，无法用 Miller 补偿）
  4. **差分补偿**：在反馈电阻上并联电容，利用 Miller 效应进行零极点对消

> **关键教训**：设计之前要做快速可行性估算（"back-of-the-envelope"）。如果在纸面上 $g_m r_O$ 乘积不够，telescopic 就不可能达标。

### 设计二：两级运放 —— 迭代优化之路

Telescopic 失败后，策略转向两级结构（Fig. 11.40a）。

#### 第一级设计

- 单端输出摆幅仅需 50 mV$_{pp}$（第二级增益约 10，总增益 500 = 50 × 10）
- 放宽 $V_{DS}$ 分配：$V_{DS,N}=150\text{ mV}$, $V_{DS,P}=200\text{ mV}$
- 结果（Fig. 11.39b）：第一级增益约 50

#### 第二级设计

- PFET 输入（因为 $V_{CM,out1}\approx 0.55\text{ V}$ 更适合 PFET 的 $V_{GS}$，且 PMOS 的 $g_m r_O$ 更高）
- 最终特性（Fig. 11.41）：增益 > 500，单端摆幅 530 mV

#### CMFB 架构

- **第一级**：沿用 telescope 的误差放大器方案
- **第二级**：电阻分压感知 CM 电平，直接反馈到 NMOS 电流源栅极（Fig. 11.42）

#### 频率补偿

- 原始开环：增益 57 dB，$\omega_u = 3.2\text{ GHz}$，相位裕度仅 $-8^\circ$
- Miller 补偿：$C_C = 4.5\text{ pF}$，主极点约 344 kHz，增益交点 350 MHz，相位裕度 18$^\circ$
- 引入调零电阻 $R_z = 190\ \Omega$：零点移动到 185 MHz 第二极点 → 相位裕度 96$^\circ$（过补偿）
- 迭代优化：$C_C=0.8\text{ pF}$, $R_z=450\ \Omega$ → GBW = 1.9 GHz, PM = 65$^\circ$
- 闭环处理：反馈电阻 + 输入电容引入额外极点，通过增大 $R_z$ 到 1500 $\Omega$ 解决（Fig. 11.47）

> **设计启示**：开环 65$^\circ$ 的相位裕度在加入闭环电阻反馈网络后可能严重恶化，因为电阻与运放输入电容形成约 95 MHz 的极点。**稳定性必须在最终闭环环境中验证。**

### 设计三：高速精密放大器 —— 速度与功耗的权衡艺术

规格演进为实际 ADC 需求的放大器：

| 参数 | 目标值 |
|------|--------|
| 闭环增益 | 4 |
| 增益误差 | $\leq$ 1% |
| 差分输出摆幅 | 1 V$_{pp}$ |
| 负载电容 | 1 pF |
| 建立时间 (0.5%) | 5 ns |
| 供电 | 1 V |

#### 精度考量

- 增益误差 < 1% → 开环增益 $A_0 \geq 500$（闭环增益为 4 时）
- 采用**容性反馈**（Fig. 11.51a）替代阻性反馈：
  - 不引入额外的低频极点（$C_1$、$C_2$ 仅与 $C_L$ 并联）
  - 闭环增益 $\approx C_1/C_2$（更精确的：需计入 $C_{in}$）

#### 速度要求

一阶近似：$t_s = -\tau \ln 0.005 = 5.3\tau$ → $\tau \leq 0.94\text{ ns}$ → 闭环带宽 $\geq 170\text{ MHz}$

由于 $\beta = 1/5$（闭环增益 4），补偿标准放松：主极点只需 1.36 MHz（而非 unity-gain 反馈所需的 344 kHz）。

#### 线性缩放法：功耗优化

当电路过设计时（建立时间远超要求），通过**线性缩放**回收功耗：

- 所有晶体管宽度和偏置电流统一缩小 $\alpha$ 倍
- 电压增益和摆幅不变（功率降低 $\alpha$ 倍）
- **$C_C$ 保持不变**（因为第一级输出阻抗放大 $\alpha$ 倍恰好抵消）
- **$R_z$ 放大 $\alpha$ 倍**（第二级输出极点降低 $\alpha$ 倍，零点也需跟随）
- 最终实现 **8 倍功耗缩减**（从约 3 mW 降至 370 $\mu$W），建立时间从 800 ps 变为 4.5 ns

#### 大信号表现

大摆幅时开环增益从 518 降至 403（$V_{DS}$ 压缩导致）。解决方案：
- 加倍第一级 cascode NMOS 和第二级 NMOS 电流源的 $W$ 和 $L$
- 但引入 cascode 源极额外极点（约 1.15 GHz），劣化相位裕度
- 采用 **cascode 补偿**（Chapter 10）解决 → 最终大信号建立 $t_s < 5\text{ ns}$

## 设计启示 —— Razavi 的三步设计哲学

本章结尾，Razavi 总结了优秀模拟设计的三个步骤：

1. **深入理解电路行为，找到问题的根本原因。** 不是盲目调参，而是搞清楚"为什么振荡"、"为什么增益不够"。
2. **只调整与根本原因相关的电路参数。** 不做随机尝试，每次修改都有明确的物理动机。
3. **持续探索新的电路技术与想法。** 有时会走入死胡同，但多数时候能找到更优解。

此外，Razavi 特别强调：**高性能模拟设计需要人的智慧，而非自动化工具。** 手算 + 物理直觉 + 迭代仿真，三者缺一不可。

### 本章的工程清单

从三个设计实例中，可以提炼出一套完整的运放设计工程清单：

1. **电压空间分配**：根据输出摆幅和供电电压，为每个晶体管分配 $V_{DS}$ 和 $I_D$
2. **参考晶体管表征与缩放**：一次仿真，反复缩放
3. **快速增益估算**：$g_m r_O$ 产品是否支撑目标增益
4. **DC Sweep 验证**：检验偏置条件和线性度
5. **偏置电路设计**：电流镜 + 低电压 cascode 偏置 + 电平移位
6. **CMFB 设计与补偿**：感知 → 比较 → 反馈，且**必须在差分反馈环境下验证稳定性**
7. **闭环瞬态分析**：同时考察 CM 和差分稳定性

## 章节关联

- **Ch02 MOSFET 基础**：本章反复用到大信号/小信号模型的区分，以及 $g_m r_O$ 的基本概念
- **Ch09 运放设计实例**：本章是 Ch09 在纳米工艺下的"实战版"——同样的 telescopic、two-stage 拓扑，但面临 1 V 供电和低 $g_m r_O$ 的严峻挑战
- **Ch10 频率补偿**：本章大量运用 Miller 补偿、调零电阻、零极点对消等技术
- **Ch17 短沟道效应**：本章仅在 11.1-11.2 节做了浅层介绍，Ch17 将给出速度饱和、DIBL、迁移率退化的完整建模
- **Ch19 版图效应**：纳米工艺的版图寄生效应更严重，Ch19 将系统讨论

## See Also

- [[wiki/数字信号处理/DSP 系统学习课程]]
- [[Ch09 - Operational Amplifiers]]
- [[Ch17 - Short-Channel Effects and Device Models]]

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd]]

## 检索关键词

`short-channel effects` `velocity saturation` `mobility degradation` `transconductance scaling` `transistor design methodology` `reference transistor` `telescopic cascode` `two-stage op amp` `CMFB` `Miller compensation` `pole-zero cancellation` `linear scaling` `capacitive feedback` `40nm CMOS` `low-voltage design`
