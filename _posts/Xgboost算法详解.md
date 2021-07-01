# Xgboost算法详解

#### 0. 调参顺序

日常调参过程中常用的参数和调参顺序：

- ①：max_depth、num_leaves
- ②：min_data_in_leaf、min_child_weight
- ③：bagging_fraction、 feature_fraction、bagging_freq
- ④：reg_lambda、reg_alpha
- ⑤：min_split_gain

#### 1. 提升树 boosting

首先要明确一点，xgboost 是基于提升树的。

什么是提升树，简单说，就是一个模型表现不好，我继续按照原来模型表现不好的那部分训练第二个模型，依次类推。

来几个形象的比喻就是：

做题的时候，第一个人做一遍得到一个分数，第二个人去做第一个人做错的题目，第三个人去做第二个人做错的题目，以此类推，不停的去拟合从而可以使整张试卷分数可以得到100分（极端情况）。

把这个比喻替换到模型来说，就是真实值为100，第一个模型预测为90，差10分，第二个模型以10为目标值去训练并预测，预测值为7，差三分，第三个模型以3为目标值去训练并预测，以此类推。

#### 2. 基本原理

基本原理的学习从以下四点开始：

(1). 构造原始目标函数问题

+ xgboost目标函数包含损失函数和基于树的复杂度的正则项；

(2). 泰勒展开问题

+ 原始目标函数直接优化比较难，如何使用泰勒二阶展开进行近似；

(3). 参数优化问题

+ 假设弱学习器为树模型，如何将树参数化，并入到目标函数中；这一步的核心就是要明白我们模型优化的核心就是优化参数，没有参数怎么训练样本，怎么对新样本进行预测呢？

(4). 如何优化化简之后的目标函数

+ 优化泰勒展开并模型参数化之后的的目标函数，求叶子节点权重，进行树模型特征分割

##### 2.1 目标函数

Xgboost算法的目标函数分为两个部分，一部分为损失函数，用于参数优化，另一部分为正则化参数，用于控制参数的复杂程度。

xgboost属于一种前向迭代的模型，会训练多棵树。

对于第t颗树，第i个样本的，模型的预测值是这样的：

$
\hat{y}_{i}^{(t)}=\sum_{k=1}^{t} f_{k}\left(x_{i}\right)=\hat{y}_{i}^{(t-1)}+f_{t}\left(x_{i}\right)
$

其中 $\hat{y}_{i}^{(t)}$ 是第t次迭代之后样本i的预测结果，$f_{t}(x_{i})$ 是第t棵树的预测模型结果；

进一步，我们可以得到我们的原始目标函数，如下：

$O b j=\sum_{i=1}^{n} l\left(y_{i}, \hat{y}_{i}\right)+\sum_{j=1}^{t} \Omega\left(f_{j}\right)$

$l\left(y_{i}, \hat{y}_{i}\right)$为模型中的损失函数，$\hat{y}_{i}$为整个模型对第i个样本的预测值，$y_{i}$是第i个样本的真实值，$\sum_{j=1}^{t} \Omega\left(f_{j}\right)$为全部t棵树的复杂度之和

从这个目标函数我们需要掌握的细节是，**前后部分是两个维度的问题**

两个累加的变量是不同的:

- 一个是`i`，`i`这边代表的是样本数量，也就是对每个样本我们都会做一个损失的计算，这个损失是第`t`个树的预测值和真实值之间的差值计算（具体如何度量损失视情况而定）。 
- 另一个是累加变量是`j`，代表的是树的数量，也就是我们每个树的复杂度进行累加。

由于Xgboost是前向迭代，我们的重点在于第`t`个树，所以涉及到前`t-1`个树变量或者说参数我们是可以看做常数的。

所以我们的损失函数进一步可以化为如下，其中一个变化是我们对正则项进行了拆分，变成可前`t-1`项，和第`t`项：

$O b j^{(t)}=\sum_{i=1}^{n} l\left(y_{i}, \hat{y}_{i}^{(t)}\right)+\sum_{j=1}^{t} \Omega\left(f_{j}\right)$
$=\sum_{i=1}^{n} l\left(y_{i}, \hat{y}_{i}^{(t-1)}+f_{t}\left(x_{i}\right)\right)+\sum_{j=1}^{t} \Omega\left(f_{j}\right)$
$=\sum_{i=1}^{n} l\left(y_{i}, \hat{y}_{i}^{(t-1)}+f_{t}\left(x_{i}\right)\right)+\Omega\left(f_{t}\right)+$constant

接下来使用泰勒公式进行展开：

$l\left(y_{i}, \hat{y_{i}}^{(t-1)}+f_{t}\left(x_{i}\right)\right)=l\left(y_{i}, \hat{y_{i}}^{(t-1)}\right)+g_{i} f_{t}\left(x_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(x_{i}\right)$

$g_i$为损失函数的一阶导数，$h_i$为损失函数的二阶导数

进而可以展开目标公式如下：

$O b j^{(t)} \simeq \sum_{i=1}^{n}\left[l\left(y_{i}, \hat{y}_{i}^{(t-1)}\right)+g_{i} f_{t}\left(x_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(x_{i}\right)\right]+\Omega\left(f_{t}\right)+$ constant

##### 2.2 树的参数化

树的参数化有两个，一个是对树模型参数化，一个是对树的复杂度参数化。

(1). **树模型参数化-如何定义一个树**

主要是定义两个部分：

- 每棵树每个叶子节点的值(或者说每个叶子节点的权重)`w`：这是一个向量，因为每个树有很多叶子节点
- 样本到叶子节节点的映射关系`q`：(大白话就是告诉每个样本落在当前这个树的哪一个叶子节点上)
- 叶子节点样本归属集合`I`：就是告诉每个叶子节点包含哪些样本

**树复杂度的参数化-如何定义树的复杂度**

定义树的复杂度主要是从两个部分出发：

- 每个树的叶子节点的个数（叶子节点个数越少模型越简单）
- 叶子节点的权重值`w`（值越小模型越简单）

对于第二点，我们可以想一下L正则不就是想稀疏化权重，从而使模型变得简单吗，其实本质是一样的。

我们树的复杂度如下：

$\Omega\left(f_{t}\right)=\gamma T+\frac{1}{2} \lambda \sum_{j=1}^{T} \omega_{j}^{2}$

这是针对第k棵树得到的正则项，其中$ \gamma$ 是正则化系数，对应参数中的gamma和lambda，（reg_lambda,reg_alpha，各种别名）， $T_k$表示第k棵数的叶节点的个数，而 ![[公式]](https://www.zhihu.com/equation?tex=%7C%7Cw_%7Bk%7D%7C%7C%5E%7B2%7D) 表示第K棵数的所有叶子节点输出的平方的和，这和l2正则化的定义非常类似。



进而我们可以对树进行了参数化，带入到目标函数我们可以得到如下：

$O b j^{(t)} \simeq \sum_{i=1}^{n}\left[g_{i} f_{t}\left(x_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(x_{i}\right)\right]+\Omega\left(f_{t}\right)$
$=\sum_{i=1}^{n}\left[g_{i} w_{q}\left(x_{i}\right)+\frac{1}{2} h_{i} w_{q}^{2}\left(x_{i}\right)\right]+\gamma T+\frac{1}{2} \lambda \sum_{j=1}^{T} \omega_{j}^{2}$
$\quad=\sum_{j=1}^{T}\left[\left(\sum_{i \in I_{j}} g_{i}\right) w_{j}+\frac{1}{2}\left(\sum_{i \in I_{j}} h_{i}+\lambda\right) w_{j}^{2}\right]+\gamma T$

叶子节点 `j` 所包含的样本的一阶导数累加之和为：

$G_{j}=\sum_{i \in I_{j}} g_{i}$

叶子节点 `j` 所包含的样本的二阶导数累加之和为：

$H_{j}=\sum_{i \in I_{j}} h_{i}$

进而我们可以进一步化简为：

$O b j^{(t)}=\sum_{j=1}^{T}\left[G_{j} w_{j}+\frac{1}{2}\left(H_{j}+\lambda\right) w_{j}^{2}\right]+\gamma T$

第一就是如何求得$w_j$:这一步其实很简单，直接使用目标函数对$w_j$求导就可以。

$w_{j}=-\frac{G_{j}}{H_{j}+\lambda}$

