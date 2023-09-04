---
layout: post
title: Raising Activities From the Dead
date: '2015-09-10T00:58:00.002+10:00'
author: Zarah Dominguez
tags:
  - android studio
  - debugging
  - android
modified_time: '2015-09-10T00:58:49.747+10:00'
thumbnail: http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s72-c/hammerheadLMY48Izarah.dominguez09102015001455.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-7618422738396956756
blogger_orig_url: http://www.zdominguez.com/2015/09/raising-activities-from-dead.html
---

One of the scenarios I admittedly ~almost~ always forget to test is "What happens when my app goes into the background, then the OS kills is to claim memory, then I try to resume?" Usually it's "Well, I handle `onSavedInstanceState` not being `null`, so I am great!" It is fine and dandy for simple apps; but once your `Activity` or `Fragment` gets beefier and you start relying on state for more and more things, it can get complicated pretty quickly (In my case, the `Fragment` has `setRetainInstance(true)`).

This scenario in particular is kind of hard to reproduce willingly. I usually see this when I leave an app running, make my phone do some heavy work overnight, then resume the app the next day.

<div class="separator" style="clear: both; text-align: center;"><a href="http://i.imgur.com/7dVFVGN.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://i.imgur.com/7dVFVGN.jpg" height="179" width="320" /></a></div>

<div class="separator" style="clear: both; text-align: left;"><br /></div>

<div class="separator" style="clear: both; text-align: left;">So what you gonna do?</div>

<div class="separator" style="clear: both; text-align: left;"><br /></div>

It turns out that Android Studio has the answer! There is this magical tiny red button that allows you to simulate this exact scenario.

1. Open your app to the `Activity` you want to test (I use a very simple app here just for demo).

<div class="separator" style="clear: both; text-align: center;"><a href="http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s1600/hammerheadLMY48Izarah.dominguez09102015001455.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><br /></a><a href="http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s1600/hammerheadLMY48Izarah.dominguez09102015001455.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><br /></a><a href="http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s1600/hammerheadLMY48Izarah.dominguez09102015001455.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"></a><a href="http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s1600/hammerheadLMY48Izarah.dominguez09102015001455.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="320" src="http://1.bp.blogspot.com/-w61yhhV8d6k/VfBEfm11T4I/AAAAAAAAF7w/moIA4PvNOIg/s320/hammerheadLMY48Izarah.dominguez09102015001455.png" width="180" /></a></div><div class="separator" style="clear: both; text-align: center;"><br /></div><div class="separator" style="clear: both; text-align: left;">

2. In Studio, go to Android Monitor (make sure that your app is selected). Note the process ID, in this case it is `25647`.<div class="separator" style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-X2AnHYfPXTY/VfBEc4MoWQI/AAAAAAAAF7Y/w-0wwfDyDKc/s1600/Screen%2BShot%2B2015-09-10%2Bat%2B00.11.48.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="94" src="http://3.bp.blogspot.com/-X2AnHYfPXTY/VfBEc4MoWQI/AAAAAAAAF7Y/w-0wwfDyDKc/s320/Screen%2BShot%2B2015-09-10%2Bat%2B00.11.48.png" width="320" /></a></div>

3. Push your app to the background. Pressing the HOME button should be sufficient. This will call `onSaveInstanceState`, which is all that matters really. It is after all what we want to test.
4. Back in Studio, press the magical tiny red button pointed to in the previous image. Notice that Studio now appends `[DEAD]` to your app's process. It is now gone. He's dead, Jim!<br /><div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-CCviehy3qVE/VfBEc_vmXAI/AAAAAAAAF7c/QiC8evaSO9I/s1600/Screen%2BShot%2B2015-09-10%2Bat%2B00.12.08.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="88" src="http://2.bp.blogspot.com/-CCviehy3qVE/VfBEc_vmXAI/AAAAAAAAF7c/QiC8evaSO9I/s320/Screen%2BShot%2B2015-09-10%2Bat%2B00.12.08.png" width="320" /></a></div>
5. Resume your app. I usually just do this via recent apps.<br /><div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-Kuhm-cSXaXQ/VfBEiJgwDKI/AAAAAAAAF74/2jwjDtNyXts/s1600/hammerheadLMY48Izarah.dominguez09102015001520.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="320" src="http://2.bp.blogspot.com/-Kuhm-cSXaXQ/VfBEiJgwDKI/AAAAAAAAF74/2jwjDtNyXts/s320/hammerheadLMY48Izarah.dominguez09102015001520.png" width="180" /></a></div>
6. If you look at Studio, you'll see that your app is now no longer dead, but has a new process ID, in this case `26742`.<br /><div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-4yDmX-81Ng4/VfBEdJHWM_I/AAAAAAAAF7g/da-vNK_Lndo/s1600/Screen%2BShot%2B2015-09-10%2Bat%2B00.12.24.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="87" src="http://2.bp.blogspot.com/-4yDmX-81Ng4/VfBEdJHWM_I/AAAAAAAAF7g/da-vNK_Lndo/s320/Screen%2BShot%2B2015-09-10%2Bat%2B00.12.24.png" width="320" /></a></div>

If at this point you step through your code, you will notice that your `Activity` will go through the whole (re-)creation process with the `Bundle` given the values you have saved in `onSaveInstanceState`. No more waiting overnight, yay!