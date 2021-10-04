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
val DEPRECATED_COLOURS = listOf("light_gray_background", "red_error", "smoke_gray", "subhead_text_color_dark", ....)
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


### Gather all resources marked as deprecated 🙅

Note: The following sections assume a working familiarity with how Lint rules are structured. Back in November, I wrote about the [basics of writing a Detector](https://zarah.dev/2020/11/19/todo-detector.html); it might be worth a read so the rest of this post will make sense.

For the Lint rule we want to write, we are concerned with usages of colours in XML files (usages in Java and Kotlin files can be encapsulated in a separate rule). We therefore tell Lint that we are interested in these kinds of resources: 

```kotlin
override fun appliesTo(folderType: ResourceFolderType): Boolean {
    return folderType in listOf(
        ResourceFolderType.LAYOUT,
        ResourceFolderType.DRAWABLE,
        ResourceFolderType.COLOR,
        ResourceFolderType.VALUES
    )
}
```

We are only concerned with the _names_ of deprecated colours, and we need to "remember" any we encounter for the current module being analysed. Saving them in a mutable list would suffice **during module analysis**:
```
private var deprecatedColourNames = mutableListOf<String>()
```

Let's start gathering colours by implementing `visitDocument`:
```kotlin
override fun visitDocument(context: XmlContext, document: Document)
```

In Android, colours are usually (only?) defined in `res/color` or `res/values`:
```kotlin
// Is file in the `color` or `values` folder?
val folderType = context.resourceFolderType

if (folderType !in listOf(ResourceFolderType.COLOR, ResourceFolderType.VALUES)) return
```

We are only concerned with colours in the files we care about:
```kotlin
// Does file name contain `_deprecated`
val isFileDeprecated = context.file.name.contains("_deprecated")
if (!isFileDeprecated) return
```

Selectors or [color state lists](https://developer.android.com/guide/topics/resources/color-list-resource) can be used in most places where plain colours can be used and are referenced using the file name. We need to remember such names.
```kotlin
// If this is a selector, the root node will be `<selector>`
// And the color name is the filename
if (document.documentElement.tagName == SdkConstants.TAG_SELECTOR) {
    deprecatedColourNames.add(context.file.name.substringBefore(SdkConstants.DOT_XML))
    return
}
```

For any other non-selector file, colors are defined like:
```xml
<color name="red_error">#d6163e</color>
```

We need to remember the `name` values of these elements:
```kotlin
// If this is not a selector but a normal resource file,
// Get all `color` tags, anything else will be ignored
val allColorNodes = document.getElementsByTagName(SdkConstants.TAG_COLOR)
allColorNodes.forEach { node ->
    // Get the attribute "name", which in our case will be the colour's name
    val namedNode = node.attributes.getNamedItem(SdkConstants.ATTR_NAME)
    val colorName = namedNode.nodeValue

    // Save that value as a deprecated colour
    deprecatedColourNames.add(colorName)
}
```


### Gather all usages of any colour reference 🎨

Whilst we are gathering deprecated resources, we also need to gather _any_ usage of any colour. So let's tell Lint to call us for any attribute it encounters in an XML file:
```kotlin
override fun getApplicableAttributes(): Collection<String>? {
    // Look at every attribute in a file
    return XmlScannerConstants.ALL
}
```

We also want to inspect elements themselves:
```kotlin
override fun getApplicableElements(): Collection<String> = listOf(
    SdkConstants.TAG_COLOR,
    SdkConstants.TAG_ITEM
)
```
(Like with anything Lint, things can be done in multiple ways. I'm sure these two getters can merged into one but I chose to keep them separate for clarity.)

Let's take attributes first as these are the most common usage of colours:
```kotlin
/**
 * In most cases, a colour will be used as an attribute:
 * ```
 * <TextView android:background="@color/a_deprecated_color">
 * ```
 * or
 * ```
 * <selector xmlns:android="http://schemas.android.com/apk/res/android">
 *     <item android:color="@color/a_deprecated_color" android:state_enabled="false" />
 *     <item android:color="@color/a_deprecated_color" />
 * </selector
 * ```
 *
 * Look for those usages here.
 */
override fun visitAttribute(context: XmlContext, attribute: Attr) {
    // Save the value and location of the XML attribute.
    saveColourUsage(
        attribute.nodeValue,
        location = context.getValueLocation(attribute),
    )
}
```

Colours can also be referenced as values of an element, like in a `style` for example:
```kotlin
/**
 * Colours can also be defined as values of an `item` (i.e, not an attribute value!)
 * so we need to visit the element.
 *
 * For example, a deprecated colour can be used in a theme or a style:
 * ```
 * <style name="Brand.SponsoredSpan">
 *  <item name="textColor">@color/a_deprecated_color</item>
 * </style>
 * ```
 */
override fun visitElement(context: XmlContext, element: Element) {
    // Colors can also be in styles as an `<item>` value
    // Find those cases here
    val tagName = element.tagName
    if (tagName != SdkConstants.TAG_ITEM) return

    // Save the value of this element
    if (element.firstChild == null) return

    val fileContents = context.getContents()
    val colorValue = element.firstChild
    val colorLocationStart = context.parser.getNodeStartOffset(context, colorValue)
    val colorLocationEnd = context.parser.getNodeEndOffset(context, colorValue)

    saveColourUsage(
        element.firstChild.nodeValue,
        location = Location.create(context.file, fileContents, colorLocationStart, colorLocationEnd),
    )
}
```

Remember that each of these colour usages is a potential candidate for an error. We cannot know for sure until we have analysed all modules, thus we pass in two elements to `saveColourUsage` -- the colour value being used, and a [`Location`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Location.kt). The `Location` we provide will be used to show the red squiggly lines for errors when Lint generates the final report.

Let's figure out if this colour usage is something we actually care about:
```kotlin
/**
 * Now that we know where to look, record the colour usage
 */
private fun saveColourUsage(
    value: String,
    location: Location,
) {
    // Attempt to parse the attribute value into a resource url reference.
    // Return immediately if the attribute value is not a resource url reference.
    val resourceUrl = ResourceUrl.parse(value) ?: return

    if (resourceUrl.type != ResourceType.COLOR) {
        // Ignore the attribute value if it isn't a color resource.
        return
    }

    if (resourceUrl.isFramework) {
        // Ignore the attribute value if this is a color resource from the Android framework
        // (i.e. `@android:color/***`).
        return
    }

    storeFoundColourReference(resourceUrl, location)
}
```

Now that we have determined that we _do_ need to remember this colour usage, we need to store it somewhere. `Location` is something very-specific to Lint, and I don't want to serialise/deserialise it myself (I spent quite a bit of time trying to understand how the `UnusedResourceDetector` serialises the things it tracks, but again, that looked to be too complicated for our use-case).

The API guide mentions a class called `LintMap` for persisting data, but barely anything else other than that. The KDoc for `LintMap` has slightly more information and it mentions that it can store several types of data including a `Location`. HAH, just what we need!

Since a colour can be referenced multiple times, we need to report each discrete `Location`. `LintMap` is just a `Map`, so to keep a record of every distinct `Location` I decided to simply concatenate the resource name and the `Location` hashcode delimited by `::`. I imagine that this might also work without concatenating things (a `LintMap` can contain another `LintMap` after all), but since this simple way worked, it's good enough for me:
```kotlin
private var colourUsagesLintMap: LintMap = LintMap()

private fun storeFoundColourReference(resourceUrl: ResourceUrl,
                                      location: Location) {
    // We "remember" this colour usage for analysis later (`checkPartialResults` callback)
    // The key in the `LintMap` must be unique, so we hash the location
    val lintMapKey = "${resourceUrl.name}$COLOUR_NAME_DELIMITER${location.hashCode()}"
    colourUsagesLintMap.put(lintMapKey, location)
}
```

### Save the results of module analysis 💾

Now that we have all the information we need from a module, we need to do the actual persisting so that we can do all the merging for the final analysis.

Going back to the API guide, there is a section called ["Module LintMaps"](

https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#partialanalysis/modulelintmaps) and this is somehow related to `checkPartialResults` (I guess)? The docs are not very clear about this, but what that section distills down to is this: intermediate analysis information ("partial results") can be stored in a [`LintMap`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/LintMap.kt) by calling [`getPartialResults`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Context.kt).


For each module that Lint analyses, Lint will call [`afterCheckEachProject`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Detector.kt;l=212;drc=d4b31d1f42ea023078516819b35c7b54a79cf71d) which sounds like a good place to persist our partial results.
```kotlin
override fun afterCheckEachProject(context: Context) {
    super.afterCheckEachProject(context)

    // Save all the information we have found for this project (aka module)
    // This information will be used later to figure out what needs to be reported
    val allColours = deprecatedColourNames.joinToString(COLOUR_NAME_DELIMITER)
    context.getPartialResults(ISSUE).map().apply {
        put(KEY_COLOUR_NAMES, allColours)
        put(KEY_COLOUR_USAGES, colourUsagesLintMap)
    }
}
```

In this case, `map()` creates a new `LintMap` that we can use to persist our partial results (this confused my tiny brain for quite a while, since it's really very similar to Kotlin's `map`). :flushed:

### Do a final analysis 👩‍🔬

After Lint runs through all the modules, we can do a final analysis of all the partial results that we have saved. There is a callback just for this purpose -- [`checkPartialResults`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Detector.kt;l=722;drc=8190da2fc7ac0b3785ed2353baee243bdf7a4012)

```kotlin
override fun checkPartialResults(context: Context, partialResults: PartialResult) {
    // Aggregate all the recorded colour usages and deprecated colours from all the projects
    val allColourUsagesLintMap = LintMap()
    val allDeprecatedColours = mutableListOf<String>()

    // Each project (aka module) would have its own PartialResults
    // Here we retrieve the values we have saved for each project
    partialResults.forEach { partialResult ->
        val partialResultValue = partialResult.value

        // Merge all the deprecated colours together into one massive list
        val colourName = partialResultValue[KEY_COLOUR_NAMES]
        allDeprecatedColours.addAll(colourName?.split(COLOUR_NAME_DELIMITER).orEmpty())

        // Merge all colour usages together into one massive LintMap
        val colourUsages = partialResultValue.getMap(KEY_COLOUR_USAGES)
        colourUsages?.let { allColourUsagesLintMap.putAll(colourUsages) }
    }

    // There are no deprecated colours, nothing to report
    if (allDeprecatedColours.isEmpty()) return

    // There are no colour usages, nothing to report
    if (allColourUsagesLintMap.isEmpty()) return

    // Check colour usages to see if any are using any of the deprecated colours
    allColourUsagesLintMap.forEach { key ->
        val colourName = key.substringBefore(COLOUR_NAME_DELIMITER)
        if (allDeprecatedColours.contains(colourName)) {
            var location = allColourUsagesLintMap.getLocation(key)

            // If for some reason the location is not available (should be impossible really)
            // Just reference the project
            if (location == null) {
                location = Location.create(context.project.dir)
            }

            val incident = Incident(context)
                .issue(ISSUE)
                .location(location)
                .message("Deprecated colours should not be used")
            context.report(incident)
        }
    }
}
```
---

I previously wrote about [getting started with Lint](https://zarah.dev/2020/11/18/todo-lint.html), [writing your own Detector](https://zarah.dev/2020/11/19/todo-detector.html), and of course [writing tests for Detectors](https://zarah.dev/2020/11/20/todo-test.html). But in July 2021, AGP 7.0 was released and with it comes a [whole bunch of Lint changes](https://developer.android.com/studio/releases/gradle-plugin?hl=lt#lint-behavior-changes). I haven't found a definitive, comprehensive, or official post from Google on what has changed since AGP7 but there seems to be a LOT.

Everthing in this post as well as in the sample project have been cobbled together from different sources such as:
- [Lint API Guide](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html)
- [Lint User Guide](http://googlesamples.github.io/android-custom-lint-rules/user-guide.html)
- [Lint sample project](https://github.com/googlesamples/android-custom-lint-rules)
- [Framework rules](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/)
