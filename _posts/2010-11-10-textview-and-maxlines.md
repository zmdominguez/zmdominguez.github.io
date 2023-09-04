---
layout: post
title: TextView and MaxLines
date: '2010-11-10T17:06:00.006+11:00'
author: Zarah Dominguez
tags:
  - autoadjust
  - textview
  - maxlines
  - android
modified_time: '2010-11-11T02:38:43.376+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-698371868535551178
blogger_orig_url: http://www.zdominguez.com/2010/11/textview-and-maxlines.html
---

I have a TextView (who doesn't?) and I want to adjust its height automatically, depending on the length of the text it will contain.  Should be easy.  It was, but it took me a couple of minutes to figure it out.

So I want my TextView to be by default one line tall, but be able to expand up to two lines.  My initial set up was to set `lines=1` and `maxLines=2`, but it was making the TextView always two lines.  Not what I wanted!  I went through the documentation again, read each word carefully, and then:

```xml
<TextView android:id="@+id/title" 
          android:layout_height="wrap_content"
          android:layout_width="fill_parent"
          android:ellipsize="end"
          android:maxLines="2"
          android:minLines="1"
          android:text="This is the text" />
```
So it turned out that you have to set both `minLines` **and** `maxLines`. TADA!