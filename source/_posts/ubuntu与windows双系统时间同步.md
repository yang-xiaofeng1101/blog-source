---
uuid: fcae918c-b77c-fa8b-9e36-cc5de56fbc1e
title: ubuntu与windows时间同步
tags:
  - Linux
  - time synchronization
categories:
  - 技术
toc: false
date: 2020-03-07 19:57:27
---

@[TOC](ubuntu与windows时间同步（简单易懂版）)
**注：本教程不研究系统计时原理，只简单解决问题，不修改系统核心设置，安全快速解决问题。网络上他人的博客已近提出了专业解决方案，大家可前往查看，在此不做重复。**
https://blog.csdn.net/zero_hzz/article/details/79205037

相信装了windows与ubuntu双系统的用户都会遇见一个问题，那就是两个系统的时间不同步，相差8小时。网上的修改教程复杂难懂，对于阅读困难症患者着实不易。
本人在实践中发现一个新的简单解决方案，在此分享给大家，即使你没有任何计算机基础，也能快速解决问题。

windows与ubuntu时间相差8小时，为什么是8小时而不是其他时间？具体原因详见文章开头链接。这个时候想到我们所学地理知识，中国就处在东8区。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314205321566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODc4NDQy,size_16,color_FFFFFF,t_70)
我们的电脑安装 是也默认了中国区：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314210056511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODc4NDQy,size_16,color_FFFFFF,t_70)
这个时候在回看我们遇到的时间不同步问题，相信聪明的大家已经想到解决办法了，解决办法就是在时间设置中将我们的电脑设置成**0时区**，来抵消时间差，这样便解决了windows与ubuntu的时间问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314205654506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODc4NDQy,size_16,color_FFFFFF,t_70)

哈哈哈，是不是非常简单呢？
如果你只是简单使用ubuntu这样设置就已经可以了，要是准备深入学习还是建议用专业的方法来解决这个问题。