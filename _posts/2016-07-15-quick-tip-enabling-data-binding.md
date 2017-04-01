---
layout: post
title: 'Quick tip: Enabling data binding'
date: '2016-07-15T16:24:00.001+10:00'
author: Zarah Dominguez
tags:
- data binding
- quick tips
- android
modified_time: '2016-07-15T16:31:14.075+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2979748123228591878
blogger_orig_url: http://www.zdominguez.com/2016/07/quick-tip-enabling-data-binding.html
---

Say you have a simple layout file:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:orientation="vertical">

        <TextView
            android:id="@+id/title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <TextView
            android:id="@+id/description"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

</LinearLayout>
```

And say you want to finally give data binding a try. So you watch [Yigit's I/O talk on data binding](https://youtu.be/DAmMN7m3wLU) and [follow the steps on implementation](https://youtu.be/DAmMN7m3wLU?t=28m57s).

You enable it in Gradle:

```shell
android {
    dataBinding {
        enabled = true
    }
}
```

You wrap your existing layout in `<layout>` tags.

```xml
<layout>
      <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:orientation="vertical">

        ...

      </LinearLayout>
<layout>
```

Then life started sucking. Your app won't compile. Well, we can fix it in two easy steps!

**Step 1:** Add namespace declaration to your root tag:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
```

This still won't compile! If you look at the logs you will see somewhere there: `"Error:(7) Error parsing XML: duplicate attribute"`

**Step 2:** Remove `xmlns` declaration in the now-non-root layout.

And that's it. Add your variables and happy data binding!

Here's a complete layout file for your perusal:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
      <data>
        <variable
            name="listing"
            type="com.zdominguez.blog.samples.Listing"/>
    </data>

      <LinearLayout 
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:orientation="vertical">

        <TextView
            android:id="@+id/title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{listing.address}" />

        <TextView
            android:id="@+id/description"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" 
            android:text="@{listing.agency}" />

      </LinearLayout>
<layout>
```