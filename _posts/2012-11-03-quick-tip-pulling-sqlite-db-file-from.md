---
layout: post
title: 'Quick Tip: Pulling an SQLite db file from device'
date: '2012-11-03T04:49:00.001+11:00'
author: Zarah Dominguez
tags:
  - debugging
  - quick tips
  - android
modified_time: '2012-11-06T21:25:38.624+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8449164210541891222
blogger_orig_url: http://www.zdominguez.com/2012/11/quick-tip-pulling-sqlite-db-file-from.html
---

I have always thought that you would need root access to pull an SQLite file from a non-rooted Android device. Turns out I thought wrong! Here's how you do it:
```shell
$ adb -d shell
$ run-as your.package.name
$ cat /data/data/your.package.name/databases/yourdatabasename  >/sdcard/yourdatabasename
```

This will copy your app's SQLite db file to the SD card's root directory. From there, you can copy the file to your computer any way you like.

Props to this [SO answer](http://stackoverflow.com/questions/6928849/debugging-sqlite-database-on-the-device)!