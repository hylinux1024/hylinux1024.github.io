---
layout: post
title: 机器学习之朴素贝叶斯
categories: ML
description: "贝叶斯定理又称贝叶斯法则。"
modified: 2017-08-08
keywords: 朴素贝叶斯,Naive Bayesian
share: false
---

本文会讲到以下内容

1. 贝叶斯定理
2. 垃圾邮件分类器

### 贝叶斯定理

贝叶斯定理又称贝叶斯法则，它的形式为

$$
P(A|B)=\frac {P(B|A)P(A)} {P(B)} \tag 1
$$

其中 P(A)、 P(B) 分别为事件 A、B出现的的概率，且 \\( P(B) \neq 0 \\)

P(A|B) 为条件概率，表示在 B 事件出现的情况下 A 的概率。

P(B) 表示全概率，它可以表示为

$$
P(B)=\sum_i^mP(B|A_i)P(A_i) \tag 2
$$

上面式子的意思是 B 事件出现的概率是由因素 m 个\\( A_i \\) 影响的。

故（1）中的条件概率又可以

$$
P(A|B)=\frac {P(B|A)P(A)} {\sum_i^mP(B|A_i)P(A_i)} \tag 3
$$

可以简单地

$$
P(A|B)=\frac {P(B|A)P(A)} {P(B|A)P(A)+P(B|\rightharpoondown A)P(\rightharpoondown A)} \tag 4
$$

其中 \\( P(A)=1-P(-A) \\)


### 垃圾邮件分类器

在机器学习中贝叶斯分类器经常见的应用是**垃圾邮件分类**。下面我们就运用贝叶斯定理推导一下垃圾邮件的分类模型算法。

假设

\\( W_1,W_2,W_3,…,W_i \\) 垃圾邮件中出现单词 i 的事件；

S 为垃圾邮件的事件，H 为正常邮件；

P(S) 垃圾邮件概率,P(H) 为正常邮件的概率；

而 P(S)、P(H) 是先验概率，表示在统计之前一封邮件的概率,这里我们假设 P(S) = P(H)=0.5；

\\( P(W_i) \\) 为单词 i 出现的概率。

那么如果有一份邮件出现了单词 \\( W_1 \\)，那么它是垃圾邮件和正常邮件的概率分别是多大呢？

根据贝叶斯定理

$$
P(S|W_1)=\frac {P(W_1|S)P(S)} {P(W_1)} \\
P(S|W_2)=\frac {P(W_2|S)P(S)} {P(W_2)} \\
\vdots \\
\\
P(S|W_i)=\frac {P(W_i|S)P(S)} {P(W_i)} \\ \tag 5
$$

这样可以计算出每个单词的垃圾邮件的条件概率。

其中 \\( P(W_i|S) \\) 表示垃圾邮件中 \\( W_i \\) 出现的概率，可以根据训练数据可以统计出来。\\( P(W_i) \\) 也可以从训练数据中得到。

但是仅仅根据其中一个单词来判断一封邮件是否是垃圾邮件，显然是不行的。这就需要计算**联合概率（Combining Probabilities）**。

#### 联合概率（Combining Probabilities）

我们从（5）的式子中选取 n 个概率最大的单词来计算它们的联合概率。

为了简单我们这里取 n=2，于是计算两个单词的联合概率

$$
P(S|W_1,W_2)=\frac {P(W_1,W_2|S)P(S)} {P(W_1,W_2)} \tag 6
$$

又由于我们假设单词 \\( W_1,W_2 \\) 是相互独立的（实际上不是，但是这个假设很有用，而且这样假设实际效果很不错，所以才称之为**朴素贝叶斯**）

所以

$$
P(W_1,W_2|S)=P(W_1|S)P(W_2|S) \tag 7
$$

又根据全概率公式有

$$
\begin{equation}\begin{split}P(W_1,W_2)&=P(W_1,W_2|S)P(S) +P(W_1,W_2|S)P(H)\\
&=P(W_1|S)P(W_2|S) P(S)+P(W_1|H)P(W_2|H)P(H) \end{split}\end{equation}\tag 8
$$

由（6）（7）（8）

$$
P(S|W_1,W_2)=\frac {P(W_1|S)P(W_2|S) P(S)} {P(W_1|S)P(W_2|S) P(S)+P(W_1|H)P(W_2|H)P(H)} \tag 9
$$

将 P(S)=P(H)=0.5 代入

$$
P(S|W_1,W_2)=\frac {P(W_1|S)P(W_2|S)} {P(W_1|S)P(W_2|S) +P(W_1|H)P(W_2|H)} \tag {10}
$$

且有

$$
P(W_1|H)=P(W_1|-S)=1-P(W_1|S)\\
P(W_2|H)=P(W_2|-S)=1-P(W_2|S) \tag {11}
$$

最终

$$
P(S|W_1,W_2)=\frac {P(W_1|S)P(W_2|S)} {P(W_1|S)P(W_2|S) +(1-P(W_1|S))(1-P(W_2|S))} \tag {12}
$$

这个就是 \\( W_1,W_2 \\) 的**联合概率**。

一般地

$$
P(S|W_1,W_2,...,W_i)=\frac {P_1P_2...P_i} {P_1P_2...P_3 +(1-P_1)(1-P_2)...(1-P_i)} \tag {13}
$$

上式就是**垃圾邮件的分类模型**。




#### 参考文档

http://cs229.stanford.edu/materials.html

http://www.mathpages.com/home/kmath267/kmath267.htm

https://en.wikipedia.org/wiki/Bayes%27_theorem

http://www.paulgraham.com/spam.html

