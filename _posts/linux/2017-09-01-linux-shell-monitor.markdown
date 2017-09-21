---
layout: post
title: "使用shell脚本监控服务器项目运行"
date: 2017-09-01 17:13:26
updated: 2017-09-18 9:14:36
categories: linux
tags: [linux, email, mail, shell]
---

之前写过一个每天定时对nginx日志文件进行切割的脚本，并在切割之后通过发送邮件来告知脚本执行是否成功。实现起来很简单，本来不想写，但前辈让一定要写，写就写吧，反正无聊。就是写完脚本后就更无聊了...
这次也还是一样的，需要写一个脚本来监控服务器中的各种服务，tomcat,redis,nginx,磁盘空间，jdk...没错，还说要监控jdk.....

<!-- more -->

不多说了，本来语文就差，组词这么粗糙。

先说下，现在这个脚本应该算是还有两个bug,后面慢慢解决。

正文

### 脚本大致功能
1.脚本每间隔5分钟运行一次，检测各个系统模块运行是否正常，如若运行不正常，脚本先自行处理一次，如果处理后该服务仍然未正常运行，则发邮件提醒。正常则无提醒
2.在脚本没5分钟运行的情况下，每间隔1个小时(24h)，脚本会对服务器中的所有已经监控的模块状态进行汇总，并发邮件通知

脚本已经提供监控代码的服务就这三项，tomcat,redis和nginx，外加个磁盘根目录信息(无自动清理功能)

### 脚本大致流程

#### 匹配项目是否运行

就拿tomcat来说，脚本通过命令
'ps -ef | grep tomcat'
来查看进程中运行的tomcat，将输出结果进行模糊匹配，这里的模糊匹配只是匹配对应tomcat安装目录的文件夹名称
例如运行该命令的输出如下

	[xm6f@localhost automatic]$ ps -ef | grep tomcat
	root      1503     1  0 8月29 ?       00:11:29 //bin/java -Djava.util.logging.config.file=/usr/local/tomcat/tomcat-8082/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms512m -Xmx4096m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/tomcat/tomcat-8082/endorsed -classpath /usr/local/tomcat/tomcat-8082/bin/bootstrap.jar:/usr/local/tomcat/tomcat-8082/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat/tomcat-8082 -Dcatalina.home=/usr/local/tomcat/tomcat-8082 -Djava.io.tmpdir=/usr/local/tomcat/tomcat-8082/temp org.apache.catalina.startup.Bootstrap start
	xm6f     10126     1  0 12:59 pts/5    00:01:34 /usr/local/java/jdk1.8.0_144/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/lotmall-8080/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms512m -Xmx4096m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/tomcat/lotmall-8080/endorsed -classpath /usr/local/tomcat/lotmall-8080/bin/bootstrap.jar:/usr/local/tomcat/lotmall-8080/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat/lotmall-8080 -Dcatalina.home=/usr/local/tomcat/lotmall-8080 -Djava.io.tmpdir=/usr/local/tomcat/lotmall-8080/temp org.apache.catalina.startup.Bootstrap start
	xm6f     12173     1  1 14:26 pts/7    00:00:54 /usr/local/java/jdk1.8.0_144/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/mango-8081/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms512m -Xmx4096m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/tomcat/mango-8081/endorsed -classpath /usr/local/tomcat/mango-8081/bin/bootstrap.jar:/usr/local/tomcat/mango-8081/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat/mango-8081 -Dcatalina.home=/usr/local/tomcat/mango-8081 -Djava.io.tmpdir=/usr/local/tomcat/mango-8081/temp org.apache.catalina.startup.Bootstrap start
	xm6f     13681  7589  0 15:39 pts/7    00:00:00 grep --color=auto tomcat
	
根据上面的输出，可以看到运行了三台tomcat,三台tomcat分别对应的安装目录的路径即上面输出中的`Dcatalina.base`这个变量的值，对比这三台tomcat的`Dcatalina.base`的值，可以知道这三台tomcat的安装文件都是存放在`/usr/local/tomcat/`这个路径下的，虽然这是安装时刻意弄的，但这样毕竟更好管理，对应写脚本也更方便。在这个路径下，存放的三个文件夹就是对应的三台tomcat安装文件，所以根据这些输出，可以进行模糊匹配来辨别当个tomcat是否运行
主要实现代码如下

	TOMCAT_STATUS=$(ps -ef | grep tomcat)
	if [[ $TOMCAT_STATUS =~ "8080" ]]
	then
		createEmail_content "* tomcat8080---已在运行" -h 1
		TOMCAT_1_LOG=$(tail -n 150 /usr/local/tomcat/tomcat-8080}/logs/catalina.out)
		if [[ $TOMCAT_1_LOG =~ "exception" ]]
		then
			createEmail_content "---日志输出异常" -h
			createEmail_errContent "===============149-8080-输出日志===============" -h
			createEmail_errContent "$TOMCAT_1_LOG" -h
		else
			createEmail_content "---日志输出正常" -h
		fi
	else
		createEmail_content "* tomcat$8080---未运行" -h
	fi
	
在知道tomcat是否运行之后，就可以对日志的输出进行解析了，以此来判断该tomcat运行时的日志输出是否正常，当然，如果tomcat未运行就不必对日志文件进行分析了。
判断日志输出是否正常的方式和判断tomcat是否运行的方式其理念是一样的，如上面的代码，获取8080端口的那台tomcat的文件末尾的150行，150行足够了，全屏下来也就50+行，获取这些输出之后同样拿来进行模糊匹配，观察之前的日志报错输出，里面都会有`exception`这个字段，匹配它就可以了，如果为结果为`true`，则说明日志报错了。
在知道tomcat是否正常运行及日志输出是否正常之后，将结果写入到txt文件，方便后面发送邮件，也方便记录
相对应的，判断nginx和redis是否运行也是通过这样的方式，这里就不说了

#### 查看磁盘空间
查看磁盘使用空间最简单的命令就是`df`命令，使用`df -h`就可以知道所有挂载点的空间使用量，获取根目录挂载点命令就如`df -h /`,其输出有6列，拿要获取第5列就用命令
`df -h / | awk '{print $5}'`
其输出如下

	[xm6f@localhost automatic]$ df -h / | awk '{print $5}'
	已用%
	1%
	[xm6f@localhost automatic]$ 

1%不大好对比，我想得到的是单纯的数字，但这个也不好截取，那就网上搜，例如我这里搜索得到的例如
`df -h | grep /dev/mapper/cl-root | awk '{print $5}' | cut -f 1 -d "%"`
这里的`/dev/mapper/cl-root`是根挂载点对应的文件系统的输出
整个命令运行输出如下

	[xm6f@localhost automatic]$ df -h | grep /dev/mapper/cl-root | awk '{print $5}' | cut -f 1 -d "%"
	1
	[xm6f@localhost automatic]$

这就ok了，这样运行得到结果之后直接放到if语句中进行对比，大于50%就发邮件提醒，就行了

### 脚本函数介绍

#### 写入内容到txt文件函数
因为总是需要写入内容到文件，总是一味的使用`echo`命令感觉太麻烦了，索性就封装成方法，后面调用即可
就如下面的代码，定义了两个函数，一个用来写记录执行错误输出的文件，一个用来写反馈结果到文件的方法
代码如下

	function createEmail_content() {
		if [[ $2 = -h ]]
		then
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> email_content-${TODAY_H}.txt
			else
				echo -e "$1\c" >> email_content-${TODAY_H}.txt
			fi
		else
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> email_content-${TODAY_HMS}.txt
			else
				echo -e "$1\c" >> email_content-${TODAY_HMS}.txt
			fi
		fi
	}

	function createEmail_errContent() {
			if [[ $2 = -h ]]
			then
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> email_err-${TODAY_H}.txt
					else
							echo -e "$1\c" >> email_err-${TODAY_H}.txt
					fi
			else
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> email_err-${TODAY_HMS}.txt
					else
							echo -e "$1\c" >> email_err-${TODAY_HMS}.txt
					fi
			fi
	}

简单的介绍下上面代码，总共可以输入三个参数

* $1:文本内容
* $2:写入到隔1小时执行结果的文件还是写入到隔5分钟执行结果的文件
* $3:写入是否换行

#### 重启函数
不多说了

	function startTomcat() {
		${1}/bin/./catalina.sh start
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对tomcat${2}执行启动命令"
		else
			createEmail_content "**-->脚本未成功对tomcat${2}执行命令"
		fi
		START_TOM_RESULT=$(ps -ef | grep tomcat)
		if [[ $START_TOM_RESULT =~ "$2" ]]
		then
			createEmail_content "**-->tomcat${2}已成功启动"
		else
			createEmail_content "**-->tomcat${2}启动未成功，请手动启动"
		fi	
	}
	
#### 1小时执行函数
其中包括检测tomcat，redis，nginx和磁盘根目录使用情况
后面打算把这个方法拆分开了，代码太臃肿了

	function automatic_h() {

		echo "" > email_content-${TODAY_H}.txt

		createEmail_content "${COMPUTER_NAME}今日${HOUR}时脚本自动执行结果" -h
		createEmail_content "======================================" -h

		TOMCAT_STATUS=$(ps -ef | grep tomcat)

		if [[ $TOMCAT_STATUS =~ "$PORT_1" ]]
		then
			createEmail_content "* tomcat${PORT_1}---已在运行" -h 1
			TOMCAT_1_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_1_FILENAME}/logs/catalina.out)
			if [[ $TOMCAT_1_LOG =~ "exception" ]]
			then
				createEmail_content "---日志输出异常" -h
				createEmail_errContent "===============${COMPUTER_NAME}-${PORT_1}-输出日志===============" -h
				createEmail_errContent "$TOMCAT_1_LOG" -h
			else
				createEmail_content "---日志输出正常" -h
			fi
		else
			createEmail_content "* tomcat${PORT_1}---未运行" -h

		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_2" ]]
		then
				createEmail_content "* tomcat${PORT_2}---已在运行" -h 1
				TOMCAT_2_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_2_LOG =~ "exception" ]]
				then
						createEmail_content "---日志输出异常" -h
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_2}-输出日志===============" -h
						createEmail_errContent "$TOMCAT_2_LOG" -h
				else
						createEmail_content "---日志输出正常" -h
				fi
		else
				createEmail_content "* tomcat${PORT_2}---未运行" -h

		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_3" ]]
		then
				createEmail_content "* tomcat${PORT_3}---已在运行" -h 1
				TOMCAT_3_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_3_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_3_LOG =~ "exception" ]]
				then
						createEmail_content "---日志输出异常" -h
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_3}-输出日志===============" -h
						createEmail_errContent "$TOMCAT_3_LOG" -h
				else
						createEmail_content "---日志输出正常" -h
				fi
		else
				createEmail_content "* tomcat${PORT_3}---未运行" -h
		fi

		REDIS_STATUS=$(ps -ef | grep 6379)
		if [[ $REDIS_STATUS =~ "redis-server" ]]
		then
			createEmail_content "* redis已在运行" -h
		else
			createEmail_content "* redis未运行" -h
		fi

		NGINX_STATUS=$(ps -ef | grep nginx)
		if [[ $NGINX_STATUS =~ "nginx:" ]]
		then
			createEmail_content "* nginx已在运行" -h
		else
			createEmail_content "* nginx未运行" -h
		fi

		#DISK_GEN_STATUS=$(df -h | grep /dev/m | awk '{print $5}')
		#createEmail_content "根目录使用量为$DISK_GEN_STATUS"

		DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
		if [[ $DISK_GEN_STATUS -lt 50 ]]
		then
			createEmail_content "* 根目录使用量较小---已使用${DISK_GEN_STATUS}%" -h
		else
			createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%" -h
		fi


		echo -e "\n\n" >> email_content-${TODAY_H}.txt

		createAll_tail -h
	}


#### 5分钟执行函数
这个方法和前面的方法大致一样，是上面方法的缩减版

	function automatic_hms() {

		TOMCAT_STATUS=$(ps -ef | grep tomcat)

		if [[ $TOMCAT_STATUS =~ "$PORT_1" ]]
		then
			TOMCAT_1_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_1_FILENAME}/logs/catalina.out)
			TOMCAT_1_DIR=${TOMCAT_DIR}/${PORT_1_FILENAME}
			if [[ $TOMCAT_1_LOG =~ "exception" ]]
			then
				createEmail_content "* tomcat${PORT_1}---已运行---但日志输出异常"
				createEmail_errContent "===============${COMPUTER_NAME}-${PORT_1}-输出日志==============="
				createEmail_errContent "$TOMCAT_1_LOG"
			fi
		else
			createEmail_content "* tomcat${PORT_1}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_1_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_1_FILENAME} ${PORT_1}
					else
							createEmail_content "**-->tomcat${PORT_1}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_1}失败"
					fi
		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_2" ]]
		then
				TOMCAT_2_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_2_LOG =~ "exception" ]]
				then
						createEmail_content "tomcat${PORT_2}---已运行---但日志输出异常"
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_2}-输出日志==============="
						createEmail_errContent "$TOMCAT_2_LOG"
				fi
		else
				createEmail_content "* tomcat${PORT_2}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_2_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_2_FILENAME} ${PORT_2}
					else
							createEmail_content "**-->tomcat${PORT_2}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_2}失败"
					fi
		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_3" ]]
		then
				TOMCAT_3_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_3_LOG =~ "exception" ]]
				then
						createEmail_content "tomcat${PORT_3}---已运行---但日志输出异常"
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_3}-输出日志==============="
						createEmail_errContent "$TOMCAT_3_LOG"
				fi
		else
				createEmail_content "* tomcat${PORT_3}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_3_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_3_FILENAME} ${PORT_3}
					else
							createEmail_content "**-->tomcat${PORT_3}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_3}失败"
					fi
		fi

		REDIS_STATUS=$(ps -ef | grep 6379)
		if [[ ! $REDIS_STATUS =~ "redis-server" ]]
		then
			createEmail_content "* redis未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "$REDIS_DIR" ]]
					then
							startRedis $REDIS_DIR
					else
							createEmail_content "**-->redis路径不存在"
							createEmail_content "**-->脚本启动redis失败"
					fi
		fi

		NGINX_STATUS=$(ps -ef | grep nginx)
		if [[ ! $NGINX_STATUS =~ "nginx:" ]]
		then
			createEmail_content "* nginx未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "$NGINX_DIR" ]]
					then
							startNginx $NGINX_DIR
					else
							createEmail_content "**-->nginx路径不存在"
							createEmail_content "**-->脚本启动nginx失败"
					fi
		fi

		#DISK_GEN_STATUS=$(df -h | grep /dev/m | awk '{print $5}')
		#createEmail_content "根目录使用量为$DISK_GEN_STATUS"

		DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
		if [[ $DISK_GEN_STATUS -gt 50 ]]
		then
			createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%"
		fi
		
		if [[ -e "email_content-${TODAY_HMS}.txt" ]]
		then
			echo -e "\n\n" >> email_content-${TODAY_HMS}.txt
			createAll_tail
		fi
	}


#### 选择执行并发送邮件

	if [[ $MIN = 30 ]]
	then
		automatic_h
		if [[ -e "email_err-${TODAY_H}.txt" ]]
		then
				mail -s "${COMPUTER_NAME}检测结果" -a email_err-${TODAY_H}.txt -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_H}.txt
		else
				mail -s "${COMPUTER_NAME}检测结果" -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_H}.txt
		fi
	else
		automatic_hms
		if [[ -e "email_content-${TODAY_HMS}.txt" ]]
		then
			if [[ -e "email_err-${TODAY_HMS}.txt" ]]
			then
				mail -s "${COMPUTER_NAME}有服务未运行" -a email_err-${TODAY_HMS}.txt -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_HMS}.txt
			else
				mail -s "${COMPUTER_NAME}有服务未运行" -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_HMS}.txt
			fi
		fi
	fi

代码中主要的功能就在这隔1小时执行的方法

#### 总共代码如下

	#!/bin/bash

	#Describe: Monitor Control System.
	# tomcat, nginx, redis
	#Author: cg
	#Time: 2017-08-31

	#----------USER CONFIG----------#

	##电脑服务器别称，这里用ip的最后一个字段
	COMPUTER_NAME=149

	##tomcat路径，该路径为tomcat的父级路径
	#前提是在此路径文件夹下包含有3个或多个子tomcat的安装文件夹
	#例如:
	#	/usr/local/tomcat--->lotmall-8080
	#					 --->mango-8081
	#					 --->tomcat-8082
	TOMCAT_DIR=/usr/local/tomcat

	##redis的可运行文件所在文件夹的路径
	#前提是redis的配置文件redis.conf和redis.server....都在该文件路径下
	REDIS_DIR=/home/xm6f/dev/redis

	##nginx安装文件路径
	NGINX_DIR=/usr/local/nginx


	##多个tomcat对应的ip
	PORT_1=8080
	PORT_2=8081
	PORT_3=8082

	##父级tomcat路径下存放的3个tomcat安装文件的文件夹名称
	PORT_1_FILENAME=lotmall-8080
	PORT_2_FILENAME=mango-8081
	PORT_3_FILENAME=tomcat-8082

	##redis端口，这里默认使用6379
	#REDIS_PORT=6379

	##根节点对应的文件系统
	#可以使用`df -Th /` 命令查看获得
	GEN_FILE_TYPE=/dev/mapper/cl-root

	##要发送的邮箱地址
	#如果有需要发送更多个，则在下面按照格式添加，之后在此脚本末尾的if语句块中添加，注意空格
	EMAIL_ADDR_1=1542723438@qq.com
	EMAIL_ADDR_2=1679055895@qq.com

	#-------------------------------#

	TODAY_HMS=$(date +%y%m%d%H%M%S)
	TODAY_H=$(date +%y%m%d%H)
	YESTERDAY=$(date -d "yesterday" +%y%m%d)
	TIME=$(date +%Y-%m-%d-%T)
	HOUR=$(date +%H)
	MIN=$(date +%M)


	function createEmail_content() {
		if [[ $2 = -h ]]
		then
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> email_content-${TODAY_H}.txt
			else
				echo -e "$1\c" >> email_content-${TODAY_H}.txt
			fi
		else
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> email_content-${TODAY_HMS}.txt
			else
				echo -e "$1\c" >> email_content-${TODAY_HMS}.txt
			fi
		fi
	}

	function createEmail_errContent() {
			if [[ $2 = -h ]]
			then
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> email_err-${TODAY_H}.txt
					else
							echo -e "$1\c" >> email_err-${TODAY_H}.txt
					fi
			else
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> email_err-${TODAY_HMS}.txt
					else
							echo -e "$1\c" >> email_err-${TODAY_HMS}.txt
					fi
			fi
	}


	function startTomcat() {
		${1}/bin/./catalina.sh start
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对tomcat${2}执行启动命令"
		else
			createEmail_content "**-->脚本未成功对tomcat${2}执行命令"
		fi
		START_TOM_RESULT=$(ps -ef | grep tomcat)
		if [[ $START_TOM_RESULT =~ "$2" ]]
		then
			createEmail_content "**-->tomcat${2}已成功启动"
		else
			createEmail_content "**-->tomcat${2}启动未成功，请手动启动"
		fi	
	}

	function startRedis() {
		${1}/./redis-server redis.conf
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对redis执行启动命令"
		else
			createEmail_content "**-->脚本未成功对reids执行命令"
		fi
		START_RED_RESULT=$(ps -ef | grep 6379)
		if [[ $START_RED_RESULT =~ "6379" ]]
		then
			createEmail_content "**-->redis已成功启动"
		else
			createEmail_content "**-->redis启动未成功，请手动启动"
		fi
	}
			

	function startNginx() {
		${1}/sbin/./nginx
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对nginx执行启动命令"
		else
			createEmail_content "**-->脚本未成功对nginx执行命令"
		fi
		START_NGINX_RESULT=$(ps -ef | grep nginx)
		if [[ $START_NGINX_RESULT =~ "nginx:" ]]
		then
			createEmail_content "**-->nginx已成功启动"
		else
			createEmail_content "**-->nginx启动未成功，请手动启动"
		fi
	}


	function createAll_tail() {

	createEmail_content "======================================" $1
	createEmail_content "---林繁" $1
	createEmail_content "---$TIME" $1

	}

	function automatic_h() {

		echo "" > email_content-${TODAY_H}.txt

		createEmail_content "${COMPUTER_NAME}今日${HOUR}时脚本自动执行结果" -h
		createEmail_content "======================================" -h

		TOMCAT_STATUS=$(ps -ef | grep tomcat)

		if [[ $TOMCAT_STATUS =~ "$PORT_1" ]]
		then
			createEmail_content "* tomcat${PORT_1}---已在运行" -h 1
			TOMCAT_1_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_1_FILENAME}/logs/catalina.out)
			if [[ $TOMCAT_1_LOG =~ "exception" ]]
			then
				createEmail_content "---日志输出异常" -h
				createEmail_errContent "===============${COMPUTER_NAME}-${PORT_1}-输出日志===============" -h
				createEmail_errContent "$TOMCAT_1_LOG" -h
			else
				createEmail_content "---日志输出正常" -h
			fi
		else
			createEmail_content "* tomcat${PORT_1}---未运行" -h

		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_2" ]]
		then
				createEmail_content "* tomcat${PORT_2}---已在运行" -h 1
				TOMCAT_2_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_2_LOG =~ "exception" ]]
				then
						createEmail_content "---日志输出异常" -h
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_2}-输出日志===============" -h
						createEmail_errContent "$TOMCAT_2_LOG" -h
				else
						createEmail_content "---日志输出正常" -h
				fi
		else
				createEmail_content "* tomcat${PORT_2}---未运行" -h

		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_3" ]]
		then
				createEmail_content "* tomcat${PORT_3}---已在运行" -h 1
				TOMCAT_3_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_3_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_3_LOG =~ "exception" ]]
				then
						createEmail_content "---日志输出异常" -h
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_3}-输出日志===============" -h
						createEmail_errContent "$TOMCAT_3_LOG" -h
				else
						createEmail_content "---日志输出正常" -h
				fi
		else
				createEmail_content "* tomcat${PORT_3}---未运行" -h
		fi

		REDIS_STATUS=$(ps -ef | grep 6379)
		if [[ $REDIS_STATUS =~ "redis-server" ]]
		then
			createEmail_content "* redis已在运行" -h
		else
			createEmail_content "* redis未运行" -h
		fi

		NGINX_STATUS=$(ps -ef | grep nginx)
		if [[ $NGINX_STATUS =~ "nginx:" ]]
		then
			createEmail_content "* nginx已在运行" -h
		else
			createEmail_content "* nginx未运行" -h
		fi

		#DISK_GEN_STATUS=$(df -h | grep /dev/m | awk '{print $5}')
		#createEmail_content "根目录使用量为$DISK_GEN_STATUS"

		DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
		if [[ $DISK_GEN_STATUS -lt 50 ]]
		then
			createEmail_content "* 根目录使用量较小---已使用${DISK_GEN_STATUS}%" -h
		else
			createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%" -h
		fi


		echo -e "\n\n" >> email_content-${TODAY_H}.txt

		createAll_tail -h
	}


	function automatic_hms() {

		TOMCAT_STATUS=$(ps -ef | grep tomcat)

		if [[ $TOMCAT_STATUS =~ "$PORT_1" ]]
		then
			TOMCAT_1_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_1_FILENAME}/logs/catalina.out)
			TOMCAT_1_DIR=${TOMCAT_DIR}/${PORT_1_FILENAME}
			if [[ $TOMCAT_1_LOG =~ "exception" ]]
			then
				createEmail_content "* tomcat${PORT_1}---已运行---但日志输出异常"
				createEmail_errContent "===============${COMPUTER_NAME}-${PORT_1}-输出日志==============="
				createEmail_errContent "$TOMCAT_1_LOG"
			fi
		else
			createEmail_content "* tomcat${PORT_1}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_1_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_1_FILENAME} ${PORT_1}
					else
							createEmail_content "**-->tomcat${PORT_1}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_1}失败"
					fi
		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_2" ]]
		then
				TOMCAT_2_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_2_LOG =~ "exception" ]]
				then
						createEmail_content "tomcat${PORT_2}---已运行---但日志输出异常"
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_2}-输出日志==============="
						createEmail_errContent "$TOMCAT_2_LOG"
				fi
		else
				createEmail_content "* tomcat${PORT_2}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_2_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_2_FILENAME} ${PORT_2}
					else
							createEmail_content "**-->tomcat${PORT_2}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_2}失败"
					fi
		fi

		if [[ $TOMCAT_STATUS =~ "$PORT_3" ]]
		then
				TOMCAT_3_LOG=$(tail -n 150 ${TOMCAT_DIR}/${PORT_2_FILENAME}/logs/catalina.out)
				if [[ $TOMCAT_3_LOG =~ "exception" ]]
				then
						createEmail_content "tomcat${PORT_3}---已运行---但日志输出异常"
						createEmail_errContent "===============${COMPUTER_NAME}-${PORT_3}-输出日志==============="
						createEmail_errContent "$TOMCAT_3_LOG"
				fi
		else
				createEmail_content "* tomcat${PORT_3}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${PORT_3_FILENAME}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${PORT_3_FILENAME} ${PORT_3}
					else
							createEmail_content "**-->tomcat${PORT_3}路径不存在"
							createEmail_content "**-->脚本启动tomcat${PORT_3}失败"
					fi
		fi

		REDIS_STATUS=$(ps -ef | grep 6379)
		if [[ ! $REDIS_STATUS =~ "redis-server" ]]
		then
			createEmail_content "* redis未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "$REDIS_DIR" ]]
					then
							startRedis $REDIS_DIR
					else
							createEmail_content "**-->redis路径不存在"
							createEmail_content "**-->脚本启动redis失败"
					fi
		fi

		NGINX_STATUS=$(ps -ef | grep nginx)
		if [[ ! $NGINX_STATUS =~ "nginx:" ]]
		then
			createEmail_content "* nginx未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "$NGINX_DIR" ]]
					then
							startNginx $NGINX_DIR
					else
							createEmail_content "**-->nginx路径不存在"
							createEmail_content "**-->脚本启动nginx失败"
					fi
		fi

		#DISK_GEN_STATUS=$(df -h | grep /dev/m | awk '{print $5}')
		#createEmail_content "根目录使用量为$DISK_GEN_STATUS"

		DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
		if [[ $DISK_GEN_STATUS -gt 50 ]]
		then
			createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%"
		fi
		
		if [[ -e "email_content-${TODAY_HMS}.txt" ]]
		then
			echo -e "\n\n" >> email_content-${TODAY_HMS}.txt
			createAll_tail
		fi
	}


	#$HOUR_INT=$(echo $HOUR | sed -r 's/\<0+([1-9]+)/\1/g')

	if [[ $MIN = 30 ]]
	then
		automatic_h
		if [[ -e "email_err-${TODAY_H}.txt" ]]
		then
				mail -s "${COMPUTER_NAME}检测结果" -a email_err-${TODAY_H}.txt -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_H}.txt
		else
				mail -s "${COMPUTER_NAME}检测结果" -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_H}.txt
		fi
	else
		automatic_hms
		if [[ -e "email_content-${TODAY_HMS}.txt" ]]
		then
			if [[ -e "email_err-${TODAY_HMS}.txt" ]]
			then
				mail -s "${COMPUTER_NAME}有服务未运行" -a email_err-${TODAY_HMS}.txt -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_HMS}.txt
			else
				mail -s "${COMPUTER_NAME}有服务未运行" -c $EMAIL_ADDR_1 $EMAIL_ADDR_2 < email_content-${TODAY_HMS}.txt
			fi
		fi
	fi


jdk就不监控了，根本不需要有...

### 代码更新-170918

	#!/bin/bash

	#Describe: Monitor Control System.
	# tomcat, nginx, redis
	#Author: cg
	#Time: 2017-08-31

	#----------USER CONFIG----------#

	COMPUTER_NAME=105
	TOMCAT_DIR=/home/xm6f/dev/tomcat-7.0.79
	REDIS_DIR=/home/xm6f/dev/redis-3.2.4
	REDIS_START_SHELL_CL=/home/xm6f/dev/shell/./redis_start_all.sh
	#NGINX_DIR=/usr/local/nginx

	TOMCAT_NUM=4

	PORT_1=8080
	PORT_2=8081
	PORT_3=8088
	PORT_4=8098

	PORT_1_FILENAME=tomcat-8080
	PORT_2_FILENAME=tomcat-8081
	PORT_3_FILENAME=tomcat-8088
	PORT_4_FILENAME=tomcat-8098

	#REDIS_PORT=6379

	#
	GEN_FILE_TYPE=/dev/mapper/cl-root

	EMAIL_LOG=/usr/scripts/automatic/monitor_log

	EMAIL_ADDR_1=1542723438@qq.com
	#EMAIL_ADDR_2=1679055895@qq.com

	#-------------------------------#

	TODAY_HMS=$(date +%y%m%d%H%M%S)
	TODAY_H=$(date +%y%m%d%H)
	YESTERDAY=$(date -d "yesterday" +%y%m%d)
	TIME=$(date +%Y-%m-%d-%T)
	HOUR=$(date +%H)
	MIN=$(date +%M)


	function createEmail_content() {
		if [[ $2 = -h ]]
		then
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> ${EMAIL_LOG}/email_content-${TODAY_H}.txt
			else
				echo -e "$1\c" >> ${EMAIL_LOG}/email_content-${TODAY_H}.txt
			fi
		else
			if [[ $3 -eq 0 ]]
			then
				echo "$1" >> ${EMAIL_LOG}/email_content-${TODAY_HMS}.txt
			else
				echo -e "$1\c" >> ${EMAIL_LOG}/email_content-${TODAY_HMS}.txt
			fi
		fi
	}

	function createEmail_errContent() {
			if [[ $2 = -h ]]
			then
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> ${EMAIL_LOG}/email_err-${TODAY_H}.txt
					else
							echo -e "$1\c" >> ${EMAIL_LOG}/email_err-${TODAY_H}.txt
					fi
			else
					if [[ $3 -eq 0 ]]
					then
							echo "$1" >> ${EMAIL_LOG}/email_err-${TODAY_HMS}.txt
					else
							echo -e "$1\c" >> ${EMAIL_LOG}/email_err-${TODAY_HMS}.txt
					fi
			fi
	}


	function startTomcat() {
		${1}/bin/./catalina.sh start
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对tomcat${2}执行启动命令"
		else
			createEmail_content "**-->脚本未成功对tomcat${2}执行命令"
		fi
		START_TOM_RESULT=$(ps -ef | grep tomcat)
		if [[ $START_TOM_RESULT =~ "$2" ]]
		then
			createEmail_content "**-->tomcat${2}已成功启动"
		else
			createEmail_content "**-->tomcat${2}启动未成功，请手动启动"
		fi	
	}

	function startRedis() {
		${REDIS_DIR}/./redis-server redis.conf
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对redis执行启动命令"
		else
			createEmail_content "**-->脚本未成功对reids执行命令"
		fi
		START_RED_RESULT=$(ps -ef | grep redis)
		if [[ $START_RED_RESULT =~ "redis-server" ]]
		then
			createEmail_content "**-->redis已成功启动"
		else
			createEmail_content "**-->redis启动未成功，请手动启动"
		fi
	}

	function startRedis_byShell() {
		${REDIS_START_SHELL_CL}
		if [[ $? -eq 0 ]]
		then
					createEmail_content "**-->脚本已对redis执行启动命令"
			else
					createEmail_content "**-->脚本未成功对reids执行命令"
			fi
			START_RED_RESULT=$(ps -ef | grep redis)
			if [[ $START_RED_RESULT =~ "redis-server" ]]
			then
					createEmail_content "**-->redis已成功启动"
			else
					createEmail_content "**-->redis启动未成功，请手动启动"
			fi

		
	}
			

	function startNginx() {
		${NGINX_DIR}/sbin/./nginx
		if [[ $? -eq 0 ]]
		then
			createEmail_content "**-->脚本已对nginx执行启动命令"
		else
			createEmail_content "**-->脚本未成功对nginx执行命令"
		fi
		START_NGINX_RESULT=$(ps -ef | grep nginx)
		if [[ $START_NGINX_RESULT =~ "nginx:" ]]
		then
			createEmail_content "**-->nginx已成功启动"
		else
			createEmail_content "**-->nginx启动未成功，请手动启动"
		fi
	}

	function try_start() {
			createEmail_content "* ${1}未运行"
			createEmail_content "**-->脚本将尝试进行启动...."
			if [[ -x "$2" ]]
			then
					if [[ ${1} -eq "redis" ]]
					then
							startRedis_byShell
					else
							startNginx
					fi
			else
					createEmail_content "**-->${1}路径不存在"
					createEmail_content "**-->脚本启动${1}失败"
			fi
	}



	function createAll_tail() {

	createEmail_content "======================================" $1
	createEmail_content "---林繁" $1
	createEmail_content "---$TIME" $1

	}


	function tomcat_h() {

			if [[ $TOMCAT_STATUS =~ "$1" ]]
			then
				createEmail_content "* tomcat${1}---已在运行" -h 1
					TOMCAT_LOG=$(tail -n 150 ${TOMCAT_DIR}/${2}/logs/catalina.out)
					if [[ $TOMCAT_LOG =~ "exception" ]]
					then
							createEmail_content "---日志输出异常" -h
							createEmail_errContent "===============${COMPUTER_NAME}-${1}-输出日志===============" -h
							createEmail_errContent "$TOMCAT_LOG" -h
					else
							createEmail_content "---日志输出正常" -h
					fi
			else
					createEmail_content "* tomcat${1}---未运行" -h

			fi
	}


	function tomcat_hms() {

		if [[ $TOMCAT_STATUS =~ "$1" ]]
		then
			TOMCAT_LOG=$(tail -n 150 ${TOMCAT_DIR}/${2}/logs/catalina.out)
			if [[ $TOMCAT_1_LOG =~ "exception" ]]
			then
				createEmail_content "* tomcat${1}---已运行---但日志输出异常"
				createEmail_errContent "===============${COMPUTER_NAME}-${1}-输出日志==============="
				createEmail_errContent "$TOMCAT_LOG"
			fi
		else
			createEmail_content "* tomcat${1}---未运行"
					createEmail_content "**-->脚本将尝试进行启动...."
					if [[ -x "${TOMCAT_DIR}/${2}" ]]
					then
							startTomcat ${TOMCAT_DIR}/${2} ${1}
					else
							createEmail_content "**-->tomcat${1}路径不存在"
							createEmail_content "**-->脚本启动tomcat${1}失败"
					fi
		fi
	}


	function for_tomcat_h() {
		
		TOMCAT_STATUS=$(ps -ef | grep tomcat)
			for(( i=1; i<=${TOMCAT_NUM}; i++ )) {
					PORT=PORT_$i
			FILENAME=PORT_${i}_FILENAME
					eval TOMCAT_PORT="$"$PORT
			eval TOMCAT_FILENAME="$"$FILENAME
					tomcat_h $TOMCAT_PORT $TOMCAT_FILENAME
			}
	}

	function for_tomcat_hms() {

			TOMCAT_STATUS=$(ps -ef | grep tomcat)
			for(( i=1; i<=${TOMCAT_NUM}; i++ )) {
					PORT=PORT_$i
					FILENAME=PORT_${i}_FILENAME
					eval TOMCAT_PORT="$"$PORT
					eval TOMCAT_FILENAME="$"$FILENAME
					tomcat_hms $TOMCAT_PORT $TOMCAT_FILENAME
			}
	}



	function redis_state() {

			REDIS_STATUS=$(ps -ef | grep redis)
			if [[ $REDIS_STATUS =~ "redis-server" ]]
			then
					createEmail_content "* redis已在运行" -h
			else
					createEmail_content "* redis未运行" -h
			fi
	}


	function nginx_state() {

			NGINX_STATUS=$(ps -ef | grep nginx)
			if [[ $NGINX_STATUS =~ "nginx:" ]]
			then
					createEmail_content "* nginx已在运行" -h
			else
					createEmail_content "* nginx未运行" -h
			fi
	}

	function gen_state() {

			DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
			if [[ $DISK_GEN_STATUS -lt 50 ]]
			then
					createEmail_content "* 根目录使用量较小---已使用${DISK_GEN_STATUS}%" -h
			else
					createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%" -h
			fi
	}


	function redis_stats_hms() {

			REDIS_STATUS=$(ps -ef | grep redis)
			if [[ ! $REDIS_STATUS =~ "redis-server" ]]
		then
			try_start "redis" $REDIS_DIR
		fi
	}

	function nginx_stats_hms() {

			NGINX_STATUS=$(ps -ef | grep nginx)
			if [[ ! $NGINX_STATUS =~ "nginx:" ]]
		then
			try_start "nginx" $NGINX_DIR
		fi
	}

	function gen_stats_hms() {

			DISK_GEN_STATUS=$(df -h | grep $GEN_FILE_TYPE | awk '{print $5}' | cut -f 1 -d "%")
			if [[ $DISK_GEN_STATUS -gt 50 ]]
			then
					createEmail_content "* 根目录使用量较大---已使用${DISK_GEN_STATUS}%"
			fi
	}

	function rm_email_log() {

			rm ${EMAIL_LOG}/email_*-${YESTERDAY}*.txt
			if [[ $? -eq 0 ]]
			then
					createEmail_content "昨天的邮件日志已经删除" -h
			else
					createEmail_content "昨天的邮件日志未成功删除" -h
			fi
	}


	function automatic_h() {

			echo "" > ${EMAIL_LOG}/email_content-${TODAY_H}.txt

			createEmail_content "${COMPUTER_NAME}今日${HOUR}时脚本自动执行结果" -h
			createEmail_content "======================================" -h

			for_tomcat_h
			redis_state
			#nginx_state
			gen_state

		if [[ $HOUR = 23 ]]
		then
				rm_email_log
		fi

			echo -e "\n\n" >> ${EMAIL_LOG}/email_content-${TODAY_H}.txt

			createAll_tail -h
	}


	function automatic_hms() {
		
		for_tomcat_hms
		redis_stats_hms
		#nginx_stats_hms
		gen_stats_hms
		
		if [[ -e "${EMAIL_LOG}/email_content-${TODAY_HMS}.txt" ]]
		then
			echo -e "\n\n" >> ${EMAIL_LOG}/email_content-${TODAY_HMS}.txt
			createAll_tail
		fi
	}


	#$HOUR_INT=$(echo $HOUR | sed -r 's/\<0+([1-9]+)/\1/g')

	if [[ $MIN = 30 ]]
	then
		automatic_h
		if [[ -e "${EMAIL_LOG}/email_err-${TODAY_H}.txt" ]]
		then
				mail -s "${COMPUTER_NAME}检测结果" -a ${EMAIL_LOG}/email_err-${TODAY_H}.txt $EMAIL_ADDR_1 < ${EMAIL_LOG}/email_content-${TODAY_H}.txt
		else
				mail -s "${COMPUTER_NAME}检测结果" $EMAIL_ADDR_1 < ${EMAIL_LOG}/email_content-${TODAY_H}.txt
		fi
	else
		automatic_hms
		if [[ -e "${EMAIL_LOG}/email_content-${TODAY_HMS}.txt" ]]
		then
			if [[ -e "${EMAIL_LOG}/email_err-${TODAY_HMS}.txt" ]]
			then
				mail -s "${COMPUTER_NAME}有服务未运行" -a ${EMAIL_LOG}/email_err-${TODAY_HMS}.txt $EMAIL_ADDR_1 < ${EMAIL_LOG}/email_content-${TODAY_HMS}.txt
			else
				mail -s "${COMPUTER_NAME}有服务未运行" $EMAIL_ADDR_1 < ${EMAIL_LOG}/email_content-${TODAY_HMS}.txt
			fi
		fi
	fi


优化了代码，使其模块化，同时解决了之前提到的一些bug
