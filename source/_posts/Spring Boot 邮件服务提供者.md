---
title: Spring Boot 邮件服务提供者
date: 2019-11-03 12:23:26
categories: 笔记
tags: [Spring boot]
type: posts
description: 使用 Spring boot 发送邮件
---
## 依赖 pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>online.cccccc.test</groupId>
    <artifactId>email</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>email</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>
    <developers>
        <developer>
            <id>cqiang</id>
            <name>沐凉</name>
            <email>cqiang102@foxmail.com</email>
        </developer>
    </developers>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>online.cccccc.test.email.EmailApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
## Spring boot 配置文件
```yaml
spring:
  application:
    name: provider-mail
  mail:
    host: smtp.qq.com
    # 你的邮箱授权码
    password: 
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
    # 发送邮件的邮箱地址
    username: 
  thymeleaf:
    cache: false # 开发时关闭缓存,不然没法看到实时页面
    mode: HTML # 用非严格的 HTML
    encoding: UTF-8
    servlet:
      content-type: text/html
# 服务端口号
server:
  port: 82
```
## 测试代码
> `@Async` 使用异步，需要在Spring boot 启动类添加 `@EnableAsync` 
### Controller
```java
/**
 * @author 你是电脑
 * @create 2019/11/3 - 12:45
 */
@RestController
public class EmailController {
    @Resource
    private EmailService emailService;
    @PostMapping("email")
    public String sendEmail(@RequestBody EmailVo emailVo){
        emailService.receive(emailVo.getBody(),emailVo.getTo());
        return "发送成功";
    }
}
```
### Service
```java
/**
 * @author 你是电脑
 * @create 2019/11/3 - 12:44
 */
@Service
public class EmailService {
    @Resource
    private ConfigurableApplicationContext applicationContext;
    @Resource
    private JavaMailSender javaMailSender;
    @Resource
    private TemplateEngine templateEngine;
    @Async
    public void receive(String body,String to) {
        Context context = new Context();
        //往 模板中塞数据
        context.setVariable("test",body) ;
        String emailTemplate = templateEngine.process("index", context);
        sendTemplateEmail("支付宝到账100万元", emailTemplate, to);
    }
    /**
     * 发送普通邮件
     * @param subject
     * @param body
     * @param to
     */
    @Async
    public void sendEmail(String subject, String body, String to) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(applicationContext.getEnvironment().getProperty("spring.mail.username"));
        message.setTo(to);
        message.setSubject(subject);
        message.setText(body);
        javaMailSender.send(message);
    }
    /**
     * 发送 HTML 模板邮件
     * @param subject
     * @param body
     * @param to
     */
    @Async
    public void sendTemplateEmail(String subject, String body, String to) {
        MimeMessage message = javaMailSender.createMimeMessage();
        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            String property = applicationContext.getEnvironment().getProperty("spring.mail.username");
            assert property != null;
            helper.setFrom(property);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(body, true);
            javaMailSender.send(message);
        } catch (Exception e) {
        }
    }
}
```
### 模板文件
```html
<!DOCTYPE html SYSTEM "http://www.thymeleaf.org/dtd/xhtml1-strict-thymeleaf-4.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>你有一封新邮件</title>
</head>
<body>
<h2>测试：<span   th:text="${test}"></span></h2>
</body>
</html>
```
## 使用 Postman 测试接口
![image](http://cccccc.online:8888/group1/M00/00/00/rBAES12-ZAyAWF8EAABuOh0Yb6Y548.PNG)