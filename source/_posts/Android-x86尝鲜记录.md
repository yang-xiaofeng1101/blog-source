---
uuid: fa437dcf-b498-2890-057c-b4d5e5d8f84d
title: Android x86尝鲜记录
tags:
  - Linux
  - Android
originContent: ''
categories:
  - 技术
toc: false
date: 2020-03-09 17:44:36
---

## 1. 下载系统镜像包
[Androidx86_64 9.0](https://mirrors.xtom.com.hk/osdn//android-x86/71931/android-x86_64-9.0-r1.iso)
百度搜索出的镜像站位于香港，下载速度比较慢。
想到国内也有类似的镜像源——[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn)，就去找了找，起初并没有找见Androidx86。索性安卓香港镜像来按图索骥，先进到osdn目录，结果跟平常页面不一样[OSDN](https://mirrors.tuna.tsinghua.edu.cn/osdn/)。里面的链接点击后会直接跳转到OSDN官网，跑到了国外。跟加速的理念不符。在香港的镜像站目录逐步回退，也是这个样子，毫无头绪。
不经意间突然想到，为何不直接把香港站的后续路径直接复制过来看是什么情况？说干就干，结果就找见了位于清华镜像站的文件[Android x86](https://mirrors.tuna.tsinghua.edu.cn/osdn/android-x86/)；
![tuna.PNG](/images/2020/03/09/ad9743a0-61e8-11ea-9477-c36447aa7d4d.PNG)
根据不同的版本号选择自己需要下载的版本，下载下来就可以进行安装。
我选择了最新版的64位安装包
![tuna2.PNG](/images/2020/03/09/d465cdd0-61e8-11ea-9477-c36447aa7d4d.PNG)
## 2. 安装操作系统
Android x86的安装与一般的Linux安装没有什么区别，都是那么个流程
![timg.jfif](/images/2020/03/09/33b11920-61e9-11ea-9477-c36447aa7d4d.jfif)
选择installation，选择安装盘即可。
然后重启，重启画面如下：
![Android.jfif](/images/2020/03/09/5953f490-61e9-11ea-9477-c36447aa7d4d.jfif)

运行界面：
![Cache_342ffdb37a778964..jpg](/images/2020/03/09/a60ff7c0-61e9-11ea-9477-c36447aa7d4d.jpg)
所有应用：
![Cache_38dc1b1137f219ff..jpg](/images/2020/03/09/b2308560-61e9-11ea-9477-c36447aa7d4d.jpg)

## 3. 尝试使用
(1) 在浏览器下载了QQ，安装运行无大碍
(2) 安装腾讯会议，卡在启动界面无法进入
(3) 安装和平精英游戏，无法运行
(4) 安装淘宝可正常运行
(5) 不能拨打电话发短信

## 4. 总结
Androidx86下无法运行多数应用，跟手机在兼容性无法相提并论。但是性能要好，安装应用速度非常快。用了几个小时就卸载了，装回deepin。
