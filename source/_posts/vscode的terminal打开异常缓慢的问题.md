---
title: vscode的terminal打开异常缓慢的问题
tags:
  - Linux
  - vscode
categories:
  - 技术
toc: false
date: 2020-06-11 23:39:05
---

## 1.问题频现
在使用了Linux的vscode后就经常发生内置terminal无法打开的问题：
![批注 20200611 233258.png](/images/2020/06/11/2fba40ed-4769-4a27-b529-60d4fcb56835.png)
## 2.解决无门
在百度更换多个关键词查找后没有有价值的信息
![批注 20200611 233544.png](/images/2020/06/11/72b03f4b-4a69-4ad8-9baa-3c91ab4b53e6.png)

## 3.出现转机
在无边无际的搜索结果中一次偶然看到了解决方法：
![批注 20200611 233322.png](/images/2020/06/11/f724c8e6-a7f4-4f74-bf9b-6a755e2576bf.png)
只要打开这项设置，就能解决问题
![批注 20200611 233343.png](/images/2020/06/11/33d3a53b-70e5-4b5a-92c0-35b723c299b8.png)
关掉设置后，又会出现之前的问题
设置信息可知：新的shells是否从vscode继承继承环境变量