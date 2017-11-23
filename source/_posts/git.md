---
title: Git
date: 2016-07-20 10:24:13
tags: tool
category: IT
---

# 典型应用（项目和第三方库管理）

需求：
```
 1. 在构建项目时，会把该项目用git管理
 2. 中间会引入第三方库，库资源只能clone，并没有push权限
 3. 项目大了后，有些子模块要分出去由专门团队维护，也就是有pull和push权限
```

引用：
```
 应用情形，一个已经在开发的branch，需要在某个directory引用一个第三方的git project。如果不是简单clone和去掉.git，然后视作同一个project，需要用到git submodule。[http://www.kafeitu.me/git/2012/03/27/git-submodule.html](http://www.kafeitu.me/git/2012/03/27/git-submodule.html)
```

思路：
```
 1. 第三方库，如果需要修改，需要单独为期构建一个git库。
  操作流程以hexo用到的theme next为例。首先theme next最后是作为一个submodule存在的。这个过程是按下面流程实现的。
  
  [kevin@localhost themes]$ cd /home/kevin/maylink-llc/private/hexo/themes
  [kevin@localhost themes]$ git clone https://github.com/iissnan/hexo-theme-next
  
  #查看remote
  [kevin@localhost next]$ git remote -v
  origin  https://github.com/iissnan/hexo-theme-next (fetch)
  origin  https://github.com/iissnan/hexo-theme-next (push)
  
  这个时候本地branch 'master' 也是追踪到远程分支origin/master
  然后可以在github上创建一个repository名字就叫next。同时执行
  git remote add next git@github.com:swordhqm/next.git

  #设置追踪远程分支 next/master
  #git push -u 远程仓库名 本地branch:远程branch
  [kevin@localhost next]$ git push -u next master:master
  分支 master 设置为跟踪来自 next 的远程分支 master。
  Everything up-to-date

  #这个时候查看下信息
  [kevin@localhost next]$ git remote -v
  next    git@github.com:swordhqm/next.git (fetch)
  next    git@github.com:swordhqm/next.git (push)
  origin  https://github.com/iissnan/hexo-theme-next (fetch)
  origin  https://github.com/iissnan/hexo-theme-next (push)

  [kevin@localhost next]$ git branch -av
  * master                         c5064e9 [modify] add search
  remotes/next/master            c5064e9 [modify] add search
  remotes/origin/5.1.0           27e0a7a Merge pull request #953 from yhlfh/5.1.0
  remotes/origin/HEAD            -> origin/master
  remotes/origin/button-refactor 6f824bb Finished need-more-share2 refactor.
  remotes/origin/dev             420a0e2 Merge branch 'button-refactor'
  remotes/origin/master          970852e Merge pull request #1996 from shenzekun/master
  remotes/origin/servant         7b96fa5 Remove vendors/jeet
  remotes/origin/testing         4bbf563 E2E Testing Setup (WIP).

  [kevin@localhost next]$ git config -l
  credential.helper=store
  alias.lg=log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
  push.default=upstream
  core.repositoryformatversion=0
  core.filemode=true
  core.bare=false
  core.logallrefupdates=true
  core.worktree=../../../../themes/next
  remote.origin.url=https://github.com/iissnan/hexo-theme-next
  remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
  #当前master默认的upstream
  #git branch --set-upstream temp origin/xxx 设置远程分支
  #git checkout -b temp --track origin/xxx 设置远程分支
  branch.master.remote=next
  branch.master.merge=refs/heads/master
  remote.next.url=git@github.com:swordhqm/next.git
  remote.next.fetch=+refs/heads/*:refs/remotes/next/*

  #当远程分支origin，也就是第三方库有发生更新时
  ##先取回origin的master分支
  [kevin@localhost next]$ git fetch origin master
  来自 https://github.com/iissnan/hexo-theme-next
  * branch            master     -> FETCH_HEAD
  ##merge 本地取回来的origin/master
  git merge origin/master 或者 git rebase origin/master
  ##push本地分支
  git push
```

```
2. 对于hexo工程而言，theme next只是其中用到的一个子模块，所以可以通过submodule来管理

```