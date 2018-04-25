---
title: git学习笔记
date: 2018-4-24 23:08:32
categories:
tags:
     - Git
---

学习Git是很有必要的，Git是目前世界上最先进的分布式版本控制系统,版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。

<!--more-->
`git init`     初始化git仓库

`git add readme.txt`用命令git add告诉Git，把文件添加到仓库

`git commit -m "add 3 files."`
`m`后面输入的是本次提交的说明。为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件
```
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```

`git status`命令可以让我们时刻掌握仓库当前的状态

`git diff` 查看具体修改了什么内容

`git log`命令显示从最近到最远的提交日志

加参数查询
```
$ git log --pretty=oneline
3628164fb26d48395383f8f31179f24e0882e1e0 append GPL
ea34578d5496d7dd233c827ed32a8cd576c5ee85 add distributed
cb926e7ea50ad11b8f9e909c05226233bf755030 wrote a readme file
```



回退

首先，`Git`必须知道当前版本是哪个版本，在`Git`中，用`HEAD`表示当前版本，也就是最新的提交3628164...882e1e0（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。
```
回退到上一个版本
$ git reset --hard HEAD^
HEAD is now at ea34578 add distributed
```
```
回退后突然后悔了，3628164是最新的那个版本
$ git reset --hard 3628164
HEAD is now at 3628164 append GPL
```
现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的commit id怎么办？

在Git中，总是有后悔药可以吃的。当你用`git reset --hard HEAD^`回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令`git reflog`用来记录你的每一次命令：
```
$ git reflog
ea34578 HEAD@{0}: reset: moving to HEAD^
3628164 HEAD@{1}: commit: append GPL
ea34578 HEAD@{2}: commit: add distributed
cb926e7 HEAD@{3}: commit (initial): wrote a readme file
```

HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。

`git log`可以查看提交历史，以便确定要回退到哪个版本

`git reflog`查看命令历史，以便确定要回到未来的哪个版本

`git checkout -- file`可以丢弃工作区的修改

把readme.txt文件在工作区的修改全部撤销，这里有两种情况

一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态

`git reset HEAD file`可以把暂存区的修改撤销掉（unstage），重新放回工作区

`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本

删除版本库的某个文件

1.`git rm test.txt`

2.`git commit -m "remove test.txt"`
这样就成功从版本库中删除

删错了？
```
git checkout -- test.txt
```

命令git rm用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

推送

本地Git仓库和GitHub仓库之间的传输是通过SSH加密
```
ssh-keygen -t rsa -C "youremail@example.com"
```
你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码

如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

关联仓库
```
git remote add origin https://github.com/fshgrym/learngit.git
远程库的名字就是origin

```
当前分支master

推送到远程库
```
git push -u origin master
```
由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

```
HTTPS的格式为：https://github.com/用户名/仓库名.git
git remote add origin https://github.com/fshgrym/learngit.git

SSH的格式为：git@github.com:用户名/仓库名.git
git remote add origin git@github.com:fshgrym/learngit.git

```

日常报错
```
fatal: remote origin already exists.
解决：git remote add origin
```
