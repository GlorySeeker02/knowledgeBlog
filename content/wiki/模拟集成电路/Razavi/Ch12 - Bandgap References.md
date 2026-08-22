---
title: Ch12 - Bandgap References
source: "Razavi, Design of Analog CMOS Integrated Circuits, 2nd Ed., Ch12"
tags: [analog-design, CMOS, bandgap-reference, voltage-reference, PTAT]
updated: 2026-08-02
---

# Ch12 — Bandgap References

## 本章定位

本章系统讲解 CMOS 工艺中**参考源（reference generator）**的设计方法，核心是带隙基准（bandgap reference）。参照源是模拟电路的基础——差分对偏置电流、运放共模电平、ADC/DAC 满量程范围，全部依赖于精确的参考电压/电流。

本章从两个子问题展开：**与电源无关的偏置**（supply-independent biasing）和**温度系数的定义**（temperature variation）。在此基础上，逐步推导出经典的带隙基准公式，并讨论运放失调、启动问题、曲率补偿、噪声、低压实现等实际设计考量。

---

## 核心概念

### 1. 参考源的三种温度特性

参考源的目标温度特性有三种形式：

| 类型 | 英文 | 应用场景 |
|------|------|---------|
| 与绝对温度成正比 | **PTAT** (Proportional To Absolute Temperature) | 温度传感器、偏置电流 |
| 恒定跨导 | **Constant-$G_m$** | 让某些晶体管的 $g_m$ 不随温度变化 |
| 温度无关 | **Temperature-Independent** | 通用电压/电流基准 |

### 2. 与电源无关的偏置：自举（Bootstrapping）

基本思想：让 $I_{REF}$ 成为 $I_{out}$ 的复制，而非依赖外部"黄金"电流源。

>[!info] 关键思路
>如果 $I_{out}$ 期望与 $V_{DD}$ 无关，那么 $I_{REF}$ 可以"自举"到 $I_{out}$——即 $I_{REF}$ 由电路自身产生。

图 12.2 的基本结构的问题是：在忽略沟道长度调制的条件下，电路只有一个方程 $I_{out} = K I_{REF}$，**任何电流值都有可能**——电流幅值不确定。

**解决方案**：引入电阻 $R_S$ 作为第二个约束（Fig. 12.3(a)）：

$$V_{GS1} = V_{GS2} + I_{D2} R_S$$

忽略体效应后得到：

$$\sqrt{\frac{2 I_{out}}{\mu_n C_{ox}(W/L)_N}} \left(1 - \frac{1}{\sqrt{K}}\right) = I_{out} R_S$$

$$I_{out} = \frac{2}{\mu_n C_{ox}(W/L)_N} \cdot \frac{1}{R_S^2} \cdot \left(1 - \frac{1}{\sqrt{K}}\right)^2$$

电流与 $V_{DD}$ 无关，但仍依赖于工艺和温度。

### 3. 启动问题（Start-Up Problem）

自偏置电路存在**简并偏置点**（degenerate bias point）：所有晶体管电流为零时，环路也可以维持该状态——电路永远不会自行启动。

>[!warning] 关键陷阱
>计算时对 $\sqrt{I_{out}}$ 两边除以 $I_{out}$ 时，**隐含假设了 $I_{out} \neq 0$**。而零电流状态也是方程的有效解。

**解决方案**：添加启动电路。例如 Fig. 12.5(a) 中的二极管连接器件 $M_5$，在启动时提供 $V_{DD} \to M_3 \to M_1 \to GND$ 的电流通路，迫使电路脱离零电流状态。条件是：
- $V_{TH1} + V_{TH5} + |V_{TH3}| < V_{DD}$（启动时导通）
- $V_{GS1} + V_{TH5} + |V_{GS3}| > V_{DD}$（正常工作时关断）

---

## 关键公式与结论

### 4. 带隙基准原理：温度系数互补

带隙基准的核心思想是：将**正温度系数**和**负温度系数**的两个量，按适当权重相加，得到零温度系数。

$$V_{REF} = \alpha_1 V_1 + \alpha_2 V_2$$

满足：$\alpha_1 \frac{\partial V_1}{\partial T} + \alpha_2 \frac{\partial V_2}{\partial T} = 0$

### 4.1 负温度系数电压：$V_{BE}$

对于一个集电极电流恒定的 BJT：

$$V_{BE} = V_T \ln\left(\frac{I_C}{I_S}\right)$$

其中 $V_T = kT/q$，$I_S \propto \mu k T n_i^2$。利用 $\mu \propto \mu_0 T^m$（$m \approx -3/2$）和 $n_i^2 \propto T^3 \exp[-E_g/(kT)]$（$E_g \approx 1.12 \text{ eV}$），推导得：

$$\frac{\partial V_{BE}}{\partial T} = \frac{V_{BE} - (4+m)V_T - E_g/q}{T}$$

>[!important] 关键数值
>在 $T = 300\text{K}$、$V_{BE} \approx 750\text{ mV}$ 时：
>$$\frac{\partial V_{BE}}{\partial T} \approx -1.5 \text{ mV/K}$$
>
>旧工艺（电流密度低，$V_{BE} \approx 700\text{ mV}$）：约 $-1.9\text{ mV/K}$
>现代工艺（电流密度高，$V_{BE} \approx 800\text{ mV}$）：约 $-1.5\text{ mV/K}$

### 4.2 正温度系数电压：$\Delta V_{BE}$

两只 BJT 工作在不同电流密度下时，**$V_{BE}$ 之差**与绝对温度成正比：

$$\Delta V_{BE} = V_{BE1} - V_{BE2} = V_T \ln\left(\frac{n I_0}{I_S} \cdot \frac{I_S}{I_0}\right) = V_T \ln n$$

$$\frac{\partial \Delta V_{BE}}{\partial T} = \frac{k}{q} \ln n \approx (0.087 \text{ mV/K}) \times \ln n$$

>[!tip] PTAT 本质
>此温度系数与温度本身无关，也**与集电极电流的温度行为无关**——这是 $\Delta V_{BE}$ 作为 PTAT 来源的根本优势。

当 $Q_2$ 由 $m$ 个单元并联构成时：$\Delta V_{BE} = V_T \ln(nm)$

### 4.3 带隙基准电压

将正负 TC 相加得到零 TC 的条件：

$$V_{REF} = V_{BE} + \alpha_2 V_T \ln n$$

令 $\alpha_2 \ln n \approx 17.2$，使得 $(17.2)(0.087\text{ mV/K}) \approx 1.5\text{ mV/K}$，恰好抵消 $\partial V_{BE}/\partial T$。

经典实现（Fig. 12.9, Kuijk 1973）：

$$V_{out} = V_{BE2} + \frac{V_T \ln n}{R_3} (R_3 + R_2) = V_{BE2} + V_T \ln n \left(1 + \frac{R_2}{R_3}\right)$$

零 TC 条件：$(1 + R_2/R_3) \ln n \approx 17.2$

>[!example] 设计示例
>选 $n=31$，则 $\ln 31 \approx 3.43$，需 $1+R_2/R_3 = 17.2/3.43 \approx 5.01$，即 $R_2/R_3 \approx 4$。

### 4.4 为何称为"带隙"基准

将零 TC 条件代入，可得：

$$V_{REF} = \frac{E_g}{q} + (4+m)V_T$$

当 $T \to 0$ 时，$V_{REF} \to E_g/q$，即硅的带隙电压（~1.12 V）。这就是"bandgap reference"名称的由来——输出电压在零温漂条件下接近硅的带隙电压。这也解释了为什么传统带隙基准输出约为 **1.25 V**。

### 4.5 PTAT 电流对 $V_{BE}$ 温度系数的影响

在 Fig. 12.9 中，BJT 的集电极电流其实是 **PTAT** 的（$\propto V_T/R_3$），而非恒定。重新推导：

$$\frac{\partial V_{BE}}{\partial T} = \frac{V_{BE} - (3+m)V_T - E_g/q}{T}$$

与恒流情况相比（分母 $4+m$ 变为 $3+m$），TC 略小于 $-1.5\text{ mV/K}$。实际设计依赖精确仿真。

---

## 重要结构

### 5. 经典 Kuijk 带隙基准（Fig. 12.9）

```
    VDD
     │
  ┌──┴──┐
  │  A1 │──→ Vout
  └──┬──┘
     ├──── R1 ──── R2 ────┐
     │    X        Y      │
     │    │        │      │
     │    ├──R3───┤      │
     │    │        │      │
     │   Q1(A)   Q2(nA)  │
     │    │        │      │
     └────┴────────┴──────┘
                         GND
```

- 运放 $A_1$ 驱动 $R_1$ 和 $R_2$ 的顶端，迫使 $V_X \approx V_Y$
- $R_1 = R_2$（匹配对）
- $R_3$ 上的压降 = $V_{BE1} - V_{BE2} = V_T \ln n$
- $V_{out} = V_{BE2} + (1 + R_2/R_3)V_T \ln n$

### 5.1 运放失调的影响

运放输入失调电压 $V_{OS}$ 被**放大**到输出：

$$V_{out} = V_{BE2} + \left(1 + \frac{R_2}{R_3}\right)(V_T \ln n - V_{OS})$$

$V_{OS}$ 被放大了 $1 + R_2/R_3$ 倍（典型约 5 倍），这是带隙基准误差的**主要来源**。更严重的是，$V_{OS}$ 本身也随温度变化，进一步恶化温度系数。

**减小失调影响的三种方法**：
1. 运放使用大尺寸器件、优化版图（Ch14, Ch19）
2. 让两路电流通过 $m$ 倍比率增加 $\Delta V_{BE}$
3. 每路串联两个 pn 结，将 $\Delta V_{BE}$ 翻倍（Fig. 12.13）

但串联 pn 结方法使 $V_{out} \approx 2.5\text{V}$，不适用于低电源电压。

### 5.2 CMOS 工艺兼容性：衬底 pnp

在标准 n-well CMOS 工艺中，pnp BJT 的形成方式：

- **p+ 注入**（PFET 的源/漏区）— 发射极
- **n-well** — 基极
- **p 型衬底** — 集电极（**必须接最低电位，通常为地**）

这限制了集电极必须接地，也因此衍生出 Fig. 12.14/12.15 中将二极管串联转换为射极跟随器的结构。

### 5.3 反馈极性

Fig. 12.9 中运放同时向两个输入端反馈——必须确保**整体为负反馈**：

$$\beta_N = \frac{1/g_{mQ1} + R_3}{1/g_{mQ1} + R_3 + R_1} \quad \text{(到反相端)}$$

$$\beta_P = \frac{1/g_{mQ2}}{1/g_{mQ2} + R_2} \quad \text{(到同相端)}$$

要求 $\beta_P < \beta_N$，且最好 $\beta_N \approx 2\beta_P$，使瞬态响应在重容性负载下保持良好。

### 6. PTAT 电流生成（Sec. 12.4）

两种基本结构：

- **Fig. 12.18**：运放驱动 PMOS 电流镜，$I_{PTAT} = V_T \ln n / R_1$——适合低电压
- **Fig. 12.19**：自偏置环 + BJT 核，结合了 Fig. 12.2 的电流镜和 PTAT 核

将 PTAT 电流流过电阻再叠加 $V_{BE}$，即可生成温度无关电压（Fig. 12.20）：

$$V_{out} = V_{BE3} + \frac{R_2}{R_1} V_T \ln n$$

### 7. 恒定 $G_m$ 偏置（Sec. 12.5）

从 Fig. 12.3 的自偏置电路出发：

$$I_{out} = \frac{2}{\mu_n C_{ox}(W/L)_N} \cdot \frac{1}{R_S^2} \cdot \left(1 - \frac{1}{\sqrt{K}}\right)^2$$

$$g_{m1} = \frac{2}{R_S} \left(1 - \frac{1}{\sqrt{K}}\right)$$

$g_{m1}$ 与电源电压和 MOS 参数无关！但 $R_S$ 仍有温度和工艺变化。

**改进方案**：用开关电容等效电阻 $R_{eq} = 1/(C_S f_{CK})$ 替代 $R_S$（Fig. 12.21）。电容的绝对值和温度系数远优于电阻。

---

## 低压带隙基准

### 8. 为什么需要低压方案

传统带隙输出 $V_{REF} \approx 1.25\text{ V}$，在 $V_{DD} < 1.2\text{ V}$ 以下无法工作。根本原因：必须将 $\sim 17.2 V_T$ 加到一个 $V_{BE}$ 上才能实现零温漂。

### 8.1 Banba 电路（Fig. 12.32）

核心思路：**先合成零 TC 电流，再转换为任意低电压**——而不是直接合成电压。

- 让 $R_2 = R_3$，则 $I_{C1} = I_{C2}$
- $I_{C2} = V_T \ln n / R_1$（PTAT 电流）
- $I_{R2} = |V_{BE1}| / R_2$（负 TC 电流）

$$|I_{D4}| = \frac{V_T \ln n}{R_1} + \frac{|V_{BE1}|}{R_2}$$

选择 $\frac{R_2}{R_1} \ln n \approx 17.2$，$I_{D4}$ 即具有零 TC。将此电流复制到 $M_5$ 并流过 $R_4$：

$$V_{BG} = \frac{R_4}{R_2} \left(|V_{BE1}| + \frac{R_2}{R_1} V_T \ln n\right)$$

通过调节 $R_4$，可在保持零 TC 的同时间接获得任意低于 $1.25\text{ V}$ 的输出电压。

>[!info] 工作电压下限
>最小 $V_{DD} = V_{BE1} + |V_{DS3}| \approx 0.7\text{ V} + 0.05\text{ V} = 0.75\text{ V}$

### 8.2 Banba 电路中的失调影响

$$V_{BG} = \frac{R_4}{R_1 \parallel R_2} \left(\frac{R_2}{R_1} V_T \ln n + |V_{BE1}| - V_{OS}\right)$$

$V_{OS}$ 被放大了 $R_4/(R_1 \parallel R_2)$ 倍。更直观地：

$$V_{BG} \approx \frac{R_4}{R_2} \left(|V_{BE2}| + \frac{R_2}{R_1} V_T \ln n\right) - \frac{R_4}{R_1} V_{OS}$$

**最小化 $V_{OS}$ 的唯一方法：最大化 $n$。**

### 8.3 另一种低压方案（Fig. 12.35）

在 Fig. 12.20 的输出端并联 $R_3$ 到地：

$$V_{out} = \frac{R_3}{R_2 + R_3} \left(V_{BE3} + \frac{R_2}{R_1} V_T \ln n\right)$$

标准带隙电压被电阻分压器 $R_3/(R_2+R_3)$ 等比例缩小。

### 8.4 五管 OTA 实现（Fig. 12.34）

Banba 电路中的运放可用最简单的五管 OTA 实现。设计要点：
1. **大尺寸器件**以降低 flicker 噪声和失调
2. $V_{GS\_Ma} + V_{headroom\_ISS} \leq |V_{BE1}|$（输入共模约束）
3. 沟道足够长以提供合理环路增益（5~10 倍）

OTA 同样需要启动电路——$V_{DD} < 1\text{ V}$ 时，可用二极管连接 NMOS 连接 $P$ 和 $X$ 节点。

---

## 速度与噪声

### 9. 输出阻抗与瞬态响应

带隙基准的输出阻抗随频率急剧变化（Fig. 12.27）：

$$|Z_{out}(\omega)| = \begin{cases} \frac{R_{out}}{1+g_{mP}R_1A_0}, & \omega < \omega_0 \\ R_{out}/2, & \omega = (1+g_{mP}R_1A_0)\omega_0 \end{cases}$$

低频时很好，但**高频时输出阻抗显著升高**，外部电路扰动可通过参考线产生串扰。

**应对策略**：
- 运放采用高速设计（功耗代价大）
- 关键节点加旁路电容 $C_B$——但需注意：运放需为单级结构以维持稳定性；$C_B$ 必须远大于耦合电容，否则会延长恢复时间

### 10. 噪声

带隙基准的噪声直接耦合到所有依赖它的电路：

$$V_{n,out} \approx V_{n,op}$$

运放的输入噪声几乎 **直接出现在输出端**。即使加大的输出旁路电容，也无法抑制低频 $1/f$ 噪声。

当参考电流通过 $N$ 倍镜像放大后，噪声电流也被放大 $N$ 倍。这在低噪声电路中（如 ADC 参考电压、LNA 偏置）可能成为限制因素。

---

## 设计启示

1. **启动电路不可省略**——自偏置环路的简并点不会在公式中出现，但仿真和实测中一定会遇到。DC sweep + 瞬态仿真双重验证。

2. **运放失调是精度瓶颈**——$V_{OS}$ 被 PTAT 放大倍数（$1+R_2/R_3$ 或 $R_4/(R_1\parallel R_2)$）放大至输出。增大 $\Delta V_{BE}$（通过大 $n$、大 $m$、串联二极管）是减轻失调影响的核心策略。

3. **沟道长度调制是电源抑制的天敌**——核心偏置环中所有晶体管都应使用长沟道器件。Cascode 结构和本地电源稳压（Sec. 12.8）可进一步改善。

4. **曲率补偿在 CMOS 中较少使用**——因为大失调和工艺变异使零 TC 温度在不同样品间变化显著（Fig. 12.17），难以可靠修正。

5. **反馈极性检查**——运放同时向同相和反相端反馈，务必确保 $\beta_N > \beta_P$，且留有足够裕度。

6. **电阻温度系数不影响零 TC 条件**——在 $V_{REF} = V_{BE} + (1+R_2/R_3)V_T\ln n$ 中，零 TC 仅取决于电阻**比率**，而非绝对值。

7. **低电压下噪声权衡严重**——Banba 电路中 $R_1, R_2$ 通常较大（如 $50\text{ k}\Omega$），热噪声大；且偏置电流按比例放大会同时放大噪声。

---

## 章节关联

- **Ch05 (Current Mirrors and Biasing)** — 电流镜和 Cascode 偏置技术是本章自偏置环路的基础；本章的 Constant-$G_m$ 偏置是 Ch05 偏置思想的自然延伸
- **Ch14 (Mismatch and Offset)** — 运放输入失调是带隙基准精度的核心限制因素，Ch14 提供失调的物理来源和版图优化方法
- **Ch19 (Operational Amplifiers II)** — 低失调运放的设计方法
- **Ch13 (Switched-Capacitor Circuits)** — Sec. 12.5 中开关电容电阻替代方案的理解需要 SC 基础

---

## 检索关键词

`bandgap reference` `带隙基准` `PTAT` `CTAT` `$\Delta V_{BE}$` `supply-independent biasing` `start-up circuit` `简并偏置点` `Banba 电路` `sub-1V bandgap` `constant-Gm bias` `curvature correction` `op amp offset` `衬底 pnp` `voltage reference` `temperature coefficient` `$V_T \ln n$`
