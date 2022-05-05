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
0    10000010    01011000000000000000000
sign exponent    fraction

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
