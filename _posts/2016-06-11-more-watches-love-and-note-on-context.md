---
layout: post
title: More Watches Love and a note on Context
date: '2016-06-11T20:13:00.002+10:00'
author: Zarah Dominguez
tags:
- debugging
- android
modified_time: '2016-06-11T20:13:31.629+10:00'
thumbnail: https://3.bp.blogspot.com/-WobygbPFehA/V1vZ4JLtGuI/AAAAAAAAY6I/Kqw-ZF0PGFEWTZCbIl2Gg4L-DfGhhoDSgCLcB/s72-c/watches_context_optim.gif
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8281877386783469270
blogger_orig_url: http://www.zdominguez.com/2016/06/more-watches-love-and-note-on-context.html
---

I have [previously](http://droidista.blogspot.com.au/2016/02/squashing-bugs.html) [written](http://droidista.blogspot.com.au/2016/02/taking-closer-look-while-debugging.html) about debugging and how Watches can help make inspecting things in your code easier. Today, I would like to reiterate how powerful Watches can be.

When something fails in my code and I'm lucky, I would have a vague idea of what may be causing it. So I launch my app then attach the debugger ([NO! to the green bug!](https://twitter.com/zarahjutz/status/695549744983093248)); I am stopped at a breakpoint and while staring at the offending line, a "Wait, what if it's this other thing that's causing it?" pops into my head.

If you are used to debugging by peppering your code with `Log`s or `Timber`s or `Toast`s (and there's nothing wrong with that!!), you would have to stop debugging, add the logs, and re-launch your app (thank God for Instant Run!).

However, we can leverage Watches to make this process a whole lot easier. While stopped at a breakpoint, just add a new Watch (⌘+N or click the + in the Watches pane) and start typing.

Let's say for some reason I wanted to see what Intent extras, if any, had been passed into the current Activity. I just add a new Watch and put in the usual way of getting an Activity's extras -- `getIntent().getExtras()` and now I can inspect what I have been given when this Activity was launched. Easy peasy.

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-WobygbPFehA/V1vZ4JLtGuI/AAAAAAAAY6I/Kqw-ZF0PGFEWTZCbIl2Gg4L-DfGhhoDSgCLcB/s640/watches_context_optim.gif"></p>

Now in the IntelliJ documentation, there is a quote that says:

> Watch expressions are always evaluated in the context of a stack frame that is currently inspected in the Frames pane.

Similar to the way "context" is defined in the real world (and in the Android world too, I guess) this means that Watches will only make sense if they are in used in the correct circumstance where it can be fully understood. Still confused?

Say I am now stopped at a breakpoint in a Fragment (The Frames pane says I am in the method `getRandomSublist()` in `CheeseListFragment.java`) and I would like to see the extras passed into this Fragment's host Activity. The previously added Watch will not work since in our current context -- that of a Fragment -- there is no such method `getIntent()`. To do what I want, I would need to call the Activity first -- `getActivity().getIntent().getExtras()= null`, there are no extras received!).

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-2XoXjHpvHUk/V1vhlNmfItI/AAAAAAAAY6g/DbBBc0rXxvQpXUB2ZjTjuNr8v1_zYqaZACLcB/s640/Screen%2BShot%2B2016-06-11%2Bat%2B20.01.22.png"></p>

Pretty coooooool. I think in a sense, you can think of Watches as coding without actually coding. Happy debugging!
