---
layout: post
title: "Referencing IDs in Data Binding"
tags:
- android
- data binding
---
Last week, I was talking to someone on my team and it became apparent that they weren't aware of one super useful feature of data binding. If you know me at all, you know that I ~~like~~ love this library, and I would take every opportunity to spread the love around.

Let's take a very simple example -- say we have a `CheckBox` and a `TextView` in a layout file, and we want the `TextView` to display if the `CheckBox` is checked or not. I made this adorable gif to illustrate:
<p style="text-align: center"><a href="Cheeky!"><img src="{{ site.baseurl }}/assets/databinding_id_ref/cheeky_checkbox.gif" ></a><br />
<small>Cheeky!</small></p>  

The corresponding layout file is pretty straightforward:
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>

    </data>
    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <CheckBox
            android:id="@+id/cheeky_checkbox"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="@string/cheeky_checkbox"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"/>
        <TextView
            android:id="@+id/textview"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:paddingTop="8dp"
            android:paddingStart="8dp"
            app:layout_constraintTop_toBottomOf="@id/cheeky_checkbox"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"/>
    </android.support.constraint.ConstraintLayout>
</layout>
```
Let us first define our string resource:
```xml
<string name="is_it_checked">Is it checked? %1$s</string>
```
where `%1$s` would be the checked state of the `CheckBox`.

All that is left is to use this string in our layout file! We can use two data binding features: [String formatting](https://developer.android.com/topic/libraries/data-binding/expressions#resources), and ID referencing.
```xml
android:text="@{@string/is_it_checked(cheekyCheckbox.checked)}"
```

Yes, my friend -- we can reference other views in our data binding expressions via their ID! **Note that data binding converts IdsToCamelCase, so be sure to follow the convention.**

Happy binding!