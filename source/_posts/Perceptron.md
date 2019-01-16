---
title: 感知机
date: 2019-01-16 10:46:07
tags:
- Statistics
- Mathematics
categories:
- Machine Learning
mathjax: true
---

纸上得来终觉浅，绝知此事要躬行，只有亲自实现一个简单的感知机，遇到各种突发奇想的疑问然后尝试解决，才能真正掌握原理。
<!-- more -->

# 数学基础
输入空间为 $\chi \in R^n$，输出空间 $\Upsilon \in \{+1,-1\}$，输入空间到输出空间映射的函数为
$$f(x) = sign(w \cdot x + b)$$

其中符号函数
$$ sign(x) = 
\begin{cases}
+1, & x>=0 \\
-1, & x<0 \\
\end{cases} $$

只要数据集线性可分，就可以把所有实例点正确划分到超平面两侧，即对于 $y = +1$ 的点，$w \cdot x > 0$，对于 $y = -1$ 的点，$w \cdot x < 0$

# 学习策略
学习的目的就是最优化问题，就是最小化误分点个数，但是误分点个数显然是离散的，无法求导，所以感知机模型采取所有 **误分点** 到超平面的距离：
$$\frac{1}{||w||} |w \cdot x_i + b|$$

注意这个公式是初中就学过的点到直线的距离公式，把点坐标代入直线方程求绝对值然后放到分子，再把直线方程系数做平方和开根号放到分母。这个公式就是那个公式的通用版，适用于高维超平面而已。

那么如何才能找到误分点呢，仔细看下本段落标题上面那句话，知道了正确划分，自然就知道了错误划分的表现。同时注意符号，求解优化问题，我们都要改成求最小值问题。设集合 $M$ 是误分点集合，所有误分点到超平面总距离为

$$-\sum_{x_i \in M}\frac{1}{||w||} y_i (w \cdot x_i + b)$$

由此我们可以定义损失函数
$$L(w, b) = -\sum_{x_i \in M} y_i (w \cdot x_i + b)$$

求解最小化问题
$$\min_{w,b} L(w, b) = -\sum_{x_i \in M} y_i (w \cdot x_i + b)$$

按照梯度的方向下降就能减少误分点了
$$\nabla_w L(w, b) = -\sum_{x \in M} y_ix_i$$
$$\nabla_b L(w, b) = -\sum_{x \in M} y_i$$

每次对参数进行更新
$$w \leftarrow w + \eta y_i x_i$$
$$b \leftarrow b + \eta y_i$$

# 代码实现
```python
class Model:
    def __init__(self, n, l=0.001):
    """ n 为数据维度，l 为梯度步长 """
        self.w = np.ones(n, dtype=np.float32)
        self.b = 0
        self.l = l
        
    def sign(self, X, y):
        return (self.w @ X + self.b) * y
        
    def fit(self, X_train, y_train):
        while True:
            wrong_count = 0
            for i in range(len(X_train)):
                X = X_train[i]
                y = y_train[i]
                if self.sign(X, y) <= 0:
                    self.w += self.l * np.dot(X, y)
                    self.b += self.l * y
                    wrong_count += 1
            if wrong_count == 0:
                return (self.w, self.b)
```

用 `sklearn` 包中 `iris` 数据集作为测试数据集，一系列处理后画图
```python
perceptron = Model(2, 0.01)
perceptron.fit(X, y)
x_points = np.linspace(4, 7, 10)
y_points = -1*(perceptron.w[0] * x_points + perceptron.b)/perceptron.w[1]
```
![perceptron](https://github.com/fudonglai/merge_reponsitories/blob/master/perceptron.png?raw=true)

# 思考
在处理数据，将输出空间的数据改成 -1 和 +1 的时候，我在纠结到底应该是在超平面以上是 +1 还是以下是 +1，事实证明都可以，因为参数会自动调整，只要跟着梯度的方向走就行了。