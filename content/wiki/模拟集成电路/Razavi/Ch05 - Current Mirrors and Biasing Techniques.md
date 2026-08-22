---
title: "Ch05 — Current Mirrors and Biasing Techniques"
source: "Razavi, Design of Analog CMOS Integrated Circuits, 2nd Edition, Chapter 5"
tags:
  - analog-design
  - CMOS
  - current-mirrors
  - biasing
date: 2026-08-02
---

# Ch05 — 电流镜与偏置技术

## 本章定位

本章在 Ch03（单级放大器）和 Ch04（差分放大器）已经大量使用电流源作为负载的基础上，系统讲解电流源本身如何设计。核心问题：如何产生一个**精确、稳定、高输出阻抗**的电流源，以及如何用它偏置各种放大器拓扑。

全章围绕两条主线展开：
1. **电流镜**（5.1--5.3）：从基本结构到高输出阻抗级联，再到有源电流镜作为差分对的负载（五管 OTA）。
2. **偏置技术**（5.4）：如何在实际电路中为 CS、CG、源跟随器、差分对提供直流工作点。

> [!NOTE] 本章范围说明（Razavi 第二版）
> 本章 **未涉及** 带隙基准、PTAT 电流生成、启动电路等参考源设计内容——这些在 Ch12（Bandgap References）中处理。本章聚焦于电流"复制"机制和放大器偏置技术。

## 核心概念

### 5.1 基本电流镜

#### 为什么电阻偏置不行

最简单的偏置方式：电阻分压提供栅极电压（Fig. 5.2）。问题在于：

$$
I_{out} = \frac{1}{2}\mu_n C_{ox}\frac{W}{L}(V_{GS} - V_{TH})^2
$$

- $V_{GS}$ 依赖 $V_{DD}$ 和 $V_{TH}$，而 $V_{TH}$ 在不同晶圆间可能变化 50--100 mV
- $\mu_n$ 和 $V_{TH}$ 均随温度变化
- 过驱动电压越小（为了 headroom），对 $V_{TH}$ 变化的敏感度越高：若过驱动 200 mV，$V_{TH}$ 变化 50 mV 会导致 **44% 的电流误差**

> [!important] 关键洞察
> 即使栅-源 **电压** 被精确定义，漏极 **电流** 也不精确！因为 MOSFET 的 $I_D$-$V_{GS}$ 关系受工艺和温度影响。因此必须采用电流"复制"策略。

#### 电流复制原理

基本思想（Fig. 5.3--5.5）：由一个精密的参考电流 $I_{REF}$ (由带隙基准等复杂电路生成，见 Ch12) 出发，通过电流镜"克隆"出需要的各路电流。

数学本质：
- 二极管接法的 $M_1$ 完成 $V_{GS}=f^{-1}(I_{REF})$（用 $I_{REF}$ **产生** $V_{GS}$）
- $M_2$ 完成 $I_{out}=f(V_{GS})=f[f^{-1}(I_{REF})]=I_{REF}$（用 $V_{GS}$ **还原** 电流）

忽略沟道长度调制效应时：

$$
I_{out} = \frac{(W/L)_2}{(W/L)_1} I_{REF}
$$

该比值的精度仅取决于 **器件尺寸比**（可以精确控制），与工艺和温度无关。

> [!warning] 因果关系的方向
> Fig. 5.6 那种用固定 $V_b$ 偏置的电路 **不是** 电流镜——$V_b$ 不由 $I_{REF}$ 引起，因此 $I_{out}$ 不会跟踪 $I_{REF}$。

#### 尺寸设计规则

1. **等长度原则**：所有镜像晶体管使用 **相同 $L$**，避免侧扩散 $L_D$ 引起的 $L_{eff}$ 误差。
2. **整倍数宽度**：通过复制 **单位晶体管** (unit transistor) 实现比例缩放，而非直接画 2 倍 $W$（gate corner 效应，Fig. 5.9）。
3. **串联实现分数比**：$I_{REF}/2$ 时用两个单位管串联（等效 $L_{eff}$ 加倍），而非用 $W/2$（Fig. 5.10）。

> [!tip] 设计启示
> 避免长链级联复制（copy of a copy），每级复制累积 mismatch 误差。

#### 电流镜也可处理信号

若 $I_{REF}$ 包含小信号分量 $\Delta I$，则 $I_{out}$ 产生 $\Delta I \cdot (W/L)_2/(W/L)_1$，即电流放大——代价是偏置电流也同比放大（Example 5.2）。

---

### 5.2 级联电流镜（Cascode Current Mirrors）

#### 沟道长度调制的问题

考虑 $\lambda$ 后：

$$
I_{out} = \frac{(W/L)_2}{(W/L)_1} \cdot \frac{1+\lambda V_{DS2}}{1+\lambda V_{DS1}} I_{REF}
$$

$V_{DS1}=V_{GS1}$ 是确定的，但 $V_{DS2}$ 由后级电路决定，不一定等于 $V_{DS1}$——这是镜像误差的主要来源。

#### 两种解决思路

| 思路 | 方法 | 代表拓扑 |
|------|------|---------|
| 强制 $V_{DS2}=V_{DS1}$ | 用 cascode 管屏蔽 $M_2$ 的漏极电压变化 | Fig. 5.12(c) 标准 cascode 镜 |
| 强制 $V_{DS1}=V_{DS2}$ | 在 $M_1$ 漏极串联电阻产生压降，使 $V_{DS1}$ 降低到与 $V_{DS2}$ 相等的低值 | Fig. 5.16(a)、Fig. 5.18(b) 低压 cascode |

#### 标准 Cascode 电流镜 (Fig. 5.12)

- $M_0$、$M_3$ 为 cascode 管，$V_b = V_{GS0}+V_{GS1} = V_{GS3}+V_{GS2}$
- 条件：$(W/L)_3/(W/L)_0 = (W/L)_2/(W/L)_1$
- 输出阻抗约 $g_{m3}r_{O3}r_{O2}$（与 Ch03 的 cascode 级相同）

**代价**：最小输出电压为 $V_{min} = 2V_{ov} + V_{TH}$，比理想 cascode 多"浪费"一个 $V_{TH}$（因为 $V_{DS2}=V_{GS2}$ 而非 $V_{GS2}-V_{TH}$）。

#### 低压 Cascode 电流镜 (Fig. 5.18)

核心思路：让 $M_1$ 也工作在饱和区边缘（$V_{DS1}=V_{GS1}-V_{TH1}$），而非二极管接法。

- $V_b$ 需满足 $V_{GS0}+(V_{GS1}-V_{TH1}) \leq V_b \leq V_{GS1}+V_{TH0}$
- 存在解的条件：$V_{GS0}-V_{TH0} < V_{TH1}$（即 $M_0$ 的过驱动远小于 $V_{TH1}$）
- 输出最小电压：$V_{min}=2V_{ov}$（不浪费 $V_{TH}$）

$V_b$ 的生成（Fig. 5.19）：
- Fig. 5.19(a)：$M_5$ 提供 $V_{GS}$ + $M_6$ 和电阻 $R_b$ 提供过驱动分量
- Fig. 5.19(b)：二极管接法的 $M_7$ 提供 $V_{GS}$ + $M_6$ 调整过驱动

#### 大信号行为 (Example 5.4)

当 $V_X$ 从高到低扫描时：
1. $V_X \geq V_N - V_{TH}$：$M_2$、$M_3$ 均饱和，$I_X = I_{REF}$
2. $V_X < V_N - V_{TH}$：**$M_3$ 先进入线性区**（而非 $M_2$），$V_B$ 下降，$I_X$ 略微减小
3. $V_X$ 进一步下降：$M_2$ 进入线性区，$I_X$ 急剧下降
4. $V_X = 0$：$I_X = 0$，两管均深线性区

### 5.3 有源电流镜——五管 OTA

#### 从无源负载到有源负载

差分对单端输出 + 电流源负载 (Fig. 5.23a)：$M_1$ 的小信号漏极电流被"浪费"了。

五管 OTA (Fig. 5.22/5.26)：$M_3$、$M_4$ 构成 PMOS 电流镜，将 $M_1$ 的电流变化**反相**后叠加到输出端——推挽效果使增益翻倍。

#### 大信号特性 (5.3.1)

- 输出摆幅受输入共模电平限制：$V_{out,min} = V_{in,CM} - V_{TH}$（$M_2$ 饱和条件）
- 平衡时（$V_{in1}=V_{in2}$）：$V_{out}=V_F=V_{DD}-|V_{GS3}|$
- **开环直流失调敏感**：极小的阈值失配就会导致 $V_{out}$ 大幅偏离——因此 OTA 很少用于开环放大微小信号，但常用于差分→单端转换 + 大摆幅信号

#### 小信号增益 (5.3.2)

**近似分析** (Fig. 5.32--5.33)：
- 节点 $F$ 阻抗低（二极管接法的 $1/g_{m3}$），可近似将 $P$ 视为虚地
- $G_m = g_{m1,2}$（两倍于无源负载版）
- $R_{out} = r_{O2} \parallel r_{O4}$
- $|A_v| \approx g_{m1,2}(r_{O2} \parallel r_{O4})$

**精确分析** (Fig. 5.34)：

$$
|A_v| = g_{m1}(r_{O1} \parallel r_{O4}) \cdot \frac{2g_{m4}r_{O4}+1}{2g_{m4}r_{O4}+2}
$$

校正因子 $\frac{2g_{m4}r_{O4}+1}{2g_{m4}r_{O4}+2} < 1$。若 $g_{m4}r_{O4}=5$，校正因子约 0.92。

#### 共模特性 (5.3.3)

**即使器件完美匹配，五管 OTA 的 CMRR 也有限**（与全差分不同）：

$$
A_{CM} = \frac{\partial V_{out}}{\partial V_{in,CM}} \approx \frac{1}{2g_{m3,4}R_{SS}}
$$

$$
\text{CMRR} = \frac{|A_{DM}|}{|A_{CM}|} \approx 2g_{m1,2} \cdot g_{m3,4} \cdot r_{O2}\parallel r_{O4} \cdot R_{SS}
$$

若 $R_{SS}=r_O$ 且 $2g_m r_O \gg 1$，则 CMRR $\approx (g_m r_O)^2$。

**$g_m$ 失配的影响**：
$$
A_{CM} \approx \frac{g_{m1}-g_{m2}}{g_{m1}+g_{m2}} \cdot \frac{r_{O3}}{R_{SS}} \cdot \frac{g_{m4}}{g_{m3}}
$$

#### 电源抑制 (Supply Rejection)

五管 OTA 的 $V_{DD} \to V_{out}$ 增益约 **0 dB（=1）**——供电变化直接传递到输出。全差分拓扑虽输出共模也会变化，但差分输出不受影响（需 CMFB，见 Ch09）。

#### 低压改进 (Fig. 5.36)

在 $M_3$ 栅极串联电阻 $R_1$、灌入恒定电流 $I_1$（$I_1 \ll I_{SS}/2$），将栅压降低 $R_1 I_1$，从而允许更高的输入共模电平。

---

### 5.4 偏置技术

#### CS 级偏置 (5.4.1)

**基本偏置** (Fig. 5.43)：
- AC 耦合 + 大电阻 $R_B$ 提供直流电平，$C_B R_B$ 高通角频率低于最低信号频率
- 直流电平必须由二极管接法的器件产生（$M_B$），不能用恒压源
- $I_B$ 通常取 $I_{D1}$ 的 $1/10 \sim 1/5$ 以节省功耗
- 低频应用时电容面积大；高频时电容寄生效应限制性能

**大电阻的 MOS 实现** (Fig. 5.43d-e)：用深线性区 MOSFET 替代 $R_B$，$V_G$ 通过 $M_C$（大 $W/L$，使 $V_{GS,C} \approx V_{TH}$）产生。

**直接耦合的限制** (Fig. 5.44)：前级输出直流电压作后级偏置，PVT 变化会被放大（与信号无法区分）。仅当每级增益很低（2--3 倍）时可用。

**CS 级 + 电流源负载** (Fig. 5.45)：
- 两个高阻抗电流源"打架"——若 $I_{D1} \neq |I_{D2}|$，$V_{out}$ 会漂移到某管进入线性区才平衡
- 解决方案 (Fig. 5.45c)：$M_2$ 栅极通过 $R_G$ 接漏极（DC 二极管接法）+ $C_G$ 到地（AC 短路）——自偏置
- 输出偏置点移位 (Fig. 5.45d)：$R_G$ 灌入小电流 $I_G$，抬高 $V_{out}$ 到 $\approx V_{DD}/2$，优化对称摆幅

**互补 CS 级** (Fig. 5.47)：
- $V_{GS1}+|V_{GS2}|=V_{DD}$ 带来强 PVT 依赖性
- 自偏置：大电阻 $R_F$ 跨接漏极与栅极 (Fig. 5.47b)
- 精确电流定义 + 自偏置 (Fig. 5.47c)：$I_1$ 确定电流，$C_1$ 短路 $M_2$ 源极退化
- 输入必须 AC 耦合 (Fig. 5.47d)

#### CG 级偏置 (5.4.2)

- Fig. 5.48(a)：电阻 $R_S$ 提供源极到地的直流路径，但 $R_S$ 的信号衰减要求 $R_S \gg 1/(g_{m1}+g_{mb1})$，导致 $R_S$ 上的直流压降过大
- Fig. 5.48(b)：用电流源替代 $R_S$，高阻抗但不消耗大 headroom
- Fig. 5.48(c)：低压 cascode 电流镜解决 $V_{DS}$ 不匹配问题

#### 源跟随器偏置 (5.4.3)

- 电流源作尾电流（Fig. 5.49a），漏极串联电阻可减小沟道长度调制误差
- $I_{D1}$ 对栅压变化的灵敏度低于 CS 级，可直接与前级耦合
- 输入直流电平变化大时用 AC 耦合（Fig. 5.49b）

#### 差分对偏置 (5.4.4)

- 输入共模电平取最低允许值 $V_{GS1,2}+V_{DS3,min}$，以最大化输出摆幅
- 直接耦合级联（Fig. 5.51b）：第一级的最优输出共模对第二级来说**过低**——这是级间共模电平冲突的经典问题，需 AC 耦合或电平移位

## 关键公式与结论

| 项目 | 公式 | 注释 |
|------|------|------|
| 基本电流镜比 | $I_{out} = \frac{(W/L)_2}{(W/L)_1}I_{REF}$ | 忽略 $\lambda$ |
| 含 $\lambda$ 的复制比 | $I_{out} = \frac{(W/L)_2}{(W/L)_1} \cdot \frac{1+\lambda V_{DS2}}{1+\lambda V_{DS1}} I_{REF}$ | $V_{DS1}=V_{GS1}$ |
| Cascode 输出阻抗 | $R_{out} \approx g_{m3}r_{O3}r_{O2}$ | |
| 标准 cascode 最小输出电压 | $V_{min}=2V_{ov}+V_{TH}$ | |
| 低压 cascode 最小输出电压 | $V_{min}=2V_{ov}$ | 条件：$V_{GS0}-V_{TH0}<V_{TH1}$ |
| 五管 OTA $G_m$（近似） | $G_m = g_{m1,2}$ | 无源负载版的 2 倍 |
| 五管 OTA $R_{out}$（近似） | $R_{out} = r_{O2} \parallel r_{O4}$ | |
| 五管 OTA $A_v$（近似） | $A_v \approx g_{m1,2}(r_{O2}\parallel r_{O4})$ | |
| 五管 OTA $A_v$（精确） | $A_v = g_{m1}(r_{O1}\parallel r_{O4}) \cdot \frac{2g_{m4}r_{O4}+1}{2g_{m4}r_{O4}+2}$ | 校正因子 $<1$ |
| 五管 OTA CMRR | $\approx (g_m r_O)^2$ | $R_{SS}=r_O$ 时 |
| CS+电流源负载增益 | $A_v = -g_{m1}(r_{O1}\parallel r_{O2})$ | 自偏置版 |
| 互补 CS 增益 | $A_v = (g_{m1}+g_{m2})(r_{O1}\parallel r_{O2})$ | |

## 重要结构

### 结构对比：三种电流镜

| 拓扑 | $R_{out}$ | $V_{min}$ | 精度 | 复杂度 |
|------|-----------|-----------|------|--------|
| 基本电流镜 (Fig. 5.5b) | $r_{O2}$ | $V_{ov}$ | 低（$\lambda$ 误差大） | 最低 |
| 标准 Cascode 镜 (Fig. 5.12c) | $g_{m3}r_{O3}r_{O2}$ | $2V_{ov}+V_{TH}$ | 高 | 中 |
| 低压 Cascode 镜 (Fig. 5.18b) | $g_{m3}r_{O3}r_{O2}$ | $2V_{ov}$ | 高 | 较高 |

### 结构对比：差分对负载方案

| 负载类型 | $G_m$ | 输出 | CMRR | 电源抑制 | 共模反馈 |
|----------|-------|------|------|----------|----------|
| 无源电流镜 (Fig. 5.23a) | $g_{m1,2}/2$ | 单端 | 有限 | 差 | 不需要 |
| 有源电流镜 OTA (Fig. 5.26b) | $g_{m1,2}$ | 单端 | 有限（$\approx (g_m r_O)^2$） | 差（$\approx 1$） | 不需要 |
| 全差分 + 电流源负载 (Fig. 5.42b) | $g_{m1,2}$ | 差分 | 理论上无穷 | 好（差分抵消） | **需要 CMFB** |

## 设计启示

1. **电流偏置优于电压偏置**：用电流复制（$I_{REF} \to V_{GS} \to I_{out}$）消除 $\mu_n C_{ox}$ 和 $V_{TH}$ 的绝对误差，只保留尺寸比误差。

2. **用 unit transistor 而非缩放 $W$**：gate corner 效应使直接 $2W$ 并不精确等于 2 倍电流，应复制单位晶体管。

3. **等 $L$ 原则 + 串联实现等效长沟道**：避免 $L_D$ 误差；需要小电流时串联 unit 管而非增大 $L$。

4. **Cascode 的 headroom-精度权衡**：标准 cascode 精度高但 waste $V_{TH}$；低压 cascode 省 headroom 但需要额外偏置生成电路。

5. **五管 OTA 的开环直流失调问题**：极小失配就能让输出饱和——开环不要用于放大微弱差分信号，但适合大摆幅差分转单端。

6. **直接级联的共模冲突**：每级最优输出共模对下一级来说往往过低，需 AC 耦合或电平移位。

7. **两个电流源串联必然"打架"**：CS 级 + 电流源负载中，自偏置（$R_G+C_G$）是优雅的解决方案。

8. **纳米级设计的特殊挑战**：即使 cascode 结构，严重的沟道长度调制也会导致 $I_X$ 随 $V_X$ 明显变化（见 Nanometer Design Notes，Fig. 5.21 旁）。

## 章节关联

- **Ch03** Single-Stage Amplifiers：CS/CG/CD 级的基本分析，cascode 输出阻抗概念，本章的偏置技术直接服务于这些放大器。
- **Ch04** Differential Amplifiers：差分对基本分析（$G_m$、$R_{out}$、CMRR），本章的五管 OTA 是其有源负载延伸。
- **Ch07** Noise：电流源的噪声分析，见 Ch07。
- **Ch09** Operational Amplifiers：全差分运放需要 CMFB，因为电流源负载的输出共模不确定（本章 5.4.4 已埋下伏笔）。
- **Ch12** Bandgap References：$I_{REF}$ 的生成——PTAT 电流、带隙基准、启动电路等，本章只假设 $I_{REF}$ 已经存在。
- **Ch14** Matching：电流镜的随机失配分析。

## 检索关键词

`current mirror` `电流镜` `cascode current mirror` `级联电流镜` `low-voltage cascode` `低压cascode` `active current mirror` `有源电流镜` `five-transistor OTA` `五管OTA` `differential pair with active load` `有源负载差分对` `biasing techniques` `偏置技术` `self-biasing` `自偏置` `CMRR` `supply rejection` `电源抑制` `channel-length modulation` `沟道长度调制` `unit transistor` `单位晶体管` `current copying` `电流复制` `diode-connected` `二极管接法` `headroom` `电压裕度`

---

## Sources

- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]] — Chapter 5 (pp. 134--172)

## See Also

- [[Ch03 - Single-Stage Amplifiers]] — 单级放大器基础，cascode 输出阻抗概念
- [[Ch04 - Differential Amplifiers]] — 差分放大器基本分析
- [[Ch09 - Operational Amplifiers]] — 运放设计，CMFB 需要
