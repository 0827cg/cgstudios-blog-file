---
layout: post
title: "Java环境变量"
date: 2017-3-08 19:17:40
categories: java
tags: [java, codeing]
---

Java环境变量的设置.这还是大一的时候学的,现在大三了,重温一下,免得忘记

### Windows系统

在官网上下载java开发包切安装完成后，接下来就是配置环境变量了。在“我的电脑”右键后点击“属性”，在弹出的控制面板主页中点击“高级系统配置”，随即在弹出的系统属性的对话框中点击“环境变量”，之后就可以在弹出的对话框中看到有用户变量和系统变量，我们配置java环境变量就是在系统变量中进行。
先直接说下我们需要配置的环境变量的名称：
有两种：

第一种是一定要配置的环境变量：PATH和CLASSPATH
第二种就是可要可不要的环境变量：JAVA_HOME

<!-- more -->

#### JAVA_HOME变量

JAVA_HOME变量可要可不要，是因为它的作用仅仅只是作为一个中间变量，是一个用来传递的作用，为了在配置PATH和CLASSPATH变量时达到简介的效果。
再者，先认识下PATH变量：

PATH变量的设定简单的来说就是为了让系统能在不同目录下都能执行PATH的值所指引的目录下的可执行文件，类似一个全局静态变量的作用，知道这个变量的作用之后再来说下这个变量的值，因为这个变量在系统中都早已存在，所赋予的值都是为了系统执行的方便，而java的值就是这样。

java程序的编译和运行需要用到两个exe程序，一个是java编译器\(javac.exe\)和java解释器\(java.exe\)，它俩是组成jvm的一部分，也就是说，要想在不同目录下执行这两个exe程序来编译运行java程序，就得需要在PATH环境变量中为其指定这两个exe程序的存放路径，所以PATH环境变量就是这样来的。

#### CLASSPATH变量

先不说怎么填写PATH的值，先再了解下CLASSPATH环境变量的作用。

CLASSPATH环境变量的在我的电脑上之前没有这个变量，估计也得都是自己添加。CLASSPATH从字面上来看(毕竟编程都讲究名字不能随便起，都需要达到看到字能够确切的想到某种东西的效果)，CLASSPATH中大概的意思就是类的路径，这也不难猜测，毕竟jdk中提供了许多类库，敲代码都少不了导入一些自带的类库，也正因为如此，如果在编译运行java程序时，该程序导入了许多类库，比如最简单的`import java.util.Scanner`;导入Scanner类，如果不指明Scanner类库的存放位置，编译就会出错，所以自然就需要一个环境变量来指定那些类库的存放目录了。
JAVA_HOME就是充当了一个中间传递者的角色，该变量的值就是jdk安装目录的值。

#### PATH变量

现在就来说PATH变量的值，就是javac.exe和java.exe的存放路径，其两个exe程序都存放在jdk的\bin目录下。假设jdk的安装目录为:

`C:\Program Files\Java\jdk1.8.0_66`;

那么PATH变量所添加的值就为:

`C:\Program Files\Java\jdk1.8.0_66\bin`

如果增加了JAVA_HOME变量，且其值为:

`C:\Program Files\Java\jdk1.8.0_66`

那么PATH变量的所添加的值就为

`%JAVA_HOME%\bin`

CLASSPATH的值就是指定类库的存放路径，其存放位置是在jdk的\jre\lib目录下的rt.jar包里面，所以它的值为:

`C:\Program Files\Java\jdk1.8.0_66\jre\lib\rt.jar;.;`

后面有“;.;”三个符号,表示可以加载该目录下的类及其子目录下的类。

同样如果设置了JAVA_HOME则值为:

`.:%JAVA_HOME%\jre\lib\rt.jar;.;`

到这里，Java的环境变量就设定好了，，

cmd下检测输入java -version和javac如若输出正常则环境变量就完全设定好了，
之后就可安装eclipse开发java程序了

总得来说：

	JAVA_HOME:C:\Program Files\Java\jdk1.8.0_66
	PATH:后面添加:;%JAVA_HOME%\bin;
	不能存在“;;”
	CLASSPATH:.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar

### Linux(Ubuntu)系统上搭建Java环境

在官网上下载Jdk后，先在/usr/lib下新建jvm文件夹，因为后面的jdk安装在jvm里面
在终端进入下载目录,将下载好的jdk解压，我这里是jdk-8u66-linux-x64.tar.gz，并将其解压至/usr/lib/jvm下
命令：`sudo tar -zvxf jdk-8u66-linux-x64.tar.gz -C /usr/lib/jvm`
完后就是得配置java开发环境了
在新打开的终端上输入命令： `gksu gedit /etc/profile`  (当然也可以是gedit ~/.bashrc，只不过.bashrc是针对先用户才管用，而/etc/profile是对全局管用)
在打开的文本后面加上如下

	# Java-environment
	export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_77 
	export PATH=$JAVA_HOME/bin:$PATH 
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 

即可保存

之后再运行命令:`source /etc/profile`

在终端上输入java -version和javac显示正常即完成了java的环境配置

![java-bookContorlSystem](/images/java/java-environmentVariable.png)

好了,谢谢
