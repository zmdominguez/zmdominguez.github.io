---
layout: post
title: Fixing a mistake in your git history
date: '2015-04-22T21:08:00.000+10:00'
author: Zarah Dominguez
tags:
  - git
modified_time: '2015-04-22T21:08:45.498+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-5424734702179850648
blogger_orig_url: http://www.zdominguez.com/2015/04/fixing-mistake-in-your-git-history.html
---

I have been using git for about five years now, but I definitely get stumped by it a lot. It is so powerful it's daunting. There has been a <strike>couple</strike> lot of times where I had been too careless and reliant on my fingers' add-commit-push muscle memory that I realised I have made a mistake too late. I have always been a proponent of clean, atomic commits, and when I find my commits all messed up, I hit myself in the head.

So, to stop myself from pulling my hair out looking for all the right StackOverflow answers that I KNOW I'VE SEEN THE SOLUTION BEFORE WHERE THE HELL IS IT???, I am writing the steps down to remind myself.

#### SCENARIO:
- I have committed some files in a previous commit that should not be there
- I want to REMOVE those files from that commit
- I want to preserve all other commits after that bad commit

#### CAVEATS:
- I am working in a local branch
- I will be removing those files forever
- I AM WORKING IN A LOCAL BRANCH*

#### SOLUTION:
##### Find out which commit you want to go back to.
```shell
$ git log
```
This should give you something like:

```shell
zarah.dominguez@R5003334 swipe-to-refresh-demo (master) 
$ git log
commit a6c00638b3d466a61e3381a98e6b44cf2d085164
Author: Zarah Dominguez
Date:   Tue Jun 17 16:34:50 2014 +0800
    
    Removed dependency on ButterKnife.
    
commit e25c6862a79270921a24d6bf2a9eb07cc3f03b36
Author: Zarah Dominguez
Date:   Tue Jun 17 16:07:33 2014 +0800
        
    First commit
```

If you just want the commit messages:
```shell
$ git log --oneline</code>
```

##### Create a new branch based on the bad commit.
```shell
$ git checkout -b fix-that-shit e25c686
```

What this does is create a new branch named `fix-that-shit`, whose `HEAD` points to commit `e25c686`. The `-b` switch tells git to go to that newly-created branch.

##### Do your thing. In this case, I want to remove files.
```shell
$ git rm BadFile.java
rm 'BadFile.java'
$ git rm AnotherBadFile.java
rm 'AnotherBadFile.java'
```

##### Let git know that you've overcome your stupidity and are now saying sorry.
```shell
<span style="font-family: monospace;">$ git commit --amend</span>
```
An editor will open, and here you can edit the commit message.

##### Go back to the original branch. In my case, it is master.
```shell
$ git checkout master
```

##### Give this branch your changes.
```shell
$ git rebase fix-that-shit
```

##### Check your log. git might try and do it's thing, and do funny merges. So you might end up with a new commit in your history, something like:

```shell
zarah.dominguez@R5003334 swipe-to-refresh-demo (master) $ git log
commit 3e08701339c35301caf269058eab6359c9d87ecd
Author: Zarah Dominguez
Date:   Tue Jun 17 16:34:50 2014 +0800

    Removed dependency on ButterKnife.

commit ca57ac7974d7a422a2225612404cb6bd555acfc4
Author: Zarah Dominguez 
Date:   Tue Jun 17 16:07:33 2014 +0800
    
    First commit

commit 679ad41bedb2d61fbea36e39e334074b7de66dcd
Author: Zarah Dominguez
Date:   Tue Jun 17 16:07:33 2014 +0800
    
    First commit
```

Examine the two commits, and you'll notice that the second in the list contains the file we just removed. I want to chuck that out completely, so I will do a rebase. This will take me back to the first commit in interactive mode:

```shell
$ git rebase -i --root
```

Now I can trash that bad commit by adding a pound sign (or fine, hashtag) before that commit's SHA:

```shell
pick 679ad41 First commit
#pick ca57ac7 First commit
pick 3e08701 Removed dependency on ButterKnife.
```

##### Check your logs again, and verify that the file is now nowhere to be found.

```shell
zarah.dominguez@R5003334 swipe-to-refresh-demo (master) $ git log
commit a6c00638b3d466a61e3381a98e6b44cf2d085164
Author: Zarah Dominguez
Date:   Tue Jun 17 16:34:50 2014 +0800
    
    Removed dependency on ButterKnife.

commit e25c6862a79270921a24d6bf2a9eb07cc3f03b36
Author: Zarah Dominguez
Date:   Tue Jun 17 16:07:33 2014 +0800

    First commit
```

##### Now kill and bury that shit:

```shell
$ git branch -d fix-that-shit
```

##### You will then need to force-push your changes if you have a remote branch. Hence the DO NOT DO UNLESS YOU ARE THE ONLY ONE USING THE BRANCH.

```shell
$ git push master --force
```

I am sure there is a more concise way to do this, but doing the verbose solution here for posterity.

-----------

I cannot stress this enough. DO NOT DO THIS IF YOU ARE SHARING YOUR BRANCH WITH SOMEBODY ELSE. If you do, you automatically give them license to punch you in the face. (It can be a remote branch, as long as YOU ARE NOT SHARING IT WITH SOMEBODY ELSE)
{: .notice--danger}