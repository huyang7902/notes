# 概述

常见工具类有

Jxl：消耗小、图片和图形支持有限

POI：功能完善

提供API对Microsoft Office格式的文档读和写的功能

# POI包结构

HSSF：读写Excel、XLS（针对07版本之前的）

XSSF：读写OOXML、XLSX（针对07版本之后的）

HWPF：读写Word

HSLF：读写PowerPoint

# 概念

POI封装的对象：

XSSFWorkbook：工作簿

XSSFSheet：工作表

Row：行

Cell：单元格

# 对excel操作

导入依赖：

poi-ooxml是poi的升级版，处理07版本之后的文档

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
</dependency>
```

# 读取

```java
public class ReadXLSXDemo {

    public static void main(String[] args) throws Exception {

        String path = Thread.currentThread().getContextClassLoader().getResource("").getPath();
        System.out.println(path);
        String fileName = "read.xlsx";

        // 1. 获取工作簿
        XSSFWorkbook workbook = new XSSFWorkbook(new FileInputStream(new File(path,fileName)));
        // 2. 获取工作表
        XSSFSheet sheet = workbook.getSheetAt(0);
        // 3. 获取行
//        for (Row row : sheet) {
//            //4. 获取单元格
//            for (Cell cell : row) {
//                String value = cell.getStringCellValue();
//                System.out.println(value);
//            }
//        }
        // 普通for，稚嫩遍历一次

        int lastRowNum = sheet.getLastRowNum();
        for (int i = 0; i <= lastRowNum; i++) {
            XSSFRow row = sheet.getRow(i);
            if (row != null) {
                short lastCellNum = row.getLastCellNum();
                for (int j = 0; j <= lastCellNum; j++) {
                    XSSFCell cell = row.getCell(j);
                    if (cell != null) {
                        //手动设置类型
                        cell.setCellType(CellType.STRING);
                        String value = cell.getStringCellValue();
                        System.out.println(value);
                    }
                }
            }
        }

        // 关闭资源
        workbook.close();

    }
}

```



# 写入

```java
public class WriteXlsxDemo {
    public static void main(String[] args) throws Exception{

        // 创建工作簿
        XSSFWorkbook workbook = new XSSFWorkbook();
        //创建工作表
        XSSFSheet sheet = workbook.createSheet("工作表一");
        // 创建行
        XSSFRow row = sheet.createRow(0);

        // 创建单元格样式
        XSSFCellStyle cellStyle = workbook.createCellStyle();
        cellStyle.setFillForegroundColor(IndexedColors.PINK.index);
        cellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

        // 字体样式
        XSSFFont font = workbook.createFont();
        font.setFontName("黑体");
        font.setColor(IndexedColors.BLUE.index);
        cellStyle.setFont(font);

        // 创建单元格
        XSSFCell cell = row.createCell(0);
        cell.setCellStyle(cellStyle);
        cell.setCellValue("这");
        XSSFCell cell1 = row.createCell(1);
        cell1.setCellStyle(cellStyle);
        cell1.setCellValue("是");
        XSSFCell cell2 = row.createCell(2);
        cell2.setCellStyle(cellStyle);
        cell2.setCellValue("一");
        XSSFCell cell3 = row.createCell(3);
        cell3.setCellStyle(cellStyle);
        cell3.setCellValue("个");
        XSSFCell cell4 = row.createCell(4);
        cell4.setCellStyle(cellStyle);
        cell4.setCellValue("单元格");

        // 创建行
        XSSFRow row1 = sheet.createRow(1);
        // 创建单元格
        row1.createCell(0).setCellValue("这");
        row1.createCell(1).setCellValue("是");
        row1.createCell(2).setCellValue("一");
        row1.createCell(3).setCellValue("个");
        row1.createCell(4).setCellValue("单元格");


        String path = Thread.currentThread().getContextClassLoader().getResource("").getPath();
        System.out.println(path);
        String fileName = "write.xlsx";
        FileOutputStream outputStream = new FileOutputStream(new File(path, fileName));

        // 将工作簿写出到文件
        workbook.write(outputStream);
        outputStream.close();
        workbook.close();
    }

}
```

