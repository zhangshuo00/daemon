+++
author = "zs"
title = "工具箱 - git相关"
date = "2022-08-05"
lastmod = "2022-08-05"
description = ""
tags = [
    "git",
    "toolbox",
]
+++

**「将当前分支修改内容提交到新分支上」**

步骤1：在当前的develop分支上的修改暂存起来

`git stash`

步骤2：暂存修改后，在本地新建分支（develop_backup为新分支的名字）

`git checkout -b develop_backup`

步骤3：将暂存的修改放到新建分支中

`git stash pop`

步骤4：使用命令进行常规的add、commit步骤

`git add.`

`git commit -m "修改内容"`

步骤5：将提交的内容push到远程服务器(在远程也同步新建分支develop_backup)

`git push origin develop_backup`

— —

**「先有本地目录再关联远程仓库」**

有一个项目名为 test

`cd test`

`git init`

`git checkout -b newBranchName` (如果不创建新的分支，可以忽略这步)

`git add .`

`git commit -m”init”`

在github或gitlab等创建仓库
获取远程仓库地址：https://github.com/xxx/test.git

`git remote add newBranchName https://github.com/xxx/test.git`

`git push —set-upstream newBranchName newBranchName`

— —

**「通过用户名和密码clone仓库」**

`git clone http://username:password@ip:port/xx/uem-vis-realtime.git`

特殊符号需要转义：

>常用：
>*邮箱中的 @ 要使用 %40 代替*
>*? 用 %3F 代替*

— —

**「查看分支」**

`git branch -r`
