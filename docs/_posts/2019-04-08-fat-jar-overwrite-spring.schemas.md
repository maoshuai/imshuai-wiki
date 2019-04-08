---
title: fat jar与spring.schemas文件覆盖
tags: Java Spring
---
# 问题
新搭建了一个Spring项目，在本地IDE启动测试没有问题，但通过maven-shade插件，将整个工程打包为一个fat jar放在服务器上运行，竟然报XML中的beans元素定义不存在：
```
cvc-elt.1: Cannot find the declaration of element 'beans'.
```
这种错误，之前出现过，经常是写错命名空间。但这一次本地IDE启动没有问题，说明是环境问题。而两者的差别主要在于maven-shade的打包方式。经排查原因整理如下。
<!--more-->
# spring.schemas

首先查看了xml中的命名空间定义，以及`xsi:schemaLocastion`发现都没有问题:
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
...
      xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
```
这里涉及一个问题，Spring是怎么知道`http://www.springframework.org/schema/beans/spring-beans-3.0.xsd`指代的xsd文件的具体位置呢？肯定不是从这个URL下载的，因为内网环境是不可能联通的。

实际上，在spring-beans.jar的org/springframework/beans/factory/xml目录下，确实是有一系列的spring-beans.xsd文件存在的，spring必然是通过某种方式映射了URL到具体的XSD文件。

spring官方文档提到如何扩展schema定义，提到META-INF/spring.schemas可用来映射XSD文件的具体位置。而spring-beans.jar里也有文件META-INF/spring.schemas，用来定义spring自带的XSD文件位置，大概内容如下：
```
http\://www.springframework.org/schema/beans/spring-beans-2.0.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-2.5.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.0.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.1.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.2.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-4.0.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-4.1.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-4.2.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
http\://www.springframework.org/schema/beans/spring-beans-4.3.xsd=org/springframework/beans/factory/xml/spring-beans.xsd
...
```
而Spring 会根据此文件的描述，去寻找对应XSD文件。

# fat jar 导致文件覆盖
明白spring.schema的作用，再来查看生成的jar文件，发现spring.schema里的内容并没有spring的xsd配置，只有两行某框架的配置。我大概明白了，是因为某个框架增加了自定义的XSD，定义了自己的spring.schema文件，IDE里运行时，是在不同的jar中，互不干扰。而合并到fat jar时覆盖了spring自带的spring.schema文件。从而导致以上报错。另一方面，也说明fat jar容易产生文件覆盖冲突。

然后就是解决maven-shade插件打包覆盖的问题，查询了一下，可以通过增加AppendingTransformer配置实现：
```xml
 <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <executions>
                ...
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.handlers</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.schemas</resource>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
```

# See also
1. [Extensible XML authoring](https://docs.spring.io/spring/docs/3.0.5.RELEASE/spring-framework-reference/html/extensible-xml.html#extensible-xml-registration)
2. <https://github.com/spring-projects/spring-framework/blob/master/spring-beans/src/main/resources/META-INF/spring.schemas>
3. [Idea to avoid that spring.handlers/spring.schemas get overwritten when merging multiple spring dependencies in a single jar](https://stackoverflow.com/questions/5586515/idea-to-avoid-that-spring-handlers-spring-schemas-get-overwritten-when-merging-m)
