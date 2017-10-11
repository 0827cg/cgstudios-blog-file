---
layout: post
title: "linux服务器运维问题"
date: 2017-08-20 08:48:40
categories: linux
tags: [linux, centos, redis, tomcat, nginx]
---

今天周末，也是无聊。昨天刚搬玩家，新的住址里公司挺近的，所以就来公司坐着了，打算总结一下这个星期工作中遇到的关于linux服务器的问题，多总结多学习，还得好好提升自己

<!-- more -->

### tomcat不能停止

一个项目中，将首先是遇到浏览器页面打不开的问题，然后打算重启tomcat,运行命令后就报错停止不了，输出如下

    Java HotSpot(TM) 64-Bit Server VM warning: Insufficient space for shared memory file:
       29581
    Try using the -Djava.io.tmpdir= option to select an alternate temp location.

看到这个问题，意识到内存不够，被占满了。然后就发现服务器的根目录已经全部被使用了，后面会详细说明根目录被全部使用的解决方法，这里先解决这个问题。把tomcat进程杀死，在将目录空间清除一些后在启动tomcat,浏览器访问发现打不开页面，页面显示下面这个错误

    dial tcp 192.168.1.7:8098: connectex: No connection could be made because the target machine actively refused it.

面对这个问题的解决方式是使用sudo命令来启动tomcat。这貌似跟系统有关，起初以为是权限问题，因为ps之后发现tomcat8098并不在运行，然后将tomcat安装目录都跟该用户所有者为xm6f普通用户，这样还是没用，依然解决不了这个问题。而7这台服务器装的是ubuntu系统，并不是centos系统，centos系统上用普通用户权限启动tomcat还没遇到过这样的问题，最多是权限不够无法写入日志。而这台，tomcat安装目录所有文件权限都有且文件所有者是普通用户，依然启动不了。而解决方式就是仅仅只需要用sudo命令来启动tomcat，即可

另一个tomcat不能停止的问题，例如如下输出

    PID file found but no matching process was found. Stop aborted.

这个问题这个是因为tomcat进程被强杀了，但是文件run.pid(这个文件来记录进程号的)中还是依然保存着被杀的进程id，也就是说系统中实际运行的tomcat进程id与run.pid中保存的进程id不一样，使得使用命令将tomcat关闭行不通
解决方法就是将电脑服务器重启或者直接将现在的tomcat进程杀死就可以了

### tomcat部署项目问题

之前测试将apple部署到远程服务器上
使用rz命令将lotmall.war这个文件上传到tomcat8080安装目录的webapps下，之后访问，出现404，后来再查看，发现自动解压出来的lotmall这个文件夹和lotmall.war都不存在，被删了，然后再重新上传，还是被删。后面才知道需要先将tomcat关闭，再上传war包，之后启动tomcat进行访问就不会自动被删除了。部署上去后，命令运行tomcat后，测试运行是否正常，发现浏览器打不开页面，再使用命令
`ps -ef | grep tomcat`
发现tomcat不在运行,使用root身份启动之后，还是不运行，查看logs也没有发现什么错误，之后索性重启机器，然后再次启动tomcat后检查是否正在运行，嗯tomcat正常运行
然而浏览器还是访问不了，于是就猜测是端口未开放的问题，命令查看所有已开放端口
`firewall-cmd --list-ports`
其输出是
`FirewallD is not running`
看来防火墙没有打开，于是便使用命令查看状态
`systemctl status firewalld` 
确认下
其输出如下

    [root@localhost bin]# systemctl status firewalld
    ● firewalld.service - firewalld - dynamic firewall daemon
       Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
       Active: inactive (dead)
         Docs: man:firewalld(1)

确实不在运行，所以就命令让它运行
`systemctl start firewalld`
即可启动，查看下状态可知inactive变为running，即在运行，然后命令查看所有已开放的端口，发现没有一个端口开放，这里需要开放8080端口
于是命令
`sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent`
将其开放，在重新加载
`sudo firewall-cmd --reload`
之后浏览器就可以访问了

### linux中的top命令

直接运行命令
`top`
其输出的讲解如下
第一行是任务队列信息，同 uptime 命令的执行结果，其解析如下：
01:06:48  当前时间
up 1:22   系统运行时间，格式为时:分
1 user    当前登录用户数
load average: 0.03, 0.04, 0.05 系统负载，即任务队列的平均长度。三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。

第二、三行为进程和CPU的信息。当有多个CPU时，这些内容可能会超过两行，意思解析如下：
total 进程总数
running 正在运行的进程数
sleeping 睡眠的进程数
stopped 停止的进程数
zombie 僵尸进程数
Cpu(s): 
0.3% us 用户空间占用CPU百分比
1.0% sy 内核空间占用CPU百分比
0.0% ni 用户进程空间内改变过优先级的进程占用CPU百分比
98.7% id 空闲CPU百分比
0.0% wa 等待输入输出的CPU时间百分比
0.0%hi：硬件CPU中断占用百分比
0.0%si：软中断占用百分比
0.0%st：虚拟机占用百分比

最后两行为内存信息。内容如下
Mem
191272k total   物理内存总量
173656k used    使用的物理内存总量
17616k free     空闲内存总量
22052k buffers    用作内核缓存的内存量
Swap
192772k total   交换区总量
0k used    使用的交换区总量
192772k free    空闲交换区总量
123988k cached 

缓冲的交换区总量,内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小,相应的内存再次被换出时可不必再对交换区写入

### 服务器根目录已使用完

关于根目录空间使用完，那么就需要清理空间，当然首先需要找着占用这些空间的文件，并将这些文件删除
首先，查看空间的大概使用情况命令
`df -Th`
输出如下

    xm6f@admin-pc:~$ df -Th
    文件系统       类型      容量  已用  可用 已用% 挂载点
    udev           devtmpfs  3.9G     0  3.9G    0% /dev
    tmpfs          tmpfs     794M  9.2M  785M    2% /run
    /dev/sda2      ext4      909G  289G  574G   34% /
    tmpfs          tmpfs     3.9G     0  3.9G    0% /dev/shm
    tmpfs          tmpfs     5.0M     0  5.0M    0% /run/lock
    tmpfs          tmpfs     3.9G     0  3.9G    0% /sys/fs/cgroup
    /dev/sda1      vfat      511M  3.5M  508M    1% /boot/efi
    cgmfs          tmpfs     100K     0  100K    0% /run/cgmanager/fs
    tmpfs          tmpfs     794M     0  794M    0% /run/user/1000

进入根目录，使用
`du -sh *`
来列出根目录下所有文件夹的使用大小情况，并再进入使用量大的文件夹再次使用次命令，以此类推找着占用空间大的文件，将其删除，一般都是tomcat的日志文件占用量大，及项目的日志文件
在删除文件后，将服务器重启，以为在删除日志文件的时候，tomcat还在运行，项目也还在跑。在这样的情况下直接删除日志文件，系统还会默认读取这个被删除的文件，其占用的空间还在继续被进程占用着，在Linux或者Unix系统中，通过rm或者文件管理器删除文件将会从文件系统的目录结构上解除链接(unlink).然而如果文件是被打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。而我删除的是log文件，删除的时候文件应该正在被使用，所以光靠删除日志文件是不够的。不过也可以将tomcat关闭，让项目不再运行，再将日志文件删除，这时就体现了nginx集群服务的重要性了，同样服务器重启也是如此
可以通过一个命令来获得一个已经被删除但是仍然被应用程序占用的文件列表，其命令为
`lsof | grep deleted`
如下部分输出

    xm6f@admin-pc:~$ lsof | grep deleted
    lsof: WARNING: can't stat() tracefs file system /sys/kernel/debug/tracing
          Output information may be incomplete.
    java      31197            xm6f    1w      REG          8,2 671033021104   41025544 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.out (deleted)
    java      31197            xm6f    2w      REG          8,2 671033021104   41025544 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.out (deleted)
    java      31197            xm6f    7w      REG          8,2  92820543856   41025989 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.2017-08-16.log (deleted)
    java      31197            xm6f    8w      REG          8,2         2937   41026959 /home/xm6f/dev/apache-tomcat-7.0.78/logs/localhost.2017-08-14.log (deleted)
    java      31197            xm6f    9w      REG          8,2         2550   41026958 /home/xm6f/dev/apache-tomcat-7.0.78/logs/manager.2017-08-14.log (deleted)
    java      31197            xm6f   10w      REG          8,2            0   41025548 /home/xm6f/dev/apache-tomcat-7.0.78/logs/host-manager.2017-08-11.log (deleted)
    java      31197  1188      xm6f    1w      REG          8,2 671029150585   41025544 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.out (deleted)
    java      31197  1188      xm6f    2w      REG          8,2 671029150585   41025544 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.out (deleted)
    java      31197  1188      xm6f    7w      REG          8,2  92816747272   41025989 /home/xm6f/dev/apache-tomcat-7.0.78/logs/catalina.2017-08-16.log (deleted)

通过上面可以看得出，以前删除的日志文件，虽然删除了，当时系统还在使用这个文件，系统一直在运行，所以空间一直在占用着，只要系统关闭重启，这样上面那些还在读取已经删除的日志文件的进程就会停止运行，空间自然就回来了
结束进程的方法有两种，一种方法是kill掉相应的进程，另一种是停掉使用这个文件的应用，也就是让系统自动停止运行，让os自动回收磁盘空间
我这服务器上了解到一个pech项目中有线程会无线循环下去，直接杀死进程也可以，而我值选择重启机器。
用到的命令
输出当前文件夹下所有文件夹的大小
`du -sh *`
`du -msh /*`
查看挂载点的大小与使用情况
`df -Th`
`df -i`

查找大于10M的文件
`sudo find / -type f -size +10000000c -exec du -sh {} \;`
或者
`sudo find / -type f -size +10M -exec du -sh {} \;`
查看当前目录下一级子文件和子目录占用的磁盘容量
`sudo du -lh --max-depth=1`
占用最多空间的前10个文件或目录
`du -cks * |sort -rn |head -n 10`

### tomcat无日志文件输出

发现160的三台tomcat没有日志文件输出，即catalina.out这个文件为空，用ls -l命令查看了下权限如下

    [xm6f@localhost logs]$ ls -l
    总用量 5844760
    -rw-r--r--. 1 root root       6653 7月  26 14:38 catalina.2017-07-17.log
    -rw-r--r--. 1 root root       9547 7月  26 14:38 catalina.2017-07-26.log
    -rw-r--r--. 1 root root      63288 7月  28 16:28 catalina.2017-07-28.log
    -rw-r--r--. 1 root root      34028 7月  31 08:30 catalina.2017-07-31.log
    -rw-r--r--. 1 root root      61923 8月   1 19:25 catalina.2017-08-01.log
    -rw-r--r--. 1 root root      36851 8月   2 16:39 catalina.2017-08-02.log
    -rw-r--r--. 1 root root     115277 8月   3 14:06 catalina.2017-08-03.log
    -rw-r--r--. 1 root root      57702 8月   4 14:22 catalina.2017-08-04.log
    -rw-r--r--. 1 root root     111603 8月   7 17:37 catalina.2017-08-07.log
    -rw-r--r--. 1 root root     152436 8月   8 17:54 catalina.2017-08-08.log
    -rw-r--r--. 1 root root      72429 8月   9 18:34 catalina.2017-08-09.log
    -rw-r--r--. 1 root root      55472 8月  10 17:57 catalina.2017-08-10.log
    -rw-r--r--. 1 root root      28248 8月  11 17:21 catalina.2017-08-11.log
    -rw-r--r--. 1 root root      31514 8月  14 17:56 catalina.2017-08-14.log
    -rw-r--r--. 1 root root      31732 8月  15 15:19 catalina.2017-08-15.log
    -rw-r--r--. 1 root root 3815423315 8月  15 15:25 catalina.out
    -rw-r--r--. 1 root root          0 7月  26 14:38 host-manager.2017-07-17.log
    -rw-r--r--. 1 root root          0 7月  26 14:38 host-manager.2017-07-26.log
    -rw-r--r--. 1 root root          0 7月  28 11:49 host-manager.2017-07-28.log

都是属于root用户，估计是没权限，
所以先关闭tomcat.......这里我也不知道为什么它能启动，毕竟这服务器不是我一个人管，别人都会动
使用sudo 命令启动，发现输出个问题，如下

    [xm6f@localhost bin]$ sudo ./catalina.sh stop
    [sudo] password for xm6f: 
    Neither the JAVA_HOME nor the JRE_HOME environment variable is defined
    At least one of these environment variable is needed to run this program

貌似是找不着jdk和jre，那就给它指定下路径，修改catalina.sh这个文件，在文件开头添加如下

    export JAVA_HOME=/home/xm6f/dev/jdk1.7.0_80
    export JRE_HOME=/home/xm6f/dev/jdk1.7.0_80/jre

之后保存，使用sudo命令运行，后面就有了日志
补充一点，防止溢出的优化项的代码添加在catalina.sh这个文件内容中间的

    #----- Execute The Requested Command -----------------------------------------
    JAVA_OPTS="$JAVA_OPTS -server -Xms2048m -Xmx2048m -XX:PermSize=1024M -XX:MaxPermSize=1024M"
    # Bugzilla 37848: only output this if we have a TTY

这个位置
代码就是上面的JAVA_OPTS

