---
uuid: ec424ad8-03f4-e6f8-d932-858c6b50cb71
title: GCC离线升级
tags:
  - Linux
  - gcc
categories:
  - 技术
toc: false
date: 2020-03-10 17:48:38
---

## 1. 下载gcc安装包
1. [gcc](https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/)
选择自己想要的版本并下载
我下载的是[7.5.0](https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.5.0/gcc-7.5.0.tar.gz)版本.然后解压。
2. 打开contrib/download_prerequisites查看依赖![yilai.PNG](/images/2020/03/10/3ecf4650-62b7-11ea-adaa-3dbf80c3d354.PNG)
可以看到这里要求了四个依赖包：[gmp](https://mirrors.tuna.tsinghua.edu.cn/gnu/gmp/)，[mpc](https://mirrors.tuna.tsinghua.edu.cn/gnu/mpc/)，[mpfr](https://mirrors.tuna.tsinghua.edu.cn/gnu/mpfr/)，[isl](http://www.mirrorservice.org/sites/sourceware.org/pub/gcc/infrastructure/)。下载的版本可以不是最新版，但是不能太旧，一般从现在往后退几代就可以了，具体要求gcc的configure时会说明。额外注意如果采用tree编译法要求mpc>=1.0.3。
## 2. 安装准备与配置
1. 下载完成后，将gmp、mpfr、mpc、isl安装放到cd /usr/local/gcc-7.1.0目录下并解压：

      tar -xf gmp-6.1.0.tar.bz2

      tar -xf mpfr-3.1.4.tar.bz2

      tar -xf mpc-1.0.3.tar.gz

      tar -xf isl-0.16.1.tar.bz2  
2. 建立软连接：

     ln -sf gmp-6.1.0 gmp

     ln -sf mpfr-3.1.4 mpfr 

     ln -sf mpc-1.0.3 mpc

     ln -sf isl-0.16.1 isl
## 3. 编译安装：
1. 创建build目录，不要在源码root目录下直接build，否则无法进行树形编译
2. 编译命令
```language
../configure --prefix=xxx -enable-checking=release -enable-languages=c,c++ -disable-multilib
make -j
make install #安装到指定的prefix目录下
```
## 4. 其他编译方式
1. 如果不采用tree编译的方式，那么需要分别编译这几个依赖包，而且顺序固定，依次编译。
	1. gmp安装：

		tar jxf gmp-4.3.2.tar.bz2

		cd gmp-4.3.2

		./configure --prefix=/usr/local/gmp-4.3.2 && make

		make install
 
	2. mpfr安装：

		tar jxf mpfr-2.4.2.tar.bz2

		cd mpfr-2.4.2

		./configure --prefix=/usr/local/mpfr-2.4.2 --with-gmp=/usr/local/gmp-4.3.2 && make

		make install

	3. mpc安装：

		tar zxvfv mpc-1.0.1.tar.gz

		cd mpc-1.0.1

		./configure --prefix=/usr/local/mpc-1.0.1 --with-gmp=/usr/local/gmp-4.3.2 --with-mpfr=/usr/local/mpfr-2.4.2 && make

		make install

 

2. 添加环境变量

	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/gmp-4.3.2/lib:/usr/local/mpc-1.0.1/lib:/usr/local/mpfr-2.4.2/lib

 

3. 安装gcc-5.4.0

	tar -xzvf gcc-5.4.0.tar.gz

	cd gcc-5.4.0

	mkdir gcc-build    //创建编译目录

	cd gcc-build

	../configure --prefix=/usr/local/gcc-5.4.0 --enable-threads=posix --disable-checking --disable-multilib --enable-languages=c,c++ --with-gmp=/usr/local/gmp-4.3.2 --with-mpfr=/usr/local/mpfr-2.4.2 --with-mpc=/usr/local/mpc-1.0.1           //执行配置

	make -j4   //多核编译，过程极其漫长～～～

	make install

这种方式我在编译gcc的时候失败了。还是tree编译方法方便