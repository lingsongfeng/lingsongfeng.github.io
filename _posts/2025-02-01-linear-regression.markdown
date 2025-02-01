---
layout: post
title:  "【机器学习】线性回归"
date:   2025-02-01 15:10:00 +0800
categories: machine learning
---
## Linear Regression

输入为$x$，输出为$h(x)$，其中：
$$
y = h(x) = \theta_0 + \theta_1x_1 + \theta_2x_2
$$
一般地，用向量来表示：
$$
y = h(x) 
	= \theta_0 + \theta_1x_1 + \cdots + \theta_nx_n
	= \theta^Tx
$$
loss function:
$$
J(\theta) = \frac12\sum_{i=1}^{m}{\left(h(x^{(i)})-y^{(i)}\right)^2}
$$
其中，$\theta$为模型的参数（parameter or weight）。

我们的目标是学习$\theta$，以获得最小的损失函数值：
$$
\min_\theta{J(\theta)} = \frac12\sum_{i=1}^{m}{\left(h(x^{(i)})-y^{(i)}\right)^2}
$$
若$J(\theta)$有全局最小值或最大值，那么此时下式一定成立：
$$
\nabla_\theta{J(\hat\theta)} = 0
$$
我们用矩阵来表示训练输入集：
$$
X = \begin{bmatrix}
{x^{(1)}}^T \\
{x^{(2)}}^T \\
\vdots \\
{x^{(n)}}^T \\
\end{bmatrix}
$$
用列向量来表示训练输出集：
$$
y = \begin{bmatrix}
y^{(1)} \\
y^{(2)} \\
\vdots \\
y^{(n)} \\
\end{bmatrix}
$$
由于$h(x) = \theta^Tx$，所以：
$$
X\theta - y = \begin{bmatrix}
h({x^{(1)}}) - y^{(1)} \\
h({x^{(2)}}) - y^{(2)} \\
\vdots \\
h({x^{(n)}}) - y^{(n)} \\
\end{bmatrix}
$$
此时，我们再重写损失函数：
$$
\frac12\sum_{i=1}^{m}{\left(h(x^{(i)})-y^{(i)}\right)^2} \\
= \frac12(X\theta - y)^T(X\theta - y) \\
= \frac12(\theta^TX^TX\theta - 2y^TX\theta - y^2)
$$
对其求关于$\theta$的梯度：
$$
\frac12\nabla_\theta\left(\theta^TX^TX\theta - 2y^TX\theta - y^2\right) \\
= X^TX\theta - X^Ty = 0
$$
注：
$$
\nabla_\theta(\theta^TX^TX\theta) \\
= \nabla_\theta(\theta^T(X^TX)\theta) \\
= 2X^TX\theta & \text{(}\nabla_x(x^TAx)=2Ax\text{ when } A = A^T\text{)}
$$
令上式为零，则可以获取到极大值、极小值的点。
$$
X^TX\theta - X^Ty = 0 \\
X^TX\theta = X^Ty \\
\theta = (X^TX)^{-1}X^Ty
$$

### Gradient descent(梯度下降)

$$
\theta_{j} \coloneqq \theta_j - \alpha\dfrac{\part}{\part\theta_j}J(\theta)
$$

其中，$\alpha$是学习率。

当只有一个 training sample $(x^{(i)}, y^{(i)})$ 时，
$$
\dfrac{\part}{\part\theta_j}J(\theta) = (h_\theta(x)-y)x_j
$$
所以对于这一个 training sample，我们有：
$$
\theta_{j} \coloneqq \theta_j + \alpha(y^{(i)}-h(x^{(i)}))x_j^{(i)}
$$
对于整个训练集，我们可以使用batch gradient descent算法：
$$
\theta_j \coloneqq \theta_j + \alpha\sum_{i=1}^m{(y^{(i)} - h_\theta(x^{(i)}))x_j^{(i)}}
$$
**重复上式描述的过程，直到收敛。**

batch gradient descent算法一次迭代会通过求和的方式，将全部sample的梯度都考虑在内。

#### Stochastic Gradient Descent (SGD)

随机或顺序选取training sample中的一个sample，来执行该迭代：
$$
\theta_{j} \coloneqq \theta_j + \alpha(y^{(i)}-h(x^{(i)}))x_j^{(i)}
$$
当 $\lVert\nabla_\theta J(\theta)\rVert$足够小时，结束迭代过程。

#### Mini-batch Sthochastic Gradient Descent

迭代过程与 batch gradient descent 相似，但不选取全部的sample，而是其中一部分
$$
\theta_j \coloneqq \theta_j + \alpha\sum_{\text{for }i\text{ in } \mathcal{P} }{(y^{(i)} - h_\theta(x^{(i)}))x_j^{(i)}}
$$

- 一般一个batch的大小是人为设定的（hyper parameter）$N$，所以总共有$m/N$个batch，因此需要$m/N$ 次迭代才能遍历一遍训练集。遍历一遍训练集被称为1个**epoch**。

- 在新epoch开始前，可以reshuffle训练集。

- 一个batch的选取有讲究，比如在识别数字场景中，如果一个mini-batch是000000，那就不好。如果是0123456这样就会更好一些。需要最大化mini-batch内label种类的不同。

速度上：比batch gradient descent快

稳定性上：比stochastic gradient descent高

