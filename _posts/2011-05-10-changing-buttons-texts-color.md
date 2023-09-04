---
layout: post
title: Changing a button's text color
date: '2011-05-10T22:23:00.006+10:00'
author: Zarah Dominguez
tags:
  - android
  - text color
  - button
modified_time: '2011-05-11T01:28:10.154+10:00'
thumbnail: http://3.bp.blogspot.com/-ae1eGx-s7x4/TclXAFIHHBI/AAAAAAAAA44/OEsrn2U1slc/s72-c/change_button_text_color_before.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-257974079637527666
blogger_orig_url: http://www.zdominguez.com/2011/05/changing-buttons-texts-color.html
---

There are times that when changing a button's background color, we also want to change the text's color. There is a method [`setTextColor(int color)`](http://developer.android.com/reference/android/widget/TextView.html#setTextColor(int)) specifically for this purpose. Seems pretty straightforward enough, but it took me a few tries to get it right the first time I tried using it. Documenting it here so that I wouldn't forget.

Here's a screenshot of my button with its default settings.
<img src="http://3.bp.blogspot.com/-ae1eGx-s7x4/TclXAFIHHBI/AAAAAAAAA44/OEsrn2U1slc/s200/change_button_text_color_before.png" style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 120px; height: 200px;" border="0" alt="" id="BLOGGER_PHOTO_ID_5605106870127107090" />

What I'm trying to do is have the color of the text change to a shade of blue once the button is pressed. But look what I got.

<img src="http://1.bp.blogspot.com/-6y3Os4MxnGI/TclXAfzAeuI/AAAAAAAAA5A/t5_Kor2q1DA/s200/change_button_text_color_wrong.png" style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 120px; height: 200px;" border="0" alt="" id="BLOGGER_PHOTO_ID_5605106877286349538" />

That is no shade of blue! Far from it! I tried changing the alpha value of the `#AARRGGBB` value, but I still end up with the same gray shade. It turns out that you cannot pass a direct `R.*` id to the `setTextColor()` method.

Here's how I finally managed to make it work:
```java
int newColor = getResources().getColor(R.color.button_new_color);
((Button)view).setTextColor(newColor);
```
And tada!

<img src="http://1.bp.blogspot.com/-BSVvhJJ5tsY/TclXAhkXL6I/AAAAAAAAA5I/JV2o54b45tw/s200/change_button_text_color_correct.png" style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 120px; height: 200px;" border="0" alt="" id="BLOGGER_PHOTO_ID_5605106877761793954" />