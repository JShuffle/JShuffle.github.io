---
layout: post
title: How to derive fully condition distribution in Gibbs sampling?
subtitle: 
date: 2020-8-10
categories: 概率论
tags: 概率论
---

# Fully condition distribution in Gibbs sampling

在gibbs sampling中，要求写出每一个变量的fully condition分布。然而，如果参数很多，这个fully condition分布会复杂的无从下手，是否有办法对其化简呢？

下面总结了一套方法。

本文主要参考PRML第8、11章以及Gibbs sampling的维基百科词条，再加上一点个人理解。

## 步骤

**1.确定所有的变量和参数，写出fully condition分布的表达式（假设我们一共有n个变量和参数，当前我们正打算对`Xj`进行采样）**

$$
p\left(x_{j} \mid x_{1}, \ldots, x_{j-1}, x_{j+1}, \ldots, x_{n}\right)
$$

**2.画出概率图(PGM)，并根据markov blanket原则简化fully condition分布**

> from PRML: The practical applicability of Gibbs sampling depends on the ease with which samples can be drawn from the conditional distributions `p(zk|z\k)`. In the case of probability distributions specified using graphical models, the conditional distributions for individual nodes depend only on the variables in the corresponding Markov blankets.

gibbs sampling中，为了完成某一个变量的抽样（或者说为了写出fully condition分布），需要借助哪些其他的变量是由markov blanket的集合决定的。
简单的总结：对于一个节点来说，其markov blanket集合就是children + parents + other parents of children.

为了演示起见，不妨假设`X1/X2/X3`就是`Xj`的markov blanket.
那么，fully condition分布可以简化为：

$$
p\left(x_{j} \mid x_{1}, x_{2}, x_{3}\right) \\
$$

**3.将fully condition分布分解为正比于joint distribution的形式(利用条件概率公式)：**

$$
p\left(x_{j} \mid x_{1}, x_{2}, x_{3}\right) \\
=\frac{p\left(x_{j},x_{1}, x_{2}, x_{3}\right)}{p\left(x_{1}, x_{2}, x_{3}\right)} \\
\propto p\left(x_{j},x_{1}, x_{2}, x_{3}\right)
$$

分母与`Xj`无关，因此可以被当作归一化因子省略掉。

> from Wikipedia：
>
> "Proportional to" in this case means that the denominator is not a function of `Xj` and thus is the same for all values of `Xj`;  it forms part of the [normalization constant](https://en.wikipedia.org/wiki/Normalization_constant "Normalization constant") for the distribution over `Xj`


**4.根据PGM中markov blanket集合内变量之间的关系，将联合分布简化**

假设在PGM中，`X1/X2/X3/Xj`的关系如下：


![1597039846(1)](https://raw.githubusercontent.com/SZJShuffle/picGo/master/1597039846(1).png)


那么，根据PGM的定义，这4个变量的联合概率可以分解(factorize)为：

$$
p\left(x_{j},x_{1}, x_{2}, x_{3}\right)\\
= p\left(x_{3}\right) * p\left(x_{2}\right) * p\left(x_{1}|x_{2},x_{3}\right) * p\left(x_{j}|x_{1}\right) \\
\propto p\left(x_{j}|x_{1}\right)
$$

其中，最后一步的目的是，进一步在分解后的表达式中，把与`Xj`无关的变量放到归一化因子内部。

> from Wikipedia：
>
> In practice, to determine the nature of the conditional distribution of a factor `Xj`, it is easiest to factor the joint distribution according to the individual conditional distributions defined by the [graphical model](https://en.wikipedia.org/wiki/Graphical_model "Graphical model") over the variables, ignore all factors that are not functions of `Xj` (all of which, together with the denominator above, constitute the normalization constant), and then reinstate the normalization constant at the end, as necessary.

**5.根据最终化简得到的形式，进行抽样。**

通过层层化简，得到：

$$
p\left(x_{j} \mid x_{1}, x_{2}, x_{3}\right) \propto p\left(x_{j}|x_{1}\right)
$$

如果`Xj`是离散型随机变量(有K个类别)，我们只需要计算如下表达式的概率，然后重新归一化并按照归一化后的概率进行采样即可：

$$
p\left(x_{j} =1|x_{1}\right)\\
p\left(x_{j}=2|x_{1}\right) \\
...\\
p\left(x_{j}=K|x_{1}\right) \\
p\left(x_{j}=i \mid x_{1}, x_{2}, x_{3}\right) = \frac{p\left(x_{j} =i|x_{1}\right)} {\sum_{k=1}^{K} p\left(x_{j} =i|x_{1}\right)}
$$

如果`Xj`是连续型随机变量（通常是指数族分布），我们可以根据其kernel判断是哪个分布，并基于该分布进行抽样。（通常，如果
如果`Xj`是连续型随机变量，其化简后的fully condition分布都会是先验分布的共轭后验）

> from Wikipedia：
>
> 1.  If the distribution is discrete, the individual probabilities of all possible values of Xj are computed, and then summed to find the normalization constant.
> 2.  If the distribution is continuous and of a known form, the normalization constant will also be known.
> 3.  In other cases, the normalization constant can usually be ignored, as most sampling methods do not require it.


## 参考

1.PRML chapter8,11

2.wikipedia:[https://en.wikipedia.org/wiki/Gibbs_sampling](https://en.wikipedia.org/wiki/Gibbs_sampling)

3.[https://stats.stackexchange.com/questions/4191/where-do-the-full-conditionals-come-from-in-gibbs-sampling](https://stats.stackexchange.com/questions/4191/where-do-the-full-conditionals-come-from-in-gibbs-sampling)