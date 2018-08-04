---
title: Java使用OpenOffice在线预览Office及PDF
date: 2017-04-11
author: asing1elife
categories:
 - java
 - openoffice
tags:
 - java
 - openoffice
 - pdf
---
> 使用OpenOffice可实现在线预览office文件内容  

## 思路
1. 用 OpenOffice 将 word / excel / ppt 转换为 pdf
2. 用 pdf.js 将转换后的 pdf 显示在浏览器中显示
	
## 准备
1. 安装 OpenOffice ，参见 [OpenOffice 在 Linux 下安装使用](http://asing1elife.com/java/openoffice/linux/2017/04/11/OpenOffice-在-Linux-下安装使用/)
2. 启动 OpenOffice ，`soffice "-accept=socket,host=localhost,port=8100;urp;" -headless -nofirststartwizard &`
	* Mac端需要跳转到 `Applications/OpenOffice.app/Contents/program` 才可启动服务
3. 下载 [JODConverter](https://sourceforge.net/projects/jodconverter/files/) ，在其 lib 目录中找到 **jodconverter-cli-2.2.2.jar** ，并引入以下 jar 包
4. 下载 [PDF.js](http://mozilla.github.io/pdf.js/) ，并引入到项目中
	* vue项目使用[vue.js pdf viewer](https://github.com/FranckFreiburger/vue-pdf)


## 在项目 pom.xml 中引入以下包
```xml
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>juh</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>jurt</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>ridl</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>unoil</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>jodconverter</artifactId>
    <version>2.2.2</version>
    <scope>system</scope>
    <systemPath>${basedir}/../lib/jodconverter-2.2.2.jar</systemPath>
</dependency>
```
	
## 调用服务并转换文件
1.  通过 OpenOfficeConnection 调用 OpenOffice 服务，并将传入文件转换为 PDF 格式

```java
public static File fileToPdf(File file) {
    // 文件全路径名
    String fileName = file.getPath();

    // 存放转换结果的 pdf 文件
    File pdfFile = new File(fileName.substring(0, fileName.lastIndexOf(".")) + Constants.REPORT_FILE_PREVIEW_SUFFIX);

    // 非空验证
    if (!file.exists() || !file.isFile()) {
        return null;
    }

    // 存在则不再转换
    if (pdfFile.exists() && pdfFile.isFile()) {
        return pdfFile;
    }

    // 获取连接
    OpenOfficeConnection connection = new SocketOpenOfficeConnection(Constants.OPEN_OFFICE_CONNECTION_PORT);

    try {
        // 建立连接
        connection.connect();

        // 开始转换
        DocumentConverter converter = new OpenOfficeDocumentConverter(connection);
        converter.convert(file, pdfFile);

        // 关闭连接
        connection.disconnect();
    } catch (ConnectException e) {
        logger.error("PDF 转换失败，OpenOffice 服务未启动！", e);
        throw new TSharkException("PDF 转换失败，OpenOffice 服务未启动！", e);
    }

    return pdfFile;
}
```

## 在页面准备显示内容的区域
1. 页面上通过 iframe 引入 pdf.js 中的 viewer.html  并传入待显示的 pdf 文件

```html
<iframe src="${ctx}/assets/plugins/pdf/web/viewer.html?file=<c:url value="/api/report/intro/preview/file/${fileNamePrefix}/${fileNameSuffix}"/>" width="100%" height="100%" frameborder="0" scrolling="hidden">
</iframe>
```

## 后端返回待显示的文件流
1. 后端将待显示的 pdf 文件通过 ResponseEntity 传入到前端

```java
public ResponseEntity<byte[]> preview(String fileName) throws IOException {
    // 获取文件
    File file = new File(Constants.REPORT_SOURCE_FILE_PATH + fileName);

    // 转换并返回结果
    byte[] pdfFileBytes = FileUtils.readFileToByteArray(OnlinePreviewUtil.fileToPdf(file));

    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.setContentType(MediaType.valueOf("application/pdf"));
    httpHeaders.setContentLength(pdfFileBytes.length);
    httpHeaders.add(HttpHeaders.ACCEPT_RANGES, "bytes");

    return new ResponseEntity<byte[]>(pdfFileBytes, httpHeaders, HttpStatus.OK);
}
```