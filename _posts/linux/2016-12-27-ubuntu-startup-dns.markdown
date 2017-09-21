---
layout: post
title: "解决ubuntu15.10开机自动更改DNS服务器问题"
date: 2016-12-27 15:23:32
categories: linux
tags: linux
---

我的系统是ubuntu15.10，由于dns服务器地址经常自动更改，不知道什么原因，但肯定不是network-manager的原因，因为在出问题之前network-manager也还在，那会儿dns服务器地址并没有因为其而自动更换。

了解:ubuntu系统的dns服务器地址的存放在/etc/resolv.conf文件中，每一次开机查看此文件可看到其地址都是127.0.1.1，

<!-- more -->

解决方法:修改/etc/resolvconf/resolv.conf.d/head 此文件，在里面添加:nameserver 202.114.0.131或者

nameserver 202.114.0.242，这两个地址是在电脑不使用锐捷客户端而是直接使用network-manager连接无线网络时得到的dns服务器地址。这里说一下resolv.conf.d这个文件夹里面的两个文件head和base,一开始在网络上查找到的方法是修改base这个文件，然而此方法对我没用，估计那会儿献此方法的人只有base这一个文件，而没有head这个文件，因此估摸这如果两个文件都有的人，就得像我一样修改head这个文件，反之，则修改base这个文件

