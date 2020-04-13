---
title: 处理文件上传(MultipartResolver)
permalink: /java-util-code/MultipartResolver/explanation
tags: 文件上传 工具类
pageview: true
show_date: true
sidebar:
  nav: docs-en-code
---
#### 简介
MultipartResolver 用于处理文件上传，当收到请求时 DispatcherServlet 的 checkMultipart() 方法会调用 MultipartResolver 的 isMultipart() 方法判断请求中是否包含文件。如果请求数据中包含文件，则调用 MultipartResolver 的 resolveMultipart() 方法对请求的数据进行解析，然后将文件数据解析成 MultipartFile 并封装在 MultipartHttpServletRequest (继承了 HttpServletRequest) 对象中，最后传递给 Controller，在 MultipartResolver 接口中有如下方法：

- boolean isMultipart(HttpServletRequest request); // 是否是 multipart
- MultipartHttpServletRequest resolveMultipart(HttpServletRequest request); // 解析请求
- void cleanupMultipart(MultipartHttpServletRequest request);

MultipartFile 封装了请求数据中的文件，此时这个文件存储在内存中或临时的磁盘文件中，需要将其转存到一个合适的位置，因为请求结束后临时存储将被清空。在 MultipartFile 接口中有如下方法：

- String getName(); // 获取参数的名称
- String getOriginalFilename(); // 获取文件的原名称
- String getContentType(); // 文件内容的类型
- boolean isEmpty(); // 文件是否为空
- long getSize(); // 文件大小
- byte[] getBytes(); // 将文件内容以字节数组的形式返回
- InputStream getInputStream(); // 将文件内容以输入流的形式返回
- void transferTo(File dest); // 将文件内容传输到指定文件中    

**MultipartResolver 是一个接口，它的实现类如下图所示，分为 CommonsMultipartResolver 类和 StandardServletMultipartResolver 类。**
其中 CommonsMultipartResolver 使用 commons Fileupload 来处理 multipart 请求，所以在使用时，必须要引入相应的 jar 包；而 StandardServletMultipartResolver 是基于 Servlet 3.0来处理 multipart 请求的，所以不需要引用其他 jar 包，但是必须使用支持 Servlet 3.0的容器才可以，以tomcat为例，从 Tomcat 7.0.x的版本开始就支持 Servlet 3.0了。

一、CommonsMultipartResolver

1 使用方式

1.1 配置文件

```xml
<!-- 定义文件上传解析器 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设定默认编码 -->
    <property name="defaultEncoding" value="UTF-8"></property>
    <!-- 设定文件上传的最大值为5MB，5*1024*1024 -->
    <property name="maxUploadSize" value="5242880"></property>
    <!-- 设定文件上传时写入内存的最大值，如果小于这个参数不会生成临时文件，默认为10240 -->
    <property name="maxInMemorySize" value="40960"></property>
    <!-- 上传文件的临时路径 -->
    <property name="uploadTempDir" value="fileUpload/temp"></property>
    <!-- 延迟文件解析 -->
    <property name="resolveLazily" value="true"/>
</bean>
```

1.2 上传表单

要在 form 标签中加入 enctype="multipart/form-data" 表示该表单要提交文件。

```html
<form action="${pageContext.request.contextPath}/test/file-upload.do" method="post" enctype="multipart/form-data">
     <input type="file" name="file">
     <input type="submit" value="提交">
</form>
```
1.3 处理文件

```java
@RequestMapping("/file-upload")
public ModelAndView upload(@RequestParam(value = "file", required = false) MultipartFile file,
　　　　　　HttpServletRequest request, HttpSession session) {
    // 文件不为空
    if(!file.isEmpty()) {
        // 文件存放路径
        String path = request.getServletContext().getRealPath("/");
        // 文件名称
        String name = String.valueOf(new Date().getTime()+"_"+file.getOriginalFilename());
        File destFile = new File(path,name);
        // 转存文件
        try {
            file.transferTo(destFile);
        } catch (IllegalStateException | IOException e) {
            e.printStackTrace();
        }
        // 访问的url
        String url = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
　　　　　　+ request.getContextPath() + "/" + name;
    }        
    ModelAndView mv = new ModelAndView();
    mv.setViewName("other/home");
    return mv;
}
```
2 源码分析

CommonsMultipartResolver 实现了 MultipartResolver 接口，resolveMultipart() 方法如下所示，其中 resolveLazily 是判断是否要延迟解析文件（通过XML可以设置）。当 resolveLazily 为 flase 时，会立即调用 parseRequest() 方法对请求数据进行解析，然后将解析结果封装到 DefaultMultipartHttpServletRequest 中；而当 resolveLazily 为 true 时，会在 DefaultMultipartHttpServletRequest 的 initializeMultipart() 方法调用 parseRequest() 方法对请求数据进行解析，而 initializeMultipart() 方法又是被 getMultipartFiles() 方法调用，即当需要获取文件信息时才会去解析请求数据，这种方式用了懒加载的思想。

```java
@Override
public MultipartHttpServletRequest resolveMultipart(final HttpServletRequest request) throws MultipartException {
    Assert.notNull(request, "Request must not be null");
    if (this.resolveLazily) {
        //懒加载，当调用DefaultMultipartHttpServletRequest的getMultipartFiles()方法时才解析请求数据
        return new DefaultMultipartHttpServletRequest(request) {
            @Override //当getMultipartFiles()方法被调用时，如果还未解析请求数据，则调用initializeMultipart()方法进行解析 protected void initializeMultipart() {
                MultipartParsingResult parsingResult = parseRequest(request);
                setMultipartFiles(parsingResult.getMultipartFiles());
                setMultipartParameters(parsingResult.getMultipartParameters());
                setMultipartParameterContentTypes(parsingResult.getMultipartParameterContentTypes());
            }
        };
    } else {
        //立即解析请求数据，并将解析结果封装到DefaultMultipartHttpServletRequest对象中
        MultipartParsingResult parsingResult = parseRequest(request);
        return new DefaultMultipartHttpServletRequest(request, parsingResult.getMultipartFiles(),
              parsingResult.getMultipartParameters(), parsingResult.getMultipartParameterContentTypes());
    }
}
```
在上面的代码中可以看到，对请求数据的解析工作是在 parseRequest() 方法中进行的，继续看一下 parseRequest() 方法源码

```java
protected MultipartParsingResult parseRequest(HttpServletRequest request) throws MultipartException {
    // 获取请求的编码类型
    String encoding = determineEncoding(request);
    FileUpload fileUpload = prepareFileUpload(encoding);
    try {
        List<FileItem> fileItems = ((ServletFileUpload) fileUpload).parseRequest(request);
        return parseFileItems(fileItems, encoding);
    } catch (...) {}
}
```
在 parseRequest() 方法中，首先调用了 prepareFileUpload() 方法来根据编码类型确定一个 FileUpload 实例，然后利用这个 FileUpload 实例解析请求数据后得到文件信息，最后将文件信息解析成 CommonsMultipartFile (实现了 MultipartFile 接口) 并包装在 MultipartParsingResult 对象中。



二、StandardServletMultipartResolver

1 使用方式

1.1 配置文件

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
</bean>
```
这里并没有配置文件大小等参数，这些参数的配置在 web.xml 中

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:springmvc.xml</param-value>  
    </init-param>  
    <load-on-startup>1</load-on-startup>
    <multipart-config>
        <!-- 临时文件的目录 -->
        <location>d:/</location>
        <!-- 上传文件最大2M -->
        <max-file-size>2097152</max-file-size>
        <!-- 上传文件整个请求不超过4M -->
        <max-request-size>4194304</max-request-size>
    </multipart-config>
</servlet>
```
1.2 上传表单

要在 form 标签中加入 enctype="multipart/form-data" 表示该表单要提交文件。

```html
<form action="${pageContext.request.contextPath}/test/file-upload.do" method="post" enctype="multipart/form-data">
     <input type="file" name="file">
     <input type="submit" value="提交">
</form>
```
1.3 处理文件

1.3.1 通过 MultipartFile 类型的参数

```java
@RequestMapping("/file-upload")
public ModelAndView upload(@RequestParam(value = "file", required = false) MultipartFile file,
　　　　　　HttpServletRequest request, HttpSession session) {
    // 文件不为空
    if(!file.isEmpty()) {
        // 文件存放路径
        String path = request.getServletContext().getRealPath("/");
        // 文件名称
        String name = String.valueOf(new Date().getTime()+"_"+file.getOriginalFilename());
        File destFile = new File(path,name);
        // 转存文件
        try {
            file.transferTo(destFile);
        } catch (IllegalStateException | IOException e) {
            e.printStackTrace();
        }
        // 访问的url
        String url = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
　　　　　　+ request.getContextPath() + "/" + name;
    }        
    ModelAndView mv = new ModelAndView();
    mv.setViewName("other/home");
    return mv;
}
```
1.3.2 通过 MultipartHttpServletRequest 类型的参数

```java
@RequestMapping("/file-upload")
public ModelAndView upload(MultipartHttpServletRequest request, HttpSession session) {
    // 根据页面input标签的name
    MultipartFile file = request.getFile("file");
    // 文件不为空
    if(!file.isEmpty()) {
        // 文件存放路径
        String path = request.getServletContext().getRealPath("/");
        // 文件名称
        String name = String.valueOf(new Date().getTime()+"_"+file.getOriginalFilename());
        File destFile = new File(path,name);
        // 转存文件
        try {
            file.transferTo(destFile);
        } catch (IllegalStateException | IOException e) {
            e.printStackTrace();
        }
        // 访问的url
        String url = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
　　　　　　+ request.getContextPath() + "/" + name;
    }        
    ModelAndView mv = new ModelAndView();
    mv.setViewName("other/home");
    return mv;
}
```
2 源码分析

StandardServletMultipartResolver 实现了 MultipartResolver 接口，resolveMultipart() 方法如下所示，其中 resolveLazily 是判断是否要延迟解析文件（通过XML可以设置）。

```java
public MultipartHttpServletRequest resolveMultipart(HttpServletRequest request) throws MultipartException {
    return new StandardMultipartHttpServletRequest(request, this.resolveLazily);
}
public StandardMultipartHttpServletRequest(HttpServletRequest request, boolean lazyParsing) throws MultipartException {
    super(request);
    // 判断是否立即解析
    if (!lazyParsing) {
        parseRequest(request);
    }
}
```
对请求数据的解析工作是在 parseRequest() 方法中进行的，继续看一下 parseRequest() 方法源码

```java
private void parseRequest(HttpServletRequest request) {
    try {
        Collection<Part> parts = request.getParts();
        this.multipartParameterNames = new LinkedHashSet<String>(parts.size());
        MultiValueMap<String, MultipartFile> files = new LinkedMultiValueMap<String, MultipartFile>(parts.size());
        for (Part part : parts) {
            String disposition = part.getHeader(CONTENT_DISPOSITION);
            String filename = extractFilename(disposition);
            if (filename == null) {
                filename = extractFilenameWithCharset(disposition);
            }
            if (filename != null) {
                files.add(part.getName(), new StandardMultipartFile(part, filename));
            } else {
                this.multipartParameterNames.add(part.getName());
            }
        }
        setMultipartFiles(files);
    } catch (Throwable ex) {}
}
```
parseRequest() 方法利用了 servlet3.0 的 request.getParts() 方法获取上传文件，并将其封装到 MultipartFile 对象中。
