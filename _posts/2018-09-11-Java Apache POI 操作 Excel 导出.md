---
title: Java Apache POI 操作 Excel 导出
date: 2018-09-11
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

## JAR 包依赖及介绍
1. 请参见 [Java Apache POI 操作 Excel 导入](http://asing1elife.com/java/poi/2017/03/31/Java-Apache-POI-操作-Excel-导入)

## 实现步骤
### 创建 Excel 实体类，用于同一接收导出数据
1. `headers` 用于存储导出数据时规定的表格头部内容
2. `cells` 是一个在 List 中嵌套 List 的结构
	* 外层的 List 表示行数据
	* 内层的 List 表示列数据
	* 结合起来也就是有多少行多少列的数据
3. 如果导出的内容存在多个 **sheet** ，只需要使用一个 Map ，通过 key-value 的形式包装 Excel 实体类即可
	* 例如 `Map<String, Excel> excels` ，Map 中的 key 就是 **sheet** 的名称

```java
public class Excel {

    // 表头
    private List<String> headers = Lists.newArrayList();

    // 内容
    private List<List<String>> cells = Lists.newArrayList();

    // 省略 Getter/Setter 方法
}

```

### 举一个上述实体类使用的例子
1. 传入的数据本身就很复杂，需要导出的数据也存在多个 **sheet** 

```java
private Map<String, Excel> wrapFeedbackQuestions(Map<String, ArrayList<FeedbackQuestion>> feedbackQuestionMap) {
    Map<String, Excel> excels = Maps.newHashMap();

    if (feedbackQuestionMap.isEmpty()) {
        throw new TSharkException("统计数据转换异常，请咨询管理员");
    }

    List<String> divide = Lists.newArrayList();

    feedbackQuestionMap.forEach((k, v) -> {
        Excel excel = new Excel();
        excel.addHeader("题目");
        excel.addHeader("小计");

        v.forEach(feedbackQuestion -> {
            // 按照约定格式每行添加数据
            List<String> row = Lists.newArrayList();
            row.add(feedbackQuestion.getContent());
            row.add(feedbackQuestion.getCount().toString());

            excel.addCell(row);
        });

        excels.put(k, excel);
    });

    return excels;
}
```

### 构建 ExcelExportUtil 工具类
1. `Map<String, Excel> excel` 是通过上述操作构建好的数据结构
2. `String filePath` 是生成 Excel 后，存放文件的地址
3. `initHeaderStyle()` 和 `initCellStyle()` 是对表格样式的初始化，在后续步骤中提供
4. `formatLongValue()` 是为了防止单元格内容过长，而设置的超过一定量长度添加换行符

```java
public static String export(Map<String, Excel> excel, String filePath) {
    HSSFWorkbook workbook = new HSSFWorkbook();

    // 表头样式
    HSSFCellStyle headerStyle = initHeaderStyle(workbook);

    // 单元格样式
    HSSFCellStyle cellStyle = initCellStyle(workbook);

    excel.forEach((key, value) -> {
        HSSFSheet sheet = workbook.createSheet(key);

        List<String> headers = value.getHeaders();
        List<List<String>> cells = value.getCells();

        // 表头包装
        HSSFRow headerRow = sheet.createRow(0);
        for (int i = 0; i < headers.size(); i++) {
            HSSFCell cell = headerRow.createCell(i);
            cell.setCellValue(headers.get(i));
            cell.setCellStyle(headerStyle);
        }

        // 内容包装
        for (int j = 0; j < cells.size(); j++) {
            HSSFRow row = sheet.createRow(j + 1);

            List<String> child = cells.get(j);
            for (int k = 0; k < child.size(); k++) {
                HSSFCell cell = row.createCell(k);
                cell.setCellValue(formatLongValue(child.get(k)));
                cell.setCellStyle(cellStyle);
            }
        }

        // 自动撑开列宽
        sheet.autoSizeColumn((short) 0);
        sheet.autoSizeColumn((short) 1);
    });

    // 保存文件并返回名称
    String fileName = FileUtil.getRandomFileNameByDate();

    try {
        String fullFileName = filePath + fileName + FILE_SUFFIX;

        FileOutputStream stream = new FileOutputStream(fullFileName);
        workbook.write(stream);
        stream.flush();
    } catch (Exception e) {
        logger.error("Excel导出失败，", e);
    }

    return fileName;
}
```

### initHeaderStyle() 初始化表头样式
1. 对表头单元格的字体、颜色、背景做一些自定义操作

```java
private static HSSFCellStyle initHeaderStyle(HSSFWorkbook workbook) {
    // 表头字体
    HSSFFont font = workbook.createFont();
    font.setFontHeightInPoints((short) 15);
    font.setColor(HSSFColor.WHITE.index);

    // 表头样式
    HSSFCellStyle headerStyle = workbook.createCellStyle();
    headerStyle.setFont(font);
    headerStyle.setWrapText(true);
    headerStyle.setFillBackgroundColor(HSSFColor.GREEN.index);
    headerStyle.setFillForegroundColor(HSSFColor.GREEN.index);
    // 需要设置此项，单元格背景色才会生效
    headerStyle.setFillPattern(CellStyle.SOLID_FOREGROUND);

    return headerStyle;
}
```

### initCellStyle 初始化单元格样式
```java
private static HSSFCellStyle initCellStyle(HSSFWorkbook workbook) {
    // 单元格字体
    HSSFFont font = workbook.createFont();
    font.setFontHeightInPoints((short) 15);

    // 单元格样式
    HSSFCellStyle cellStyle = workbook.createCellStyle();
    cellStyle.setFont(font);
    cellStyle.setWrapText(true);

    return cellStyle;
}
```

## formatLongValue 格式化超长内容
1. 需要单元格启用 `setWrapText = true` 才会生效

```java
private static String formatLongValue(String value) {
    // 长度限制
    int limit = 50;

    // 没有达到限制直接返回
    if (value.length() < 50) {
        return value;
    }

    StringBuilder stringBuilder = new StringBuilder();

    // 设置长度限制的正则匹配
    Pattern p = Pattern.compile("(.{" + limit + "}|.*)");
    Matcher m = p.matcher(value);

    while (m.find()) {
        String group = m.group();

        // 字符不存在则跳过
        if (group == null || group.trim().equals("")) {
            continue;
        }

        // 超过限制添加换行符
        stringBuilder.append(group);
        stringBuilder.append("\n");
    }

    return stringBuilder.toString();
}
```