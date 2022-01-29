---
uuid: ada4d1fc-5e37-7332-b23d-c1e5234218a7
title: Dart学习笔记（1）：Hello，world！
tags:
  - Dart
categories:
  - 编程
toc: false
date: 2020-03-20 12:45:43
---

任何一门编程语言的第一节课基本上都会是Hello,world!
估计很少有人会打破这个传统

最初官方推荐的编辑器是Dart Editor
而现在已弃用，改为推荐WebStorm、Atom
编辑器本站并没有提供下载链接 需要的可自行百度，查找最新版本

Dart SDK的安装非常简单， 资源下载 页面提供有墙内下载地址
将zip文件解压到任意目录，并将bin目录添加至PATH环境变量即可

在新建工程的时候，WebStorm 提供有多个工程模板
这里选择命令行应用
![](http://www.cndartlang.com/wp-content/uploads/2017/07/6.jpg)
编辑bin/main.dart，同C++一样，Dart程序是从main()函数开始执行的
在main.dart中添加如下代码：
```dart
void main()
{
  print("Hello, world!");
}
```
添加完之后点击工具栏的绿色运行按钮，或者dart文件右键菜单中的 Run 菜单
便能够在命令行界面看到运行结果了
![](http://www.cndartlang.com/wp-content/uploads/2017/07/7.jpg)
顺带一提，Dart有 两种运行模式：

- 检查模式（checked）：进行类型检查，如果发现实际类型与声明或期望的类型不匹配就报错
- 生产模式（production）：不进行类型检查，忽略声明的类型信息，忽略 assert 语句

检查模式运行较慢，生产模式运行快
但检查模式可以及早地发现程序在的问题，所以建议在开发过程中使用检查模式
而在正式环境中使用生产模式运行

Dart VM 默认在生产模式下运行，而我们用 WebStorm 开发时默认在检查模式下运行
通过 Run—>Edit Configurations 选项可以设置使用不同的模式
![](http://www.cndartlang.com/wp-content/uploads/2017/07/8.jpg)