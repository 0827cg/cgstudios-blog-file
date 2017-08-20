---
layout: post
title: "为静态博客添加gitalk评论插件"
date: 2017-08-20 13:18:30
categories: gitalk
tags: gitalk
---

多说跟网易云跟帖都倒闭这么久了，一直在找适合自己的评论系统，之前接触到一个gitment这个开源的基于github的issue评论，测试用了下还不错，不过就是在页面上显示不那么协调，也就没用了。前几天凑巧找着个gitalk这个插件，和gitment类似，一样是基于github的issue来实现评论的。同样的功能，给我的感觉就是gitalk的ui真是不错，于是就打算使用这个了

<!-- more -->

这里是链接
* [项目地址][]
* [演示地址][]

项目地址中有中文说明，详细的使用方式参考项目地址中的就可以了，这里我只是对我这次的使用进行一次简单的总结，做个记录

首先申请一个Github Application，到[这个页面]去，具体的填写方式如下图中的

![github Application](/images/gitalk/github-application.jpg)

其中Homepage URL和Authorization callback URL这两个都填写自己博客站点的url，如我的博客现在浏览器地址中显示的根地址是`https://cgspace.date`那我就填这个，这两个填对了就好说了
粘贴代码到自己的post页面中了
我是直接使用url链接引入的方式，来使用gitalk的，如下我的引入代码格式

    <div id="gitalk-container"></div>
    <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
    <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
    <script>
    var gitalk = new Gitalk({
      clientID: '你申请得到的client id',
      clientSecret: '你申请得到的client secret',
      repo: '存放评论的仓库',
      owner: 'github登录id名',
      admin: ['拥有对仓库进行操作的成员id名'],
    })
    gitalk.render('gitalk-container')
    </script>

上面代码中列出的参数都是必须的，必须为他们进行赋值。当然也有一些其他的参数，具体可以参考gitalk项目地址里面的中文说明。上面`admin`参数的值是一个集合来存放，里面存放着这个项目仓库的所有开发人员的github的id名，这是对应那些团体开发的项目来说的，而对于我们一些个人项目，就比如博客，开发维护人员就只有我们自己，所以`admin`这个参数对应的集合只有一个元素，与`owner`的值一样
这样引入代码完成之后，提交到github，就需要自己对每一篇文章进行初始化，开放与文章对应的issue，之后就可以被其他github用户进行评论了

对于上面的样式引入链接，自己也可以把两文件下载下来然后存放到自己的仓库中进行引入

[项目地址]:https://github.com/gitalk/gitalk
[演示地址]:https://gitalk.github.io/
[这个页面]:https://github.com/settings/applications/new
