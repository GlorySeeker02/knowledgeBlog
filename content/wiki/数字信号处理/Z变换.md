# Z 变换

> 第 3 章 — Z 变换将差分方程转化为代数方程，是离散系统频域分析的核心工具。

## 定义

**双边 Z 变换**：$X(z) = \sum_{n=-\infty}^{\infty} x[n] z^{-n}$，其中 $z = re^{j\omega}$

**单边 Z 变换**：$X(z) = \sum_{n=0}^{\infty} x[n] z^{-n}$（适用于因果信号）

## 收敛域 (ROC)

ROC 是 $z$ 平面上的环形区域 $R_1 < |z| < R_2$：
- ROC 内不含任何极点
- 右边序列的 ROC 在最大极点半径之外
- 左边序列的 ROC 在最小极点半径之内
- 双边序列的 ROC 为环形区域

## 常用 Z 变换对

| 序列 $x[n]$ | $X(z)$ | ROC |
|-------------|--------|-----|
| $\delta[n]$ | $1$ | 全部 $z$ |
| $u[n]$ | $\frac{1}{1 - z^{-1}}$ | $\|z\| > 1$ |
| $a^n u[n]$ | $\frac{1}{1 - a z^{-1}}$ | $\|z\| > \|a\|$ |
| $n a^n u[n]$ | $\frac{a z^{-1}}{(1 - a z^{-1})^2}$ | $\|z\| > \|a\|$ |
| $\cos(\omega_0 n) u[n]$ | $\frac{1 - \cos(\omega_0) z^{-1}}{1 - 2\cos(\omega_0) z^{-1} + z^{-2}}$ | $\|z\| > 1$ |

## Z 变换的性质

| 性质 | 时域 | Z 域 |
|------|------|------|
| 线性 | $ax[n] + by[n]$ | $aX(z) + bY(z)$ |
| 时移 | $x[n-k]$ | $z^{-k}X(z)$ |
| 尺度变换 | $a^n x[n]$ | $X(z/a)$ |
| 卷积 | $x[n] * h[n]$ | $X(z)H(z)$ |
| 初值定理 | $x[0] = \lim_{z\to\infty} X(z)$ | — |

## 逆 Z 变换

1. **部分分式展开法**：将 $X(z)$ 展开为部分分式，利用常用变换对
2. **幂级数展开法**：展开为 $z^{-1}$ 的幂级数，系数即为 $x[n]$
3. **围线积分法**：$x[n] = \frac{1}{2\pi j} \oint_C X(z) z^{n-1} dz$

> `scipy.signal.residuez` 可实现部分分式展开。

## 系统函数

$$
H(z) = \frac{Y(z)}{X(z)} = \frac{\sum_{k=0}^{M} b_k z^{-k}}{\sum_{k=0}^{N} a_k z^{-k}} = \frac{b_0}{a_0} \frac{\prod (1 - c_k z^{-1})}{\prod (1 - d_k z^{-1})}
$$

- 分子根 → **零点**，分母根 → **极点**
- **因果 LTI 系统稳定 $\Leftrightarrow$ 所有极点位于单位圆内**

## Python 实践要点

1. `scipy.signal` 绘制零极点图，判断稳定性
2. 从零极点手动推导频率响应 $H(e^{j\omega})$
3. `scipy.signal.residuez` 部分分式展开求逆 Z 变换
4. `scipy.signal.freqz` / `group_delay` 分析系统特性
5. 多项式乘法验证卷积定理（时域卷积 ↔ Z 域相乘）

> 完整 Python 代码参见 `raw/dsp_learning/ch03_z_transform.md`

## See Also

- [[离散时间信号与系统]] — 差分方程的时域求解
- [[离散傅里叶变换 DFT]] — DTFT 的频域采样
- [[DSP 系统学习课程]] — 课程总览

---

*Updated: 2026-07-21*
