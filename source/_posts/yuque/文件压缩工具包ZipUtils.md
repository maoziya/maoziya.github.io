---
title: 文件压缩工具包ZipUtils
urlname: cu7e18
date: 2020-06-19 09:58:12 +0800
tags: []
categories: []
---

废话不多说了，参考代码如下，工具来自认识的朋友 GM。

```java
package com.mhtech.platform.msrv.sharing.utils;

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import net.lingala.zip4j.core.ZipFile;
import net.lingala.zip4j.exception.ZipException;
import net.lingala.zip4j.model.ZipParameters;
import net.lingala.zip4j.util.Zip4jConstants;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.StringUtils;

/**
 * 文件压缩工具包，压缩目录时不能嵌套目录
 * <p>{@code zipFileName} 为文件名时，生成的压缩文件在调用者根目录；<br>
 * {@code zipFileName} 参数也可以是文件绝对路径加文件名
 * </p>
 * @author GM
 */
public abstract class ZipUtils {

	private static final Logger logger = LoggerFactory.getLogger(ZipUtils.class);

	/**
	 * 压缩文件
	 * @param file
	 * @param zipFileName
	 */
	public static void zipFile(String file, String zipFileName) {
		zipFileAndEncrypt(file, zipFileName, null);
	}

	/**
	 * 压缩文件并加密压缩包
	 * @param file
	 * @param zipFileName
	 * @param password
	 */
	public static void zipFileAndEncrypt(String file, String zipFileName,
			String password) {
		zipFileAndEncrypt(Arrays.asList(file), zipFileName, password);
	}

	/**
	 * 压缩目录并加密压缩包
	 * @param dir
	 * @param zipFileName
	 * @param password
	 */
	public static void zipDirAndEncrypt(String dir, String zipFileName,
			String password) {
		File $file = new File(dir);
		if(!$file.isDirectory()) {
			throw new RuntimeException(String.format("%s is not a dir", dir));
		}
		String path = $file.getAbsolutePath();
		List<String> files = new ArrayList<String>();
		for (String _file : $file.list()) {
			files.add(path + "\\" + _file);
		}
		zipFileAndEncrypt(files, zipFileName, password);
	}

	/**
	 * 压缩多个文件并加密压缩包
	 * @see #zipFileAndEncrypt(String, String, String)
	 * @param files
	 * @param zipFileName
	 * @param password
	 */
	public static void zipFileAndEncrypt(List<String> files, String zipFileName,
			String password) {
		try {
			// 压缩文件,并生成压缩文件
			ArrayList<File> filesToAdd = new ArrayList<File>();
			for (String $file : files) {
				File file = new File($file);
				if(!file.isFile()) {
					throw new RuntimeException(String.format("%s is not a file", $file));
				}
				filesToAdd.add(file);
			}

			ZipParameters parameters = setParam(password);
			ZipFile zipFile = new ZipFile(zipFileName);
			zipFile.addFiles(filesToAdd, parameters);
		} catch (ZipException e) {
			logger.error("file ziped failed", e);
		}catch (Exception e) {
			logger.error(e.getMessage());
		}
	}

	/**
	 * 以流的形式压缩文件
	 * @param is
	 * @param fileName
	 * @param zipFileName
	 * @param password
	 */
	public static void zipFileStream(InputStream is, String fileName, String zipFileName,
			String password) {
		try {
			ZipParameters parameters = setParam(password);
			parameters.setFileNameInZip(fileName);
			parameters.setSourceExternalStream(true);

			ZipFile zipFile = new ZipFile(zipFileName);
			zipFile.addStream(is, parameters);
		} catch (ZipException e) {
			logger.error("file ziped failed", e);
		}
	}

	private static ZipParameters setParam(String password) {
		// 设置压缩文件参数
		ZipParameters parameters = new ZipParameters();
		// 设置压缩方法
		parameters.setCompressionMethod(Zip4jConstants.COMP_DEFLATE);
		// 设置压缩级别
		parameters.setCompressionLevel(Zip4jConstants.DEFLATE_LEVEL_NORMAL);
		if(!StringUtils.isEmpty(password)) {
			// 设置压缩文件是否加密
			parameters.setEncryptFiles(true);
			// 设置aes加密强度
			parameters.setAesKeyStrength(Zip4jConstants.AES_STRENGTH_256);
			// 设置加密方法
			parameters.setEncryptionMethod(Zip4jConstants.ENC_METHOD_AES);
			// 设置密码
			parameters.setPassword(password.toCharArray());
		}
		return parameters;
	}
}

```
