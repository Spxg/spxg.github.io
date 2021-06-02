---
title: C语言程序设计CAP-第6章编程题-分解质因数
tags: []
id: '525'
categories:
  - - C
date: 2019-07-02 15:08:45
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下 

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190702140550.png)

**简单解析一下**

题目说的很明了了，废话不多说，先分析一下算法：如12除以第一个素数2得6，6除以第一个素数2得3，3为素数，即12=2x2x3；又如15能被第二个素数3整除为5，5为素数，所以15=3x5。我们不难发现只要一个数能被从2开始的某个素数整除，这个素数为分解质因数里的一个值，整除后的值循环反复，直到为素数时停止。 这部分代码可以这样写

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int i, n;
    scanf("%d", &n);
    for (i = 2; i <= n;)
    {
        if (n % i == 0)
        {
            printf("%d", i);
            n = n / i;
        }
        else
        {
            i++; //从2开始，如果2不行就加上去
        }
    }
    return 0;
}
```

可以发现的是，质因数全部被算出来了![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190702142632.png) 可题目要求的是输出格式为n=axbxcxd或n=n，我们可以在%d前面或后面加个x,但按照这个程序来说，输出结果会变成x2x2x3或2x2x3x，很显然是不正确的，所以我们可以将第一个质数单独输出，或将最后一个单独输出。这里选择后者，经过调试后，我们不难发现，最后一个质数的结果就是最后的i的值![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190702143501.png) 经过调整，程序可以改成这样

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int i, n;
    scanf("%d", &n);
    printf("%d=", n);
    for (i = 2; i < n;) //为了避免重复输出，循环条件应为i<n
    {
        if (n % i == 0)
        {
            printf("%dx", i);
            n = n / i;
        }
        else
        {
            i++; //从2开始，如果2不行就加上去
        }
    }
    printf("%d", i);
    return 0;
}
```

做到这应该发现了个问题，这个程序和第六章学习的内容没有半毛钱关系，所以我们可以强行做个函数![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)

## ~正确~做法

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190702144927.png) 代码可为

```c
#include <stdio.h>
void number(int n)
{
    int i;
    printf("%d=", n);
    for (i = 2; i < n;)
    {
        if (n % i == 0)
        {
            printf("%dx", i);
            n = n / i;
        }
        else
        {
            i++;
        }
    }
    printf("%d", i);
}

int main(int argc, char const *argv[])
{
    int n;
    scanf("%d", &n);
    number(n);
    return 0;
}
```

**思考**

发现程序更啰嗦了，或许我们可以整个头文件，眼不见为净![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)，如图，建立个.h的文件![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190702145345.png) 然后直接引用这个头文件

```c
#include "decomposition-factor.h"

int main(int argc, char const *argv[])
{
    int n;
    scanf("%d", &n);
    number(n);
    return 0;
}
```

需要注意的是，对于引用自己的头文件，include后面应该用""而不是<>

**PS**

这个程序我写了两次，第一次真是又臭又长，不介意的话可以吐口痰再走

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int n;
    scanf("%d", &n);
    int x, z = 1;
    printf("%d=", n);
    for (int i = 2; i <= n;)
    {
        if (n % i == 0)
        {
            x = n;
            n = n / i;
            for (int m = 2; m < x; m++)
            {
                if (x % m == 0)
                {
                    z = 0;
                    break;
                }
            }
            if (z == 0)
            {
                printf("%dx", i);
                z = 1;
            }
            else
            {
                printf("%d", x);
            }
        }
        else
        {
            i++;
        }
    }
    return 0;
}
```

**写的什么玩意儿！**
