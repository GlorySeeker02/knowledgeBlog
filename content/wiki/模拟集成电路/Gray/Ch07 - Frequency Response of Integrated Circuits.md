---
title: "第7章 集成电路的频率响应"
source: "Gray, Hurst, Lewis, Meyer — Analysis and Design of Analog Integrated Circuits, 5th Edition, Ch07"
tags:
  - analog-design
  - analog-IC
  - frequency-response
  - Bode-plot
  - Miller-effect
  - zero-value-time-constant
  - dominant-pole
---

# 第7章 集成电路的频率响应

## 本章定位

本章是 Gray 教材中从 **低频小信号分析** 到 **高频行为** 的转折点。前面章节 (Ch03–Ch06) 都在低频假设下分析电路增益、输入输出阻抗等，忽略寄生电容。本章系统引入 **寄生电容如何限制电路带宽** ，覆盖单级放大器、多级放大器，并以 NE5234 运放为实例演示分析方法。

核心问题是：**频率升高时，电路中哪些电容开始起作用？它们如何限制带宽？怎样快速估算 −3 dB 频率？**

---

## 核心概念

### 1. Miller 效应与 Miller 近似

> [!important] Miller 效应
> 跨接在反相放大器输入-输出之间的反馈电容 $C_f$，在输入端等效为放大了 $(1+|A_v|)$ 倍的 **Miller 电容**：
> $$C_M = C_f(1 + g_m R_L)$$
> 这是限制共源/共射放大器带宽的 **最主要因素**。

- **物理本质**：输入小电压 $v_1$ 在输出端产生反相大电压 $-g_m R_L v_1$，$C_f$ 两端电压差为 $(1+g_m R_L)v_1$，需要大电流驱动，等效于输入端有一个大电容。
- **Miller 近似**：用低频增益 $A_{v0} = -g_m R_L$ 代替频率相关的 $A_v(s)$ 来计算 $C_M$。当极点分离良好时，近似精度很高。

### 2. 通用小信号模型

教材建立了一个 **统一模型**，通过参数替换可同时适用于 BJT 和 MOS：

| 通用模型 | BJT | MOS |
|:---:|:---:|:---:|
| $r_x$ | $r_b$ | 0 |
| $r_{in}$ | $r_\pi$ | $\infty$ |
| $C_{in}$ | $C_\pi$ | $C_{gs}$ |
| $C_f$ | $C_\mu$ | $C_{gd}$ |
| $r_o$ | $r_o$ | $r_o$ |

> [!note] 器件特有的寄生
> MOS 还有 $g_{mb}$、$C_{sb}$、$C_{db}$、$C_{gb}$ 不在此统一模型中，需单独处理。

### 3. 主极点近似 (Dominant-Pole Approximation)

当极点分离良好 ($|p_1| \ll |p_2|, |p_3|, \dots$)，分母系数满足：
$$b_1 = -\sum \frac{1}{p_i} \approx -\frac{1}{p_1}$$

因此：
$$\omega_{-3\text{dB}} \approx |p_1| \approx \frac{1}{b_1}$$

### 4. 零值时间常数分析 (Zero-Value Time Constant / Open-Circuit Time Constant)

**最重要的手工估算工具。** 不需要完整推导传递函数，直接估算 $\omega_{-3\text{dB}}$。

$$\omega_{-3\text{dB}} \approx \frac{1}{\sum \tau_{i0}}$$

其中 $\tau_{i0} = C_i R_{i0}$，$R_{i0}$ 是将 **所有其他电容开路** 时，从 $C_i$ 两端看进去的驱动点电阻。

> [!tip] 核心价值
> - 大幅减少计算量
> - 直观看出 **哪个电容是带宽瓶颈**
> - 即使极点不严格分离，误差也只在 ~22% 以内

### 5. 短路时间常数分析 (Short-Circuit Time Constant)

**对偶方法**，用于估算 **非主极点**（最高频极点）的位置：
$$\frac{1}{|p_2|} \approx \sum \tau_{si}$$

其中 $\tau_{si}$ 是将所有其他电容 **短路** 后求出的时间常数。

> [!warning] 前提
> 仅当两极 **实、分离良好** 时成立。电容回路存在时该方法失效。

### 6. 共模频率响应

差分放大器的 CMRR 随频率升高而 **恶化**：
- 尾电流源输出电容 $C_T$ 使共模增益 $A_{cm}$ 在 $\omega > 1/R_T C_T$ 后以 +6 dB/oct 上升
- 差模增益 $A_{dm}$ 在主极点后以 −6 dB/oct 下降
- CMRR 以 **−12 dB/oct** 下降，高频共模抑制能力大幅削弱

### 7. 带宽-上升时间关系

对于单极点系统：
$$t_r = \frac{0.35}{f_{-3\text{dB}}}$$

$f_{-3\text{dB}} = 10\text{ MHz} \implies t_r = 35\text{ ns}$。

---

## 关键公式与结论

### 共射/共源放大器 (CE/CS)

| 参数 | 公式 |
|:---|:---|
| Miller 电容 | $C_M = C_f(1 + g_m R_L)$ |
| 总输入电容 | $C_t = C_{in} + C_M$ |
| 主极点 (Miller近似) | $\omega_{-3\text{dB}} \approx \dfrac{1}{R C_t}$，$R = (R_S+r_x)\;\|\;r_{in}$ |
| 非主极点 | $\|p_2\| \approx \dfrac{g_m}{C_{in} + C_f(1 + C_{in}/C_f \cdot R/R_L)} > \omega_T$ |
| 传输零点 | $z = +\dfrac{g_m}{C_f}$（正实零点，源自 $C_f$ 前馈） |

> [!important] 结论
> Miller 电容 $C_M$ 几乎总是带宽的主导限制因素，因为 $g_m R_L \gg 1$。

### 射随/源随 (EF/SF)

| 特点 | 说明 |
|:---|:---|
| 增益 | 低频约 1，无 Miller 效应（$C_\mu$/$C_{gd}$ 一端接地） |
| 带宽 | 可达 $f_T$ 量级，远超 CE/CS |
| 极点-零点对 | $p_1 \approx -\omega_T$，$z_1$ 略大于 $\|p_1\|$，两者接近 |
| 输入阻抗 | 容性，等效电容 $C_{in}/(1+g_m R_L)$ |
| 输出阻抗 | 可呈 **感性**（当 $1/g_m < R_S$），驱动容性负载可能振荡 |

### 共基/共栅 (CB/CG)

| 特点 | 说明 |
|:---|:---|
| 电流增益 | $\approx 1$，主极点在 $\omega_T$ 附近 |
| Miller 效应 | **无**（反馈电容不跨接反相端） |
| 输入阻抗 | 低阻 ($\approx 1/g_m$)，高频可呈感性 |
| CS vs CG 对比 | CG 带宽远大于 CS，适合宽频应用 |

### Cascode 结构

- $T_1$ (CE/CS) 的负载是 $T_2$ (CB/CG) 的低输入阻抗 $\approx 1/g_{m2}$
- $v_i \to v_x$ 增益 $\approx 1$，$C_f$ 的 Miller 效应被大幅抑制
- 同时提供高输出阻抗和优秀反向隔离

### 电流镜负载差分对

- 电流镜节点电容 $C_x$ 引入一个 **极点-零点对**
- $|p| = g_{m3}/C_x$，$|z| = 2g_{m3}/C_x$，间隔一个倍频程
- 跨导从低频 $g_{m1}$ 降到高频 $g_{m1}/2$
- 极点-零点对的影响远小于单独的极点或零点

---

## 重要分析方法

### 零值时间常数法的标准步骤
1. 画出小信号等效电路，标注所有电容
2. 对每个电容 $C_i$，将所有其他电容开路，求 $R_{i0}$
3. 计算 $\tau_{i0} = C_i R_{i0}$
4. $\omega_{-3\text{dB}} \approx 1 / \sum \tau_{i0}$

### $R_{gd0}$ 通式（最重要）

对于 $C_{gd}/C_\mu$ 的零值电阻，有通用公式：
$$R_{gd0} = R_A + R_B + g_m R_A R_B$$

其中 $R_A$ 是从电容"输入端"看进去的电阻，$R_B$ 是从"输出端"看进去的负载电阻。该公式对 CE/CS/cascode 各级均适用。

### NE5234 分析要点
- 5.5 pF 补偿电容 $C_{22}$ 经 Miller 放大后等效约 7000 pF
- 与第一级输出电阻 ~610 kΩ 形成 $\sim$22 Hz 的主极点
- 非主极点由仿真确定，影响反馈稳定性（Ch09）

---

## 设计启示

1. **带宽瓶颈通常是 Miller 电容**——减小 $R_L$ 或使用 Cascode 结构可显著改善
2. **零值时间常数法告诉你"该优化哪里"**——找出贡献最大的 $\tau_{i0}$ 下手
3. **射随/源随带宽远优于 CE/CS**，但输出阻抗可能呈感性，驱动容性负载需谨慎
4. **Cascode 用带宽换 Miller 效应**——$T_1$ 增益压低到 $\approx 1$，整体带宽大幅提高
5. **差分管尾电流源的 $C_T$ 直接限制高频 CMRR**
6. **电流镜负载的极点-零点 doublet 影响高频 PSRR 和 settling**

---

## 章节关联

- **前置章节**：[[Ch03 - Single-Transistor and Multiple-Transistor Amplifiers|Ch03]] 提供低频增益和阻抗分析基础；[[Ch06 - Operational Amplifiers with Single-Ended Outputs|Ch06]] 提供 NE5234 的电路结构
- **后续章节**：[[Ch08 - Feedback|Ch08]] 将频率响应引入反馈系统分析；[[Ch09 - Frequency Response and Stability of Feedback Amplifiers|Ch09]] 讨论极点位置如何影响稳定性及频率补偿技术
- **本章分析方法 (零值时间常数、短路时间常数、主极点近似) 是 Ch09 反馈稳定性分析的必要前提**

---

## 检索关键词

`频率响应` `Miller效应` `Miller近似` `零值时间常数` `ZVT` `开路时间常数` `短路时间常数` `主极点` `非主极点` `共模抑制比频率特性` `Cascode带宽` `射随带宽` `源随带宽` `电流镜极点零点对` `NE5234` `−3dB频率` `上升时间` `带宽` `$C_{gd}$` `$C_\mu$`

---

## Sources

- [[raw/模拟集成电路/Gray/Ch07 - Frequency Response of Integrated Circuits|Gray Ch07: Frequency Response of Integrated Circuits]]

## See Also

- [[Ch03 - Single-Transistor and Multiple-Transistor Amplifiers|Ch03 单级与多级放大器]]
- [[Ch06 - Operational Amplifiers with Single-Ended Outputs|Ch06 运放]]
- [[Ch08 - Feedback|Ch08 反馈]]
- [[Ch09 - Frequency Response and Stability of Feedback Amplifiers|Ch09 稳定性与频率补偿]]
