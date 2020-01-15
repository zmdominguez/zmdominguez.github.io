---
layout: post
title: "Which is Which: Named Breakpoints"
tags:
- android studio
- tools
---
I have always believed that one of the biggest factors that influence a person's enjoyment and delight in doing their job are the tools. Having the right tools and using them the best way possible helps direct our energy on the _what_ rather than the _how_.

It is for this reason that I have [written](https://zarah.dev/2016/02/05/squashing-bugs.html) and [spoken](https://www.youtube.com/watch?v=gjHY7jl4rrA) about making the most out of Android Studio a few times over the past few years.

I have talked about the debugging tools within Android Studio and how we can better utilise breakpoints. Today, I'm hammering the point in even more with described breakpoints!

I have lost count of how many times I stopped in the middle of a debugging session only to find myself drowning in breakpoints. I have put in so many that I have lost track of which breakpoint does what, or why I even put a breakpoint on a particular line in the first place.

<center>
    <a href="https://imgur.com/vmkOFZm"><img src="https://i.imgur.com/vmkOFZm.png" title="source: imgur.com" /></a>
    <br />
    <small>What are even these lines</small>
</center>

It is situations like this that breakpoint descriptions become super handy. These descriptions appear after the line number and provide an easy way of keeping track AND searching for breakpoints.

For example, adding "drag dismiss" after the breakpoint on line 84 serves as a short note to myself what that piece of code is doing. It gives me a hint about what I may want to do with that breakpoint when I'm grouping, muting, cleaning up, or managing all the other breakpoints.

<center>
    <a href="https://imgur.com/NUkFHvM"><img src="https://i.imgur.com/NUkFHvM.png" title="source: imgur.com" /></a>
    <br />
    <small>Breakpoint with a description</small>
</center>

To add a description, open the Breakpoints Dialog.

First open the Edit Breakpoint dialog using any of the following:
- pressing `ALT + ENTER` on any line with a breakpoint and then choosing "Edit Breakpoint"
- pressing `SHIFT+COMMAND+F8` on any line with a breakpoint
- right-clicking on a breakpoint in the gutter
<center>
    <a href="https://imgur.com/Y9sLNlV"><img src="https://i.imgur.com/Y9sLNlV.png" title="source: imgur.com" /></a>
    <br /><small>Edit Breakpoint dialog</small>
</center>

We can then click on More or press `SHIFT+COMMAND+F8` to open the Breakpoints Dialog. Right click on the breakpoint in question and choose Edit Description:  
<center>
    <a href="https://imgur.com/AaBDnWo"><img src="https://i.imgur.com/AaBDnWo.png" title="source: imgur.com" /></a>
    <br /><small>Adding a description</small>
</center>

This is admittedly an over-simplified example (in my main project I have so many breakpoints accumulated over so many debugging sessions and I'm afraid to remove _any_ of them. Don't @ me.), but finding a specific breakpoint is now easy as!

<center>
    <a href="https://imgur.com/7XpgQv4"><img src="https://i.imgur.com/7XpgQv4.gif" title="source: imgur.com" /></a>
    <br /> <small>Search through all breakpoints easily</small>
</center>


