---
layout: post
title:  Pytorch 张量乘法
date:   2024-02-04
Author: CBD
tags:   [Pytorch]
---

# Docs

- https://pytorch.org/docs/stable/generated/torch.mm.html
- https://pytorch.org/docs/stable/generated/torch.matmul.html
- https://pytorch.org/docs/stable/generated/torch.mul.html


mm/matmul/mul 是在 Pytorch 的 tensor 的概念下的计算.

`torch.tensor([1,2,3])` 的形状是 `torch.Size([3])`,
表示一个一维张量, 也表示一个向量, 在表示向量时不区分行向量和列向量.




```python
print(torch.tensor([1,2,3]).size()) # 一维张量
print(torch.tensor([[1,2,3]]).size()) # 二维张量
```




    torch.Size([3])
    torch.Size([1, 3])




```python
import torch
```

# 二维张量乘法 mm

nxm * mxp = nxp



```python
a = torch.randn(2, 3)
b = torch.randn(3, 4)
c = torch.mm(a,b)
print(c.size())
print(c)
```

    torch.Size([2, 4])
    tensor([[ 0.8508,  0.6344, -0.2687, -0.2596],
            [ 2.0103,  1.3551, -0.4776, -0.3899]])


# 高维张量乘法 matmul

- 如果两个一维张量相乘, 得到 Dot Product, 即 `torch.dot`;
- 如果两个二维张量相乘, 得到 matrix-matrix Product, 即 `torch.mm`;
- 一维张量乘以二维张量, 先将一维张量 prepended 提升为二维张量然后进行 matrix-matrix product, 再将结果降维为一维张量;
- 二维张量乘以一维张量, 先将一维张量 appended  提升为二维张量然后进行 matrix-matrix product, 再将结果降维为一维张量, 即 matrix-vector product, 即 `torch.mv`;
- **高维张量乘法**: `ax(nxm) * ax(mxp) = ax(nxp)`, 批处理的二维张量相乘.




```python
a = torch.randn(5, 2, 3)
b = torch.randn(5, 3, 4)
c = torch.matmul(a,b)
print(c.size())
```

    torch.Size([5, 2, 4])


# 哈达玛乘积 mul (*乘法，广播)

axnxm * axnxp = axnxp

axnxp *   nxp = axnxp

axnxp * 1xnxp = axnxp

axnxp * axnx1 = axnxp

axnxp * 1x1x1 = axnxp


也支持张量跟一个标量相乘.


```python
torch.mul(torch.randn(2,3,4), torch.randn(2,3,4)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(3,4)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(1,3,4)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(2,3,1)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(1,1,1)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(1,1)).size()
```




    torch.Size([2, 3, 4])




```python
torch.mul(torch.randn(2,3,4), torch.randn(1)).size()
```




    torch.Size([2, 3, 4])




```python
# RuntimeError: The size of tensor a (3) must match the size of tensor b (4) at non-singleton dimension 1
# torch.mul(torch.randn(2,3), torch.randn(3,4)).size()
```
