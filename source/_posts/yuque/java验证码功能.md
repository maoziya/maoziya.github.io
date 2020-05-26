---
title: java验证码功能
urlname: maegsc
date: 2020-05-26 17:25:32 +0800
tags: []
categories: []
---

简单代码：

@GetMapping("/verification")
@ApiOperation("验证码")
public void verification(HttpServletResponse resp) throws FileNotFoundException, IOException {
 int w = 70;
 int h = 35;
 Random r = new Random();//创建一个新的随机数生成器
 BufferedImage image = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
 Graphics2D g2 = (Graphics2D) image.getGraphics();
 g2.fillRect(0, 0, w, h);
 // 向图片中画 4 个字符
 StringBuffer sb = new StringBuffer(6);
 for (int i = 0; i < 4; i++) { // 循环四次每次生成一个字符
  String s = r.nextInt(10) + "";// 随机生成一个字母
  sb.append(s);
  float x = i _ 1.0F _ w / 4; // 设置当前字符的 X 轴坐标
  g2.setColor(Color.BLUE);
  g2.drawString(s, x, h - 5); // 画图
  // 增加线条干扰
  g2.setColor(Color.YELLOW);
  g2.drawLine(r.nextInt(20), r.nextInt(50), r.nextInt(50), r.nextInt(50));
  g2.setColor(Color.RED);
  g2.drawLine(r.nextInt(20), r.nextInt(50), r.nextInt(50), r.nextInt(50));
  g2.setColor(Color.GREEN);
  g2.drawLine(r.nextInt(20), r.nextInt(50), r.nextInt(50), r.nextInt(50));
 }
 ByteArrayOutputStream baos = new ByteArrayOutputStream();
 ImageIO.write(image, "JPEG", baos);
 byte[] byteArray = baos.toByteArray();
 // 存放图片验证码 唯一编码
 String id = UUID.randomUUID().toString();
 redisUtils.hset("verification", id, sb, 1000*60*2);//redis 缓存
 resp.setHeader("RandomUUID", id);// 返回给前端，无法返回数据，直接添加在 header 中
 resp.setHeader("Access-Control-Expose-Headers", "RandomUUID");
 resp.getOutputStream().write(byteArray);
 resp.setContentType("image/jpeg;charset=UTF-8");
}

更新测试！！
第 5 次测试！！
第 N 次，，1232132，测试啊啊啊啊啊！！
未完成，测试语雀自动推送 github
