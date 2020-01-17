---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 损失函数最小化

神经网络可以表征线性回归和逻辑回归, 其中主要的差别在于激励函数g的区别. 在这里, 我们使用最常见的sigmoid函数作为逻辑回归的激励函数, 并使用梯度下降进行损失函数最小化.

## 线性回归

当表征线性函数时, 损失函数为:

$$
J(W) 
= \frac{1}{2m}
(\sum_{i=1}^m
(
h(X^{(i)})
- y^{(i)}
)^2 + 
\lambda
\sum_{l=2}^L
\sum_{i=1}^{S_j}
\sum_{j=1}^{S_{(j - 1)}}
(w_{ij}^{(l)})^2
)
$$

## 逻辑回归

当使用神经网络表征逻辑回归时, 因为逻辑回归的不同分布, 需要将各个可能项的损失函数累加:

$$
J(W) =
-\frac{1}{m}(
\sum_{i=1}^m
\sum_{k=1}^K
(
y_k^{(i)} \times log(h(X^{(i)}))_k + 
(1 - y_k^{(i)}) \times log(1 - h(X^{(i)}))_k
)
- \frac{\lambda}{2}
\sum_{l=2}^L
\sum_{i=1}^{S_j}
\sum_{j=1}^{S_{(j - 1)}}
(w_{ij}^{(l)})^2
)
$$

## 损失函数求导

对于神经网络而言, 无论是线性回归还是逻辑回归, 梯度下降的核心问题依然是损失函数J的求导. **为了简化问题, 我们假设有且仅有一个样本集**, 假设我们某个权重w求取损失函数的偏微分:

$$
\frac
{\partial J(W)}
{\partial w}
$$

我们假设一个中间向量Z:

$$
z_i^{(l)} = 
W_{i}^{(l)} \cdot A^{(l-1)} = 
\sum_{j=0}^{S_{(l-1)}}w_{ij}^{(l)}a_i^{(l-1)}
$$

由线性回归和逻辑回归的梯度下降求导可以得知, 他们的偏微分都表示为:

$$
\frac{\partial J(W)}{\partial w}
= 
\sum_{i=1}^{S_L}
(
(a_i^{(L)} - y_i)
\frac{\partial z_i^{(L)}}{\partial w}
)
$$

$$
\frac
{\partial z_i^{(L)}}
{\partial w}  
=
\frac
{\partial (W_i^{(L)} \cdot A^{(L-1)})}
{\partial w}
=
\frac{\partial}{\partial w}
(
\sum_{j=0}^{S_{(L-1)}}w_{ij}^{(L)}a_j^{(L-1)}
)
$$

$$
= \sum_{j=1}^{S_{(L-1)}}
(
w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
\frac{\partial z_j^{(L-1)}}{\partial w}
)
$$

由此可以得知, 对于任意l层的第i个单元而言:

$$
\frac{\partial z_i^{(l)}}{\partial w}
=
\sum_{j=1}^{S_{(l-1)}}
w_{ij}^{(l)}
\frac{\partial a_j^{(l-1)}}{\partial z_j^{(l-1)}}
\frac{\partial z_j^{(l-1)}}{\partial w}
$$

{% hint style="info" %}
对任意的偏置单元求权重的偏微分都等于0
{% endhint %}

因此我们可以逐层推导出:

$$
\frac{\partial J(W)}{\partial w} =
\sum_{i=1}^{S_L}
(
(a_i^{(L)} - y_i)
\frac{\partial z_i^{(L)}}{\partial w}
)
$$

$$
= 
\sum_{i=1}^{S_L}
\sum_{j=1}^{S_{(L-1)}}
(
(a_i^{(L)} - y_i)
(
w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
)
\frac{\partial z_j^{(L-1)}}{\partial w}
)
$$

$$
= 
\sum_{i=1}^{S_L}
\sum_{j=1}^{S_{(L-1)}}
\sum_{k=1}^{S_{(L-2)}}
(
(a_i^{(L)} - y_i)
(
w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
)
)(
w_{jk}^{(L-2)}
\frac{\partial a_k^{(L-2)}}{\partial z_k^{(L-2)}}
)
\frac{\partial z_k^{(L-2)}}{\partial w}
)
$$

对于线性回归而言:

$$
\frac{\partial a_i^{(l)}}{\partial z_i^{(l)}} = 1
$$

对于逻辑回归的sigmoid函数而言:

$$
\frac{\partial a_i^{(l)}}{\partial z_i^{(l)}} 
= 
a_i^{(l)}(1- a_i^{(l)})
$$

至此, 神经网络的损失函数J求导完毕. 我们可以通过上述的公式计算得出损失函数对任意权重w的偏微分. 神经网络的嵌套结构虽然使得其能够表征具有复杂多项式的目标函数, 但若不能有效地优化梯度下降的效率, 那神经网络依然不具备优势. 如果我们直接通过套用公式来运算, 将存在大量的重复运算, 这对提高神经网络表征复杂多项式目标函数的能力是不利的.

而幸好地是, 通过**特征值的正向传播**和**误差修正的反向传播**, 可以有效解决这个问题.

## 正向传播

所谓正向传播, 即是从特征向量X开始, 计算每一层逻辑单元的激励值a, 最终通过输出层, 得到预测的y值. 每个单元的的激励值, 都可以通过上一层激励值或特征向量计算得出.

$$
a_i^{(j)} = g(\vec{W}_{i}^{(j)} \cdot \vec{A}^{(j-1)})
$$

因此我们计算出了神经网络中每一个逻辑单元的激励值, 用以进行下一步的反向传播.

## 反向传播

当我们通过正向传播获得了每个逻辑单元的激励值后, 我们便可以计算出由目标函数h计算出的输出值与真实的输出值之间的误差. 对于输出层的每个单元而言, 误差的产生是隐藏层以及输出层权重的误差所导致的, 这种误差通过层层传递, 最终影响了目标函数h的输出. 我们最终的目标, 是希望输出层的误差能够足够地小. 因此我们希望误差能够从输出层反向传播给隐藏层的每个单元, 用以修整每个单元的权重. 这就是反向传播的由来.

让我们调整一下损失函数的求导公式:

$$
\frac{\partial J(W)}{\partial w} =
\sum_{i=1}^{S_L}
\sum_{j=1}^{S_{(L-1)}}
\sum_{k=1}^{S_{(L-2)}}
(
(a_i^{(L)} - y_i)
(
w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
)
)(
w_{jk}^{(L-2)}
\frac{\partial a_k^{(L-2)}}{\partial z_k^{(L-2)}}
)
\frac{\partial z_k^{(L-2)}}{\partial w}
)
$$

$$
=
\sum_{k=1}^{S_{(L-2)}}
\sum_{j=1}^{S_{(L-1)}}
\sum_{i=1}^{S_L}
(
(a_i^{(L)} - y_i)
(
w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
)
)(
w_{jk}^{(L-2)}
\frac{\partial a_k^{(L-2)}}{\partial z_k^{(L-2)}}
)
\frac{\partial z_k^{(L-2)}}{\partial w}
)
$$

我们以δ来表示误差:

$$
\delta_i^{(L)} = a_i^{(L)} - y_i \\
\Delta^{(L)} = A^{(L)} - Y
$$

$$
\delta_{j}^{(L-1)} 
= 
\sum_{i=1}^{S_{L}}
(a_i^{(L)} - y_i)w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
=
\sum_{i=1}^{S_{L}}
\delta_i^{(L)}w_{ij}^{(L)}
\frac{\partial a_j^{(L-1)}}{\partial z_j^{(L-1)}}
$$

$$
\delta_{k}^{(L-2)} = 
\sum_{j=1}^{S_{L-1}}
\delta_j^{(L-1)}w_{jk}^{(L-1)}
\frac{\partial a_k^{(L-2)}}{\partial z_k^{(L-2)}}
$$

由此我们可以推导出通用的误差:

$$
\delta_i^{(L)} = a_i^{(L)} - y_i \\
\delta_{i}^{(l)} = 
\sum_{j=1}^{S_{(l+1)}}
\delta_j^{(l+1)}w_{ji}^{(l+1)}
\frac{\partial a_i^{(l)}}{\partial z_i^{(l)}}
$$

{% hint style="danger" %}
对于误差而言, 需要特别注意权重的下标. 与正向传播相反, 我们是对下一层的单元的每个第i个权重进行循环累加. 并且输入层是没有误差的.
{% endhint %}

当我们通过通用的误差公式从输出层反向计算出每一个单元的误差时, 就能够通过后一层的误差计算出当前单元的误差, 这能有效避免重复计算. 

## 梯度下降

当我们通过反向传播计算出了每个节点的δ之后, 就可以通过梯度下降的公式对权重进行修正了, 假设我们需要对第l层的第i个节点的第w个权重进行梯度下降:

$$
w_{ij}^{(l)} := 
w_{ij}^{(l)} - \alpha
\frac{\partial J(\vec{W})}{\partial w_{ij}^{(l)}}
$$

$$
\frac{\partial J(\vec{W})}{\partial w_{ij}^{(l)}}
=
\delta_j^l
\frac{\partial z_i^{(l)}}{\partial w_{ij}^{(l)}}
$$

$$
\frac{\partial z_i^{(l)}}{\partial w_{ij}^{(l)}}
=
a_{j}^{(l-1)}
$$

$$
w_{ij}^{(l)} := 
w_{ij}^{(l)} - 
\alpha a_j^{(l-1)} \delta_i^{(l)}
$$

