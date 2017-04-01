---
layout: post
title: A Little UX Love Goes A Long Way
date: '2016-10-27T17:47:00.003+11:00'
author: Zarah Dominguez
tags:
- ux
- android
modified_time: '2016-10-27T17:52:30.609+11:00'
thumbnail: https://3.bp.blogspot.com/-iD6EmMlKl-A/WBGS6ukRoFI/AAAAAAAAsHI/AUh8IwHhPTwa2O5XNKhgEW9a8mWnMnHjQCLcB/s72-c/device-2016-10-27-163405.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-5292628407477799029
blogger_orig_url: http://www.zdominguez.com/2016/10/a-little-ux-love-goes-long-way.html
excerpt_separator: <!--more-->
---

Yesterday, my bank pushed a notification asking if I'm going to travel. Yes! I am! I filled up the form they asked me to fill up, and tapped _Submit_.
<!--more-->

And then I got this:

[![](https://3.bp.blogspot.com/-iD6EmMlKl-A/WBGS6ukRoFI/AAAAAAAAsHI/AUh8IwHhPTwa2O5XNKhgEW9a8mWnMnHjQCLcB/s320/device-2016-10-27-163405.png)](https://3.bp.blogspot.com/-iD6EmMlKl-A/WBGS6ukRoFI/AAAAAAAAsHI/AUh8IwHhPTwa2O5XNKhgEW9a8mWnMnHjQCLcB/s1600/device-2016-10-27-163405.png)

I try again, same error. I try again, the app crashes.

I know some of the devs for this app, so I messaged one of them. As any proper developer does, he asks me "_What version are you on?_"

I went to the app's settings, and I can't find it there. Hmmm. Opened their navigation drawer. Not there. Where is it?! Apparently, it's nowhere in the app. The dev told me to go to my device's settings, look for the app, and look for the version number.

I thought that was weird. I asked the dev why they don't have it, and he said "I can find you in our crash logs using your user ID. I can find your device, version number, user name. Users do not need the version number."

I have always put in the version number somewhere in all the apps I have worked on ever.  Am I doing it wrong? And so I shouted on Twitter:

> <div dir="ltr" lang="en">Hey, devs! Question: Do you have your app's version number visible anywhere inside your app? Why or why not?</div>
> 
> — Zarah Dominguez 🦉 (@zarahjutz) [October 27, 2016](https://twitter.com/zarahjutz/status/791449151976255488)

A bunch of people replied to me, all with some variation of "Yes". I needed the internet on my side today, thank you for heeding the call, Twitter friends.

I really don't understand why you would **_not_** want to put the version number in your app.

It's just one `TextView`. Use the tiniest, thinnest font you have. You don't even need to think about the code. It's all there, in `BuildConfig`.

If users care enough about your app to report issues, then that means they are really comfortable using it. Why would you kick them out of their comfort zone just to ask them for this piece of information that should be plastered on the wall? Not all users are as tech savvy as you, 23-year-old developer.

I can just imagine someone calling support:

> Customer: The application keeps on stopping.
> Support: What version of the app are you on?
> Customer: Where do I find that?
> Support: Open your device's settings....
> Customer: What? How?
> Support: What phone are you using?
> Customer: I don't know, my children gave this to me.

Remember, going to Settings > Applications may be different for each manufacturer. (I'm looking at you, Samsung). And it may not be as easy as telling someone "Go to device settings".

Having the version number is a good excuse to put in an easter egg. Why pass up this chance?

I guess what I want to make out of this is that something as inconsequential as this for you as a developer may be important to end users, or to other people who help you make your app better. It is bad enough that users encounter bugs so severe they feel like it's worth reporting. And surely we do not want to lose those users because simply by the act of reporting issues, they show us that they care about our product. The least we can do as developers is to make it as easy as possible for them to help us.
