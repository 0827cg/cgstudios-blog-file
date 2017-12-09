---
layout: post
title: "Java-android反编译"
date: 2017-05-09 12:44:00
updated: 2017-10-11 14:59:25
categories: java
tags: [java, codeing, android]
---

今天在读公司之前开发的一个项目的代码,对其中的一段代码挺感兴趣,就尝试着自己敲,下载需要导入的jar包.自己单独运行看看效果.
不过其中一段小插曲,就是下载的包导入项目中后一直提示不出来,就是代码import不到,于是就想着去看看jar包里的代码是什么样的,心里怀疑这难不成是包里的类名换了,,,,?
于是就想到了我之前弄的android反编译.
时隔太久,担心忘了,就总结在此

<!-- more -->

后来查看之后才发现下载的jar包其实是个压缩包,里面才是真正需要用到的jar包.重新导入里面的包后,就ok了

### 反编译用到的工具及下载地址:

* [apktool_2.2.1][]
* [dex2jar-2.0][]
* [jd-gui-0.3.5][]

#### apktool_2.2.1

下载解压后文件夹下原只包含两个文件

apktool和apktool.jar
apktool为脚本,为运行apktool.jar

作用: 获取apk安装包里的资源文件和xml布局文件
使用方法: 运行命令
`sudo ./apktool`
可显示使用方法参数
反编译方法: 命令
`sudo ./apktool d xxx.apk`
来反编译xxx.apk文件,以此来获得apk文件的xml布局文件和资源文件
详细参数使用:
`sudo ./apktool --help`

#### dex2jar-2.0

作用:这个工具用于将项目里的dex文件转换成jar文件 
示例
这里测试反编译apk安装包
此前测试
过程:
apk包名:baidutieba_v7.9.2.apk,将此安装包重命名为后缀为`.zip`(或其他)的压缩文件,之后解压,即得到baidutieba_v7.9.2文件夹及内容,其中就有 classes.dex文件
将文件夹baidutieba_v7.9.2内的classes.dex文件复制到dex2jar-2.0文件夹中在dex2jar-2.0文件夹内运行命令
`sudo ./d2j-dex2jar.sh classes.dex`
后,即可在dex2jar-2.0中得到classes-dex2jar.jar文件
如若运行不了,则为脚本添加权限
剩余的任务交给jd-gui来查看即可

#### jd-gui-0.3.5

作用: 用来查看.jar文件的源代码
双击运行即可

### 总结

* apktool_2.2.1用来获取.apk文件的资源文件
* dex2jar-2.0和jd-gui-0.3.5两个工具结合起来用来获取apk安装包的源代码
* jd-gui-0.3.5是用来查看.jar的java源代码,可以用在很多地方

我已经将三个工具打包上传到百度云,链接: https://pan.baidu.com/s/1kVSJozH 密码: xuq4


[apktool_2.2.1]:https://ibotpeaches.github.io/Apktool/
[dex2jar-2.0]:https://sourceforge.net/projects/dex2jar/files/
[jd-gui-0.3.5]:http://jd.benow.ca/jd-gui/downloads/jd-gui-0.3.5.linux.i686.tar.gz
