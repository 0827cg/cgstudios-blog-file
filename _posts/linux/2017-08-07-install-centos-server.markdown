---
layout: post
title: "搭建centos服务器"
date: 2017-08-07 20:46:22
categories: linux
tags: [linux, centos, redis, tomcat, nginx]
---

7月31号第一天上班，boss找那个java开发的项目经理带我。第一天的工作内容就是搭建个centos服务器，其中已经可以外网访问了。centos中需要配置三台tomcat,不同端口访问，并配置redis数据库。工作不难，下面这些就是我在搭建过程中的记录

### 安装Centos7系统

原本主机中安装的是Ubuntu系统，需要将它全部删除，安装centos系统，这个不难不扯。不过在安装过程中需要考虑分区问题及是否需要图形化界面，当然也需要创建账户名和密码及root密码，这里我使用了图形化界面，安装了gnome桌面环境，要安装图形化界面在安装过程中取消最小安装，选择自己所需要的即可。

<!-- more -->

不过安装完centos系统后，其中需要配置一项，就是设置sudo命令可以被普通用户xm6f所使用，不然将会报
    xm6f 不在 sudoers 文件中。此事将被报告。
这个错，所以使用命令操作/etc/sudoers这个文件，在文件末尾添加如下一行
`xm6f ALL=(ALL) ALL`
完后就可以使用在普通用户下使用sudo命令来暂时使用root.按需求，在安装完系统后需要将主机ip设为静态ip，将ip固定。

#### 设置静态ip

可以使用命令ifconfig查看ip信息，但现在需要进行更改，就需要修改网卡的配置文件，该配置文件存放在/etc/sysconfig/network-scripts目录下，其名字为"ifcfg-"加上网卡名，首先先来到network-scripts目录下，如下输出

	[xm6f@localhost ~]$ cd /etc/sysconfig/network-scripts/
	[xm6f@localhost network-scripts]$ ls
	ifcfg-enp2s0  ifdown-ppp       ifup-ib      ifup-Team
	ifcfg-lo      ifdown-routes    ifup-ippp    ifup-TeamPort
	ifdown        ifdown-sit       ifup-ipv6    ifup-tunnel
	ifdown-bnep   ifdown-Team      ifup-isdn    ifup-wireless
	ifdown-eth    ifdown-TeamPort  ifup-plip    init.ipv6-global
	ifdown-ib     ifdown-tunnel    ifup-plusb   network-functions
	ifdown-ippp   ifup             ifup-post    network-functions-ipv6
	ifdown-ipv6   ifup-aliases     ifup-ppp
	ifdown-isdn   ifup-bnep        ifup-routes
	ifdown-post   ifup-eth         ifup-sit
	[xm6f@localhost network-scripts]$ sudo gedit ifcfg-enp2s0
	...

其中上面的ifcfg-enp2s0就是本centos7服务器主机的网卡配置文件，我们需要修改的就是这个配置文件，起初这个配置文件的内容如下

	TYPE=Ethernet
	BOOTPROTO=dhcp
	DEFROUTE=yes
	PEERDNS=yes
	PEERROUTES=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_PEERDNS=yes
	IPV6_PEERROUTES=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=enp2s0
	UUID=e65481a0-7c13-426d-99cb-b8b281294cb8
	DEVICE=enp2s0
	ONBOOT=no

这里我们需要将其更改，首先修改如下两项

	BOOTPROTO=static
	ONBOOT=yes

之后，再在末尾添加下面几项

	IPADDR=192.168.1.149	#最后要的ip地址
	GATEWAY=192.168.1.1	#网关
	DNS1=192.168.1.1

之后运行命令
`sudo systemctl restart network.service`
或
`sudo service network restart`
重启网卡服务,之后查看ip信息则用命令
`ip add`
用此命令来查看ip信息，ip更改过来并且能上网，才算成功，之后就是为这台centos7服务器主机搭建java环境环境安装tomcat等，搭建java环境之前，需要将centos系统中的openjdk卸载，这是centos原装的，openjdk用得少，一般都用oraclejdk,所以看下面如何卸载openjdk

#### 卸载openjdk

卸载centos7上的openjdk
先查看,使用命令
`rpm -qa | grep java`
例如我的输出,显示如下信息
    java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64
    javapackages-tools-3.4.1-11.el7.noarch
    java-1.8.0-openjdk-headless-1.8.0.102-4.b14.el7.x86_64
    tzdata-java-2016g-2.el7.noarch
    python-javapackages-3.4.1-11.el7.noarch
    java-1.7.0-openjdk-headless-1.7.0.111-2.6.7.8.el7.x86_64
    java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64
卸载命令
    rpm -e --nodeps java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64
    rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.102-4.b14.el7.x86_64
还可以这样卸载
    java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64
    java-1.8.0-openjdk-headless-1.8.0.102-4.b14.el7.x86_64
最后我是删成了这样
    [xm6f@localhost jdk1.7.0_80]$ rpm -qa | grep java
    javapackages-tools-3.4.1-11.el7.noarch
    tzdata-java-2016g-2.el7.noarch
    python-javapackages-3.4.1-11.el7.noarch
    [xm6f@localhost jdk1.7.0_80]$ java
    -bash: /usr/bin/java: 没有那个文件或目录
然后就是安装oraclejdk，配置环境变量，搭建java环境可以看这移步到这篇文章:[java环境搭建][]

### 安装Tomcat

在安装Tomcat之前，因为Tomcat是需要java环境，所以先要安装为centos服务器安装jdk。然而需要在自己电脑上操作服务器，公司给我配置的电脑是windows笔记本，那么就需要在电脑上安装xshell这个软件，用它来连接操作centos服务器，下载安装xshell后，打开xshell，利用ip和账户名和密码连接到centos7这台电脑，并了解centos7这台电脑上是否安装lrzsz这个文件传输工具，无则命令安装
`sudo yum install -y lrzsz`
之后再输入命令'rz'用来上传文件，这里需要注意到，上传之前可能需要切换到/tmp文件夹下来进行上传，貌似是权限的问题
上传完成之后，使用tar命令来解压文件，如命令
`sudo tar -zvxf jdk-8u144-linux-x64.tar.gz`
通常解压到/usr/local目录...解压完后直接在安装目录尝试启动tomcat看看，因为这里需要配置多个tomcat，那么就不能为单一的tomcat配置环境变量，这样会导致在哪都只能启动配置了环境变量的tomcat，而不能启动其他的tomcat。所以现在便捷的方式是不为任何一个tomcat配置环境变量，那么这样的话，启动tomcat需要到相应的tomcat文件目录中的bin目录下运行命令`./catalina.sh start` 或`./startup.sh`来启动，关闭则为`./catalina.sh stop`或`./shutdown.sh`

#### 启动tomcat并开放8080端口

有可能cd不能进入tomcat里的/bin文件夹，会提示权限不够，或者在启动Tomcat的时候提示权限不够，可以先用命令
`ls -l`
来查看下文件或文件夹的运行权限和所属用户组，如果运行权限不足则增加相应的权限，或者更改文件的拥有者，例如命令
`sudo chown -R xm6f bin`
即将bin目录文件的拥有者更改成xm6f普通用户，这样就可以进入bin目录了，进入bin目录后，就可以运行命令来启动tomcat了
命令
`./catalina.sh start`或者`./startup.sh`
因为使用的是8080端口，大于1024端口，而且文件所有者已经更改成了xm6f普通用户，所以不需要使用root来进行启动，启动之后再centos服务器电脑上就使用http://localhost:8080这个链接来访问tomcat服务器主页，那我在自己电脑上来进行访问，其链接为:http://192.168.1.149:8080来进行访问。不过需要注意的是，这样访问需要centos服务器已经对外开放了8080端口，默认对外开发的端口为20，并没有8080，所以需要手动开启，不然照样访问不了
其开启对外开发8080端口命令如下
`sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent`
运行上面这条命令后，如果运行正常返回success那么再运行下面这条命令
`sudo firewall-cmd --reload`
若同样正常返回success那么就成功的对外开发了8080端口，另外提一点，因为现在操作的是centos7系统，这里可以有两种方式来开放端口，一种就是上面的使用firewall-cmd命令，另一种就是操作iptables,iptables我不大会操作，不过后期再补上。如果没使用firewall-cmd来开放端口的的话，估计就是iptables里面开放了。我之前就是，同事使用iptables开放了端口，而我又不知道，使用firewall-cmd命令
`sudo firewall-cmd --list-ports`
查看了所以的已经开放了的端口号，发现我要开放的端口在这个输出中并不存在，那就肯定是iptables上进行开放了。

#### 配置三台tomcat

而需要配置三台tomcat，那就需要有三个tomcat安装文件，如下

	[xm6f@localhost tomcat]$ ls
	apache-tomcat-7.0.79  apache-tomcat-7.0.79-2  apache-tomcat-7.0.79-3

依照上面的输出顺序，我配置的端口依次是8080,8081,8082,具体的配置中，apache-tomcat-7.0.79配置文件可以不变，只需要更改后面两个tomcat的配置文件，避免同时启动三台tomcat时出现端口占用的情况。其tomcat配置文件即安装目录的/conf/server.xml，其原始内容为

	<?xml version='1.0' encoding='utf-8'?>
	<!--
	  Licensed to the Apache Software Foundation (ASF) under one or more
	  contributor license agreements.  See the NOTICE file distributed with
	  this work for additional information regarding copyright ownership.
	  The ASF licenses this file to You under the Apache License, Version 2.0
	  (the "License"); you may not use this file except in compliance with
	  the License.  You may obtain a copy of the License at

		  http://www.apache.org/licenses/LICENSE-2.0

	  Unless required by applicable law or agreed to in writing, software
	  distributed under the License is distributed on an "AS IS" BASIS,
	  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	  See the License for the specific language governing permissions and
	  limitations under the License.
	-->
	<!-- Note:  A "Server" is not itself a "Container", so you may not
		 define subcomponents such as "Valves" at this level.
		 Documentation at /docs/config/server.html
	 -->
	<Server port="8005" shutdown="SHUTDOWN">
	  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
	  <!-- Security listener. Documentation at /docs/config/listeners.html
	  <Listener className="org.apache.catalina.security.SecurityListener" />
	  -->
	  <!--APR library loader. Documentation at /docs/apr.html -->
	  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
	  <!--Initialize Jasper prior to webapps are loaded. Documentation at /docs/jasper-howto.html -->
	  <Listener className="org.apache.catalina.core.JasperListener" />
	  <!-- Prevent memory leaks due to use of particular java/javax APIs-->
	  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
	  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
	  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

	  <!-- Global JNDI resources
		   Documentation at /docs/jndi-resources-howto.html
	  -->
	  <GlobalNamingResources>
		<!-- Editable user database that can also be used by
			 UserDatabaseRealm to authenticate users
		-->
		<Resource name="UserDatabase" auth="Container"
				  type="org.apache.catalina.UserDatabase"
				  description="User database that can be updated and saved"
				  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
				  pathname="conf/tomcat-users.xml" />
	  </GlobalNamingResources>

	  <!-- A "Service" is a collection of one or more "Connectors" that share
		   a single "Container" Note:  A "Service" is not itself a "Container",
		   so you may not define subcomponents such as "Valves" at this level.
		   Documentation at /docs/config/service.html
	   -->
	  <Service name="Catalina">

		<!--The connectors can use a shared executor, you can define one or more named thread pools-->
		<!--
		<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
			maxThreads="150" minSpareThreads="4"/>
		-->


		<!-- A "Connector" represents an endpoint by which requests are received
			 and responses are returned. Documentation at :
			 Java HTTP Connector: /docs/config/http.html (blocking & non-blocking)
			 Java AJP  Connector: /docs/config/ajp.html
			 APR (HTTP/AJP) Connector: /docs/apr.html
			 Define a non-SSL HTTP/1.1 Connector on port 8080
		-->
		<Connector port="8080" protocol="HTTP/1.1"
				   connectionTimeout="20000"
				   redirectPort="8443" />
		<!-- A "Connector" using the shared thread pool-->
		<!--
		<Connector executor="tomcatThreadPool"
				   port="8080" protocol="HTTP/1.1"
				   connectionTimeout="20000"
				   redirectPort="8443" />
		-->
		<!-- Define a SSL HTTP/1.1 Connector on port 8443
			 This connector uses the BIO implementation that requires the JSSE
			 style configuration. When using the APR/native implementation, the
			 OpenSSL style configuration is required as described in the APR/native
			 documentation -->
		<!--
		<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
				   maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
				   clientAuth="false" sslProtocol="TLS" />
		-->

		<!-- Define an AJP 1.3 Connector on port 8009 -->
		<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />


		<!-- An Engine represents the entry point (within Catalina) that processes
			 every request.  The Engine implementation for Tomcat stand alone
			 analyzes the HTTP headers included with the request, and passes them
			 on to the appropriate Host (virtual host).
			 Documentation at /docs/config/engine.html -->

		<!-- You should set jvmRoute to support load-balancing via AJP ie :
		<Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
		-->
		<Engine name="Catalina" defaultHost="localhost">

		  <!--For clustering, please take a look at documentation at:
			  /docs/cluster-howto.html  (simple how to)
			  /docs/config/cluster.html (reference documentation) -->
		  <!--
		  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
		  -->

		  <!-- Use the LockOutRealm to prevent attempts to guess user passwords
			   via a brute-force attack -->
		  <Realm className="org.apache.catalina.realm.LockOutRealm">
			<!-- This Realm uses the UserDatabase configured in the global JNDI
				 resources under the key "UserDatabase".  Any edits
				 that are performed against this UserDatabase are immediately
				 available for use by the Realm.  -->
			<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
				   resourceName="UserDatabase"/>
		  </Realm>

		  <Host name="localhost"  appBase="webapps"
				unpackWARs="true" autoDeploy="true">

			<!-- SingleSignOn valve, share authentication between web applications
				 Documentation at: /docs/config/valve.html -->
			<!--
			<Valve className="org.apache.catalina.authenticator.SingleSignOn" />
			-->

			<!-- Access log processes all example.
				 Documentation at: /docs/config/valve.html
				 Note: The pattern used is equivalent to using pattern="common" -->
			<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
				   prefix="localhost_access_log." suffix=".txt"
				   pattern="%h %l %u %t &quot;%r&quot; %s %b" />

		  </Host>
		</Engine>
	  </Service>
	</Server>

其需要更改的地方只有三处
第一处如下

	<Server port="8005" shutdown="SHUTDOWN">

第二台更改成9005,第三台更改成7005
第二处如下

	<Connector port="8080" protocol="HTTP/1.1"
		connectionTimeout="20000"
		redirectPort="8443" />

第二台更改成8081,9443，第三台更改成8082,7443
第三处如下

	<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

第二胎更改成9009,9443，第三台更改成7009,7443
并且将8081和8082这两个端口都进行对外开放，这样就可以了，要测试的话，分别到各个tomcat目录中将其都启动，浏览器输入ip地址和对应的端口号即可。

至此，就完成了tomcat的安装

### 安装redis

centos7服务器电脑还需要安装redis
还没接触过redis，后面跟同事了解到，redis是使用c编写的，redis一般在系统开发中只用作缓存，因为如果要使用的数据量太大的话，而且需要多次使用，如果不使用缓存，那么在每一次请求数据的时候都需要从数据库中进行查找，这样不仅耗时也好资源，所以如果使用redis缓存机制，用户请求数据的时候，如果数据量过大，就会使用redis将该数据暂时放入内存中，以键值对的方式存放着，并于该请求用户的临时id进行关联，使得再次使用到该数据的时候可以直接使用get方式从内存中读取，而不需要从数据库中查.....ok
那么下载安装,下载redis-2.8.4.tar压缩包后，将压缩包上传到centos7服务器中，现在操作的这台主机不知道为什么，只能通过tmp目录上传，那么每次上传完成之后在tmp目录下进行解压，之后直接cd进入解压得到的文件夹，例如这里解压得到的文件夹名为redis-2.8.6，使用命令进行编译，再编译完成之后会在目录redis-2.8.6/src下会得到可执行程序，最好把可执行程序单独拿出来，放到/usr/local目录下，同样也把存放在/redis-2.8.6目录下的redis.conf这个配置文件也拿出来，这是解压之后得到的。在/usr/local/目录下创建一个redis文件夹来一起存放，之后手动启动和关闭redis都在这个目录进行操作，后期配置开机启动也使用这个文件夹里的可执行程序。
因为需要gcc编译，查看了后知道未安装gcc，那么先安装gcc,其命令
`sudo yum install gcc make`
之后解压redis-2.8.4.tar,并编译
解压命令
`sudo tar -zvxf redis-2.8.4.tar`
解压之后cd到解压得到的目录中，运行命令sudo make来编译，出了个错误

    zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录

网上查了之后才知道命令中需要添加一个参数变量,运行命令sudo make MALLOC=libc,就不会出现这个问题，然后测试redis是否安装成功，运行命令
`make test`
时，出现个问题

	You need tcl 8.5 or newer in order to run the Redis test

看着抛出的错误，提示这需要下载tcl8.5并编译安装

	cd  /usr/local/tcl8.6.1/unix/  
	sudo ./configure  
	sudo make  
	sudo make install 

之后，再`make test`，这里我的还是不行，不知道为什么,我选择的是先不管它，直接将redis的服务器和客户端等复制到/usr/local/bin/目录下
`sudo cp redis-server redis-cli /usr/local/bin/`
及其他一些可运行程序
`sudo cp redis-sentinel redis-benchmark redis-check-aof redis-check-dump /usr/local/bin/`
之后新开一个终端连接，直接命令redis-server和redis-cli发现都可以运行,并且在redis-cli下测试都正常，如下

	[xm6f@localhost redis-2.8.4]$ ./src/redis-cli 
	127.0.0.1:6379> set name aa
	OK
	127.0.0.1:6379> get name
	"aa"
	127.0.0.1:6379> 

只要这样测试结果正常，那就意味着redis安装成功，不过，redis需要配置开机启动，redis运行的端口号一般默认时6379

#### 配置redis开机启动

所以为了方便启动和管理，我在/usr/local/下新建了个redis文件夹，并将redis-server，redis-cli，redis-sentinel，redis-benchmark，redis-check-aof，redis-check-dump及redis.conf复制到该redis文件夹下，配置开机启动的方式就是在/etc/rc.d/init.d目录下创建一个启动脚本，命令如下
`sudo vim redis`
脚本是从网上找的，只是进行了修改，其内容如下

	#!/bin/sh
	#chkconfig: 345 86 14
	#description: Startup and shutdown script for Redis
	 
	PROGDIR=/usr/local/redis
	PROGNAME=redis-server
	DAEMON=$PROGDIR/$PROGNAME
	CONFIG=/usr/local/redis/redis.conf
	PIDFILE=/var/run/redis.pid
	DESC="redis daemon"
	SCRIPTNAME=/etc/rc.d/init.d/redis
	 
	start()
	{
		if test -x $DAEMON
			then
			echo -e "Starting $DESC: $PROGNAME"
					  if $DAEMON $CONFIG
					  then
								echo -e "OK"
					  else
								echo -e "failed"
					  fi
			else
					  echo -e "Couldn't find Redis Server ($DAEMON)"
			fi
	}
	 
	stop()
	{
			if test -e $PIDFILE
			then
					  echo -e "Stopping $DESC: $PROGNAME"
					  if kill `cat $PIDFILE`
					  then
								echo -e "OK"
					  else
								echo -e "failed"
					  fi
			else
					  echo -e "No Redis Server ($DAEMON) running"
			fi
	}
	 
	restart()
	{
		echo -e "Restarting $DESC: $PROGNAME"
		stop
			start
	}
	 
	list()
	{
			ps aux | grep $PROGNAME
	}
	 
	case $1 in
			start)
					  start
			;;
			stop)
			stop
			;;
			restart)
			restart
			;;
			list)
			list
			;;
	 
			*)
			echo "Usage: $SCRIPTNAME {start|stop|restart|list}" >&2
			exit 1
			;;
	esac
	exit 0

之后就需要修改对redis进行chkconfig配置，其命令如下

	sudo chkconfig --add redis
	sudo chkconfig --level 345 redis on    #345这个数字根据脚本中来定
	sudo chkconfig --list redis

之后就可以重启系统进行测试了，只要确定redis能在开机后启动就可以了
获取详细信息命令
`./redis-server --help`
`./redis-cli --help`
关闭命令
`pkill redis-server`
或
`redis-cli shutdown`

#### 手动启动redis

手动启动redis一般需要到redis的安装目录的src文件夹下运行redis-server，不过这里我已经将这些可运行程序及配置文件拷贝到/usr/local/redis/目录下，所以可以直接在这个目录下手动启动redis,其命令如下

	cd /usr/local/redis
	./redis-server redis.conf	#启动redis
	./redis-cli -p 端口号 shutdown	#关闭redis服务

	./redis-cli		#启动redis客户端

查看redis服务是否启动，命令如下
`ps -aux | grep redis`
查看某端口是否被占用,命令如下
`netstat –tunpl | grep 端口`
后期配置
在/etc/sysctl.conf文件中添加下面代码:
`vm.overcommit_memory=1`
这个代码可以解决下面这个问题

	[3876] 01 Aug 14:09:11.809 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run 	the command 'sysctl vm.overcommit_memory=1' for this to take effect.

### 安装nginx服务

nginx是负责均衡服务器，是c语言开发的，相当于一个调配工具，那个tomcat压力大就调用压力轻的来顶

#### 配置ngin环境

安装nginx之前先安装好其所需环境
1.安装gcc
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，
命令
`yum install gcc-c++`
2.安装PCRE库  pcre-devel库
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库
命令
`yum install -y pcre pcre-devel`
3.zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库
命令
`yum install -y zlib zlib-devel`
4.OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议,nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库
命令
`yum install -y openssl openssl-devel`

#### 安装nginx

在tmp文件夹下，使用`rz`命令上传下载好的nginx-1.12.1.tar压缩文件，之后对其进行解压，将得到一个nginx-1.12.1文件夹，之后安装的命令如下

	cd nginx-1.12.1
	sudo ./configure
	sudo make && sudo make install

这样的编译安装方式会自动将nginx安装到/usr/local/nginx下，如果我们需要自定义nginx的安装目录，例如我需要安装在/home/xm6f/dev/nginx-1.12.1目录下，那么可以使用下面的命令
`./configure --prefix=/home/xm6f/dev/nginx-1.12.1`
然后我们进入到安装目录下的sbin文件夹中，使用命令
`sudo ./nginx`
就可以运行nginx了
提示
如果不使用sudo来启动nginx就会出现如下错误

	nginx: [emerg] bind() to 0.0.0.0:80 failed (13: Permission denied)

这个错误不是文件权限错误，而是端口权限的错误，在linux系统中，所有启动数字小于1024的端口都需要root权限，而nginx使用的80端口，所以，就需要sudo使用root身份

#### nginx启动与关闭

启动、停止nginx的命令如下

	cd /usr/local/nginx/sbin/
	sudo ./nginx  : 启动nginx
	sudo ./nginx -s stop  : 此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程
	sudo ./nginx -s quit  : 此方式是待nginx进程处理任务完毕进行停止
	sudo ./nginx -s reload  : 重启
 	
之后，跟redis一样，也需要将nginx添加开机启动，开机启动我不是特别了解，网上了解到好像有两种区别，一种是如果直接使用编译安装的方式，那其添加到开机启动的方法就是编辑/etc/rc.local这个文件，往其中添加下面的代码
`/usr/local/nginx/sbin/nginx`
然后就是修改/etc/rc.local这个文件的权限，在/etc目录下使用命令
`sudo chmod 755 rc.local`
如果不修改这个文件的权限，那其开机后只会启动一个进程，如下

	Last login: Thu Aug  3 11:22:37 2017 from 192.168.1.70
	[xm6f@localhost ~]$ ps -ef | grep nginx
	xm6f      2963  2919  0 13:53 pts/0    00:00:00 grep --color=auto nginx

这个进程只能被普通用户xm6f控制，而正常启动的nginx会有三个进程，如下

	Type `help' to learn how to use Xshell prompt.
	[c:\~]$ open

	Connecting to 192.168.1.149:22...
	Connection established.
	To escape to local shell, press 'Ctrl+Alt+]'.
	
	Last login: Thu Aug  3 13:53:21 2017 from 192.168.1.70
	[xm6f@localhost ~]$ ps -ef | grep nginx
	root      1245     1  0 13:55 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
	nobody    1246  1245  0 13:55 ?        00:00:00 nginx: worker process
	xm6f      2974  2923  0 13:57 pts/0    00:00:00 grep --color=auto nginx
	[xm6f@localhost ~]$ 

另一个区别就是如果是使用yum命令安装的方式，就是用systemctl来启动，具体我也不了解......
到现在为止，已经成功的安装了nginx并设为开机启动了,那现在需要做的就是配合前面安装的三台tomcat来实现负载均衡，打开nginx安装目录中conf文件夹下的nginx.conf这个配置文件，它是nginx的配置文件，其原始的内容如下


	#user  nobody;
	worker_processes  1;

	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	#pid        logs/nginx.pid;


	events {
		worker_connections  1024;
	}


	http {
		include       mime.types;
		default_type  application/octet-stream;

		#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
		#                  '$status $body_bytes_sent "$http_referer" '
		#                  '"$http_user_agent" "$http_x_forwarded_for"';

		#access_log  logs/access.log  main;

		sendfile        on;
		#tcp_nopush     on;

		#keepalive_timeout  0;
		keepalive_timeout  65;

		#gzip  on;

		server {
			listen       80;
			server_name  localhost;

			#charset koi8-r;

			#access_log  logs/host.access.log  main;

			location / {
				root   html;
				index  index.html index.htm;
			}

			#error_page  404              /404.html;

			# redirect server error pages to the static page /50x.html
			#
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}

			# proxy the PHP scripts to Apache listening on 127.0.0.1:80
			#
			#location ~ \.php$ {
			#    proxy_pass   http://127.0.0.1;
			#}

			# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
			#
			#location ~ \.php$ {
			#    root           html;
			#    fastcgi_pass   127.0.0.1:9000;
			#    fastcgi_index  index.php;
			#    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
			#    include        fastcgi_params;
			#}

			# deny access to .htaccess files, if Apache's document root
			# concurs with nginx's one
			#
			#location ~ /\.ht {
			#    deny  all;
			#}
		}


		# another virtual host using mix of IP-, name-, and port-based configuration
		#
		#server {
		#    listen       8000;
		#    listen       somename:8080;
		#    server_name  somename  alias  another.alias;

		#    location / {
		#        root   html;
		#        index  index.html index.htm;
		#    }
		#}


		# HTTPS server
		#
		#server {
		#    listen       443 ssl;
		#    server_name  localhost;

		#    ssl_certificate      cert.pem;
		#    ssl_certificate_key  cert.key;

		#    ssl_session_cache    shared:SSL:1m;
		#    ssl_session_timeout  5m;

		#    ssl_ciphers  HIGH:!aNULL:!MD5;
		#    ssl_prefer_server_ciphers  on;

		#    location / {
		#        root   html;
		#        index  index.html index.htm;
		#    }
		#}

	}

在nginx.conf配置文件中添加如下代码，位置可以在gzip这一行后面，只要在location前面即可

	upstream tomcat{
		server 192.168.1.149:8080 weight=1;
		server 192.168.1.149:8081 weight=1;
		server 192.168.1.149:8082 weight=1;
	}

并将其中的`gzip  on`前面的注释，这个功能是将需要发送到浏览器的文件进行压缩，之后再修改location块中的内容修改成如下

	location / {
		proxy_buffering on;

		proxy_connect_timeout 3;
		proxy_send_timeout 3;
		proxy_read_timeout 3;
		proxy_pass  http://tomcat;
		# root   html;
		# index  index.html index.htm;
	}

这算是最基本的修改了，后面还得涉及到nginx性能的优化配置，这里先不扯。继续看上面的原始配置文件，nginx的server块中表明了nginx使用的地址server_name和端口号listen，其端口号是80端口，为使服务器能够外网访问就需要开放80端口，所以我们需要开放端口，其命令为
`sudo firewall-cmd --zone=public --add-port=80/tcp --permanent`
之后运行下面的命令
`sudo firewall-cmd --reload`
两者都正常运行返回success即为开放成功，另外提下，列出所有已经开放的端口命令如下
`sudo firewall-cmd --list-ports`
目前，将nginx服务启动后，在我自己的笔记本上使用centos7服务器的ip及80端口通过浏览器即可访问到tomcat，但是负载均衡不能得到体现，所以我们可以修改三台tomcat安装目录下的/webapps/ROOT/index.jsp页面，来区分三台不同的tomcat，为了方便，我就修改了index.jsp页面的title，每个title的值就是相应tomcat所在的端口号，浏览器中输入http://192.168.1.149:80出现的页面就会使title=8080的tomcat欢迎页面，每次点击刷新，title都会更换，即tomcat都会更换，这就实现了nginx的负载均衡。
至此nginx安装与配置已经完成
查询nginx进程：
`ps aux|grep nginx`
查看服务的状态
`service nginx status或systemctl nginx status`

### 其他

存储帐号的文件：/etc/passwd
存储密码的文件：/etc/shadow
查看所有用户命令
`cat /etc/passwd`
便捷的输出：
`grep bash /etc/passwd`
查看所有用户对于的密码
`cat /etc/shadow | grep 用户名`
通过/etc/shadow获取的只是密码加密后的Hash散列值，要获取明文密码，需要自己进行破解
root下修改普通用户命令
`passwd 用户名`
root下修改自己密码命令
`passwd`



[java环境搭建][https://cgspace.date/2017/03/08/java/2017-3-08-java-EnvironmentVariable/]
