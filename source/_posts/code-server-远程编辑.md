---
title: code-server 远程编辑
tags:
  - Linux
categories:
  - 技术
toc: false
uuid: 961e60bb-8587-6d3e-0785-6691eddd8994
date: 2020-05-13 18:11:42
---

## 1. 背景
随着项目规模扩大，个人计算机已经无法满足测试需求，需要在服务器上运行项目程序。当需要对源码进行修改的时候，就非常麻烦，先下载再上传或者使用vim直接编辑，许多功能都没有。一次偶然发现了code-server这个软件，将它部署在服务器上，就可以直接在本地编辑远程文件了，而且具有vscode的所有功能。
## 2. 安装
常见的安装方法见链接：
[知乎](https://zhuanlan.zhihu.com/p/62570740?utm_source=ZHShareTargetIDMore&utm_medium=social&utm_oi=41299306610688)

1. 文中提到需要下载code-server二进制包，位于github上，由于买不起代理，一直下载失败。但是在之前为Windows10里的wsl配置code-server时可以直接在wsl里./code自动下载数据包，下载源位于微软的服务器，速度快。根据这一特性，但是找了半天也没找见下载地址，索性将解压后的数据一律打包上传。
![coe.PNG](/images/2020/03/08/7aff3ba0-611b-11ea-a0f6-2761b1af4254.PNG)
安装位置位于home/.vscode-server/bin/XXXXXXX
[code-server github](https://github.com/cdr/code-server/releases)
---
2. 为本地vscode安装remote-ssh插件，设置服务器登录ip和用户名，在连接过程中会尝试自动下载数据包，由于服务器未连接到外网，下载失败，但是服务器对应目录已经创建，进到home/.vscode-server/bin/XXXXXXX下（xxxxx表示commit码）将在个人设备上打包的数据解压在此。再次连接就能正常使用了。

## 3.安装过程中遇到的问题
1. 由于默认生成的.ssh/config位于home下，对于windows来说，这个目录是有权限要求的，所以在选择保存目录时选到普通目录下。
---
2. 在连接过程中密码反复错误，原因是登录名错误，因为默认登录名是个人电脑用户名，而不是服务器用户名。
解决方法：在setting中下面一项填写保存的config地址
![sshconfig.PNG](/images/2020/03/08/f8b050d0-611a-11ea-a0f6-2761b1af4254.PNG)
3. 版本匹配问题，这个插件还会检测客户端(vscode)和服务端(服务器)的版本是否一致, 所以还需要在 vscode 中禁用插件的自动更新.
在 settings.json 添加如下行即可
   ```c
   "extensions.autoUpdate": false
   ```
   或者将本地安装版本选择与服务器一致，定期同步更新

---
分割线
2020.5.13更新
采用remote-ssh的连接方式在服务器的.vscode目录下产生多个目录，难以管理。又换成了浏览器连接的方法。
## 3. 浏览器vscode编辑
1. 下载2进制文件 [code-server](https://github.com/cdr/code-server/releases)
2. 启动服务端vscode于后台运行
```
nohup ./code-server -port 8082 > nohup.out 2>&1 &
```
默认密码随机生成字符串。可以通过设置PASSWORD使用用户自定义密码，也可以设置auth=none免密登录（不建议）

## 4. 设置进程监控脚本
在使用code-server过程中，偶尔会发生code-server进程意外停止的事情（多由插件造成）。每次都要重新ssh上去重启很是麻烦
写一个shell脚本，来监控code-server进程是否存在，若不存在则启动进程。
```sh
#进程名字可修改
PRO_NAME=code-server

while true ; do

#    用ps获取$PRO_NAME进程数量
  NUM=`ps aux | grep ${PRO_NAME} | grep -v grep |wc -l`
#  echo $NUM
#    少于1，重启进程；start_server.sh是自写的启动脚本
    if [ "${NUM}" -lt "1" ];then
        echo "${PRO_NAME} was killed"
        /home/yxf/download/code-server2.1698-vsc1.41.1-linux-x86_64/start_server.sh
    fi
#    每隔10秒钟检测一次
    sleep 10
#    大于1，杀掉所有进程，重启
#   elif [ "${NUM}" -gt "1" ];then
#     echo "more than 1 ${PRO_NAME},killall ${PRO_NAME}"
#     killall -9 $PRO_NAME
#     ${PRO_NAME} -i ${WLAN}
#   fi
# #    kill僵尸进程
#   NUM_STAT=`ps aux | grep ${PRO_NAME} | grep T | grep -v grep | wc -l`

#   if [ "${NUM_STAT}" -gt "0" ];then
#     killall -9 ${PRO_NAME}
#     ${PRO_NAME} -i ${WLAN}
#   fi
done

exit 0
```
此时，如果code-server进程意外停止，10秒内就会重启

如果需要开机自启，则可以将这个监控脚本放在/etc/init.d目录下，实现开机自启动。