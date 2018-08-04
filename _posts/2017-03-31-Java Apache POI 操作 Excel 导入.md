---
title: Java Apache POI 操作 Excel 导入
date: 2017-03-31
author: asing1elife
categories:
 - java
 - poi
tags:
 - java
 - poi
 - excel
---
> Java 可以通过 Apache POI 操作 Excel 的导入导出  
> Apache POI 是一套操作 Microsoft Office 套件的开源 Java API  

## 官网
[Apache POI](http://poi.apache.org)

## 依赖
1. 要使用 Apache POI 的功能需要引入以下两个 jar 包
	* 操作 Excel 2003 (.xls) 及之前的版本，只用导入 poi 即可
	* 操作 Excel 2007 (.xlsx) 及以后的版本，则还需要导入 poi-ooxml 才可

``` xml
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi</artifactId>
	<version>3.10.1</version>
</dependency>
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>3.10.1</version>
</dependency>
```

## 判断文件版本
1. 通过上传的文件类型，判断 Excel 的版本，根据具体版本返回解析类
	* 上传 Excel 文件的方式和普通的文件上传没有差异
	* 对于 springMVC 则可以使用 MultipartFile 进行文件获取

``` java
private static Workbook getWorkbook(MultipartFile excelFile) throws IOException {
    // 获取文件输入流
    InputStream inputStream = excelFile.getInputStream();
    // 获取文件内容
    String fileName = excelFile.getOriginalFilename();

    // 获取文件类型
    String fileType = fileName.substring(fileName.indexOf("."));

    // 判断文件类型
    if (("xls").equalsIgnoreCase(fileType)) {
        return new HSSFWorkbook(inputStream);
    } else if (("xlsx").equalsIgnoreCase(fileType)) {
        return new XSSFWorkbook(inputStream);
    } else {
        logger.error("Excel 文件解析失败，格式错误！");
        throw new TSharkException("Excel 文件解析失败，格式错误！");
    }
}
```

## 获取到 Excel 文件解析类后则对文件进行解压
* 解压的方式是依次遍历 Excel 内容：sheet-row-column
* 遍历时需要考虑到每个阶段非空的情况
* 遍历 row 的时候需要考虑过滤表头的情况，一般表头可能是表格信息（具体情况需要根据模版决定）

``` java
public static List<List<Object>> getExcelContent(MultipartFile excelFile) {
    // 行列表
    List<List<Object>> contents = Lists.newArrayList();

    Workbook workbook;

    try {
        // 解析 Excel 获取 Workbook
        workbook = getWorkbook(excelFile);
    } catch (IOException e) {
        logger.error("获取 Excel 内容失败！", e);
        throw new TSharkException("获取 Excel 内容失败！", e);
    }

    // 遍历所有表
    for (int i = 0; i < workbook.getNumberOfSheets(); i++) {
        // 获取当前表
        Sheet sheet = workbook.getSheetAt(i);

        // 非空验证
        if (null == sheet) {
            continue;
        }

        // 遍历当前表中所有行,过滤掉标题行
        for (int j = sheet.getFirstRowNum() + 1; j <= sheet.getLastRowNum(); j++) {
            // 获取当前行
            Row row = sheet.getRow(j);

            // 非空验证
            if (null == row) {
                continue;
            }

            // 单元格列表
            List<Object> cells = Lists.newArrayList();
            // 遍历当前行中所有单元格
            for (int k = row.getFirstCellNum(); k <= row.getLastCellNum(); k++) {
                // 获取当前单元格
                Cell cell = row.getCell(k);

                // 非空验证
                if (null == cell) {
                    continue;
                }

                // 获取单元格格式化后的内容
                cells.add(formatCellValue(cell));
            }

            // 获取所有行的单元格列表
            contents.add(cells);
        }
    }

    return contents;
}
```

## 遍历文件内容
1. 遍历内容到最后是获取单元格内容，而单元格的内容获取需要进行一定的格式化处理

```java
private static Object formatCellValue(Cell cell) {
    Object cellValue = null;

    switch (cell.getCellType()) {
        case Cell.CELL_TYPE_STRING:
            cellValue = cell.getRichStringCellValue().toString();
            break;
        case Cell.CELL_TYPE_NUMERIC:
            // 获取具体类型
            String dataFormat = cell.getCellStyle().getDataFormatString();

            // 具体类型判断
            if (("General").equalsIgnoreCase(dataFormat)) {
                cellValue = new DecimalFormat("0").format(cell.getNumericCellValue());
            } else if (("m/d/yy").equalsIgnoreCase(dataFormat)) {
                cellValue = DateUtil.dateToString(cell.getDateCellValue());
            } else {
                cellValue = new DecimalFormat("0.00").format(cell.getNumericCellValue());
            }

            break;
        case Cell.CELL_TYPE_BOOLEAN:
            cellValue = cell.getBooleanCellValue();
            break;
        case Cell.CELL_TYPE_BLANK:
            cellValue = "";
            break;
        default:
            break;
    }

    return cellValue;
}
```

## 处理过滤得到的数据
4. 将 Excel 中的内容遍历到 List 列表中后，则需要对列表中的有效内容进行读取
	* 读取的具体内容索引根据模版实际情况决定
	* 在读取内容的同时则可以对数据进行持久化操作

``` java
private List<UniversityMemberDTO> generateUniversityMembers(List<List<Object>> contents) {
    List<UniversityMemberDTO> universityMembers = Lists.newArrayList();

    // 遍历内容列表
    for (List<Object> content : contents) {
        UniversityMemberDTO universityMember = new UniversityMemberDTO();
        // 登录名
        universityMember.setLoginName((String) content.get(0));
        // 真实姓名
        universityMember.setRealName((String) content.get(1));
        // 联系方式
        universityMember.setMobile((String) content.get(2));
        // 邮箱
        universityMember.setEmail((String) content.get(3));
        // 专业名称
        universityMember.setMajorName((String) content.get(4));
        // 学院名称
        universityMember.setInstituteName((String) content.get(5));
        // 用户类型
        universityMember.setType(getUniversityMemberType((String) content.get(6)));

        universityMembers.add(universityMember);
    }

    return universityMembers;
}
```