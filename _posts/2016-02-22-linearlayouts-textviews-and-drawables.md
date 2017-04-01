---
layout: post
title: LinearLayouts, TextViews and Drawables
date: '2016-02-22T18:21:00.003+11:00'
author: Zarah Dominguez
tags:
- android
- layout
modified_time: '2016-02-22T22:05:25.003+11:00'
thumbnail: https://3.bp.blogspot.com/-P6ZP6ilNZrU/VsquL6hzm9I/AAAAAAAAJTU/MSmwDetg27A/s72-c/hammerheadMMB29Qzarah.dominguez02222016174304.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-7829682970772302568
blogger_orig_url: http://www.zdominguez.com/2016/02/linearlayouts-textviews-and-drawables.html
---

I sent out a series of tweets today about `LinearLayout`s and unexpectedly, quite a few people like them. I decided to get off my lazy ass and actually write it down in a post for easy reference.

Let's start with the `LinearLayout` root view.  One of the most common things designers ask us to do is to put dividers in. Quick! Think! What should we do? Add a generic `View` for each divider? How about NO? Instead, we can delegate the task of displaying these dividers to the `LinearLayout` itself:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:orientation="vertical"
              android:layout_height="match_parent"
              android:divider="@drawable/divider_horizontal_dark"
              android:showDividers="middle">
```

Where `divider_horizontal_dark` can be anything you want to be the divider (I recommend using a `shape`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle" >
    <solid android:color="#24000000" />
    <size android:height="1dp" />
</shape>
```

Doing this prevents us from littering our view hierarchy with useless empty views. `LinearLayout` is actually extremely powerful, and a lot of the stuff most apps usually need is already built in. PS: For some reason I cannot find the `showDividers` attribute in the Android docs, but the available attributes are `middle`, `beginning`, `end`, `none` (or a combination of those). **EDIT**: [+Nick Butcher](https://plus.google.com/118292708268361843293) showed us the light: [docs here](https://developer.android.com/intl/es/reference/android/R.attr.html#showDividers)!

**EDIT AGAIN**: Nick has also lovingly pointed out that `setShowDividers()` was added in API 11. If by some cruel twist of fate you need to support anything below that, use [`LinearLayoutCompat`](https://developer.android.com/intl/es/reference/android/support/v7/widget/LinearLayoutCompat.html#setShowDividers(int)). Also, you poor, poor thing.

Another common thing that we are asked to do is to have an image +  text displayed side by side. We use this quite a bit in the Domain app, most notably in the main navigation drawer:

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-P6ZP6ilNZrU/VsquL6hzm9I/AAAAAAAAJTU/MSmwDetg27A/s640/hammerheadMMB29Qzarah.dominguez02222016174304.png"></p>

Instead of having one huge `RelativeLayout` or (horror!) nested `LinearLayout`s, we can just have a bunch of `TextView`s in one `LinearLayout`.

You see, `TextView`s have this magical ability to add `Drawable`s to themselves. You can position the `Drawable` anywhere you want, and even tint it from XML! :gasp:

Drawable placement can be in any of the [cardinal directions](http://developer.android.com/intl/es/reference/android/widget/TextView.html#attr_android:drawableBottom) (`bottom`, `top`, `left`, `right`) and the tint should be a defined colour in XML (not actually required, but encouraged. By me. I encourage it.). If the `Drawable` is too close to the text for your or your designer's taste, you can adjust the distance via the `drawablePadding` attribute.

So here's a screen of four `TextView`s in one `LinearLayout`:

<p style="text-align: center"><img src="https://3.bp.blogspot.com/-vSmDgK2T03k/VsqtnOhWv5I/AAAAAAAAJTA/D8Sw8AUlZdU/s640/hammerheadMMB29Qzarah.dominguez02222016173932.png"></p>

And the full code (also in [github](https://github.com/zmdominguez/sdk_sandbox/blob/master/app/src/main/res/layout/activity_linear_layout.xml)):

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:orientation="vertical"
              android:layout_height="match_parent"
              android:divider="@drawable/divider_horizontal_dark"
              android:showDividers="middle">

    <TextView
        android:id="@+id/text1"
        android:drawableTop="@drawable/ic_notifications_black_24dp"
        android:drawableTint="@color/lemongrab"
        android:drawablePadding="8dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:padding="16dp"
        android:text="Text 1"/>

    <TextView
        android:id="@+id/text2"
        android:textSize="20sp"
        android:drawableRight="@drawable/ic_notifications_black_24dp"
        android:drawablePadding="8dp"
        android:padding="16dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Text 2"/>

    <TextView
        android:id="@+id/text3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:drawableLeft="@drawable/ic_notifications_black_24dp"
        android:drawablePadding="8dp"
        android:padding="16dp"
        android:text="Text 3"
        android:textSize="20sp"/>

    <TextView
        android:id="@+id/text4"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:drawableLeft="@drawable/ic_notifications_black_24dp"
        android:drawablePadding="8dp"
        android:padding="16dp"
        android:text="Text 4"
        android:textSize="20sp" />

</LinearLayout>
```

If you need to update the `Drawable` tint at runtime (if you are basing it on some status field, for example), you can do so via code:

```java
// left, top, right, and bottom
DrawableCompat.setTint(mText3.getCompoundDrawables()[0].mutate(), ContextCompat.getColor(this, R.color.red));
```

Note that we need to call `mutate()` or else the tint will be everywhere and it's gonna be a mess!

So there you have it! Remember kids, #perfmatters!