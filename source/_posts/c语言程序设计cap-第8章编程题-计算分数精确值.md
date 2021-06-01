---
title: C语言程序设计CAP-第8章编程题-计算分数精确值
tags: []
id: '564'
categories:
  - - C
date: 2019-08-19 16:54:16
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190819164000.png)  

## 简单分析一下

题目看起来很长，但是读完后发现问题不大，按照题目要求，较容易写出程序，值得注意的是，输出时要换行

**整体代码可为**

```c
#include <stdio.h>

int main(int argc, char const \*argv\[\])
{
    int a, b, c;

    scanf("%d/%d", &a, &b);
    printf("0.");
    
    for (int i = 0; i < 200; i++) //最多200位
    {
        c = a \* 10 / b;
        a = a \* 10 % b;
        printf("%d", c);
        
        if (a == 0)
        {
            printf("\\n"); //换行
            break; //余数等于0时退出循环
        }
        
        if (i == 199)
            printf("\\n");//换行
    }

    return 0;
}
```

**结果**

因为懒，博文写的太晚，现在再登陆已经暂时看不到结果，但是分数是都拿到手了![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)
