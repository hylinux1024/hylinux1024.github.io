---
layout: post
title: 机器学习之线性回归
categories: ML
description: "线性回归属于监督学习算法中的一种，也是最简单的一种算法。"
modified: 2017-08-08
keywords: 线性回归,Linear Regression
share: false
---

线性回归属于**监督学习**算法中的一种，也是最简单的一种算法。本文将讲述以下内容

1. Linear Regression 线性回归
2. Cost Function 成本函数
3. Gradient Decent 梯度下降法
4. Normal Equations 正规方程
5. Probabilistic interpretation 概率论解释

## Linear Regression 线性回归

简单地
$$
h_\theta(x)=\theta_0+\theta_1x
$$

若有5个样本，令 \\(x_0=1\\) ，那么
$$
\begin{bmatrix}h_\theta(x^{(1)})\\
h_\theta(x^{(2)})\\
h_\theta(x^{(3)})\\
h_\theta(x^{(4)})\\
h_\theta(x^{(5)})\\
\end{bmatrix}=\begin{bmatrix}
1&x^{(1)}\\
1&x^{(2)}\\
1&x^{(3)}\\
1&x^{(4)}\\
1&x^{(5)}\\
\end{bmatrix}
\begin{bmatrix}
\theta_0\\
\theta_1
\end{bmatrix}
$$

若有2个变量

$$
h_\theta(x)=\theta_0+\theta_1x_1+\theta_2x_2
$$

那么

$$
\begin{bmatrix}
h_\theta(x^{(1)})\\
h_\theta(x^{(2)})\\
h_\theta(x^{(3)})\\
h_\theta(x^{(4)})\\
h_\theta(x^{(5)})\\
\end{bmatrix}=\begin{bmatrix}
1&x_1^{(1)}&x_2^{(1)}\\
1&x_1^{(2)}&x_2^{(2)}\\
1&x_1^{(3)}&x_2^{(3)}\\
1&x_1^{(4)}&x_2^{(4)}\\
1&x_1^{(5)}&x_2^{(5)}\\
\end{bmatrix}
\begin{bmatrix}
\theta_0\\
\theta_1
\end{bmatrix}
$$

一般地

$$
h_\theta(x) = \theta_0 + \theta_1x_1+...+\theta_nx_n
$$

写成向量的形式

$$
h_\theta(x)=\sum_i^m\theta_ix_i=\theta^Tx
$$

其中 \\(\theta\\) 是权重，\\( x\_i \\) 是feature，\\( h_\theta(x) \\)是 hypothesis。
对于有 m 个样本，更一般地为

$$
h_\theta(x^{(i)})=\theta^Tx^{(i)}
$$

### Cost Function 成本函数

成本函数定义为

$$
J(\theta) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2
$$

其中 y 是真实值，而 \\(h(x)\\) 为机器预测值，i 表示第几个样本，前面的 1/2 是为了后面的计算方便引用的，当 y 和 h 的差的平方和为最小时，这样的直线就是我们要找的最佳拟合直线。

那如何找到这样\\(\theta\\) 呢？这就用到了**Gradient Descent**

### Gradient Descent 梯度下降法

假设只有单变量的线性函数，那么 \\(J(\theta)\\) 可以看出是一个碗状的面，如

![](http://angrycode.qiniudn.com/1346902099_4852.png)



我们使用一些迭代公式进行搜索，找到最小值

$$
\theta_j:=\theta_j-\alpha\frac{\partial}{\partial \theta_j} J(\theta)
$$

其中 **:=**表示赋值符号，=表示比较两个值是否相等，\\(\alpha\\) 是学习率。我们需要对 \\(\theta_j\\) 求偏导

为了简单起见，我们考虑只有一个样本的情况，这样可以把**累加符号**去掉

$$
\begin{equation}\begin{split}\frac{\partial J(\theta)} {\partial \theta_j}&=\frac{\partial } {\partial \theta_j}\frac 1 2 (h_\theta(x)-y)^2\\
&=2 \cdot\frac 1 2 (h_\theta(x)-y)\frac {\partial} {\partial \theta_j}(h_\theta(x)-y)\\
&= (h_\theta(x)-y)\frac {\partial} {\partial \theta_j}(\sum_i^n\theta_ix_i-y)\\
&= (h_\theta(x)-y)x_j\end{split}\end{equation}
$$

于是，当只有一个样本的情况时

$$
\theta_j:=\theta_j+\alpha(y^{(i)}-h_\theta(x^{(i)}))x_j
$$

推广到 n 个样本

$$
\theta_j:=\theta_j+\alpha\sum_i^n(y^{(i)}-h_\theta(x^{(i)}))x_j
$$

### Normal Equations

假设X 是一个 m x n 的矩阵，那么可以

$$
X=\begin{bmatrix}-{(x^{(1)})}^T-\\-{(x^{(2)})}^T-\\\vdots\\-{(x^{(m)})}^T-\end{bmatrix}
$$

\\(\vec y\\) 是一个 m 维向量

$$
\vec y = \begin{bmatrix}y^{(1)}\\y^{(2)}\\\vdots\\y^{(m)}\end{bmatrix}
$$

由于

$$
h_\theta(x^{(i)})=(x^{(i)})^T\theta
$$

$$
\begin{equation}\begin{split}X\theta-\vec y
&=\begin{bmatrix}(x^{(1)^T})\\(x^{(2)^T})\\
\vdots\\
(x^{(m)^T})\end{bmatrix}
\begin{bmatrix}y^{(1)}\\
y^{(2)}\\
\vdots\\
y^{(m)}\end{bmatrix}\\
&=\begin{bmatrix}(x^{(1)})^T-y^{(1)}\\
(x^{(2)})^T-y^{(2)}\\\vdots\\(x^{(m)})^T-y^{(m)}\end{bmatrix}\end{split}\end{equation}
$$

又如果 z 是一个向量，那么有

$$
z^Tz=\sum_iz_i^2
$$

所以

$$
\begin{equation}\begin{split}\frac 1 {2m}(X\theta-\vec y)^T(X\theta-\vec y^)
&=\frac 1 {2m} \sum_i^m(h_\theta(x^{i})-y^{(i)})^2\\
&=J(\theta)\end{split}\end{equation}
$$

那么要求 \\(J(\theta)\\) 的最小值，那么需要对其求导，求得极值点。

这时候会用到对矩阵的求导公式

$$
\begin{equation}\begin{split}\nabla_\theta J(\theta)&=\nabla_\theta\frac 1 2(X\theta-\vec y)^T(X\theta-\vec y^)\\
&=\frac 1 2 \nabla_\theta(\theta^T X^T X\theta-\theta^TX^T\vec y-{\vec y}^TX\theta+{\vec y}^T\vec y)\\
&=\frac 1 2 \nabla_\theta tr(\theta^T X^T X\theta-\theta^TX^T\vec y-{\vec y}^TX\theta+{\vec y}^T\vec y)\\
&=\frac 1 2\nabla_\theta (tr\theta^TX^TX\theta-tr\theta^TX^T\vec y-tr\vec y^TX\theta)\\
&=\frac 1 2\nabla_\theta (tr\theta^TX^TX\theta-tr(\vec y^T(X\theta))^T-tr\vec y^TX\theta)\\
&=\frac 1 2\nabla_\theta (tr\theta^TX^TX\theta-2tr\vec y^TX\theta)\\
&=\frac 1 2(X^TX\theta+X^TX\theta-2X^T\vec y)\\
&=X^TX\theta-X^T\vec y\end{split}\end{equation}
$$

第3步用到

$$
trR=R,\text {$R$是实数}
$$

第5步用到

$$
 trA=trA^T
$$

最后用到以下公式

$$
\nabla_AtrAB=\nabla_AtrBA=B^T\tag 1
$$

$$
\nabla_{A^T}f(A)=(\nabla_Af(A))^T\tag 2
$$

$$
\nabla_AtrABA^TC=CAB+C^TAB^T\tag 3
$$

由公式(2)(3)

$$
\nabla_{A^T}trABA^TC=B^TA^TC^T+BA^TC\tag 4
$$

若 C=I，即 C 为单位矩阵，那么

$$
\nabla_{A^T}ABA^T=B^TA^T+BA^T\tag 5
$$

故要求得 \\(J(\theta)\\) 最小值，就令导数等于0

$$
X^TX\theta=X^T\vec y
$$

于是就得到所谓的 **normal equations**

$$
\theta=(X^TX)^{-1}X^T\vec y
$$
### Probabilistic interpretation 概率论解释

从上文可以知道成本函数为

$$
J(\theta) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2
$$

那么为什么会这样定义呢？下文我们来从概率论的角度来说明。

假设目标变量 y 与输入变量存在以下关系

$$
y^{(i)}=\theta^Tx^{(i)}+\epsilon^{(i)}
$$

\\(\epsilon^{(i)}\\) 表示误差项，例如样本的测量误差等因素。我们再假设这个误差是**独立同分布**的，即它是符合**高斯分布或正态分布**，那么

$$
p(\epsilon^{(i)})=\frac 1 {\sqrt {2\pi\sigma^2}}exp\left(-\frac {( \epsilon^{(i)})^2}{2\sigma^2}\right)
$$

于是

$$
p(y^{(i)}|x^{(i)};\theta)=\frac 1 {\sqrt {2\pi\sigma^2}}exp\left(-\frac {(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)
$$

\\(p(y^{(i)}|x^{(i)};\theta)\\) 表示给定以\\(\theta\\)为参数的\\(x^{(i)} y^{(i)}\\)的分布。同时需要注意的时，中间的是分号不能写成逗号，因为\\(\theta\\)不是随机变量。

通常我们把\\(p(y^{(i)}|x^{(i)};\theta)\\) 称之为 **Likelihood Function（似然函数）**

$$
L(\theta)=L(\theta;X,\vec y)=p(\vec y|X;\theta)
$$

还可以写成

$$
\begin{equation}\begin{split}L(\theta)&=\prod_{i=1}^mp(y^{(i)}|x^{(i)};\theta)\\
&=\prod_{i=1}^m\frac 1 {\sqrt {2\pi\sigma^2}}exp\left(-\frac {(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)\end{split}\end{equation}
$$

要求 \\(\theta\\) 使得样本中的值出现的概率是最大的，于是要求似然函数的最大值。

把上面的式子的连乘运算转化成加和运算，可以化简计算

$$
\begin{equation}\begin{split}l(\theta)&=logL(\theta)\\
&=log\prod_{i=1}^m\frac 1 {\sqrt {2\pi\sigma^2}}exp\left(-\frac {(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)\\
&=\sum_{i=1}^mlog\frac 1 {\sqrt {2\pi\sigma^2}}exp\left(-\frac {(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)\\
&=\sum_{i=1}^mlog\frac 1 {\sqrt{2\pi\sigma^2}}+\sum_{i=1}^m\left(-\frac {(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)\\
&=m\cdot log \frac 1 {\sqrt{2\pi\sigma^2}}-\frac 1 {\sigma^2}\cdot \frac 1 2\sum_{i=1}^m(y^{(i)}-\theta^Tx^{(i)})^2\end{split}\end{equation}
$$

于是，要求 \\( L(\theta) \\) 的最大值，就是要求以下式子的最小值

$$
\frac 1 2 \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2
$$

而这个就是我们的 **Cost Function（成本函数）**。

### 参考文档

http://cs229.stanford.edu/materials.html


