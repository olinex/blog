---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 神经网络的表示

神经网络的多层次架构可以通过向量的形式表达, 对于第j层的隐藏层而言, 若其中单元个数为i, 则权重矩阵W\(j\)为:

$$
W^{(j)} = 
\left [
\begin{matrix}
(W_1^{(j)})^T\\
(W_2^{(j)})^T\\
...\\
(W_{S_{j}}^{(j)})^T
\end{matrix}
\right ]\\
$$

$$
W_{S_{j} \times (S_{(j - 1)} + 1)}^{(j)} = 
\left [
\begin{matrix}
w_{10}^{(j)} & w_{11}^{(j)} & ... & 
w_{1S_{(j - 1)}}^{(j)}\\
w_{20}^{(j)} & w_{21}^{(j)} & ... & 
w_{2S_{(j - 1)}}^{(j)}\\
...\\
w_{S_{j}0}^{(j)} & w_{S_{j}1}^{(j)} & ... & 
w_{S_{j}S_{(j - 1)}}^{(j)}\\
\end{matrix}
\right ]\\
$$

由隐藏层激励值的通用公式可以得知:

$$
a_i^{(j)} = g(W_{i}^{(j)} \cdot A^{(j-1)})
$$

$$
A^{(j)} = g(W^{(j)} \cdot A^{(j-1)})
$$

$$
A^{(j)} =
g(
\left [
\begin{matrix}
w_{10}^{(j)} & w_{11}^{(j)} & ... & w_{1S_{(j-1)}}^{(j)}\\
w_{20}^{(j)} & w_{21}^{(j)} & ... & w_{2S_{(j-1)}}^{(j)}\\
...\\
w_{S_{j}0}^{(j)} & w_{S_{j}1}^{(j)} & ... & w_{S_{j}S_{(j-1)}}^{(j)}\\
\end{matrix}
\right ]
\left [
\begin{matrix}
a_0^{(j - 1)}\\
a_1^{(j -1)}\\
...\\
a_{S_{(j-1)}}^{(j -1)}
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



