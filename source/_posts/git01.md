---
layout: post
title: git
date: 2018-02-09 01:12:47
tags:
---

git中最重要的三个点：工作区、暂存区和树形分支

### git基本操作

- `$ git init`： 初始化一个Git仓库
- `git remote add origin https://github.com/suguoqiang/hexo.git`：添加一个origin变量代替url地址
- `$ git add <file>`： 添加文件到暂存区，可反复使用。`.`代表当前路径下所有文件
- `$ git commit -m <message>`：提交到树形分支
- `$ git status`：查看当前仓库状态
- `$ git diff <file>`：查看文件异同，如：`git diff HEAD -- readme.txt `
- `$ git log`：查看最近到最远的提交日志。可选简化的参数：`--pretty=oneline`
- `$ git rm <file>`: 删除版本库中的某一个文件
- `$ git remote -v` ：详细远程库的信息

<!-- more -->

### git其他

####  版本回退
- `$ git reset --hard HEAD^`：回退到上一个版本
`HEAD`表示当前版本,上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本太多，所以写成`HEAD~100`

- `$ git reset --hard 1094a`：回退到指定版本，版本号为1094a...

- `$ git reflog`：查看命令历史，以便确定要回到未来的哪个版本。

#### 文件回退
- `$ git checkout -- file`：把file文件在工作区的修改全部撤销，回到最初状态

- `$ git reset HEAD <file>`：把暂存区的修改撤销掉（unstage），重新放回工作区

#### 库操作
- `$ git remote add origin git@server-name:path/repo-name.git`：关联一个远程库，库的别名为origin
- `$ git push -u origin branch-name`：推送代码到库中的master分支（第一次），以后使用`$ git push origin branch-name`
- `$ git clone origin`：克隆仓库到本地

#### 分支操作
- `$ git checkout -b <name>`：创建并切换分支到name分支
- `$ git branch <name>`：创建name分支
- `$ git checkout <name>`：切换分支
- `$ git branch`：查看所有分支，当前分支前面会标一个`*`号
- `$ git merge <name>`：合并某分支name到当前分支
- `$ git branch -d <name>`：删除指定分支（已合并的分支）
- `$ git branch -D <name>`：强行删除没有被合并的分支

>分支操作容易造成合并冲突，需手动解决冲突后，在提交
>Git默认采用Fast forward模式，如果不想使用，用命令`$ git merge --no-ff -m "merge with no-ff" dev`,请注意`--no-ff`参数，表示禁用Fast forward,采用普通模式合并，合并后的有分支历史可查。

- `$ git checkout -b branch-name origin/branch-name`：在本地创建和远程分支对应的分支

- `$ git log --graph`：查看分支合并图

#### 工作现场保存
>场景：当前正在编写一个功能，突然有紧急的BUG要修复，但是当前功能还没有完成，不能提交，只能稍后再回来操作，需要暂存。
- `$ git stash`：保存工作现场
- `$ git stash list`：查看所有工作现场
- `$ git stash apply`：恢复工作现场。恢复完成后，就要将原现场删除
- `$ git stash drop`：删除工作现场
- `$ git stash pop`：恢复的同时并删除工作现场

#### 多人协作工作模式

1、首先，可以试图用`$ git push origin <branch-name>`推送自己的修改；

2、如果推送失败，则因为远程分支比你的本地更新，需要先用`$ git pull`试图合并；

3、如果合并有冲突，则解决冲突，并在本地提交；

4、没有冲突或者解决掉冲突后，再用`$ git push origin <branch-name>`推送就能成功！

5、如果`$ git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`$ git branch --set-upstream-to <branch-name> origin/<branch-name>`。 

#### git标签
- `git tag <tagname> <branch-name>`：切换到某一分支，新建一个标签，默认为HEAD，也可以指定一个commit id
- `git tag -a <tagname> -m "blablabla..."`：可以指定标签信息
- `git tag`：查看所有标签
- `git show <tagname>`：查看标签信息
- `git tag -d <tagname>`：删除标签
- `git push origin <tagname>`：推送一个本地标签到远程
- `git push origin --tags`：推送全部未推送过的本地标签
- `git push origin :refs/tags/<tagname>`：删除一个远程标签

#### 自定义
最常见的设置：

- ` git config --global user.name gqsu`
- ` git config --global user.email 597009281@qq.com`






