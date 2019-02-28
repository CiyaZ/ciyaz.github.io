# POI 读写Excel文档

读写Excel文档的需求非常常见，比如：教务处管理系统中的导出课程表功能。除此之外，分析数据时，可能提供数据的有关人员不是专业的程序员，他们可能习惯使用Excel，而直接提供`.xls`或是`.xlsx`文档。解析Excel文档确实会为我们编程带来一点困难，不过Apache POI能够解决这个问题。

但是这个库太复杂了，实际上我们并不需要太多的功能，这里只是记录一下如何以最简单的方式读写Excel文档。

还有一个JXL库提供了读写`.xls`文档的API，使用很方便，但是不支持`.xlsx`格式。

## 生成xls文档

```java
public static void writeXLS() throws Exception
{
  File file = new File("/home/ciyaz/test.xls");

  HSSFWorkbook workbook = new HSSFWorkbook();
  HSSFSheet sheet = workbook.createSheet("sheet1");
  for(int i = 0; i <= 5; i++)
  {
    HSSFRow row = sheet.createRow(i);
    for(int j = 0; j < 5; j++)
    {
      HSSFCell cell = row.createCell(j);
      cell.setCellValue("row" + i + "cell" + j);
    }
  }
  workbook.write(file);
}
```

我们知道一个Excel文档分为若干`sheet`，每个`sheet`里有若干行和列，每一个单元格叫做一个`cell`。上面代码所做的操作实际上就是对应这些概念的。

## 生成xlsx文档

```java
public static void writeXLS() throws Exception
{
  File file = new File("/home/ciyaz/test.xlsx");

  XSSFWorkbook workbook = new XSSFWorkbook();
  XSSFSheet sheet = workbook.createSheet("sheet1");
  for(int i = 0; i <= 5; i++)
  {
    XSSFRow row = sheet.createRow(i);
    for(int j = 0; j < 5; j++)
    {
      XSSFCell cell = row.createCell(j);
      cell.setCellValue("row" + i + "cell" + j);
    }
  }
  OutputStream outputStream = new FileOutputStream(file);
  workbook.write(outputStream);
}
```

写法几乎与上面相同，只不过把`HSSF`换成了`XSSF`。

## 读取xls文档

```java
public static void readXLS() throws Exception
{
  InputStream inputStream = new FileInputStream("/home/ciyaz/test.xls");
  HSSFWorkbook workbook = new HSSFWorkbook(inputStream);

  HSSFSheet sheet = workbook.getSheetAt(0);
  Iterator rowIterator = sheet.rowIterator();
  while(rowIterator.hasNext())
  {
    HSSFRow row = (HSSFRow) rowIterator.next();
    Iterator cellIterator = row.cellIterator();
    while(cellIterator.hasNext())
    {
      HSSFCell cell = (HSSFCell) cellIterator.next();
      System.out.println(cell.getStringCellValue());
    }
  }

  inputStream.close();
}
```

读取`row`和`cell`需要获取迭代器。实际上这个API设计的有点别扭，名字起的就不是很顺手（HSSF什么的），写法也怪怪的。

## 读取xlsx文档

```java
public static void readXLS() throws Exception
{
  InputStream inputStream = new FileInputStream("/home/ciyaz/test.xlsx");
  XSSFWorkbook workbook = new XSSFWorkbook(inputStream);

  XSSFSheet sheet = workbook.getSheetAt(0);
  Iterator rowIterator = sheet.rowIterator();
  while(rowIterator.hasNext())
  {
    XSSFRow row = (XSSFRow) rowIterator.next();
    Iterator cellIterator = row.cellIterator();
    while(cellIterator.hasNext())
    {
      XSSFCell cell = (XSSFCell) cellIterator.next();
      System.out.println(cell.getStringCellValue());
    }
  }

  inputStream.close();
}
```

和上面一样，把`H`全换成`X`就行了。
