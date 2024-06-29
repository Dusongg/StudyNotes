# 1 朴素贝叶斯原理，流程

![image-20240621175220583](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240621175220583.png)



![image-20240626191215946](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626191215946.png)



# 2 决策树

- 可做分类，可做回归

![image-20240621185753120](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240621185753120.png)

- 如何选择：衡量标准 —— 熵 ：pi为选择其中一个类别的概率，pi越小，不确定性越大，熵值越大

![image-20240621190316629](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240621190316629.png)

- 剪枝策略：预剪枝（限制深度，叶子节点，信息增益量等），后剪枝



## 信息增益的计算

![image-20240626171222391](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626171222391.png)

![image-20240626171233002](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626171233002.png)

![image-20240626171054363](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626171054363.png)

![image-20240626171157749](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626171157749.png)

## 信息增益率和gini洗漱

![image-20240626171833317](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626171833317.png)





## 计算题

![image-20240620014644989](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620014644989.png)



# 3 聚类

![image-20240603170153765](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603170153765.png)

无监督问题无法计算TP, FP， TN, FN



## k均值和k近邻的区别

K均值（K-means）和K近邻（K-nearest neighbors, KNN）是两种常见的机器学习算法，它们在应用和原理上有显著的区别。以下是它们的详细对比：

### K均值（K-means）算法

**定义**：K均值是一种无监督学习算法，主要用于聚类问题。它通过将数据分成K个簇，使得每个簇中的样本彼此之间的相似度最大。

**主要步骤**：

1. **初始化**：随机选择K个初始质心（簇的中心点）。
2. **分配**：将每个数据点分配给最近的质心，形成K个簇。
3. **更新**：计算每个簇的质心，更新质心位置为簇中所有点的平均值。
4. **重复**：重复分配和更新步骤，直到质心位置不再改变或达到预定的迭代次数。

**使用场景**：

- 图像压缩
- 文本分类
- 数据预处理（如降维）

**优缺点**：

- **优点**：
  - 简单易实现
  - 计算效率高
  - 对大数据集效果较好

- **缺点**：
  - 需要预先指定K值
  - 对初始质心敏感，可能陷入局部最优
  - 只适用于线性可分的数据

### K近邻（K-nearest neighbors, KNN）算法

**定义**：K近邻是一种监督学习算法，用于分类和回归问题。它通过计算与输入样本距离最近的K个训练样本，利用这些邻居的信息进行预测。

**主要步骤**：

1. **计算距离**：对给定的输入样本，计算它与所有训练样本之间的距离（如欧氏距离、曼哈顿距离）。
2. **选择邻居**：选择距离最近的K个训练样本。
3. **投票或平均**：对于分类问题，采用多数投票法决定输入样本的类别；对于回归问题，计算邻居的平均值作为预测值。

**使用场景**：

- 图像识别
- 文本分类
- 推荐系统

**优缺点**：

- **优点**：
  - 简单易实现
  - 无需训练过程，适用于小数据集
  - 可以处理多分类问题

- **缺点**：
  - 计算复杂度高，特别是对大数据集
  - 存储复杂度高，需要存储所有训练数据
  - 对噪声和不平衡数据敏感
  - 选择适当的K值和距离度量是关键

### 对比总结

| 特性           | K均值 (K-means)                       | K近邻 (KNN)                             |
| -------------- | ------------------------------------- | --------------------------------------- |
| **类型**       | 无监督学习（聚类）                    | 监督学习（分类和回归）                  |
| **目标**       | 将数据分成K个簇                       | 预测输入样本的类别或数值                |
| **输入**       | 需要输入K值                           | 需要输入K值                             |
| **计算复杂度** | 低，O(n \* K \* t)，t为迭代次数       | 高，O(n \* d)，d为特征维度              |
| **存储复杂度** | 低，只需存储K个质心                   | 高，需要存储所有训练数据                |
| **优点**       | 简单、快速、适用于大数据集            | 简单、直观、无需训练过程                |
| **缺点**       | 需预定义K值，对初始质心敏感，线性可分 | 计算和存储复杂度高，对K值和距离度量敏感 |
| **应用场景**   | 图像压缩、文本分类、数据预处理        | 图像识别、文本分类、推荐系统            |

K均值和K近邻算法各有优势，适用于不同的任务和数据特征。在选择算法时，应根据具体的应用场景、数据规模和计算资源等因素进行综合考虑。



## K-MEANS算法

### 基本概念

![image-20240603171230741](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603171230741.png)

### 工作流程

可视化网站：https://www.naftaliharris.com/blog/visualizing-k-means-clustering/

1. k = 2，则随机初始化两个点（初始化的点对最后分类结果影响较大），通过欧氏距离分成两个类

2. 更新质心

3. 重新分配，到第二步

   ![image-20240604095904250](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240604095904250.png)**

### 优缺点

![image-20240603171731614](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603171731614.png)

### 评估标准

1. inertia

中心点到该类其他点的距离总和

1. 轮廓系数

![image-20240603190304674](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603190304674.png)

![image-20240603190826759](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603190826759.png)

### 如何找到合适的k

![image-20240603185801826](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603185801826.png)

 随着k值的增大，inertia值会减小，可以找一个拐点作为k值

## DBSCAN算法

### 基本概念

![image-20240603172841631](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603172841631.png)

- 阈值minPts和半径r难选

![image-20240603173417143](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603173417143.png)

![image-20240603173537025](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603173537025.png)

### 工作流程

可视化网站：https://www.naftaliharris.com/blog/visualizing-dbscan-clustering/

1. 输入参数：数据集，半径，密度阈值
2. 类似于BFS

![image-20240603173956437](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603173956437.png)

### 优缺点

![image-20240603174159909](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603174159909.png)





## 层次聚类

两种：分裂层次聚类、凝聚层次聚类



# 4 线性回归

## [例题](https://wenku.baidu.com/view/1e7fe905a6c30c2259019e6d?pcf=2&bfetype=new&bfetype=new&_wkts_=1719401418170)

![image-20240626192537299](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626192537299.png)

![image-20240626192546514](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626192546514.png)

![image-20240626192601980](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626192601980.png)





## 相关系数与指数系数

1. 相关系数

   ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626193252958.png)

   ![image-20240626193242534](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626193242534.png)



#  5 逻辑回归

## [例题](https://blog.csdn.net/qq_63976098/article/details/132652062)

- 主要做二分类问题

![image-20240626212428210](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626212428210.png)

![image-20240626212603276](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626212603276.png)



## 交叉熵损失函数

![image-20240626231911191](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626231911191.png)



# 6 深度学习与神经网络

## 强化学习、深度 学习、机器学习

**强化学习 (Reinforcement Learning, RL)**、**机器学习 (Machine Learning, ML)**、和**深度学习 (Deep Learning, DL)** 是人工智能领域的三个重要分支，它们之间既有联系又有区别。

### 机器学习 (ML)

**机器学习**是人工智能的一个子集，研究如何使计算机通过数据学习，进行模式识别和预测。机器学习主要包括三种类型：

1. **监督学习 (Supervised Learning)**：模型在训练过程中学习输入数据和目标标签之间的映射关系。常见算法包括线性回归、决策树、支持向量机等。
2. **无监督学习 (Unsupervised Learning)**：模型在没有目标标签的情况下，从输入数据中寻找结构或模式。常见算法包括聚类（如K-means）、降维（如PCA）。
3. **半监督学习 (Semi-Supervised Learning)**：结合了监督学习和无监督学习的特点，利用少量标注数据和大量未标注数据进行训练。
4. **强化学习 (Reinforcement Learning)**：模型通过与环境互动，学习在不同状态下采取的行动，以最大化累积回报。

### 深度学习 (DL)

**深度学习**是机器学习的一个子领域，特别关注使用神经网络（尤其是深度神经网络）来学习数据的多层次表示。深度学习的关键特征在于：

1. **多层神经网络 (Multi-layer Neural Networks)**：通过多层神经元（层）来提取数据的逐层特征，自动化地发现复杂的模式。
2. **自动特征提取**：相比传统机器学习依赖手工特征工程，深度学习能够自动从原始数据中提取特征。
3. **大数据和计算资源**：深度学习依赖于大量的数据和强大的计算资源（如GPU）来进行训练。

深度学习的主要应用包括图像识别、语音识别、自然语言处理等。

### 强化学习 (RL)

**强化学习**是一种学习框架，强调智能体在动态环境中通过试错和反馈进行学习，目标是找到一个策略，以最大化累积回报。强化学习的基本组成部分包括：

1. **智能体 (Agent)**：进行决策的主体。
2. **环境 (Environment)**：智能体与之交互的外部世界。
3. **状态 (State)**：描述环境的一个时刻的特征。
4. **动作 (Action)**：智能体在某一状态下可以采取的行为。
5. **回报 (Reward)**：智能体采取某一动作后获得的反馈。
6. **策略 (Policy)**：从状态到动作的映射。

### 联系

1. **机器学习和深度学习**：深度学习是机器学习的一个子集，使用深度神经网络来解决问题。
2. **强化学习和机器学习**：强化学习是机器学习的一个分支，注重通过与环境的交互来学习最优策略。
3. **深度学习和强化学习**：深度强化学习结合了深度学习和强化学习，使用深度神经网络来近似策略和价值函数，以解决高维度和复杂的强化学习问题。

### 区别

1. **学习方式**：机器学习主要包括监督和无监督学习，强化学习通过环境反馈学习，深度学习通过神经网络多层特征学习。
2. **应用场景**：机器学习用于各种数据分析任务，深度学习擅长处理大规模和复杂的数据，如图像和语音，强化学习多用于动态决策和控制问题，如游戏AI和机器人控制。
3. **模型复杂度**：深度学习模型通常比传统机器学习模型更复杂，需要更多的数据和计算资源，强化学习模型则强调决策和策略优化。

总之，机器学习、深度学习和强化学习是相互关联但又各有侧重的三个领域，共同推动了人工智能技术的发展。

## 深度学习与神经网络

![image-20240523222128973](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523222128973.png)

- 将得分值转为概率值

![image-20240523222643195](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523222643195.png)

### 反向传播 —— 梯度下降

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523223735820.png)

![image-20240523224343546](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523224343546.png)

### 神经网络结构

![image-20240523225955849](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523225955849.png)

- 非线性操作：

  加在每次矩阵计算之后：sigmod 或relu：`f = W2(max(0, W1x))`

### 神经元个数的影响

  可视化网站：https://cs.stanford.edu/people/karpathy/convnetjs/demo/classify2d.html 

![image-20240523231606129](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523231606129.png)

### 正则化和激活函数

- 正则化的作用

![image-20240523231336484](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523231336484.png)

-  激活函数

![image-20240523231800237](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523231800237.png)

### 数据预处理

 ![image-20240523232711529](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523232711529.png)



- 解决过拟合问题

1. 加大正则化的惩罚力度 

2. drop-out

   ![image-20240523233229826](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523233229826.png)

## BP算法

![image-20240627011222251](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240627011222251.png)

# 7 支持向量机

1. 解决的问题：最好的决策边界，数据本身很难分->升维



## 计算

![image-20240620011418860](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620011418860.png)

## 例题

![image-20240620010950156](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620010950156.png)

y1 = 1  y2 = 1 y3 = -1  ,因为Σaiyi = 0  -> a1 + a2 - a3 = 0

xi （4, 3）点乘 xj（1，1）  = 4 * 1 + 3 * 1（第一个位置乘第一个位置，第二个乘第二个）

![image-20240620012239252](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620012239252.png)

![image-20240620012540654](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620012540654.png)

![image-20240620014024928](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240620014024928.png)



## 核函数

![image-20240626200157724](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626200157724.png)



# 8 评估标准与正则化

## ROC曲线如何计算

ROC（Receiver Operating Characteristic）曲线是一种用于评估二分类模型性能的图形工具。它通过考察不同阈值下分类器的表现，反映了模型在不同敏感性（True Positive Rate）和特异性（1 - False Positive Rate）水平下的权衡情况。以下是计算ROC曲线的步骤：

### 1. 计算混淆矩阵的元素

对于给定的分类阈值，计算以下四个值：

- **True Positive (TP)**: 被正确预测为正类的样本数。
- **False Positive (FP)**: 被错误预测为正类的负样本数。
- **True Negative (TN)**: 被正确预测为负类的样本数。
- **False Negative (FN)**: 被错误预测为负类的正样本数。

### 2. 计算性能度量

![image-20240603170528667](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240603170528667.png)

### 3. 设定不同的阈值

通过改变分类器的阈值，从0到1（包括所有可能的分数），计算每个阈值下的TPR和FPR。

### 4. 绘制ROC曲线

在平面坐标系中，以FPR为横轴（X轴），TPR为纵轴（Y轴），绘制所有阈值下的(TPR, FPR)点，形成ROC曲线。

### 5. 计算AUC值

AUC（Area Under the Curve）即曲线下的面积，是评估分类器性能的一个数值指标。AUC值越接近1，模型的分类性能越好。

### 示例

假设有一个二分类问题，我们可以用Python中的`scikit-learn`库来计算和绘制ROC曲线。代码如下：

```python
import numpy as np
from sklearn.metrics import roc_curve, auc
import matplotlib.pyplot as plt

# 假设y_true是实际标签，y_scores是模型预测的概率分数
y_true = np.array([0, 0, 1, 1])
y_scores = np.array([0.1, 0.4, 0.35, 0.8])

# 计算FPR和TPR
fpr, tpr, thresholds = roc_curve(y_true, y_scores)

# 计算AUC
roc_auc = auc(fpr, tpr)

# 绘制ROC曲线
plt.figure()
plt.plot(fpr, tpr, color='darkorange', lw=2, label='ROC curve (area = %0.2f)' % roc_auc)
plt.plot([0, 1], [0, 1], color='navy', lw=2, linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver Operating Characteristic')
plt.legend(loc="lower right")
plt.show()
```

这个代码首先计算了每个阈值下的FPR和TPR，然后绘制出ROC曲线，并计算了AUC值。通过这些步骤，可以有效评估二分类模型的性能。



## mae、召回率、rmse、精度、准确率等使用场景

在机器学习和数据科学中，不同的评价指标适用于不同类型的任务和问题。以下是常见评价指标及其使用场景：

### 1. 平均绝对误差（MAE, Mean Absolute Error）

**定义**：MAE是预测值与真实值之差的绝对值的平均值。

![image-20240626214824172](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214824172.png)

**使用场景**：

- **回归问题**：适用于衡量回归模型的预测误差，如房价预测、温度预测等。
- **特点**：对异常值不敏感，因为它对所有误差给予同等权重。

### 2. 均方根误差（RMSE, Root Mean Square Error）

**定义**：RMSE是预测值与真实值之差的平方的平均值的平方根。

![image-20240626214836041](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214836041.png)

**使用场景**：

- **回归问题**：同样适用于衡量回归模型的预测误差。
- **特点**：对异常值敏感，因为它对较大误差给予更高权重。

### 3. 召回率（Recall）

**定义**：召回率是指在所有实际为正类的样本中，被正确预测为正类的比例。

![image-20240626214920280](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214920280.png)

**使用场景**：

- **分类问题**：特别是在对假反例敏感的场景，如医疗诊断（病人的漏诊率）、垃圾邮件检测（漏掉垃圾邮件的比例）等。
- **特点**：侧重于找出所有正类样本，减少假反例。

### 4. 精度（Precision）

**定义**：精度是指在所有被预测为正类的样本中，实际为正类的比例。

![image-20240626214930215](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214930215.png)
其中 \( FP \) 是假正例。

**使用场景**：

- **分类问题**：特别是在对假正例敏感的场景，如垃圾邮件检测（误将正常邮件当成垃圾邮件的比例）等。
- **特点**：侧重于确保预测为正类的样本中，尽可能减少假正例。

### 5. 准确率（Accuracy）

**定义**：准确率是指被正确预测的样本数占总样本数的比例。

![image-20240626214941044](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214941044.png)

**使用场景**：

- **分类问题**：适用于样本类别分布均衡的场景。
- **特点**：容易受到类别不平衡的影响，在类别不平衡的情况下，单独使用准确率可能会产生误导性结果。

### 6. F1-Score

**定义**：F1-Score是精度和召回率的调和平均数，用于衡量模型的综合表现。

**公式**：
![image-20240626214410219](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626214410219.png)

**使用场景**：

- **分类问题**：特别是在类别不平衡的情况下，提供精度和召回率的平衡评价。

### 7. MSE

![image-20240626232757866](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626232757866.png)

### 小结

- **MAE**和**RMSE**主要用于==回归问题==，MAE对异常值不敏感，RMSE对异常值敏感。
- **召回率**和**精度**主要用于==分类问题==，召回率侧重于找出所有正类样本，精度侧重于减少假正例。
- **准确率**适用于类别分布均衡的==分类问题==，但在类别不平衡的情况下可能会产生误导。
- **F1-Score**用于在精度和召回率之间找到平衡，适合类别不平衡的==分类问题==。

根据具体应用场景和目标选择合适的评价指标，可以更准确地评估模型的性能。

## 防止过拟合 —— 正则化

![image-20240626233054126](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240626233054126.png)

