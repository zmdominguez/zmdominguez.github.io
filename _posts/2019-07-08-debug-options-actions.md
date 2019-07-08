---
layout: post
title: "On-Device Debugging Part IV: Log All The Things!"
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
- [Part III: Inspect, Reset, Repeat](https://zarah.dev/2019/07/01/debug-options-info.html)

(If you are not familiar with [`PreferenceFragmentCompat`](https://developer.android.com/reference/kotlin/androidx/preference/PreferenceFragmentCompat.html), I highly suggest to read about that first before proceeding. You can start with this [AndroidX guide](https://developer.android.com/guide/topics/ui/settings.html) on Settings.)

---

As I mentioned in [part II](2019-06-24-debug-options-timber.md), we leverage Timber a lot. Those stacktrace log screens have been really useful when the app misbehaves, but what about those times when we just want to have checkpoints whilst using the app?

Enter our next set of developer options:
<center><a href="https://imgur.com/asPf70G"><img src="https://i.imgur.com/asPf70G.png" title="source: imgur.com" /></a><br />
<small>More debugging!</small></center>

### Show Debug Toasts

[`Toast`](https://developer.android.com/guide/topics/ui/notifiers/toasts)s are a great way of setting visual checkpoints in an app. They are relatively unobtrusive and easy to create.

Enabling "Show debug Toasts" in our developer settings allows us to surface these checkpoints when we are testing something unpredictable:

<center><a href="https://imgur.com/9us9Y6I"><img src="https://i.imgur.com/9us9Y6I.png?1" title="source: imgur.com" /></a><br />
<small></small></center>

I previously talked about a geofencing feature we had to implement, and we have that feature to thank for spurning the idea for this debugging feature.

In this feature, we want the app to notify the user when they enter a geofence. Testing geofences in the middle of the city can be tricky, and having visual cues that let us know if we have entered or exited a geofence has been super helpful for us.

This function lives in our `DebugExtensions` class:
{% gist 3b4047bd17f3bceda4e7b9bf70c9994c debug_toast.kt %}

(In this case, `SettingsInteractor` takes care of exposing some of the debug options that our main source set needs access to.)

Having a developer option toggle to control these mean we can turn them off when it gets too annoying, and we lower the risk of leaving a debugging Toast when we release to production as well.

### Enable Leak Canary

We have some really old Activities in the app that drive [Leak Canary](https://github.com/square/leakcanary) crazy. Until we get around to cleaning them up, it also drives our QAs crazy. This toggle gives them the power to turn the tool off so they can test in peace.

As a rule, developers keep this toggle on during active development. As for those old Activities, we are in the process of reworking them to make them perform better.

### Show Debug Notification

When enabled, the app shows a persistent notification that serves as the shortcut to this Developer Settings screen (we also provided an entry to this screen in our app's junk drawer aka the "More" menu for debug builds). Pretty straightforward. 

### Logs

When connected to our development computers, we have  a wide array of debugging tools at our disposal. We can use `Timber`, or `System.out`, or `Toast`s, or breakpoints, etc.

But going back to our geofencing feature, all of these options are not really viable to us during testing. Sure we can walk around Surry Hills with a laptop and keep an eye on Logcat whilst debugging. But we really can't expect everyone who is testing the app to do this.

When this option is turned on, we send all the Logcat information from  our app into a text file:
{% gist be341240af8e9637ac6ffd5605665cc4 save_logcat.kt %}

(Note to self: Make the filename more human readable! :thinking:)

Getting those logs out of a particular device can be fiddly, dealing with multiple OEMs and USB variations. And those logs are pretty useless just sitting there, so we put in the option to send those files to us:
{% gist be341240af8e9637ac6ffd5605665cc4 send_files.kt %}

Using [`ACTION_SEND_MULTIPLE`](https://developer.android.com/training/sharing/send#send-multiple-content) with the Intent allows us to automatically attach all the log files we can find in the user's debug file storage. 

<center><a href="https://imgur.com/VOIY4yq"><img src="https://i.imgur.com/VOIY4yq.png?1" title="source: imgur.com" /></a><br/>
<small>Gimme those logs!</small></center>

Depending on the email client they choose to send the files with, they can remove or add more files as they please.

To be good citizens, we have also provided the option to clear all existing logs so that users can reclaim their precious storage space.

Out of all the debugging features that we have, this one is probably my favourite. :green_heart:

(Massive thanks go out to [@ataul](https://twitter.com/ataulm) for all his patience reviewing my crazy ramblings :laughing: )
---

In the next post, we will look at how our developer settings help us work better with the designers in our team. Stay tuned! :radio:

