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
g(\vec{W}^{(1)} \cdot 
\vec{X}^{(i)}))) 
- y^{(i)}
)^2
$$

$$
= \frac{1}{2m}\sum_{i=1}^m
(
(\prod_{j=1}^L \vec{W}^{(j)} ) \cdot
\vec{X}^{(i)}
- y^{(i)}
)^2
$$



