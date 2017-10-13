---
layout: post
title: "Hexo博客电脑迁移"
date: 2017-08-30 16:25:39
updated: 2017-10-12 17:27:26
categories: hexo
tags: hexo
---

公司配了电脑，所以就不用自己带电脑去上班，而出租屋里自己也没wifi，也就导致不好更博。原先自己是将笔记整理好，用simpleNote来进行同步更新，等到了周末就带自己笔记本来公司总结自己写的笔记，并将其更新到博客中。总觉得麻烦，于是就在公司配的笔记本上搭建好环境来更博....

<!-- more -->

自己在之前就已经把博客主题和博客source文件都上传到github中，所以换个电脑来更博也不麻烦

### 配置好hexo环境
这个windows上直接下载node.js和git,安装时一路enter就好了。安装完成之后在cmd窗口下运行命令
`npm install -g hexo-cli`
来安装hexo

### 配置git
安装好git后，需要填写用户名和邮箱作为一个标识，运行命令
`git config --global user.name "cgstudios"`
`git config --global user.email "1732821152@qq.com"`
参数--global表示全局设置，使用`git config --help`来查看其他参数

### 创建本地站点
使用hexo init命令创建一个本地博客站点

### 为hexo指定git
在创建的本地站点目录下运行命令
`npm install hexo-deployer-git -save`
这样才能通过hexo来操作git
另外还需安装插件，实现local search搜索，运行命令
`npm install hexo-generator-searchdb --save`

### git连接github
git上传文件到github会进行加密，且使用rsa加密，这需要让github知道公有密钥，git bash下使用命令
`ssh-keygen -t rsa -C "1732821152@qq.com"`
之后会让你选择确认存放密钥的文件路径，为方便一路enter下去就可以，知道密钥存放路径就行了。将公有密钥id_rsa.pub文件里的内容放到github上

### 其他配置
这项配置本人自己需要弄的
1.删除本地站点根目录中的source这个文件夹，并从github上下载cgstudios-blog-file,直接在本地站点根目录下运行命令
`git colne https://github.com/cgstudios/cgstudios-blog-file.git`
将clone得到的cgstudios-blog-file这个文件夹重命名为source,并将该文件夹里的site-config-file文件夹中的_config.yml复制到根目录覆盖根目录下的_config.yml

2.在themes文件夹下运行
`git clone https://github.com/cgstudios/myselfBlog-theme-next-myself.git`
再将clone得到的myselfBlog-theme-next-myself重命名为next-myself

3.本地站点目录下运行命令
`npm install aplayer --save`
`npm install hexo-tag-aplayer`

之后运行命令`hexo deploy`就可以查看是否连接到github仓库了