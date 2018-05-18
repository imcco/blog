---
title: JDK版本共存切换研究
tags: JDK
category: JDK
abbrlink: 20841
date: 2018-04-01 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0166.jpg)
JDK版本共存切换研究
<!--more-->
> **场景**：公司项目使用的jdk为1.7，最近不是很忙，找到一个爬虫系统学习。该系统使用到了jdk1.8的特性，所以I need 俩版本，开整！！！

#### 准备两个版本的jdk

```
jdk-7u67-windows-x64.exe
jdk-8u162-windows-x64.exe
```

#### 安装jdk路径为：

```
D:\Java\JDK\JDK1.7\jdk1.7.0_67
D:\Java\JDK\JDK1.8\jdk1.8.0_162
```

> **tips**: win7以下电脑推荐使用Rapid Environment Editor修改电脑环境变量。

####设置两个子JAVA_HOME

一个总，设置两个子JAVA_HOME

```
JAVA_HOME7 = D:\Java\JDK\JDK1.7\jdk1.7.0_67
JAVA_HOME8 = D:\Java\JDK\JDK1.8\jdk1.8.0_162
```

此处JAVA_HOME设置即为你更换jdk版本是所要修改的地方

```
JAVA_HOME = %JAVA_HOME8%
```

#### 设置path 

添加如下内容(注意添加’;’)

```
;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
```

#### 添加classpath变量

> 变量值

```
%JAVA_HOME%lib\dt.jar;%JAVA_HOME%\lib\tools.jar
```

#### 查看版本是否更换成功

```
java -version javac -version
```

> 若未成功，请看接下来的6

#### 未成功解决方案

```
删除C:\Windows\System32目录下的java.exe,javaw.exe，javaws.exe删除即可。 若java -version和javac -version版本不一致 将%JAVA_HOME%\bin加在PATH变量的头，执行java -version和javac -version，版本已然一致。
```
#### 注册表

```
HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment
```

#### 注意：

`C:\Windows\System32底下的java.exe，javaw.exe，javaws.exe是JDK7的，版本就是JDK7；是JDK8的，版本就是JDK8。`

安装jdk8的时候，安装过程中会在系统变量Path的最前面加上了`C:\ProgramData\Oracle\Java\javapath;`，这是安装jdk8的时候带出来的，并且在Path的最前面，所以无论修改注册表还是Java控制台都没有用，执行的指令在系统变量中搜寻命令时最先找到的就是`C:\ProgramData\Oracle\Java\javapath;，`始终是jdk8的。那么，我们需要把Path最前面的`C:\ProgramData\Oracle\Java\javapath;`删除，这样才能对JAVA_HOME修改来切换需要的jdk环境。

#### 参考：

- JDK1.6,JDK1.7,JDK1.8安装共存问题 https://blog.csdn.net/just_coming_here/article/details/51775909

- JDK7与JDK8环境共存与切换 https://www.cnblogs.com/iathanasy/p/8507647.html

- JDK7与JDK8共存问题小思 https://www.cnblogs.com/fxmemory/p/7234848.html
