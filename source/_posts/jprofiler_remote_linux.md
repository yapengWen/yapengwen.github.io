---
title: jprofiler_监控远程linux服务器的JVM进程
date: 2017-12-05 11:37:00
tags: [jprofiler,linux]
categories: java
---

### jprofiler_监控远程linux服务器的JVM进程(实践)
几天前写了一篇文章，[jprofiler_监控远程linux服务器的tomcat进程(实践)](http://www.cnblogs.com/gossip/p/6090979.html)，介绍了使用jprofiler怎样监控远程linux的tomcat进程，这两天想了想，除了可以监控tomcat进程，是不是也可以监控其它的Java进程，可是找了一圈都是监控tomcat，于是就打算亲手实验一下本文打算把一个简单的java程序打包成jar包，并部署到linux服务器运行，然后使用jprofiler监控该jar包java程序可参考文章：[Java部署_IntelliJ创建一个可运行的jar包（实践）](http://www.cnblogs.com/gossip/p/6093705.html)</div><div>
**操作步骤如下：**  
1、将jar包拷贝到linux并运行  
![](https://yapengwen.github.io/img/35158-20161123152222393-97209354.png)  
2、使用jprofiler的``jpenable``工具对jar包进行监控（需要先在linux安装解压jprofiler）  
jpenable目录：/usr/local/jprofiler9/bin  
最后输出：You can now use the JProfiler...代表运行成功![](https://yapengwen.github.io/img/35158-20161123152222784-1667486593.png)  
3、JProfiler启动远程监控  
Session-->NewSession![](https://yapengwen.github.io/img/35158-20161123152223300-1537569135.png)  
监控到对象![](https://yapengwen.github.io/img/35158-20161123152223893-970376784.png)
