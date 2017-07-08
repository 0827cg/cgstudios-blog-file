---
layout: post
title: "git使用"
date: 2017-05-10 16:08:50
categories: git
tags: [git, codeing]
---

今天经理分配一个问题让去解决,就是git连接码云来进行项目的版本控制时,分支与主分支合并时的冲突问题

于是就打算今天拿出一天时间来弄这个问题

截止到现在,问题算是解决了,不过并不是采用我的方法.我的方法更老套,就是操作命令.而最后采用的是直接在idea上操作

<!-- more -->

折腾这么久,想想还是写在这里总结一下

### 操作概述

#### git连接gitosc
如果在本地已建立仓库,且本地也与gitosc建立ssh安全连接

`git remote add origin https://git.oschina.net/用户名/仓库名.git`

注意:本地仓库名和远程建立的仓库名要一样

#### 第一步正常后

输入`git pull origin master`,可提交并同步
git pull命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并
注意:最后可以再输入`git push origin master`提交,这在github上可以一并完成

#### 本地更新后,同样

命令:
* 1.`git status`检查工作区是否有内容未被提交
* 2.`git add .`将所有未被提交的内容添加到暂存区
* 3.`git commit -m "test again"`为本次提交设置备注
* 4.`git push origin master`将master主分支提交到远程仓库

#### 创建合并分支

命令:
* 1.`git branch xxxx`创建xxxx分支
* 2.`git checkout xxxx`切换到xxxx分支(可将上面1,2两条命令整合:`git checkout -b xxxx`)
* 3.`git merge xxxx`合并xxxx分支到当前分支
* 4.`git branch -d xxxx`删除xxxx分支

注意:
第3步合并分支时,git一般使用”Fast forward”模式.
在这种模式下,删除分支后,也会丢掉分支信息
所以未避免丢掉分支信息,就可以添加参数来禁用"Fast forward"模式
例如:
`git merge -no-ff -m "注释" xxxx`

如果使用xxxx分支提交到远程仓库,那么,提交完成之后,远程仓库同样会有两条分支选择,一条master,一条xxxx
那么,在还没合并前,master主分支上是看不到xxxx分支提交的内容的,这是合乎情理的

#### 分支冲突描述:
例如有两条分支,一条master主分支,一条xxxx分支,当前个分支上的内容都一样
此时,在master主分支上添加内容a后提交到远程仓库,xxxx分支上添加内容b后提交到远程仓库
当前都未合并,所以说双方都看不到各自提交的内容


#### 主分支和次分支的提交过程:

(主分支与次分支在同一本地仓库)当次分支更新了内容a到暂存区后,切换到主分支后依然可以查看次分支更新的内容a,但当次分支更新了内容a到暂存区后,又将内容a提交到远程仓库中,此时主分支就不能status到是否有内容未被提交,
也就不能查看到内容a了
那在此前提下继续操作
在主分支下,更新内容b到暂存区并提交到远程仓库,那么
在远程仓库中,主分支和次分支所现实的内容是不一样的,本地也是如此

#### 解决冲突的方法

* 1.`git reset --merge`:将主分支回退到merge前(首先一般都是在主分支合并次分支的)
* 2.更改需要保留的内容,使两个分支更新的内容一致,如果需要保留分支上的内容,那么则切换到主分支上,更改内容使其与分支上的内容一致
* 3.使内容一致后在将更改的分支提交到远程仓库


另外说下,
1.本地创建分支后,是用新分支提交内容到远程仓库会同步创建分支,而本地删除分支后,status检查不到更改.打算提交更改内容,以此来尝试远程删除分支,依然没用
但远程仓库里提供了删除分支
所以,git新建分支时用分支push内容到远程仓库可以同步新建分支,git删除分支时就不会同步删除分支.可以手动在远程仓库进行删除操作
2.在分支合并的时候如果出现一些问题,导致下一步进行不下去,可以输入命令将其回退到合并前

一些其他的命令:

* git branch:列出所有分支,输出的前面有*号表示是当前分支
* git log:查看所有日志
* git diff xx:查看xx文件修改了那些内容
* git remote:查看远程库的信息
* git remote –v:查看远程库的详细信息
* git reset --merge:回退到merge前
* git reset  --hard HEAD^:回退到上一个版本('^'有多少个就回到多少个版本)
* git rm xx:删除xx文件
* git pull origin next:master 取回origin主机的next分支，与本地的master分支合并,如果远程分支是与当前分支合并，则冒号后面的部分可以省略:git pull origin next


### 操作实例

#### git连接gitosc

	cg@cg-ThinkPad-Edge-E540 ~ $ cd Work/GitoscWork
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork $ ls
	test-1
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork $ cd test-1
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git init
	Initialized empty Git repository in /home/cg/Work/GitoscWork/test-1/.git/
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ pwd
	/home/cg/Work/GitoscWork/test-1
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master

	Initial commit

	Untracked files:
	  (use "git add <file>..." to include in what will be committed)

		testDemo

	nothing added to commit but untracked files present (use "git add" to track)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add testDemo
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "first create"
	[master (root-commit) a391ca3] first create
	 1 file changed, 2 insertions(+)
	 create mode 100644 testDemo
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	nothing to commit, working directory clean
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ ssh -T git@git.oschina.net
	The authenticity of host 'git.oschina.net (116.211.167.14)' can't be established.
	ECDSA key fingerprint is 27:e5:d3:f7:2a:9e:eb:6c:93:cd:1f:c1:47:a3:54:b1.
	Are you sure you want to continue connecting (yes/no)? y
	Please type 'yes' or 'no': yes
	Warning: Permanently added 'git.oschina.net,116.211.167.14' (ECDSA) to the list of known hosts.
	Connection closed by 116.211.167.14
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ ^C
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git --help
	usage: git [--version] [--help] [-C <path>] [-c name=value]
		   [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
		   [-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
		   [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
		   <command> [<args>]

	The most commonly used git commands are:
	   add        Add file contents to the index
	   bisect     Find by binary search the change that introduced a bug
	   branch     List, create, or delete branches
	   checkout   Checkout a branch or paths to the working tree
	   clone      Clone a repository into a new directory
	   commit     Record changes to the repository
	   diff       Show changes between commits, commit and working tree, etc
	   fetch      Download objects and refs from another repository
	   grep       Print lines matching a pattern
	   init       Create an empty Git repository or reinitialize an existing one
	   log        Show commit logs
	   merge      Join two or more development histories together
	   mv         Move or rename a file, a directory, or a symlink
	   pull       Fetch from and integrate with another repository or a local branch
	   push       Update remote refs along with associated objects
	   rebase     Forward-port local commits to the updated upstream head
	   reset      Reset current HEAD to the specified state
	   rm         Remove files from the working tree and from the index
	   show       Show various types of objects
	   status     Show the working tree status
	   tag        Create, list, delete or verify a tag object signed with GPG

	'git help -a' and 'git help -g' lists available subcommands and some
	concept guides. See 'git help <command>' or 'git help <concept>'
	to read about a specific subcommand or concept.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git config  -help
	usage: git config [options]

	Config file location
	    --global              use global config file
	    --system              use system config file
	    --local               use repository config file
	    -f, --file <file>     use given config file
	    --blob <blob-id>      read config from given blob object

	Action
	    --get                 get value: name [value-regex]
	    --get-all             get all values: key [value-regex]
	    --get-regexp          get values for regexp: name-regex [value-regex]
	    --get-urlmatch        get value specific for the URL: section[.var] URL
	    --replace-all         replace all matching variables: name value [value_regex]
	    --add                 add a new variable: name value
	    --unset               remove a variable: name [value-regex]
	    --unset-all           remove all matches: name [value-regex]
	    --rename-section      rename section: old-name new-name
	    --remove-section      remove a section: name
	    -l, --list            list all
	    -e, --edit            open an editor
	    --get-color <slot>    find the color configured: [default]
	    --get-colorbool <slot>
		                  find the color setting: [stdout-is-tty]

	Type
	    --bool                value is "true" or "false"
	    --int                 value is decimal number
	    --bool-or-int         value is --bool or --int
	    --path                value is a path (file or directory name)

	Other
	    -z, --null            terminate values with NUL byte
	    --includes            respect include directives on lookup

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git remote add origin https://git.oschina.net/cg0827/test-1.gitcg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git remote add origin https://git.oschina.net/cg0827/test-1.git
	fatal: remote origin already exists.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push -help
	usage: git push [<options>] [<repository> [<refspec>...]]

	    -v, --verbose         be more verbose
	    -q, --quiet           be more quiet
	    --repo <repository>   repository
	    --all                 push all refs
	    --mirror              mirror all refs
	    --delete              delete refs
	    --tags                push tags (can't be used with --all or --mirror)
	    -n, --dry-run         dry run
	    --porcelain           machine-readable output
	    -f, --force           force updates
	    --force-with-lease[=<refname>:<expect>]
		                  require old value of ref to be at this value
	    --recurse-submodules[=<check>]
		                  control recursive pushing of submodules
	    --thin                use thin pack
	    --receive-pack <receive-pack>
		                  receive pack program
	    --exec <receive-pack>
		                  receive pack program
	    -u, --set-upstream    set upstream for git pull/status
	    --progress            force progress reporting
	    --prune               prune locally removed refs
	    --no-verify           bypass pre-push hook
	    --follow-tags         push missing but relevant tags

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push -u origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	To https://git.oschina.net/cg0827/test-1.git
	 ! [rejected]        master -> master (fetch first)
	error: failed to push some refs to 'https://git.oschina.net/cg0827/test-1.git'
	hint: Updates were rejected because the remote contains work that you do
	hint: not have locally. This is usually caused by another repository pushing
	hint: to the same ref. You may want to first integrate the remote changes
	hint: (e.g., 'git pull ...') before pushing again.
	hint: See the 'Note about fast-forwards' in 'git push --help' for details.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	nothing to commit, working directory clean
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git remote add origin https://git.oschina.net/cg0827/test-1.git
	fatal: remote origin already exists.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git pull
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	warning: no common commits
	remote: Counting objects: 3, done.
	remote: Total 3 (delta 0), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.
	From https://git.oschina.net/cg0827/test-1
	 * [new branch]      master     -> origin/master
	There is no tracking information for the current branch.
	Please specify which branch you want to merge with.
	See git-pull(1) for details

	    git pull <remote> <branch>

	If you wish to set tracking information for this branch you can do so with:

	    git branch --set-upstream-to=origin/<branch> master

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git pull -h
	usage: git pull [-n | --no-stat] [--[no-]commit] [--[no-]squash] [--[no-]ff] [--[no-]rebase|--rebase=preserve] [-s strategy]... [<fetch-options>] <repo> <head>...

	Fetch one or more remote refs and integrate it/them with the current HEAD.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git-pull(1)
	bash: syntax error near unexpected token `1'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git pull origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	From https://git.oschina.net/cg0827/test-1
	 * branch            master     -> FETCH_HEAD
	Merge made by the 'recursive' strategy.
	 README.md | 1 +
	 1 file changed, 1 insertion(+)
	 create mode 100644 README.md
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push -u origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 6, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (5/5), 550 bytes | 0 bytes/s, done.
	Total 5 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   eac4c33..a8d7464  master -> master
	Branch master set up to track remote branch master from origin.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

#### 提交更新
在本地仓库创建文件后,提交更新到远程仓库

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add .
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "test second"
	[master f1f28d4] test second
	 1 file changed, 1 insertion(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 5, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 317 bytes | 0 bytes/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   a8d7464..f1f28d4  master -> master
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

#### 创建合并分支
创建一个test2分支

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	* master
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test2
	Switched to branch 'test2'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test2
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	M	testDemo
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test2
	M	testDemo
	Switched to branch 'test2'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test2
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add .
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "branch test2 to create"
	[test2 a95ff74] branch test2 to create
	 1 file changed, 2 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': ^C
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin test2
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 5, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 347 bytes | 0 bytes/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	 * [new branch]      test2 -> test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ ls
	README.md  testDemo
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test2
	Switched to branch 'test2'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge test2
	Updating f1f28d4..a95ff74
	Fast-forward
	 testDemo | 2 ++
	 1 file changed, 2 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Total 0 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   f1f28d4..a95ff74  master -> master
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

#### 冲突情况一
不确定这是否算是冲突,在本地同一仓库中的分支上更改内容后,并提交到暂存区后,切换到主分支上后,status可以检查到,也能查看到相应的更改内容

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test2
	Switched to branch 'test2'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test2
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add testDemo
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	M	testDemo
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes to be committed:
	  (use "git reset HEAD <file>..." to unstage)

		modified:   testDemo

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes to be committed:
	  (use "git reset HEAD <file>..." to unstage)

		modified:   testDemo

	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add testDemo
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg082702
	Password for 'https://cg082702@git.oschina.net': 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Everything up-to-date
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "branch master update"
	[master 6ce4589] branch master update
	 1 file changed, 4 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	* master
	  test2
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg0827 
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 5, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 391 bytes | 0 bytes/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   a95ff74..6ce4589  master -> master
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test2
	Switched to branch 'test2'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test2
	nothing to commit, working directory clean
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge master
	Updating a95ff74..6ce4589
	Fast-forward
	 testDemo | 4 ++++
	 1 file changed, 4 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

#### 分支冲突合并
当次分支和主分支都更改了内容时,输入代码
`git merge test3`后
说是将分支合并到主分支上,
其实是将两分支更改的内容都显示在主分支的文件里,这样是为了能在主分支上更方便的对比选择保留那个分支上更改的内容

![分支合并后选择](/images/git/git-gitUse.png)

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test3
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add .
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "push branch test3"
	[test3 7e54227] push branch test3
	 1 file changed, 2 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin test3
	Username for 'https://git.oschina.net': cg0827 
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 5, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 343 bytes | 0 bytes/s, done.
	Total 3 (delta 1), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	 * [new branch]      test3 -> test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test3
	Already on 'test3'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	nothing to commit, working directory clean
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add .
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "test in master for test3"
	[master fe73e5d] test in master for test3
	 1 file changed, 2 insertions(+)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin master
	Username for 'https://git.oschina.net': cg0827
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 7, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 340 bytes | 0 bytes/s, done.
	Total 3 (delta 1), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   889a0f8..fe73e5d  master -> master
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	* master
	  test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	9.验证master关于test3分支----------2017-0510-11:34:15
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test3
	Switched to branch 'test3'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	9.push分支test3---------2017-0510-11:28:03
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git branch
	  master
	* test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge test3
	Auto-merging testDemo
	CONFLICT (content): Merge conflict in testDemo
	Automatic merge failed; fix conflicts and then commit the result.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	<<<<<<< HEAD
	9.验证master关于test3分支----------2017-0510-11:34:15
	=======
	9.push分支test3---------2017-0510-11:28:03
	>>>>>>> test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	You have unmerged paths.
	  (fix conflicts and run "git commit")

	Unmerged paths:
	  (use "git add <file>..." to mark resolution)

		both modified:      testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

#### 分支合并解决方法
merate遇到error,那么回退到merge,再把分支上的更改成master上

	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge test3
	Auto-merging testDemo
	CONFLICT (content): Merge conflict in testDemo
	Automatic merge failed; fix conflicts and then commit the result.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	You have unmerged paths.
	  (fix conflicts and run "git commit")

	Unmerged paths:
	  (use "git add <file>..." to mark resolution)

		both modified:      testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	<<<<<<< HEAD
	9.验证master关于test3分支----------2017-0510-11:34:15
	=======
	9.push分支test3---------2017-0510-11:28:03
	>>>>>>> test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test3
	testDemo: needs merge
	error: you need to resolve your current index first
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git reset --merge
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge test3
	Auto-merging testDemo
	CONFLICT (content): Merge conflict in testDemo
	Automatic merge failed; fix conflicts and then commit the result.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	You have unmerged paths.
	  (fix conflicts and run "git commit")

	Unmerged paths:
	  (use "git add <file>..." to mark resolution)

		both modified:      testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	<<<<<<< HEAD
	9.验证master关于test3分支----------2017-0510-11:34:15
	=======
	9.push分支test3---------2017-0510-11:28:03
	>>>>>>> test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test3
	testDemo: needs merge
	error: you need to resolve your current index first
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git reset --merge
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout test3
	Switched to branch 'test3'
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	9.push分支test3---------2017-0510-11:28:03
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch test3
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   testDemo

	no changes added to commit (use "git add" and/or "git commit -a")
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ cat testDemo
	1.测试-----2017-0510-08:59:58

	2.test----2017-0510-09:50:12

	3.分支:test2----2017-0510-10:08:02

	4.分支test2存入--------2017-0510-10:30:24

	5.主分支master写入--------2017-0510-10:32:50

	6.分支test2写入---------2017-0510-10:48:22

	7.删除test2分支--------2017-0510-10:58:21

	8.添加test3分支----------2017-0510-11:15:38

	9.验证master关于test3分支----------2017-0510-11:34:15
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	error: Your local changes to the following files would be overwritten by checkout:
		testDemo
	Please, commit your changes or stash them before you can switch branches.
	Aborting
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git add .
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git commit -m "test3 change to master"
	[test3 e00cda5] test3 change to master
	 1 file changed, 1 insertion(+), 1 deletion(-)
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git push origin test3
	Username for 'https://git.oschina.net': cg0827  
	Password for 'https://cg0827@git.oschina.net': 
	Counting objects: 1, done.
	Writing objects: 100% (1/1), 188 bytes | 0 bytes/s, done.
	Total 1 (delta 0), reused 0 (delta 0)
	To https://git.oschina.net/cg0827/test-1.git
	   7e54227..e00cda5  test3 -> test3
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git merge test3
	Merge made by the 'recursive' strategy.
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ git status
	On branch master
	Your branch is ahead of 'origin/master' by 3 commits.
	  (use "git push" to publish your local commits)

	nothing to commit, working directory clean
	cg@cg-ThinkPad-Edge-E540 ~/Work/GitoscWork/test-1 $ 

