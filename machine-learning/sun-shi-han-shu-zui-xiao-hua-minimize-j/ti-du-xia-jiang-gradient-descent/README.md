---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 梯度下降\(Gradient Descent\)

机器学习, 在本质上是要在我们设定的一个函数集合H中, 找到一个函数h, 这个函数h能够很好地拟当前的训练集D, 并能够很好地拟合未来的数据. 而要找到这个函数, 就需要借助损失函数J. 所谓的损失函数量化了训练集D与函数的拟合情况, 损失函数值越小, 说明函数对训练数据的拟合情况越好. 因此, 寻找拟合效果好的函数的问题, 被转变成了寻找参数集Θ, 使得损失函数值最小.

那么找到使损失函数值最小的参数集Θ, 便是我们目前的目标:

$$
minJ(\theta_0,\theta_1,\theta
_2, ..., \theta_n)
$$

## 什么是梯度下降

我们假设有一个游客, 他的眼神不太好, 没戴眼镜的情况下只能看到自己十米范围内的事物. 有一天, 他独自一人去爬山, 爬到山腰的时候突然失足从山道中跌落到未开发的林木当中. 幸运的是, 他并没有受伤, 但是眼镜却找不到了, 通讯设备也遗失了. 为了自救, 他需要回到山下的村庄, 可惜因为眼神不好且在林木中迷路了, 因此他并不知道哪个方向才能下山. 

但是他却知道, 山下的山庄是这附近地势最低的地方. 因此游客用了最简单直接的办法: 观察十米范围内, 哪里地势最低, 就往那里走, 总能回到山下的村庄. 这看似简单的方法, 便是梯度下降法.

### 凸函数

很多同学可能会想到, 梯度下降有一个很严重的局限性, 那就是能够使用梯度下降的损失函数J必须为凸函数.

所谓的凸函数, 可以形象地理解为, 游客想要找到的山庄, 即使山庄是地势最低的地方, 山上的地形也不能出现盆地或者峡谷. 否则眼神不好的游客找到了一个自认为是最低点的地方, 却发现山庄不见了!

## 梯度下降的步骤

1. 从任意的Θ参数集开始, 计算损失函数J的值j1
2. 调整参数集Θ, 并重新计算损失函数J的值j2, 使得新的j2 &lt; j1
3. 不断重复上一步, 直到j小于一个阈值.

当j小于一个阈值时, 我们则认为已经找到了最小的损失函数. 由此将得到的Θ参数集代入到目标函数当中, 就能够确定我们需要的目标函数h.

这三步过程中, 步骤二依然不够高效, 毕竟参数集Θ的取值有无限种可能, 靠运气随便调整, 总是不够可靠. 那有没有一种办法, 让我们总能找到更好的参数集Θ, 游客毕竟能一眼看到十米范围内地势最低的地方, 但我们没法凭借肉眼一眼就观察到数据中哪个参数集Θ比当前的参数集Θ更加好.

幸运地是, 损失函数的"倾斜程度和倾斜方向"我们还是有办法观察到的, 那就是对固定参数集Θ的损失函数J进行偏微分.

在这里, 我们先给出这个方法的结论, 也就是梯度下降的核心算法:

$$
\theta_n := \theta_n - \alpha\frac{\partial}{\partial\theta_n}J(\Theta)
$$

这个算法有以下几个细节需要强调:

1. 其中α被称为**学习速率 \(learning rate\)**且必须大于0, 该参数与梯度下降的效率和结果息息相关, 当α过小时, 我们也许进行很多次计算都无法让损失函数值j小于阈值. 当α过大时, 我们也许无论如何都无法让损失函数值j减小. 为什么会这样呢, 让我们依然以游客的例子说明. 假设游客每走一步, 就要观察一下四周哪里的地势更低, 才能走下一步. 而如果他的步伐很小, 那他则需要走很多步才能下山. 而如果他是一个高达千米的巨人, 每走一步就跨过一个山头, 即使他能找到十米范围内地势最低的地方, 但他只要移动, 则总是走到另一个山头去了.
2. := 代表着赋值, 而并非需要解方程的等于号.
3. 每次调整参数θ时, 每个参数θ都需要进行同步调整:

$$
temp0 := \theta_0 - \alpha\frac{\partial}{\partial\theta_0}J(\Theta)\\
temp1 := \theta_1 - \alpha\frac{\partial}{\partial\theta_1}J(\Theta)\\
\theta_0 := temp0\\
\theta_1 := temp1\\
$$

## 梯度下降的类型

根据我们在进行梯度下降时, 每次更新参数集Θ时使用的样本数量m, 以及样本的选取方式, 我们可以获得不同类型的梯度下降算法, 这些算法在本质上相同, 但是会因为具体的细节而影响梯度下降的效果.

### 批量梯度下降 \(Batch Gradient Descent\)

每次更新参数集Θ, 批量使用全部的训练集D, 这种算法能够保证每次更新都能得到更好的结果, 但是计算开销大, 速度慢.

### 随机梯度下降 \(Random Gradient Descent\)

每使用一次样本, 更新一次参数集Θ, 优点是速度快, 但不能保证每次更新都能有更好的结果, 有时甚至两次更新相互抵消.

### 小批量梯度下降 \(Mini-Batch Gradient Descent\)

把样本集D分为若干部分, 分批来计算损失函数和更新参数集Θ, 这是一种折中方案, 计算开销不大, 且能保证大部分情况下每次更新都能有更好的结果.

