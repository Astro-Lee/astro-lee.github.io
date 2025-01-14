```python
import matplotlib.pyplot as plt

# 将宽度从 pt 转换为英寸 (1 pt = 1/72 inch)
width_pt = 242.26653 # one column 242.26653pt
#width_pt = 513.11743 # two column 513.11743pt

width_in = width_pt / 72.27  # Matplotlib 使用 TeX 的 pt 标准 (72.27 pt = 1 inch)

# 黄金分割比
golden_ratio = (1 + 5 ** 0.5) / 2

# 计算高度
height_in = width_in / golden_ratio

# 设置图片大小
fig, ax = plt.subplots(figsize=(width_in, height_in))

# 示例绘图
ax.plot([1, 2, 3], [1, 4, 9])
ax.set_title("Example Plot")
ax.set_xlabel("X-axis")
ax.set_ylabel("Y-axis")

# 显示图像
plt.show()
```