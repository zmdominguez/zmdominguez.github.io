---
layout: post
title: "On-Device Debugging Part II: Timbeeeeeeer!"
tags:
- android
---
Over the past year, my team have been steadily building a Developer Options screen for our app. It is a simple [`PreferenceScreen`](https://developer.android.com/reference/androidx/preference/PreferenceScreen.html) available on debug builds that aims to help us:
- figure out what's going on without needing to be attached to a computer
- test various configurations without re-installing
- have a host for various experimentations we are trying to explore

In this series of posts, I will share what these various options are and how we made them.

(If you are not familiar with [`PreferenceFragmentCompat`](https://developer.android.com/reference/kotlin/androidx/preference/PreferenceFragmentCompat.html), I highly suggest to read about that first before proceeding. You can start with this [AndroidX guide](https://developer.android.com/guide/topics/ui/settings.html) on Settings.)

Read Part I (Now It's On, Now It's Off) [here](https://zarah.dev/2019/06/22/debug-options-toggles.html).

---

One of the tools we utilise a lot at Woolworths is [Timber](https://github.com/JakeWharton/timber). We use it not just to help us with debugging during development, but also to figure out problems early during testing as well as to keep an eye on things in production.

Before, when we have a build for testing and things don't go to plan, we get sent a screenshot pretty much like this:
<center>
    <a href="https://imgur.com/Bzleb1m"><img src="https://i.imgur.com/Bzleb1m.png" title="source: imgur.com" /></a><br/>
<small>Uh :scream_cat:</small>
</center>

We try to figure out what they were trying to do before the app crashed, replicate the exact conditions they had prior, and try and try and try. More often than not, we hit the "that shouldn't have happened" wall -- even though it obviously just did. :woman_facepalming:

To help in these situations, we turned to Timber. We log those "We should never get here!!!" scenarios using `Timber.e`. In production, those logs get sent to Fabric via `Crashlytics.log()`. In debug builds, those logs get sent to a crash log screen.

This screen shows the stacktrace, if there's any, and some basic information about the build and the device. We also decided to use this screen to log any uncaught exceptions. At the bottom of the screen is a button to send the crash log through via an [Intent Chooser](https://developer.android.com/training/sharing/send).

<center>
    <a href="https://imgur.com/StEAFvr"><img src="https://i.imgur.com/StEAFvr.png" title="source: imgur.com" /></a><br/>
<small>Crash log screen variations</small>
</center>

To bring this to life, we override `Timber`'s `e`:
```kotlin
class LoggingTree(private val app: Application) : Timber.DebugTree() {

    override fun e(message: String, vararg args: Any) {
        super.e(message, *args)
        printStackTrace(null, message, *args)
    }

    override fun e(t: Throwable, message: String?, vararg args: Any) {
        super.e(t, message, *args)
        printStackTrace(t, message, *args)
    }
```

And `printStackTrace` simply passes on the information to a dedicated `Activity` that would display the error.

We provide the kind of error as an `Intent` extra with a `CrashType`:
```kotlin
enum class LogType(val additionalMessage: String, @ColorRes val background: Int) {
    UNCAUGHT_EXCEPTION("!!! DO NOT IGNORE! THIS WILL CAUSE A CRASH IN PRODUCTION !!!", R.color.error_color),

    LOG("We are monitoring an issue. You may still send this log if you want.", R.color.white);
}
```

The crash screen implementation is pretty straightforward, we retrieve the extras and use data binding to display the message and style the header. (For the curious, I previously wrote about [using resource references in databinding](https://zarah.dev/2016/07/19/using-resource-ids-in-data-binding.html))

Since we use the crash log screen for uncaught exceptions as well, we directly call the `Activity` when something goes wrong. This means that it skips the `Timber` processing but sometimes it's still useful to see the logs in Logcat, so we log the stacktrace there as well:
```kotlin
@SuppressLint("LogNotTimber")
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // ...

    // We still want to display the stack trace in Logcat for uncaught exceptions
    // but are not using Timber.e here lest we get into an infinite loop
    Log.e("FIXME", Log.getStackTraceString(stackTrace))

}
```

The "Send crash" button at the bottom of the screen sends out an `ACTION_SEND` `Intent` so that users can send the information via email, Slack, or whatever other app they choose (I also wrote about that [here](https://zdominguez.com/2017/03/31/sharing-is-caring.html)!).

This may seem like a simple thing, but time and again it has helped us quickly figure out issues in a more meaningful way. Even if those helping us test choose to send a screenshot, there is more relevant information rather than the generic crash dialog. :revolving_hearts:

