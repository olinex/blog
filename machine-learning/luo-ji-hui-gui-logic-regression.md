---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 逻辑回归\(Logic Regression\)

## 什么是逻辑回归

逻辑回归与线性回归同属于回归分析. 和线性回归不同的是, 逻辑回归的结果并不连续. 如:

* 分析猪肉价格和通货膨胀的关系, 为线性回归
* 分析父母的身高与子女身高, 为线性回归
* 分析肿瘤大小与是否为恶性肿瘤的关系, 为逻辑回归
* 分析邮件的内容与是否为垃圾邮件的关系, 为逻辑回归

逻辑回归最重要的特征, 是结果**有限且非连续**. 

### 二项分布和多项分布

根据逻辑回归可能的结果值的数量不同, 可以将逻辑回归分为**二项分布**逻辑回归和**多项分布**逻辑回归. 一般的是非对错问题, 都是二项分布. 例如: "是否为垃圾邮件?"就是二项分布. 我们习惯将"对"/"是"/"阳性"等认为是正值1, "错"/"否"/"阴性"等认为是负值0.

而多项分布的逻辑回归问题, 例如: "用户是哪个国家的人?", 可以分解成多个二项分布问题, 例如:

* 是否为中国人?
* 是否为美国人?
* 是否为日本人?
* ...

那么对于具有k项分布的问题, 我们应该拆分成多少个二项分布问题呢?

这主要取决于k项分布是否满足完全的概率, 若k项分否取尽了所有的可能项, 则只需要分解成k-1个二项分布, 否则则需要分解成k个二项分布.

## 如何表征逻辑回归

当我们尝试去归纳特征值的集合X与结果值y的关系, 我们假设存在这么一个函数f:

$$
f(X) = \left \{
\begin{matrix}
1\ \ if\ h(X) \ge p\\
0\ \ if\ h(X) \lt p\\
\end{matrix}
\right \}\\
$$

而因为多项分布逻辑回归问题可以转化为多个二项分布逻辑回归问题, 所以只要找到合适的目标函数h和阈值p, 就能够通过函数表征任意的逻辑回归问题了.

从概率的角度出发, 我们很自然地可以假设, 目标函数h是一种可以输出概率的函数, 并且p = 0.5:

$$
H = \{ h | y = h(X), 0 \le y \le 1, X \in R^n \}
$$

我们大胆根据现实的概率分布, 假设一下这个函数的一些特性: 

* 当x趋向于正无穷大时, y无限趋向于1, 当x趋向于负无穷大时, y无限趋向于0
* 当x值增大时, y随之增大, 当x值减小时, y随之减小

那么真的有这种函数吗? 有的, 让我们直接给出结论:

$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$

这个函数我们称之为逻辑函数, 或Sigmoid函数. 将其与n元一次函数复合, 就成为了逻辑回归的目标函数h:

$$
z = \Theta^T \cdot X\\
h(X) = \sigma(X) = \frac{1}{1 + e^{-\Theta^T \cdot X}}\\
$$

Sigmoid有以下特点:

$$
\lim_{z \to +\infty}\sigma(z) = 1 \\
\lim_{z \to -\infty}\sigma(z) = 0 \\
\sigma(0) = 0.5
$$

## 决策边界 \(Decision Boundary\)

从上面表征逻辑回归的Sigmoid函数可以看出, 我们根据现实生活的经验, 给出了做出决策的一个边界值, 或者说是阈值, 即:

$$
f(X) = \left \{
\begin{matrix}
1\ \ if\ h(X) \ge 0.5\\
0\ \ if\ h(X) \lt 0.5\\
\end{matrix}
\right \}\\
$$

由此可以推导出决策边界方程:

$$
0 = \Theta^T \cdot X
$$

只要我们能够通过某种方式求的参数集Θ的最优解, 我们就能够通过一个函数来判断逻辑回归问题.

## 损失函数 \(Cost Function\)

要寻找参数集Θ的最优解的方法与线性回归的类似, 通过梯度下降, 找到损失函数J的最小值, 就可以获得Θ. 让我们使用逻辑回归的目标函数h来代入方差形式的损失函数J.

$$
J(\Theta) = \frac{1}{2m}\sum_{i=1}^m(\sigma(X^{(i)}) - y^{(i)})^2\\
J(\Theta) = \frac{1}{2m}\sum_{i=1}^m(\frac{1}{1 + e^{-\Theta^T \cdot X^{(i)}}} - y^{(i)})^2\\
$$

我们尝试对损失函数J计算其对Θ的二阶偏微分可以发现, 方差形式的损失函数并不是一个凸函数. 因此无法使用梯度下降来计算参数集Θ的最优解.

现在我们需要寻找一个新的损失函数J:

$$
J(\Theta) = \frac{1}{m}\sum_{i=1}^mCost(h(X^{(i)}), y^{(i)})\\
$$

当y = 1时:

$$
h(X) \to 1,\ then\ Cost(X, y) \to 0\\
h(X) \to 0,\ then\ Cost(X, y) \to +\infty\\
$$

这个函数我们很熟悉:

$$
Cost(X, y) = 
- y \times log(h(X)) = 
- log(h(X))
$$

同样的, 当y = 0时:

$$
h(X) \to 1,\ then\ Cost(X, y) \to +\infty\\
h(X) \to 0,\ then\ Cost(X, y) \to 0\\
$$

这个函数也同样容易找到:

$$
Cost(X, y) = (y - 1) \times log(1 - h(X)) = - log(1 - h(X))
$$

y非1即0, 因此可以将两个情况的公式直接相加:

$$
J(\Theta) = 
\frac{1}{m}
\sum_{i=1}^m
(
- y^{(i)} \times log(h(X^{(i)}))
+ (y^{(i)} - 1) \times log(1 - h(X^{(i)}))
)\\
= - \frac{1}{m}
\sum_{i=1}^m
(
y^{(i)} \times log(h(X^{(i)}))
+ (1 - y^{(i)}) \times log(1 - h(X^{(i)}))
)
$$

## 损失函数最小化

重新定义损失函数J后, 我们使用梯度下降来进行损失函数最小化:

$$
\frac{\partial}{\partial\theta_n}J(\Theta) = 
- \frac{1}{m} \sum_{i=1}^m
(
y^{(i)}
\frac{1}{h(X^{(i)})}
\frac{\partial h(X^{(i)})}{\partial
 \theta_n}
-
(1 - y^{(i)})
\frac{1}{1- h(X^{(i)})}
\frac{\partial h(X^{(i)})}{\partial
 \theta_n}
)
$$

$$
= - \frac{1}{m} \sum_{i=1}^m
\frac{y^{(i)} - h(X^{(i)})}
{h(X^{(i)})(1- h(X^{(i)}))}
\frac{\partial h(X^{(i)})}{\partial
 \theta_n}
$$

$$
\frac{\partial h(X)}
{\partial\theta_n} = 
\frac{-1}{(1 + e^{-\Theta^T \cdot X})^2}
\frac{\partial(e^{-\Theta^T \cdot X})}{\partial\theta_n}
$$

$$
= \frac
{e^{-\Theta^T \cdot X}}
{(1 + e^{-\Theta^T \cdot X})^2}
\frac{\partial (\Theta^T \cdot X)}{\partial\theta_n}
$$

$$
= h(X)(1- h(X))
\frac{\partial (\Theta^T \cdot X)}{\partial\theta_n}
$$

$$
\frac{\partial}{\partial\theta_n}J(\Theta) = 
- \frac{1}{m} \sum_{i=1}^m
((y^{(i)} - h(X^{(i)}))
\frac{\partial (\Theta^T \cdot X^{(i)})}{\partial\theta_n})
$$

从上面的推导, 是不是感觉特别经验? 线性回归和逻辑回归的梯度下降表达式, 竟然是一模一样的!

$$
\theta_0 :=\theta_0 - \alpha\frac{1}{m}\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})\\
\theta_n := \theta_n - \alpha\frac{1}{m}\sum_{i=1}^m((h(X^{(i)}) - y^{(i)})
\frac{\partial (\Theta^T \cdot X^{(i)})}{\partial\theta_n})
$$

