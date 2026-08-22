---
title: Ch04 - Differential Amplifiers
source: Razavi-Design of Analog CMOS Integrated Circuits 2nd
tags: [analog-design, CMOS, differential-amplifiers]
chapter: 4
updated: 2026-08-02
---

# Ch04 - Differential Amplifiers

> 差分放大器是模拟电路史上最重要的发明之一，从真空管时代延续至今，已成为现代高性能模拟和混合信号电路的事实标准。

## 1 本章定位

本章在 Razavi 教材中处于单级放大器（Ch03）与电流镜（Ch05）之间，承担从单端到差分的过渡。它系统阐述差分对的工作原理（大信号 + 小信号）、共模响应（CMRR）、半电路概念、有源负载差分对和 Gilbert 单元，为后续章节的多级运放设计打下基础。

**前置依赖**：Ch03 单级放大器（CS/CG/CD 三种组态、小信号模型、gm/rO 概念）、MOSFET 饱和区平方律模型。

**后续桥梁**：
- Ch05 电流镜 → 差分对的偏置电流源实现
- Ch07 噪声 → 差分对噪声分析
- Ch09 运算放大器 → 差分对是运放输入级核心
- Ch14 线性化与失配分析 → 拓展差分对线性范围的方法

## 2 核心概念

### 2.1 单端信号 vs 差分信号

| 特性 | 单端 (Single-Ended) | 差分 (Differential) |
|------|---------------------|---------------------|
| 参考点 | 固定电位（通常为地） | 两个节点之间 |
| 信号关系 | 单一波形 | 大小相等、极性相反 |
| 共模电平 | 无 | $\text{CM} = \frac{V_1+V_2}{2}$ |
| 差分摆幅 | — | 2 倍单端峰值振幅 |

> [!important] 差分信号的核心优势
> 1. **抗环境噪声**：邻近时钟线耦合噪声同时加到两条差分线上，差值不变
> 2. **抗电源噪声**：$V_{DD}$ 的波动同时影响 $V_X$ 和 $V_Y$，$V_X-V_Y$ 不变
> 3. **更大的电压摆幅**：1 V 供电可实现 1.6 V$_{pp}$ 差分摆幅
> 4. **偏置更简单**、线性度更高（Ch14 详述）
> 5. **进攻者 + 受害者同时差分化**：双绞线（twisted pair）布局可完全消除耦合

### 2.2 基本差分对结构

基本差分对由两个完全相同的晶体管 $M_1$、$M_2$ 和一个尾电流源 $I_{SS}$ 组成。

> [!info] 尾电流源的关键作用
> 使 $I_{D1}+I_{D2}$ 与输入共模电平 $V_{in,CM}$ 无关，保证偏置电流恒定，从而稳定跨导和输出共模电平。

**输入共模范围 (Input CM Range)**：

$$
V_{GS1} + (V_{GS3} - V_{TH3}) \;\leq\; V_{in,CM} \;\leq\; V_{DD} - \frac{R_D I_{SS}}{2} + V_{TH}
$$

- **下限**：由尾电流管 $M_3$ 进入饱和区决定
- **上限**：由输入管 $M_1$ / $M_2$ 不进入线性区的条件决定

**最大输出摆幅（单端）**：$V_{DD} - (V_{GS1} - V_{TH1}) - (V_{GS3} - V_{TH3})$，即 $V_{DD}$ 减两个过驱电压。

> [!warning] 纳米级设计注意点
> 在纳米工艺中，沟长调制严重且供电电压有限，差分增益很难超过 5。此时峰值 **输入** 摆幅也会限制输出摆幅（因为负增益使输出朝正向摆时压到饱和边界）。

### 2.3 半电路概念 (Half-Circuit Concept)

差分对分析中最强大的工具。**核心引理**：

> 在对称差分对施加完全差分输入（$+\Delta V_{in}$ 和 $-\Delta V_{in}$）时，**节点 $P$（源极公共点）的电压保持不变**——它是交流地 / 虚地（virtual ground）。

证明思路：
- 对称电路无法"偏袒"某个输入
- Thevenin 等效：两侧 $V_T$ 等量反向、$R_T$ 相等 → $V_P$ 不变
- 更正式的证明：$g_m \Delta V_1 + g_m \Delta V_2 = 0 \;\Rightarrow\; \Delta V_1 = -\Delta V_2$，而 $V_P = V_{in1} - V_1$，故 $V_P$ 不变

> 将电路沿 $P$ 点对称轴切开，每半边就退化为一个 CS 放大器，直接写出增益 $-g_m R_D$。

**任意输入的处理**：将 $V_{in1}$、$V_{in2}$ 分解为差模分量 $(\frac{V_{in1}-V_{in2}}{2})$ 和共模分量 $(\frac{V_{in1}+V_{in2}}{2})$，分别用半电路法和 CM 分析处理，再叠加。

### 2.4 退化差分对 (Degenerated Differential Pair)

在源极加入串联电阻 $R_{S1}=R_{S2}=R_S$：

- **线性范围扩大**：关断一侧所需的 $\Delta V_{in}$ 增加约 $\pm R_S I_{SS}$
- **增益降低**：$A_v = -\dfrac{g_m R_D}{1+g_m R_S}$
- **增益对 $g_m$ 变化不敏感**：深度退化下 $A_v \approx -R_D/R_S$
- **电压裕度消耗**：每个电阻静态压降 $R_S I_{SS}/2$

**改进方案**：将尾电流源拆分为两半分别接到源极，$R_S$ 接在两源极之间——静态无电流流过 $R_S$，不消耗裕度。但两个独立尾电流源会各自贡献差分噪声和失调。

## 3 关键公式与结论

### 3.1 大信号传输特性

假设平方律模型、$\lambda=0$、两管饱和：

$$
I_{D1} - I_{D2} = \frac{1}{2}\mu_n C_{ox}\frac{W}{L}(V_{in1}-V_{in2})\sqrt{\frac{4I_{SS}}{\mu_n C_{ox} (W/L)} - (V_{in1}-V_{in2})^2}
$$

> $\Delta I_D$ 是 $\Delta V_{in}$ 的奇函数——尽管每管的 $I_D$ 是 $V_{GS}$ 的平方函数，但差值消除了偶次项。

**平衡点等效跨导**（$\Delta V_{in}=0$ 时取最大值）：

$$
G_m|_{\Delta V_{in}=0} = \sqrt{\mu_n C_{ox}\frac{W}{L} I_{SS}} = g_{m1} = g_{m2}
$$

其中 $g_{m1}=g_{m2}$ 是每管偏置在 $I_{SS}/2$ 时的跨导。

### 3.2 单侧关断条件

当 $\Delta V_{in}$ 增大到使一管完全关断（全部 $I_{SS}$ 流入另一管）：

$$
\Delta V_{in1} = \sqrt{\frac{2I_{SS}}{\mu_n C_{ox} (W/L)}} = \sqrt{2}\;(V_{GS}-V_{TH})_{eq}
$$

> [!tip] 设计启示
> $\Delta V_{in1}$ = 平衡过驱电压的 $\sqrt{2}$ 倍。扩大线性输入范围需增大过驱电压，要么降低 $W/L$（损失 $g_m$ 和增益），要么增大 $I_{SS}$（增加功耗）。

纳米工艺中，实测（$W/L = 5\mu\text{m}/40\text{nm}, I_{SS}=0.25\text{mA}$）仍近似遵循 $\sqrt{2}$ 关系。

### 3.3 小信号差分增益

$$
\boxed{A_{v,\text{diff}} = \frac{V_{out1}-V_{out2}}{V_{in1}-V_{in2}} = -g_m R_D}
$$

- 若输出取单端（$V_{out1}$ 或 $V_{out2}$ 对地）→ 增益减半
- 差分对的总 $G_m$（相对于总偏置 $I_{SS}$）是同样尺寸单管偏置在 $I_{SS}$ 下跨导的 $1/\sqrt{2}$ 倍

### 3.4 共模响应与 CMRR

**纯对称电路 + 有限尾阻抗 $R_{SS}$**（共模增益）：

$$
A_{v,CM} = -\frac{R_D}{\frac{1}{g_m}+2R_{SS}}
$$

**不对称电路 → 共模-差模转换**：

- 负载电阻失配 $\Delta R_D$：
  $$A_{CM-DM} = \frac{\Delta R_D}{R_D}\cdot\frac{g_m R_D}{1+2g_m R_{SS}}$$

- 跨导失配 $\Delta g_m$：
  $$A_{CM-DM} = -\frac{\Delta g_m \cdot R_D}{1+2g_m R_{SS}}$$

**共模抑制比**：

$$
\boxed{\text{CMRR} = \left|\frac{A_{v,DM}}{A_{CM-DM}}\right|}
$$

仅考虑 $g_m$ 失配时：

$$
\text{CMRR} \approx \frac{g_m}{\Delta g_m}(1+2g_m R_{SS}) \approx \frac{2g_m^2 R_{SS}}{\Delta g_m}\quad(\text{当 }2g_m R_{SS}\gg 1)
$$

> [!warning] 高频退化
> 即使 $R_{SS}$ 很大，尾节点寄生电容 $C_P$ 会在高频提供低阻抗通路，恶化 CMRR：
> $$A_{CM-DM} \propto \frac{\Delta g_m \cdot R_D}{\sqrt{1+(2g_m/\omega C_P)^2}}$$
> **频率越高，CMRR 越差。**
>
> 体效应失配（$g_{mb1}\neq g_{mb2}$）即使理想 $I_{SS}$ 也会导致 CM-DM 转换——因为 $V_P$ 随 $V_{in,CM}$ 变化，$V_{BS}$ 变化不等量地调制两管电流。

## 4 重要结构

### 4.1 基本电阻负载差分对

```
        VDD
        │
    ┌───┴───┐
    RD      RD
    │       │
Vout1     Vout2
    │       │
  M1│     M2│
Vin1─┘   Vin2─┘
        │
      P─┴─
        │
       ISS (M3 + Vb)
        │
       GND
```

- 增益：$A_v = -g_m R_D$
- 输出摆幅受限（$V_{DD}$ 减两个过驱）
- 最简单，适合教学和低增益场景

### 4.2 二极管连接负载差分对

PMOS 二极管连接作负载：

$$
A_v = -\frac{g_{mN}}{g_{mP}} = -\sqrt{\frac{\mu_n (W/L)_N}{\mu_p (W/L)_P}}
$$

- 增益与偏置电流无关
- 二极管连接消耗电压裕度 → 增益与输出摆幅相互制约
- 输出共模电平自动确定：$V_{DD} - |V_{GSP}|$

**增益提升技巧**：并联 PMOS 电流源 $M_5$、$M_6$ 抽走大部分偏置电流（如 80%），流过 $M_3$、$M_4$ 的电流降为 1/5 → $g_{mP}$ 降为 1/5 → 增益提高 5 倍（$\lambda=0$ 假设下）。

### 4.3 电流源负载差分对

PMOS 电流源（由 $V_b$ 偏置）作负载：

$$
A_v = -g_{mN}(r_{ON} \parallel r_{OP})
$$

- 增益受限于 $r_O$，纳米工艺中典型 5–10
- 输出共模电平不确定 → 需额外 CMFB 电路（Ch09）

### 4.4 共源共栅差分对 (Cascode)

NMOS 和 PMOS 两侧均采用 cascode 结构提升输出阻抗：

```
      VDD
      │
    M7,M8 ← Vb3 (PMOS cascode)
      │
    M5,M6 ← Vb2 (PMOS 电流源)
      │
    ──┼── Vout
      │
    M3,M4 ← Vb1 (NMOS cascode)
      │
    M1,M2 ← Vin
      │
     ISS
```

半电路增益：

$$
A_v \approx -g_{m1}\left[(g_{m3}r_{O3}r_{O1}) \parallel (g_{m5}r_{O5}r_{O7})\right]
$$

代价：消耗更多电压裕度。

### 4.5 Gilbert 单元

6 管结构，顶部两个差分对 ($M_{1-4}$) 处理信号，底部一个差分对 ($M_{5-6}$) 控制增益：

```
        VDD
        │
    ┌───┴───┐
    RD      RD
    │       │
M1─┤   M3─┤   M2─┤   M4─┤
    │       │
A───┘   B───┘
    │       │
   M5       M6  ← Vcont
    │       │
    └─┬───┬─┘
      │ ISS
     GND
```

**核心特性**：
1. **可变增益放大器 (VGA)**：$A_v$ 由 $V_{cont}$ 连续控制，可从负值到正值
2. **模拟乘法器**：$V_{out} = \alpha \cdot V_{in} \cdot V_{cont}$（一阶近似）
3. **输入和控制角色可互换**：既可从顶部输入信号、底部控制增益，也可底部输入信号（$V \to I$ 转换）→ 顶部路由电流

> [!warning] 电压裕度
> Gilbert 单元中顶部差分对与底部控制对"堆叠"，$V_{CM,cont} \leq V_{CM,in} - V_{GS1} + V_{TH5,6}$，控制共模至少比输入共模低一个过驱电压。

## 5 设计启示

1. **差分对 vs CS 单管**：相同总偏置电流下，差分对总跨导是单管 $1/\sqrt{2}$，但换来所有差分优势——几乎永远是更好的选择。

2. **增益-线性度-功耗三角**：
   - 增大 $W/L$ → 增益上升，线性范围缩小
   - 增大 $I_{SS}$ → 增益和线性范围均上升，功耗增加
   - 加入源极退化 → 线性度上升，增益和摆幅下降

3. **输入共模电平选择**：越低越好（最大化输出摆幅），但不能低于 $V_{GS1}+V_{OD3}$。

4. **CMRR 设计优先级**：
   - 首要措施：使用高输出阻抗的尾电流源（cascode 电流源）
   - 其次：减小尾节点杂散电容（布局优化）
   - 再次：匹配输入管和负载管（版图对称、大面积）

5. **全差分放大器**：输出共模电平必须由额外电路定义（共模反馈 CMFB）——这是 Ch09 的核心课题。

## 6 章节关联

- **前置**：[[Ch03 - Single-Stage Amplifiers]] 的单管小信号分析是所有差分级推导的基础
- **后置**：
  - [[Ch05 - Current Mirrors and Biasing Techniques]]：如何实现高输出阻抗的尾电流源
  - [[Ch07 - Noise]]：差分对噪声分析——差分对噪声功率 = 两管噪声之和（不相关噪声叠加），等效输入噪声为两管贡献之和
  - [[Ch09 - Operational Amplifiers]]：两级运放的输入级几乎总是差分对，共模反馈 (CMFB) 解决高增益差分放大器输出 CM 电平定义问题
  - [[Ch14 - Nonlinearity and Mismatch]]：拓展差分对线性输入范围的多种方法

## 7 检索关键词

`differential pair`, `差分对`, `common-mode rejection`, `CMRR`, `共模抑制比`, `half-circuit`, `半电路`, `virtual ground`, `虚地`, `tail current source`, `尾电流源`, `Gilbert cell`, `吉尔伯特单元`, `VGA`, `可变增益放大器`, `cascode differential`, `共源共栅差分对`, `degeneration`, `源极退化`, `common-mode to differential conversion`, `共模-差模转换`, `input common-mode range`, `输入共模范围`, `output swing`, `输出摆幅`, `ΔVin1`, `平衡过驱电压`

---

## Sources
- [[raw/模拟集成电路/Razavi/Razavi-Design of Analog CMOS Integrated Circuits 2nd.md]]

## See Also
- [[Ch03 - Single-Stage Amplifiers]]
- [[Ch05 - Current Mirrors and Biasing Techniques]]
- [[Ch07 - Noise]]
