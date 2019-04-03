---
layout: post
title: "Moving On Up (Or Down)"
tags:
- android
---
<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Using CMD+SHIFT+UP/DOWN when reordering element in an enum respects the semicolon. 😍 It&#39;s the small things! <a href="https://t.co/ct6y537G6a">pic.twitter.com/ct6y537G6a</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/1113242947460325376?ref_src=twsrc%5Etfw">April 3, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

I tweeted that this morning after rediscovering CMD+SHIFT+UP. I have known about this shortcut for a while, but I guess I haven't had reason to use it until lately.

And then I remembered a variation of this shortcut that would literally "move things don't care".

<center><blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">EDIT: (versus SHIFT+ALT+UP/DOWN which just moves the line regardless of context) <a href="https://t.co/zOtMUky9DC">pic.twitter.com/zOtMUky9DC</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/1113298274838929408?ref_src=twsrc%5Etfw">April 3, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

This being IntelliJ, these actions work on more than just properties. They work for basically anything in your code that has a scope -- individual lines, functions, classes even!

There are two main movement actions, _move statement_ and _move line_. Depending on what you want to do, each of these have their own benefits.

Moving a _statement_ means that the IDE will respect the context within which that particular element exists. In the case of a single line of code, this means maintaining its current scope. What this means exactly is -- if a line is inside a function, using CMD+SHIFT+UP/DOWN will *never* move it out of the function.

To wit:
<p style="text-align: center"><a href="{{ site.baseurl }}/assets/moving_up/cmd_shift_line.gif"><img src="{{ site.baseurl }}/assets/moving_up/cmd_shift_line.gif"></a><br /></p>

But this line wants to move to another function! It says so itself! This is when CMD+ALT+UP/DOWN comes in handy:

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/moving_up/cmd_alt_line_2.gif"><img src="{{ site.baseurl }}/assets/moving_up/cmd_alt_line_2.gif"></a><br /></p>

This shortcut is really handy when reordering or refactoring functions:

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/moving_up/cmd_shift_fun.gif"><img src="{{ site.baseurl }}/assets/moving_up/cmd_shift_fun.gif"></a><br /></p>

I leave it as an exercise for the reader to see how the shortcut behaves with classes. Get moving and stop copy-pasting! :dancers:


