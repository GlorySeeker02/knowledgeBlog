---
title: "Ch04 - Analog CMOS Subcircuits"
source: "Allen-CMOS analog circuit design 3e"
tags:
  - analog-design
  - CMOS
  - subcircuits
  - current-mirrors
  - references
---

## 本章定位

本章是 Allen 模拟集成电路设计的第一个"building block"章节，承接第2-3章的器件与工艺建模，为第6-7章的运算放大器设计奠定子电路基础。全书将模拟电路分为简单子电路（本章）+ 较复杂放大器（第5章），本章涵盖了运放层级图中最低一层的所有基本单元：开关、有源电阻、电流沉/源、电流镜、电压/电流基准。

## 核心概念

### 4.1 MOS 开关

MOS 开关是模拟电路（尤其是开关电容电路）中最基础的单元。关键模型参数：

- **导通电阻 `r_ON`**：由非饱和区沟道电阻主导，`r_ON ≈ 1/[K'(W/L)(V_GS - V_T)]`（当 `v_DS` 很小时）。增大 `W/L` 或提高 `V_GS` 可降低 `r_ON`。
- **关断电阻 `r_OFF`**：理想无限大，实际受漏-衬底、源-衬底 pn 结漏电流（~1 fA/μm^2 室温，每 8°C 翻倍）及亚阈值漏电限制。
- **电荷注入（Charge Injection）**：开关关断时沟道电荷 `Q_ch = -C_ox·W·L(V_H - V_in - V_T)` 分别注入源/漏端，近似对半分流，在负载电容上产生 `ΔV = -C_ox·W·L(V_H - V_in - V_T)/(2C_L)` 的误差。
- **时钟馈通（Clock Feedthrough）**：栅极电压跳变通过交叠电容 `C_GS0`、`C_GD0` 耦合到开关两端，无论开关导通与否均存在。
- **慢速/快速关断区分**：
  - 慢速：`β·V_HT^2 / (C_L·U) >> 1`，关断前沟道电荷流回源端，误差 `V_error ≈ (W·C_GD0/C_L)·(V_S + V_T - V_L)`
  - 快速：`β·V_HT^2 / (C_L·U) << 1`，更多沟道电荷落入负载，误差 `V_error ≈ [C_ch/2 + C_GD0]/C_L · (V_S + V_T - V_L)`
- **CMOS 开关**：nMOS 与 pMOS 并联，大幅扩展 ON 状态下的模拟信号动态范围。`r_ON` 在整个信号摆幅内更平坦（nMOS 在低电压主导，pMOS 在高电压主导，中间并联取最小值）。
- **栅极过驱动（Charge Pump）**：低电源电压下，通过电荷泵电路为开关栅极提供高于 `V_DD` 的驱动电压。

### 4.2 MOS 二极管 / 有源电阻

- **MOS 二极管**：栅漏短接 → `v_DS = v_GS` → 始终饱和区。`I_D = (K'/2)·(W/L)·(V_GS - V_T)^2`。小信号电阻 `r_out ≈ 1/g_m`（因为 `g_m >> g_mbs, g_ds`）。
- **浮空有源电阻**：MOS 管工作在非饱和区，`r_ON ≈ 1/[K'(W/L)(V_GS - V_T)]`（`v_DS` 很小时），非线性较强，但变化范围大。

### 4.3 电流沉与电流源

- **基本电流沉/源**：恒定 `V_GS` 偏置，`r_out = 1/(λ·I_D)`，最小工作电压 `V_MIN = V_GS - V_T = V_ON`。
- **共源共栅（Cascode）电流沉**：M2 共栅级将 M1 的 `r_ds1` 放大约 `g_m2·r_ds2` 倍，总 `r_out ≈ g_m2·r_ds2·r_ds1`（典型 9.25 MΩ vs 250 kΩ）。
- **`V_ON` 偏置原理**：`V_GS = V_ON + V_T`，饱和条件 `v_DS ≥ V_ON`。两个串联同类型 MOS 管且电流相等时：`(W/L)_1·V_ON1^2 = (W/L)_2·V_ON2^2`。若 `V_GS` 相等，则 `I_D1/I_D2 = (W/L)_1/(W/L)_2`。
- **高摆幅 Cascode（High-Swing Cascode）**：将偏置管 `W/L` 减为 1/4，使其 `V_GS = V_T + 2V_ON`，此时 `V_MIN = 2V_ON`（相比标准 cascode 的 `V_T + 2V_ON` 减少了一个 `V_T`）。
- **改进型 High-Swing Cascode**：增加 M5 使 M1 和 M3 的 `V_DS` 相等，消除沟道长度调制引起的镜像误差。
- **自偏置 High-Swing Cascode**：仅需一路参考电流，但需要电阻 R（小电流时 R 很大）且引入额外极点。
- **Blackman 公式**：`R_out = R_out(gm=0) · [1+RR(port shorted)] / [1+RR(port opened)]`。串联反馈（电压控制）使端口开路时返回比为零 → 增大电阻；并联反馈（电流控制）使端口短路时返回比为零 → 减小电阻。

### 4.4 电流镜

电流镜是电流沉/源的延伸，基于"相同 `V_GS` 产生相同（或比例）`I_D`"的原理。三种非理想效应：

1. **沟道长度调制**：`i_O/i_I = (1+λ·v_DS2)/(1+λ·v_DS1)`。`v_DS` 差 1V、`λ=0.02` 时误差~2%。λ 越小、`v_DS` 越匹配，误差越小。
2. **阈值电压失调**：`ΔV_T` 对小电流镜像的影响更严重（`v_GS` 小 → `ΔV_T` 占比大）。`i_O/i_I ≈ 1 - 2ΔV_T/(V_GS - V_T) + ΔK'/K'`。
3. **几何失配**：`W`、`L` 的绝对误差。使用单位晶体管并联（例如 1:4 用 4 个相同单元）可使误差乘以标称增益而非叠加。

重要电流镜结构：
- **基本电流镜**：`r_out = 1/(λ·I_D)`
- **Cascode 电流镜**：`r_out ≈ g_m4·r_ds4·r_ds2`，大幅提升输出阻抗，降低 `v_DS` 失配误差
- **Wilson 电流镜**：通过电流负反馈提升输出阻抗，`r_out ≈ g_m3·r_ds3·r_ds2`
- **Regulated Cascode**：额外放大器驱动 cascode 栅极，`r_out ∝ g_m^2·r_ds^3`，输出阻抗最高

`V_MIN` 与 `V_I(min)`：Cascode 输入 ≥ `2V_T + 2V_ON`，Wilson 输入 ≥ `2V_T + V_ON`。可通过增大 `W/L` 降低。

### 4.5 电流与电压基准

- **灵敏度**：`S^(V_REF)_(V_DD) = (∂V_REF/∂V_DD)·(V_DD/V_REF)`。灵敏度为 1 表示 10% 电源变化引起 10% 基准变化。
- **pn 结电压基准**：`V_REF ≈ V_T·ln(V_DD/(R·I_s))`。对数关系使灵敏度远小于 1（如 `I=1mA`、`I_s=10^-15 A` 时灵敏度约 0.0362），但温度系数约 -5000 ppm/°C。
- **MOS 增强型基准**：`V_REF = V_T + sqrt(2I/(K'(W/L)))`，灵敏度约 0.283（优于纯电阻分压但仍不够好）。
- **自举基准（Bootstrap / V_T-Referenced）**：通过电流镜强制两侧电流相等，反馈建立平衡点 `I_Q = V_T/R + 1/(βR^2)(1 + sqrt(1+2βR V_T))`。理论上对 `V_DD` 灵敏度为零（实际受 λ 影响）。需要**启动电路**防止电路停留在零电流平衡点。
- **`V_BE` 参考基准**：类似自举，用 BJT 的 `V_BE` 代替 `V_T`，`I_2 = V_BE1/R`。
- **简单自偏置基准**（Fig 4.5-7）：4 管 + 1 电阻，`I_2 = (V_GS1 - V_GS2)/R = (1/R)·sqrt(2I/(K'(W/L)_1))·(1 - 1/sqrt(m))`。可同时输出偏置电流和 `V_GS` 基准电压。
- **温度系数**：`V_T` 基准约 -2300 ppm/°C，`V_BE` 基准约 -3330 ppm/°C。8 位精度、100°C 温漂要求 TC < 39 ppm/°C，普通基准无法满足，需要 bandgap 方案。

### 4.6 温度无关基准（Bandgap Reference）

核心原理：**PTAT 电压 + K·CTAT 电压 = 零温度系数电压**。

- **PTAT 电压**：两个不同面积二极管的 `ΔV_D = (kT/q)·ln(A_2/A_1)`，斜率为 `k/q = 0.086 mV/°C`。
- **CTAT 电压**：`V_D(T) = V_G0·(1 - T/T_0) + V_D(T_0)·(T/T_0) + (γ - α)·(kT/q)·ln(T/T_0)`，其中 `γ` 与温度有关的指数参数。最后一项是 bandgap 曲率问题的根源。
- **伪-PTAT 电流**：将 `ΔV_D` 加在电阻上。电阻有温度系数 → 电流不是真正 PTAT，但若后续经过同类型电阻转回电压，则电阻温度系数互相抵消。
- **伪-CTAT 电流**：将 `V_BE`（或 `V_D`）加在电阻上，通过负反馈强制电流。
- **串联形式**：`V_REF = V_CTAT + (R_2/R_1)·V_PTAT`。零温度系数条件 `R_2/R_1 = -(∂V_CTAT/∂T)/(∂V_PTAT/∂T)`，基准值接近 `V_G0 + (γ-1)·V_t0 ≈ 1.262 V`。
- **并联形式**：`V_REF = R_3·(I_PTAT' + I_CTAT')`，通过 `R_3/R_2` 比值获得任意基准电压值（如 0.5 V）。
- **曲率校正（Curvature Correction）**：在 PTAT + CTAT 基础上额外引入一路与温度无关的电流 `I_Const`，驱动额外的 pn 结产生 `I_α=0'` 电流，消去 `ln(T/T_0)` 非线性项。校正后的温度系数可低至 < 1 ppm/°C（0-100°C）。
- **运放的影响**：运放失调电压 `V_OS` 直接叠加在基准电压上，需要低失调、低温漂运放。
- **温度无关电流**：将 bandgap 电压施加在电阻上 → 电阻温度系数影响电流温度特性。修正条件为 `R_2/R_1 = -(∂V_CTAT/∂T)/(∂V_PTAT/∂T) · [1 + (dR/dT)·I_REF/(∂V_CTAT/∂T)]`。

## 关键公式与结论

| 公式 | 含义 |
|------|------|
| `r_ON ≈ 1/[K'(W/L)(V_GS - V_T)]` | MOS 开关导通电阻（非饱和区） |
| `ΔV = -C_ox·W·L(V_H - V_in - V_T)/(2C_L)` | 沟道电荷注入误差 |
| `r_out(MOS diode) ≈ 1/g_m` | MOS 二极管小信号电阻 |
| `r_out(cascode) ≈ g_m2·r_ds2·r_ds1` | Cascode 电流沉输出阻抗 |
| `V_MIN(high-swing) = 2V_ON` | 高摆幅 cascode 最小输出电压 |
| `i_O/i_I = (W/L)_2/(W/L)_1 · (1+λ·v_DS2)/(1+λ·v_DS1)` | 电流镜增益（含沟长调制） |
| `R_out = R_out(gm=0)·[1+RR(sc)]/[1+RR(oc)]` | Blackman 阻抗公式 |
| `ΔV_D = (kT/q)·ln(A_2/A_1)` | PTAT 电压（两二极管面积比） |
| `V_REF = V_G0 + (γ-1)·V_t0 ≈ 1.262 V` | 串联 bandgap 基准电压 |
| `TCF = (1/X)·(∂X/∂T)` | 温度系数定义 |

## 重要子电路结构

1. **CMOS 传输门开关**（Fig 4.1-12）：nMOS + pMOS 并联，全摆幅低导通电阻
2. **Dummy 开关抵消**（Fig 4.1-11）：反向时钟驱动 dummy 管，部分抵消电荷注入
3. **电荷泵栅极过驱动**（Fig 4.1-14）：产生 2×`V_DD` 的栅极驱动电压
4. **Cascode 电流沉**（Fig 4.3-4）：`r_out` 提升 `g_m·r_ds` 倍
5. **高摆幅 Cascode**（Fig 4.3-6）：M4 的 `W/L` 取 1/4，`V_MIN = 2V_ON`
6. **改进型高摆幅 Cascode**（Fig 4.3-7）：加入 M5 强制 `V_DS1 = V_DS3`
7. **自偏置高摆幅 Cascode**（Fig 4.3-8）：单参考电流，省一路偏置
8. **Cascode 电流镜**（Fig 4.4-6）：M3-M4 cascode 提升输出阻抗并减少 `v_DS` 失配
9. **Wilson 电流镜**（Fig 4.4-8）：电流负反馈提升输出阻抗
10. **Regulated Cascode 电流镜**（Fig 4.4-10）：有源反馈，`r_out ∝ g_m^2·r_ds^3`
11. **V_T 自举基准**（Fig 4.5-5）：电源无关，需启动电路
12. **简单自偏置基准**（Fig 4.5-7）：4 MOS + 1 R，输出偏置电流和 `V_GS` 电压
13. **运放型串联 Bandgap**（Fig 4.6-7）：经典 Kujik bandgap 实现
14. **无运放串联 Bandgap**（Fig 4.6-8）：Cascode 电流镜强制两侧电流相等
15. **并联 Bandgap**（Fig 4.6-9）：PTAT + CTAT 电流求和，灵活输出任意电压
16. **曲率校正 Low-TC 基准**（Fig 4.6-14）：三路电流（`I_PTAT'` + `I_CTAT'` + `I_α=0'`），消除 `ln(T/T_0)` 非线性

## 设计启示

1. **开关设计**：最小化 `W×L` 以减少电荷注入，但需要足够的 `W/L` 以保证 `r_ON·C << T` 建立时间。使用 CMOS 开关 + dummy 管抵消是常规做法。
2. **输出阻抗提升**：Cascode 是最常用手段（约 `g_m·r_ds` 倍提升），Regulated cascode 进一步提升至 `g_m^2·r_ds^3` 量级，但代价是电压裕度（headroom）和额外功耗/极点。
3. **`V_MIN` 与 headroom 折中**：标准 cascode 的 `V_MIN = V_T + 2V_ON`，高摆幅版本降到 `2V_ON`。低电源电压设计必须使用高摆幅结构。
4. **电流镜精度**：尺寸越大、`V_DS` 越匹配、电流越大（`V_GS-V_T` 大），精度越高。layout 上应使用单位晶体管并联 + 共质心布局。
5. **基准设计**：简单的 `V_T`/`V_BE` 基准温度系数约数千 ppm/°C，日常偏置够用；高精度应用必须上 bandgap（10-50 ppm/°C）+ 曲率校正（< 1 ppm/°C）。
6. **启动电路**：所有自偏置基准都存在零电流简并态，必须加启动电路。
7. **运放影响**：Bandgap 中运放的 `V_OS` 和 PSRR 直接影响基准精度。曲率校正后基准的温漂可能小于噪声/失调，必须全盘考虑。

## 章节关联

- **前继章节**：第2章（CMOS 工艺）、第3章（CMOS 器件建模）——提供本章所有器件的 `K'`、`V_T`、`λ`、`γ` 等模型参数。
- **后续章节**：第5章（CMOS 放大器）——用本章子电路构建共源级、共栅级、源跟随器、差分对等；第6-7章（运放）——用本章电流沉/源做偏置，用电流镜做有源负载，用基准提供稳定偏置。
- **平行阅读**：Razavi 第5章（电流镜与偏置）、Gray 第4章（基本集成电路构建模块）覆盖类似内容，可交叉参考电路拓扑的差异和不同教材的讲解重点。

## 检索关键词

`MOS开关` `导通电阻` `电荷注入` `时钟馈通` `CMOS传输门` `电荷泵` `MOS二极管` `有源电阻` `电流沉` `电流源` `Cascode` `高摆幅Cascode` `Blackman公式` `输出阻抗` `V_MIN` `V_ON` `电流镜` `Wilson电流镜` `Regulated Cascode` `沟道长度调制` `阈值失调` `灵敏度` `自举基准` `启动电路` `PTAT` `CTAT` `Bandgap` `带隙基准` `曲率校正` `温度系数` `V_GO` `ZTC`

## Sources

- [[raw/模拟集成电路/Allen/Allen-CMOS analog circuit design 3e.md]] (Chapter 4)

## See Also

- Razavi, *Design of Analog CMOS Integrated Circuits*, Ch05 (Current Mirrors and Biasing Techniques)
- Gray, Hurst, Lewis, Meyer, *Analysis and Design of Analog Integrated Circuits*, Ch04 (Basic Integrated-Circuit Building Blocks)
