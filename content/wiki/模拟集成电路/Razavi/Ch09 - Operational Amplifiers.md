---
title: "Ch09 - Operational Amplifiers"
source: "Razavi-Design of Analog CMOS Integrated Circuits 2nd"
tags:
  - analog-design
  - CMOS
  - op-amp
  - operational-amplifier
  - CMFB
date: 2026-08-02
---

# Ch09 — Operational Amplifiers

## 本章定位

本章是 CMOS 运算放大器设计的**核心章节**，在第4章（差分放大器）、第5章（电流镜与偏置）和第8章（反馈）的基础上，系统介绍四种经典运放架构：telescopic cascode、folded-cascode、两级运放和增益增强（gain-boosting）。同时深入讨论共模反馈（CMFB）、压摆率（slew rate）、电源抑制（PSRR）和噪声等关键工程问题。本章是理解后续章节（Ch10 稳定性与频率补偿、Ch11 纳米级运放设计、Ch13 开关电容电路）的**必要前提**。

## 9.1 性能参数概览

运放不再追求"通用理想化"，而是在以下参数间多维折中：

| 参数 | 意义 | 典型约束 |
|---|---|---|
| **增益 (Gain)** | 决定闭环精度和非线性抑制 | 应用从 $10^1$ 到 $10^5$ 量级 |
| **小信号带宽** | 单位增益频率 $f_u$ 和 3dB 频率 $f_{3\text{-dB}}$ | 闭环建立时间 $\tau \approx \frac{1+R_1/R_2}{A_0\omega_0}$ |
| **大信号行为** | 大瞬态信号下进入非线性区 | 压摆率限制 |
| **输出摆幅** | 信号动态范围 | 低电压设计中最关键的约束 |
| **线性度** | 开环非线性由闭环反馈抑制 | 线性度要求往往而非增益误差决定所需 $A_0$ |
| **噪声与失调** | 决定可处理最小信号 | 至少四个器件贡献输入噪声 |
| **电源抑制 (PSRR)** | 抑制电源纹波 | 全差分拓扑有优势 |

> [!important] 关键折中
> 噪声与输出摆幅之间存在根本性折中：为增大摆幅而降低电流源过驱动电压时，其跨导 $g_m$ 增大，漏极噪声电流随之增大。

## 9.2 单级运放

### 9.2.1 基础拓扑

最简单的运放就是差分对 + 电流源负载（图9.6）：

$$A_v = g_{mN}(r_{ON} \parallel r_{OP})$$

纳米级工艺下这不超过10倍。单端输出版本含有镜像极点（Ch6），限制了反馈系统稳定性（Ch10）。

### 9.2.2 Telescopic Cascode 运放

在差分对上堆叠共源共栅管（图9.8）：

$$A_v \approx g_{m1}\left[(g_{m3}r_{O3}r_{O1}) \parallel (g_{m5}r_{O5}r_{O7})\right]$$

**优点**：增益高、功耗低、速度最高。

**缺点**：
- **输出摆幅受限**：差动输出摆幅 = $2[V_{DD} - (V_{OD1}+V_{OD3}+V_{ISS}+|V_{OD5}|+|V_{OD7}|)]$
- **输入输出短路困难**：作为单位增益缓冲器时，$V_{out}$ 可允许范围仅为 $V_{TH4} - (V_{GS4} - V_{TH2})$，即"一个阈值减一个过驱动"
- 偏置电压 $V_{b1}, V_{b2}$ 需要精确控制，通常需要与输入共模电压跟踪生成（图9.12）

### 9.2.3 Folded-Cascode 运放

将输入管类型翻转，解决 telescopic 的摆幅和输入共模问题（图9.15）：

$$A_v \approx g_{m1}\left\{[(g_{m7}+g_{mb7})r_{O7}r_{O9}] \parallel [(g_{m3}+g_{mb3})r_{O3}(r_{O1}\parallel r_{O5})]\right\}$$

| | Telescopic | Folded-Cascode |
|---|---|---|
| **增益** | 较高 | 低 2~3 倍（PMOS输入 $g_m$ 小；$r_{O1}\parallel r_{O5}$ 降低输出阻抗）|
| **功耗** | 低 | 高（输入对和共源共栅支路分别需要偏置电流）|
| **输出摆幅** | 受限（多一层尾电流源过驱动）| 稍大 |
| **输入共模范围** | 受限 | 更宽（PMOS输入可接受低至零的共模电平）|
| **折叠点极点** | — | 更低（电容 $C_{tot}$ 更大）|
| **噪声** | 低 | 较高 |

> [!info] Folded-Cascode 为何更常用？
> 尽管增降低、功耗高、极点低、噪声高，但两个决定性优势：(1) 输入输出共模电平可以相等而不限制输出摆幅；(2) 输入共模范围更宽。

### 9.2.4 线性缩放设计方法

设计从功率预算出发，依次分配过驱动电压和尺寸：

1. **功率预算** $\to$ 支路电流分配
2. **输出摆幅** $\to$ 过驱动电压分配
3. **增益** $\to$ 调整沟道长度 $L$（$g_m r_O \propto \sqrt{WL/I_D}$，$\lambda \propto 1/L$）
4. $g_m = 2I_D/(V_{GS}-V_{TH})$，$r_O = 1/(\lambda I_D)$

**线性缩放**：等比例放大所有宽度的同时保持长度不变 → 功耗翻倍，增益和摆幅不变，速度翻倍，噪声降为 $1/\sqrt{\text{比例}}$。

## 9.3 两级运放

当单级增益/输出摆幅不足时（如 0.9V 电源需要 0.8V 单端摆幅），采用两级结构（图9.23）：

- **第一级**：提供高增益（可含 cascode 进一步提高，图9.24）
- **第二级**：简单共源级，提供最大输出摆幅（$V_{DD} - |V_{OD5,6}| - V_{OD7,8}$）

$$A_v = g_{m1,2}(r_{O1,2}\parallel r_{O3,4}) \times g_{m5,6}(r_{O5,6}\parallel r_{O7,8})$$

> [!warning] 级数限制
> 多于两级极少使用，因为每级至少引入一个极点，使反馈系统稳定性难以保证（Ch10）。

## 9.4 增益增强 (Gain Boosting)

### 原理

在不增加 cascode 层数（不牺牲摆幅）的前提下，用辅助放大器 $A_1$ 提高输出阻抗。

**两种等价视角**：
1. $A_1$ 将 cascode 管的等效 $g_m$ 增大为 $(A_1+1)g_m$
2. $A_1$ 监测并钳制 cascode 管源极电压，使漏极电流不随漏极电压变化

$$R_{out} \approx (A_1+1)g_{m2}r_{O2}r_{O1} + r_{O1} + r_{O2}$$

增益 = $g_{m1} \times R_{out}$，等效于"四重 cascode"。

### 电路实现

辅助放大器通常用折叠共源共栅结构（图9.34），以避免额外的电压摆幅限制。增益增强可同时施加于 NMOS cascode 和 PMOS 负载，实现极高的总增益（图9.36）。

### 频率响应

增益增强电路具有**单级特性**：大部分信号直接流经 cascode 到达输出，只有小部分"误差"分量经过辅助放大器。

主极点：$\omega_{p1} \approx \dfrac{1}{(A_1+1)g_{m2}r_{O2}r_{O1}C_L}$

非主极点：$\omega_{p2} \approx \dfrac{1}{g_{m2}r_{O2}r_{O1}C_L} + (A_0+1)\omega_0$

左半平面零点：$\omega_z \approx -(A_0+1)\omega_0$

> [!tip] 关键结论
> 辅助放大器贡献一个位于原始-3dB带宽以上的第二极点（约 $A_0\omega_0$ 高处），和一个近似于辅助放大器单位增益带宽的零点。

## 9.5 四种架构对比

| | Telescopic | Folded-Cascode | Two-Stage | Gain-Boosted |
|---|---|---|---|---|
| **增益** | Medium | Medium | High | High |
| **输出摆幅** | Medium | Medium | Highest | Medium |
| **速度** | Highest | High | Low | Medium |
| **功耗** | Low | Medium | Medium | High |
| **噪声** | Low | Medium | Low | Medium |

## 9.6 输出摆幅的仿真验证

> 由于纳米器件饱和/线性区边界模糊，不能用简单的区域判断。

**方法**：施加增长的正弦输入，监测增益-摆幅关系。允许增益下降约 1dB (≈10%) 对应的摆幅即为最大可接受摆幅。更精确的做法是在闭环环境下测量输出失真（谐波）。

## 9.7 共模反馈 (CMFB)

### 为什么需要 CMFB？

高增益差分放大器中，P型电流源与N型电流源的微小失配（$\Delta I \times R_{out}$）产生巨大的输出电压误差，将一侧电流源推入线性区。

> [!important]
> **差分反馈不能确定输出共模电压**。不能用仿真中精细调 $V_b$ 来规避 CMFB —— 实际电路中随机失配始终存在。

CMFB 的三要素：**感知**输出共模电平 $\to$ **比较**参考电压 $\to$ **反馈**至偏置网络。

### 共模感知方法

| 方法 | 优点 | 缺点 |
|---|---|---|
| 电阻分压器（图9.45）| 简单，线性好 | 负载效应降低开环增益；大电阻面积大、寄生电容大 |
| 源极跟随器+电阻（图9.46）| 消除电阻负载 | 输出摆幅减小约 $V_{TH}$；大差分摆幅时可能电流饥饿 |
| 深线性区 MOS 管（图9.48）| $R_{tot} \propto 1/(V_{out1}+V_{out2})$ 只与 CM 有关 | 摆幅受限；若脱离深线性区则 CM 感知失真 |
| 差分对感知（图9.50）| 无电阻负载 | 非线性严重，大摆幅时 $I_{CM}$ 失真 |

### 反馈控制技术

- **控制尾电流源**（图9.51、9.52）：误差放大器调节 NMOS/PMOS 尾电流
- **深线性区 MOS 反馈**（图9.53、9.54）：$R_{on7}\parallel R_{on8}$ 调节偏置电流，但输出 CM 依赖器件参数
- **改进型**（图9.56、9.57）：利用电流镜追踪参考电压，使 $V_{out,CM}$ 接近 $V_{REF}$，且加入 cascode 抵消沟道调制误差
- **电阻 CMFB**（图9.58）：二极管连接 + 电阻，对差分信号 PMOS 作电流源、对 CM 作二极管

### 两级运放的 CMFB

- 仅控制第二级：第一级 CM 无控制 → 不可行
- 从第二级感知、反馈到第一级尾电流：极点过多（3~4个），稳定性难保证
- **推荐方案**：两级各自独立的 CMFB 环路（图9.62）

## 9.8 输入共模范围扩展

通过并行 NMOS 和 PMOS 输入对实现**轨到轨输入**（图9.66）：

- 输入接近地：NMOS 对关闭，PMOS 对工作
- 输入接近 $V_{DD}$：PMOS 对关闭，NMOS 对工作

代价：总跨导 $G_{m,tot} = g_{mn} + g_{mp}$ 随输入共模电平变化，导致增益、速度、噪声波动。

## 9.9 压摆率 (Slew Rate)

### 定义与起因

当输入阶跃足够大，差分对一侧完全截止，尾电流全部用于对负载电容充放电，输出呈恒定斜率斜线：

$$\text{SR} = \frac{I_{SS}}{C_L}$$

> [!important] Slew Rate 是**非线性**现象
> 线性系统：斜率正比于输入幅度 → 双倍输入则每点输出翻倍
> 发生 Slewing 时：输出斜坡斜率与输入幅度**无关**

### 各类运放的 Slew Rate

- **Telescopic** (图9.74)：$\text{SR}_{\text{diff}} = I_{SS}/C_L$
- **Folded-Cascode 单端** (图9.75)：$\text{SR} = I_{SS}/C_L$（需 $I_P \ge I_{SS}$，否则折叠点电压大幅摆动导致长时间恢复）
- **解决办法**：加箝位管（图9.77）限制折叠点电压摆幅

### 什么时候退出 Slewing？

粗略估计：$\Delta V_G \approx \alpha I_{SS}/g_m \approx \alpha\sqrt{I_{SS}/[\mu_n C_{ox}(W/L)]}$ 时差分管开始线性导通（$\alpha \approx 0.1$ 对应约 1% 非线性）。

## 9.10 高压摆率运放 — Class-AB

利用**推挽（push-pull）**结构，使负载电流源在大信号时变为"有源上拉"：

- 图9.79(d)：差分 SR = $[I_{SS1} + I_{SS2}(W_4/W_8)]/C_L$
- 图9.80（两级Class-AB）：第一级输出节点（P/Q）可摆动到接近 $V_{DD}$，对输出级施加远超静态偏置的过驱动

代价：镜像极点降低，传递函数引入零点 $\omega_z = -[1+(W_4/W_8)(g_{m5}/g_{m1})]\omega_{out}$。

## 9.11 电源抑制 (PSRR)

低频率下：
$$\text{PSRR} = \frac{A_{v,\text{diff}}}{A_{v,\text{sup}}} \approx g_{mN}(r_{OP}\parallel r_{ON})$$

- 简单差分对的 $\partial V_{out}/\partial V_{DD} \approx 1$（二极管连接钳制了 $V_X$）
- 闭环反馈以相同因子降低 $\partial V_{out}/\partial V_{DD}$ 和 $\partial V_{out}/\partial V_{in}$，因此 PSRR 基本不变

## 9.12 运放噪声

**Telescopic** 输入参考热噪声（忽略 cascode 管）：

$$\overline{V_n^2}_{,\text{in}} \approx 8kT\left(\frac{2}{3g_{m1,2}} + \frac{2g_{m7,8}}{3g_{m1,2}^2}\right)$$

**Folded-Cascode** 输入参考热噪声（含两组负载电流源 $M_{7,8}$ 和 $M_{9,10}$）：

$$\overline{V_n^2}_{,\text{in}} \approx 8kT\left(\frac{2}{3g_{m1,2}} + \frac{2g_{m7,8}}{3g_{m1,2}^2} + \frac{2g_{m9,10}}{3g_{m1,2}^2}\right)$$

> Folded-cascode 的噪声比 telescopic 大（多了 $M_{9,10}$ 的贡献）。

**两级运放**：第二级噪声除以第一级增益后通常可忽略：

$$\overline{V_n^2}_{,\text{in}} \approx 8kT\left(\frac{2}{3g_{m1}} + \frac{2g_{m3}}{3g_{m1}^2}\right) + \frac{8kT\left(\frac{2}{3g_{m5}} + \frac{2g_{m7}}{3g_{m5}^2}\right)}{[g_{m1}(r_{O1}\parallel r_{O3})]^2}$$

**闪烁噪声**敏感的场合优先选用 PMOS 输入对（PMOS 通常具有更低的 $1/f$ 噪声系数）。

**噪声-功耗缩放**：所有宽度和 $I_{SS}$ 等比例缩小一半 → 功耗减半，$\overline{V_n^2}_{,\text{in}}$ 翻倍，增益和摆幅不变。

## 设计启示

1. **摆幅是低压设计的首要约束**——先分配过驱动电压，再满足增益和噪声。
2. **增益不够先加 $L$ 而非加级数**——信号路径上的器件保持最短沟道，负载器件可以加长（沟道调制减小）。
3. **Folded-cascode 是工程上最实用的单级架构**——牺牲少量增益和噪声，换取输入/输出共模兼容性和更宽的输入范围。
4. **CMFB 不是可选项**——高增益差分电路必须有 CMFB，且两级各自独立 CMFB 环路是稳定性最友好的方案。
5. **Slew rate 是区分"纸面带宽"和"真实大信号速度"的关键**——大信号应用下 SR 可能才是速度瓶颈。
6. **噪声优化的核心是输入对 $g_m$ 最大化和负载 $g_m$ 最小化**——低过驱动给输入对，高过驱动给负载电流源。

## 章节关联

- **Ch03** 单级放大器 — CS, CG, CD, cascode 基本原理
- **Ch04** 差分放大器 — 差分对分析、共模抑制
- **Ch05** 电流镜与偏置 — 偏置电压生成、低压 cascode
- **Ch08** 反馈 — 闭环增益误差、环路增益、反馈对阻抗的影响
- **Ch10** 稳定性与频率补偿 — Miller 补偿、两级运放稳定性、极点分裂

## 检索关键词

`运放` `operational amplifier` `OTA` `telescopic cascode` `folded-cascode` `两级运放` `two-stage op-amp` `gain boosting` `regulated cascode` `CMFB` `共模反馈` `slew rate` `压摆率` `PSRR` `电源抑制` `输出摆幅` `output swing` `linear scaling` `class-AB` `push-pull` `噪声` `noise` `输入共模范围` `input CM range`

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]

## See Also

- [[Ch03 - Single-Stage Amplifiers]] — CS/CG/CD 和基本 cascode
- [[Ch04 - Differential Amplifiers]] — 差分对分析
- [[Ch05 - Current Mirrors and Biasing Techniques]] — 偏置与低压 cascode
- [[Ch08 - Feedback]] — 反馈系统基础
- [[Ch10 - Stability and Frequency Compensation]] — 稳定性与频率补偿
- [[Ch11 - Nanometer Design Studies]] — 深亚微米运放设计
- [[Ch13 - Introduction to Switched-Capacitor Circuits]] — 开关电容电路中的运放应用
