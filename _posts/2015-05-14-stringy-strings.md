---
layout: post
title: Stringy strings
date: '2015-05-14T20:15:00.000+10:00'
author: Zarah Dominguez
tags:
  - android studio
  - strings
  - quick tips
  - android
modified_time: '2016-02-06T21:22:15.052+11:00'
thumbnail: https://2.bp.blogspot.com/-cM24MgClctQ/VVQTF5ITtiI/AAAAAAAAByM/xURNHX8TBFE/s72-c/animation.gif
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-7460169598181856247
blogger_orig_url: http://www.zdominguez.com/2015/05/stringy-strings.html
---

While we are on the subject of strings, here are more ways of dealing with them in Android Studio. We all know that we should not hardcode strings in code, right? But sometimes, we forget and tend to do code first before defining them in `strings.xml`.

There are a couple of ways that Android Studio/IntelliJ makes this easy for us. The gif below (which took me a while to figure out how to do, by the way), shows how to deal with:
1. Moving a hardcoded string into `strings.xml`
2. Giving a previously undefined string ID a value in `strings.xml`<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;"><tbody><tr><td style="text-align: center;"><a href="http://2.bp.blogspot.com/-cM24MgClctQ/VVQTF5ITtiI/AAAAAAAAByM/xURNHX8TBFE/s1600/animation.gif" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" height="203" src="https://2.bp.blogspot.com/-cM24MgClctQ/VVQTF5ITtiI/AAAAAAAAByM/xURNHX8TBFE/s400/animation.gif" width="400" /></a></td></tr><tr><td class="tr-caption" style="text-align: center;">(Click to embiggen)</td></tr></tbody></table>

#### Move hardcoded string into XML:  
This is probably the more common scenario. You happily put in texts into your `TextView`s, and now you have to copy and paste them into `strings.xml`. Don't! There is a shortcut for that.

Put your cursor somewhere in the string, press `ALT+Enter` to bring up the context menu, choose `Extract string resource` and give your string resource a name. This will create a new `<string name="my_string_name">My string value</string>` in `strings.xml`.

Studio magically also calls `getString(R.string.xxxxx)` for you. Neat, huh?

#### Make new string from ID:
This is for when you suddenly remember that you need a new string and want to sort of try to do it correctly so you type in the string ID. Only it hasn't been defined yet, so Studio complains. But it's fine. There is a shortcut for that.

Put your cursor somewhere in the as of yet undefined ID, press `ALT+Enter` to bring up the context menu, choose `Create string value for resource my_string_id`, and type in the actual string value you want. Again, this will create a new `<string name="my_string_id">My other string value</string>` in `strings.xml`.

Remember, in both of these cases, Studio will create the new strings in `strings.xml`, but you can modify it to put in whatever variant you want it to be in (for localisations, screen size support, etc).