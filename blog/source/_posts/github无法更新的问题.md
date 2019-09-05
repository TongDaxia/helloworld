---
title: github无法更新的问题
date: 2019-05-27 
tags: [git]
---

使用github 无法正常更新

<!--more-->

更新选项：

Update Type:

​	Merge  

​	Rebase 

√	Branch Default 

Clean working tree before update

√	Using Stash  

​	Using Shelve



注意：

Merge 是合并

*rebase*操作可以把本地未push的分叉提交历史整理成直线; *rebase*的目的是使得我们在查看历史提交的变化时更容易,因为分叉的提交需要三方对比。

git *stash* 可用来暂存当前正在进行的工作, 比如想pull 最新代码, 又不想加新commit

git  Shelve 暂存

提示：

Can't Update

No tracked branch configured for branch master or the branch doesn't exist. To make your branch track a remote branch call, for example, git branch --set-upstream-to origin/master master