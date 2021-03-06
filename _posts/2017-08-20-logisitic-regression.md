---
layout: post
title: "机器学习之逻辑回归"
categories: ML
description: "逻辑回归（Logistic Regression）是机器学习中的监督学习算法，主要用于分类任务。这些任务预测的标签（label）通常为{0,1}，或者说是一个概率值。 例如我们预测一封邮件是否为垃圾邮件；预测天气是否为晴天、阴天、雨天和雪天等。"
modified: 2017-08-20
keywords: 逻辑回归,Logistic Regression
share: false
---

[TOC]

本文主要内容

- 逻辑回归（Logistic Regression）模型
- 决策边界（Decision Boundary）
- 成本函数（Cost Function）
- 梯度下降法（Gradient Descent）
- 多分类问题


逻辑回归（Logistic Regression）是机器学习中的监督学习算法，主要用于分类任务。这些任务预测的标签（label）通常为 {0,1}，或者说是一个概率值。 例如我们预测一封邮件是否为垃圾邮件；预测天气是否为晴天、阴天、雨天和雪天等。

### 逻辑回归（Logistic Regression）模型

从上文《机器学习之线性回归》我们知道线性回归的模型为
$$
h_\theta(x)=\theta_0+\theta_1x_1+...+\theta_nx_n=\theta^Tx
$$
可以从上面模型通过函数 g 对线性回归进行变换
$$
h_\theta(x)=g(\theta^Tx) \tag 1
$$
其中
$$
g(z)=\frac 1 {1+e^{-z}} \tag 2
$$


其中g(z) 是 **sigmoid 函数或者叫逻辑函数**。可以看出 0<= g(z)<=1

![sigmoid function](http://angrycode.qiniudn.com/sigmoid_functio.jpeg)



由 (1) 和 (2)
$$
h_\theta(x)=\frac 1 {1+e^{-\theta^Tx}} \tag 3
$$


这就是**逻辑回归的数学模型**。

因为预测值 h(x) 是在{0,1}，故还可以用概率的角度进行解析这个模型的预测结果。

以预测垃圾邮件为例，如果 y=1 表示这封邮件时垃圾邮件，那么预测模型可以表示


$$
h_\theta(x)=P(y=1|x;\theta) \tag 4
$$



### 决策边界（Decision Boundary）

我们知道 sigmoid 函数的值是在 {0,1} 范围，当 \\(h_\theta(x)\geq0.5\\) 时，预测y=1，反之则预测 y=0。

根据 sigmoid 函数可以知道要预测 \\(h_\theta(x)\geq0.5\\) 那么等价于 \\(\theta^Tx\geq0\\)

我们以这张来自于 Andrew Ng 机器学习课程的 ppt 来说明。

![](http://angrycode.qiniudn.com/decision_boundary.png)

假设通过训练得出
$$
\theta=\begin{bmatrix}
-3\\
1\\
1
\end{bmatrix}
$$


那么
$$
x_1+x_2=3 \tag 5
$$


就是**决策边界**。当


$$
x_1+x_2\geq3
$$


表示预测值 y=1，反之预测值 y=0。

除了线性决策边界，还可以是曲线的。

![non-linear-decision-boundary](http://angrycode.qiniudn.com/non-linear-decision-boundary.png)

 这里的决策边界就是


$$
x_1+x_2=1 \tag 6
$$


以上两个例子都是简单的边界，当然实际情况下，决策边界有能是复杂的图形，甚至是高维度的。

### 成本函数（Cost Function）

回忆一下线性回归中的成本函数


$$
J(\theta)=\frac 1 {2m} \sum_{i=1}^m(h_\theta(x^{(i)})-y^{(i)})^2
$$
我们定义函数


$$
Cost(h_\theta(x),y)=\frac 1 2 (h_\theta(x)-y)^2
$$


那么成本函数就可以写成


$$
J(\theta)=\frac 1 m \sum_{i=1}^mCost(h_\theta(x),y)
$$


**逻辑回归的成本函数（Cost Function）**


$$
Cost(h_\theta(x),y)=
\begin{cases}
-log(h_\theta(x)) & \text{if $y$ =1} \\
-log(1-h_\theta(x)) & \text{if $y$ =0}
\end{cases} \tag 7
$$


当 y=1 时

![](http://angrycode.qiniudn.com/minus-logx.png?imageView2/2/w/200/h/200)

当 y=0 时

![](http://angrycode.qiniudn.com/minus-log1-x.png?imageView2/2/w/200/h/200)

由于 y 取值是{0,1}，我们可以把公式 (7) 写成


$$
Cost(h_\theta(x),y)=-ylog(h_\theta(x))-(1-y)log(1-h_\theta(x)) \\ y\in {\{0,1\}}\tag 8
$$

### 梯度下降法（Gradient Descent）

由上文可以知道成本函数为


$$
J(\theta)=-\frac 1 m \sum_{i=1}^{m}(y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)})))
$$


要求\\(min_{\theta}J(\theta)\\)，需要使用微分求导公式


$$
f(x)'=(log_ax)'=\frac 1 x lna
$$


同时为了简单起见，求导时可以假设只有一个样本，这样可以把∑符号去掉，方便推导。

因为上文中 log 函数默认是以 e 底的


$$
log(h_\theta(x))' =\frac 1 {h_\theta(x)} \frac d {d\theta}h_\theta(x)
$$


因为


$$
g(z)=\frac 1 {1+e^{-z}}
$$


有
$$
\begin{equation}\begin{split}g(z)'&=\frac d {dz}(1+e^{-z})^{-1} \\
&=-(1+e^{-z})^{-2} \cdot (e^{-z})\cdot(-1)\\
&=\frac {e^{-z}} {(1+e^{-z})^2}\\
&=g(z) \cdot (\frac {e^{-z}} {1+e^{-z}})\\
&=g(z)(1-g(z))
\end{split}\end{equation}
$$


那么有


$$
\frac d {d\theta_j}g(\theta^Tx)=g(\theta^Tx)(1-g(\theta^Tx))\theta_j
$$


于是
$$
J(\theta_j)'=\frac 1 m \sum_{i=1}^m(h_\theta(x^{(i)})-y^{(i)})x^{(i)}_j \\
j \in {\{0,1,2...n\}}
\tag 9
$$


这个式子形式上与线性回归的\\(J(\theta)\\)的偏导数是很相似的，但是需要注意的是
\
预测函数 h(x) 是不一样的。

根据梯度下降法


$$
\begin{equation}\begin{split}repeat &\{\\
&\theta_j := \theta_j-\alpha\frac 1 m \sum_{i=1}^m(h_\theta(x^{(i)})-y^{(i)})x^{(i)}_j \\
\}
\end{split}\end{equation}
$$

### 多分类问题

对于二分类问题，我们很容易就可以建立模型。如果预测一个多分类问题，我们应该如何做呢？我们还是以Andrew Ng的ppt来说明

![multi-class](http://angrycode.qiniudn.com/multi-class.png)

例如要预测一个三分类的问题，假设要预测class1，那么就可以其他两类class2，class3当做一个分类，这样转换成一个二分类问题。这时候我们就需要根据已有的训练数据构造新的训练数据。预测class2，class3时，以此类推。这样就需要有三个不同训练数据。

当有一个新的数据进行预测时，我们只要计算下面式子的最大值就可以了

ß
$$
\max_i h_\theta(x^{(i)})\\ i\in\{1,2,3\}
$$

### 参考文献

Andrew Ng coursera 机器学习课程

