---
layout: post
title: "Bug Reports: A Story"
tags:
- android
---
"This tool sucks :facepunch::rage::rage::rage::rage:"

"Ugh WTF this is so stupid :poop::fire::fire::fire:"

These are just some of the things I have heard developers say over the years whilst working on Android. Sometimes they are trying to figure something out and there is not enough documentation, other times they are trying to make their tools do something and it just won't work. Oftentimes, it is because they are expecting something to happen and it does not.

I know it can be frustrating, I have been a dev for more than a 12 years now after all. But then again, no matter how much or how loudly you curse the Android gods, where does that get you? Does it fix the issue? Does it improve the situation? I do not think so.

So what should we do then?

#### My answer has, and always will be, "have you filed a bug for it?"

Android has its own dedicated issue tracker  organised into several components. [This page](https://source.android.com/setup/contribute/report-bugs) enumerates those components and have some very good tips on how and when to file a bug.

It is important to select the correct component because (1) it makes the relevant team aware of your issue, and (2) each component may have a template for reporting issues.
 
<p style="text-align: center"><a href="{{ site.baseurl }}/assets/bug_reports/components_filter.png"><img src="{{ site.baseurl }}/bug_reports/components_filter.png" width="478" height="315"></a><br />
<small>Android Studio component with template</small></p>

When writing up a bug report, think of what you, as a developer, would find useful if it were _you_ receiving the issue. _"This app sucks"_ is way less helpful than _"I have trouble logging in with my credentials that work on my computer"_.

I try to be as helpful to the developer as much as I can because we both have the same end-goal: **to fix a verified issue**.

It might be time-consuming, but when the bug you reported gets fixed and other devs start to notice and benefit from it, it is very rewarding. :green_heart:

<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">WOOOOW The error messages in Android Data Binding improved so much! SOOOOOOOO USEFUL right now 👏👏 Who should I thank <a href="https://twitter.com/yigitboyar?ref_src=twsrc%5Etfw">@yigitboyar</a> <a href="https://twitter.com/georgemount1?ref_src=twsrc%5Etfw">@georgemount1</a>? <a href="https://twitter.com/AndroidDev?ref_src=twsrc%5Etfw">@AndroidDev</a> <a href="https://twitter.com/hashtag/AndroidDev?src=hash&amp;ref_src=twsrc%5Etfw">#AndroidDev</a></p>&mdash; Bartek Lipinski (@blipinsk) <a href="https://twitter.com/blipinsk/status/1097832566512644096?ref_src=twsrc%5Etfw">February 19, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>
I might be totally wrong but I'd like to think [my bug report](https://issuetracker.google.com/issues/62685775) helped nudge this along :sweat_smile:  

<br />
<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">oh you can convert your root layout to a data binding layout in Android Studio(In case you are lazy as me) <a href="https://twitter.com/hashtag/AndroidDev?src=hash&amp;ref_src=twsrc%5Etfw">#AndroidDev</a> <a href="https://twitter.com/hashtag/Androidstudio?src=hash&amp;ref_src=twsrc%5Etfw">#Androidstudio</a> <a href="https://t.co/1M0HedNhXz">pic.twitter.com/1M0HedNhXz</a></p>&mdash; Stephan (@TheEarlOfEgo) <a href="https://twitter.com/TheEarlOfEgo/status/969138357338038272?ref_src=twsrc%5Etfw">March 1, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>
Sometimes, [my feature requests](https://issuetracker.google.com/issues/37136823) actually get built too! :dancer:

#### Do you want to get _your_ issues fixed too?

The trick is to make the bug report as great as possible. Let's take [this bug for example](https://issuetracker.google.com/issues/38213600). I encountered an issue whilst running "Remove Unused Resources" from Android Studio and wanted to report it.

To gather the relevant information about my machine, I opened Android Studio and then chose "About Android Studio". I then clicked on the copy icon in the dialogue box that appeared; that copies into my clipboard almost all the information asked for by the template!
<p style="text-align: center"><a href="{{ site.baseurl }}/assets/bug_reports/copy_info.png"><img src="{{ site.baseurl }}/bug_reports/copy_info.png" width="468" height="228"></a><br />
<small>Click that!</small></p>

I then provided a short overview of the reproduction steps; I try to follow the given-when-then pattern. Set the scene first to give the developers some context of what we are trying to achieve. In this case, I have provided a zip file of sample project that reproduces the error.

Next, I reported what I was trying to do and what happened as a result. If there are multiple steps that need to happen first, I enumerate them all in order. In this bug, I have also provided a screenshot so that the team can quickly verify that they have reproduced my error exactly as I see it.

And finally, I tried to provide as much as detail as possible. Were there scenarios where this does not happen?
> It looks like Studio detects layouts inflated through the generated binding file's `.inflate()` method as unused. Views inflated using DataBindingUtil are not removed. ([From here](https://issuetracker.google.com/issues/38213600#comment1))

If it is a feature request, what would I like to see as the end result? What are the benefits of fixing this issue?

> If I am beginning data binding, "user defined types" might be unfamiliar. ([From here](https://issuetracker.google.com/issues/62685775#comment1))

Are there any workarounds that I am aware of?
> Found another way to fix it. Remove the spaces before and after the arrow, as such:
`android:onClick="@{()->handlers.onSendNotificationClick()}"` ([From here](https://issuetracker.google.com/issues/37136809#comment2))

When I found a use case that was missed by the initial fix, I provided another sample project.

Really, a sample project is like a picture. It paints a thousand words.

#### A good bug report is like a good story.

It presents the characters, introduces conflict, and finally, a denouement.

Be a good storyteller and hold your audience captive. After all, it's the stories that make this world much more fun and interesting. :relaxed:


