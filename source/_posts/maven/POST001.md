---
title: maven项目的tomcat配置
tags: maven
category: maven
abbrlink: 42040
date: 2018-03-20 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0159.jpg)

maven项目的tomcat配置
<!--more-->

### 一、没有Tomcat情况

  `Run As-->Maven build-->tomcat:run`    (不支持jdk1.8+tomcat8;项目启动没问题,访问报错500;jdk1.7 可以正常访问)

### 二、war包形式

1. 使用maven导出war包

   `Run As-->Maven build... -->package`

   成功后会提示生成的war包路径。一般在项目的target目录下。 (`注：pom.xml的<packaging/>配置为war打包后的就是war包，配置为jar时打包后的就是jar包`)

2. 将war包部署至tomcat中

将war放到Tomcat的webapps目录下，配置`conf\server.xml`文件`在<Host>中添加配置信息：(C:\Java\apache-tomcat-7.0.79\conf)`

- ① path:启动项目后访问项目的路径

- ② docBase:项目路径，可以使用绝对路径或相对路径，相对路径是相对于webapps

- ③ 你还可以在server.xml中配置你的端口号和项目名称，从而改变访问的url。    


3. 启动tomcat

注: 在部署项目的时候直接将web项目编译后的文件放在webapps也是同样的

     JavaEE项目部署 `默认存放在webapp-->WEB-INF-->classes下面(如：C:\Java\apache-tomcat-7.0.79\webapps\mobile_scm\WEB-INF\classes)`

      maven项目部署 `默认存放在target下面(如：E:\MavenWorks\babasport\target)`

### 三、Tomcat启动

项目发布 直接到webapps下

#### maven工程的tomcat热部署

教程：tomcat热部署：webapps下的项目正在运行，直接把开发的新版本发布到正在运行的Tomcat下(不能关闭Tomcat再发布新版本

   开发者本地将代码通过Git push到服务器端，服务器自动编译-打包-发布等等；也就是说发布到tomcat中后，不需要重启tomcat。

1. 热部署前准备：

- 配置Tomcat登录的用户名和密码

  热部署需要用户名和密码进行远程发布，修改user配置文件一是为了管理员进入tomcat管理页面并提高其安全性，二是为了在maven设置正确的用户名；

      `C:\Java\apache-tomcat-7.0.79\conf\tomcat-users.xml`

- 找到tomcat-user配置文件:

```xml
<!-- 配置Tomcat登录的用户名和密码 -->
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="admin"/>
<role rolename="manager"/>
<user username="tomcat" password="123456" roles="manager-gui,manager-script,admin,manager"/>
```

2. 重启Tomcat，访问`http://localhost:8080`

   进入tomcat服务器根目录，点击“manager app”输入用户名和密码，成功进入管理页面，说明第一步配置成功。

   注：端口号改为80 可以默认不显示；访问项目应该隐藏项目名称；

#### 热部署

1. Maven的Server的配置    

   在Maven的安装路径找到`conf目录下的setting.xml文件("E:\software\apache-maven-3.5.0\conf")，在<servers>节点中添加tomcat7`下配置的用户信息    

   ```xml
   <server>
       <id>tomcat</id>
       <username>admin</username>
       <password>password</password>
   </server>
   ```

   注：如果在pom.xml中没有配置用户名和密码，则使用setting里的配置，如果有的话，就是用pom里的配置（就近原则）配置完记得要`Maven-->Update projiect`(刷新)

2. pom.xml中添加tomcat插件

- ① `可以在pom.xml中右键-->Maven-->Add Plugin-->tomcat 自动添加插件`
- ② 可以手动配置插件和Tomcat的访问路径

```xml
<build>
<finalName>babasport</finalName>
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>2.0.2</version>
        <!-- 本地jdk版本 -->
        <configuration>
            <source>1.7</source>
            <target>1.7</target>
        </configuration>
    </plugin> 
    <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        
        <!-- 配置tomcat的访问 -->
        <configuration>
            <!-- 访问路径 -->
            <url>http://localhost:8080/manager/text</url>
            <!-- 项目发布到根目录,覆盖ROOT，URL访问可以省略项目名称 -->
            <!-- <path>/</path> -->
            <path>/babasport</path>
            <server>tomcat</server>  <!-- 此处的名字必须和setting.xml中配置的ID一致-->
            <username>tomcat</username>
            <password>123456</password>
        </configuration>
    </plugin>
</plugins>
</build>
```

3. 最后来验证一下：启动tomcat服务器，保证里面没有发布任何项目

- 如果是eclipse

  `直接右键项目---run as ---maven bulid...输入“tomcat7:deploy”(二次发布以后输入"tomcat7:redeploy")`

- 如果使用的是命令行

  `直接输入“mvn tomcat7:redeploy”`

测试通过，输入地址可以正常的访问！

#### server下设置tomcat发布位置

maven项目发布后默认存放到target目录下(如：E:\MavenWorks\babasport\target)；开发web项目时，还需要手动复制到web服务器下(Tomcat)

如果能自动部署到Web服务器，而不用每次手动把target下编译好的war包拷贝到Tomcat下就更好了。

对servers设置：

1. 修改发布路径到webapps下
2. 修改timeout服务器启动和停止时间为300秒
3. 发布到webapps下的根目录(项目访问URL不需要填写项目名， 取消勾选自动发布!
4. 浏览器访问 `http://localhost:8080/` (省略了项目名称)    

注：如果用eclipse，`http://localhost:8080/项目名称`  也可以访问； 

    用myeclipse，只能访问`http://localhost:8080/`

注：如果不能成功发布到 / 目录下，或许需要做以下修改(一般默认完成，不用做修改)   

- 1 删除webContext的发布
- 2 增加webapp发布到根目录下
- 3 增加`Maven库(jar包)到WEB-INF/lib 下`

### 四、控制台不输出log没有反应

MyEclipse中Maven build...项目控制台不输出log 没有反应

  	一开始项目与都是可以通过maven build... 然后输入tomcat:run 跑起来的，后来忘记做了啥操作跑步起来了，控制台也不输出log了，总是一闪就过去了，后来也是试了各种方法，可能是自己换了当前项目的jdk版本，下面是解决方案

右击maven项目: `Build Path ---> configure Build Path... ---> 先选中你当前的jdk remove掉 ，然后重新添加 ----> Add Library --->  选择 JRE System Library  ---> 选择Alternate JRE---> 然后点击installed JRES 进入界面后选择你要用的jdk版本 ---> edit ---> 在Default VM arguments 添加 :Dmaven.multiModuleProjectDirectory=$MAVEN_HOME`

这样就可以，希望可以解决你的问题 !
