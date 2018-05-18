---
title: Java NIO Path and Files
tags: NIO
category: NIO
abbrlink: 58642
date: 2018-03-23 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0162.jpg)

Java NIO Path接口和Files类配合操作文件 
<!--more-->
## Path接口
------
1、Path表示的是一个目录名序列，其后还可以跟着一个文件名，路径中第一个部件是根部件时就是绝对路径，例如 / 或 C:\ ，而允许访问的根部件取决于文件系统；

2、以根部件开始的路径是绝对路径，否则就是相对路径；

3、静态的**Paths.get**方法接受一个或多个字符串，字符串之间**自动使用**默认文件系统的路径分隔符连接起来（Unix是 /，Windows是 \ ），这就解决了跨平台的问题，接着解析连接起来的结果，如果不是合法路径就抛出InvalidPathException异常，否则就返回一个Path对象；

```java
 //假设是Unix的文件系统
 Path absolute = Paths.get("/home", "cat"); //绝对路径 
 Path relative = Pahts.get("ixenos", "config", "user.properties"); //相对路径
```

4、由String路径获取Path对象

**get还可以获取一整条路径**（即多个部件构成的单个字符串），例如从配置文件中读取路径：

```java
String baseDir = properties.getProperty("base.dir");
 //可能获得 /opt/ixenos 或者 C:\Program Files\ixenos
 Path basePath = Paths.get(baseDir);
```

5、组合或解析路径

　　1) 调用 **p.resolve(q)** 将按下面的规则返回一个Path：如果q是绝对路径，则返回q，否则**追加路径**返回 p/q 或者 p\q

```java
 Path workRelative = Paths.get("work");
 Path workPath = basePath.resolve(workRelative);
  //resolve也可以接受字符串形参
 Path workPath = basePath.resolve("work");
```

　　2) 调用 **p.resolveSibling("q")** 将解析指定路径 p 的父路径 o ，并**产生兄弟路径** o/q

```java
Path tempPath = workPath.resolveSibling("temp");
 /*
   如果workPath是 /opt/ixenos/work
    那么将创建 /opt/ixenos/temp  
 */
```

　　3) 调用 **p.relativize(r)** 将产生一个冗余路径q，对q进行解析将产生**相对路径r，**最终r不包含和p的交集路径

```java
/*
    pathA为 /home/misty
    pathB为 /home/ixenos/config 

    现已pathA对pathB进行相对化操作，将产生冗余路径
*/
Path pathC = pathA.relativize(pathB); //此时pathC为 ../ixenos/config

/*
    normalize方法将移除冗余部件
*/
Path pathD = pathC.normalize(); //pathD为 /ixenos/config
```



　　4) **toAbsolutePath** 将产生给定路径的**绝对路径**，从根部件开始

　　5) Path类还有一些有用的断开和组合路径的方法，比如 **getParent**、**getFileName**、**getRoot**//获得根目录

　　6) Path有个**toFile**方法用来跟**遗留类File类**打交道，File类也有个toPath方法

## Files工具类

------

### 1、**读写文件**

方法签名:

```java
  static path **write**(Path path, byte[] bytes, OpenOption... options)
　static path write(Path path, Iterable<? extends CharSequence> lines, OpenOption... options)
```

这里只列举下面用到的方法，更多方法请看API文档...

**其中OpenOption是个nio接口，StandardOpenOption是其枚举实现类，各枚举实例功能请查看API文档**

```java
/*
    Files提供的简便方法适用于处理中等长度的文本文件
    如果要处理的文件长度较大，或者二进制文件，那么还是应该使用经典的IO流 
*/
//将文件所有内容读入byte数组中
byte[] bytes = Files.readAllBytes(path); //传入Path对象

//之后可以根据字符集构建字符串
String content = new String(bytes, charset);

//也可以直接当作行序列读入
List<String> lines = Files.readAllLines(path, charset);

//相反，也可以写一个字符串到文件中，默认是覆盖
Files.write(path, content.getBytes(charset)); //传入byte[]

//追加内容，根据参数决定追加等功能
Files.write(path, content.getBytes(charset), StandardOpenOption.APPEND); //传入枚举对象，打开追加开关

//将一个行String的集合List写出到文件中
Files.write(path, lines);
```

###  **2、复制、剪切、删除**

方法签名:

```java
　	static path copy(Path source, Path target, CopyOption... options)
　　static path move(Path source, Path target, CopyOption... options)
	static void delete(Path path) //如果path不存在文件将抛出异常，此时调用下面的比较好
　　static boolean deleteIfExists(Path path)
```

　　[这里只列举下面用到的方法，更多方法请看API文档...]()

**其中CopyOption是个nio接口，StandardCopyOption是其枚举实现类，各枚举实例功能请查看API文档**

　　其中有个**ATOMIC_MOVE可以填入用来保证原子性操作**，要么移动成功完成，要么源文件保持在原位置

```java
//复制
Files.copy(fromPath, toPath);
//剪切
Files.move(fromPath, toPath);
/*
    以上如果toPath已存在，那么操作失败，
    如果要覆盖，需传入参数REPLACE_EXISTING
    还要复制文件属性，传入COPY_ATTRIBUTES
*/
Files.copy(fromPath, toPath, StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.COPY_ATTRIBUTES);
```

###  **3、创建文件和目录**

```java
//创建新目录，除了最后一个部件，其他必须是已存在的
Files.createDirectory(path); 
//创建路径中的中间目录，能创建不存在的中间部件
Files.createDirectories(path);
/*
   创建一个空文件，检查文件存在，如果已存在则抛出异常
   而检查文件存在是原子性的，因此在此过程中无法执行文件创建操作
*/
Files.createFile(path);
//添加前/后缀创建临时文件或临时目录
Path newPath = Files.createTempFile(dir, prefix, suffix);
Path newPath = Files.createTempDirectory(dir, prefix);
```

### **4、获取文件信息**

略，具体看API文档，或者corejava page51

### 5、迭代目录中的文件

　　**旧的File类**有两个方法获取目录中所有文件构成的字符串数组，String[] list() 和String[] list(FileFilter filter)，但是**当目录中包含大量文件时，这两方法性能会非常低。**

**原因分析：**

```java
//File类list所有文件
    public String[] list() {
        SecurityManager security = System.getSecurityManager(); //文件系统权限获取
        if (security != null) {
            security.checkRead(path);
        }
        if (isInvalid()) {
            return null;
        }
        return fs.list(this); //底层调用FileSystem的list
    }

  //FileSystem抽象类的list
 //File类中定义fs是由DefaultFileSystem静态生成的
private static final FileSystem fs = DefaultFileSystem.getFileSystem();

//因此我们来看一下DefaultFileSystem类，发现是生成一个WinNtFileSystem对象
class DefaultFileSystem {

    /**
     * Return the FileSystem object for Windows platform.
     */
    public static FileSystem getFileSystem() {
        return new WinNTFileSystem();
    }
}
//而WinNtFileSystem类继承于FileSystem抽象类，这里我们主要观察它的list(File file)方法
    @Override
public native String[] list(File f);
/*我们可以看到这是个native方法，说明list的操作是由操作系统的文件系统控制的，当目录中包含大量的文件时，这个方法的性能将会非常低。
由此为了替代，NIO的Files类设计了newDirectoryStream(Path dir)及其重载方法，将生成Iterable对象（可用foreach迭代）*///~


 //回调过滤
    public String[] list(FilenameFilter filter) { //采用接口回调
        String names[] = list(); //调用list所有
        if ((names == null) || (filter == null)) {
            return names;
        }
        List<String> v = new ArrayList<>();
        for (int i = 0 ; i < names.length ; i++) {
            if (filter.accept(this, names[i])) {  //回调FilenameFileter对象的accept方法
                v.add(names[i]);
            }
        }
        return v.toArray(new String[v.size()]);
    }
```

###  这时候高科技来了——Files获得可迭代的目录流，

####  传入一个目录Path，遍历子孙目录返回一个目录Path的Stream，注意这里所有涉及的Path都是目录而不是文件！

因此，Files类设计了**newDirectoryStream(Path dir)**及其重载方法，将生成Iterable对象（可用foreach迭代）

遍历目录得到一个可迭代的子孙文件集合

| `staticDirectoryStream<Path>` | `newDirectoryStream(Path dir)`Opens a directory, returning a [`DirectoryStream`]() to iterate over all entries in the directory. |
| ----------------------------- | ------------------------------------------------------------ |
| `staticDirectoryStream<Path>` | `newDirectoryStream(Path dir, DirectoryStream.Filter<? superPath> filter)`Opens a directory, returning a [`DirectoryStream`]() to iterate over the entries in the directory. |
| `staticDirectoryStream<Path>` | `newDirectoryStream(Path dir, String glob)`                  |

　　返回一个 **目录流 ，可以看成一个存放着全部Path的实现了Iterable的集合**，

　　　　因此可用迭代器或foreach迭代，只是使用迭代器的时候要注意不能invoke另一个Iterator：

- - **While DirectoryStream extends Iterable, it is not a general-purpose Iterable as it supports only a single Iterator; invoking the iterator method to obtain a second or subsequent iterator throws IllegalStateException.** 

示例：

```java
try(DirectoryStream<Path> entries = Files.newDirectoryStream(dir))
{
    for(Path entry : entries)
    {
         ...
    }
}
```


　　可以传入glob参数，即使用**glob模式**来**过滤文件**（以取代`list(FileFilter filter)：newDirectoryStream(Path dir, String glob)` 注意是String类型

```java
 try(DirectoryStream<Path> entries = Files.newDirectoryStream(dir, "*.java")) //
 {
     ...
 }
```

　　**glob模式**

所谓的 glob 模式是指 shell 所使用的简化了的**正则表达式**。

1.星号 * 匹配**路径组成部分**0个或多个字符；例如 *.java 匹配**当前目录中**的所有Java文件

2.两星号 ** 匹配**跨目录边界**0个或多个字符；例如 **.java 匹配在**所有子目录中**的Java文件

3.问号（?）只匹配一个字符；例如 ????.java 匹配所有**四个字符**的Java文件，不包括扩展名；使用?是因为*是通配符不指定数量

4.[...] 匹配**一个**字符集合，可以用连线 [0-9] 和取反符 [!0-9]；例如 Test[0-9A-F].java 匹配Testx.java，假设x是一个十六进制数字，[0-9A-F]是匹配单个字符为十六进制数字，比如B（十六进制不区分大小写）

　　*如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）*。

5.{...} 匹配由逗号隔开的**多个可选项**之中的一个；例如 *.{java,class} 匹配所有Java文件和类class文件

6.\ 转义上述任意模式中的字符；例如 *\** 匹配所有子目录中文件名包含*的文件，这里为 ** 转义，前面是匹配0个或多个字符


下面是网友总结的Glob模式：

| Glob模式     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| *.txt        | 匹配所有扩展名为.txt的文件                                   |
| *.{html,htm} | 匹配所有扩展名为.html或.htm的文件。{ }用于组模式，它使用逗号分隔 |
| ?.txt        | 匹配任何单个字符做文件名且扩展名为.txt的文件                 |
| *.*          | 匹配所有含扩展名的文件                                       |
| C:\Users\*   | 匹配所有在C盘Users目录下的文件。反斜线“\”用于对紧跟的字符进行转义 |
| /home/**     | UNIX平台上匹配所有/home目录及子目录下的文件。**用于匹配当前目录及其所有子目录 |
| [xyz].txt    | 匹配所有单个字符作为文件名，且单个字符只含“x”或“y”或“z”三种之一，且扩展名为.txt的文件。方括号[]用于指定一个集合 |
| [a-c].txt    | 匹配所有单个字符作为文件名，且单个字符只含“a”或“b”或“c”三种之一，且扩展名为.txt的文件。减号“-”用于指定一个范围，且只能用在方括号[]内 |
| [!a].txt     | 匹配所有单个字符作为文件名，且单个字符不能包含字母“a”，且扩展名为.txt的文件。叹号“!”用于否定 |

### 遍历得到某个目录的所有子孙文件集合再迭代不够爽？来，我们来直接遍历某个目录的所有子孙成员（包括目录和文件）

　　我们可以调用Files类的**walkFileTree**方法，并传入一个FileVisitor接口类型的对象（还有更多方法在API里等你发现……）

```java
/*传入一个FileVisitor子类的匿名对象*/
Files.walkFileTree(dir, new SimpleFileVisitor<Path>()
    {
         //walkFileTree回调此方法来遍历所有子孙
         public FileVisitResult visitFile(Path path, BasicFileAttributes attrs) throws IOException
         {
              if(attrs.isDirectory()) //自定义的选择，属于业务代码，这和walkFileTree的宗旨(遍历所有子孙成员)无关
                  System.out.println(path);
              return FileVisitResult.CONTINUE;
         }

         public FileVisitResult visitFileFailed(Path path, IOException exc) throws IOException
         {
              return FileVisitResult.CONTINUE;
         }
    });
```

#### 咱们来总结一下，

```java
Files.newDirectoryStream(Path dir) //遍历后返回一个可迭代的子孙文件集合；
Files.walkFileTree(Path dir, FileVisitor fv) //是一个遍历子孙目录和文件的过程；
```

## ZIP文件系统

------

由上文知道，Paths类会在**默认的文件系统**中查找路径，即在用户本地磁盘中的文件。

其实，我们也可以有其他的文件系统，比如**ZIP文件系统**。

```java
 /*假设zipname是某个ZIP文件的名字*/
FileSystem fs = FileSystems.newFileSystem(Paths.get(zipname), null);
```

 上述代码将建立一个**基于zipname的文件系统，它包含ZIP文档中的所有文件**。

　　1）如果知道**文件名（String类型）**，那么从这个ZIP文档中复制出这个文件就很容易：

```java
 Files.copy(fs.getPath(fileName), targetPath);
```

　　　　Q：fs.getPath是使用了ZIP文件系统来getPath，那么默认的文件系统能调用吗？

　　　　A：能。FileSystem类中有一个静态的getDefault()方法，返回一个默认的文件系统对象，同样可以由文件名getPath。

　　　　　　*具体getPath(String name)是遍历还是随机访问，有空再去看源码实现。
　　2）要列出ZIP文档中的所有文件，同样可以用walkFileTree遍历文件树

```java
FileSystem fs = FileSystems.newFileSystem(Paths.get(fileName), null);

//walkFileTree需要传入一个要被遍历的目录Path，和一个FileVisitor对象
Files.walkFileTree(fs.getPath("/"), 
        newSimpleFileVisitor<Path>(){
               public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws Exception{
                     System.out.println(file);
                     return FileVisitResult.CONTINUE; 
        });
```
