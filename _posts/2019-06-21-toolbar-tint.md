---
layout: post
title: "Tintable Toolbar Things"
tags:
- android
---
A few weeks ago, I merged a pull request that updates our app's theme to Material Components [from the Bridge version](https://material.io/develop/android/docs/getting-started/).

<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">My team is SMASHING IT. 💪 <a href="https://t.co/YoJmRF1ZLJ">pic.twitter.com/YoJmRF1ZLJ</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/1135432513688465408?ref_src=twsrc%5Etfw">June 3, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

I have been referencing the [MDC Catalog](https://github.com/material-components/material-components-android/tree/master/catalog) app a lot during this exercise; and I thought it would be a good idea to pattern how we organise and apply our theming and styling based on that app. (Hey they must know what they're doing, right?)

We have two kinds of Toolbars in our app -- the first one with our standard primary colour, and a "clean" variant.

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/tintables/toolbar_types.jpg"><img src="{{ site.baseurl }}/assets/tintables/toolbar_types.jpg"  width="581" height="588"></p>

When we initially set up our theming, we made this theme overlay (read [about theme overlays here](https://medium.com/androiddevelopers/theming-with-appcompat-1a292b754b35)) to style our toolbar and give it that green arrow:
```xml
<style name="ThemeOverlay.Toolbar.Inverse" parent="ThemeOverlay.AppCompat.Light">
    <item name="colorControlNormal">?colorAccent</item>
    <item name="android:background">@color/white</item>
</style>
```

We then set this in our Toolbars as `android:theme`:
```xml
<androidx.appcompat.widget.Toolbar 
    ...
    android:theme="@style/ThemeOverlay.Toolbar.Inverse" />
```

The MDC Catalog app uses **_styles_** instead of **_themes_**, so I set about copying [their technique](https://github.com/material-components/material-components-android/blob/master/catalog/java/io/material/catalog/application/theme/res/values/styles.xml#L19). (Side note: Anita Singh has an [amazing talk on styles, themes, and material design](https://speakerdeck.com/anitas3791/styles-themes-material-theming-oh-my))

We immediately run into one problem: in our theme-based approach, we can set the `colorControlNormal` attribute in our theme overlay, but we cannot do that anymore with our style-based approach.

We definitely want our green arrow though, and we can look at the MDC source code to get some clues. It's always a good idea to start off with what we want to be our [parent style](https://github.com/material-components/material-components-android/blob/8f622283d18466620a280f6f6bbb32fafb157efd/lib/java/com/google/android/material/appbar/res/values/styles.xml#L65):
```xml
<style name="Widget.MaterialComponents.Toolbar.Surface">
    <item name="android:background">?attr/colorSurface</item>
    <item name="titleTextColor">?attr/colorOnSurfaceEmphasisHighType</item>
    <item name="subtitleTextColor">?attr/colorOnSurfaceEmphasisMedium</item>
    <!-- Note: this theme overlay will only work if the style is applied directly to a Toolbar. -->
    <item name="android:theme">@style/ThemeOverlay.MaterialComponents.Toolbar.Surface</item>
</style>
```
:thinking: Interesting note there, we should totally remember that.

Taking a peek at the [referenced theme overlay](https://github.com/material-components/material-components-android/blob/8f622283d18466620a280f6f6bbb32fafb157efd/lib/java/com/google/android/material/appbar/res/values/styles.xml#L78), we see the attribute we want :tada: :
```xml
<style name="ThemeOverlay.MaterialComponents.Toolbar.Surface" parent="">
    <item name="colorControlNormal">?attr/colorOnSurfaceEmphasisMedium</item>
    <item name="actionMenuTextColor">?attr/colorOnSurfaceEmphasisMedium</item>
  </style>
```

Adapting this to our code, we now have:
```xml
<style name="Widget.Toolbar.Inverse" parent="Widget.MaterialComponents.Toolbar.Surface">
    <item name="android:background">@color/white</item>
    <item name="android:theme">@style/ThemeOverlay.ToolbarTint</item>
</style>

<style name="ThemeOverlay.ToolbarTint" parent="ThemeOverlay.MaterialComponents.Toolbar.Surface">
    <item name="colorControlNormal">?colorSecondary</item>
</style>
```

And applying this to our Toolbar (remember it is now a **_style_** !):
```xml
 <androidx.appcompat.widget.Toolbar
    ...
    style="@style/Widget.Toolbar.Inverse"
    app:navigationIcon="?homeAsUpIndicator" />
```

And this works really well, until....
<p style="text-align: center"><a href="{{ site.baseurl }}/assets/tintables/no_x.png"><img src="{{ site.baseurl }}/assets/tintables/no_x.png" width="270" height="540"></a><br /><small>:scream:</small></p>

The only difference between this screen and the others is that we provide a vector drawable for the navigation icon:
```xml
<androidx.appcompat.widget.Toolbar
    ...
    style="@style/Widget.Toolbar.Inverse"
    app:navigationContentDescription="@string/close"
    app:navigationIcon="@drawable/ic_close" />
```

So it **should** work, right? MDC Catalog has a [style with a close button](https://github.com/material-components/material-components-android/blob/2de39fafe0285aab7e6e101549c4bc93f184a7e5/catalog/java/io/material/catalog/application/theme/res/values/styles.xml#L21) like we have, the vector they use also has a `fillColor` hardcoded in it (I even tried making their fill colour red :joy:), but it is always always getting tinted correctly.

I always find theming and styling really hard to debug, so I gave up on this for a while and conceded my defeat. The next day, I decided to timebox myself to a couple of hours -- if I can't figure it out by then I will just stick to how it was before. I re-read all the blog posts, re-reviewed the source code, and gave StackOverflow another shot.

And there... in [one random comment exchange](https://stackoverflow.com/questions/28219178/toolbar-icon-tinting-on-android#comment76399857_38650854):
>What's the difference between using ?attr/colorControlNormal at android:tint vs android:fillColor attributes? Thanks! – Thomas Vos Jun 20 '17 at 15:46 
@SuperThomasLab The example is from Chris Banes's blog where it illustrates replacing a tinted image with a tinted vector. Note that the fillColor was effectively hard-coded in the source image. I do not know if other color representations are now supported in AppCompat. – Joe Bowbeer Jun 22 '17 at 20:06

And sure enough, our vector didn't have `android:tint` defined in it! :woman_facepalming: Adding it into our vector makes everything work perfectly :ok_hand:
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24"
    android:tint="?colorControlNormal">
  <path
      android:fillColor="#FFFFFFFF"
      android:pathData="M19,6.41L17.59,5 12,10.59 6.41,5 5,6.41 10.59,12 5,17.59 6.41,19 12,13.41 17.59,19 19,17.59 13.41,12z"/>
</vector>
```

I know it may be obvious to some people (*of course* you need a tint to tint something, duh???), but it wasn't super obvious to me.

So, TL;DR: Make sure your vectors _actually_ support tinting before trying to tint it. :rainbow:
