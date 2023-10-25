---
layout: post
title: "git command"
date: 2018-06-28 13:22:00
description: ""
category: command
tags: [git]
---

![]({{"/assets/images/post/git-command.jpg" | absolute_url }})

## [git仓库](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%8E%B7%E5%8F%96-Git-%E4%BB%93%E5%BA%93)

```
git clone https://username:ghp_TOKEN@github.com/<USERNAME>/<REPO>.git #private repo
git remote set-url origin https://username:ghp_TOKEN@github.com/<USERNAME>/<REPO>.git #ignore password
git config --global http.sslVerify "false" #OpenSSL SSL_read:SSL_ERROR_SYSCALL, errno 10054
```

**新建本地仓库**（local repository）方法有两种：

1. 根据本地已有项目，进入该项目目录，运行：

   ```sh
   $ git init
   ```

2. 克隆远端仓库（remote repository）

   ```sh
   $ git clone https://github.com/***/***.git
   ```

git clone完成之后，git会自动将remote repository命名为**origin**（可以采用-o <name>改名），并基于origin/master，创建了一个local branch：master，此时两者指向同一个commit对象。

文件的状态有四种：untracked、unmodified、modified、staged，查看和修改文件状态可以采用：

```sh
$ git status #查看文件状态
$ git add . #如果是目录，会递归处理目录下所有文件。这是一个多功能命令，可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。
$ git diff #显示unstaged的改动
$ git diff --staged #显示已经staged的改动
$ git commit -m "issue 181: fix some bug"
$ git commit -a -m "issue 181: fix some bug" #加上-a，会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add
```

**删除、移动文件**

```sh
$ git rm log/\*.log #从repository和工作目录同时删除，由于git有自己的文件扩展，不需要shell帮忙，所以加上反斜杠\转义星号*
$ git rm --cached README #从repository删除，在工作目录保持
$ git mv file_from file_to #相当于运行了下面三条命令：
$ mv file_from file_to
$ git rm file_from
$ git add file_to
```

**查看commit历史**

```sh
$ git log -p -2 #-p显示每次提交的内容差异，-2显示最近两次改动
$ git log --stat #简略的统计信息
$ git log --pretty=oneline --graph #按某些格式显示，可选oneline，short，full，fuller，format:"%h - %an, %ar : %s"，结合--graph选项添加了一些ASCII字符串来形象地展示你的分支、合并历史
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" --before="2008-11-01" --no-merges -- t/ #用两个短划线（--）隔开之前的选项和后面限定的路径名，表示只关心某些文件或者目录的历史提交
```

**撤销操作**

```sh
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend #最终只会有一个提交 - 第二次提交将代替第一次提交的结果

$ git reset --hard [<commit>] #commit之后的修改被撤销
$ git reset HEAD CONTRIBUTING.md #将已staged的文件改为unstaged
$ git checkout -- CONTRIBUTING.md #撤销某个文件的修改，由于未commit，撤销就再也找不到了，慎用！
```

**远程仓库**

```sh
$ git remote -v #查看远程仓库
origin  https://github.com/ohmygodlin/ohmygodlin.github.io.git (fetch)
origin  https://github.com/ohmygodlin/ohmygodlin.github.io.git (push)
$ git remote add pb https://github.com/paulboone/ticgit #添加远程仓库
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
$ git remote set-url origin https://... #更改远程仓库
$ git fetch pb #从远程仓库拉取
$ git push origin master #[remote repository] [remote branch]可省略
$ git remote show origin #查看某个远程仓库详细信息
$ git remote rename pb paul #重命名
$ git remote rm paul #删除
```

[将项目同时提交到多个git仓库](http://feitianbenyue.iteye.com/blog/2376791)

```sh
$ git remote set-url --add origin https://... #让远程库origin拥有多个url地址，push到多个，从第一个get，如果需要改变get的路径，直接修改.git/config文件中url的顺序
$ git push -f origin master #本地内容强制覆盖推向远程仓库
```

注意：`git fetch` 命令会将数据拉取到你的本地仓库，它并不会自动合并或修改你当前的工作。而`git pull` 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。 可以认为`git pull`是`git fetch`和`git merge`两个步骤的结合。

**打标签**

```sh
$ git tag -l 'v1.8.5*' #不带-l列出所有标签
$ git tag -a v1.4 -m 'my version 1.4' #创建附注标签
$ git tag v1.4-lw #创建轻量标签
$ git tag -a v1.2 9fceb02  #为某个commit打标签
$ git push origin v1.5 #git push命令不会传送标签到远程仓库服务器上。在创建完标签后必须显式地推送标签
$ git push origin --tags #推送所有未推送的标签
$ git checkout -b version2 v2.0.0 #基于特定标签v2.0.0创建新branch：version2，如果在这之后又进行了一次提交，version2会因为改动向前移动，那么version2 branch就会和v2.0.0标签稍微有些不同，这时就应该当心了
```

**使用别名**

```sh
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.unstage 'reset HEAD --'
$ git config --global alias.last 'log -1 HEAD'
$ git config --global alias.visual '!gitk' #命令前面加入!符号，表示执行外部命令，因此git visual，等价于运行gitk
```

## [git分支](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E4%BD%95%E8%B0%93%E5%88%86%E6%94%AF) 

提交（commit）对象：包含一个指向暂存内容快照的指针，包含本次提交的作者等相关附属信息，包含零个或多个指向该提交对象的父对象指针。git可以认为是管理commit的系统，由一个commit列表（old<-new）构成一个分支（branch），branch本质上是指向commit列表头的可变指针，随着每次提交向新commit的对象移动。

在git服务器（如github）上的是远端分支（remote branch），在本地是本地分支（local branch），在git push之前，所有改动都是在local branch进行，和remote branch是独立（并行）的。gitk显示的是local repository。

remote server/remote repository/remote branch

local host/local repository/local branch

默认创建的branch是**master**，创建新的branch，本质上是添加一个指向当前commit对象的指针，git保存了一个名为**HEAD**的特别指针，指向正在工作的local branch ![]({{"/assets/images/post/advance-testing.png" | absolute_url }})

```sh
$ git branch testing #创建branch
$ git checkout testing #转换到testing branch
$ git checkout -b testing #新建并切换branch，等效于上面两条指令
```

开发新功能的一般步骤：

1. 在master切换到新的branch：new_feature，并进行相应的改动

   ```sh
   $ git checkout -b new_feature
   $ vim index.html
   $ git commit -a -m 'added a new footer [new_feature]'
   ```

2. 突然需要在master进行漏洞修复，先基于master创建一个新的branch：bug_fix，待完整测试无误后再合并到master

   ```sh
   $ git checkout master #切换回master
   $ git checkout -b bug_fix #新建并切换到bugfix
   $ vim index.html
   $ git commit -a -m 'fix bug'
   $ git checkout master
   $ git merge bug_fix #由于当前master所在的提交对象是要并入的bug_fix分支的直接上游，只需把master指针直接右移（快进合并）
   $ git branch -d bug_fix #由于bug_fix和master指向相同commit，可以删掉
   ```

3. 回到new_feature，继续工作。

   ```sh
   $ git checkout new_feature
   $ vim index.html
   $ git commit -a -m 'finished the new footer [new_feature]'
   ```

   注意之前bug_fix的修改并没有在new_feature中。如果需要纳入此次修改，可以用 `git merge master` 把`master`合并到 `new_feature`；或者等 `new_feature` 完成之后，再将 `new_feature` 分支中的更新并入`master`。

4. 等new_feature完成之后，合并回master

   ```sh
   $ git checkout master
   $ git merge new_feature
   $ git status #由于master不是new_feature直接上游，出现文件冲突时，先通过git status查阅
   $ git mergetool #之后可以通过手工修改，然后git add；或调用可视化工具
   $ git status #再运行一次 git status 来确认所有冲突都已解决
   ```

   或者采用变基（rebase）方式，可以得到一个较为整洁的commit历史，建议采用此方式

   ```sh
   $ git checkout new_feature
   $ git rebase master #将master变为new_feature直接上游
   $ git rebase master new_feature #等效于上两个命令
   $ git checkout master
   $ git merge new_feature #可以进行快进合并
   ```

   奇妙的变基也并非完美无缺，要用它得遵守一条准则：

   **不要对在你的仓库外有副本的分支执行变基。** 
