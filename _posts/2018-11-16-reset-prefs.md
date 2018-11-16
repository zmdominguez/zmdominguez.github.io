---
layout: post
title: "Selectively Resetting SharedPreferences"
tags:
- android
- shared preferences
---
I have recently been working on a feature that has a bunch of pre-conditions. Things like the user must be logged in AND the feature flag has to be turned on AND it only appears the first time the user lands on the screen.

Normally I would use the [ADB plugin](https://plugins.jetbrains.com/plugin/7380-adb-idea) to Clear App Data and Restart, but doing so proves to be a bit too tedious in this case. I do not want to reset everything, just _some_ things.

All I really want is to reset the "Has the user seen this screen before?" flag. I was in this same exact situation three years ago, and I wrote up a utility method to include in our dev drawer for handling this. I thought it was time to resurrect it and include it in my current project.  

Massive thanks to [Michael Evans](https://twitter.com/m_evans10) for helping me with the Kotlin-isation!    

This function is meant to be used within a dev or debug drawer. It aims to provide a quick way to reset individual `SharedPreference` files to their default values. It scans your app's `shared_prefs` folder and lists the available files. You can then tick the box for those files you want reset.  

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/reset_prefs/alert_choose_file.png"><img src="{{ site.baseurl }}/assets/reset_prefs/alert_choose_file.png" width="320"></a><br />
<small>Human-readable and neat 👌</small></p>  

Aside from the default file generated via [`PreferenceManager.getDefaultSharedPreferences()`](https://developer.android.com/reference/android/preference/PreferenceManager.html#getDefaultSharedPreferences(android.content.Context)), the system does not really enforce a naming convention. They _do_ recommend [prefixing files with the application ID](https://developer.android.com/training/data-storage/shared-preferences#GetSharedPreferences) though!  

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/reset_prefs/device_explorer.png"><img src="{{ site.baseurl }}/assets/reset_prefs/device_explorer.png"></a><br />
<small>Naming is hard</small></p>  

This function assumes that the `SharedPreferences` files your app owns follow this naming convention: `MY.APPLICATION.ID_my_preference_file`. Of course, there's nothing stopping you from tweaking the current implementation to adapt to your needs!  

For better readability, the application ID, the leading underscore, and the file extension are stripped out in the UI.  

A neat thing to add here is put up a `Toast` or a `Snackbar` to confirm that the files were reset.    

Anyway! Enough babble, enjoy the gist!  

{% gist 04a0fa053a00f45fe535d310656ec50e FUN_reset_preferences.kt %}
