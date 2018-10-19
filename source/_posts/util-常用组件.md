---
layout: post
title: 常用组件
date: 2018-08-08 06:55:06
tags: util
categories: util
---

### 使用POI组件导出excel
1、添加依赖:
```xml
dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>3.9</version>
</dependency>
```

<!-- more -->

2、创建实体类：

```java
package excel;

import java.util.Date;

public class Student {
	private int id;
	private String name;
	private int age;
	private Date birth;

	public Student() {
	}

	public Student(int id, String name, int age, Date birth) {
		this.id = id;
		this.name = name;
		this.age = age;
		this.birth = birth;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public Date getBirth() {
		return birth;
	}

	public void setBirth(Date birth) {
		this.birth = birth;
	}

}
```

3、具体的excel操作:

```java
package excel;

import java.io.FileOutputStream;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;

import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFCellStyle;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
/**
 * 利用POI 导出Excel
 * @author GIE
 *
 */
public class CreateSimpleExcelToDisk {
	/**
	 * @功能：手工构建一个简单格式的Excel
	 */
	private static List<Student> getStudent() throws Exception {
		List list = new ArrayList();
		SimpleDateFormat df = new SimpleDateFormat("yyyy-mm-dd");

		Student user1 = new Student(1, "张三", 16, df.parse("1997-03-12"));
		Student user2 = new Student(2, "李四", 17, df.parse("1996-08-12"));
		Student user3 = new Student(3, "王五", 26, df.parse("1985-11-12"));
		list.add(user1);
		list.add(user2);
		list.add(user3);

		return list;
	}

	public static void main(String[] args) throws Exception {
		// 第一步，创建一个webbook，对应一个Excel文件
		HSSFWorkbook wb = new HSSFWorkbook();
		// 第二步，在webbook中添加一个sheet,对应Excel文件中的sheet
		HSSFSheet sheet = wb.createSheet("学生表一");
		// 第三步，在sheet中添加表头第0行,注意老版本poi对Excel的行数列数有限制short
		HSSFRow row = sheet.createRow((int) 0);
		// 第四步，创建单元格，并设置值表头 设置表头居中
		HSSFCellStyle style = wb.createCellStyle();
		style.setAlignment(HSSFCellStyle.ALIGN_CENTER); // 创建一个居中格式

		HSSFCell cell = row.createCell((short) 0);
		cell.setCellValue("学号");
		cell.setCellStyle(style);
		cell = row.createCell((short) 1);
		cell.setCellValue("姓名");
		cell.setCellStyle(style);
		cell = row.createCell((short) 2);
		cell.setCellValue("年龄");
		cell.setCellStyle(style);
		cell = row.createCell((short) 3);
		cell.setCellValue("生日");
		cell.setCellStyle(style);

		// 第五步，写入实体数据 实际应用中这些数据从数据库得到，
		List list = CreateSimpleExcelToDisk.getStudent();

		for (int i = 0; i < list.size(); i++) {
			row = sheet.createRow((int) i + 1);
			Student stu = (Student) list.get(i);
			// 第四步，创建单元格，并设置值
			row.createCell((short) 0).setCellValue((double) stu.getId());
			row.createCell((short) 1).setCellValue(stu.getName());
			row.createCell((short) 2).setCellValue((double) stu.getAge());
			cell = row.createCell((short) 3);
			cell.setCellValue(new SimpleDateFormat("yyyy-mm-dd").format(stu.getBirth()));
		}
		// 第六步，将文件存到指定位置
		try {
			FileOutputStream fout = new FileOutputStream("E:/students.xls");
			wb.write(fout);
			fout.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

### 利用iText 组件导出PDF
1、添加依赖
```xml
<dependency>
	<groupId>com.itextpdf</groupId>
	<artifactId>itextpdf</artifactId>
	<version>5.2.0</version>
</dependency>
<!-- PDF输出中文的扩展包 -->
<dependency>
	<groupId>com.itextpdf</groupId>
	<artifactId>itext-asian</artifactId>
	<version>5.2.0</version>
</dependency>
```
2、具体操作：
```java
package excel;

import java.io.FileOutputStream;
import java.io.FileOutputStream;

import com.itextpdf.text.Document;
import com.itextpdf.text.Font;
import com.itextpdf.text.Paragraph;
import com.itextpdf.text.pdf.BaseFont;
import com.itextpdf.text.pdf.PdfWriter;

/**
 * 利用Itext导出PDF 文档
 * 
 * @author GIE
 *
 */
public class CreatPdf {
	public static void main(String[] args) {
		Document doc = null;
		try {
			doc = new Document();
			PdfWriter.getInstance(doc, new FileOutputStream("C:\\itext.pdf"));
			doc.open();
			doc.addTitle("测试标题");
			doc.addAuthor("gie");
			doc.addCreationDate();
			doc.addSubject("测试主题");
			// itext 中文的处理
			BaseFont bfChinese = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
			Font FontChinese = new Font(bfChinese, 12, Font.NORMAL);
			Paragraph pragraph = new Paragraph("你好", FontChinese);
			doc.add(pragraph);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			doc.close();
		}
	}
}
```


### 上传文件到七牛云

1、引入依赖：
```xml
<dependency>
    <groupId>com.qiniu</groupId>
    <artifactId>qiniu-java-sdk</artifactId>
    <version>7.2.6</version>
</dependency>
```

2、具体操作：
```java
/**
 * 上传文件到七牛云存储demo
 * @author GIE
 *
 */
public class QiniuTest {
	//七牛的密钥
	private static String ACCESS_KEY = "";
	private static String SECRET_KEY = "";
	public static void main(String[] args) throws Exception{
		 Auth auth = Auth.create(ACCESS_KEY, SECRET_KEY);
		 UploadManager uploadManager = new UploadManager();
		    try {
		    /**
              * 上传文件
              *
              * @param file  上传的文件对象
              * @param key   上传文件保存的文件名(下载时候使用,不能重复)
              * @param token 上传凭证
            */
           Response  response = uploadManager.put(new File("d:/1.png"), "1.png", getUpToken(auth));
		    } catch (QiniuException e) {
		        Response r = e.response;
		        System.out.println(r.toString());
		    }
	}
	
	/**
     * 生成上传token
     *
     * @param bucket  空间名
     * @param key     key，可为 null
     * @param expires 有效时长，单位秒
     * @param policy  上传策略的其它参数，如 new StringMap().put("endUser", "uid").putNotEmpty("returnBody", "")。
     *                scope通过 bucket、key间接设置，deadline 通过 expires 间接设置
     * @return 生成的上传token
     */
	private static String getUpToken(Auth auth){
		 return auth.uploadToken("hdwx", null, 3600, new StringMap()
		            .putNotEmpty("returnBody", ""));
	}

}
```

### hutool工具箱
```xml
<!--hutool-->
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
	<version>${hutool.version}</version>
</dependency>
```
