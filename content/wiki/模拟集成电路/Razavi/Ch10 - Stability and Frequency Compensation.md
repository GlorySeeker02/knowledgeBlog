---
title: Ch10 - Stability and Frequency Compensation
source: "Razavi, Design of Analog CMOS Integrated Circuits, 2nd Ed., Ch10"
tags: [analog-design, CMOS, stability, frequency-compensation, Nyquist]
updated: 2026-08-02
---

# Ch10 — Stability and Frequency Compensation

## 本章定位

本章系统讲解负反馈放大器的**稳定性分析与频率补偿**技术。第 9 章对运放的分析表明，多级电路存在多个极点，负反馈闭环后可能振荡。本章回答三个核心问题：(1) 如何判断一个反馈系统是否稳定？(2) 稳定裕度需要多大？(3) 不稳定时如何补偿（即修改开环传递函数）？

从 10.1–10.3（稳定性判据与相位裕度）到 10.4–10.7（各种补偿技术），再到 10.8（Nyquist 判据的理论基础），本章是连接运放设计（Ch9）与实际闭环应用的关键桥梁。

---

## 核心概念

### 1. 振荡条件：Barkhausen 判据 (10.1)

闭环传递函数 $\frac{Y}{X} = \frac{H(s)}{1+\beta H(s)}$。当 $\beta H(j\omega_1) = -1$ 时，分母为零，增益无穷大——电路在 $\omega_1$ 处振荡。这等价于：

$$
|\beta H(j\omega_1)| = 1 \quad\text{且}\quad \angle\beta H(j\omega_1) = -180^\circ
$$

>[!important] Barkhausen 判据
>负反馈系统在 $\omega_1$ 处振荡的条件：环路增益的**大小** = 1 且**相位** = $-180^\circ$。注意负反馈自身已贡献 $180^\circ$ 相移，因此环路总相移达到 $360^\circ$ 时反馈变为正反馈。

核心要点：
- 判据只涉及**环路增益** $\beta H$，与输入输出位置无关。
- **单极点系统无条件稳定**，因为一个极点最多贡献 $-90^\circ$ 相移，永远达不到 $-180^\circ$。
- **两极点系统**：相位渐近逼近 $-180^\circ$，但只在 $\omega \to \infty$ 时达到 $180^\circ$，此时 $|H| \to 0$，所以仍然稳定——但接近振荡时阶跃响应会出现振铃。
- **三极点及以上的系统**：可能不满足稳定条件，需要补偿。

### 2. 增益交越频率 vs 相位交越频率

| 术语 | 定义 | 符号 |
|------|------|------|
| Gain Crossover (GX) | $|\beta H| = 1$ (0 dB) 的频率 | $\omega_{\text{GX}}$ |
| Phase Crossover (PX) | $\angle\beta H = -180^\circ$ 的频率 | $\omega_{\text{PX}}$ |

>[!info] 稳定性的直观判据
>- **稳定**：GX 在 PX 之前（即 $|\beta H|$ 先降到 1，相位才到 $-180^\circ$）。
>- **不稳定**：PX 在 GX 之前（相位已到 $-180^\circ$，增益仍大于 1）。
>
>增益交越频率同时也是环路增益的**单位增益带宽**。

### 3. 反馈深度与稳定性 (10.1 例 10.1)

>[!tip] 关键结论
>**反馈越弱（$\beta$ 越小），系统越稳定**。降低 $\beta$ 使 $20\log|\beta H|$ 整体下移，GX 左移而 PX 不变 → 稳定性改善。
>
>因此 **最坏情况是 $\beta=1$（单位增益反馈）**。我们通常按 $\beta H = H$ 来分析稳定性。

### 4. 根轨迹 (Root Locus) 概念 (10.2)

闭环极点随环路增益变化的轨迹，在复平面上直观展示系统何时接近振荡：

- **单极点系统**：闭环极点沿负实轴从 $-\omega_0$ 向 $-\infty$ 移动，始终在 LHP → 无条件稳定。
- **两极点系统**：极点从两个实轴位置出发，相向移动，相遇后变成复共轭对，轨迹始终在 LHP → 不会振荡（但可能严重振铃）。
- **三极点系统**：极点可能穿过 $j\omega$ 轴进入 RHP → 可能振荡。

根轨迹比代数求解更直观，但高阶系统代数复杂——所以 Bode 图在工程上更实用。

---

## 稳定性判据

### Bode 法：相位裕度 (10.3)

**定义**：

$$
\text{PM} = 180^\circ + \angle\beta H(\omega = \omega_{\text{GX}})
$$

即增益交越频率处，相位离 $-180^\circ$ 还有多少度。

**PM 与闭环响应的关系**：

| PM | 闭环频率响应 | 阶跃响应 |
|----|------------|---------|
| $45^\circ$ | $|Y/X| \approx 1.3/\beta$（30% 频率峰化） | 明显振铃，欠阻尼 |
| $60^\circ$ | $|Y/X| \approx 1/\beta$（几乎无峰化） | 轻微振铃，快速建立 |
| $90^\circ$ | 无峰化 | 无振铃，但建立较慢 |

>[!important] 设计目标
>**PM = $60^\circ$ 是最优选择**——兼顾稳定性与速度。

### Nyquist 法 (10.8)

>[!warning] Bode 法的盲区
>Bode 图只考虑 $s = j\omega$（纯正弦），无法判断**增长型正弦**（$s = \sigma_1 + j\omega_1$, $\sigma_1 > 0$）是否使 $1+\beta H(s) = 0$。一个 Bode 图显示"不稳定"的系统未必会振荡——Nyquist 判据填补了这一盲区。

**Nyquist 判据**：
闭环系统 $\dfrac{H(s)}{1+\beta H(s)}$ 稳定的条件是——当 $s$ 绕临界区（$j\omega$ 轴 + RHP）顺时针一周时，$\beta H(s)$ 的极坐标图**不顺时针包围**点 $(-1, 0)$。

**构造 Nyquist 图的核心方法**：
- 从 $s = j\omega$ 的 $\omega = 0 \to +\infty$ 画曲线，再将这部分关于实轴反射得到完整图形。
- 包围圈数 $N$（顺时针为正）与 $1+\beta H(s)$ 在临界区的零点数 $Z$ 和极点数 $P$ 满足 **Cauchy 辐角原理**：$N = Z - P$。
- 通常开环系统稳定（$P = 0$），所以 $N = Z$——包围 $(-1, 0)$ 就有 RHP 闭环极点，不稳定。

**Bode vs Nyquist 对照**：

| 系统类型 | Bode 判断 | Nyquist 判断 |
|---------|----------|-------------|
| 单极点 | 无条件稳定 | 不包围 $(-1, 0)$，稳定 |
| 两极点 | 永远稳定 | 不包围 $(-1, 0)$，稳定 |
| 三极点 | $|H|$ 在 PX 处大小决定 | 是否包围 $(-1, 0)$ 取决于交点 $Q$ 的位置 |
| 两个积分器 $A_0/s^2$ | 相位恒为 $-180^\circ$，看似振荡 | $\beta H$ 穿过 $(-1,0)$，确实有 $j\omega$ 轴极点 |

**原点极点的处理**：若 $H(s)$ 含 $s=0$ 的极点，$s$ 绕原点走一个半径为 $\epsilon$ 的小圆绕过去——对应 Nyquist 图上产生一段**大半径弧线**。

**Nyquist 判据的直接推论**：如果 $\angle\beta H$ 跨越 $-180^\circ$ **偶数次**且每次 $|\beta H| > 1$，系统可能仍是稳定的；跨越**奇数次**且 $|\beta H| > 1$ 则不稳定。

---

## 关键公式与结论

### 稳定性基本关系

| 公式 | 含义 |
|------|------|
| $\beta H(j\omega_1) = -1$ | Barkhausen 振荡条件 |
| $\text{PM} = 180^\circ + \angle\beta H(\omega_{\text{GX}})$ | 相位裕度定义 |
| $\text{GM} = -20\log|\beta H(\omega_{\text{PX}})|$ | 增益裕度定义（dB） |
| $\omega_{\text{UG BW}} \leq \omega_{p2}$ (for PM $> 45^\circ$) | 单位增益带宽不超过第二极点 |
| 闭环带宽 $\approx A_0\omega_{p,\text{dom}}$ | 主管极点决定的增益带宽积 |

### 基本极点补偿 (10.4)

单纯通过增大**负载电容**来压低主管极点：

$$
\omega'_{p,\text{out}} = \left[\text{所需降幅}\right] \times \omega_{p,\text{out}}
$$

代价：**带宽与补偿量成反比**。提高 $R_{\text{out}}$ **不能**补偿（只提高低频增益）。

### Miller 补偿 (10.5)

在两极运放的第一级输出与第二级输出之间跨接电容 $C_C$：

**极点分裂效应**：

> 补偿前：$\omega_{p,\text{out}} \approx 1/(R_L C_L)$
>
> 补偿后：
> $$
> \omega_{p1} \approx \frac{1}{g_{m9} R_L R_S C_C}, \quad \omega_{p2} \approx \frac{g_{m9}}{C_E + C_L}
> $$

- **主管极点** $\omega_{p1}$ 被 Miller 效应推向原点 → $C_C$ 的等效值为 $(1 + A_{v2})C_C$，用小电容实现大时间常数。
- **输出极点** $\omega_{p2}$ 被推到更高频率 → 等效输出电阻从 $R_L$ 降到 $g_{m9}^{-1}$。

**Miller 补偿电容初估**（PM $= 45^\circ$, $\beta = 1$）：

$$
C_C \approx \frac{g_{m1}}{g_{m9}} C_L
$$

> 更精确的公式（含 $\omega_{p2}$ 效应）：$C_C = \dfrac{g_{m1}}{\sqrt{2}\,g_{m9}} C_L$。

### RHP 零点与消除

Miller 电容形成从输入到输出的**前馈路径**，产生 RHP 零点：

$$
\omega_z = +\frac{g_{m9}}{C_C + C_{GD9}}
$$

**RHP 零点的危害**：
- 贡献负相移（和 LHP 极点同向）→ 将 PX 推向原点
- 减缓增益下降 → 将 GX 推向更高频率
- **双重恶化稳定性**

**消除方法——串联电阻 $R_Z$**（Fig. 10.32）：

$$
\omega_z = \frac{1}{C_C(g_{m9}^{-1} - R_Z)}
$$

- $R_Z = g_{m9}^{-1}$ → $\omega_z = \infty$（消除）
- $R_Z = \frac{1}{g_{m9}}\left(1 + \frac{C_L}{C_C}\right)$ → $\omega_z = -\omega_{p2}$（移到 LHP 并与输出极点对消）

>[!warning] Pole-zero doublet 问题
>零极点对消依赖 $R_Z$ 与 $g_{m9}$ 精确匹配。不匹配时产生 **doublet**，阶跃响应中出现慢衰减项 $(1 - \omega_z/\omega_{p2})e^{-\omega_{p2}t}$，严重拖长建立时间（Problem 10.19）。

**$R_Z$ 的 PVT 跟踪实现**：
- 用 triode 区 MOS 管实现 $R_Z$，栅压由 $g_m$ 复制偏置电路生成（Fig. 10.34）→ $R_Z \propto 1/g_m$，跟踪良好。
- 或外接电阻 $R_S$ 定义偏置电流 $I_b \propto R_S^{-2}$，使 $g_m \propto R_S^{-1}$（Fig. 10.35）。

### Ahuja 补偿（共栅级隔离）(10.7)

用**共栅管** $M_2$ 代替串联电阻放在 $C_C$ 路径中（Fig. 10.43）：

$$
V_{\text{out}} \to C_C \to \text{source of } M_2 \to \text{drain of } M_2 \to \text{gate of output}
$$

**效果**：
- 阻断直通前馈路径 → 零点变为 **LHP 零点**
- 第二极点上推因子额外增加 $g_{m2}R_S$
- $$\omega_{p1} \approx \frac{1}{g_{m1} R_L R_S C_C}, \quad \omega_{p2} \approx \frac{g_{m1} g_{m2} R_S}{C_L}$$

**代价**：源跟随器或共栅级限制输出电压摆幅（需要额外的电压裕量）；正负摆率不对称。

### 前馈补偿（Feedforward / Cascode Miller）

将 $C_C$ 接在 cascode 管的**源极**和输出之间（Fig. 10.46）：

$$
\omega_z \approx \frac{g_{m4}R_{eq}\,g_{m9}}{C_C}
$$

零点频率比基本 Miller 的上推约 $g_{m4}R_{eq}$ 倍。可与串联电阻法组合使用（$C_C + C_C'$）。

### 全差分 cascode 运放的优势 (10.4)

全差分 telescopic cascode（Fig. 10.23）：
- **无镜像极点**（不需要单端转换）
- cascode 管的内部极点与输出极**合并**（$Z_{\text{out}}$ 推导证明极点合并，不产生额外极点）
- 只有一个非主管极点（在 cascode NMOS 源极）且处于高频
- **通常不需要补偿**

---

## 两级运放的摆率 (10.6)

Miller 补偿的两级运放中，正摆率 $\approx I_{SS}/C_C$，负摆率受限于尾电流和电流镜能力。

**带 $C_L$ 时的分析**：
- 节点 $X$ 为虚地点（输出级的高增益）
- $C_C$ 的充电电流 $I_{SS}$ 产生输出斜坡
- **摆率与负载电容 $C_L$ 无关**——只要输出管有足够电流同时供给 $C_C$ 和 $C_L$
- 但如果 $I_1 < (1 + C_L/C_C)I_{SS}$，输出管关断，摆率降为 $(I_1 - I_{SS})/C_L$

**Class-AB 两级运放**：摆率 $= [\alpha(W_{p1}/W_{p2}) - 1] I_{SS}/C_L$，可通过大 $I_1$ 获得更高摆率。

---

## 设计启示

1. **最坏情况就是单位增益**。按 $\beta = 1$ 优化补偿，其他增益配置自动稳定。
2. **补偿带宽极限 = 第一非主管极点位置**。要提升速度，必须把第一非主管极点推远——这是选择拓扑的核心考量。镜像极点是大忌。
3. **Miller 补偿的好处是双向的**：不仅压低主管极点，还推远输出极点。用 $C_C \approx (g_{m1}/g_{m9})C_L$ 起步。
4. **RHP 零点必须处理**。要么用串联电阻移到 LHP 对消输出极点，要么用 Ahuja/前馈结构从根本上阻断前馈路径。Doublet 不精确对消的建立时间惩罚不可忽视。
5. **全差分 telescopic cascode 是"免费午餐"**——不需要补偿，只需注意 cascode 栅的偏置。
6. **单级运放 vs 两级运放对 $C_L$ 的敏感方向相反**。单级运放增大 $C_L$ → 压低主管极点 → **更稳定**（过阻尼）；两级运放增大 $C_L$ → 输出极点内移 → **更不稳定**（可能振荡）。
7. **大信号瞬态不能用小信号 PM 完全预测**。偏置点和极点位置在大摆幅时会漂移——必须做瞬态仿真验证。
8. **Nyquist 判据是理解 Bode 法局限性的理论基础**，但在常规 CMOS 运放设计中 Bode + PM 足够——前提是开环极点全在 LHP 且没有奇怪的频率响应形状。

---

## 章节关联

- **Ch6 (Frequency Response)** — 频率响应分析基础：极点/零点、Bode 图构造、Miller 定理，本章所有 Bode 图分析依赖 Ch6 的方法论。
- **Ch8 (Feedback)** — 反馈的四大类型、环路增益分析、闭环特性。本章研究的对象正是 Ch8 所建立的负反馈系统。
- **Ch9 (Operational Amplifiers)** — 各种运放拓扑（telescopic、folded-cascode、两级）的极点分布直接决定补偿策略。本章的 Miller/Ahuja 补偿为 Ch9 的两级运放提供稳定性保障。
- **Ch13 (Switched-Capacitor Circuits)** — SC 电路中 $C_L$ 随时钟相位变化，对补偿提出了动态跟踪的要求（doublet 问题在 SC 中更突出）。
- **Ch15 (Oscillators)** — 振荡器正是利用不稳定性来产生周期信号。Barkhausen 判据在 Ch15 中被重新解读为**设计目标**而非需要避免的条件。
- **Ch16 (PLL)** — PLL 是含有两个积分器的反馈系统（VCO = 一个理想积分器），Nyquist 10.8 节对原点极点的分析直接适用。

---

## See Also

- [[Ch06 - Frequency Response of Amplifiers]] — Bode 图与极点/零点分析的必备前置
- [[Ch08 - Feedback]] — 负反馈系统的基本理论
- [[Ch09 - Operational Amplifiers]] — 本章补偿技术所服务的运放拓扑

---

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]] — Ch10: Stability and Frequency Compensation, pp. 410–458

## 检索关键词

`#stability` `#frequency-compensation` `#Nyquist` `#Bode-plot` `#phase-margin` `#Miller-compensation` `#Ahuja-compensation` `#RHP-zero` `#pole-splitting` `#slew-rate` `#Barkhausen-criterion` `#root-locus` `#pole-zero-doublet` `#two-stage-opamp` `#cascode-compensation`
