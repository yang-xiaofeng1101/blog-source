---
title: vscode图形界面在qemu上调试裸机程序
tags:
  - vscode
  - 汇编
  - 调试
categories:
  - 技术
toc: false
date: 2022-02-23 23:14:44
---

## 1. 准备工作
#### 1）从扩展安装ms-vscode.cpptools插件
#### 2）配置文件launch.json
&emsp;&emsp;从顶层目录打开miniOS项目，编辑debug配置文件launch.json，点击debug，创建launch.json文件，输入以下内容，重点在miDebuggerServerAddress设置为qemu对外暴露的调试端口IP填写qemu虚拟机所在的主机IP。program填写调试目标文件路径，绝对路径或相对路径都可，workspace表示项目目录，fileDirname表示当前focus所在编辑页面的文件目录。MIMode填写gdb，miDebuggerPath填写gdb程序的路径，如果是ARM程序需要使用ARM版GDB，x86程序使用x86的GDB。
```json
"version": "0.2.0",
"configurations": [
{
	"name": "kernel-debug",
	"type": "cppdbg",
	"request": "launch",
	"miDebuggerServerAddress": "127.0.0.1:1234",
	"program": "${fileDirname}/kernel.gdb.bin",
	"args": [],
	"stopAtEntry": false,
	"cwd": "${fileDirname}",
	"environment": [],
	"externalConsole": false,
	"logging": {
		"engineLogging": false
	},
	"MIMode": "gdb",
	"miDebuggerPath": "/usr/bin/gdb",
}
]
```
---

## 2. 启动调试
#### 1）首先在命令行界面启动带gdb调试的qemu
```shell
qemu-system-i386 -m 2048 -hda b.img -boot order=a -ctrl-grab -gdb tcp::1234 -S -monitor stdio
```
#### 2) 在vscode中启动调试
可以看到左侧有调试窗口分别是变量，监视，调用堆栈，断点。

C语言部分的调试已经很熟悉了，重点说一下汇编的调试.
&emsp;&emsp;汇编部分要先设置断点后启动vscode调试程序，这点很重要，否则程序将直接执行到C语言的main函数入口。比如要调试kernel.asm的_start函数，则在断点窗口添加_start断点，注意由于汇编程序不能通过鼠标在源码上设置断点，只能在断点窗口输入断点位置后启动debuger程序，程序将停在_start函数入口处。如下图所示：
![image.png](/images/2022/02/23/58492d35-8cf4-40bb-9a18-b27941be8ac3.png)
&emsp;&emsp;虽然停在_start了，可是编辑器的源码上没有任何光标指示，只能在调试控制台根据提示 用-exec disassemble这样的命令来执行GDB指令。
&emsp;&emsp;这样的原因推测是miniOS项目用的是nasm编译器来编译汇编程序，与GDB兼容性低造成的，因为ARM移植工作的miniOS完全采用gcc编译器，就能显示光标在源文件上，如下图所示为使用GCC编译汇编程序的调试效果。
![image.png](/images/2022/02/23/76ca1448-4e7c-40aa-aacc-c782cb1cc299.png)
&emsp;&emsp;虽然x86的汇编上没有光标，但是可以通过反汇编来实现汇编级别的图形化调试，如下所示。可以看到反汇编的指令
![image.png](/images/2022/02/23/9fbd17ad-31b4-478c-87ee-3398bddd1447.png)
![image.png](/images/2022/02/23/581d739f-1841-41cb-9ab4-93bc04cc4a43.png)
&emsp;&emsp;这样还是有问题，发现有几条反汇编的指令与汇编源码不同，这是因为编译和反汇编采用的编译器不同造成的。但除了前2条指令其他反汇编指令还算正常，所以在通过图形界面设置断点的时候不能设置在错误的反汇编指令上，否则是不能起作用的甚至可能奔溃。可以在调试控制台 -exec b xxx来设置正确的断点。
&emsp;&emsp;**总之对于用nasm编译的汇编程序更推荐在调试控制台来调试，反汇编界面只作为参考。**
&emsp;&emsp;即使要用图形界面调试汇编也要**先用-exec ni在调试控制台将反汇编错误的指令过去**了再用图形界面调试。

#### 3）错误的反汇编指令例子：
![image.png](/images/2022/02/23/b811ea13-34e1-45cf-a3d0-2c12f0080b1c.png)
&emsp;&emsp;可以看到当前指令的前面2条指令反汇编结果错误，在反汇编无误的时候才能在图形界面设置断点和单步调试。否则vscode会奔溃。