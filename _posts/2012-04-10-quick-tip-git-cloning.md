---
layout: post
title: 'Quick Tip: git cloning'
date: '2012-04-10T12:48:00.000+10:00'
author: Zarah Dominguez
tags:
  - clone
  - eclipse
  - quick tips
  - git
modified_time: '2014-05-13T06:32:38.030+10:00'
thumbnail: http://2.bp.blogspot.com/-RExj5aMT3j8/T4OfC_wxrBI/AAAAAAAABBE/ZTBJMNyxOY4/s72-c/git_clone.PNG
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2055833932626466505
blogger_orig_url: http://www.zdominguez.com/2012/04/quick-tip-git-cloning.html
---

A user-friendly way of cloning a `git` repo is through the [eGit](http://www.eclipse.org/egit/) plug-in in Eclipse. But sometimes, especially on Windows machines, Eclipse has trouble cleaning up after itself after completing a clone operation. The best workaround for this is to clone the repo from `git bash` and then import the repo in Eclipse.
```shell
default@ZDOMINGUEZ-T420 ~
$ git clone git@github.com:<your git repo> <local folder to check out to>
```

When git finishes cloning your repo, import it to Eclipse.
<div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-RExj5aMT3j8/T4OfC_wxrBI/AAAAAAAABBE/ZTBJMNyxOY4/s1600/git_clone.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-RExj5aMT3j8/T4OfC_wxrBI/AAAAAAAABBE/ZTBJMNyxOY4/s1600/git_clone.PNG" /></a></div>

Browse to the folder you checked out to, click OK, and the newly-cloned repo should now appear on the Git Repositories view.