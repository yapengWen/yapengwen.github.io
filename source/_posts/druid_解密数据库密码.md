---
title: 配置Druid解密数据库密码
date: 2018-10-17 11:37:00
tags: [DRUID]
categories: java
---

首先我们得下载一个druid-1.0.16.jar的包

其次键入命令&nbsp;java -cp druid-1.0.16.jar com.alibaba.druid.filter.config.ConfigTools your_password

这时候会生成privatekey，publickey，以及password，相关的截图如下

![](https://yapengwen.github.io/img/20161123020626355.png)


注意1：如果使用的不是druid-1.0.16.jar可能只会生成一个password。

注意2：如果没配置好，可能会报一大推奇奇怪怪的错误，比如：**org.springframework.beans.factory.BeanCreationException:&nbsp;Error&nbsp;creatingbean&nbsp;with name 'shiroFilter':....**



配置Druid解密数据库密码

```java
jdbc.type=mysql
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/jesng?useUnicode=true&amp;characterEncoding=utf-8
jdbc.username=root
jdbc.password=IVpkS/WvZQKLcm4+f7xlLFo5FzxGIj3O1br9TcvLlq2a17mmt0SWe9Qq1hyVKsnbsRdU6FKTItc6vVIF9RRpTw==
jdbc.publickey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKYBLQ067pCDwEfysD6rAIWZD4C2K7BO09NFYMmA+VD4i+28znGk9F3w3uCFp6vYf633rPJpx+hoHU/+9gBIewUCAwEAAQ==
```


配置Druid解密数据库密码的主要新增的两行配置。

```java
1、
<property name="connectionProperties"
value="druid.stat.slowSqlMillis=5000;config.decrypt=true;config.decrypt.key=${jdbc.publickey}"/>

作用：配置ConfigFilter解密密码  ，注意出的publickey对应
2、<property name="filters" value="config" />  
作用：提示Druid数据源需要对数据库密码进行解密  
```
