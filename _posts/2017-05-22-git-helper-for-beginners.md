---
layout: post
tId: 1705001
title: "给Git新手的入门级快速上手方式"
date: 2017-02-04 13:00:00 +0800
categories: 架构,分布式
codelang: csharp
desc: "以前都是使用p4v以及svn做版本控制，后来开始试着接触git，最初不熟悉git命令行，通过借助git图形交互窗体等辅助一些复杂的操作，能够快速上手git，实现版本协同开发"
---
搜索Git for windows下载widows上的git工具，安装过程比较简单的，基本上一路next就好，有选择的地方，可以阅读下说明，参考个人习惯选择。安装完成后，可进一步选择安装TortoiseGit，因为习惯他们家Svn工具的，使用这个可以更加方便。

工具安装完成后，可在目标文件件下，右键选择Git Bash Here来打开命令行窗口，也可以直接双击桌面上的程序icon运行命令行窗口，只不过这样还需要通过linux下的cd和ls命令来进行目录跳转（linux命令）。

### 1.git clone 项目地址

   一般来说，第一步首先是要clone项目到本地，命令比较简单,克隆完成后，通过cd命令进入到项目文件夹目录

### 2.git status

   用于查看目录状态，如果项目中有文件变更（文件的添加、修改、删除等），输出信息会列出文件信息，并且以红色字体显示。


    On branch master
    Your branch is up-to-date with 'origin/master'.
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
        _posts/2016-09-08-thinking-in-cms.md
        _posts/2017-05-22-git-helper-for-beginners.md

### 3.git add .

   通过此命令，可以将所有当前目录以下的变更文件添加进去。此时通过git status再次查看目录状态，会发现变更文件会以绿色字体显示，对于不再该目录以及其子目录下的文件，因为命令执行不到，仍然是红色字体显示，便于参考确定是否有文件遗漏

   执行：git add _posts/2017-05-22-git-helper-for-beginners.md后


    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)
      new file:
        _posts/2017-05-22-git-helper-for-beginners.md
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
        _posts/2016-09-08-thinking-in-cms.md

### 4.git commit -m '描述版本变更信息'

   该步骤执行提交命令，将上一步骤中的绿色字体显示的文件提交版本控制，这一步只是做本地的版本控制，在远程系统中，目前是无法获取到提交的信息的。

    [master 790c5da] —add git post
     1 file changed, 50 insertions(+)
     create mode 100644
     _posts/2017-05-22-git-helper-for-beginners.md

   此时，仍然可以通过git status检查提交状态，一般会显示你比远程服务器领先1 commits，当然如果你提交多了，就不是1 commits了。

    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
      (use "git push" to publish your local commits)
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
        _posts/2016-09-08-thinking-in-cms.md


### 5. git pull origin master

   获取远程最新版本内容（命令只针对于master分支，git库默认存在master分支，分支操作比较复杂，后期熟悉后在进一步学习操作）。

    From https://github.com/Skysper/Skysper.github.io
      * branch            master     -> FETCH_HEAD
    Already up-to-date.

   此时，可能会因为本地和服务器上已经存在的版本，针对与某个文件存在冲突，这种情况下，就要优先处理冲突，再次提交，这时候，可以通过TortoiseGit的图形界面来进行版本比较，合并处理，这样直观，操作方便。解决冲突的处理方式跟svn等其他版本控制工具类似。

### 6. git push origin master

   在本地提交没有问题的情况下，便可以通过该命令，将本地的提交同步到远程服务器上，git commit提交的版本，仅仅是通过本地做了版本控制，在没有同步到服务器的情况下，其他协同人员是无法获取到的，如果本能地硬盘损坏等情况发生，内容也就丢失了。

    Username for 'https://github.com': wtwpfwd0708@gmail.com
    Password for 'https://wtwpfwd0708@gmail.com@github.com':
    Counting objects: 4, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (4/4), 2.52 KiB | 0 bytes/s, done.
    Total 4 (delta 2), reused 0 (delta 0)
    remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
    To https://github.com/Skysper/Skysper.github.io.git
      d5db2e4..790c5da  master -> master

### 最后

git clone 命令简单直观，而且也就是在第一次克隆项目到本地的时候才执行，在整个项目中，最初只需要记住和使用的也就是剩下的5条命令，应该是方便理解和掌握的。

对于其他的情况，比如说想取消文件修改，恢复到未修改前的状态或者忽略某个文件夹不做版本控制以及删除某个文件或文件夹的版本控制等等复杂的情况，这些都可以通过TortoiseGit为我们提供的右键菜单内的命令来实现，另外VS2015对Git有了更强大的支持，更加的方便。

其实在有了以上两种工具的情况下，你甚至可以不需要熟悉任何命令操作，直接使用工具，不过Git bash是很强大的，是Git的核心，同时熟悉命令行能够让你对于Git的理解更深一点，另外就是有些操作图形化工具会出现一些异常情况，这时候，用命令行工具反而更直接更直观。

另外，针对与远程服务器的操作，每次都需要输入用户名和密码，可以通过配置ssh，以免于每次都要登录的繁琐，不过这种情况下，需要在clone的时候，使用ssh地址，已经使用https的，可以在git的配置中进行调整，这方面的内容可在熟悉以后，再详细了解。
