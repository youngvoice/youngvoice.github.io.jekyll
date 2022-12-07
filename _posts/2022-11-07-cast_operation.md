---
title: complement encode
description: information
categories:
 - complement encode
 - cast operation
tags: [note]
---


1. 补码系统
2. 长度扩展之后的补位（特别的有符号数和无符号数的补位方式不一样）


几次尝试：

```c
int main(void)
{
    char a = 0;
    for (int i = 0; i < 512; i++) {
        a++;
        printf("%d ", a);
    }

    printf("\n");

    return 0;
}

```
上面的例子，可以看出来对于范围在 [-128, 127] 的变量 a 而言，当正的最大再加上1之后，便开始从回到负最大开始循环


```c
int main(void)
{
    char a = 0;
    for (int i = 0; i < 512; i++) {
        a++;
        //printf("%d ", a);
        printf("%x ", a);
    }

    printf("\n");

    return 0;
}

```

# 为什么出现不想要的结果??
上面的例子，在正最大再加 1 之后，出现了一个非常大的值，与想要的结果（想要看到打出无符号数的结果，就像下面的例子一样的结果）不一样，猜想前面的多个 1 应该是符号位的 1

```c
int main(void)
{
    unsigned char a = 0;
    for (int i = 0; i < 512; i++) {
        a++;
        //printf("%d ", a);
        printf("%x ", a);
    }

    printf("\n");

    return 0;
}

```

通过下面的程序来验证
```c
int main(void)
{
    unsigned char a = 0;
    unsigned int a1;
    char b = 0;
    unsigned int b1;
    int b2;
    for (int i = 0; i < 512; i++) {
        a++;
        b++;

        b1 = (unsigned int)b;
        b2 = (int)b;

        printf("%x ", b1);
        printf("%x ", b2);

    }

    printf("\n");

    return 0;
}


```


通过上面的程序可以看出来，在将 a 转换的时候，扩展位补上的是符号位，也就是按照有符号数的方法进行的



# 结论
可以看出来，计算机使用的是补码系统

# 待使用汇编确认?????



# Representing and manipulating information (ref CSAPP)
计算机确实更侧重于信息的处理！！！
数据结构也用来表示各种信息


# 2 10 16进制相互转换（无符号数）
# boolean ring



# integer representations
# conversions between signed and unsigned
# expanding the bit representation of a number
```c
short sx = -12345;
unsigned uy = sx;
```
the program first changes the size and then the type

这就是，我上面问题的答案！！！



