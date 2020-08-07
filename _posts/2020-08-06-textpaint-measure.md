---
layout: post
title: "Accurate Measurements With getTextBounds()"
tags:
    - android
---
We have a few custom [spans](https://developer.android.com/reference/android/text/style/package-summary) in our app and over the last few days I have been poring over one of them. I was trying to see if the implementation could be improved but before that could happen I needed to understand what it was trying to do first.

This particular span, among other things, deals with drawing some text on a `Canvas`. We provide some styling information -- text size, the text colour, and font -- and in the process of drawing there's a lot of measurements and maths.
<center>
    <a href="https://imgur.com/zQ5efS9"><img src="https://i.imgur.com/zQ5efS9.jpg?1" title="source: imgur.com" /></a>
    <br /> <small>Me</small>
</center>

We use this span when we want to display a bunch of text where different parts may have differing font sizes. Something similar to this:
<center>
    <a href="https://imgur.com/czZpKB5"><img src="https://i.imgur.com/czZpKB5.jpg" title="source: imgur.com" /></a>
</center>

Under normal circumstances, we could use [`RelativeSizeSpan`](https://developer.android.com/reference/android/text/style/RelativeSizeSpan) to set the font size. Doing that however, both words would be drawn referencing the same baseline (more on that later):
<center>
    <a href="https://imgur.com/3anytpr"><img src="https://i.imgur.com/3anytpr.jpg" title="source: imgur.com" /></a>
</center>

We want the word with the smaller font size to be centred vertically in relation to the rest of the text, so we need to figure out _where_ to draw it. That, then, is the main purpose of our custom span. Unfortunately, our piece of code that's currently doing this is not very well documented and the `TextPaint` documentation is very... sparse.

### Hold your horses

Before we get in too deep about the implementation, here's a quick primer on some of the terms in typography:
<center>
    <a href="https://imgur.com/ORDLUgO"><img src="https://i.imgur.com/ORDLUgO.png" title="source: imgur.com" /></a>
    <br /><small> <a href="https://www.quora.com/What-is-a-baseline-in-typography">Source (Quora)</a> </small>
</center>

In other words, we want to figure out how we should adjust the _baseline_ of the text in our span so we can tell Android where we want it to be drawn. To figure this out, the code calls [`TextPaint.getTextBounds()`](https://developer.android.com/reference/kotlin/android/graphics/Paint#gettextbounds).

```kotlin
open fun getTextBounds(text: String!, start: Int, end: Int, bounds: Rect!): Unit
```

In this post we will talk about what this method does but more importantly about what it _means_.

### And let them go

Official Android documentation says the following:
>Retrieve the text boundary box and store to bounds. Return in bounds (allocated by the caller) the smallest rectangle that encloses all of the characters, with an implied origin at (0,0).

... and it did not mean anything to me. :sweat_smile:

Now that we know some of the key terms in typography, let's look at what this method tells us when we give it a piece of text. Let's take "`NEW`" from the string above, for example.

The method requires several values:
- `text` is the text we want to measure,
- `start` is the position of the character where we should start measuring,
- `end` is position+1 of the character where we should stop measuring,
- `bounds` is where the results of the measurement will be stored

The Javadoc says that the caller (me, Zarah) should allocate the [`Rect`](https://developer.android.com/reference/kotlin/android/graphics/Rect?hl=en) and then give it to `getTextBounds()`:
```kotlin
val textBounds = Rect() // allocate the Rect
val text = "NEW" // text we want to measure
val start = 0 // position of the character to start with (N)
val end = 3 // position+1 of character to end with (W)
textPaint.getTextBounds(text, start, end, textBounds)
```

And now our `textBounds` will contain a bunch of values. Remember that the font and the font size affects how much space we need to draw our text, and this is what I got for a custom font and size of `10sp`:
```
Rect(
    bottom = 0
    left = 2
    right = 66
    top = -20
    )
```

Turned into a photo:
<center>
    <a href="https://imgur.com/Io0xxAT"><img src="https://i.imgur.com/Io0xxAT.jpg" title="source: imgur.com" /></a>
</center>

The `"implied origin at (0,0)"` the Javadoc refers to is the bottom-leftmost corner of the baseline and the values in our `textBounds` are relative to this point (not to be confused with the `Canvas` where `(0, 0)` is the top-leftmost corner). Since our text is all caps, the `bottom` coordinate of our bounding box aligns with the baseline (how convenient). Values in `textBounds` **increase** as you go down and to the right of the origin, and **decrease** as you go the opposite way.

It is important to note here that the text _may not be_ drawn exactly over the origin; there _may be_ a leading space between the first letter and the leftmost edge of our bounding box, just like in our example here. (I don't know enough about typography but from my experiments it looks like this depends on how the font renders each letter).

<center>
    <a href="https://imgur.com/MwvS3Aq"><img src="https://i.imgur.com/MwvS3Aq.jpg" title="source: imgur.com" /></a>
    <br /><small>Convenience, sort of</small>
</center>

Displaying `NEW` is all fine and good, but what if we give it "`Something`"? We now have a mix of lower- and uppercase letters, as well as letters with _ascenders_ (things that go up) and _descenders_ (things that go down -- but in my head I call them tails).

With the same custom font and size, our `textBounds` now look like this:
```
Rect(
    bottom = 6
    left = 1
    right = 133
    top = -20
    )
```

Turned into a photo:
<center>
    <a href="https://imgur.com/pSHYhig"><img src="https://i.imgur.com/pSHYhig.jpg" title="source: imgur.com" /></a>
</center>

Now that we have enough information about how far down from the baseline our descent line is (0 to `bottom`) and how far up from the baseline our ascent line is (0 to `top`) (look at us using typography terms! Go us!), we can do the necessary maths to tell Android where exactly we want our text to be drawn with respect to the `Canvas`.

In our specific case, given our custom font and the font size, we came up with this formula:
```kotlin
val textY = ((bottomOfBg - textBounds.height()) / 2F) + // half of remaining space in the bg unoccupied by text
    (0F - textBounds.top) + // distance from top of text to baseline of text
    (textBounds.bottom / 2F) // half of "tails"
```

<center>
    <a href="https://imgur.com/pO6mYGf"><img src="https://i.imgur.com/pO6mYGf.jpg" title="source: imgur.com" /></a>
    <br /><small>Purple (0,0) is the Canvas origin<br />Green (0,0) is the text origin<br />Grey outline is <tt>textBounds</tt> for each word</small>
</center>

Some notes:
- `bottomOfBg` here is the bottom of the blue box in the image above
- we decided to add `(textBounds.bottom / 2F)` to nudge the text even more to make it more visually pleasing (this is what works for us and the custom font we use, so your mileage might vary)
- `textBounds` can also give the full height of the bounding box, but if you want to know precise vertical distances relative to the baseline, that information is contained within `textBounds` itself

If you want to know more about Android typography, [here](https://proandroiddev.com/android-and-typography-101-5f06722dd611) is an excellent article about it.

---
Many many thanks to [Ataul Munim](https://twitter.com/ataulm?lang=en), [Florina Muntenescu](https://medium.com/@florina.muntenescu), and [Mike Evans](https://twitter.com/m_evans10) for the reviews!
