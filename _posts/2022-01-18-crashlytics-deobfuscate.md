---
layout: post
title: "📣 PSA: Disabling mapping file uploads with Crashlytics"
tags:
    - firebase
    - crashlytics
---

One of the more famous crash reporting tools used in Android development is probably [Crashlytics](https://firebase.google.com/docs/crashlytics). It offers up a lot of insight into an app's performance -- from device characteristics to insights on issue commonalities. If, like my current project, [obfuscation](https://developer.android.com/studio/build/shrink-code#obfuscate) is enabled in an app, Crashlytics has a Gradle plugin that uploads the mapping file so that we end up with readable crash reports.

There are times though where I may not want to upload the mapping file, like if I am just debugging an issue locally. According to the [official documentation](https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=android
), the way to do this is to add this configuration to the `build.gradle.kts` file:
```
firebaseCrashlytics {
    mappingFileUploadEnabled = false
}
```

However, when I tried it with v2.8.1 of the Crashlytics Gradle plugin, I get an unresolved reference error:
<center>
    <a href="https://imgur.com/GXTHLXx"><img src="https://i.imgur.com/GXTHLXx.png" title="source: imgur.com" width="450" /></a><br />
    <small>Unresolved reference error</small>
</center>

The correct syntax for Kotlin Gradle DSL is somewhat buried in a comment in this [Github issue](https://github.com/firebase/firebase-android-sdk/issues/2665):
```
import com.google.firebase.crashlytics.buildtools.gradle.CrashlyticsExtension

buildTypes{
    getByName("debug") {
        ...
        configure<CrashlyticsExtension> {
            mappingFileUploadEnabled = false 
        }
    }
}
```

It's unfortunate that the official documentation hasn't been updated in the eight months since this issue has been known (apparently it appeared in v2.6.0 of the plugin), but such is life. :crying_cat_face: