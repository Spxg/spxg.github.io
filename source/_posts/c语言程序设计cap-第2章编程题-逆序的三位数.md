---
title: C语言程序设计CAP-第2章编程题-逆序的三位数
tags: []
id: '481'
categories:
  - - C
date: 2019-07-01 13:38:18
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701125528.png)

## 简单分析一下

根据提示可以简单完成大部分工作 需要注意的是700的逆序不是007，这就说明程序不能写成这样：

```c
#include <stdio.h>

int main(int argc, char const \*argv\[\])
{
    int number;
    scanf("%d", &number);
    int a = number / 100;
    int b = number / 10 % 10;
    int c = number % 10;
    printf("%d%d%d", c, b, a); //这里简单粗暴的调换位数是不可取的，700会被逆序成007
    return 0;
}
```

## 正确做法

用个数学式子解决: `c \* 100 + b \* 10 + a`

代码可为

```c
#include <stdio.h>

int main(int argc, char const \*argv\[\])
{
    int number;
    scanf("%d", &number);
    int a = number / 100;
    int b = number / 10 % 10;
    int c = number % 10;
    printf("%d", c \* 100 + b \* 10 + a);
    return 0;
}
```

**结果**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701132954.png)
