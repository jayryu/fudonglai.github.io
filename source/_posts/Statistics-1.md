---
title: 最小二乘法和正则化
date: 2019-01-12 01:51:14
tags:
    - Statistics
    - Mathematics
categories:
    - Machine Learning
mathjax: true
---


经过半年的数学，英语，算法学习，阿东又要重回机器学习算法的摸爬滚打中了，现在从统计学习方法开始边学边总结。之前学习过一遍，但是由于当时的知识有限，学起来略有困难，现在不仅有了更强的能力，而且开源社区对这方面的资源也日益完善，首先感谢各位大佬对我学习的帮助，尤其感谢[黄海广博士](https://github.com/fengdu78/lihang-code)将本书翻译成代码，对我们来说确实是一条学习捷径。

<!-- more -->

系列文章文按照本人学习李航[《统计学习方法》]()的顺序，结合吴恩达老师的机器学习课程，记录一些重点和个人的思考，希望能帮助到大家。

# 最小二乘法

高斯于1823年在误差 $e_1, … ,e_n$ 独立同分布的假定下,证明了最小二乘方法的一个最优性质: 在所有无偏的线性估计类中,最小二乘方法是其中方差最小的！

最小二乘法就是设计出一个假设函数 $f(x)$，利用原始训练数据集 $T=(x_i, y_i), i \in [1, m]$，拟合出一组参数 $\omega(\omega_0, \omega_1...\omega_m)$，计算残差 $r_i=f(x_i)-y_i$ 使得损失函数 $Loss(\omega, f) = \sum_{i=1}^mr_i^2$ 最小。其实这个很好想明白吧？我写以上一堆数学符号只是为了重新熟悉一下 Mathjax 的语法，向 Markdown 工程师的方向更进一步，哈哈。

我们常用的假设函数是多项式函数 
$$H(x)=w_0+w_1x+w_2x^2+...w_nx^n$$

为啥来？因为多项式函数的次数足够大的时候，是可以无限逼近很多曲线的，不过次数并不是越多越好的，看具体实现。

机器学习本质就是拟合，以便对未知作出预测，我们的目的就是最小化损失函数 $Loss(x)$，即
$$\min_\omega \sum_{i=1}^m(H(x_i)-y_i)^2$$

这里用 Python 代码模仿一个简单的最小二乘法示例，先生成了一个含有噪声的 $sin(2\pi x)$图像，然后用不同次数的多项式拟合，代码如下。

```python
def func(x):
    return np.sin(x * np.pi * 2)

def fit_func(p, x):
    # 这里 p 需要是一个序列，作为每个多项式的系数
    # 相当于之前的权值向量 w
    f = np.polyld(p)
    return f(x)

def residuals(p, x, y):
    return fit_func(x, p) - y

def fitting(M=0):
    """
    M 为多项式的次数
    """    
    # 随机初始化多项式参数
    p_init = np.random.rand(M+1)
    # 最小二乘法
    p_lsq = leastsq(residuals_func, p_init, args=(x, y))
    print('Fitting Parameters:', p_lsq[0])
    
    # 可视化
    plt.plot(x_points, real_func(x_points), label='real')
    plt.plot(x_points, fit_func(p_lsq[0], x_points), label='fitted curve')
    plt.plot(x, y, 'bo', label='noise')
    plt.legend()
    return p_lsq
```


p_lsq_1 = fitting(M=1)]

![picture](https://github.com/fudonglai/merge_reponsitories/blob/master/Screenshot%20from%202018-12-10%2011-49-09.png?raw=true)

p_lsq_3 = fitting(M=3)

![picture](https://github.com/fudonglai/merge_reponsitories/blob/master/download.png?raw=true)

p_lsq_9 = fitting(M=9)

![picture](https://github.com/fudonglai/merge_reponsitories/blob/master/download%20%281%29.png?raw=true)

显然最后一个过拟合，第一个拟合不足，第二个还差不多。

# 正则化

为了解决过拟合，添加正则化项，模型越复杂，正则化项越大，越能抑制过拟合。
正则化项的一般形式为
$$\min_{f\in F} \frac{1}{N} \sum_{i=1}^n L(y_i, f(x_i)) + \lambda J(f) $$

其中 $L(y_i, f(x_i))$ 表示损失函数，说白了就是误差，就是说着一块要越小越好，越小越准确啊。一般我们损失函数都是平方函数，正则化项可以考虑 $L_1$ 或 $L_2$ 范数，一般用 $L_2$ 范数比较多吧。

那么我们解决的问题就是最小化损失函数

$$ L(w) = \frac{1}{N} (f(x_i;w)-y_i)^2 + \frac{\lambda}{2}||w||^2 $$

$||w||$ 是 $L_2$ 范数，$||w||= \sqrt{w~w^T}$，至于为什么，我推荐[一篇博客](https://www.cnblogs.com/weizc/p/5778678.html)。

以下是算法代码，从这里我发现了一点之前没有注意过的问题：
```python
def residuals_func_regularization(p, x, y):
    ret = fit_func(p, x) - y
    ret = np.append(ret, np.sqrt(0.5*regularization*np.square(p))) # L2范数作为正则化项
    return ret
```

我发现一个问题：np.append() 函数是直接在向量之后增加，相当于增加了原始向量的长度，而我记得以前学习正则化的时候，是直接把罚项加到损失函数里面，这样最小化损失函数的时候也可以同时最小化罚项，也就是简化了模型参数。而且上面那个公式不是写了要对 $L_2$ 范数平方吗，那对罚项再开根号有什么意义，都是求最小值，为何要徒增计算量？

我觉得这个函数这样写都可以:
```python
def my_residuals_func_regularization(p, x, y):
    ret = fit_func(p, x) - y
    ret = np.append(ret, np.sqrt(0.5*regularization*np.square(p)))
    # ret = np.append(ret, 0.5*regularization*np.square(p))
    # ret = ret + 0.5*regularization*np.square(p)
    return ret
```

我试了下，果然都可以，应为思想都没变，都是要同时最小化损失函数和权值向量，只是 append 的话就是把正则项同时作为损失参考指标而已，相当于把损失函数和正则项分开，同时优化。

我对这里正则化的理解是，多项式系数越高，模型越复杂，就是说参数越多，可以把每个次数的单项式理解成一个特征，然而有的特征并不是重点，却占有较大权值，应该舍弃，但是如果没有正则化项的话，这些特征就是过拟合的元凶。而有了正则化，迫使参数和模型的复杂度同时降低，就使这种特征的权值很低，接近于 0，相当于减少了冗余特征，增加了模型的泛化能力。
