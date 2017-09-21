---
layout: post
title: "Note"
date: 2017-06-21 16:26:11
categories: note
tags: [note, codeing, daily]
---

这里将记录日常工作中的一些简单笔记.个人有个习惯,喜欢将开发中遇到的一些需要记住的东西,问题及相对应的解决方法记录下来.

说是喜欢,其实就是记忆力不好,想记下来,就用写笔记这种方式,久而久之就成了习惯

<!-- more -->

***

#### 2017-06-20

1. 控制器类调用与jsp页面相对应的控制器(方法),控制器再调用相应的server类进行操作
最后控制器(方法)将jsp页面所需要的数据进行返回,并将jsp页面进行渲染成html页面,

2. Hibernate使用标签,那么将不需要.cfg.xml配置文件,
其中数据库表的字段和实体类之间的关联则依靠标签,
在每一个实体类中,
类的开头将使用'@Table'标签指明表
例如:
	@Table(name = "category_type")
	public class CategoryType extends com.framework.hibernate.util.Entity implements java.io.Serializable {
	}

3. 代码使用svn更新遇到问题
	Error:svn: E210002: Network connection closed unexpectedly
	svn: E210002: Additional errors:
 这个还没找到解决方法

4. 使用jsp页面的好处
将实体类对象设置成全局变量,这样的话该对象就可以直接存放到Request,Response中,
只要控制器中实力类的对象是全局变量,那么就可以被自动放进Request中,返回给请求者,如果对象中含有数据,
那么浏览器将配合jsp页面中的标签将其整合渲染成html页面

5. 在一对多的关系中,对象值会一次性获取到
例如:实体类A对实体类B是一对多的关系,在Hibernate中需要将实体类B的对象当作变量放入实体类A中
那么在控制器中获取实体类A的对象值后,同样会自动获取实体类B对象的值

6. 一海游项目中,如下路径下的jsp页面:
	/home/cg/Work/JavaEEWork/Intellij-IDEA/sgyy-svn/soutuu_website/data/projects/lvxbang/yhypcweb/src/main/
	webapp/WEB-INF/jsp/yhypc/cruiseShip/cabinDetails.jsp
对应的实体类为:
	/home/cg/Work/JavaEEWork/Intellij-IDEA/sgyy-svn/soutuu_website/data/business/cruiseship/cruiseshipservice/src/main/
	java/com/data/data/hmly/service/cruiseship/entity/CruiseShipProject.java
其控制器类为:

	/home/cg/Work/JavaEEWork/Intellij-IDEA/sgyy-svn/soutuu_website/data/projects/lvxbang/yhypcweb/src/main/
	java/com/data/data/hmly/action/yhypc/CruiseShipWebAction.java

***

#### 2017-06-21

电脑太卡了,打算重装系统,无线网卡也不稳定
.....

1. 下载linuxmint系统镜像文件

2. 拷贝linuxmint系统上的工作文件,将其有用的进行打包
例如:各类编程源代码,项目源代码,系统主题,chrome-extension,bolg

3. 安装搜狗输入法,或者sunpinyin,谷歌拼音.前提需要安装fcitx

4. 更新源列表
linux其他系统可以使用命令:sudo gedit /etc/apt/sources.list
但linuxmint系统不再使用这个,有工具提供
以前用的软件源
rosa:
`http://mirrors.ustc.edu.cn/linuxmint`
trusty:
`http://mirrors.hust.edu.cn/ubuntu`

5. 安装jdk,node.js
并配置环境变量,其命令
`sudo gedit /etc/profile`
现在环境配置
	#Java-environment
	export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_77 
	export PATH=$JAVA_HOME/bin:$PATH 
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
	#set for nodejs-environment
	#export NODE_HOME=/usr/local/src/node-v7.3.0
	#export PATH=$NODE_HOME/bin:$PATH
	#set for hexo-environment
	export HEXO_HOME=/opt/node-v6.9.2-linux-x64/lib/node_modules/hexo-cli
	export PATH=$HEXO_HOME/bin:$PATH
	#Maven-environment
	export MAVEN_HOME=/home/cg/Work/JavaEEWork/Server/apache-maven-3.5.0
	export PATH=$MAVEN_HOME/bin:$PATH

6. 安装
eclipse
intellij idea
brackets
sublime-text
tomcat
mysql
git
svn
hexo
python3

7. 可以安装新得力软件管理器

*** 

#### 2017-06-27

找着一款云笔记应用--Simplenote。

界面风格简洁

并且有Linux版

打算以后用它做笔记



