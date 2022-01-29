---
uuid: ba62f9dd-5be0-d9a0-f7ff-22c5e96dbea2
title: 给linux终端设置代理
tags:
  - Linux
  - proxy
categories:
  - 技术
toc: false
date: 2020-03-07 17:35:36
---

在安装showdsocks的情况下，浏览器就可以正常实现通过代理访问了，但是对于linux用户来说，终端是个常用的工具，如何给终端设置代理让终端也能通过代理进行访问呢？

#### 1. curl，wget

这俩是常用的命令，实现代理的方式是

###### （1）安装polipo

```
sudo apt-get install polipo
```

###### （2）配置 polipo

```
sudo vim /etc/polipo/config 
```

在它的末尾添加以下命令，或者去掉以下命令前面的注释符号；若此文件不存在，则自己创建

```
socksParentProxy = "localhost:1080"
socksProxyType = socks5 
```

###### （3）重新启动 polipo 服务

```
sudo systemctl restart polipo.service
```

查看状态

```
sudo systemctl status polipo.service
```

得到类似如下结果

```
polipo.service - LSB: Start or stop the polipo web cache
   Loaded: loaded (/etc/init.d/polipo; generated; vendor preset: enabled)
   Active: active (running) since Tue 2019-12-03 03:24:15 CST; 34min ago
     Docs: man:systemd-sysv-generator(8)
  Process: 4799 ExecStop=/etc/init.d/polipo stop (code=exited, status=0/SUCCESS)
  Process: 4806 ExecStart=/etc/init.d/polipo start (code=exited, status=0/SUCCESS)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/polipo.service
           └─4817 /usr/bin/polipo -c /etc/polipo/config pidFile=/var/run/polipo/polipo.pid daemonise=true

12月 03 03:24:15  systemd[1]: Starting LSB: Start or stop the polipo web cache...
12月 03 03:24:15  polipo[4817]: Established listening socket on port 8123.
12月 03 03:24:15  polipo[4806]: Starting polipo: polipo.
12月 03 03:24:15  systemd[1]: Started LSB: Start or stop the polipo web cache.
```

###### （4）终端设置代理环境，如果想长期生效的那么在 `.zshrc` 或者 `.bashrc` 下增加

```
export http_proxy=http://localhost:8123 
export https_proxy=http://localhost:8123
```

polipo 的默认端口时 8123

这时wget和curl都可以通过代理实现访问了，但是想要通过apt安装软件却任然不行

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120304242658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODc4NDQy,size_16,color_FFFFFF,t_70)
#### 2. apt

在apt的配置文件目录下创建代理配置文件：

```
sudo vim /etc/apt/apt.conf.d/10proxy   #这个文件正常不存在，会新建一个
#编辑内容为：
Acquire::http::Proxy "http://localhost:8123";
```

退出终端，重新打开。此时便可以通过代理使用apt安装软件了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191203042855705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODc4NDQy,size_16,color_FFFFFF,t_70)
#### 3. git

git是软件从业者常用的一个软件，世界上最大的开源社区github由于服务器在国外，下载速度非常慢，常常需要使用代理服务。

设置代理

```
git config --global https.proxy http://localhost:8123

git config --global https.proxy https://localhost:8123
```

取消代理

```
git config --global --unset http.proxy

git config --global --unset https.proxy
```
测试效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191203042741286.png)
参考：
[https://www.cnblogs.com/andrewwang/p/9293031.html]