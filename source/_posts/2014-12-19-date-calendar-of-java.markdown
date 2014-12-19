---
layout: post
title: "Date与Calendar对于周信息表示的差异"
date: 2014-12-19 23:02:20 +0800
comments: true
categories: Java
keywords: Java Date Calerdar week

---

>    最近在做一个项目中，同时使用了Java自身携带的java.util.Date类，和java.util.Calendar类来表示和处理时间。项目期间遇见了一个时间上的bug，主要是表示周几的问题上出现了差异。因为Java中，Date类的getDay方法，和Calendar.get(Calendar.DAY_OF_WEEK)返回的周信息表示分别为:

  + Date： 周日(0),周一(1),周二(2),周三(3),周四(4),周五(5),周六(6)
  + Calendar： 周日(1),周一(2),周二(3),周三(4),周四(5),周五(6),周六(7)
<!-- more -->
  因为我用数字来表示周几，且出现了混用两个类，导致了bug的出现。测试代码:
  
  ``` java
  public static void dateDesc(){
    Date date = new Date();
    int week1 = date.getDay();
    Calendar calendar = Calendar.getInstance();
    int week2 = calendar.get(Calendar.DAY_OF_WEEK);
    System.out.println("Week1="+week1);
    System.out.println("Week2="+week2);
  }
  ```
