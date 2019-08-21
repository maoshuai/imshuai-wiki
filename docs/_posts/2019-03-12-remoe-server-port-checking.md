---
title: 检查远程端口是否开放
tags: Linux Network Shell
---

初次搭建环境，或者排查运维问题，通常需要做连通性验证。

连通性验证的重要任务之一是，检查目标机器的服务端口是否可联通。常见的端口检测方法有 telnet、nc 等方法，整理如下。

<!--more-->

# telnet

telnet 是最普遍的方法。非常简单，语法如下：

```
telnet $host $port
```

输出如下，表示端口联通（通过`ctl+]`退出）：

```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ telnet 127.0.0.1 22
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

如下，表示端口不通：

```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ telnet 127.0.0.1 222
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection refused
```

# nc

telnet 是交互式的，适合单次手工检测，对于批量检测，利用nc更为方便。基本用法如下：

```
nc -z $host $port
```

然后检测上述命令的exit code，为0表示可联通，否则表示不联通：


```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ nc -z 127.0.0.1 22
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ echo $?
0
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ nc -z 127.0.0.1 222
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ echo $?
1
```

假如需要检测一批IP下的端口是否联通，可以用下面的脚本：

```
#!/bin/bash
# IPs and ports to check
IP_PORT="127.0.0.1 22
127.0.0.1 21
127.0.0.1 222
192.168.3.1 80"

# checking
echo "$IP_PORT" |
while read line;do
  nc -z -w 2 $line
  echo $line $?
done
```
上述脚本，可能输出如下，第二列0表示联通，否则为不通：
```
127.0.0.1 22 0
127.0.0.1 21 1
127.0.0.1 222 1
192.168.3.1 80 1
```
脚本中增加了`-w`选项，用于控制最大探测超时时间为2秒。


当然，也可以通过`-v`选项直接输出探测信息，适合单次手工查验，：

```
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ nc -zv 127.0.0.1 22
Connection to 127.0.0.1 22 port [tcp/ssh] succeeded!
maoshuai@maoshuai-ubuntu-desktop-18:/tmp$ nc -zv 127.0.0.1 222
nc: connect to 127.0.0.1 port 222 (tcp) failed: Connection refused

```

# 通过写入设备文件

由于Linux里，一切都是文件，网络连接也对应一个文件。因此可以通过直接写入文件判断端口是否联通。

这种方法有点awkward，但是最兼容的方法。如果所在的服务器中没有安装nc甚至telnet命令（比如某些docker容器中），用法如下：

```
echo > /dev/tcp/$host/$port
```

判断上述命令的退出码：

```
maoshuai@maoshuai-ubuntu-desktop-18:/dev$ echo > /dev/tcp/127.0.0.1/22
maoshuai@maoshuai-ubuntu-desktop-18:/dev$ echo $?
0
maoshuai@maoshuai-ubuntu-desktop-18:/dev$ echo > /dev/tcp/127.0.0.1/222
bash: connect: Connection refused
bash: /dev/tcp/127.0.0.1/222: Connection refused
maoshuai@maoshuai-ubuntu-desktop-18:/dev$ echo $?
1
```

当然，也可以通过脚本，批量检测：
```
#!/bin/bash
# IPs and ports to check
IP_PORT="127.0.0.1 22
127.0.0.1 21
127.0.0.1 222
192.168.3.1 80"

# checking
echo "$IP_PORT" |
while read line;do
  ip=$(echo $line | awk '{print $1}')
  port=$(echo $line | awk '{print $2}')
  (echo > /dev/tcp/$ip/$port) >/dev/null 2>&1
  echo $line $?
done
```

但有个地方注意，必须用bash执行写入。

# 在服务部署前检测端口

有些情况，我们需要在服务部署前检测该服务的端口是否可访问。问题是此时这个端口并没有被监听，直接检测的结果自然是不通，我们需要排除，这不是因为网络本身不通（比如有防火墙）造成的。

一种办法是，在目标机器的该端口部署一个简单的监听，而`nc`命令恰巧可以完成。

# 参考文档

* [Check whether a remote server port is open on Linux](https://www.pixelstech.net/article/1514049471-Check-whether-a-remote-server-port-is-open-on-Linux)
* [Test from shell script if remote TCP port is open](https://stackoverflow.com/questions/4922943/test-from-shell-script-if-remote-tcp-port-is-open/5398366)
* [bash and /dev/tcp - how does that work ?](https://ubuntuforums.org/showthread.php?t=1656623)
* [5 Linux Utility to Test Network Connectivity](https://geekflare.com/linux-test-network-connectivity/amp/)