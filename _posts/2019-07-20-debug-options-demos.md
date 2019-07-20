---
layout: post
title: "On-Device Debugging Part V: Strut Your Stuff"
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
- [Part IV: Log All The Things!](https://zarah.dev/2019/07/08/debug-options-actions.html)

(If you are not familiar with [`PreferenceFragmentCompat`](https://developer.android.com/reference/kotlin/androidx/preference/PreferenceFragmentCompat.html), I highly suggest to read about that first before proceeding. You can start with this [AndroidX guide](https://developer.android.com/guide/topics/ui/settings.html) on Settings.)

---

Working on an app means we are constantly tweaking how things look. Change this shade of green, use this widget instead of that, move this thing to there.

In the last part of our on-device debugging series, we look at some of the ways we showcase how things should look in our app.

<center>
    <a href="https://imgur.com/mOr5Aou"><img src="https://i.imgur.com/mOr5Aou.png?1" title="source: imgur.com" /></a>
    <br />
<small>Design Playground and Shortcuts</small></center>

### Design Playground

A few weeks ago, we went all in and switched our whole app to use the new [Material Design Components theme](https://material.io/design/material-theming/implementing-your-theme.html). (Nick Rout has an [awesome article](https://medium.com/over-engineering/setting-up-a-material-components-theme-for-android-fbf7774da739) showing you more details on how to do this for your app.)

<center>
    <blockquote class="twitter-tweet"><p lang="en" dir="ltr">My team is SMASHING IT. 💪 <a href="https://t.co/YoJmRF1ZLJ">pic.twitter.com/YoJmRF1ZLJ</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/1135432513688465408?ref_src=twsrc%5Etfw">June 3, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>

We fully expected that some widgets might look strange and we might have to tweak some things a bit. Taking inspiration from the [MDC Catalog app](https://github.com/material-components/material-components-android/blob/master/docs/catalog-app.md), we came up with a Theme Showcase page.

This page has all the widgets as well as the custom components we use throughout the app.

<center>
    <a href="https://imgur.com/jfqSBoB"><img src="https://i.imgur.com/jfqSBoB.png?1" title="source: imgur.com" /></a>
    <a href="https://imgur.com/zbzyEd0"><img src="https://i.imgur.com/zbzyEd0.png?1" title="source: imgur.com" /></a><br />
<small>Widget showcase</small></center>

This has been really useful in helping us figure out how things look as we go through adapting MDC. This page has also been handy when we are tweaking something with our design team, and as a reference page for when we are visually testing a new feature.

Since we also have multiple themes in the app, we included a theme switcher that allows us to see how everything looks in each of the themes.

<center><a href="https://imgur.com/pC3qSNY"><img src="https://i.imgur.com/pC3qSNY.png?1" title="source: imgur.com" /></a><br />
<small>Dynamically switch themes</small></center>

This switcher is accessed through the overflow menu. When the user chooses a theme, we save the value to a `SharedPreferences` file.

{% gist 6eaa2788c16988d391f45d8445d561f4 switchable_themes.kt %}

For simplicity, we only save the position of the chosen theme:

{% gist 6eaa2788c16988d391f45d8445d561f4 choose_theme.kt %}

We then need to make sure that we get back the chosen theme:

{% gist 6eaa2788c16988d391f45d8445d561f4 get_theme.xml %}

And set it on our Activity's `onCreate` via `setTheme()`.

### Shortcuts

Some parts of the app are a bit cumbersome to get to.

When we were building our app's order confirmation page, it was getting annoying having to complete an order each time to see how it looks whenever we change something in the layout.

As we saw in my previous post about [app shortcuts](https://zdominguez.com/2019/06/08/static-shortcuts.html), we can put in a `targetPackage` and a `targetClass` in our `Intent`s:

{% gist dc33b53af6aaef3f19f59599eb41ba60 shortcuts.xml %}

Having these shortcuts available helped us skip having to navigate through multiple steps to get to a particular screen.

---

And with that, we wrap up this series on on-device debugging options. We have seen how we can [make feature toggles more user-friendly](https://zarah.dev/2019/06/22/debug-options-toggles.html), how we can [surface  stacktraces](https://zarah.dev/2019/06/24/debug-options-timber.html) for quicker debugging, and how we can [retrieve Logcat traces out of our testers' devices](https://zarah.dev/2019/07/08/debug-options-actions.html).

Thank you for sticking with the series, and massive thanks to everyone who has given me their time helping get these posts through! :green_heart:
