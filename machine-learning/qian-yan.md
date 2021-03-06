---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 前言

## 什么是机器学习 \(Machine Learning\)

机器学习, 是上个世纪五十年代提出的一个新概念, 经过几十年的发展, 已经成为了一个跨越数学 /神经学/电子机械工程等领域的交叉学科. 简单地用一句话来说, 就是研究如何让机器像人类一样学习. 但是事实上, 目前的机器学习更多的是**一种利用机器来获得数据背后所蕴含的规律的一种手段**, 而要让机器能够像人类一样学习, 仍然是一种夙愿.

## 机器学习的组成

要完成一次机器学习之旅, 有三个必不可少的装备, 他们共同组成了机器学习:

* 训练集D\(或称呼为样本集/特征集\)
* 目标函数集H
* 训练方法E

用一句话来概括机器学习所做的事情, 那便是: 为了在目标函数集合H找到理想的目标函数h, 需要通过训练集D和训练方法E, 输出某个函数f, 使得这个函数能够满足某个表现P. 

引用大家比较认同的, Tom Mitchell对机器学习的定义:

> 为了实现任务T, 需要通过训练E, 逐步体现表现P的一个过程

## 机器学习的使用领域

机器学习是和计算机前后脚出生的, 它可以被认为是对计算机的一种使用方式. 而任何事物的出生, 都有它的理由. 在计算机问世之前, 十九世纪下半旬至二十世纪上半旬是基础学科蓬勃发展的时候. 当时大量的理论研究问世, 科研工作者需要进行规模庞大的运算工作. 这使得科研工作出现了很大的瓶颈. 而计算机也因此应运而生, 帮助科研工作者解决了棘手的运算量问题. 但很快, 人们便不仅满足于此, 随着计算机的快速发展, 计算力呈指数级暴涨, 人们又有了新的渴求.

科研工作作为一种探索性工作, 便存在一定的不确定性. 科研工作者往往需要在大量的数据中寻找规律和灵感, 由此提出一些假设, 然后再通过试验的方式进行验证. 因此便有人提出了一种想法: **能否让计算机帮助我们在数据中发现规律?**

得益于计算机那人类难以企及的计算力, 即使计算机发现的十个假设九个是错的, 也比人们在茫茫的数据里大海捞针要强的多. 也因此, 机器学习这门学科才得以面世.

因此, 适合机器学习的使用领域, 一般要满足以下几个情况:

* 需要大量重复的运算工作
* 影响数据结果的因素太多无法通过肉眼直接归纳出规律

## 机器学习的分类

机器学习的分类, 根据分类标准的不同, 有不同的分类方式:

根据输入方式的不同, 我们将机器学习分为:

* 监督学习: 样本包含标签
* 半监督学习: 仅有少部分样本包含标签
* 无监督学习: 没有样本包含标签
* 强化学习: 以正负信号反馈作为输入, 而不是固定的样本

