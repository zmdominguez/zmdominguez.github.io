---
layout: post
title: Squashing Bugs
date: '2016-02-05T22:35:00.003+11:00'
author: Zarah Dominguez
tags:
- android studio
- debugging
- android
modified_time: '2016-02-05T22:35:33.211+11:00'
thumbnail: https://1.bp.blogspot.com/-KAg55fLsCqs/VrSFhWmIwDI/AAAAAAAAIcg/T_KDZiZILOI/s72-c/java_exceptions.gif
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-3515493845604849417
blogger_orig_url: http://www.zdominguez.com/2016/02/squashing-bugs.html
---

This has been one hell of a busy week for me. I think you can sort of tell from my Tweets and G+ posts that I have been debugging A LOT.

I was helping the new guy on our team look at something, and I think I almost gave him a heart attack.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Scared new guy when I yelled &quot;NOOO!&quot; as he was about to click green bug. Run then attach when debugging. <a href="https://twitter.com/hashtag/AndroidDev?src=hash">#AndroidDev</a> <a href="https://t.co/cFUNKDu0f1">pic.twitter.com/cFUNKDu0f1</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/695549744983093248">February 5, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

It takes FOREVER to launch the app when you click that green bug. I hate that green bug. Save yourself some heartache:

1.  Put in your breakpoints
2.  Launch the app as you normally would
3.  Navigate to the offending activity
4.  `Attach Debugger to Android Process`

App encountering what looks like random crashing? Put in some exceptionally useful Exception Breakpoints! In the Breakpoints dialog (⌘+⇧+F8 / `CMD+SHIFT+F8`), click on the +, choose Java Exception Breakpoints and add the exception you are interested in.

<p style="text-align: center"><img src="https://1.bp.blogspot.com/-KAg55fLsCqs/VrSFhWmIwDI/AAAAAAAAIcg/T_KDZiZILOI/s640/java_exceptions.gif"></p>

This means that even without actual breakpoints -- as long as the debugger is attached to the process -- Android Studio will suspend the process on the exact line that will throw the exception. Very easy way of narrowing down on the root cause!

But what if (God forbid!) you uncover another issue while debugging? Now you have a ton of breakpoints but you don't want to remove them because what if you still haven't fixed that other thing and this line is really important but you don't want to stop _all the time_. Ugh. It's a mess!

Make some semblance of order out of the chaos. Group your breakpoints. A breakpoint group can contain any number of any kind of breakpoint that Android Studio supports. This allows you to mute/unmute a set of breakpoints without having to hunt them down one by one.

Just choose the interesting breakpoints, right click, then choose Move to group. From here you can either create a new group or add them to an existing group. You can configure each breakpoint to behave how you want them to: suspend the thread, log a message, hit and forget, etc.

Here's the whole thing, in one magnificent gif.

<p style="text-align: center"><img src="https://4.bp.blogspot.com/-Gg8fdDdMEh0/VrR9yBs_Y6I/AAAAAAAAIcU/r7poki-Ne6U/s640/breakpoint_groups.gif"></p>

Again, apologies to [+Nick Butcher](https://plus.google.com/118292708268361843293). I owe you a beer next time you're in Sydney, Nick.