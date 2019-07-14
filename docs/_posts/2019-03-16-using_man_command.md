---
title: man用法总结
tags: Linux Shell
---

遇到不熟悉的 Linux 命令，除了上网搜外，最好是查看系统自带的帮助文档。一来，这是第一手资料，肯定是最准确的；二来，相对网上东一榔头西一棒子的解释，更具有系统性。

虽然 `man` 命令非常简单，但也会碰到一些疑惑，下面我总结了一下，让以后使用 `man` 更游刃有余。

<!--more-->

# 多个man结果

使用 `man` 的第一个坑是，同一个关键字可能会有多个结果。如果对此不了解，阅读错误的文档，会搞得自己一团浆糊，甚至造成阅读man文档的心理阴影。

`man` 将帮助文档分为多个 section，所谓 section 可以理解为不同的主题。section 用数字或字母区分，比如最基本的划分标准是这样的：

```
       1   Executable programs or shell commands
       2   System calls (functions provided by the kernel)
       3   Library calls (functions within program libraries)
       4   Special files (usually found in /dev)
       5   File formats and conventions eg /etc/passwd
       6   Games
       7   Miscellaneous  (including  macro  packages  and  conventions), e.g. man(7), groff(7)
       8   System administration commands (usually only for root)
       9   Kernel routines [Non standard]
```
对于一般用户来说，最常用的 section 是 1，即 shell 命令或可执行程序。阅读 man 文档时，系统几乎都是会指明 section 是什么，比如下面是 `printf` 的 man 文档，在第一行会显示 `PRINTF(1) `  字样，其中括号内的数字就是 sectioin 。因此，我们基本明白这是一个 shell 命令。

```
PRINTF(1)                   User Commands                  PRINTF(1)

NAME
       printf - format and print data

SYNOPSIS
       printf FORMAT [ARGUMENT]...
       printf OPTION

DESCRIPTION
       Print  ARGUMENT(s)  according to FORMAT, or execute according
       to OPTION:
```

因此，要准确的定位一份man文档，应该指出**名字 + section** 。同样 `printf` 还是 C 语言的库函数，即对应 section 是 3，显示如下：

```
PRINTF(3)             Linux Programmer's Manual            PRINTF(3)

NAME
       printf,   fprintf,   dprintf,   sprintf,  snprintf,  vprintf,
       vfprintf, vdprintf, vsprintf, vsnprintf  -  formatted  output
       conversion

SYNOPSIS
       #include <stdio.h>

       int printf(const char *format, ...);
```

# 指定 section
如果有多个 section，有些系统会直接给出第一个匹配 section（比如 Ubuntu），而有些会提示多个结果，让用户选择（比如 SUSE）。但不管怎样，都可以通过如下方式显示指定要查找命令的section：

```
man [section] [comman]
```

比如，查找 `printf` 的 C 语言库函数用法，可以输入 `man 3 printf` 。

如果要列出所有 section，可以增加 `-f` 选项，比如 `man -f printf`，输出如下：

```
maoshuai@maoshuai-ubuntu-desktop-18:~$ man -f printf
printf (1)           - format and print data
printf (3)           - formatted output conversion
```

关于 section，有时候还会出现类似 `1p` 这样的命名，这是带有后缀的 section 名，相当于是子 section。


# man 文档格式

man 文档基本遵循一套写作模板，对读者重要的章节有：
* NAME：名字和一句话简单描述。
* SYNOPSIS：语法格式，比如 shell 命令会罗列所有的 option 和 argument 用法。
* DESCRIPTION：详细用法解释，内容比较多。可以根据自己感兴趣的点搜索。
* EXAMPLES：用法举例，有时直接翻到最后看例子，会更容易理解。

# 语法格式

SYNOPSIS 介绍语法，但一大串很难懂。掌握一些规则更容易理解。

* [-abc] ：表示中括号内任一一个 option 都是可选的。
* -a\|-b：竖线分隔的 option 不可同时出现
* ... ：表示可重复的
* <>：表示必选

# man 的其他用法

## 模糊搜索
`-k` 选项，可以通过正则表达式，模糊匹配某个命令，比如  `man -k printf` 会搜索所有 man 文档中名字或描述包含 printf 关键字的。

```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ man -k printf
asprintf (3)         - print to allocated string
dprintf (3)          - formatted output conversion
fprintf (3)          - formatted output conversion
fwprintf (3)         - formatted wide-character output conversion
printf (1)           - format and print data
printf (3)           - formatted output conversion
snprintf (3)         - formatted output conversion
sprintf (3)          - formatted output conversion
swprintf (3)         - formatted wide-character output conversion
vasprintf (3)        - print to allocated string
vdprintf (3)         - formatted output conversion
vfprintf (3)         - formatted output conversion
vfwprintf (3)        - formatted wide-character output conversion
vprintf (3)          - formatted output conversion
vsnprintf (3)        - formatted output conversion
vsprintf (3)         - formatted output conversion
vswprintf (3)        - formatted wide-character output conversion
vwprintf (3)         - formatted wide-character output conversion
wprintf (3)          - formatted wide-character output conversion
```

当然可以用正则，比如 `man -k "^print"`，搜索名字或描述以 print 开头的：
```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ man -k "^print"
arch (1)             - print machine hardware name (same as uname...
asprintf (3)         - print to allocated string
bsd-from (1)         - print names of those who have sent mail
date (1)             - print or set the system date and time
dmesg (1)            - print or control the kernel ring buffer
egrep (1)            - print lines matching a pattern
fgconsole (1)        - print the number of the active VT.
fgrep (1)            - print lines matching a pattern
fmtmsg (3)           - print formatted error messages
from (1)             - print names of those who have sent mail
getkeycodes (8)      - print kernel scancode-to-keycode mapping t...
git-grep (1)         - Print lines matching a pattern
grep (1)             - print lines matching a pattern
```

## 查找 man 文档的存储位置

man 文档是存储一个特定位置，通过命令 `man -w [command]` 可以定位文档，比如：

```
maoshuai@maoshuai-ubuntu-desktop-18:~$ man -w ls
/usr/share/man/man1/ls.1.gz
```
# man查看内置命令
有时，你会发现一些常用的命令（比如cd），man命令竟然找不到帮助文档，或者直接打开一个的“BUILTIN(1)”的man页面。原因是这些命令是shell内建的命令，没有单独的man文档。如果要看其使用方法，可以进入bash的man文档搜索即可。

在某些平台（如Ubuntu）会直接提供一个help命令完成这个操作，比如`help cd`。如果没有这个命令，网上也有人给出了一个代替方法，把它放到~/.bashrc下，以后就可以用类似 `bashman cd` 的方法查看了：
```
bashman () 
{ 
    man bash | less -p "^       $1 "
}
```

# See also

* [man page - wikipedia](https://en.wikipedia.org/wiki/Man_page)
* [Linux man Command Tutorial for Beginners (8 Examples)](https://www.howtoforge.com/linux-man-command/)
* manual of man
* [Where to view man pages for builtin commands?](https://stackoverflow.com/questions/22991942/where-to-view-man-pages-for-builtin-commands)