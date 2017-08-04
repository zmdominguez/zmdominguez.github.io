---
layout: post
title: "I Like Walls"
tags:
- android
- instant apps
---
I have always been told I'm stubborn. And I am. So jumping off this [post]({{ site.baseurl }}{% link _posts/2017-07-28-ia-migration.md %}), I continued doing instant apps stuff and I learnt more things this week.

- More than I figured last week, data binding _is_ broken. As in broken broken. Do not use AS3.0 Canary 9 if you want to look at multi-feature instant apps and your data binding files will __NOT__ be in the base module.
-  I seemed to have hurt AAPT2's feelings. It seems to have a vendetta against me. First it was complaining of (insert eyeroll) 9-patch image errors, then it became Manifest errors, and then I gave up and turned it off again. To do that, add `android.enableAapt2=false` to your `gradle.properties` file.
- BUT! If you are doing the thin feature approach described in my previous post, AAPT1 fails if your instant app has multiple features. Remember that for multi-feature instant apps, common resources should be in a `base feature` module. That means your library which your feature depends on (which in turn depends on the `base feature`) does not have access to any of the things declared in `values.xml` and such. I [filed a bug for this](https://issuetracker.google.com/issues/64371522), please star!
- There are a lot of improvements in AS3.0, but there are also some caveats. To test an instant app in the emulator, you *have* to run it from Studio (`adb install` for an instant app *DOES NOT* work -- AS will install it as a full app). You can (should?) use `adb install --ephemeral FEATURE_APK1 FEATURE_APK2 FEATURE_APK3` on an O device (but there is a [bug on O emulators](https://issuetracker.google.com/issues/64100568) so YMMV)
- Set your run configuration to launch a URL. If AS won't let you change the default launch options, delete the configuration and make a new one
- Signing configs need to be in all feature modules. Once that's defined, absolutely do not forget to reference it in your build types!!

I haven't figured out everything, so it is very likely that I will have more of things to say!