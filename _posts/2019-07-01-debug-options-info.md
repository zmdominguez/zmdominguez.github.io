---
layout: post
title: "On-Device Debugging Part III: Inspect, Reset, Repeat"
tags:
- android
---
Over the past year, my team have been steadily building a Developer Options screen for our app. It is a simple [`PreferenceScreen`](https://developer.android.com/reference/androidx/preference/PreferenceScreen.html) available on debug builds that help us:
- figure out what's going on without needing to be attached to a computer
- test various configurations without re-installing
- have a host for various experimentations we are trying to explore

In this series of posts, I will share what these various options are and how we made them.

Read the other posts in this series:
- [Part I: Now It's On, Now It's Off](https://zarah.dev/2019/06/22/debug-options-toggles.html)
- [Part II: Timbeeeeeeer!](https://zarah.dev/2019/06/24/debug-options-timber.html)

(If you are not familiar with [`PreferenceFragmentCompat`](https://developer.android.com/reference/kotlin/androidx/preference/PreferenceFragmentCompat.html), I highly suggest to read about that first before proceeding. You can start with this [AndroidX guide](https://developer.android.com/guide/topics/ui/settings.html) on Settings.)

---

A lot of things affect how the Woolworths app behaves. Sometimes databases get out of sync between production and development environments, there are one-shot screens and tooltips, the UI may look different based on device qualities, or even changes to available options to users based on the type of account they have.

To manage and keep track of all these things, we included a set of actions in our developer options.

<center>
    <a href="https://imgur.com/hyYqCEf"><img src="https://i.imgur.com/hyYqCEf.png?5" title="source: imgur.com" /></a><br/>
<small>Additional actions</small></center>

### Environment Switcher

We have a few different environments that the apps can use. Having a switcher built into the app means we can easily test our app's behaviour on any of the available environments.

<center>
    <a href="https://imgur.com/HvK6quk"><img src="https://i.imgur.com/HvK6quk.png" title="source: imgur.com" /></a><br/>
<small>Switcher confirmation</small>
</center>

For us, switching environments mean revoking access tokens and everything else that come with that. Since this is a destructive action (they would need to log back in), we opted to present a confirmation dialog before proceeding.

Your testers are users too, so UX is important. :wink:

### Inspect and Reset Preferences

Last year, Woolworths have completely [removed single-use plastic bags](https://www.woolworthsgroup.com.au/page/media/Latest_News/single-use-plastic-shopping-bags-gone-for-good-at-woolworths) from all stores nationwide. To help with the transition to reusable bags, we built a feature into the app that allows users to enable bag reminders for their chosen store. We also included a "nudge" feature to assist with discovery -- the app would nudge you three times to set up your reminders, after which it assumes you do not want to set it up.

This is one of the many use cases where we leveraged `SharedPreference`s. This whole feature is based on geofencing around a store, which can be a little finicky. We decided to include a `SharedPreference` inspector so that we can easily answer questions from our testers like "I did not get any set up notifications, I'm pretty sure I have only gotten two?".

Being able to see what current values have been saved in the app allows us to quickly triage issues in the wild.

<center>
<a href="https://imgur.com/rlVuxP7"><img src="https://i.imgur.com/rlVuxP7.png" title="source: imgur.com" /></a><br/>
<small>Remember that override Firebase kill switch?</small>
</center>

I have already written about resetting `SharedPreferences` [here](https://zarah.dev/2018/11/16/reset-prefs.html), and the "inspect" bit follows the same principle as that. Mainly:
> This function assumes that the SharedPreferences files your app owns follow this naming convention: MY.APPLICATION.ID_my_preference_file.

But we do need some modifications to the gist quoted in that post. For one, we can extract out the bit that reads the files:

{% gist 421497ece83c84b80d473d4fcc6e782e get_prefs.kt %}

We need to do a bit of processing because we want to have readable names (again, UX!) as well as the name that Android would need to retrieve the `SharedPreference`. As an example:

--- | ---
File location | `/data/user/0/com.woolworths.debug/shared_prefs/com.woolworths.debug_prefs.xml`
Android-readable name | `com.woolworths.debug_prefs`
Nice name | `debug_prefs`

To keep things simple, we only display a dialog with all the existing values in the `SharedPreference`:
{% gist 421497ece83c84b80d473d4fcc6e782e inspect_prefs.kt %}

This option has helped us a lot debugging our nudges out in the wild. It eliminated the need to be connected to a computer, or to fiddle with file viewers, etc.

### Show Device Info

Sometimes we want the UI to change based on some device qualifications like say the smallest available width. Using the emulator makes it super easy for us to test these visual breakpoints. But for someone who does not have access to the emulator, it can be quite hard to verify the UI changes we expect.

This screen shows some basic information that might affect how our app displays something to the user:

<center>
    <a href="https://imgur.com/Qz5kEDi"><img src="https://i.imgur.com/Qz5kEDi.png?1" title="source: imgur.com" /></a><br/>
<small>Minimum information to debug device-dependent UI</small>
</center>

Everything under the "Build Information" header is retrieved from Android's `Build.VERSION` API or the `BuildConfig`. And for the "Device Information" section, we utilised `windowManager.defaultDisplay.getMetrics(DisplayMetrics())`.

The "Account Information" section contains some basic information about the user that might affect the UI. For example, if "Is legal adult" is `false`, the user will not be able to see any liquor products in the app.

### Show Diagnostic Info

There is a lot more information that affects a user's experience with the Woolworths app other than if they are logged in or not. Inventory is different store-to-store, Rewards and offers may be targetted, legal requirements are different based on the state, among other things.

All of these things may affect the app one way or another, and we use a Diagnostics screen to display all those information.


And with that we wrap up this post on our developer options for UI-affecting variables. In the next post, we will talk about logging, Toasts, and more fun stuff! :confetti_ball:
