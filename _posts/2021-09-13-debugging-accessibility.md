---
layout: post
title: "Debugging Accessibility 🔍"
tags:
    - a11y, accessibility
---

One of the things we should be doing as Android developers is to ensure that our apps are as accessible as possible. There are a bunch of talks and articles that discuss the [motivations behind current MDC a11y support](https://youtu.be/nTNwZXVRGdY), the [basic steps](https://youtu.be/bTodlNvQGfY) to [support a11y](https://youtu.be/1by5J7c5Vz4), [testing overviews](https://developer.android.com/guide/topics/ui/accessibility/testing), even [creating your own a11y service](https://developer.android.com/guide/topics/ui/accessibility/service)!

There's a lot of resources that tell me what I should do, but what I found sorely lacking is information on helping me figure out what to do when something goes wrong (and knowing me, something is always bound to go wrong).

For example, this is the upper part of my app's homepage.
<center>
    <a href="https://imgur.com/xvM4iA2"><img src="https://i.imgur.com/xvM4iA2.png" title="Partial screenshot with a cart icon highlighted" width="256" /></a>
    <small>Talkback says "2 items in cart"</small>
</center>

When the cart action menu item is focused, we expect Talkback to announce how many items are currently in the cart. However I noticed that sometimes Talkback just out of the blue says the number and just the number. Weird!

If only I could dive into what Talkback "sees" so I could figure out how to fix the problem and make our Talkback announcements less confusing. I haven't found any mention of how to do this in the official Android docs, and it is by sheer luck that I stumbled [upon this :sparkles: amazing :sparkles: article](https://withintent.uncorkedstudios.com/tutorial-debugging-android-accessibility-818cfd361414) by [Midori Bowen](https://medium.com/@midori.bowen) from 2018(!).

### Wait, what! :heart_eyes_cat:

It turns out that deep in the bowels of Talkback's developer settings is an option to "Enable node tree debugging". Midori links to the Android documentation on enabling this setting but that page has since been deleted. :crying_cat_face:

<center>
    <a href="https://imgur.com/1g9EfIG"><img src="https://i.imgur.com/1g9EfIG.png" title="Screenshot of a screen showing a grid of items" width="256"/></a>
    <small>Turn it on! (While you're there, turn on "Display speech output" as well if you prefer. This will put up a `Toast` of the Talkback announcements)</small>
</center>

The "node tree" being referred to here is basically how Talkback interprets your view hierarchy. Having visibility on this would surely give us a lot of insight into what is going on under the hood.

Some things have changed in Android and in Talkback a bit since Midori's post, but in general the steps in there should give you an idea of how to enable logging. For instance, instead of looking for "Unassigned", assignable gestures are now subtitled "Tap to assign". On some devices, Talkback allows multi-finger gestures, so there's a lot of options to use to trigger node tree log dumps.

<center>
    <a href="https://imgur.com/ZyblUEq"><img src="https://i.imgur.com/ZyblUEq.png" title="Screenshots showing how to change Talkback gestures" width="520"/></a>
    <small>I settled on "Tap with 3 fingers"</small>
</center>

### What can we learn? :thinking:

We can now trigger a dump of the node tree on any screen by using the gesture we have set in Talkback.

<center>
    <a href="https://imgur.com/XUMTJdg"><img src="https://i.imgur.com/XUMTJdg.png" title="Screenshot showing Talkback debugging gesture" width="256"/></a>
    <small>Talkback will tell you it has been done</small>
</center>

At this point I want to reiterate to please do not be like me and spend an hour looking for where the logs actually are (I forgot that I have Logcat filters on :woman_facepalming:). They _are_ in Logcat, with the tag `TreeDebug`.

Here's the partial output of the node tree (timestamps remove for verbosity):
{% gist a30af9e8b5b21e89f5ef3df729aa3aeb recyclerview_node_tree_less_verbose.txt %}

The first few lines (lines 2-9) pertain to the status bar stuff, so let's just ignore that. Our application's contents start at line 10 (`type=TYPE_APPLICATION`) with all the views on the screen in the following lines. Each `ViewGroup` is tabbed which is really helpful in figuring out how each node maps to the view hierarchy. There's a lot of information here and some things have changed since Midori's post, so I thought it would be good to review what we can see in the logs. Let's take line 18 for example:
```
(1100966)652.Switch:(668, 225 - 800, 357):CONTENT{See only Specials}:STATE{OFF}:not checked(action:FOCUS/A11Y_FOCUS/CLICK):focusable:clickable
```

Content |
--- | ---
`(1100966)`|The node's hashcode (which the [Talkback source code](https://github.com/google/talkback/blob/f5d564fdc915a74d8cde4868608f307de9ccf957/utils/src/main/java/com/google/android/accessibility/utils/TreeDebug.java#L75) refers to as a "poor man's ID")
`652`|The window ID
`Switch`|The node's class name (usually type of widget)
`invisible`|This is not shown in logs but is appended only if the view is invisible
`(668, 225 - 800, 357)`|Coordinates of the view on the screen, (Left, Top - Right, Bottom)
`TEXT{xxx}`|Text that's visible to the user (this `Switch` is unlabeled so this does not appear in the logs)
`CONTENT{See only Specials}`|The content description provided by the widget
`STATE{OFF}`|If a widget is stateful, such as this `Switch`, the current state
`(action:FOCUS/A11Y_FOCUS/CLICK)`|Actions available on the node, as defined by [`AccessibilityNodeInfoCompat`](AccessibilityNodeInfoCompat)
`:focusable:clickable`|All other properties of the widget follow, delimited by `:`. Possible values, in the order that they may appear are `focusable`, `screenReaderfocusable`, `focused`, `selected`, `scrollable`, `clickable`, `longClickable`, `accessibilityFocused`, `supportsTextLocation`, `disabled`
"Collection" information|If things are in a `RecyclerView`

The screen _is_ actually a `RecyclerView`, so let's also take a look at what information we receive:
{% gist d6708fba02585b81f31b8a24040e13af %}

At the end of:

Line|
--|---
1| The `RecyclerView` node itself, we see `:collection:R10C2`. This indicates that this is a collection of views consisting of 10 rows, with two columns in each row. Talkback will announce the collection information the first time an item in the collection is selected. For example, if we tap on the Caramello Koala tile, Talkback will announce all the product information (based on the content description of the `ViewGroup`) plus the location of the tile and the collection information ("Row 1, Column 2, In grid, 10 rows, 2 columns").
2|The item on the top right, we see `:item#r0c0`. This is the row- and column-index (starting at 0) of the item relative to the list.
5|The item on the top left, we see `:item#r0c1`. Talkback will announce this item's location as "Column 2".
9, 10|The `Button` is marked as `:invisible` because it's there, but not visible on the screen.

I was able to glean all this information from the [LogTree file](https://github.com/google/talkback/blob/f5d564fdc915a74d8cde4868608f307de9ccf957/utils/src/main/java/com/google/android/accessibility/utils/TreeDebug.java) in [Talkback's repo](https://github.com/google/talkback).

Note that aside from logging the node tree, Talkback also logs the traversal order which may be useful when trying to figure out the order in which elements on the screen gets focus.


### Fixing our issue... maybe :eyes:

Going back to our original issue, the node tree gives us a clue (the cart menu item is in most screens of my app):
```
(1094239)652.ViewGroup:(948, 77 - 1080, 209):CONTENT{Cart: 2 items in Cart}(action:FOCUS/A11Y_FOCUS/CLICK):focusable:clickable
  (1095200)652.TextView:(1008, 107 - 1023, 140):TEXT{2}(action:A11Y_FOCUS):supportsTextLocation
```

AHA! It looks like both the `ViewGroup` as a whole and the `TextView` itself can have focus (it looks like the `TextView` itself does not have a value for `CONTENT`, which is what Talkback announces though), which _may_ explain why I sometimes hear just the number?

I guess I still don't have a definitive answer, but setting `android:importantForAccessibility="no"` on the `TextView` should not present a problem since enough context is already given to the user when the `ViewGroup` gets focus.

---

I hope that as more and more people become a11y allies that we also get more attention on a11y tooling, and more technical articles focused on supporting a11y beyond the basics. For instance, did you know that we can change what Talkback says to provide more context on actionable content? For example, when selecting this `ViewGroup`:
<center>
    <a href="https://imgur.com/AjZ6aIk"><img src="https://i.imgur.com/AjZ6aIk.png" title="Partial screenshot with highlight" width="256" /></a>
    <small>Double-tapping will bring the user to the edit store or delivery address screen</small>
</center>
Talkback will announce "Double-tap to change" instead of the default "Double-tap to activate". The former gives the user more context about what is expected to happen when they interact with the element. Come to think of it, that's a good idea for our next a11y post! :ok_hand:
