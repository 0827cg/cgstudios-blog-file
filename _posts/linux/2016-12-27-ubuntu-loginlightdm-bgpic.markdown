---
layout: post
title: "ubuntu15.10修改默认登录器lightdm的登录界面背景图片"
date: 2016-12-27 16:43:35
categories: linux
tags: linux
---
现在自己所用的系统是ubuntu15.10，其系统默认的显示管理器是lightdm，用它来登录ubuntu15.10的unity桌面环境。
对lightdm的登录界面没什么多大的好感，linux系统对于我来讲只要够稳定就行，用来工作就是要求稳定。

想要更改lightdm的背景图片,网上搜了下有好多方法，这里我介绍下我的方法(参照wiki得到的)

在文件夹/usr/share/glib-2.0/schemas下有很多关于lightdm的配置文件，修改此文件夹下的50_unity-greeter.gschema.override这个文件，这里注意下，不同型号的系统其配置文件的文件名也许会有差异，所以并不一定在你的linux系统下面的会有跟我这个文件名称相同的文件，废话不多说，打开就行了，
如下图:

<!-- more -->

![查看lightdm配置文件](/images/linux/t1.png)

上面是我已经修改好了的样子(其实只要添加一条代码就可以了)
先来看看这几行代码
这里注意下这个文件里面开头的那个标签[com.canonical.unity-greeter]，也就是标明了是unity桌面环境的登录界面，
第一行代码:draw-user-backgrounds=false，这是我自己添加进去的唯一一条代码，也就是这个文件只需要修改这里。
这代码的作用是使lightdm的背景图片不会被切换，如果没这行代码，那么在你开机到出现登录界面时，会出现你自己定义的界面背景图片，但又会自动马上切换成桌面背景图片，不知道你们会不会遇到这种情况，方正我是遇到了。
第二行代码:,,,很显然，它是指明登录界面的背景图片的存放地址，这张名为warty-final-ubuntu.png的图片，就是ubuntu登录界面默认的背景图片，所以我们只需要拿自己准备的图片替换它就可以了。
在替换时，需要注意其原始图片的权限，

![结果](/images/linux/t2.png)

上面依然是我已经替换好了的样子，
warty-final-ubuntu-old.png是默认的背景图片，warty-final-ubuntu.png是我自定义的图片，我们需要做的是将自定义的图片的权限修改成系统默认的背景图片的权限，
这里需要用到命令是chmod，
在此文件夹下运行命令:sudo chmod 644 warty-final-ubuntu.png

后面reboot就行了，

其实还有一个不完美的地方，每次开机lightdm显示的主题样式不是自己使用的主题样式，有时候自己该了下也能改变其样式，但不是预想的效果。嗯，这得自己好好琢磨琢磨

这里顺便贴下那篇wiki的地址:[https://wiki.ubuntu.com/LightDM][wiki-url]   没事多看看wiki。
多说一句，arch wiki更值得去看

[wiki-url]:https://wiki.ubuntu.com/LightDM


