---
title: Excel工具类
urlname: qa0dqt
date: 2020-06-18 17:10:39 +0800
tags: []
categories: []
---

_**xls 版本 Excel 工具类**_
_**本工具只对单文件操作**_
_**方法一传入 list 对象集合-保存到 ecexl 文件**_
_**方法二往\*\***ecexl 文件追加数据\*\*_

```java
package com.mhtech.platform.msrv.sharing.utils;

import jxl.Workbook;
import jxl.WorkbookSettings;
import jxl.write.Label;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;
import jxl.write.WriteException;

import java.io.File;
import java.io.IOException;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.*;

/**
 * Excel工具类
 * Excel单文件操作，xls_版本
 * @author MaoJiaLe 2019-7-5
 */
public class ExcelUtil {

	/**
	 * 将list对象集合，存为Excel文件
	 *
	 * @param list     对象集合
	 * @param path     保存路径
	 * @param fileName 文件名
	 * @param map      实体类属性对应解释（name,名称）
	 * @return 文件路径
	 */
	public static String createExcel(List<? extends Object> list, String path, String fileName,
			Map<String, Object> map) {
		String result = "";
		if (list.size() == 0 || list == null) {
			result = "没有对象信息";
		} else {
			Object o = list.get(0);
			Class<? extends Object> clazz = o.getClass();
			String className = clazz.getSimpleName(); // 获得类简写名称
			Field[] fields = clazz.getDeclaredFields(); // 这里通过反射获取字段数组
			File folder = new File(path);
			if (!folder.exists()) {
				folder.mkdirs();
			}

			String name = fileName.concat(".xls");
			WritableWorkbook book = null;
			File file = null;
			try {
				file = new File(path.concat(File.separator).concat(name));
				WorkbookSettings settings = new WorkbookSettings();
				settings.setUseTemporaryFileDuringWrite(true);
				book = Workbook.createWorkbook(file, settings); // 创建xls文件
				WritableSheet sheet = book.createSheet(className, 0);
				// 额外创建9个sheet页面-用于数据量大，分页功能 不需要可以删除这个循环
				for (int i = 1; i <= 9; i++) {
					book.createSheet(className + i, i);
				}

				int i = 0; // 列
				int j = 0; // 行
				for (Field f : fields) {

					j = 0;
					String nameValue = "";
					for (Map.Entry<String, Object> a : map.entrySet()) {
						if (a.getKey().equals(f.getName())) {
							nameValue = a.getValue().toString();
						}
					}
					Label utfName = new Label(i, j, nameValue); // 字段对应中文名称写入第一行中
					sheet.addCell(utfName);

					j = 1;
					Label label = new Label(i, j, f.getName()); // 这里把字段名称写入excel第二行中
					sheet.addCell(label);
					j = 2;
					for (Object obj : list) {
						Object temp = getFieldValueByName(f.getName(), obj);
						String strTemp = "";
						if (temp != null) {
							strTemp = temp.toString();
						}
						sheet.addCell(new Label(i, j, strTemp)); // 把每个对象此字段的属性写入这一列excel中
						j++;
					}
					i++;
				}
				book.write();
				result = file.getPath();
			} catch (Exception e) {
				result = "SystemException";
				e.printStackTrace();
			} finally {
				fileName = null;
				name = null;
				folder = null;
				file = null;
				if (book != null) {
					try {
						book.close();
					} catch (WriteException e) {
						result = "WriteException";
						e.printStackTrace();
					} catch (IOException e) {
						result = "IOException";
						e.printStackTrace();
					}
				}
			}
		}
		return result; // 最后输出文件路径
	}

	/**
	 * 获取属性值
	 *
	 * @param fieldName 字段名称
	 * @param o         对象
	 * @return Object
	 */
	private static Object getFieldValueByName(String fieldName, Object o) {
		try {
			String firstLetter = fieldName.substring(0, 1).toUpperCase();// 截取第1个，去除大小写
			String getter = "get" + firstLetter + fieldName.substring(1); // 获取方法名
			Method method = o.getClass().getMethod(getter, new Class[] {}); // 获取方法对象
			Object value = method.invoke(o, new Object[] {}); // 用invoke调用此对象的get字段方法
			return value; // 返回值
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	/**
	 * 向ecexl追加数据
	 *
	 * @param excelPath 文件路径
	 * @param list      要添加的对象集合
	 * @param sheetId   页码，往第几页插入
	 * @return
	 * @throws IOException
	 * @throws Exception
	 */
	public static String addExcel(String excelPath, List<? extends Object> list, int sheetId)
			throws IOException, Exception {

		File file = new File(excelPath);
		Workbook workbook = Workbook.getWorkbook(file);
		WritableWorkbook writableWorkbook = Workbook.createWorkbook(file, workbook);
		// 获取指定sheet
		WritableSheet sheet = writableWorkbook.getSheet(sheetId);
		int row = sheet.getRows();// 最大行：使用变量，存下来，避免再次获取（动态变化）
		int i = 0;// 列
		int j = row; // 行： 从这里，开始向后追加数据
		String result = "成功";
		if (list.size() == 0 || list == null) {
			result = "没有对象信息";
		} else {
			Object o = list.get(0);
			Class<? extends Object> clazz = o.getClass();
			Field[] fields = clazz.getDeclaredFields(); // 这里通过反射获取字段数组--（即类定义的属性）

			for (Field f : fields) {
				// 添加列数据
				for (Object obj : list) {
					Object temp = getFieldValueByName(f.getName(), obj);
					String strTemp = "";
					if (temp != null) {
						strTemp = temp.toString();
					}
					sheet.addCell(new Label(i, j, strTemp));
					j++;
				}
				j = row; // 重新回到指定行
				i++;
			}
			writableWorkbook.write();
		}
		writableWorkbook.close();
		workbook.close();
		return result;
	}

}
```
