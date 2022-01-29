---
uuid: 3da22f12-92ae-b6ee-99ff-c9e6b9e9458e
title: openmpi with cuda
tags:
  - cuda
  - Linux
categories:
  - 技术
toc: false
date: 2020-03-30 17:10:12
---

openMPI 1.7之后的版本才支持CUDA

### 1.对于1.7~2.0的版本， 配置和编译、安装如下：
```shell
sudo ./configure --prefix=<用户指定的openMPI的安装目录>  --with-cuda=<cuda的include目录> --with-cuda-libdir=<cuda的lib64目录>

sudo make all install
```

### 2.openmpi-2.0以及以上版本
建议安装ucx+gdrcopy来获取新的mpi功能和更好的性能

首先安装gdrcopy，这里建议安装gdrcopy-1.3版本。因为2.0版本会出现如下错误：
![TIM图片20200330170544.png](/images/2020/03/30/ada163c0-7265-11ea-8037-8139a2a0fbbb.png)
安装gdrcopy需要四个依赖：check，check-devel，subunit，subunit-devel，注意下载可执行文件，而不是源码；下载源——[RPM](http://rpmfind.net/)，遇到的问题见博客 《[linux rpm 错误](https://xiao_feng_yang993.gitee.io/2020/03/29/linux-rpm-%E9%94%99%E8%AF%AF/)》
1. 编译gdrcopy
执行如下命令
```bash
make PREFIX=/public/home/asc02/yangxf/local/gdrcopy-1.3  all install
./insmod.sh #root
```
这里不需要指定CUDA=XXX，因为makefile里的路径本身是正确的，指定了反而出现找不见cuda.h的错误。
```makefile
PREFIX ?= /usr/local
DESTLIB ?= $(PREFIX)/lib64
CUDA ?= /usr/local/cuda

GDRAPI_ARCH := $(shell ./config_arch)

CUDA_LIB := -L $(CUDA)/lib64 -L $(CUDA)/lib -L /usr/lib64/nvidia -L /usr/lib/nvidia
CUDA_INC += -I $(CUDA)/include

CPPFLAGS := $(CUDA_INC) -I gdrdrv/ -I $(CUDA)/include -D GDRAPI_ARCH=$(GDRAPI_ARCH)
LDFLAGS  := $(CUDA_LIB) -L $(CUDA)/lib64
COMMONCFLAGS := -O2
CFLAGS   += $(COMMONCFLAGS)
CXXFLAGS += $(COMMONCFLAGS)
LIBS     := -lcudart -lcuda -lpthread -ldl
```
参考[gdrcopy](https://www.kutu66.com//GitHub/article_119493)
2. 编译ucx [ucx](https://www.openucx.org/)
执行命令
```bash
./autogen.sh
./configure --prefix=/public/home/asc02/yangxf/local/ucx --with-cuda=/usr/local/cuda --with-gdrcopy=/public/home/asc02/yangxf/local/gdrcopy-1.3 --with-mlx5-dv --with-avx --with-sse41 --with-sse42
make -j8
make install
```
3. 编译openmpi
这里用的是openmpi-3.0
执行命令
```bash
./configure --prefix=/public/home/asc02/yangxf/local/openmpi-3.0.0 --with-cuda=/usr/local/cuda --with-ucx=/public/home/asc02/yangxf/local/ucx --enable-mpi-cxx
make -j8
make install
```
编译安装完成，在生成的lib目录下可以看到多了关于cuda的库
![cudalib.PNG](/images/2020/03/30/bb12de90-7269-11ea-8ac6-2dde3af873f9.PNG)
把相关路径添加到环境变量就可。
**注：** 对于多机运行的程序，需要将mpi环境变量添加到.bashrc中，否则在其他机器上确实mpi的环境变量。