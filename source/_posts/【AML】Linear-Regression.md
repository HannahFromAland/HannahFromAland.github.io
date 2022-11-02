---
title: 【AML】Linear Regression
date: 2022-03-06 14:53:27
tags: 
- AML
- OLS
- Lasso
- Ridge
- Elastic-Net
category: Machine Learning
---

总结线性回归的基本要点及正则化的方法和改进。

<!-- more -->

### Assumptions

线性回归的基本假设：
- Linearity: 输出值是可以通过输入值的线性组合进行表示的，即线性回归的标准表达式$$\hat y = X \vec w+ \vec b$$    
    - $X$ 为 $m\times n$ 矩阵，即观测数据量为$m$，特征个数为$n$
    - $y$ 为$m\times 1$向量，对应m个样本中的输出标签
    - $\vec w$ 为$n\times1$ 向量，分别对应n个特征的权重取值
- Independence: 样本数据之间是独立的。如果样本之间存在共线性/相关性，可能会在模型构建时影响各个特征真实的权重，出现类似辛普森悖论的现象。
- Homoscedasticity & Normality: 假设误差是均值为0的正态分布。
![](https://s2.loli.net/2022/03/07/Yo2y3ViMdtzpjKD.png)
在进行线性回归之后可以对产生的residual进行可视化。由于使用MSE作为损失函数时residual的差值会被平方项放大，因此模型可能会为了减小整体的error rate以偏向于异常值。当出现这种情况时，可以选择对异常值进行删除，也可以对特征进行对数处理来削弱异常值的影响（如右图，residual的分布近似于均值为0的正态分布）

### Close-form Solution
当线性回归的损失函数为MSE时，即等同于最小二乘（OLS）问题，此时存在close-form solution：
$$ \vec w = (X^TX)^{-1}X^Ty
$$
线性回归同样可以使用梯度下降进行loss求解。在closeform和梯度下降之间进行选择时可以考虑以下几点：
- 当X是较高维矩阵时，对$(X^TX)^{-1}$的计算可能会非常复杂且耗费较高的计算成本，此时可以选择梯度下降进行求解
- 梯度下降要求被求解问题一定是凸问题。在非凸函数中进行梯度下降可能会求得局部最优解，此时可以选择close-form function进行求解

在计算矩阵的逆的过程中可能会出现的问题如下：
- 矩阵$X$中n的值远大于m，即拥有的样本较少但特征较多，此时线性回归方程可能不只有一个最优解，且在该情况下很容易出现对于较少数据点产生的过拟合现象。
- 矩阵存在多重共线性时，$(X^TX)$可能不是可逆矩阵，因此无法进行最优解的求解。

### Bias Variance Tradeoff
![](https://s2.loli.net/2022/03/07/haPgIpz8kZC6UTr.png)
复杂模型：high variance, low bias
简单模型：low variance, high bias

一般通过**正则化方法**解决模型的过拟合问题。正则化的本质是对权重w的约束。某个特征的权重越小，该特征就越不能起决定作用，无关紧要的特征只能对模型进行微调，扰动较小，可以让模型专注于有决定性的那些特征。

### Ridge Regression -- L2 normalization

L2正则化即是在最小二乘法的基础上加入$L2-norm$的平方。Ridge regression同样存在close-form解，其中$\alpha$为超参数，可以进行调整。

![](https://s2.loli.net/2022/03/07/dNUYQcJDgBsuWpb.png)

### Lasso Regression -- L1 normalization
> 思想：L2相比OLS，计算出来的系数更稳定，但是多重共线性仍然存在，有没有一种类似这种添加惩罚项的办法来解决掉多重共线性的问题？将L2正则化中的惩罚项替换成L0-norm(系数中非0的个数)，即无关的维度系数为0去除掉该维度。但为了避免L0-norm存在的很多问题（函数非连续，很难求解），用L1-norm(系数的绝对值)来近似地取代L0-norm。

损失函数增加L1损失项，即参数的1-范数，用此损失函数没有close-form solution。Lasso 回归中的$\alpha$同样为超参数，可以进行参数选择。

![](https://s2.loli.net/2022/03/07/mURnApctyE2jviF.png)

L1范数可以进行特征选择，因此在出现多重共线性时，Lasso Regression将随机选择共线特征中的任一作为模型的非0参数，并将其他特征的权重设为0以解决多重共线性问题。

### Lasso & Ridge -- Comparison
![](https://s2.loli.net/2022/03/07/HZRLGwJs5le8Itj.png)
图中$\hat \beta$ 为OLS回归下达到最小MSE的取值。等高线代表不同MSE取值的位置，蓝色区域代表L1和L2正则化的约束区间。等高线与约束区间相切/相交的位置为最优取值。因此Lasso回归中必有某些参数的取值为0，起到了特征选择的效果。

### Elastic-Net Regression -- L1+L2
将L1和L2范数进行加权组合。由于加入了L1范数，因此同样不存在close-form solution. 其中$\lambda, \alpha$均为超参数，可以进行调整。
![](https://s2.loli.net/2022/03/07/QFnf3jedZqUOsx1.png)
