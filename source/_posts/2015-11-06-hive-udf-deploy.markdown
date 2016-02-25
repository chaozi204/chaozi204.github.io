---
layout: post
title: "hive udf 部署方式小结"
date: 2015-11-06 18:37:17 +0800
comments: true
categories: hive
---
### Hive Hook 部署方式：
	本方式为我根据hive的hook的特性开发一种部署方式，优点是针对单个客户端，
	无需修改HIVE的源码，纯外接的方式来完成udf的部署，比临时udfs方便快捷且永久，且容易管理。
	每个组或每个团队之间的udf彼此不受影响。
	相信安装部署见：https://github.com/chaozi204/hive-udf-hook

 <!-- more -->

### 临时UDF方法：
	这个是最常见的Hive使用方式，通过hive的命令来完成UDF的部署
	hive> add jar xxxx.jar
	hive> CREATE TEMPORARY FUNCTION function_name AS 'udf类路径';
这种方法的的缺点很明显就是每次都需要使用这两个命令来完成操作

### 永久部署UDF方法
	这种方式是 hive 0.13版本以后开始支持的注册方法，详情可以参考官方说明。
	使用方式下如：
	CREATE FUNCTION [db_name.]function_name AS [USING JAR|FILE|ARCHIVE 'file_uri' [, JAR|FILE|ARCHIVE 'file_uri'] ];
    例如：
    CREATE FUNCTION udfDecoder AS  'com.sohu.tv.udf.udfDecoder' USING JAR 'hdfs://nn2.tvhadoop.sohuno.com/user/hive/udf/vr-hive-udf-0.0.1.jar'

这种方法的优点为全局可见，一次添加完成即可永久使用。支持数据库级别的函数名称。之所以能够永久性的部署，是因为hive将函数的数据存储到了数据库表FUNCS和FUNC_RU中。通过desc function function_name命令也可以知道这个函数是否是通过永久方式注册的
