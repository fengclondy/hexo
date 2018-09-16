---
layout: post
title: 图片验证码的实现
date: 2018-08-08 06:55:06
tags: util
categories: util
---

>以springBoot项目为例
>kaptcha 是生成验证码的一个工具类，其原理是将随机生成字符串保存到session中，同时以图片的形式返回给页面，之后前台页面提交到后台进行对比。

### 图片验证码

1、添加依赖：
```xml
<dependency>
    <groupId>com.github.axet</groupId>
    <artifactId>kaptcha</artifactId>
    <version>0.0.9</version>
</dependency>

```

2、定义KaptchaConfiguration：
```java
@Configuration
public class KaptchaConfiguration {

	private static final String KAPTCHA_BORDER = "kaptcha.border";
	private static final String KAPTCHA_TEXTPRODUCER_FONT_COLOR = "kaptcha.textproducer.font.color";
	private static final String KAPTCHA_TEXTPRODUCER_CHAR_SPACE = "kaptcha.textproducer.char.space";
	private static final String KAPTCHA_IMAGE_WIDTH = "kaptcha.image.width";
	private static final String KAPTCHA_IMAGE_HEIGHT = "kaptcha.image.height";
	private static final String KAPTCHA_TEXTPRODUCER_CHAR_LENGTH = "kaptcha.textproducer.char.length";
	private static final Object KAPTCHA_IMAGE_FONT_SIZE = "kaptcha.textproducer.font.size";

	/**
	 * 默认生成图形验证码宽度
	 */
	private static final String DEFAULT_IMAGE_WIDTH = "100";

	/**
	 * 默认生成图像验证码高度
	 */
	private static final String DEFAULT_IMAGE_HEIGHT = "40";

	/**
	 * 默认生成图形验证码长度
	 */
	private static final String DEFAULT_IMAGE_LENGTH = "4";
	/**
	 * 边框颜色，合法值： r,g,b (and optional alpha) 或者 white,black,blue.
	 */
	private static final String DEFAULT_COLOR_FONT = "black";

	/**
	 * 图片边框
	 */
	private static final String DEFAULT_IMAGE_BORDER = "no";
	/**
	 * 默认图片间隔
	 */
	private static final String DEFAULT_CHAR_SPACE = "5";
	/**
	 * 验证码文字大小
	 */
	private static final String DEFAULT_IMAGE_FONT_SIZE = "30";

	@Bean
	public DefaultKaptcha producer() {
		Properties properties = new Properties();
		properties.put(KAPTCHA_BORDER, DEFAULT_IMAGE_BORDER);
		properties.put(KAPTCHA_TEXTPRODUCER_FONT_COLOR, DEFAULT_COLOR_FONT);
		properties.put(KAPTCHA_TEXTPRODUCER_CHAR_SPACE, DEFAULT_CHAR_SPACE);
		properties.put(KAPTCHA_IMAGE_WIDTH, DEFAULT_IMAGE_WIDTH);
		properties.put(KAPTCHA_IMAGE_HEIGHT, DEFAULT_IMAGE_HEIGHT);
		properties.put(KAPTCHA_IMAGE_FONT_SIZE, DEFAULT_IMAGE_FONT_SIZE);
		properties.put(KAPTCHA_TEXTPRODUCER_CHAR_LENGTH, DEFAULT_IMAGE_LENGTH);
		Config config = new Config(properties);
		DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
		defaultKaptcha.setConfig(config);
		return defaultKaptcha;
	}
}
```

3、添加处理类：
```java
@ReController
public class KcaptchaController {

    @Autowired
    private Producer captchaProducer;

    @GetMapping("getKaptchaImage")
    public void getKaptchaImage(HttpServletRequest request, 
            HttpServletResponse response) throws Exception {

        response.setDateHeader("Expires", 0); 

        // Set standard HTTP/1.1 no-cache headers. 
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");  
        // Set IE extended HTTP/1.1 no-cache headers (use addHeader). 
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");  
        // Set standard HTTP/1.0 no-cache header. 
        response.setHeader("Pragma", "no-cache");  
        // return a jpeg 
        response.setContentType("image/jpeg");  
        // create the text for the image
        String capText = captchaProducer.createText();  
        // store the text in the session 
        request.getSession().setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);  
        // create the image with the text  
        BufferedImage bi = captchaProducer.createImage(capText);  
        ServletOutputStream out = response.getOutputStream();  
        // write the data out  
        ImageIO.write(bi, "jpg", out);  
        try {  
            out.flush();  
        } finally {  
            out.close();  
        }  
    }
}
```

