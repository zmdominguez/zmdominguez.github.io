---
layout: post
title: Inspecting your Shared Preferences
date: '2012-05-17T15:20:00.001+10:00'
author: Zarah Dominguez
tags:
  - shared preferences
  - debugging
  - android
modified_time: '2012-05-18T17:49:21.425+10:00'
thumbnail: http://1.bp.blogspot.com/-R6ocgn09tYo/T7SKg4fcR4I/AAAAAAAABDQ/9qvbJKc8waE/s72-c/download_prefs.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-1983220554965120211
blogger_orig_url: http://www.zdominguez.com/2012/05/inspecting-your-shared-preferences.html
---

Did you know that you can look at your SharedPreferences file?

If during development you want to inspect what your SharedPreferences now contain, you can pull a copy of the XML file from the emulator.

In Eclipse, you can do that by following these steps:

- Open the DDMS perspective (Window > Open Perspective > Other > DDMS).
- Click on the File Explorer tab and navigate to your app's `data` folder (that's under data/data/<your package name>/shared_prefs/).
- Choose the file you want to download.
- Click on the Pull file from device button.

<div class="separator" style="clear: both; text-align: center;"><a href="http://1.bp.blogspot.com/-R6ocgn09tYo/T7SKg4fcR4I/AAAAAAAABDQ/9qvbJKc8waE/s1600/download_prefs.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="125" src="http://1.bp.blogspot.com/-R6ocgn09tYo/T7SKg4fcR4I/AAAAAAAABDQ/9qvbJKc8waE/s400/download_prefs.png" width="400" /></a></div>