---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 神经网络的表示

神经网络的多层次架构可以通过向量的形式表达, 对于第j层的隐藏层而言, 若其中单元个数为i, 则权重矩阵W\(j\)为:

$$
\vec{W}^{(j)} = 
\left [
\begin{matrix}
(\vec{W}_1^{(1)})^T\\
(\vec{W}_2^{(1)})^T\\
...\\
(\vec{W}_i^{(1)})^T
\end{matrix}
\right ]\\
$$

$$
\vec{W}^{(j)} = 
\left [
\begin{matrix}
w_{10}^{(1)} & w_{11}^{(1)} & ... & w_{1n}^{(1)}\\
w_{20}^{(1)} & w_{21}^{(1)} & ... & w_{2n}^{(1)}\\
...\\
w_{i0}^{(1)} & w_{i1}^{(1)} & ... & w_{in}^{(1)}\\
\end{matrix}
\right ]\\
$$

由隐藏层激励值的通用公式可以得知:

$$
a_i^{(j)} = s(\vec{W}_{i}^{(j)} \cdot \vec{A}^{(j-1)})
$$

$$
\vec{A}^{(j)} = s(\vec{W}^{(j)} \cdot \vec{A}^{(j-1)} )
$$

$$
\vec{A}^{(j)} =
s(
\left [
\begin{matrix}
w_{10}^{(1)} & w_{11}^{(1)} & ... & w_{1n}^{(1)}\\
w_{20}^{(1)} & w_{21}^{(1)} & ... & w_{2n}^{(1)}\\
...\\
w_{i0}^{(1)} & w_{i1}^{(1)} & ... & w_{in}^{(1)}\\
\end{matrix}
\right ]
\left [
\begin{matrix}
a_0^{(j - 1)}\\
a_1^{(j -1)}\\
...\\
a_n^{(j -1)}
\end{matrix}
\right ]
)
$$

则我们可以将各个层次嵌套的目标函数h表达为:

$$
h(\vec{X}) = s(
\vec{W}^{(j+1)} \cdot
s(\vec{W}^{(j)} \cdot
s(\vec{W}^{(j -1)} \cdot
... \cdot
s(\vec{W}^{(1)} \cdot 
\vec{X}
)))
$$

需要注意的是, W\(j+1\)是输出层的权重向量.
