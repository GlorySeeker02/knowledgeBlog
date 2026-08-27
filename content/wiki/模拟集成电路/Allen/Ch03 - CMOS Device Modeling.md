---
title: "Ch03 - CMOS Device Modeling"
source: "Allen-CMOS analog circuit design 3e"
tags: [analog-design, CMOS, device-modeling, MOSFET]
created: 2026-08-02
---

## 本章定位

本章是 Allen 模拟集成电路设计教材中 CMOS 器件建模的核心章节。全书后续所有电路分析与设计都依赖本章建立的模型体系。本章围绕 **SPICE 仿真框架**，从简单手算模型 (LEVEL 1) 出发，逐步引入短沟道效应、亚阈值效应，最终建立一套从直流大信号、交流小信号、噪声、电容到温度效应的完整 MOSFET 模型。作者强调：模型只是对真实器件的近似，设计者始终需要清楚所选模型的适用范围。

---

## 核心概念

### 模型层级分类

| 模型 | 适用工艺节点 | 用途 | 复杂度 |
|---|---|---|---|
| **LEVEL 1** (Sah / Shichman-Hodges) | $\ge 3\,\mu\mathrm{m}$ | 手算、设计直觉 | 低 |
| **LEVEL 3** | $\ge 0.8\,\mu\mathrm{m}$ | 计算机仿真 | 中 |
| **BSIM3v3** | 深亚微米 | 工业标准 | 高 |

### 三种工作区域

| 区域 | 条件 (NMOS) | 电流特性 |
|---|---|---|
| **截止区 (Cutoff)** | $v_{GS} - V_T \le 0$ | $i_D = 0$（忽略亚阈值） |
| **非饱和区/线性区/三极管区** | $0 \le v_{DS} \le v_{GS}-V_T$ | $i_D$ 同时受 $v_{GS}$ 和 $v_{DS}$ 控制 |
| **饱和区/有源区** | $v_{DS} \ge v_{GS}-V_T$ | $i_D$ 与 $v_{DS}$ 近似无关 |

> Gray 第四版将 "saturation" 区域改称为 "active"，使 MOS 与 BJT 命名一致。

---

## 关键公式与结论

### LEVEL 1 大信号模型（Section 3.1）

**完整 Sah 方程（非饱和 + 饱和统一）：**

$$
i_D = \beta\Big[(v_{GS}-V_T)v_{DS} - \frac{v_{DS}^2}{2}\Big](1+\lambda v_{DS}),\quad v_{DS} \le v_{DS}(\text{sat})
$$

其中 $\beta = K' \frac{W}{L} = \mu_0 C_{ox} \frac{W}{L}$。

**饱和电压：**

$$
v_{DS}(\text{sat}) = v_{GS} - V_T
$$

**饱和区电流（含沟道长度调制）：**

$$
i_D = \frac{\beta}{2}(v_{GS}-V_T)^2(1+\lambda v_{DS})
$$

**阈值电压（n 沟道）：**

$$
V_T = V_{T0} + \gamma\left(\sqrt{2\phi_F + v_{SB}} - \sqrt{2\phi_F}\right)
$$

**过驱动电压 $V_{ON}$：**

$$
V_{ON} = v_{GS} - V_T = \sqrt{\frac{2i_D}{\beta}} = v_{DS}(\text{sat})
$$

### 速度饱和效应

包含速度饱和的修正模型：

$$
i_D = \frac{\beta(v_{GS}-V_T)^2}{2[1 + \theta(v_{GS}-V_T)]},\quad\Big(0.5\theta(v_{GS}-V_T) < 1\Big)
$$

速度饱和也可以等效为源极串联退化电阻 $R_{SX}$：

$$
R_{SX} = \frac{1}{K' \cdot W \cdot E_c}
$$

其中 $E_c$ 为临界电场。物理含义：$\theta \neq 0$ 使跨导特性由平方律向线性过渡。

### 小信号模型参数（Section 3.3）

| 参数 | 饱和区表达式 | 与偏置关系 |
|---|---|---|
| **跨导 $g_m$** | $\sqrt{2K' \frac{W}{L} I_D}$ | $\propto \sqrt{I_D}$ |
| | $K' \frac{W}{L}(V_{GS}-V_T)$ | $\propto V_{ON}$ |
| **体跨导 $g_{mbs}$** | $\frac{\gamma g_m}{2\sqrt{2\phi_F + V_{SB}}}$ | — |
| **沟道电导 $g_{ds}$** | $\lambda I_D$ | $\propto I_D$ |

**$g_m$ 的多种等效表达：**

$$
g_m = \sqrt{2\beta I_D} = \frac{2I_D}{V_{GS}-V_T} = \beta(V_{GS}-V_T)
$$

### 小信号参数典型值（$I_D = 50\,\mu\mathrm{A}$，$W/L = 1\,\mu\mathrm{m}/1\,\mu\mathrm{m}$）

| 参数 | NMOS | PMOS |
|---|---|---|
| $g_m$ | 105 μA/V | 70.7 μA/V |
| $g_{mbs}$ | 12.8 μA/V | 12.0 μA/V |
| $g_{ds}$ | 2.0 μA/V | 2.5 μA/V |

### 非饱和区小信号参数

| 参数 | 表达式 |
|---|---|
| $g_m$ | $\beta\,V_{DS}$ |
| $g_{mbs}$ | $\frac{\gamma\beta V_{DS}}{2\sqrt{2\phi_F + V_{SB}}}$ |
| $g_{ds}$ | $\beta(V_{GS}-V_T-V_{DS})$ |

---

## 器件模型

### 完整大信号模型（Fig. 3.2-1）

包含如下组件：
- **漏源电流源 $i_D$**（由操作区域决定）
- **漏/源体二极管** (p-n 结，正常工作时反偏，仅贡献漏电流)
- **漏/源欧姆电阻 $r_D, r_S$**（典型 50-100 Ω，silicided 工艺约 5-10 Ω）
- **三种电容：**
  - 结耗尽电容 $C_{BD}, C_{BS}$
  - 栅极电荷存储电容 $C_{GD}, C_{GS}, C_{GB}$（与工作区域有关）
  - 寄生交叠电容

### 电容模型

#### 结耗尽电容（分段模型）

$$
C_{BX} = \begin{cases} \dfrac{C_{BX0}}{\left(1 + \dfrac{v_{BX}}{PB}\right)^{MJ}}, & v_{BX} < FC \cdot PB \\ \dfrac{C_{BX0}}{(1-FC)^{1+MJ}}\left(1 - FC(1+MJ) + MJ\dfrac{v_{BX}}{PB}\right), & v_{BX} \ge FC \cdot PB \end{cases}
$$

其中 $X = S$ 或 $D$。实际建模中分为底面 (bottom) 和侧壁 (sidewall) 两部分：

$$
C_{BX} = \frac{CJ \cdot AX}{(1 + v_{BX}/PB)^{MJ}} + \frac{CJSW \cdot PX}{(1 + v_{BX}/PB)^{MJSW}}
$$

#### 栅极电荷存储电容（与工作区有关）

| 区域 | $C_{GS}$ | $C_{GD}$ | $C_{GB}$ |
|---|---|---|---|
| **截止** | $C_1$ (重叠) | $C_3$ (重叠) | $C_2 + 2C_5$ |
| **饱和** | $C_1 + \frac{2}{3}C_2$ | $C_3$ | $2C_5$ |
| **非饱和** | $C_1 + \frac{1}{2}C_2$ | $C_3 + \frac{1}{2}C_2$ | $2C_5$ |

其中 $C_2 = C_{ox}WL_{eff}$（栅-沟道电容），$C_1, C_3$ 为覆盖电容 $CGSO \cdot W_{eff}$、$CGDO \cdot W_{eff}$，$C_5$ 为栅-体交叠电容 $CGBO \cdot L_{eff}$。

> 物理直觉：饱和时沟道在漏端夹断，故 $C_{GD}$ 仅剩交叠电容；非饱和时沟道贯通源漏，$C_2$ 平分给 $C_{GS}$ 和 $C_{GD}$。

### 噪声模型

漏电流噪声由**热噪声**和**闪烁噪声（$1/f$ 噪声）**组成：

$$
\overline{i_{nD}^2} = \left[\frac{8kT g_m(1+\eta)}{3} + \frac{KF \cdot I_D^{AF}}{C_{ox}L_{eff}^2 \cdot f}\right]\Delta f
$$

将其折算为等效输入电压噪声（源极交流接地）：

$$
\overline{e_{n}^2} = \frac{\overline{i_{nD}^2}}{g_m^2} \approx \frac{B}{WLf} \quad(\text{低频时 } 1/f \text{ 噪声主导})
$$

- KF 典型值 $\sim 10^{-28}$ F-A
- 实测表明：$1/f$ 噪声在 100 kHz 以下为主导（取决于偏置条件）
- PMOS 的 $1/f$ 噪声通常比 NMOS 低（对低噪声设计有利）

### 温度模型

- **跨导温度系数**：$K'(T) = K'(T_0)(T/T_0)^{-1.5}$
- **阈值电压温度系数**：$V_T(T) = V_T(T_0) + \alpha(T-T_0)$，其中 $\alpha$ 对 NMOS 为负，PMOS 为正
- **零温度系数点 (ZTC)**：存在一个 $V_{GS}$ 使 $\partial I_D/\partial T = 0$

$$
V_{GS}(\mathrm{ZTC}) = V_T(T_0) + \frac{\alpha}{1.5/T_0}
$$

> 当 $V_{GS} > V_{GS}(\mathrm{ZTC})$ 时，$I_D$ 随温度升高而降低——MOSFET 比 BJT 具有更好的热稳定性。

### LEVEL 3 模型要点（Section 3.4）

适用于小尺寸器件（$\ge 0.8\,\mu\mathrm{m}$），在 LEVEL 1 基础上增加：

- **窄沟道/短沟道阈值电压修正**
- **迁移率退化**（$\mu_{eff}$ 随垂直电场下降）：`THETA`
- **速度饱和**（`VMAX`）
- **DIBL**（漏致势垒降低）影响阈值电压
- **静态反馈系数**（`ETA`）影响饱和电压
- **改进的沟道长度调制**（`KAPPA`）

### BSIM3v3 模型特性（Section 3.4）

工业标准模型，约 40 个 DC 参数，主要覆盖：

- 阈值电压降低
- 垂直电场引起的迁移率退化
- 速度饱和
- 漏致势垒降低 (DIBL)
- 沟道长度调制
- 源漏寄生电阻
- 亚阈值 (弱反型) 导电
- 热电子效应影响输出电阻

图 3.4-2 对比了 LEVEL 1、LEVEL 3、BSIM3v3 对同一 20/0.8 器件的仿真差异，LEVEL 1 与 BSIM3v3 偏差巨大，LEVEL 3 匹配较好但在非饱和-饱和过渡区仍有差异。

### 亚阈值模型（Section 3.5）

当 $v_{GS} < V_T$ 时，MOS 管并非完全截止，而是进入弱反型区，电流呈**指数关系**：

**SPICE LEVEL 3 模型：**
$$
i_D = i_{DS} \cdot \exp\left(\frac{v_{GS} - V_{ON}}{n kT/q}\right),\quad v_{GS} \le V_{ON}
$$

其中 $V_{ON} = V_T + n kT/q$。

**手算简化模型：**
$$
i_D = I_{D0}\frac{W}{L}\exp\left(\frac{v_{GS}}{n kT/q}\right)
$$

- $n$ 为亚阈值斜率因子，通常 $1 < n < 3$
- 弱反型跨导 $g_m = I_D/(n kT/q)$，跨导效率高于强反型
- 三个操作区：**弱反型 → 中度反型 → 强反型**，过渡区建模最为困难

> 亚阈值区的温度特性：$V_T$ 负温系数使同 $V_{GS}$ 下 $I_D$ 随温度升高——与强反型相反。

---

## 设计启示

1. **$g_m/I_D$ 效率**：强反型 $g_m/I_D = 2/V_{ON}$，弱反型 $g_m/I_D = q/(nkT) \approx 25$ —— 弱反型以更小的电流获得同等跨导，适合低功耗设计。
2. **Body Effect 不可忽视**：$V_{SB}$ 增加使 $V_T$ 升高，影响偏置和动态范围，共源共栅和叠管结构中尤为显著。
3. **PMOS 噪声更优**：低噪声前置放大器优选 PMOS 输入对。
4. **速度饱和改变设计规则**：深亚微米下 $i_D$ 与 $V_{GS}$ 趋近线性，传统平方律直觉需要修正。
5. **SPICE 仿真准则（十诫）**：
   - 仿真前必须预知答案大致范围
   - 只仿真必要的电路部分
   - 始终从最简单的模型开始
   - DC 收敛：让大多数器件从导通状态开始
   - 一次只改变一个参数
   - 使用正确量纲乘数（M, U, N, P, F）
   - 区分字母 O 和数字 0

---

## 章节关联

- **前置**：[[../../../raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e|Ch02]] — MOS 器件物理基础（能带、阈值电压推导、平带电压、体效应、栅氧化层电容）
- **后继**：Ch04 — 模拟 CMOS 子电路（偏置、电流镜、差分对等），核心小信号模型 $g_m, r_o$ 即本章所得
- **交叉引用**：
  - Razavi Ch02 — MOS 器件物理与模型（大/小信号、速度饱和、弱反型）
  - Razavi Ch17 — 与开关电容电路相关的器件噪声与匹配
  - Gray Ch01 — 器件物理基础（包含 $g_m r_o$ 本征增益等设计指标）

---

## 检索关键词

`MOSFET 大信号模型` `LEVEL 1` `LEVEL 3` `BSIM3v3` `小信号模型` `$g_m$` `$g_{ds}$` `$g_{mbs}$` `体效应` `沟道长度调制` `速度饱和` `亚阈值` `弱反型` `闪烁噪声` `$1/f$ 噪声` `热噪声` `ZTC` `零温度系数` `栅电容` `覆盖电容` `耗尽电容` `SPICE 仿真` `.MODEL` `NMOS` `PMOS` `VTO` `KP` `GAMMA` `LAMBDA` `PHI` `THETA` `ETA` `KAPPA` `NFS`

Sources: [[raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e]]

See Also: Razavi Ch02/Ch17, Gray Ch01
