---
title: C语言程序设计CAP-第10章编程题-GPS数据处理
tags: []
id: '567'
categories:
  - - C
date: 2019-08-19 17:44:18
---

_即将上大学了，准备上一些先修课，[翁恺老师](https://www.icourse163.org/u/wengkai)的课感觉非常不错，我是一个初学者，有啥错误大佬们别打我。_

依照学术诚信条款，我保证此作业是本人独立完成的，未完成该项作业的同学请完成后再来。

此编程题的题目如下

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190819165743.png) ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/08/QQ截图20190819165726.png)  

## 简单分析一下

题目依旧很长，但是这次每一句都得仔细看，我的直觉告诉我这题不简单，这题我接触一个新的东西：异或，也让我学到了进制的转化，更让我学到了看清题目的重要性，题目重点如下

1.  每个字段的大小（长度）不一 （说明当找字符时不能只是数数）
2.  “*”为校验和识别符，其后面的两位数为校验和，代表了“$”和“*”之间所有字符（不包括这两个字符）的异或值的十六进制值。上面这条例句的校验和是十六进制的50，也就是十进制的80
3.  十六进制值中是会出现A-F的大写字母的
4.  你的程序要读入一系列GPS输出，其中包含$GPRMC，也包含其他语句。在数据的最后，有一行单独的 END
5.  从中找出$GPRMC语句，计算校验和，找出其中校验正确，并且字段2表示已定位的语句，从中计算出时间，换算成北京时间。一次数据中会包含多条$GPRMC语句，以最后一条语句得到的北京时间作为结果输出
6.  其中，hh是两位数的小时，不足两位时前面补0；mm是两位数的分钟，不足两位时前面补0；ss是两位数的秒，不足两位时前面补0

读完题目，分析重点完后，发现此题最好需要用到字符串函数（为什么说最好呢？因为讨论区内有位厉害的同学没用到），所以需要有string.h头文件，至于字符串大小就整个1000吧，大点保险些。作为压轴题，我们最好写的优秀些，比如用上函数的知识啥的![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)，初步可以写出如下代码

```c
#include <stdio.h>
#include <string.h>
#define MAX 1000

void gettime(char *gps, int *hour, int *minutes, int *second); //获取时间，且因为变量不是全局的，所以传指针

int main(int argc, char const *argv[])
{
    int hour, minutes, second;
    char gps[MAX] = {'0'}; //日常初始化
    char end[] = "END"; //END的字符串
    
    do
    {
        scanf("%s", gps); //因为字符串是一连串的，中间没有空格，所以使用scanf即可，用gets也行
        gettime(gps, &hour, &minutes, &second);
    } while (strcmp(gps, end) != 0); //判断是否为END，是，就停止

    printf("%02d:%02d:%02d", hour, minutes, second); //最后的正确语句输出

    return 0;
}
```

然后我们就该考虑如何获取正确时间了，看完重点后，就可以知道第一步应该是判断是否为$GPRMC语句，之后再判断是否为有效语句。按照题目，我们得先算异或值，然后把它转化成16进制，与校验值相符后，再判断是否已定位。都满足的话，换算时间。代码可以这样写

```c
void gettime(char *gps, int *hour, int *minutes, int *second)
{
    if (gps[0] == '$' && gps[1] == 'G' && gps[2] == 'P' && gps[3] == 'R' && gps[4] == 'M' && gps[5] == 'C') //判断是否为$GPRMC语句
    {
        char hex[] = {'A', 'B', 'C', 'D', 'E', 'F'}; //转换过程中会出现A-F， 代表10 - 15的值
        int xor, hexxor, i;
        char savehex[MAX] = {'0'}, check[MAX] = {"0"}; //日常初始化

        xor = gps[1] ^ gps[2];
        for (i = 3; gps[i] != '*'; i++)
            xor ^= gps[i];
        xor = xor % 65536; //获取异或值

        for (i = 0; xor != 0; i++)
        {
            hexxor = xor % 16;
            if (hexxor >= 10) //余数如大于等于10， 就说明应该是A-F某个数
                savehex[i] = hex[hexxor - 10];//获取大写字母
            else
                savehex[i] = hexxor + '0'; //获取数字字符
            xor = xor / 16;
        }

        for (int i = strlen(gps) - 1, j = 0; gps[i] != '*'; i--, j++) //获取校验值
            check[j] = gps[i];

        for (i = 7; gps[i] != '*'; i++) //得到字段2的地址
            if (gps[i] == ',')
                break;

        if (gps[i + 1] == 'A' && strcmp(savehex, check) == 0) //各种换算，这里有个有趣的地方，因为savehex与check的校验值都是倒着的，所以直接比较，以及时间地址是确定的
        {
            *hour = (gps[7] - '0') * 10 + gps[8] - '0';
            *hour = (*hour + 8) % 24;
            *minutes = (gps[9] - '0') * 10 + gps[10] - '0';
            *second = (gps[11] - '0') * 10 + gps[12] - '0';
        }
    }
}
```

**整体代码可为**

```c
#include <stdio.h>
#include <string.h>
#define MAX 1000

void gettime(char *gps, int *hour, int *minutes, int *second);

int main(int argc, char const *argv[])
{
    int hour, minutes, second;
    char gps[MAX] = {'0'};
    char end[] = "END";
    
    do
    {
        scanf("%s", gps);
        gettime(gps, &hour, &minutes, &second);
    } while (strcmp(gps, end) != 0);

    printf("%02d:%02d:%02d", hour, minutes, second);

    return 0;
}

void gettime(char *gps, int *hour, int *minutes, int *second)
{
    if (gps[0] == '$' && gps[1] == 'G' && gps[2] == 'P' && gps[3] == 'R' && gps[4] == 'M' && gps[5] == 'C') 
    {
        char hex[] = {'A', 'B', 'C', 'D', 'E', 'F'};
        int xor, hexxor, i;
        char savehex[MAX] = {'0'}, check[MAX] = {"0"};

        xor = gps[1] ^ gps[2];
        for (i = 3; gps[i] != '*'; i++)
            xor ^= gps[i];
        xor = xor % 65536;

        for (i = 0; xor != 0; i++)
        {
            hexxor = xor % 16;
            if (hexxor >= 10) 
                savehex[i] = hex[hexxor - 10];
            else
                savehex[i] = hexxor + '0';
            xor = xor / 16;
        }

        for (int i = strlen(gps) - 1, j = 0; gps[i] != '*'; i--, j++) 
            check[j] = gps[i];

        for (i = 7; gps[i] != '*'; i++)
            if (gps[i] == ',')
                break;

        if (gps[i + 1] == 'A' && strcmp(savehex, check) == 0)
        {
            *hour = (gps[7] - '0') * 10 + gps[8] - '0';
            *hour = (*hour + 8) % 24;
            *minutes = (gps[9] - '0') * 10 + gps[10] - '0';
            *second = (gps[11] - '0') * 10 + gps[12] - '0';
        }
    }
}
```

**结果**

因为懒，博文写的太晚，现在再登陆已经暂时看不到结果，但是分数是都拿到手了![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)
