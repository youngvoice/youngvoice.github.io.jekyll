---
title: floor and ceil function
description: how to align on boundary?
categories:
 - language
 - memory
tags: [learning]
---
具体数学：Concrete Mathematics
Chapter 3

$$
\lfloor x \rfloor = \text{the greatest integer less than or equal to x}\\
\lceil x \rceil = \text{the least integer greater than or equal to x}
$$

the functions are reflections of each other about both axes:
$$
\lfloor -x \rfloor = -\lceil x \rceil; \lceil -x \rceil = -\lfloor x \rfloor
$$

function transformation
In Mathematics, a transformation of a function is a function that turns one function or graph into another, usually related function or graph.

translation transformation
rotation transformation
reflection transformation
dilation transformation

<https://www.khanacademy.org/math/algebra2>

复合函数
函数的变换和组合

---

在分配内存时，为保证内存边界的 align 性，我们经常需要进行向上、向下取整操作，它的 c 程序实现
```c
#define ROUNDDOWN(a, n)                                         \
({                                                              \
        uint32_t __a = (uint32_t) (a);                          \
        (typeof(a)) (__a - __a % (n));                          \
})

#define ROUNDUP(a, n)                                           \
({                                                              \
        uint32_t __n = (uint32_t) (n);                          \
        (typeof(a)) (ROUNDDOWN((uint32_t) (a) + __n - 1, __n)); \
})

```


求对一个整数 n 表示为二进制数需要多少位？
