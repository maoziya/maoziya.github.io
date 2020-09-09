---
title: SpringBoot定时任务保存日志为Ecexl文件
urlname: qh9t1d
date: 2020-06-18 18:48:31 +0800
tags:
  - springboot
  - java方法
categories: java
---

定时任务，保存前天日志数据，形成 Execl 文件

<!--more-->

需求：定时任务，保存前天日志数据，形成 Execl 文件，并生成压缩包，然后删除日志数据
参考代码如下：
注：我的数据库日志表 id 是有顺序的 id 排列，有改动需要自行修改，适配
注：以下代码只能参考，无法直接拿来用，一些涉及自己业务的，必须自行实现
注：ExcelUtil 工具类在我的文章列表中的"Excel 工具类"里面可以复制
还有 ZipUtils"压缩包工具"也可以在我的文章列表里面寻找复制，如果自己有那就不用去找了。
有许多命名并不是很规范，哈哈，写的比较急，管不了那么多了。

```java
package com.mhtech.platform.msrv.sharing.task;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.math.BigDecimal;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFCellStyle;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.formula.functions.T;
import org.apache.poi.ss.usermodel.HorizontalAlignment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.client.OkHttp3ClientHttpRequestFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import com.mhtech.platform.log.pojo.MsrvLogDTO;
import com.mhtech.platform.msrv.sharing.dao.model.AlertLog;
import com.mhtech.platform.msrv.sharing.dao.model.GatewayAccessLog;
import com.mhtech.platform.msrv.sharing.dao.model.MonitorPlans;
import com.mhtech.platform.msrv.sharing.dao.model.MsrvLog;
import com.mhtech.platform.msrv.sharing.dao.model.ServerMonitorLog;
import com.mhtech.platform.msrv.sharing.dao.model.SpUserLog;
import com.mhtech.platform.msrv.sharing.dao.model.UserActionLog;
import com.mhtech.platform.msrv.sharing.request.AlertLogVO;
import com.mhtech.platform.msrv.sharing.request.AllLogVO;
import com.mhtech.platform.msrv.sharing.request.GatewayAccessLogVO;
import com.mhtech.platform.msrv.sharing.request.LogDTO;
import com.mhtech.platform.msrv.sharing.request.MonitorPlansVO;
import com.mhtech.platform.msrv.sharing.request.MsrvLogVO;
import com.mhtech.platform.msrv.sharing.request.ServerMonitorLogVO;
import com.mhtech.platform.msrv.sharing.service.AlertLogService;
import com.mhtech.platform.msrv.sharing.service.IGatewayAccessLogService;
import com.mhtech.platform.msrv.sharing.service.MonitorPlansService;
import com.mhtech.platform.msrv.sharing.service.MsrvLogService;
import com.mhtech.platform.msrv.sharing.service.ServerMonitorLogService;
import com.mhtech.platform.msrv.sharing.service.UserActionDetailService;
import com.mhtech.platform.msrv.sharing.service.UserActionLogService;
import com.mhtech.platform.msrv.sharing.utils.ExcelUtil;
import com.mhtech.platform.msrv.sharing.utils.RedisUtils;
import com.mhtech.platform.msrv.sharing.utils.ZipUtils;

@SuppressWarnings("all")
@Component
public class AllLogRefresher {

	@Autowired
	RedisUtils redisUtils;

	@Autowired
	MsrvLogService mls;// 业务接口1
	@Autowired
	AlertLogService als;// 业务接口2


	// 定时任务，查询(昨天)日志数据，保存日志数据为Ecexl_文件并生成压缩包，然后删除日志数据
	// 注意：一天执行一次，数据没有问题，一天多次执行数据如果发生改变，ecexl将会有之前保存的遗留数据，压缩文件内容也有出入
	@Scheduled(fixedDelay = 1000 * 60 * 60 * 24)
	@Transactional(rollbackFor = Exception.class)
	public void getBlackIpsList() throws Exception {

		// 本地路径
		String url = "D:\\log\\";
		// 压缩包存放路径
		String ysUrl = "D:\\log\\CompressedLog\\";
		// 创建对应路径
		fileCj(url);
		fileCj(ysUrl);
		// 设置文件名上带上日期
		String format =fingTime(-1,"yyyy-MM-dd");// 昨天日期用于设置文件名称
		String fingTime =fingTime(-1,"yyyy-MM-dd 00:00:00");
		String fingDayTime = fingTime(0,"yyyy-MM-dd 00:00:00"); // 今天日期

	    // 查询日志数据生成_ecexl文件
		Map<String, Object> alsMap = als.selectManMin(fingTime, fingDayTime);
		List<String> alsfiles = log(url, alsMap, "Alert_",mapAlertLog());

		Map<String, Object> selectManMin = mls.selectManMin(new LogDTO(fingTime, fingDayTime));
		List<String> msrvfiles = log(url, selectManMin, "Msrv_",mapMsrcLog());

		// 压缩并删除数据库数据
		if (alsfiles.size() > 0) {
			ZipUtils.zipFileAndEncrypt(alsfiles, ysUrl + "Alert日志"+format+".rar", "alert");
			//als.deleteDate(fingTime, fingDayTime); 按时间段删除日志
		}
		if (msrvfiles.size() > 0) {
			ZipUtils.zipFileAndEncrypt(msrvfiles, ysUrl + "Msrv日志"+format+".rar", "msrv");
			//mls.deleteDate(fingTime, fingDayTime);
		}
	}

	/**
	 * 保存日志 每到60万创建一个_ecexl，每个ecexl_生成10页，每页保存6万行数据
	 * @param url 保存路径
	 * @param map 查询条件（）
	 * @param logName 指定生成的日志文件名
	 * @param interpretation 英文字段的中文解释
	 * @return 日志文件名称集合
	 * @throws Exception
	 *
	 * 我因为我的日志表id为有顺序的id，所以下面的findlist方法，我去查询数据库时，按照最大id最小id,然后每次固定取limit xx条数，提高查询速度
	 */
	public List<String> log(String url, Map<String, Object> map, String logName,Map<String, Object> interpretation) throws Exception {

		List<String> list = new ArrayList<>();// 返回日志保存url
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.sss");
		String format =ExcelUtil.fingTime2();//昨天日期

		if (map.get("max") == null && map.get("min") == null) {
			return list;
		}
		Long maxId = (Long) map.get("max");// 要查询的最大值
		Long minId = (Long) map.get("min");// 要查询的最小值
		int sum = Integer.parseInt(map.get("size").toString()); // 要查询的总条数

		int count = 60000;// 每次查询条数
		int topSize = 0; // 累积查询条数
		int cishu = 0; // Execl页码

		do {
			Map<String, Object> maps = findlist(logName, maxId, minId, sum, sdf, count);
			List<? extends Object> findlist = (List<? extends Object>) maps.get("list");
			if (topSize == 0 || topSize % 600000 == 0) {
				if (topSize % 600000 == 0) {
					cishu = 0;// 重置 确保下面追加数据页码正确
				}
				// 生成ecexl
				String name = ExcelUtil.createExcel(findlist, url, logName + topSize / 600000 + "日志" + format,interpretation);
				list.add(name);
			} else {
				String fileUrl = url + logName + topSize / 600000 + "日志" + format + ".xls";// 文件路径
				int row = 0;
				// 追加数据 -不让它一次性插入-避免堆溢出
				while (row < 60000) {
					if (sum < 30000) {
						ExcelUtil.addExcel(fileUrl, findlist, cishu);
						row += 30000;
					} else {
						ExcelUtil.addExcel(fileUrl,
								findlist.subList(row, (sum - row) < 30000 ? row + (sum - 30000) : row + 30000), cishu);
						row += 30000;
					}
				}
			}
			topSize = topSize + findlist.size();
			sum = sum - count;
			cishu++; // 更新页码
			minId = (Long) maps.get("minId");// 更新要查询的最小值
			System.err.println("----------"+logName+"-----还有多少条未插入 :" + sum);
			findlist.clear();
		} while (sum > 0);
		return list;
	}

	/**
	 * 查询日志数据
	 * @param logName 名称-对应生成的日志文件名
	 * @param maxId 最大id
	 * @param minId 最小id
	 * @param sum 总条数
	 * @param sdf 时间字段处理
	 * @param count 查询条数
	 * @return map：返回集合对象与minId
	 */
	public Map<String, Object> findlist(String logName, Long maxId, Long minId, int sum, SimpleDateFormat sdf,int count) {
		Map<String, Object> map = new HashMap<>();
		switch (logName) {
		case "Alert_":
			List<AlertLogVO> alertLog = findAlertLog(als.allLog(maxId, minId, (sum - count) < 0 ? sum : count), sdf);
			Long id3 = alertLog.get(alertLog.size() - 1).getId();
			map.put("minId", id3);
			map.put("list", alertLog);
			return map;
		case "Msrv_":
			List<MsrvLogVO> msrcLog = findMsrvLog(mls.allLog(maxId, minId, (sum - count) < 0 ? sum : count), sdf);// 查询数据
			Long logId = msrcLog.get(msrcLog.size() - 1).getLogId();
			map.put("minId", logId);
			map.put("list", msrcLog);
			return map;
		default:
			return null;
		}
	}

	// 如果文件夹不存在则创建
	public void fileCj(String name) {
		File file =new File(name);
		if  (!file .exists()  && !file .isDirectory()){
		    file .mkdir();
		}
	}

	// findMsrvLog()findAlertLog()这个2方法是因为我的日期字段还为做处理，所以在这个强转了下你可以自行删除，无需调用
	// MsrvLog格式转化
	public List<MsrvLogVO> findMsrvLog(List<MsrvLog> list, SimpleDateFormat sdf) {
		List<MsrvLogVO> log = new ArrayList<>();
		list.forEach(sp -> {
			log.add(new MsrvLogVO(sp.getLogId(), sp.getName(),sdf.format(sp.getCreatedTime()),sp.getXX()));
		});
		list.clear();
		return log;
	}

	// AlertLog格式转化
	public List<AlertLogVO> findAlertLog(List<AlertLog> list, SimpleDateFormat sdf) {
		List<AlertLogVO> log = new ArrayList<>();
		list.forEach(sp -> {
			log.add(new AlertLogVO(sp.getId(), sp.getXX(), sdf.format(sp.getAlertTime())));
		});
		list.clear();
		return log;
	}

    // 这个2个方法，设置对象属性的中文意思，保存在Ecexl的时候，会显示在Ecexl第1行属性必须一一对应
	// 设置MsrcLog字段中文释义
	public Map<String, Object> mapMsrcLog() {
		Map<String, Object> MapMsrcLog = new HashMap<String, Object>();
		MapMsrcLog.put("logId", "主键编码");
		MapMsrcLog.put("name", "保密");
		MapMsrcLog.put("xx", "保密");
		MapMsrcLog.put("xx", "保密");
		return MapMsrcLog;
	}
	// 设置AlertLog字段中文释义
	public Map<String, Object> mapAlertLog() {
		Map<String, Object> alertLog = new HashMap<String, Object>();
		alertLog.put("id", "主键编码");
		alertLog.put("xx", "保密 ");
		alertLog.put("xx", "保密 ");
		return alertLog;
	}

	/**
	 * 获取日期
	 * @param day -1昨天，0今天，1明天
	 * @param sdf 日期格式
	 * @return
	 */
	public static String fingTime(int day ,String sdf) {
		Date date = new Date();
		Calendar calendar = new GregorianCalendar();
		calendar.setTime(date);
		calendar.add(calendar.DATE, day);
		date = calendar.getTime();
		SimpleDateFormat formatter = new SimpleDateFormat(sdf);
		String dateString = formatter.format(date);
		return dateString;
	}

}

```

我在运行中，有遇到以下的 redis 超时异常
org.springframework.dao.QueryTimeoutException: Redis command timed out;
方法一：设置超时时间
方法二：springboot1.x 升级 springboot2.x 需要去更改下版本依赖
解决方案连接：
[https://blog.csdn.net/hello_world_qwp/article/details/81184279](https://blog.csdn.net/hello_world_qwp/article/details/81184279)
[https://blog.csdn.net/single_cong/article/details/101019146](https://blog.csdn.net/single_cong/article/details/101019146)
\_
_\*\*![mjl.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1360300/1593759316804-29ca263b-2671-4a5c-8af2-2f91bfe941e4.jpeg#align=left&display=inline&height=661&margin=%5Bobject%20Object%5D&name=mjl.jpg&originHeight=661&originWidth=567&size=136414&status=done&style=none&width=567)_
\_
_**最后：注意如果你重复运行，日志数据增加了，没关系，文件会被覆盖，文件还是正确的。如果你数据减少了，那样的话，以前多余的数据还会保留在 ecexl 文件里，这个我还未做处理，需要你自己去实现了。最简单粗暴的，就是开始进入方法后，直接先清空这个本地文件夹了。**_
\_
\_
