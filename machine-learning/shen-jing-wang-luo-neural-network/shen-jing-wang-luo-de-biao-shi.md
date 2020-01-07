---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 神经网络的表示

神经网络的多层次架构可以通过向量的形式表达, 对于第j层的隐藏层而言, 若其中单元个数为i, 则权重矩阵W\(j\)为:

$$
\vec{W}^{(j)} = 
\left [
\begin{matrix}
(\vec{W}_1^{(j)})^T\\
(\vec{W}_2^{(j)})^T\\
...\\
(\vec{W}_{S_{j}}^{(j)})^T
\end{matrix}
\right ]\\
$$

$$
\vec{W}_{S_{j} \times (S_{(j - 1)} + 1)}^{(j)} = 
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
a_i^{(j)} = g(\vec{W}_{i}^{(j)} \cdot \vec{A}^{(j-1)})
$$

$$
\vec{A}^{(j)} = g(\vec{W}^{(j)} \cdot \vec{A}^{(j-1)} )
$$

$$
\vec{A}^{(j)} =
g(
\left [
\begin{matrix}
w_{10}^{(j)} & w_{11}^{(j)} & ... & w_{1S_{j-1}}^{(j)}\\
w_{20}^{(j)} & w_{21}^{(j)} & ... & w_{2S_{j-1}}^{(j)}\\
...\\
w_{S_{j}0}^{(j)} & w_{S_{j}1}^{(j)} & ... & w_{S_{j}S_{j - 1}}^{(j)}\\
\end{matrix}
\right ]
\left [
\begin{matrix}
a_0^{(j - 1)}\\
a_1^{(j -1)}\\
...\\
a_{S_{j - 1}}^{(j -1)}
\end{matrix}
\right ]
)
$$

则我们可以将各个层次嵌套的目标函数h表达为:

$$
h(\vec{X}) = g(
\vec{W}^{(j)} \cdot
g(\vec{W}^{(j -1)} \cdot
... \cdot
g(\vec{W}^{(1)} \cdot 
\vec{X}
)))
$$



