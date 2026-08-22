---
title: "Gray Chapter 12: 全差分运算放大器"
source: "Analysis and Design of Analog Integrated Circuits, 5th Edition, Chapter 12"
tags:
  - analog-design
  - analog-IC
  - fully-differential
  - CMFB
  - op-amp
---

## 本章定位

本章是 Gray 教材中全差分运算放大器的专题章节。第 6 章讨论了单端输出运放，本章则聚焦于**差分输入、差分输出**的运放架构。全差分运放是现代模拟集成电路（尤其是低电源电压、混合信号 SoC）的主流选择，本章系统性地覆盖了其核心特性、小信号建模方法、共模反馈（CMFB）原理与电路实现、典型运放拓扑、失配效应及 CMFB 环路带宽设计。

---

## 核心概念

### 全差分运放 vs 单端运放的四大优势

| 优势 | 原理 |
|---|---|
| **输出摆幅加倍** | 单端峰值摆幅 $V_{\max}-V_{\min}$；差分两端反相摆动，峰值差分摆幅为 $2(V_{\max}-V_{\min})$ |
| **SNR 提高 3 dB** | 信号功率变为 4 倍，噪声功率变为 2 倍（两路 $R_1$ 噪声不相关），$\text{SNR}_{\max}=V_{sig}^2/(8kTR_1\cdot BW_N)$ 提高 3 dB |
| **共模噪声抑制** | 衬底耦合、电源噪声在平衡电路中仅产生共模输出，不影响差模输出 |
| **偶次谐波抵消** | 平衡电路差模传输特性为奇函数 $V_{od}=f(V_{id})$ 且 $f(x)$ 为奇函数，偶次非线性仅出现在单端输出 $V_{o1}$、$V_{o2}$ 中，差模相减时抵消 |

### 代价

- 需要两个匹配的反馈网络
- 必须引入**共模反馈（CMFB）**电路来稳定共模输出电压

---

## 关键公式与结论

### 小信号建模

差模/共模半电路是分析全差分电路的基本工具，对称轴在差模下为 ac ground，在共模下为开路。

**DM 开环增益：**
$$v_{od}=a_{dm}v_{id}+a_{cm-dm}v_{ic}$$

**CM 开环增益（无 CMFB）：**
$$v_{oc}=a_{cm}v_{ic}+a_{dm-cm}v_{id}$$

**加入 CMC 输入后的 CM 输出：**
$$v_{oc}=a_{cm}v_{ic}+a_{dm-cm}v_{id}+a_{cmc}v_{cmc}$$

**CMFB 闭合后的等效 CM 增益：**
$$a'_{cm}=\frac{a_{cm}}{1+a_{cms}(-a_{cmc})}$$

> 若环路增益 $|a_{cms}(-a_{cmc})|\gg 1$，则 $|a'_{cm}|\ll|a_{cm}|$，CMFB 可将 CM 增益压制两个数量级以上。

**CMFB 闭环直流精度：**
$$A_{CMFB}=\frac{v_{oc}}{v_{CM}}=\frac{a_{cms}(-a_{cmc})}{1+a_{cms}(-a_{cmc})}$$

> 高环路增益使 $V_{oc}$ 精确跟踪 $V_{CM}$。

### 负载阻抗

差模负载 $Z_{Ld}$ 和共模负载 $Z_{Lc}$ 不同：
- 对称轴上的元件在 DM 下被 ac ground 分半 → $Z_{Ld}$ 为 $Z_{L1}\parallel (Z_{L2}/2)$
- 对称轴上的元件在 CM 下被开路隔离 → $Z_{Lc}$ 不受 $Z_{L2}$ 影响

---

## 重要电路结构

### 1. 全差分两级运放（Fig. 12.23）

```
VDD
 │
 ├── M10 ──┬── M7
 │         │
Vo2 ←     Vo1 ←
         C   C
Vi1 → M1 M2 ← Vi2
      M3 M4
      │   │
      M9  M6
      └───┴─── VSS
(CMC = gate of M5)
```

- 输入级为互补差分对 (M1-M4)，第二级为共源级 (M6-M7, M9-M10)
- CMC 输入为 M5 栅极
- 两个 Miller 补偿电容 C 同时补偿 DM 和 CM 环路
- DM 半电路增益：$a_{dm0}=g_{m2}(r_{o2}\parallel r_{o4})\cdot g_{m6}(r_{o6}\parallel r_{o7})$
- CMC 增益：$a_{cmc0}\approx -g_{m5h}\, r_{o2}\, g_{m2}\, (r_{o2}\parallel r_{o4})\, g_{m6}(r_{o6}\parallel r_{o7})$

> [!tip] 补偿策略
> 分别按 DM 和 CMFB 环路计算所需补偿电容，取较大值。若 CMFB 所需 C 过大导致 DM 过补偿，可拆分 $M_5$ 降低 $g_{m5h}$。

### 2. Telescopic Cascode（Fig. 12.30）

- 仅 NMOS 传导时变电流，最大化速度
- 两级 NMOS cascode + 一级 PMOS cascode → 极高输出阻抗
- 补偿由输出负载电容提供
- 输出摆幅受 cascode 堆叠限制

### 3. Folded-Cascode（Fig. 12.31 & 12.40）

- 三种 CMC 输入选择：$M_5$ 栅极、$M_3/M_4$ 栅极、$M_{11}/M_{12}$ 栅极
- $M_{11}/M_{12}$ 作 CMC 输入时环路节点最少，且 NMOS 共源级 $g_m$ 大
- 12.9 节给出完整设计实例：包括偏置电路、DM/CM 增益计算、噪声分析、频率响应、slew rate

### 4. 双输入级运放（Fig. 12.34）

- 两对差分对共享有源负载
- 可实现**全差分同相放大器**（极高输入阻抗）

---

## 共模反馈 (CMFB)

> [!important] 为什么需要 CMFB
> 在 Fig. 12.2 的简单全差分放大器中，$I_{D5}$ 与 $|I_{D3}|+|I_{D4}|$ 独立设定，$V_{OC}$ 对失配极度敏感。CMFB 通过负反馈调节 $V_{GS5}$ 强制 $V_{OC}=V_{CM}$。

### 四种 CMFB 实现方式

| 方案 | 原理 | 优点 | 缺点 |
|---|---|---|---|
| **电阻分压 + 放大器** (12.5.1) | $R_{cs}$ 检测 $V_{oc}$，差分对放大 $V_{oc}-V_{CM}$ | 简单直观，CM 检测输入近似恒定 | 电阻加载 DM 输出，需加 buffer；电流镜极点影响 CMFB 相位裕度 |
| **电流注入式** (Fig. 12.18) | 拆分 $M_{21}$ 为 $M_{21A}/M_{21B}$，电流直接注入输出节点 | 消除 $M_5$-$M_{23}$ 电流镜极点 | 增加输出节点阻性/容性负载 |
| **双差分对** (12.5.2) | $M_{21}$-$M_{24}$ 检测 $V_{o1},V_{o2}$ 与 $V_{CM}$ 之差 | 无电阻，全晶体管实现 | 大信号下 $I_{cms}$ 出现 $V_{od}^2$ 项导致共模偏移；输出摆幅受差分对线性范围限制 |
| **开关电容** (12.5.4) | $C_1$ 在 $\phi_1$ 采样 $V_{CM}-V_{CS\,BIAS}$，$\phi_2$ 接至 $V_{oc}$ 和 $V_{cmc}$ | 不限制输出摆幅，无电阻负载 | 需要非重叠时钟；MOS 开关电荷注入引入 offset $\Delta Q/C_1$；需要 replica biasing 产生 $V_{CS\,BIAS}$ |
| **Triode 区晶体管** (12.5.3) | $M_{31},M_{32}$ 在 triode 区作压控电阻，检测 $V_{oc}$ | 全晶体管，无需电阻 | $g_m$ 小 → 环路增益和带宽受限；$V_{o1},V_{o2}$ 不能低于 $-V_{SS}+V_{tn}$ |

### CMFB 环路稳定性

CMFB 环路的主导极点：
$$\omega_{p1c}=\frac{1}{(R_o(\text{down})\parallel r_{o3})\,C_{Lc}}$$

高频下：
$$|a_{cmc}(j\omega)|\approx \frac{g_{m5h}}{\omega C_{Lc}},\quad\omega\gg|p_{1c}|$$

- CMFB 环路通常比 DM 环路有更多高频极点（CM-sense 放大器极点、源极节点极点）
- 若不想增加 $C_{Lc}$（会过补偿 DM 环路），可拆分 tail 管降低 $g_{m5h}$（Fig. 12.15）

---

## 失配效应

> 实际电路中，器件失配导致 DM-CM 交叉增益非零。

**闭环交叉增益（平衡反馈网络 + 非平衡运放）：**

$$A_{cm-dm}\approx\frac{a_{cm-dm}}{1+T_{dm}}+\frac{a'_{cm}\cdot a_{cms}\cdot a_{cmc-dm}}{1+T_{dm}}$$

$$A_{dm-cm}\approx\frac{a_{dm-cm}}{1+T_{dm}}+\frac{a_{adm-cms}\cdot a_{cmc}}{1+T_{cmfb}}$$

关键结论：
- 增大 $|T_{dm}|$ 可同时压制 $A_{cm-dm}$ 和 $A_{dm-cm}$
- 增大 $|T_{cmfb}|$ 可压制 $A_{dm-cm}$
- 反馈网络电阻失配也会产生交叉增益（参见 Fig. 12.36 的耦合半电路分析）

---

## 设计启示

1. **CMC 输入选择**影响 CMFB 环路极点数量：选节点少、$g_m$ 大的路径
2. **输出共模电平** $V_{CM}$ 应选择上下摆幅限制的中点：$V_{CM}=(V_{o1,\min}+V_{o1,\max})/2$
3. **CMFB 环路带宽**理想上应与 DM 环路带宽相等，但在多极点约束下通常需要折中
4. **补偿电容**取 DM 和 CMFB 所需值中较大者；若牺牲 DM 带宽不可接受，拆分 tail 管是有效的替代方案
5. **开关电容 CMFB** 适合 SC 电路应用，需注意时钟馈通/电荷注入导致的共模偏移
6. **共模输入范围 (CMIR)** 和**共模输出电压摆幅**是两个独立约束，需分别验证
7. **电容中和 (neutralization)** 可以消除 Miller 效应对输入电容的放大，用 cutoff 区 MOS 实现 $C_n=C_{gd}$

---

## 章节关联

- **Ch06** -- 单端运放基础（差分对、两级运放、折叠 cascode），本章是其全差分扩展
- **Ch03 (Section 3.5)** -- 差分放大器基本概念、DM/CM 半电路、交叉增益定义
- **Ch08** -- 反馈理论，本章用 return ratio 分析 DM 和 CM 环路稳定性
- **Ch09** -- Miller 补偿和极点分裂，本章两级运放的核心补偿手段
- **Ch04 (Section 4.3.5.1)** -- 偏置稳定性问题，为 CMFB 的必要性提供背景
- [[Ch11 - Noise in Integrated Circuits]] -- 全差分运放噪声分析（热噪声 + 闪烁噪声）

## See Also

- [[../Razavi/Ch09 - Operational Amplifiers]]
- Ch06 single-ended op-amps
- Ch08 feedback
- Ch09 stability

## 检索关键词

`全差分运放` `共模反馈` `CMFB` `差模半电路` `共模半电路` `交叉增益` `telescopic cascode` `folded-cascode` `开关电容 CMFB` `Miller 补偿` `return ratio` `失配分析` `中和电容` `输出摆幅` `CMRR`

---
Sources: [[raw/模拟集成电路/Gray/Ch12 - Fully Differential Operational Amplifiers]]
