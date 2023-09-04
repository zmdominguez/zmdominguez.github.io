---
layout: post
title: Styling tab selectors for ActionBarSherlock
date: '2012-12-28T02:12:00.000+11:00'
author: Zarah Dominguez
tags:
  - viewpagerindicator
  - actionbarsherlock
  - style
  - layout
modified_time: '2012-12-28T02:19:24.646+11:00'
thumbnail: http://2.bp.blogspot.com/-i_NyU9nRePA/UNxdOdGXatI/AAAAAAAABE8/IemfSgPtQwI/s72-c/device-2012-12-27-222032.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2524435053378956031
blogger_orig_url: http://www.zdominguez.com/2012/12/styling-tab-selectors-for.html
---

This post builds on the [previous post for ABS&nbsp;+ VPI](http://droidista.blogspot.com/2012/08/making-actionbarsherlock-and.html).

I have gotten a lot of questions on styling the ABS, specifically the tab selected indicators. This task may seem daunting, but in reality it is relatively simple. I am also quite confused why I didn't find any straightforward tutorials when I tried googling this. Anyway, this post will hopefully help developers who aim to style their action bars.

We will need to use Jeff Gilfelt's [awesome styling tool](http://jgilfelt.github.com/android-actionbarstylegenerator/). To use it, just input in your app's color scheme for the highlights and what-not then download the generated zip file. If you want to just change the tab underline color, change the value under "Accent color". To make the change obvious from the default settings, I have used a shade of orange for the tab selected indicators. [Here are the settings I used for this demo](http://jgilfelt.github.com/android-actionbarstylegenerator/#name=styling_abs&amp;compat=sherlock&amp;theme=light&amp;actionbarstyle=transparent&amp;backColor=e4e4e4%2C100&amp;secondaryColor=d6d6d6%2C100&amp;tertiaryColor=F2F2F2%2C100&amp;accentColor=ff4d00%2C100).

Jeff's tool conveniently generates all the XML files and drawables and organizes them into the correct `/res/drawable` folders. Unzip the file and just drag the generated files into your project, or merge the contents if you already have such existing files.

See how easy it was? Seriously, we should all lie prostrate at Jeff Gilfelt's and Jake Wharton's feet. These guys are AWESOME.

The two photos below show the difference between the old app and the styled app. As you can see, I only changed the tab selected color. You can explore what happens if you use the other styles you can get from Jeff's tool.

| Default                                                                                                                                                                                                                                                                                                                                                  | Styled |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|
| <a href="http://2.bp.blogspot.com/-i_NyU9nRePA/UNxdOdGXatI/AAAAAAAABE8/IemfSgPtQwI/s1600/device-2012-12-27-222032.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://2.bp.blogspot.com/-i_NyU9nRePA/UNxdOdGXatI/AAAAAAAABE8/IemfSgPtQwI/s1600/device-2012-12-27-222032.png" height="200" width="112" /></a> | <a href="http://2.bp.blogspot.com/-9BTaJnt5Uy8/UNxdP3L7DbI/AAAAAAAABFE/vydg5l7AWyA/s1600/device-2012-12-27-222613.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://2.bp.blogspot.com/-9BTaJnt5Uy8/UNxdP3L7DbI/AAAAAAAABFE/vydg5l7AWyA/s1600/device-2012-12-27-222613.png" height="200" width="112" /></a> |

The hard work is done, let's now try to understand the automagically generated configurations we did.

First, we have to configure the theme to use custom tab indicators. We do this by changing the tab styles in `themes.xml`. In my sample app, these lines customize the tabs we are using.
```xml
<!-- Styling the tabs -->
<style name="CustomTabPageIndicator" parent="Widget.TabPageIndicator">
    <!-- Tab text -->   
    <item name="android:textSize">15dip</item>
    <item name="android:textColor">#FFB33E3E</item>
    
    <!--  Lines under tabs -->
    <item name="background">@drawable/tab_indicator_ab_styling_abs</item>  
    <item name="android:background">@drawable/tab_indicator_ab_styling_abs</item>
</style>
```

The "drawable" tab_indicator_ab_styling_abs is actually a [state list drawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList) that provides different images for each state of a tab -- focused and non-focused, pressed and not pressed.

You can see part of this in action in the image above. Try long pressing on a tab and you can see the tab's background change to use the unselected-pressed drawable.

Again, HUGE thanks to Jeff Gilfelt and Jake Wharton for creating tools that make our lives easier. :)

I have pushed a [new branch in github](https://github.com/zmdominguez/vpi-abs-demo/tree/styling_abs) that uses this style. The master branch of that repo still uses the default tab selectors.