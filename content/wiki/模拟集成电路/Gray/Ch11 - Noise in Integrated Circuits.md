---
title: "Gray Ch11：集成电路中的噪声"
source: "Analysis and Design of Analog Integrated Circuits, 5th Edition, Chapter 11"
tags:
  - analog-design
  - analog-IC
  - noise
  - thermal-noise
  - flicker-noise
  - low-noise-design
---

# Gray Ch11：集成电路中的噪声

Sources: [[raw/模拟集成电路/Gray/Ch11 - Noise in Integrated Circuits]]

## 本章定位

本章系统地论述了集成电路中**电噪声**的物理来源、器件级建模、电路级分析方法以及低噪声设计指南。噪声代表信号能被放大而不显著劣化的**下限**，也决定了放大器增益的**实际上限**——增益无限增大会使前级放大的噪声在后级将器件推出有源区。本章从基础噪声物理（散粒、热、闪烁、爆裂、雪崩）出发，构建双极和 MOS 晶体管的含噪声小信号模型，推导等效输入噪声发生器，并将其应用于反馈放大器、差分对和运放的噪声分析，最后引入噪声带宽、噪声系数与噪声温度等工程度量指标。

> **与 Razavi Ch7 的区别**：Gray 更侧重噪声的**双端口等效表示**和**反馈对噪声的影响**，对运放 (NE5234) 做了详尽的定量分析；Razavi 更侧重噪声在放大器设计中单级/差分级的直观理解。

## 核心概念

### 噪声的加法规则

多个**独立**噪声源：
- 串联噪声电压源：均值平方相加：$\overline{v_T^2} = \overline{v_1^2} + \overline{v_2^2}$
- 并联噪声电流源：均值平方相加：$\overline{i_T^2} = \overline{i_1^2} + \overline{i_2^2}$

> 独立噪声源乘积的均值为零 $\overline{v_1(t)v_2(t)} = 0$。所有本章的器件噪声源（除11.16式的栅极耦合噪声外）因出自不同的物理机制，可视为相互独立。

### 噪声电路计算的核心思路

在带宽 $\Delta f$ 内的噪声电流可**近似为一个同有效值的正弦电流源**。由此将随机噪声分析转化为常规正弦稳态电路分析——每个噪声源分别用正弦源代替，计算各自的输出贡献，然后将各输出贡献的**均值平方相加**得到总输出噪声。这一方法的正确性建立在噪声源的独立性假设之上。

## 关键公式与结论

### 一、噪声源的物理类型

#### 1. 散粒噪声 (Shot Noise)

> 总是与直流电流流动相关联，存在于二极管、BJT、MOSFET 中。

$$
\overline{i^2} = 2q I_D \Delta f \qquad \text{(11.2)}
$$

- $q = 1.6 \times 10^{-19}\ \mathrm{C}$
- 白噪声谱（频谱平坦直到频率≈ $1/\tau$，$\tau$ 为载流子穿越耗尽区时间）
- **幅值服从高斯分布**
- **与温度无关**

电路表示：在二极管/晶体管小信号模型中并联一个值为 $2qI_D\Delta f$ 的均值平方电流源。

#### 2. 热噪声 (Thermal Noise)

> 由电子随机热运动产生，与是否有直流电流无关。

串联电压源形式：

$$
\overline{v^2} = 4kTR \Delta f \qquad \text{(11.4)}
$$

并联电流源形式：

$$
\overline{i^2} = \frac{4kT}{R} \Delta f = \frac{4kT}{R} \Delta f \qquad \text{(11.5)}
$$

- **与绝对温度 $T$ 成正比**（$T \to 0$ 时热噪声消失）
- 白噪声谱，频率上限约 $10^{13}$ Hz
- **幅值服从高斯分布**
- **与散粒噪声在电路中不可区分**（均为平坦频谱 + 高斯分布）

> [!tip] 记忆用数值
> 室温 (300 K) 下：
> - 1 kΩ 电阻的热噪声谱密度 $\overline{v^2}/\Delta f \simeq 16 \times 10^{-18}\ \mathrm{V^2/Hz}$
> - rms 形式：$v \simeq 4\ \mathrm{nV/\sqrt{Hz}}$
> - 1 kΩ 电阻的热噪声电流 = 50 μA 直流的散粒噪声

> [!warning] 重要
> 小信号模型中的 $r_\pi$、$r_o$ 是虚构电阻，**不产生热噪声**。只有物理电阻（$r_b$、$r_c$、$r_s$ 等）才有热噪声。

#### 3. 闪烁噪声 (Flicker Noise / $1/f$ Noise)

> 由晶格缺陷和污染引起的陷阱对载流子的随机捕获与释放造成，存在于所有有源器件及碳质电阻中。

$$
\overline{i^2} = K_1 \frac{I^a}{f^b} \Delta f \qquad \text{(11.7)}
$$

- $a$: 0.5~2, $b \approx 1$
- **只在有直流电流时存在**——碳质电阻若不通直流则不产生闪烁噪声
- $K_1$ 在同一晶圆上也可能显著变化（与污染和晶体缺陷有关）
- **幅值分布通常非高斯**

> [!warning] 外部元件选型提示
> - 需要直流的低噪声低频电路外部电阻：**选用金属膜电阻**（无闪烁噪声），而非碳质电阻
> - 不需直流的偏置电阻：碳质电阻可用

#### 4. 爆裂噪声 (Burst / Popcorn Noise)

$$
\overline{i^2} = K_2 \frac{I^c}{1 + (f/f_c)^2} \Delta f \qquad \text{(11.8)}
$$

- 频谱在 $f_c$ 处呈现"驼峰"，高频段以 $1/f^2$ 衰减
- 与重金属离子污染有关（掺金器件爆裂噪声极高）
- 多时间常数时出现多峰叠加于 $1/f$ 噪声之上

#### 5. 雪崩噪声 (Avalanche Noise)

- 由PN结反向击穿（齐纳/雪崩）产生
- 齐纳二极管的典型测量值：$\overline{v^2}/\Delta f \simeq 10^{-14}\ \mathrm{V^2/Hz}$ @ $I_Z = 0.5\ \mathrm{mA}$，等效于600 kΩ的热噪声
- **低噪声电路应避免使用齐纳二极管**

### 二、器件噪声模型

#### 双极晶体管 (BJT)

![fig11.13]

三个独立噪声源：

$$
\boxed{\overline{v_b^2} = 4kT r_b \Delta f} \qquad \text{(11.11)}
$$

$$
\boxed{\overline{i_c^2} = 2q I_C \Delta f} \qquad \text{(11.12)}
$$

$$
\boxed{\overline{i_b^2} = \underbrace{2q I_B \Delta f}_{\text{散粒噪声}} + \underbrace{K_1 \frac{I_B^a}{f} \Delta f}_{\text{闪烁噪声}} + \underbrace{K_2 \frac{I_B^c}{1+(f/f_c)^2} \Delta f}_{\text{爆裂噪声}}} \qquad \text{(11.13)}
$$

- 集电极电流 $I_C$ 呈现**全散粒噪声**
- 基极电流 $I_B$ 呈现**全散粒噪声** + 闪烁噪声 + 爆裂噪声
- 基极电阻 $r_b$ 呈现热噪声
- 集电极串联电阻 $r_c$ 的热噪声因与高阻节点串联通常可忽略
- $r_\pi$、$r_o$ **无热噪声**——它们是虚构建模电阻
- 闪烁噪声拐角频率 $f_a$：散粒噪声与闪烁噪声渐近线交点。精心处理可达 100 Hz，差的可到 10 MHz

#### MOS 晶体管

![fig11.15]

漏-源噪声电流（热噪声 + 闪烁噪声合并）：

$$
\overline{i_d^2} = \underbrace{4kT\left(\frac{2}{3}g_m\right) \Delta f}_{\text{沟道热噪声}} + \underbrace{K \frac{I_D^a}{f} \Delta f}_{\text{闪烁噪声}} \qquad \text{(11.14)}
$$

- 与 BJT 不同，MOS 管的沟道材料是**电阻性的**，因此主导噪声是**热噪声**而非散粒噪声
- $g_m$ 是工作点跨导
- **短沟道 ($L < 1\ \mu m$)**：热噪声可达上述的 2~5 倍（热电子效应）

栅极噪声：

$$
\boxed{\overline{i_g^2} = 2q I_G \Delta f} \qquad \text{(11.15)}
$$

- $I_G$ 通常 < $10^{-15}$ A，故此项通常极小

高频栅极耦合噪声（长沟道饱和区）：

$$
\overline{i_g^2} = 4kT \delta g_g \Delta f, \quad g_g = \frac{\omega^2 C_{gs}^2}{5g_m} \qquad \text{(11.16)}
$$

- 该项与 (11.14) 的沟道热噪声**存在相关性**（相关系数 0.39），因为两者都源于沟道热涨落
- 短沟道下可能更大

#### 电阻

- **所有物理电阻**均产生热噪声（11.4 / 11.5）
- 碳质电阻 = 热噪声 + 闪烁噪声（有直流时）
- 金属膜电阻 = 仅热噪声

#### 电容与电感

- **理想**电容和电感**无噪声源**
- 实际器件的寄生电阻按 (11.4)/(11.5) 产生热噪声

### 三、等效输入噪声发生器

> [!abstract] 核心思想
> 将电路内部所有噪声源折算到输入端，用**一个等效输入噪声电压源 $\overline{v_i^2}$** 和**一个等效输入噪声电流源 $\overline{i_i^2}$** 代表，使得折算后的输出噪声与原电路相同。这两个发生器可用于**任意源阻抗**（需考虑相关性），但多数实际电路中相关性可忽略。

![fig11.22]

#### BJT 等效输入噪声

**等效输入噪声电压**（假设 $r_b \ll r_\pi$）：

$$
\boxed{\frac{\overline{v_i^2}}{\Delta f} = 4kT \underbrace{\left(r_b + \frac{1}{2g_m}\right)}_{R_{eq}}} \qquad \text{(11.50), (11.52)}
$$

- $R_{eq}$ = 等效输入噪声电阻
- $r_b$ 项：基极电阻的物理热噪声
- $1/2g_m$ 项：集电极散粒噪声向输入端折算

**等效输入噪声电流**（忽略爆裂噪声）：

$$
\boxed{\frac{\overline{i_i^2}}{\Delta f} = 2q I_{eq}, \quad I_{eq} = I_B + \frac{K_1 I_B^a}{2q f} + \frac{I_C}{|\beta(j\omega)|^2}} \qquad \text{(11.57)-(11.59)}
$$

- 低频段：$I_{eq} \approx I_B$（集电极噪声折算项 $I_C/\beta_0^2$ 忽略不计）
- 此时 $\overline{v_i^2}$ 与 $\overline{i_i^2}$ **完全独立**
- 高频段：$|\beta(j\omega)|$ 下降导致 $\overline{i_i^2}$ 以 $f^2$ 上升，且与 $\overline{v_i^2}$ 产生相关性（两者均含 $i_c^2$ 贡献）

**高频噪声带宽 $f_b$**（等效输入噪声电流高频上升与中频渐近线交点）：

$$
f_b = \frac{f_T}{\beta_0} \qquad \text{(11.63)}
$$

> [!tip] BJT噪声设计矛盾
> - 低源阻抗 → $\overline{v_i^2}$ 主导 → 需要**大 $I_C$**（减 $1/2g_m$）+ 小 $r_b$
> - 高源阻抗 → $\overline{i_i^2}$ 主导 → 需要**小 $I_C$** + 高 $\beta$
>
> 两者**相互矛盾**，最优偏置需折中。

#### MOS 等效输入噪声

**等效输入噪声电压**：

$$
\boxed{\frac{\overline{v_i^2}}{\Delta f} = 4kT R_{eq} + \frac{K_f}{W L C_{ox} f}} \qquad \text{(11.68a, 11.68b)}
$$

其中 $R_{eq} = \dfrac{2}{3g_m}$（高于闪烁噪声拐角频率时）

- 与 BJT 不同，MOS 的 $\overline{v_i^2}$ **包含闪烁噪声**
- 闪烁噪声分量通常可写为 $\dfrac{K_f}{W L C_{ox} f}$，与偏置电流近似无关
- 典型 $K_f = 3 \times 10^{-24}\ \mathrm{V^2\text{-}F}$ (即 $3 \times 10^{-12}\ \mathrm{V^2\text{-}pF}$)
- 增大 $(W \cdot L)$ 面积通过**平均效应**降低闪烁噪声

**等效输入噪声电流**（忽略栅耦合噪声 (11.16)）：

$$
\boxed{\frac{\overline{i_i^2}}{\Delta f} = 2q I_G + \frac{4kT\left(\frac{2}{3}g_m\right) + \frac{K I_D^a}{f}}{|A_I|^2}} \qquad \text{(11.72)}
$$

- 低频段由 $I_G$ 决定，通常 < $10^{-15}$ A → **远优于 BJT**
- 因此 MOS 在高源阻抗下噪声性能**优于** BJT

> [!summary] BJT vs MOS 噪声对比
> | 条件 | BJT | MOS |
> |------|-----|-----|
> | $\overline{v_i^2}$ (低源阻抗) | **更优**（$g_m$ 更大） | 较差（$R_{eq} \propto 1/g_m$） |
> | $\overline{i_i^2}$ (高源阻抗) | 较差（$I_{eq} \approx I_B$） | **更优**（$I_G$ 极小） |
> | 闪烁噪声 | 集总在 $\overline{i_i^2}$ | 集总在 $\overline{v_i^2}$ |
> | 高频恶化 | $i_i^2$ 以 $f^2$ 上升 | 栅耦合噪声 |

### 四、总等效输入噪声（含源电阻）

当源电阻为 $R_S$ 时（忽略相关性）：

$$
\boxed{\overline{v_{iN}^2} = \overline{v_s^2} + \overline{v_i^2} + \overline{i_i^2} R_S^2} \qquad \text{(11.65)}
$$

其中 $\overline{v_s^2} = 4kT R_S \Delta f$ 为源电阻热噪声。

## 噪声分析技术

### 1. 两级联等效输入噪声发生器法

步骤：
1. 分别求取 BJT/MOS 的 $\overline{v_i^2}$ 和 $\overline{i_i^2}$
2. 短接输入端，令输出噪声相等以求 $\overline{v_i^2}$
3. 开路输入端，令输出噪声相等以求 $\overline{i_i^2}$
4. 用 (11.65) 计算含 $R_S$ 的总等效输入噪声

### 2. 反馈对噪声的影响

> [!important] 核心结论
> **理想反馈不改变等效输入噪声发生器**。反馈降低增益的同时等比例降低信号和噪声，信噪比不变。

**实际反馈引入的额外噪声来源**：

| 反馈类型 | 额外噪声贡献 |
|---------|------------|
| 串联反馈（输入） | 反馈电阻的**热噪声**叠加到 $\overline{v_i^2}$：$\overline{v_i^2} = \overline{v_{ia}^2} + 4kT(R_E \parallel R_F)\Delta f$ |
| 并联反馈（输入） | 反馈电阻的**热噪声**叠加到 $\overline{i_i^2}$：$\overline{i_i^2} = \overline{i_{ia}^2} + 4kT/R_F \Delta f$ |
| 输出端 | 噪声出现在输出节点并联，不影响等效输入噪声发生器 |

> 计算反馈电阻噪声贡献的方法：用第 8 章双端口加载法确定等效输入串联/并联电阻，再用这些电阻计算热噪声贡献。

### 3. 差分对噪声表示

差分对需用**两组**等效输入发生器（每输入端各一组 $v_i^2$, $i_i^2$）表示：

![fig11.36b]

简化（高CMRR时）：

$$
\boxed{\overline{v_{dp}^2} = 2\overline{v_i^2}, \quad \overline{i_{dp}^2} = \overline{i_i^2}} \qquad \text{(11.102)}
$$

- 等效输入噪声电压为单管的两倍（+3 dB）
- 等效输入噪声电流与单管相同
- 尾电流源 $I_{EE}$ 的噪声在完全平衡时产生共模信号，**不影响差分输出**

### 4. 运放噪声分析（以 NE5234 为例）

运放需视为**三端口**器件（差分输入）：

![fig11.39]

NE5234 关键分析结果：
- 等效输入噪声电阻 $R_{eq} \approx 22\ \mathrm{k\Omega}$
- **有源负载是主要噪声贡献者**（约 13 kΩ），远大于输入差分对自身的贡献（约 9 kΩ）
- 源电阻 1 kΩ 时，运放自身噪声功率是源电阻噪声的 22 倍

CMOS 输入级（图 11.40）关键设计公式：

**闪烁噪声**（folded 到输入端）：

$$
\overline{v_{eqT}^2} = 2\overline{v_{eq1}^2} \left[1 + \left(\frac{K_p}{K_n}\right)\left(\frac{L_1}{L_3}\right)^2\right] \qquad \text{(11.121)}
$$

**热噪声**（folded 到输入端）：

$$
\overline{v_{eqT}^2} = 2 \cdot 4kT\left(\frac{2}{3g_{m1}}\right) \left[1 + \frac{g_{m3}}{g_{m1}}\right] \qquad \text{(11.123)}
$$

> [!tip] CMOS运放低噪声设计策略
> - 使输入管 $g_m$ 远大于负载管 $g_m$
> - 负载管沟长 $L_3, L_4$ 取输入管 $L_1, L_2$ 的 2 倍以上，有效抑制负载闪烁噪声贡献
> - 负载管宽度 $W$ 对闪烁噪声无影响

### 5. 噪声带宽 (Noise Bandwidth)

当等效输入噪声谱为**白噪声**时，总输出噪声计算可简化为：

$$
\overline{v_{oT}^2} = S_{i0} \int_0^\infty |A_v(jf)|^2 df = S_{i0} A_{v0}^2 f_N \qquad \text{(11.125)}
$$

其中**等效噪声带宽**：

$$
\boxed{f_N = \frac{1}{A_{v0}^2} \int_0^\infty |A_v(jf)|^2 df} \qquad \text{(11.126)}
$$

- **单极点系统**：$f_N = \frac{\pi}{2} f_1 \approx 1.57 f_1$（$f_1$ 为 −3 dB 频率）
- **两极点在 45° 的复极点系统**：$f_N$ 仅比 −3 dB 带宽大约 11%
- 频率滚降越陡峭，$f_N$ 越接近 −3 dB 带宽

### 6. 闪烁噪声积分的低频发散问题

$1/f$ 噪声的积分 $\int_{f_L}^{f_H} \frac{1}{f} df = \ln\frac{f_H}{f_L}$，当 $f_L \to 0$ 时发散（理论上无限功率）。

工程处理：**积分下限由观测周期决定**。
- 1 次/天 → $f_L \approx 1.16 \times 10^{-5}$ Hz
- 1 次/年 → $f_L \approx 3.2 \times 10^{-8}$ Hz
- 因 $\ln$ 函数收敛极慢，$f_L$ 的进一步降低对结果影响有限

### 7. 噪声系数 (Noise Figure) 与噪声温度

**噪声系数**定义：

$$
F = \frac{\text{总输出噪声功率}}{\text{仅由源电阻产生的输出噪声功率}} \qquad \text{(11.142)}
$$

用等效输入噪声发生器表示（忽略相关性）：

$$
\boxed{F = 1 + \frac{\overline{v_i^2} + \overline{i_i^2}R_S^2}{4kTR_S \Delta f}} \qquad \text{(11.149)}
$$

$F$ 在 $R_S$ 变化时存在最小值。最优源电阻：

$$
\boxed{R_{S(opt)} = \sqrt{\frac{\overline{v_i^2}/\Delta f}{\overline{i_i^2}/\Delta f}}} \qquad \text{(11.150)}
$$

**BJT 噪声系数算例** ($I_C = 1\ \mathrm{mA}, \beta_F = 100, r_b = 50\ \Omega$)：$R_{S(opt)} = 572\ \Omega$, $F_{opt} = 0.9\ \mathrm{dB}$。

**MOS 噪声系数**：$R_{S(opt)} \to \infty$, $F_{opt} \to 0\ \mathrm{dB}$——MOS 高源阻抗下噪声性能极佳。

**噪声温度**：

$$
T_n = T(F - 1) \qquad \text{(11.148)}
$$

- $F = 2\ (3\ \mathrm{dB})$ → $T_n = 290\ \mathrm{K}$
- $F = 1.1\ (0.4\ \mathrm{dB})$ → $T_n = 29\ \mathrm{K}$

主要用于极低噪声放大器 ($F$ 接近 1 时) 提供更精细的度量。

## 低噪声设计

### 器件级

| 策略 | 效果 |
|------|------|
| BJT: 增大 $I_C$ | 降低 $\overline{v_i^2}$（$R_{eq}$ 中 $1/2g_m$ 项减小） |
| BJT: 减小 $I_C$ | 降低 $\overline{i_i^2}$（$I_{eq}$ 减小） |
| BJT: 高 $\beta$ | 降低 $\overline{i_i^2}$（$I_{eq}$ 中 $I_B$ 减小） |
| BJT: 小 $r_b$ | 降低 $\overline{v_i^2}$ |
| MOS: 增大 $W \cdot L$ | 降低闪烁噪声（面积平均效应） |
| MOS: 增大 $g_m$ | 降低 $\overline{v_i^2}$ |
| 避免齐纳二极管 | 消除雪崩噪声 |
| 碳质电阻不通直流 | 消除碳质电阻闪烁噪声 |

### 电路结构级

| 策略 | 效果 |
|------|------|
| 输入级用 CE/CS 而非 CB/CG | CB 电流增益 ≈ 1，后级噪声完全折回输入端 |
| 有源负载使 $g_{m,\text{load}} \ll g_{m,\text{input}}$ | 抑制负载噪声向输入端折算 |
| 有源负载用长沟道器件 | 抑制负载闪烁噪声 |
| 反馈电阻用金属膜电阻 | 无闪烁噪声 |
| 变压器耦合到最优源阻抗 | 同时实现最低噪声系数和功率匹配 |

### 设计矛盾

> [!warning] BJT 偏置电流的基本矛盾
> - 低 $R_S$：噪声以 $\overline{v_i^2}$ 为主 → 需要**大** $I_C$
> - 高 $R_S$：噪声以 $\overline{i_i^2}$ 为主 → 需要**小** $I_C$
>
> 必须根据实际源阻抗选择最优偏置点。

### 常见级联输入级噪声特点

| 组态 | 等效输入噪声 | 注意事项 |
|------|------------|---------|
| 共射 (CE) / 共源 (CS) | 基准 | 标准低噪声输入级 |
| 共基 (CB) | 与 CE 相同 | 电流增益 ≈ 1，后级和负载噪声全折回输入端 → **不适合作低噪声输入级** |
| 射随器 | 与 CE 相同 | 电压增益 = 1，后级 $\overline{v_i^2}$ 直接折回输入 |
| 差分对 | $\overline{v_{dp}^2} = 2\overline{v_i^2}$, $\overline{i_{dp}^2} = \overline{i_i^2}$ | 噪声电压 +3 dB 于单管 |

## 设计启示

1. **噪声和增益的天花板关系**：放大器增益被噪声制约——增益过大时放大的前级噪声会将后级推出线性区。这构成了增益设计的实际上限。

2. **BJT还是MOS**：低源阻抗选BJT（$\overline{v_i^2}$ 更低），高源阻抗选MOS（$\overline{i_i^2}$ 更低）、且MOS闪烁噪声可以通过增大面积抑制。

3. **有源负载是运放噪声的主要瓶颈**：NE5234 实例表明有源负载贡献了运放输入噪声的大头（13 kΩ vs 输入对 9 kΩ），设计中必须让负载管 $g_m$ 远小于输入管 $g_m$。

4. **反馈降增益不改善SNR**：理想反馈不改变等效输入噪声发生器，信号和噪声等比例衰减。实际反馈因电阻热噪声会恶化噪声性能。

5. **$1/f$ 噪声积分的低频极限**由观测周期决定——这不是物理问题，而是工程测量问题。

6. **外部元件选材直接影响噪声**：金属膜电阻（无闪烁噪声）vs 碳质电阻（有直流时产生闪烁噪声）；齐纳二极管需避免。

## 章节关联

- [[wiki/模拟集成电路/Razavi/Ch07 - Noise]]：互补阅读——Razavi 侧重单级放大器噪声的直观设计方法与 CS/CG/CD 组态对比
- [[wiki/抖动与相位噪声/总览 MOC]]：噪声的时域表现（抖动）与频域表现（相位噪声）的系统视角
- [[wiki/随机过程/白噪声与线性系统]]：白噪声通过线性系统的数学基础——与本章噪声带宽计算直接相关

## 检索关键词

`噪声` `散粒噪声` `shot noise` `热噪声` `thermal noise` `闪烁噪声` `flicker noise` `1/f噪声` `爆裂噪声` `popcorn noise` `雪崩噪声` `等效输入噪声` `equivalent input noise` `等效输入噪声电阻` `噪声系数` `noise figure` `噪声温度` `噪声带宽` `noise bandwidth` `NE5234` `差分对噪声` `BJT噪声模型` `MOS噪声模型` `低噪声设计` `low-noise design` `最小可检测信号` `MDS`
