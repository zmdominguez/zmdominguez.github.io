---
layout: post
title: "'I will just migrate this real quick', said no one ever"
tags:
- android
- instant apps
---
When [Android Studio 3.0 was announced](https://www.youtube.com/watch?v=rHiA66zUv8c) at I/O, I was very keen to try it out. Mostly because of all the cool stuff in it, but also because of more support for instant apps.

We want to continue supporting our instant app, so I thought it'd be a worthy investment to get a headstart and migrate our EAP version to the post-I/O SDK 1.0 version.

Two months later and I am still migrating. 😅

If you are in the same boat as I am (or maybe you are starting out with instant apps?) here are some of the things I have learnt so far:
- Android Studio 3.0 threw its hands up when I opened our pre-v1.0 version of the app. It refuses to acknowledge that there used to be a `com.android.atom` plugin.
- [Butterknife](http://jakewharton.github.io/butterknife/) works with Android applications and Android libraries, but not with `com.android.feature` (henceforth referred to as `feature`) modules.
- To go around the Butterknife issue, you can put all your code in a `com.android.library` module. Your `feature` module would then declare this library as a dependency and nothing else. [Ben](https://twitter.com/benlikestocode) calls this the _thin module_.
- However, if you use data binding in your instant app, the thin module approach will not work by default. And by not work, I mean it will complain of not being able to find an object you use in your data binding files if that object comes from some other module.
If I have this in a data binding file:
```
    <variable name="media" type="com.fairfax.domain.base.model.Media"/>
```
it will complain that:
```
/Users/zarah.dominguez/Android/.../src/main/res/layout/item_gallery.xml
Error:(27, 35) Cannot resolve type for media 
```
because `Media` is in another module called `base`.

- Remember that with the new Android Gradle plugin, dependencies *do not* bubble up automatically. What I opted to do here is to make my `library` dependencies use `api` instead of `implementation`. You can stick with `implementation` but make sure you include the full list of depedencies in your `feature`  module as well.
- Make sure you enable data binding everywhere. As in everywhere. As in all `build.gradle` files in all modules required by your instant app. 
- I am not quite sure yet if it is a bug or an intended behaviour with `feature` modules, but Manifest merger seems to behave differently that with an `application`. To fix the issue of Android Studio complaining that there is no default Activity, move your instant app entry point activity declaration (and all the intent filters it requires) into the `feature` module's `AndroidManifest` file.
- If you want to test using the emulator, create an image that does not have the Play Store. The Pixel should work fine. Remember to log in with a Google Account and to opt in to instant apps!


PS: At one point I gave up on this and shelved the migration for several weeks, hence the "still going on" thing.  

PPS: No idea what I am blabbering on about? I talked to Kaushik and Donn about [instant apps on Fragmented](http://fragmentedpodcast.com/episodes/90/)!
