---
layout: post
title: Save Logcat contents to file
date: '2012-07-19T12:30:00.000+10:00'
author: Zarah Dominguez
tags:
  - logcat
  - adb
  - quick tips
  - android
modified_time: '2012-07-19T12:30:20.947+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8574610583883851666
blogger_orig_url: http://www.zdominguez.com/2012/07/save-logcat-contents-to-file.html
---

Note to self: to save the contents of Logcat to a text file:

- Navigate to the SDK installation directory.
- Go to the /platform-tools folder.
```shell
adb logcat -d > my_logcat_dump.txt
```

If there is more than one device connected to adb, specify which device's log to dump:
```shell
adb -s emulator-5558 logcat -d > my_logcat_dump.txt
```
