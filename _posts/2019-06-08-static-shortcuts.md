---
layout: post
title: "Shortcuts to Shortcuts"
tags:
- android
---
It has been a few years since I last looked at implementing [app shortcuts](https://developer.android.com/guide/topics/ui/shortcuts), and lately I have been looking at them again. I remember implementing them the first time they were released for Android N, but as with life, things have changed a bit.

In this post I want to share some interesting learnings I had whilst implementing static app shortcuts. (Note: This post assumes the reader is familiar with the [basic requirements of implementing app shortcuts](https://developer.android.com/guide/topics/ui/shortcuts/creating-shortcuts)). As always, [all the code](https://github.com/zmdominguez/sdk_sandbox/tree/feature/app-shortcuts) for this post is in my Sandbox App.

### Android Studio support is... limited.
:point_up: For this exercise, I am using Android Studio 3.5 Beta 4.

This becomes tricky as some of the required attributes _can_ use resource references. The table below shows what those are:

Allowed | Not allowed
--- | ---
`shortcutShortLabel` | `shortcutId`
`shortcutLongLabel` | `Intent` attributes (`targetPackage`, `targetClass`)
`icon` | 

:warning: My non-scientific test shows that on Pixel launchers, `shortcutShortLabel` is used unless `shortcutLongLabel` is 17 characters or less.

### There are changes in design guidelines between APIs 25 and 26
Android O introduced [adaptive icons](https://developer.android.com/guide/practices/ui_guidelines/icon_design_adaptive.html) and if we take into consideration that users can pin shortcuts onto their homescreens, we need to support this format if we want to provide the best possible user experience.

[Nick Butcher](https://medium.com/@crafty) talked about this in detail in his [post on implementing](https://medium.com/androiddevelopers/implementing-adaptive-icons-1e4d1795470e) adaptive icons.

:warning: Using a `VectorDrawable` for the icon foreground as Nick showed, it didn't seem to like using theme references for the fill colour. Using a resource reference is okay.
<img src="https://i.imgur.com/GvfiOMX.png">

Read [this post](https://medium.com/google-design/designing-adaptive-icons-515af294c783) to learn more about designing adaptive icons, and [this document](https://commondatastorage.googleapis.com/androiddevelopers/shareables/design/app-shortcuts-design-guidelines.pdf) for designing app shortcut icons.

###  Mo' package names, mo' files
For each app shortcut we want to support, there needs to be an `Intent` associated with it; with each `Intent` needing a `targetPackage` and a `targetClass`. To wit:
```xml
<intent
    android:action="android.intent.action.VIEW"
    android:targetPackage="com.zdominguez.sdksandbox"
    android:targetClass="com.zdominguez.sdksandbox.MainActivity" />
```
This might present a problem if your app have different package names for each build type. Someone has [written a function](https://stackoverflow.com/a/44840413/395576) to support variables but unfortunately it does not work on more modern versions of the Gradle Plugin.

This means that we would need multiple `shortcuts.xml` files for all build variants that we want to support:  
<img src="https://i.imgur.com/g3M5YgK.png">

###  Passing intent extras is possible, but sometimes a trampoline activity comes in handy
If we need to pass some extras to the `Activity` we are targetting, we can do so via the XML file:
```xml
<intent ...>
    <extra android:name="target_tab" android:value="settings" />
</intent>
```

One important statement that is a bit buried in the documentation for app shortcuts is:
> [W]hen the app is already running, all the existing activities in your app are destroyed when a static shortcut is launched. 

This is when a trampoline `Activity` becomes useful so we can prepare our back stack or even get a handle on some requirements the receiving activity would need (for example, a value to be read from the database). You can read more about trampoline activities [here](https://developer.android.com/guide/topics/ui/shortcuts/managing-shortcuts#trampoline). :curly_loop:


### Handling up navigation
Similar to how we should [handle navigation](https://developer.android.com/training/notify-user/navigation#define_your_apps_activity_hierarchy) when starting an `Activity` from a notification, we need to consider how the app would behave when the user navigates away from our target `Activity`.

Thus, we need to provide multiple `Intent`s to the shortcut and the last one in the list would be what the user initially sees.

```xml
<shortcut ...>
    <intent
        android:action="android.intent.action.VIEW"
        android:targetPackage="com.zdominguez.sdksandbox"
        android:targetClass="com.zdominguez.sdksandbox.MainActivity">
        <extra android:name="target_tab" android:value="settings" />
    </intent>
    <intent
        android:action="android.intent.action.VIEW"
        android:targetPackage="com.zdominguez.sdksandbox"
        android:targetClass="com.zdominguez.sdksandbox.DemoActivity">
    </intent>
</shortcut>
```

It took me a bit longer than I liked to figure out all of this :weary:, so I'm writing it down to have a reference for future me.

:dancers: Make sure to read the rest of the [best practices](https://developer.android.com/guide/topics/ui/shortcuts/best-practices) document to learn more.
