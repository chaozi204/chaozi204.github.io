---
layout: post
title: "high syscpu 问题排查经历"
date: 2016-02-24 23:10:10 +0800
comments: true
categories: java
keywords: high,syscpu, system cpu,pidstat , thread
---

#### 问题描述

>  最近工作中发现有一些的应用程序导致系统的syscpu比较高，大概30%以上，对于我所在的公司来说，这么较高的sys cpu基本反映了当时运行的应用程序中应该有存在问题，如图:
![image](/images/my/syscpu-high-1)
<!-- more -->

#### 问题排查 ####

> 1 、利用vmstat 命令检查当前系统的状态。判断系统的上下文切换和软终端等统计数据，这些状态数据容易导致sys cpu较高
{%  codeblock lang:bash %}
vmstat  1
{%  endcodeblock %}
![image](/images/my/syscpu-high-2.png)
 途中可以看出来上下文切换时比较多，先假设是又由于部分程序线程切换和竞争导致，因此继续排查是哪些程序占用syscpu

> 2、利用pidstat，查找syscpu比较高的程序的进程号pid
{%  codeblock lang:bash %}
pidstat -u 1
{%  endcodeblock %}
![image](/images/my/syscpu-high-3.png)
因此发现14744 这个程序的system cpu比较高，下面就需要排查14744这个程序那个线程导致比较高的sys cpu

> 3、利用pidstat，查找特定程序下消耗syscpu比较高的线程id
{%  codeblock lang:bash %}
pidstat -t -p 14744 1
{%  endcodeblock %}
![image](/images/my/syscpu-high-4.png)
这时已经定位到15497这个线程id，有比较高的sys cpu比较高，这个线程id是10进制的，这里需要转换为16进制，用以在下面的jstack的数据中查找对应的线程信息。（16进制数—3c89）

> 4、 利用 jstack打出线程堆栈
{%  codeblock lang:bash %}
   jstack 14744 > 14744
{%  endcodeblock %}

> 5、利用转换的16进制线程id，在14744中堆栈信息中查找特定的线程
![image](/images/my/syscpu-high-5.png)
到此，整个定位就排查完成，最后是因为队列锁等待导致sys cpu高。

