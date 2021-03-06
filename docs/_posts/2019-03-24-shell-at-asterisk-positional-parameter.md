---
title: Shell特殊参数\$@和\$*举例解析
tags: Shell Linux
---

Shell 中若当前脚本只是作为一个包装器（wrapper），需要把所有位置参数继续传递给实际的脚本或函数。这时候，可以选择特殊参数完成：艾特符号`$@` 和星号 `$*`，它们的基本含义是扩展为所有位置参数。多数情况下，使用起来都没什么区别，但不追究明白很容易出现bug，下面举例说明。

<!--more-->

# man文档
关于这两个参数的说明，首先可以在man bash里的PARAMETERS章节下的Special Parameter查看说明，下面是节选：

```
The shell treats several parameters specially.  These parameters may only be referenced; assignment to them is not allowed.
*      Expands to the positional parameters, starting from one.  When the expansion occurs within double quotes, it expands to a single word  with the  value  of  each  parameter  separated by the first character of the IFS special variable.  That is, "$*" is equivalent to "$1c$2c...",
      where c is the first character of the value of the IFS variable.  If IFS is unset, the parameters are separated by spaces.  If IFS is null,
      the parameters are joined without intervening separators.
@      Expands to the positional parameters, starting from one.  When the expansion occurs within double quotes, each parameter expands to a sepa-
      rate word.  That is, "$@" is equivalent to "$1" "$2" ...  If the double-quoted expansion occurs within a word, the expansion of  the  first
      parameter  is  joined with the beginning part of the original word, and the expansion of the last parameter is joined with the last part of
      the original word.  When there are no positional parameters, "$@" and $@ expand to nothing (i.e., they are removed).
```

man文档太抽象，我们先放下。

# 四种引用形式

使用@或*符号，加上是否使用双引号两种情况，我们一共可以得到四种引用形式，到底应该怎样选择呢？

* $*
* $@
* "$*"
* "$@"

# 举例说明
man文档的说明太抽象，我们编写一个测试脚本（假如叫test.sh），加深理解。脚本中bar()函数用来接受参数，我们依次使用四种形式将位置参数传递过去，看一下实际的解析情况。代码如下：

```bash
#!/bin/bash

bar(){
  echo "num of args: $#"
  echo "arg1: $1"
  echo "arg2: $2"
  echo "arg3: $3"
  echo "arg4: $4"
  echo ""
}

echo 'using $*'
bar $*

echo 'using $@'
bar $@

echo 'using "$*"'
bar "$*"

echo 'using "$@"'
bar "$@"
```

在测试之前，我们还要在当前目录touch一个名为javac的文件，如下：
```
maoshuai@maoshuai-ubuntu-desktop-18:~/test$ touch javac
maoshuai@maoshuai-ubuntu-desktop-18:~/test$ ls -l
total 4
-rw-rw-r-- 1 maoshuai maoshuai   0 Mar 24 15:18 javac
-rwxr-xr-x 1 maoshuai maoshuai 233 Mar 24 15:13 test.sh
```

最后，执行命令`/test.sh "I like" coding "java*"`观察输出：
```
maoshuai@maoshuai-ubuntu-desktop-18:~/test$ ./test.sh "I like" coding "java*"
using $*
num of args: 4
arg1: I
arg2: like
arg3: coding
arg4: javac

using $@
num of args: 4
arg1: I
arg2: like
arg3: coding
arg4: javac

using "$*"
num of args: 1
arg1: I like coding java*
arg2:
arg3:
arg4:

using "$@"
num of args: 3
arg1: I like
arg2: coding
arg3: java*
arg4:

```

可以看出，输出都不一样，按照我们的本意，第4个是符合预期要求的，其他情况都有所改变。

1. 使用`$*`

    相当于将所有位置参数不带引号的形式再次传递出去。由于不带引号，最大的问题是参数个数会被重新解析。比如第一个参数`"I like"`作为一个整体被拆成了2个位置参数。
另外，由于每个位置参数都没有双引号，会再次做变量扩展，因此`java*`变成了`"javac"`。
    这种使用方式，相当于`bar $1 $2 $3 $4`。所以最终输出结果是4个位置参数，并且`java*`被扩展为`"javac"`。

2. 使用`@*`

    与  `$*`没有区别。

3. 使用`""$*"`

    根据man文档解释，相当于将所有位置参数连接成一个word。连接符就是IFS（一般就是空格），所以，此时输出的结果只有一个位置参数，位置参数格式变成了1个。由于有双引号的作用，`java*`没有被扩展为`"javac"`。
    这种方式，相当于`bar "$1 $2 $3 $4" `

4. 使用`"$@"`
    根据man文档解释，相当于拆分成多个word，并且每个word都有双引号。即相当于`bar "$1" "$2" "$3" "$4"`。因此，位置参数格式保持不变，同时`java*`没有被扩展为`"javac"`

总结下来，下面四种情况的等价写法如注释：
```bash
#!/bin/bash

bar(){
  echo "num of args: $#"
  echo "arg1: $1"
  echo "arg2: $2"
  echo "arg3: $3"
  echo "arg4: $4"
  echo ""
}

echo 'using $*'
# bar $1 $2 $3 $4
bar $*

echo 'using $@'
# bar $1 $2 $3 $4
bar $@

echo 'using "$*"'
# bar "$1 $2 $3 $4"
bar "$*"

echo 'using "$@"'
# bar "$1" "$2" "$3" "$4"
bar "$@"
```

# 优秀代码印证
大多数成熟的软件提供的脚本wrapper里，都是使用`"$@"`的，比如：
maven的bin/mvn命令：
```bash
exec "$JAVACMD" \
  $MAVEN_OPTS \
  $MAVEN_DEBUG_OPTS \
  -classpath "${CLASSWORLDS_JAR}" \
  "-Dclassworlds.conf=${MAVEN_HOME}/bin/m2.conf" \
  "-Dmaven.home=${MAVEN_HOME}" \
  "-Dlibrary.jansi.path=${MAVEN_HOME}/lib/jansi-native" \
  "-Dmaven.multiModuleProjectDirectory=${MAVEN_PROJECTBASEDIR}" \
  ${CLASSWORLDS_LAUNCHER} "$@"
```

flink的bin/flink命令
```bash
exec $JAVA_RUN $JVM_ARGS "${log_setting[@]}" -classpath "`manglePathList "$CC_CLASSPATH:$INTERNAL_HADOOP_CLASSPATHS"`" org.apache.flink.client.cli.CliFrontend "$@"
```

tomcat的bin/startup

```bash
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

但也有写存在问题的，比如maven的[deploySite.sh](https://github.com/apache/maven/commit/d13c2884879d6bc2b8f18012b8455c06b2fe0756)内容是：
```bash
mvn -Preporting site site:stage $@
mvn scm-publish:publish-scm $@
```
修改为带双引号则更为安全：
```bash
mvn -Preporting site site:stage "$@"
mvn scm-publish:publish-scm "$@""
```
# 总结
通过测试，可以得出下面的经验：
1. 如果希望位置参数**原封不动**的将参数传递出去，需使用带双引号的@符号，即`"$@"`。
2. 脚本里对变量的引用**最好总是加上双引号**，否则可能引起不可预知的变量扩展（variable expansion），比如上面例子，当前目录有个javac文件，导致`"java*"`被扩展成了`"javac"`

# See also
* [Advanced Bash-Scripting Guide: Chapter 9. Another Look at Variables](http://www.tldp.org/LDP/abs/html/internalvariables.html)