---
layout: post
title: Annotating all (or most of) the things
date: '2015-11-02T23:10:00.001+11:00'
author: Zarah Dominguez
tags:
  - android
modified_time: '2015-11-02T23:19:16.270+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-960053846760652194
blogger_orig_url: http://www.zdominguez.com/2015/11/annotating-all-or-most-of-things.html
---

If, like me, you are old and have been developing for Android for a while, you should, like me, appreciate the fact that the backwards compatibility of the OS has come a long way. Sure, they may toy with my feelings from time to time, but we all need a little excitement every now and then.

I have recently decided that I will invest more time into learning how all the tools at an Android developer's disposal can make me code better, faster, cleaner, and less buggy (I initially said "buggier" because I want to rhyme but someone who supposedly does English better complained).

To start with, I have been trying recently to consistently use the [Resource Type annotations](http://tools.android.com/tech-docs/support-annotations). These annotations prevent code like this from exploding:

```java
private void setThingsToTextView(int res1, int res2, int res3, int res4) {
    // do stuff
}
```

This will explode because:
1. Fields are named horrendously 
2. Without reading what the method does, it is so easy to pass the wrong resource ID (I can only assume that it wants resource IDs)

Resource annotations help with reason #2 by letting you and the compiler know just what type of resource is expected. There are a lot of available annotations ([Read the docs](https://developer.android.com/reference/android/support/annotation/package-summary.html)!) but I find that the things I use the most are, well, the things I use the most:

`@StringRes` - expects an `R.string.*`  
`@DrawableRes` - expects an `R.drawable.*`  
`@IdRes` - expects an `R.id.*`  
`@ColorRes` - expects an `R.color.*`

I have [updated my SDK Sandbox](https://github.com/zmdominguez/sdk_sandbox/commit/c34eb061cc8c2488c55eaa445d342c4030ac0afd) project with an Activity to illustrate use of these annotations. <span style="color: red;"><b>FAIR WARNING: IT USES ENUMS.</b></span> If this annoys you, DO NOT click through.

So how do we stop the method above from exploding? Let's fix all the things!

```java
private void setThingsToTextView(@IdRes int textView, @StringRes int introText, @DrawableRes int heroImage, @ColorRes int backgroundColour) {
    // do stuff<
}
```

 Ahhh. Easy. And so much better.