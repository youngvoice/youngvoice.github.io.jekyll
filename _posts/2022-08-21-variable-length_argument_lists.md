Variadic functions in C

# How to write a variadic function in C?
应用场景

minprintf.c reference book The C programming language
```c

#include <stdarg.h>

void minprintf(char *fmt, ...)
{
    va_list ap;
    char *p;
    char *sval;
    int ival;
    double dval;

    va_start(ap, fmt);
    for (p = fmt; *p; p++) {
        if (*p != %) {
            putchar(*p);
            continue;
        }

        switch (*++p) {
            case 'd':
                ival = va_arg(ap, int);
                printf("%d", ival);
                break;
            case 'f':
                dval = va_arg(ap, double);
                printf("%f", dval);
                break;
            case 's':
                for (sval = va_arg(ap, char *); *sval; sval++)
                    putchar(*sval);
                break;
            default:
                putchar(*p);
                break;
        }

    }
    va_end(ap);
}
```


# How to implement the mechanism?
下面的解释仅供参考：

if we call function minprintf as follow:
```c
int a = 2;
double b = 6.5;
char *c = "hello";
minprintf("%d %f %s", a, b, c);
```
那么我们可以根据函数参数的传递，得到参数在内存的布局，然后就可以根据参数的类型对所在内存进行访问，并移动指向参数的指针。

As above function call situation, we have below stack layout:

```c
//begin
va_start(ap, fmt);
/*
let the ap point to the next argument
char * ap = (char *)&fmt + [sizeof(fmt) or sizeof(char *)];
*/

//process
ival = va_arg(ap, int);
/*
cast type conversion the ap to pointer pointing to int, and then take the content as the type, finally move the ap in the size of the type, to point to next argument;

ival = *((int *)ap);
ap += sizeof(int);
*/

dval = va_arg(ap, double);
/*
the same as above
dval = *((double*)ap);
ap += sizeof(double);
*/

sval = va_arg(ap, char *);
/*
the same as above
sval = *((char **)ap);
ap += sizeof(char *);
*/


//end
va_end(ap);
/*
ap = NULL;
*/

```
由于在各个 hardware architecture 下，call convention 差异比较多，所以上面的解释仅仅参考，具体处理细节还有得通过 gcc 源码来了解。

Reference 

https://stackoverflow.com/questions/12371450/how-are-variable-arguments-implemented-in-gcc
