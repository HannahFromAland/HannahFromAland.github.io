---
title: 【AML】Ensemble Method
date: 2022-03-07 09:28:51
tags: 
- AML
- Bagging
- Boosting
category: Machine Learning
---

Motivation: 单个决策树的不稳定性（模型整体会因为较小的数据变动而发生较大改变），以及利用决策树处理回归问题时不能给出连续预测值（预测值个数取决于叶节点的个数）的局限性，尝试合并多个模型对一个数据集进行训练以达到1+1 > 2的效果。

根据训练集的构建和模型输出整合方式的不同可以分为Bagging和Boosting两大类。

### Bagging(Boot-strap Aggregating)

在只有一个分类器的情况下通过bootstrap在数据集中采样进行训练，最后整个全部模型的训练结果以提升性能。

- Bootstrap Sampling(拔靴法采样)
    - 数据集D含有m个数据点
    - 从D中有放回得取出m个数据点，构成了$D_1$数据集
    - $D_1$数据集就是D的一个子集

- bagging可以显著提升决策树的效果，但是不能提升KNN的效果的原因是：
Bagging只能帮助那些unstable的模型("The vital element is the instability of the prediction method",e.g. Decision tree, neural network)
- Unstable: 训练集改变一点点，会导致模型的巨大变化的算法。Decision Tree是unstable的，而KNN是stable的。

### Random Forest

Motivation: 在集成学习中，使用的模型相互之间关联越小，越能提升模型的泛化能力和整体性能。

随机森林即为决策树模型的Smart Bagging版本。通过bootstrap进行采样后，在每个模型上随机选取部分特征进行训练（利用Motivation中的特点），输出全部模型的平均值作为最终预测。

- 超参数：树的个数`num of trees`和每个树选择的feature个数`num of features`。此外，还可以对随机森林中的决策树进行参数调整。

- 模型评估和选择：通过out-of-bag score实现。在随机森林的训练过程中实现了对样本的抽取，因此对于每个样本，在整个模型中均有其未参与训练的决策树（或决策树的集合）。利用每个样本对其未参与训练的树进行预测的结果的均值进行评估。

- Feature Importance: 通过计算加权信息增量实现。权重的计算方式为经过该node进行分类的样本个数/总样本量。

### Boosting






