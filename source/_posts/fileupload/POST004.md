---
title: 文件的上传和下载
tags: fileUpload
category: fileUpload
abbrlink: 64131
date: 2018-04-02 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0167.jpg)
文件的上传和下载
<!--more-->
### 文件的上传和下载绝对路径

#### 上传 

存的是绝对路径D:\tomct.......


```java
/**
  * @Description: 生成上传文件的文件名，文件名以时间+3位随机数+"_"+文件的原始名称
  * @param filename 文件的原始名称
  * @return 时间+3位随机数+"_"+文件的原始名称
  */
 private String makeFileName(String filename) { // 2.jpg
  // 为防止文件覆盖的现象发生，要为上传文件产生一个唯一的文件名
  // return UUID.randomUUID().toString() + "_" + filename;
  // 日期格式化
  DateFormat df = new SimpleDateFormat("yyyyMMddhhmmssSSS");
  // 当前的时间
  String name = df.format(new Date());

  // 三位的随机数
  Random r = new Random();
  // name=时间+三位的随机数
  for (int i = 0; i < 3; i++) {
   name += r.nextInt(10); // +=会自动的将int转换为string
  }
  return name + "_" + filename;
 }
```

```java
/**
  * 为防止一个目录下面出现太多文件，要使用hash算法打散存储
  *  使用"/" 支持linux系统
  * @param filename 文件名，要根据文件名生成存储目录
  * @param savePath 文件存储路径  
  * @return 新的存储目录
  */
  private String makePath(String filename,String savePath){
     //得到文件名的hashCode的值，得到的就是filename这个字符串对象在内存中的地址
     int hashcode = filename.hashCode();
     int dir1 = hashcode & 0xf; //0--15
     int dir2 = (hashcode & 0xf0)>>4; //0-15
     //构造新的保存目录
     String dir = savePath + "/" + dir1 + "/" + dir2+"/"; //upload\2\3 upload\3\5
     //File既可以代表文件也可以代表目录   
     File file = new File(dir);
    //如果目录不存在
     if(!file.exists()){
      //创建目录
      file.mkdirs();
    }
     return dir;
       String savePath = this.getServletContext().getRealPath("/WEB-INF/upload");
      File root = new File(savePath);
      if (!root.exists()) {
       //创建临时目录
       root.mkdir();
      }
         InputStream is = new FileInputStream(file); 
         //得到上传文件的扩展名 .xle
         String fileExtName = fileFileName.substring(fileFileName.lastIndexOf(".")+1);
      
         //得到文件的真实的保存目录
         String realSavePath = makePath(fileFileName, savePath);
         //修改文件的存储名字,防止文件多了出现重名保存失败
        String  storeName = makeFileName(fileFileName);
        
         //创建一个文件输出流
         OutputStream os = new FileOutputStream(realSavePath + "/" + storeName);
         
         byte[] buffer = new byte[1024];
         int length = 0;
         
         while(-1 != (length = is.read(buffer, 0, buffer.length)))
         {
           os.write(buffer);
         }
         os.close();
         is.close();
       
     //数据库存储的字段和名字 
     String  urlAndName = realSavePath +storeName ;
     notice.setAttachment(urlAndName);
     
        }catch (Exception e) {
      
          e.printStackTrace();
          return SUCCESS ; 
        } 
```

#### 下载

文件

```java
public String download() throws ServletException, Exception {
  HttpServletRequest req = ServletActionContext.getRequest();
  Long id = Long.parseLong(req.getParameter("id"));
  noticeQuery.setId(id);
  System.out.println();
  // 获取文件的全名和绝对磁盘地址
  Notice notice = noticeService.selectByPrimaryKey(noticeQuery.getId());
  // String urlAndName = kbm.getKbmAdjunct();
  String urlAndName = notice.getAttachment();
  // 处理获取到的上传文件的文件名的路径部分，只保留文件名部分
   String url = urlAndName.substring(0, urlAndName.lastIndexOf("/"));
         String storeName =  urlAndName.substring( urlAndName.lastIndexOf("/")+1);
         String realName = storeName.substring(storeName.indexOf("_")+1);
  // 上传的文件都是保存在/WEB-INF/upload目录下的子目录当中
  File file = new File(url + "/" + storeName);
  // 如果文件不存在
  if (!file.exists()) {
   // 错误提示
      this.getServletResponse().setCharacterEncoding("utf-8");
            this.getServletResponse().getWriter().write("您要下载的资源已被删除！！");
//   servletRequest.setAttribute("message", "您要下载的资源已被删除！！");
//   servletRequest.getRequestDispatcher("/message.jsp").forward(
//     servletRequest, servletResponse);
   return "";
  }

  // 处理文件名
  // String realname = fileStoreName.substring(fileStoreName.indexOf("_")
  // + 1);
  // 设置响应头，控制浏览器下载该文件
  servletResponse.setHeader("content-disposition", "attachment;filename="
    + URLEncoder.encode(realName, "UTF-8"));
  // 读取要下载的文件，保存到文件输入流
  FileInputStream in = new FileInputStream(file);
  // 创建输出流
  OutputStream out = servletResponse.getOutputStream();
  byte buffer[] = new byte[1024];
  int len = 0;
  // 循环将输入流中的内容读取到缓冲区当中
  while ((len = in.read(buffer)) > 0) {
   // 输出缓冲区的内容到浏览器，实现文件下载
   out.write(buffer, 0, len);
  }
  // 关闭文件输入流
  in.close();
  // 关闭输出流
  out.close();
  return NONE;

 }
```

### 文件上传和下载相对路径
#### 上传
存的是相对路径 `http://127.0.0.1/tomcat.....`

```java
public  String  upload  ()  {

File filePath = new File(ServletActionContext.getServletContext().getRealPath("/"));
      String savePath = filePath.getParentFile().getParent()+"/attachment/securitysupervision";
       File root = new File(savePath);
       if (!root.exists()) {
        //创建临时目录
        root.mkdirs();
        System.out.println( root.mkdir());
       }
          InputStream is = new FileInputStream(file);
          //得到上传文件的扩展名 .xls
          String fileExtName = fileFileName.substring(fileFileName.lastIndexOf(".")+1);
       
          //得到文件的真实的保存目录
          String realSavePath = servletRequest.getScheme() + "://"+ servletRequest.getServerName() + ":" + servletRequest.getServerPort()+servletRequest.getContextPath()+"/attachment/securitysupervision/";
          //String realSavePath = makePath(fileFileName, savePath);
          //修改文件的存储名字,防止文件多了出现重名保存失败
         String  storeName = makeFileName(fileFileName);
         
          //创建一个文件输出流
          OutputStream os = new FileOutputStream(savePath + "\\" + storeName);
          byte[] buffer = new byte[1024];
          int length = 0;
          while(-1 != (length = is.read(buffer, 0, buffer.length)))
          {
            os.write(buffer);
          }
          os.close();
          is.close();
        
      //数据库存储的字段和名字 
      String  urlAndName = realSavePath +storeName ;
      kbm.setKbmAdjunct(urlAndName);
      
         }catch (Exception e) {
       
           e.printStackTrace();
           return  "error" ; 
         }
}
```

#### 下载

文件

```java
Kbm kbm = kbmService.selectAddrByKbmId(kbmQuery.getId()) ;
   String urlAndName = kbm.getKbmAdjunct();
  if (urlAndName != null && !urlAndName.equals("")) {
    // 处理获取到的上传文件的文件名的路径部分，只保留文件名部分
    HttpServletRequest request=ServletActionContext.getRequest();
    String basePath=request.getScheme() + "://"+ request.getServerName() + ":" + request.getServerPort()+request.getContextPath();
   System.out.println(basePath);
    String url = urlAndName.substring(basePath.length()+1, urlAndName.lastIndexOf("/"));
   String storeName = urlAndName.substring(urlAndName.lastIndexOf("/") + 1);
   String realName = storeName.substring(storeName.indexOf("_") + 1);
   // 上传的文件都是保存在/WEB-INF/upload目录下的子目录当中
   //File file = new File(url + "" + storeName);
   File filePath = new File(ServletActionContext.getServletContext().getRealPath("/"));
    String savePath = filePath.getParentFile().getParent()+"/attachment/securitysupervision";
   System.out.println(savePath);
   File file = new File(savePath + "/" + storeName);
   
   // 如果文件不存在
   if (!file.exists()) {
    // 错误提示
    servletRequest.setAttribute("message", "您要下载的资源已被删除！！");
    servletRequest.getRequestDispatcher("/message.jsp").forward(
      servletRequest, servletResponse);
    return "";
   }

   // 处理文件名
   // String realname =
   // fileStoreName.substring(fileStoreName.indexOf("_") + 1);
   // 设置响应头，控制浏览器下载该文件
   servletResponse.setHeader(
     "content-disposition",
     "attachment;filename="
       + URLEncoder.encode(realName, "UTF-8"));
   // 读取要下载的文件，保存到文件输入流
   FileInputStream in = new FileInputStream(file);
   // 创建输出流
   OutputStream out = servletResponse.getOutputStream();
   byte buffer[] = new byte[1024];
   int len = 0;
   // 循环将输入流中的内容读取到缓冲区当中
   while ((len = in.read(buffer)) > 0) {
    // 输出缓冲区的内容到浏览器，实现文件下载
    out.write(buffer, 0, len);
   }
   // 关闭文件输入流
   in.close();
   // 关闭输出流
   out.close();
    return NONE;
```
