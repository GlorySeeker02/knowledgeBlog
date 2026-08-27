---
title: Ch02 - Basic MOS Device Physics
source: Razavi-Design of Analog CMOS Integrated Circuits 2nd
tags:
  - analog-design
  - CMOS
  - device-physics
  - MOSFET
  - Razavi
---

# Ch02 - Basic MOS Device Physics

## 本章定位

本章是模拟 CMOS 电路设计的物理基础，目标是为电路分析提供**简化的器件模型**而非全面的器件物理。Razavi 从 MOS 结构出发推导 I/V 特性，逐步引入二阶效应（体效应、沟道长度调制、亚阈值导通），最后建立小信号模型和 SPICE Level 1 模型。第 3-14 章的电路分析都基于本章的平方律模型展开，第 17 章回头处理短沟道器件的精细效应。

## 2.1 核心概念

### 2.1.1 MOSFET 结构与阈值电压

**NMOS 结构四要素**：p 型衬底、n+ 源漏区、多晶硅栅极、栅氧化层 (SiO₂)。

- **有效沟道长度**：$L_{\text{eff}} = L_{\text{drawn}} - 2L_D$，侧向扩散使实际沟道比绘制的短。
- **阈值电压 $V_{TH}$** 的物理来源：栅极电压需先排斥 p-sub 中的空穴形成耗尽层，再吸引电子形成反型层（沟道）。定义式为界面"与衬底同浓度反型"时的栅压。
- **耗尽层电荷**：$Q_{\text{dep}} = \sqrt{4q\epsilon_{si}|\phi_F|N_{\text{sub}}}$，$\phi_F = (kT/q)\ln(N_{\text{sub}}/n_i)$。
- **氧化层单位电容**：$C_{ox} = \epsilon_{ox}/t_{ox}$，记忆：$t_{ox} \approx 20\text{Å}$ 时 $C_{ox} \approx 17.25\text{ fF/}\mu\text{m}^2$。
- **源漏可互换**（器件对称），定义"提供载流子"的为源端、"收集载流子"的为漏端。

### 2.1.2 三种工作区

| 工作区 | 条件 (NFET) | 物理图像 |
|---|---|---|
| **截止** | $V_{GS} < V_{TH}$ | 无沟道，$I_D \approx 0$ |
| **线性/三极管区** | $V_{GS} > V_{TH},\; V_{DS} < V_{GS} - V_{TH}$ | 沟道从 S 连续延伸到 D |
| **饱和区** | $V_{GS} > V_{TH},\; V_{DS} \geq V_{GS} - V_{TH}$ | 漏端沟道夹断 (pinch-off) |

- 夹断的直观判断：不必知道源电压，只需看 **$V_G - V_D$ (NFET) / $V_D - V_G$ (PFET)** 是否小于阈值。
- **深三极管区**：$V_{DS} \ll 2(V_{GS} - V_{TH})$ 时，沟道可视为线性电阻 $R_{\text{on}} = \frac{1}{\mu_n C_{ox} \frac{W}{L}(V_{GS} - V_{TH})}$。

### 2.1.3 三种二阶效应

| 效应 | 数学表达 | 物理成因 | 对模拟设计的影响 |
|---|---|---|---|
| **体效应** | $V_{TH} = V_{TH0} + \gamma\left(\sqrt{2\phi_F + V_{SB}} - \sqrt{2\phi_F}\right)$ | $V_{SB}$ 增大会扩展耗尽层，提高 $V_{TH}$ | 源跟随器增益 < 1，共源级体跨导退化 |
| **沟道长度调制** | $I_D = \frac{1}{2}\mu_n C_{ox}\frac{W}{L}(V_{GS}-V_{TH})^2(1+\lambda V_{DS})$ | 夹断点随 $V_{DS}$ 移动，有效沟道变短 | 有限 $r_O$ 限制放大器本征增益 |
| **亚阈值导通** | $I_D = I_0\exp\frac{V_{GS}}{\xi V_T}$，$V_T = kT/q$ | 弱反型层仍可导电，电流随 $V_{GS}$ 指数衰减 | 关断不彻底、大芯片泄漏功耗大；超低功耗设计可利用亚阈值区 |

- **体效应系数** $\gamma = \sqrt{2q\epsilon_{si} N_{\text{sub}}}/C_{ox}$，典型 0.3--0.4 V$^{1/2}$。
- **体效应不总是坏事**：可以通过 **正向体偏置** ($V_{SB} < 0$) 降低阈值，在低电压设计中利用（PFET 的独立 n-well 易于实现）。
- **亚阈值摆幅**：$V_{GS}$ 每降低约 80 mV（$\xi \approx 1.5$），$I_D$ 下降一个 decade。
- **强反型与弱反型的过渡**：在 $(V_{GS}-V_{TH}) \approx 2\xi V_T \approx 80\text{ mV}$ 处交叉。

## 2.2 关键公式与结论

### I/V 特性（平方律模型，长沟道）

**NFET 三极管区**：
$$
I_D = \mu_n C_{ox} \frac{W}{L}\left[(V_{GS} - V_{TH})V_{DS} - \frac{1}{2}V_{DS}^2\right]
$$

**NFET 饱和区**（$V_{DS} \geq V_{GS} - V_{TH}$）：
$$
I_D = \frac{1}{2}\mu_n C_{ox} \frac{W}{L}(V_{GS} - V_{TH})^2(1 + \lambda V_{DS})
$$

**PFET**：将 $\mu_n \to \mu_p$，所有电压取负号，$I_D$ 约定从漏极流向源极。

### 跨导 $g_m$（饱和区）——三种等价形式

$$
g_m = \mu_n C_{ox}\frac{W}{L}(V_{GS} - V_{TH}) \quad \text{(正比于过驱动})
$$
$$
g_m = \sqrt{2\mu_n C_{ox}\frac{W}{L} I_D} \quad \text{(给定电流，增大W/L提高} g_m\text{)}
$$
$$
g_m = \frac{2I_D}{V_{GS} - V_{TH}} \quad \text{(给定电流，减小过驱动提高} g_m\text{)}
$$

> **亚阈值跨导**：$g_m = I_D/(\xi V_T)$，永远小于同电流下 BJT 的 $I_C/V_T$。

**重要关系**：$g_m$ 在饱和区的值 = $1/R_{\text{on}}$（深三极管区导通电阻的倒数）。

### 输出电阻

$$
r_O = \frac{\partial V_{DS}}{\partial I_D} \approx \frac{1}{\lambda I_D}
$$

- **本征增益** $g_m r_O \propto 1/\sqrt{I_D}$，长沟道模型 $g_m r_O$ 很大但实际短沟道远低于理论值（第 17 章详述）。
- 沟道长度翻倍 → 斜率 ÷4（给定 $V_{GS} - V_{TH}$）。

### 体跨导

$$
g_{mb} = \frac{g_m \gamma}{2\sqrt{2\phi_F + V_{SB}}} = \eta g_m
$$

$\eta \approx 0.25$（典型值），$V_{SB}$ 增大 → $g_{mb}$ 减小。

## 2.3 重要结构与模型

### 2.3.1 器件电容

| 电容 | 截止区 | 深三极管区 | 饱和区 |
|---|---|---|---|
| $C_{GS}$ | $WC_{ov}$ | $\frac{1}{2}WLC_{ox} + WC_{ov}$ | $\frac{2}{3}WLC_{ox} + WC_{ov}$ |
| $C_{GD}$ | $WC_{ov}$ | $\frac{1}{2}WLC_{ox} + WC_{ov}$ | $WC_{ov}$ |
| $C_{GB}$ | $\frac{WLC_{ox} \cdot C_d}{WLC_{ox} + C_d}$ | 可忽略 | 可忽略 |
| $C_{SB}, C_{DB}$ | $C_j A + C_{jsw} P$（与偏压相关） | ← | ← |

- **交叠电容** $C_{ov}$ 是 gate poly 与 S/D 区域的重叠，单位宽度 (F/m)。
- **结电容** = 底面积电容 $C_j$ × 面积 + 侧壁电容 $C_{jsw}$ × 周长，均与反偏电压相关：$C_j = C_{j0} / [1 + V_R/\phi_B]^m$。
- **Fold 结构**：将 $W$ 拆为两条 $W/2$ 并联，漏结电容 + 栅电阻均减少 4 倍。

### 2.3.2 小信号模型（低频 + 高频）

**低频核心**：
- $g_m v_{gs}$：栅压→漏流的跨导电流源
- $r_O = 1/(\lambda I_D)$：沟道长度调制 → 并联电阻
- $g_{mb} v_{bs}$：体效应 → 第二栅极，极性与 $g_m v_{gs}$ 相同

**高频完整模型**：在上述低频模型上并联 5 个电容：$C_{GS}, C_{GD}, C_{GB}, C_{SB}, C_{DB}$。

> **NMOS 与 PMOS 小信号模型完全相同**，不因器件类型改变电流源方向。

### 2.3.3 SPICE Level 1 模型参数速查

| 参数 | 含义 | 典型值 (NMOS) |
|---|---|---|
| VTO | $V_{TH0}$ | 0.7 V |
| GAMMA | $\gamma$ | 0.45 V$^{1/2}$ |
| PHI | $2\phi_F$ | 0.9 V |
| UO | $\mu_n$ | 350 cm²/V/s |
| LAMBDA | $\lambda$ | 0.1 V$^{-1}$ |
| TOX | $t_{ox}$ | $9 \times 10^{-9}$ m |
| CJ / CJSW | 结电容 | 0.56 mF/m² / 0.35 nF/m |
| CGDO | 栅漏交叠电容/宽度 | 0.4 nF/m |
| LD | 侧向扩散长度 | 0.08 μm |

PMOS 的 $\mu_p \approx 0.5\mu_n$，$\lambda$ 约为 NMOS 两倍（输出电阻更低）。

### 2.3.4 FinFET 关键概念

- 等效宽度 $W = W_F + 2H_F$（三面导电：顶面 + 两侧壁）。
- **宽度量化**：只能按鳍数 (fin count) 取离散值（如每鳍 100 nm），不能任意调整 $W$。
- **优势**：I/V 特性更接近平方律行为，沟道长度调制和亚阈值泄漏更小。
- **代价**：需深 n-well 隔离不同 NFET 时面积开销大。

### 2.3.5 栅氧作为电容的 C-V 曲线

| 工作模式 | $V_{GS}$ 范围 | 单位面积电容 |
|---|---|---|
| 积累区 | $V_{GS} \ll 0$ | $C_{ox}$ |
| 耗尽/弱反型 | $0 < V_{GS} < V_{TH}$ | $C_{ox}$ 与 $C_{dep}$ 串联 (最小) |
| 强反型 | $V_{GS} > V_{TH}$ | $C_{ox}$ |

## 2.4 设计启示

1. **过驱动电压的权衡**：$V_{GS} - V_{TH}$ 决定 $V_{D,\text{sat}}$（电压裕度）和 $g_m$。大过驱动 → 高速但小裕度，小过驱动 → 高 $g_m/I_D$ 效率但速度低。
2. **$W/L$ 是核心设计自由度**：给定电流下，大 $W/L$ → 低 $V_{GS} - V_{TH}$ → 可能进入亚阈值区（$g_m$ 由 $I_D/\xi V_T$ 决定，不再随 $W$ 增长）。
3. **长沟道 = 更理想电流源**：$r_O$ 大且线性度高，但需等比例增大 $W$ 维持 $I_D$，消耗更大面积。
4. **体效应恶化源跟随器性能**：$V_{out}$ 升高 → $V_{SB}$ 增大 → $V_{TH}$ 升高 → 增益进一步下降。
5. **PFET 弱于 NFET**：$\mu_p \approx 0.5\mu_n$，同尺寸下 $g_m$ 和 $r_O$ 均较差。优先用 NFET 做关键路径（除了 flicker noise 考量）。
6. **折叠结构 (folding) 降寄生**：适用于高频和低噪声设计，栅电阻和漏结电容均大幅降低。

## 2.5 章节关联

| 方向 | 涉及章节 | 关系 |
|---|---|---|
| **前置** | 第 2 章（本章） | 建立器件模型 |
| **后序直接依赖** | 第 3 章 单级放大器 | 全部电路分析基于 $g_m, r_O, g_{mb}$ 和 I/V 方程 |
| **后序进阶** | 第 17 章 短沟道效应 | 修正平方律模型，引入速度饱和、迁移率退化、漏致势垒降低 (DIBL) 等 |
| **工艺** | 第 18 章 制造工艺 | MOSFET 的物理实现与 layout 细节 |
| **应用** | 第 13 章 开关电路 | MOSFET 作为开关使用 ($R_{\text{on}}$ 模型) |

## 检索关键词

`阈值电压`, `VTH`, `跨导`, `gm`, `体效应`, `body effect`, `沟道长度调制`, `channel-length modulation`, `亚阈值`, `subthreshold`, `平方律`, `square-law`, `小信号模型`, `small-signal model`, `输出电阻`, `rO`, `FinFET`, `过驱动电压`, `overdrive voltage`, `SPICE Level 1`, `结电容`, `junction capacitance`, `氧化层电容`, `Cox`

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]

## See Also

- Ch03 Single-Stage Amplifiers — 本章 $g_m$, $r_O$, $g_{mb}$ 模型的直接应用
- Ch17 Short-Channel Effects — 修正及扩展本章的长沟道模型
- Ch18 CMOS Fabrication — 器件的物理实现
