---
title: "Ch03 单管与多管放大器"
source: "Gray, Hurst, Lewis, Meyer — Analysis and Design of Analog Integrated Circuits, 5th Ed., Chapter 3"
tags:
  - analog-design
  - analog-IC
  - amplifiers
  - single-stage
  - differential-pair
date: 2026-08-02
---

# 第三章 单管与多管放大器

## 本章定位

第三章是 Gray 教材从器件模型迈向实际电路设计的**第一个桥梁章节**。第一章建立了小信号模型，第二章介绍了工艺，而本章系统性地讲解了用这些器件能搭出哪些基本放大结构、它们的性能如何计算、以及不同结构之间如何组合。全章围绕三个层次展开：

1. **单管放大器** — CE/CB/CC (BJT) 与 CS/CG/CD (MOS) 六种基本组态
2. **多管组合放大器** — Darlington、Cascode、Active Cascode、Super Source Follower
3. **差分对** — 大信号传输特性、小信号半电路分析、失调与失配

本章的分析工具是**低频小信号两端口模型**，以 `Ri`、`Ro`、`Gm`（或 `av`）三个参数刻画放大器的低频行为。

---

## 核心概念

### 1. 两端口建模

放大器用两端口网络抽象以解耦源阻抗和负载阻抗的影响。

- **导纳参数 (y-parameter):** 电流用电压表示，`i1 = y11·v1 + y12·v2`, `i2 = y21·v1 + y22·v2`
- `y21` = 短路跨导 **`Gm`**，`y11` 和 `y22` 的倒数即输入/输出阻抗
- **单向化 (unilateral):** 当 `y12 = 0`，前馈为主要路径，分析大大简化
- **Norton ↔ Thevenin 转换:** `av = -Gm·Zo`
- **加载效应:** 源阻抗分压 + 负载分压，总增益 = `(Ri/(RS+Ri)) · av · (RL/(Ro+RL))`

> **hand analysis 的核心原则:** 选**最简单但仍能抓到关键物理效应的模型**。手算的目标是建立直觉，精确值留给仿真。

### 2. 单管放大器六种基本组态

| 组态 | 输入 | 输出 | 公共端 |
|------|------|------|--------|
| CE / CS | Base / Gate | Collector / Drain | Emitter / Source |
| CB / CG | Emitter / Source | Collector / Drain | Base / Gate |
| CC / CD | Base / Gate | Emitter / Source | Collector / Drain |

- **BJT vs MOS 核心差异:**
  - MOS 栅极输入阻抗 `Ri → ∞`；BJT `Ri = rπ = β0/gm`
  - 相同偏置电流下 BJT 的 `gm` 比 MOS 大约一个数量级
  - 因此 BJT 更容易获得高增益，MOS 更容易获得高输入阻抗
  - MOS 的体效应 (`gmb`) 对 CG 和 CD 组态有显著影响

### 3. T 模型与 CB/CG 分析

CB/CG 组态使用 hybrid-π 模型分析不便（受控源跨接在输入输出之间），因此转换为 **T 模型**:

- 集电极到发射极的 `gm·v1` 电流源拆成两个：一个从集电极到基极，一个从基极到发射极
- 基极-发射极间的受控源由其自身电压控制 → 等效为一个电阻 `re = 1/gm`
- 并联 `re ∥ rπ` 即为发射极电阻（BJT），或 `1/(gm+gmb)` 为源极电阻（MOS）

### 4. 单向化与反馈

- **CE/CS:** 忽略 `rμ` 和 `Cμ` 时是**单向**的（`y12 = 0`）
- **CB/CG:** `ro` 有限时是**双向**的——输入阻抗依赖输出端负载，输出阻抗依赖输入端源阻抗
- **CC/CD:** 由于输出跟随输入，存在**负反馈**，本质上不是单向的，因此直接分析整个电路而非抽取两端口参数更有意义
- **退化 (degeneration):** `RE` 或 `RS` 引入串联-串联负反馈，降低 `Gm`，提高 `Ro`，扩展线性范围

### 5. 差分对

差分对是**模拟 IC 中使用最广泛的两管子电路**，核心价值：

1. **直接耦合:** 级联无需隔直电容
2. **共模抑制:** 对差分信号敏感，对共模信号不敏感

#### 5.1 半电路分析法

对称差分放大器可通过**半电路**分解为两个独立单端电路：

- **差分半电路:** 尾电流源接地短路（因对称，差分激励时尾节点电压不变）
- **共模半电路:** 尾电流源等效为 `2·RTAIL` 接地的退化电阻（两半之间无电流，可断开）
- 总输出由叠加得到

#### 5.2 CMRR

$$ \text{CMRR} = \left|\frac{A_{dm}}{A_{cm}}\right| $$

- 理想对称时 `Acm-dm = 0` 且 `Adm-cm = 0`
- 对于射极耦合对：$\text{CMRR} \approx 1 + 2g_m R_{\text{TAIL}}$
- 对于源极耦合对：$\text{CMRR} \approx 1 + 2(g_m + g_{mb})R_{\text{TAIL}}$
- **提高 `RTAIL` 是改善 CMRR 的关键途径**（第 4 章将详细讨论如何用晶体管电流源实现高 `RTAIL`）

### 6. 失调与失配

#### 6.1 输入失调电压 (BJT)

$$ V_{OS} = V_T \left( -\frac{\Delta R_C}{R_C} - \frac{\Delta I_S}{I_S} \right) $$

- 电阻失配和饱和电流失配通过 `VT` (26mV) 缩放
- 失调 ≈ 失配百分比 × `VT`
- **失调漂移:** `dVOS/dT = VOS/T`，失调与漂移成正比

#### 6.2 输入失调电压 (MOS)

$$ V_{OS} = \Delta V_t + \frac{V_{ov}}{2} \left( -\frac{\Delta R_D}{R_D} - \frac{\Delta (W/L)}{W/L} \right) $$

- **阈值失配项** (`ΔVt`) 是 MOS 特有且最主要的失调来源
- 几何/负载失配项缩放因子是 `Vov/2`（典型 50–500mV），远大于 BJT 的 `VT` (26mV)
- 因此 MOS 差分对的失调电压通常比 BJT 差约一个数量级
- **失调漂移:** MOS 的 `ΔVt` 项和 `Vov` 项温度系数不同，没有 BJT 那种 `VOS ∝ T` 的简单关系

#### 6.3 输入失调电流 (BJT only)

$$ I_{OS} \approx \frac{I_C}{\beta_0} \left( \frac{\Delta R_C}{R_C} - \frac{\Delta \beta_0}{\beta_0} \right) $$

- MOS 栅极输入电流为零，几乎无失调电流

#### 6.4 失配差分对的小信号分析

失配使 `Acm-dm` 和 `Adm-cm` 不再为零。使用**耦合半电路**模型：

- 差分半电路中，共模电流通过 `ΔR` 和 `Δgm` 控制受控源，产生差分输出
- 共模半电路中，差分信号同理耦合回共模
- 当失配较小时，可以先从无失配半电路估算控制信号（`iRc`、`v` 等），再代入含失配的半电路，得到近似闭合表达式

---

## 关键公式与结论

### 单管放大器参数速查

| 参数 | CE | CB | CC |
|------|-----|-----|-----|
| `Ri` | `rπ` | `re ≈ 1/gm` | `rπ + (β0+1)(ro∥RL)` |
| `Ro` | `ro∥RC` | `ro(1+gm·RS∥rπ)` | `(1/gm) + RS/(β0+1)` |
| `Gm` | `gm` | `gm` | ≈ `1/(1/gm + RL)` |
| `av` | `-gm(ro∥RC)` | `+gm(ro∥RC)` | ≈ 1 |
| `ai` | `β0` | `α0 ≈ 1` | `β0+1` |

| 参数 | CS | CG | CD |
|------|-----|-----|-----|
| `Ri` | `∞` | `1/(gm+gmb)` | `∞` |
| `Ro` | `ro∥RD` | 见下 | `1/(gm+gmb)` |
| `Gm` | `gm` | `gm+gmb` | — |
| `av` | `-gm(ro∥RD)` | `+(gm+gmb)(ro∥RD)` | `(gm+gmb)ro/(1+(gm+gmb)ro) < 1` |

- **CG 输出电阻 (含有限 ro):** $R_o \approx r_o \left[1 + (g_m+g_{mb})R_S\right]$
- **CE 退化:** `Ri ≈ rπ(1+gmRE)`, `Gm ≈ gm/(1+gmRE)`, `Ro ≈ ro(1+gmRE)`
- **CS 退化:** `Gm ≈ gm/(1+(gm+gmb)RS)`, `Ro ≈ ro[1+(gm+gmb)RS] + RS`（MOS 的 `Ro` 随 `RS → ∞` 持续增大，不饱和；BJT 的 `Ro` 饱和于 `β0·ro`）

### 多管结构关键结论

**Cascode (CE-CB / CS-CG):**
- `Gm ≈ gm1`（几乎不变）
- `Ro(BJT) ≈ β0·ro2`（比单管高约 `β0` 倍）
- `Ro(MOS) ≈ (gm2+gmb2)·ro2·ro1`（比单管高约 `(gm+gmb)ro` 倍）
- MOS cascode 的 `Ro` 不饱和，可**多级级联**进一步提高（受限于电源电压和信号摆幅）

**Active Cascode:**
- 用一个放大器 `-a` 驱动 cascode 管的栅极
- 效果相当于将 cascode 管的 `gm2` 替换为 `(a+1)·gm2`
- `Ro ≈ [(a+1)gm2 + gmb2]·ro2·ro1`

**Darlington (CC-CE / CC-CC):**
- `βc ≈ β0²`, `rπc ≈ 2rπ`（`IBIAS=0` 时），`gmc ≈ gm2`
- 主要用于提升 BJT 的有效电流增益和输入阻抗，纯 MOS 电路无此需求

**Super Source Follower:**
- `Ro ≈ 1/[(gm1+gmb1)·gm2·ro1]`，比普通源跟随器低约 `gm2·ro1` 倍
- `av` 略低于普通源跟随器，但差距很小

### 差分对关键公式

- **BJT 差分输出:** $V_{od} = \alpha_F I_{\text{TAIL}} R_C \tanh\left(\frac{V_{id}}{2V_T}\right)$
- **BJT 线性范围:** $|V_{id}| < V_T \approx 26\text{mV}$
- **MOS 差分输出电流:** $\Delta I_d = \frac{1}{2} \mu_n C_{ox} \frac{W}{L} V_{id} \sqrt{\frac{2I_{\text{TAIL}}}{\mu_n C_{ox} (W/L)} - V_{id}^2}$
- **MOS 线性范围:** $|V_{id}| \le \sqrt{2} V_{ov}$（由过驱动电压决定，可调节）
- **差分半电路增益:** $A_{dm} = -g_m R$（CE/CS 半电路）
- **共模半电路增益:** $A_{cm} \approx -\frac{g_m R}{1+2g_m R_{\text{TAIL}}}$
- **差分输入电阻 (BJT):** $R_{id} = 2r_\pi$

---

## 重要电路结构

### 1. CE 放大器 + 射极退化
串联 `RE` 引入局部负反馈 → 降低增益 (`Gm ≈ gm/(1+gmRE)`)、提高输入/输出阻抗、扩展线性范围。代价是牺牲增益换取更好的线性度和更高的输出电阻。

### 2. 射极跟随器 / 源极跟随器
- **高输入阻抗、低输出阻抗、增益 ≈ 1**
- 用作阻抗变换器（buffer）或电平移位器
- MOS 源极跟随器的增益受**体效应**影响，通常 `0.8–0.9`
- 将 MOS 管做在独立阱中并让阱与源极相连可消除体效应，但会增加寄生电容降低带宽

### 3. Cascode 放大器
- CE-CB (BJT) 或 CS-CG (MOS)
- **核心目的:** 提高输出电阻、减小密勒反馈电容，从而获得高增益和高带宽
- Cascode 管隔离了输出节点对输入管 `ro` 的影响

### 4. 差分对 + 尾电流源
- 尾电流源提供差分模式下的**虚地**、共模模式下的**退化阻抗**
- 用电阻做尾电流源 → CMRR 有限
- 用晶体管电流源做尾（第 4 章）→ CMRR 大幅提升

### 5. 发射极退化差分对
- 串联 `RE` 扩展线性输入范围
- 线性范围扩展量 ≈ `ITAIL × RE`
- 代价是增益同比例下降

---

## 设计启示

1. **手算时选最简单的模型。** dc 分析忽略 `ro`、小信号分析先忽略 `rμ` 和 `Cμ`，必要时再加回来。

2. **BJT vs MOS 的取舍是模拟 IC 设计的核心权衡。**
   - 需要高增益 → BJT（`gm` 高，`av_max` 与电流无关）
   - 需要高输入阻抗 → MOS（栅极为绝缘体）
   - 需要低失调 → BJT（`VT` 缩放，无 `ΔVt` 项）
   - MOS 增益随 `ID` 减小而增大（`av_max ∝ 1/√ID`），亚阈值区可接近 BJT

3. **Cascode 是提高增益的高效手段。** 在 MOS 中可多级级联，代价是消耗电压裕度。Active cascode 用反馈在相同电压裕度下获得更高的输出阻抗。

4. **差分对的线性范围和增益是一对权衡。** BJT 线性范围固定约 ±26mV，MOS 可通过调节 `Vov` 灵活设定。发射极/源极退化扩展线性范围但降低增益——这是反馈的固有效应。

5. **失调主要由随机失配决定。** BJT 失调 ~ mV 级 (σ ≈ 2% × 26mV + 负载失配)，MOS 失调更差 (~ 2mV `ΔVt` + `Vov/2` × 几何失配)。大尺寸器件 + 对称版图是降低失调的基本手段。失调服从高斯分布，标准偏差按 $\sigma_{\text{total}} = \sqrt{\sum \sigma_i^2}$ 叠加。

6. **共模抑制依赖于尾电流源的高阻抗。** 单级差分对的 CMRR ≈ `1 + 2gm·RTAIL`，用电阻做尾远远不够，第 4 章的晶体管电流源是解决方案。

---

## 章节关联

- **前序:**
  - [[Ch01 - Models for Integrated-Circuit Active Devices|Ch01 器件模型]] — hybrid-π 模型、T 模型的来源，`gm`、`ro`、`rπ` 的定义
  - [[Ch02 - Bipolar, MOS, and BiCMOS Integrated-Circuit Technology|Ch02 集成电路工艺]] — BJT 和 MOS 的工艺参数背景
- **后续:**
  - **Ch04 电流镜与有源负载** — 如何用晶体管实现高阻尾电流源和负载，以替代本章的电阻
  - **Ch07 频率响应** — cascode 如何减小密勒效应提高带宽、CB/CG 的高频特性
  - **Ch08 反馈** — 退化 (degeneration)、差分对共模反馈的系统性处理
  - **Ch09 稳定性** — Active Cascode 和 Super Source Follower 中负反馈环路的稳定性问题

---

## 检索关键词

`CE amplifier`, `CS amplifier`, `CB amplifier`, `CG amplifier`, `emitter follower`, `source follower`, `emitter degeneration`, `source degeneration`, `cascode`, `CE-CB`, `CS-CG`, `active cascode`, `Darlington`, `CC-CE`, `super source follower`, `differential pair`, `half circuit`, `differential-mode gain`, `common-mode gain`, `CMRR`, `input offset voltage`, `input offset current`, `offset drift`, `Gaussian distribution`, `two-port model`, `T model`, `unilateral`, `bilateral`, `Gm`, `transconductance`, `output resistance`
