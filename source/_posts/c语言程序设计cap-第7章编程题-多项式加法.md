---
title: C语言程序设计CAP-第7章编程题-多项式加法
tags: []
id: '548'
categories:
  - - C
date: 2019-08-02 12:52:54
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190802095458.png)

## 简单分析一下

题目要求输入的数合并同类项后按照幂次大小排序，这里值得注意的是，如系数或幂次为1都应该省略；当输入为0 0，0 0时，输出0；可以有负数输入；了解了这些细节以后，就该考虑怎么做了。 我首先想到的是用二维数组存数据，结果写了600多行依然没解决（没考虑到可以有负数输入），只能另想办法。最后我发现可以利用数组的[]来储存幂次，值来存系数，这样工作量大大减少，故开始部分可以这样写

```c
int number[101] = {0}; //题目说处理最大的幂为100，加上0即是最大量为101；初始化为0
int max = 0, cnt = 0;

for (int i = 0; i < 2; i++) //输入两组数据
{
    int sum, j;
    do
    {
        scanf("%d %d", &j, &sum);
        if (j > cnt)
            cnt = j; //记录输入的最大幂次
        number[j] += sum; //相同幂次的值相加
    } while (j > 0); //输入0停止
}

for (int i = cnt; i >= 0; i--)
{
    if (number[i] != 0)
    {
        max = i; //记录有效的最大幂次，因为'某个幂次的系数为0，就不出现在输入数据中'不代表我不能输入如 6 2， 6 -2;
        break;
    }
}
```

值得注意的是，我们创建了大小为101的数组，每个量都被初始化为0，即系数为零，所以我们应该跳过输出这些量；还要判断当输入的值都是 0 0的情况；判断为系数为1的情况；判断幂次为1的情况；判断幂次为0的情况；判断系数与幂次都为1的情况。种种情况考虑到的话，可以写出如下代码

```c
for (int i = max; i >= 0; i--)
{
    if (number[i] == 0)
    {
        if (max == 0)
            printf("0");
        else
            continue;
    }
    else if (i == 0)
    {
        if (number[i] != 0)
            if (i == max  number[i] < 0)
                printf("%d", number[i]);
            else
                printf("+%d", number[i]);
    }
    else if (i == 1)
    {
        if (number[i] == 1)
            if (i == max)
                printf("x");
            else
                printf("+x");
        else if (i == max  number[i] < 0)
            if (number[i] == -1)
                printf("-x");
            else
                printf("%dx", number[i]);
        else
            printf("+%dx", number[i]);
    }
    else
    {
        if (number[i] == 1)
            if (i == max)
                printf("x%d", i);
            else
                printf("+x%d", i);
        else if (i == max  number[i] < 0)
            if (number[i] == -1)
                printf("-x%d", i);
            else
                printf("%dx%d", number[i], i);
        else
            printf("+%dx%d", number[i], i);
    }
}
```
**整体代码可为**

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int number[101] = {0};
    int max = 0, cnt = 0;

    for (int i = 0; i < 2; i++)
    {
        int sum, j;
        do
        {
            scanf("%d %d", &j, &sum);
            if (j > cnt)
                cnt = j;
            number[j] += sum;
        } while (j > 0);
    }

    for (int i = cnt; i >= 0; i--)
    {
        if (number[i] != 0)
        {
            max = i;
            break;
        }
    }

    for (int i = max; i >= 0; i--)
    {
        if (number[i] == 0)
        {
            if (max == 0)
                printf("0");
            else
                continue;
        }
        else if (i == 0)
        {
            if (number[i] != 0)
                if (i == max  number[i] < 0)
                    printf("%d", number[i]);
                else
                    printf("+%d", number[i]);
        }
        else if (i == 1)
        {
            if (number[i] == 1)
                if (i == max)
                    printf("x");
                else
                    printf("+x");
            else if (i == max  number[i] < 0)
                if (number[i] == -1)
                    printf("-x");
                else
                    printf("%dx", number[i]);
            else
                printf("+%dx", number[i]);
        }
        else
        {
            if (number[i] == 1)
                if (i == max)
                    printf("x%d", i);
                else
                    printf("+x%d", i);
            else if (i == max  number[i] < 0)
                if (number[i] == -1)
                    printf("-x%d", i);
                else
                    printf("%dx%d", number[i], i);
            else
                printf("+%dx%d", number[i], i);
        }
    }

    return 0;
}
```

**结果**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190802125031.png) ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190802125131.png)
