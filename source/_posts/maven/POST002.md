---
title: maven项目操作技巧
tags: maven
category: maven
abbrlink: 44283
date: 2018-03-26 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0164.jpg)
maven项目技巧
<!--more-->


## Eclipse导入Maven项目并启动的步骤

1. 空白处右击Import选择General里的Existing Projects into Workspace→Next，Browse找到要导入项目所在，选中要导入的项目点击确定；一定要勾选Copy projects into eorkspace→Finish；

2. 导入本地项目之后先进行配置，配置步骤如下：

   (1).导入项目 右击maven→update project →OK；

   (2).项目名→右击→properties

   (3).resource 修改字体 utf-8

​	(4).deployment  assembly  点击  add  添加maven dependencies

​	(6).java compiler 中 compiler compliance settings 选1.6

​	(7)找到Project Facets 查看Java是改为1.6；

​	(8)如果JRE版本不对，应该首先右击Properties选择Workspacec default JRE(jre6)点击OK；

3. 导入数据库数据步骤如下:

   (1).在localhost创建一个数据库，名字与导入文件一致；

   (2)右击选择运行SQL文件；数据库中导入数据文件，选择好要导入的文件，不勾选“每个运行中运行多重查询”和“SETAUTOCOMMIT=0”；开始即可

4. 导入之后将资源文件中关于数据库的配置用户名密码改成需要的即可；

5. 建server部署项目启动即可



## eclipse maven项目导出所使用的jar包

### 方法

1. 在eclipse中定位到maven项目的pom.xml文件

2. 右击pom.xml文件，选择Run As--》Run Configurations（或者Maven build…）

3. 在打开的页面中，Goals输入“dependency:copy-dependencies”，后点击“Run”即可

4. 在当前项目的目录的“targed/dependency”下就可以看见。

### 方法2

1. 或者：在dos环境，进入到pom.xml所在的文件夹。

   输入命令：mvn dependency:copy-dependencies

   也可在当前项目的目录的“targed/dependency”下开间maven项目引用的jar包。
   eclipse 中maven项目的运行

## eclispe maven项目运行


1. 项目构建时，右键pom.xml文件，选择Run As--》Run Configurations（或者Maven build…）

Golas:中输入项目构建命令，比如compile, run后就开始编译了。

2. 开发中代码的运行调试，找到要运行的java代码运行，和普通的Java代码一样运行就可以了。
