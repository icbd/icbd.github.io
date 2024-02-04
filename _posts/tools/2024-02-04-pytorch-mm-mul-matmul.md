---
layout: post
title:  Pytorch 的各种乘法
date:   2022-02-04
Author: CBD
tags:   [Pytorch]
---

# Docs

- https://pytorch.org/docs/stable/generated/torch.mm.html
- https://pytorch.org/docs/stable/generated/torch.matmul.html
- https://pytorch.org/docs/stable/generated/torch.mul.html

```python
import torch
```

# 二维矩阵乘法 mm

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


# 高维矩阵乘法 matmul

axnxm * axmxp = axnxp

（ a 的部分适用于广播， 见下面 mul 的规则）



```python
a = torch.randn(5, 2, 3)
b = torch.randn(5, 3, 4)
c = torch.matmul(a,b)
print(c.size())
```

    torch.Size([5, 2, 4])


# 点乘 mul (*乘法，广播)

axnxm * axnxp = axnxp

axnxp *   nxp = axnxp

axnxp * 1xnxp = axnxp

axnxp * axnx1 = axnxp

axnxp * 1x1x1 = axnxp


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
