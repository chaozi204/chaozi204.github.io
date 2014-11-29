---
layout: post
title: "intellij idea 自定义缩写"
date: 2014-11-28 19:02:22 +0800
categories: Intellij
comments: true
keywords: java ide intellij 缩写
categories: 
---
  缩写是开发中利器，能够节约工程师打太多重复的代码好工具，作为一款比较牛叉的开发工具，Intellij当然包含了这个功能，而且还非常丰富，下面就记录如何查找默认定义的这些缩写，以及如何自定义缩写。 说明环境: Intellij IDEA 13 
- 默认缩写在 Preference —> Live Templates 中,如图：
![image](/images/intellji/abbreviation_find.png)

- 自定义Abbreviation（缩写）过程：
 1. 自定义Abbrevatuion，首选要选择它所在的组，比如，选择上图中的other
 2. 点击图中右上角的 ’+’ 号
 3. 选择Live Template
 4. 根据要求输入缩写名称和代表的真实名称
 5. 选择定义的 Applicable Contexts （也就是选择应用到的语言中）
 6. 最后结果如下所示(这里以定义java中的 main 为例 )
 ![image](/images/intellji/abbreviation_custom.png)
 
- 完成