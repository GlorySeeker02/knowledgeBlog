---
title: "第1章 绪论与背景"
source: "Allen & Holberg, CMOS Analog Circuit Design, 3rd Ed., Ch1"
tags:
  - analog-design
  - CMOS
  - analog-IC
  - introduction
updated: 2026-08-02
---

# 第1章 绪论与背景

## 本章定位

本章是 Allen & Holberg《CMOS Analog Circuit Design》全书的总纲。它为后续所有章节提供了三个基础框架：

1. **概念框架** —— 模拟与数字信号的定义、分析与设计的区别、分立与集成的差异；
2. **符号框架** —— 全书统一的符号约定和器件图例；
3. **层次框架** —— 器件 → 简单电路 → 复杂电路 → 系统的四级层次化设计视角（Table 1.1-2）。

读完全书后回头再读本章，会发现它已经勾勒出整本书的结构蓝图——正如本章小结建议的："翻到每一章时，强烈建议先回顾 Table 1.1-2。"

## 核心概念

### 1.1 模拟集成电路设计

#### 三种信号类型

| 信号类型 | 时间 | 幅度 | 示例 |
|---------|------|------|------|
| **模拟信号** (Analog) | 连续 | 连续 | 传感器输出、语音信号 |
| **数字信号** (Digital) | 离散 | 离散（量化） | 二进制编码数据 |
| **模拟采样数据信号** (Sampled-data) | 离散 | 连续 | S/H 电路输出、SC 滤波器 |

数字信号是模拟信号的量化近似：

$$
A(t) = \sum_{i=0}^{N-1} b_i(t) 2^{-i} \approx A(t)
$$

其中 $b_i \in \{0, 1\}$。

> [!important] 分析与设计的根本区别
> - **分析 (Analysis):** 给定电路 → 求属性，解是**唯一的**。
> - **设计/综合 (Design/Synthesis):** 给定目标属性 → 找电路，解是**不唯一的**。
>
> 这正是设计的创造性所在：一个 1.5V 电阻可以用串联、并联等多种方式实现。

#### 分立 vs. 集成的关键差异

| 差异维度 | 分立设计 | 集成电路设计 |
|---------|---------|------------|
| 器件匹配 | 匹配困难，依赖外部元件 | 同基板上相邻器件天然匹配，可将匹配作为设计工具 |
| 几何控制 | 不可控 （封装的固定尺寸） | 完全可控，增加设计自由度 |
| 原型验证 | 可面包板调试 | 不可面包板，必须依赖计算机仿真 |
| 可用元件 | 丰富的分立元件库 | 受工艺限制的有限元件类 |

#### 模拟 IC 设计流程

设计过程分为七个主要步骤:

1. **定义 (Definition)** —— 明确设计指标
2. **综合/实现 (Synthesis/Implementation)** —— 确定电路拓扑
3. **仿真/建模 (Simulation/Modeling)** —— 前仿验证功能
4. **几何描述 (Geometrical Description)** —— 版图绘制
5. **含寄生仿真 (Simulation with Parasitics)** —— 提取寄生，后仿
6. **制造 (Fabrication)** —— 投片（设计师不直接参与）
7. **测试与验证 (Testing and Verification)** —— 芯片回片测试

设计师**负责除制造外的所有步骤**。前仿和后仿之间通常需要多次迭代。

> [!tip] 计算机仿真的优劣
> **优点：**
> - 无需面包板
> - 可监测电路内任意节点
> - 可断开反馈环路
> - 可方便修改电路
> - 可在不同工艺角和温度下分析
>
> **缺点：**
> - 模型精度有限
> - 收敛性问题
> - 大规模电路仿真时间长
> - 容易用计算机替代理性思考

#### 设计的三种描述格式

设计与图 1.1-3 的流程相对应，设计师需要在三种描述间切换：

| 描述格式 | 内容 | 对应阶段 |
|---------|------|---------|
| **设计描述 (Design)** | 规格说明、电压/电流关系 | 定义与综合 |
| **物理描述 (Physical)** | 版图几何、多边形数据库 | 几何描述 |
| **模型/仿真描述 (Model/Simulation)** | 器件模型、行为模型、宏模型 | 仿真验证 |

#### 层次化设计视角

全书的组织核心，贯穿始终的四级层次：

$$
\boxed{\text{器件 Device}} \rightarrow \boxed{\text{简单电路 Simple}} \rightarrow \boxed{\text{复杂电路 Complex}} \rightarrow \boxed{\text{系统 System}}
$$

| 层次 | 设计描述 | 物理描述 | 模型描述 | 对应章节 |
|------|---------|---------|---------|---------|
| **系统** | 系统指标 | 芯片平面规划 (Floor plan) | 行为级模型 | Ch9, App E |
| **复杂电路** | 电路指标 | 参数化单元 | 宏模型 | Ch6-8 |
| **简单电路** | 电路指标 | 参数化布局 | 宏模型 | Ch4-5 |
| **器件** | 器件指标 | 几何描述 | 器件模型 | Ch2-3, App C |

### 1.2 符号约定

#### 信号命名规则（全书统一）

这是理解后续所有小信号分析的基础。Allen 的约定与标准教材一致：

| 信号含义 | 物理量 | 下标 | 示例 |
|---------|--------|------|------|
| 总瞬时值 | 小写 | 大写 | $i_D$ （含直流+交流） |
| 直流量 (dc) | 大写 | 大写 | $I_D$ （静态工作点） |
| 交流量 (ac) | 小写 | 小写 | $i_d$ （小信号分量） |
| 复变量/相量/rms | 大写 | 小写 | $I_d$ （频域分析） |

> [!note] 应用场景
> - 大信号建模：用 $i_D$ （总瞬时值）
> - 偏置设计：用 $I_D$ （直流量）
> - 小信号分析：用 $i_d$ （交流量）
> - 频率响应：用 $I_d$ （复变量）

#### MOS 器件符号

- 增强型 NMOS/PMOS，衬底接源极 → 四端子简化为三端子符号
- 衬底未接源极 → 显式画出 **B** (Bulk) 端
- p 沟道衬底通常接最高电位，n 沟道接最低电位

#### 受控源符号

| 类型 | 符号 | 增益 |
|------|------|------|
| 电压控制电压源 (VCVS) | 菱形 | $A_v$ |
| 电压控制电流源 (VCCS) | 菱形 | $G_m$ |
| 电流控制电压源 (CCVS) | 菱形 | $R_m$ |
| 电流控制电流源 (CCCS) | 菱形 | $A_i$ |

MOSFET 的小信号模型本质上就是 VCCS（$g_m v_{gs}$）。

### 1.3 模拟信号处理

#### 典型信号处理链路

$$
\boxed{\text{预处理（模拟）}} \rightarrow \boxed{\text{数字信号处理器}} \rightarrow \boxed{\text{后处理（模拟）}}
$$

- **预处理**：滤波器 + AGC + A/D 转换器。速度与精度要求最苛刻。
- **数字处理**：可在最小尺寸工艺下实现，线性相位滤波器、可编程等优势。
- **后处理**：D/A 转换 + 放大 + 滤波。

> [!important] 模拟/数字边界决策
> 输入几乎总是模拟的（传感器、天线等），但处理在哪儿数字化取决于：
> - 应用需求
> - 性能指标
> - 芯片面积
>
> **趋势**：尽可能用 CMOS 数字 + CMOS 模拟（按需），实现高集成度、高可靠的紧凑系统方案。

#### 信号频段与工艺选择

- **地震信号** < 1 Hz 到 **微波** ~30 GHz
- 工艺速度能力排序：光互连 > GaAs > 双极模拟 > SiGe BiCMOS > MOS 数字 > MOS 模拟
- 选工艺的考量：带宽、速度、成本、集成度

### 1.4 设计实例：磁盘驱动读写通道

这是本章篇幅最大的部分（占据约 5 页），通过一个完整的商业芯片（0.8 um 双金属 CMOS，支持 64 Mbits/s）展示全书的层次化设计方法如何落地。

**读路径链路：**

$$
\boxed{\text{VGA}} \rightarrow \boxed{\text{7极2零低通滤波器}} \rightarrow \boxed{\text{6-bit Flash ADC}} \rightarrow \boxed{\text{FIR滤波器}} \rightarrow \boxed{\text{序列检测器 (Viterbi)}} \rightarrow \boxed{\text{RLL译码}}
$$

**关键技术要点：**

- **低通滤波器**：跨导单元 ($g_m$) + 电容实现，频率响应通过 $V_{\text{CON}}$（模拟调谐）和数字控制电容阵列（两级调谐）两个机制标定
- **主 PLL 调谐**：用滤波器的复制品构成 VCO，锁相于外部晶振，消除工艺/温度/电源变化
- **Flash ADC**：63 个比较器，电容采样同时吸收比较器失调电压（自零消失调）
- **时序恢复**：数字环控制 ADC 的 VCO 频率
- **序列检测**：Viterbi 算法实现最大似然检测，预补偿消除非线性符号间干扰
- **伺服通道**：独立 AGC 环、位检测器、Burst 解调器，全部由主滤波器调谐，伺服间隙断电省功耗

**频率合成器**：
$$
f_{\text{write clock}} = \frac{M}{2N} \times f_{\text{reference}}
$$

$M, N \in [2, 256]$，支持分区位记录 (ZBR)。

> [!summary] 这个例子展示了什么？
> 1. **层次化设计** —— 从 VGA、滤波器、ADC 等子电路到完整读写通道系统
> 2. **模拟-数字协同** —— 数字环控制模拟 VGA 增益和 ADC 频率
> 3. **片上校准** —— 主 PLL 自动调谐，消除 PVT 变化
> 4. **混合信号 SoC** —— 模拟前端 + 数字处理 + 频率合成 + 伺服通道集于一身

### 1.5 小结

本章作为绪论，完成了三件事：
1. 定义了模拟、数字、采样数据三种信号，辨析了分析与设计、分立与集成的本质区别；
2. 建立了全书统一的符号体系（Table 1.2-1）；
3. 用读写通道芯片实例展示了层次化设计方法如何在工程中生效。

> [!quote] 章节建议
> "It is strongly recommended that the reader refer to Table 1.1-2 at the beginning of each chapter." ——Allen & Holberg

在进入第 2 章之前，建议复习：
- 附录 A：模拟电路分析方法（节点法、网孔法、米勒定理等）
- 电子器件建模基础
- 拉普拉斯变换与 z 变换
- 半导体器件物理

## 关键公式与结论

| 公式/结论 | 含义 |
|-----------|------|
| $A(t) = \sum b_i 2^{-i}$ | 数字信号是模拟信号的量化近似 |
| $i_D = I_D + i_d$ | 总瞬时值 = 直流量 + 交流小信号 |
| `小写大写` 下标 → 总瞬时值；`大写大写` → dc；`小写小写` → ac；`大写小写` → 相量 | 全书信号命名铁律 |
| 分析是唯一的，设计是不唯一的 | 设计需要创造性 |
| 集成设计不可面包板 → 必须依赖仿真 | 仿真能力 = 设计能力的一部分 |
| 器件→简单电路→复杂电路→系统 | 全书四层结构 |
| $f_{\text{write}} = \frac{M}{2N} f_{\text{ref}}$ | 频率合成公式 |

## 设计启示

1. **匹配是集成设计的核心武器** —— 在同一基板上，相邻器件的匹配远超分立元件，应善加利用。
2. **掌握几何控制权** —— 在 IC 设计中 W/L 是你手中的设计变量，分立设计中没有这个自由度。
3. **仿真可以帮你，也可能害你** —— 永远先手工分析理解电路，再用仿真验证；不要用仿真"代替思考"。
4. **建立层次化思维** —— 拿到一个新电路，先定位它在 Table 1.1-2 中的位置。复杂的电路由哪些简单电路组成？它服务于哪个系统级功能？
5. **混合信号 SoC 是大趋势** —— 模拟前端 + 数字处理 + 片上自校准，是当前 IC 设计的主流范式。

## 章节关联

- **Ch2 (CMOS Technology)** 和 **Ch3 (CMOS Device Modeling)** —— 对应 Table 1.1-2 的 **器件层**，提供工艺基础和器件模型
- **Ch4 (Subcircuits)** 和 **Ch5 (Amplifiers)** —— **简单电路层**，构成复杂电路的积木
- **Ch6-8 (Op Amps, High-Performance Op Amps, Comparators)** —— **复杂电路层**
- **Ch9 (D/A and A/D Converters)** 和 **Appendix E (Switched Capacitor Circuits)** —— **系统层**
- **Appendix A (Circuit Analysis)** —— 本章习题引用的分析方法基础
- **Appendix C (CMOS Device Characterization)** —— 器件模型参数提取，与 Ch3 互补

## 检索关键词

`analog signal definition` `digital signal definition` `sampled-data signal` `analysis vs synthesis` `design vs analysis` `discrete vs integrated circuit design` `analog IC design flow` `design hierarchy` `Table 1.1-2` `MOS symbol notation` `signal subscript convention` `total instantaneous variable` `dc variable` `ac variable` `phasor variable` `VCVS` `VCCS` `CCVS` `CCCS` `analog signal processing` `bandwidth` `CMOS technology capability` `read/write channel` `PRML` `master PLL` `Viterbi algorithm` `write precompensation` `frequency synthesizer` `gm-C filter`

## Sources

[[raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e.md]]

## See Also

- [[wiki/模拟集成电路/Razavi/Ch01 - Introduction to Analog Design]]
- Gray & Meyer, *Analysis and Design of Analog Integrated Circuits*, Ch1
