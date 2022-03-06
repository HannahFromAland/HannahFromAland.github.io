---
title: 【AML】Intro-and-EDA
date: 2022-03-05 15:27:39
tags: AML
category: Machine Learning
---
两天速成Applied Machine Learning （不是）

### What is and why Machine Learning?

两个lecture中出现的定义：

- Machine learning (ML) is the study of computer algorithms that can improve automatically through **experience** and by the use of **data**.
- Machine learning involves computers discovering how they can perform tasks without being explicitly programmed to do so.

总结几个个人感觉比较能够凸显ML特质的关键点：

- Machine learning和传统system的区别在于没有hardcoded rules，即不通过人为筛选出来的判断条件做决定。
- 传统规则定义存在的两个明显缺点：

  - 定义或系统一旦出现更新，整个判定逻辑可能会面临推翻重做的重大变化。即使是很小部分的新特性加入也可能会造成整个系统的调整或重写
  - 规则的建立是高度依赖因果关系，经验和专家决策的，可能存在片面性或主观性，并且不能触及仅存在相关性的决策因素

  <!-- more -->
- 人脸识别的例子：

  > One example of where this handcoded approach will fail is in detecting faces in images. Today, every smartphone can detect a face in an image. However, face detection was an unsolved problem until as recently as 2001. The main problem is that the way in which pixels (which make up an image in a computer) are “perceived” by the computer is very different from how humans perceive a face. This difference in representation makes it basically impossible for a human to come up with a good set of rules to describe what constitutes a face in a digital image.
  >
- 机器学习可以分为监督学习和无监督学习。区别来源于所提供给算法的数据集的不同，监督学习的数据集含有特征和预期结果，如广为人知的信用卡诈骗和借贷违约风险；而无监督学习的数据集只包含输入的特征数据，没有期望或实际的输出数据或标签


### Exploratory Data Analysis -- Visualization

上学期被5702折磨了一学期，现在放在特征工程之前看这个话题莫名觉得还是有些收获的。结合AML的关注点和我自己对于EDA的理解写一些在进行exploratory analysis时需要注意的地方。

#### Visualizing Distributions

- histogram & kernel density plots
  - 首先，直方图只有一个变量即x轴，展示的是整个数据集中变量的分布特征。y轴可以是频数也可以是频率，只有scale上的变化，整体的趋势并不会发生改变。
  - histogram的趋势可能会因为bin width的选择发生变化，如果区间太大可能会模糊掉具体的分布特征，但区间过小也可能导致趋势被异常值影响而不明显。此外，区间边界选择的不同也可能会导致图像发生较大的变化（左开右闭区间和左闭右开区间因为临界值个数的不同而不同），一般来讲当变量的值全部为整数时可以选择小数作为临界值来避免该问题。
  - 存在两个变量，既有x轴和y轴的图像不叫histogram！（hist在python和R语言中的命令都仅有一个输入变量，所以当遇到x和y轴是两个变量的*pseudo histogram*时...尝试把barplot的间距调成0吧)
  - 在EDA中可以将kernel density plots理解为histogram的曲线图，更适合通过曲线形式表示出连续变量的分布和波动趋势。当需要同时探索多个变量的分布时（或同一变量的不同分类），kernel density plots更加清晰直观。
  - 一般来说随机样本的分布在大样本情况下都会近似于正态分布（credits to CLT），但可能会出现左偏/右偏的情况，此时可以通过对样本取对数来判断其分布是否遵循对数正态分布
- box plot, violin plot and ridge plot
  - 同一变量的不同分类分布同样常用box plot 或violin plot，同时也有将kernel density plot进行横向排列形成的ridge plot （在使用ridge plot时，尽量按照分布中位数的值进行排序以实现aesthetic effect）

#### Visualizing relationships

- 最常用的相关性探索图非scatter plot莫属
  - scatter plot需要两个变量实现x和y轴变量之间的相关性交互。（因此出现仅有一个变量的*pesudo scatter*...尝试把histogram的柱子变成点的形式吧）
  - 散点图最直观的用处即为判断x和y是否存在线性关系，当然也可以拓展为其他具有特殊图像特征的关系或对数线性关系。

#### Visualizing proportions

广为流传的EDA冷笑话 `i hate pie chart`即为可视化比例的一个（反）例。除此之外，经常使用的还有stacked bar chart和side-by-side bar chart（懒得配图hhh）

- 饼图的问题即在于仅能对各个比例的大小进行定性的感知（并且当各分类的比例差不多/分类过多的时候，定性的感知能力会大幅下降），因此个人觉得只适用于某一类特征存在*overwhelmingly priority*且确实需要表现这一压倒性优势时才考虑采用。在EDA语境下，探索性分析需要利用图像挖掘信息，因此饼图基本没有什么合适的用武之地。
- Stacked bar chart相比饼图的优势在于人对长度和宽度的感知能力是优于角度或面积的。缺点是在分类进行横向比较时（比如不同stacks之间需要比较相同颜色的占比），感知能力会下降。因此，stacked barplot的适用于在分类较少的情况下，观察整体的比例随x轴变量变动的情况。
- side-by-side bar chart可以用于解决stacked barplot的分类比较问题，适用于需要突出各部分比例变化的场景。
