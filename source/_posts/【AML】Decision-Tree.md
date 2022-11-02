---
title: 【AML】Decision Tree
date: 2022-03-06 23:34:22
tags: 
- AML
- Tree
- Entropy
category: Machine Learning
---

决策树可以用来解决分类(classification)和回归(regression)问题。模型本身优越的可解释性及对缺失值不敏感的特性使得其被广泛采用。

<!-- more -->

### Measure of Impurity
由于决策树利用贪心算法自顶向下递归地寻找最优选择，因此算法的关键在于选择好的分裂标准。衡量分裂结果的好坏将通过计算节点的Impurity实现。

分类问题常用的两种计算方式为信息熵和基尼系数。二者均为凹的二次函数，entropy的顶点高于gini，趋近于1。Impurity的计算方式要求了各节点为了达到最小化Impurity的程度，必须选择`majority vote`作为节点的预测值。

- Entropy（信息熵）：
$$-\sum_{i=1}^kp_i log_2p_i$$
- Gini Index(基尼系数)：
$$1- \sum_{i=1}^kp_i^2$$


回归问题常用的两个指标为MSE和MAE。同样，该计算方式要求各节点预测值由样本均值决定。

### Information Gain
信息增益是指在得知A的特征之后，样本信息集合减少不确定性的程度。在分类决策树中，当一个parent node进行分裂产生两个child nodes时，信息增益指的就是整体Impurity的下降。由于决策树是贪心算法，因此在每一次分裂中都会选择信息增益最大的点以最快达到pure node的返回要求。

在回归树中，信息增益的衡量方式为计算$SSE$。

由于需要在每次分裂时选择信息增益最大的分裂方式，因此对于取值为L的分类特征，可能的分裂方式为$2^L - 2$种。为了降低时间复杂度，可以通过target encoding的方式利用输出变量对所有可能取值进行排序，然后按顺序创建子集。该油画方式可以将分裂的时间复杂度降至$O(L)$.
![](https://s2.loli.net/2022/03/07/TAXJuKZR4zoQ7Nc.png)



### Overfitting and Pruning

​树的过拟合是一个非常严重的问题。在极端情况下，每个叶子节点可能只有一个元素，这样整棵树变成了一个look-up table. 这样的树几乎没有泛化能力，不能实现generalization。
pre-pruning 和 post-pruning可以用于解决决策树的过拟合问题。
- pre-pruning: 也可以称为early stop，即当节点分类带来的收益不统计学上显著时,停止分裂或者是人工预设定停止条件使树在规定大小和范围内训练以防止过拟合。
    可以修改的参数有：`Maximum depth` `Maximum leaf nodes` `Minimum samples split` `Minimum impurity decrease`
- post-pruning：先生成整棵树，然后后剪枝。
    - Reduced error: 在决策树构造完成后，自底向上对非叶节点进行评估，如果将其换成叶节点能提升泛化性能，则将该子树换成叶节点(相当于将该节点以下的分裂全部剪掉)。后剪枝决策树欠拟合风险很小，泛化性能往往优于预剪枝决策树。
    - Cost complexity: 衡量信息增益和分裂次数之间的比例$\alpha$.避免了为达到pure node而进行很大成本的分裂的情况，也可以防止过拟合。










