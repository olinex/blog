---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 损失函数 \(Cost Function\)

当我们在进行机器学习的时候, 我们的目标是在函数集H内找到一个函数h, 这个函数h能够很好的拟合当前的训练集D和一些未知的测试数据. 而要找出这个函数h, 我们需要一些依据或者标准, 那就是**损失函数**, 不需要对这个名词过于恐慌 \(这些专有名词总是好像哈利波特的咒语一样故弄玄虚\). 他在本质上只是函数, 不过它用于衡量另外一个函数是否令人满意, 仅此而已.

我们使用大写字母J代表损失函数, 根据不同的理论和角度, 损失函数有很多种类型, 而不同类型的损失函数在不同的使用场景中效果也不尽相同. 在这里我们使用最常见的**方差\(Squre Loss\)**作为我们的损失函数:

$$
J(\Theta) = \frac{1}{2m}\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})^2
$$

其中:

$$
\frac{1}{m}\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})^2
$$

便是目标函数h与样本输出值y的方差和和. 

为什么损失函数要在方差的基础上除以2呢? 原因是为了在未来进行梯度下降寻找最小的损失函数值时, 能够将方差微分后的2次方项抵消. 


