# 噪声 RMS 与方差

> [!abstract] 一句话结论
> 对于零均值噪声，**RMS = 标准差 = $\sqrt{\text{方差}}$**。相关性不改变这个关系，但会影响**方差本身的数值**。

---

## 一、基本定义

| 量 | 数学定义 | 物理含义 | 与自相关的关系 |
|---|---|---|---|
| **均方值** (mean square) | $E[X^2]$ | 总平均功率（含 DC） | $R_X(0)$ |
| **方差** $\sigma^2$ (variance) | $E[(X-\mu)^2]$ | AC 功率 | $R_X(0) - \mu^2$ |
| **标准差** $\sigma$ (std deviation) | $\sqrt{\sigma^2}$ | AC 有效值 | $\sqrt{R_X(0) - \mu^2}$ |
| **RMS** (root mean square) | $\sqrt{E[X^2]}$ | 有效值（含 DC） | $\sqrt{R_X(0)}$ |

### 核心关系式

$$
\boxed{X_{rms}^2 = \sigma_X^2 + \mu_X^2}
$$

**RMS 平方 = 方差 + 均值平方** —— 这是基本恒等式，与相关性无关。

---

## 二、噪声的特例：零均值

电路中的物理噪声源（热噪声、散粒噪声、闪烁噪声）都是**零均值**的：

$$
\mu_X = 0 \;\Longrightarrow\; \boxed{X_{rms} = \sigma_X = \sqrt{R_X(0)}}
$$

**对于零均值噪声，RMS、标准差、自相关在零滞后处的平方根，三者合一。**

| 概念 | 值 |
|---|---|
| 白噪声 | $X_{rms} = \sigma_W$ |
| AR(1) 噪声 | $X_{rms} = \displaystyle\frac{\sigma_W}{\sqrt{1-a^2}}$ |
| 一般零均值噪声 | $X_{rms} = \sqrt{R_X(0)}$ |

---

## 三、相关性影响的是方差本身，不是关系

虽然 $X_{rms} = \sigma_X$ 始终成立，但**相关噪声的 $\sigma_X$ 比白噪声更大**。

### 物理解释

白噪声的能量均匀分布在所有频率：

$$
S_X(\omega) = \sigma_W^2, \qquad \sigma_X^2 = \frac{1}{2\pi}\int_{-\pi}^{\pi} \sigma_W^2 d\omega = \sigma_W^2
$$

相关噪声的低频能量更集中（以 AR(1) 为例）：

$$
S_X(\omega) = \frac{\sigma_W^2}{|1-ae^{-j\omega}|^2}, \qquad
\sigma_X^2 = \frac{1}{2\pi}\int_{-\pi}^{\pi} S_X(\omega) d\omega = \frac{\sigma_W^2}{1-a^2}
$$

相关噪声的方差被 **$1/(1-a^2)$ 因子放大**。

### 能量视角

| 噪声类型 | PSD 形状 | $\sigma_X^2$ | RMS |
|---|---|---|---|
| 白噪声 $W[n]$ | 平直 $\sigma_W^2$ | $\sigma_W^2$ | $\sigma_W$ |
| AR(1) $a=0.5$ | 低频隆起 | $\sigma_W^2/0.75$ | $1.15\sigma_W$ |
| AR(1) $a=0.9$ | 低频强烈隆起 | $\sigma_W^2/0.19$ | $2.29\sigma_W$ |
| AR(1) $a \to 1$ | 趋向 1/f | $\to \infty$ | $\to \infty$ |

---

## 四、经过 LTI 系统后：RMS 的计算路径

### 标准流程

无论输入是否相关，RMS 计算路径相同：

$$
S_X(f) \xrightarrow{\times|H(f)|^2} S_Y(f)
\xrightarrow{\int df} \sigma_Y^2 = R_Y(0)
\xrightarrow{\sqrt{\;}} Y_{rms}
$$

### 示例：一阶 FIR 差分器

$$
h[n] = h_0\delta[n] - h_1\delta[n-1], \quad |H(\omega)|^2 = h_0^2 + h_1^2 - 2h_0h_1\cos\omega
$$

| 输入       | $\sigma_X^2$                | $\sigma_Y^2$                                           | $Y_{rms}$                                            |
| -------- | --------------------------- | ------------------------------------------------------ | ---------------------------------------------------- |
| 白噪声      | $\sigma_W^2$                | $\sigma_W^2(h_0^2 + h_1^2)$                            | $\sigma_W\sqrt{h_0^2 + h_1^2}$                       |
| AR(1) 相关 | $\dfrac{\sigma_W^2}{1-a^2}$ | $\sigma_W^2\,\dfrac{h_0^2 + h_1^2 - 2h_0h_1 a}{1-a^2}$ | $\sigma_W\sqrt{\dfrac{h_0^2+h_1^2-2h_0h_1a}{1-a^2}}$ |

> 详细推导见 [[离散系统噪声分析示例]]

---

## 五、时域 vs 频域计算 RMS

两种方法等价（Parseval 定理）：

$$
\boxed{\sigma_Y^2 = R_Y(0) = \frac{1}{2\pi} \int_{-\pi}^{\pi} S_Y(\omega) d\omega}
$$

| 路径 | 操作 | 适用场景 |
|---|---|---|
| 时域 | $R_X \to R_Y = R_X * R_h \to \sigma_Y^2 = R_Y(0)$ | 解析推导，$R_h$ 简单时 |
| 频域 | $S_X \to S_Y = S_X \cdot \|H\|^2 \to \sigma_Y^2 = \int S_Y df$ | 任意 $S_X$，数值计算首选 |

---

## 六、模拟电路中的实用速查

### 常见噪声源 RMS

| 噪声源 | PSD | RMS（带宽 $B$ 内） |
|---|---|---|
| 热噪声（电阻） | $4kTR$ | $\sqrt{4kTRB}$ |
| 热噪声（MOS 沟道） | $4kT\gamma/g_m$ | $\sqrt{4kT\gamma B/g_m}$ |
| 闪烁噪声 | $K_f/(C_{ox}WL f)$ | $\sqrt{\displaystyle\int_{f_1}^{f_2} \frac{K_f}{C_{ox}WL f} df}$ |
| $kT/C$ 噪声 | — | $\sqrt{kT/C}$ |

### 频谱密度 vs RMS 的换算

| 概念 | 符号 | 单位 | 关系 |
|---|---|---|---|
| 功率谱密度 | $S(f)$ | $V^2/Hz$ 或 $A^2/Hz$ | — |
| 电压噪声密度 | $e_n(f)$ | $V/\sqrt{Hz}$ | $e_n^2 = S(f)$ |
| RMS 噪声 | $V_{rms}$ | $V$ | $V_{rms}^2 = \int S(f) df$ |

> **实用技巧**：数据手册给 $e_n = 5\,nV/\sqrt{Hz}$，带宽 $B = 1\,MHz$，则 RMS = $5\,nV \times \sqrt{1 \times 10^6} = 5\,\mu V$

---

## 总结

1. **RMS 和方差的关系是普适的**：$X_{rms}^2 = \sigma_X^2 + \mu^2$，与噪声是否相关无关
2. **零均值噪声简化**：$X_{rms} = \sigma_X = \sqrt{R_X(0)}$
3. **相关性影响的是方差的数值**：相关噪声有更多低频能量 → 更大方差 → 更大 RMS
4. **RMS 的工程计算链**：PSD $\to$ 积分 $\to$ 方差 $\to$ 开方 $\to$ RMS

## See Also

- [[白噪声与线性系统]] — 白噪声输入基础
- [[色噪声与线性系统]] — 色噪声通解
- [[离散系统噪声分析示例]] — 具体 FIR 计算
- [[随机过程的概率分布]] — 概率分布基础
- [[../数字信号处理/随机信号处理与功率谱估计]] — PSD 估计方法

---

*Updated: 2026-07-29*
