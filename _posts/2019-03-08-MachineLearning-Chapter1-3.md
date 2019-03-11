---
layout:     post
title:      Machine Learning Chapter1/2/3
subtitle:   绪论 模型评估与选择 线性模型
date:       2019-03-08
author:     VickeeX
header-img: img/post-bg-casual-0.jpeg
catalog: true
tags:
    - MachineLearning
---

# Chapter 1: 绪论

## 术语
* learning/traning,学习/训练: 从数据中学得模型的过程(找到与训练集匹配的假设)
* model, 模型: 从数据中学的的结果
* pattern, 模式: 局部性结果（如一条规则）
* data set, 数据集
* instace/sample, 示例/样本
* attribute/feature, 属性/特征(or called feature vector, 特征向量)
* attribute/sample space, 属性/样本空间
* hypothesis, 假设: 学得模型所对应的关于数据的某种潜在规律
* label, 标记: 关于实例结果的信息(标记空间即输出空间)
* testing, 测试: 使用学得模型进行预测的过程, 被预测的样本称为测试样本(testing sample)
* generalization, 泛化: 学得模型适用于新样本的能力
* independent and identically distributed, 独立同分布(i.d.d):

> * learning
    * supervised learning, 监督学习: 训练数据具有标记信息
        * classification, 分类: 预测的是离散值
        * regressiong, 回归: 预测的是连续值
    * unsupervised learning, 无监督学习: 训练数据不拥有标记信息
        * clustering, 聚类: 将训练集中的样本分为若干类(cluster)
        
## 假设空间
* induction, 归纳: 从特殊到一般的“泛化”(generalization)过程 
* deduction, 演绎: 从一般到特殊的“特化”(specialization)过程
从样本中学习，属于归纳过程，即归纳学习(inductive learning)
* version space, 版本空间: 与训练集一致的假设集合

## 归纳偏好
* inductive bias, 归纳偏好: 机器学习算法在学习过程中对某种类型假设的偏好
归纳偏好可看做是学习算法在一个可能庞大的假设空间中对假设进行选择的启发式或“价值观”.若一个算法没有偏好, 则无法产生确定的学习结果.
* Occam's razor, 奥卡姆剃刀: 若有多个假设与观察一致, 选与经验观察一致的最简单假设(如: 更平滑的曲线).
NFL(No Free Lunch Theorem)定理: 无论学习算法的好坏,期望性能是相同. 总误差与学习算法无关.
故讨论算法的优劣, 需要针对具体的学习问题. 学习算法的归纳偏好与问题是否相配, 对算法的相对优劣的评估起到决定性的作用.

***
# Chapter 2: 模型评估与选择

## 经验误差与过拟合
* error rate, 错误率: 分类错误的样本数占样本总数的比例. m个样本中a个分类错误,错误率*E=a/m*
* accuracy, 精度: 分类错误的样本数占样本总数的比例. 精度=1-错误率
* error, 误差: 学习器的实际预测输出与样本的真实输出之间的差异
    * training/empirical error, 训练/经验误差: 学习器在训练集上的误差
    * generalization error, 泛化误差: 学习器在新样本上的误差
* overfitting, 过拟合: 学习器把训练样本自身的一些特点当作了所有潜在样本都会具有的一般性质(学习能力过于强大), 导致泛化性能下降. 与之相对的是欠拟合(underfitting):对训练样本的一般性质尚未学好.

>* Q: 如何进行**模型选择(model selection)**？

***
## 评估方法
模型选择的理想解决方案: 对候选模型的泛化误差进行评估, 选择泛化误差最小的模型. 但泛化误差无法直接获得, 使用训练误差又会得到过于"乐观"的结果.
故使用**测试集(testing sest)**来测试学习器对新样本的判别能力, 以测试机上的测试误差(testing error)作为泛化误差的近似.
要求: 测试集应该尽可能与训练集互斥. 
>* Q: 包含m个样例的数据集 *D={(x<sub>1</sub>,y<sub>1</sub>), (x<sub>2</sub>,y<sub>2</sub>), ..., (x<sub>m</sub>,y<sub>m</sub>)}*, 如何对D进行适当处理以从中产出训练集*S*和测试集*T*？

***
#### 留出法(hold-out)
直接将数据集D划分为两个互斥的集合, 分别作为训练集S和测试集T.即 *D=S∪T, S∩T=∅*.
note:
1. 训练/测试集的划分应尽可能保持数据分布的一致性(如: 类别比例, 等), 避免引入额外的偏差.
2. S与D的容量影响所得模型的性能: S太大->模型更接近于D训练出的模型, 结果不够稳定准确; S太小->降低了评估结果的保真性(fidelity). 一般将大约2/3~4/5的样本用于训练, 剩余样本用于测试.

* stratified sampling, 分层采样: 保留类别比例的采样方式.

单次使用留出法得到的结果不够稳定可靠, 一般采用**若干次随机划分**, 重复进行实验评估后取平均值作为留出法的评估结果.

***
#### 交叉验证法(cross validation)
将数据集D划分为k个大小相似的互斥子集, 即: *D=D<sub>1</sub>∪D<sub>2</sub>∪...∪D<sub>k</sub> , D<sub>i</sub>∪D<sub>j</sub>=∅(i≠j)*.每个子集D<sub>i</sub>都尽可能保持数据分布的一致性(即从D中分层采样得到). 每次用k-1个子集的并集作为训练集, 余下的子集作为测试集. 如此获得k组训练/测试集, 进行k次训练和测试, 最终返回这k次测试结果的均值. 此方法亦称为"k折交叉验证"(k-fold cross validation), k常取10、5、20等.

* **p次k折交叉验证**: 随机使用不同的划分重复p次, 最终结果为这p次划分的k折交叉验证结果的均值, 以减小因样本划分不同而引入的差别.
* **留一法**: k=m(m为数据集D的样本数). 即只有一种划分: 每个子集包含一个样本. 训练集与初始数据集相似, 模型与期望的用D训练出的模型相似, 结果在多数时候比较准确. 但数据集较大时, 训练开销很大. 

***
#### 自助法(bootstrapping)
**自助采样**: 给定包含m个样本的数据集*D*, 每次随机从*D*中挑选一个样本并将其拷贝放入*D'*, 该样本放回*D*中; 重复此过程m次, 得到包含m个样本的数据集*D'*. 
样本在m次采样中始终不被采到的概率为 lim (1-1/m)<sup>m</sup> -> 1/e ≈ 0.368
即*D*中约有36.8%的样本未出现在*D‘*中, 故将*D/D'*用作测试集. 这样的测试结果亦称为"包外估计"(out-of-bag estimate).

自助法适用于数据集较小、难以有效划分训练/测试集时. 且能产生多个不同的训练集, 有利于集成学习. 但自助法产生的数据集该变量初始数据集的分布, 引入了估计偏差, 故不如前两种方法常用.

***
#### 调参(parameter tuning)与最终模型
很多参数在实数范围内取值, 通常通过对每个参数选定一个范围和变化步长, 来减少调参工作量.
在模型选择完成后(学习算法和参数配置已确定), 应用数据集D重新训练模型以作为最终提交给用户的模型.
validation set, 验证集: 模型评估与选择中用于评估测试的数据集.

***
## 性能度量(performance measure)
性能度量: 衡量模型泛化能力的评价标准. 如何评估: 把学习器预测结果与真实标记进行比较.
mean squared error, 均方误差: 回归任务中的常用性能度量, 表示为: *1/m· ∑(f(x<sub>i</sub>)-y<sub>i</sub>)<sup>2</sup>*; 
对于概率密度函数p(·), 均方误差: *∫(f(x)-y)<sup>2</sup>p(x)dx*
***
#### 错误率与精度
错误率与精度是分类任务中常用的性能度量. (见前文)
>* 公式略

***
#### 查准率(precision)、查全率(recall)与F1
查准率 P=TP/(TP+FP); 查全率 R=TP/(TP+FN)
两者是矛盾的度量.
| 真相/预测结果 | 正例 | 反例 |
| ------ | ------ | ------ |
| 正例 | TP(真正例, true positive) | FN(假反例, false negative) |
| 反例 | FP(假正例, false positive) | FN(真反例, true negative) |

P-R图: 以查准率为纵轴、查全率为横轴作图, 得到P-R图.
>* Q: 如何根据P-R曲线比较学习器的性能?

**break-even point, 平衡点(BEP)**: 查准率=查全率时的取值, 查找平衡点以进行性能比较. 此度量过于简单, 不适用于用户对查准率和查全率有所偏好的情况.
**F1度量**: 一般形式为*F<sub>β</sub>* , 
$$ F_β =\frac{(1+β^2)×P×R}{(β^2×P)+R} $$
其中β度量了查全率/查准率的相对重要性: β=1时, 标准的F1; β>1时, 查全率有更大影响; β<1时, 查准率有更大影响.

多个二分类混淆矩阵: 多次训练/测试, 每次得到一个混淆矩阵, 在这多个矩阵上综合考察"查准/查全"->

* 每个混淆矩阵上分别计算查准率和查全率, 计算平均值, 得到**"宏查准率(macro-P)"、"宏查全率(macro-R)"、"宏F1(macro-F1)"**.
* 将各混淆矩阵的对应元素进行平均, 得到TP、FP、TN、FN的平均值, 再根据平均值计算得到**"微查准率(micro-P)"、"微查全率(micro-R)"、"微F1(micro-F1)"**.

***
#### ROC与AUC
根据任务需求采用不同的截断点(测试样本产生一个实数值), 截断点前一部分判作正例, 后一部分作为反例.
**receiver operating characteristic, 受试者工作特征曲线(ROC)**: 以FPR为横轴、TPR为纵轴作图.

* true positive rate, 真正例率(TPR): TPR=TP/(TP+FN) (等于查全率)
* false positive rate, 假正例率(FPR): FPR=FP/(TN+FP)

area under R0C curve, ROC曲线下的面积(AUC): 用于比较两个学习器的性能. *AUC=1-l<sub>rank</sub>* (l<sub>rank</sub> 为排序损失)

***
#### 代价敏感错误率与代价曲线
unequal cost, 非均等代价: 不同类型错误所造成的损失不同. 
代价矩阵(cost matrix): cost<sub>ij</sub>表示将第i类样本预测为第j类样本的代价.
根据代价矩阵计算的代价敏感(error-sensitive)错误率, 以二分类的代价敏感错误率举例: 
$$ E(f;D;cost) = \frac{1}{m}(\sum_{x_i∈D^+}Ⅱ(f(x_i)≠y_i)×cost_{01}+ \sum_{x_i∈D^+}Ⅱ(f(x_i)≠y_i)×cost_{10})$$

cost curve, 代价曲线: 以正例概率代价为横轴、归一化代价为纵轴作图所得曲线. 由曲线可计算学习器在固定条件下的期望总体代价.
$$ 正例概率代价: P(+)cost = \frac{p×cost_{01}}{p×cost_{01}+(1-p)×cost_{10}}$$
$$ 归一化代价: cost_{norm} = \frac{FNR×p×cost_{01}+FPR×(1-P)×cost_{10}}{p×cost_{01}+(1-p)×cost_{10}}$$

***
## 比较检验(理解不到, 暂放)
* 假设检验
    * binomial test, 二项检验
    * t-test, t检验
* 交叉验证t检验
* McNemar检验
* Friedman检验 与 Nemenyi后续检验

***
## 偏差与方差
偏差-方差分解: 解释学习算法泛化性能的重要工具

* variance, 方差: 预测输出与期望输出的差别.
* bias, 偏差: 期望输出与真实标记的差别.
* noise, 噪声: 数据集中的标记与真实标记的差别.
**泛化误差 = 偏差 + 方差 + 噪声**

***
# Chapter 3: 线性模型

线性模型基本形式: f(**x**)= **w**<sup>T</sup>**x**+b
## 线性回归(linear regression)
离散属性:

* 属性值间存在序关系: 连续化以转化为连续值
* 属性值间不存在序关系: 若有k个属性值, 则转化为k维向量

least square method, 最小二乘法: 基于**均方误差最小化**来进行模型求解.
multivariate linear regression, 多元线性回归: 需要对向量矩阵进行转化. (可能有过个节)
log-linear regression, 对数线性回归: ln *y*= **w**<sup>T</sup>**x**+b, 求取输入空间到输出空间的非线性函数映射.
考虑单调可微函数g(·), 广义线性模型: y= g<sup>-1</sup>(**w**<sup>T</sup>**x**+b), 其中g(·)称为联系函数(link function).

***
## 对数几率回归(logistic regression)
针对分类学习, 通过找一个单调可微函数将分类的真实标记y与线性回归模型的预测值联系起来. 
unit-step function, 单位越阶函数: 不连续.
logistic function, 对数几率函数: y = 1/(1+e<sup>-z</sup>) = 1/(1+e<sup>-**w**<sup>T</sup>**x**+b</sup>), 称为对数几率回归模型.
通过极大似然法(maximum likelihood method)估计此模型的**w**和b, 求得最优解可用梯度下降法、牛顿法等.

***
## 线性判别分析(linear discriminant analysis)
亦称Fisher判别分析, LDA思想: 给定训练样例集, 设法将样例投影到一条直线上, 使得同类样例的投影点尽可能接近、异类样例的投影点尽可能远离; 在对新样例进行分类时, 将其投影到同样的这条直线上, 再根据投影点的位置来确定新样本的类别.
>* 使同类样例的投影点尽可能接近, 通过计算使同类样例投影点的协方差尽可能小;
而异类样例的投影点应尽可能远离, 使其类中心之间的距离尽可能大.

***
## 多分类学习
基本思路为"拆解法", 利用二分类学习器: 先对问题进行拆分, 为拆出的每个二分类任务训练一个分类器; 在测试时, 对这些分类器的预测结果进行集成以获得最终的多分类结果. 
经典拆分策略:

* one vs one, 一对一(OvO): N个类别两两配对, N(N-1)/2个二分类任务.
* one vs rest, 一对其余(OvR): 将每次将一个类的样例作为正例, 所有其它类的样例作为反例, 训练N个分类器.
* many vs many, 一对一(MvM): 每次将若干个类作为正类, 若干个其它类作为反类.
    * error correcting output codes, 纠错输出码(ECOC): 一种常用MvM技术, 对分类器的错误有一定的容忍和修正能力.

***
## 类别不平衡(class-imbalance)问题
指分类任务中不同类别的训练样例数目差别很大的情况.
假设"训练集是真是样本总体的无偏采样", 通过"再缩放(rescaling)"对预测值进行调整. y'/(1-y') = y/(1-y) * m<sup>-</sup>/m<sup>+</sup> <式3.1>
该假设一般不成立, 三种策略:

* **undersampling, 欠采样**: 去除一些反例使得正、反例数目接近, 再进行学习.
* **oversampling, 过采样**: 增加一些正例使得正、反例数目接近, 再进行学习.
* **threshold-moving, 阈值移动**: 直接基于原始训练集进行学习, 但在用训练好的分类器进行预测时, 将上式3.1嵌入到决策过程中.
