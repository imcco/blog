---
title: Java 文件上传及下载
tags: fileupload
category: fileupload
abbrlink: 25709
date: 2018-03-24 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0163.jpg)

Java 文件上传及下载
<!--more-->
### 文件上传

　　文件上传操作通常会附加一些限制，如：文件类型、上传文件总大小、每个文件的最大大小等。除此以外，作为一个通用组件还需要考虑更多的问题，如：支持自定义文件保存目录、支持相对路径和绝对路径、支持自定义保存的文件的文件名称、支持上传进度反馈和上传失败清理等。另外，本座也不想重新造车轮，本组件是基于 Commons File Upload 实现，省却了本座大量的工作 ^_^ 下面先从一个具体的使用例子讲起：

#### 上传请求界面及代码


![img](http://filesimg.111cn.net/2012/02/24/20120224013306316.jpg)

 

```html
<form action="checkupload.action" method="post" enctype="multipart/form-data">
    First Name: <input type="text" name="firstName" value="丑">
    <br>
    Last Name: <input type="text" name="lastName" value="怪兽">
    <br>
    Birthday: <input type="text" name="birthday" value="1978-11-03">
    <br>
    Gender: 男 <input type="radio"" name="gender" value="false">
        &nbsp;女 <input type="radio"" name="gender" value="true" checked="checked">
    <br>
    Working age: <select name="workingAge">
        <option value="-1">-请选择-</option>
        <option value="3">三年</option>
        <option value="5" selected="selected">五年</option>
        <option value="10">十年</option>
        <option value="20">二十年</option>
    </select>
    <br>
    Interest: 游泳 <input type="checkbox" name="interest" value="1" checked="checked">
        &nbsp;打球 <input type="checkbox" name="interest" value="2" checked="checked">
        &nbsp;下棋 <input type="checkbox" name="interest" value="3">
        &nbsp;打麻将 <input type="checkbox" name="interest" value="4">
        &nbsp;看书 <input type="checkbox" name="interest" value="5" checked="checked">
    <br>
    Photo 1.1: <input type="file"" name="photo-1">
    <br>
    Photo 1.2: <input type="file"" name="photo-1">
    <br>
    Photo 2.1: <input type="file"" name="photo-2">
    <br>
    <br>
    <input type="submit" value="确 定">&nbsp;&nbsp;<input type="reset" value="重 置">
</form>
```
从上面的 HTML 代码可以看出，表单有 6 个普通域和 3 个文件域，其中前两个文件域的 name 属性相同。

#### 上传处理代码

```java
import com.bruce.util.BeanHelper;
import com.bruce.util.Logger;
import com.bruce.util.http.FileUploader;
import com.bruce.util.http.FileUploader.FileInfo;
import static com.bruce.util.http.FileUploader.*;
@SuppressWarnings("unused")
public class CheckUpload extends ActionSupport
{
    // 上传路径
    private static final String UPLOAD_PATH        = "upload";
    // 可接受的文件类型
    private static final String[] ACCEPT_TYPES    = {"txt", "pdf", "doc", ".Jpg", "*.zip", "*.RAR"};
    // 总上传文件大小限制
    private static final long MAX_SIZE            = 1024 * 1024 * 100;
    // 单个传文件大小限制
    private static final long MAX_FILE_SIZE        = 1024 * 1024 * 10;
    
    @Override
    public String execute()
    {
        // 创建 FileUploader 对象
        FileUploader fu = new FileUploader(UPLOAD_PATH, ACCEPT_TYPES, MAX_SIZE, MAX_FILE_SIZE);
        
        // 根据实际情况设置对象属性（可选）
        /*
        fu.setFileNameGenerator(new FileNameGenerator()
        {
            
            @Override
            public String generate(FileItem item, String suffix)
            {
                return String.format("%d_%d", item.hashCode(), item.get().hashCode());
            }
        });
        
        fu.setServletProgressListener(new ProgressListener()
        {
            
            @Override
            public void update(long pBytesRead, long pContentLength, int pItems)
            {
                System.out.printf("%d: length -> %d, read -> %d.n", pItems, pContentLength, pBytesRead);
            }
        });
        */
        
        // 执行上传并获取操作结果
        Result result = fu.upload(getRequest(), getResponse());
        
        // 检查操作结果
        if(result != FileUploader.Result.SUCCESS)
        {
            // 设置 request attribute
            setRequestAttribute("error", fu.getCause());
            // 记录日志
            Logger.exception(fu.getCause(), "upload file fail", Level.ERROR, false);
            
            return ERROR;
        }
        
        // 通过非文件表单域创建 Form Bean
        Persion persion = BeanHelper.createBean(Persion.class, fu.getParamFields());
        // 图片保存路径的列表
        List<String> photos    = new ArrayList<String>();
        
        /* 轮询文件表单域，填充 photos */
        Set<String> keys = fu.getFileFields().keySet();
        for(String key : keys)
        {
            FileInfo[] ffs = fu.getFileFields().get(key);
            for(FileInfo ff : ffs)
            {
                photos.add(String.format("(%s) %s%s%s", key, fu.getSavePath(), File.separator, ff.getSaveFile().getName()));
            }
        }
        
        // 设置 Form Bean 的 photos 属性
        persion.setPhotos(photos);
        // 设置 request attribute
        setRequestAttribute("persion", persion);
        
        return SUCCESS;
    }
}
 
public class Persion
{
    private String firstName;
    private String lastName;
    private Date birthday;
    private boolean gender;
    private int workingAge;
    private int[] interest;
    private List<String> photos;
}
 
public static class FileInfo
{
    private String uploadFileName;
    private File saveFile;
}
```

　　分析下上面的 Java 代码，本例先根据保存目录、文件大小限制和文件类型限制创建一个 FileUploader 对象，然后调用该对象的 upload() 方法执行上传并返回操作结果，如果上传成功则 通过 getParamFields() 方法获取所有非文件表单域内容，并交由 BeanHelper 进行解析创建 Form Bean，再调用 getFileFields() 方法获取所有文件表单域的 FileInfo（FileInfo 包含上传文件的原始名称和被保存文件的 File 对象），最后完成 Form Bean 所有字段的填充并把 Form Bean 设置为 request 属性。

#### 上传结果界面及代码

![img](http://filesimg.111cn.net/2012/02/24/20120224013312738.jpg)

```html
<table border="1">
<caption>Persion Attributs</caption>

    <tr><td>Name</td><td><c:out value="${persion.firstName} ${persion.lastName}"/>&nbsp;</td></tr>
    <tr><td>Brithday</td><td><c:out value="${persion.birthday}"/>&nbsp;</td></tr>
    <tr><td>Gender</td><td><c:out value="${persion.gender}"/>&nbsp;</td></tr>
    <tr><td>Working Age</td><td><c:out value="${persion.workingAge}"/>&nbsp;</td></tr>
    <tr><td>Interest</td><td><c:forEach var="its" items="${persion.interest}">
                                 <c:out value="${its}" /> &nbsp;
                          </c:forEach>&nbsp;</td></tr>
    <tr><td>Photos</td><td><c:forEach var="p" items="${persion.photos}">
                                 <c:out value="${p}" /><br>
                          </c:forEach>&nbsp;</td></tr>
</table>
```

 　　从上面的处理结果可以看出，文件上传组件 FileUploader 正确地处理了表单的所有文件域和非文件域名，并且，整个文件上传操作过程非常简单，无需用户过多参与。下面我们来详细看看组件的主要实现代码：

```java
public class FileUploader
{
    / 不限制文件上传总大小的 Size Max 常量 */
    public static final long NO_LIMIT_SIZE_MAX        = -1;
    / 不限制文件上传单个文件大小的 File Size Max 常量 */
    public static final long NO_LIMIT_FILE_SIZE_MAX    = -1;
    / 默认的写文件阀值 */
    public static final int DEFAULT_SIZE_THRESHOLD    = DiskFileItemFactory.DEFAULT_SIZE_THRESHOLD;
    / 默认的文件名生成器 */
    public static final FileNameGenerator DEFAULT_FILE_NAME_GENERATOR = new CommonFileNameGenerator();
    
    / 设置上传文件的保存路径（不包含文件名）
     * 
     * 文件路径，可能是绝对路径或相对路径<br>
     *     1) 绝对路径：以根目录符开始（如：'/'、'D:'），是服务器文件系统的路径<br>
     *     2) 相对路径：不以根目录符开始，是相对于 WEB 应用程序 Context 的路径，（如：mydir 是指
     *         '${WEB-APP-DIR}/mydir'）<br>
     *     3) 规则：上传文件前会检查该路径是否存在，如果不存在则会尝试生成该路径，如果生成失败则
     *         上传失败并返回 {@link Result#INVALID_SAVE_PATH}
     * 
     */
    private String savePath;
    / 文件上传的总文件大小限制 */
    private long sizeMax                        = NO_LIMIT_SIZE_MAX;
    / 文件上传的单个文件大小限制 */
    private long fileSizeMax                    = NO_LIMIT_FILE_SIZE_MAX;
    / 可接受的上传文件类型集合，默认：不限制 */
    private Set<String> acceptTypes                = new LStrSet();
    / 非文件表单域的映射 */
    private Map<String, String[]> paramFields    = new HashMap<String, String[]>();
    / 文件表单域的映射 */
    private Map<String, FileInfo[]> fileFields    = new HashMap<String, FileInfo[]>();
    / 文件名生成器 */
    private FileNameGenerator fileNameGenerator    = DEFAULT_FILE_NAME_GENERATOR;
    
    // commons file upload 相关属性
    private int factorySizeThreshold            = DEFAULT_SIZE_THRESHOLD;
    private String factoryRepository;
    private FileCleaningTracker factoryCleaningTracker;
    private String servletHeaderencoding;
    private ProgressListener servletProgressListener;
    
    / 文件上传失败的原因（文件上传失败时使用） */
    private Throwable cause;
    
    / 执行上传
     * 
     * @param request    : {@link HttpServletRequest} 对象
     * @param response    : {@link HttpServletResponse} 对象
     * 
     * @return            : 成功：返回 {@link Result#SUCCESS} ，失败：返回其他结果，
     *                       失败原因通过 {@link FileUploader#getCause()} 获取
     * 
     */
    @SuppressWarnings("unchecked")
    public Result upload(HttpServletRequest request, HttpServletResponse response)
    {
        reset();

        // 获取上传目录绝对路径
        String absolutePath     = getAbsoluteSavePath(request);
        if(absolutePath == null)
        {
            cause = new FileNotFoundException(String.format("path '%s' not found or is not directory", savePath));
            return Result.INVALID_SAVE_PATH;
        }
        
        ServletFileUpload sfu    = getFileUploadComponent();
        List<FileItemInfo> fiis    = new ArrayList<FileItemInfo>();
        
        List<FileItem> items    = null;
        Result result            = Result.SUCCESS;
        
        // 获取文件名生成器
        String encoding                    = servletHeaderencoding != null ? servletHeaderencoding : request.getCharacterEncoding();
        FileNameGenerator fnGenerator    = fileNameGenerator != null ? fileNameGenerator : DEFAULT_FILE_NAME_GENERATOR;
        
        try
        {
            // 执行上传操作
            items = (List<FileItem>)sfu.parseRequest(request);
        }
        catch (FileUploadException e)
        {
            cause = e;
            
            if(e instanceof FileSizeLimitExceededException)        result = Result.FILE_SIZE_EXCEEDED;
            else if(e instanceof SizeLimitExceededException)    result = Result.SIZE_EXCEEDED;
            else if(e instanceof InvalidContentTypeException)    result = Result.INVALID_CONTENT_TYPE;
            else if(e instanceof IOFileUploadException)            result = Result.FILE_UPLOAD_IO_EXCEPTION;
            else                                                result = Result.OTHER_PARSE_REQUEST_EXCEPTION;
        }
        
        if(result == Result.SUCCESS)
        {
            // 解析所有表单域
            result = parseFileItems(items, fnGenerator, absolutePath, encoding, fiis);    
            if(result == Result.SUCCESS)
                // 保存文件
                result = writeFiles(fiis);
        }
        
        return result;
    }

    // 解析所有表单域
    private Result parseFileItems(List<FileItem> items, FileNameGenerator fnGenerator, String absolutePath, String encoding, List<FileItemInfo> fiis)
    {
        for(FileItem item : items)
        {
            if(item.isFormField())
                // 解析非文件表单域
                parseFormField(item, encoding);
            else
            {
                if(item.getSize() == 0)
                    continue;
                
                // 解析文件表单域
                Result result = parseFileField(item, absolutePath, fnGenerator, fiis);
                if(result != Result.SUCCESS)
                {
                    reset();
                    
                    cause = new InvalidParameterException(String.format("file '%s' not accepted", item.getName()));
                    return result;
                }
            }
        }
        
        return Result.SUCCESS;
    }

    // 解析文件表单域
    private Result parseFileField(FileItem item, String absolutePath, FileNameGenerator fnGenerator, List<FileItemInfo> fiis)
    {
        String suffix            = null;
        String uploadFileName    = item.getName();
        boolean isAcceptType    = acceptTypes.isEmpty();
        
        if(!isAcceptType)
        {
            suffix = null;
            int stuffPos = uploadFileName.lastIndexOf(".");
            if(stuffPos != -1)
            {
                suffix = uploadFileName.substring(stuffPos, uploadFileName.length()).toLowerCase();
                isAcceptType = acceptTypes.contains(suffix);
            }
        }
        
        if(!isAcceptType)
            return Result.INVALID_FILE_TYPE;
        
        // 通过文件名生成器获取文件名
        String saveFileName = fnGenerator.generate(item, suffix);
        if(!saveFileName.endsWith(suffix))
            saveFileName += suffix;
        
        String fullFileName    = absolutePath + File.separator + saveFileName;
        File saveFile        = new File(fullFileName);
        FileInfo info        = new FileInfo(uploadFileName, saveFile);
        
        // 添加表单域文件信息
        fiis.add(new FileItemInfo(item, saveFile));
        addFileField(item.getFieldName(), info);
        
        return Result.SUCCESS;
    }

    private void parseFormField(FileItem item, String encoding)
    {
        String name = item.getFieldName();
        String value = item.getString();
        
        // 字符串编码转换
        if(!GeneralHelper.isStrEmpty(value) && encoding != null)
        {
            try
            {
                value = new String(value.getBytes("ISO-8859-1"), encoding);
            }
            catch(UnsupportedEncodingException e)
            {
    
            }
        }
        
        // 添加表单域名/值映射
        addParamField(name, value);
    }

    / 文件名生成器接口
     * 
     * 每次保存一个上传文件前都需要调用该接口的 {@link FileNameGenerator#generate} 方法生成要保存的文件名
     *  
     */
    public static interface FileNameGenerator
    {
        / 文件名生成方法
         * 
         * @param item        : 上传文件对应的 {@link FileItem} 对象
         * @param suffix    : 上传文件的后缀名
         * 
         */
        String generate(FileItem item, String suffix);
    }
    
    / 默认通用文件名生成器
     * 
     * 实现 {@link FileNameGenerator} 接口，根据序列值和时间生成唯一文件名
     *  
     */
    public static class CommonFileNameGenerator implements FileNameGenerator
    {
        private static final int MAX_SERIAL            = 999999;
        private static final AtomicInteger atomic = new AtomicInteger();
        
        private static int getNextInteger()
        {
            int value = atomic.incrementAndGet();
            if(value >= MAX_SERIAL)
                atomic.set(0);
            
            return value;
        }
        
        / 根据序列值和时间生成 'XXXXXX_YYYYYYYYYYYYY' 格式的唯一文件名 */
        @Override
        public String generate(FileItem item, String suffix)
        {
            int serial        = getNextInteger();
            long millsec    = System.currentTimeMillis();

            return String.format("%06d_%013d", serial, millsec);
        }
    }
    
    / 上传文件信息结构体 */
    public static class FileInfo
    {
        private String uploadFileName;
        private File saveFile;
        
        // getters and setters ...
    }
    
    private class FileItemInfo
    {
        FileItem item;
        File file;
        
        // getters and setters ...
    }
    
    / 文件上传结果枚举值 */
    public static enum Result
    {
        / 成功 */
        SUCCESS,
        / 失败：文件总大小超过限制 */
        SIZE_EXCEEDED,
        / 失败：单个文件大小超过限制 */
        FILE_SIZE_EXCEEDED,
        / 失败：请求表单类型不正确 */
        INVALID_CONTENT_TYPE,
        / 失败：文件上传 IO 错误 */
        FILE_UPLOAD_IO_EXCEPTION,
        / 失败：解析上传请求其他异常 */
        OTHER_PARSE_REQUEST_EXCEPTION,
        / 失败：文件类型不正确 */
        INVALID_FILE_TYPE,
        / 失败：文件写入失败 */
        WRITE_FILE_FAIL,
        / 失败：文件的保存路径不正确 */
        INVALID_SAVE_PATH;    
    }
}
```

- 应用可以实现自己的 FileNameGenerator 类替代默认的文件名生成器。

- 上传操作通过 FileUploader.Result 返回结果，并没有采用抛出异常的方式，因为本座认为在这里采用异常方式报告结果其实并不方便使用；另一方面，程序可以通过 getCause() 获取详细的错误信息。


### 文件下载

　　相对于文件上传，文件下载则简单很多，主要实现流程是根据文件名找到实际文件，并利用 Java 的相关类对 I/O 流进行读写。下面先看看一个使用示例：

```java
import static com.bruce.util.http.FileDownloader.*;

public class TestDownload extends ActionSupport
{
    // 绝对路径
    private static final String ABSOLUTE_PATH    = "/Server/apache-tomcat-6.0.32/webapps/portal/download/下载测试 - 文本文件.txt";
    // 相对路径
    private static final String RELATE_PATH        = "download/下载测试 - 项目框架.jpg";
    
    @Override
    public String execute()
    {
        int type            = getIntParam("type", 1);
        String filePath        = (type == 1 ? ABSOLUTE_PATH : RELATE_PATH);
        
        // 创建 FileDownloader 对象
        FileDownloader fdl    = new FileDownloader(filePath);
        // 执行下载
        Result result = fdl.downLoad(getRequest(), getResponse());
        // 检查下载结果
        if(result != Result.SUCCESS)
        {
            // 记录日志
            Logger.exception(fdl.getCause(), String.format("download file '%s' fail", fdl.getFilePath()), Level.ERROR, false);
        }
        
        return NONE;
    }
}
```
从这个示例可以看出，文件下载组件的使用方法更简单，因为它不需要对下载结果进行很多处理。可以看出该组件也支持相对路径和绝对路径。下面我们来详细看看组件的主要实现代码：

```java
/ 文件下载器 */
public class FileDownloader
{
    / 默认字节交换缓冲区大小 */
    public static final int DEFAULT_BUFFER_SIZE        = 1024 * 4;
    / 下载文件的默认 Mime Type */
    public static final String DEFAULT_CONTENT_TYPE    = "application/force-download";
    
    / 设置下载文件的路径（包含文件名） 
     * 
     * filePath    : 文件路径，可能是绝对路径或相对路径<br>
     *                 1) 绝对路径：以根目录符开始（如：'/'、'D:'），是服务器文件系统的路径<br>
     *                 2) 相对路径：不以根目录符开始，是相对于 WEB 应用程序 Context 的路径，（如：mydir/myfile 是指 '${WEB-APP-DIR}/mydir/myfile'）
     */
    private String filePath;
    / 显示在浏览器的下载对话框中的文件名称，默认与  filePath 参数中的文件名一致 */
    private String saveFileName;
    / 下载文件的 Mime Type，默认：{@link FileDownloader#DEFAULT_CONTENT_TYPE} */
    private String contentType        = DEFAULT_CONTENT_TYPE;
    / 字节缓冲区大小，默认：{@link FileDownloader#DEFAULT_CONTENT_TYPE} */
    private int bufferSize            = DEFAULT_BUFFER_SIZE;
    / 获取文件下载失败的原因（文件下载失败时使用） */
    private Throwable cause;
    
    / 执行下载
     * 
     * @param request    : {@link HttpServletRequest} 对象
     * @param response    : {@link HttpServletResponse} 对象
     * 
     * @return            : 成功：返回 {@link Result#SUCCESS} ，失败：返回其他结果，
     *                       失败原因通过 {@link FileDownloader#getCause()} 获取
     * 
     */
    public Result downLoad(HttpServletRequest request, HttpServletResponse response)
    {
        reset();
        
        try
        {
            // 获取要下载的文件的 File 对象
            File file = getFile(request);
            // 执行下载操作
            downLoadFile(request, response, file);
        }
        catch(Exception e)
        {
            cause = e;
            
            if(e instanceof FileNotFoundException)    return Result.FILE_NOT_FOUND;
            if(e instanceof IOException)            return Result.READ_WRITE_FAIL;
                                                    return Result.UNKNOWN_EXCEPTION;
        }
        
        return Result.SUCCESS; 
    }

    // 执行下载操作
    private void downLoadFile(HttpServletRequest request, HttpServletResponse response, File file) throws IOException
    {
        String fileName            = new String(saveFileName.getBytes(), "ISO-8859-1");
        // 解析 HTTP 请求头，获取文件的读取范围
        Range<Integer> range    = parseDownloadRange(request);    
        int length                = (int)file.length();
        int begin                = 0;
        int end                    = length - 1;
        
        // 设置 HTTP 响应头
        response.setContentType(contentType);
        response.setContentLength(length);
        response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
        
        // 确定文件的读取范围（用于断点续传）
        if(range != null)
        {
            if(range.getBegin()    != null)
            {
                begin = range.getBegin();
                if(range.getEnd() != null)
                    end = range.getEnd();
            }
            else
            {
                if(range.getEnd() != null)
                    begin = end + range.getEnd() + 1;
            }
            
            String contentRange = String.format("bytes %d-%d/%d", begin, end, length);
            response.setHeader("Accept-Ranges", "bytes");
            response.setHeader("Content-Range", contentRange);
            response.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);
        }
        
        // 实际执行下载操作
        doDownloadFile(response, file, begin, end);
    }

    // 实际执行下载操作
    private void doDownloadFile(HttpServletResponse response, File file, int begin, int end) throws IOException
    {
        InputStream is    = null;
        OutputStream os    = null;
        
        try
        {
            byte[] b    = new byte[bufferSize];
            is            = new BufferedInputStream(new FileInputStream(file));
            os            = new BufferedOutputStream(response.getOutputStream());
            
            // 跳过已下载的文件内容
            is.skip(begin);
            // I/O 读写
            for(int i, left = end - begin + 1; left > 0 && ((i = is.read(b, 0, Math.min(b.length, left))) != -1); left -= i)
                os.write(b, 0, i);
            
            os.flush();
        }
        finally
        {
            if(is != null) {try{is.close();} catch(IOException e) {}}
            if(os != null) {try{os.close();} catch(IOException e) {}}
        }
    }
    
    / 文件下载结果枚举值 */
    public static enum Result
    {
        / 成功 */
        SUCCESS,
        / 失败：文件不存在 */
        FILE_NOT_FOUND,
        / 失败：读写操作失败 */
        READ_WRITE_FAIL,
        / 失败：未知异常 */
        UNKNOWN_EXCEPTION;
    }
}
```
