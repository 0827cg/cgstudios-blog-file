---
layout: post
title: "Linux上安装python3"
date: 2017-10-11 14:44:23
updated: 2017-10-11 19:05:56
categories: python
tags: [python, linux]
---

目的是安装在cento服务器上安装python3，实现python3和python2共存，因为linux系统中很多是依赖于python2，所以避免出错python2不能卸载。这就想到了防火墙iptables也不能卸载，之前折腾公司服务器差点把iptables卸了...
所以这里就安装python3与系统自带的python2共存

<!-- more -->

首先创建文件夹python3来存放安装文件，例如我在/usr/local/创建python3目录
这里我下载的是`Python-3.5.4.tar.xz`这个版本，将其上传到服务器
然后命令解压
`sudo tar -xf Python-3.5.4.tar.xz`
解压后`cd Python-3.5.4`进入解压目录
再运行命令
`sudo ./configure --prefix=/usr/local/python3/`
设定安装目录为`/usr/local/python3/`这个，运行之后它将检测源码，当然如果直接运行`sudo ./configure`也可以，默认也是安装到`/usr/local/`下

然后
`sudo make`
来构建源码
之后
`sudo make install`
才算安装

像这种源代码编译安装的方式，其卸载方法就是删除安装目录即可
之后`cd /usr/loca/python3/bin`中

`sudo cp pip3 python3 /usr/bin/`
将其复制到/usr/bin中，这样就可以直接使用`python3`命令来运行python3了，这样也就使得系统自带的python2和python3共存了。同时不过为方便管理，通

常也会将pip3复制到`/usr/bin/`下，这样安装python3的第三方包就使用pip3命令了

至于复制到`/usr/bin/`目录下而不复制到别的目录，例如`/bin`,`/usr/sbin/`....的原因，
网上搜到个解释感觉很ok，如下

`/bin`
是存放系统的一些指令。bin为binary的简写主要放置一些系统的必备执行档例如: cat、cp、chmod、df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等。

`/sbin`
一般是指超级用户指令。主要放置一些系统管理的必备程式例如: cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown等。

`/usr/bin`
是用户在后期安装的一些软件的运行脚本。主要放置一些应用软体工具的必备执行档例如: c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome、gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb、wget等。

`/usr/sbin`
放置一些用户安装的系统管理的必备程式例如: dhcpd、httpd、imap、in.*d、inetd、lpd、named、netconfig、nmbd、samba、sendmail、squid、swap、tcpd、tcpdump等。