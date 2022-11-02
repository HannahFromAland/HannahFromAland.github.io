---
title: 【AML】Model Evaluation
date: 2022-03-07 10:52:33
tags: 
- AML
category: Machine Learning
---

### Evaluation Metrics for Classification

#### Threshold-based metrics -- Confusion Matrix

混淆矩阵，指真实数据和预测结果的四种可能情况：
![](https://s2.loli.net/2022/03/08/qe6pQSZOR3jmW5s.png)

注意：
- 四种情况的Positive/Negative对应预测出来的结果，True/False对应预测正确与否。因此，正确预测的样本为True Negative和True Positive
- 一般来说，为了保证分类中较小类型标签预测的准确程度尽可能地高，在评估时要选择占比较小的标签类型作为Positive。

利用混淆矩阵定义的评估指标如下：
- Accuracy：最常用的评估指标。即所有样本中预测正确的概率：
$$
Accuracy = \frac{TN+TP}{TN+TP+FN+FP}
$$
    - 在数据不平衡的情况下，可能会出现由于模型只输出某一标签但准确率很高的情况。因此，当数据不平衡的情况出现时，需要结合以下其他的指标对模型的预测能力进行评估。
- Precision：所有预测为阳性的样本中预测正确的概率：
$$Precision = \frac{TP}{FP+TP}$$

- Recall(TPR): 所有实际为阳性的样本中预测正确的概率：
$$Recall = \frac{TP}{FN+TP}$$

- F1-score: 结合Precision和Recall对模型进行评估：
$$ 2\frac{Recall*Precision}{Recall+Precision}$$

- Average Metrics: Macro & weighted
由于Precision，recall和F1-score都仅关注了positive样本的预测情况，如果要同时考虑所有的分类就可以使用macro/weighted metrics，即分别考虑各类样本作为positive的情况后求均值。
$$Macro = \frac{1}{L} \sum_{l\in L} R(y_l , \hat y_l)
)$$
$$Weighted = \frac{1}{n} \sum_{l\in L} n_l R(y_l , \hat y_l)
)$$

#### Ranking-based metric
- PR-curve: PR曲线下的面积即为Average Precision。AP越大，模型的表现越好。

-  Receiver Operating Curve(ROC): ROC曲线是FPR和TPR(recall)之间的关系。FPR表示实际负样本中被错误预测为正类的比例。

