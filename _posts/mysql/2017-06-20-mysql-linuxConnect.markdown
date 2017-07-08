---
layout: post
title: "Linux上远程连接mysql数据库服务器"
date: 2017-06-20 10:43:30
categories: mysql
tags: [mysql, codeing]
---

昨天有需要从远程mysql数据库服务器中读取数据库信息,自己多于在Linux系统上做开发,而且都是使用xampp软件集成包来完成数据库等服务,并没有涉及到连接远程数据库服务器的需求,所以,就在linux系统上装了个mysql使用

这里记录的只是当时的过程

<!-- more -->

过程分类:

* 安装mysql
* 远程连接mysql服务器
* 查询操作

### 安装mysql

终端运行命令

`sudo apt-get install mysql-server`

之后选择输入y

再在终端显示的面板中输入root用户的密码

#### 安装过程如下:

	cg@cg-ThinkPad-Edge-E540 ~ $ sudo apt-get install mysql-server
	正在读取软件包列表... 完成
	正在分析软件包的依赖关系树       
	正在读取状态信息... 完成
	将会安装下列额外的软件包：
	  libdbd-mysql-perl libdbi-perl libmysqlclient18 libterm-readkey-perl
	  mysql-client-5.5 mysql-client-core-5.5 mysql-common mysql-server-5.5
	  mysql-server-core-5.5
	建议安装的软件包：
	  libmldbm-perl libnet-daemon-perl libplrpc-perl libsql-statement-perl tinyca
	  mailx
	推荐安装的软件包：
	  libhtml-template-perl
	下列【新】软件包将被安装：
	  libdbd-mysql-perl libdbi-perl libmysqlclient18 libterm-readkey-perl
	  mysql-client-5.5 mysql-client-core-5.5 mysql-common mysql-server
	  mysql-server-5.5 mysql-server-core-5.5
	升级了 0 个软件包，新安装了 10 个软件包，要卸载 0 个软件包，有 220 个软件包未被升级。
	需要下载 9,642 kB 的软件包。
	解压缩后会消耗掉 96.9 MB 的额外空间。
	您希望继续执行吗？ [Y/n] y
	获取：1 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-common all 5.5.55-0ubuntu0.14.04.1 [13.0 kB]
	获取：2 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main libmysqlclient18 amd64 5.5.55-0ubuntu0.14.04.1 [597 kB]
	获取：3 http://mirrors.hust.edu.cn/ubuntu/ trusty/main libdbi-perl amd64 1.630-1 [879 kB]
	获取：4 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main libdbd-mysql-perl amd64 4.025-1ubuntu0.1 [87.6 kB]
	获取：5 http://mirrors.hust.edu.cn/ubuntu/ trusty/main libterm-readkey-perl amd64 2.31-1 [27.4 kB]
	获取：6 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-client-core-5.5 amd64 5.5.55-0ubuntu0.14.04.1 [707 kB]
	获取：7 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-client-5.5 amd64 5.5.55-0ubuntu0.14.04.1 [1,586 kB]
	获取：8 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-server-core-5.5 amd64 5.5.55-0ubuntu0.14.04.1 [3,737 kB]
	获取：9 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-server-5.5 amd64 5.5.55-0ubuntu0.14.04.1 [1,996 kB]
	获取：10 http://mirrors.hust.edu.cn/ubuntu/ trusty-updates/main mysql-server all 5.5.55-0ubuntu0.14.04.1 [11.3 kB]
	下载 9,642 kB，耗时 39秒 (247 kB/s)                                            
	正在预设定软件包 ...
	Selecting previously unselected package mysql-common.
	(正在读取数据库 ... 系统当前共安装有 177786 个文件和目录。)
	Preparing to unpack .../mysql-common_5.5.55-0ubuntu0.14.04.1_all.deb ...
	Unpacking mysql-common (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package libmysqlclient18:amd64.
	Preparing to unpack .../libmysqlclient18_5.5.55-0ubuntu0.14.04.1_amd64.deb ...
	Unpacking libmysqlclient18:amd64 (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package libdbi-perl.
	Preparing to unpack .../libdbi-perl_1.630-1_amd64.deb ...
	Unpacking libdbi-perl (1.630-1) ...
	Selecting previously unselected package libdbd-mysql-perl.
	Preparing to unpack .../libdbd-mysql-perl_4.025-1ubuntu0.1_amd64.deb ...
	Unpacking libdbd-mysql-perl (4.025-1ubuntu0.1) ...
	Selecting previously unselected package libterm-readkey-perl.
	Preparing to unpack .../libterm-readkey-perl_2.31-1_amd64.deb ...
	Unpacking libterm-readkey-perl (2.31-1) ...
	Selecting previously unselected package mysql-client-core-5.5.
	Preparing to unpack .../mysql-client-core-5.5_5.5.55-0ubuntu0.14.04.1_amd64.deb ...
	Unpacking mysql-client-core-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package mysql-client-5.5.
	Preparing to unpack .../mysql-client-5.5_5.5.55-0ubuntu0.14.04.1_amd64.deb ...
	Unpacking mysql-client-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package mysql-server-core-5.5.
	Preparing to unpack .../mysql-server-core-5.5_5.5.55-0ubuntu0.14.04.1_amd64.deb ...
	Unpacking mysql-server-core-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
	正在设置 mysql-common (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package mysql-server-5.5.
	(正在读取数据库 ... 系统当前共安装有 178142 个文件和目录。)
	Preparing to unpack .../mysql-server-5.5_5.5.55-0ubuntu0.14.04.1_amd64.deb ...
	Unpacking mysql-server-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	Selecting previously unselected package mysql-server.
	Preparing to unpack .../mysql-server_5.5.55-0ubuntu0.14.04.1_all.deb ...
	Unpacking mysql-server (5.5.55-0ubuntu0.14.04.1) ...
	Processing triggers for ureadahead (0.100.0-16) ...
	ureadahead will be reprofiled on next reboot
	Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
	正在设置 libmysqlclient18:amd64 (5.5.55-0ubuntu0.14.04.1) ...
	正在设置 libdbi-perl (1.630-1) ...
	正在设置 libdbd-mysql-perl (4.025-1ubuntu0.1) ...
	正在设置 libterm-readkey-perl (2.31-1) ...
	正在设置 mysql-client-core-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	正在设置 mysql-client-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	正在设置 mysql-server-core-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	正在设置 mysql-server-5.5 (5.5.55-0ubuntu0.14.04.1) ...
	170619 13:44:23 [Warning] Using unique option prefix key_buffer instead of key_buffer_size is deprecated and will be removed in a future release. Please use the full name instead.
	170619 13:44:23 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
	170619 13:44:23 [Note] /usr/sbin/mysqld (mysqld 5.5.55-0ubuntu0.14.04.1) starting as process 17651 ...
	mysql start/running, process 17782
	Processing triggers for ureadahead (0.100.0-16) ...
	正在设置 mysql-server (5.5.55-0ubuntu0.14.04.1) ...
	Processing triggers for libc-bin (2.19-0ubuntu6.9) ...
	/sbin/ldconfig.real: /usr/lib/nvidia-375/libEGL.so.1 is not a symbolic link

	/sbin/ldconfig.real: /usr/lib32/nvidia-375/libEGL.so.1 is not a symbolic link

	cg@cg-ThinkPad-Edge-E540 ~ $ 


### 远程连接mysql服务器

先测试下连接本地数据库

	cg@cg-ThinkPad-Edge-E540 ~ $ mysql
	ERROR 1045 (28000): Access denied for user 'cg'@'localhost' (using password: NO)
	cg@cg-ThinkPad-Edge-E540 ~ $ mysql -u root -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 43
	Server version: 5.5.55-0ubuntu0.14.04.1 (Ubuntu)

	Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	mysql> show databases
	    -> show databases;
	ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'show databases' at line 2
	mysql> show databases
	    -> 
	  -> ^Z
	[1]+  Stopped                 mysql -u root -p

正常即可连接远程数据库

	cg@cg-ThinkPad-Edge-E540 ~ $ mysql -h 182.254.213.111 -P 3306 -u root -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 4618
	Server version: 5.7.15 MySQL Community Server (GPL)

	Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

###  查询操作
连接完成之后,可以使用命令

`show databases;`

来显示数据库的库信息

使用命令

`show tables;`

来显示当前库中表的信息,当然前提需要指定库

指定库使用命令

`use xxxx;`

这跟mysql语句都是一样的

当时过程如下:

	mysql> show databases;
		+--------------------------+
		| Database                 |
		+--------------------------+
		| information_schema       |
		| cruise                   |
		| jinbang                  |
		| jinbangH5                |
		| jinbang_dev_shiqingju    |
		| jinbang_dev_shiqingkun   |
		| jinbang_dev_zhouminxiang |
		| jinbang_portal           |
		| jinbang_portal1          |
		| mysql                    |
		| performance_schema       |
		| sys                      |
		| test1                    |
		| test2                    |
		| test3                    |
		| test4                    |
		| test5                    |
		| wordpress                |
		| yihaiyou_test            |
		+--------------------------+
		19 rows in set (0.03 sec)

	mysql> help

	For information about MySQL products and services, visit:
	   http://www.mysql.com/
	For developer information, including the MySQL Reference Manual, visit:
	   http://dev.mysql.com/
	To buy MySQL Enterprise support, training, or other products, visit:
	   https://shop.mysql.com/

	List of all MySQL commands:
	Note that all text commands must be first on line and end with ';'
	?         (\?) Synonym for `help'.
	clear     (\c) Clear the current input statement.
	connect   (\r) Reconnect to the server. Optional arguments are db and host.
	delimiter (\d) Set statement delimiter.
	edit      (\e) Edit command with $EDITOR.
	ego       (\G) Send command to mysql server, display result vertically.
	exit      (\q) Exit mysql. Same as quit.
	go        (\g) Send command to mysql server.
	help      (\h) Display this help.
	nopager   (\n) Disable pager, print to stdout.
	notee     (\t) Don't write into outfile.
	pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
	print     (\p) Print current command.
	prompt    (\R) Change your mysql prompt.
	quit      (\q) Quit mysql.
	rehash    (\#) Rebuild completion hash.
	source    (\.) Execute an SQL script file. Takes a file name as an argument.
	status    (\s) Get status information from the server.
	system    (\!) Execute a system shell command.
	tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
	use       (\u) Use another database. Takes database name as argument.
	charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
	warnings  (\W) Show warnings after every statement.
	nowarning (\w) Don't show warnings after every statement.

	For server side help, type 'help contents'

	mysql> use yihaiyou_test;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	mysql> use yihaiyou_test;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	mysql> show tables;
	+---------------------------------+
	| Tables_in_yihaiyou_test         |
	+---------------------------------+
	| acc                             |
	| account_log                     |
	| activities                      |
	| ads                             |
	| advice                          |
	| api_monitor                     |
	| article_category                |
	| bak_sys_menu                    |
	| bak_sys_role                    |
	| bak_sys_role_menu               |
	| bak_sys_user_role               |
	| bankcard                        |
	| board_location                  |
	| category                        |
	| category_type                   |
	| cmbpaylog                       |
	| coderepository                  |
	| comment                         |
	| comment_photo                   |
	| comment_score                   |
	| comment_score_type              |
	| commission                      |
	| contract                        |
	| contract_appendices             |
	| credit_card                     |
	| cruise_ship                     |
	| cruise_ship_date                |
	| cruise_ship_deck                |
	| cruise_ship_deck_facility       |
	| cruise_ship_extend              |
	| cruise_ship_plan                |
	| cruise_ship_project             |
	| cruise_ship_project_classify    |
	| cruise_ship_project_image       |
	| cruise_ship_room                |
	| cruise_ship_room_date           |
	| cruise_ship_visa                |
	| ctrip_city                      |
	| ctrip_region                    |
	| ctrip_scenics                   |
	| ctrip_user                      |
	| custom_require                  |
	| custom_require_destination      |
	| cw_productitem                  |
	| data_tasks                      |
	| delicacy                        |
	| delicacy_extend                 |
	| delicacy_restaurant             |
	| destination                     |
	| elong_geo_cn                    |
	| feed_back                       |
	| ferrymember                     |
	| ferryorder                      |
	| ferryorderitem                  |
	| flight_city                     |
	| globalmoneyrecord               |
	| hand_draw_map                   |
	| hand_draw_scenic                |
	| hotel                           |
	| hotel_amenities                 |
	| hotel_area                      |
	| hotel_brand                     |
	| hotel_city                      |
	| hotel_city_brand                |
	| hotel_city_service              |
	| hotel_elong_api_log             |
	| hotel_elong_static_info         |
	| hotel_extend                    |
	| hotel_price                     |
	| hotel_price_calendar            |
	| hotel_region                    |
	| hotel_room                      |
	| hotel_service_tmp               |
	| hotel_to_update                 |
	| impression                      |
	| impression_gallery              |
	| invoice                         |
	| jszx_order                      |
	| jszx_order_detail               |
	| label                           |
	| label_item                      |
	| line                            |
	| line_contact                    |
	| line_departure                  |
	| line_departure_info             |
	| line_insurance                  |
	| lineacrosscitys                 |
	| linecategory                    |
	| linecategorydetail              |
	| linecost                        |
	| linedayplaninfo                 |
	| linedays                        |
	| linedaysplan                    |
	| linedaysproductprice            |
	| lineexplain                     |
	| lineimages                      |
	| lineplaytitle                   |
	| linestatistic                   |
	| linetypeprice                   |
	| linetypeprice_agent             |
	| linetypepricedate               |
	| lxb_advice                      |
	| lxb_coupon                      |
	| lxb_friend_link                 |
	| lxb_insurance                   |
	| lxb_labels                      |
	| lxb_objectlabels                |
	| lxb_sales_images                |
	| lxb_user_coupon                 |
	| member                          |
	| msg_template                    |
	| multi_date                      |
	| nctrip_access_token             |
	| nctrip_addinfo_detail           |
	| nctrip_api_log                  |
	| nctrip_display_tag              |
	| nctrip_display_tag_group        |
	| nctrip_order_contact_info       |
	| nctrip_order_form_info          |
	| nctrip_order_form_resource_info |
	| nctrip_order_passenger_info     |
	| nctrip_product_addinfo          |
	| nctrip_resource_addinfo         |
	| nctrip_resource_price_calendar  |
	| nctrip_scenic_spot_city_info    |
	| nctrip_scenic_spot_id           |
	| nctrip_scenic_spot_info         |
	| nctrip_scenic_spot_poi_info     |
	| nctrip_scenic_spot_product      |
	| nctrip_scenic_spot_resource     |
	| order_refund                    |
	| orderalias                      |
	| other_favorite                  |
	| other_message                   |
	| other_visit_history             |
	| outer_collect_info              |
	| outer_mascot_bename             |
	| outer_participator_answer       |
	| outer_question                  |
	| outer_question_candidate        |
	| paylog                          |
	| pinglun                         |
	| pinglunreply                    |
	| plan                            |
	| plan_day                        |
	| plan_statistic                  |
	| plan_trip                       |
	| plan_urban_traffic_rel          |
	| playtitle                       |
	| product                         |
	| product_activity                |
	| product_validate_channel        |
	| product_validate_code           |
	| productimage                    |
	| productvalidatecode             |
	| productvalidaterecord           |
	| promotion                       |
	| qrtz_blob_triggers              |
	| qrtz_calendars                  |
	| qrtz_cron_triggers              |
	| qrtz_fired_triggers             |
	| qrtz_job_details                |
	| qrtz_locks                      |
	| qrtz_paused_trigger_grps        |
	| qrtz_scheduler_state            |
	| qrtz_simple_triggers            |
	| qrtz_simprop_triggers           |
	| qrtz_triggers                   |
	| quantity_sales                  |
	| quantity_sales_detail           |
	| quantity_unit_num               |
	| qunar_sight                     |
	| recday_scenics_num              |
	| recommend_plan                  |
	| recommend_plan_day              |
	| recommend_plan_photo            |
	| recommend_plan_tag              |
	| recommend_plan_trip             |
	| recpday_cityids                 |
	| recplan_days                    |
	| recplan_scenics                 |
	| refund_log                      |
	| region                          |
	| reply                           |
	| restaurang                      |
	| restaurant                      |
	| restaurant_extend               |
	| restaurant_geoinfo              |
	| scenic                          |
	| scenic_area                     |
	| scenic_extend                   |
	| scenic_extend_tmp               |
	| scenic_gallery                  |
	| scenic_geoinfo                  |
	| scenic_geoinfo_tmp              |
	| scenic_relation                 |
	| scenic_statistics               |
	| scenic_theme                    |
	| scenic_theme_relation           |
	| scenic_tmp                      |
	| sendingmsg                      |
	| sendingmsg_his                  |
	| serialscode                     |
	| shenzhouaccesstoken             |
	| shenzhouorder                   |
	| supplier                        |
	| suppliercity                    |
	| supplierconfig                  |
	| supplierservice                 |
	| sys_action_log                  |
	| sys_menu                        |
	| sys_resource                    |
	| sys_resource_map                |
	| sys_role                        |
	| sys_role_menu                   |
	| sys_role_resource               |
	| sys_site                        |
	| sys_unit                        |
	| sys_unit_detail                 |
	| sys_unit_image                  |
	| sys_unit_qualification          |
	| sys_user                        |
	| sys_user_role                   |
	| t_base_pinyin                   |
	| tb_area                         |
	| tb_area_extend                  |
	| tb_area_optimize_support_city   |
	| tb_area_relation                |
	| tb_dis                          |
	| tb_ticket_validate_info         |
	| tb_transportation               |
	| tciket_price_type_extend        |
	| testperson                      |
	| third_party_user                |
	| ticket                          |
	| ticket_receiver                 |
	| ticketdateprice                 |
	| ticketexplain                   |
	| ticketprice                     |
	| ticketprice_agent               |
	| tmp_time_desc                   |
	| torder                          |
	| tordercontact                   |
	| torderdetail                    |
	| torderdetailflattened           |
	| torderinsurance                 |
	| torderinvoice                   |
	| torderlog                       |
	| tordertourist                   |
	| tourist                         |
	| traffic                         |
	| traffic_price                   |
	| traffic_price_calendar          |
	| user                            |
	| user_copy                       |
	| user_exinfo                     |
	| user_relation                   |
	| user_share_record               |
	| userlevel                       |
	| wx_account                      |
	| wx_account_menu                 |
	| wx_data_img                     |
	| wx_data_img_text                |
	| wx_data_item                    |
	| wx_data_text                    |
	| wx_follower                     |
	| wx_location_log                 |
	| wx_qrcode                       |
	| wx_receive_msg_log              |
	| wx_reply_keyword                |
	| wx_reply_rule                   |
	| wx_resource                     |
	| wx_support_account              |
	| wx_visit_log                    |
	| yihaiyou                        |
	| zhouandcountry                  |
	| zmy_ticket                      |
	+---------------------------------+
	277 rows in set (0.06 sec)

	mysql> select * from sys_role;
	+----+-------------------------+----------+--------+------+--------------------------------------------------------------------+--------+-------------+
	| id | name                    | del_flag | status | seq  | remark                                                             | siteid | displayName |
	+----+-------------------------+----------+--------+------+--------------------------------------------------------------------+--------+-------------+
	| -2 | 系统管理员              |        0 |      0 |    0 | 内置超级管理员                                                     |     -1 | NULL        |
	| -1 | 站点管理员              |        0 |      0 |    0 | 站点管理员,拥有所有权限                                            |     -1 | NULL        |
	|  1 | 景点管理员              |        0 |      0 |    0 |                                                                    |     -1 | NULL        |
	|  3 | 公司管理员              |        0 |      0 |    0 | 公司通过后授予该角色                                               |     -1 | NULL        |
	|  4 | 酒店民宿系统            |        0 |      0 |    0 | 商户系统角色                                                       |     -1 | NULL        |
	|  5 | 海上休闲系统            |        0 |      0 |    0 | 商户系统角色                                                       |     -1 | NULL        |
	|  6 | 邮轮系统                |        0 |      0 |    0 | 商户系统角色                                                       |     -1 | NULL        |
	|  7 | 景点门票系统            |        0 |      0 |    0 | 商户系统角色                                                       |     -1 | NULL        |
	| 10 | 临时站点管理员          |        0 |      0 |    0 | 安全扫描临时使用角色2017.1.23                                      |     -1 | NULL        |
	| 11 | 公司管理员(默认)        |        0 |      0 |    0 | 添加公司或公司入驻时，授予公司管理员默认角色                       |     -1 | NULL        |
	+----+-------------------------+----------+--------+------+--------------------------------------------------------------------+--------+-------------+
	10 rows in set (0.02 sec)

	mysql> select * from wx_visit_log;
	Empty set (0.04 sec)

	mysql> 


数据库服务器连接完成之后,后面的操作命令则跟mysql语句都一样

***

#### update-2017-06-28

查看表中的字段的备注信息，对字段名的含义了解

命令：

`show full columns from xxx;`

查看表的注释：

`show create table xxx;`

![mysql-show](/images/mysql/mysql-show-table.png)






