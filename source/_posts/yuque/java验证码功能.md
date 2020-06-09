---
title: java验证码功能
urlname: maegsc
date: 2020-05-26 21:23:12 +0800
tags: []
categories: []
---

<!--more-->

思路：
生成一个 UUID，将 4 位数验证码与 id 绑定，存入 redis,并设置有效时间。登录时前端传入 UUID，然后你去             redis 查询匹配的验证码，是否存在，是否正确。

说明：
UUID 确保是同一用户操作，避免多人请求，无法确定当前验证码是那个用户的请求
redis 有效时间，删除掉无用的验证码数据
下方代码中，redisUtils 工具需要你自己编写
接口返回的 UUID 直接放入了 Header 了中，因为我不想保存验证码的图片，所以是直接返回了图片，但是又需要 返回这个 UUID.所以遇到了一个问题怎么返回：图片+数据，换了个方式：直接放入请求头中。

_**代码如下：**_

```java
// 验证码
@GetMapping("/verification")
public void verification(HttpServletResponse resp) throws FileNotFoundException, IOException {
        int w = 70;
        int h = 35;
        Random r = new Random();//创建一个新的随机数生成器
        BufferedImage image = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
        Graphics2D g2 = (Graphics2D) image.getGraphics();
        g2.fillRect(0, 0, w, h);
        // 向图片中画4个字符
        StringBuffer sb = new StringBuffer(6);
        for (int i = 0; i < 4; i++) { // 循环四次每次生成一个字符
            String s = r.nextInt(10) + "";// 随机生成一个字母
            sb.append(s);
            float x = i * 1.0F * w / 4; // 设置当前字符的X轴坐标
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
        redisUtils.hset("verification", id, sb, 1000*60*2);//redis缓存
        resp.setHeader("RandomUUID", id);// 返回给前端，无法返回数据，直接添加在header中
        resp.setHeader("Access-Control-Expose-Headers", "RandomUUID");
        resp.getOutputStream().write(byteArray);
        resp.setContentType("image/jpeg;charset=UTF-8");
}

```
