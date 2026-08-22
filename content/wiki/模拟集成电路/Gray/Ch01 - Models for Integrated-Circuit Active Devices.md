---
title: "Ch01 - 集成电路有源器件模型"
source: "Gray, Hurst, Lewis, Meyer, Analysis and Design of Analog Integrated Circuits, 5th Ed., Ch1"
tags:
  - analog-design
  - analog-IC
  - device-models
  - bipolar
  - MOS
---

# Ch01 - 集成电路有源器件模型

## 本章定位

第1章是全书的基础。它为后续所有电路分析建立了器件层面的理论根基。本章从 **pn 结**出发，系统推导了 **双极型晶体管 (BJT)** 和 **MOS 场效应管 (MOSFET)** 的大信号与小信号模型，涵盖寄生效应、频率响应和短沟道/弱反型等进阶话题。核心思想：**任何分析的精度不可能超过所用模型的精度**，设计者必须深刻理解每个模型的来源及近似程度。

>[!info] **核心思想**
>There is no such thing as a "perfect" model — only models that are adequate for a given purpose. 手工分析用简单模型，计算机仿真用复杂模型。关键在于知道何时省略哪些元件。

---

## 核心概念

### 1. pn 结基础 (Section 1.2)

pn 结是所有集成电路有源器件的基本构成单元。

- **耗尽区 (depletion region)**：反向偏压下，载流子被扫出结区，留下固定离化杂质，形成空间电荷区。
- **内建电势 (built-in potential)**：
  $$
  \psi_0 = \frac{kT}{q} \ln\frac{N_A N_D}{n_i^2}
  $$
  室温下 Si 中 $n_i \simeq 1.5 \times 10^{10} \text{ cm}^{-3}$。
- **耗尽层宽度**与掺杂浓度成**反比**——耗尽区几乎完全延展到轻掺杂一侧。
- **突变结耗尽电容**：
  $$
  C_j = \frac{C_{j0}}{\sqrt{1 - V_D / \psi_0}}
  $$
  其中 $V_D$ 正向偏压为正、反向偏压为负。梯度结用指数 $1/3$ 代替 $1/2$。正向偏压超过 $\psi_0/2$ 时公式不再准确。
- **击穿机制**：
  - **雪崩击穿 (avalanche)**：载流子在耗尽区获得足够能量碰撞电离产生电子-空穴对。乘法因子 $M = 1/[1-(V_R/BV)^n]$，$n \approx 3\sim6$。
  - **齐纳击穿 (Zener)**：仅发生在重掺杂结（$BV < 6\text{V}$），机理是隧穿而非倍增。操作在击穿区的二极管用作电压基准（齐纳二极管）是非破坏性的，但必须限流。
- 临界电场 $\mathcal{E}_{\text{crit}} \approx 3\times 10^5 \text{ V/cm}$（掺杂 $10^{15}\sim 10^{16}\text{ cm}^{-3}$），随掺杂增加而缓增。

### 2. 双极型晶体管大信号模型 (Sections 1.3)

**正向有源区 (Forward-Active Region)**：发射结正偏、集电结反偏。

- **集电极电流 (理想)**：
  $$
  I_C = I_S \exp\frac{V_{BE}}{V_T}
  $$
  其中 $V_T = kT/q \approx 26\text{mV}$ @ 25C，$I_S$ 是饱和电流（典型 $10^{-14}\sim 10^{-16} \text{A}$）。

- **饱和电流**的物理表达式：
  $$
  I_S = \frac{q A D_n n_i^2}{Q_B}
  $$
  其中 $Q_B = W_B N_A$ 为单位面积基区掺杂原子数。对非均匀基区，用有效扩散常数 $\overline{D}_n$ 代替。

- **正向电流增益 $\beta_F$**：
  $$
  \beta_F = \frac{I_C}{I_B} = \frac{1}{\frac{D_p}{D_n}\frac{W_B}{L_p}\frac{N_A}{N_D} + \frac{W_B^2}{2\tau_b D_n}}
  $$
  $\beta_F$ 的典型值：$npn$ 管 50~500，横向 $pnp$ 管 10~100。
  - **发射极注入效率** $\gamma$：注入基区的电子电流占总发射结电流的比例，通过大 $N_D/N_A$ 比优化。
  - **基区输运系数** $\alpha_T$：从发射极注入基区到达集电区的载流子比例，通过窄基区 $W_B$ 优化。

- **Early 效应**：$V_{CE}$ 增加 $\to$ 集电结耗尽层展宽 $\to$ 有效基区宽度 $W_B$ 减小 $\to$ $I_C$ 增加。
  $$
  I_C = I_S \exp\frac{V_{BE}}{V_T} \left(1 + \frac{V_{CE}}{V_A}\right)
  $$
  $V_A$ 为 **Early 电压**，典型 15~100V。窄基区管 Early 效应更显著。

- **Ebers-Moll 方程**（适用于所有工作区）：
  $$
  I_C = \alpha_F I_{ES}\left(e^{V_{BE}/V_T} - 1\right) - I_{CS}\left(e^{V_{BC}/V_T} - 1\right)
  $$
  $$
  I_E = -I_{ES}\left(e^{V_{BE}/V_T} - 1\right) + \alpha_R I_{CS}\left(e^{V_{BC}/V_T} - 1\right)
  $$
  满足互易关系：$\alpha_F I_{ES} = \alpha_R I_{CS}$。$\alpha_R$ 典型 0.5~0.8，$\beta_R$ 典型 1~5。

- **饱和区**：两结都正偏，$V_{CE}(\text{sat}) \approx 0.05\sim0.3\text{V}$。集电极表现为低阻抗。基区存储大量电荷，$I_B$ 增大。强制 $\beta$ 始终小于 $\beta_F$。

- **击穿电压**：
  - $BV_{CBO}$：发射极开路时的集电结击穿电压。
  - $BV_{CEO}$：基极开路时集电极-发射极击穿电压。
    $$
    BV_{CEO} = \frac{BV_{CBO}}{\sqrt[n]{\beta_F}}
    $$
    高 $\beta$ 管 $BV_{CEO}$ 低。典型 $BV_{CEO}$ 约为 $BV_{CBO}$ 的一半（考虑边缘效应后）。
  - $BV_{EBO}$：发射结击穿电压，仅 6~8V（发射极重掺杂导致）。**发射结击穿是破坏性的**，会导致 $\beta_F$ 大幅退化。

- **$\beta_F$ 与工作点**：
  - **低温/小电流 (Region I)**：$\beta_F \downarrow$，因为耗尽区复合电流 $I_{BX} \propto \exp(V_{BE}/mV_T)$（$m \approx 2$）。
  - **中电流 (Region II)**：$\beta_F \approx \beta_{FM}$ 常数。
  - **大电流 (Region III)**：$\beta_F \downarrow$，因为大注入效应 ($I_C \propto \exp(V_{BE}/2V_T)$) 和 Kirk 效应。
  - **温度系数**：$\beta_F$ 温度系数约 +7000 ppm/C（高温下 $\beta_F$ 增大）。

### 3. 双极型晶体管小信号模型 (Section 1.4)

完整小信号模型 = 基本 hybrid-$\pi$ 模型 + 寄生元件。

**基本 hybrid-$\pi$ 参数**：

| 参数 | 公式 | 说明 |
|------|------|------|
| 跨导 $g_m$ | $g_m = \frac{q I_C}{kT} = \frac{I_C}{V_T}$ | @1mA, $g_m$ = 38 mA/V，**与极性、尺寸、材料无关** |
| 输入电阻 $r_\pi$ | $r_\pi = \frac{\beta_0}{g_m}$ | $\beta_0$ 为小信号电流增益 |
| 输出电阻 $r_o$ | $r_o = \frac{V_A}{I_C} = \frac{1}{\eta g_m}$ | $\eta = V_T/V_A$，典型 $r_o$ = 50~100k$\Omega$ @1mA |
| 基区充电电容 $C_b$ | $C_b = \tau_F g_m$ | $\tau_F$ 为正向基区渡越时间 |
| 集-基电阻 $r_\mu$ | $r_\mu > 10\beta_0 r_o$ (npn) | 反馈电阻，因基区宽度调制引起 |

**小信号分析适用条件**：$\Delta V_{BE} = v_i \ll V_T \approx 26\text{mV}$。$v_i < 10\text{mV}$ 时误差 < 10%。

**寄生元件**：
- **耗尽层电容**：$C_{je}$（发射结）、$C_\mu$（集电结）、$C_{cs}$（集电极-衬底）。通用公式：
  $$
  C = \frac{C_0}{\left(1 - \frac{V}{\psi_0}\right)^n}
  $$
  典型零偏值（最小尺寸 npn）：$C_{je0} \simeq 10\text{fF}$, $C_{\mu 0} \simeq 10\text{fF}$, $C_{cs0} \simeq 20\text{fF}$。
- **寄生电阻**：$r_b = 50\sim500\Omega$、$r_c = 20\sim500\Omega$、$r_{ex} = 1\sim3\Omega$。$r_b$ 随 $I_C$ 增大而减小（电流集边效应）。
- **总输入电容**：$C_\pi = C_b + C_{je}$。

**频率响应与 $f_T$**：
- 特征频率 $f_T$：共射极短路电流增益降至 1 的频率。
  $$
  \omega_T = 2\pi f_T = \frac{g_m}{C_\pi + C_\mu}
  $$
  $$
  \tau_T = \frac{1}{\omega_T} = \tau_F + \frac{C_{je}}{g_m} + \frac{C_\mu}{g_m}
  $$
- $f_T$ 随 $I_C$ 变化曲线呈钟形：
  - 小电流：$C_{je}$ 和 $C_\mu$ 项主导，$f_T \downarrow$。
  - 中等电流：$f_T \to f_{T\text{max}}$，$\tau_T \approx \tau_F$。
  - 大电流：大注入/Kirk 效应使 $\tau_F$ 增大，$f_T \downarrow$。

>[!info] **BJT 小信号参数速记**
>$g_m = I_C/V_T$，$r_\pi = \beta_0/g_m$，$r_o = V_A/I_C$，$C_\pi = \tau_F g_m + C_{je}$，$f_T \approx g_m/[2\pi(C_\pi + C_\mu)]$。
>
>**全部参数由 $\beta_0$, $\tau_F$, $\eta$, $I_C$ 四个量决定。**

### 4. MOS 晶体管大信号模型 (Sections 1.5, 1.7, 1.8)

**阈值电压**：
$$
V_t = V_{t0} + \gamma\left(\sqrt{2\phi_f + V_{SB}} - \sqrt{2\phi_f}\right)
$$
- $V_{t0}$：$V_{SB}=0$ 时的阈值（增强型 NMOS：0.3~1.5V；耗尽型 NMOS：-1~-4V）。
- $\gamma = \frac{\sqrt{2q\epsilon N_A}}{C_{ox}}$：体效应参数，典型 $0.5\text{ V}^{1/2}$。
- $C_{ox} = \epsilon_{ox}/t_{ox}$：$t_{ox}=100\text{\AA}$ 时 $C_{ox}=3.45\text{ fF/}\mu\text{m}^2$。
- $dV_t/dT < 0$：阈值电压随温度升高而降低（约 -0.5~-4 mV/C）。

**平方律模型（长沟道，强反型）**：
- **三极管区 (Triode)**：$V_{DS} < V_{GS} - V_t$
  $$
  I_D = \frac{k'}{2} \frac{W}{L} \left[2(V_{GS} - V_t)V_{DS} - V_{DS}^2\right]
  $$
- **饱和/有源区 (Saturation/Active)**：$V_{DS} \ge V_{GS} - V_t$
  $$
  I_D = \frac{k'}{2} \frac{W}{L} (V_{GS} - V_t)^2 (1 + \lambda V_{DS})
  $$
  其中 $k' = \mu_n C_{ox}$，典型 $200\text{ }\mu\text{A/V}^2$（NMOS, $t_{ox}=100\text{\AA}$）。

- **沟道长度调制**：$\lambda \propto 1/L$，典型 0.005~0.05 V$^{-1}$。$V_A = 1/\lambda$ 为 MOS 的 Early 电压。

>[!warning] **术语差异**
>**BJT** 的 "saturation" 指双结正偏、$V_{CE}$ 接近零。**MOS** 的 "saturation" 指沟道夹断、电流饱和。
>Gray 教材为避免混淆，将 MOS 的饱和区称为 **active region**（有源区）。

**过驱动电压 (Overdrive)**：
$$
V_{ov} = V_{GS} - V_t = \sqrt{\frac{2 I_D}{k'(W/L)}}
$$
- $V_{ov}$ 随温度**升高**（因为迁移率下降）。
- $V_t$ 随温度**降低**。

**电压限制**：
- **结击穿**：漏-衬底 pn 结雪崩击穿，不破坏性（限流即可）。
- **穿通 (Punchthrough)**：漏耗尽区触碰到源耗尽区，长沟道下发生。不是本征破坏性的。
- **热载流子效应**：高电场下载流子注入氧化层被俘获 $\to$ $V_t$ 漂移 $\to$ **破坏性的**。短沟道工艺尤其需关注。
- **氧化层击穿**：$E_{\text{ox}} \approx 6\sim7 \times 10^6 \text{ V/cm}$，对应 $t_{ox}=100\text{\AA}$ 时 6~7V。**破坏性的**，需用二极管/电阻做 ESD 保护。

### 5. 短沟道效应 (Section 1.7)

当 $L \lesssim 1\mu\text{m}$ 时，MOS 器件偏离平方律特性。

**速度饱和 (Velocity Saturation)**：
- 高横向电场下，载流子漂移速度趋于散射极限速度 $v_{scl}$。
  $$
  v_d = \frac{\mu_n \mathcal{E}}{1 + \mathcal{E}/\mathcal{E}_c}
  $$
  其中 $\mathcal{E}_c \simeq 1.5\times 10^6 \text{ V/m}$，$v_{scl} = \mu_n \mathcal{E}_c$。
- **饱和区电流增加线性化**：完全速度饱和时
  $$
  I_D \propto W C_{ox} (V_{GS} - V_t) v_{scl}
  $$
  **与 $L$ 无关**（电荷 $\propto L$，渡越时间 $\propto L$，电流比值不变）。
- 速度饱和使 $V_{DS}(\text{act}) < V_{ov}$，即更早进入饱和。
- 速度饱和可用理想平方律器件串联一个源极电阻 $R_{SX}$ 来一阶建模。
  $$
  R_{SX} = \frac{1}{\mathcal{E}_c} \cdot \frac{1}{\mu_n C_{ox}(W/L)}
  $$

**垂直场迁移率退化**：
- 栅压增大 $\to$ 垂直电场增强 $\to$ 载流子被压向 Si-SiO$_2$ 界面 $\to$ 表面缺陷散射增加 $\to$ 有效迁移率降低。
  $$
  \mu_{\text{eff}} = \frac{\mu_n}{1 + \theta(V_{GS} - V_t)}
  $$
  $\theta$ 典型 0.1~0.4 V$^{-1}$（与 $t_{ox}$ 成反比）。

**对跨导和 $f_T$ 的影响**：
- 速度饱和使 $g_m$ 趋于常数（不再随 $V_{ov}$ 增大）：
  $$
  g_m \to W C_{ox} v_{scl} \quad (\mathcal{E}_c \to 0)
  $$
- $f_T$ 从 $\propto 1/L^2$（无速度饱和）变为 $\propto 1/L$（速度饱和）。**速度饱和削弱了尺寸缩小带来的速度收益**。

### 6. 弱反型 (Section 1.8)

当 $V_{GS} < V_t$（但足以在表面形成耗尽层）时，器件工作于**弱反型 (weak inversion)** 或称 **亚阈值 (subthreshold)** 区。

- **工作机制**：类似 $npn$ 双极型管——源极=发射极，衬底=基极，漏极=集电极。电流由**扩散**主导，而非漂移。

- **传输特性**：
  $$
  I_D = I_t \frac{W}{L} \exp\left(\frac{V_{GS} - V_t}{n V_T}\right) \left[1 - \exp\left(-\frac{V_{DS}}{V_T}\right)\right]
  $$
  其中 $n = 1 + C_{js}/C_{ox}$，典型 1.5。当 $V_{DS} > 3V_T \approx 78\text{mV}$ 时电流饱和（与 $V_{ov}$ 无关）。

- **弱反型跨导**：
  $$
  g_m = \frac{I_D}{n V_T}
  $$
  形式与 BJT 相同，仅多因子 $1/n$。跨导-电流比恒定（不再随 $V_{ov}$ 变化）。

- **过渡边界**：弱反型与强反型的分界大致在：
  $$
  V_{GS} - V_t \approx 2 n V_T \approx 78\text{mV} \quad (n=1.5)
  $$
  中间区为**中度反型 (moderate inversion)**，扩散和漂移都显著，无简单解析模型。

- **弱反型 $f_T$**：
  $$
  f_T \approx \frac{\mu_n V_T}{2\pi L^2}
  $$
  与 $L^2$ 成反比，与过驱动电压无关。频率远低于强反型（因 $g_m$ 小且 $C_{gb}$ 大）。

- **主要应用**：极低功耗、低频模拟电路。

### 7. MOS 小信号模型 (Section 1.6)

**基本 hybrid-$\pi$ 参数**：

| 参数 | 公式 | 说明 |
|------|------|------|
| 跨导 $g_m$ | $g_m = \sqrt{2k'(W/L) I_D} = \frac{2I_D}{V_{ov}}$ | 与 $\sqrt{I_D}$ 成正比 |
| 体跨导 $g_{mb}$ | $g_{mb} = \chi g_m$ | $\chi \approx 0.1\sim0.3$，体效应作为"第二栅极" |
| 输出电阻 $r_o$ | $r_o = \frac{1}{\lambda I_D}$ | $\lambda$ 为沟长调制参数 |
| 栅-源电容 $C_{gs}$ | $C_{gs} = \frac{2}{3}WLC_{ox}$（饱和） | 含覆盖电容 $C_{ol}$ |
| 栅-漏电容 $C_{gd}$ | $C_{gd} = C_{ol}$（饱和区本征部分为零） | 在 triode 区 $C_{gd} = \frac{1}{2}WLC_{ox}$ |

**关键差异：BJT vs MOS**：

| 属性 | BJT | MOS |
|------|-----|-----|
| $g_m/I$ | $1/V_T \approx 38.5\text{ V}^{-1}$ | $2/V_{ov} \approx 5\sim20\text{ V}^{-1}$ |
| 输入阻抗 | 有限 ($r_\pi$) | 无穷（低频） |
| $g_m$ 与电流 | 线性 $\propto I_C$ | 平方根 $\propto \sqrt{I_D}$ |
| 本征 $f_T$ | $\propto 1/W_B^2$ | $\propto 1/L^2$（无速度饱和） |

>[!important] **BJT vs MOS 的核心差异**
>**$g_m/I$ 比率**是模拟设计最关键的品质因数。BJT 在单位电流下提供远大于 MOS 的跨导，这是 BJT 在独立模拟 IC 中仍有优势的根本原因。MOS 设计的主要挑战是**用低 $g_m/I$ 实现高性能**。

### 8. 衬底电流 (Section 1.9)

- **机理**：NMOS 沟道电子在漏端耗尽区碰撞电离产生电子-空穴对。电子流向漏极，空穴流向衬底形成 $I_{DB}$（衬底电流）。
- **经验公式**：
  $$
  I_{DB} = K_1 (V_{DS} - V_{DS}(\text{act})) I_D \exp\left(-\frac{K_2}{V_{DS} - V_{DS}(\text{act})}\right)
  $$
  典型 NMOS：$K_1 = 5\text{ V}^{-1}$, $K_2 = 30\text{ V}$。PMOS 中该效应显著更弱。
- **电路影响**：产生漏极到衬底（ac 地）的**寄生电导** $g_{db}$：
  $$
  g_{db} = \frac{\partial I_{DB}}{\partial V_{DS}}
  $$
  在高 $V_{DS}$ 下 $g_{db}$ 可能与 $1/r_o$ 相当，限制高输出阻抗电流镜性能。

---

## 关键公式与结论

>[!summary] **BJT 核心公式**
>$$
>I_C = I_S e^{V_{BE}/V_T},\quad g_m = \frac{I_C}{V_T},\quad r_\pi = \frac{\beta_0}{g_m},\quad r_o = \frac{V_A}{I_C},\quad f_T \approx \frac{g_m}{2\pi(C_\pi + C_\mu)}
>$$

>[!summary] **MOS 核心公式（平方律）**
>$$
>I_D = \frac{k'}{2}\frac{W}{L}(V_{GS}-V_t)^2(1+\lambda V_{DS}),\quad g_m = \sqrt{2k'\frac{W}{L}I_D} = \frac{2I_D}{V_{ov}},\quad f_T \approx \frac{g_m}{2\pi C_{gs}}
>$$

>[!summary] **最重要的单一结论**
>BJT 的 $g_m/I_C = 1/V_T \approx 38.5\text{ V}^{-1}$，MOS 的 $g_m/I_D = 2/V_{ov} \approx 5\sim20\text{ V}^{-1}$。单位电流下 BJT 的跨导效率比 MOS 高 2~8 倍。这是理解两种器件在模拟 IC 设计中不同定位的核心。

---

## 重要模型

### 模型层次总结

| 层级 | BJT | MOS |
|------|-----|-----|
| 大信号 dc | $I_C = I_S e^{V_{BE}/V_T}(1+V_{CE}/V_A)$ | $I_D = \frac{k'}{2}\frac{W}{L}(V_{GS}-V_t)^2(1+\lambda V_{DS})$ |
| 通用大信号 | **Ebers-Moll** 方程 | 平方律（triode） + 平方律（saturation） |
| 基本小信号 | Hybrid-$\pi$（$g_m, r_\pi, r_o, C_b$） | Hybrid-$\pi$（$g_m, r_o, C_{gs}, C_{gd}$） |
| 完整小信号 | $+C_{je}, C_\mu, C_{cs}, r_b, r_c, r_{ex}, r_\mu$ | $+C_{sb}, C_{db}, C_{gb}, g_{mb}, g_{db}$ |
| 进阶 | Gummel-Poon（SPICE 内置） | BSIM 短沟道模型（速度饱和、迁移率退化、弱反型） |

### Ebers-Moll 模型直观理解

将晶体管视为**两个背靠背的二极管**加上**互耦电流源**。总电流是正向工作电流和反向工作电流的**线性叠加**。这是一个通用模型，覆盖截止、正向有源、饱和、反向有源四个区域。

### MOS 速度饱和的等效电路模型

速度饱和效应可近似为在理想平方律器件源极串联电阻 $R_{SX} = 1/[\mathcal{E}_c \mu_n C_{ox} (W/L)]$。对 $W=2\mu\text{m}$，该电阻约 1700$\Omega$。

---

## 设计启示

1. **模型选择原则**：手工分析用最简单的够用模型。低频计算可省略寄生电阻和电容。高频或大电流时寄生元件变得关键。

2. **BJT 的优势领域**：高 $g_m/I$ 意味着在给定的功耗下获得更高的增益带宽积。这在需要高精度、低噪声的前端放大器中至关重要。

3. **MOS 的优势领域**：栅极无限输入阻抗使 MOS 成为采样保持电路、开关电容电路的首选。此外，MOS 工艺成本更低、集成度更高。

4. **Early 电压的影响**：$V_A$ 直接决定放大器的本征增益 $g_m r_o = V_A/V_T$（BJT）或 $g_m r_o = 2V_A/V_{ov}$（MOS）。高增益电路需选择高 $V_A$ 的器件或采用 cascode 结构。

5. **过驱动电压的折中**：$V_{ov}$ 大 $\to$ 速度快（$f_T \uparrow$）但 $g_m/I$ 低（功耗效率差）。$V_{ov}$ 小 $\to$ $g_m/I$ 高但速度慢，且逐渐进入弱反型。$V_{ov} \approx 150\sim300\text{mV}$ 是常见的折中区间。

6. **弱反型的应用场景**：极低功耗（nA 级电流）低频（kHz~MHz）电路，如植入式医疗设备、IoT 传感器接口。

7. **短沟道设计的注意事项**：速度饱和使 $I_D$-$V_{GS}$ 关系线性化，$g_m$ 饱和。不要再依赖 $L$ 缩小来无限提升速度。垂直场迁移率退化需要通过薄氧化层（增大 $C_{ox}$）来部分补偿。

8. **可靠性约束**：栅氧击穿和热载流子退化是破坏性的，必须在设计中通过 ESD 保护和电压限幅来避免。发射结反向击穿也会导致 $\beta_F$ 永久退化。

---

## 章节关联

- **Ch02**：本章推导的所有模型参数（$I_S$, $\beta_F$, $V_A$, $k'$, $V_t$, $\gamma$, $\lambda$ 等）的**工艺实现**在 Ch02 中详细描述。理解这些参数与工艺步骤（扩散、注入、氧化）的关系是连接器件物理与电路设计的桥梁。
- **Ch03**：本章建立的小信号模型（特别是 hybrid-$\pi$ 模型及其简化形式）将在 Ch03 中直接用于单级放大器的增益、输入/输出阻抗和频率响应分析。
- **Ch04**：电流镜设计中涉及的输出阻抗、匹配精度直接依赖本章的 $r_o$, $V_A$, $g_m$ 等参数。

---

## 检索关键词

`pn junction`, `depletion capacitance`, `avalanche breakdown`, `Ebers-Moll`, `Gummel-Poon`, `Early voltage`, `hybrid-π model`, `transconductance`, `base transit time`, `transition frequency fT`, `MOSFET square-law`, `threshold voltage`, `body effect`, `channel-length modulation`, `velocity saturation`, `mobility degradation`, `weak inversion`, `subthreshold conduction`, `substrate current`, `impact ionization`, `overdrive voltage`, `BJT operating regions`, `MOS operating regions`, `Kirk effect`, `current crowding`

---

## Sources

- [[raw/模拟集成电路/Gray/Ch01 - Models for Integrated-Circuit Active Devices]]

## See Also

- Ch02: 集成电路工艺技术（BJT 和 MOS 制造工艺、寄生参数的物理来源）
- Ch03: 单级放大器（本章小信号模型的直接应用）
- Ch04: 电流镜和偏置电路（$V_A$, $r_o$, $g_m$ 对匹配和输出阻抗的影响）
