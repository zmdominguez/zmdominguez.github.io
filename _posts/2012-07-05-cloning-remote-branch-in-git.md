---
layout: post
title: Cloning a remote branch in git
date: '2012-07-05T13:28:00.000+10:00'
author: Zarah Dominguez
tags:
  - submodule
  - remote branch
  - git
  - android
modified_time: '2012-08-07T11:54:28.080+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-3182110615591203974
blogger_orig_url: http://www.zdominguez.com/2012/07/cloning-remote-branch-in-git.html
---

My current project at work uses git, and I have always been a CVS/SVN baby so I'm still trying to find my way around it. Today I wanted to clone a remote branch to my local computer. This remote branch also has [submodules](http://git-scm.com/book/en/Git-Tools-Submodules), so I want to get those too.

This assumes that you use [Git Bash](http://msysgit.github.com/). First, navigate to the folder in you local computer where you want git to clone the remote branch. Once there, we can start cloning the repo. The following steps do the dirty work:
```shell
$ git init
$ git fetch <git url> <branch name>:refs/remotes/origin/<branch name>
$ git checkout -b <branch name> origin/<branch name>
```

This retrieves the contents of the remote branch and copies it to our local computer in a local branch (confused yet?). To update our copy of the submodules, the following commands should work:
```shell
$ git submodule init
$ git submodule update
```
