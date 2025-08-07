bilby + PyMultiNest + MultiNest

```python
def multi_sbpl(x, c, breakpoints, alphas, deltas):
    """
    多段平滑破幂律函数，幂律指数为负值形式 (-alpha_i)
    参数:
        x: 自变量 (array-like)
        c: 归一化幂指数
        breakpoints: 拐点列表 [xb1, xb2, ..., xb(n-1)]
        alphas: 正的幂律指数绝对值列表 [alpha1, alpha2, ..., alphan]
        deltas: 平滑参数列表 [delta1, delta2, ..., delta(n-1)]
    返回:
        f(x): SBPL 函数值
    """
    x = np.array(x)
    # 初始幂律指数为 -alpha1
    result = 10**c * x**(-alphas[0])
    # 计算每个拐点的平滑过渡
    for i in range(len(breakpoints)):
        term = 1 + (x / breakpoints[i])**(1 / deltas[i])
        # 指数差为 -alpha_(i+1) - (-alpha_i) = alpha_i - alpha_(i+1)
        result *= term**(-deltas[i] * (alphas[i + 1] - alphas[i]))
    return result
```

```python
import pandas as pd

bilby_data = pd.read_csv('https://gitee.com/Astro-Lee/storage/raw/master/bilby_data.csv')
x_time = bilby_data['x_time']
x_flux = bilby_data['x_flux']
x_flux_err = bilby_data['x_flux_err']
```

```python
import os
# 设置 DYLD_LIBRARY_PATH（适用于 macOS）
multinest_lib_path = os.path.expanduser("~/Downloads/MultiNest/lib")
os.environ["DYLD_LIBRARY_PATH"] = multinest_lib_path + ":" + os.environ.get("DYLD_LIBRARY_PATH", "")
# （如果需要的话）显示设置是否成功
print("DYLD_LIBRARY_PATH =", os.environ["DYLD_LIBRARY_PATH"])
# 或者cp -v ~/Downloads/MultiNest/lib/lib* /anaconda3/lib/
```

```python
#!/usr/bin/env python
"""
An example of how to use bilby to perform parameter estimation for
non-gravitational wave data consisting of a Gaussian with a mean and variance
"""
import bilby
import numpy as np
from bilby.core.utils.random import rng, seed

# Sets seed of bilby's generator "rng" to "123" to ensure reproducibility
seed(1234)

# Here is minimum requirement for a Likelihood class to run with bilby. In this
# case, we setup a GaussianLikelihood, which needs to have a log_likelihood
# method. Note, in this case we will NOT make use of the `bilby`
# waveform_generator to make the signal.

class SimpleGaussianLikelihood(bilby.Likelihood):
    def __init__(self, x, y, yerr):
        """
        A very simple Gaussian likelihood

        Parameters
        ----------
        data: array_like
            The data to analyse
        """
        super().__init__(parameters=dict())
        self.x = x
        self.y = y
        self.yerr = yerr

    def log_likelihood(self):
        c = self.parameters["c"]  
        breakpoint1 = self.parameters["breakpoint1"]
        breakpoint2 = self.parameters["breakpoint2"]
        alpha1 = self.parameters["alpha1"]
        alpha2 = self.parameters["alpha2"]
        alpha3 = self.parameters["alpha3"]
        log_f = self.parameters["log_f"]     

        model = multi_sbpl(self.x, c, 
            breakpoints=[breakpoint1,breakpoint2], 
            alphas=[alpha1,alpha2,alpha3], 
            deltas=[0.1,0.1])

        res = self.y - model
        sigma2 = self.yerr**2 + model**2 * np.exp(2 * log_f)
        
        loglike = -0.5 * np.sum(res**2 / sigma2 + np.log(2 * np.pi * sigma2))

        return loglike

likelihood = SimpleGaussianLikelihood(x_time, x_flux, x_flux_err)

priors = dict(
    c = bilby.core.prior.Uniform(-4, -3, "k",r"$c$"),
    breakpoint1 = bilby.core.prior.Uniform(200, 500, "breakpoint1",r"$breakpoint1$"),
    breakpoint2 = bilby.core.prior.Uniform(500,10000, "breakpoint2",r"$breakpoint2$"),
    alpha1 = bilby.core.prior.Uniform(-1, 1, "alpha1",r"$alpha1$"),
    alpha2 = bilby.core.prior.Uniform(1, 2, "alpha2", r"$alpha2$"),
    alpha3 = bilby.core.prior.Uniform(1, 2, "alpha3", r"$alpha3$"),
    log_f = bilby.core.prior.Uniform(-10., 1., "log_f", r"$\log f_\mathrm{sys}$"),
)
```

```python
# A few simple setup steps
label = "10keV-2broken"
outdir = "outdir"

# And run sampler
result = bilby.run_sampler(
    likelihood=likelihood,
    priors=priors,
    outdir=outdir,
    label=label,
    sampler="pymultinest",
    importance_nested_sampling=True,
    verbose=False,
    nlive=2000,
    resume=True,
    clean=False,
)
```

```python
result.plot_corner(save=True, filename='corner.pdf',
show_titles=True, quantiles=[0.16, 0.5, 0.84], 
labels=[prior_dist.latex_label for key, prior_dist in priors.items()]
)
```

<img width="1578" height="1600" alt="Image" src="https://github.com/user-attachments/assets/ed77ce4b-7ed3-4d2c-9b49-35d36ee72ccb" />

```python
print(label)
print(f'log Z = {result.log_evidence} ± {result.log_evidence_err}\n')

quantile_tab = result.posterior.quantile([0.16, 0.5, 0.84])
# 遍历所有列，打印中位数和上下误差
for col in quantile_tab.columns[:-2]:
    q16 = quantile_tab.loc[0.16, col]
    q50 = quantile_tab.loc[0.50, col]
    q84 = quantile_tab.loc[0.84, col]

    err_low = q50 - q16
    err_high = q84 - q50

    print(f"{col} = {q50:.6f} (+{err_high:.6f}/-{err_low:.6f})")
```

输出结果
```bash
# 10keV-2broken
# log Z = 12270.607387903234 ± 0.09746941609724716

# c = -3.661216 (+0.147197/-0.145641)
# breakpoint1 = 272.599363 (+12.038642/-11.232960)
# breakpoint2 = 1325.521828 (+424.605908/-306.887697)
# alpha1 = -0.033724 (+0.066284/-0.066135)
# alpha2 = 1.273952 (+0.029515/-0.033641)
# alpha3 = 1.550395 (+0.028099/-0.022251)
# log_f = -1.433169 (+0.029608/-0.029620)

# 10keV-1broken
# log Z = 12250.889339996882 ± 0.08563383228929447

# c = -3.359253 (+0.130664/-0.124555)
# breakpoint1 = 333.251729 (+11.848272/-10.852528)
# alpha1 = 0.107049 (+0.057266/-0.054615)
# alpha2 = 1.471793 (+0.011757/-0.011525)
# log_f = -1.400355 (+0.028854/-0.029786)
```
计算对数贝叶斯因子（ln Bayes Factor）
$\ln B = \log Z_{\text{2break}} - \log Z_{\text{1break}} \approx 20$

根据 Jeffreys scale 判断
| ln(Bayes Factor)      | 强度解释（Jeffreys scale)          |
| :------------------: | :-----------------------------: |
| 0–1                            | 不可区分 (Inconclusive)              |
| 1–2.5                         | 弱证据 (Weak evidence)             |
| 2.5–5                        | 中等证据 (Moderate evidence)   |
| >5                             | 强有力支持 (Strong evidence)     |

**Bayes 因子 $\ln B \approx 20$ 表明强有力的统计证据支持使用 2 个断点的模型。**

```python
import matplotlib.pyplot as plt
fig, ax = plt.subplot_mosaic([['lc']])

ax['lc'].errorbar(x_time, x_flux, 
    #xerr = x_time_err, 
    yerr = x_flux_err, 
    fmt='+', color='k', ms=2, capsize=0, label='flux density @ 10 keV',alpha=0.2)

x = np.logspace(1.5, 6, 1000)
c = -3.661216
deltas = [0.1,0.1]                        # 2个平滑参数
breakpoints = [272.599363, 1325.521828]   # 2个拐点
alphas = [-0.033724, 1.273952, 1.550395]   # 3个幂律指数
y = multi_sbpl(x, c, breakpoints, alphas, deltas)
ax['lc'].plot(x,y)

for bp in breakpoints:
    ax['lc'].axvline(bp, color='r', linestyle='--', alpha=0.5)

ax['lc'].set_xscale('log')
ax['lc'].set_yscale('log')
ax['lc'].set_xlabel('Time since trigger [s]')
ax['lc'].set_ylabel('Flux density [Jy]')
ax['lc'].legend(frameon=True,framealpha=0.5)
```

<img width="578" height="438" alt="Image" src="https://github.com/user-attachments/assets/434879b7-7d8d-496f-a2c6-9421995a18a7" />