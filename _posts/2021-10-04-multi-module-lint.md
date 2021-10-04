---
layout: post
title: "Multi-module Lint Rules 🤹‍♀️"
tags:
    - android, lint
---

I have been learning a LOT about Lint the past year. Our team has grown 5x since I joined more than three years ago, and it became really obvious really quickly that we should be letting robots do a lot of the mundane and repetitive enforcement of our team's code conventions.

At some point, we also started fixing up our app's theming and styling to be adhere to our design system. There was also a design shift where new screens being built have a cleaner, sleeker look. Our app is _not small_ (there are 100+ `Activity` declarations in 70+ modules), so redoing all of the screens will understandably take time and a lot of effort.

### The problem 🤨

Since a new colour palette was introduced by the design team, the first step was to move all of the old, not-to-be-used-anymore colours into a separate file. I named the file `colours_deprecated.xml` and left a comment on top of the file.
```xml
<!--  ************************************
      *   LEGACY COLOURS - DO NOT USE    *
      *   Consult Figma for new colours  *
      ************************************ -->
```

This _may_ be helpful, but really, who clicks into a colour definition to see the containing file? Sure, each of the colours in there could have been renamed to append `_deprecated` to each and every one of them, but that is a lot of effort that will touch a large number files, and needless to say everyone found something else more interesting to do. :speak_no_evil:

So our problem remains: how do we prevent people from using these deprecated colours?

### First stab 🗡️

The answer of course, is to write a Lint rule. [Alex Lockwood](https://twitter.com/alexjlockwood) [open sourced a rule](https://github.com/alexjlockwood/android-lint-checks-demo/blob/master/checks/src/main/java/com/lyft/android/lint/checks/DeprecatedPurpleColorXmlDetector.kt) that flags usages of a specific colour in XML files, and that was a good starting off point for us. Instead of checking just a specific colour like he does, we check against a list of all the colours in the aforementioned `colours_deprecated.xml` file.

```kotlin
val DEPRECATED_COLOURS = listOf("light_gray_background", "red_error", smoke_gray", "subhead_text_color_dark", ....)
```

It was good enough for a few months, but as more and more people finish uplifting screens to the new designs, it's easy to forget to update the Lint rule to also remove colours from the list when their definitions are deleted from the file (and also to _add_ colours to the list if needed).

I personally don't want to be babysitting this list -- surely there's a way to do this better? Back to the books!

### You're gonna need a bigger boat 🛶

The problem sounds simple enough: Gather all the colour resources in files named with the `_deprecated` suffix, and flag any usages of any of those colours as an error.

I initially tried leveraging [`requestRepeat`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Context.kt;l=507;drc=d4b31d1f42ea023078516819b35c7b54a79cf71d) -- do a pass and gather all the resources, request for another pass and then check all the usages. However, this does not seem to do what I want, [as the API guide helpfully points out](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#writingalintcheck:basics/scannerorder):
> Note however that this repeat is only valid within the current module; you can't re-run the analysis through the whole dependency graph

This means that `requestRepeat` is not really that helpful since some colours might be defined in one module and then consumed in a separate module. What we want to do with our check is close enough to what the [`UnusedResourceDetector`](https://cs.android.com/android/_/android/platform/tools/base/+/d1e1b12f309057515188763da1b2a55562b8f7cf:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/UnusedResourceDetector.java) framework check does, so let's look into that for inspiration. It is a LOT of code and it looks stupendously complicated to me, but let's ignore as much as we can for now and look just for the _flow_ of information and what hooks it uses to do its thing.

What stands out is their use of [`checkPartialResults`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Detector.kt;l=722;drc=d4b31d1f42ea023078516819b35c7b54a79cf71d) (which seems to be new in AGP7?). The API guide has [a whole section on partial analysis](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#partialanalysis), but in all honesty, it doesn't really jive with me (I feel like it glosses over some of the basic concepts of what "partial analysis" means for Lint in words that are way too advanced for me).

### Working Concepts 🧂

From what I can gather, as of AGP7, Lint can analyse all the modules of your project in parallel (side note: not to be confusing, but in Lint world, what we usually call a "module" is called a "project"). In cases like ours where we need to cross check information across different modules, we need to wait until all the modules have been analysed before we can figure out if there is an error we are interested in.

So if we need to wait until the very end, we need to store some intermediate information from each module that will help us do our final analysis. _This_ is when "partial results" come into the picture.

What this means for us is that for each of our modules, we need to:
1. gather all resources that are marked as `deprecated`
2. gather all _usages_ of any colour reference

At the end of analysis, we:
- merge together all the (1)s from all the modules
- merge together all the (2)s from all the modules
- check for usages of any (1) in the (2)


### Gather all resources marked as deprecated




For each module that Lint analyses, Lint will call [`afterCheckEachProject`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Detector.kt;l=212;drc=d4b31d1f42ea023078516819b35c7b54a79cf71d)

https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#partialanalysis/modulelintmaps

---

I previously wrote about [getting started with Lint](https://zarah.dev/2020/11/18/todo-lint.html), [writing your own Detector](https://zarah.dev/2020/11/19/todo-detector.html), and of course [writing tests for Detectors](https://zarah.dev/2020/11/20/todo-test.html). But in July 2021, AGP 7.0 was released and with it comes a [whole bunch of Lint changes](https://developer.android.com/studio/releases/gradle-plugin?hl=lt#lint-behavior-changes). I haven't found a definitive, comprehensive, or official post from Google on what has changed since AGP7 but there seems to be a LOT.

Everthing in this post as well as in the sample project have been cobbled together from different sources such as:
- [Lint API Guide](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html)
- [Lint User Guide](http://googlesamples.github.io/android-custom-lint-rules/user-guide.html)
- [Lint sample project](https://github.com/googlesamples/android-custom-lint-rules)
- [Framework rules](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/)
