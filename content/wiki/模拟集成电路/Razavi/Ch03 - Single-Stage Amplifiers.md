---
title: "Ch03 - Single-Stage Amplifiers"
source: "Razavi - Design of Analog CMOS Integrated Circuits, 2nd Ed."
tags: [analog-design, CMOS, amplifiers, single-stage]
date: 2026-08-02
---

# Ch03 - 单级放大器

## 本章定位

本章是 Razavi 模拟 CMOS 设计教材中**放大器基础理论的基石**。在引入偏置（Ch05）和频率响应（Ch06）之前，本章系统地建立了四种基本单级放大器拓扑的低频小信号与大信号分析方法，并通过大量实例培养"**由观察直接写出增益/阻抗**"的直觉。

> "An important part of a designer's job is to use proper approximations so as to create a simple mental picture of a complicated circuit."

---

## 核心概念

### 模拟设计八边形 (Analog Design Octagon)

放大器设计是一个多维优化问题。以下参数互相制约：

- **增益 (Gain)** — 输出电压变化与输入电压变化之比
- **速度 (Speed/Bandwidth)** — 由节点时间常数决定
- **噪声 (Noise)** — 晶体管和电阻的热噪声与闪烁噪声
- **线性度 (Linearity)** — 大信号下增益是否恒定
- **功耗 (Power Dissipation)** — 静态偏置电流乘以电源电压
- **输入/输出阻抗 (I/O Impedance)** — 影响级间匹配与信号传递
- **电压摆幅 (Voltage Swings)** — 输出能达到的最大/最小电压范围
- **电源电压 (Supply Voltage)** — 工艺决定的可用电压区间

### 非线性本质

$$
y(t) = \alpha_0 + \alpha_1 x(t) + \alpha_2 x^2(t) + \alpha_3 x^3(t) + \dots
$$

增益是输入-输出特性的斜率。当信号幅度大、或 $g_m$ 随信号显著变化时，增益不再恒定 → **非线性失真**。非线性的详细处理见 Ch14。

---

## 关键公式与结论

### 0. 前置引理：$A_v = -G_m R_{out}$

> 任意线性电路的电压增益 = $-G_m \times R_{out}$，其中
> - $G_m = \left.\frac{I_{out}}{V_{in}}\right|_{V_{out}=0}$ 是**输出短路**时的等效跨导
> - $R_{out}$ 是**输入置零**时的输出电阻

这条引理将增益计算拆分为两个可分别求解的子问题，贯穿全章。

### 1. 共源级 (Common-Source, CS)

| 负载类型 | 电压增益 $A_v$ | 输出阻抗 | 特点 |
|:--|:--|:--|:--|
| **电阻负载** | $-g_m (r_O \parallel R_D)$ | $r_O \parallel R_D$ | 最基础；增益与压摆折中 |
| **二极管连接负载** | $\displaystyle -\frac{g_{m1}}{g_{m2}+g_{mb2}} \approx -\sqrt{\frac{(W/L)_1}{(W/L)_2}}$ (NMOS) 或 $-\sqrt{\frac{\mu_n(W/L)_1}{\mu_p(W/L)_2}}$ (PMOS) | $\approx 1/g_{m2}$ | 增益相对恒定→线性度好；输出摆幅受限 ($V_{out,max}=V_{DD}-\vert V_{TH}\vert$) |
| **电流源负载** | $-g_{m1}(r_{O1} \parallel r_{O2})$ | $r_{O1} \parallel r_{O2}$ | 达到单管本征增益 $g_m r_O$ (~5-10 in 短沟道) |
| **有源负载 (互补 CS)** | $-(g_{m1}+g_{m2})(r_{O1} \parallel r_{O2})$ | $r_{O1} \parallel r_{O2}$ | 跨导叠加增益最高；偏置电流对 PVT 敏感|
| **线性区负载** | $-g_{m1}R_{on2}$ | $R_{on2}$ | 电压摆幅大 ($V_{out,max}=V_{DD}$)，但电阻精度差 |
| **源极退化 (degeneration)** | $\displaystyle -\frac{R_D}{\frac{1}{g_m}+R_S}$ (简化) | $[1+(g_m+g_{mb})r_O]R_S+r_O$ | 线性度改善，但增益降低、噪声增加 |

#### 源极退化的关键结果

等效跨导：
$$
G_m = \frac{g_m}{1+(g_m+g_{mb})R_S}
$$

当 $g_m R_S \gg 1$ 时 $G_m \approx 1/R_S$，电流变成输入电压的线性函数。

输出电阻（退化后）：
$$
R_{out} = [1+(g_m+g_{mb})R_S]r_O + R_S \approx [1+(g_m+g_{mb})r_O]R_S + r_O
$$

> **直观理解**：$r_O$ 被放大了 $(1+g_m R_S)$ 倍。

> **增益由观察法口诀**：$|A_v| = \dfrac{\text{漏极看到的电阻}}{\text{源极路径中的总电阻}}$

### 2. 源跟随器 (Source Follower / Common-Drain)

$$
A_v = \frac{R_{eq}}{R_{eq} + \frac{1}{g_m}}, \quad R_{eq} = \frac{1}{g_{mb}} \parallel r_{O1} \parallel r_{O2} \parallel R_L
$$

| 参数 | 公式 |
|:--|:--|
| 最大增益 (无体效应, $R_S\to\infty$) | $A_v = 1$ |
| 最大增益 (有体效应) | $\displaystyle A_v = \frac{g_m}{g_m+g_{mb}} = \frac{1}{1+\eta}$ |
| 输出阻抗 | $\displaystyle R_{out} = \frac{1}{g_m+g_{mb}} \parallel r_O$ |

- **高输入阻抗、适中输出阻抗** — 用作电压缓冲器
- **缺点 1**：体效应导致增益小于 1 且非线性 ($V_{TH}$ 随 $V_{SB}$ 变化)
- **缺点 2**：电平移位 $V_{GS}$ 消耗电压裕度；Signal swing 减少一个 $V_{GS}$
- **消除体效应**：PMOS 源跟随器 + 独立 N 阱 (Fig 3.44)

> [!warning] 设计准则
> 除非绝对必要，**避免使用源跟随器**。在驱动 50Ω 负载时，CS 级的增益可能比源跟随器更高。

### 3. 共栅级 (Common-Gate, CG)

$$
A_v = \frac{(g_m+g_{mb})r_O + 1}{R_D + r_O + [1+(g_m+g_{mb})r_O]R_S} \cdot R_D
$$

- 电压增益为**正** (同相)
- 体效应**增大**等效跨导：$G_m = g_m + g_{mb}$
- **低输入阻抗**：$R_{in} \approx \dfrac{R_D + r_O}{1+(g_m+g_{mb})r_O} \approx \dfrac{1}{g_m+g_{mb}}$ (当 $R_D$ 较小时)

| 场景 | 输入阻抗 |
|:--|:--|
| $R_D = 0$ | $R_{in} = \frac{1}{g_m+g_{mb}}$ |
| $R_D = \infty$ (理想电流源) | $R_{in} = \infty$ |

> 低输入阻抗使 CG 级适合做**宽带匹配**（如 50Ω 终端阻抗匹配）而无需电阻负载。

### 4. 共源共栅级 (Cascode)

#### Telescopic Cascode (Fig 3.59)

- $M_1$ (CS) 将 $V_{in}$ 转化为电流 → $M_2$ (CG) 将电流传导至 $R_D$
- 增益（$\lambda=0$）：$A_v = -g_{m1}R_D$，与普通 CS 相同

| 参数 | 表达式 |
|:--|:--|
| $G_m$ | $\approx g_{m1}$ (有 $r_{O1}$ 分流修正) |
| $R_{out}$ | $[1+(g_{m2}+g_{mb2})r_{O2}]r_{O1} + r_{O2} \approx (g_{m2}+g_{mb2})r_{O2} \cdot r_{O1}$ |
| $A_v$ (电流源负载) | $\approx g_{m1} \cdot [(g_{m2}+g_{mb2})r_{O2}r_{O1} \parallel (g_{m3}+g_{mb3})r_{O3}r_{O4}]$ |
| 输出摆幅 | $V_{DD}$ 减去 **4 个过驱动电压**（双 cascode） |

> **核心优势**：输出阻抗被放大约 $(g_{m2}+g_{mb2})r_{O2}$ 倍——接近**本征增益的平方**。
> 
> **代价**：输出电压摆幅至少损失一个 $V_{DS,sat}$（stacking penalty）。

#### 阻抗变换视角

MOSFET 对两端阻抗的"变换"作用：

- **源端电阻向上看** (degenerated CS 的 $R_{out}$)：电阻被放大 $1+(g_m+g_{mb})r_O$ 倍
- **漏端电阻向下看** (CG 的 $R_{in}$)：电阻被缩小 $1+(g_m+g_{mb})r_O$ 倍

#### Cascode 的屏蔽特性 (Shielding)

Cascode 管将输出节点的电压波动隔绝，使输入管漏端几乎不受影响。这可以：
- 提高恒流源的输出阻抗
- 减小因 $V_{DS}$ 失配导致的电流失配（缩小因子 $\approx (g_m+g_{mb})r_O$）

> [!warning] 纳米工艺注意事项
> 在 40nm 工艺中，cascode 电流源的输出阻抗仅比单管略好（当 $V_X < 0.2V$ 时），因为短沟道器件的本征增益低。

#### Folded Cascode (Fig 3.74)

- 输入管和 cascode 管**不同类型**（PMOS-NMOS 或 NMOS-PMOS）
- 小信号电流被"折叠"上或下
- **输出阻抗低于 telescopic**（因为 $r_{O3}$ 并联）
- 总偏置电流 = $I_{D1} + I_{D2} = I_1$，功耗更高

#### Poor Man's Cascode

- 省略 cascode 管的偏置电压 → $M_2$ 在线性区
- 等效于两倍沟道长度的单管，**不是真正的 cascode**
- 使用不同阈值电压可使其正常工作

---

## 重要结构速查

| 拓扑 | 输入节点 | 输出节点 | 增益极性 | 典型 $A_v$ |
|:--|:--|:--|:--|:--|
| **CS (电阻负载)** | Gate | Drain | 反相 (-) | $g_m R_D$ |
| **CS (电流源负载)** | Gate | Drain | 反相 (-) | $g_m(r_{O1}\parallel r_{O2})$ |
| **CS (有源负载)** | Gate | Drain | 反相 (-) | $(g_{m1}+g_{m2})(r_{O1}\parallel r_{O2})$ |
| **Source Follower** | Gate | Source | 同相 (+) | $\approx 1/(1+\eta)$ |
| **Common-Gate** | Source | Drain | 同相 (+) | $(g_m+g_{mb})R_D$ |
| **Cascode (telescopic)** | Gate | Drain | 反相 (-) | $g_{m1} \cdot g_{m2}r_{O2}r_{O1}$ |
| **Folded Cascode** | Gate | Drain | 反相 (-) | 低于 telescopic |

---

## 设计启示

1. **增益-摆幅-速度三元折中**：$|A_v| = g_m R_D = \frac{2I_D}{V_{GS}-V_{TH}} \cdot \frac{V_{RD}}{I_D} = \frac{2V_{RD}}{V_{GS}-V_{TH}}$。增加 $V_{RD}$ 提高增益但压缩摆幅；减小 $I_D$ 需增大 $R_D$ 导致输出极点更低。

2. **非线性最小化策略**：
   - 二极管连接负载：平方律→平方根抵消，$\propto \sqrt{(W/L)_1/(W/L)_2}$
   - 源极退化：$g_m R_S \gg 1$ 时 $G_m \approx 1/R_S$（线性）

3. **最大化增益的路径**：
   - 增大 $R_D$：受限直流压降
   - 电流源负载：$r_O$ 可独立于电压裕度增大（增加沟道长度 $L$）
   - Cascode：输出阻抗提高到 $(g_m r_O) \times r_O$

4. **模型选择策略（Sec 3.7）**：
   - 第一遍：饱和区用理想 $g_m V_{GS}$ 模型
   - 漏极接高阻抗节点 → 加 $r_O$
   - 源/体非交流地 → 加 $g_{mb}$
   - 偏置计算初遍可忽略 $\lambda, \gamma$

5. **选择合适的负载**：
   - 追求精度 → 电阻或电流源负载
   - 追求线性 → 二极管连接负载或源极退化
   - 追求高增益 + 大摆幅 → 电流源负载
   - 追求极高增益 → Cascode + Cascode 负载

---

## 章节关联

- **Next**: [[Ch04 - Differential Amplifiers]] — 差分放大器将本章的单端拓扑扩展为差分
- **Next**: [[Ch05 - Current Mirrors and Biasing Techniques]] — 为本章的放大器提供稳定的偏置电流和电压
- **Next**: [[Ch06 - Frequency Response of Amplifiers]] — 本章的低频分析加上电容效应，得到频率响应
- **Back**: Ch02 — 本章依赖的 MOS 小信号模型 ($g_m, r_O, g_{mb}$)

---

## 检索关键词

`CS stage`, `common-source`, `source follower`, `common-drain`, `common-gate`, `cascode`, `folded cascode`, `diode-connected load`, `active load`, `source degeneration`, `intrinsic gain`, `gm*rO`, `output impedance boosting`, `shielding`, `analog design octagon`, `Gm*Rout lemma`, `voltage swing`, `level shift`

---

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd]]

## See Also

- [[Ch04 - Differential Amplifiers]]
- [[Ch06 - Frequency Response of Amplifiers]]
- [[Ch02 - Basic MOS Device Physics]]
