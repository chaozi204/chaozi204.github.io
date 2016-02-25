---
layout: post
title: "hadoop v2 snappy 安装及部署"
date: 2015-11-10 11:19:54 +0800
comments: true
categories: hadoop
keywords: hadoop snappy install 

---
* TOC
{:toc}

> 最近同事有需求要使用在hadoop 2.5.1的集群上使用snappy压缩。因为之前没有使用过snappy，所以没有在现有的集群上配置相关信息，于是乎就在官方和google查了一下相关的配置信息。之所以写这篇文章，主要是因为网上发的文章（尤其是中国的某些网站）的内容不靠谱，没有仔细阅读文档，就随便写一些不经过实践的文章来忽悠大家，导致大家如果根据他们的方式来安装和使用，可能会导致莫名其妙的问题。闲话不说了，写下我的操作过程
<!-- more -->
##### 1 安装 snappy lib库
snappy是google的开源项目，目前代码已经迁移到github上，
地址为：http://google.github.io/snappy/ 。不过比较操蛋的事情是，git上的代码里面没有configure文件，导致我这种c菜鸟不会整了，于是只能在中国的不靠谱网站-csdn找到一个版本，比如我现在用的1.1.2版本。剩下的就是编译安装snappy库了，过程如下(前提你已经安装了相关的依赖库，这里不罗嗦)：</br>

	./configure
	make
	make install
 
这样默认安装snappy库到/usr/local/lib下
 
##### 2 重新编译hadoop
编译前建议仔细阅读Hadoop的BUILDING文件，使用如下命令进行编译

	mvn clean package -Pdist,native -DskipTests -Dtar  -Drequire.snappy  -Dbundle.snappy -Dsnappy.lib=/usr/local/lib
	
利用这个命令编译后的hadoop就会将snappy的native库拷贝到了hadoop/lib/native下面了，并且libhadoop的so文件中也会携带了相关的信息。

#####3 部署
1. 将新编译的hadoop native库下的内容替换线上的。
2. 修改hadoop的配置属性，增加hadoop对snappy encode和decode类的配置
  {% codeblock lang:XML configuration %}
<property>
  <name>io.compression.codecs</name>
  <value>org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec,com.hadoop.compression.lzo.LzoCodec, com.hadoop.compression.lzo.LzopCodec, org.apache.hadoop.io.compress.SnappyCodec </value>
 </property>
  {% endcodeblock %}
3. 重启集群
4. 测试snappy。方法很简单，找一个snappy的文件，利用hadoop fs -text如果能打开则证明安装完成。或者使用hadoop的命令 hadoop checknative -a

#####4 解疑答惑
1. 网上很多文章说还要安装hadoop-snappy，我想说的是，这个是针对hadoop v1的，因为hadoop v2中已经携带了snappy的decode和encode代码了。所以根本不需要安装这个jar，而且安装后使用hadoop checknative -a会报错，错误如下：
{% codeblock 异常信息 %}
Exception in thread "main" java.lang.NoSuchMethodError: 
org.apache.hadoop.io.compress.SnappyCodec.isNativeCodeLoaded()Z
        at org.apache.hadoop.util.NativeLibraryChecker.main(NativeLibraryChecker.java:82)
{% endcodeblock %}

2. 如果snappy为安装好或安装成功，有可能出现的一种问题如下：

{% codeblock 异常堆栈 %}
java.lang.RuntimeException: native snappy library not available: this version of libhadoop was built without snappy support.
        at org.apache.hadoop.io.compress.SnappyCodec.checkNativeCodeLoaded(SnappyCodec.java:64)
        at org.apache.hadoop.io.compress.SnappyCodec.createDecompressor(SnappyCodec.java:201)
        at org.apache.hadoop.io.compress.SnappyCodec.createInputStream(SnappyCodec.java:161)
        at org.apache.hadoop.fs.shell.Display$Text.getInputStream(Display.java:150)
        at org.apache.hadoop.fs.shell.Display$Cat.processPath(Display.java:98)
        at org.apache.hadoop.fs.shell.Command.processPaths(Command.java:306)
        at org.apache.hadoop.fs.shell.Command.processPathArgument(Command.java:278)
        at org.apache.hadoop.fs.shell.Command.processArgument(Command.java:260)
        at org.apache.hadoop.fs.shell.Command.processArguments(Command.java:244)
        at org.apache.hadoop.fs.shell.Command.processRawArguments(Command.java:190)
        at org.apache.hadoop.fs.shell.Command.run(Command.java:154)
        at org.apache.hadoop.fs.FsShell.run(FsShell.java:287)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:84)
        at org.apache.hadoop.fs.FsShell.main(FsShell.java:340)
 {% endcodeblock %}
