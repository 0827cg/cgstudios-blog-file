---
layout: post
title: "Ubuntu15.10打开unity-tweak-tool出错scheme missing"
date: 2016-12-28 11:04:35
categories: linux
tags: linux
---

ubuntu15.10安装unity-tweak-tool后打不开显示scheme missing！并提示org.gnome.settings-daemon.peripherals.touchpad
百度后才得到解决方法：
命令cd到/usr/lib/python3/dist-packages/UnityTweakTool/section目录下
	命令：cd /usr/lib/python3/dist-packages/UnityTweakTool/section

<!-- more -->

![查看unitytweak配置文件](/images/linux/t1.png)

如上图，修改system.py文件
	命令：gksu gedit system.py
找到行数为182,193,205，将这3行都修改成  'schema' : 'org.gnome.desktop.peripherals.touchpad'
之后在section目录下再cd 进如spaghetti文件夹，修改gsettings.py
	命令：gksu gedit gsettings.py
找到第120行，将其修改成  touch = gnome('desktop.peripherals.touchpad')
保存退出
之后就可以运行unity-tweak-tool了

