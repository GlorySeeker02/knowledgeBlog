---
title: "Ch14: Nonlinearity and Mismatch"
source: "Razavi-Design of Analog CMOS Integrated Circuits 2nd"
tags:
  - analog-design
  - CMOS
  - nonlinearity
  - mismatch
  - distortion
updated: 2026-08-02
---

# Ch14: Nonlinearity and Mismatch

## 本章定位

本章与 Ch6 (频率响应)、Ch7 (噪声) 并列为模拟电路三大非理想效应的系统分析。前两章分别处理频域和随机噪声对性能的限制，本章引入另外两个关键非理想因素：

- **非线性 (Nonlinearity)** — 大信号下增益随输入幅度变化，产生谐波失真，限制信号保真度。
- **失配 (Mismatch)** — 名义上相同的器件因工艺随机涨落而存在差异，导致直流失调、偶次失真和共模抑制退化。

本章是连接器件物理（Ch2）和实际电路设计（Ch13 SC电路、Ch19 版图）的桥梁。

---

## 核心概念

### 14.1 非线性

#### 14.1.1 非线性的一般描述

电路输入-输出特性在小信号下近似线性，大信号下偏离直线。非线性亦可视为**小信号增益随输入直流电平变化**（Fig. 14.3）。

在小非线性假设下，用多项式逼近 I/O 特性：

$$
y(t) = \alpha_1 x(t) + \alpha_2 x^2(t) + \alpha_3 x^3(t) + \cdots
$$

其中 $\alpha_1$ 为 $x \approx 0$ 附近的小信号增益。

**静态非线性度定义**（Fig. 14.4）：在输入范围 $[0, V_{\text{in,max}}]$ 内，过实际特性两端点画一条直线，设最大偏差为 $\Delta V$，归一化到最大输出摆幅 $V_{\text{out,max}}$：

$$
\text{Nonlinearity} = \frac{\Delta V}{V_{\text{out,max}}}
$$

> **Example 14.1**: 若 $y = \alpha_1 x + \alpha_3 x^3$，输入范围 $[-x_{\max}, +x_{\max}]$，则归一化非线性度为 $\frac{\alpha_3 x_{\max}^2}{\alpha_1 \cdot 3\sqrt{3}}$，非线性与输入幅度的**平方**成正比。

#### 14.1.2 差分电路的非线性

差分电路具有**奇对称** I/O 特性：$f(-x) = -f(x)$，因此多项式展开中所有偶次项系数为零：

$$
y(t) = \alpha_1 x(t) + \alpha_3 x^3(t) + \alpha_5 x^5(t) + \cdots
$$

**结论：差分电路（差模驱动）不产生偶次谐波。** 这是差分操作的又一重要优势。

**CS 级 vs. 差分对 失真对比**（Fig. 14.6，两者电压增益相同）：

| | CS级 (单端) | 差分对 |
|---|---|---|
| HD2(归一化) | $\frac{V_m}{4(V_{GS}-V_{TH})}$ | **0** (奇对称) |
| HD3(归一化) | — | $\frac{V_m^2}{8(V_{GS}-V_{TH})^2}$ |
| 示例 ($V_m = 0.2(V_{GS}-V_{TH})$) | 5% THD | 0.125% THD |

差分对功耗为 CS 级两倍（$I_{SS} = 2I$），但即使 CS 级偏置电流翻倍，失真仅下降 $\sqrt{2}$ 倍。差分结构在**线性度、共模抑制**方面的收益远超功耗代价。

#### 14.1.3 负反馈对非线性的抑制

非线性可视为小信号增益随输入电平的变化，负反馈使闭环增益对开环增益变化不敏感，因此也抑制非线性。

对于前馈放大器 $y \approx \alpha_1 x + \alpha_2 x^2$ 的反馈系统（Fig. 14.7），闭环相对二次谐波为：

$$
\frac{b}{a} = \frac{\alpha_2 V_m}{2\alpha_1} \cdot \frac{1}{(1 + \beta \alpha_1)^2}
$$

**关键结论：反馈使相对二次谐波除以 $(1 + \beta\alpha_1)^2$，增益除以 $(1 + \beta\alpha_1)$。** 即非线性改善比增益减小快一个因子。

**设计准则**（Fig. 14.8）：若小信号增益随输入单调下降，则增益误差 $\Delta y_1$ **总是大于** 非线性度 $\Delta y_2$。因此保证 $\Delta y_1 < \epsilon$（通过足够大的开环增益）就足以保证 $\Delta y_2 < \epsilon$。这是工程中常用的保守但实用的方法。

#### 14.1.4 电容非线性

SC 电路中电容的电压依赖性引入失真。线性电容 $Q = CV$，但对电压依赖电容 $dQ = C\,dV$，电荷取决于电压**历史**。

考虑 $C_1 \approx M C_0 (1 + \alpha_1 V)$、$C_2 \approx C_0 (1 + \alpha_1 V)$ 的非反相放大器（Fig. 14.9），求解输出得：

$$
V_{out} \approx M V_{\text{in}0} \left[ 1 - \frac{\alpha_1 M - \alpha_1}{2} V_{\text{in}0} + \cdots \right]
$$

第二项表征由电容非线性引入的失真。

#### 14.1.5 采样电路的非线性

MOS 开关的导通电阻 $R_{\text{on}}$ 随输入/输出电压变化（Fig. 14.10），导致输出相位偏移周期性波动，产生谐波。

关键分析技巧：周期性输入使 $R_{\text{on}}$ 也**周期性**变化，可用傅里叶级数逼近：

$$
R_{\text{on}}(t) \approx R_0 + R_1 \cos\omega_0 t + R_2 \cos 2\omega_0 t + \cdots
$$

代入输出表达式后在 $R_{\text{on}}C_1\omega_0 \ll 1$ 条件下展开，得：
- 二次谐波幅度：$\frac{V_0 R_1 C_1 \omega_0}{2}$
- 三次谐波幅度：$\frac{V_0 R_2 C_1 \omega_0}{2}$

差分采样开关则抑制偶次谐波。

#### 14.1.6 线性化技术

核心思想：使增益尽量独立于晶体管偏置电流。

**1. 电阻源极退化（Fig. 14.11-14.12）**

整体跨导：
$$
G_m = \frac{g_m}{1 + g_m R_S}
$$

$g_m R_S$ 足够大时 $G_m \approx 1/R_S$，与输入无关。线性化程度由 $g_m R_S$ 决定而非单独的 $R_S$。退化电阻引入了**线性度-增益-噪声-电压裕度**的折中。

**2. 深三极管区 MOSFET 退化（Fig. 14.13-14.14）**

用深三极管区 MOS 替代线性电阻。需注意 $V_b$ 跟踪输入共模电平以确保 $R_{on3}$ 精确。Fig. 14.14 的两管方案在一管进入饱和区时仍能维持线性，最优尺寸比 $(W/L)_{1,2} \approx 7(W/L)_{3,4}$。

**3. 宽度偏移技术（Fig. 14.15）**

利用尺寸失配产生 $G_m$ 水平偏移，并联两个偏移方向相反的差分对，使总 $G_m$ 在更宽输入范围保持平坦。

**4. 三极管区输入器件（Fig. 14.16）**

MOS 在 $V_{DS}$ 恒定且很低时 $I_D$ 与 $V_{GS}$ 线性（$I_D = \frac{1}{2}\mu C_{ox}\frac{W}{L}[2(V_{GS}-V_{TH})V_{DS} - V_{DS}^2]$）。用放大器 + 共源共栅迫使 $V_X, V_Y = V_b$。缺点包括跨导小、输入共模控制苛刻、噪声大。

**5. 后矫正（Post-Correction，Fig. 14.17-14.18）**

将放大器视为 V/I 级 ($I_{out}=f(V_{in})$) 和 I/V 级 ($V_{out}=f^{-1}(I_{in})$) 的级联，后级矫正前级非线性。差分对 + 二极管连接负载的组合增益为 $\sqrt{\frac{(W/L)_{1,2}}{(W/L)_{3,4}}}$，与偏置电流无关。

**6. 局部反馈（Fig. 14.19）**

通过 PMOS 器件感应输出电压，将比例电流反馈回输入器件源极。反馈保证输入管 $V_{GS}$ 恒定，流经 $R_S$ 的信号电流 $I_{sig} = V_{in}/R_S$ 与 $V_{in}$ **线性**成比例。整个电路（不含 $R_D$）实现线性 V-I 转换。

---

## 关键公式与结论

### 谐波失真 (Harmonic Distortion)

输入 $x(t) = A\cos\omega t$ 的多项式响应：

$$
\begin{aligned}
y(t) &= \alpha_1 A\cos\omega t + \alpha_2 A^2\cos^2\omega t + \alpha_3 A^3\cos^3\omega t + \cdots \\
&= \frac{\alpha_2 A^2}{2} + \left(\alpha_1 A + \frac{3\alpha_3 A^3}{4}\right)\cos\omega t \\
&\quad + \frac{\alpha_2 A^2}{2}\cos 2\omega t + \frac{\alpha_3 A^3}{4}\cos 3\omega t + \cdots
\end{aligned}
$$

- **偶次项产生偶次谐波，奇次项产生奇次谐波**。
- 第 $n$ 次谐波幅度大致正比于 $A^n$。

**总谐波失真 (THD)**，对三阶非线性：

$$
\text{THD} = \frac{\sqrt{\text{power of all harmonics except fundamental}}}{\text{power of fundamental}} \approx \sqrt{\left(\frac{\text{HD2}}{\text{fund}}\right)^2 + \left(\frac{\text{HD3}}{\text{fund}}\right)^2}
$$

典型需求：
- 高质量音频（CD）：THD $\approx 0.01\%$ ($-80$ dB)
- 视频产品：THD $\approx 0.1\%$ ($-60$ dB)

### 差分电路 HD3 公式

对于差分对（$|V_{in}| \ll V_{GS}-V_{TH}$）：

$$
\frac{A_{HD3}}{A_F} \approx \frac{V_m^2}{8(V_{GS}-V_{TH})^2}
$$

### 反馈对失真的抑制

$$
\left.\frac{b}{a}\right|_{\text{closed-loop}} = \left.\frac{b}{a}\right|_{\text{open-loop}} \cdot \frac{1}{(1 + \beta\alpha_1)^2}
$$

### 互调失真 (Intermodulation) 与 IP2/IP3

本章以谐波失真（单频测试）为主要度量。在实际宽带系统中，**互调失真 (IMD)** 更为关键：

- 输入双音信号 $x(t) = A(\cos\omega_1 t + \cos\omega_2 t)$。
- 二阶非线性产生 $\omega_1 \pm \omega_2$ 分量，三阶产生 $2\omega_1 \pm \omega_2$ 和 $2\omega_2 \pm \omega_1$ 分量。
- **二阶截点 (IP2)**：基频外推与二阶互调分量相等时的输入/输出功率。
- **三阶截点 (IP3)**：基频外推与三阶互调分量相等时的输入/输出功率。III P3 是 RF/宽带系统最常用的线性度指标。

差分电路固有地抑制偶次分量，因此 IIP2 通常远高于 IIP3。

### 动态范围

**无杂散动态范围 (SFDR)**：从噪底到使最大杂散分量（谐波或互调）等于噪底时的输入功率范围。SFDR 受噪声和线性度**共同**限制。

本章核心洞见：
- **噪声**限制最小可检测信号。
- **非线性**限制最大可接受信号。
- 动态范围是两者之间的可利用区间。反馈和失配消除可推升低端（降噪）、扩展动态范围。

---

## 失配分析

### 失配的来源与统计模型

MOSFET 饱和区 $I_D = \frac{1}{2}\mu C_{ox}\frac{W}{L}(V_{GS}-V_{TH})^2$。工艺随机涨落导致 $\mu C_{ox}$、$W$、$L$、$V_{TH}$ 的失配。

**基本规律**：器件面积 $WL$ 越大，失配越小（更充分的"平均化"）。实验验证：

$$
\Delta V_{TH} = \frac{A_{VTH}}{\sqrt{WL}}, \quad \frac{\Delta K}{K} = \frac{A_K}{\sqrt{WL}}
$$

其中 $K = \mu C_{ox} W/L$，$A_{VTH}$、$A_K$ 为工艺相关的比例因子。

> **Example 14.4**: 40 nm 工艺 $A_{VTH}=4\text{ mV}\cdot\mu\text{m}$，要求 $\Delta V_{TH} \leq 2\text{ mV}$，则 $W = \left(\frac{4}{2}\right)^2 / L$。若 $L=40\text{ nm}$，$W \approx 100\ \mu\text{m}$ → 纳米工艺下低失调需要极大的 $W/L$。

**$\Delta V_{TH}$ 与沟道电容的折中**：$C_{\text{ch}} \propto WLC_{ox}$，降低失配意味着更大的输入电容和更低的带宽。

### 14.2.1 失配的影响

#### 1. 直流失调电压

差分对的输入参考失调：

$$
\begin{aligned}
V_{OS,in} &= \Delta V_{TH} + \frac{V_{GS} - V_{TH}}{2} \left[ -\frac{\Delta R_D}{R_D} - \frac{\Delta(W/L)}{(W/L)} \right]
\end{aligned}
$$

独立统计量叠加（方差形式）：

$$
\overline{V_{OS,in}^2} = \overline{\Delta V_{TH}^2} + \left( \frac{V_{GS}-V_{TH}}{2} \right)^2 \left[ \overline{\left(\frac{\Delta R_D}{R_D}\right)^2} + \overline{\left(\frac{\Delta(W/L)}{(W/L)}\right)^2} \right]
$$

**关键设计结论**：
- $\Delta V_{TH}$ 直接以 1:1 折算到输入——第一重要项。
- 负载电阻失配和尺寸失配的贡献**随过驱动电压增大而增大**。
- 为减小失调，应**降低 $V_{GS}-V_{TH}$**（减小尾电流或增大宽度）。

> **失调与噪声的类比**：失调可视为极低频的噪声分量，可将失调建模为串联在栅极的电压源，直接复用 Ch7 的噪声分析框架。

**Example 14.5 — 两级放大器的失调**（Fig. 14.27）：

$$
V_{OS,in}^2 = V_{OS,N}^2 + V_{OS,P}^2 \cdot \left(\frac{g_{mP}}{g_{mN}}\right)^2
$$

与噪声相同，PMOS 失调贡献按 $g_{mP}/g_{mN}$ 缩放。

#### 2. 电流源的失配

$$
\frac{\Delta I_D}{I_D} \approx \frac{\Delta(W/L)}{(W/L)} - \frac{2\Delta V_{TH}}{V_{GS}-V_{TH}}
$$

**重要反差**：为减小**电流失配**，应**最大化** $V_{GS}-V_{TH}$，与电压失调的要求相反。解释：过驱动越大，阈值失配对电流的相对影响越小。

| 目标 | 过驱动电压方向 |
|---|---|
| 低输入失调电压 | 减小 $V_{GS}-V_{TH}$ |
| 低电流失配 | 增大 $V_{GS}-V_{TH}$ |

这与 Ch7 中噪声的对应趋势一致。

#### 3. 失配导致的偶次失真

差分电路奇对称性依赖精确匹配。失配破坏对称性 → 引入有限偶次非线性。对两路分别建模 $y_1 \approx \alpha_1 x_1 + \alpha_2 x_1^2 + \cdots$，$y_2 \approx \beta_1 x_2 + \beta_2 x_2^2 + \cdots$：

差分输出中二次谐波幅度：
$$
\frac{|\alpha_2 - \beta_2|}{2} A^2
$$

即二次谐波正比于**两路二阶系数之差**。频率较高时，相位失配也会贡献偶次失真。功率器件引起的片上热梯度可能加剧不对称。

#### 4. CMRR 退化

共模抑制度与失调的关系：

$$
\frac{\partial V_{OS,out}}{\partial V_{in,CM}} = A_{CM-DM}, \quad \text{CMRR} = \frac{\partial V_{in,CM}}{\partial V_{OS,in}}
$$

**通俗理解**：CMRR 退化 = 输入共模电压变化导致输出失调变化，折算回输入。若 PMOS 差分对源极不接衬底（Fig. 14.40b），体效应使 $V_{TH}$ 随共模电压变化，失配导致失调随共模变化 → CMRR 恶化。

### 14.2.2 失调消除技术

| 技术 | 原理 | 优点 | 缺点 |
|---|---|---|---|
| **输出失调存储** (Fig. 14.30-14.31) | 输入短路，放大后的失调储存在输出串联电容上 | 结构简单 | $A_v$ 过大时输出饱和，一般 $A_v < 10$ |
| **输入失调存储** (Fig. 14.32) | 放大器接成单位增益反馈，失调电压复制并储存在输入串联电容上 | 可以处理高增益需求 | 开关电荷注入失配可能使大增益放大器饱和 |
| **辅助放大器** (Fig. 14.33-14.35) | 用辅助差分对 ($G_{m2}$) 感知并抵消主放大器失调，储能电容不在信号通路中 | 信号路径不被电容加载，可在折叠共源共栅中实现 | $S_3/S_4$ 的电荷注入失配无法被反馈环路校正 |

**辅助放大器方案的残余失调**（Fig. 14.36，考虑 $G_{m2}$ 自身失调）：

$$
V_{OS,tot} \approx \frac{V_{OS1}}{G_{m2}R} + \frac{V_{OS2}}{G_{m1}R}
$$

当 $G_{m2}R$ 和 $G_{m1}R$ 都大时，残余失调极小。典型设计中 $G_{m2} \approx 0.1 G_{m1}$ 以控制电荷注入误差。

> **注意**：所有失调消除技术都需要**周期性刷新**——结漏电和亚阈值漏电会缓慢破坏电容上储存的矫正电压，刷新率至少 kHz 级别。

### 14.2.3 失调消除降低噪声——相关双采样 (CDS)

失调可视为极低频噪声 → 周期性失调消除也可抑制低频噪声。

**CDS 原理**（Fig. 14.37-14.39）：
1. 失调取消阶段：输入断开，放大器输入短路，失调存储在 $C_1$、$C_2$ 上。
2. 采样阶段：输入接通，采样完成后关断开关。

从失调消除结束 ($t_1$) 到采样结束 ($t_2$) 的间隔 $\Delta t$ 内，只有 **频率高于 $\sim 1/\Delta t$** 的噪声分量有时间显著变化。

**数值示例**：$\Delta t=10\text{ ns}$，1 MHz 分量在 $\Delta t$ 内最大变化 6.3%，10 MHz 分量变化 63%。频率低于数 MHz 的分量来不及响应——相当于一个**高通滤波**操作。

> CDS 是抑制 MOS 电路 $1/f$ 噪声的强大技术，但会引发宽频噪声的混叠（aliasing）。

---

## 设计启示

1. **差分结构是线性度的第一道防线**：消除偶次谐波，在同等增益和摆幅下 THD 远优于单端。
2. **反馈是线性度的第二道防线**：相对谐波按 $(1+\text{loop gain})^2$ 衰减，效果比增益下降更快。
3. **增益误差可保守估计非线性度**：确保开环增益足够大 → 增益误差足够小 → 非线性度自动满足（"懒人准则"，但会浪费增益）。
4. **线性化方法 = 使增益与偏置脱钩**：源极退化、后矫正、局部反馈，本质都是让 $G_m$ 趋于常数。
5. **失调与噪声共享设计框架**：相同公式，相同折中，可复用 Ch7 分析方法。
6. **失调与电流匹配对 $V_{GS}-V_{TH}$ 的要求相反**：低失调 → 小过驱动；低电流失配 → 大过驱动。设计时需明确哪个是主要精度瓶颈。
7. **面积即精度**：$\Delta V_{TH} \propto 1/\sqrt{WL}$。低失调 → 大器件 → 大电容 → 低带宽/高功耗。这是精度-速度-功耗的不可能三角。
8. **CDS 同时消除失调和 $1/f$ 噪声**：在采样系统中一举两得，但需规划时序裕度。
9. **SC 电路额外关注电容非线性**：$C(V)$ 使电荷不再简单正比于瞬时电压，大信号摆幅下不可忽略。
10. **版图对称性是失配控制的物理基础**：Ch19 将系统阐述共质心、虚拟器件等技术。

---

## 章节关联

- **[[Ch03 - Single-Stage Amplifiers|Ch03]] / [[Ch04 - Differential Amplifiers|Ch04]]**：单级和差分放大器的大信号分析中首次遇到非线性和失配现象，本章是理论深化。
- **[[Ch06 - Frequency Response of Amplifiers|Ch06]]**：频率响应与非线性同为限制信号保真度的非理想效应。
- **[[Ch07 - Noise|Ch07]]**：噪声与失配的数学框架高度统一。失调 = 极低频噪声；CDS 同时抑制两者。输入参考噪声和输入参考失调的折中方向一致（对过驱动电压的依赖）。
- **[[Ch08 - Feedback|Ch08]]**：反馈对非线性的抑制机理直接来自反馈对增益的稳定化。
- **[[Ch13 - Introduction to Switched-Capacitor Circuits|Ch13]]**：SC 电路中的电容非线性和采样开关非线性在本章展开分析；SC 放大器本身也实现失调消除。
- **[[Ch19 - Layout and Packaging|Ch19]]**：本章给出失配的数学描述 ($\propto 1/\sqrt{WL}$) 和效应分析，Ch19 提供版图层面的最小化技术。

---

## 检索关键词

`nonlinearity`, `harmonic distortion`, `THD`, `HD2`, `HD3`, `intermodulation`, `IP2`, `IP3`, `SFDR`, `differential nonlinearity`, `source degeneration`, `linearization`, `post-correction`, `mismatch`, `dc offset`, `offset voltage`, `VOS`, `input offset storage`, `output offset storage`, `auxiliary amplifier`, `correlated double sampling`, `CDS`, `CMRR`, `common-mode rejection`, `overdrive voltage`, `Pelgrom`, `$A_{VTH}$`, `$1/\sqrt{WL}$`, `threshold mismatch`, `current mismatch`, `capacitor nonlinearity`, `sampling distortion`, `thermal gradient`

---

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]

## See Also

- Ch03 Single-Stage Amplifiers — 大信号分析、非线性初见
- Ch04 Differential Amplifiers — 差分对非线性、CMRR
- Ch07 Noise — 噪声与失调的统一框架
- Ch08 Feedback — 反馈稳定增益 = 抑制非线性
- Ch13 Introduction to Switched-Capacitor Circuits — 电容非线性、采样非线性、SC 失调消除
- Ch19 Layout and Packaging — 失配的版图控制技术
