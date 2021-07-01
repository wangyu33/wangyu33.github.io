# ESL

ESL 指的是 [The Elements of Statistical Learning](https://esl.hohoweiya.xyz/book/The Elements of Statistical Learning.pdf)

## 目录

- 第一章：导言
- 第二章：监督学习的综述
- 第三章：回归的线性方法（**新：**LAR算法和lasso的一般化）
- 第四章：分类的线性方法（**新：**逻辑斯蒂回归的lasso轨迹）
- 第五章：基本的扩展和正则化（**新：**RKHS的补充说明）**RKHS（再生核希尔伯特空间）**
- 第六章：核光滑方法
- 第七章：模型评估与选择（**新：**交叉验证的长处与陷阱）
- 第八章：模型推论与平均
- 第九章：补充的模型、树以及相关的方法
- 第十章：Boosting和Additive Trees（**新：**生态学的新例子，一些材料分到了16章）
- 第十一章：神经网络（**新：**贝叶斯神经网络和2003年神经信息处理系统进展大会(NIPS)的挑战）
- 第十二章：支持向量机和灵活的判别式（**新：**SVM分类器的路径算法）
- 第十三章：原型方法和邻近算法
- 第十四章：非监督学习（**新：**谱聚类，核PCA，离散PCA，非负矩阵分解原型分析，非线性降维，谷歌pagerank算法，ICA的一个直接方法）
- 第十五章：随机森林
- 第十六章：实例学习
- 第十七章：无向图模型
- 第十八章：高维问题

## 第二章：监督学习的综述

输入变量=》预测变量

输出变量=》响应变量

预测定量的输出=》回归	输出值为连续值

预测定性的输出=》分类	输出值为离散值

#### 简单的预测方法

**线性模型和最小二乘**

$\hat{Y}=\hat{\beta}_{0}+\sum_{j=1}^{p} X_{j} \hat{\beta}_{j}$

$\hat{\beta}_{0}$ 为截距，即机器学习中的偏差，经常为了方便起见把常数变量 1 放进 $X$（列向量），把 $\hat{\beta}_0$ 放进系数变量$\hat{\beta}$中，然后把用向量内积形式写出线性模型:
$$
\begin{gather*}
\hat{Y}=X^{T} \hat{\beta}
\end{gather*}
$$
通过选取系数$\beta$来约束残差平方和最小

$\operatorname{RSS}(\beta)=\sum_{i=1}^{N}\left(y_{i}-x_{i}^{T} \beta\right)^{2}$

化简后

$\operatorname{RSS}(\beta)=(\mathbf{y}-\mathbf{X} \beta)^{T}(\mathbf{y}-\mathbf{X} \beta)$

对其进行求导操作：

$X^{T}(y-X\beta)=0$

若$X^{T}X$非奇异，可得唯一解：

$\hat{\beta}=(X^{T}X)^{-1}X^Ty$

预测值$\hat{y}(x_0)=x_0^T\hat{\beta}$





**最近邻方法**

最邻近方法用训练集 $T$中在输入空间中离 $x$最近的观测组成 $\hat{Y}$．特别地，对 $\hat{Y}$的 k-最近邻拟合定义如下：

$\hat{Y}(x)=\frac{1}{k} \sum_{x_{i} \in N_{k}(x)} y_{i}$



#### 损失函数

损失函数的意义：

使用一个函数来惩罚预测中的错误预测
$$
\begin{aligned}
\mathrm{EPE}(f) &=E(Y-f(X))^{2} \\
&=\int[y-f(x)]^{2} \operatorname{Pr}(d x, d y)
\end{aligned}
$$
优化目标为在给定X的条件下使得EPE最小：

$f(x)=\operatorname{argmin}_{c} \mathrm{E}_{Y \mid X}\left([Y-c]^{2} \mid X=x\right)$

解为

$f(x)=E(Y|X=x)$



#### 维度灾难

维度越高，样本分布就越稀疏，因此需要更密集的采样

如对于一个p维空间，采样1%空间的区域，对于每一维的采样密度需要覆盖$0.01^{1/p}$的区间



#### 函数逼近

1. **线性逼近**

$f(x)=x^T\beta$

近似都与一系列系数 θθ 有关，可以修改这些系数去适应手头上数据；

2. **线性基展开**

$f_{\theta}(x)=\sum_{k=1}^{K} h_{k}(x) \theta_{k}$

其中，$h_k$ 是适合输入向量$x$的一系列函数或变换关系．传统的例子有多项式或者三角函数，其中$h_k$可能是 $x_1^2$,$x_1$,$x_2^2$,$cos(x_1)$ 等等；

3. **非线性展开**

$$h_{k}(x)=\frac{1}{1+\exp \left(-x^{T} \beta_{k}\right)}$$

如神经网络中的激活函数sigmoid变换函数;



#### 参数估计

1. **最小二乘法**

在处理线性模型时，可以用最小二乘来估计$f_\theta$中的参数 $\theta$，也就是最小化下面关于$\theta$的残差平方和

$RSS(\theta)=\sum_{i=1}^{N}(y_i-f_\theta(x_i))^2$

2. **极大似然估计**

极大似然的原则是假设最合理的$\theta$值会使得观测样本的概率最大

假设我们有一个指标为$\theta$ 的密度为$Pr_\theta(y)$的随机样本 $y_i$,i=1,…,N。观测样本的概率对数值为

$L(\theta)=\sum_{i=1}^{N} \log \operatorname{Pr}_{\theta}\left(y_{i}\right)$

通常求$\theta$的偏导数，来获取$L(\theta)$的极大值



3. 最大后验
4. 贝叶斯估计









