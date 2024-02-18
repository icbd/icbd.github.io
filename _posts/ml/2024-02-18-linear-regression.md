---
layout: post
title:  Linear Regression
date:   2024-02-18
Author: CBD
tags: [Pytorch]
---

以下是最基本的线性回归模型的实现, 每个关键步骤都是对理论的偷工减料.

- 损失函数, 理论上需要求点到线的距离, 为了减少计算量, 只计算 y 轴上的距离;
- 梯度下降, 理论上需要求出全局梯度下降最快的位置, 为了减少计算量, 只在小批量的范围内求近似解;

```python
import torch


def synthetic_data(w, b, num_examples):
    """人造数据
    y = Xw + b + noise

    X 是一个 (num_examples, 2) 的矩阵;
    w 是一个 (2,) 的向量;
    b 是一个标量的截距;
    noise 代表测量误差;v 吧

    y 返回 (num_examples, 1) 的矩阵
    """

    X = torch.normal(0, 1, (num_examples, len(w)))
    w = w.reshape((-1, 1))  # 转为 (2, 1) 的矩阵
    y = torch.mm(X, w) + b  # 得到 (num_examples,1) 的矩阵
    noise = torch.normal(0, 0.01, y.shape)
    y += noise
    return X, y


true_w = torch.tensor([2.0, -3.4])
true_b = 4.2
features, labels = synthetic_data(true_w, true_b, 1000)


def model(X, w, b):
    """model of linear regression"""
    return torch.mm(X, w) + b


def loss(y_hat, y):
    """
    y_hat 为估计值, y 为测量值
    返回结果的 shape 跟 y_hat 一致
    """
    y = y.reshape(y_hat.shape)
    return (y_hat - y) ** 2 / 2


def SGD(params, lr, batch_size):
    """
    小批量随机梯度下降 Mini-batch Stochastic Gradient Descent
    params: [w, b]
    """
    with torch.no_grad():
        for param in params:
            param -= lr * param.grad / batch_size
            param.grad.zero_()  # 主动清除梯度


# 求解
w = torch.normal(0, 0.01, size=(2, 1), requires_grad=True)
b = torch.zeros(1, requires_grad=True)

# 训练
lr = 0.03
epochs = 3
batch_size = 10
features_size = len(features)


def to_list(tensor):
    """trans tensor to python list"""
    return [element for row in tensor.tolist() for element in row]


for epoch in range(epochs):
    for i in range(0, features_size, batch_size):
        idx_limit = min(i + batch_size, features_size)
        X = features[i:idx_limit]
        y = labels[i:idx_limit]

        y_hat = model(X, w, b)
        l = loss(y_hat, y)  # (10, 1)
        s = l.sum()  # 标量
        s.backward()  # 求导. w.grad/b.grad 即为在该标量处的导数的值

        SGD([w, b], lr, batch_size)

    with torch.no_grad():
        """观察效果"""
        full_y_hat = model(features, w, b)
        full_loss = loss(full_y_hat, labels)
        print(f"epoch: {epoch + 1}, \t loss: {float(full_loss.mean()):f}")
        print(f"w: {to_list(w)}\nb: {b.item()}")
        print("-----")

print(f"\ntrue_w: {true_w.tolist()} \t true_b: {true_b}")

"""
epoch: 1, 	 loss: 0.036817
w: [1.909234881401062, -3.2400403022766113]
b: 4.000451564788818
-----
epoch: 2, 	 loss: 0.000128
w: [1.9957903623580933, -3.3924434185028076]
b: 4.1909003257751465
-----
epoch: 3, 	 loss: 0.000047
w: [1.999753475189209, -3.3996286392211914]
b: 4.199950218200684
-----

true_w: [2.0, -3.4000000953674316] 	 true_b: 4.2
"""

```