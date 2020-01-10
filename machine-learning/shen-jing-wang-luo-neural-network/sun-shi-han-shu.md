---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 损失函数

神经网络可以表征线性回归和逻辑回归, 其中主要的差别在于激励函数g的区别. 

## 表征线性回归

当表征线性函数时, 损失函数:

$$
J(\vec{\Theta}) = 
\frac{1}{2m}\sum_{i=1}^m
(
g(
\vec{W}^{(j)} \cdot
g(\vec{W}^{(j -1)} \cdot
... \cdot
g(\vec{W}^{(2)} \cdot 
\vec{X}^{(i)}))) 
- y^{(i)}
)^2
$$

$$
= \frac{1}{2m}\sum_{i=1}^m
(
(\prod_{j=2}^L \vec{W}^{(j)} ) \cdot
\vec{X}^{(i)}
- y^{(i)}
)^2
$$

根据权重矩阵定义可以得知:

$$
\vec{W}_{S_3 \times n}^{(2)}\\
\vec{W}_{1 \times S_{(j - 1)}}^{(j)}\\
(\prod_{j=2}^L \vec{W}^{(j)})_{1 \times n}
$$

## 表征逻辑回归

当使用神经网络表征逻辑回归时, 激励函数g可以使用sigmoid函数, 并且因为逻辑回归的不同分布, 需要将各个可能项的损失函数累加:

$$
J(\vec{\Theta}) = 
-\frac{1}{m}
\sum_{i=1}^m
\sum_{k=1}^K
(
y_k^{(i)} \times log(h(\vec{X}^{(i)}))_k + 
(1 - y_k^{(i)}) \times log(1 - h(\vec{X}^{(i)}))_k
)
$$

