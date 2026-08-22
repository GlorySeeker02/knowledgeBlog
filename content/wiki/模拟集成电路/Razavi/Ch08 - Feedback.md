---
title: "Ch08 - Feedback"
source: "Razavi-Design of Analog CMOS Integrated Circuits 2nd"
tags:
  - analog-design
  - CMOS
  - feedback
  - Bode
  - Middlebrook
date: 2026-08-02
---

## 本章定位

第 8 章是 Razavi 教材中反馈理论的系统入门。在此之前，前 7 章建立了器件物理、单级/差分/共源共栅放大器、频率响应的基础。本章将这些零散的局部反馈（如源极退化）抽象为统一的反馈系统框架，给出四种反馈拓扑的增益与阻抗公式，并引入三种分析反馈电路的方法。

本章的内容直接为：
- **Ch09** 运算放大器设计（高增益多级放大器及其反馈结构）
- **Ch10** 稳定性与频率补偿（反馈对系统稳定性的影响）
- **Ch14** 非线性分析（反馈对线性度的提升）

打下必要基础。

---

## 核心概念

### 8.1 反馈系统通用模型

反馈系统的四个基本元素：

1. **前馈放大器 (Feedforward Amplifier)** — 提供高增益的信号放大
2. **输出传感机制 (Sense)** — 检测输出电压或电流
3. **反馈网络 (Feedback Network)** — 将输出量的一部分返回输入端
4. **误差生成 (Subtractor/Adder)** — 输入信号减去反馈信号，产生误差量

$$
Y(s) = \frac{H(s)}{1 + G(s)H(s)} X(s) \approx \frac{1}{G(s)} X(s) \quad (\text{当 } GH \gg 1)
$$

其中 $G(s)$ 常用频率无关量 $\beta$ 代替，称为反馈因子 (Feedback Factor)。

**环路增益 (Loop Gain)** $\beta A$ 是所有反馈分析中最核心的参数，决定了增益精度、带宽扩展、阻抗变换和非线性抑制的程度。

> [!important] 环路增益的计算方法
> 将主输入置零，在环路上某点断开，注入一个沿 "正确方向" 的测试信号，跟踪信号绕环一周后回到断点处的值，取负号即为环路增益。

### 8.2 反馈带来的四大好处

| 性质 | 机制 | 量化 |
|:---|:---|:---|
| **增益脱敏 (Gain Desensitization)** | 闭环增益由反馈网络（被动元件比率）决定，不依赖有源器件参数 | $\displaystyle \frac{Y}{X} \approx \frac{1}{\beta}$，相对灵敏度降低 $1+\beta A$ 倍 |
| **带宽扩展** | 增益随频率下降被反馈补偿，-3dB 带宽增加 | $\omega_{-3\text{dB,closed}} = (1+\beta A_0)\omega_0$（单极点系统） |
| **阻抗变换** | 根据传感/返回方式，分别提高或降低输入/输出阻抗 | 乘以或除以 $1+\beta A$ |
| **非线性抑制** | 开环增益随信号幅度的变化被反馈平均化 | 闭环增益比率更接近 1 |

### 8.3 四种放大器类型

根据输入/输出信号是电压还是电流，共有四种放大器：

| 类型 | 理想 $Z_{in}$ | 理想 $Z_{out}$ | 增益量纲 |
|:---|:---|:---|:---|
| Voltage Amplifier | $\infty$ | 0 | 无 (V/V) |
| Transimpedance Amplifier | 0 | 0 | 阻抗 ($\Omega$) |
| Transconductance Amplifier | $\infty$ | $\infty$ | 导纳 (S) |
| Current Amplifier | 0 | $\infty$ | 无 (A/A) |

---

## 关键公式与结论：四种反馈拓扑

### 8.4.1 Voltage-Voltage Feedback (Series-Shunt)

- **传感**：输出电压 → 并联于输出端口
- **返回**：电压 → 串联于输入端口
- 又称 **series-shunt** 反馈

$$
A_\text{closed} = \frac{A_0}{1 + \beta A_0}, \quad \beta = \frac{V_F}{V_\text{out}}
$$

| 参数 | 变化 |
|:---|:---|
| 增益 | $\displaystyle A_\text{closed} = \frac{A_0}{1 + \beta A_0}$ |
| 输入阻抗 | $Z_\text{in,closed} = Z_\text{in,open}(1 + \beta A_0)$ ↑ |
| 输出阻抗 | $\displaystyle Z_\text{out,closed} = \frac{Z_\text{out,open}}{1 + \beta A_0}$ ↓ |

> [!example] 典型电路
> 同相放大器 (non-inverting amplifier)：电阻分压器 $\beta = R_2/(R_1+R_2)$，运放为前馈放大器。

### 8.4.2 Current-Voltage Feedback (Series-Series)

- **传感**：输出电流（串联小电阻） → 串联于输出端口
- **返回**：电压 → 串联于输入端口
- 反馈因子 $R_F$ 具有阻抗量纲

$$
I_\text{out} = \frac{G_m}{1 + G_m R_F} V_\text{in}
$$

| 参数 | 变化 |
|:---|:---|
| 跨导增益 | $\displaystyle G_{m,\text{closed}} = \frac{G_m}{1 + G_m R_F}$ |
| 输入阻抗 | $Z_\text{in,closed} = Z_\text{in,open}(1 + G_m R_F)$ ↑ |
| 输出阻抗 | $Z_\text{out,closed} = Z_\text{out,open}(1 + G_m R_F)$ ↑ |

> [!example] 典型电路
> 电池充电器恒流源：用小电阻 $r_M$ 传感输出电流，经放大器反馈到 MOS 管栅极。输出电流 $I_\text{out} \approx V_\text{REF}/r_M$。

### 8.4.3 Voltage-Current Feedback (Shunt-Shunt)

- **传感**：输出电压 → 并联于输出端口
- **返回**：电流 → 并联于输入端口
- 反馈因子 $g_{mF}$ 具有导纳量纲

$$
V_\text{out} = \frac{R_0}{1 + g_{mF} R_0} I_\text{in}
$$

| 参数 | 变化 |
|:---|:---|
| 跨阻增益 | $\displaystyle R_{0,\text{closed}} = \frac{R_0}{1 + g_{mF} R_0}$ |
| 输入阻抗 | $\displaystyle Z_\text{in,closed} = \frac{Z_\text{in,open}}{1 + g_{mF} R_0}$ ↓ |
| 输出阻抗 | $\displaystyle Z_\text{out,closed} = \frac{Z_\text{out,open}}{1 + g_{mF} R_0}$ ↓ |

> [!example] 典型电路 — 跨阻放大器 (TIA)
> 光纤接收器用 TIA 将光电二极管的电流转换为电压。反馈电阻 $R_F$ 跨接在反相放大器两端，输入阻抗降低 $1+A$ 倍，带宽因此提升。

### 8.4.4 Current-Current Feedback (Shunt-Series)

- **传感**：输出电流 → 串联于输出端口
- **返回**：电流 → 并联于输入端口

$$
I_\text{out} = \frac{A_I}{1 + \beta A_I} I_\text{in}
$$

| 参数 | 变化 |
|:---|:---|
| 电流增益 | $\displaystyle A_{I,\text{closed}} = \frac{A_I}{1 + \beta A_I}$ |
| 输入阻抗 | $\displaystyle Z_\text{in,closed} = \frac{Z_\text{in,open}}{1 + \beta A_I}$ ↓ |
| 输出阻抗 | $Z_\text{out,closed} = Z_\text{out,open}(1 + \beta A_I)$ ↑ |

### 总结速查

| 反馈类型 | 别名 | $Z_{in}$ | $Z_{out}$ | 传感量 | 返回量 |
|:---|:---|:---|:---|:---|:---|
| Voltage-Voltage | Series-Shunt | ↑ | ↓ | $V_{out}$ | $V$ |
| Current-Voltage | Series-Series | ↑ | ↑ | $I_{out}$ | $V$ |
| Voltage-Current | Shunt-Shunt | ↓ | ↓ | $V_{out}$ | $I$ |
| Current-Current | Shunt-Series | ↓ | ↑ | $I_{out}$ | $I$ |

记忆口诀：
- **传感电压 → 并联，降低 $Z_{out}$**；**传感电流 → 串联，提高 $Z_{out}$**
- **返回电压 → 串联输入，提高 $Z_{in}$**；**返回电流 → 并联输入，降低 $Z_{in}$**

---

## 反馈对噪声的影响

反馈不会改善电路的噪声性能。

对于反馈网络无噪声的情况，闭环电路的输入参考噪声与开环时相同：

$$
V_\text{out} = \frac{A_1}{1 + \beta A_1}(V_\text{in} + V_n)
$$

反馈网络本身含电阻/晶体管时，会引入额外噪声，整体噪声性能反而恶化。

> [!warning] 注意
> 当输出端口与传感端口 **不一致** 时（如退化 CS 级，输出在漏极但传感在源极），闭环输入参考噪声可能与开环值不同，即使反馈网络无噪声。

---

## 重要分析方法

### 方法一：Two-Port Method（二端口法，含加载效应）

**适用**：四种经典拓扑，单反馈环路
**思路**：将反馈网络建模为二端口网络（Z/Y/H/G 模型），将其终端阻抗纳入前馈放大器的等效负载后 "断开" 环路。

**三步操作**：

1. 按 Fig. 8.67 的规则正确断开环路（保留反馈网络的加载），计算加载后的开环增益 $A_{OL}$ 及开路输入/输出阻抗
2. 确定反馈因子 $\beta$，计算环路增益 $\beta A_{OL}$
3. 用 $1 + \beta A_{OL}$ 缩放开环参数得到闭环值

**四种拓扑断开规则**（Fig. 8.67）：

| 拓扑 | 输入加载处理 | 输出加载处理 | $\beta$ 定义 |
|:---|:---|:---|:---|
| V-V | 反馈网络输出端口 **短路** | 反馈网络输入端口 **开路** | $\beta = V_2/V_1 \vert_{I_2=0}$ |
| C-V | 反馈网络输出端口 **短路** | 反馈网络输入端口 **开路** | $\beta = V_2/I_1 \vert_{I_2=0}$ |
| V-C | 反馈网络输出端口 **开路** | 反馈网络输入端口 **短路** | $\beta = I_2/V_1 \vert_{V_2=0}$ |
| C-C | 反馈网络输出端口 **开路** | 反馈网络输入端口 **短路** | $\beta = I_2/I_1 \vert_{V_2=0}$ |

> [!warning] 局限性
> - 忽略前馈放大器内部反馈（如 $C_{GD}$）
> - 忽略输入信号通过反馈网络的正向传输
> - 不适用于非经典拓扑

### 方法二：Bode's Method（Bode 法）

**适用**：任意拓扑，无需断开环路
**思路**：将电路中某个受控源（如 MOS 管的 $g_m V_{GS}$）替换为独立源 $I_1$，计算四个系数 $A, B, C, D$。

**四个系数**（Fig. 8.70）：

| 系数 | 定义 | 物理含义 |
|:---|:---|:---|
| $A$ | $\displaystyle\frac{V_\text{out}}{V_\text{in}}\vert_{g_m=0}$ | 晶体管失效时的直接馈通 |
| $B$ | $\displaystyle\frac{V_\text{out}}{I_1}\vert_{V_\text{in}=0}$ | $I_1$ 到输出的传输 |
| $C$ | $\displaystyle\frac{V_1}{V_\text{in}}\vert_{g_m=0}$ | 晶体管失效时 $V_{GS}$ 的馈通 |
| $D$ | $\displaystyle\frac{V_1}{I_1}\vert_{V_\text{in}=0}$ | $I_1$ 到 $V_{GS}$ 的传输 |

**闭环增益**：

$$
\frac{V_\text{out}}{V_\text{in}} = A + \frac{g_m BC}{1 - g_m D}
$$

**Return Ratio (RR)** = $-g_m D$，表示与所选受控源关联的回路增益。

当电路中只有 **一个反馈机制** 且环路经过所选晶体管时，RR 等于真实环路增益。

> [!tip] Bode 法的优势
> - 结果精确，无需任何近似
> - 可解析存在多重反馈机制时各晶体管的 RR
> - 直接揭示 $g_m \to \infty$ 时的渐近增益（Asymptotic Gain）

**渐近增益公式 (Asymptotic Gain Equation)**：

$$
H = H_\infty \frac{T}{1+T} + H_0 \frac{1}{1+T}
$$

其中：
- $H_\infty = A - BC/D$：受控源无穷强 ($g_m \to \infty$) 时的 "理想" 增益
- $H_0 = A$：$g_m = 0$ 时的直接馈通
- $T = -g_m D$：Return Ratio

### 方法三：Middlebrook's Method（Middlebrook 法）

**适用**：任意拓扑，可处理非单向环路中的反向传输
**思路**：在环路中注入电压源 $V_t$ 和电流源 $I_t$，测量 $V_1, V_2, I_1, I_2$，通过 Dissection Theorem 将传递函数分解。

**关键量**：

- $H_\infty$：调节 $V_t, I_t$ 使误差信号 $V_1, I_1$ 同时置零时得到的 "理想" 传递函数
- $T_1, T_2$：分别对应于输入或输出置零时的回路增益

$$
H = H_\infty \frac{1 + 1/T_1}{1 + 1/T_2}
$$

**洞察**：在非单向环路中，正向环路增益 $T_{fwd}$ 和反向环路增益 $T_{rev}$ 共同影响等效环路增益：

$$
T = \frac{T_{fwd}}{1 + T_{rev}}
$$

反向传输 $T_{rev}$ 会 **降低** 有效环路增益。

> [!warning] 局限性
> Middlebrook 法通常比 Bode 法更繁琐，且要求注入点位于主反馈环内部、次环外部——这一条件在实际电路中往往难以界定。

### 方法对比

| | Two-Port | Bode | Middlebrook |
|:---|:---|:---|:---|
| 是否需要断环 | 是 | 否 | 否 |
| 适用范围 | 经典四拓扑 | 任意拓扑 | 任意拓扑 |
| 是否精确 | 近似（忽略 feedthrough） | 精确 | 精确 |
| 计算量 | 简单 | 中等 | 较大 |
| 对多反馈机制 | 不支持 | 返回各晶体管的 RR | 需区分主/次环 |
| 揭示反向传输 | 不 | 不 | 是 |

---

## Blackman 阻抗定理

Blackman 定理是 Bode 法在端口阻抗计算上的延伸：任意端口的阻抗可以表示为

$$
Z_\text{port} = A \cdot \frac{1 + T_{sc}}{1 + T_{oc}}
$$

其中：
- $A$：晶体管失效 ($g_m = 0$) 时端口的阻抗
- $T_{oc}$：端口开路时的 Return Ratio
- $T_{sc}$：端口短路时的 Return Ratio

> [!tip] 直观理解
> - 若 $|T_{sc}| \ll 1$：$Z \approx A / (1 + T_{oc})$，阻抗被除以环路增益
> - 若 $|T_{oc}| \ll 1$：$Z \approx A (1 + T_{sc})$，阻抗被乘以环路增益

**重要**：一般情况下，端口阻抗并不能简单写成 $Z_{open} \cdot (1+T)$ 或 $Z_{open}/(1+T)$ 的形式——Blackman 定理揭示了阻抗变换的完整二元性。

**局限性**：当 $A = \infty$ 或 $A = 0$ 时推导失效（如源跟随器的输出阻抗），需通过并联外接电阻取极限的方法解决。

---

## 双零法 (Double-Null Method)

将 Blackman 定理推广到传递函数：

$$
H = A \cdot \frac{1 + T_{in,0}}{1 + T_{out,0}}
$$

- $T_{in,0}$：输入置零时的 RR
- $T_{out,0}$：输出置零时的 RR

**局限**：当 $A=0$ 时公式失效（在 CMOS 电路中经常出现，如退化 CS 级以 $M_2$ 为关注器件时）。

---

## 环路增益计算的实践要点

1. **最佳断环位置**：MOSFET 的栅极。只有 CS 级产生信号反转，负反馈环必定经过至少一个栅极，在栅极处断环可避免加载效应。

2. **测试信号类型**：
   - 电压注入：断开栅极，注入 $V_t$，测量返回的栅-源电压
   - 电流注入：替换受控电流源为独立源 $I_t$，测量返回的 $V_{GS}$，乘以 $g_m$ 得环路增益

3. **多反馈机制**：不同晶体管的 Return Ratio 可能不同。原因在于：禁用一个晶体管可能同时消除多条反馈路径，而禁用另一个晶体管可能只消除部分反馈。究竟哪个 RR 应视为 "环路增益" 需要根据稳定性分析的需求来判断。

4. **非单向环路**：$C_{GD}$ 等寄生电容产生反向信号通路，使传统的断环测量方法不再精确。

---

## 设计启示

1. **高精度信号处理**：负反馈使闭环增益由被动元件比率决定（如 $C_1/C_2$ 或 $R_2/R_1$），显著降低 PVT 敏感性。

2. **增益-带宽权衡**：反馈将单极点系统带宽扩展 $1+\beta A_0$ 倍，但增益同比例下降。如需同时获得高增益和大带宽，考虑多级反馈放大器级联。

3. **阻抗匹配**：
   - 需要高输入阻抗、低输出阻抗 → Voltage-Voltage 反馈（电压缓冲器）
   - 需要低输入阻抗 → Voltage-Current 反馈（TIA，光纤接收器）
   - 需要高输出阻抗 → Current-Voltage / Current-Current 反馈（恒流源）
   - 需要低输出阻抗 → Voltage-Voltage / Voltage-Current 反馈

4. **环路增益最大化**：更高的环路增益意味着更好的增益精度、更强的非线性抑制和更大的带宽扩展，但会恶化频率稳定性（Ch10）。

5. **加载问题**：设计反馈网络时，其输入阻抗应远大于前馈放大器输出阻抗（减小输出加载），其输出阻抗应远小于前馈放大器输入阻抗（减小输入加载）。

6. **Bode 法的实用价值**：当电路无法清晰地映射到四种经典拓扑时（如输出端口与传感端口不一致），Bode 法提供了一种系统化的精确求解途径。

---

## 章节关联

- [[Ch06 - Frequency Response of Amplifiers]] —— 频率响应是本章反馈带宽分析的基础
- [[Ch09 - Operational Amplifiers]] —— 运放是电压-电压反馈最典型的应用场景
- [[Ch10 - Stability and Frequency Compensation]] —— 反馈提高精度的代价是潜在的振荡，Ch10 处理频率稳定性
- [[Ch14 - Nonlinearity and Mismatch]] —— 反馈对非线性的抑制作用在 Ch14 中深入分析
- Blackman 定理与 Middlebrook 法的原始文献：Bode (1945), Blackman (1943), Middlebrook (1975, 2006)

## See Also

- [[Ch09 - Operational Amplifiers]]
- [[Ch10 - Stability and Frequency Compensation]]
- [[Ch06 - Frequency Response]]

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]

## 检索关键词

反馈, feedback, 环路增益, loop gain, 电压-电压反馈, 跨阻放大器, TIA, Bode 法, Middlebrook 法, Blackman 定理, Return Ratio, 二端口法, 加载效应, 串联-并联反馈, 并联-并联反馈, 渐近增益公式, Asymptotic Gain Equation, 增益脱敏, 阻抗变换, 反馈拓扑
