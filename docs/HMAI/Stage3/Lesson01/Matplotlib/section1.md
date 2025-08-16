# Matplotlib之HelloWorld

## 1 实现一个简单的Matplotlib画图 — 以折线图为例

### 1.1 matplotlib.pyplot模块

matplotlib.pytplot包含了一系列类似于matlab的画图函数。

```python
import matplotlib.pyplot as plt
```

### 1.2 图形绘制流程：

1.创建画布 -- plt.figure()

```
plt.figure(figsize=(), dpi=)
```

- figsize: 指定图的长宽
- dpi: 图像的清晰度
- 返回fig对象

2.绘制图像 -- plt.plot(x, y)

3.显示图像 -- plt.show()

### 1.3 折线图绘制与显示

**举例：展现上海一周的天气，比如从星期一到星期日的天气温度如下**

```python
import matplotlib.pyplot as plt

# 1.创建画布
plt.figure(figsize=(10, 10), dpi=100)

# 2.绘制折线图
plt.plot([1, 2, 3, 4, 5, 6 ,7], [17,17,18,15,11,11,13])

# 3.显示图像
plt.show()
```

## 2 认识Matplotlib图像结构(了解)

![img](https://gitlab.com/iknowledge/CourseImage/-/raw/main/HMAI/Matplotlib/matplotlib_structure.jpeg)
