---
title: 记一次寻找 OpenWrt 问题的过程
tags: []
categories:
  - - 技术相关
date: 2022-04-05 10:05:10
---

固件是我自己编译的，源码来自 [lean](https://github.com/coolsnowwolf/lede)，是个十分不错的项目。


**问题：**
今天我发现我 `OpenWrt` 存在 `luci` 界面下修改设置无法生效的问题，就像这样：
![无法生效](/images/记一次寻找openwrt问题的过程/issue.png)
万能的重启后，仅第一次可以成功设置。

<!-- more -->

**过程：**
有着运气加成

开始由 `ZeroTier` 入手，原本以为是插件的问题，但是寻找了相关提交也没发现有端倪。然后试了试其他插件也是如此，便判定是固件问题。

翻看了系统日志和内核日志也没有收获，直到我查看了系统进程：
![进程](/images/记一次寻找openwrt问题的过程/top.png)
~~这，这不对吧？这今天谁，谁要陷害我？~~

我们可以看到一排排的 
```sh
/bin/sh /sbin/luci-reload xxx
lock /var/run/luci-reload
```

根据我的直觉，判断出可能出现死锁了。尝试 `kill` 所有 `luci-reload`  和 `lock` 进程，之后修改设置成功生效！但是也只成功一次，且发现 `lock` 进程依旧存在。

搜索一波[源文件](https://github.com/coolsnowwolf/luci/blob/3fe64081aa5c14895d25dbadf7484dde61ed7a1d/modules/luci-base/root/sbin/luci-reload)，可以看到有这么几行

```sh
lock "/var/run/luci-reload"

config_load ucitrack

for i in $*; do
	config_foreach apply_config $i $i
done

rm -f "/var/run/luci-reload-status"
lock -u "/var/run/luci-reload"
```

意思就是，应用配置文件的时候加个锁，应用完成后解锁。

是没有执行到解锁程序吗？我做了个测试，创建了 `/var/run/luci-reload-status` 文件，随后应用设置，发现文件被删除，所以可以得出解锁程序有被执行。

至此问题很清晰了，**第一次应用设置后，锁未释放，导致接下来的应用设置无法获取锁，所以无法进行设置**。

试了试，确实是这样：
![尝试](/images/记一次寻找openwrt问题的过程/try.jpg)
现在的问题转变为什么解锁失败了

一顿搜索，找到了 `lock` 程序的[源代码](https://github.com/coolsnowwolf/lede/blob/master/package/utils/busybox/patches/220-add_lock_util.patch)

```c
/*
 * Copyright (C) 2006 Felix Fietkau <nbd@nbd.name>
 *
 * This is free software, licensed under the GNU General Public License v2.
 */

//config:config LOCK
//config:	bool "lock"
//config:	default n
//config:	help
//config:	  Small utility for using locks in scripts

//applet:IF_LOCK(APPLET(lock, BB_DIR_BIN, BB_SUID_DROP))

//kbuild:lib-$(CONFIG_LOCK) = lock.o

//usage:#define lock_trivial_usage NOUSAGE_STR
//usage:#define lock_full_usage ""

#include <sys/types.h>
#include <sys/file.h>
#include <sys/stat.h>
#include <signal.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include "busybox.h"

static int unlock = 0;
static int shared = 0;
static int waitonly = 0;
static int try_lock = 0;
static int fd;
static char *file;

static void usage(char *name)
{
	fprintf(stderr, "Usage: %s [-suw] <filename>\n"
	                "	-s	Use shared locking\n"
	                "	-u	Unlock\n"
	                "	-w	Wait for the lock to become free, don't acquire lock\n"
			"	-n	Don't wait for the lock to become free. Fail with exit code\n"
					"\n", name);
	exit(1);
}

static void exit_unlock(int sig)
{
	flock(fd, LOCK_UN);
	exit(0);
}

static int do_unlock(void)
{
	FILE *f;
	int i;

	if ((f = fopen(file, "r")) == NULL)
		return 0;

	fscanf(f, "%d", &i);
	if (i > 0)
		kill(i, SIGTERM);

	fclose(f);

	return 0;
}

static int do_lock(void)
{
	int pid;
	int flags;
	char pidstr[8];

	if ((fd = open(file, O_RDWR | O_CREAT | O_EXCL, 0700)) < 0) {
		if ((fd = open(file, O_RDWR)) < 0) {
			fprintf(stderr, "Can't open %s\n", file);
			return 1;
		}
	}

	flags = shared ? LOCK_SH : LOCK_EX;
	flags |= try_lock ? LOCK_NB : 0;

	if (flock(fd, flags) < 0) {
		fprintf(stderr, "Can't lock %s\n", file);
		return 1;
	}

	pid = fork();

	if (pid < 0)
		return -1;

	if (pid == 0) {
		signal(SIGKILL, exit_unlock);
		signal(SIGTERM, exit_unlock);
		signal(SIGINT, exit_unlock);
		if (waitonly)
			exit_unlock(0);
		else
			while (1)
				sleep(1);
	} else {
		if (!waitonly) {
			lseek(fd, 0, SEEK_SET);
			ftruncate(fd, 0);
			sprintf(pidstr, "%d\n", pid);
			write(fd, pidstr, strlen(pidstr));
			close(fd);
		}

		return 0;
	}
	return 0;
}

int lock_main(int argc, char **argv) MAIN_EXTERNALLY_VISIBLE;
int lock_main(int argc, char **argv)
{
	char **args = &argv[1];
	int c = argc - 1;

	while ((*args != NULL) && (*args)[0] == '-') {
		char *ch = *args;
		while (*(ch) > 0) {
			switch(*ch) {
				case 'w':
					waitonly = 1;
					break;
				case 's':
					shared = 1;
					break;
				case 'u':
					unlock = 1;
					break;
				case 'n':
					try_lock = 1;
					break;
			}
		}
		c--;
		args;
	}

	if (c != 1)
		usage(argv[0]);

	file = *args;
	if (unlock)
		return do_unlock();
	else
		return do_lock();
}
```

通过源码我们可知加解锁步骤：
* 加锁后，`fork` 出子进程注册 `signal`，当 `SIGKILL` 等三种情况时候触发解锁代码解锁
* 父进程将子进程的 `pid` 写入文件，之后退出
* 解锁通过读取文件，获取 `pid` 值 `kill` 掉子程序解锁

如果工作正常的话，我们应该可以在加锁后的文件中得到 pid，可是发现文件为空:
![文件为空](/images/记一次寻找openwrt问题的过程/file_empty.png)

可以知道 `pid` 并没有写入文件里。看着代码思来想去，并没有发现有什么问题。

而这时的我看了看这个文件[最近的提交](https://github.com/coolsnowwolf/lede/commit/d17bbf492dc6d44691726d16f25a03cb8550921a)：
![最近的提交](/images/记一次寻找openwrt问题的过程/commit.png)
`lean` 把它 `revert` 回去了，这是为啥呢？看了看上一个提交是扩大了 `pidstr` 的空间，因为不扩大的话，当 `pid` 过大的时候会造成 `buffer overflow`，提交本身意图并没有问题。此时的我并没有注意到问题的严重性，直到我看到了：
```c
// 参数写反了，函数的定义是
// int snprintf(char *str, size_t size, const char *format, ...);
snprintf(sizeof(pidstr), pidstr, "%d\n", pid);
```
*让我不由得在心里默念 `Rust` 大法好！*

看了看我的版本，刚好是这两个提交之间（运气太差）。吓得我连夜跑路：
![更新](/images/记一次寻找openwrt问题的过程/build.png)

**总结：**
这个问题主要是由于提交者参数使用错误，仓库 `reviewer` 也没发现问题以及 `C` 对类型检查的不严格导致的, ~~Rust 大法好~~。发现问题到解决的过程还是很愉快的。
