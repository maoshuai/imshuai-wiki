---
title: base64屏幕传递文件法
tags: Shell
---
# 问题

有些情况下，两个不连通的系统很难传递文件，或者流程很麻烦（比如我司，生产环境两个服务器传文件，必须让领导审批，有时候为了快速传递一个小文件，可能要个把小时的流程）。幸亏，**在客户端可以同时ssh两台机器**。下面介绍一种**base64+终端字符复制**的方式跨系统传文件。

<!--more-->


# 方法

如果是文本文件（比如shell脚本），我们可以直接通过在A终端拷贝到本地剪切板，然后在B终端vi粘贴输入保存。但若是二进制则比较麻烦，可通过**base64**的方式间接完成，步骤如下：

比如A机器下有一个8.3K的文件java（二进制的），要传递给B文件：

```
-rwxr-xr-x 1 maoshuai maoshuai 8.3K 4月  17 20:31 java
```

我们先通过gzip把文件压缩一下（-9表示最大压缩比）：

```shell
gzip -9 -c java > java.gz
```

生成压缩文件java.gz只有2.4k

```
-rwxr-xr-x 1 maoshuai maoshuai 8.3K 4月  17 20:31 java
-rw-r--r-- 1 maoshuai maoshuai 2.4K 4月  17 20:36 java.gz
```

然后讲java.gz文件转换为base64文本：

```shell
base64 java.gz
```

拷贝如下文本到剪切板

```
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
```

到另外一台机器，使用vi，将剪切板内容保存到文本，比如java.txt。然后用base64解码，并重定向到文件：

```shell
base64 -d java.txt > java.gz
```

解压：

```shell
gunzip java.gz
```

# 总结

如果服务器端口开放，可以通过在A服务器开一个HTTP服务，比如用python的SimpleHTTPServer：

```shell
python -m "SimpleHTTPServer" 9001
```
然后在另外一台服务器用curl或wget下载：

```
wget xxxx:9001/xxx
```
这种方法，可能会被审计发现，随便在生产服务器开端口也是很危险的。而上述的拷贝终端屏幕，可以说是完全合规的，对于小文件的传递还算方便，一般10k以内都很容易。

也适合多个小文本文件的传递（比如10个shell脚本，每一个只有1k），可以先tar打包成.tar.gz文件，再base64转码；到对方服务器解码后再解压。

