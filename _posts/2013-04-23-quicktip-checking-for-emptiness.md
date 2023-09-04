---
layout: post
title: 'Quick Tip: Checking for emptiness'
date: '2013-04-23T13:37:00.001+10:00'
author: Zarah Dominguez
tags:
  - utils
  - quick tips
  - android
modified_time: '2013-04-23T13:37:32.611+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-3467355221618609723
blogger_orig_url: http://www.zdominguez.com/2013/04/quicktip-checking-for-emptiness.html
---

Always using `if(myString != null && myString.length() > 0)`? Use [`!TextUtils.isEmpty(myString)`](http://developer.android.com/reference/android/text/TextUtils.html#isEmpty%28java.lang.CharSequence%29) instead.

Check out the class for more utility methods that you won't have to implement yourself.