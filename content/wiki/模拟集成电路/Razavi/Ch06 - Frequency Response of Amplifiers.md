---
title: "Ch06 - 放大器频率响应"
source: "Razavi - Design of Analog CMOS Integrated Circuits (2nd Edition)"
tags: [analog-design, CMOS, frequency-response, Bode-plot]
date: 2026-08-02
---

## 本章定位

本章在直流/低频分析基础上，将 MOSFET 的四个寄生电容 $(C_{GS}, C_{GD}, C_{DB}, C_{SB})$ 纳入考量，系统研究单级和差分放大器的高频行为。频率响应决定了电路的速度——增益、带宽、功耗、噪声之间存在着深层折中关系。

## 核心概念

### Miller 定理与 Miller 近似

**Miller 定理**：若图 6.2(a) 的电路可转换为图 6.2(b)，则

$$Z_1 = \frac{Z}{1 - A_v}, \quad Z_2 = \frac{Z}{1 - A_v^{-1}}, \quad A_v = \frac{V_Y}{V_X}$$

它将一个"浮动"阻抗 $Z$ 分解为两个"接地"阻抗，使每个节点可以独立关联一个极点。

**Miller 乘法效应**：对于电容 $C_F$ 跨接在增益为 $-A$ 的反相放大器两端，等效输入电容为 $C_F(1+A)$。物理解释：输入端 $\Delta V$ 的阶跃使 $C_F$ 两端电压变化 $(1+A)\Delta V$，所需电荷量倍增。

**Miller 近似的三个局限**（使用低频增益近似高频行为时）：
1. 可能消除零点（如 CS 级的 $s_z = +g_m/C_{GD}$）
2. 可能预测额外的极点
3. 不能正确计算输出阻抗

**Miller 定理有效条件**：$Z$ 需与主信号通路并联（图 6.6），而非构成唯一的信号通路。图 6.5 的电阻分压器就是误用案例。

### 极点-节点关联

在许多电路中，每个节点对地贡献一个极点：

$$\omega_j = \frac{1}{\tau_j}, \quad \tau_j = R_{\text{eq},j} \cdot C_{\text{eq},j}$$

$R_{\text{eq},j}$ 和 $C_{\text{eq},j}$ 分别为节点 $j$ 对地的总等效电阻和总电容。

> [!warning] 局限性
> 当节点之间存在耦合元件（如图 6.11 的 $R_3$ 和 $C_3$ 造成 $X$ 与 $Y$ 节点交互），或存在电感性阻抗时，极点-节点关联不再成立。

### Bode 图基本规则

- 每个极点使幅频斜率 $-20\text{ dB/dec}$，相位 −90°
- 每个零点使幅频斜率 $+20\text{ dB/dec}$，相位 +90°
- 主导极点近似：若 $|\omega_{p1}| \ll |\omega_{p2}|$，则分母 $s$ 项系数 $\approx 1/\omega_{p1}$

### 零点与 feedthrough

右半平面零点源于输入到输出的前馈通路（如 CS 级中 $C_{GD}$ 的直通路径）。求零点的一种有效方法：令 $V_{out}(s_z)=0$，即输出可短路到地而无电流流过负载——此时通过 $C_{GD}$ 和通过 $M_1$ 的电流大小相等方向相反。

## 关键公式与结论

### 6.2 共源级 (Common-Source Stage)

Miller 近似下的极点估计：

$$\omega_{in} = \frac{1}{R_S\left[C_{GS} + (1 + g_m R_D)C_{GD}\right]}$$

$$\omega_{out} \approx \frac{1}{R_D\left(C_{DB} + C_{GD}\right)} \quad (\text{当 } A_v \gg 1)$$

直接分析得到的精确传输函数（$\lambda=0$）：

$$\frac{V_{out}}{V_{in}} = \frac{(C_{GD}s - g_m)R_D}{R_S \xi s^2 + \left[R_S C_{GS} + R_S(1+g_m R_D)C_{GD} + R_D(C_{GD}+C_{DB})\right]s + 1}$$

其中 $\xi = C_{GS}C_{GD} + C_{GS}C_{DB} + C_{GD}C_{DB}$。

> [!important] 关键结论
> - 三个电容但只有二阶系统：三个电容形成环路，只能施加两个独立初始条件
> - 右半平面零点：$s_z = +g_m/C_{GD}$，位于 $f_T$ 之外（因为 $C_{GD} < C_{GS}$）
> - 当 $R_D \to \infty$（电流源负载）且 $C_{DB}$ 或负载电容很大时，Miller 效应消失——因为高频增益已大幅下降

主导极点近似（$|\omega_{p1}| \ll |\omega_{p2}|$）：

$$\omega_{p1} \approx \frac{1}{R_S C_{GS} + R_S(1+g_m R_D)C_{GD} + R_D(C_{GD}+C_{DB})}$$

$$\omega_{p2} \approx \frac{R_S C_{GS} + R_S(1+g_m R_D)C_{GD} + R_D(C_{GD}+C_{DB})}{R_S R_D \xi}$$

输入阻抗（高频）：当 $C_{GD}$ 很大时，$1/g_{m1}$ 和 $R_D$ 并联出现在输入端——输入不再纯容性。

### 6.3 源极跟随器 (Source Follower)

传输函数（忽略沟长调制和体效应）：

$$\frac{V_{out}}{V_{in}} = \frac{C_{GS}s + g_m}{R_S C_{GS} C_L s^2 + (C_L + g_m R_S C_{GS})s + g_m}$$

> [!important] 左半平面零点
> 零点 $s_z = -g_m/C_{GS}$（约为 $-2\pi f_T$），位于左半平面。高频时通过 $C_{GS}$ 传导的信号与晶体管本征信号同极性相加。

**自举效应 (Bootstrapping)**：当 $C_L=0$、$\lambda=\gamma=0$ 时，增益为 1，$C_{GS}$ 两端电压不变——$C_{GS}$ 既不贡献极点也不贡献零点。

输入阻抗特性：
- 低频：$C_{in} = C_{GD} + \frac{g_{mb}}{g_m + g_{mb}}C_{GS}$（仅为 $C_{GS}$ 的一小部分）
- 高频（$g_{mb} \ll |C_L s|$）：$Z_{in} \approx \frac{1}{C_{GS}s} + \frac{1}{C_L s} - \frac{g_m}{C_{GS}C_L \omega^2}$
  - 包含一个**负电阻** $-\frac{g_m}{C_{GS}C_L \omega^2}$，可能导致不稳定

输出阻抗：

$$Z_{out} = \frac{R_S C_{GS} s + 1}{C_{GS} s + g_m}$$

- 低频：$Z_{out} \approx 1/g_m$
- 高频：$Z_{out} \approx R_S$（因 $C_{GS}$ 短接栅源）

由于 $Z_{out}$ 随频率增大（通常 $1/g_m < R_S$），输出阻抗表现为**感性**：

$$L = \frac{C_{GS}}{g_m}\left(R_S - \frac{1}{g_m}\right)$$

这就是"有源电感"原理，可用于带宽扩展（图 6.30b），但代价是消耗电压余度。

### 6.4 共栅级 (Common-Gate Stage)

传输函数（忽略沟长调制）：

$$\frac{V_{out}}{V_{in}} = \frac{g_m R_D}{(1 + \frac{s}{\omega_{p,in}})(1 + \frac{s}{\omega_{p,out}})}$$

$$\omega_{p,in} = \frac{1}{R_S(C_{GS}+C_{SB})}, \quad \omega_{p,out} = \frac{1}{R_D(C_{GD}+C_{DB})}$$

> [!important] 无 Miller 乘法
> CG 级不存在 Miller 乘法效应，因此具有宽带特性。但低输入阻抗（$\approx 1/g_m$）可能成为前级的负载。

当考虑沟长调制时，输入阻抗依赖于漏极负载：$Z_{in} = \frac{R_D \parallel (1/C_D s) + r_O}{1 + (g_m + g_{mb})r_O}$，使输入节点极点难以孤立计算。

若栅极有串联电阻 $R_G$，极点降低为：

$$\omega_p = \frac{1}{(R_S + R_G)C_{GS} + \left[R_S + R_G(1+g_m R_S)\right]C_{GD}}$$

### 6.5 共源共栅级 (Cascode Stage)

核心优势：利用 CG 级抑制 CS 级的 Miller 效应。

$M_1$ 的 $C_{GD1}$ 的 Miller 乘法因子由 $A$ 到 $X$ 的增益决定（而非整体增益）：

$$\text{Miller 因子} \approx 1 + \frac{g_{m1}}{g_{m2} + g_{mb2}} \approx 2 \quad (\text{两管尺寸相近})$$

三个极点估计：

$$\omega_{p,A} \approx \frac{1}{R_S\left[C_{GS1} + \left(1 + \frac{g_{m1}}{g_{m2}+g_{mb2}}\right)C_{GD1}\right]}$$

$$\omega_{p,X} \approx \frac{g_{m2} + g_{mb2}}{2C_{GD1} + C_{DB1} + C_{SB2} + C_{GS2}} \quad (\approx 2\pi f_T / 2)$$

$$\omega_{p,Y} \approx \frac{1}{R_D(C_{DB2} + C_{GD2} + C_L)}$$

一般 $\omega_{p,X}$ 远高于另外两个极点。

> [!important] 高阻抗负载时的特殊行为
> 当 $R_D$ 替换为电流源（高输出阻抗）时，$X$ 节点的阻抗升高，但传输函数几乎不受影响——高频时 $C_Y$ 将输出节点短路到地，抑制了通过 $r_{O2}$ 的 Miller 效应，$\omega_{p,X}$ 仍约为 $g_{m2}/C_X$。

### 6.6 差分对 (Differential Pair)

**无源/电流源负载**：差模半电路等价于 CS 级，直接套用式 (6.30)。共模半电路中 $P$ 点总电容（$C_{GD3}, C_{DB3}, C_{SB1}, C_{SB2}$）决定高频特性。

共模抑制比 CMRR 随频率退化：

$$\text{CMRR}(s) \approx \frac{g_m \cdot (1 + 2g_{m3}r_{O3}) \cdot r_{O3}}{1 + r_{O3}C_P s} \cdot \frac{1}{\Delta g_m}$$

含零点 $s_z \approx -2g_{m3}/C_P$ 和极点 $s_p = -1/(r_{O3}C_P)$。零点的幅度远大于极点，使得 $1/(r_{O3}C_P)$ 是 CMRR 恶化的起始频率。

> [!warning] 电压余度与 CMRR 的折中
> 为了减小 $M_3$ 消耗的余度，$W_3$ 最大化，导致 $P$ 点电容增大，恶化高频 CMRR。低电源电压下此问题更严重。

**有源电流镜负载（单端输出）**：

- 包含"镜像极点"：$\omega_{p,E} \approx g_{m3}/C_E$
- $C_E$ 包括 $C_{GS3}, C_{GS4}, C_{DB3}, C_{DB1}$ 和 $C_{GD1}, C_{GD4}$ 的 Miller 效应
- 存在左半平面零点：$s_z = -2g_{mP}/C_E$（源于"慢通路" $M_1 \to M_3 \to M_4$ 与"快通路" $M_1 \to M_2$ 的并联）

> [!important] 全差分 vs 单端
> 全差分电路无镜像极点——这是全差分拓扑优于单端的重要优势。但含折叠共源共栅的全差分电路仍可能存在镜像极点。

### 6.7 增益-带宽折中

**单极点系统**（输出负载电容主导）：

$$\text{GBW} = |A_0| \cdot \omega_p = \frac{g_{m1}}{C_L} \quad (\text{约等于单位增益带宽 } \omega_u)$$

- 示例：$g_{m1} = (100\Omega)^{-1}$，$C_L = 50\text{ fF}$，则 GBW = 31.8 GHz
- Cascode 不增大 GBW：输出阻抗和增益各升 $K$ 倍，但带宽降 $K$ 倍，乘积不变

**多极点系统**（级联 $N$ 级）：

- $N$ 级级联可将 GBW 提升约 $0.64A_0$ 倍（$N=2$ 时），但带宽本身下降
- $N$ 级相同放大器的 −3 dB 带宽：$\omega_{-3\text{dB}} = \omega_p \sqrt{\sqrt[N]{2} - 1}$
- 级联的代价：功耗倍增、多极点可能导致反馈环路不稳定

## 重要分析方法

### 1. Extra Element Theorem (EET)

Middlebrook 定理：当向已知传输函数 $H(s)$ 的电路中添加额外元件 $Z_1$，新传输函数为：

$$G(s) = H(s) \cdot \frac{1 + \frac{Z_{out,0}}{Z_1}}{1 + \frac{Z_{in,0}}{Z_1}}$$

其中 $Z_{out,0}$ 是在维持 $V_{out}=0$ 条件下 $Z_1$ 端口的电压-电流比（$V_{in}$ 不为零），$Z_{in,0}$ 是 $V_{in}=0$ 时 $Z_1$ 端口的阻抗。

**应用策略**：从无电容的低频增益开始，逐个添加电容并用校正因子修正——每次只处理一个电抗元件。

### 2. Zero-Value Time Constant (ZVTC) Method

**核心思想**：主导极点（若存在）由所有零值时间常数之和的倒数给出：

$$\omega_{p1} \approx \frac{1}{\sum_{j=1}^{n} R_j C_j}$$

其中 $R_j$ 是电容 $C_j$ 两端看到的等效电阻，此时**所有其他电容设为零**（开路），且**输入设为零**。

> [!warning] 注意
> - 该方法给出的是 $\sum \tau_j$ 的倒数，而非每个 $\tau_j$ 单独对应一个极点
> - 忽略零点的影响
> - CS 和 CG 级设定 $V_{in}=0$ 后拓扑相同，ZVTC 给出相同的带宽估计——这不与 CG 无 Miller 效应矛盾（CG 级中 $R_G$ 应避免，而 CS 的 $R_G$ 是前级输出电阻，不可避免）

**应用示例**——电阻退化 CS 级的 −3 dB 带宽估算：

$$\tau_{CGS} = \frac{R_S + R_G}{1 + g_m R_S} C_{GS}$$

$$\tau_{CGD} = \left[R_D + R_G\left(1 + \frac{g_m R_D}{1 + g_m R_S}\right)\right] C_{GD}$$

$$\tau_{CL} = R_D C_L$$

$$\omega_{-3\text{dB}} \approx \frac{1}{\tau_{CGS} + \tau_{CGD} + \tau_{CL}}$$

## 设计启示

1. **Miller 效应是最主要的带宽限制因素**：CS 级中 $C_{GD}$ 被放大约 $g_m R_D$ 倍；Cascode 结构将其抑制到约 2 倍；CG 级完全消除。
2. **源极跟随器存在自举效应**：有效输入电容远小于 $C_{GS}$，但驱动容性负载时产生负阻，可能引发振荡。
3. **差分对有源负载存在"镜像极点"**：全差分结构可避免此问题。
4. **GBW 由 $g_m/C_L$ 决定**：与输出电压摆幅无关，Cascode 不能提升 GBW。
5. **多级级联提升 GBW 但降低带宽**：且可能引发反馈稳定性问题。
6. **ZVTC 方法提供快速带宽估算**：无需推导完整传输函数，设计初期即可判断各电容的相对贡献。

## 章节关联

- **Ch03 (Single-Stage Amplifiers)** — 本章直接建立在 Ch03 的低频增益分析基础上，将寄生电容加入模型
- **Ch08 (Feedback)** — 反馈放大器的频率响应分析需要本章的极点/零点概念
- **Ch10 (Stability and Frequency Compensation)** — 稳定性分析和频率补偿的核心工具来自本章的传输函数建模和 Bode 图方法；$C_{GD}$ 的 right half-plane zero 是补偿设计中的关键挑战

## 检索关键词

`Miller effect`, `Miller theorem`, `pole-node association`, `dominant pole`, `Bode plot`, `common-source frequency response`, `source follower input impedance`, `bootstrap effect`, `cascode bandwidth`, `mirror pole`, `CMRR frequency`, `gain-bandwidth product`, `GBW`, `extra element theorem`, `zero-value time constant`, `right half-plane zero`, `feedforward`, `active inductor`

## See Also

- [[Ch03 - Single-Stage Amplifiers|Ch03 - 单级放大器]]
- [[Ch08 - Feedback|Ch08 - 反馈]]
- [[Ch10 - Stability and Frequency Compensation|Ch10 - 稳定性与频率补偿]]

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd|Razavi - Design of Analog CMOS Integrated Circuits (2nd Edition), Chapter 6]]
