---
title: C语言程序设计CAP-第3章编程题-时间换算
tags: []
id: '494'
categories:
  - - C
date: 2019-07-01 15:20:44
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下 

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701134832.png)

**简单解析一下**

第一眼印象是：题目真**长** 。其实题目没有那么复杂，它的意思是输入输出整数表示时间，而且输出是最简的，如803输出为3。要**注意**的是703应该输出的是2303，这也是本题所测验的地方（即提示：要注意跨日的计算），只要用个条件判断相减的数是否小于0，然后再做其他计算即可。 提取个位与十位数这两位数可以%100，提取千分位和百分位或仅提取百分位均可以/100。 UTC与BJT相差为8，可以很容易的写出代码。 一天有24小时，且在题目中明确了有效的输入范围是0到2359，所以很明确输入输出为24小时制，我们可以知道7-8+24即为23，所以算法可以很容易的写出来。那如果没有注意跨日的计算，就会出现以下情况，大于等于8小时的结果计算正常，而小于的则异常（以我这段代码来说）![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701150903.png)

## 正确做法

加一条if语句来加以约束

```c
if (hour < 0)
{
     hour = hour + 24;
}
```

如图 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701151546.png) 可见问题已经解决，代码可为

```c
#include <stdio.h>

int main(int argc, char const \*argv\[\])
{
    int BJT, UTC, minute, hour;
    scanf("%d", &BJT);
    minute = BJT % 100;
    hour = BJT / 100;
    hour = hour - 8;
    if (hour < 0)
    {
        hour = hour + 24;
    }
    UTC = hour \* 100 + minute;
    printf("%d", UTC);
    return 0;
}
```
**结果**

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/TIM截图20190701151855.png)
