---
layout: post
title:  C 在内存里如何保存 float
Author: CBD
tags:   [C]
---

```cpp
#include <stdio.h>
#include <stdlib.h>

#if 0
Reference:
https://zh.wikipedia.org/zh/IEEE_754
https://www.log2base2.com/storage/how-float-values-are-stored-in-memory.html

Output:
41 2C 00 00

High                                 Low
0       10000010        01011000000000000000000
sign(1) exponent(8)     fraction(23)

float 的指数位和精度都是基于二进制.

指数位共 8 位. 指数有正负, 取值范围是: -127 ~ 128, 对应的 float 的数值范围也就是: 2^-127 ~ 2^128 .
    为什么不是 -128 ~ 127 呢? 这里的存储方式使用的是偏移法. -127 对应 0, -1 对应 126, 0 对应 127, 1 对应 128, 128 对应 256 .
    特别注意, 我们并不关心 2^-127 的值, 它是一个特别小的接近 0 的正数, 影响取值范围的是上限 2^128=3.4028237e+38 .
    所以 float 的取值范围就是: -3.4e38 ~ 3.4e38 .

精度位共 23 位. 2^23=8388608 一共 7 位有效数字, 去掉一个四舍五入, 最终一定能保证的精度是 6 位有效数字.

10 进制:    10.75
2  进制:    1010.11
float表示:  1.01011 * 2 ^ 3
           sigh: 正数, 0
           exponent: 八位无符号需要添加负数部分的偏移 2^n-1 = 127, 3 + 127 = 130, ob10000010
           fraction: 所有 float 都归一化表示(1.xxx), 只需取小数点后的部分, 01011
#endif

void print_binary(unsigned int p);

int main(void) {
    float n = 10.75f;
    unsigned char *p = (unsigned char *)&n;

    for (int i = (sizeof n) -1 ; i >= 0  ; i--){
        printf("%02X\t\t", p[i]);
    }
    printf("\n");
    for (int i = (sizeof n) -1 ; i >= 0  ; i--){
        print_binary(p[i]);
        printf("\t");
    }
    printf("\n");

    exit(0);
}

void print_binary(unsigned int p)
{
    int byte_width = 8;
    char memory[byte_width]; // [Low,,,High]
    for (int i = 0; i < byte_width; i++) {
        memory[i] = (p & 1) ? '1' : '0';
        p >>= 1;
    }

    // print from high-end
    for (int i = byte_width -1; i >=0; i--) {
        printf("%c", memory[i]);
    }
}

```
