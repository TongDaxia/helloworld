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



绑定远程库

```
要关联一个远程库，使用命令git remote add origin git@github.com:path/repo-name.git；
关联后，使用命令git push -u origin master第一次推送 master 分支的所有内容；
此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改
后续推送命令： git push origin master
```



从远程获取

```
 远程库已经准备好了， 下一步是用命令git clone克隆一个本地库：
$ git clone git@github.com:michaelliao/gitskills.git 

```



分支管理

```
首先， 我们创建dev分支， 然后切换到dev分支：
$ git checkout -b dev
git checkout命令加上-b参数表示创建并切换， 相当于以下两条命令：
$ git branch dev
$ git checkout dev
然后， 用git branch命令查看当前分支：
$ git branch

提交修改过后

dev分支的工作完成， 我们就可以切换回master分支（文件看上去恢复了）：
$ git checkout master

现在， 我们把dev分支的工作成果合并到master分支上：
$ git merge dev

# Fast-forward信息， Git告诉我们， 这次合并是“快进模式”， 也就是直接把master指向dev的当前提交， 所
以合并速度非常快

删除分支     git branch -d dev


准备两个冲突 feature1 master
然后在master上面  git merge feature1
此时看git status 可以看到冲突

必须修改完冲突后手动的add commmit

查看分支情况
git log --graph --pretty=oneline --abbrev-commit
git log --graph

```



分支管理策略

```
创建  git checkout -b dev
修改，提交
切回 master
准备合并dev分支， 请注意--no-ff参数， 表示禁用Fast forward：
git merge --no-ff -m "merge with no-ff" dev

git log --graph --pretty=oneine --abbrev-commit

合并分支时， 加上--no-ff参数就可以用普通模式合并， 合并后的历史有分支， 能看出来曾经做过合并， 而fast
forward合并就看不出来曾经做过合并。
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





bug分支

```
当前：
两类文件，一个修改未add；一个add未commit.此时需要解决bug.
可以把当前工作现场“储藏”起来， 等以后恢复现场后继续工作：
git stash
首先确定要在哪个分支上修复bug， 假定需要在master分支上修复， 就从master创建临时分支：
$ git checkout master
git checkout -b  issue-1011
切换到新增的 bug分支，并解决问题。
切回master,合并。
 git merge --no-ff -m "merged bug fix 101" issue-101
 删除issue-1011的分支
 git branch -d issue-1011


切换回 dev 继续干活。
git stash list  查看已经暂存的。

一是用git stash apply恢复， 但是恢复后， stash内容并不删除， 
你需要用git stash drop来删除；
另一种方式是用git stash pop， 恢复的同时把stash内容也删了：
 
有多个stash 的时候 ，可以恢复到指定的stash
git stash apply stash@{0}


修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场；

在master分支上修复的bug，想要合并到当前dev分支，可以用git cherry-pick <commit>命令，把bug提交的修改“复制”到当前分支，避免重复劳动。
```



Feature 分支

```
有新功能需要开发，新建一个分支
git checkout -b feature-vulcan
……
完成后 准备合并
git checkout dev

合并取消，准备完全放弃：
git branch -d feature-vulcan
摧毁失败，强行删除
git branch -D feature-vulcan

```



多人协作

```
查看远程库的信息， 用
git remote
git remote -v

git push origin dev

推送失败， 因为你的小伙伴的最新提交和你试图推送的提交有冲突， 解决办法也很简单， Git已经提示我们， 先用git
pull把最新的提交从origin/dev抓下来， 然后， 在本地合并， 解决冲突， 再推送：
git pull
git pull也失败了， 原因是没有指定本地dev分支与远程origin/dev分支的链接， 根据提示， 设置dev和origin/dev的链接：
git branch --set-upstream dev origin/dev


因此， 多人协作的工作模式通常是这样：
1. 首先， 可以试图用git push origin branch-name推送自己的修改；
2. 如果推送失败， 则因为远程分支比你的本地更新， 需要先用git pull试图合并；
3. 如果合并有冲突， 则解决冲突， 并在本地提交；
4. 没有冲突或者解决掉冲突后， 再用git push origin branch-name推送就能成功！
如果git pull提示“no tracking information”， 则说明本地分支和远程分支的链接关系没有创建， 用命令git branch --set -upstream branch-name origin/branch-name。


```



标签管理

```
Git的标签虽然是版本库的快照， 但其实它就是指向某个commit的指针（跟分支很像对不对？ 但是分支可以移动， 标签不能移动） ， 所以， 创建和删除标签都是瞬间完成的。

git tag v1.0 -m "gidainlkafhfo"
查看之前的版本
 git log --pretty=oneline --abbrev-commit
 给之前的版本打标签
 git tag v0.9 6224937 -m "gidainlkafhfo"
 
 查看所有tag
 git tag 
 git show v1.0
 
 删除标签
 git tag  -d v0.5
 
 推送标签到远程
 git push origin v1.0 
 git push origin --tags
 
 已经推送的标枪删除
 git tag -d v0.9
 git push origin :refs/tags/v0.9
 

```



忽略列表

```
强制推送
git add -f App.class
查看推送和忽略列表的冲突
git check-ignore -v App.class

```



配置别名

```
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch

git config --global alias.unstage 'reset HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

cat .git/config

```






























