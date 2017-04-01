---
layout: post
title: String formatting and Lint
date: '2016-09-17T02:24:00.002+10:00'
author: Zarah Dominguez
tags:
- strings
- android
modified_time: '2016-09-17T02:24:51.918+10:00'
thumbnail: https://4.bp.blogspot.com/--swGfBoDOD8/V9wWiOuB0rI/AAAAAAAAlqE/VgstkTVguzktiQjSvB2dUDyAH_WvaOhGwCLcB/s72-c/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.01.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-5199855446490160622
blogger_orig_url: http://www.zdominguez.com/2016/09/string-formatting-and-lint.html
---

One piece of advice that we keep hearing over and over is to extract strings into resources. There really is no reason for you to hard code strings in code.

I wrote before about [easily moving strings between Java and XML]({{ site.baseurl }}{% post_url 2015-05-14-stringy-strings %}), and today I'd like to focus on string formatting. The [Android dev guide](https://developer.android.com/guide/topics/resources/string-resource.html#FormattingAndStyling) gives a good overview of the support Android has for passing arguments into a `String.format(String, Object...)`.

Worried about using incorrect syntax? I was! I can never remember which of the % or the $ comes first, or if I'm passing arguments in the correct order. To be clear, the syntax is

```xml
%[arg number]$[arg type]
```

Thankfully, Android Studio has come a long way with pointing us in the right direction when working with strings.

First off, a very useful warning when setting manually concatenated text into a TextView:

<p style="text-align: center"><img src="https://4.bp.blogspot.com/--swGfBoDOD8/V9wWiOuB0rI/AAAAAAAAlqE/VgstkTVguzktiQjSvB2dUDyAH_WvaOhGwCLcB/s640/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.01.png"><img src="https://4.bp.blogspot.com/--swGfBoDOD8/V9wWiOuB0rI/AAAAAAAAlqE/VgstkTVguzktiQjSvB2dUDyAH_WvaOhGwCLcB/s1600/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.01.png">

 And by "placeholders" they mean the format arguments.

You can mix and match multiple arguments and argument types in one string too!

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-gJ_fchakE10/V9wZVRsZgdI/AAAAAAAAlqY/tKvSuTpGG4QceQ7jpOuaXUeikVM5cDMIQCLcB/s640/Screen%2BShot%2B2016-09-17%2Bat%2B02.09.36.png"><img src="https://3.bp.blogspot.com/-gJ_fchakE10/V9wZVRsZgdI/AAAAAAAAlqY/tKvSuTpGG4QceQ7jpOuaXUeikVM5cDMIQCLcB/s1600/Screen%2BShot%2B2016-09-17%2Bat%2B02.09.36.png">

And in case you removed an argument in the XML file but forgot to edit code, another helpful error for you!

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-yR9Qee97I1s/V9wWiFexQTI/AAAAAAAAlqI/_L0wZ6-bkN8u2T7PqJBLBFmOsbQjgbJZACLcB/s640/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.18.png"><img src="https://3.bp.blogspot.com/-yR9Qee97I1s/V9wWiFexQTI/AAAAAAAAlqI/_L0wZ6-bkN8u2T7PqJBLBFmOsbQjgbJZACLcB/s1600/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.18.png"></p>

If you change the argument type, Studio will tell you about it too.

<p style="text-align: center"><img src="https://4.bp.blogspot.com/-IJ11XzbvWpU/V9wWiukzpiI/AAAAAAAAlqQ/j82z8DdpJQAXqMZ-tqOyT3TVNtcPt4ceACLcB/s640/Screen%2BShot%2B2016-09-17%2Bat%2B01.55.12.png"><img src="https://4.bp.blogspot.com/-IJ11XzbvWpU/V9wWiukzpiI/AAAAAAAAlqQ/j82z8DdpJQAXqMZ-tqOyT3TVNtcPt4ceACLcB/s1600/Screen%2BShot%2B2016-09-17%2Bat%2B01.55.12.png"></p>

<p style="text-align: center">If you forget the syntax, fear not for Studio will tell you. (May not be too obvious here but I used the incorrect "_**<span style="color: red;">$</span>**_1$s" format.)</p>

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-7_uHOdyTDX8/V9wWiFigQBI/AAAAAAAAlqM/YvAMlNoEp6UjJTNJdjqapv9ETVRlHIPwACLcB/s640/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.34.png"><img src="https://3.bp.blogspot.com/-7_uHOdyTDX8/V9wWiFigQBI/AAAAAAAAlqM/YvAMlNoEp6UjJTNJdjqapv9ETVRlHIPwACLcB/s1600/Screen%2BShot%2B2016-09-17%2Bat%2B01.54.34.png">"</p>

Android Studio is really so helpful now, we are running out of excuses to be lazy coders. 😱