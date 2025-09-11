Kolmogorov-Smirnov（K-S）检验是一种非参数检验，用于比较样本与参考概率分布（单样本K-S检验）或比较两个样本（双样本K-S检验）。该检验测量样本的经验分布函数与参考分布的累积分布函数（或两个样本的经验分布函数）之间的最大距离。

在假设检验的背景下，将检验的显著性水平（p值）用正态分布的标准差（sigma）表示通常是有用的。以下是将K-S检验的p值与正态分布的sigma水平联系起来的方法：

1. **K-S检验的p值**：K-S检验产生的p值表示在零假设下观察到数据（或更极端情况）的概率。

2. **Sigma水平**：在正态分布中，sigma水平对应于特定的累积概率。例如：
   - 1 sigma 对应于大约68.27%的分布。
   - 2 sigma 对应于大约95.45%的分布。
   - 3 sigma 对应于大约99.73%的分布。

3. **转换**：要将p值转换为sigma水平，可以使用标准正态分布的累积分布函数（CDF）的反函数（也称为Z分数）。

具体步骤如下：

1. 计算K-S检验的p值。
2. 使用标准正态分布的反CDF（或百分位点函数）找到对应的sigma水平。

这里有一个使用Python的简单示例，说明这种转换：

```python
import scipy.stats as stats

# K-S检验得到的示例p值
p_value = 0.05

# 将p值转换为sigma水平
sigma_level = stats.norm.ppf(1 - p_value / 2)  # 双尾检验
sigma_level
```

---

在这个示例中，p值为0.05对应于双尾检验中的大约1.96个sigma。

在统计检验中，双尾检验用于检测数据是否显著偏离零假设的两侧。对于 Kolmogorov-Smirnov (K-S) 检验来说，一般是关心样本分布与理论分布之间是否存在显著差异，这种差异可能发生在分布的任何一端，因此通常采用双尾检验。

让我们具体来看为什么在将 p 值转换为 sigma 水平时使用双尾检验：

1. **双尾检验**：假设我们有一个 p 值为 0.08%，它表示在零假设为真时，样本与理论分布之间的差异大于或等于观测到的差异的概率为 0.08%。在双尾检验中，我们需要将这 8% 的概率分配到分布的两端（每端 0.04%），因为我们关心的是任意一端的偏差。因此，在计算 sigma 水平时，我们使用 `1 - p_value / 2`。

2. **单尾检验**：如果我们只关心一个方向上的偏差，例如样本是否显著大于理论分布，则可以使用单尾检验。在这种情况下，我们会直接使用 `1 - p_value`。

下面是计算的 Python 代码示例：

```python
import scipy.stats as stats

# K-S 检验得到的 p 值
p_value = 0.08/100

# 将 p 值转换为 sigma 水平（双尾检验）
sigma_level = stats.norm.ppf(1 - p_value / 2)
sigma_level
```

根据以上解释，对于 p 值为 0.08%，双尾检验对应的 sigma 水平大约为 3.35。换句话说，这个 p 值相当于正态分布中约 3.35 个标准差（sigma）的显著性水平。

---

- [68–95–99.7 rule](https://en.wikipedia.org/wiki/68–95–99.7_rule)
- [Confidence interval](https://en.wikipedia.org/wiki/Confidence_interval)
- [Confidence Intervals](https://sphweb.bumc.bu.edu/otlt/mph-modules/bs/bs704_confidence_intervals/bs704_confidence_intervals_print.html)
- [Understanding Confidence Intervals](https://www.scribbr.com/statistics/confidence-interval/)
- [Confidence Interval Calculator](https://www.omnicalculator.com/statistics/confidence-interval)
- [How to use norm.ppf()?](https://stackoverflow.com/questions/60699836/how-to-use-norm-ppf)

Example:

- [Line Chart with Confidence Interval in Python](https://www.pythoncharts.com/python/line-chart-with-confidence-interval/)
- [seaborn.lineplot](https://seaborn.pydata.org/generated/seaborn.lineplot.html)
- [Probability & Statistics](https://www.youtube.com/playlist?list=PLeB45KifGiuHesi4PALNZSYZFhViVGQJK)
- [Prob/Stat for CS](https://www.alextsun.com/prob_stat_cs.html)