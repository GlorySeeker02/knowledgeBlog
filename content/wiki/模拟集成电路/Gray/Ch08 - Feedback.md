---
title: "Ch08 反馈"
source: "Analysis and Design of Analog Integrated Circuits, 5th Edition, Paul R. Gray et al."
tags:
  - analog-design
  - analog-IC
  - feedback
  - two-port
  - return-ratio
---

# Ch08 反馈

## 本章定位

本章是模拟集成电路设计中**反馈理论**的核心章节。它系统讲述了负反馈的四方面内容：
1. 负反馈的基本原理与理想反馈方程；
2. 四种反馈组态的划分及其对端口阻抗、增益稳定性的影响；
3. 实际电路中反馈网络负载效应的处理方法（两端口法）；
4. 另一种不依赖两端口的分析方法——Return-Ratio 法（含 Blackman 阻抗公式）。

本章是整个放大器设计理论的"枢纽"：第 6 章的运放提供了基本增益级，第 7 章的频率响应决定了基本放大器的频域特性，本章告诉你如何用反馈闭环该基本放大器来获得稳定、可控的性能，而第 9 章则处理闭环之后的稳定性与频率补偿问题。

> [!NOTE]
> Chapter 9 负责解决本章反复提到的问题——**反馈引起的振荡**。两章应配合阅读。

## 核心概念

### 1. 理想负反馈方程

反馈放大器的核心数学框架是：

$$
A = \frac{a}{1 + af} = \frac{a}{1 + T}
$$

其中 $a$ 是基本放大器增益，$f$ 是反馈网络传递函数，$T = af$ 为**环路增益**（loop gain），$A$ 为闭环增益。

> [!important] 关键结论
> 当 $T \gg 1$ 时，$A \approx 1/f$，闭环增益**仅由无源反馈网络决定**，与有源器件的参数漂移无关。

反馈通过**强制 $S_{fb} \approx S_i$** 来工作——环路不断放大误差信号 $S_\epsilon = S_i - S_{fb}$，使其趋近于零。

### 2. 增益灵敏度

对 $A$ 求微分可知：

$$
\frac{dA}{A} = \frac{1}{1+T} \cdot \frac{da}{a}
$$

> [!tip] 设计直觉
> 反馈将增益的相对变化量缩减了 $(1+T)$ 倍。若 $T=100$，$a$ 变化 10%，$A$ 仅变化 0.1%。

### 3. 失真抑制

反馈**不消除失真，而是抑制失真**。由于失真本质上就是基本放大器转移特性斜率的变化，反馈使闭环增益 $A$ 对 $a$ 的变化不敏感，从而平滑了整体转移特性。但这一效果受放大器硬饱和（$a=0$）的限制——在硬饱和区反馈无能为力。

### 4. 四种反馈组态总览

反馈组态由两个维度决定：输出采样的是电压还是电流；输入端是串联注入还是并联注入。

| 组态 | 采样 | 注入 | 稳定化的传递函数 | $Z_i$ | $Z_o$ | 两端口参数 | 理想放大器类型 |
|------|------|------|-------------------|-------|-------|-------------|-----------------|
| Series-Shunt | 输出电压 | 串联电压 | $v_o/v_i$ (电压增益) | 升高 $(1+T)$ | 降低 $(1+T)$ | $h$ 参数 | 电压放大器 |
| Shunt-Shunt | 输出电压 | 并联电流 | $v_o/i_i$ (互阻) | 降低 $(1+T)$ | 降低 $(1+T)$ | $y$ 参数 | 互阻放大器 |
| Shunt-Series | 输出电流 | 并联电流 | $i_o/i_i$ (电流增益) | 降低 $(1+T)$ | 升高 $(1+T)$ | $g$ 参数 | 电流放大器 |
| Series-Series | 输出电流 | 串联电压 | $i_o/v_i$ (互导) | 升高 $(1+T)$ | 升高 $(1+T)$ | $z$ 参数 | 互导放大器 |

**记忆规则**：
- 输入端串联反馈 → $Z_i$ 升高
- 输入端并联反馈 → $Z_i$ 降低
- 输出端并联（shunt、采样电压）→ $Z_o$ 降低
- 输出端串联（series、采样电流）→ $Z_o$ 升高

### 5. 理想驱动源条件

反馈效果要充分发挥，驱动源阻抗必须与反馈类型匹配：
- **输入端串联**（Series）：$Z_s \ll Z_i$，理想驱动源是电压源
- **输入端并联**（Shunt）：$Z_s \gg Z_i$，理想驱动源是电流源

## 关键公式与结论

### 闭环增益公式

$$
A = \frac{a}{1+T}, \quad T = af
$$

### 端口阻抗公式

| 组态 | 输入阻抗 | 输出阻抗 |
|------|----------|----------|
| Series-Shunt | $Z_i(1+T)$ | $\dfrac{Z_o}{1+T}$ |
| Shunt-Shunt | $\dfrac{Z_i}{1+T}$ | $\dfrac{Z_o}{1+T}$ |
| Shunt-Series | $\dfrac{Z_i}{1+T}$ | $Z_o(1+T)$ |
| Series-Series | $Z_i(1+T)$ | $Z_o(1+T)$ |

### Return-Ratio 法的闭环增益

$$
A = A_\infty \frac{\mathfrak{R}}{1+\mathfrak{R}} + \frac{d}{1+\mathfrak{R}}
$$

其中：
- $\mathfrak{R}$ 是 return ratio（与被控源 $k$ 的选点有关）
- $A_\infty$ 是 $\mathfrak{R} \to \infty$ 时的理想闭环增益（通常仅由无源反馈网络决定，等价于 $1/f$）
- $d$ 是直接馈通（$k=0$ 时输入到输出的传递函数）

> [!abstract] Return Ratio 与 Loop Gain 的区别
> $\mathfrak{R}$ 与两端口法中的 $T = af$ 不同。$\mathfrak{R}$ 针对某一特定受控源计算，而 $T$ 是对整个环路统一建模的结果。两者在数值上通常接近但不一定相等。本书将 $T$ 称为 loop gain，将 $\mathfrak{R}$ 称为 return ratio，以避免混淆。

### Blackman 阻抗公式

$$
Z_{\text{port}} = Z_{\text{port}}(k=0) \; \frac{1 + \mathfrak{R}(\text{port open})}{1 + \mathfrak{R}(\text{port shorted})}
$$

这是 Return-Ratio 分析中最强大的工具之一——**一个公式适用于任意端口、任意反馈类型**，无需事先判断反馈组态。

## 重要分析方法

### 方法一：两端口法（Two-Port Analysis）

**适合场景**：反馈网络可以明确地从基本放大器中分离出来。

**步骤**：

1. **识别反馈类型**——判断输入端是 series 还是 shunt，输出端采样电压还是电流。
2. **选择正确的两端口参数**——根据 [[#四种反馈组态总览|总览表]] 选取 $y$/$z$/$h$/$g$ 参数。
3. **计算反馈网络负载效应**：
   - 输入端并联 → 短路输出反馈节点，求输入端加载阻抗
   - 输入端串联 → 开路输出反馈节点
   - 输出端并联 → 短路输入反馈节点，求输出端加载阻抗
   - 输出端串联 → 开路输入反馈节点
4. **计算 $f$**：
   - 输入端并联 → 用电压源驱动反馈网络输出端，求输入端短路电流
   - 输入端串联 → 用电流源驱动反馈网络输出端，求输入端开路电压
5. **吸收负载效应**：将 $z_{11f}$/$y_{11f}$/$h_{11f}$/$g_{11f}$ 等并入基本放大器的相应端子，**此时的"基本放大器"已包含反馈网络的负载效应**。
6. 计算此新基本放大器的 $a$、$Z_i$、$Z_o$，再用理想反馈公式推算闭环性能。

> [!warning] 关键细节
> 两端口的输入输出阻抗等所有参数必须在**已包含反馈负载效应**的基本放大器上计算，然后再乘以或除以 $(1+T)$ 得到闭环值——不是拿原始未加载的 $a$ 回路去算。

**简化假设**（几乎总是成立）：
- 基本放大器的反向传输忽略（$y_{12a} \approx 0, z_{12a} \approx 0$ 等）
- 反馈网络的正向馈通忽略（$y_{21f} \approx 0, z_{21f} \approx 0$ 等）

### 方法二：Return-Ratio 法

**适合场景**：电路复杂、反馈网络与基本放大器难以解耦；不想判断反馈组态时。

**核心思想**：在电路中选定一个**受控源 $k$**（如某个晶体管的 $g_m$），断开反馈环，计算回到同一点的 return ratio $\mathfrak{R}$。

**计算 $\mathfrak{R}$ 的步骤**：
1. 将独立源置零。
2. 断开受控源与电路的连接。
3. 在断开处未连接受控源的那一侧，接一个与受控源同类型的独立测试源 $s_t$。
4. 计算受控源产生的返回信号 $s_r$。
5. $\mathfrak{R} = -s_r / s_t$。

**闭环增益的快速计算**：

$$
A = A_\infty \frac{\mathfrak{R}}{1+\mathfrak{R}} + \frac{d}{1+\mathfrak{R}}
$$

- $A_\infty$：令 $k \to \infty$ 计算闭环增益（此时控制信号 $s_{ic} \to 0$，在负反馈电路中可直接由电路拓扑推出）
- $d$：令 $k=0$ 计算输入→输出传递函数
- 低频时通常 $|d| \ll |A_\infty \mathfrak{R}|$，可忽略 $d$ 项

**Blackman 公式的应用**：
- 端口开路时的 return ratio $\to$ 分子
- 端口短路时的 return ratio $\to$ 分母
- 通常其中一个为零，此时反馈升高或降低阻抗 $(1+\mathfrak{R})$ 倍

> [!example] Super-Source Follower
> Chapter 8 中用 Blackman 公式分析了 super-source follower 的输出阻抗，得到 $R_o \approx 1/(g_{m1} g_{m2} r_{o1})$，远低于普通 source follower 的 $1/g_m$。该电路是负反馈降低输出阻抗的典型应用。

### 两种方法的对比

| 维度 | 两端口法 | Return-Ratio 法 |
|------|----------|------------------|
| 是否需要判断反馈组态 | 必须 | 不需要 |
| 两端口参数选择 | 每种组态对应一种参数 | 不需要 |
| 推导过程 | 需要解耦和重新组装电路 | 直接在小信号等效电路上操作 |
| 闭环阻抗公式 | 分情况乘/除 $(1+T)$ | Blackman 公式统一处理 |
| 前向路径建模 | 所有前向增益汇聚为 $a$ | 分为有效前向增益 $\mathfrak{R}A_\infty$ 和直接馈通 $d$ 两条路径 |

## 设计启示

### 1. 选组态就是选"好的"端口

- 需要**电压放大器**（高输入阻抗、低输出阻抗）→ Series-Shunt
- 需要**电流放大器**（低输入阻抗、高输出阻抗）→ Shunt-Series
- 需要**互阻放大器**（常用于传感器电流→电压转换）→ Shunt-Shunt
- 需要**互导放大器**（OTA 级联）→ Series-Series

### 2. 负载效应不可忽略

两端口分析的核心价值在于**把反馈网络的负载效应显式地吸收进基本放大器**。如果忽略 $y_{11f}$/$z_{22f}$ 等负载项，算出来的 $a$、$T$、阻抗都会偏大，导致设计误差。MC 1553 和 723 的案例充分展示了这一步骤的必要性。

### 3. 局部反馈 vs 整体反馈

单级反馈（如 emitter degeneration、source follower）本身就是反馈电路，它可以嵌套在更大反馈环内。分析时应分别计算局部的 $T_{\text{local}}$ 和整体的 $T_{\text{overall}}$。

### 4. 源/负载未知时的建模策略

- 负载已知、源未知 → 用理想源驱动计算 $R_i'$ 和 $A'$，外接源阻抗做分压
- 源已知、负载未知 → 用理想负载（开路/短路）计算 $R_o''$ 和 $A''$，外接负载做分压

### 5. 串联稳压器即反馈放大器

723 稳压器本质是一个 Series-Shunt 反馈放大器，其负载调整率（load regulation）为：

$$
\frac{\Delta V_O}{V_O} = \left[ \frac{r_{oa}}{a V_R} \right] \Delta I_O
$$

输出电压越稳定，等效输出阻抗越低——这正是负反馈改善输出阻抗的直接体现。

## 章节关联

- **[[Ch06 - Operational Amplifiers with Single-Ended Outputs|Ch06]]**：本章分析反馈放大器所需的基本放大器（op amp、差分对等）在第 6 章详细阐述
- **[[Ch07 - Frequency Response of Integrated Circuits|Ch07]]**：基本放大器 $a$ 的频率特性决定了 $T(\omega)$ 的频率行为，是稳定性分析的前提
- **[[Ch09 - Frequency Response and Stability of Feedback Amplifiers|Ch09]]**：本章反复提到的"反馈引起的振荡问题"在第 9 章正式解决。$T$ 和 $\mathfrak{R}$ 的概念在稳定性分析中继续使用
- **[[Ch04 - Current Mirrors, Active Loads, and References|Ch04]]**：电流镜本身就是反馈电路（如 Wilson 电流源），Blackman 公式可直接用于求其输出阻抗

## 检索关键词

`负反馈` `反馈组态` `环路增益` `loop gain` `return ratio` `Blackman 阻抗公式` `Series-Shunt` `Shunt-Shunt` `Series-Series` `Shunt-Series` `两端口分析` `$y$ 参数` `$z$ 参数` `$h$ 参数` `$g$ 参数` `反馈负载效应` `发射极退化` `emitter degeneration` `source follower` `串联稳压器` `723 稳压器` `负载调整率` `增益灵敏度` `失真抑制` `super-source follower` `Wilson 电流源` `闭环增益` `闭环阻抗` `MC 1553` `理想反馈方程` `局部反馈` `直接馈通`

## Sources

- [[raw/模拟集成电路/Gray/Ch08 - Feedback]]

## See Also

- [[Ch06 - Operational Amplifiers with Single-Ended Outputs]]
- [[Ch07 - Frequency Response of Integrated Circuits]]
- [[Ch09 - Frequency Response and Stability of Feedback Amplifiers]]
- [[Ch04 - Current Mirrors, Active Loads, and References]]
