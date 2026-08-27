---
title: "第9章 数模与模数转换器"
source: "Allen-CMOS analog circuit design 3e"
tags:
  - analog-design
  - CMOS
  - DAC
  - ADC
  - data-converters
---

## 本章定位

本章是 Allen 教材的系统级章节，覆盖 CMOS 工艺下数模转换器 (DAC) 和模数转换器 (ADC) 的表征、架构分类、设计方法与性能权衡。DAC 先讲（因其常作为 ADC 的组成部分），ADC 后讲。DAC/ADC 的主要性能三要素是**转换速度、分辨率和功耗**。

---

## 核心概念

### DAC 静态参数

- **分辨率 (Resolution)**：数字输入字的位数 $N$，对应 $2^N$ 个可能的输出电平。
- **LSB**：$\displaystyle \text{LSB} = \frac{V_{REF}}{2^N}$（不垂直偏移时），若输入特性向下平移 0.5 LSB 则公式为 $\frac{V_{REF}}{2^N} \cdot \frac{2^N - 1}{2^N}$。
- **满量程 (FS)**：$\displaystyle FS = V_{REF} \left(1 - \frac{1}{2^N}\right) = \frac{V_{REF}(2^N - 1)}{2^N}$
- **满量程范围 (FSR)**：FSR = $V_{REF}$
- **量化噪声**：理想无限分辨率特性与实际有限分辨率特性之差，峰值 $1\,\text{LSB}$（不偏移）或 $\pm 0.5\,\text{LSB}$（偏移后）。
- **动态范围 (DR)**：$\displaystyle DR = \frac{FSR}{LSB} = 2^N$，用 dB 表示为 $20\log_{10}(2^N) \approx 6.02N\,\text{dB}$。
- **SNR（信号量化噪声比）**：
  - 量化噪声 rms 值：$V_{Q(\text{rms})} = \frac{LSB}{\sqrt{12}} = \frac{V_{REF}}{2^N\sqrt{12}}$
  - 输出满幅正弦 rms：$\frac{V_{REF}}{2\sqrt{2}}$
  - 最大 SNR：$\displaystyle SNR_{max} = 6.02N + 1.76 \;\text{dB}$
- **有效位数 (ENOB)**：$\displaystyle ENOB = \frac{SNR_{\text{actual}} - 1.76}{6.02}$

### DAC / ADC 静态误差

1. **偏移误差 (Offset Error)**：实际特性相对于理想特性的恒定垂直偏移。
2. **增益误差 (Gain Error)**：实际斜率与理想斜率之差，与输出幅度成正比。
3. **积分非线性 (INL)**：实际有限分辨率特性与理想有限分辨率特性之间的最大垂直偏差，单位 LSB。
4. **微分非线性 (DNL)**：相邻台阶步长与理想 1 LSB 之差，$DNL = \frac{V_{cx}}{V_s} - 1$（$V_{cx}$ 实际步长，$V_s$ 理想步长）。$DNL \leq -1\,\text{LSB}$ 必然导致非单调。
5. **单调性 (Monotonicity)**：数字输入递增时模拟输出永不下降。DAC 中 $DNL < -1\,\text{LSB}$ 必然非单调。

### 各比特位精度要求

第 $i$ 位（$i=0$ 为 MSB）的输出权重为 $2^{N-i-1}$ LSB。MSB 精度要求最苛刻（允许不确定度 $\pm 0.5\,\text{LSB}$），精度百分比为 $\frac{100}{2^{N-i}}\%$。16 位时 MSB 在理想其他位条件下需要 0.0015% 精度。

### ADC 特有概念

- **Nyquist 采样定理**：$f_S \geq 2f_B$，抗混叠滤波器必须抑制 $f > 0.5 f_S$ 的分量。
- **孔径抖动 (Aperture Jitter)**：时钟抖动 $\Delta t$ 引起幅度不确定度 $\Delta V \approx \omega_{in} V_p \Delta t$，限制高频动态范围。
- **数字编码**：常用 Binary、Thermometer、Gray、Two's complement。Thermometer 和 Gray 码每次只变 1 位。

---

## DAC 关键公式与结构

### 输出方程
$$
v_{OUT} = K \cdot V_{REF} \cdot D, \quad D = \sum_{i=0}^{N-1} \frac{b_i}{2^{i+1}}
$$

### 并行 DAC 三大缩放方法

| 方法 | 核心原理 | 优势 | 劣势 |
|------|----------|------|------|
| **电流缩放 (Current Scaling)** | 二进制加权电阻/电流源 $\to$ 运放反相求和 | 快，对开关寄生不敏感 | 元件值展宽大（$R_{LSB}/R_{MSB}=2^{N-1}$），可能非单调 |
| **电压缩放 (Voltage Scaling)** | 电阻串分压 $V_{REF} \to 2^N$ 个抽头 $\to$ 解码器选择 | 保证单调，电阻值相同 | 面积大（$N\geq8$ 时很大），对寄生电容敏感 |
| **电荷缩放 (Charge Scaling)** | 二进制加权电容阵列在 $\phi_2$ 阶段连接 $V_{REF}$ | 精度最好，兼容开关电容电路 | 元件值展宽大（$C_{MSB}/C_{LSB}=2^{N-1}$），可能非单调 |

#### 电流缩放子类型

- **二进制加权电阻 DAC**：$v_{OUT} = -\frac{K R}{2} \sum_{i=0}^{N-1} \frac{b_i V_{REF}}{2^i R}$，元件值展宽 $1/2^{N-1}$。
- **R-2R 梯形 DAC**：只用 R 和 2R 两种值，消除大展宽问题；电流每向右经过一级减半，$I_i = \frac{V_{REF}}{2^{i+1}R}$。
- **MOSFET 电流镜 DAC**：通过 $W/L$ 比实现电流加权，$2^N$ 个相同管并联分组合成二进制权重。

#### 电压缩放 INL/DNL 最坏分析
对 $2^N$ 个电阻串 ($\pm \Delta R/R$ 公差)：
$$
INL_{\text{worst}} = \frac{2^{N-1}\Delta R/R}{1 - (\Delta R/R)^2} \cdot LSB
$$
$$
DNL_{\text{worst}} = \frac{\Delta R}{R} \cdot LSB
$$

#### 电荷缩放 INL/DNL 最坏分析
$$
INL_{\text{worst}} \approx 2^{N-1} \frac{\Delta C}{C} \cdot LSB
$$
$$
DNL_{\text{worst}} \approx (2^N - 1) \frac{\Delta C}{C} \cdot LSB
$$

### 扩展 DAC 分辨率

**两种策略**：
1. **同类缩放组合**：M 位 MSB SubDAC + K 位 LSB SubDAC，将 LSB 输出除以 $2^M$（分压法）或将 $V_{REF}$ 除以 $2^M$ 后供给 LSB DAC（Subranging 法）。
2. **异类缩放组合**：
   - 电压缩放 MSB + 电荷缩放 LSB：MSB 保证单调，LSB 用电容确保精度。
   - 电荷缩放 MSB + 电压缩放 LSB：MSB 精度更高，LSB 保证单调。**这是性能更优的组合。**

**组合 DAC 的 INL/DNL 加和公式**：
- MSB 电压 + LSB 电荷：$INL \approx 2^{N-1}\frac{\Delta R}{R} + 2^{K-1}\frac{\Delta C}{C}$，$DNL \approx 2^K \frac{\Delta R}{R} + (2^K - 1)\frac{\Delta C}{C}$
- MSB 电荷 + LSB 电压：$INL \approx 2^{M-1}\frac{\Delta R}{R} + 2^{N-1}\frac{\Delta C}{C}$，$DNL \approx \frac{\Delta R}{R} + (2^N - 1)\frac{\Delta C}{C}$

**关键设计启发**：电阻容差在 MSB 电压组合中被放大 $2^{N-1}$ 倍，导致必须修调电阻；而在 MSB 电荷组合中，电容容差被放大。应该让精度可控的方向做 MSB。

### 串行 DAC

| 架构 | 工作原理 | 特点 |
|------|----------|------|
| **串行电荷再分配** | $C_1=C_2$，从 LSB 开始每个时钟：若 bit=1 则 $C_1$ 接 $V_{REF}$，再与 $C_2$ 电荷分享 | 极简，面积最小，单调，慢（需 $N(N+1)$ 步）|
| **串行 Algorithmic** | 迭代 $V_{out}^{(i)} = b_i V_{REF} + \frac{1}{2} V_{out}^{(i-1)}$（$b_i=\pm 1$） | 结构简单，需精确 $\times\frac{1}{2}$ 增益 |

---

## ADC 关键公式与结构

### ADC 分类

| 转换速率 | Nyquist ADC | Oversampled ADC |
|----------|-------------|-----------------|
| **慢速** | 积分型（单/双斜率） | 极高分辨率 ($\geq14$ bit) |
| **中速** | 逐次逼近、1-bit Pipeline、Algorithmic | 中等分辨率 ($\sim10$ bit) |
| **快速** | Flash、多比特 Pipeline、Folding/Interpolating | 低分辨率 ($\sim6$ bit) |

### 采样保持电路 (S/H)

**关键参数**：
- **捕获时间 $t_a$**：采样模式下输出稳定到误差带内的时间。
- **建立时间 $t_s$**：保持命令后输出瞬态和振铃衰减到误差带内的时间。
- **最小转换时间**：$T_{\text{sample}} = t_a + t_s$
- **孔径时间 / 孔径抖动**：开关断开时间及其抖动，直接限制高频 SNR。

**S/H 基本架构**：
1. 开环缓冲型（图 9.5-8a）：开关 + 保持电容 + 单位增益缓冲器，简单快速。
2. 开关电容型（图 9.5-9）：利用 $\phi_{1d}$ 延迟时钟减小电荷注入。
3. 二极管桥型（图 9.5-10）：时钟馈通与输入无关，速度快。
4. 闭环反馈型（图 9.5-11/12）：精度高，可自调零消除运放失调。

**S/H 中运放建立时间**：单位增益闭环下，
$$
\epsilon(t) \approx e^{-\omega_a t/2} \cdot \frac{GB}{\omega_a}, \quad t_s = \frac{2}{\omega_a} \ln\left(\frac{GB/\omega_a}{\epsilon}\right)
$$
对 N 位精度 $\epsilon = 0.5\,\text{LSB}$。

### 串行 ADC

| 架构 | 原理 | 特点 |
|------|------|------|
| **单斜率 (Single-Slope)** | 斜坡发生器积分 $V_{REF}$，当 $v_{ramp} \geq v_{in}^*$ 时停止计数 | 简单，慢（最坏 $2^N T$），依赖斜坡线性度 |
| **双斜率 (Dual-Slope)** | 先积分 $v_{in}^*$ 固定 $N_{REF}$ 周期，再反向积分 $V_{REF}$ 直到回零 | $N_{out} = N_{REF} \cdot \frac{v_{in}^*}{V_{REF}}$，精确，不依赖斜坡斜率/比较器阈值/时钟频率，但最坏需 $2(2^N)T$ |

### 中速 ADC

#### 逐次逼近 (SAR) ADC
- 核心：比较器 + DAC + SAR 逻辑。
- 流程：从 MSB 开始，猜测 bit=1，DAC 输出与 $v_{in}^*$ 比较，高则保留 1 否则清零，逐位推进，N 个时钟完成 N 位。
- 典型实现：电压缩放 (MSB) + 电荷缩放 (LSB) DAC 组合 + 自调零比较器。

#### Pipeline Algorithmic ADC
- $N$ 级流水线，每级：$\times 2 \pm V_{REF}$。
- 输出数字码延迟 $N$ 个时钟，但每个时钟出一个新结果。
- $V_i = 2V_{i-1} - b_i V_{REF}$，其中 $b_i = \begin{cases} +1 & V_{i-1} \geq 0 \\ -1 & V_{i-1} < 0 \end{cases}$
- **误差分析**：第一级增益误差最敏感；输入电压状态影响容错能力（$\pm1$、$0$ 附近最鲁棒，中间值最脆弱）。

#### 迭代 Algorithmic ADC
- 只用一个 $\times 2$ 放大器和一个 S/H，反复循环。硬件最少，精度由单一放大器决定。

#### 自校准 ADC
- 校准阶段逐一测量每位电容的失配误差，存储为数字校正量。
- 正常转换时通过校准 DAC 施加补偿电压，可显著提高 SAR ADC 分辨率。

### 高速 ADC

#### Flash (全并行) ADC
- $2^N - 1$ 个比较器同时工作，一个时钟周期完成转换。
- 架构：$V_{REF}$ 经电阻串产生 $2^N$ 个参考电平，各接一个比较器的同相端，输入 $v_{in}^*$ 接所有比较器反相端。
- **限制**：比较器数量随 N 指数增长（6 位需 63 个），输入电容大、带宽受限，功耗高，比较器失调导致 $DNL$ 和缺失码。
- 典型性能：6 位 @ 400 Msps（亚微米 CMOS）。

#### 插值 (Interpolating) ADC
- 减少输入端放大器数量，用电阻串插值产生中间参考电平。
- 保持 Flash 的一周期转换速度，但大幅降低输入电容。
- 需均衡各比较器路径延迟（加补偿电阻）。

#### 折叠 (Folding) ADC
- 粗量化器（$N_1$ 位）+ 折叠预处理器（将 $2^{N_1}$ 个子区间映射到同一子区间）+ 细量化器（$N_2$ 位）。
- 比较器总数 $\approx 2^{N_1} + 2^{N_2}$ vs Flash 的 $2^{N_1+N_2}$，大幅减少比较器。
- **折叠+插值 (Folding & Interpolating)** 是当前高速 CMOS ADC 的主流架构，典型 8 位 @ 100-400 Msps。

#### 多比特 Pipeline ADC
- 每级 $K$ 位 ADC + DAC，残差放大 $2^K$ 后送入下一级。
- **数字纠错 (Digital Error Correction)**：后级增加额外位，可纠正前级比较器误判。显著降低比较器失调要求。
- **Subranging 替代残差放大**：将后级参考电压除以 $2^K$ 而非放大残差，避免高频放大器带宽限制。
- 典型：10 位 @ 40 Msps，14 位 @ 10 Msps。

#### 时间交织 (Time-Interleaved) ADC
- $M$ 个 ADC 并行，分时交替采样，等效采样率提高 $M$ 倍。
- 挑战：各通道增益、失调、延迟必须高度匹配。

---

## 过采样转换器 (Oversampling Converters)

### 核心思想

以**时间精度换幅度精度**——采样率远高于 Nyquist 率，配合噪声整形和数字抽取滤波，用粗糙量化器实现高分辨率。

- **过采样率** $M = \frac{f_S}{2f_B}$（$f_S$ 采样率，$f_B$ 信号带宽）
- **噪声整形**：通过反馈环路中的积分器将量化噪声推向高频，信号带内噪声大幅降低。

### Delta-Sigma ($\Delta\Sigma$) 调制器

**一阶调制器**：
$$
Y(z) = z^{-1}X(z) + (1 - z^{-1})Q(z)
$$
- STF (信号传递函数) = $z^{-1}$（纯延迟）
- NTF (噪声传递函数) = $1 - z^{-1}$（高通特性，在直流处有零点）

**$L$ 阶调制器**：NTF = $(1 - z^{-1})^L$，信号带内噪声功率：
$$
n_0 \approx \frac{\Delta^2}{12} \cdot \frac{\pi^{2L}}{(2L+1)} \cdot \frac{1}{M^{2L+1}}
$$
其中 $\Delta$ 为量化台阶 ($2 V_{REF}$ for single-bit)。

**动态范围**：$DR_{dB} \approx 6.02B - 3.41 + 10(2L+1)\log_{10} M$（$B$ 为内部量化器位数）

**关键结论**：
- 每加倍 $M$：一阶提升 9 dB（1.5 bit），二阶提升 15 dB（2.5 bit），三阶提升 21 dB（3.5 bit）。
- 每增加 1 位内部量化器：提升 6 dB。

**稳定性**：
- 单环 $L>2$ 时（1-bit 量化器）会不稳定，需使用分布式反馈 (DFB)、分布式前馈 (DFF) 或级联 (MASH) 结构。
- 多比特量化器可提高稳定性并降低过采样率，但需解决反馈 DAC 的线性度问题（动态元件匹配 DWA 等技术）。

### 抽取滤波 (Decimation Filter)
- 多级实现：梳状滤波器 (Comb filter) + FIR/IIR 滤波器。
- Comb filter（$L+1$ 级级联）：$H_D(z) = \left(\frac{1 - z^{-D}}{1 - z^{-1}}\right)^{L+1}$，零点置于抽取后 Nyquist 频率的谐波位置，无需乘法器。

### $\Delta\Sigma$ DAC
- 数字插值 + 数字 $\Delta\Sigma$ 调制器降至 1-bit + 1-bit DAC + 模拟低通滤波。
- 1-bit DAC 天然线性（只有两个电平），解决了多比特 DAC 的匹配问题。

---

## 设计启示

1. **MSB 精度决定整体精度**：DAC 和 ADC 的 MSB 元件容差必须在 $\pm 0.5\,\text{LSB}$ 以内，位数越高挑战越大（10 位 MSB 需 $<0.1\%$ 精度）。
2. **电荷缩放 + 电压缩放的组合是 CMOS DAC 的最优解**：MSB 用电荷缩放（电容匹配好于电阻），LSB 用电压缩放（保证单调）。
3. **SAR ADC 是面积和速度的最优折中**：N 个周期换 N 位精度，适配多数中速应用。
4. **Pipeline ADC 的误差按级递减**：第一级要求最严（增益 2 精度影响最大），后级逐级放宽，是数字纠错的基础。
5. **Flash ADC 只适合低分辨率**：比较器数量随位数指数增长，6-8 位为 CMOS 实用上限。
6. **$\Delta\Sigma$ 是 CMOS 实现高分辨率的首选方案**：利用数字电路的精度和速度优势，模拟部分极简，对元件匹配不敏感。
7. **S/H 电路常是 ADC 速度瓶颈**：运放 GBW 和建立时间直接决定最高采样率。
8. **孔径抖动在高频时严重限制 SNR**：对于 $f_{in}=250\text{MHz}$、8 位分辨率，时钟精度需 $<1\text{ps}$。

---

## 章节关联

- [[Ch08 - Comparators]] — 比较器是 Flash ADC、SAR ADC、$\Delta\Sigma$ ADC 的核心组件，其失调、速度、亚稳态直接限制 ADC 性能。
- [[Ch06 - CMOS Operational Amplifiers]] — 运放的增益、GBW、建立时间、摆率是 DAC 输出级和 S/H 电路的设计基础。
- [[Ch04 - Analog CMOS Subcircuits]] — 精密参考电压源是 DAC/ADC 精度的基准，本章未展开讨论。
- [[Ch07 - High-Performance CMOS Op Amps]] — 折叠共源共栅和全差分运放在高速 ADC 和 $\Delta\Sigma$ 调制器中被广泛采用。
- [[Appendices - Device Characterization, Time-Frequency Relations, Switched Capacitor Circuits]] — 开关电容电路是电荷缩放 DAC、S/H 电路和 $\Delta\Sigma$ 调制器实现的基础。

**See Also**：[[../Razavi/Ch13 - Introduction to Switched-Capacitor Circuits]]（Razavi 教材 SC 电路章节，与本节 S/H 和电荷缩放结构互补）

---

## 检索关键词

`DAC` `ADC` `数据转换器` `量化噪声` `SNR` `ENOB` `INL` `DNL` `单调性` `电流缩放` `电压缩放` `电荷缩放` `R-2R` `二进制加权` `SAR` `逐次逼近` `Pipeline ADC` `Flash ADC` `折叠插值` `Folding` `Interpolating` `采样保持` `S/H` `孔径抖动` `Aliasing` `过采样` `Delta-Sigma` `$\Delta\Sigma$` `噪声整形` `NTF` `STF` `MASH` `抽取滤波` `Comb Filter` `数字纠错` `Subranging` `时间交织`

---

Sources: [[raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e.md]]
Updated: 2026-08-02
