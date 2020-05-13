---
title: git命令介绍
abbrlink: 5220878b-git-command-introduction
date: 2018-01-11 22:00:32
tags:
---
`git`的命令跟`Linus系统`的命令部分是一样的，因为创作者是同一个人 **Linus Torvalds**，本文主要是为了记录下学习git的过程以及常用的命令。如果是初学者，请拖到文章末尾，点击参考引用处的链接。
<!-- more -->
### 一、操控文件夹
- 创建文件夹
`$ mkdir learngit`
- 进入文件夹
`$ cd learngit`
- 显示当前路径
```
$ pwd   
/Users/michael/learngit
```
### 二、初始化仓库及操控文件
- 把这个目录变成Git可以管理的仓库
```
$ git init
Initialized empty Git repository in f:/dev-doc/gitlearn/.git/
```
此时会生成一个`.git`目录，默认是被隐藏了的，可使用`ls -ah`进行查看。
- 在当前目录新建 .txt 文件后填写一些内容，然后把文件添加到版本库(暂存区stage)中。
`git add readme.txt`
- 提交(暂存区stage)上一步添加的内容
`git commit -m "第一次提交readme.txt"`
- 时刻掌握仓库当前的状态
`git status`
-   查看difference
`git diff`
- 查看从最近到最远的提交日志
```
git log
$ git log --pretty=oneline  //优雅展示
```

### 三、版本回退
在 Git 中，用`HEAD`表示当前版本,上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本可以简写成`HEAD~100`
```
git reset --hard HEAD^
```
当刚才那个版本回退了，又不想回退了，通过commit id进行回退（前提：刚才那个命令行对话框还没有关闭） 
```
git reset --hard <版本号>
//版本号没必要写全，前几位就可以了，Git会自动去找
```
Git 的版本回退速度非常快，因为 Git 在内部有个指向当前版本的 HEAD 指针，当你回退版本的时候，Git 仅仅是把 HEAD 从指向你回退的版本。
- 查看命令历史，以便确定要回到未来的哪个版本
`git reflog`
- 撤销文件在工作区的修改,让这个文件回到最近一次`git commit`或`git add`时的状态
`git checkout -- readme.txt` 命令中的`--`别漏了。
- 把暂存区的修改撤销掉，重新放回工作区
`git reset HEAD readme.txt`
git reset命令既可以**回退版本**，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本。
- 删除文件
```
rm haha.txt  //删除文件，还未添加到 暂存区stage 
git rm haha.txt //删除文件后同时添加到暂存区stage
```

### 四、远程仓库
- 关联远程仓库
```
git remote add origin git@git.coding.net:shandianlala/LearnGit.git
git push -u origin master  //第一次带上-u,后续不用
git pull git@git.coding.net:shandianlala/LearnGit.git
git push origin master
```
- 克隆远程仓库
`git clone git@git.coding.net:shandianlala/LearnGit.git`


### 五、分支策略
- 分支
```
查看分支：git branch
创建分支：git branch <name>
切换分支：git checkout <name>
创建+切换分支：git checkout -b <name>
合并某分支到当前分支：git merge <name>
删除分支：git branch -d <name>

git log --graph --pretty=oneline --abbrev-commit  //分支合并图
git log --graph //分支合并图
//打印日志太多按 q 退出查看，按 space 空格下一页日志。
```
- 分支合并策略
```
git merge --no-ff -m "merge with no-ff" dev
//准备合并dev分支，请注意--no-ff参数，表示禁用Fast forward
//因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去。
```
合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

- 开发一个新feature，最好新建一个分支；
- 如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <branchName>`强行删除。
- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。


### 六、标签
- 命令`git tag <name> [commitId]`用于新建一个标签，默认为`HEAD`，也可以指定一个`commit id`；
- `git tag -a <tagname> -m "打个标签..."`可以指定标签信息；
- `git tag -s <tagname> -m "打个标签..."`可以用PGP签名标签；
- 命令`git tag`可以查看所有标签。
- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签（先删除本地，在远程）。


> 所有的版本控制系统，其实只能跟踪文本文件的改动，而图片、视频、word这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只知道图片从99KB改成了178KB，但到底改了哪些，版本控制系统不知道，也没法知道。
 
**参考**
https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000