---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 正则化\(Regularization\)

在很多情况下, 我们无法通过增加样本数量的方式来解决过拟合的问题, 原因是在与目标函数h的次数过高. 若特征向量X的维度n较小, 我们尚且可以通过尝试降低次数的方式来优化高方差的问题. 但现实中, 我们往往需要面对几十上百种特征值x. 因此我们需要通过优化损失函数来解决这个问题, 而这就是正则化.

## 线性回归正则化

我们知道, 线性回归的梯度下降为:

$$
\theta_n := \theta_n - \alpha\frac{1}{m}\sum_{i=1}^m((h(X^{(i)}) - y^{(i)})
\frac{\partial h(X^{(i)})}
{\partial\theta_n})\\
$$

由微分项可知, 任意一项的特征值x次数越高, 系数θ的值也越大. 若是我们能通过某种方式, 抑制高次数特征值x的系数, 便能有效抑制过拟合的问题. 让我们重新定义损失函数:

$$
J(\Theta) = 
\frac{1}{2m}(\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})^2
 + \lambda\sum_{j=1}^n\theta_j^2)\\
\frac{\partial}{\partial\theta_n}
J(\Theta) = 
\frac{1}{m}(\sum_{i=1}^m((h(X^{(i)}) - y^{(i)})
\frac{\partial h(X^{(i)})}
{\partial\theta_n})
+ \lambda\theta_n)
$$

其中λ为正则化参数. 它是个人为选择的参数, 因此它可以和m相除视为同一个参数λ. 则线性回归的梯度下降:

$$
\theta_0 :=\theta_0 - \alpha\frac{1}{m}\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})\\
\theta_n := (1 - \alpha\frac{\lambda}{m})\theta_n - \alpha\frac{1}{m}\sum_{i=1}^m((h(X^{(i)}) - y^{(i)})
\frac{\partial h(X^{(i)})}
{\partial\theta_n})\\
$$

需要注意的是, θ0不需要进行抑制, 因为它的特征值永远为1, 无论次数多高. 

## 逻辑回归正则化

同样的, 由于我们通过逻辑函数, 将不连续的逻辑回归问题转换为了概率的线性回归问题. 因此同样可以使用正则化优化过拟合的问题, 首先需要重新定义损失函数:

$$
J(\Theta) = 
 - \frac{1}{m}(
\sum_{i=1}^m
(
y^{(i)} \times log(h(X^{(i)}))
+ (1 - y^{(i)}) \times log(1 - h(X^{(i)})))
+  \frac{\lambda}{2}\sum_{j=1}^n\theta_j^2
)
$$

同样地代入梯度下降后:

$$
\theta_0 :=\theta_0 - \alpha\frac{1}{m}\sum_{i=1}^m(h(X^{(i)}) - y^{(i)})\\
\theta_n := (1 - \alpha\frac{\lambda}{m})\theta_n - \alpha\frac{1}{m}\sum_{i=1}^m((h(X^{(i)}) - y^{(i)})
\frac{\partial (\Theta^T \cdot X^{(i)})}{\partial\theta_n})
$$

