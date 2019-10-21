---
title: Git 介绍和学习
date: 2019-02-1 19:92:00
tags: [git]
---





```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```



```
$ mkdir learngit
$ cd learngit
$ pwd
$ git init
```



```
git add readme.txt
```



```
 git status
 git diff readme.txt 
 git add readme.txt
 git commit -m "add distributed"
 git status
```



```
git log
git reset --hard HEAD^
git reset --hard 1094a
git reflog  #显示每一次的提交


```



```
要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。

Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。
```







```
小结
Git鼓励大量使用分支：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>或者git switch <name>

创建+切换分支：git checkout -b <name>或者git switch -c <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>
```





```
Git分支十分强大，在团队开发中应该充分应用。

合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
```



```
修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场；

在master分支上修复的bug，想要合并到当前dev分支，可以用git cherry-pick <commit>命令，把bug提交的修改“复制”到当前分支，避免重复劳动。
```




































