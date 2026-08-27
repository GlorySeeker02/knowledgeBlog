---
title: "Ch06 - CMOS Operational Amplifiers"
source: "Allen-CMOS analog circuit design 3e"
tags: [analog-design, CMOS, op-amp, operational-amplifier]
---
# Ch06 - CMOS 运算放大器

## 本章定位

本章系统介绍无缓冲（unbuffered）CMOS 运算放大器的设计与补偿理论。所谓"无缓冲"运放本质上属于运算跨导放大器（OTA），输出阻抗较高，适合驱动纯容性负载。本章是整个模拟集成电路设计课程的核心——运放是几乎所有模拟系统的基础构建块。

全章围绕两条主线展开：
- **经典两级运放**（two-stage op amp）——结构简单、鲁棒性强，是理解运放设计的起点；
- **折叠共源共栅运放**（folded-cascode op amp）——针对两级运放 PSRR 不足的改进方案，具备自补偿特性。

## 核心概念

### 1. 运算放大器的基本原理

运放的本质是**增益足够高的放大器 + 负反馈**。当开环增益足够大时，闭环传输函数几乎完全由反馈网络决定，与运放本身增益无关。

- **Null Port（虚端口）概念**：当负反馈施加且增益趋于无穷时，运放输入端口成为 null port——电压差为零、电流也为零。若一端接地，则另一端成为虚地（virtual ground）。
- **理想运放**：无限差分增益、无限输入阻抗、零输出阻抗。
- **实用增益要求**：无缓冲 CMOS 运放通常 2000 倍（66 dB）以上的开环增益已足够。

### 2. 运放的非理想特性模型

运放的完整非理想模型（Fig. 6.1-5）包含：

| 非理想因素 | 建模方式 | CMOS 运放中的重要性 |
|---|---|---|
| 有限差分输入阻抗 | _Rid_, _Cid_ | 不重要（MOS 栅极高阻） |
| 输出阻抗 | _Rout_ | 重要（无缓冲运放输出阻抗高） |
| 共模输入阻抗 | _Ricm_ | 不重要 |
| 输入失调电压 | _VOS_ | 重要（系统失调+随机失配） |
| 共模抑制比 | CMRR（受控源 _v₁_/CMRR） | 重要 |
| 噪声 | _e²n_ (电压), _i²n_ (电流) | 重要（1/_f_ 噪声+热噪声） |

CMOS 运放的优势在于：_Rid_ 极高（~10¹⁴ Ω）、_I_OS 和 _I_B 近似为零。

### 3. 频率响应与稳定性

运放开环传递函数的一般形式：

$$
A_v(s) = \frac{A_v(0)}{\left(1 - \frac{s}{p_1}\right)\left(1 - \frac{s}{p_2}\right)\cdots}
$$

关键频域指标：
- **_GB_**（Unity-Gain Bandwidth）：单位增益带宽，即开环增益下降到 0 dB 时的频率。
- **主极点 _p₁_**：使增益以 -6 dB/oct 下降的最低频率极点。
- **相位裕度 PM**：当环路增益 |_A(jω)F(jω)_| = 1 时，相位与 -180° 的距离。典型要求 PM > 45°，推荐 60°。

> PM 越大，闭环阶跃响应的过冲和振铃越小。60° PM 对应单位阶跃响应约 10% 过冲。

### 4. 补偿的必要性

未补偿的两级运放有两个靠得很近的极点（分别对应第一级和第二级输出节点），导致环路增益在 0 dB 穿越前就积累了接近 -180° 的相移——几乎没有相位裕度。补偿的目标是**极点分裂**（pole splitting）：将主极点推向低频、输出极点推向高频，使其在 _GB_ 处仅受主极点支配。

## 关键公式与结论

### 两级运放基本关系式（Fig. 6.3-1）

| 参数 | 表达式 | 说明 |
|---|---|---|
| 单位增益带宽 | $$GB = \frac{g_{m1}}{C_c}$$ | _g_m1_ 为输入对管跨导 |
| 摆率 | $$SR = \frac{I_5}{C_c}$$ | _I_5 为尾电流 |
| 直流增益 | $$A_v(0) = \frac{g_{m1}g_{m6}}{G_I G_{II}}$$ | _G_I_ = _g_ds2_+_g_ds4_, _G_II_ = _g_ds6_+_g_ds7_ |
| 输入共模范围（正） | $$V_{in}(max) = V_{DD} - V_{SG3} + V_{TN1}$$ | 由 M3 的源栅压限 |
| 输入共模范围（负） | $$V_{in}(min) = V_{SS} + V_{DS5}(sat) + V_{GS1}$$ | 由 M5 饱和+ M1 栅源压限 |

### Miller 补偿的极点与零点

加入 Miller 电容 _Cc_ 后（Fig. 6.2-6）：

$$
p_1 \approx \frac{-1}{g_{mII}R_{II}R_I C_c} \quad \text{(Miller 极点)}
$$

$$
p_2 \approx \frac{-g_{mII}}{C_{II}} \quad \text{(输出极点)}
$$

$$
z_1 = \frac{g_{mII}}{C_c} \quad \text{(RHP 零点)}
$$

**RHP 零点**是 CMOS 运放的关键问题。它来自 _Cc_ 的信号前馈路径与 M6 的放大路径在高频处等幅反相抵消。该零点同时**增大增益幅度**和**增加相位滞后**，严重恶化稳定性。BJT 运放中因跨导大，此零点位置很高影响较小；CMOS 中跨导较小，必须处理。

### 60° 相位裕度的设计要求

假设 RHP 零点 _z₁_ >= 10 _GB_：

$$
|p_2| \geq 2.2 \cdot GB
$$

$$
\frac{g_{mII}}{C_c} > 10 \cdot GB
$$

$$
\Rightarrow C_c \geq 0.22 \cdot C_L
$$

### RHP 零点的三种处理方法

| 方法 | 原理 | 优缺点 |
|---|---|---|
| **单位增益缓冲器** | 在 _Cc_ 反馈路径插入 buffer，阻断前馈 | 消除零点，但 buffer 输出阻抗引入额外极点 |
| **调零电阻 _Rz_** | _Rz_ 与 _Cc_ 串联 | 灵活控制零点位置：消除（_Rz_=1/_g_mII），或移至 LHP 抵消 _p₂_ |
| **增益反馈路径** | 在 _Cc_ 反馈路径加 M8 增益级 | 输出极点增大约 _g_m _r_ds_ 倍，但仍有 RHP 零点 |

**调零电阻实现**：用偏置在线性区的 MOS 管 M8（Fig. 6.3-4），其电阻值通过偏置电压 _V_A = _V_B_ 保证与 1/_g_m6 匹配。

### 镜像极点 _p₃_

由于电流镜负载 M3 的 _C_gs3_ 和 _C_gd3_，输入级存在额外的镜像极点 _p₃_ 及其关联零点 _z₃_ = -2_p₃_：

$$
p_3 \approx \frac{-g_{m3}}{C_{gs3}+C_{gd3}}
$$

一般而言，若 _p₃_ > 10 _GB_，其对稳定性影响可忽略——相关零点会部分抵消极点的影响。

### PSRR 分析

两级运放的 PSRR 存在**不对称性**：

- **PSRR⁺**（正电源）：较差。原因是 M6 的栅-源电压必须保持恒定，_V_DD 的纹波通过 M6 栅极经 _Cc_ 耦合到输出。直流值约 68.8 dB（例 6.3-1），在高频处以 -20 dB/dec 滚降。

$$
PSRR^+ \approx \frac{G_{II}A_v(0)}{g_{ds6}} \cdot \frac{\left(1 + \frac{s}{GB}\right)\left(1 + \frac{s}{|p_2|}\right)}{\left(1 + \frac{s}{GB/A_v(0)}\right)\left(1 + \frac{s}{|p_2|}\right)}
$$

- **PSRR⁻**（负电源）：强烈依赖于 _V_BIAS 的接法。若 _V_BIAS 独立于 _V_SS（由与电源无关的电流源产生），PSRR⁻ 远优于 PSRR⁺。

> 折叠共源共栅运放的 PSRR 显著改善——没有 Miller 电容耦合路径，电源纹波主要通过 _C_gd 耦合，影响小得多。

## 重要运放结构

### 1. 经典两级运放（Fig. 6.3-1）

**结构**：n-channel 输入差分对 + p-channel 电流镜负载（第一级）+ p-channel 共源放大器 + n-channel 电流沉负载（第二级）+ Miller 补偿电容 _Cc_。

$$
V_{in} \xrightarrow{V\to I} \text{[差分对]} \xrightarrow{I\to V} \text{[电流镜]} \xrightarrow{V\to I} \text{[共源级 M6]} \xrightarrow{I\to V} \text{[电流沉 M7]} = V_{out}
$$

**特点**：
- 结构简单、鲁棒、广泛使用
- 输出级为 Class A（只能主动 sourcing 或 sinking）
- PSRR⁺ 较差
- 增益 ~ (_g_m _r_ds_)² 量级

### 2. 折叠共源共栅运放（Fig. 6.5-9）

**结构**：n-channel 差分对 + p-channel 电流源负载（折叠节点）+ 共源共栅电流镜输出级。

**关键创新**——**折叠**：差分对漏极连接到 PMOS 电流源（非电流镜），信号电流被"折叠"到相反极性方向，再通过 cascode 电流镜合成到单端输出。

**特点**：
- **输入共模范围**更宽：正 ICMR 可达 _V_DD + (_V_TN_ - _V_SD3_)，可超过电源轨
- **自补偿**：主极点由输出节点的高阻和 _C_L_ 决定，无需 Miller 电容
- **PSRR 极好**：没有 Miller 电容从电源到输出的前馈路径
- **增益**与两级运放相当：~ _g_m _r_ds_² 量级，具体为

$$
A_v \approx \left(\frac{2+k}{2(1+k)}\right) g_{m1} R_{out}
$$

其中 _k_ 为电流分配不平衡因子，_R_out_ ≈ cascode 输出阻抗。

- **六个极点**：主极点在输出，非主极点位于折叠节点和 cascode 的源端。M8 源端的极点因 M10 的并联反馈而推到很高频率。

**设计要点**（Table 6.5-1）：
- _I₃_ = _SR_ × _C_L_（差分对尾电流）
- _I₄_ = _I₅_ = (1.2~1.5) × _I₃_（避免 cascode 电流归零）
- _S₁_ = _S₂_ = _g²_m1_ / (_K'_N _I₃_)，其中 _g_m1_ = _GB_ × _C_L_
- _S₄_、_S₅_、_S₆_、_S₇_、_S₈_、_S₉_、_S₁₀_、_S₁₁_ 依次由输出电压范围和输入共模范围确定

### 3. 增益增强型折叠共源共栅运放（Fig. 6.5-14）

在 cascode 管栅极加入反相放大增益级 _-A_，将输出阻抗提升 _A_ 倍：

$$
R_{out} \approx A \cdot g_m r_{ds}^2
$$

增益可达 (_g_m _r_ds_)³ 量级。增强放大器本身通常采用简单的反相器结构（Fig. 6.5-15），注意增强放大器的主极点必须远高于运放的主极点。

### 4. Cascode 第一级 / 第二级

- **Cascode 第一级**（Fig. 6.5-1）：增益 ~ (_g_m _r_ds_)²，自补偿。需要浮动电池偏置 cascode 管栅极。
- **Cascode 第二级**（Fig. 6.5-5）：增益更高 ~ (_g_m _r_ds_)³，但因使用 Miller 补偿，PSRR 仍然较差。

### 两级运放设计流程（Table 6.3-2）

1. 选择最小沟道长度（保证 λ 恒定和匹配）
2. _Cc_ >= 0.22 _C_L_（60° PM）
3. _I₅_ = _SR_ × _Cc_
4. _(W/L)₃_ 由正 ICMR 确定
5. 验证 _p₃_ > 10 _GB_
6. _g_m1_ = _GB_ × _Cc_ → _(W/L)₁_ = _(W/L)₂_
7. _(W/L)₅_ 由负 ICMR 确定
8. _g_m6_ ≈ 2.2 _g_m2_(_C_L_/_Cc_) → 两种策略确定 _(W/L)₆_ 和 _I₆_
9. _(W/L)₇_ = (_I₆_/_I₅_) × _(W/L)₅_
10. 检查增益和功耗
11. 若噪声不达标：增大器件面积（降 1/_f_ 噪声）、增大 _g_m_（降热噪声）、降低 _g_m3_/_g_m1_ 比例

## 设计启示

1. **80-20 法则**：手工计算约占 20% 时间，完成 80% 设计工作；剩余 20% 微调需 80% 时间。手工计算让设计者**理解参数敏感性**，纯靠仿真迭代无法获得这种直觉。

2. **增益提升策略**：增大 _r_out（通过 cascode 或减小电流）比增大 _g_m_（需增加电流的平方根关系）更**功耗高效**。

3. **版图即设计**：CMOS 模拟 IC 的物理设计直接影响电气性能：
   - **单位匹配原则**（unit-matching）：用相同单元 copy-paste 保证匹配
   - **共质心布局**（common-centroid）：用于非整数比器件对
   - **环形晶体管**（donut transistor）：减小 _C_gd_
   - **远端偏置**：将基准电压转为电流传输到芯片各处，在目的地再生偏置电压
   - **分离电源总线**：模拟和数字电路绝不共用电源线

4. **仿真前先估算寄生**：在版图前粗略估算源漏面积和周长（_AD_ = _AS_ = (_L1+L2+L3_) × _W_），在仿真中计入结电容，减小版图前后差异。

5. **注意负摆率不对称**：两级运放中，正摆率由 _I₅_ 决定，负摆率可能受限于 _C_L_ 放电能力。输出级的 sink 电流减去 _Cc_ 放电电流后才是驱动 _C_L_ 的净电流。

## 章节关联

- **前序章节**：
  - [[../Razavi/Ch03 - Single-Stage Amplifiers|Razavi Ch03 / Allen Ch05]] — 单级放大器是运放各级的构建基础
  - [[../Razavi/Ch04 - Differential Amplifiers|Razavi Ch04 / Allen Ch05-Sec5.2]] — 差分对是运放输入级
  - [[../Razavi/Ch05 - Current Mirrors and Biasing Techniques|Razavi Ch05 / Allen Ch04]] — 电流镜是运放负载和偏置的核心

- **后续章节**（Allen Ch07）：
  - 高性能 CMOS 运放（Class AB 输出、全差分、rail-to-rail 等）
  - 缓冲运放（低输出阻抗电压运放）

- **相关章节**（其他教材）：
  - [[../Razavi/Ch09 - Operational Amplifiers|Razavi Ch09]] — 运放设计，侧重补偿与两级/折叠结构
  - [[../Razavi/Ch10 - Stability and Frequency Compensation|Razavi Ch10]] — 频率补偿的深入分析
  - [[../Gray/Ch06 - Operational Amplifiers with Single-Ended Outputs|Gray Ch06]] — 单端输出运放，包含 BJT 和 CMOS

## 检索关键词

`op-amp` `OTA` `two-stage op amp` `folded-cascode` `Miller compensation` `pole splitting` `RHP zero` `nulling resistor` `phase margin` `slew rate` `gain bandwidth` `GB` `PSRR` `CMRR` `ICMR` `offset voltage` `self-compensation` `cascode op amp` `gain enhancement` `common-centroid` `unit matching` `settling time` `input common-mode range` `output swing`

## Sources

[[../../../raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e.md|Allen-CMOS analog circuit design 3e, Chapter 6]]

## See Also

- [[../Razavi/Ch09 - Operational Amplifiers|Razavi Ch09 - 运算放大器]]
- [[../Gray/Ch06 - Operational Amplifiers with Single-Ended Outputs|Gray Ch06 - 单端输出运放]]
