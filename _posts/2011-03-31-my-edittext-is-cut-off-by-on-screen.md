---
layout: post
title: My EditText is cut off by the on-screen keyboard!
date: '2011-03-31T13:07:00.007+11:00'
author: Zarah Dominguez
tags:
  - ScrollView
  - EditText
  - android
  - layout
modified_time: '2011-04-04T17:07:21.319+10:00'
thumbnail: http://3.bp.blogspot.com/-JftkETrkB2c/TZlrHJOji5I/AAAAAAAAA4I/cX4jSBJRiGY/s72-c/edittext_hidden.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4451513562320730508
blogger_orig_url: http://www.zdominguez.com/2011/03/my-edittext-is-cut-off-by-on-screen.html
---

With clients demanding left and right that my app should look like an iPhone app, I tend to be unappreciative of the way Android natively handles UI interactions and such.  Notice how the screen automagically scrolls up when you click on an `EditText`? It turns out that in iPhone development, the developer does this manually (indicate how much the view should scroll when the on-screen keyboard appears, then scroll it back down afterwards). HA!

But even magic fails sometimes. Has this ever happened to you?
<a href="http://3.bp.blogspot.com/-JftkETrkB2c/TZlrHJOji5I/AAAAAAAAA4I/cX4jSBJRiGY/s1600/edittext_hidden.png"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 134px; height: 200px;" src="http://3.bp.blogspot.com/-JftkETrkB2c/TZlrHJOji5I/AAAAAAAAA4I/cX4jSBJRiGY/s200/edittext_hidden.png" /></a>
The bottommost `EditText` is cut off. And we don't want that!

So what do we do? Do we programmatically scroll the view up? I don't want to do that! It turns out that we can just wrap the whole view in a `ScrollView` and it will scroll up properly!
<a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://1.bp.blogspot.com/-J2UCMLF_E3k/TZlrk_lKBdI/AAAAAAAAA4Q/c7G321XalBo/s1600/edittext_shown.png"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 134px; height: 200px;" src="http://1.bp.blogspot.com/-J2UCMLF_E3k/TZlrk_lKBdI/AAAAAAAAA4Q/c7G321XalBo/s200/edittext_shown.png" border="0" /></a>

The only downside to this is that you might want to hide the scrollbar when the view moves up to accommodate the on-screen keyboard. And to do that, I was trying to set the android:scrollbars="none" attribute but for one reason or another the scrollbar is still being drawn. To make the scrollbar disappear, we can do it from code as such:
```java
((ScrollView)findViewById(R.id.my_scrollview)).setVerticalScrollBarEnabled(false);
```
And we're done!