---
layout: post
title: "tomcat配置session共享"
date: 2017-08-12 23:46:06
categories: linux
tags: [linux, centos, redis, tomcat]
---

现在在公司里需要管理四台服务器，在这四台服务器中都各自装有三台tomcat,其他redis和nginx在其中一些有装，项目部署到不同ip的tomcat，需要实现session共享，也就是需要实现集群。之前配置的是nginx集群，并未涉及到session事物的共享，所以现在要进行配置，实现session集群。

<!-- more -->

首先需要准备三个jar包，放到tomcat安装目录的lib文件夹下，每个tomcat都要存放
这里我使用到的jar包名如下

    tomcat-redis-session-manager-tomcat7.jar 
    jedis-2.5.2.jar 
    commons-pool2-2.2.jar

将其全部复制到tomcat安装路径下的lib文件夹中，然后编辑安装路径下的conf文件夹中的context.xml文件，增加内容如下

    <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
    <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
           host="localhost"
           port="6379"
           database="0"
           maxInactiveInterval="60" />

其中host是配置redis所在机器的ip地址，如果是本台机器，则配置localhost即可，其测试用到的获取sessions的代码如下
`<%= request.getSession().getId() %>`
将其插入到html页面中，我一般就将其设置为html页面的title标签值，这样简单又有用

### 第一次测试

这里首先测试了两种情况，
第一种：2台tomcat和redis及nginx在同一台机器上
结果: 获取的session id一样，这样实现了tomcat的session共享

第二种: 两台不同的机器，机器105上只有1台测试tomcat，机器149上有redis和1台测试tomcat，两台机器上都装的nginx
结果: 获取的session id不一样，

这我就一脸懵逼了，差看日志也没啥输出
猜测是不能跨ip来进行session共享，不过有感觉有点浮夸，抱着试一试的心态进行了下一步测试

### 第二次测试

出现第三种情况，就是将前两种情况进行整合
就是两台机器，105和149，
机器105中有两台测试的tomcat,即tomcat8081和tomcat8082，有个nginx,nginx并为这两台tomcat进行负载均衡
机器149中有两台测试的tomcat,即tomcat8081和tomcat8082, 有个nginx，nginx将这两台tomcat进行了负载均衡，有个redis
将机器105中的tomcat8081进行负载均衡添加到149中的nginx里面，
为四台tomcat都配置redis的session共享，并都指向149中的redis

结果: 机器105中的tomcat8081和机器149中的两台tomcat使用的session id是一样的，单单只有机器105中的tomcat8082使用的是另一个session id

出现这样的结果让我很吃惊，跨ip是肯定可以跨的，不过为什么又会有一台获取不到session id，其区别也只是tomcat8082和其他tomcat不在同一个nginx负载均衡集群中，所以猜想难道redis的session会和nginx的负载均衡结合起来？所以还是进行下一步的测试

### 第三次测试

这次是一台机器，即149，有三台tomcat，分别为tomcat8080,tomcat8081,tomcat8082.还有个nginx和redis
将tomcat8080和tomcat8081使用nginx来达到负载均衡，tomcat8082不添加进去
并都配置好session共享

结果: 三台tomcat使用的session id都相同
这可得怎么搞......

总结上面的三次测试，感觉单靠redis并不能达到跨ip的session共享的结果，只能达到本台机器下的tomcat进行seession共享，而如果使用nginx来配合redis，解决跨ip问题，那么就可以实现跨ip的session共享，这看感觉有点不符合逻辑，因为redis和nginx并不是同一家公司开发的啊，为什么还需要进行互补...看网上的一些教程，貌似都是单靠redis就可以实现session共享，但我也不能确定，总之就是，现在如果我要达到跨ip的session共享，那么就需要nginx的配合，就可以实现

测试过程的终端输出就不贴了。
