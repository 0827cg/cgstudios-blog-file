---
layout: post
title: "centos搭建redis集群"
date: 2017-09-19 15:32:46
categories: linux
tags: [linux, redis]
---

为一台服务器搭建redis集群，存在ssh外网ip与服务器内网ip。
redis集群最少需要6个节点，而我这只有一台服务器，所以6个节点都在这服务器上

<!-- more -->

这里就撤步骤，完整下来分这么几步
1.检查防火墙和端口
2.安装最基本的当个redis环境
3.集群配置

### 检查防火墙和端口
系统是centos7,centos原本使用的防火墙是iptables，但在centos7版本以后就已经废弃了iptables,改用firewall-cmd了。由于公司前辈要求使用iptables，所以没办法只能跟着来

#### iptables操作
安装iptables之前需要检查firewall-cmd的运行状况，如若firewall-cmd在运行，则需将其关闭
停止firewall命令`systemctl stop firewalld`
禁止firewall开机启动` systemctl disable firewalld`
安装iptables
`yum -y install iptables-services`

开发端口则输入下面命令来修改该文件
`vim /etc/sysconfig/iptables`
如开发8080端口，则在该文件中添加如下代码
`-A INPUT -m state –state NEW -m tcp -p tcp –dport 8080 -j ACCEPT`

之后重启iptables
`systemctl restart iptables`
并设置防火墙开机启动
`systemctl enable iptables`
同样查看iptables状态
`service iptables status`或`systemctl status iptables`

#### 其他
关闭防火墙`service iptables stop`
启动防火墙`service iptables start`
重启防火墙`service iptables restart`
查看防火墙状态`service iptables status`
永久关闭防火墙`chkconfig iptables off`
永久关闭后启用`chkconfig iptables on`

### 安装基本的redis环境
安装redis之前文章提过，详情看这篇文章[搭建centos服务器][]

### 集群配置

#### 增添修改配置文件
集群至少需要6个节点，所以就需要6个端口，
安装完redis后，在/home/xm6f/dev/目录下新建`redis_cluster`文件夹，之后在该文件夹中创建7000~70056个文件夹，并每个文件夹包含子文件夹logs
命令如下

	mkdir redis_cluster
	mkdir -p 7000/logs 7001/logs 7002/logs 7003/logs 7004/logs 7005/logs

之后再将redis安装目录下的`redis.conf`这个文件分别拷贝到7000等文件夹下，之后为每个节点修改`redis.conf`这个文件
从上到下依次需要求改的内容如下

	bind 机器ip			#使用ifconfig命令获得的ip,有可能处于局域网
	port 7000			#而后依次增加7001,7002....
	daemonize yes		#后台运行
	pidfile /var/run/redis_7000.pid		#而后依次增加7001,7002....
	logfile "/root/dev/redis_cluster/7000/logs/redis.log"		#而后依次增加7001,7002....
	dbfilename dump_7000.rdb			#而后依次增加7001,7002....
	dir "/root/dev/redis-3.2.4/src"		#redis安装目录下的src
	requirepass "123456"				#配置密码
	appendonly yes
	appendfilename "appendonly_7000.aof"
	cluster-enabled yes					#开启集群
	cluster-config-file nodes-7000.conf
	cluster-node-timeout 5000

修改完上面之后，还需在该文件末尾添加一行

	masterauth "123456"				#集群密码，与requirepass一样

这里需要注意的问题就是bind的值，bind的值最好是本机ip，使用icfonfig命令得到的ip。千万不要相信手头的ip，即用ssh连接linux的ip，有可能ssh连接的ip是外网ip，而此服务器有一个内网ip.....我今天配置的就是，一直启动不了。bing使用外网ip是，命令启动redis没反应，启动不了。浪费一天了一上午的时间.....

写个脚本来一次性启动6个节点的redis

#### 启动redis

脚本代码

	cd /root/dev/redis-3.2.4/src
	./redis-server ../../redis_cluster/7000/redis.conf &
	./redis-server ../../redis_cluster/7001/redis.conf &
	./redis-server ../../redis_cluster/7002/redis.conf &
	./redis-server ../../redis_cluster/7003/redis.conf &
	./redis-server ../../redis_cluster/7004/redis.conf &
	./redis-server ../../redis_cluster/7005/redis.conf &

然后看redis运行情况，使用命令`ps -ef | grep redis`
若输出如下

	[root@VM_176_139_centos shell]# ps -ef | grep redis
	root      8833     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7000 [cluster]
	root      8834     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7001 [cluster]
	root      8835     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7005 [cluster]
	root      8836     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7002 [cluster]
	root      8837     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7004 [cluster]
	root      8838     1  0 15:12 ?        00:00:00 ./redis-server 10.104.176.139:7003 [cluster]
	root      8863  6015  0 15:12 pts/6    00:00:00 grep --color=auto redis

注意redis的url，url需要是本机ip地址
只要这样，就可以进行下一步

#### 关闭redis
命令
`pkill -9 redis`
即可同时关闭这6个redis

#### 集群连接操作
此时需要使用到redis安装目录src下的`redis-trib.rb`这个文件，这个文件是用ruby语言写的，所以需要安装ruby环境


使用命令

	./redis-trib.rb create --replicas 1 10.104.176.139:7000 10.104.176.139:7001 10.104.176.139:7002 10.104.176.139:7003 10.104.176.139:7004 10.104.176.139:7005
	
正常输出如下

	[root@VM_176_139_centos src]# ./redis-trib.rb create --replicas 1 10.104.176.139:7000 10.104.176.139:7001 10.104.176.139:7002 10.104.176.139:7003 10.104.176.139:7004 10.104.176.139:7005
	>>> Creating cluster
	>>> Performing hash slots allocation on 6 nodes...
	Using 3 masters:
	10.104.176.139:7000
	10.104.176.139:7001
	10.104.176.139:7002
	Adding replica 10.104.176.139:7003 to 10.104.176.139:7000
	Adding replica 10.104.176.139:7004 to 10.104.176.139:7001
	Adding replica 10.104.176.139:7005 to 10.104.176.139:7002
	M: 7fed05243fa5e8d44b04b3ef5b4b2d246a20bf85 10.104.176.139:7000
	   slots:0-5460 (5461 slots) master
	M: 3cb3569fb1efec5cdb5ec11f4ac863279a702bd9 10.104.176.139:7001
	   slots:5461-10922 (5462 slots) master
	M: 26e7e5c215ff685a06908da9418bb0d6145d2eb2 10.104.176.139:7002
	   slots:10923-16383 (5461 slots) master
	S: a2ebe312b7dbab29aa60c404e347e8570f598273 10.104.176.139:7003
	   replicates 7fed05243fa5e8d44b04b3ef5b4b2d246a20bf85
	S: a346f606aa0a1f2bd96c95b1831582f30aae04e2 10.104.176.139:7004
	   replicates 3cb3569fb1efec5cdb5ec11f4ac863279a702bd9
	S: 54d3fc53cde48495dcf709c1be03b483001bd873 10.104.176.139:7005
	   replicates 26e7e5c215ff685a06908da9418bb0d6145d2eb2
	Can I set the above configuration? (type 'yes' to accept): yes   
	>>> Nodes configuration updated
	>>> Assign a different config epoch to each node
	>>> Sending CLUSTER MEET messages to join the cluster
	Waiting for the cluster to join...
	>>> Performing Cluster Check (using node 10.104.176.139:7000)
	M: 7fed05243fa5e8d44b04b3ef5b4b2d246a20bf85 10.104.176.139:7000
	   slots:0-5460 (5461 slots) master
	   1 additional replica(s)
	S: a346f606aa0a1f2bd96c95b1831582f30aae04e2 10.104.176.139:7004
	   slots: (0 slots) slave
	   replicates 3cb3569fb1efec5cdb5ec11f4ac863279a702bd9
	S: 54d3fc53cde48495dcf709c1be03b483001bd873 10.104.176.139:7005
	   slots: (0 slots) slave
	   replicates 26e7e5c215ff685a06908da9418bb0d6145d2eb2
	M: 26e7e5c215ff685a06908da9418bb0d6145d2eb2 10.104.176.139:7002
	   slots:10923-16383 (5461 slots) master
	   1 additional replica(s)
	M: 3cb3569fb1efec5cdb5ec11f4ac863279a702bd9 10.104.176.139:7001
	   slots:5461-10922 (5462 slots) master
	   1 additional replica(s)
	S: a2ebe312b7dbab29aa60c404e347e8570f598273 10.104.176.139:7003
	   slots: (0 slots) slave
	   replicates 7fed05243fa5e8d44b04b3ef5b4b2d246a20bf85
	[OK] All nodes agree about slots configuration.
	>>> Check for open slots...
	>>> Check slots coverage...
	[OK] All 16384 slots covered.
	[root@VM_176_139_centos src]# 

连接完成之后，就可以测试各个节点的连接情况了，这里先不撤...

#### 重新连接集群
如果需要重新连接集群，那么就需要找到redis.conf配置文件中
`dir "/root/dev/redis-3.2.4/src"`
这个文件路径，并删除里面的带端口的文件，就是上面提到的配置方法中的文件
直接在该文件目录下运行命令

`sudo rm appendonly_*.conf nodes-*.conf dump_*.rdb`

引用: [Redis 3.2.4集群实战][]

[搭建centos服务器]:http://cgspace.date/2017/08/07/linux/2017-08-07-install-centos-server/
[Redis 3.2.4集群实战]:http://www.cnblogs.com/linjiqin/p/7451822.html

