---
layout: post
title: "Okay Google, Show Me a Million Errors"
tags:
- android
---
EDIT:
Aaaaaaaand an update literally two minutes after I posted this: I was told that the data binding fix made it to the latest [Android Studio Canary release](https://androidstudio.googleblog.com/2018/12/android-studio-34-canary-7-available.html).

I still opt to keep the config in my gradle file, but should be less necessary now.
/EDIT


Our team has been super busy lately.

Aside from working on two big feature changes, we have been working on merging our AndroidX migration. It is a lot changes!

If you have used data binding before, chances are you have come across this one pervasive issue: when any annotation processor fails, data binding throws up and you end up seeing a million errors in your logs.

We use data binding quite a bit in the app, so when a build fails (and there's a good chance they will!) during this transition period, we see _so many_ errors. So. Many.

There are plans to fix this though!
<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Data Binding users, I have a fix for &quot;Too many errors reported from Data Binding if another annotation processor fails&quot; problem. It is hacky but backwards compatible (with a deprecation).<br>Details here, lmk what you think: <a href="https://t.co/02ZiUHvIOr">https://t.co/02ZiUHvIOr</a></p>&mdash; Yigit Boyar (@yigitboyar) <a href="https://twitter.com/yigitboyar/status/1064914985287991296?ref_src=twsrc%5Etfw">November 20, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

Anyway, we have so many errors that all I see are data binding issues and the real actual issue is invisible. I have no idea where to look first.

BUT! TIL that we have a way of actually seeing more errors!

If you're using Kotlin, Kapt provides some [Java compiler options](http://kotlinlang.org/docs/reference/kapt.html#java-compiler-options):

```gradle
kapt {
    javacOptions {
        // Increase the max count of errors from annotation processors.
        // Default is 100.
        option("-Xmaxerrs", 500)
    }
}
```

And for the [Java variant](https://github.com/google/dagger/issues/306#issuecomment-180283287):
```gradle
gradle.projectsEvaluated {
    tasks.withType(JavaCompile) {
        options.compilerArgs << "-Xmaxerrs" << "500"
    }
}
```

Thank you so much to [@Yigit Boyar](https://twitter.com/yigitboyar) for sharing this tip with me!
