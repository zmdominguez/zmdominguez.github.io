---
layout: post
title: Pidcat <3 Android M
date: '2016-09-06T14:28:00.000+10:00'
author: Zarah Dominguez
tags:
- tools
- android
modified_time: '2016-09-06T14:28:27.662+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-6813372916199651844
blogger_orig_url: http://www.zdominguez.com/2016/09/pidcat-android-m.html
---

If you use [Pidcat](https://github.com/JakeWharton/pidcat), there might be issues running the tool if your device is on M+. The issue has been [fixed on master](https://github.com/JakeWharton/pidcat/issues/117) but hasn't been released yet.

The readme says to fix the issue, get the latest off master:

```shell
12:46 $ brew unlink pidcat
Unlinking /usr/local/Cellar/pidcat/HEAD... 1 symlinks removed
✔ ~/Android 
12:48 $ brew install --HEAD pidcat
Warning: pidcat-HEAD already installed, it's just not linked
✔ ~/Android 
```

That didn't quite work, so let's link it:

```shell
12:48 $ brew link pidcat
Linking /usr/local/Cellar/pidcat/HEAD... 1 symlinks created
✔ ~/Android 
```

I tried re-running pidcat, but it's still not displaying anything. Which probably means we really don't have the latest code off the repo. The `--force` switch has been deprecated, so to force an update we would have to call `reinstall` instead:

```shell
12:49 $ brew reinstall --HEAD pidcat
==> Reinstalling pidcat
==> Cloning https://github.com/JakeWharton/pidcat.git
Cloning into '/Library/Caches/Homebrew/pidcat--git'...
remote: Counting objects: 10, done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 10 (delta 0), reused 7 (delta 0), pack-reused 0
Unpacking objects: 100% (10/10), done.
Checking connectivity... done.
==> Checking out branch master
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d
==> Summary
🍺  /usr/local/Cellar/pidcat/HEAD: 5 files, 19.5K, built in 4 seconds
✔ ~/Android
```

Or, if do not want to do all that, you can also do:

```shell
$ adb logcat -v brief | pidcat my.package.name```