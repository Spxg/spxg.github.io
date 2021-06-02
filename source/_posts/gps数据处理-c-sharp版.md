---
title: GPS数据处理-C#版
tags: []
id: '621'
categories:
  - - C Sharp
date: 2019-10-05 15:37:49
---

_C版本见这_

[C语言程序设计CAP-第10章编程题-GPS数据处理](https://unsafe.me/archives/567)

国庆总得给自己找点事做，不打游戏，不想做作业，还能干啥?

```cs
using System;

namespace program
{
    class Program
    {
        static void Main(string[] args)
        {
            Gps gps = new Gps();
            string hour = null;
            string minute = null;
            string second = null;
            string str;

            str = Convert.ToString(Console.ReadLine());

            while (!str.Equals("END"))
            {
                if (gps.CheckHeader(str) && gps.CheckNum(str) && gps.CheakLocated(str))
                {
                    hour = gps.GetHour(str);
                    minute = gps.GetMinute(str);
                    second = gps.GetSecond(str);
                }
                str = Convert.ToString(Console.ReadLine());
            }

            Console.WriteLine(hour + ":" + minute + ":" + second);
        }
    }

    class Gps
    {
        public bool CheckHeader(string str)
        {
            return str.Substring(0, 6).Equals("$GPRMC");
        }

        public bool CheckNum(string str)
        {
            //对比C版，对异或值的计算更合理。
            int i;
            int xornum = 0;

            for (i = 1; str[i] != '*'; i++)
                xornum ^= str[i];
            xornum %= 65536;

            return Convert.ToString(xornum, 16).Equals(str.Substring(i + 1));
        }

        public bool CheakLocated(string str)
        {
            int i;
            for (i = 7; str[i] != ','; i++)
                continue;
            return str[i + 1].Equals('A');
        }

        public string GetHour(string str)
        {
            int hour = (str[7] - '0') * 10 + str[8] - '0';
            hour = (hour + 8) % 24;
            return hour.ToString().PadLeft(2, '0');
        }

        public string GetMinute(string str)
        {
            int minute = (str[9] - '0') * 10 + str[10] - '0';
            return minute.ToString().PadLeft(2, '0');
        }

        public string GetSecond(string str)
        {
            int second = (str[11] - '0') * 10 + str[12] - '0';
            return second.ToString().PadLeft(2, '0');
        }
    }
}
```
