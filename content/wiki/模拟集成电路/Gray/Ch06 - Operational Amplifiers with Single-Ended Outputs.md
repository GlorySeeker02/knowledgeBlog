---
title: "第6章：单端输出运算放大器"
source: "Gray, Hurst, Lewis, Meyer - Analysis and Design of Analog Integrated Circuits, 5th Edition, Ch06"
tags:
  - analog-design
  - analog-IC
  - op-amp
  - single-ended
  - NE5234
  - cascode
  - rail-to-rail
---

# 第6章：单端输出运算放大器

## 本章定位

本章以单端输出运算放大器为主题，既是前四章基本单元电路（差分对、电流镜、有源负载、输出级）的综合应用，也是后续章节（第7章频率响应、第8章反馈、第9章稳定性、第12章全差分运放）的铺垫。全章按照"应用概览" -> "非理想特性分类" -> "CMOS运放架构演进" -> "Bipolar运放实战分析（NE5234）"四条主线展开，构成从理论到工程设计的完整闭环。

---

## 核心概念

### 1. 运放应用原理 (6.1)

> [!abstract] 虚短虚断（Summing-Point Constraints）
> 在深度负反馈且开环增益 $a \to \infty$ 的条件下：
> - **虚短**: $V_i = V_o/a \approx 0$，即差分输入电压被驱动到零
> - **虚断**: 运放输入电阻无穷大，无电流流入输入端
>
> 这两个约束是分析所有理想运放电路的基础。

**基本应用拓扑**——所有闭环增益在 $T = af \gg 1$ 时仅由外部无源元件决定：

| 拓扑 | 闭环增益 |
|------|----------|
| 反相放大 | $\dfrac{V_o}{V_s} \approx -\dfrac{R_2}{R_1}$ |
| 同相放大 | $\dfrac{V_o}{V_s} \approx 1 + \dfrac{R_2}{R_1}$ |
| 电压跟随器 | $\dfrac{V_o}{V_s} \approx 1$ |
| 差分放大 | $V_o = \dfrac{R_2}{R_1}(V_1 - V_2)$ |

此外还讨论了：
- **非线性应用**：对数放大器 $V_o = -V_T \ln(V_s / R I_S)$
- **积分器/微分器**：$V_o = -\frac{1}{RC}\int V_s dt$，$V_o = -RC \frac{dV_s}{dt}$
- **开关电容放大器与积分器**：内部放大器的重要应用，增益由 **电容比值** $C_1/C_2$ 决定，对寄生不敏感，时间常数 $\tau = C_I/(f C_S)$ 由电容比和时钟频率精确确定

### 2. 运放非理想特性全览 (6.2)

| 非理想因素 | 典型值 (Bipolar) | 典型值 (MOS) | 影响 |
|------------|---------------------|------------------|------|
| 输入偏置电流 $I_{BIAS}$ | 10-100 nA | < 0.001 pA | 在反馈电阻上产生直流压降 |
| 输入失调电流 $I_{OS}$ | 几 nA | 极小 | 差分放大器中产生输出误差 $V_O = I_{OS}R_2$ |
| 输入失调电压 $V_{OS}$ | 0.1-2 mV | 1-20 mV | 限制最小可分辨直流信号 |
| 共模输入范围 | $\approx \pm 13$ V (741, $\pm15$V供电) | 现代芯片轨到轨 | 超出范围导致饱和、增益反转 |
| CMRR | ~80 dB | ~80 dB | $\Delta V_{OS} = \Delta V_{ic} / \text{CMRR}$ |
| PSRR | — | — | 电源纹波等效为差分输入 $v_{ic} = v_{dd}/\text{PSRR}^+ + v_{ss}/\text{PSRR}^-$ |

> [!important] PSRR 定义
> $v_o = A_{dm}\left(v_{id} + \dfrac{v_{dd}}{\text{PSRR}^+} + \dfrac{v_{ss}}{\text{PSRR}^-}\right)$，其中 $\text{PSRR}^+ = \dfrac{A_{dm}}{A^+}$，$\text{PSRR}^- = \dfrac{A_{dm}}{A^-}$

### 3. 失调电压的两个分量

$$
V_{OS} = V_{OS(\text{systematic})} + V_{OS(\text{random})}
$$

- **系统性失调**：由电路拓扑决定，即使完美匹配也存在。需满足 $(W/L)_6/(W/L)_4 = 2(W/L)_7/(W/L)_5$ 以使各晶体管电流密度相等
- **随机失调**：由器件失配引起

$$
V_{OS(\text{random})} = \Delta V_{t(1,2)} + \Delta V_{t(3,4)}\frac{g_{m3,4}}{g_{m1,2}} + \frac{V_{ov(1,2)}}{2}\left[\frac{\Delta(W/L)_{1,2}}{(W/L)_{1,2}} + \frac{\Delta(W/L)_{3,4}}{(W/L)_{3,4}}\right]
$$

---

## 关键公式与结论

### 基本两级 CMOS 运放 (Fig. 6.16)

**电压增益**：

$$
A_v = \frac{v_o}{v_i} = G_{m1}R_{o1} \cdot G_{m2}R_o \approx \left(\frac{2}{V_{ov1}}\right)\left(\frac{V_{A1}}{2}\right) \cdot g_{m6}(r_{o6}\parallel r_{o7}) \propto (g_m r_o)^2
$$

> 增益由 $(g_m r_o)^2$ 决定，$g_m r_o \propto V_A/V_{ov}$。增大沟道长度提高 Early 电压，减小过驱动电压也可提高增益——但这与频率响应存在基本折衷。

**输出摆幅**：

$$
V_{ov6} - V_{SS} < V_o < V_{DD} - |V_{ov7}|
$$

**共模输入范围** (p沟输入对)：

$$
V_{IC} > -V_{SS} + V_{ov5} + V_{t1} + V_{ov1}
$$
$$
V_{IC} < V_{DD} - |V_{ov5}| - |V_{t1}|
$$

**CMRR** (单端输出)：

$$
\text{CMRR} \approx \frac{2g_{m(dp)}r_{tail}}{g_{m(mir)}}\cdot\frac{1}{r_{o(dp)}}
\approx \frac{V_{A(dp)}}{V_{ov(dp)}}\cdot\frac{V_{A(tail)}}{V_{ov(mir)}}
$$

**PSRR**:
- $\text{PSRR}^+ \to \infty$ (理想匹配下，第一级和第二级的电源耦合相互抵消)
- $\text{PSRR}^- \approx \dfrac{g_{m6} r_{o6} r_{o7}}{r_{o6} + r_{o7}} \cdot g_{m1} r_{o1}$

### 过驱动电压的全局影响

> [!tip] 核心设计折衷
> 减小 $V_{ov}$ 同时改善：电压增益 $\uparrow$、输出摆幅 $\uparrow$、失调电压 $\downarrow$、CMRR $\uparrow$、共模范围 $\uparrow$、PSRR $\uparrow$
>
> 但代价是：$f_T \propto V_{ov}/L^2$ **下降**，频率特性恶化。这构成了 CMOS 运放设计最基本的 trade-off。

---

## 重要电路结构

本章介绍了五种从简单到复杂的 CMOS 运放架构，以及一种 Bipolar 商业运放的完整设计：

### 1. 基本两级 CMOS 运放 (Section 6.3, Fig. 6.16)

- 架构：p沟差分对 + n沟电流镜负载 (第一级) + n沟共源放大 + p沟电流源负载 (第二级)
- 增益 $\approx (g_m r_o)^2$，输出摆幅距两电源各一个 $V_{ov}$
- 需米勒补偿电容 $C_C$

### 2. Cascode 两级运放 (Section 6.4, Fig. 6.25)

- 在第一级差分对和电流镜中均插入 cascode 管，增益提升至 $\approx (g_m r_o)^3$
- 代价：共模输入范围显著减小

### 3. Telescopic-Cascode 运放 (Section 6.5)

- 单级 cascode 提供约 $(g_m r_o)^2$ 的增益，无需米勒补偿
- 缺点：输出摆幅小，往下距电源两个 $V_{ov}$，往上距电源三个 $V_{ov}$
- 五管叠在电源间的五层过驱动限制了低压应用：

$$
V_{DD} - V_{SS} \geq V_{o(p-p)} + 5|V_{ov}|
$$

### 4. Folded-Cascode 运放 (Section 6.6, Fig. 6.28)

- 将 cascode 管极性反转（折叠），信号路径从朝上折叠回朝下
- 增益 $A_v = G_m R_o \approx g_{m1} \cdot (g_{m2A}r_{o2A}r_{o2} \parallel g_{m4A}r_{o4A}r_{o4})$
- **优势**：
  - 输出摆幅距两电源各两个 $V_{ov}$（比 telescopic 好）
  - 共模输入范围可扩展至负电源轨
  - 负载电容 $C_L$ 兼做补偿，无需额外 $C_C$，高频 PSRR 更好

### 5. Active-Cascode 运放 (Section 6.7, Fig. 6.30)

- 在 cascode 管栅极插入辅助放大器，有效跨导提升 $(a+1)$ 倍
- 增益可达 $(g_m r_o)^3$ 而不额外牺牲输出摆幅
- 需注意辅助放大器反馈环路的稳定性，通常加补偿电容 $C_{C1}, C_{C2}$

### 6. NE5234 Bipolar 运放 (Section 6.8) —— 完整设计范例

这是本章最重要的实战案例，展示了一个商业 rail-to-rail 运放的完整设计：

**架构**：三级放大
- **输入级** (Fig. 6.36)：互补折叠-cascode（npn + pnp 差分对并行），轨到轨共模输入，**恒跨导**设计——总偏置电流恒定 6 $\mu$A，无论哪个差分对工作，$G_m$ 不变
- **第二级** (Fig. 6.37)：分裂差分对 $Q_{25}$-$Q_{28}$，产生同相双输出
- **输出级** (Fig. 6.39)：共射输出管 $Q_{74}/Q_{75}$，轨到轨输出（距电源仅 $V_{CE(sat)} \approx 0.1$V）

**关键设计技巧**：

> [!note] 共模反馈环 (CMFB)
> 第一级共模输出电压 $V_{cmout1}$ 经第二级感知→电平移位→反馈至 BiasCM，构成负反馈环。该环仅作用于共模信号，不影响差模信号。工作在 $V_{cmout1} \approx 0.5$V 的稳定工作点。

> [!note] Class AB 偏置创新
> 传统 Class AB 设 $\prod I_C$ 为常数，大信号时非驱动管会关断。NE5234 通过 $Q_{45}$-$Q_{46}$ 差分对 **比较两输出管的 $V_{BE}$**，以负反馈调节小电流管的电流，使非驱动管在大摆幅下**永不完全关断**，最小电流为 $\approx 6\mu$A（零负载偏置的一半）。

> [!note] 相位反转防护
> 输入差分对饱和会导致增益极性反转（负反馈变正反馈）。NE5234 通过添加短路基-射结的 **"哑"晶体管** $Q_{1d}$-$Q_{4d}$（面积为输入管两倍），在饱和时接管信号耦合路径，保持正常相位关系。

**总体增益** (最小估算，$R_L=2$k$\Omega$):

$$
A_v = A_{v1} \cdot A_{v2} \cdot A_{v3} = G_{m1}R_{o1} \cdot G_{m2}R_{in3} \cdot G_{m3}R_L
$$

---

## 设计启示

1. **增益与频率响应的基本折衷**：$A_v \propto 1/V_{ov}^2$，$f_T \propto V_{ov}/L^2$。选择过驱动电压时必须在 dc 精度与速度之间权衡。

2. **失调优化的三管齐下**：减小负载管 $g_m$（增大 $L$）、减小输入管 $V_{ov}$（增大 $W/L$）、使用单位晶体管阵列保证比例精准。

3. **版图对称性**：平移对称优于镜像对称（抗掩模偏移）；共质心布局对抗工艺梯度；敏感节点（运放输入端）避免金属走线穿越。

4. **架构选型指南**：
   - 需要最大输出摆幅 -> 基本两级或 folded-cascode
   - 需要最高增益且不牺牲摆幅 -> active-cascode
   - 驱动纯电容负载 -> 不一定需要输出缓冲级
   - 单电源低电压 -> folded-cascode + 轨到轨输入级

5. **PSRR 关注的电源电容效应**：$C_{gd}$ 耦合、尾电流源调制、体效应、版图交叉——四种机制都可能将电源纹波耦合到求和节点，全面差分可有效抑制。

6. **开关电容电路的独特需求**：运放输入漏电流必须极小（MOS 栅极输入），否则破坏电荷守恒；offset 在每个时钟周期被采样到电容上，因此对 offset 不敏感。

---

## 章节关联

- **第3-4章** (单级放大器 & 电流镜)：本章所有运放的基本单元——差分对增益、电流镜负载、有源 cascode
- **第5章** (输出级)：Class AB 偏置原理，用于理解 NE5234 输出级设计
- **第7章** (频率响应)：运放的极点/零点分析、米勒效应
- **第8章** (反馈)：环路增益 $T = af$、闭环特性、虚短虚断的理论基础
- **第9章** (稳定性与频率补偿)：补偿电容 $C_C$ 的选取、相位裕度
- **第11章** (噪声)：输入参考噪声与器件尺寸/偏置的关系
- **第12章** (全差分运放)：共模反馈 (CMFB) 的系统性处理、全差分架构的优势

---

## 检索关键词

`运放` `单端输出` `两级运放` `cascode` `telescopic` `folded-cascode` `active-cascode` `NE5234` `轨到轨` `rail-to-rail` `共模反馈 CMFB` `Class AB` `失调电压` `CMRR` `PSRR` `过驱动电压` `开关电容` `switched-capacitor` `虚短虚断` `补偿电容` `版图对称` `共质心`

---

## Sources

- [[raw/模拟集成电路/Gray/Ch06 - Operational Amplifiers with Single-Ended Outputs]]

## See Also

- 第3章：单级放大器（差分对、共源共栅）
- 第4章：电流镜与有源负载
- 第5章：输出级（Class AB 偏置）
- 第7章：频率响应分析
- 第8章：反馈理论
- 第9章：稳定性与频率补偿
- 第11章：噪声
- 第12章：全差分运算放大器
