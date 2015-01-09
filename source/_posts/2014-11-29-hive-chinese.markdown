---
layout: post
title: "Hive正则匹配中文问题"
date: 2014-11-29 17:34:37 +0800
comments: true
categories: hive
description: somethings for hive work
keywords: hive 正则 中文

---

利用Hive的正则匹配中文时需要注意：
 
 * 中文的字符集合为[\u4e00-\u9fa5]
 * 但是hive在hive执行中会被转义，因此需要增加一次java的转义字符才能够正确使用
 
例如：
`select title from vid_title where type='my' and title rlike '^[\\\u4e00-\\\u9fa5]{1,2}$' `
