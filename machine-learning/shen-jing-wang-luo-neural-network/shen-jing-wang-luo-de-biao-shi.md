---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 神经网络的表示

神经网络的多层次架构可以通过向量的形式表达, 对于第l层的隐藏层而言, 若其中单元个数为i, 则权重矩阵W\(l\)为:

$$
W^{(l)} = 
\left [
\begin{matrix}
(W_1^{(l)})^T\\
(W_2^{(l)})^T\\
...\\
(W_{S_{l}}^{(l)})^T
\end{matrix}
\right ]\\
$$

$$
W_{S_{l} \times (S_{(l - 1)} + 1)}^{(l)} 
= 
\left [
\begin{matrix}
w_{10}^{(l)} & w_{11}^{(l)} & ... & 
w_{1S_{(l-1)}}^{(l)}\\
w_{20}^{(l)} & w_{21}^{(l)} & ... & 
w_{2S_{(l - 1)}}^{(l)}\\
...\\
w_{S_{l}0}^{(l)} & w_{S_{l}1}^{(l)} & ... & 
w_{S_{l}S_{(l-1)}}^{(l)}\\
\end{matrix}
\right ]\\
$$

由隐藏层激励值的通用公式可以得知:

$$
a_i^{(l)} = g(W_{i}^{(l)} \cdot A^{(l-1)})
$$

$$
A^{(l)} = g(W^{(l)} \cdot A^{(l-1)})
$$

$$
A^{(l)} =
g(
\left [
\begin{matrix}
w_{10}^{(l)} & w_{11}^{(l)} & ... & w_{1S_{(l-1)}}^{(l)}\\
w_{20}^{(l)} & w_{21}^{(l)} & ... & w_{2S_{(l-1)}}^{(l)}\\
...\\
w_{S_{l}0}^{(l)} & w_{S_{l}1}^{(l)} & ... & w_{S_{l}S_{(l-1)}}^{(l)}\\
\end{matrix}
\right ]
\left [
\begin{matrix}
a_0^{(l - 1)}\\
a_1^{(l -1)}\\
...\\
a_{S_{(l-1)}}^{(l -1)}
\end{matrix}
\right ]
)
$$

则我们可以将各个层次嵌套的目标函数h表达为:

$$
h(X) = 
g(W_{K \times (S_{(L - 1)} + 1)}^{(L)} \cdot
... \cdot
g(W_{S_3 \times (S_2 + 1)}^{(3)} \cdot 
g(W_{S_2 \times (n + 1)}^{(2)} \cdot 
X_{(n + 1) \times 1}
))))_{K \times 1}
$$



