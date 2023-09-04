---
layout: post
title: That damn seekbar thumb
date: '2011-02-02T21:18:00.006+11:00'
author: Zarah Dominguez
tags:
  - thumboffset
  - seekbar
  - android
  - slider
modified_time: '2011-02-02T22:04:11.484+11:00'
thumbnail: http://2.bp.blogspot.com/_p3DeelS9Gp8/TUk2WORSilI/AAAAAAAAA3g/BRRe0YagQhU/s72-c/seekbar_no_offset.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8963045768492659748
blogger_orig_url: http://www.zdominguez.com/2011/02/that-damn-seekbar-thumb.html
---

If you have ever needed to use a `SeekBar`, you definitely would have noticed how hard it is to move the slider (aka thumb) when it is set to the minimum or maximum value. The slider tends to be cut in half, and fitting your finger into it to press it becomes a test of patience.

<div><a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://2.bp.blogspot.com/_p3DeelS9Gp8/TUk2WORSilI/AAAAAAAAA3g/BRRe0YagQhU/s1600/seekbar_no_offset.png"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 120px; height: 200px;" src="http://2.bp.blogspot.com/_p3DeelS9Gp8/TUk2WORSilI/AAAAAAAAA3g/BRRe0YagQhU/s200/seekbar_no_offset.png" border="0" alt="" id="BLOGGER_PHOTO_ID_5569042169635965522" /></a></div>

See how small the slider becomes when it reaches the far ends of the `SeekBar`? Crazy!

Luckily, I found a way (just today!) to move the slider just a little tiny bit to make it easier to press. Apparently, there is a method called [`setThumbOffset()`](http://developer.android.com/reference/android/widget/AbsSeekBar.html#setThumbOffset(int)) that allows us to nudge the slider by a number of pixels.

It's pretty easy to use, aside from the fact that it accepts pixels and not dip measurements. Anyway, here's how to do it:

```java
int pixels = convertDipToPixels(8f);
SeekBar mySeekBar = (SeekBar) findViewById(R.id.quiz_settings_seekbar;
mySeekBar.setOnSeekBarChangeListener(mySeekBarListener);
mySeekBarsetThumbOffset(pixels);
```

I convert dip measurements to pixels to better manage the growing number of resolutions of screen sizes present. Here's the code to do that:

```java
private int convertDipToPixels(float dip) {
    DisplayMetrics metrics = new DisplayMetrics();
    getWindowManager().getDefaultDisplay().getMetrics(metrics);
    float density = metrics.density;
    return (int)(dip * density);
}
```

Aaaaaaaand this is now how our slider looks:
<a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://2.bp.blogspot.com/_p3DeelS9Gp8/TUk2WK_am4I/AAAAAAAAA3o/pS5bd37Btpk/s1600/seekbar_with_offset.png"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 120px; height: 200px;" src="http://2.bp.blogspot.com/_p3DeelS9Gp8/TUk2WK_am4I/AAAAAAAAA3o/pS5bd37Btpk/s200/seekbar_with_offset.png" border="0" alt="" id="BLOGGER_PHOTO_ID_5569042168755690370" /></a>

Applause! Confetti! Applause!