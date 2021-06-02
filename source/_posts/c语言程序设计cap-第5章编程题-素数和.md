---
title: C语言程序设计CAP-第5章编程题-素数和
tags: []
id: '517'
categories:
  - - C
date: 2019-07-01 17:41:23
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701163507.png)

## ~简单分析一下~胡乱分析一下

看到这题目我内心是拒绝的，因为不知道如何入手，题目中的n，m含义特殊，之间的素数也难以判断，我认为得整个素数表出来（大佬勿喷），可确实难以下手。好在我是学到第七章再回来做题的，意识到数组方面似乎可以利用，于是乎整个容量为200的数组来储存0-200个素数（建议看我这渣代码前先去看看第七章，我实在想不出啥办法了）但是问题是如何得到每一个素数呢？之前老师演示过如何判断输入的数是否为素数，相关代码如下

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int origin, i, a = 1;
    scanf("%d", &origin);
    for (i = 2; i < origin; i++)
    {
        if (origin % i == 0)
        {
            a = 0;
            break;
        }
    }

    if (a == 0)
    {
        printf("不是素数");
    }
    else
    {
        printf("是素数");
    }

    return 0;
}
```

输入数为origin，从2开始除，如在2到origin中，有数能够被orgin整除，则origin不为素数。那我们是否可以利用这个原理，一个个列出素数知道得到我们想要的数呢？ 答案当然是可行的。我们可以设置一个变量count，当origin为素数时加上1，如当origin的初始化为2时，count为1，在循环中，当origin加到3，count加1为2，当origin加到4，count不为所动，当origin加到5，count加1为3......以此类推，直到count=n或m时。在这过程中，每个count对应的数都应该被记录。相关代码可以这样写

```c
int count = 0;
int number[200];
int a = 1, origin = 2;
while (n != count)
{
    for (int i = 2; i < origin; i++)
    {
        if (origin % i == 0)
        {
            a = 0;
            break;
        }
    }
    if (a != 0)
    {
        count++;
        number[count] = origin;
    }
    origin++;
    a = 1;
}
```

  值得注意的是，在这段代码中初始化a为1，这样做是为了避免程序直接从内存中抽出个0来就不好玩了。循环中的a赋值为一也十分重要，否则判断一次偶数以后，程序就死循环了，也就是n大于等于3时。对于m也相同的办法，可以copy一下（“代码复制”是程序不良的体现![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)，可以整成函数，可是好像我不会改这个） ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/IMG20190701173034.jpg) 两段代码这样的

```c
while (n != count)
{
    for (int i = 2; i < origin; i++)
    {
        if (origin % i == 0)
        {
            a = 0;
            break;
        }
    }
    if (a != 0)
    {
        count++;
        number[count] = origin;
    }
    origin++;
    a = 1;
}

while (m != count)
{
    for (int i = 2; i < origin; i++)
    {
        if (origin % i == 0)
        {
            a = 0;
            break;
        }
    }
    if (a != 0)
    {
        count++;
        number[count] = origin;
    }
    origin++;
    a = 1;
}
```

相加之间素数就简单多了，如下，从number[n]加到number[m]即可

```c
for (int i = n; i <= m; i++)
{
    sum = sum + number[i];
}
```

**/*正确做法*/能够跑起来的做法**

整体代码

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int n, m;
    int sum = 0;
    scanf("%d %d", &n, &m);
    int count = 0;
    int number[200];
    int a = 1, origin = 2;
    while (n != count)
    {
        for (int i = 2; i < origin; i++)
        {
            if (origin % i == 0)
            {
                a = 0;
                break;
            }
        }
        if (a != 0)
        {
            count++;
            number[count] = origin;
        }
        origin++;
        a = 1;
    }

    while (m != count)
    {
        for (int i = 2; i < origin; i++)
        {
            if (origin % i == 0)
            {
                a = 0;
                break;
            }
        }
        if (a != 0)
        {
            count++;
            number[count] = origin;
        }
        origin++;
        a = 1;
    }

    for (int i = n; i <= m; i++)
    {
        sum = sum + number[i];
    }

    printf("%d", sum);
    return 0;
}
```

**结果**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701173942.png)

_本以为内存会很大的，表现还行？_
