---
layout: post
title: "在线设置 log4j 日志级别"
date: 2016-02-08 22:19:14 +0800
comments: true
categories: log4j
keywords: log4j dynamically set logger level,log4j,online
---
* TOC
{:toc}

#### 在线设置Log4j 日志级别 ####
 hadoop RM,NN,DN,NM等支持动态在线设置log4j的日志级别，参考：[daemonlog](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/CommandsManual.html#daemonlog)

改功能的原理是，利用log4j的Logger API中setLevel方法，在线修改当前Logger使用的日志级别。然而，这种方式并不适用所有的log4j的实现，它只能适用于Logger类中对外开放setLevel方法的，目前知道的两个logger的实现带有这个方法：
<!-- more -->
* org.apache.log4j.Logger
* java.util.logging.Logger  

我在实际开发工作中，经常会需要在线的修改日志级别来方便调试和问题排查，因此我根据hadoop对这个功能的实现进行了抽取，独立形成一个简单的小工具，用以在实际开发中方便的使用logg4j的在线动态修改日志级别(采用restful方式，进行修改)。例子参考：
[com.lee.knife.runtime.rest.LOGTest](https://github.com/chaozi204/grindstone/tree/master/knife)


#### log4j 配置属性
 Log4j的配置文件log4j-properties,支持预先声明属性var,并通过 ${var} 的方式使用如：
 {% codeblock 配置样例 %}
 ## 这里定义的logs.dir属性,后面使用${logs.dir}的方使用
 logs.dir=./logs
 log4j.appender.DRFAppender=org.apache.log4j.DailyRollingFileAppender
 log4j.appender.DRFAppender.DatePattern='.'yyyy-MM-dd-HH
 log4j.appender.DRFAppender.File=${logs.dir}/server.log
 log4j.appender.DRFAppender.layout=org.apache.log4j.PatternLayout
 log4j.appender.DRFAppender.layout.ConversionPattern=%-5p %c.%M [%t] - %m%n
 {% endcodeblock %}

 上诉配置表示默认情况下，日志存储路径就是java工作目录下的logs目录（没有则创建）。

设置属性var的值，一般有两种方式：

 1.  通过修改log4j.properties文件，将属性 logs.dir=./logs 修改为新的路径地址logs.dir=/x1/x2
 2. 通过jvm options的方式修改log4j的属性的值，而且通过 jvm options 的方式修改的值优先于 log4j.properties文件内定义的值。也就是说如果jvm的启动options中带有 -Dlogs.dir=xx ，那么这个值会优先被 log4j 系统使用（而不会再考虑配置文件中同样定义的属性）