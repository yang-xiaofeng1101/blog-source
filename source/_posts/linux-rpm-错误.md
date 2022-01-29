---
uuid: ba1b4074-65b3-f5d1-e3f0-fc563a7777ce
title: linux rpm 错误
tags:
  - Linux
categories:
  - 技术
toc: false
date: 2020-03-29 21:27:59
---

## linux 使用rpm安装软件时,遇到"warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY" 错误
#### 建议的做法：

warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY 
 
网上资料说这是由于yum安装了旧版本的GPG keys造成的
 ```shell
rpm --import /etc/pki/rpm-gpg/RPM*
```

#### 不建议的方法如下：
1、安装时提示：warning: *.rpm: Header V3 RSA/SHA256 Signature, keykey ID c105b9de: NOKEY

解决的方法就是在rpm 语句后面加上 --force --nodeps 即原本为 rpm -ivh *.rpm 现在改成 rpm -ivh *.rpm --force --nodeps就可以了。 nodeps的意思是忽视依赖关系。因为各个软件之间会有多多少少的联系。有了这两个设置选项就忽略了这些依赖关系，强制安装或者卸载

2、尝试卸载： 造成这个问题的主要原因是套件被重複 (强制) 安装了两次以上. 尝试了--nodeps, --force, --justdb都不行。 结果碰巧解决! 通过man rpm，发现--allmatches应该可以解决这个问题. [root@testserver openssl-0.9.8l]# rpm -e --allmatches --nodeps openssl*

## CentOS系统bash: groupadd: command not found问题
如果我们需要在CentOS执行新建用户组命令的时候，需要进入到ROOT权限，如果你用以下命令：
```shell
su
su root
```
进入到ROOT账户，那么会出现上述的错误信息：“bash: groupadd: command not found”

这是因为执行这个两个进入ROOT命令不会把你的PATH环境变量带过去，你需要执行命令：
```shell
su - root
```
这样子进入ROOT权限，执行groupadd或者useradd命令就不会有问题了。