---
layout: post
title: Snazzy git blaming
date: '2016-06-22T17:22:00.002+10:00'
author: Zarah Dominguez
tags:
- android studio
- git
modified_time: '2016-06-22T17:22:51.463+10:00'
thumbnail: https://1.bp.blogspot.com/-W_1Xg2Aj98c/V2o02kjqM1I/AAAAAAAAaa0/kpLaEMQSbsYO6TTpuvsBOTB2EBGerIbbACK4B/s72-c/git_annotations.gif
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4436754458423140081
blogger_orig_url: http://www.zdominguez.com/2016/06/snazzy-git-blaming.html
---
Sometimes, you can't help it. You need to look at what happened in the past to understand what is happening in the present (wow).

If you know the specific line you are interested in, showing history for selection is the way to go.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Change can be confusing. Filter history to actual piece of code you&#39;re interested in. <a href="https://twitter.com/hashtag/AndroidDev?src=hash">#AndroidDev</a> <a href="http://t.co/nd27tgv20f">pic.twitter.com/nd27tgv20f</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/625992332735725568">July 28, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

However, if you want to look at how the file has changed over time, showing annotations (the nice way of saying `git blame`) might help you. To enable it, right click the gutter then choose Annotations.

From here, you have a ton of options on how you want to display the summary in the Annotations gutter. It's always fun to see which line has been there since the beginning of time.

<p style="text-align: center"><img src="https://1.bp.blogspot.com/-W_1Xg2Aj98c/V2o02kjqM1I/AAAAAAAAaa0/kpLaEMQSbsYO6TTpuvsBOTB2EBGerIbbACK4B/s640/git_annotations.gif"></p>

<span style="font-size: x-small;">** Source code is Roman Nurik's Muzei that we forked for an [Innovation Day project last year](http://tech.domain.com.au/2015/09/innovation-day-august-2015-domain-device-wall/).</span>

The most recent version of the file is marked with an asterisk, like so:

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-Pin6vgKqbPQ/V2o4UWHJyGI/AAAAAAAAabg/yurn_LPBEwQTIzv94nTdrvAj64Omt4x1gCK4B/s640/Screen%2BShot%2B2016-06-22%2Bat%2B17.02.58.png"></p>

Hovering over an annotation will show the full commit information, including the commit message.

<p style="text-align: center"><img src="https://1.bp.blogspot.com/-VzYyE2td6X0/V2o5OVxLDGI/AAAAAAAAabs/MSIogEPXgIMRL7WpW1zWMNkYuou5m_NOQCK4B/s640/Screen%2BShot%2B2016-06-22%2Bat%2B17.04.49.png"></p>

 And clicking on it will open a list of the paths affected by this particular revision.

<p style="text-align: center"><img src="https://4.bp.blogspot.com/-_dhUlen0wbk/V2o5QhUlBAI/AAAAAAAAab0/XFfVBGBXU28cH7i4zi09Ic47PJEu27suQCK4B/s640/Screen%2BShot%2B2016-06-22%2Bat%2B17.07.16.png"></p>

Look at all those options at the top of the dialog! Have a play and enjoy diff-ing!
