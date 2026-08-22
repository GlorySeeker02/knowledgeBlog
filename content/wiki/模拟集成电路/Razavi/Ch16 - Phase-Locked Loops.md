---
title: Ch16 - Phase-Locked Loops
source: "[[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]"
tags: [analog-design, CMOS, PLL, phase-locked-loop, charge-pump, frequency-synthesis]
date: 2026-08-02
---

# Ch16 - Phase-Locked Loops

## 本章定位

本章是 Razavi 教材中最完整的 PLL 导论章节，涵盖从基本概念到电荷泵 PLL（CPPLL）、延迟锁定环（DLL）和实际应用的完整链路。PLL 作为反馈系统的独特之处在于：比较的是**相位**而非电压或电流，锁定后 $\omega_{out} = \omega_{in}$ **精确相等**。

PLL 几乎无处不在——微处理器时钟生成、手机频率合成器、SerDes 时钟恢复——但其具体设计因工艺和应用差异极大。本章的目标是为后续深入学习打下系统级基础。

---

## 核心概念

### 锁相环的本质

PLL 是一个**相位负反馈系统**：鉴相器（PD）比较输入输出相位差 $\Delta \phi$，产生的误差电压经低通滤波后控制 VCO 频率，使输出相位追踪输入相位。

> [!important] 锁定条件
> PLL **锁定** 的充要条件： $\phi_{out} - \phi_{in}$ **不随时间变化**。
>
> 推论：
> $$\frac{d\phi_{out}}{dt} = \frac{d\phi_{in}}{dt} \quad\Rightarrow\quad \boxed{\omega_{out} = \omega_{in}}$$
>
> 这就是 PLL 区别于频率锁定环（FLL）的核心优势——锁定后**频率完全相等**，不存在确定性的频率误差。

### 相位检测器 (Phase Detector)

> [!abstract] PD 定义
> 鉴相器是一个电路，其**平均输出** $V_{out}$ 与两输入之间的相位差 $\Delta\phi$ 线性成正比。增益 $K_{PD}$ 单位：V/rad。

**XOR 鉴相器**是最直观的例子：
- 相位差增大 → 输出脉冲宽度增宽 → 直流分量增大
- 增益：$K_{PD} = V_0 / \pi$（与输入频率无关）
- 特性在 $\Delta\phi = 0, \pm 2\pi, \dots$ 处过零，呈周期性，兼具正负增益区

**Bang-Bang 鉴相器**（D 触发器实现）：
- $\Delta\phi > 0$ 时输出恒定高电平，$\Delta\phi < 0$ 时恒定低电平
- 仅在 $\Delta\phi = 0, \pm\pi, \dots$ 处有极高增益，其余区域增益为零

### 鉴频鉴相器 (PFD)

PFD 是 PLL 演进中的关键发明，能**同时检测相位和频率差异**：

> [!abstract] PFD 三态机
> - 初始 $Q_A = Q_B = 0$
> - A 上升沿先到 → $Q_A=1, Q_B=0$（UP 脉冲）
> - B 上升沿后到 → $Q_A=0$（复位）
> - 反之亦然——$Q_B$ 产生 DOWN 脉冲
>
> **频率检测**：若 $\omega_A > \omega_B$，$Q_A$ 持续产生脉冲而 $Q_B$ 保持零，直流分量提供频率误差信息。

PFD 实现（双 DFF + AND 复位）：
- 两个边沿触发、可复位的 DFF，D 端接逻辑 1
- A、B 分别作为 CK 输入
- $Q_A$ 和 $Q_B$ 同时为高时，AND 门复位两个 DFF
- 复位脉冲宽度 $\approx 5$ 个门延迟

### 电荷泵 (Charge Pump)

电荷泵将 PFD 的逻辑输出转换为**对电容的充放电电流**：

- $Q_A$ 高、$Q_B$ 低 → $I_1$（UP 电流）对 $C_P$ 充电，$V_{out}$ 上升
- $Q_A$ 低、$Q_B$ 高 → $I_2$（DOWN 电流）对 $C_P$ 放电，$V_{out}$ 下降
- $Q_A = Q_B = 0$ → $V_{out}$ 保持不变

> [!tip] 无限增益特性
> PFD/CP/LPF 级联有一个关键性质：**有限的输入相位误差会产生无限的输出电压变化**。因为即使是固定相位差，也会在每个周期持续充/放电，$V_{out}$ 向 $+\infty$ 或 $-\infty$ 增长。
>
> 这意味着 CPPLL 锁定后的**静态相位误差理论上为零**——与 Type-I PLL 形成鲜明对比。

### 延迟锁定环 (DLL)

DLL 用**压控延迟线（VCDL）**替代 VCO：

| 特性 | PLL | DLL |
|------|-----|-----|
| 核心模块 | VCO（积分器：$K_{VCO}/s$） | VCDL（无积分：$K_{VCDL}$） |
| 传递函数 | $\Phi_{out}(s)/V_{cont}(s) = K_{VCO}/s$ | $\Phi_{out}(s)/V_{cont}(s) = K_{VCDL}$ |
| 系统阶数 | LPF 阶数 + 1 | 等于 LPF 阶数 |
| 噪声敏感性 | 高（振荡器循环累积噪声） | 低（延迟线末端噪声消失） |
| 频率合成 | 支持（乘/除） | **不支持** |
| 抖动响应 | $\Phi_{in}$ 低通，VCO 噪声高通 | 近似全通（慢和快抖动都通过） |

DLL 的主要局限：**不能生成可变频率**，且可能存在锁定延迟歧义（总延迟可对应 $T_{in}$ 或 $2T_{in}$）。

---

## 关键公式与结论

### Type-I PLL（简单 PLL，一阶 LPF）

**开通传递函数**（PD + 一阶 LPF + VCO）：

$$H_{open}(s) = K_{PD} \cdot \frac{1}{1 + s/\omega_{LPF}} \cdot \frac{K_{VCO}}{s}$$

闭环传递函数：

$$H(s) = \frac{\Phi_{out}}{\Phi_{in}}(s) = \frac{K_{PD}K_{VCO} \cdot \omega_{LPF}}{s(s + \omega_{LPF}) + K_{PD}K_{VCO} \cdot \omega_{LPF}}$$

写成标准二阶形式 $s^2 + 2\zeta \omega_n s + \omega_n^2$：

$$\boxed{\omega_n = \sqrt{K_{PD}K_{VCO} \cdot \omega_{LPF}},\qquad \zeta = \frac{1}{2}\sqrt{\frac{\omega_{LPF}}{K_{PD}K_{VCO}}}}$$

> [!warning] Type-I PLL 的四重折中
> 1. **相位误差** vs **稳定性**：$\phi_{out} - \phi_{in} \propto 1/(K_{PD}K_{VCO})$，但 $\zeta \propto 1/\sqrt{K_{PD}K_{VCO}}$——降低相位误差会恶化稳定性
> 2. **建立速度** vs **控制电压纹波**：$\zeta\omega_n = \omega_{LPF}/2$——降低 $\omega_{LPF}$ 抑制纹波，但减慢建立
> 3. **捕获范围** $\approx \omega_{LPF}$——降低 $\omega_{LPF}$ 缩小可锁定的频率范围
> 4. 阻尼因子一般选 $\zeta \geq \sqrt{2}/2$ 或 1，避免过度振荡

**欠阻尼阶跃响应**（$\zeta < 1$）：

$$\omega_{out}(t) = \Delta\omega \left[1 - \frac{1}{\sqrt{1-\zeta^2}} e^{-\zeta\omega_n t} \sin\left(\omega_n \sqrt{1-\zeta^2}\,t + \theta\right)\right]$$

建立时间常数：$\tau = (\zeta\omega_n)^{-1}$

**误差传递函数**：

$$H_e(s) = \frac{\Phi_{in} - \Phi_{out}}{\Phi_{in}} = \frac{s^2 + s\omega_{LPF}}{s^2 + s\omega_{LPF} + K_{PD}K_{VCO}\,\omega_{LPF}}$$

频率阶跃 $\Delta\omega$ 引起的稳态相位误差：

$$\boxed{\phi_{out} - \phi_{in} = \frac{\Delta\omega}{K_{PD}K_{VCO}}}$$

### Type-II PLL（电荷泵 PLL）

**PFD/CP/LPF 传递函数**（仅电容）：

$$\frac{V_{out}}{\Delta\phi}(s) = \frac{I_P}{2\pi C_P} \cdot \frac{1}{s}$$

**加入串联电阻 $R_P$ 后**（引入零点稳定系统）：

$$\frac{V_{out}}{\Delta\phi}(s) = \frac{I_P}{2\pi}\left(R_P + \frac{1}{C_P s}\right)$$

开通传递函数：

$$H_{open}(s) = \frac{I_P}{2\pi}\left(R_P + \frac{1}{C_P s}\right)\frac{K_{VCO}}{s}$$

闭环传递函数（标准二阶形式）：

$$\boxed{\omega_n = \sqrt{\frac{I_P K_{VCO}}{2\pi C_P}},\qquad \zeta = \frac{R_P}{2}\sqrt{\frac{I_P C_P K_{VCO}}{2\pi}}}$$

> [!important] Type-II vs Type-I 稳定性趋势相反
> - **Type-I**：$K_{PD}K_{VCO}$ **增大** → $\zeta$ **减小** → 稳定性**恶化**
> - **Type-II**：$I_P K_{VCO}$ **增大** → $\zeta$ **增大** → 稳定性**改善**
>
> 因此，Type-II PLL 可以在不牺牲稳定性的前提下提高环路增益。

**加入分频器 $\div M$ 后**：

$$\omega_n = \sqrt{\frac{I_P K_{VCO}}{2\pi M C_P}},\qquad \zeta = \frac{R_P}{2}\sqrt{\frac{I_P C_P K_{VCO}}{2\pi M}}$$

分频器使 $\zeta$ 和建立速度均乘以 $1/\sqrt{M}$。补偿方法：按比例增加 $I_P$。

**第三阶电容 $C_2$**（并联于 $R_P$-$C_P$ 串联支路）：

- 作用：抑制每次电荷泵注入时 $V_{cont}$ 上的电压跳变
- 取值：$C_2 \approx C_P/5 \sim C_P/10$
- 代价：系统变为三阶，稳定性分析更复杂

### 锁定范围与捕获

- **Type-I PLL**：捕获范围 $\approx \omega_{LPF}$（非常粗略估计），VCO 频率偏离输入超过此范围则无法锁定
- **Type-II PLL（带 PFD）**：由于 PFD 的频率检测能力，捕获范围扩展到 VCO 的**整个调谐范围**（对周期性信号而言）

### 抖动特性

> [!abstract] PLL 对抖动的滤波特性
> - **输入抖动** → 输出：**低通**特性。慢抖动无衰减传播，快抖动被抑制。PLL 相当于中心频率 $f_{in}$ 处的窄带滤波器。
> - **VCO 抖动** → 输出：**高通**特性。VCO 的慢扰动被环路纠正，快扰动无法被抑制。

VCO 相位噪声到输出的传递函数（Type-II）：

$$\frac{\Phi_{out}}{\Phi_{VCO}}(s) = \frac{s^2}{s^2 + \frac{I_P K_{VCO}}{2\pi}(R_P C_P s + 1)}$$

环路带宽的选择需要在输入抖动抑制和 VCO 噪声抑制之间折中。

---

## 重要模块详解

### 鉴相器 (XOR PD)

- **增益**：$K_{PD} = V_0 / \pi$
- **特性**：周期性，$\Delta\phi = 0, \pm\pi, \pm 2\pi, \dots$ 处过零
- **局限**：只能检测相位，不能检测频率；对占空比敏感
- **锁定点**：若 $K_{PD}K_{VCO}$ 足够大，XOR-PD 型 PLL 自然锁定在 $\Delta\phi \approx 90^\circ$

### PFD + Charge Pump

**PFD 实现要点**：

- 双复位 DFF + AND 复位路径
- 复位脉冲宽度 $\approx 5$ 个门延迟
- 输入-输出特性在 $-360^\circ$ 到 $+360^\circ$ 范围内单调

**电荷泵非理想效应**：

| 非理想效应 | 成因 | 影响 | 解决方案 |
|-----------|------|------|---------|
| **死区 (Dead Zone)** | 小相位差时 UP/DOWN 脉冲来不及达到有效逻辑电平 | $|\Delta\phi| < \phi_0$ 内环路增益为零，VCO 可自由积累随机相位误差（→ 抖动） | 利用 PFD 的窄复位脉冲保证开关总能导通 |
| **电流失配** | $I_{UP} \neq I_{DOWN}$（短沟道 MOSFET 输出阻抗低） | 锁定后产生**静态相位偏移**以便每周期平均净电流为零；$V_{cont}$ 仍有周期纹波 | 提高电流源输出阻抗；差分布局 |
| **电荷共享** | $S_1, S_2$ 关断时 $X$ 被拉至 GND、$Y$ 被拉至 $V_{DD}$；导通时与 $C_P$ 的电荷重新分配 | $V_{cont}$ 跳变，纹波增大 | "自举"技术：用单位增益缓冲器在开关关断时保持 $V_X = V_Y = V_{cont}$ |
| **UP/DOWN 时序偏差** | $Q_A$ 和 $\overline{Q_B}$ 到达开关的延迟不同 | 即使锁定，每周期仍有短暂电流注入 | 在 $\overline{Q_B}$ 路径加传输门匹配延迟 |
| **时钟馈通 & 电荷注入** | 开关管 $M_3, M_4$ 的寄生效应 | 进一步增大相位误差和纹波 | 差分布局、互补开关、增大 $C_P$ |

### VCO

直接继承自 Ch15，本章关注的要点：
- 数学模型：$\omega_{out} = \omega_0 + K_{VCO}V_{cont}$，相位 = $\int \omega_{out}\,dt$ → 拉普拉斯域 $K_{VCO}/s$（积分器）
- 分频器等效：VCO + $\div M$ 链 **无法区分于** 一个截距频率 $\omega_0/M$、增益 $K_{VCO}/M$ 的 VCO

### 分频器 ($\div M$)

- 实现：计数器，每 $M$ 个输入脉冲输出一个脉冲
- 作用：**频率乘法** —— $f_{out} = M \cdot f_{in}$（精确整数倍关系）
- 对环路影响：等效于将 $K_{VCO}$ 除以 $M$，需增加 $I_P$ 补偿

---

## 设计启示

1. **Type-I 还是 Type-II？** 现代 IC 设计几乎全部使用 Type-II（电荷泵 PLL）。Type-I 局限于窄捕获范围 + 相位误差/稳定性/纹波三重折中无法同时满足。唯有需要极简实现（如分立器件）时才考虑 Type-I。

2. **PFD 比纯 PD 好在哪里？** 频率检测能力使捕获范围扩展到 VCO 整调谐范围，同时零静态相位误差（理想情况下）解决了 Type-I 的根本局限。

3. **$R_P$ 零点的关键性**——Type-II PLL 的两个原点极点导致 180 度相移，不加 $R_P$ 必然振荡。$R_P$ 引入零点，使相位在增益交越频率处回升，保证相位裕量。

4. **$C_2$ 的必要性**——仅有 $R_P$-$C_P$ 时每次电荷泵操作都会产生控制电压跳变。$C_2$（$C_P/5 \sim C_P/10$）抑制跳变，代价是将二階系统变为三階，需要更谨慎的稳定性设计。

5. **$I_P$ 的上限由功耗和面积决定，下限由稳定性**（Type-II 中 $I_P K_{VCO}$ 降低使 $\zeta$ 减小→相位裕量恶化）。

6. **环路带宽选择**是输入抖动抑制（低通）和 VCO 噪声抑制（高通）的折中——没有单一最优值，取决于应用场景。

7. **DLL 选型考量**：若不需要频率合成，DLL 比 PLL 更简单、更抗噪声。但需注意延迟歧义锁定和级间失配。

8. **分频比 $M$ 增大时**，必须**线性增大 $I_P$** 以维持 $\zeta$ 和建立速度不变。

---

## 章节关联

- **Ch15 Oscillators** → 本章的 VCO 模型（$\omega_{out} = \omega_0 + K_{VCO}V_{cont}$, $K_{VCO}/s$ 传递函数）直接来自 Ch15。VCDL 中每个延迟单元也基于 Ch15 的环形振荡器级。
- **Ch10 Stability and Frequency Compensation** → Bode 图、相位裕量、根轨迹分析方法是本章环路稳定性分析的基础工具。
- **Ch8 Operational Amplifiers** → PLL 与运放反馈的对称类比（分压器 ↔ 分频器、输入失调 ↔ 静态相位误差）帮助建立直觉。
- **Ch4 Differential Amplifiers** → Gilbert 单元可用作模拟乘法器型鉴相器（小信号正弦输入时）。

---

## See Also

- [[../抖动与相位噪声/总览 MOC]] — PLL 抖动与相位噪声的系统级分析（12 篇笔记）
- 第 15 章 Oscillators — VCO 设计基础
- 第 10 章 Stability — 反馈稳定性工具
- Gardner, *Phaselock Techniques* — PLL 经典参考
- Gardner, "Charge-Pump Phase-Locked Loops," *IEEE Trans. Comm.*, 1980 — CPPLL 开创性论文

---

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]] — Chapter 16, pp. 651–689
- Best, *Phase-Locked Loops*, 2nd ed., McGraw-Hill, 1993
- Gardner, *Phaselock Techniques*, 2nd ed., Wiley, 1979

---

## 检索关键词

`PLL` `锁相环` `相位检测器` `鉴相器` `鉴频鉴相器` `PFD` `电荷泵` `charge-pump PLL` `CPPLL` `Type-I PLL` `Type-II PLL` `环路滤波器` `环路动态` `自然频率` `阻尼因子` `$\zeta$` `$\omega_n$` `锁定范围` `捕获范围` `死区` `dead zone` `抖动` `jitter` `相位噪声` `延迟锁定环` `DLL` `VCDL` `频率合成` `频率乘法` `分频器` `时钟恢复` `skew reduction` `根轨迹` `Bode 图` `相位裕量` `Razavi` `CMOS` `模拟集成电路`
