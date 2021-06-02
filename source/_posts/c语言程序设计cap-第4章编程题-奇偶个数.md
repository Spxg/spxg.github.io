---
title: C语言程序设计CAP-第4章编程题-奇偶个数
tags: []
id: '507'
categories:
  - - C
date: 2019-07-01 16:14:14
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下

  ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/微信截图_20190701153959.png)

## 简单分析一下

本题的意思是输入一些整数（-1表示结束，并不计入），读出奇数偶数分别为多少。我们知道分别奇数与偶数有个很简单的判断方法：是否能被2整除。输入是连续的，输入-1停止，而且scanf要先执行一次，说明scanf应该在循环以内（可以使用do-while；当然while也行，把一个scanf扔到外面），且循环条件为输入的数不等于-1。循环体的代码可以写成这样（大佬勿喷）

```c
do
{
    scanf("%d", &number);
    if (number % 2 == 0)
    {
        even++;
    }
    else
    {
        odd++;
    }

} while (number != -1);
```

当初始化even与odd为0时，我们会发现一个现象，输出结果odd总是比正确的值大1，细心一点可以发现，当输入-1时，循环又执行了一次。所以可以初始化odd为-1（大佬勿喷）

## 正确做法

代码可为

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int number, even = 0, odd = -1;
    do
    {
        scanf("%d", &number);
        if (number % 2 == 0)
        {
            even++;
        }
        else
        {
            odd++;
        }

    } while (number != -1);
    printf("%d %d", odd, even);
    return 0;
}
```

**结果**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701161338.png)
