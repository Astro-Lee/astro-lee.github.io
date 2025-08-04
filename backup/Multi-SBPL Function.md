$f(x) = A\cdot x^{-\alpha_1}\cdot\prod_{i=1}^{n-1}\left[1+\left(\frac{x}{x_{bi}}\right)^{1/\delta_i}\right]^{-\delta_i(\alpha_{i+1}-\alpha_i)}$

其中：
- $A$: 归一化常数。
- $x_{bi}$: 第 $i$ 个拐点，满足 $x_{b1} < x_{b2} < \dots < x_{b(n-1)}$。
- $\alpha_i$: 正数，表示第 $i$ 个区间的幂律指数的绝对值（实际指数为 $-\alpha_i$).
- $\delta_i$: 平滑参数，控制第 $i$ 个拐点的过渡宽度（$\delta_i$ 越小，过渡越平滑；$\delta_i \to \infty$，退化为传统破幂律的尖锐拐点，一般设为固定值）。
- $x > 0$ : 自变量，确保幂律定义有效。

```python
import numpy as np
import matplotlib.pyplot as plt

def multi_sbpl(x, A, breakpoints, alphas, deltas):
    """
    多段平滑破幂律函数，幂律指数为负值形式 (-alpha_i)
    参数:
        x: 自变量 (array-like)
        A: 归一化常数
        breakpoints: 拐点列表 [xb1, xb2, ..., xb(n-1)]
        alphas: 正的幂律指数绝对值列表 [alpha1, alpha2, ..., alphan]
        deltas: 平滑参数列表 [delta1, delta2, ..., delta(n-1)]
    返回:
        f(x): SBPL 函数值
    """
    x = np.array(x)
    # 初始幂律指数为 -alpha1
    result = A * x**(-alphas[0])
    # 计算每个拐点的平滑过渡
    for i in range(len(breakpoints)):
        term = 1 + (x / breakpoints[i])**(1 / deltas[i])
        # 指数差为 -alpha_(i+1) - (-alpha_i) = alpha_i - alpha_(i+1)
        result *= term**(-deltas[i] * (alphas[i + 1] - alphas[i]))
    return result

# 示例：三段 SBPL
x = np.logspace(-1, 5, 1000)
A = 1.0
deltas = [0.1, 0.1]       # 两个平滑参数
breakpoints = [200, 5000]  # 两个拐点
alphas = [2.5, 1., 3.5]  # 三个幂律指数
y = multi_sbpl(x, A, breakpoints, alphas, deltas)

plt.plot(x, y)
plt.xscale('log')
plt.yscale('log')
```

<img width="649" height="614" alt="Image" src="https://github.com/user-attachments/assets/3797235b-86c4-4556-868e-3af40e2ea4e2" />