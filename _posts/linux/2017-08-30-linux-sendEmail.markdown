---
layout: post
title: "使用shell发送qq邮件"
date: 2017-08-30 17:18:40
categories: linux
tags: [linux, email, mail]
---

前些天折腾了下在linux系统上发送qq邮件，发现使用shell命令来实现这个很简单，于是就弄了。弄这个的主要目的是为了将脚本监控或者操作系统后得到的结果以邮件的方式反馈给我，这样我也就能及时知道自己写的脚本执行情况怎么样。

<!-- more -->

### 安装mail
首先，查看系统中是否又mail和mailx，运行命令
`mail -V`
或者
`mailx -V`
有输出即表示系统中已经存在,如下

	[xm6f@localhost cut_nginx_log]$ mail -V
	12.5 7/5/10
	[xm6f@localhost cut_nginx_log]$ mailx -V
	12.5 7/5/10

centos一般自带了这个命令，如果别的系统如ubuntu，要没有则运行命令
`sudo apt-get install heirloom-mailx`
来安装

### mail配置
之后就是需要对mail进行配置，修改其配置文件，配置好用来发送邮件的邮件地址和密码，密码都是stmp授权码
ubuntu下的配置文件为/etc/s-nail.rc
centos下的配置文件为/etc/mail.rc
在该文件末尾添加如下代码

	set from=1732821152@qq.com(cg) 				#设置发送邮件的邮箱和发邮件名称为cg
	set smtp=smtp.qq.com						#设置smtp服务器地址
	set smtp-auth-user=1732821152@qq.com		#设置邮箱账户
	set smtp-auth-password=xxx					#设置密码，为授权码
	set smtp-auth=login							#SSL验证信息
	set smtp-use-starttls
	set ssl-verify=ignore
	set nss-config-dir=/etc/pki/nssdb/			#设置证书存放路径

将其保存，之后就可以使用mail命令来发送邮件了，发送方就是上面填写的邮件地址

### 发送测试
例如
#### 读取文件内容发送
` mail -v -s "定时任务" 1542723438@qq.com < mail_content.txt`
上述命令表示，将名为mail_content.txt的文件中的内容作为邮件内容发送到`1542723438@qq.com`这个邮箱地址,
`-v`是显示输出日志，`-s`表示邮件主题名称,详情使用
`man mail`
或者
`mail --help`
来查看如何使用

#### 管道发送
`echo "hey,how are you " | mail -v -s "test" 1542723438@qq.com`
使用管道命令，将`echo`输出的内容作为邮件内容发送出去

#### 发送附件
`mail -s "测试" -a email_err.txt 1542723438@qq.com < email_content.txt`
读取email_content.txt这个文件的内容作为邮件内容发送，并将email_err.txt这个文件作为附件一起发送到邮箱

### 定时发送
需要修改这个文件/etc/crontab，这相当于一个定时器，例如我在这个文件末尾添加如下代码
`11 23 * * * root /bin/bash /usr/scripts/automatic/automatic_util.sh`
即表示在每天的23点11分以root身份，使用`/bin/bash`这个解释器来运行automatic_util.sh这个shell脚本


### 管道发送实例输出

	[xm6f@localhost cut_nginx_log]$ echo "hey,how are you " | mail -v -s "test" 1542723438@qq.com
	Resolving host smtp.qq.com . . . done.
	Connecting to 14.17.57.241:smtp . . . connected.
	220 smtp.qq.com Esmtp QQ Mail Server
	>>> EHLO localhost
	250-smtp.qq.com
	250-PIPELINING
	250-SIZE 73400320
	250-STARTTLS
	250-AUTH LOGIN PLAIN
	250-AUTH=LOGIN
	250-MAILCOMPRESS
	250 8BITMIME
	>>> STARTTLS
	220 Ready to start TLS
	Error in certificate: Peer's certificate issuer is not recognized.
	Comparing DNS name: "upload.mail.qq.com"
	Comparing DNS name: "hwsmtp.exmail.qq.com"
	Comparing DNS name: "hwimap.exmail.qq.com"
	Comparing DNS name: "cloudmx.qq.com"
	Comparing DNS name: "imap.exmail.qq.com"
	Comparing DNS name: "hwpop.exmail.qq.com"
	Comparing DNS name: "smtp.qq.com"
	SSL parameters: cipher=AES-128, keysize=128, secretkeysize=128,
	issuer=CN=GeoTrust SSL CA - G3,O=GeoTrust Inc.,C=US
	subject=CN=pop.qq.com,OU=R&D,O=Shenzhen Tencent Computer Systems Company Limited,L=Shenzhen,ST=Guangdong,C=CN
	>>> EHLO localhost
	250-smtp.qq.com
	250-PIPELINING
	250-SIZE 73400320
	250-AUTH LOGIN PLAIN
	250-AUTH=LOGIN
	250-MAILCOMPRESS
	250 8BITMIME
	>>> AUTH LOGIN
	334 VXNlcm5hbWU6
	>>> MTczMjgyMTE1MkBxcS5jb20=
	334 UGFzc3dvcmQ6
	>>> YWlpcmptcWhwdmxlZWFnZw==
	235 Authentication successful
	>>> MAIL FROM:<1732821152@qq.com>
	250 Ok
	>>> RCPT TO:<1542723438@qq.com>
	250 Ok
	>>> DATA
	354 End data with <CR><LF>.<CR><LF>
	>>> .
	250 Ok: queued as 
	>>> QUIT
	221 Bye
	[xm6f@localhost cut_nginx_log]$

### 其它
mail发送邮件平常使用25端口，也可以使用587，465(ssl)，587和25端口一样，465端口就是加密端口。平常一些服务器运营商会禁止使用25端口，使得用上面的配置则发送不去出，因此可以使用587端口
设置使用465端口来发送邮件的mail.rc配置如下
`set smtp=smtps://smtp.qq.com:465`
设置使用587端口则如下
`set smtp=smtp.qq.com:587`