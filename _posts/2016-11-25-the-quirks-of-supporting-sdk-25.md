---
layout: post
title: The Quirks of Supporting SDK 25
date: '2016-11-25T16:40:00.002+11:00'
author: Zarah Dominguez
tags:
- android
- nougat
modified_time: '2016-11-25T17:38:08.326+11:00'
thumbnail: https://3.bp.blogspot.com/-cD11wUwTcyY/WDe565vxn2I/AAAAAAAAwmE/3wbmRsCm6wA29bLoZ-UhG33eNCxke1WUQCLcB/s72-c/edit_sdk_optim.gif
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-5520364401760820831
blogger_orig_url: http://www.zdominguez.com/2016/11/the-quirks-of-supporting-sdk-25.html
excerpt_separator: <!--more-->
---

The last developer preview of Android 7.1 [has started shipping](http://android-developers.blogspot.com.au/2016/11/final-update-to-android-7-1-developer-preview.html), which means APIs are (based on past experience) more or less stable. There is a [very good write up](https://developer.android.com/preview/api-overview.html) on developer.android.com on how to get started with supporting these new features. I set about trying it out, and here's what happened.

### Update SDK version

Let's start with the (supposedly) easy bit -- updating target SDK to 25\. It should be as easy as updating your `build.gradle` file, but just like when the API 24 sources became available, I found that I have to edit the SDK path before Studio starts to recognise the new APIs.

Go to Preferences > Appearance and Behavior > System Settings > Android SDK, click Edit beside Android SDK Location and just keep on clicking Next until you exit the wizard.

[![](https://3.bp.blogspot.com/-cD11wUwTcyY/WDe565vxn2I/AAAAAAAAwmE/3wbmRsCm6wA29bLoZ-UhG33eNCxke1WUQCLcB/s1600/edit_sdk_optim.gif)](https://3.bp.blogspot.com/-cD11wUwTcyY/WDe565vxn2I/AAAAAAAAwmE/3wbmRsCm6wA29bLoZ-UhG33eNCxke1WUQCLcB/s1600/edit_sdk_optim.gif)

### Circular Launcher Icons

The next (supposedly) easier bit is having a circular launcher icon. First stop is to generate the new icon. Android Studio has a built-in icon generator, in fact the documentation encourages you to use just that. So launch Asset Studio (CMD+SHIT+A then Asset Studio, or, right click in Project Pane > New > Image Asset).

Choose _Image_ as the Asset Type, then click the three dots to choose your existing icon. Now I know that pointing it to the current icon in `mipmap/` or `drawable/` should work but it didn' t for me. I had to copy the asset to somewhere else (in my case Desktop) and point the tool to there.

Next choose _Circle_ as the shape and voila, you have your asset. Not.

[![](https://3.bp.blogspot.com/-pd6MtPttY1M/WDe9zwjfszI/AAAAAAAAwmQ/S2IR44sQ798926pK2M6jdlD9NZHc6xrfACLcB/s320/Screen%2BShot%2B2016-11-25%2Bat%2B15.26.44.png)](https://3.bp.blogspot.com/-pd6MtPttY1M/WDe9zwjfszI/AAAAAAAAwmQ/S2IR44sQ798926pK2M6jdlD9NZHc6xrfACLcB/s1600/Screen%2BShot%2B2016-11-25%2Bat%2B15.26.44.png)

So it looks like the tool does not trim the icon to be a circle. It tries to, you can see the shape. I [filed a bug for it](https://code.google.com/p/android/issues/detail?id=227642), but I don't think it has been triaged yet.

In the meantime, you can use Roman Nurik's excellent [online Asset Studio](https://romannurik.github.io/AndroidAssetStudio/) to generate your icons.

[![](https://1.bp.blogspot.com/-17xJ_HDDOZA/WDfFf7-ZkHI/AAAAAAAAwmo/b0t6KGssjJMhVMLAO_lK1SUs237jWB6IACLcB/s320/Screen%2BShot%2B2016-11-12%2Bat%2B21.49.34.png)](https://1.bp.blogspot.com/-17xJ_HDDOZA/WDfFf7-ZkHI/AAAAAAAAwmo/b0t6KGssjJMhVMLAO_lK1SUs237jWB6IACLcB/s1600/Screen%2BShot%2B2016-11-12%2Bat%2B21.49.34.png)

### App Shortcuts

One of the biggest features on this version is app shortcuts (please stop calling it _force touch_, that's not ours).

The Developer site is [quite verbose](https://developer.android.com/preview/shortcuts.html) on how to implement app shortcuts. For an overview on what app shortcuts are and the different types (static and dynamic), head on over to the developer site right now and read the intro.

But in a nutshell, and I'm quoting here:

> Android 7.1 allows you to define shortcuts to specific actions in your app. These shortcuts can be displayed in a supported launcher, such as the one provided with Nexus and Pixel devices. Shortcuts let your users quickly start common or recommended tasks within your app.

#### Static shortcuts

Static shortcuts are simple enough. It is so simple in fact, that I doubted myself.

> <div dir="ltr" lang="en">Tried adding app shortcuts ([https://t.co/Z83J0a6j84](https://t.co/Z83J0a6j84)) today. It was suspiciously easy... Like "I must be doing something wrong" easy. 🤔 [pic.twitter.com/fb3Q5geEaN](https://t.co/fb3Q5geEaN)</div>
> 
> — Zarah Dominguez 🦉 (@zarahjutz) [November 12, 2016](https://twitter.com/zarahjutz/status/797444275373883392)

Basically [copy-paste stuff from the dev guide](https://developer.android.com/preview/shortcuts.html#static), tweak it to use your own app's Intents and it just works. To try it out, I re-used existing PNG icons from our app. But of course, we want to do it right, right?

So first up is to make the icons follow the [design guidelines](https://commondatastorage.googleapis.com/androiddevelopers/shareables/design/app-shortcuts-design-guidelines.pdf).
- icons should be 24dp x 24 dp
- should be centred in a circle that's 44dp wide
- circle should be in material grey `#F5F5F5`
- there should be a 2dp padding all around the circle

I needed to convert the icons to vectors, and I found [this SVG to vector converter](http://a-student.github.io/SvgToVectorDrawableConverter.Web/) to work best for my purposes. And by best I mean it doesn't mangle the circle, it doesn't lose any of the holes, and is as close as possible to the SVG input.

As always, I tried to define strings in `strings.xml`. This works well enough for the shortcut labels, but it strangely does not for the `shortcutId` attribute. And by "does not work" I mean your shortcuts will not appear at all.

#### Dynamic Shortcuts

App shortcuts can also be dynamic. For my use case, I wanted to expose a few more shortcuts for users who are logged in.

Again, the Developer site [has a very good guide](https://developer.android.com/preview/shortcuts.html#dynamic) on how to get started with including these shortcuts in your app. I highly recommend reading the [ShortcutManager Javadoc](https://developer.android.com/reference/android/content/pm/ShortcutManager.html), as there are more details there than is available in the guide.

A bunch of people have already written about how to implement shortcuts in your app, but I think what most don't mention is that if you need to build up a backstack for the Activity triggered by your shortcut, the docs recommend to use [`TaskStackBuilder`](https://developer.android.com/reference/android/app/TaskStackBuilder.html).

This applies to my use case, which is opening the user's Shortlist. In normal circumstances, users get to their shortlist from our main (search) screen. This means that once they are on their shortlist, pressing the back button will take them back to the main screen. We want to replicate this behaviour when they go through the shortcut, and `TaskStackBuilder` helps us do just that.

Sure you can just manually write out your own `Intents[]`, but remember to set the correct flags for the first Activity in your stack. It's something that's easy to forget, but if we create our `Intents` via `TaskStackBuilder`, it will automatically add the correct flags for us.

#### Reporting Usage

The docs exhort you to report using shortcuts regardless of how the users get there. The API name may be a bit misleading -- `reportShortcutUsed` -- but think of it as just adding analytics tracking for the screen. So please report usage of **_both_** static and dynamic shortcuts!

#### What's next?

There is so much more stuff to explore with app shortcuts. What happens when the users restore your app from a backup? What if they pinned a shortcut you removed in a new version? How do you re-arrange shortcuts?

I have yet to explore these, so if you have, what are the other quirkiness you encountered?

In the meantime, watch out for our new updates!

[![](https://3.bp.blogspot.com/-gJHQmPXQ8wU/WDfPwDnqvuI/AAAAAAAAwnA/2M6oryPkjLY2J2Fp4CXZyfjDqDtvkx8lgCLcB/s320/device-2016-11-25-164408.png)](https://3.bp.blogspot.com/-gJHQmPXQ8wU/WDfPwDnqvuI/AAAAAAAAwnA/2M6oryPkjLY2J2Fp4CXZyfjDqDtvkx8lgCLcB/s1600/device-2016-11-25-164408.png)
